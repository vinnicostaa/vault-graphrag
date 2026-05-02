---
title: Threat Model — STRIDE por BC + Multi-tenant — Master Síndico v2
type: spec
tags: [security, threat-model, stride, linddun, multi-tenant, master-sindico, v2]
source: 90-ingested legacy threat-model.md + 13-research/beyond-corp/_destilado
created: 2026-04-23
updated: 2026-04-23
aliases: [threat-model, STRIDE]
---

# Threat Model — STRIDE por BC + Multi-tenant

Modelagem de ameaças com **STRIDE** (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege) + **LINDDUN** (privacidade) + seções específicas multi-tenant e por sub-produto.

**Escopo**: todos os 7 bounded contexts (identity, billing, institutional, commercial, content, assembly, cross-domain) + integrações externas (Stripe, Mux, Livekit, Zitadel, Twilio, SES, FCM).

**Herança**: expande [[../90-ingested/from-vault-50-projects-master-sindico/08-security/threat-model|threat-model legado]] adicionando threats multi-tenant, provider compromise, assembly vote integrity, content moderation bypass, Connect Me fraud, reputation gaming.

**Revisão**: trimestral + toda feature que atravessa superfície pública + incident retrospective.

---

## 1. Assets críticos

| Asset | Sensibilidade | Impacto de compromisso | BC |
|---|---|---|---|
| Credenciais (Zitadel tokens, API keys) | 🔴 Crítico | Comprometimento total do tenant / plataforma | identity |
| PII (CPF, CNPJ, endereço, email, telefone) | 🔴 Crítico LGPD | Multa LGPD (Art. 52) + reputação | identity, institutional |
| Atas de assembleia | 🔴 Crítico legal | Fraude eleitoral condominial + disputa judicial | assembly, institutional |
| Votos | 🔴 Crítico legal | Fraude de quórum fracional + nulidade assembleia | assembly |
| Fundos via Stripe (PAN não tocado — SAQ-A) | 🔴 Crítico | Fraude financeira + incident report Stripe | billing |
| Contratos Connect Me | 🟡 Alto | Disputa contratual + multa | commercial |
| Vídeos Mux privados/institucionais | 🟡 Alto | Exposição indevida + violação de propriedade intelectual | content |
| Reputação (Bronze → Diamante) | 🟡 Alto | Manipulação de ranking = fraude de mercado | commercial |
| Timeline institucional imutável | 🟡 Alto legal | Manipulação de memória auditável | institutional |
| Planos/assinaturas (quotas) | 🟢 Médio | Burla de quota; impacto financeiro limitado | billing |
| Device fingerprints | 🟢 Médio LGPD | Tracking + fraude (votação) | identity |
| audit_log | 🔴 Crítico | Apagar trilha = colapso compliance | cross-domain |
| Código-fonte backend | 🟢 Médio | Facilita ataque; por si só não vaza dados | — |

---

## 2. Actors

| Actor | Motivação | Acesso base | Risco |
|---|---|---|---|
| Morador legítimo | Usar app | Auth + role=morador + unit | 🟢 Baixo |
| Síndico legítimo | Gerir condomínio | Auth + role=sindico + condo | 🟡 Médio (privilegiado) |
| Empresa Plus/Pro | Vender serviços | Auth + role=empresa | 🟡 Médio |
| Parceiro Vizinhança | Vender ao morador | Auth + role=parceiro | 🟢 Baixo |
| Admin plataforma | Ops + suporte | Auth + role=admin + is_admin=true | 🔴 Alto (bypass) |
| Atacante externo | Fraude, extorsão, espionagem | Zero | 🔴 Alto |
| Insider malicioso (ex-síndico, ex-empresa) | Vingança, sabotagem | Auth legítimo abusando | 🔴 Alto |
| Bot / credential stuffing | Fraude massa, spam | Zero | 🔴 Alto |
| Supply chain (dep comprometida) | Injeção, exfiltração | Via código nosso | 🔴 Alto |
| Provider comprometido (Stripe/Mux/Zitadel) | Fora do nosso controle | Acesso ao que o provider vê | 🟡 Médio-Alto |

---

## 3. Trust boundaries

```
[Cliente browser / Flutter app]                       TRUST ZONE 0 — untrusted
           │ HTTPS TLS 1.3
           ▼
[Edge CDN / Cloudflare ou CloudFront + AWS WAF]       TRUST ZONE 1 — bot protect, DDoS
           │
           ▼
[Railway Edge (M1) / AWS ALB (M2+)]                    TRUST ZONE 2 — TLS termination
           │
           ▼
[API Gateway / Go Gin abstraído + middleware stack]    TRUST ZONE 3 — auth + ABAC
           │ internal pool
           │
     ┌─────┼──────────┬─────────────┬──────────────────┐
     ▼     ▼          ▼             ▼                  ▼
[PostgreSQL 18]  [Redis 7]  [Stripe API]  [Mux API]  [Livekit]  [Zitadel]
   TZ 4            TZ 4       TZ 5 (ext)   TZ 5       TZ 5       TZ 5
     ↓                         HTTPS         HTTPS
[S3 backups / Object Lock]                            TRUST ZONE 4 — internal privileged
  TZ 6 (WORM)
```

Boundaries críticas:
- **TZ 0 → TZ 3**: autenticação + ABAC + rate limit. Pilar BeyondCorp.
- **TZ 3 → TZ 4**: conn string com credentials; rede privada Railway. mTLS M2+.
- **TZ 3 → TZ 5** (provider externo): HMAC webhook + API key + TLS 1.2+.
- **TZ 3 → TZ 6** (WORM): append-only; sem UPDATE/DELETE grants.
- **Dev ↔ Railway**: 2FA + audit log (Ops).

---

## 4. STRIDE por superfície

### 4.1 Superfície: `/auth/*` — endpoints públicos autenticação

**Assets**: Zitadel tokens, session cookies, device fingerprints.

| Threat (STRIDE) | Exemplo concreto | Controle implementado | Gap + mitigação M2 |
|---|---|---|---|
| **S**poofing | Request fingindo callback Zitadel | PKCE `code_verifier` em cookie encrypted (gorilla/securecookie) + state validation | — |
| **T**ampering | Modificar `code` na URL do callback | Code Zitadel vale 1x; TTL 10min | — |
| **R**epudiation | "Eu não fiz login ontem" | audit_log com IP + device fingerprint + timestamp | A-023: adicionar diff device |
| **I**nfo Disclosure | Token em logs / error response | Proibido logar token; verificação manual + CI grep regex | M2: enforce via `slog` handler com allow-list |
| **D**oS | Brute force login | Rate limit `/auth/*` 20 rpm / IP burst 5 | M2: CAPTCHA após 3 falhas |
| **E**oP | Privilege escalation via token forjado | Zitadel introspection em toda request (não confia em claims client-side) | — |

### 4.2 Superfície: `/api/v1/*` — API REST autenticada

**Assets**: dados de todos os BCs.

| Threat | Exemplo | Controle | Gap |
|---|---|---|---|
| **S**poofing | Session hijack via cookie stolen | httpOnly + Secure + SameSite=Lax + 1-device policy invalida session antiga | M2: bind session a device_fp |
| **T**ampering | Modificar body in-transit | TLS 1.3 + body size limit 1MB | — |
| **R**epudiation | Ação não rastreável | IAuditLogger (DT-010) + audit_log append-only | M2: hash-chain rolling |
| **I**nfo Disclosure | Cross-tenant read (IDOR) | ABAC tenant_mismatch + 100+ cross-tenant tests + RLS Postgres (M2) | Ver §5 Tenant Boundary |
| **D**oS | Query expensiva (N+1, full scan) | Pagination cursor obrigatória; A-020 fix N+1 UPDATE | M2: query budget (pg_stat_statements + timeouts) |
| **E**oP | Morador vê dados de síndico | ABAC matrix + PBT 400 casos | — |

### 4.3 Superfície: `/webhooks/*` — webhooks de provider

**Assets**: estado Stripe (subscription), Mux (asset state), Livekit (session state).

| Threat | Exemplo | Controle | Gap |
|---|---|---|---|
| **S**poofing | Request forjando Stripe | HMAC-SHA256 verify ANTES de parse; timestamp ≤ 5min | Ver §6 Webhook Tampering |
| **T**ampering | Alterar event body | HMAC em body completo; idempotency via `event.id` UNIQUE | — |
| **R**epudiation | Evento processado 2x | `stripe_webhook_events(event_id UNIQUE)` INSERT ON CONFLICT | — |
| **I**nfo Disclosure | Payload vazado em log | Log só metadata (event.id, type); nunca body cru | — |
| **D**oS | Replay massivo | Idempotency descarta replay; rate limit externo do provider | M2: body size limit enforced |
| **E**oP | Webhook "criando" subscription privilegiada | Webhook só **atualiza** estado; nunca cria User / Subscription novo | — |

### 4.4 Superfície: WebSocket `/api/v1/assemblies/:id/live` — sessão live

**Assets**: votos, falas, presença.

| Threat | Exemplo | Controle | Gap |
|---|---|---|---|
| **S**poofing | User não-elegível entrando | JWT Livekit gerado no backend **após** ABAC (role + unit + elegibilidade) | — |
| **T**ampering | Votar com payload manipulado | Voto server-side com UoW + UNIQUE(agenda_item_id, voter_id) | Ver §7 Assembly Vote Integrity |
| **R**epudiation | "Não votei assim" | audit_log do voto com hash do payload + session + fp + timestamp | — |
| **I**nfo Disclosure | Outros moradores vendo fala privada | ACL Livekit + room por (condominium, session, role) | — |
| **D**oS | Flood de WS conn | Rate limit 5 conn/user + heartbeat timeout 30s | M2: max_concurrent_rooms por tenant |
| **E**oP | Morador moderando sessão | Role check explícito (sindico only) na emissão do token | — |

### 4.5 Superfície: Upload direto Mux (presigned URL)

**Assets**: bytes dos vídeos.

| Threat | Exemplo | Controle | Gap |
|---|---|---|---|
| **S**poofing | URL presigned reusada por outro user | URL one-use, TTL 1h, bound a `asset_id` | — |
| **T**ampering | Subir malware como "vídeo" | Mux valida MIME; moderação manual antes de publish; Mux faz transcoding | ⚠️ Scanner antivírus M4+? |
| **R**epudiation | Uploader nega ter enviado | audit_log `video.upload_initiated` + `video.asset_ready` webhook Mux | — |
| **I**nfo Disclosure | Vídeo privado vazado | Signed playback URL com TTL (CDN signed URL) | — |
| **D**oS | Upload massivo (cota flood) | Quota por plano (trial 0, basic N, pro M); Saga A-027 previne órfão | M2: burst detection |
| **E**oP | User sem plano upload premium | ABAC + plan_tier check antes de emitir presigned URL | — |

### 4.6 Superfície: Integração Stripe `/billing/*`

**Assets**: subscriptions, invoices, customer IDs.

| Threat | Exemplo | Controle | Gap |
|---|---|---|---|
| **S**poofing | Spoof de customer_id | `stripe_customer_id` é idempotent e bound a user_id na nossa DB | — |
| **T**ampering | Alterar `amount` em checkout | Preço definido server-side; cliente nunca manda amount | — |
| **R**epudiation | "Não assinei" | Stripe webhook event + audit_log | — |
| **I**nfo Disclosure | PAN (cartão) na nossa DB | **Nunca tocamos PAN** — Stripe Checkout/Elements faz tudo (SAQ-A) | — |
| **D**oS | Flood de checkout | Rate limit + Stripe tem proteção própria | — |
| **E**oP | Burlar trial | `trial_ends_at` DB constraint + webhook `customer.subscription.trial_will_end` | Ver §9 Business Logic |

### 4.7 Superfície: Busca OpenSearch / PG tsvector

**Assets**: índice de condomínios, empresas, vídeos, parceiros.

| Threat | Exemplo | Controle | Gap |
|---|---|---|---|
| **S**poofing | Query forjada | Não aplicável (query formulada pelo user autenticado) | — |
| **T**ampering | Query injection no índice | Parâmetros sanitizados (sqlc parametriza; OS client valida JSON) | — |
| **R**epudiation | — | audit_log em `search.executed` (M2+) | — |
| **I**nfo Disclosure | Cross-tenant search leak | Resultados filtrados por tenant **antes** de retornar; documentos indexados com `tenant_id` filter em toda query | Ver §5 |
| **D**oS | Query expensiva (wildcard `**`) | Timeout query 2s + limit 50 results/page | M2: query budget per user |
| **E**oP | Buscar recurso que não verias direto | ABAC no result set + filter por `visibility` | — |

---

## 5. Tenant boundary threat model (multi-tenant específico)

**Superfície crítica** — invariante principal do master-sindico: dados do condomínio X **nunca** vazam pro condomínio Y.

### 5.1 Cross-tenant data leak

| Threat | Vector | Controle | Teste |
|---|---|---|---|
| Query sem `WHERE condominium_id` | `SELECT * FROM units WHERE id = $1` | **Convenção**: `WHERE id = $1 AND condominium_id = $2` obrigatório + code review grep | 100+ cross-tenant integration tests |
| ABAC bypass via cache poisoning | Cache key não versionada por tenant | Cache key: `abac:v{N}:{user}:{tenant}` — bump N invalida global | PBT: cache per tenant nunca serve outro |
| OpenSearch filter esquecido | Query `match_all` sem `tenant_id` filter | Wrapper obrigatório `searchTenant(tenant_id, query)` + grep lint | Integration test por BC |
| Domain event cross-BC sem tenant check | BC-commercial publica evento consumido por BC-institutional sem revalidar tenant | Todo consumer revalida `event.tenant_id ∈ user.tenants` antes de agir | Integration test |
| URL path traversal (`../tenant-X/resources`) | Client manipula path | Route params validados UUID + ABAC re-check no resource | — |

**Controle defesa em profundidade (M2+)**:
- **PostgreSQL RLS (Row-Level Security)** policy: `POLICY tenant_isolation ON units USING (condominium_id = current_setting('app.tenant_id')::uuid)`.
- Conexão seta `SET app.tenant_id = $user_tenant` antes de queries.
- Mesmo com SQL injection, RLS bloqueia cross-tenant.

### 5.2 ABAC bypass

| Threat | Exemplo | Controle |
|---|---|---|
| `is_admin=true` forjado | Atacante edita claim | Zitadel introspection confirma is_admin do IdP, não do token self-contained |
| Bypass ordem matriz | Pular tenant check | Matriz **ordenada**: admin_bypass → role → action → tenant → scope (§3.2 [[BEYOND_CORP]]) |
| Role escalation via update_role | Morador vira síndico via PATCH | ABAC: `action=update_membership` exige role=sindico OU is_admin; tenant match |
| Cache staleness | Stripe cancela plan mas cache ainda diz "pro" | Webhook invalida cache {user, tenant} imediatamente |

### 5.3 Query injection sem `WHERE condominium_id`

Catálogo de queries vulneráveis (grep em PR review):

```bash
# Grep proibido em PR
rg 'SELECT.*FROM (units|memberships|timeline_entries|votes|minutes|announcements|contracts|videos)' backend/internal --type go \
  | rg -v 'condominium_id' \
  | rg -v '-- SKIP-TENANT-CHECK: <justificativa>'
```

Exceções documentadas:
- Queries cross-tenant intencionais (ex: admin dashboard) com comment `-- SKIP-TENANT-CHECK: admin report`.
- Queries em `users` sem tenant scope (user é global; filter é por user_id).
- Queries em `sessions` globais.

---

## 6. Provider compromise threat model

### 6.1 Stripe key leak

| Threat | Impacto | Controle |
|---|---|---|
| `STRIPE_SECRET_KEY` vaza | Atacante cria refunds, subscriptions fraudulentas | Rotação imediata via Stripe dashboard; restrição IP (Stripe Restricted Keys); audit Stripe events |
| `STRIPE_WEBHOOK_SECRET` vaza | Atacante envia webhooks forjados | Rotação webhook endpoint; idempotency protege de duplicata |
| Stripe account compromise | Total | MFA obrigatório na Stripe dashboard; review activity log; 2FA hardware key |

**Controle preventivo**: ver §10.1-2 [[BEYOND_CORP]] (secrets gitignored + gitleaks + AWS Secrets Manager M2+).

### 6.2 Mux key leak

| Threat | Impacto | Controle |
|---|---|---|
| `MUX_TOKEN_SECRET` vaza | Atacante faz upload em nome nosso (quota burn) | Rotação; restrição IP (Mux feature); audit Mux events |
| `MUX_SIGNING_KEY` vaza | Atacante forja signed playback URLs | Rotação; TTL curto das URLs (15min) limita dano |

### 6.3 Zitadel compromise

| Threat | Impacto | Controle |
|---|---|---|
| Zitadel key JSON vaza (A-018 legado) | Atacante emite tokens arbitrários | Rotação Zitadel; pipeline CI com gitleaks; `.gitignore` reforçado |
| Zitadel account master compromise | Total ruína do IAM | MFA obrigatório; hardware key; IP allowlist no Zitadel |
| Zitadel Cloud outage | Todos usuários deslogados | Plan B: cache introspection result 5min; graceful degradation (read-only mode) |

### 6.4 Livekit compromise

| Threat | Impacto | Controle |
|---|---|---|
| `LIVEKIT_API_SECRET` vaza | Atacante gera JWT pra qualquer room | Rotação; JWT TTL curto (duração da sessão + 5min); verify room name corresponde a assembly legítima |

### 6.5 Twilio / SES / FCM

| Threat | Impacto | Controle |
|---|---|---|
| Twilio key vaza | SMS phishing em nosso nome; bill shock | Rotação; quota daily; monitoring envio atípico |
| SES key vaza | Email phishing | Rotação; DKIM/SPF/DMARC; quota daily |
| FCM key vaza | Push notif spam | Rotação; FCM token scope per user |

---

## 7. Webhook tampering

Detalhado em §4.3. Adicional:

### 7.1 HMAC bypass

| Threat | Exemplo | Controle |
|---|---|---|
| Timing attack em signature compare | Vazar HMAC byte a byte medindo tempo | `hmac.Equal()` comparação tempo constante |
| Replay attack | Reenviar webhook legítimo antigo | Timestamp check ≤ 5min + idempotency event.id UNIQUE |
| Bypass por parse ANTES de verify | Se verify falha, body já foi parseado com lixo | **Raw body** armazenado; HMAC verify primeiro; parse só após OK |

### 7.2 Signature verification timing attack

- Toda comparação de signature via `hmac.Equal()` / `crypto/subtle.ConstantTimeCompare`.
- Nunca `bytes.Equal` em MAC/signature.

---

## 8. Assembly vote integrity

**Superfície crítica**: voto em assembleia tem valor legal (Lei 4.591/64 + Lei 10.406/02 CC Art. 1.334).

### 8.1 Threats

| Threat | Vector | Controle |
|---|---|---|
| Duplicate vote | User vota 2x no mesmo agenda_item | UNIQUE(agenda_item_id, voter_id) DB constraint + `ErrConflict` (A-025 resolvido) |
| Vote in closed session | Votar após `close()` (A-030) | State machine strict: `vote_cast` exige `session.state = open` (check no use case + DB CHECK) |
| Proxy fraud | Procuração forjada | Proxy registrado com assinatura digital **antes** da sessão; audit_log; limite de proxies por outorgante |
| Coerção | Pressão presencial ou remota pra votar X | Fora do escopo técnico; mitigação: voto secreto opcional (M3 — só grava resultado agregado) |
| Vote stuffing via bot | Atacante cria N contas fantasmas votando | Device fingerprint + Play Integrity (M2) + verificação `user.unit_id ∈ condominium.units` + fração ideal |
| Manipulação de quórum | Voto com fração ideal incorreta | `vote.weight = user.unit.fracao_ideal` computado server-side; invariante Σ ≤ 100% (PBT) |
| Minutes tampering | Editar ata pós-publish | Minutes **imutável** pós-publish; correção vira nova entry timeline (A-023 legado) |
| Vote replay via session fixation | Reutilizar session da sessão anterior | Vote session_id ≠ HTTP session_id; voto bound a `assembly_session_id` + `user_id` + `cast_at` |

### 8.2 PBT obrigatório

- Σ vote.weight por agenda_item ≤ 100% (fração ideal).
- Vote com `closed_at != null` → sempre erro.
- Duplicate (agenda_item, voter) → sempre `ErrConflict`.
- Proxy: se `proxy_granted_to_user_id` preenchido, `voter_id = proxy_granted_to_user_id` e `on_behalf_of_user_id = original_voter`.

---

## 9. Video moderation bypass

### 9.1 Threats

| Threat | Vector | Controle |
|---|---|---|
| Upload bypass da moderação | Publicar direto sem aprovação | Estado `VideoModeration.state = pending` bloqueia `publish`; ABAC exige state=approved |
| Moderação stale (admin inativo > 48h) | Vídeos ficam pending indefinidamente | Alerta ops se fila > 48h; SLA moderação 48h |
| Moderador comprometido aprova malicious | Insider admin | Dual-approval M3 (2 moderadores concordam); audit_log todas aprovações |
| Publicar → apagar → republicar conteúdo diferente | Burlar lock 90d | Lock 90d enforced em `published_at` field; `unpublish + re-upload` vira novo asset (novo lock); transição explícita |
| Upload com conteúdo adulto/ilegal | Burla moderação | Moderação manual MVP + Mux audio/video analysis API M3+ + ML moderation M4+ |
| DMCA / direito autoral | Publicar conteúdo protegido | ToS + DMCA takedown process; audit de takedowns |

### 9.2 Playback leak

- Signed playback URL TTL 15min.
- CDN signed URL bound a `user_id` + `asset_id`.
- Preview 25% pra não-assinantes (Mux manifest trimming).
- Hotlink prevention via Referer check + signed URL.

---

## 10. Connect Me fraud

### 10.1 Fake RFP

| Threat | Controle |
|---|---|
| Síndico fantasma abre RFP pra drenar propostas | Membership `role=sindico` validado no tenant + quota por plano (N1: 2/ano, N2: 4/ano) |
| RFP com contato externo ("mande proposta pro email X") | Moderação de conteúdo RFP (campos estruturados; anexo opcional com scan) |
| Price fishing (coleta preços sem intenção) | Histórico de fechamento por síndico público; taxa de fechamento < X% flag |

### 10.2 Fake proposta

| Threat | Controle |
|---|---|
| Empresa não existe (CNPJ falso) | Validação CNPJ via Receita Federal API (M2+); bronze tier default até N contratos |
| Proposta fake pra subir reputação | Proposta só conta pra reputação se **fechada** (contrato formalizado + pagamento) |
| Empresa A finge ser Empresa B | Zitadel verifica CNPJ via documento; audit membership de Company |

### 10.3 Rating fraud

| Threat | Controle |
|---|---|
| Síndico avalia empresa parceira dele (conflito de interesse) | M3: detecção de padrão (mesmo síndico avalia mesma empresa > N vezes em tenants diferentes); flag pra review |
| Empresa compra avaliações | Avaliação só de síndico com contrato fechado real + pagamento; PBT: N avaliações ≤ N contratos fechados |
| Spam de denúncias | Rate limit denúncias; moderação de denúncias; contra-avaliação da empresa reportada |

---

## 11. Reputation gaming

### 11.1 Threats

| Threat | Vector | Controle |
|---|---|---|
| Farm de contratos pra subir tier | Empresa cria condomínios fantasmas + síndicos fantasmas + fecha contratos "com ela mesma" | Verificação condomínio real (CNPJ do condomínio — MP 2.230/01); N síndicos distintos no cálculo (unique condos ≥ threshold); detecção de anéis |
| Fake evaluations | Síndico fantasma avalia | Mesmo controle: síndico real + unit + CNPJ condo |
| Manipulação de fórmula | Tentar explorar edge case numérico | Fórmula pública + determinística + PBT cobrindo input space |
| Degradation attack (tier down rival) | Bombardear rival com avaliações 1-star | Rate limit avaliações por síndico-empresa-par; detecção de padrão coordenado |
| Tier freeze exploit | Empresa para de trabalhar mas mantém tier alto | **Decay**: avaliações antigas pesam menos (half-life 12m); tier recalcula mensal |

### 11.2 Invariantes PBT

- `tier = f(contratos, avaliações, condomínios, vídeos, tempo_plataforma)` é função pura.
- `Σ contratos ≥ Σ avaliações` (não pode avaliar sem contrato fechado).
- `unique(condominium_id) ≥ threshold_tier` pra cada tier.
- Nenhum input humano direto altera tier (sem override admin exceto flag `suspended`).

---

## 12. LINDDUN (privacidade LGPD)

| Threat | Aplicação master-sindico | Mitigação |
|---|---|---|
| **L**inkability | Correlacionar CPF entre tenants | CPF é único, mas `user_id` UUID é usado externamente; audit_log opcional vincula |
| **I**dentifiability | Identificar user via padrão de uso | Métricas agregadas only; sem tracking individual em dashboards externos |
| **N**on-repudiation (falha) | User nega ação | audit_log append-only + digital sign em atas (MP 2.200-2) |
| **D**etectability | Saber que X existe no sistema | Busca só `status=published`; perfis privados por default |
| **D**isclosure of information | Vazamento PII | Encryption at rest (Postgres) + in transit (TLS 1.3) + no log (§7 [[BEYOND_CORP]]) |
| **U**nawareness | User não sabe que dado foi coletado | Termos versionados + consent explícito + onboarding transparente |
| **N**on-compliance | Violar LGPD | Ver [[lgpd]] — consent, exclusão SLA 15d, portabilidade, breach 72h |

---

## 13. Superfícies específicas a vigiar (legado preserved)

### 13.1 Upload de vídeo (Mux)

- Saga A-027: órfão evitado via compensação (`CompensateVideoUpload`).
- Validação MIME/size no Mux direct upload (`allowed_asset_types`).
- Moderação manual antes de `published`.
- Trava 90d por política de churn (lock transition enforced em domain service).

### 13.2 Assembleia Live (Livekit)

- Saga A-028: room órfão evitado via compensação.
- JWT de participante gerado com ACL mínimo (microfone próprio, chat próprio).
- Retry + reconciliação em `ListParticipants` (A-033 Sprint 10 pendente).
- `EndRoom` idempotente (A-034 Sprint 10 pendente).

### 13.3 Stripe Webhook

- Idempotência por `event.id` (UNIQUE).
- Processamento defensivo: evento desconhecido → log warning + 200 OK (evita retry infinito).
- Review mensal de eventos ignorados.

### 13.4 Busca OpenSearch

- Resultados filtrados por tenant **antes** de retornar.
- Não indexar PII sensível (email completo, telefone) — usar hash se precisar match.
- Query timeout 2s.

---

## 14. Regras PBT (property-based) obrigatórias — consolidado

Herda legado + novas:

1. **Fração ideal**: Σ units.fracao_ideal ≤ 100.00% em qualquer condomínio.
2. **Quotas billing**: `Consume` nunca deixa saldo < 0; `Unlimited` nunca decrementa.
3. **ABAC matrix**: admin_bypass sempre PERMIT; tenant_mismatch sempre DENY; role_not_allowed sempre DENY; step_up_required sem mfa_fresh sempre DENY.
4. **Money BRL**: op sempre em int64 centavos; no floats.
5. **Tenant isolation**: cross-tenant read/write sempre 403 em qualquer combinação (100+ testes).
6. **Vote unique**: duplicate (agenda_item, voter) sempre `ErrConflict`.
7. **Vote weight**: vote.weight = user.unit.fracao_ideal (computado server-side).
8. **Timeline append-only**: nenhum UPDATE/DELETE grant em timeline_entries.
9. **Minutes immutable**: Minutes.publish → nenhum UPDATE permitido (domain invariant + DB trigger M2).
10. **Reputation deterministic**: `tier = f(inputs)` é função pura; mesmo input → mesmo output.
11. **Lock 90d video**: publish_date + 90d > NOW → `ErrLocked` em unpublish.
12. **HMAC constant-time**: toda verify signature usa `hmac.Equal()`.

---

## 15. Gaps + mitigações previstas (consolidado A-### esperados)

Findings abertos do legado + novos previstos pra Fase 5:

### Gaps herdados (A-### Sprint 10 pendentes)

- **A-020** N+1 UPDATE (institutional) — 🟡.
- **A-021** `SELECT *` em commercial — 🟡.
- **A-023** device change audit comparativo — 🟡 (impacto direto threat §4.1 Repudiation).
- **A-024** `rawBodyReader` interface privada — 🟡.
- **A-029** Company Evaluation TOCTOU — 🟡 (impacto §10 rating fraud).
- **A-030** Commercial Vote status race (voto pós-close) — 🟡 (impacto §8 assembly integrity pattern).
- **A-032** Goroutines lifecycle cleanup — 🟡.
- **A-033/034** Livekit retry — 🟡.

### Gaps novos esperados Fase 5 (A-SEC-### a nomear)

- **Gap-SEC-001**: device posture real (Play Integrity / DeviceCheck) ausente — threat §8 vote stuffing. **Mitigação**: M2.
- **Gap-SEC-002**: step-up MFA ausente em endpoints críticos — threat §4.2 EoP. **Mitigação**: M2.
- **Gap-SEC-003**: PostgreSQL RLS não implementado — threat §5.1 cross-tenant leak defesa em profundidade. **Mitigação**: M2.
- **Gap-SEC-004**: hash-chain no audit_log ausente — threat §12 Non-repudiation. **Mitigação**: M2.
- **Gap-SEC-005**: rate limit distribuído (Redis) ausente — threat §4.1 DoS em múltiplas réplicas. **Mitigação**: M2 quando escalar horizontalmente.
- **Gap-SEC-006**: WAF no edge não decidido — threat §4.2 DoS + bot protection. **Mitigação**: ADR-### M2.
- **Gap-SEC-007**: CNPJ validação via Receita Federal ausente — threat §10.2 fake proposta. **Mitigação**: M2 (API externa).
- **Gap-SEC-008**: reputation decay ausente — threat §11 tier freeze. **Mitigação**: M3 decay mensal.
- **Gap-SEC-009**: dual-approval moderation ausente — threat §9 moderator compromise. **Mitigação**: M3.
- **Gap-SEC-010**: reputation ring detection ausente — threat §11 farm contracts. **Mitigação**: M4 ML-based.
- **Gap-SEC-011**: Mux antivirus scan ausente — threat §4.5 upload malware. **Mitigação**: M4+.
- **Gap-SEC-012**: PDP sidecar (OPA) ausente — threat §5.2 ABAC lógica espalhada. **Mitigação**: M2 ADR.
- **Gap-SEC-013**: mTLS leste-oeste ausente — threat §3 TZ 3→TZ 4 network compromise. **Mitigação**: M2 (3 caminhos críticos) + M3 (mesh).
- **Gap-SEC-014**: continuous auth / risk scoring ausente — threat §4 generic E**+D. **Mitigação**: M3.
- **Gap-SEC-015**: secret manager (AWS SM) ausente — threat §6 provider key leak + rotação manual. **Mitigação**: M2 quando migrar ECS.

Esses Gaps viram **A-SEC-### em [[../AUDIT]]** (Fase 5 pentester pass).

---

## 16. Atualização e revisão

Este doc é revisado:
- **Trimestralmente** (review completo; update de gaps resolvidos).
- Quando **feature nova** atravessa nova superfície pública ou cross-tenant.
- Em **incident retrospective** se expôs threat nova (postmortem → update).
- Em **pentest externo** — findings viram A-SEC-### + update aqui.

---

## 17. Links

- [[_moc]]
- [[BEYOND_CORP]] — pilar central
- [[owasp-top10]] — overlay OWASP
- [[cve-process]] — vulnerability management
- [[lgpd]] — privacidade (LINDDUN concreto)
- [[pentest-checklist]] — checklist pentester
- [[beyond-corp-references]] — fontes
- [[../AUDIT]] — findings A-###
- [[../13-research/beyond-corp/_destilado]]
- [[../13-research/beyond-corp/source-02|NIST SP 800-207]]
- [[../00-product/portfolio-de-produtos]] — sub-produtos e BCs
- [[../01-domain/_moc]] — (a escrever) bounded contexts
