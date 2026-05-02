---
title: "Connect Me — 4 Direções (Marketplace Unidirecional + LGPD)"
updated: 2026-04-22
maintainer: agent-3 (analista)
type: module
status: stable
source: requirements.md Req 19, 19.1, 19.2, 19.3 + PDFs Empresa, Síndico, Vizinhança
related-spec: backend/.kiro/specs/master-sindico/requirements/commercial.md (Req 19-19.3)
---

# Connect Me — 4 Direções

> **Regra de Ouro #3** (inviolável): **Connect Me ≠ Chat**. Formulário unidirecional, sem reply, sem thread, sem histórico de conversa.
>
> **Regra de Ouro #10** (LGPD): toda revelação de dados pessoais é **logada** (timestamp + IP + finalidade + versão dos termos).

---

## Visão geral

4 fluxos confirmados (Decision 5):

| # | Direção | Quem inicia | Quem recebe | Quotas |
|---|---|---|---|---|
| 1 | **Síndico → Empresa** | Síndico | Empresa | N1: 2/ano · N2: 4/ano · N3: ∞ |
| 2 | **Morador → Síndico** | Morador | Síndico | Base: 2/ano · Pagante: 4/ano (Decision 10) |
| 3 | **Empresa → Empresa** | Empresa Plus/Pro | Empresa | "conforme contrato" (ver Q8 quebras) |
| 4 | **Síndico → Parceiro Vizinhança** | Síndico | Parceiro | Quotas Síndico aplicam |

### Direções **proibidas** (anti-prospecção)
❌ Empresa → Morador (anti-spam)
❌ Empresa → Síndico ativo (Empresa só RECEBE Connect Me, não inicia)
❌ Agência de Marketing → Qualquer (Marketing Link é diferente — registra intenção mas não revela contato)
❌ Parceiro → Qualquer (Parceiro só recebe)

---

## 1. Connect Me Síndico → Empresa (Req 19)

### Fluxo
1. Síndico preenche formulário no perfil da empresa: categoria_servico, descricao_demanda, urgencia, budget_range
2. Empresa recebe notificação "Novo Connect Me"
3. **Dados do síndico OCULTOS** até empresa clicar "Tenho Interesse"
4. Empresa clica "Tenho Interesse" OR "Recusar"
   - **Tenho Interesse**: revela `nome_sindico, telefone, email, descricao_completa_demanda` + log LGPD
   - **Recusar**: 6 motivos estruturados + envia mensagem automática ao síndico
5. Auto-close em 48h sem interesse
6. Síndico recebe notificação de auto-close

### Status do Connect Me (Req 19.9)
```
open | interested | proposal_sent | em_negociacao | closed | expired
```

### Motivos de Recusa (Empresa, 6)
1. Sem disponibilidade operacional
2. Escopo incompatível
3. Capacidade técnica insuficiente
4. Fora da área de atendimento
5. Orçamento incompatível
6. Outro motivo (campo aberto obrigatório)

### Anti-Assédio (Req 19.14)
Botão "Denunciar" em cada Connect Me + ações admin (warning, suspension, ban). Loga `timestamp, company_id, sindico_id, description, evidence`.

### Implementação backend
- Tabela: `connect_me_requests`
- Cache quota: Redis `ms:quota:{userID}:connect_me:{year}` (reset 1º jan)
- Tracking quota: `feature_usages` table
- Evento NATS: `connect_me.request.created` → notif. empresa
- Evento NATS: `connect_me.request.interested` → reveal + log LGPD
- LGPD log: `lgpd_logs` (append-only, retenção 5 anos)

---

## 2. Connect Me Morador → Síndico (Req 19.1)

### Fluxo
1. Morador preenche formulário no app
2. Síndico recebe notificação
3. Síndico responde com mensagem OU denega

### ⚠️ Conflito conceitual (ver Q quebras-detectadas)
- **Decision 9**: "Connect Me Morador→Síndico — formulário unidirecional **frio** (sem reply, sem histórico)"
- **Req 19.1 Acceptance Criteria**: "allow síndico to respond with: message, status_update" + "display request history to morador"

**Decisão pendente** (Decision 11 do requirements.md): Síndico tem reply ou não? Duas opções:
- **A** (consistente com Decision 9 / Regra 3): unidirecional, síndico só marca status (visualizado/em_análise/resolvido), morador vê só status
- **B** (conforme Req 19.1 escrito): ticket-system com asynchronous conversation, histórico, reply do síndico

**Provável**: Opção A para consistência com Regra de Ouro 3.

### Categorias (5)
```
duvida_gestao | solicitacao_manutencao | reclamacao | sugestao | outro
```

### Urgência (4)
```
baixa | media | alta | urgente
```

### Quotas (Decision 10)
- Morador Base: **2/ano**
- Morador Pagante: **4/ano**
- Reset: 1º janeiro (anual)

> ⚠️ **Q2** (CRITICAL): `functional-matrix.md` diverge — diz Base=0, Pagante=2/ano. Decision 10 vence. Ver [[../regras-de-negocio/quebras-detectadas]].

### Implementação
- Tabela: `connect_me_resident_requests`
- Sem tabela de respostas (assimétrico — opção A)

---

## 3. Connect Me Empresa → Empresa (Req 19.2)

### Restrição
**Apenas planos Plus e Pro** (validação de plan_tier). Empresa Trial não pode.

### Tipos de Parceria (5)
```
subcontratacao | parceria_tecnica | fornecimento_materiais | indicacao_servicos | outro
```

### Fluxo (similar Síndico→Empresa)
1. Empresa A preenche formulário
2. Dados de A ocultos até B clicar "Tenho Interesse"
3. B revela contato + LGPD log
4. Auto-close em 48h

### Quota
"Conforme contrato — ilimitado para Pro, configurável para Plus".

> ⚠️ **Q8**: 3 redações diferentes em 3 arquivos (`personas-and-plans` "não aplicável" / `functional-matrix` "E→E apenas" / `commercial.md` "ilimitado/configurável"). Padronizar.

### Implementação
- Tabela: `connect_me_enterprise_requests`

---

## 4. Connect Me Síndico → Parceiro da Vizinhança (Req 19.3)

### Tipos de Promoção (5)
```
desconto_exclusivo | promocao_dia | campanha_sazonal | parceria_continua | outro
```

### Fluxo
1. Síndico cria solicitação para parceiro local
2. Dados do síndico OCULTOS até parceiro clicar "Tenho Interesse"
3. Parceiro revela `nome_sindico, condominio, telefone, email, descricao_completa` + LGPD log
4. Parceiro pode responder com proposta de promoção exclusiva
5. Auto-close em 48h

### Quotas
Aplicam quotas Síndico (N1: 2/ano · N2: 4/ano · N3: ∞).

### Implementação
- Tabelas: `connect_me_neighborhood_requests`, `neighborhood_benefits`
- Tracking conversion rate: requests → proposals → activated_promotions

---

## Marketing Link — NÃO É Connect Me (clarificação)

Marketing Link é o formulário **Empresa → Agência de Marketing** para registrar intenção de contratação de marketing.

### Diferenças de Connect Me
- Marketing Link **NÃO cria vínculo automático** entre empresa e agência
- Vínculo oficial **só com convite via "Gestão de Agência de Marketing"** (E13-E14)
- Não tem regras de quota (não é volume sensitive)
- Não é unidirecional puro — é formulário de intenção de contato

### Tipos de Apoio (6)
```
producao_videos_institucionais | gestao_conteudos_tecnicos | estrategia_posicionamento
gestao_redes_sociais | consultoria_marketing | outro
```

### Prioridade (3)
```
imediato | proximos_30_dias | sem_prazo_definido
```

### Status do Marketing Link (6)
```
sent | viewed | proposal_sent | atendido | accepted | rejected
```

### Implementação
- Tabela: `marketing_link_requests`
- Aparece em painel da agência (MK8)

---

## Regras transversais (todas as direções)

### LGPD obrigatório (Regra de Ouro #10)
- Log na revelação de dados: `timestamp, source_id, target_id, ip_address, terms_version, purpose`
- Retenção: **5 anos** (LGPD)
- Direito ao esquecimento: purga após vencimento
- Tabela: `lgpd_logs` (append-only, integrada com todos os módulos)

### Anti-fraude (Req 102)
- 1 dispositivo ativo por conta (Regra de Ouro #8)
- CAPTCHA após 3 tentativas falhadas
- Bloqueio por IP em múltiplos cadastros < 1h

### Soft-block (Decision 4)
Quando subscription expira: 403 `QUOTA_EXCEEDED` em quotas; 403 `PLAN_REQUIRED` em features; banner "Reativar assinatura" + link Stripe Customer Portal. **Sem grace period.**

### Visibilidade
Anti-prospecção: empresas NÃO listam moradores, agências NÃO acessam Connect Me.

---

## ABAC enforcement (atual)

```go
// Handler: criar Connect Me Síndico → Empresa
requiredAction := "commercial.connect_me.create"
abac.RequirePermission(ctx, requiredAction, "connect_me_request")
// ABAC valida: role=syndic, plan_tier (N1/N2/N3), quota disponível → 403 se não
```

Quotas validadas dentro do use case (não no middleware), com `feature_usages` table + Redis cache. PBT em `feature_usage_pbt_test.go` cobre Consume/Unlimited/ZeroLimit (300 casos rapid ~10ms — F31).

---

## Inconsistências cross-fonte (resumo, ver quebras-detectadas)

| Quebra | Tema | Severidade |
|---|---|---|
| Q2 | Quota Morador Base divergente entre 3 fontes | 🔴 Critical |
| Q8 | E→E descrita em 3 redações diferentes | 🟡 Medium |

---

## Referências

- [[../regras-de-negocio/regras-canonicas]] — Regra de Ouro 3 (Connect Me ≠ Chat) + Decision 5 (4 fluxos) + Decision 9 (M→S frio) + Decision 10 (quotas anuais)
- [[../regras-de-negocio/quebras-detectadas]] Q2, Q8
- [[../02-Jornadas/Inventario-Telas]] — telas S18/S19/S20 (Síndico) e E2/E3/E4 (Empresa) e E16 (Marketing Link)
- `backend/.kiro/specs/master-sindico/requirements/commercial.md` — Req 19, 19.1, 19.2, 19.3 (canônico)
- `backend/.kiro/specs/master-sindico/requirements/identity.md` — Req 102 (Antifraude)
