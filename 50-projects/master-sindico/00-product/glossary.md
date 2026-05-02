---
title: Glossary — Ubiquitous Language Master Síndico
type: note
tags: [product, glossary, ubiquitous-language, master-sindico, v2]
source: vault/50-projects/master-sindico/01-domain/ubiquitous-language.md + client-vault/00 - Visão Geral + contextos/*
created: 2026-04-23
updated: 2026-04-23
---

# Glossary — Ubiquitous Language

Dicionário canônico em PT-BR. Termos jurídicos brasileiros preservados com precisão; não traduzir pra termo genérico/anglicismo quando o cliente já usa o termo BR.

**Regra**: código (Go identifiers, types, functions) em EN; **corpo de spec + UI + termos-domínio em PT-BR**. Frontmatter EN. Docstrings bilíngues só em API pública.

**Origem**: consolidação de `vault/50-projects/master-sindico/01-domain/ubiquitous-language.md` + `client-vault/02 - Domínios e Requisitos/` + `contextos/CONTEXT_BRAIN.md` + destilado Fase 1 relatório domínio.

---

## A

### Ata (de Assembleia)

Documento formal registrando o ocorrido numa assembleia. Contém: presença, proxies outorgados, pauta, discussões resumidas, resultado de votação por item, quórum atingido.

**Invariantes**:
- **Imutável pós-publish**. Correção vira nova entry na timeline institucional (nunca edita ata existente).
- Assinada digitalmente (MP 2.200-2 / Lei 14.063) pelo síndico e secretário da assembleia.

**Código (Go)**: `Minutes` aggregate em `assembly` BC.

### Assembleia

Reunião formal dos condôminos. Tipos:
- **Ordinária** — anual, prevista em convenção. Pauta inclui aprovação de contas, orçamento, prestação de contas.
- **Extraordinária** — convocada pontualmente para decisões específicas (obras, alteração de convenção, destituição de síndico).
- **Instalação** — primeira assembleia pós-habite-se ou pós-constituição do condomínio.

**Código**: `Assembly` aggregate; `AssemblyType` VO.

### Autenticação (Auth)

Processo de verificar identidade. Master Síndico usa **Zitadel OIDC Authorization Code + PKCE**. Cookie httpOnly em web; Bearer token em mobile. MFA obrigatório pra Admin (M2+).

### Avaliação (de Empresa)

Feedback estruturado pós-contrato emitido por síndico/morador sobre empresa que executou serviço. Inputs obrigatórios: nota 1-5, data, condomínio. Inputs opcionais: comentário texto, flags (pontualidade, preço-justo, comunicação).

**Invariante**: uma avaliação única por (company, evaluator, condominium) — UNIQUE DB. Alimenta fórmula de **Reputação**.

**Código**: `CompanyEvaluation` aggregate em `commercial`.

---

## B

### Banco de Talentos

Sub-produto: vídeo-currículos de moradores (90 segundos, via Mux, lock 90d pós-upload) + perfil com habilidades e disponibilidade. Empresas Plus/Pro buscam candidatos; convidam via Connect Me fluxo especial.

### BeyondCorp (adaptado)

Modelo de segurança zero-trust implementado no Master Síndico: ausência de "rede confiável"; toda request auth+autz; identidade + device fingerprint como perímetros. Pilares concretos em [[../08-security/BEYOND_CORP]] (a destilar).

### Bronze, Prata, Ouro, Platina, Diamante

Tiers de **Reputação** determinísticos. Ver [[portfolio-de-produtos#produto-3-reputação-bronze--diamante]].

### Bounded Context (BC)

Conceito DDD: fronteira de consistência semântica. Master Síndico tem 6 BCs nominais + cross-domain:
- `identity`, `billing`, `institutional`, `commercial`, `content`, `assembly` + `cross-domain`.
- Catálogo completo em [[../01-domain/bounded-contexts]] (a destilar).

---

## C

### Cadastro (vs Perfil)

Distinção crítica no produto:
- **Cadastro** — dados obrigatórios que **bloqueiam** ativação até preenchidos. Ex: CPF, data nascimento, telefone. Detalhes por persona em [[../04-requirements/registration-data]] (a destilar).
- **Perfil** — dados opcionais de customização. **Não bloqueiam** ativação. Ex: foto, mini bio, vídeo institucional.

Campo de controle: `visibilityReady` (bool) = true quando cadastro essencial completo (aparece em buscas apenas após).

### Carimbo do Tempo

Atestado de data/hora via autoridade certificadora. Usado em edital + ata assinados digitalmente (Lei 14.063).

### Condomínio

**Tenant primário** do sistema. Conjunto de unidades habitacionais/comerciais com fração ideal distribuída. Conceitualmente = edificação(s) + pessoas (moradores, síndico, subsíndico, conselho) + timeline imutável + plano diretor.

**Invariante**: Σ fração ideal de todas unidades ≤ 100% (DB CHECK constraint + PBT).

**Código**: `Condominium` aggregate raiz em `institutional`.

### Conselho (Fiscal / Consultivo)

Colegiado que aprova contratos acima de threshold configurável; participa de `SupplierVoteSession` (votação colegiada para contratos). Sub-role operacional sob ABAC; não é persona própria.

**Código**: `Council` VO ou flag em `Membership` (decisão pendente D-??? na Fase 3).

### Connect Me

**Marketplace de contratação unidirecional**. 4 fluxos. **Não é chat**. Ver [[portfolio-de-produtos#produto-2-connect-me-marketplace-de-contratação-formal]].

**Código**: `ConnectMeRequest` + `Proposal` + `ClosedDeal` aggregates em `commercial`.

### CPF / CNPJ

Identificadores únicos de pessoa física / jurídica no Brasil. Master Síndico usa como chave natural: **1 CPF = 1 User** (PF); **1 CNPJ = 1 User** (PJ).

**Validação**:
- CPF: algoritmo de dígito verificador (Luhn-like BR) + unicidade no sistema.
- CNPJ: algoritmo de dígito verificador + unicidade + (M2+) validação ativa Receita Federal.

**Value Object**: `CPFVO`, `CNPJVO` (imutáveis, auto-validados no construtor `New`).

### Cota (Quota)

Limite numérico por plano aplicado a features: vídeos/mês, Connect Me/ano, lives/mês, vagas de conselho, etc. Resetadas em períodos definidos (mensal/anual/sem reset).

**Código**: `FeatureUsage` aggregate em `billing` (contador + limite + período).

---

## D

### Device Fingerprint

Identificador único de dispositivo que acessa a plataforma. Composto de: IP + User-Agent + canvas hash (browser) ou device ID (mobile). Usado para **1-device policy** — login novo invalida sessão anterior do mesmo User em device diferente.

**Código**: `DeviceFingerprint` VO em `identity`.

---

## E

### Edital (de Convocação)

Documento PDF publicado antes de assembleia. Contém: data, hora, local (ou link live), pauta detalhada, instruções de quórum, orientações sobre proxy.

**Invariante**: Assembly **não pode iniciar** sem edital publicado (A-013 do legado).

**Código**: gerado via lib PDF + persistido com hash para imutabilidade.

### Empresa (do Mercado Condominial)

Persona: PJ com CNPJ ativo que presta serviço condominial (portaria, limpeza, manutenção, segurança, reforma, assessoria). Distinção do **Parceiro Vizinhança** (comércio local ao morador, não ao condomínio).

**Código**: `Company` aggregate em `commercial`.

---

## F

### Fração Ideal

Percentual atribuído a cada unidade condominial que define:
- Peso de voto em assembleia
- Rateio de despesas condominiais (cálculo externo; Master Síndico não faz ERP)
- Participação em quórum

**Precisão**: 4 casas decimais (permite condomínios com 400+ unidades: 1/400 = 0.25%; precisa diferenciar 0.2500% de 0.2501% em edge cases).

**Invariante**: Σ frações ≤ 100% (CHECK constraint + PBT).

**Código**: `IdealFraction` VO (wraps `decimal.Decimal` ou `int` basis-points * 10⁴ = milésimo-ponto-percentual).

---

## G

### Governança Aplicada

Posicionamento do produto: não é ERP condominial, não é app de comunicação. É **aplicação de critério, registro e reputação** pra governança transparente. Ver [[vision]].

---

## I

### Identidade (User)

Registro único no sistema (CPF ou CNPJ). Pode operar como múltiplas **Personas** via **Memberships** em tenants distintos.

**Código**: `User` aggregate em `identity`.

### Imutável (Invariante)

Propriedade de entidades/eventos que **nunca** são atualizados/apagados. Master Síndico tem múltiplos:
- `TimelineEntry` (nunca UPDATE/DELETE)
- `Minutes` (ata, pós-publish)
- `Announcement` (comunicado, pós-publish)
- `Vote` (voto registrado é final)
- `Proposal` (proposta Connect Me enviada é final)

### Invariante (de Domínio)

Regra inviolável do domínio. Deve ser enforçada no código (não só documentada) via CHECK constraint + validação de construtor/factory + teste PBT. Catálogo em [[../01-domain/invariants]] (a destilar).

---

## J

### Jornada do Usuário

Sequência de ações que uma persona executa para atingir um objetivo. Master Síndico tem jornadas canônicas documentadas no legado `_archive/content-discovery-2026-03/`:
- **Morador**: M1–M13
- **Síndico**: S1–S60 (+ Compliance C1–C11)
- **Empresa**: E1–E16

---

## L

### LGPD

Lei Geral de Proteção de Dados (Lei 13.709/2018). Master Síndico implementa:
- Consentimento versionado (`terms_version`, `lgpd_consent_version`)
- Direito de exclusão com SLA 15 dias
- Portabilidade (`GET /me/export`)
- Audit trail `lgpd_logs` append-only 5 anos
- Breach notification 72h à ANPD
- DPO registrado

### Livekit

Provider de WebRTC SFU usado para sessões live de assembleia. Cross-references: [[../13-research/google-meet/_destilado]] (em processamento).

### LMS (Academia Master Síndico)

Sub-produto: trilhas de capacitação para síndicos. 5 áreas, 3 trilhas piloto M3. Ver [[portfolio-de-produtos#produto-6-lms-academia--banco-de-talentos]].

### Lock 90d

Regra: vídeo institucional (empresa/síndico N2+) ou vídeo-currículo (morador) **não pode ser substituído/apagado** nos 90 dias seguintes à publicação/upload. Motivo: churn prevention + compromisso editorial.

---

## M

### Master Síndico

O próprio produto. Também: **Admin Master Síndico** refere-se ao time interno operando a plataforma.

### Matriz Funcional

Tabela feature × persona × plano × quota (80+ features). Destilado em [[../04-requirements/matrix-functional]] (a preencher). Fonte legado: `contextos/Matriz Funcional.md`.

### Membership

Relação `User ↔ Condomínio ↔ Unit (se aplicável) ↔ Role (síndico/morador/subsíndico/conselho/dependente)`. Chave primária de autorização ABAC em `institutional`.

**Invariante**: membership ativa única por (user, condominium) em dado momento.

**Código**: `Membership` aggregate em `institutional`.

### Mux

Provider de vídeo (upload presigned, transcoding HLS, signed playback, webhooks). Core do sub-produto Conteúdo/Vídeos.

---

## N

### NR-1 (Norma Regulamentadora 1)

Normativa brasileira sobre gerenciamento de riscos ocupacionais (PGR — Programa de Gerenciamento de Riscos). Aplicável a empresas prestadoras de serviço em condomínio (portaria, limpeza). Master Síndico pode oferecer "selo NR-1 verificada" (M4+, em conjunto com compliance ABRACS).

---

## O

### OIDC (OpenID Connect)

Protocolo de autenticação sobre OAuth 2.0. Master Síndico usa OIDC Authorization Code + **PKCE** via Zitadel.

### Onboarding

Processo que leva usuário recém-cadastrado até `visibilityReady = true` (aparece em buscas, pode exercer features principais). Detalhes em [[../04-requirements/registration-data]].

---

## P

### Parceiro da Vizinhança

Persona: comércio local (não-condominial). Ver [[personas#5-parceiro-da-vizinhança-comércio-local]].

### PKCE (Proof Key for Code Exchange)

Extensão OAuth 2.0 obrigatória no Master Síndico. `code_verifier` armazenado em cookie encrypted `gorilla/securecookie`.

### PII (Personally Identifiable Information)

Dados pessoais (CPF/CNPJ/email/telefone/endereço/token). **Nunca** em logs. Usar `Masked()` helper. CI bloqueia via grep regex.

### Plano

Assinatura de um User em um tenant específico. Determina quotas + features. Vinculado a Stripe Subscription.
- Síndico: N1, N2, N3
- Morador: Base (gratuito), Pagante
- Empresa: Plus, Pro
- Parceiro: único tier (M1-M3)
- Agência: compartilha plano Empresa cliente (M1-M3)

### Plano Diretor (do Condomínio)

Documento plurianual (3-5 anos) que define atividades, marcos, orçamento previsto para gestão. Template + acompanhamento de execução (%). Diferencial do Master Síndico.

**Código**: `MasterPlan` aggregate em `institutional`.

### Procuração (Proxy)

Outorga de voto de um morador para outro em assembleia. Pode ser: específica (para um item da pauta) ou geral (toda a assembleia). Registrada antes da sessão; rastreável.

**Código**: `Proxy` aggregate em `assembly`.

---

## Q

### Quórum

Percentual mínimo de frações ideais presentes (ou representadas via proxy) para assembleia deliberar. Varia por tipo de decisão:
- Ordinária aprovação de contas: maioria simples (50% + 1 fração ideal presente)
- Alteração de convenção: 2/3 das frações (conforme Lei do Condomínio BR)
- Destituição de síndico: 1/2 + 1 das frações

**Fracional**: calculado em decimais (0.50, 0.6667), não em cabeça de pessoa.

---

## R

### RFP (Request for Proposal)

Formulário estruturado aberto no Connect Me fluxo Síndico→Empresa. Contém: categoria, descrição, orçamento teto, prazo, entregáveis, anexos. Empresas respondem com proposta formal.

**Código**: `ConnectMeRequest` aggregate.

### Reputação

Tier automático e determinístico de empresa (Bronze→Diamante). Ver [[portfolio-de-produtos#produto-3-reputação-bronze--diamante]].

---

## S

### Saga (Pattern)

Padrão distribuído para transações que cruzam provider externo. Implementado com compensação explícita (`UploadVideo` → falha → `CancelUpload` em Mux). Usado quando UnitOfWork não pode abranger (provider externo = fora do DB).

**Aplicações**: Mux upload (A-027 legado), Livekit sessão (A-028), Stripe charge+ativar-feature.

### Síndico

Persona principal: gestor do condomínio. Tipos: morador eleito, profissional externo. Níveis: N1, N2, N3.

### Subsíndico

Delegado do síndico titular. Sub-role operacional com maioria das permissões de síndico, exceto encerrar mandato próprio e designar novo síndico (T8 do legado).

### Suplemento de Subscrição

Documento de atualização de cadastro do condomínio (novos moradores, unidades, alteração de fração). Registra na timeline como auditoria; não é contrato (termo preservado do cliente).

---

## T

### TAO (Tenant-Aware Object)

Conceito: toda entidade sensível a tenant (condomínio) deve ter `condominium_id` no schema + queries escopadas por `WHERE condominium_id = $tenant_id`. Enforçado sistematicamente (100+ testes cross-tenant no legado).

**NÃO confundir** com TAO do Meta (datastore objetos+associações). Aqui é só convenção nossa.

### Timeline

Sequência append-only imutável de eventos/decisões do condomínio. **Memória institucional auditável**. Entries: membership mudada, comunicado publicado, assembleia realizada, ata publicada, contrato Connect Me fechado, avaliação de empresa, harassment report, moderação de vídeo.

**Código**: `TimelineEntry` aggregate em `institutional` (INSERT-only via UoW).

### Trial

Período gratuito de avaliação por persona:
- Síndico: 15 dias
- Empresa: 7 dias
- Parceiro: 30 dias
- Morador: — (não se aplica)

### Turma (cohort — LMS)

Grupo de alunos sob uma trilha LMS específica. M3+ (thin); M4+ pleno.

---

## U

### Ubiquitous Language

Conjunto de termos do domínio compartilhados entre time técnico e cliente. Este glossário é a expressão canônica.

### Unidade

Espaço individual (apartamento, casa, sala comercial) dentro de um Condomínio. Tem fração ideal, titular (Morador), dependentes.

**Código**: `Unit` aggregate em `institutional`.

### UoW (UnitOfWork)

Padrão Unit of Work: transação encapsulada pra atomicidade entre múltiplos repositories no **mesmo BC**. Se cruza provider externo: Saga em vez de UoW. Uso: `uow.Run(ctx, fn)`.

---

## V

### Value Object (VO)

Entidade imutável identificada por valor (não ID). Exemplos Master Síndico: `EmailVO`, `CPFVO`, `CNPJVO`, `MoneyVO` (int64 centavos BRL), `PhoneVO`, `IdealFractionVO`, `DeviceFingerprintVO`. Auto-validados no `New(...)` factory.

### Vídeo Institucional

Vídeo publicado por empresa ou síndico N2+ via Mux. Lock 90d pós-publish. Alimenta reputação (Platina+ requer ≥ 1 vídeo aprovado).

### Vídeo-Currículo

Vídeo 90s do morador no Banco de Talentos. Lock 90d pós-upload. Destaque em busca de empresas.

### Visibilidade

Flag `visibilityReady` em Profile: true quando cadastro essencial completo. Controla se persona aparece em buscas (perfil search, Connect Me search, Vizinhança search).

### Vizinhança

Sub-produto: feature geográfica de comércio local. Ver [[portfolio-de-produtos#produto-7-vizinhança-comércio-local]].

### Votação / Voto

Ação em item de pauta da assembleia. Peso = fração ideal do votante. Único por (agenda_item, voter) — UNIQUE DB (A-025 legado).

**Código**: `Vote` aggregate em `assembly`.

---

## W

### Webhook

Endpoint recebedor de evento assíncrono de provider externo. Master Síndico recebe de Stripe (invoice.*, subscription.*, customer.*), Mux (asset.ready/errored), Livekit (room_started/ended), Twilio (SMS delivery status).

**Regra dura**: HMAC signature verification **antes** de parse. Rejeita body inválido com 400. Idempotência via `event.id` único em ledger.

---

## Z

### Zero Trust

Princípio de segurança: ausência de rede confiável. Toda request passa por auth + autz. Base do [[../08-security/BEYOND_CORP]] (a destilar com input do [[../13-research/beyond-corp/_destilado]]).

### Zitadel

Provider de identidade OIDC usado pra autenticação + gerenciamento de usuários. Substituiu JWT próprio do TS legacy. Service Accounts + introspection + MFA + roles/grants.

---

## Índice cruzado (termo → BC)

| Termo | BC principal |
|---|---|
| User, Session, DeviceFingerprint, Authentication | identity |
| Subscription, Plan, Trial, Quota, FeatureUsage | billing |
| Condominium, Unit, Membership, TimelineEntry, MasterPlan, Announcement, Event, Poll | institutional |
| Company, ConnectMeRequest, Proposal, ClosedDeal, CompanyEvaluation, Reputation, SupplierVoteSession, Partner (Vizinhança) | commercial |
| Video, VideoModeration, Course, Enrollment, Certificate | content |
| Assembly, AgendaItem, Vote, Minutes, LiveSession, Proxy, AttendanceRecord | assembly |
| DomainEvent, AbacDecision, AuditEntry (LGPD) | cross-domain |

---

## Referências

- Legado: `../vault/50-projects/master-sindico/01-domain/ubiquitous-language.md`
- Cliente: `../vault/50-projects/master-sindico/client-vault/00 - Visão Geral/00 - Índice Master Síndico.md`
- Links: [[vision]], [[personas]], [[portfolio-de-produtos]], [[business-model]], [[../01-domain/bounded-contexts]] (a destilar).
