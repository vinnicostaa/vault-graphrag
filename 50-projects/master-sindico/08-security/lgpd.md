---
title: LGPD — Controles Lei 13.709/2018 — Master Síndico v2
type: spec
tags: [security, lgpd, privacy, compliance, master-sindico, v2]
source: Lei 13.709/2018 + 90-ingested legacy
created: 2026-04-23
updated: 2026-04-23
aliases: [lgpd, lgpd-compliance]
---

# LGPD — Master Síndico v2

Controles concretos da **Lei Geral de Proteção de Dados** (Lei 13.709/2018) aplicados ao master-sindico. Este doc é consultado por DPO, jurídico e auditor externo em due-diligence.

**Fonte normativa**: https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm (Lei 13.709 de 14/08/2018, alterada pela Lei 13.853/2019).

**Autoridade**: ANPD (Autoridade Nacional de Proteção de Dados) — https://www.gov.br/anpd.

**Princípios**: herda LINDDUN em [[threat-model]] §12 + se integra a [[BEYOND_CORP]] §11 (audit trail).

---

## 1. Bases legais aplicáveis (Art. 7º)

A LGPD exige **base legal explícita** para cada tratamento. Mapeamento para master-sindico:

| Base legal (Art. 7º inciso) | Quando usar em master-sindico | Campo DB / Consentimento |
|---|---|---|
| **I — Consentimento** | Criação de conta; opt-in de marketing; vídeo institucional público; vídeo-currículo Banco Talentos; dados de vizinhança (geolocalização) | `user.consents[]` versionados |
| **V — Execução de contrato** | Billing Stripe; Connect Me contratos; ata oficial de assembleia | Contract aggregate + audit |
| **VI — Exercício regular de direitos** | Storage de atas e timeline pra defesa em disputa judicial (LGPD Art. 16 compatível) | Timeline imutável 5y+ |
| **IX — Legítimo interesse** | Prevenção de fraude (device fingerprint, risk scoring); segurança (audit_log); telemetria agregada | Documentado em DPIA |
| **II — Cumprimento de obrigação legal** | Atas de assembleia (Lei 4.591/64); assinatura digital (MP 2.200-2); documentos fiscais (quando aplicável) | Retention legal |

**⚠️ Crítico**: **nunca** usar "legítimo interesse" pra tratar **dados sensíveis** (Art. 11). Sensíveis exigem consentimento específico ou hipóteses do Art. 11 §2º.

**Sensíveis no master-sindico**: geralmente ausentes. Possíveis casos de borda:
- Saúde (se algum benefício/plano envolver) — não no escopo.
- Religião (se afiliação a associações religiosas em comunicado de vizinhança) — evitar.
- Biometria (se passkey/Face ID for armazenado) — **armazenar só no device**; nunca no servidor.

---

## 2. Data classification (sensibilidade)

Toda coluna no schema DB rotulada com classe:

| Classe | Exemplos | Controle obrigatório |
|---|---|---|
| 🔴 **CRITICAL** | CPF, CNPJ (empresa), RG, senha (nunca armazenada aqui), saldo financeiro, dados de cartão (**nunca**, Stripe) | TLS + encryption at rest + mask em logs + audit em toda leitura |
| 🟠 **HIGH** | Email, telefone, endereço completo, geolocalização, IP history, device fingerprint | TLS + encryption at rest + mask em logs |
| 🟡 **NORMAL** | Nome, apelido, foto de perfil, bio pública, preferências | TLS + standard logging (sem mask obrigatório) |
| 🟢 **PUBLIC** | Nome do condomínio (quando público opt-in), vídeos institucionais publicados | Sem restrição |

**Rotulagem**: em `01-domain/data-classification` *(matriz a destilar — classificação de dados pessoais por sensibilidade)* (a escrever) + comentário em sqlc query + tag em VO:

```go
// pkg/cpf/cpf.go
// @data-class: CRITICAL
// @lgpd-article: 7.I + 5.I
type CPF struct { ... }
```

---

## 3. Consentimento versionado (Art. 8º §5º)

### 3.1 Schema

```sql
CREATE TABLE user_consents (
    id                    UUID PRIMARY KEY DEFAULT uuidv7(),
    user_id               UUID NOT NULL REFERENCES users(id),
    consent_type          TEXT NOT NULL,             -- 'terms_of_service' | 'privacy_policy' | 'marketing_emails' | 'location_sharing' | ...
    version               TEXT NOT NULL,             -- '2026-04-23-v1.2'
    granted_at            TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    revoked_at            TIMESTAMPTZ,
    ip                    INET,
    user_agent            TEXT,
    consent_text_hash     TEXT NOT NULL              -- SHA256 do texto legal exato que user leu
);

CREATE UNIQUE INDEX idx_user_consents_active
  ON user_consents(user_id, consent_type)
  WHERE revoked_at IS NULL;
```

### 3.2 Tipos de consentimento

- `terms_of_service` — Termos de Uso.
- `privacy_policy` — Política de Privacidade (LGPD explicit).
- `marketing_emails` — opt-in comunicações promocionais.
- `location_sharing` — Vizinhança (geolocalização).
- `video_public_publish` — vídeo institucional público (empresa) / vídeo-currículo.
- `device_fingerprint` — legítimo interesse, mas **informar** na política (Art. 9º transparência).
- `third_party_sharing` — DPA com Stripe/Mux/Livekit/Zitadel (obrigatório no cadastro).

### 3.3 Fluxo

1. User cadastra → modal com `terms_of_service@2026-04-23` + `privacy_policy@2026-04-23` (checkbox obrigatório).
2. Insert em `user_consents` com `version = current_version` + `consent_text_hash = SHA256(<texto>)`.
3. Se política muda → next login, modal com diff + re-consent; usuário pode recusar (mas funcionalidade limitada ou bloqueio).
4. Revoke: `PATCH /api/v1/me/consents/:type { revoked: true }` → update `revoked_at`; audit_log.

### 3.4 Prova de consentimento

- **Versioning imutável** do texto legal em repo (commit hash em git) + hash SHA256 no DB.
- Retention: 5 anos pós-revogação.

---

## 4. Direitos do titular (Art. 18)

Endpoints REST implementando cada direito:

### 4.1 Direito de acesso (Art. 18 II) — `GET /api/v1/me`

Retorna todos os dados pessoais do titular em formato legível:

```http
GET /api/v1/me
Authorization: Bearer <token>

200 OK
{
  "id": "...",
  "email": "...",
  "name": "...",
  "cpf": "***.***.***-**",   // mask em response default; opt-in revelar via step-up MFA
  "phone": "(11) 9****-****",
  "memberships": [...],
  "consents": [...],
  "devices": [...],
  "created_at": "...",
  "updated_at": "..."
}
```

### 4.2 Direito de portabilidade (Art. 18 V) — `GET /api/v1/me/export`

Retorna **dump completo** em JSON (formato interoperável):

```http
GET /api/v1/me/export
Authorization: Bearer <token>
X-Step-Up-Token: <fresh-mfa-token>

200 OK
Content-Type: application/json
Content-Disposition: attachment; filename="master-sindico-export-<user_id>-2026-04-23.json"

{
  "export_version": "1.0",
  "exported_at": "2026-04-23T10:00:00Z",
  "user": {...},
  "memberships": [...],
  "votes": [...],              // só votos do titular
  "rfps_created": [...],       // Connect Me requests criadas pelo titular
  "proposals_submitted": [...], // se empresa
  "videos_uploaded": [...],
  "contracts_as_party": [...],
  "evaluations_given": [...],
  "consents": [...],
  "devices": [...],
  "audit_log_summary": [...]   // sumário de ações do user (não todo audit)
}
```

SLA: **15 dias** (Art. 19 §1º).

### 4.3 Direito de correção (Art. 18 III) — `PATCH /api/v1/me`

```http
PATCH /api/v1/me
{
  "name": "...",
  "phone": "...",
  "email": "..."        // email muda → re-verificação + step-up MFA
}
```

Mudanças em CPF/CNPJ **não permitidas** via self-service (requer ticket + documento + review admin).

### 4.4 Direito de exclusão (Art. 18 VI) — `DELETE /api/v1/me`

**SLA 15 dias** (workflow async):

```http
DELETE /api/v1/me
X-Step-Up-Token: <fresh-mfa-token>
Body: { "reason": "<optional>" }

202 Accepted
{
  "status": "scheduled",
  "execution_deadline": "2026-05-08T10:00:00Z",
  "export_url": "/api/v1/me/export"   // sugere export antes
}
```

### 4.4.1 Política de deleção: soft_delete universal (D-101 Fase 11 — 2026-04-24)

**Princípio canônico** (ver [[../STATE#d-101-—-lgpd-soft-delete-universal-pseudonimização-hmac-keyed-fecha-a-dc-sec-004-lgpd-m1-007|D-101]] + [[../02-architecture/adr/0037-soft-delete-universal-pseudonymize|ADR-0037]]):

**NUNCA hard_delete durante runtime de produto. SEMPRE soft_delete universal.**

- **Coluna `deleted_at TIMESTAMPTZ NULL`** em todas as tabelas com dados pessoais ou transacionais do usuário (~34 tabelas M1+M2).
- **`DELETE /me` (LGPD Art. 18 VI)**: setar `deleted_at = NOW()` + pseudonimizar campos pessoais (nome/email/cpf/phone via HMAC-keyed por [[../02-architecture/adr/0028-lgpd-keyed-hmac|ADR-0028]]). **Não executar `DELETE FROM` físico.**
- **Retenção 5 anos** para audit trail (LGPD Art. 15 + Art. 16 + normativa fiscal).
- **Hard-delete físico** acontece apenas em cron noturno `lgpd_retention_cleanup` (03:00 UTC) que roda `DELETE FROM users WHERE deleted_at < NOW() - INTERVAL '5 years' LIMIT 1000`.
- **Exceção única**: tabela `audit_log_entries` onde o hard-delete físico pós-retenção é a operação final (sem soft-delete intermediário).
- **Race condition com webhooks Stripe resolvida por construção**: webhooks sempre encontram row (UPDATE em colunas transacionais funciona; pseudonimização só afeta campos pessoais). Sem necessidade de soft-flag `_pending_hard_delete` janela 48h.
- **Reativação 30d**: usuário pode reativar conta dentro de 30d (restore de backup criptografado `users_pii_backup`); após 30d, dados pessoais pseudonimizados tornam-se irreversíveis.
- **ABAC**: `user.deleted_at IS NOT NULL` = equivalente a `role=banned` (403 em todo endpoint de negócio; apenas endpoints LGPD respondem).
- **Linter AST** enforça `WHERE deleted_at IS NULL` em todas queries de leitura de tabelas com soft_delete.

**Resolve**: [[../AUDIT#bucket-a-—-segurança-dual-check-sec|A-DC-SEC-004]] 🔴 → 🟢 + [[../AUDIT#🔴 Bloqueadores LGPD M1 — MANTIDOS ABERTOS|LGPD-M1-007]] 🔴 → 🟢.

---

### 4.5 Workflow de exclusão (atualizado 2026-04-24 — D-101 Fase 11: soft_delete universal + keyed HMAC)

Reformulado por **[[../STATE#d-101-…|D-101]] + [[../02-architecture/adr/0037-soft-delete-universal-pseudonymize|ADR-0037]]** (sobrescreve versão 2026-04-23 que usava soft-flag `_pending_hard_delete` + janela 48h). Resolve [[../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-004]] (race com webhook Stripe) + [[../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-038]] (pseudonimização determinística) **por construção**.

Implementado como `AccountDeletionSaga` orchestrated ([[../02-architecture/adr/0030-saga-orchestration-default]]) + keyed HMAC ([[../02-architecture/adr/0028-lgpd-keyed-hmac]]) + soft_delete universal ([[../02-architecture/adr/0037-soft-delete-universal-pseudonymize|ADR-0037]]).

```
Day 0: User emite DELETE /me (requer step-up MFA — IDN-036)
  → saga.start({user_id})
  → UPDATE users SET _pending_hard_delete = TRUE, deletion_scheduled_at = NOW() + 15d, status = 'pending_deletion'
  → lgpd_logs entry {event_type: "deletion_requested", user_id, user_id_hash: Pseudonymize(user_id, "audit_anon", NOW())}
  → Stripe pre-cancel API (cancela subscription ativa)
  → send email "Sua conta será excluída em 15 dias; clique aqui pra cancelar"
  → hard-lock conta (can't login; can cancel via token email)

Day 0-14: Grace period
  → User pode cancelar via email link → saga.compensate → UPDATE users SET _pending_hard_delete = FALSE
  → Webhook Stripe `customer.subscription.deleted` chega → handler consulta soft-flag:
       - IF _pending_hard_delete = TRUE AND user existe → skip UPDATE + log em lgpd_logs
       - ELSE processa normal
  → Cobre A-DC-SEC-004 (race condition)

Day 15: Execution async worker (AccountDeletionSaga advance)
  → verify _pending_hard_delete AINDA setado (se cancelado, saga encerra)
  → verify nenhum webhook em-trânsito (webhook_events status=pending com user match)
      IF webhooks pendentes: saga.wait(48h) e retry — janela 48h cobre retries Stripe
  → PSEUDONIMIZE via keyed HMAC (não SHA256 determinístico):
       anon_timeline := Pseudonymize(user_id, "timeline_anon", NOW())
       UPDATE timeline_entries SET actor_id = NULL, actor_pseudonym = anon_timeline WHERE actor_id = user_id
       anon_vote := Pseudonymize(user_id, "vote_anon", NOW())
       UPDATE votes SET voter_id = NULL, voter_pseudonym = anon_vote WHERE voter_id = user_id
       anon_contract := Pseudonymize(user_id, "contract_anon", NOW())
       UPDATE contracts SET participant_user_id = NULL, participant_pseudonym = anon_contract WHERE participant_user_id = user_id
       -- idem para minutes, evaluations, connect_me_requests
  → SOFT DELETE (D-101 Fase 11 — não mais hard-delete no Day 15):
       - users (UPDATE deleted_at = NOW() + pseudonimizar fields pessoais)
       - sessions (UPDATE deleted_at; revoga + tombstone)
       - device_fingerprints (UPDATE deleted_at)
       - user_consents (UPDATE deleted_at; preserva versioning para audit)
       - video uploads não-publicados (UPDATE deleted_at + Mux API delete de blobs físicos — Mux não faz parte do DB relacional, aplica hard-delete do blob com eventual-consistency)
  → Hard-delete físico só acontece em cron `lgpd_retention_cleanup` noturno após 5y (ver §4.4.1 + ADR-0037).
  → lgpd_logs entry {event_type: "deletion_executed", user_id_hash: Pseudonymize(user_id, "audit_anon", NOW())}
  → send email confirmação "Sua conta foi excluída"
  → saga.complete
```

**Propriedades garantidas**:

- **INV-104 (atualizado D-101)**: nenhum webhook re-materializa user deletado — resolvido **por construção** (soft_delete universal; webhook UPDATE funciona normalmente em row com `deleted_at IS NOT NULL`). Janela `_pending_hard_delete` 48h não é mais necessária.
- **INV-soft-delete-universal** (novo D-101): nenhuma tabela com dados pessoais sofre `DELETE` físico direto em runtime; apenas `UPDATE ... SET deleted_at = NOW()`. Hard-delete é exclusivamente executado pelo cron `lgpd_retention_cleanup` após retenção 5 anos.
- **INV-109** (keyed HMAC rotating): pseudônimos de anos diferentes são incorrelatáveis; após 5y retenção da key, pseudônimos daquele ano ficam matematicamente irreversíveis.
- **Purpose scope** (`timeline_anon` ≠ `vote_anon` ≠ `audit_anon`): impede correlação cross-contexto mesmo dentro do mesmo year.
- Preserva utilidade legal (Lei 4.591/64 — contagem de votos, presença, fração ideal) sem re-identificação.

### 4.6 Justificativa da pseudonimização vs hard delete (Art. 16)

**Votos e atas NÃO são hard-deleted** porque:

- **Art. 16 II** — "dados que precisam ser conservados para cumprimento de obrigação legal ou regulatória" (Lei 4.591/64 exige ata + registro de voto por 5+ anos).
- **Art. 16 IV** — "uso exclusivo do controlador, vedado seu acesso por terceiro, e desde que anonimizados os dados".
- Invalidar voto passado = **nulidade da assembleia inteira** → dano coletivo desproporcional.

**Mitigação**: pseudonimização (UUIDv7 neutro no lugar de user_id) preserva utilidade legal sem identificar.

### 4.7 Direito de revogação de consentimento (Art. 18 IX) — `PATCH /me/consents/:type`

Ver §3.4. Revogação é imediata; se tratamento depende do consent, feature bloqueia.

### 4.8 Direito à informação sobre uso (Art. 18 I)

Política de Privacidade pública em `/privacy-policy` (versioned). Indica: finalidades, base legal, compartilhamentos, retention.

### 4.9 Revisão de decisão automatizada (Art. 20)

**Aplicável a**:
- **Moderação automatizada de vídeo** (M4+ ML) — user pode pedir review humano.
- **Ranking Connect Me** (quando virar ML-based, M3+) — empresa pode pedir justificativa + review.
- **Risk scoring** (suspender device automaticamente) — appeal humano obrigatório.
- **Tier de reputação** (apesar de determinístico, user pode contestar inputs).

**Workflow**:
```
User gets automated decision (video rejected / blocked)
  → App shows reason + "Solicitar revisão humana"
  → POST /api/v1/lgpd/review-request {decision_id, reason}
  → audit_log + ticket criado pra admin
  → SLA 10 dias úteis pra review
  → Admin aprova/mantém/reverte → email + audit_log
```

---

## 5. Audit trail LGPD (`lgpd_logs` — apart de audit_log)

Dedicado a eventos LGPD pra facilitar audit regulatório.

```sql
CREATE TABLE lgpd_logs (
    id              UUID PRIMARY KEY DEFAULT uuidv7(),
    occurred_at     TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    user_id         UUID,                               -- pode ser null após hard delete (mas hash retido)
    user_id_hash    TEXT NOT NULL,                      -- hash irreversível pra rastreio pós-delete
    event_type      TEXT NOT NULL,                      -- 'consent.granted' | 'consent.revoked' | 'data.accessed' |
                                                        -- 'data.exported' | 'data.corrected' | 'data.deleted' |
                                                        -- 'review_requested' | 'data.breach'
    data_class      TEXT,                               -- CRITICAL | HIGH | NORMAL | PUBLIC
    triggered_by    TEXT NOT NULL,                      -- 'user_self' | 'admin:<user_id>' | 'system' | 'cron'
    ip              INET,
    device_fp_hash  TEXT,
    metadata        JSONB                               -- SEM PII — apenas ids + flags
);

-- Append-only
REVOKE UPDATE, DELETE ON lgpd_logs FROM app_role;

-- Retention 5 anos (partitioned monthly; drop após 5y + export S3)
```

**Retention**: 5 anos (mesmo pós-user-delete — hash permite audit regulatório sem PII).

---

## 6. Breach notification (Art. 48) — 72h

### 6.1 Definição de breach

Qualquer incidente que possa acarretar risco ou dano relevante aos titulares:

- Vazamento de PII (CPF, CNPJ, email, telefone, endereço, senha).
- Acesso indevido a audit_log.
- Comprometimento de credencial (Zitadel key leak, Stripe key leak).
- Cross-tenant data leak confirmado.
- Ransomware em backups.

### 6.2 Workflow

```
T+0:   Detection (Sentry alert, pentest finding, user report, gov disclosure)
T+30m: Ops on-call triage; sev assessment; containment (revoke keys, isolate service)
T+2h:  DPO notified; legal escalation; preserve evidence
T+6h:  Impact assessment — quem foi afetado, qual dado, extensão
T+24h: Draft notification to ANPD + affected users
T+48h: Review by legal + DPO
T+72h: Send to ANPD via gov.br + email to affected users
```

### 6.3 Template de notificação ANPD

Via portal https://www.gov.br/anpd (ou email se portal indisponível).

Conteúdo (Art. 48 §1º):
1. Descrição da natureza dos dados afetados.
2. Informações sobre os titulares (número, categorias).
3. Indicação das medidas técnicas/segurança utilizadas.
4. Riscos relacionados ao incidente.
5. Motivos de eventual demora.
6. Medidas que foram ou serão adotadas.

### 6.4 Template comunicação user afetado

Email + in-app:

```
Assunto: [Master Síndico] Aviso importante de segurança — LGPD

Prezado(a) <Nome>,

Detectamos em <data> um incidente que pode ter exposto alguns de seus dados:
- Tipos: <email, telefone>
- Período: <inicio - fim>

O que estamos fazendo:
- <ação 1>
- <ação 2>

O que recomendamos:
- Trocar sua senha
- Habilitar MFA
- Monitorar e-mails / SMS suspeitos

Estamos à disposição: lgpd@mastersindico.com.br
```

---

## 7. DPO — Data Protection Officer (Art. 41)

### 7.1 Papel

Obrigatório por Art. 41. Responsável por:
- Aceitar reclamações dos titulares.
- Receber comunicações da ANPD.
- Orientar funcionários.
- Executar demais atribuições do controlador.

### 7.2 Contato público

- Email: **lgpd@mastersindico.com.br** (obrigatório público na Política de Privacidade).
- Formulário web: `/lgpd/contato`.
- Endereço físico: divulgado em caso de reclamação formal ANPD.

### 7.3 Identidade

- **Pessoa física nomeada** com CPF + vínculo (funcionário ou contratado externo especializado).
- Nome público na Política de Privacidade.
- **⚠️ PENDÊNCIA M1**: nomeação formal (D-### em STATE).

---

## 8. DPA — Data Processing Agreement com terceiros

### 8.1 Terceiros com DPA obrigatório

| Terceiro | Função | Tipo de dado | DPA localizado em | Status |
|---|---|---|---|---|
| **Stripe** | Processamento de pagamento | Financeiro (sem PAN); email; país | Stripe Data Processing Addendum (público) | ✅ Stripe oferece default |
| **Mux** | Transcoding + CDN vídeo | Bytes de vídeo (pode conter PII em vídeo-currículo) | Mux DPA | ⚠️ Solicitar Mux DPA assinado |
| **Livekit** | WebRTC assembleia live | Audio + video stream real-time (não-persistido por default) | Livekit Cloud DPA | ⚠️ Validar M2+ |
| **Zitadel** | IdP (OIDC) | CPF, email, phone, senha hash | Zitadel Cloud DPA | ⚠️ Solicitar Zitadel DPA |
| **Twilio** | SMS (MFA, notificações) | Telefone, conteúdo SMS | Twilio DPA | ⚠️ Assinar M1 |
| **Resend / AWS SES** | Email transacional | Email, conteúdo email | Provider DPA | ⚠️ Decidir D-026 + assinar |
| **FCM (Google)** | Push mobile | Device token, conteúdo | Google Cloud DPA | ⚠️ Assinar via Firebase TOS |
| **AWS** (quando migrar) | Hosting + S3 | Todos | AWS GDPR/LGPD DPA (público) | ✅ AWS oferece default |
| **Railway** (M1) | Hosting | Todos | Railway TOS | ⚠️ Verificar cobertura LGPD |
| **OpenSearch managed** | Search index | Metadados (sem PII se seguir §2) | — | — |

### 8.2 Cláusulas obrigatórias no DPA

- Finalidade específica do tratamento.
- Tipos de dados + categorias de titulares.
- Duração do tratamento.
- Obrigações + direitos do controlador.
- Instruções documentadas.
- Confidencialidade dos funcionários do operador.
- Medidas de segurança (Art. 46).
- Sub-operadores — aprovação prévia.
- Auxílio ao controlador (atendimento a direitos dos titulares).
- Notificação de breach ao controlador em ≤ 24h.
- Retorno/exclusão dos dados pós-contrato.

---

## 9. Transferência internacional (Art. 33-36)

### 9.1 Princípio

Dados só saem do BR se país destino tiver **nível de proteção adequado** ou se houver **garantias específicas**.

### 9.2 Providers fora do BR

| Provider | País | Base legal |
|---|---|---|
| Stripe | EUA / Irlanda | Cláusulas contratuais específicas + DPA |
| Mux | EUA | Cláusulas contratuais + DPA |
| Livekit | EUA | Cláusulas contratuais + DPA |
| Zitadel Cloud | Suíça (ADEQUADO pelo GDPR) | Decisão de adequação GDPR + DPA |
| Twilio | EUA | Cláusulas contratuais + DPA |
| AWS | EUA + regiões BR (preferir sa-east-1) | Região BR preferida; cláusulas se fora |
| FCM (Google) | EUA + regiões | Cláusulas contratuais + DPA |

### 9.3 Controle

- **Preferir provedor com região BR** quando disponível (AWS sa-east-1, Google southamerica-east1).
- **Documentar em Política de Privacidade** a transferência com base legal.
- **Cláusulas contratuais específicas** no DPA (Art. 33 II).

### 9.4 ⚠️ PENDÊNCIA

- ADR avaliando se migramos certos providers pra versão BR (Stripe tem BR; Mux não tem BR).
- Decisão Resend vs AWS SES considerando localização (D-026).

---

## 10. Retention policy por entidade

| Entidade | Retention | Base legal |
|---|---|---|
| `users` (ativo) | indefinido (enquanto conta ativa) | Execução contrato |
| `users` (deleted) | **soft_delete + pseudonimização no Day 15; hard_delete físico após 5 anos via cron** (D-101 Fase 11 + ADR-0037) | Art. 15, Art. 16, Art. 18 VI |
| `sessions` | 30d pós-expiration | Segurança |
| `device_fingerprints` | 30d pós-último uso | Legítimo interesse (fraude) |
| `audit_log` | 5 anos online + 5 anos S3 WORM | Art. 16 + disputa judicial |
| `lgpd_logs` | 5 anos | Art. 16 compliance |
| `timeline_entries` | 10 anos (ou indefinido se institutional) | Lei 4.591/64 + LGPD Art. 16 |
| `minutes` | 10 anos (ou indefinido) | Lei 4.591/64 |
| `votes` | 10 anos (pseudonimizados pós-delete) | Lei 4.591/64 |
| `contracts` (Connect Me fechados) | 5 anos pós-término | Art. 16 Código Civil |
| `videos` (published) | enquanto plano ativo + 90d pós-unpublish | Produto |
| `videos` (deleted) | **soft_delete metadata (`deleted_at`) + hard delete imediato do blob Mux** (D-101 Fase 11); hard_delete row pós 5y | LGPD Art. 18 VI |
| `stripe_webhook_events` | 3 anos | Compliance financeiro |
| `user_consents` | 5 anos pós-revogação | Art. 8º §5º prova |

---

## 11. DPIA — Data Protection Impact Assessment (Art. 38)

### 11.1 Quando fazer DPIA

Obrigatório em tratamentos de alto risco:
- Tratamento em larga escala de dados pessoais.
- Decisão automatizada com efeito significativo (Art. 20).
- Dados sensíveis (não temos, mas vigilância).
- Monitoramento sistemático (device fingerprint, risk scoring).

### 11.2 DPIAs a fazer no master-sindico

- **DPIA-001**: device fingerprint + 1-device policy (legítimo interesse). **M1**.
- **DPIA-002**: moderação manual/automatizada de vídeos. **M2**.
- **DPIA-003**: reputation scoring + ranking Connect Me. **M2**.
- **DPIA-004**: geolocalização em Vizinhança. **M2**.
- **DPIA-005**: ML moderation (quando M4+). **M4**.

Template DPIA em [[../12-docs/dpia-template]] (a criar).

---

## 12. Minors / Menores (Art. 14)

### 12.1 Política

- **Titular menor de 18**: cadastro com consentimento específico e destacado de pelo menos um dos pais ou responsável (Art. 14 §1º).
- **Menor de 12**: dados só tratados no melhor interesse.
- Dados coletados devem ser **mínimos necessários**.

### 12.2 Master Síndico — política concreta

- **Usuário tem que ser ≥ 18 anos** para cadastrar como titular (síndico, morador, empresa). Enforce via data de nascimento no onboarding + termo.
- **Dependentes menores em unidade habitacional**: síndico pode registrar unidade com moradores dependentes (nome apenas), mas **sem coleta direta de dados do menor** pelo menor. Dados mínimos: nome + data de nascimento (opcional).
- **Vídeo-currículo Banco de Talentos**: obrigatório ≥ 18 anos.

### 12.3 Enforce

- Onboarding: campo `birth_date` + validação idade.
- ABAC check: `action=upload_video_curriculum` exige `user.age >= 18`.

---

## 13. Integração com outros controles

### 13.1 PII em logs (§7 [[BEYOND_CORP]])

Ver controles de PII masking. CI grep + VO Masked().

### 13.2 Audit trail (§11 [[BEYOND_CORP]])

`audit_log` + `lgpd_logs` separados mas correlacionáveis via `request_id`.

### 13.3 Threat model LINDDUN (§12 [[threat-model]])

Privacy threats mapeadas separadamente.

### 13.4 Consent UX

Frontend mostra banner versionado no primeiro login pós-update de política.

---

## 14. Roadmap LGPD

### 14.1 M1 (2026-05-08)

- [ ] `user_consents` schema + endpoints GET/PATCH.
- [ ] `lgpd_logs` table + eventos básicos.
- [ ] `GET /api/v1/me` (direito acesso).
- [ ] `DELETE /api/v1/me` workflow async (15d grace + hard delete users/sessions/devices; pseudonimize votes/minutes).
- [ ] Política de Privacidade v1 publicada + versionada.
- [ ] Termos de Uso v1 publicados.
- [ ] DPO nomeado (D-### em STATE).
- [ ] Endpoint público `/privacy-policy`, `/terms-of-service`, `/lgpd/contato`.
- [ ] DPA com Stripe, Mux, Zitadel, Twilio assinados.
- [ ] DPIA-001 (device fingerprint) executada + documentada.

### 14.2 M2 (2026-06-20)

- [ ] `GET /api/v1/me/export` (portabilidade) com step-up MFA.
- [ ] `PATCH /api/v1/me` (correção) com validações.
- [ ] `POST /api/v1/lgpd/review-request` (revisão decisão automatizada).
- [ ] Breach notification playbook + runbook.
- [ ] DPIA-002 (moderação vídeo) + DPIA-003 (reputation) + DPIA-004 (geo) executadas.
- [ ] DPA com Livekit, SES/Resend, FCM assinados.
- [ ] Field-level encryption CPF/CNPJ em DB (pgcrypto ou envelope encryption).
- [ ] S3 WORM Object Lock para audit_log + lgpd_logs (backup + export pós-5y).

### 14.3 M3 (2026-07-20)

- [ ] Cancelamento self-service de consentimento granular por tipo.
- [ ] Dashboard LGPD interno (métricas: % users com consent ativo; SLAs de exclusão).
- [ ] Review LGPD por auditor externo (1º ciclo).
- [ ] Integração com portal ANPD pra submissão formal (se houver API).

### 14.4 M4+

- [ ] DPIA-005 (ML moderation).
- [ ] Certification LGPD (quando ANPD publicar selo).
- [ ] Continuous compliance monitoring.

---

## 15. Checklist de due-diligence (para auditor externo)

- [ ] Política de Privacidade publicada + versionada + hash commitado.
- [ ] Termos de Uso publicados.
- [ ] DPO nomeado + contato público.
- [ ] DPAs com todos os operadores (Stripe, Mux, Livekit, Zitadel, Twilio, SES, FCM, AWS).
- [ ] Transferência internacional documentada com base legal.
- [ ] Direitos do titular implementados (acesso, correção, portabilidade, exclusão).
- [ ] SLA 15 dias para exclusão.
- [ ] Audit trail append-only 5 anos.
- [ ] Breach notification runbook.
- [ ] DPIAs executados em tratamentos de alto risco.
- [ ] PII masking em logs.
- [ ] Encryption at rest + in transit.
- [ ] Consentimento versionado.
- [ ] Minors policy.
- [ ] Retention schedule documentado.

---

## 16. Links

- [[_moc]]
- [[BEYOND_CORP]] §11 audit + §7 PII + §10 secrets
- [[threat-model]] §12 LINDDUN + §5 cross-tenant
- [[owasp-top10]] A09
- [[pentest-checklist]]
- [[../00-product/personas]] — personas-alvo
- [[../00-product/portfolio-de-produtos]] — data flows por sub-produto
- [[../12-docs/dpia-template]] (a criar)
- [[../12-docs/privacy-policy-template]] (a criar)
- [[../06-execution/playbooks/incident-response]] (a criar)
- Lei 13.709/2018: https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm
- ANPD: https://www.gov.br/anpd
