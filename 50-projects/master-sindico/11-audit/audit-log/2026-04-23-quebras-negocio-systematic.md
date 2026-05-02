---
title: Quebras de negócio sistemáticas — pentester + risk analyst + QA adversarial
type: audit
tags:
  - audit-log
  - pentest
  - risk-analysis
  - quebras-negocio
  - master-sindico
  - v2
  - fase-5
  - adversarial
  - race-conditions
  - fraud
  - edge-cases
  - data-loss
  - compliance
source: >-
  leitura integral dos 13 arquivos fonte (invariants, business-rules,
  state-machines, billing.md, assembly.md, commercial.md, threat-model,
  pentest-checklist, aggregates/Subscription + FeatureUsage, sub-produtos
  03/05/08 + AUDIT)
created: 2026-04-23T00:00:00.000Z
updated: 2026-04-23T00:00:00.000Z
status: initial
dual_check: pending
---

# Quebras de negócio — Master Síndico v2 (Q-### systematic)

Finding exaustivo focado **exclusivamente** em quebras de negócio (race conditions, fraudes, edge-cases, perda de dados, compliance). Nenhum finding já coberto em `AUDIT.md` é repetido.

**Severidade**: 🔴 bloqueia M1 / 🟠 high (primeira semana pós-M1) / 🟡 médio (Sprint 11).
**Categorias**: `race` | `fraud` | `edge-case` | `data-loss` | `compliance`.

---

## §1. Sumário executivo

| Severidade | Race | Fraud | Edge-case | Data-loss | Compliance | **Total** |
|---|---|---|---|---|---|---|
| 🔴 | 3 | 3 | 3 | 3 | 3 | **15** |
| 🟠 | 2 | 3 | 3 | 2 | 2 | **12** |
| 🟡 | 1 | 0 | 1 | 1 | 1 | **4** |
| **Total** | **6** | **6** | **7** | **6** | **6** | **31** |

**Findings novos**: 31 (meta ≥25 atingida). Nenhum duplica A-### / A-DC-### / A-FASE8-### existentes.

**Top bloqueadores M1**: Q-014 (ata UPDATE direto DB), Q-005 (homologação reversível), Q-021 (LGPD erasure vs 5y), Q-028 (edital <5d proibido mas Lei 4.591 exige 8d), Q-002 (cupom lock 4h Redis), Q-018 (Stripe webhook downtime), Q-009 (síndico destituído mid-assembleia), Q-023 (Mux upload órfão quota), Q-010 (plano cancelado mid-upload), Q-013 (síndico 15 condomínios destituído).

---

## §2. Quebras Q-### por categoria

### Categoria 1 — Race conditions / concorrência (6 findings)

---

#### Q-001 — Voto simultâneo em 2 devices da mesma conta burla "1-device" (INV-005) antes da invalidação

- **Severidade**: 🔴
- **Categoria**: race
- **Onde**: `01-domain/invariants.md` INV-005 + `01-domain/state-machines.md` §7 (User Session) + `04-requirements/functional/assembly.md` ASM-015
- **Como explorar**: atacante abre app em device A, faz login, abre app em device B, faz login (sessão A é invalidada via `InvalidatePreviousByUserID`). Dentro da janela de **~50-200ms** entre o commit da sessão B e a revogação propagada (cache Redis TTL 5min — INV-100), atacante na aba aberta de device A (que ainda não recebeu 401) posta `POST /api/v1/assemblies/{id}/votes` simultaneamente com device B. Ambos podem passar pelo middleware auth se consultam cache stale. UNIQUE(agenda_item_id, voter_id) impede duplicata, **mas** o primeiro a chegar no DB vence, permitindo que o dono do device A (supostamente revogado) vote antes do legítimo device B — com isso o atacante fixou voto em contradição à intenção atual.
- **Impacto**: voto fraudulento em assembleia; ata contém registro inconsistente → nulidade (Lei 4.591/64 Art. 24); reputação da plataforma judicializada.
- **Gap atual**: INV-005 enforça no UoW de criação de session, mas nada enforça que o **voto** valide `session.created_at` mais recente que o current request's session. O cache ABAC é baseado em `ms:abac:v{N}:{user}:{tenant}` mas não revalida session age per-request.
- **Correção nominal**: adicionar INV-111 em `01-domain/invariants.md` §identity — "Vote handler revalida server-side `session.created_at >= MAX(sessions.created_at WHERE user_id = $1)` dentro da UoW do voto; mismatch → 401 + audit"; alternativamente, pub/sub NATS `session.revoked` dispara DEL de cache ABAC do user (<100ms).

---

#### Q-002 — Cupom Vizinhança: lock 4h via Redis é TOCTOU — 2 resgates concorrentes passam se Redis crash entre SET e TTL

- **Severidade**: 🔴
- **Categoria**: race
- **Onde**: `01-domain/invariants.md` INV-051 + `04-requirements/functional/billing.md` BIL-034 + `00-product/sub-produtos/08-vizinhanca.md` §8.3/§10.3
- **Como explorar**: morador com 2 devices (burlado — ver Q-001), ou mesmo device com 2 requests paralelas via browser: POST `/coupons/generate?partner_id=X` duas vezes em < 50ms. Ambas consultam Redis `ms:coupon_lock:{user}:{partner}` → MISS (não existe ainda). Ambas executam `SELECT NOT EXISTS(...WHERE generated_at > NOW()-4h)` em conexões DB separadas → ambas retornam `false` (sem cupom). Ambas inserem em `coupon_generations`. Resultado: 2 cupons gerados em < 4h, violando INV-051. `unique(partner_id, status='active')` proposto em `08-vizinhanca.md` §8.3 só impede 2 cupons **ativos** globalmente no partner — não 2 do **mesmo user** no mesmo partner dentro de 4h.
- **Impacto**: spam vale-ouro (10 cupons/hora se automatizado); parceiro explorado financeiramente; confiança no "lock anti-spam" erodida.
- **Gap atual**: spec menciona "trava 4h via Redis" + "UNIQUE parcial + index" — mas índice proposto em INV-051 é `(user_id, vizinhanca_user_id, created_at)` não-único (só acelera query); sem CHECK constraint nem `SELECT FOR UPDATE`.
- **Correção nominal**: em `invariants.md` INV-051 trocar enforcement para: "UNIQUE parcial PARTIAL DEFERRED `(user_id, partner_id) WHERE generated_at > NOW() - interval '4h'`" — **não suportado em PG**, então alternativa: tabela `coupon_cooldowns (user_id, partner_id, locked_until)` com `UPSERT ... WHERE locked_until < NOW()` atomically em transação + RETURNING. Registrar em `04-requirements/functional/billing.md` BIL-034 a obrigatoriedade de UoW + `SELECT FOR UPDATE SKIP LOCKED`.

---

#### Q-003 — Connect Me quota decrementada duas vezes se publish + compensate racing (quota negativa ou negada legítima)

- **Severidade**: 🟠
- **Categoria**: race
- **Onde**: `01-domain/invariants.md` INV-045, INV-016 + `01-domain/aggregates/FeatureUsage.md` INV-FU-009 + `04-requirements/functional/commercial.md` COM-006
- **Como explorar**: síndico Plus (quota 4 Connect Me/ano) envia 3 requests paralelas via script em < 100ms quando `quota_used=3` (resta 1). Cada request entra em seu próprio UoW com `SELECT FOR UPDATE` — seriam serializadas. Mas a factory `NewFeatureUsage` roda fora do lock no **primeiro** request do ano (quando FeatureUsage ainda não existe): cada request tenta `INSERT ... ON CONFLICT (user_id, feature_key, period_key) DO NOTHING` — uma ganha o insert, duas dão `ON CONFLICT`. Se as duas reprocessam o lookup (`FindByUserForUpdate`) sem lock adequado, podem ver `quota_used=3` e ambas decrementar para `used=4` passando ambas — ou ambas reportarem `ErrQuotaExceeded` mesmo com quota disponível.
- **Impacto**: (a) fraude quota negativa (síndico passa de 4 → 5/ano); (b) falso-negativo nega ação legítima e quota não reembolsada (BR-021 "quota NÃO é devolvida").
- **Gap atual**: `INV-FU-009` exige `FindByUserForUpdate` mas não trata o caso de **primeira criação** (exemplo COM-006 em aggregates/FeatureUsage.md §`PublishConnectMeUseCase` usa `errors.Is(err, repoerr.ErrNotFound)` sem lock do upsert).
- **Correção nominal**: em `aggregates/FeatureUsage.md` §Métodos adicionar método `FindOrCreateForUpdate(ctx, userID, feature, periodKey, plan)` que faz `INSERT ... ON CONFLICT DO UPDATE SET updated_at=NOW() RETURNING *` dentro do lock. Adicionar INV-FU-011 em `invariants.md`: "FeatureUsage criação-ou-consumo é atomic upsert com `RETURNING *` — sem lookup-then-insert patterns".

---

#### Q-004 — Banco de Talentos: substituir vídeo pós-90d concorrente com "3º vínculo" gera arquivamento inconsistente

- **Severidade**: 🟡
- **Categoria**: race
- **Onde**: `01-domain/invariants.md` INV-059 (trava trimestral) + `04-requirements/functional/commercial.md` COM-029 + `00-product/sub-produtos/07-banco-de-talentos.md` (referenciado)
- **Como explorar**: morador Pagante com vídeo-currículo publicado dia 90 exato dispara dois uploads simultâneos (web + mobile). `INV-059` valida `NOW() >= original.published_at + 90d` na factory `Video.Replace(original_id)` — passa em ambos. Ambos invocam Mux presigned → 2 assets novos criados. Only one consegue update em `curriculum_videos` (UNIQUE por user_id); outro fica órfão Mux → compensator saga (INV-065) apaga, **mas** se a ordem for (a) asset_ready_1 → publish → delete asset_2, (b) asset_2 era-sem-ser já está viável para publicar e o user poderia ter 2 currículos simultâneos por ~1s.
- **Impacto**: janela curta de violação semântica "1 vídeo-currículo por user" (COM-029); em casos extremos 2 ofertas simultâneas a empresas Plus/Pro navegando Banco de Talentos.
- **Gap atual**: não há lock pré-upload em `curriculum_videos.user_id`; Saga UploadVideo compensa órfãos Mux mas não impede 2 inflight.
- **Correção nominal**: em `04-requirements/functional/commercial.md` COM-029 adicionar: "pre-emissão de presigned URL, lockar row `curriculum_videos WHERE user_id=$1 FOR UPDATE`; se upload in-flight existe (< 1h timeout), 409"; em `invariants.md` INV-059 enforcement trocar para "factory + UNIQUE(user_id) em curriculum_videos + lock pré-Mux".

---

#### Q-005 — Homologação de ata (INV-085): race entre segunda votação aprovando e "cancelar homologação" admin cria ata sem estado consistente

- **Severidade**: 🔴
- **Categoria**: race
- **Onde**: `01-domain/invariants.md` INV-072, INV-085 + `01-domain/state-machines.md` §3 Assembly + `01-domain/business-rules.md` BR-049
- **Como explorar**: ata está em `closed`; `HomologateMinutesUseCase` está rodando (votação de maioria simples aprovada → transição `closed → homologated` prestes a commit). Simultaneamente admin dispara "cancelar assembleia" ou outra ação mutativa. Se segunda votação termina com maioria e publica `homologated_at`, mas antes do commit outro handler (ex: correção Timeline via `corrects_entry_id`) mexer no snapshot de frações... pior: `closed → homologated` **e** retorno `closed → canceled` podem colidir se rotas admin não se coordenam com a rota de homologação. State machine §3 declara `homologated → qualquer` proibido, mas **não** declara `closed → canceled` proibido após reopen; se admin canceler um já-`closed`, ata fica em limbo.
- **Impacto**: ata "homologada mas cancelada" = peça jurídica inválida; judicial suspeita-direta; invariante INV-072 violada em semântica.
- **Gap atual**: state-machines §3 lista apenas transições positivas. `scheduled/published → canceled` está documentado, mas `closed → canceled` **não está listado proibido** — interpretação aberta. INV-072 diz "imutável após `published`+`homologated`" mas não cobre hard-delete por admin.
- **Correção nominal**: em `01-domain/state-machines.md` §3 Assembly §Transições proibidas adicionar: "❌ `closed → canceled` (assembleia encerrada é terminal exceto p/ homologação); ❌ `homologated → canceled` (imutável absoluto)"; em `invariants.md` INV-072 adicionar "após `closed_at != NULL` apenas transição `closed → homologated` é permitida; admin cancel só em `scheduled|published`".

---

#### Q-006 — Reputação recompute síncrono UoW em 2 CompanyEvaluationSubmitted simultâneas calcula sobre snapshot inconsistente

- **Severidade**: 🟠
- **Categoria**: race
- **Onde**: `01-domain/invariants.md` INV-036, INV-054 + `00-product/sub-produtos/03-reputacao.md` §7 + `04-requirements/functional/commercial.md` COM-025
- **Como explorar**: empresa Platina recebe 2 avaliações 1/5 simultâneas de síndicos diferentes. Cada UoW roda `ReputationCalculator.Recompute(companyID)` consultando `SELECT COUNT(*), AVG(...) FROM company_evaluations WHERE company_id=X`. UoW_A ainda não commitou a evaluation A quando UoW_B dispara recálculo → UoW_B vê apenas 1 nova avaliação, não 2; score calculado é entre o real antigo e o real novo (valor intermediário errado). Segundo recálculo rola quando commitar A → mas entre os dois commits o evento `ReputationTierChanged` foi emitido com tier errado, UI exibiu e notificou empresa com tier inválido.
- **Impacto**: (a) empresa recebe notificação errada (rebaixada quando na verdade rebaixada mais); (b) `reputation_snapshots` gravado com valores inconsistentes polui auditabilidade LGPD Art. 20; (c) Connect Me ranking usa tier-bonus (ADR-0023) errado por curto período → empresa com tier errado ganha vantagem comercial ilegítima.
- **Gap atual**: `sub-produtos/03-reputacao.md` §7 diz "síncrono na UoW" mas sem lock explícito em `companies.id FOR UPDATE` antes do recálculo; 03-reputacao.md P-SM-006 reconhece pendência mas sem fix.
- **Correção nominal**: em `aggregates/Company.md` (via reputacao.md §10.1) documentar: "`RecalculateReputation(companyID)` obrigatório dentro de `SELECT ... FROM companies WHERE id=$1 FOR UPDATE`"; em `invariants.md` adicionar INV-112: "Reputation recompute serializado por company_id via pg advisory lock (`pg_advisory_xact_lock(hash(company_id))`) OU `FOR UPDATE`".

---

### Categoria 2 — Fraudes exploráveis (6 findings)

---

#### Q-007 — Síndico cria 15 condomínios fictícios pra multiplicar quota Connect Me anual via trial "renovável"

- **Severidade**: 🔴
- **Categoria**: fraud
- **Onde**: `01-domain/invariants.md` INV-027 + `04-requirements/functional/billing.md` BIL-032 + BIL-030 + `01-domain/business-rules.md` BR-007, BR-015
- **Como explorar**: atacante cria 1 conta síndico (trial 15d Plus — 4 Connect Me/ano na matriz de quotas). Em 15d consume 4. Faz downgrade para Base → consumir Connect Me 0/ano (BIL-030). Cancela, cria **nova conta** com CPF diferente (familiar, laranja); novo trial 15d; 4 Connect Me novos. Alternativa mais sofisticada: síndico Pro (ilimitado Connect Me) cria 15 condomínios (INV-027 hard limit) mas 2 desses condomínios são fictícios (CNPJ condomínio validado via Receita Federal é GAP-SEC-007 — só stub M1). Cria RFPs de Connect Me apontando a condomínios fictícios → empresas reais respondem → dados vazam sem contrato real → price fishing (§10.1 threat-model).
- **Impacto**: (a) trial bypass serial multiplicando cadastros; (b) uso abusivo de quota para espionagem comercial (empresas revelam preços achando que vão fechar); (c) drenagem de tempo de empresas respondendo RFPs fantasma — custo operacional de concorrentes, reputação plataforma.
- **Gap atual**: BR-007 diz "15 condomínios por síndico hard limit" mas validação de realidade do condomínio é deferida (CNPJ do condomínio validação real só M2+); BIL-014 impede trial novo com mesmo CPF, **mas** não impede cadastro com CPF diferente (laranja). Nenhum invariante cruza `condominiums` e `connect_me_requests` com realness signals (ex: ≥1 morador real cadastrado por X dias antes de liberar Connect Me no condomínio).
- **Correção nominal**: em `04-requirements/functional/billing.md` BIL-014 estender: "CPF/CNPJ + device fingerprint + IP cluster check → bloqueio de trials seriais"; em `04-requirements/functional/commercial.md` COM-006 adicionar pré-condição: "Connect Me from syndic requires `condominium.status = active AND condominium.residents_count ≥ 3 AND condominium.created_at < NOW() - 30d` (condomínio maduro)"; em `invariants.md` adicionar INV-113: "ConnectMeRequest source tenant precisa `maturity_score ≥ threshold` (residentes ≥3, idade ≥30d, ≥1 validação externa)".

---

#### Q-008 — Empresa avalia a si mesma via conta laranja morador (ou: síndico-empresa vota em contrato da própria empresa)

- **Severidade**: 🔴
- **Categoria**: fraud
- **Onde**: `01-domain/invariants.md` INV-043 + `00-product/sub-produtos/03-reputacao.md` §11.4 + `04-requirements/functional/commercial.md` COM-024
- **Como explorar**: empresa X de limpeza cria conta **morador** laranja em condomínio onde ela presta serviço (ou em qualquer condomínio dummy); contrata a si mesma via proposta fantasma → contrato fecha com duplaSignature → execução registrada → síndico laranja valida → evaluation submetida 10/10 em todos 5 critérios. INV-043 UNIQUE(company_id, evaluator_user_id, condominium_id) não bloqueia isso — o evaluator (morador/síndico laranja) é **diferente** do owner da empresa. Nenhum invariante cross-verifica `company.owner_user_id != evaluator_user_id` nem CPF equivalence nem CNPJ family tree.
- **Impacto**: empresa sobe para Platina/Diamante artificialmente; contaminação de reputação → síndicos legítimos contratam baseado em signal envenenado; "empresa fraudulenta premiada" estoura no jornal.
- **Gap atual**: threat-model §10.2/§11.1 cita "farm contracts" / "fake evaluations" mas gap-SEC-010 (reputation ring detection) é M4+. Invariantes de commercial INV-036..055 não enforçam **separação entre evaluator e company owner** nem **síndico-empresa conflict of interest** (BR-112 implícito em `conflict_of_interest_declarations` INV-035 é **apenas do síndico**, não da **empresa**).
- **Correção nominal**: em `invariants.md` adicionar INV-114: "CompanyEvaluation rejeita se `evaluator_user_id == company.owner_user_id` OR `evaluator.cpf_household == company.representante_legal.cpf_household` OR `company.owners` contém evaluator" + INV-115: "síndico declarou `conflict_of_interest_declaration` por empresa X → SupplierVoteCast + CompanyEvaluation em X retornam 403 `CONFLICT_OF_INTEREST`"; em `04-requirements/functional/commercial.md` COM-024 documentar guards.

---

#### Q-009 — Admin moderator ban síndico em condomínio A, síndico elegível em condomínio B → ban não-transitivo = reincidência

- **Severidade**: 🟠
- **Categoria**: fraud
- **Onde**: `01-domain/invariants.md` INV-026, INV-027 + `04-requirements/functional/billing.md` BIL-032 + `01-domain/business-rules.md` BR-012 (Subsíndico/Conselho)
- **Como explorar**: síndico cometeu fraude grave em condomínio A (ex: desvio documentado na Timeline após auditoria); admin MS executa ação de "banir síndico do condomínio A" (mandate.ended_at=NOW(), membership revogada). Mas síndico mantém mandate ativo em condomínio B (INV-026 é por `(user, condominium)`, não global). Ele continua criando Connect Me, votando, assinando contratos. Pior: após 1h ele pode criar **novo condomínio** (até INV-027 hard limit 15) e trazer moradores via invite.
- **Impacto**: fraude reincidente; condomínios C, D, E ficam expostos sem saber do histórico de A; moradores que vêem "síndico banido" de outro condomínio não têm sinal; dano reputacional direto.
- **Gap atual**: INV-027 é "15 condomínios por síndico" mas **não** cruza com "moderação: bans por condomínio somam para fraud score". Nenhum flag `user.fraud_strike_count` nem `user.banned_from_any_syndic_role_at`.
- **Correção nominal**: em `01-domain/aggregates/` criar seção ModerationAction / adicionar a `User`: campo `fraud_strikes_count INT` + trigger: ao banir mandate em 1 condomínio via fraud confirmada, dispara `UserFraudStrikeAdded` → após 2 strikes, `user.can_be_syndic = false` (similar a Q-008 INV-115); em `invariants.md` adicionar INV-116 "Ban por fraude em condomínio N propaga soft-ban em `assign_syndic_mandate` novos (requer 4-eyes admin unlock)"; em `04-requirements/functional/institutional.md` documentar hook no onboarding de síndico.

---

#### Q-010 — Delegação agência: agência pega empresa Plus/Pro cliente, exfiltra Banco de Talentos via grant escopo amplo

- **Severidade**: 🟠
- **Categoria**: fraud
- **Onde**: `01-domain/invariants.md` INV-049, INV-066 + `04-requirements/functional/commercial.md` COM-040, COM-041 + `01-domain/business-rules.md` BR-020, BR-038
- **Como explorar**: empresa Plus contrata agência marketing X (ABAC policy `delegated_marketing` → `videos:upload|edit, analytics:read` — INV-049 "zero acesso a deals/propostas/Connect Me"). Agência precisa, porém, "olhar currículos para fazer posts" — é feature de COM-028 (Banco de Talentos Plus/Pro tem acesso). INV-049 não menciona `talent:read`. Como a matriz de empresa Plus **tem** acesso a Banco de Talentos, e delegação transitivamente concede subset dos escopos de vídeo+analytics, agência pode acessar via rotas genéricas `/talents` se ABAC engine usa `user.current_company_id` do X sem filtrar `acting_as_agency=true`. Agência baixa currículos (com PII) para "posts" e revende para caçadores de talentos fora da plataforma.
- **Impacto**: vazamento LGPD massivo (CPF + foto + dados pessoais de moradores Pagantes); agência usa infra Master Síndico como scraper; multa LGPD Art. 52.
- **Gap atual**: INV-049/INV-066 listam "videos:*, analytics:read" permitidos — mas Banco de Talentos (`talents:read`) não está em allowlist nem blocklist explicit. Fallback default-deny (INV-087) **deveria** negar, mas se a rota genérica `/api/v1/users/{id}?include=curriculum_video` passa check `videos:read` como alias, vaza.
- **Correção nominal**: em `invariants.md` INV-049 reforçar: "ABAC `delegated_marketing` nega `talents:*, curriculum_videos:read, resident.pii:*` explicitamente"; em `04-requirements/functional/commercial.md` COM-028 adicionar: "acesso a Banco de Talentos requer `is_acting_as_agency=false` no token"; criar A-### novo em `AUDIT.md` exigindo pentest de todas as rotas empresa-scope x delegação para detectar bypasses.

---

#### Q-011 — Cupom Vizinhança ID_LOJA+SEQ+PALAVRA: SEQ previsível permite brute-force cupons de terceiros (ou falsificação)

- **Severidade**: 🔴
- **Categoria**: fraud
- **Onde**: `01-domain/invariants.md` INV-052, INV-053 + `00-product/sub-produtos/08-vizinhanca.md` §10 + `04-requirements/functional/commercial.md` COM-033
- **Como explorar**: formato `ID_LOJA(5) + SEQ(3) + PALAVRA(5)` = 13 chars. `ID_LOJA` é público (está no perfil do parceiro), `PALAVRA` é pública (palavra do dia — COM-034). `SEQ` é sequencial (000-999) e cresce 1 a cada cupom emitido. Atacante com cupom `00123055PAO` sabe que último cupom emitido hoje foi SEQ=055; tenta `00123054PAO`, `00123053PAO`, etc., apresentando visualmente no estabelecimento. Se parceiro não valida online no momento da ativação (espera "apresentar tela"), cupom anterior de outro morador pode ser reapresentado. Pior: atacante conhecendo parceiros vizinhos advinha SEQ atual.
- **Impacto**: fraude contra parceiros; cupom de morador X apresentado por morador Y; morador X reclama (seu cupom já foi resgatado sem seu uso); perda de confiança no sistema.
- **Gap atual**: INV-052 define formato mas não entropia. SEQ=3 dígitos = 1000 combinações, PALAVRA pública → 5 chars upper = razoável, mas se PALAVRA é a mesma do dia para todos os cupons daquele parceiro, **só SEQ varia** = 1000 tentativas.
- **Correção nominal**: em `invariants.md` INV-052 redesenhar: "CouponCode formato `ID_LOJA(5) + HMAC-randomSuffix(8)` — SEQ+PALAVRA sozinhos insuficientes contra brute-force"; em `sub-produtos/08-vizinhanca.md` §10 adicionar: "ativação exige validação server-side contra tabela `coupon_redemptions (coupon_code, redeemed_by)` UNIQUE com `status=active`; apresentação visual no parceiro exige scan QR com token assinado TTL 5min". Abrir A-### em AUDIT sobre o trade-off UX vs fraud.

---

#### Q-012 — CNPJ empresa forjado no signup vira tenant ativo: validação Receita é stub M1

- **Severidade**: 🟠
- **Categoria**: fraud
- **Onde**: `04-requirements/functional/commercial.md` COM-001, COM-005 + `01-domain/invariants.md` INV-003 + `08-security/threat-model.md` §10.2 Gap-SEC-007
- **Como explorar**: atacante cria empresa com CNPJ que passa apenas dígito verificador (INV-003 + COM-005 M1 = stub). Entra em trial 7d, vira ativa (status=active) sem validação Receita, submete propostas a síndicos, fecha deals, recebe dados PII de condomínio, pagamento via dupla assinatura. Em M2+ (validação Receita plena) o tenant já está dentro com histórico.
- **Impacto**: phishing B2B; extração massiva de PII (síndicos e condomínios); contrato assinado com entidade fictícia — litígio impossível de recuperar dinheiro ou responsabilizar.
- **Gap atual**: COM-005 é S (M2+), Gap-SEC-007 mencionado mas mitigação "M2" — no M1 qualquer CNPJ matemático válido passa.
- **Correção nominal**: em `04-requirements/functional/commercial.md` COM-005 elevar para M (M1) com stub **exigindo** verificação manual de documento enviado (RG representante + contrato social PDF) — sem validação Receita, transição de `pending_verification → active` bloqueada por admin review. Ou: abrir A-### e D-### decidindo se M1 faz validação manual ou se aceita risco trial-only (não deal-ready).

---

### Categoria 3 — Edge cases em estados críticos (7 findings)

---

#### Q-013 — Síndico destituído mid-assembleia: votação prossegue? voto dele conta?

- **Severidade**: 🔴
- **Categoria**: edge-case
- **Onde**: `01-domain/state-machines.md` §3 Assembly + `01-domain/invariants.md` INV-081 (presidente não vota) + `04-requirements/functional/assembly.md` ASM-014, ASM-017 + `01-domain/business-rules.md` BR-048
- **Como explorar**: assembleia `in_progress`, item da pauta é "destituição do síndico". Votação aprova destituição — síndico é ex-síndico em t+1s. **Mas** síndico é o presidente (INV-081: "presidente síndico **não** vota" — imparcialidade). Paradoxo: se ele estava proibido de votar enquanto presidente, quem preside após destituição? Continua presidindo como ex-síndico (sem autoridade jurídica)? Ou assembleia entra em limbo? **E** se ele votou em itens anteriores **como morador** (não presidente), isso conta? **E** se imediatamente ele abre novo item do mesmo turno e é destituído, os votos dele nos itens anteriores se anulam retroativamente?
- **Impacto**: assembleia judicialmente contestável; ata contém incoerências; litígio prolongado.
- **Gap atual**: state-machines.md §3 não cobre transição implícita `membership.ended_at` (institutional) → assembly.current_president_id update. ASM-017 diz "`role_type=principal` ou designado" mas não define o que acontece **durante** uma sessão live quando o role muda. BR-048 é silencioso sobre destituição.
- **Correção nominal**: em `01-domain/state-machines.md` §3 adicionar seção "Mandate change mid-assembly": "destituição de síndico durante assembly in_progress pausa a sessão (`status: in_progress → paused_reorg`), conselho ou subsíndico (INS-004) assume presidência; votos pré-destituição permanecem imutáveis; votação corrente é anulada e re-aberta"; em `invariants.md` adicionar INV-117: "syndic destituído mid-assembly → `assembly.status=paused_reorg`, `assembly.current_president_id` re-elegido por conselho"; `04-requirements/functional/assembly.md` ASM-014 documentar.

---

#### Q-014 — Morador transferido de unidade mid-votação: fração ideal snapshot ao publish, mas ele foi para outra unidade — voto peso errado

- **Severidade**: 🟠
- **Categoria**: edge-case
- **Onde**: `01-domain/invariants.md` INV-083 (snapshot ao publish), INV-029 (1 titular por unidade), INV-073 (quórum fracional) + `04-requirements/functional/assembly.md` ASM-007 + `01-domain/state-machines.md` §3
- **Como explorar**: morador A é titular da unit 101 (fração 2.5%). Pauta publicada (snapshot congela). Transferência de titularidade 101 → B, A vira titular 202 (fração 3.0%). Assembleia in_progress. A faz check-in via CPF — sistema reconhece como morador. Mas qual fração ele carrega? **Snapshot** do publish (2.5% unit 101) ou **atual** (3.0% unit 202)? INV-083 diz "fracao_ideal_weight vem de unit.fracao_ideal no momento do voto (snapshot)" — snapshot **do voto**, não do publish. Conflito com INV-073 que diz "quorum calculado por Σ FracaoIdeal presente". Se snapshot publish diz 101=A, mas atual diz 101=B, A aparece duas vezes? B não dá check-in (não é mesmo user)?
- **Impacto**: quórum calculado errado → assembleia declarada aprovada com quórum insuficiente → nulidade; ou rejeitada com quórum suficiente → minoria impõe.
- **Gap atual**: INV-083 vs INV-073 vs ASM-007 snapshot: 3 invariantes, semântica ambígua (snapshot do voto = unit atual do user? ou unit registrada ao publish?); state-machines.md §3 não lista "mandate/titular change mid-assembly".
- **Correção nominal**: em `invariants.md` INV-083 precisar: "`fracao_ideal_weight` = `assembly_fractions.fracao_ideal WHERE assembly_id=X AND unit_id = (morador.unit_at_snapshot_time)`"; em `04-requirements/functional/assembly.md` ASM-007 adicionar: "membership.user→unit transfers during assembly `in_progress` são bloqueadas via `assembly_lock_memberships` UoW; fluxo só libera pós-encerramento"; em `invariants.md` INV-118: "mid-assembly unit transfer → 409 `ASSEMBLY_IN_PROGRESS`".

---

#### Q-015 — Procuração revogada mid-votação: voto do procurador já lançado expira retroativamente? (vs INV-079 "revogar até 1h antes")

- **Severidade**: 🟠
- **Categoria**: edge-case
- **Onde**: `01-domain/invariants.md` INV-077, INV-078, INV-079 + `04-requirements/functional/assembly.md` ASM-010 + `01-domain/business-rules.md` BR-047
- **Como explorar**: procurador P vota em nome do titular T no item 1 (voto registrado às 09:03). Às 09:15 T aparece fisicamente na assembleia, quer revogar procuração. INV-079 diz "revogar até 1h antes" — T passou do prazo, revogação bloqueada. T não pode votar (INV-077 "proxy não pode votar diferente do titular" — mas titular não votou ainda, só o procurador, então podeT votar?). Pior: no item 2, P abstém-se (não vota). T chega e vota no item 2 diretamente — passa? (sim, não há voto do titular no item 2 ainda). E se P votou "sim" no item 3 mas T quer "não"? INV-077 diz "proxy não pode divergir" mas **P votou antes** — não sabia intenção de T. State machine permite este deadlock.
- **Impacto**: votação nula por conflito de vontade; ata contesta na justiça; invariante INV-079 estática não cobre caso dinâmico.
- **Gap atual**: INV-077/078/079 tratam procuração como unit-of-validation (prazos, unicidade), não como estado dinâmico com prioridade titular-vs-procurador.
- **Correção nominal**: em `invariants.md` adicionar INV-119: "quando titular T fez check-in fisicamente durante assembly_in_progress, votos do procurador P em itens ainda-abertos são automaticamente revogados (`status=superseded_by_titular`) e T pode votar; votos P em itens já-fechados permanecem"; em `04-requirements/functional/assembly.md` ASM-010 documentar fluxo de "check-in titular supersede proxy"; em state-machines §5 VoteSession adicionar transição `proxy_vote → superseded`.

---

#### Q-016 — Plano cancelado mid-upload vídeo institucional: upload termina? vídeo aparece? quota consumida?

- **Severidade**: 🔴
- **Categoria**: edge-case
- **Onde**: `01-domain/invariants.md` INV-019 (soft-block sem grace), INV-065 (Saga UploadVideo), INV-045 (quota publish-time) + `04-requirements/functional/billing.md` BIL-011 + `01-domain/state-machines.md` §1
- **Como explorar**: empresa Plus inicia upload de vídeo institucional (15min de upload para vídeo 10min). Durante upload, subscription é cancelada via webhook Stripe (ex: chargeback). INV-019 diz "soft-block imediatamente ao expirar". Upload continua porque Mux é external. Mux webhook `video.ready` chega 20min depois — saga UploadVideo persiste `videos` row. Mas subscription está canceled → ABAC deveria negar publish. **Edge case**: quota foi "reservada" ao iniciar upload? Spec INV-045 diz "quota decrementa no publish, não no draft" — upload é draft. Então quota está intacta. Video row existe mas inaccessible. **Limpeza**: video órfão custa storage Mux, quem paga?
- **Impacto**: dados órfãos em Mux (custo operacional); video "fantasma" poderia ser resgatado se subscription reativada — mas política é unclear; worst case: empresa processa pleiteando "vídeo pago antes do cancel".
- **Gap atual**: INV-065 Saga UploadVideo compensa falha local mas não cobre "política cancelou durante upload"; BIL-011 soft-block imediato mas sem policy sobre assets em Mux in-flight.
- **Correção nominal**: em `invariants.md` INV-120 novo: "Saga UploadVideo valida `subscription.GrantsAccess() == true` no webhook Mux `video.ready` (não só no `presigned URL`); se false ao processar ready → compensate `CancelMuxUpload` + audit log + email user explicando"; em `04-requirements/functional/billing.md` BIL-011 + BIL-031 documentar política de assets in-flight ao expirar.

---

#### Q-017 — Empresa pausada mid-execução registrada: registro visível? reputação afeta?

- **Severidade**: 🟡
- **Categoria**: edge-case
- **Onde**: `01-domain/invariants.md` INV-055 (flag suspended ≠ tier change) + `04-requirements/functional/commercial.md` COM-020, COM-021 + `00-product/sub-produtos/03-reputacao.md` §9
- **Como explorar**: empresa Y tem deal ativo, registrou execução (parcial). Síndico está prestes a validar. Antes da validação, harassment confirmado → `company.suspended_until = NOW() + 72h`. Deal ainda é válido (não anulado); execução continua registrada (COM-020). Síndico tenta validar execução (COM-021) — ABAC filter: empresa suspensa, pode o síndico concluir validação? Se sim, reputation recompute adiciona ao contrato positivo enquanto empresa está em "suspensa". Se não, deal trava — mas empresa já cobrou, síndico pagou, serviço feito. Spec diz "fluxo comercial bloqueado durante 72h" — mas validação é institutional (não commercial direto).
- **Impacto**: confusão fluxo; síndico nem cancela nem valida → lock dos dois lados; empresa retorna 72h depois, valida, mas reputation score contém período estranho onde "suspended_until != NULL" e ainda recebeu positivo contrato.
- **Gap atual**: INV-055 é "flag paralela ao tier", mas política sobre continuação de deals in-flight during suspension é implícita; sub-produtos/03-reputacao.md §9 só menciona "acesso a fluxos comerciais bloqueado" (não detalhado).
- **Correção nominal**: em `04-requirements/functional/commercial.md` COM-021 adicionar explícito: "se `company.suspended_until > NOW()`, validação de execução e pagamento ficam em queue `awaiting_suspension_lift` — retomam automaticamente pós-72h"; em `invariants.md` INV-121: "`execution_record.status=validated` durante `company.suspended_until > NOW()` → 409 `COMPANY_SUSPENDED`".

---

#### Q-018 — Síndico 15 condomínios, destituído em 1 → ainda tem 15 slot? limit é condomínio ativo ou histórico?

- **Severidade**: 🟡
- **Categoria**: edge-case
- **Onde**: `01-domain/invariants.md` INV-027 + `04-requirements/functional/billing.md` BIL-032 + `01-domain/business-rules.md` BR-007
- **Como explorar**: síndico tem 15 mandates ativos. É destituído de 1 → 14 ativos, 1 encerrado. Spec diz "15 condomínios por síndico hard limit". Ele pode criar o 16º? Ou ele está "queimou uma vaga" permanentemente? Se for histórico, uma rotação normal (sair voluntariamente de 1 para entrar em outro) geraria queda progressiva. Se for ativo, fraudulento pode destituir-se e reassinar a cada 12 meses escalando.
- **Impacto**: política ambígua; dev implementa interpretação A, auditor reclama interpretação B; moradores veem síndico em 16 condomínios "não é possível".
- **Gap atual**: INV-027 diz "≤ 15 condomínios por síndico" sem qualificar ativo vs histórico; BR-007 repete sem clareza.
- **Correção nominal**: em `invariants.md` INV-027 precisar: "`COUNT(*) FROM mandates WHERE user_id=$1 AND ended_at IS NULL` ≤ 15 (ativos)"; P-INV-003 já listado como pendente em invariants.md — **fechar**: "hard limit por ativos; histórico ilimitado; sem cooldown na destituição".

---

#### Q-019 — Empresa downgrade Plus → Base mid-grant delegação agência: agência perde acesso imediato? grant órfão?

- **Severidade**: 🟠
- **Categoria**: edge-case
- **Onde**: `01-domain/invariants.md` INV-019 (soft-block sem grace), INV-049 (delegação marketing) + `04-requirements/functional/commercial.md` COM-040 + `01-domain/business-rules.md` BR-017 (upgrade/downgrade)
- **Como explorar**: empresa Plus com delegação ativa p/ agência marketing X (token scope `videos:*, analytics:read`). Via Stripe Customer Portal, downgrade Plus → Base (ou cancela). Webhook dispara `subscription.updated` → plan_tier=base. ABAC para empresa enforça perda de Connect Me Plus, Banco de Talentos etc. Mas delegação é **aggregate separado** (`marketing_delegations`) e `INV-049` não tem regra dinâmica "se tenant_plan baixa tier, delegações existentes ficam…" — grant fica órfão. Agência continua acessando analytics e videos da empresa por horas/dias até detectar.
- **Impacto**: vazamento de analytics enterprise (e mesmo vídeos recentes) a agência que empresa deixou de pagar para manter; confusão do modelo "subscription vs acesso delegado".
- **Gap atual**: INV-049 é "ABAC policy" mas não gatilha `DowngradeHandler` → `RevokeMarketingDelegations`; BR-017 trata upgrade/downgrade mas não delegações.
- **Correção nominal**: em `invariants.md` INV-122: "subscription downgrade que reduz tier → dispara evento `TenantPlanDowngraded` → handlers cancelam delegações ativas se novo tier não cobre escopo"; em `04-requirements/functional/commercial.md` COM-040 + `billing.md` BIL-026 documentar hook.

---

### Categoria 4 — Perda de dados / inconsistências (6 findings)

---

#### Q-020 — Ata imutável INV-072 + Timeline imutável INV-024: garantia é **app-level** — quem impede UPDATE/DELETE direto no DB?

- **Severidade**: 🔴
- **Categoria**: data-loss
- **Onde**: `01-domain/invariants.md` INV-024, INV-072, INV-090 + `08-security/pentest-checklist.md` §10.3
- **Como explorar**: DBA interno malicioso (ou RCE de webhook vulnerável → DB access) executa `UPDATE assembly_minutes SET text='...' WHERE id=...` ou `DELETE FROM timeline_entries WHERE id=...`. INV-024/INV-072 dizem "sem UPDATE/DELETE em timeline_entries" / "imutável após publish" mas enforcement é **app-level + CI grep + trigger opcional**. Sem DB-level grant revocation, o DB não impede. Pior: `REVOKE UPDATE, DELETE ON timeline_entries FROM app_role` está **proposto** em pentest-checklist §10.3 mas não confirmado nos schemas. Ata é peça jurídica — uma mudança invisível = falsificação documental.
- **Impacto**: forjar atas, apagar Timeline, apagar audit_log — cobertura de crimes; perda total de valor jurídico da plataforma.
- **Gap atual**: INV-024 enforcement declara "Schema sem updated_at/deleted_at; CI grep bloqueia UPDATE timeline_entries/DELETE FROM timeline_entries; trigger Postgres opcional". Opcional não é suficiente.
- **Correção nominal**: em `invariants.md` INV-024, INV-072, INV-090 mudar enforcement de "opcional" para "obrigatório via `REVOKE UPDATE, DELETE ON {tables} FROM app_role` em migração + trigger `BEFORE UPDATE OR DELETE RAISE EXCEPTION`"; em `08-security/pentest-checklist.md` §10.3 marcar como blocker M1; em `AUDIT.md` elevar de "opcional" para critical.

---

#### Q-021 — Stripe webhook down 24h: subscriptions desalinham; job reconciliação noturna existe?

- **Severidade**: 🔴
- **Categoria**: data-loss
- **Onde**: `04-requirements/functional/billing.md` BIL-020 + `01-domain/invariants.md` INV-015, INV-105 + `01-domain/aggregates/Subscription.md`
- **Como explorar**: Railway tem outage 30min (improvável) ou bug de deploy nosso faz 5xx em `/webhooks/stripe` por 24h. Stripe retry automático tenta até 72h (BIL-020) e segura eventos. **Mas**: se backend fica 5xx > 3 dias, Stripe desiste dos eventos perdidos (política Stripe). Nesse caso, eventos críticos (`invoice.payment_succeeded`, `customer.subscription.deleted`) são perdidos para sempre; subscriptions BR-013 fica desalinhada (status local `active` mas Stripe diz `canceled`). ABAC continua permitindo acesso até admin perceber manualmente.
- **Impacto**: usuários com plano `canceled` no Stripe acessam premium local por dias/semanas; perda de revenue; LGPD-risco "dados processados sem base legal" (contrato expirou).
- **Gap atual**: BIL-020 diz "backend retorna ≠ 2xx → confia retry Stripe" + "Sentry alert `webhook:stripe:failed` > 24h" + "admin inspect manual". Nenhum job noturno de **reconciliação proativa** (`GET /subscriptions Stripe; compare com local; dishma diffs`).
- **Correção nominal**: em `01-domain/aggregates/Subscription.md` §Repository adicionar `FindMisalignedWithStripe` + job `ReconcileStripeJob` diário; em `04-requirements/functional/billing.md` BIL-020 elevar prioridade para M + documentar "cron diário `0 4 * * *` compara `stripe.subscription.list` com tabela local"; em `invariants.md` INV-123: "Subscription status reconciliado com Stripe pelo menos 1x/24h".

---

#### Q-022 — NATS JetStream perda de mensagens: quais fluxos críticos não podem perder evento?

- **Severidade**: 🟠
- **Categoria**: data-loss
- **Onde**: `01-domain/invariants.md` INV-095 (outbox pattern) + `02-architecture/` (não lido)
- **Como explorar**: NATS JetStream crash (raro mas possível) ou broker misconfigured (retention curta). Eventos `ReputationTierChanged`, `ClosedDealSigned→TimelineEntry`, `VotingClosed→VideoVisibilityRevoked` são perdidos. Reputation não atualiza; Timeline falta entry de ClosedDeal (BR-025 quebra); moradores continuam vendo vídeos empresa pós-votação fechada (INV-063 quebra).
- **Impacto**: visão inconsistente (síndico vê contrato fechado mas Timeline não tem); vazamento prolongado (vídeo continua acessível); reputation freeze.
- **Gap atual**: INV-095 exige outbox pattern ("events publicados dentro da tx via tabela domain_events") — **bom**, mas não especifica retry+DLQ nem reconciliação offline se subscriber não consumir; threat-model §3 cita JetStream implicitamente mas sem bancos de resiliência.
- **Correção nominal**: em `invariants.md` INV-124: "Outbox publisher tem retry exponencial + DLQ PostgreSQL `failed_domain_events`; reconciliação cron 5min re-publica do DLQ"; listar fluxos críticos em `02-architecture/event-flow.md` (a criar): `ReputationTierChanged`, `ClosedDealSigned`, `VotingClosed`, `SubscriptionCanceled`, `HarassmentReportConfirmed`.

---

#### Q-023 — Mux upload incompleto (rede morre): file órfão em S3/Mux? quota consumida antes do complete?

- **Severidade**: 🟠
- **Categoria**: data-loss
- **Onde**: `01-domain/invariants.md` INV-065 (Saga UploadVideo) + `04-requirements/functional/billing.md` BIL-031 + `08-security/threat-model.md` §4.5
- **Como explorar**: user inicia upload 1GB via presigned Mux URL. Rede cai a 50%. Presigned URL tem TTL 1h (INV-070 é para playback, não upload — mas vale o princípio). Mux tem upload parcial armazenado temporário. User retry de zero → 2 assets iniciados; um órfão. Saga UploadVideo (INV-065) compensa Mux creates sem local row — mas se user tentar retry via POST `/videos` com `replaces=last_upload_id`, ambos criam, quota `video_upload_institutional` (BIL-031) decrementa 2x se decrementa ao publicar.
- **Impacto**: custo storage Mux órfão (paga Master Síndico); quota consumida injustamente; experiência ruim.
- **Gap atual**: INV-065 cobre "Mux sucesso + local falha → CancelMuxUpload"; não cobre "Mux half-upload + user retry → 2 asset IDs órfãos".
- **Correção nominal**: em `invariants.md` INV-065 precisar: "Saga UploadVideo tem compensator para asset criado mas sem `asset.ready` webhook em 24h (job `cleanup_dangling_uploads_job`)"; em `04-requirements/functional/billing.md` BIL-031 documentar "quota decrementa no `video.published`, não no `upload_initiated`"; em invariants INV-045 estender para vídeo: "quota consome no published".

---

#### Q-024 — Cross-tenant data leak via ABAC bypass: alguma rota passa `tenant_id` do client sem validar?

- **Severidade**: 🔴
- **Categoria**: data-loss
- **Onde**: `01-domain/invariants.md` INV-089, INV-108 + `08-security/threat-model.md` §5.1 + `08-security/pentest-checklist.md` §2.5
- **Como explorar**: algumas rotas aceitam `?tenant_id=X` em query string (ex: `/api/v1/timeline?condominium_id=X`). Se backend não valida `ctx.user_id ∈ members(X)`, vaza. threat-model §5.1 lista como primary attack vector. INV-089 é "toda query escopada por condominium_id/company_id" (convenção sqlc), **mas** não enforça o mapping request→ctx.tenant_id vem do token, não do URL. Se route parser do Go extrai `tenant_id` da URL sem override server-side, IDOR cross-tenant é trivial.
- **Impacto**: vazamento massivo — síndico A lê Timeline do condomínio B; acesso Livekit a assembleia de outro condo; etc.
- **Gap atual**: INV-089 é "convenção sqlc + 100+ testes cross-tenant"; INV-108 (tenant-id-typed — A-DC-SEC-005) é proposto. Nenhum dos dois enforça MIDDLEWARE obrigatório "tenant_id do path/query === tenant_id do ctx".
- **Correção nominal**: em `invariants.md` INV-125: "Middleware `TenantBoundaryGuard` ANTES de handler: se request tem `{condominium_id, company_id, partner_id}` em path/query, valida que matches `ctx.user.active_tenants`; 404 (não 403) em mismatch"; em `08-security/pentest-checklist.md` §2.5 marcar item como M1 blocker; adicionar A-### em AUDIT para elevar prioridade.

---

#### Q-025 — Backup + PITR: master-sindico-v2 especifica estratégia? último backup testado quando?

- **Severidade**: 🟡
- **Categoria**: data-loss
- **Onde**: `08-security/pentest-checklist.md` §12.3 + `12-docs/runbooks/dr.md` (referenciado em AUDIT)
- **Como explorar**: cenário perda total DB (hardware, rm -rf, ransomware). Checklist §12.3 lista "DB backup diário + retention" + "Backup restore tested: trimestral" como M2. Em M1, se Railway tem bug, dados se perdem inteiros. Ata imutável vai embora; LGPD Art. 48 (segurança dos dados) violado.
- **Impacto**: perda absoluta ou parcial de dados de produção; empresa fecha (criticidade jurídica das atas); LGPD fines.
- **Gap atual**: pentest-checklist §12.3 é "M2" — ok, mas runbooks/dr existe no AUDIT "resolvido". Verificar se contém procedimento de **teste restore regularly** — se é só "configurar cron" sem validação periódica, falha silenciosa.
- **Correção nominal**: em `12-docs/runbooks/dr.md` (já criado) adicionar seção "Trimestral drill: destruir DB replica + restaurar do backup mais recente + validar integridade (COUNT(*) de tabelas críticas = baseline)"; em `invariants.md` INV-126: "Backup restore drill executado ≥ 1x/quarter; falha de drill = incidente P1"; em `AUDIT.md` elevar M2 → M1 blocker para LGPD.

---

### Categoria 5 — Compliance / regulatório (6 findings)

---

#### Q-026 — LGPD right-to-erasure (Art. 18 VI) vs retenção contratual/audit 5 anos: conflito não resolvido

- **Severidade**: 🔴
- **Categoria**: compliance
- **Onde**: `01-domain/invariants.md` INV-091, INV-033, INV-090 + `04-requirements/functional/billing.md` BIL-051 + `01-domain/business-rules.md` BR-053, BR-054
- **Como explorar**: user solicita exclusão LGPD Art. 18 VI. BR-053 especifica "hard delete: User, Sessions, DeviceFingerprints, dados pessoais em Company" + "soft archive + pseudonimização: votes, avaliações, minutes". MAS INV-033 (audit_trail append-only) + INV-090 (retenção 5y) + BR-057 (audit 5y) diz NÃO apagar. BR-054 diz "LGPD log 5y; direito ao esquecimento após vencimento → hard delete" — **ou seja, só após 5y**. **Conflito**: user exige erasure hoje, audit precisa reter 5y. P-INV-005 está listado em `invariants.md` como pendência — aberto. SLA 15d (BR-053) vs 5y retention: como reconciliar?
- **Impacto**: ANPD multa se demorar mais de 15d; auditor reclama se apagar audit; litígio interno time legal x engenharia; decisão deve ser tomada antes M1.
- **Gap atual**: INV-091 diz "retenção 5y + esquecimento após vencimento"; BR-053 diz "SLA 15d"; contradição explícita.
- **Correção nominal**: em `invariants.md` fechar P-INV-005 com decisão: "hard-delete apenas de `users` (PII) em SLA 15d + **pseudonimização imediata** em `audit_trail/votes/minutes` via HMAC keyed (A-DC-PEN-038 / ADR-0028) mantém retention 5y sem violar erasure" — registrar como INV-127 com fluxo: "LGPD erasure = hard-delete PII tables + pseudonymize references in append-only tables, mesmo UoW"; em `04-requirements/functional/billing.md` BIL-051 detalhar; fechar bloqueador LGPD-M1-007 em AUDIT.md apenas após este fluxo implementado.

---

#### Q-027 — Assembleia edital 8 dias antecedência (Lei 4.591/64 Art. 24): spec diz 5 dias (ASM-001) — compliance

- **Severidade**: 🔴
- **Categoria**: compliance
- **Onde**: `04-requirements/functional/assembly.md` ASM-001 + `01-domain/invariants.md` INV-074 + `01-domain/business-rules.md` BR-041 + Lei 4.591/64 Art. 24 §1
- **Como explorar**: atual spec ASM-001 valida "data futura, mín 5 dias de antecedência". Lei 4.591/64 Art. 24 §1 exige **8 dias** para 1ª convocação ordinária em edital. Síndico cria assembleia com 5d → passa validação Master Síndico → contestada em justiça por morador que recebeu notificação com menos de 8d → assembleia nula; votos anulados; ata perde validade. Condomínio processa Master Síndico por propaganda enganosa ("meio tecnológico de organização para assembleia Lei 4.591 válida").
- **Impacto**: risco legal direto; Master Síndico co-responsável por não enforçar compliance; condomínios cancelam assinatura em massa.
- **Gap atual**: ASM-001 "mín 5 dias" — contradiz Lei. INV-074 diz "não inicia sem edital publicado" mas não valida prazo de edital.
- **Correção nominal**: em `04-requirements/functional/assembly.md` ASM-001 mudar "mín 5 dias" para "mín 8 dias entre edital publicado e scheduled_date (Lei 4.591/64 Art. 24 §1); extraordinária segue convenção"; em `invariants.md` INV-128 novo: "Assembly `PublishEdital` valida `scheduled_date >= NOW() + 8d` (ordinária) || `convenção.edital_prazo`"; em `business-rules.md` BR-041 documentar.

---

#### Q-028 — Ata assinada digitalmente (ICP-Brasil): fluxo existe ou é assinatura visual?

- **Severidade**: 🔴
- **Categoria**: compliance
- **Onde**: `04-requirements/functional/assembly.md` ASM-024 + `01-domain/invariants.md` INV-072 + Lei 14.063/2020 (gov.br) + MP 2.200-2/2001 ICP-Brasil + `01-domain/business-rules.md` BR-049
- **Como explorar**: ASM-024 diz "assinatura digital síndico (M4+)". MVP M3 = sem assinatura digital. Ata gerada = PDF com "assinatura visual" (imagem). Apresentada em cartório ou justiça → rejeitada como prova judicial; valor jurídico comprometido. Master Síndico vendeu "ata imutável auditável" — proprietários cobram garantia.
- **Impacto**: toda a promessa de governança digital colapsa; condomínios migram para software concorrente com assinatura Lei 14.063.
- **Gap atual**: ASM-024 lista como M4+ (não M1). Produto vende "ata judicialmente válida" mas M1 sem assinatura. 
- **Correção nominal**: em `04-requirements/functional/assembly.md` ASM-024 elevar prioridade para M (M1) **ou** explicitar em `00-product/sub-produtos/05-assembleia-live.md` que "M1 suporta assembleia interna informal; valor jurídico pleno requer Sprint 3+ assinatura digital"; em `invariants.md` INV-129: "Minutes.status = 'homologated' + 'judicial_grade' apenas se `signatures IS NOT EMPTY AND signature_method = icp_brasil|gov_br`"; threat-model §9 e §14 documentar risco.

---

#### Q-029 — Voto secreto exigido em certos tipos assembleia (ex: destituição de síndico, eleição de síndico): sistema suporta?

- **Severidade**: 🟠
- **Categoria**: compliance
- **Onde**: `04-requirements/functional/assembly.md` ASM-014 + `01-domain/invariants.md` INV-071, INV-083 + Código Civil Art. 1.355
- **Como explorar**: Código Civil Art. 1.355 e regulamentos municipais podem exigir **voto secreto** em temas sensíveis (destituição de síndico, questões envolvendo família, matéria disciplinar). Atual sistema (ASM-014) tem 4 tipos (simples, fração ideal, fornecedor, eleição) — todos **rastreáveis** (vote.voter_id public). Nenhum suporta voto secreto (campo `voter_id` hash/anonimizado no resultado, só DBA vê pra recount). threat-model §8.1 cita "mitigação: voto secreto opcional (M3 — só grava resultado agregado)" — ou seja, reconhece mas não implementa.
- **Impacto**: assembleia que precisa voto secreto não pode rodar na plataforma; feature faltante em pauta específica; judicial anula quando descoberta.
- **Gap atual**: ASM-014 não tem subtipo "secreto"; INV-071 força UNIQUE(agenda_item, voter_id) — incompatível com anonymization.
- **Correção nominal**: em `04-requirements/functional/assembly.md` ASM-014 adicionar quinto tipo `votacao_secreta`: "voter_id é hashed + salt único por sessão; só aggregate results públicos"; em `invariants.md` INV-130: "votos secretos têm `voter_id_hash` em vez de `voter_id`; UNIQUE sobre hash; nenhum endpoint exposto retorna voter_id em clear"; priorizar M2 ou assumir como GAP documentado M1.

---

#### Q-030 — Síndico vota em contrato da **própria empresa** (síndico-empresa conflict of interest): bloqueado?

- **Severidade**: 🟠
- **Categoria**: compliance
- **Onde**: `01-domain/invariants.md` INV-035 (conflict_of_interest_declarations onboarding) + `04-requirements/functional/commercial.md` COM-044 (empresa-síndica M5+) + `01-domain/business-rules.md` BR-012 (conselho aprova contratos)
- **Como explorar**: síndico **é dono** de empresa Y (feature "empresa-síndica" M5+ — COM-044). Empresa Y propõe serviço para condomínio de síndico; SupplierVoteSession aberta; conselho aprova contratos > threshold (BR-012). Mas se o síndico for o **presidente da assembleia** (INV-081 não vota) ele passou. E se ele for conselheiro? BR-012 "conselho aprova contratos acima de threshold" — ele vota no próprio contrato. Código Civil Art. 1.348 §2 diz síndico não pode ser parte em contratos com o condomínio que administra (salvo autorização). 
- **Impacto**: contrato juridicamente nulo; litígio; reputation Master Síndico.
- **Gap atual**: INV-035 exige onboarding declaration — **estática**, uma vez. Dinâmica de "este síndico tem empresa-síndica e proposta vem da mesma" não está enforced em SupplierVoteCast ou Connect Me.
- **Correção nominal**: em `invariants.md` INV-115 (já sugerido em Q-008) expandir: "SupplierVoteCast + CompanyEvaluation + ConnectMeRequest rejeitam se `user.is_owner_of(company_id)` OR `conflict_of_interest_declaration WHERE user_id=X AND company_id=Y OR family_connection_cnpj`; hard-block com 403 `CONFLICT_OF_INTEREST`"; documentar em BR-012 + COM-044.

---

#### Q-031 — LGPD Art.18 V (portabilidade) + Art.18 VI (acesso): self-service `/me/export` existe — mas rate limit + formato?

- **Severidade**: 🟡
- **Categoria**: compliance
- **Onde**: `04-requirements/functional/billing.md` BIL-052 + `01-domain/invariants.md` INV-091 + LGPD Art. 18 V-VI
- **Como explorar**: BIL-052 documenta `GET /me/data-export` mas prioridade S (M2+). M1 não tem. User solicita → admin processa manualmente → pode demorar mais de 15d → ANPD risco. Também: quando exportar, o user recebe JSON estruturado — mas inclui **dados de outros users vinculados** (ex: avaliações que fez a empresas, que incluem `company_id`)? Se inclui, empresa pode solicitar portabilidade e receber dados de síndicos; fica difícil separar.
- **Impacto**: não-compliance LGPD Art. 18; ANPD fine.
- **Gap atual**: BIL-052 prioridade S (M2+); M1 sem self-service.
- **Correção nominal**: em `04-requirements/functional/billing.md` BIL-052 elevar para M (M1) ou documentar workaround admin manual com SLA ≤15d em runbook `12-docs/runbooks/lgpd-export.md` (a criar); em `invariants.md` INV-131: "LGPD data export gera JSON com apenas dados do requester; cruzamentos com outros tenants (deal, evaluation) incluem IDs mas não PII de outros".

---

## §3. Top 10 quebras críticas priorizadas M1

Em ordem decrescente de urgência pré-M1 (2026-05-08):

1. **Q-020** 🔴 — Ata/Timeline UPDATE direto no DB não bloqueado (REVOKE grants + trigger obrigatório). Data-loss + fraude documental; bloqueia toda promessa de governança.
2. **Q-027** 🔴 — Edital 5d (spec) vs 8d (Lei 4.591/64). Compliance direto; assembleias nulas.
3. **Q-005** 🔴 — Homologação race `closed → canceled` não proibido em state machine. Data-loss estrutural.
4. **Q-026** 🔴 — LGPD erasure 15d vs audit retention 5y (P-INV-005 aberto). Compliance blocker ANPD.
5. **Q-024** 🔴 — Cross-tenant leak via tenant_id em path/query sem guard. Data-loss massivo se ABAC engine miss.
6. **Q-002** 🔴 — Cupom lock 4h race (Redis TOCTOU). Fraude direta alto volume.
7. **Q-001** 🔴 — 1-device bypass em janela 50-200ms + voto fraudulento. Integridade assembleia (jurídica).
8. **Q-013** 🔴 — Síndico destituído mid-assembleia: votação prossegue? voto dele conta? State machine lacônica; litígio garantido se acontece em produção.
9. **Q-008** 🔴 — Empresa avalia a si mesma via morador laranja + síndico-empresa. Reputação envenenada → Connect Me ranking + trust destroyed.
10. **Q-011** 🔴 — Cupom ID_LOJA+SEQ+PALAVRA previsível via brute-force SEQ (1000 tentativas). Fraude escalável direto no sistema.

**Severidade 🟠 próximos da fila**: Q-009 (síndico banido não-transitivo), Q-018 (Stripe webhook down 24h sem reconcile), Q-028 (ata sem assinatura digital plena), Q-023 (Mux upload órfão), Q-017 (plano cancelado mid-upload), Q-016 (empresa pausada mid-execução), Q-007 (trial bypass serial), Q-012 (CNPJ forjado).

---

## §4. Metadados do audit

- **Método**: leitura integral dos 13 arquivos obrigatórios + `AUDIT.md` (Fase 5 consolidado).
- **Persona**: pentester + risk analyst + QA adversarial senior — foco EXCLUSIVO em quebras de negócio (sem UX, sem features, sem ADRs genéricos).
- **Escopo coberto**: 5 categorias; 31 findings novos; ≥5 findings/categoria.
- **Não cobertos (fora do escopo)**: vulnerabilidades técnicas de código Go (ex: SQLi — coberto em pentest-checklist); melhorias de UI; proposals de arquitetura nova.

## §5. Links

- [[../AUDIT]] (Fase 5 consolidado — este é um filho direto)
- [[../dual-check-log/2026-04-23-security-gaps]]
- [[./2026-04-23-pentest-specs]]
- [[./2026-04-23-qa-review]]
- [[../../01-domain/invariants]]
- [[../../01-domain/state-machines]]
- [[../../04-requirements/functional/billing]]
- [[../../04-requirements/functional/assembly]]
- [[../../04-requirements/functional/commercial]]
- [[../../08-security/threat-model]]
- [[../../08-security/pentest-checklist]]
- [[_moc]]
