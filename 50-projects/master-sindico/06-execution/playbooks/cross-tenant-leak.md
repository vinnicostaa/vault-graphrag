---
title: "Playbook — Cross-Tenant Data Leak (BLOCKER ABSOLUTO)"
type: playbook
tags: [playbook, incident, master-sindico, ops, security, lgpd, multi-tenant]
severity: P0
trigger: "Detected via cross-tenant test em prod, PgAudit, customer report, audit log"
owner: oncall-rotation
created: 2026-04-24
updated: 2026-04-24
---

# Playbook — Cross-Tenant Data Leak

> **BLOCKER ABSOLUTO**: invariante #1 do master-sindico. Dados do condomínio X **nunca** podem aparecer para o condomínio Y. Violação = LGPD breach + perda de contrato + reputação + risco jurídico.
>
> Threat Model §5 (multi-tenant boundary). ABAC engine + `RequireTenantMatch` + `ICondominiumIDsProvider`. Defesa em profundidade prevista (RLS PostgreSQL — Gap-SEC-003 M2).

---

## 1. Trigger

Vetores de detecção (em ordem de gravidade — qualquer um aciona P0):

1. **Customer report** — síndico/morador relata "vejo dados de outro condomínio" via suporte
2. **PgAudit** alert: query cruzou `tenant_id`/`condominium_id` em sessão sem flag admin
3. **Test cross-tenant** falhou em CI/CD (gate de prod) — bug entrou em prod
4. **Test cross-tenant** falhou em runtime (smoke test pós-deploy)
5. **Audit log review** — `audit_log_entries` mostra acesso a `target_id` fora do `actor_user.condominium_ids`
6. **External report** — pesquisador de segurança / pentest / bug bounty

Dashboards:

- Sentry: tag `bc:*` AND `security.cross_tenant=true`
- Grafana: `Per-BC dashboard / Security` → painel "Cross-tenant ABAC denials" (esperado: > 0 — atacantes legítimos sendo bloqueados; queda abrupta = engine quebrado)

---

## 2. Severidade + impacto

**P0 absoluto** — sempre. Nenhum cenário atenua:

| Cenário | Impacto |
|---|---|
| Read leak (síndico A vê dados de B, mas não modifica) | LGPD breach + perda confiança |
| Write leak (síndico A modifica dados de B) | LGPD breach + fraude potencial + responsabilidade civil |
| Leak via ABAC bypass | Erro de matriz — afeta múltiplos endpoints |
| Leak via query SQL sem `condominium_id` filter | Pode afetar TODOS os tenants |
| Leak via cache poisoning | Pode persistir até TTL expirar (5min cache auth) |

SLO afetado: **identity** SLI-3 "0 autorizações incorretas (cross-tenant)" — qualquer evento queima budget e exige postmortem (observability §3.1).

LGPD: **breach Art. 48** — notificação obrigatória ANPD em 72h + titulares afetados.

---

## 3. Detecção — confirmar antes de agir

```bash
railway connect postgres-prod
```

### 3.1 Confirmar leak via audit_log

```sql
-- ✅ Acessos suspeitos: actor_user em tenant X acessou target em tenant Y
WITH user_tenants AS (
  SELECT u.id AS user_id, ARRAY_AGG(DISTINCT m.condominium_id) AS allowed_tenants
  FROM identity.users u
  LEFT JOIN institutional.memberships m ON m.user_id = u.id AND m.status = 'active'
  GROUP BY u.id
),
target_tenants AS (
  -- mapping de target_id → tenant; depende do tipo
  -- exemplo: announcement target → condominium_id da announcement
  SELECT a.id AS target_id, a.condominium_id AS target_tenant
  FROM institutional.announcements a
  -- UNION ALL ... outros tipos de target
)
SELECT
  ale.id,
  ale.actor_user_id,
  ale.entry_type,
  ale.target_id,
  tt.target_tenant,
  ut.allowed_tenants,
  ale.created_at
FROM cross_domain.audit_log_entries ale
JOIN user_tenants ut ON ut.user_id = ale.actor_user_id
JOIN target_tenants tt ON tt.target_id = ale.target_id
WHERE NOT (tt.target_tenant = ANY(ut.allowed_tenants))
  AND ale.created_at > NOW() - INTERVAL '24 hours'
ORDER BY ale.created_at DESC
LIMIT 100;
-- Linhas retornadas = ACESSOS CROSS-TENANT CONFIRMADOS
```

### 3.2 Reproduzir em staging

```bash
# ✅ Staging — reproduzir endpoint suspeito com user de tenant A → ID de tenant B
SMOKE_TOKEN_A="<token tenant A>"
TENANT_B_ID="<UUID tenant B>"

curl -i https://api-staging.mastersindico.com.br/api/v1/condominiums/$TENANT_B_ID/timeline \
  -H "Authorization: Bearer $SMOKE_TOKEN_A"
# expected: 403 ou 404 (ABAC bloqueia)
# bug confirmado: 200 com dados
```

### 3.3 Estimar escopo

```sql
-- ✅ Quantos titulares afetados? (cross-domain audit)
WITH cross_tenant_events AS (
  -- query do §3.1 acima
  SELECT actor_user_id, target_id, target_tenant, created_at FROM /* ... */
)
SELECT
  COUNT(DISTINCT actor_user_id) AS attacker_users,
  COUNT(DISTINCT target_tenant) AS tenants_affected,
  MIN(created_at) AS first_leak,
  MAX(created_at) AS last_leak,
  COUNT(*) AS total_unauthorized_accesses
FROM cross_tenant_events;
```

---

## 4. Contenção imediata (< 30min) — NÃO COMUNICAR PUBLICAMENTE AINDA

Investigar **primeiro**, comunicar **depois**. Comunicação prematura sem dados completos = pânico desnecessário + dificulta investigação.

### 4.1 Identificar tenants afetados

```sql
-- ✅ Capturar snapshot dos tenants afetados em tabela temp (para comms posterior)
CREATE TABLE IF NOT EXISTS audit.cross_tenant_leak_2026MMDD AS
WITH /* query §3.3 */
SELECT DISTINCT
  target_tenant AS condominium_id,
  ARRAY_AGG(DISTINCT actor_user_id) AS attacker_users,
  MIN(created_at) AS first_leak,
  MAX(created_at) AS last_leak,
  COUNT(*) AS access_count
FROM cross_tenant_events
GROUP BY target_tenant;

-- Inspecionar
SELECT * FROM audit.cross_tenant_leak_2026MMDD;
```

### 4.2 Bloqueio: revogar tokens dos atacantes

Atacantes podem ser usuários legítimos explorando bug (não-malicioso) ou maliciosos. Em ambos casos, revogar sessões previne escalation:

```sql
-- ✅ Invalidar todas sessões dos atacantes
UPDATE identity.sessions
SET invalidated_at = NOW(),
    invalidation_reason = 'cross_tenant_leak_containment'
WHERE user_id = ANY(
  SELECT DISTINCT actor_user_id
  FROM audit.cross_tenant_leak_2026MMDD
)
  AND invalidated_at IS NULL;
```

```bash
# ✅ Limpar cache Redis das auth desses users
railway connect redis-prod
# Dentro do redis-cli — para cada user_id atacante:
DEL ms:auth:<USER_ID>
```

### 4.3 Bloqueio temporário do endpoint vazado

Se feature flag disponível (Sprint 11+) — em M1 não existe; alternativa:

```bash
# ⚠️ M1: deploy hotfix removendo rota OU adicionando guard hardcoded
# Preferência: hotfix em main com guard 403-fail-closed específico ao endpoint
# Exemplo (pseudocódigo do guard):
#   if !user.IsAdmin && !sliceContains(user.CondominiumIDs, ctx.Param("cid")) {
#       return 403
#   }

# Deploy fast-track via Railway
git commit -m "hotfix: cross-tenant guard em <endpoint> [INC-2026MMDD]"
git push origin main
# Railway auto-deploys; monitorar em Grafana
```

Alternativamente — se bug é em query SQL (não ABAC), patch imediato no use case.

### 4.4 Garantir audit detalhado em curso

```sql
-- ✅ Habilitar pgaudit para capturar queries futuras (se já não está)
-- Configurar pre-incident é melhor; em incident ativo é tarde
SHOW shared_preload_libraries;
-- expected: 'pgaudit' presente

-- ✅ Em ausência de pgaudit, ativar log_statement = 'all' temporariamente
-- (CUSTO: log volume alto; só durante investigation)
ALTER SYSTEM SET log_statement = 'all';
SELECT pg_reload_conf();
-- LEMBRE de reverter pós-investigation
```

---

## 5. Investigação (< 2h)

### 5.1 Reproduzir bug em staging

Já feito em §3.2 — agora aprofundar:

- Qual condição dispara o leak? (parâmetro específico, role específica, plan_tier?)
- ABAC bypass ou query bypass?
- Reprodução determinística — script que re-executa e demonstra

### 5.2 Determinar escopo

| Pergunta | Como responder |
|---|---|
| Quantos titulares afetados? | `audit.cross_tenant_leak_*` agregado |
| Que tipo de dado expôs? | Inspeccionar `entry_type` + tabela do `target_id` |
| Quanto tempo a janela aberta? | `MIN(created_at)` na tabela snapshot |
| Foi leitura ou escrita? | Inspeccionar `entry_type` (ex: `*.read` vs `*.update`) |
| Há PII? | Inspeccionar tabela do target — `users` (CPF/email), `companies` (CNPJ), `assemblies` (votos), `videos` (PII visual) |

### 5.3 Determinar causa raiz

Categorias comuns:

| Categoria | Exemplo | Detecção |
|---|---|---|
| **RLS missing** | Tabela sem POLICY (não implementado em M1 — Gap-SEC-003 M2) | `\d+ <table>` em psql; sem `Row Security: enabled` |
| **ABAC rule missing** | `RequireTenantMatch: false` num endpoint que deveria ser `true` | grep `abac_rules.go` por action |
| **Query SQL sem `WHERE condominium_id`** | `WHERE id = $1` sem composto | grep §13.3 backend/security |
| **Cache poisoning** | Cache key sem tenant scope | Inspeccionar Redis key naming |
| **Cross-module event sem revalidate** | Consumer não verifica `event.tenant_id ∈ user.tenants` | code review do handler |
| **Path traversal** | Client manipula UUID na URL | abac engine deve cobrir; verificar |

```bash
# ✅ Grep auditoria de queries vulneráveis
grep -rnE 'WHERE\s+(id|user_id|.*_id)\s*=\s*\$[0-9]' \
  /home/.../backend/internal/modules/*/infrastructure/database/queries/ \
  | grep -v 'condominium_id\|tenant_id\|SKIP-TENANT-CHECK'
# expected: zero matches em endpoints multi-tenant
```

---

## 6. Comunicação obrigatória LGPD

Timeline rigoroso (LGPD Art. 48 + ANPD Res. 15/2024):

```
T+0      → Detecção
T+30min  → Contenção iniciada (§4)
T+1h     → DPO interno notificado (Slack + e-mail)
T+4h     → Análise: quantos titulares, tipo de dado, vetor — finalizada
T+24h    → Plano de notificação aos titulares afetados aprovado
T+72h    → ANPD notificada (formulário Res. 15/2024) + titulares notificados
T+7d     → Postmortem técnico interno
T+30d    → Plano de prevenção documentado + implementado
```

### 6.1 T+1h — DPO interno (Slack + e-mail)

Canal `#ms-dpo-incident` (privado):

```text
[INCIDENT-LGPD] Sev P0 — Cross-tenant data leak
Detectado: <HH:MM UTC> via <vetor>
Status investigação: <em curso | concluída>

Resumo preliminar:
- Tipo de dado: <PII | dados financeiros | dados sensíveis | metadados>
- Titulares afetados: <N estimado>
- Tenants (condomínios): <K>
- Janela: <início_ts> até <fim_ts>
- Leak read | write: <tipo>
- Vetor: <ABAC bypass | RLS missing | query bug | cache>

Contenção:
- Endpoint <X> bloqueado: <sim|não>
- Tokens revogados: <N usuários>
- Bug reproduzido em staging: <sim|não>

Próximas 4h:
- Finalizar análise de escopo
- Preparar lista titulares afetados (DPO + jurídico)
- Plano de notificação rascunhado
```

### 6.2 T+24h — Plano de notificação

Decidir canal de notificação (LGPD Art. 48 — "comunicação aos titulares"):

- **E-mail registrado** (canal preferencial — não pop-up; precisa ser comprovável)
- Mensagem in-app secundária (reforço)
- Em casos massivos > 10.000 titulares: comunicação pública (site, redes oficiais)

Plano deve incluir:

- Quem notifica (DPO assina)
- Quando (data/hora)
- Como (template oficial — §6.4)
- Lista de titulares (tabela `audit.cross_tenant_leak_*`)
- Acompanhamento (FAQ, canal dúvidas)

### 6.3 T+72h — ANPD via formulário Res. 15/2024

**Não é opcional**. Formulário oficial: <https://www.gov.br/anpd/pt-br/canais_atendimento/agente-de-tratamento/comunicado-de-incidente-de-seguranca-cis>

Template (campos obrigatórios):

```
1. Identificação do controlador
   Nome: Master Síndico LTDA
   CNPJ: <CNPJ>
   DPO: <nome> — dpo@mastersindico.com.br

2. Descrição do incidente
   Natureza: vazamento por falha de controle de acesso multi-tenant
   Data ocorrência: <DD/MM/YYYY HH:MM>
   Data conhecimento: <DD/MM/YYYY HH:MM>
   Duração janela: <X horas>

3. Dados afetados
   Categorias: <CPF | e-mail | endereço | CNPJ | dados de votação | etc>
   Volume estimado de titulares: <N>
   Categorias de titulares: condôminos (moradores e síndicos), empresas

4. Possíveis consequências
   - Exposição de dados pessoais a terceiros (outros condomínios)
   - Risco de fraude condominial (votação, deliberações)
   - Risco reputacional para titulares

5. Medidas técnicas e administrativas
   - Contenção: bug isolado e patcheado em <data>
   - Revogação de sessões dos usuários envolvidos
   - Implementação de controle adicional (RLS PostgreSQL — Gap-SEC-003)
   - Revisão de matriz ABAC e queries SQL

6. Comunicação aos titulares
   Realizada em <data> via e-mail registrado
   Canal de dúvidas: dpo@mastersindico.com.br
```

### 6.4 T+72h — Titulares afetados (template, sem juridiquês)

```text
Assunto: [Master Síndico] Comunicado importante sobre seus dados

Prezado(a) <Nome>,

Em <DD/MM/YYYY>, identificamos uma falha técnica no controle de acesso do
Master Síndico que permitiu, por um período de <X horas>, que alguns dados
do seu condomínio fossem visíveis a usuários de outros condomínios.

O que aconteceu:
- Tipo de dado afetado: <descrever em linguagem simples — ex: "lista de
  comunicados", "atas de assembleia", "informações de unidade">
- Período: <início> até <fim>
- Causa: falha em controle de isolamento entre condomínios

O que fizemos:
- Corrigimos a falha imediatamente em <data>
- Revogamos sessões dos usuários envolvidos
- Implementamos controles adicionais para impedir recorrência
- Notificamos a ANPD (Autoridade Nacional de Proteção de Dados) conforme
  exige a LGPD

Seus direitos (LGPD):
- Solicitar mais informações: dpo@mastersindico.com.br
- Solicitar exclusão dos dados (Art. 18, VI LGPD)
- Reclamar à ANPD: https://www.gov.br/anpd

Pedimos sinceras desculpas. Estamos comprometidos em proteger seus dados.

DPO Master Síndico
<assinatura>
```

### 6.5 Imprensa (se vazou publicamente — bug bounty público, blog post de pesquisador, etc)

Resposta coordenada Tech Lead + jurídico cliente:

- Comunicado oficial em < 24h pós-publicação externa
- Não negar; reconhecer + comunicar mitigação
- Não atribuir culpa a indivíduos
- Foco no que foi feito para corrigir + prevenir

---

## 7. Recovery

### 7.1 Patch + rollout

**ATENÇÃO: rollback nem sempre é seguro.**

| Situação | Ação |
|---|---|
| Bug em deploy recente, < 24h | **Rollback** (`rollback.md`) — reverte ABAC ou query bug |
| Bug existe há > 24h em prod | **Patch forward** — rollback pode reabrir bug se foi em migration anterior |
| Bug em migration | Forward com migration nova (`expand-contract`); nunca DROP |
| Bug em ABAC rule | Patch + deploy fast-track + smoke test cross-tenant em prod |

### 7.2 Re-testar cross-tenant em prod

Pós-patch, smoke test obrigatório:

```bash
# ✅ Bateria de cross-tenant em prod (read-only)
TOKEN_A="<smoke tenant A>"
TENANT_B="<UUID tenant B>"

# Para cada endpoint multi-tenant em endpoints.md:
ENDPOINTS=(
  "/api/v1/condominiums/$TENANT_B/timeline"
  "/api/v1/condominiums/$TENANT_B/events"
  "/api/v1/condominiums/$TENANT_B/announcements"
  "/api/v1/condominiums/$TENANT_B/master-plan"
  "/api/v1/condominiums/$TENANT_B/dashboard"
  "/api/v1/condominiums/$TENANT_B/assemblies"
  "/api/v1/condominiums/$TENANT_B/proposals"
  # ... 36+ endpoints institutional + 41 commercial + 25 assembly
)

for ep in "${ENDPOINTS[@]}"; do
  CODE=$(curl -s -o /dev/null -w "%{http_code}" "https://api.mastersindico.com.br$ep" \
    -H "Authorization: Bearer $TOKEN_A")
  if [[ "$CODE" != "403" && "$CODE" != "404" ]]; then
    echo "❌ FAIL $ep → $CODE"
  else
    echo "✅ OK $ep → $CODE"
  fi
done
```

### 7.3 PgAudit retroativo — descobrir histórico

Se pgaudit não estava ligado pre-incident (gap):

```sql
-- ✅ Aproximação via audit_log_entries — accesses cross-tenant nos últimos 30d
-- Re-executa query §3.1 com janela WHERE created_at > NOW() - INTERVAL '30 days'

-- ✅ Estimar exfil: queries massivas suspeitas
SELECT
  actor_user_id,
  COUNT(*) AS access_count,
  COUNT(DISTINCT target_id) AS unique_targets
FROM cross_domain.audit_log_entries
WHERE created_at > NOW() - INTERVAL '30 days'
GROUP BY actor_user_id
HAVING COUNT(*) > 1000
ORDER BY access_count DESC;
-- Padrão suspeito: 1 user fazendo > 1000 accesses em < 30d
```

Documentar findings retroativos no postmortem (§8).

---

## 8. Pós-mortem obrigatório (≤ 7d)

Template estendido em [[../../11-audit/postmortems/template]]:

### 8.1 Linha do tempo

```
T+0     → Detecção: <vetor>
T+Xmin  → Contenção iniciada
T+1h    → DPO notificado
T+Yh    → Bug reproduzido em staging; causa raiz identificada
T+Zh    → Patch deploy em prod
T+24h   → Plano notificação aprovado
T+72h   → ANPD notificada + titulares notificados
T+7d    → Postmortem publicado
```

### 8.2 Causa raiz (5 whys)

Exemplo:

```
1. Por que houve leak? → Endpoint X retornou dados de outro tenant.
2. Por que retornou? → Query SQL não tinha filtro condominium_id.
3. Por que não tinha? → Code review não detectou.
4. Por que não detectou? → Não há lint automático grep'ando WHERE sem condominium_id.
5. Por que não há lint? → Sprint 10 prevê (§13.3 backend/security); não chegamos lá.

Fix permanente: implementar lint + executar em CI obrigatório.
```

### 8.3 Fix permanente

- [ ] Lint cross-tenant em CI (grep §13.3 backend/security) — Sprint 10
- [ ] PostgreSQL RLS implementado (Gap-SEC-003) — M2
- [ ] PgAudit habilitado em prod permanentemente — Sprint 10
- [ ] Cross-tenant test gate **bloqueante** em CI (não pode merge sem passar)
- [ ] Smoke test cross-tenant em prod pós-deploy automatizado
- [ ] Treinamento dev: revisão obrigatória de queries multi-tenant

### 8.4 Comms templates pós-mortem

- DPO interno (Slack + e-mail)
- Cliente individual (e-mail registrado §6.4)
- ANPD (formulário Res. 15/2024 §6.3)
- Imprensa (se vazou publicamente §6.5)

---

## 9. Escalação

| Quando | Quem | Como |
|---|---|---|
| Detecção | Tech Lead + DPO | imediato — PagerDuty + Slack `#ms-dpo-incident` |
| T+1h | DPO + jurídico cliente | reunião de status |
| T+24h | CEO + jurídico (cliente master-sindico João) | aprovação plano notificação |
| T+72h | DPO submete ANPD | formulário Res. 15/2024 |
| Imprensa nos contata | CEO + jurídico + comms | no comment até ter posição oficial |
| Bug bounty externo | Security team | reconhecer recebimento + investigar — não dar detalhes premature |

Contatos:

- DPO interno: `dpo@mastersindico.com.br`
- ANPD canal de comunicação: <https://www.gov.br/anpd>
- Jurídico cliente master-sindico: João — contato em `1Password`
- CEO master-sindico: João

---

## 10. LGPD checklist (mandatório — não opcional)

- [ ] T+0 detecção registrada com timestamp UTC
- [ ] T+1h DPO interno notificado (registrar e-mail/Slack)
- [ ] T+4h escopo finalizado: tipo dado + titulares + janela + vetor
- [ ] T+24h plano notificação aprovado pelo DPO + jurídico
- [ ] T+72h ANPD notificada via formulário Res. 15/2024 (anexar comprovante)
- [ ] T+72h titulares afetados notificados (e-mail registrado, comprovável)
- [ ] T+7d postmortem técnico interno publicado
- [ ] T+30d plano prevenção implementado (lint, RLS, test gates)
- [ ] Audit log: snapshot da tabela `audit.cross_tenant_leak_*` preservado em S3 WORM
- [ ] Eventual processo administrativo ANPD acompanhado pelo DPO

**Falha em qualquer T+ → multa LGPD Art. 52 (até 2% faturamento, max R$ 50mi).**

---

## 11. Vizinhos

- [[../incident]] — fluxo geral incident response (LGPD timeline T+1h/T+72h)
- [[../rollback]] — procedimento rollback Railway (cuidado: pode reabrir bug)
- [[../../11-audit/postmortems/template]]
- [[../../03-subprojects/backend/security]] §5 (ABAC), §13 (multi-tenancy row-based)
- [[../../02-architecture/observability]] §3.1 (SLO identity SLI-3 — 0 cross-tenant)
- [[../../08-security/threat-model]] §5 (Tenant boundary STRIDE), §15 Gap-SEC-003 (RLS)
- [[../../01-domain/bounded-contexts|bounded-contexts]] — todos contexts são multi-tenant
- [[stripe-down]] — caso PII vaze nos logs durante incident Stripe
- [[zitadel-down]] — caso PII vaze durante incident Zitadel
- [[mux-down]] — caso vídeo privado vaze cross-tenant
- [[livekit-down-assembly]] — caso voto/gravação vaze cross-tenant
