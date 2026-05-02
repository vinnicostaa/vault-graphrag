---
title: Dual-Check — Security Gaps Master Síndico v2 (15 Gap-SEC-### + varredura pentester)
type: audit
tags: [dual-check, security, stride, owasp, lgpd, pentest, master-sindico, v2]
decision_ref: —
adr_ref: —
created: 2026-04-23
propositor: Fase 5 — destilador 08-security
revisor: Dual-check agent (contexto limpo)
status: consolidado
---

# Dual-Check — Security Gaps Master Síndico v2

Revisão independente dos **15 Gap-SEC-###** listados em [[../../08-security/threat-model]] §15 + varredura pentester buscando gaps adicionais nos 6 artefatos de `08-security/`. Resultado: findings A-DC-SEC-### com severidade STRIDE + CVSS informal + priorização M1/M2/M3.

Fontes revisadas:
- [[../../08-security/BEYOND_CORP]]
- [[../../08-security/threat-model]]
- [[../../08-security/owasp-top10]]
- [[../../08-security/cve-process]]
- [[../../08-security/lgpd]]
- [[../../08-security/pentest-checklist]]
- [[../../08-security/beyond-corp-references]]
- [[../../13-research/beyond-corp/_destilado]] + sources 01-12.

---

## 1. Sumário executivo

### 1.1 Revisão dos 15 Gap-SEC-### existentes

| Veredito | Contagem | Descrição |
|---|---|---|
| ✅ **Concordo como especificado** | **7** | Gap identificado com clareza, priorização sugerida adequada, mitigação referenciada em doc oficial reconhecido. |
| ⚠️ **Concordo com caveats** | **6** | Gap correto, mas ou a priorização está branda vs risco real, ou a mitigação sugerida é incompleta, ou falta âncora em doc oficial. |
| ❌ **Discordo / reclassificar** | **2** | Gap subestimado — deve ser M1 bloqueante (não M2/M3) ou tem prerequisite ausente que torna mitigação ineficaz. |

### 1.2 Findings novos descobertos

**A-DC-SEC-### adicionais: 18** distribuídos:

| Severidade | Contagem | Ação |
|---|---|---|
| 🔴 **Crítico** (M1 bloqueante) | **4** | Pré-requisito pra ship M1 — ausência significa LGPD multa / fraude confirmável. |
| 🟡 **Relevante** (M1 high / M2) | **9** | Não bloqueia ship mas expõe superfície explorável em pentest externo Fase 5+. |
| 🟢 **Melhoria** (M2/M3) | **5** | Defesa em profundidade / hardening / anti-fraud sofisticado. |

### 1.3 TL;DR 10 linhas

1. Dos 15 Gap-SEC existentes, **7 estão bem** (003 RLS, 004 hash-chain, 006 WAF, 008 decay, 012 OPA, 013 mTLS, 015 Secrets Manager).
2. **6 caveats**: 001 Play Integrity precisa nomear SafetyNet→Play Integrity migration; 002 step-up MFA deve especificar token-side TTL + server denylist; 005 rate-limit Redis é M1 (não M2) se houver ≥ 1 réplica; 007 Receita Federal precisa LGPD DPA + fallback offline; 011 antivirus Mux não resolve — proposta falha; 014 continuous auth depende de 014-pré (baseline behavior signals).
3. **2 reclassificar**: Gap-SEC-009 dual-approval moderação é **M1** (não M3) — insider admin sozinho aprova vídeo sensível pré-ship; Gap-SEC-005 rate-limit distribuído **é M1** se API rodar com ≥ 2 réplicas no boot (decisão escalabilidade pode trazer réplica antes do previsto).
4. **4 críticos novos** M1 bloqueantes: A-DC-SEC-001 IDOR em `/me/devices/:id` (revocar device de outro); A-DC-SEC-002 webhook `/webhooks/livekit` sem body size limit; A-DC-SEC-003 CORS `credentials=true` permite cookie theft cross-subdomain; A-DC-SEC-004 LGPD hard-delete tem race com webhook Stripe pós-delete.
5. **9 relevantes** M1-M2: SSRF via Mux thumbnail URL (M2); path traversal em export filename (M1); proxy vote TOCTOU (M2); HMAC timing em `subtle.ConstantTimeCompare` não-alinhada em comprimentos diferentes; supply-chain transitive `golang.org/x/crypto` mais frequente que documentado; token leak via Sentry release tracking; admin_bypass audit sem `admin_reason` enforced.
6. **5 melhorias** M2-M3: passkeys WebAuthn antes de M3 pra admin; SBOM desde M1 (trivial com `syft`); egress firewall M2 (não M3); typo-squatting monitoring; log rate limit per user.
7. **LGPD bloqueador pré-M1**: nomeação formal do DPO ([[../../08-security/lgpd]] §7.3) + DPA Mux/Zitadel/Twilio assinados — sem isso ship M1 é **violação direta Art. 41 + Art. 39** (tratador sem representante identificado).
8. **Multi-tenant**: ABAC cache key `abac:v{N}:{user}:{tenant}` está bem, mas falta enforce tipo `tenant_id UUID` em runtime (Go admite string concat trivial) — A-DC-SEC-005.
9. **Assembly**: PBT de fração ideal está listada mas a invariante em [[../../01-domain/invariants.md]] precisa cobrir caso `SUM = 100.00` exato (float rounding em condomínios com 1000 units) — A-DC-SEC-006.
10. **Recomendação consolidada**: postergar ship M1 em 2026-05-08 até fechar os 4 críticos + LGPD DPO + Gap-SEC-009 reclassificado. Estimativa: 7-10 dias úteis. Alternativa: ship M1 em 2026-05-18 com segurança respaldada e dual-check fechado.

---

## 2. Revisão dos 15 Gap-SEC-### existentes

### Gap-SEC-001 — Device posture real (Play Integrity / DeviceCheck / reCAPTCHA Enterprise)

**Veredito**: ⚠️ **Concordo com caveats**

- **Priorização sugerida**: M2 (hardening).
- **Priorização revisor**: **M2 mantida** para mobile voting + upload; **M1 recomendado** para admin console web (reCAPTCHA Enterprise no login admin).
- **Caveats**:
  1. Google descontinuou SafetyNet em janeiro/2025 — [[../../08-security/BEYOND_CORP]] §4.3 fala "Play Integrity API" mas não explicita a migração; quem leu o legado pode confundir com SafetyNet.
  2. **Play Integrity API** tem quota free tier 10k requests/dia ([docs Google](https://developer.android.com/google/play/integrity/overview)) — pode virar custo em escala; não está em [[../../08-security/BEYOND_CORP]].
  3. **iOS App Attest** exige Xcode 13.0+ e iOS 14.0+ — baseline mobile da v2 não está documentada.
  4. **reCAPTCHA Enterprise** é GCP-only; se ADR final for AWS, precisa alternativa (AWS WAF Bot Control ou Cloudflare Turnstile).
- **Mitigação concreta (refinada)**:
  - Play Integrity API Standard request (`PRODUCT_SERIAL_IMEI` reply) em [voting + upload] — [doc oficial](https://developer.android.com/google/play/integrity/verdicts).
  - App Attest + DeviceCheck no iOS — [Apple doc](https://developer.apple.com/documentation/devicecheck).
  - Cloudflare Turnstile (vendor-neutral) como fallback web — [Cloudflare doc](https://developers.cloudflare.com/turnstile/).

### Gap-SEC-002 — Step-up MFA em endpoints críticos

**Veredito**: ⚠️ **Concordo com caveats**

- **Priorização sugerida**: M2.
- **Priorização revisor**: **M1** para DELETE /me/data + publish minutes + approve >R$10k; **M2** para demais.
- **Justificativa reclassificação parcial**: [[../../08-security/BEYOND_CORP]] §1 Tenet 6 lista 6 ações que **pedem step-up**. Dessas, 3 são LGPD/financeiro/legal **pré-M1**: DELETE /me (Art. 18 VI), publish minutes (Lei 4.591/64), transação >R$10k. Não faz sentido M1 permitir DELETE /me sem step-up — é auto-exploit bypass trivial (roubo de sessão elimina conta + dados).
- **Caveats**:
  1. [[../../08-security/BEYOND_CORP]] §1 Tenet 6 fala "MFA fresh (≤ 5min)" mas não especifica **onde** o step-up token é armazenado — cookie separado? Header `X-Step-Up-Token`? [[../../08-security/lgpd]] §4.2 mostra `X-Step-Up-Token: <fresh-mfa-token>` mas não diz como o JWT foi emitido.
  2. **Denylist de step-up tokens** em Redis não mencionada — logout deve revogar.
- **Mitigação concreta**:
  - `POST /auth/step-up` emite JWT curto (5min) com claim `amr=["mfa"]` + `acr=2` (OIDC spec padrão — [NIST SP 800-63-3 AAL2](https://pages.nist.gov/800-63-3/sp800-63b.html#aal2)).
  - Handler crítico exige header `X-Step-Up-Token` + verify JWT + verify denylist Redis.
  - Revogação: logout invalida + rotação session invalida.

### Gap-SEC-003 — PostgreSQL Row-Level Security (RLS)

**Veredito**: ✅ **Concordo**

- **Priorização**: M2 ✅
- **Mitigação concreta**: [PostgreSQL docs — Row Security Policies](https://www.postgresql.org/docs/16/ddl-rowsecurity.html). Exemplo adequadamente descrito em [[../../08-security/threat-model]] §5.1.
- **Observação**: garantir que `SET app.tenant_id` seja emitido **dentro da transação** (não no pool), senão pool reuse vaza entre tenants. Adicionar em [[../../08-security/pentest-checklist]] §7.3.

### Gap-SEC-004 — Hash-chain rolling audit_log

**Veredito**: ✅ **Concordo**

- **Priorização**: M2 ✅
- **Mitigação concreta**: pattern clássico merkle/chain — `hash(prev_hash || canonical_json(row))` por insert + `hash_root` publicado externamente (ex: email mensal pra DPO + cópia em S3 Object Lock). Referência: [Amazon QLDB Journal](https://docs.aws.amazon.com/qldb/latest/developerguide/journal-contents.html) como arquétipo.
- **Observação**: hash-chain sem publicação externa periódica = tampering só detectável pós-facto. Adicionar em [[../../08-security/BEYOND_CORP]] §11.4 o step "publish hash_root em 3rd-party WORM mensal".

### Gap-SEC-005 — Rate limit distribuído Redis

**Veredito**: ❌ **Discordo — reclassificar para M1 condicional**

- **Priorização sugerida**: M2.
- **Priorização revisor**: **M1 obrigatório se API rodar com ≥ 2 réplicas em prod** (provável no M1 se houver blue/green ou rolling deploy).
- **Justificativa**: [[../../08-security/BEYOND_CORP]] §6 diz "M1 `sync.Map` + cleanup goroutine é suficiente". Falso se houver 2+ instâncias — rate limit vira `20 rpm × N réplicas` por IP na prática. Atacante faz enumeration descobrindo load balancer sticky routing e contorna.
- **Mitigação concreta**:
  - Redis `INCR` + `EXPIRE` pattern — [Redis docs — rate limiting](https://redis.io/docs/latest/develop/use/patterns/distributed-locks/) + [token bucket via Lua script](https://redis.io/commands/eval/).
  - Decisão escalabilidade (1 réplica vs N) deve estar em `02-architecture/topology-multitenant.md` com justificativa.
  - Se monolito single-replica M1: rate limit in-memory aceitável **com ADR explicitando a restrição** + roadmap hard M2.

### Gap-SEC-006 — WAF edge (Cloudflare vs AWS WAF)

**Veredito**: ✅ **Concordo**

- **Priorização**: M2 ✅
- **Mitigação concreta**: ADR pendente reconhecido em [[../../08-security/BEYOND_CORP]] §6 + [[../../08-security/beyond-corp-references]] §⚠️ PENDÊNCIAS. Base: [Cloudflare WAF Managed Rules](https://developers.cloudflare.com/waf/managed-rules/) + [AWS WAF + OWASP CRS](https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups-list.html).
- **Observação**: WAF vazio custa pouco (Cloudflare Free: OWASP CRS básico grátis) — vale ligar **modo monitor-only M1** mesmo sem decisão final. Agrega telemetria pra tuning.

### Gap-SEC-007 — CNPJ validação Receita Federal

**Veredito**: ⚠️ **Concordo com caveats**

- **Priorização sugerida**: M2.
- **Priorização revisor**: **M2 mantida** mas com **prerequisite M1**: DPA com Receita (via SERPRO ou BrasilAPI) e base legal LGPD.
- **Caveats**:
  1. Receita Federal não tem API pública gratuita oficial. Alternativas: [BrasilAPI](https://brasilapi.com.br/docs#tag/CNPJ) (aberto, sem SLA formal), [Receitaws](https://receitaws.com.br/) (paga). Não documentado em [[../../08-security/threat-model]] §10.2.
  2. LGPD: consulta CNPJ → provider externo envia CNPJ. CNPJ de pessoa física (MEI) é dado pessoal → base legal + DPA necessários.
  3. **Fallback offline**: se provider cair, fluxo de proposta bloqueia — precisa degraded mode (marcar "validação pendente" + fila retry).
- **Mitigação concreta**:
  - [BrasilAPI CNPJ endpoint](https://brasilapi.com.br/api/cnpj/v1/{cnpj}) como default + ADR.
  - DPA / base legal `legítimo interesse` (prevenção fraude) documentado em [[../../08-security/lgpd]] §8.1.
  - Circuit breaker + cache 24h resultado + degraded mode doc.

### Gap-SEC-008 — Reputation decay temporal

**Veredito**: ✅ **Concordo**

- **Priorização**: M3 ✅
- **Mitigação concreta**: half-life exponencial 12m conforme [[../../08-security/threat-model]] §11.1. Referência conceitual: [Elo rating system decay](https://en.wikipedia.org/wiki/Elo_rating_system) + [Glicko-2 rating deviation over time](http://www.glicko.net/glicko/glicko2.pdf).
- **Observação**: PBT ([[../../08-security/threat-model]] §11.2) precisa cobrir: `tier(empresa_ativa) >= tier(empresa_inativa_12m)` com todo o resto igual. Adicionar.

### Gap-SEC-009 — Dual-approval moderation

**Veredito**: ❌ **Discordo — reclassificar para M1**

- **Priorização sugerida**: M3.
- **Priorização revisor**: **M1 pra conteúdo sensível** (denúncias, menores, contratos Connect Me) + **M3 pra vídeos institucionais**.
- **Justificativa**: [[../../08-security/threat-model]] §9.1 lista "moderador comprometido aprova malicious" como threat. No M1, time é **≤ 5 pessoas** (solo-ish); probabilidade de colusão é alta, e é trivial via social engineering. Dual-approval não é sofisticação M3 — é **separation of duties** básica (OWASP ASVS V12.1.2) aplicável desde dia 1.
- **Mitigação concreta**:
  - Workflow: action sensível → 1 moderador "propõe" → 2º moderador "confirma" → efeito aplicado. Rollback trivial.
  - Audit trail: ambos moderators em `audit_log.metadata.approvers`.
  - Referência: [SOC 2 CC6.1 Logical & Physical Access — separation of duties](https://www.aicpa-cima.com/topic/audit-assurance/audit-and-assurance-greater-than-soc-2). Essencial pra eventual cert M3+.
  - **Exceção M1**: basic video moderation pode ser single approver se time físico = 1 pessoa; documentar como aceite de risco em STATE D-### até segunda pessoa entrar.

### Gap-SEC-010 — Reputation farm detection

**Veredito**: ✅ **Concordo**

- **Priorização**: M4 (ML-based) ✅
- **Observação**: até M4, heurística simples `unique(condominium.cnpj) per empresa ≥ threshold_tier` conforme [[../../08-security/threat-model]] §11.1 é mitigação M1 válida — o Gap-SEC-010 é sobre detecção **estatística/ML**, não sobre bloqueio básico.
- **Mitigação concreta futura**: clustering de padrões (mesmo timestamp, IP próximo, síndicos sem outros tenants) — referência [Graph Neural Networks for fraud detection](https://arxiv.org/abs/2004.10118). **Backlog** é ok.

### Gap-SEC-011 — Antivírus scan em upload Mux

**Veredito**: ⚠️ **Concordo com caveats pesados** — proposta atual não resolve

- **Priorização sugerida**: M4+.
- **Priorização revisor**: **M2 com ClamAV/Macie** — M4 é atraso perigoso.
- **Caveats**:
  1. Mux faz transcoding (re-encode), o que **quebra payload executável puro**, mas **NÃO** detecta: (a) vídeo legítimo com metadados EXIF maliciosos; (b) deep fakes usados em fraude (c) conteúdo ilegal (abuso, DMCA); (d) polyglot files (JPEG+PHP).
  2. [[../../08-security/threat-model]] §4.5 diz "Scanner antivírus M4+?" com interrogação — revisor confirma a dúvida é real: antivírus **per se** não serve pra vídeo; o que serve é **content moderation** (ML moderation API ou manual SLA).
  3. Não existe mitigação "AV" apropriada — o gap correto é **content moderation pipeline**.
- **Mitigação concreta (reformulada)**:
  - **M1**: moderação manual obrigatória pre-publish + SLA 48h ([[../../08-security/threat-model]] §9.1).
  - **M2**: hash match contra base known-bad (PhotoDNA Microsoft, ThreatExchange Meta) — [PhotoDNA Cloud Service](https://www.microsoft.com/en-us/photodna).
  - **M2**: file type/magic-bytes validation server-side além do Mux MIME check.
  - **M3**: ML moderation automatizada (AWS Rekognition Content Moderation / Google Video Intelligence API) + human-in-the-loop.
  - **M4+**: ClamAV ou Macie apenas pra uploads não-Mux (anexos RFP, PDFs contratuais).
- **Renomear Gap-SEC-011**: "Content moderation pipeline" — não "antivirus scan".

### Gap-SEC-012 — PDP sidecar (OPA)

**Veredito**: ✅ **Concordo**

- **Priorização**: M2 ✅
- **Mitigação concreta**: ADR aberto ([[../../08-security/beyond-corp-references]] §⚠️ PENDÊNCIAS) reconhecido. Base: [OPA docs](https://www.openpolicyagent.org/docs/latest/) + [Rego language](https://www.openpolicyagent.org/docs/latest/policy-language/) vs [AWS Cedar](https://docs.cedarpolicy.com/).
- **Observação**: PDP externalizado é NIST SP 800-207 §3.1 (separação PE/PA/PEP) — alinhado.

### Gap-SEC-013 — mTLS leste-oeste

**Veredito**: ✅ **Concordo**

- **Priorização**: M2 (3 caminhos críticos) + M3 (service mesh) ✅
- **Mitigação concreta**: [Linkerd automatic mTLS](https://linkerd.io/2.16/features/automatic-mtls/) ou [Istio mTLS](https://istio.io/latest/docs/concepts/security/#mutual-tls-authentication). Alinhado com BeyondProd ([[../../08-security/beyond-corp-references]] §Fonte 05).
- **Observação**: em M1 (monolito-modular), mTLS é overkill correto — dep inter-BC é in-process. Sem caveat.

### Gap-SEC-014 — Continuous auth / risk scoring

**Veredito**: ⚠️ **Concordo com caveats**

- **Priorização sugerida**: M3.
- **Priorização revisor**: **M3 ok** mas com **prerequisite M2**: baseline de behavior signals.
- **Caveats**:
  1. Risk score sem baseline estatístico = threshold arbitrário. Precisa coletar dados M1 (IP history, login hour distribution, device change frequency) e calcular percentis M2 antes de agir M3.
  2. [[../../08-security/BEYOND_CORP]] §1 Tenet 7 fala "falhas ABAC seguidas" mas não quantifica "seguidas".
  3. Risk scoring é decisão automatizada LGPD Art. 20 — exige review humano ([[../../08-security/lgpd]] §4.9).
- **Mitigação concreta**:
  - **M1**: ingest de `risk_signals` em tabela (sem agir ainda).
  - **M2**: dashboard Grafana com distribuições p50/p95/p99 por sinal.
  - **M3**: thresholds calibrados + soft-lock (step-up forced) não hard-lock (exceto em p99+).
  - Referência: [Google Cloud reCAPTCHA Enterprise score interpretation](https://cloud.google.com/recaptcha/docs/interpret-assessment-website) — 0.0 muito provável bot / 1.0 muito provável humano.

### Gap-SEC-015 — AWS Secrets Manager

**Veredito**: ✅ **Concordo**

- **Priorização**: M2 ✅
- **Mitigação concreta**: [AWS Secrets Manager docs](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html) + rotação automática via Lambda. Alternativas mencionadas ok (HashiCorp Vault open-source se multi-cloud).
- **Observação**: alinhado com Railway M1 → ECS M2 migration path em [[../../08-security/BEYOND_CORP]] §10.2.

---

## 3. Findings novos — A-DC-SEC-### (varredura pentester)

### 🔴 Críticos (M1 bloqueantes)

---

#### A-DC-SEC-001 — IDOR em `DELETE /api/v1/me/devices/:id`

- **Severidade**: 🔴 **Crítico**
- **STRIDE**: **E**levation of Privilege + **T**ampering
- **CVSS informal**: **8.1 High** (AV:N/AC:L/PR:L/UI:N/S:U/C:L/I:H/A:H)
- **Categoria pentest**: AuthZ (Cap. 2.1 IDOR)
- **Exploit hypothesis**:
  - Morador A loga. Faz `DELETE /api/v1/me/devices/<uuid-device-do-sindico-X>`.
  - Se handler valida "este device pertence ao user autenticado?" **após** a query, atacante revoga device de síndico — síndico é kicked out (1-device trigger + revocation cascade).
  - Em assembleia live, atacante revoga device do síndico no meio da votação, invalidando sessão Livekit → DoS direcionado.
- **Prerequisite**: endpoint `DELETE /me/devices/:id` existe (implícito no fluxo 1-device + §4 [[../../08-security/BEYOND_CORP]]; não está OpenAPI'd).
- **Mitigação**:
  - ABAC action `device.revoke` exige `resource.owner_id == subject.id` **antes** de qualquer query.
  - PBT: `∀ user_A, device_B onde device_B.owner != user_A → DELETE /me/devices/device_B.id == 403`.
  - Audit test em cross-tenant test suite ([[../../08-security/threat-model]] §5.1).
- **Priorização**: **M1 bloqueante** — IDOR é OWASP A01 fundamental.
- **Doc oficial**: [OWASP A01:2021 — Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/).

---

#### A-DC-SEC-002 — Body size limit ausente em `/webhooks/livekit`

- **Severidade**: 🔴 **Crítico**
- **STRIDE**: **D**oS
- **CVSS informal**: **7.5 High** (AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:N/A:H)
- **Categoria pentest**: Cap. 6 Webhook + Cap. 5.3 Cost-per-request
- **Exploit hypothesis**:
  - `/webhooks/livekit` é pública ([[../../08-security/BEYOND_CORP]] §8.3). Diferente de Stripe/Mux, **não está listada com `BodySizeLimit` 1MB** explicitamente — [[../../08-security/BEYOND_CORP]] §2 webhooks middleware mostra `BodySizeLimit (≤ 1MB)` pra Stripe/Mux mas o middleware aplicado a Livekit JWT não é explicitado.
  - Atacante envia payload 100MB. Verify JWT tem que parse header parcial; se implementação ingenua consumir body todo pré-verify → memory exhaustion.
  - Livekit JWT é body inteiro — tamanho esperado ≤ 2KB. Limit defensivo 64KB é seguro.
- **Mitigação**:
  - Middleware `BodySizeLimit(64KB)` específico pra `/webhooks/livekit` (signed JWT é pequeno).
  - Documentar explicitamente em [[../../08-security/BEYOND_CORP]] §2 stack webhooks.
- **Priorização**: **M1 bloqueante** — endpoint público descartado.
- **Doc oficial**: [OWASP ASVS V5.2.1 — Input Validation / Body Size](https://owasp.org/www-project-application-security-verification-standard/).

---

#### A-DC-SEC-003 — CORS `credentials=true` + subdomain wildcard = cookie theft

- **Severidade**: 🔴 **Crítico**
- **STRIDE**: **S**poofing + **I**nfo Disclosure
- **CVSS informal**: **8.8 High** (AV:N/AC:L/PR:N/UI:R/S:U/C:H/I:H/A:N)
- **Categoria pentest**: Cap. 1.2 Session (CSRF variant)
- **Exploit hypothesis**:
  - [[../../08-security/BEYOND_CORP]] §9 CORS: "Credentials allowed apenas pra `.mastersindico.com.br`". Ambíguo — se implementação faz regex `*.mastersindico.com.br`, subdomains arbitrários servem cookie.
  - Se `user-upload.mastersindico.com.br` virar qualquer dia CDN Mux/S3 com controle de terceiro (Mux domain alias, staging subdomain comprometido), atacante hospeda JS que lê cookie `ms_session` via `fetch(..., credentials: 'include')`.
  - Cookie `ms_session` tem `Domain=.mastersindico.com.br` ([[../../08-security/BEYOND_CORP]] §5.2) → qualquer subdomain recebe.
- **Mitigação**:
  - Allowlist **estrita** de origins: `app.mastersindico.com.br`, `admin.mastersindico.com.br` — não wildcard subdomain.
  - Cookie `Domain=app.mastersindico.com.br` sem ponto-inicial (não compartilha com siblings).
  - Adicionar teste CORS preflight: `Origin: evil.mastersindico.com.br` → rejeitado (not in allowlist).
- **Priorização**: **M1 bloqueante** — cookie theft = full account takeover.
- **Doc oficial**: [Mozilla MDN — Access-Control-Allow-Credentials](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials) + [Portswigger — CORS misconfigurations](https://portswigger.net/web-security/cors).

---

#### A-DC-SEC-004 — LGPD hard-delete race condition com webhook Stripe

- **Severidade**: 🔴 **Crítico**
- **STRIDE**: **R**epudiation + **I**nfo Disclosure (privacidade)
- **CVSS informal**: **6.5 Medium** (LGPD weight eleva pra Crítico)
- **Categoria pentest**: Cap. 16 LGPD
- **Exploit hypothesis**:
  - Day 15 cron job executa hard-delete user X. Delete acabou 00:00:05.
  - Webhook Stripe `customer.subscription.updated` chega 00:00:10 pra `user_X.stripe_customer_id`.
  - Handler tenta atualizar `users(id=user_X)` → row não existe → handler faz INSERT (comum em saga compensation) ou **log com email do user** → PII **re-materializada pós-delete** ferindo Art. 18 VI + Art. 16.
- **Prerequisite**: implementação de webhook Stripe sem double-check de `user.deletion_executed_at != null`.
- **Mitigação**:
  - Pre-delete: **cancela Stripe subscription proativamente** (via API) + aguarda `customer.subscription.deleted` webhook antes de hard-delete DB.
  - Handler webhook: check `user.id` em `users_deleted` (tombstone table) → skip + log em `lgpd_logs`.
  - Tombstone mantém `user_id_hash` (permite rastreio sem PII).
- **Priorização**: **M1 bloqueante** — LGPD Art. 18 VI + ANPD fine risk.
- **Doc oficial**: [LGPD Art. 16](https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm#art16) + [Stripe API — Cancel subscription](https://docs.stripe.com/api/subscriptions/cancel).

---

### 🟡 Relevantes (M1 high / M2)

---

#### A-DC-SEC-005 — `tenant_id` type confusion (string vs UUID)

- **Severidade**: 🟡 **Relevante**
- **STRIDE**: **E**levation of Privilege + **T**ampering
- **CVSS informal**: **6.8 Medium** (AV:N/AC:H/PR:L/UI:N/S:C/C:H/I:H/A:N)
- **Categoria pentest**: Cap. 2.5 Cross-tenant
- **Exploit hypothesis**:
  - Cache key `abac:v{N}:{user}:{tenant}` ([[../../08-security/BEYOND_CORP]] §3.3). Se `tenant` é string em Go (`context.Value("tenant_id")`), atacante faz request com header `X-Tenant-Hint: "' OR '1'='1"` → cache key vira `abac:v1:uuidA:' OR '1'='1` — não injeta SQL, mas se cache key é usada em log (`log.Info("abac check", "key", key)`) + log shipping tem parser → log injection.
  - Pior: se middleware **deriva** tenant_id do header (não do token), type confusion: `tenant_ids: []string{"uuid1,uuid2"}` (string única com vírgula) passa no `IN` slice contra DB → queries cross-tenant sem DENY.
- **Mitigação**:
  - `tenant_id` sempre tipo domain `type TenantID uuid.UUID` — nunca string bare.
  - Middleware TenantIsolation valida `uuid.Parse` retorna nil error.
  - Log emite `tenant_id.String()` — nunca raw.
  - PBT: fuzz header `X-Tenant-Hint` com strings arbitrárias → middleware rejeita com 400.
- **Priorização**: **M1 high** — defense in depth.
- **Doc oficial**: [OWASP Type Confusion](https://owasp.org/www-community/vulnerabilities/Type_Juggling).

---

#### A-DC-SEC-006 — Fração ideal rounding edge case

- **Severidade**: 🟡 **Relevante**
- **STRIDE**: **T**ampering + **R**epudiation
- **CVSS informal**: **5.3 Medium** (AV:N/AC:H/PR:L/UI:N/S:U/C:N/I:H/A:N)
- **Categoria pentest**: Cap. 8.1 Vote
- **Exploit hypothesis**:
  - Condomínio com 1000 units, cada fração = 0.1000% — `SUM = 100.0000%` exato **se** stored em decimal fixo. Se armazenado em `float64`, soma acumula erro → `99.99999...%` ou `100.00001...%`.
  - Invariante [[../../08-security/threat-model]] §14 item 1 "Σ units.fracao_ideal ≤ 100.00%" pode ser violada trivialmente por roundup com 1000 units — ou aprovar assembleia com quorum `50%` mas Σ votos favoráveis = `49.999%` (quorum flip por rounding).
  - Atacante disputa judicialmente — ata inválida.
- **Mitigação**:
  - Storage `NUMERIC(10,4)` (não `float`) em PostgreSQL.
  - Go `decimal.Decimal` (github.com/shopspring/decimal) — nunca `float64`.
  - PBT com N units ∈ {1, 2, 10, 100, 1000, 10000} e SUM == 100.0000 exato.
- **Priorização**: **M1 high** — affects vote integrity Lei 4.591/64.
- **Doc oficial**: [PostgreSQL NUMERIC type](https://www.postgresql.org/docs/16/datatype-numeric.html) + [IEEE 754 rounding errors](https://0.30000000000000004.com/).

---

#### A-DC-SEC-007 — SSRF via Mux thumbnail URL configurável

- **Severidade**: 🟡 **Relevante**
- **STRIDE**: **I**nfo Disclosure + **E**oP
- **CVSS informal**: **6.4 Medium** (AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:L/A:N)
- **Categoria pentest**: Cap. 14 SSRF
- **Exploit hypothesis**:
  - [[../../08-security/owasp-top10]] A10 diz "thumbnail upload por URL → não aplicável (upload direto Mux)" — correto pra upload.
  - **Porém** Mux permite thumbnail **custom** via `POST /assets/{id}/tracks` com URL externa ([Mux API — Tracks](https://docs.mux.com/api-reference#tag/tracks)). Se master-sindico expõe `POST /api/v1/videos/:id/thumbnails { url }` → SSRF via URL arbitrária (Mux fetch da URL).
  - Atacante passa `http://169.254.169.254/latest/meta-data/` (AWS metadata) → se Mux infra é AWS, exfil.
- **Prerequisite**: feature "thumbnail custom" existe (não documentado nos specs atuais; pode chegar com requirement Conteúdo).
- **Mitigação**:
  - Se feature existir: allowlist de URLs (só Mux delivery URLs já existentes no DB).
  - PBT: `thumbnail_url.host ∈ {"stream.mux.com", "image.mux.com"}`.
- **Priorização**: **M2** — feature não está em M1.
- **Doc oficial**: [OWASP A10:2021 — SSRF](https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/) + [Mux Tracks API](https://docs.mux.com/api-reference#tag/tracks).

---

#### A-DC-SEC-008 — Path traversal em LGPD export filename

- **Severidade**: 🟡 **Relevante**
- **STRIDE**: **I**nfo Disclosure
- **CVSS informal**: **4.3 Medium** (AV:N/AC:L/PR:L/UI:N/S:U/C:L/I:N/A:N)
- **Categoria pentest**: Cap. 3.5 Path traversal
- **Exploit hypothesis**:
  - [[../../08-security/lgpd]] §4.2 `Content-Disposition: attachment; filename="master-sindico-export-<user_id>-2026-04-23.json"`.
  - Se `user_id` vem de `context` (não sanitizado), atacante com conta criada com nome CPF-like → `filename=";alert(1);.json"` → header injection. Alguns browsers executam como XSS (raro) ou confunde parser proxy.
  - Mais provável: se admin exporta outro user e `user_id` é parameter, `../` em filename — clientes desktop salvam em path traversed.
- **Mitigação**:
  - Filename: `master-sindico-export-<uuid>.json` (UUIDv7, no dots, no slashes).
  - Server valida `uuid.MustParse`.
  - `Content-Disposition` valor quotado + regex `^[a-zA-Z0-9_.-]+$`.
- **Priorização**: **M1 high** — endpoint M2 mas spec corrigida agora é trivial.
- **Doc oficial**: [RFC 6266 Content-Disposition](https://www.rfc-editor.org/rfc/rfc6266) + [OWASP Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload).

---

#### A-DC-SEC-009 — Proxy vote TOCTOU (registro pós-sessão aberta)

- **Severidade**: 🟡 **Relevante**
- **STRIDE**: **T**ampering
- **CVSS informal**: **6.1 Medium** (AV:N/AC:H/PR:L/UI:N/S:U/C:L/I:H/A:N)
- **Categoria pentest**: Cap. 8.5 Proxy fraud
- **Exploit hypothesis**:
  - [[../../08-security/pentest-checklist]] §8.5 "Proxy registrado ANTES da sessão (`POST /api/v1/assemblies/:id/proxies` fecha antes de `session.open_at`)".
  - TOCTOU: proxy grant criado `session.open_at - 1s`. Sessão abre em paralelo. Grant está DB mas vote_cast usa snapshot `open_at_grants` — se não há snapshot, grant ainda passa.
  - Alternativa: síndico aliado registra proxy após open_at via race (abre assembly, cria proxy em paralelo, vota com procuração de alguém que mudou de ideia).
- **Mitigação**:
  - Ao abrir sessão: `INSERT INTO assembly_proxies_snapshot SELECT * FROM assembly_proxies WHERE assembly_id = X AND granted_at < open_at`.
  - Vote_cast consulta **snapshot** (read-only) — não tabela live.
  - PBT: `∀ proxy com granted_at >= open_at → não conta`.
- **Priorização**: **M2** — vote integrity é Lei 4.591/64.
- **Doc oficial**: [OWASP Race Conditions](https://owasp.org/www-community/vulnerabilities/Race_Conditions) + MP 2.200-2 (assinatura proxy).

---

#### A-DC-SEC-010 — HMAC compare com comprimentos diferentes = timing leak

- **Severidade**: 🟡 **Relevante**
- **STRIDE**: **I**nfo Disclosure
- **CVSS informal**: **4.8 Medium** (AV:N/AC:H/PR:N/UI:N/S:U/C:L/I:L/A:N)
- **Categoria pentest**: Cap. 6.3 Signature timing
- **Exploit hypothesis**:
  - Go `hmac.Equal` é constant-time **apenas** se slices têm mesmo comprimento. Se atacante envia `Stripe-Signature` com payload curtíssimo (`v1=a`), implementação ingênua pode cair em early-return (não-constant) antes do Equal.
  - `subtle.ConstantTimeCompare` retorna 0 imediatamente se lens diferentes — ainda não-constante.
  - Pattern seguro: **sempre** calcular HMAC esperado + compare com input (ambos têm comprimento fixo).
- **Mitigação**:
  - Parse signature header primeiro; se comprimento do v1 candidato != len(expected_hex) → rejeitar `400` **sem** comparação.
  - Usar `subtle.ConstantTimeCompare` apenas após comprimentos normalizados.
- **Priorização**: **M1 high** — trivial de implementar certo, cobrado em pentest externo.
- **Doc oficial**: [Go crypto/subtle docs](https://pkg.go.dev/crypto/subtle) + [Stripe HMAC verification guide](https://docs.stripe.com/webhooks/signatures#verify-manually).

---

#### A-DC-SEC-011 — Supply chain: `golang.org/x/crypto` transitive (frequência subestimada)

- **Severidade**: 🟡 **Relevante**
- **STRIDE**: **T**ampering (dep comprometida)
- **CVSS informal**: varia por CVE
- **Categoria pentest**: Cap. 11 Deps
- **Exploit hypothesis**:
  - [[../../08-security/cve-process]] §4 exemplo usa `stripe-go` → `golang.org/x/crypto`. Revisor observa: `golang.org/x/crypto` tem **média 3-4 CVEs HIGH/ano** ([Go Vuln DB histórico 2023-2025](https://pkg.go.dev/vuln/)). SLA 5d ([[../../08-security/cve-process]] §1) é apertado pra deps transitive + upstream lento.
  - `replace` directive workaround é trivial mas introduz divergência cross-team (outras apps podem pegar `replace` errado).
- **Mitigação**:
  - Dependabot config `directive: "security-only"` + auto-merge pra patch-level em `golang.org/x/*` (bumps são geralmente seguros).
  - CI enforce `go.mod` tem `go 1.23` mínimo (tooling mais recente = fix mais rápido).
  - SLA especial `golang.org/x/*`: 3d (mais curto que HIGH padrão).
- **Priorização**: **M1** — config Dependabot + CI é trivial.
- **Doc oficial**: [Go Vulnerability Database](https://pkg.go.dev/vuln/) + [GitHub Dependabot config](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file).

---

#### A-DC-SEC-012 — Sentry release tracking pode vazar tokens via context

- **Severidade**: 🟡 **Relevante**
- **STRIDE**: **I**nfo Disclosure
- **CVSS informal**: **5.5 Medium** (AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:N/A:N)
- **Categoria pentest**: Cap. 10.1 PII leakage
- **Exploit hypothesis**:
  - [[../../08-security/BEYOND_CORP]] §15 menciona Sentry. Sentry captura stack trace + **local variables** by default. Se panic em handler `func(c *gin.Context)` — o `*Context` inclui headers, incluindo `Authorization: Bearer <token>`.
  - Sentry server-side pode "scrub" mas default scrubbing pega só chaves típicas (password, cc). Bearer tokens em header não estão no default denylist.
- **Mitigação**:
  - Sentry SDK Go: `beforeSend` hook strip `c.Request.Header["Authorization"]`, `Cookie`, `X-Device-Fingerprint`.
  - Test: inject panic em staging + inspect Sentry event → zero sensitive fields.
  - [Sentry Data Scrubbing docs](https://docs.sentry.io/data-management/sensitive-data/).
- **Priorização**: **M1 high** — Sentry vai ligar no M1; config errada = leak continuous.
- **Doc oficial**: [Sentry — Sensitive Data Scrubbing](https://docs.sentry.io/platforms/go/data-management/sensitive-data/).

---

#### A-DC-SEC-013 — `admin_reason` não enforced em admin_bypass

- **Severidade**: 🟡 **Relevante**
- **STRIDE**: **R**epudiation
- **CVSS informal**: **3.5 Low** (eleva via compliance LGPD)
- **Categoria pentest**: Cap. 2.3 admin_bypass
- **Exploit hypothesis**:
  - [[../../08-security/BEYOND_CORP]] §11.1 "Admin override: is_admin=true bypass registrado com justificativa (campo admin_reason)".
  - Não há enforce em middleware: se admin faz request sem header `X-Admin-Reason: "<texto>"`, audit_log tem `admin_reason: null`. Compliance review perde rastro.
  - Inside attacker admin pode operar cover de "bug".
- **Mitigação**:
  - Middleware ABAC: se `is_admin == true && action is_sensitive → require(header X-Admin-Reason length >= 10)`.
  - Audit log rejeita null + retorna 400 "admin reason required".
  - Dashboard LGPD mostra % admin_reasons populated (alvo 100%).
- **Priorização**: **M1** — enforce trivial, compliance SOC 2 / LGPD Art. 37.
- **Doc oficial**: [SOC 2 CC6.3 Logical Access — Privileged Account Monitoring](https://www.aicpa-cima.com/).

---

### 🟢 Melhorias (M2/M3)

---

#### A-DC-SEC-014 — Passkeys WebAuthn antes de M3 pra admin

- **Severidade**: 🟢 **Melhoria**
- **STRIDE**: preventivo **S**poofing
- **CVSS informal**: N/A
- **Categoria pentest**: Cap. 1.4 MFA
- **Mitigação**:
  - Zitadel suporta WebAuthn nativo ([Zitadel docs — WebAuthn](https://zitadel.com/docs/guides/solution-scenarios/passwordless)).
  - Adiantar passkeys **obrigatórios pra admin** M2 (não M3).
  - Usuários finais mantêm TOTP M1-M2; passkey default M3.
- **Priorização**: **M2** para admin (não M3).
- **Doc oficial**: [WebAuthn W3C spec](https://www.w3.org/TR/webauthn-2/) + [NIST SP 800-63B AAL3](https://pages.nist.gov/800-63-3/sp800-63b.html#aal3).

---

#### A-DC-SEC-015 — SBOM geração desde M1

- **Severidade**: 🟢 **Melhoria**
- **Categoria**: Cap. 11.3
- **Mitigação**:
  - `syft` é CLI single-binary — rodar em CI é 1-liner: `syft . -o cyclonedx-json > sbom.json`.
  - Artifact do CI salva SBOM por build.
  - Nenhum custo; facilita muito resposta a CVE futuro.
- **Priorização**: **M1** (move de M2).
- **Doc oficial**: [Anchore Syft](https://github.com/anchore/syft) + [CycloneDX spec](https://cyclonedx.org/).

---

#### A-DC-SEC-016 — Egress firewall M2 (não M3)

- **Severidade**: 🟢 **Melhoria**
- **Categoria**: Cap. 12.2
- **Mitigação**:
  - Railway não suporta egress control — se M2 migra ECS (D-024), security group egress allowlist é trivial.
  - Lista allowlist: Stripe API, Mux API, Livekit API, Zitadel, Twilio, SES, FCM, BrasilAPI (CNPJ).
- **Priorização**: **M2** (adiantar de M3).
- **Doc oficial**: [AWS Security Groups — Egress](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html).

---

#### A-DC-SEC-017 — Typosquatting detection em new direct deps

- **Severidade**: 🟢 **Melhoria**
- **STRIDE**: **T**ampering (supply chain)
- **Categoria**: Cap. 11.2
- **Mitigação**:
  - CI hook em PR que adiciona dep direta: check `go list -m -json <dep>` + validate maintainer match esperado (ex: `golang.org/*` = Google).
  - Check download count (< 1000 downloads/month = flag).
  - [Socket.dev](https://socket.dev/) API gratuita pra Go/NPM/PyPI — integra CI.
- **Priorização**: **M2**.
- **Doc oficial**: [Socket Security Docs](https://docs.socket.dev/).

---

#### A-DC-SEC-018 — Log rate limit per user (anti log-flooding DoS)

- **Severidade**: 🟢 **Melhoria**
- **STRIDE**: **D**oS (log exhaustion)
- **Categoria**: Cap. 10 Logs
- **Mitigação**:
  - Usuário mal-intencionado pode abusar endpoints raros que logam muito (e.g. `/health` se logar; ou 401 loop que logue cada tentativa).
  - Rate limit **de logs** per user: sample 1 em 100 se > 1000 events/min do mesmo user_id.
  - zerolog/slog não tem rate limit nativo — wrapper interno.
- **Priorização**: **M2**.
- **Doc oficial**: [Google SRE Book — Monitoring Ch6](https://sre.google/sre-book/monitoring-distributed-systems/) (discusses log cardinality).

---

## 4. Priorização consolidada M1/M2/M3

### 4.1 M1 (2026-05-08) — bloqueantes pré-ship

| ID | Título | Classificação |
|---|---|---|
| **Gap-SEC-002-partial** | Step-up MFA para DELETE /me + publish minutes + approve >R$10k | Reclassificado M2→M1 |
| **Gap-SEC-005-cond** | Rate limit Redis se ≥ 2 réplicas | Reclassificado condicional M1 |
| **Gap-SEC-009-partial** | Dual-approval moderation sensível (denúncia, menor, contratos) | Reclassificado M3→M1 |
| **A-DC-SEC-001** | IDOR `/me/devices/:id` | 🔴 Novo |
| **A-DC-SEC-002** | Body size limit `/webhooks/livekit` | 🔴 Novo |
| **A-DC-SEC-003** | CORS credentials + subdomain wildcard | 🔴 Novo |
| **A-DC-SEC-004** | LGPD hard-delete race Stripe | 🔴 Novo |
| **A-DC-SEC-005** | tenant_id type confusion | 🟡 Novo |
| **A-DC-SEC-006** | Fração ideal rounding (decimal type) | 🟡 Novo |
| **A-DC-SEC-008** | Path traversal LGPD export filename | 🟡 Novo |
| **A-DC-SEC-010** | HMAC length-diff timing | 🟡 Novo |
| **A-DC-SEC-011** | Dependabot config transitive Go x/crypto | 🟡 Novo |
| **A-DC-SEC-012** | Sentry scrubbing Bearer tokens | 🟡 Novo |
| **A-DC-SEC-013** | admin_reason enforce | 🟡 Novo |
| **A-DC-SEC-015** | SBOM via syft | 🟢 Novo |
| **LGPD-pre-M1** | DPO nomeado + DPA Mux/Zitadel/Twilio | 🔴 compliance |

### 4.2 M2 (2026-06-20) — hardening

| ID | Título |
|---|---|
| Gap-SEC-001 | Play Integrity / DeviceCheck mobile (+ admin reCAPTCHA M1) |
| Gap-SEC-002 | Step-up MFA nos demais endpoints |
| Gap-SEC-003 | PostgreSQL RLS |
| Gap-SEC-004 | Hash-chain audit_log (+ publish root mensal S3 WORM) |
| Gap-SEC-005 | Rate limit distribuído Redis (full) |
| Gap-SEC-006 | WAF edge ADR decided (monitor-only pode M1) |
| Gap-SEC-007 | CNPJ validação Receita via BrasilAPI + DPA |
| Gap-SEC-011 | Content moderation pipeline (hash match PhotoDNA) — renamed |
| Gap-SEC-012 | PDP sidecar (OPA/Cedar ADR) |
| Gap-SEC-013 | mTLS 3 caminhos críticos |
| Gap-SEC-014-pre | Ingest risk_signals (baseline) |
| Gap-SEC-015 | AWS Secrets Manager (ECS migration) |
| A-DC-SEC-007 | SSRF Mux thumbnail (se feature chegar) |
| A-DC-SEC-009 | Proxy vote snapshot |
| A-DC-SEC-014 | Passkeys WebAuthn admin |
| A-DC-SEC-016 | Egress firewall ECS |
| A-DC-SEC-017 | Typosquatting detection |
| A-DC-SEC-018 | Log rate limit per user |

### 4.3 M3 (2026-07-20) — maturidade

| ID | Título |
|---|---|
| Gap-SEC-008 | Reputation decay |
| Gap-SEC-009 | Dual-approval moderation full |
| Gap-SEC-011 | Content moderation ML (Rekognition / Video Intelligence) |
| Gap-SEC-013 | Service mesh Linkerd |
| Gap-SEC-014 | Continuous auth / risk scoring (full) |
| Gap-SEC-015 | Hash-chain S3 Object Lock Compliance imediato |

### 4.4 Backlog (M4+)

| ID | Título |
|---|---|
| Gap-SEC-010 | Reputation farm detection ML-based |
| Gap-SEC-011 | AV scan non-Mux uploads (ClamAV/Macie) |
| Gap-SEC-014 | ML anomaly detection feed PDP |

---

## 5. Recomendações regulatórias LGPD bloqueadoras pré-M1

Consolidação dos itens LGPD que **não podem esperar M2**. Ausência = violação direta ANPD + risco multa até 2% faturamento (Art. 52 §1º II).

### 5.1 Bloqueadores absolutos

1. **DPO nomeado + contato público**  
   Art. 41 obriga. [[../../08-security/lgpd]] §7.3 marca ⚠️ PENDÊNCIA M1.  
   Sem DPO nomeado + email `lgpd@mastersindico.com.br` publicado, **ship M1 é violação direta**. Resolver: D-### em STATE + página `/lgpd/contato` + assinatura no rodapé do app.

2. **Política de Privacidade publicada + versionada + hash commitado**  
   Art. 9º (transparência). Hash SHA256 do texto no `user_consents.consent_text_hash` ([[../../08-security/lgpd]] §3.1). Sem isso não há prova de consent.

3. **Termos de Uso publicados + versionados**  
   Complementar; mesmo critério.

4. **DPA com operadores que recebem PII cross-border**:  
   - **Stripe** ✅ (tem DPA default).  
   - **Mux** ⚠️ solicitar assinado.  
   - **Zitadel** ⚠️ solicitar assinado.  
   - **Twilio** ⚠️ assinar M1 (SMS transmite telefone).  
   - SES/Resend ⚠️ decidir D-026 + assinar.  
   Sem DPA assinado, transferência internacional é **ilegal** (Art. 33-36).

5. **`DELETE /api/v1/me`** (direito exclusão Art. 18 VI)  
   SLA 15d workflow async já planejado M1 ([[../../08-security/lgpd]] §4.5).  
   **Dependência A-DC-SEC-004** (race Stripe) tem que ser resolvida junto — caso contrário o hard-delete vaza.

6. **`GET /api/v1/me`** (direito acesso Art. 18 II)  
   Está M1 ✅. Confirmar implementação no sprint.

7. **DPIA-001 device fingerprint** (Art. 38 tratamentos de alto risco)  
   [[../../08-security/lgpd]] §11.2. Obrigatório formalizar antes de ship — ANPD pode exigir em fiscalização.

### 5.2 Alinhados-mas-verificar

8. **Consentimento versionado** — schema OK ([[../../08-security/lgpd]] §3). Verificar `consent_text_hash` é enforced em boot.

9. **Minors policy** ≥18 enforced — verificar implementação dos campos `birth_date` + ABAC check ([[../../08-security/lgpd]] §12.3).

10. **Audit trail append-only** — [[../../08-security/BEYOND_CORP]] §11.1. `REVOKE UPDATE, DELETE ON audit_log` em migration M1 confirmada.

11. **Retention policy** — [[../../08-security/lgpd]] §10. Cron job de partition drop pós-5y documentado M1? Verificar.

### 5.3 Checklist ship-gate M1

- [ ] DPO nomeado (nome + CPF + papel) em `STATE.md` D-###.
- [ ] `lgpd@mastersindico.com.br` live + reply.
- [ ] `/privacy-policy` + `/terms-of-service` + `/lgpd/contato` publicados.
- [ ] Política v1 hash commitado em `04-requirements/lgpd-texts/privacy-policy-v1.md` + hash em boot config.
- [ ] DPA Mux, Zitadel, Twilio assinados + armazenados em `12-docs/legal/dpas/`.
- [ ] DPIA-001 executada + arquivada em `12-docs/dpias/dpia-001-device-fingerprint.md`.
- [ ] `GET /me` + `DELETE /me` endpoints live + cross-tenant tested.
- [ ] A-DC-SEC-004 hard-delete race resolvido + teste integration.
- [ ] audit_log migration com `REVOKE UPDATE, DELETE`.
- [ ] Retention cron scheduled + dry-run tested.

---

## 6. Action items consolidados

- [ ] Atualizar [[../AUDIT]] com 18 A-DC-SEC-### (referenciando este dual-check).
- [ ] Atualizar [[../../08-security/threat-model]] §15 reclassificando Gap-SEC-002/005/009.
- [ ] Atualizar [[../../08-security/BEYOND_CORP]] §14.1 M1 com os novos itens bloqueantes.
- [ ] Criar D-### em [[../../STATE]] para decisões derivadas (rate-limit Redis condicional; dual-approval M1 scope).
- [ ] Abrir ADR candidatos: ADR-### WAF (monitor-only M1); ADR-### CNPJ provider; ADR-### Sentry scrubbing config; ADR-### DPO nomeação.
- [ ] Atualizar [[../../08-security/pentest-checklist]] §7.3 (`SET app.tenant_id` dentro da transaction) + §6.3 (HMAC length normalize pré-compare).
- [ ] Atualizar [[../../08-security/lgpd]] §4.5 com A-DC-SEC-004 mitigation (pre-cancel Stripe subscription).
- [ ] Atualizar [[../../08-security/owasp-top10]] A01 com A-DC-SEC-001 (IDOR /me/devices).
- [ ] Programar review trimestral próxima: 2026-07-23.

---

## 7. Links

- [[_moc]]
- [[template]]
- [[../AUDIT]]
- [[../../STATE]]
- [[../../08-security/BEYOND_CORP]]
- [[../../08-security/threat-model]]
- [[../../08-security/owasp-top10]]
- [[../../08-security/cve-process]]
- [[../../08-security/lgpd]]
- [[../../08-security/pentest-checklist]]
- [[../../08-security/beyond-corp-references]]
- [[../../13-research/beyond-corp/_destilado]]
- [[../../06-execution/playbooks/dual-check]]
