---
title: Personas — Master Síndico (consolidado Fase 8)
type: note
tags: [product, personas, master-sindico, v2, fase-8, consolidado]
source: ADR-0032 + STATE.md D-056..D-069 Fase 8 + 00-product/sub-produtos/01..11 + backend/.kiro/specs/master-sindico/requirements/personas-and-plans.md + functional-matrix.md (traduzida)
created: 2026-04-23
updated: 2026-04-23
status: consolidado-fase-8
dual_check: pending
aliases: [personas-canonicas, personas-fase-8]
---

# Personas — Master Síndico

Catálogo canônico das **6 personas** operantes na plataforma. Este documento substitui integralmente a versão Fase 3 (que usava slugs compostos `syndic_n1/n2/n3`, `enterprise_plus/pro`, `resident_base/paid`, `local_company_standard`, `marketing_standard`). Todos esses slugs foram **abolidos** pela ADR-0032 (D-057 Fase 8 / D-066 Fase 7) e pelo D-067 Fase 7.

## Regras arquiteturais invioláveis (não repetir em cada persona)

- **Planos globais Stripe-style**: `plan_tier ∈ {trial, base, plus, pro}` universal para TODAS as personas. Nomes comerciais ("Plano Plus Síndico", "Plano Empresa Pro") são rótulos — o dado canônico é sempre o tier universal (ADR-0032).
- **Sem slugs compostos**: zero `syndic_n1`, `enterprise_plus`, `resident_base`, `marketing_standard`, `local_company_standard` no código, banco, UI, marketing ou docs.
- **N1/N2/N3 abolidos** inclusive na UI (D-067 Fase 7) — só aparecem em citações históricas de fontes legadas, sempre com tradução explícita para tier universal.
- **Admin = `apps/admin` dedicada** no monorepo web com Cloudflare Access SSO+MFA (D-134 Fase H 2026-04-25 revoga D-058/D-072 que diziam admin transversal embutido em CMS). Acesso via `admin.mastersindico.com.br`.
- **Agência de Marketing = sub-produto interno próprio** (D-059 Fase 7 / D-060 Fase 8) com login próprio via Zitadel (role `marketing`) + delegação ABAC explícita sobre empresas clientes. Não é impersonation.
- **Agnosticismo de provedor** (D-056): stack atual é Railway + Zitadel + Resend + Mux + Livekit + MinIO + Twilio + PostgreSQL 18 + Redis + NATS — mas tudo atrás de interfaces (`IEmailProvider`, `IVideoProvider`, `IPaymentGateway`, etc.).
- **Morador Base Connect Me = 2/ano** (D-094 Fase 11 2026-04-24 revoga D-058/D-079 que diziam 0/ano). Matriz canônica em `04-requirements/matrix-functional.md:148-152` atualizada.
- **Roles Zitadel canônicas** (6 valores — `backend/.kiro/specs/master-sindico/requirements/identity.md:52` + `personas-and-plans.md:130`):
  - `syndic | resident | enterprise | marketing | local_company | admin`
- **Role identidade ortogonal ao tier do plano**: o mesmo User (1 CPF ou 1 CNPJ) pode ter memberships com personas diferentes em tenants diferentes.
- **Sub-roles operacionais** via enum `user_role_type` (Zitadel não conhece — lógica interna, Decision T8):
  - `principal | auxiliary | assistant | titular | dependent | administrator | operator`
- **Admin bypass** curto-circuita ABAC (hierarquia 1 — ADR-0032 §Decisão).

---

## 1. Síndico (`role=syndic`)

### 1.1 Perfil

Gestor do condomínio. Decisor principal das contratações. Convoca e preside assembleia. Responsável pela governança documentada e memória institucional (Timeline imutável, Plano Diretor plurianual 3-5 anos, Compliance Cap. VII CC). Publica comunicados oficiais (imutáveis pós-publish). Registrado como persona de maior privilégio funcional no produto (depois de Admin MS).

Fontes canônicas: [[../00-product/sub-produtos/01-governanca-institucional]] §4, `institutional.md:220` R1 matriz, `personas-and-plans.md:18`.

### 1.2 Sub-tipos operacionais (`user_role_type`)

- **`principal`** — Síndico eleito/contratado ativo; 1 por condomínio em dado momento (invariante Postgres `uq_syndic_mandates_active_principal` — migration 00201:72-74).
- **`auxiliary`** — Subsíndico; maioria das ações exceto encerrar mandato e designar novo síndico.
- **`assistant`** — Assistente de síndico; acesso operacional sem voto ou decisão vinculante.

### 1.3 Permissões por tier universal

> Matriz compacta traduzida de `functional-matrix.md:55-78` (original usa N1/N2/N3 — banidos por D-067). Mapping histórico: N1≡`base`, N2≡`plus`, N3≡`pro`.

| Ação | trial | base | plus | pro |
|---|---|---|---|---|
| Criar condomínio | ⏳ (1 soft-block) | 1 | até 15 | até 15 |
| Número máximo condomínios ativos | 1 | 1 | 15 | 15 |
| Criar atividade Timeline | ⏳ | ❌ | ✅ | ✅ |
| Criar comunicado oficial | ⏳ | ❌ | ✅ | ✅ |
| Publicar vídeo institucional | ⏳ | 0 | 4/mês | 12/mês |
| Mini-bio + vídeo institucional no perfil público | ❌ | ❌ | ✅ | ✅ |
| Convocar assembleia | ⏳ | ✅ | ✅ | ✅ |
| Publicar pauta / painel presidente | ⏳ | ✅ | ✅ | ✅ |
| Connect Me Síndico → Empresa | 0 | 2/ano | 4/ano | ilimitado |
| Connect Me Síndico → Parceiro Vizinhança | 0 | ⚠️ | ✅ | ✅ |
| Validar execução de empresa | ✅ | ✅ | ✅ | ✅ |
| Validar propostas | ✅ | ✅ | ✅ | ✅ |
| Exportar relatório PDF/CSV da timeline | ⏳ | ⏳ | ✅ | ✅ |
| Editar governance/compliance markers | ✅ | ✅ | ✅ | ✅ |
| Publicar cupom Vizinhança | ❌ | ❌ | ❌ | ❌ |
| Conceder seal "Síndico-aprovado" (Parceiros) | ❌ | ❌ | ✅ | ✅ |
| Criar pauta supplier vote em assembleia | ⏳ | ✅ | ✅ | ✅ |

**Hard limit universal**: **15 condomínios ativos por síndico** — `create_condominium_use_case.go:23 MaxCondominiumsPerSyndic = 15`.

**Hard paywalls** (403 `PLAN_REQUIRED`): criar comunicado (mínimo `plus`), exportar relatório (mínimo `plus`), publicar vídeo institucional (mínimo `plus`).

### 1.4 Dados obrigatórios (cadastro — bloqueia acesso até completar)

Fonte: `Development/contextos/DADOS PARA CADASTRO.md` + `registration-data.md` §Síndico.

- Nome completo
- Email profissional (usado para login Zitadel)
- Senha (política Zitadel — 8+ chars, 1 maiúscula, 1 número)
- CPF (validação dígitos verificadores; 1 CPF = 1 User)
- Data de nascimento (≥18 anos)
- Endereço profissional com CEP (validação ViaCEP)
- Telefone (E.164)
- Sub-tipo (morador-síndico / síndico profissional — não altera role, só posicionamento)
- Cidade / estado de atuação (derivado do CEP; editável)
- Tempo de atuação (anos)
- Quantidade de condomínios que administra atualmente

### 1.5 Dados de perfil opcionais

- Foto (≤5 MB, 200×200 px recomendado)
- Mini-bio (≤500 chars)
- Vídeo institucional (trava 90d via Mux — GR-029; só tier `plus`+)
- Formação / certificações
- Administradoras vinculadas (M5+)
- 25 compliance/governance markers autodeclarados (ABRACS, Seguro RC, Assessorias, LGPD, NR-1, Selo Reclame Aqui, Premiações, etc.) — `institutional.md:55-90 Req 6.3`

### 1.6 Trial

**15 dias** (`personas-and-plans.md:67`). Durante trial: tudo visível em leitura; bloqueia criar atividade Timeline, criar comunicados, exportar relatórios, gerenciar múltiplos condomínios.

### 1.7 Device policy

**1 dispositivo ativo por conta** (ADR-0014 one-device-policy; `identity.md:25` Rule 8). Override apenas via suporte interno.

### 1.8 Jornada canônica

**S1–S60** (`_archive/content-discovery-2026-03/jornada-sindico-completa.md` + `Inventario-Telas.md` §Síndico): cadastro → onboarding plano → criar condomínio → vincular unidades → publicar plano diretor → convocar assembleia → registrar contratação via Connect Me → avaliar empresa → publicar timeline → encerrar mandato.

**Compliance C1-C11** (`Compliance-Detalhado.md:1-285`): Painel → Declaração Anual → Assinaturas → Conflito de Interesse → Auditoria → Imutabilidade/Adendos → Risco → Contratual → LGPD → Score Governança → Dossiê Exportável. Bloqueios: 4 gatilhos (encerrar_mandato, gerar_relatorio_final, marcar_negocio_fechado, exportar_dossie).

---

## 2. Morador (`role=resident`)

### 2.1 Perfil

Condômino titular ou dependente de uma unidade residencial. Vota em assembleia (peso = fração ideal da unidade). Consome conteúdo institucional (Timeline, comunicados, atas). Pode contratar serviços via Connect Me (Morador→Síndico, cota limitada). Pode publicar vídeo-currículo no Banco de Talentos (lock 90d).

Fontes canônicas: [[../00-product/sub-produtos/01-governanca-institucional]] §4, [[../00-product/sub-produtos/07-banco-de-talentos]], `personas-and-plans.md:19`.

### 2.2 Sub-tipos operacionais (`user_role_type`)

- **`titular`** — Titular da unidade; 1 ativo por (user, unit). Vota com fração ideal da unidade (Req 9, Req 57).
- **`dependent`** — Dependente do titular (filho menor, cônjuge, etc.); leitura apenas, SEM voto em assembleia (`institutional.md:200` Req 9). Revogação do titular cascateia bloqueio dos dependentes (Req 17 — motivos: `moved_out / sold_unit / contract_ended / removed_by_syndic`).

### 2.3 Permissões por tier universal

> Morador tem 2 níveis comerciais: `base` (gratuito, via vinculação pelo síndico) e tiers pagos (`plus`/`pro`). Historicamente chamados "Base / Pagante". O tier universal é o dado; "Pagante" é rótulo que mapeia para qualquer tier ≠ `base`.

| Ação | base (free) | plus / pro (pagante) |
|---|---|---|
| Ler Timeline institucional | ✅ | ✅ |
| Ler comunicados | ✅ | ✅ |
| Responder enquetes | ✅ | ✅ |
| Dar ciência de evento/assembleia | ✅ | ✅ |
| Ver pauta de assembleia | ✅ | ✅ |
| **Votar em assembleia** | ❌ (hard paywall) | ✅ |
| Procuração (conceder/receber) | ❌ | ✅ |
| Avaliar gestão do síndico (bimestral 1-10) | ✅ | ✅ |
| Vídeos síndico (ilimitado) | ✅ | ✅ |
| Vídeos empresa preview 25% | ✅ | ✅ |
| Vídeos empresa integral | ❌ | apenas em contexto de votação de fornecedor (Rule 4, Req 103) |
| Cursos/certificados Academia | ❌ | ✅ (certificado exige pagante) |
| Fórum LMS — ver | ❌ | ✅ |
| Fórum LMS — comentar | ❌ | ✅ |
| Fórum LMS — criar tópico | ❌ | ❌ (ninguém pode criar tópico exceto sub-roles específicos) |
| Connect Me Morador → Síndico | **0/ano** (D-058 Fase 8) | **4/ano** (`functional-matrix.md:43`) |
| Vídeo-currículo 90s (Banco de Talentos, lock 90d) | ❌ | ✅ |
| Ver parceiros Vizinhança (bairro) | ❌ | ✅ |
| Resgatar benefícios/cupons Vizinhança | ❌ | ✅ |

**Hard paywall crítico**: votar em assembleia exige tier ≠ `base` (`functional-matrix.md:190` — "Votar em assembleia | Morador Base"). Trial grátis vê pauta mas não muda resultado.

### 2.4 Dados obrigatórios (cadastro)

Fonte: `registration-data.md` §Morador + `DADOS PARA CADASTRO.md`.

- Nome completo
- Email
- Senha
- Telefone (E.164)
- CPF (dígitos verificadores; 1 CPF = 1 User)
- Data de nascimento (≥18 — dependentes menores são cadastrados pelo titular com vínculo)
- CEP (validação ViaCEP)
- Condomínio vinculado (ID ou nome — ou via invite token do síndico)

### 2.5 Dados de perfil opcionais

- Foto (≤5 MB, 200×200 px)
- Endereço completo (rua, número, complemento)
- Vídeo-currículo (MP4/WebM, ≤500 MB, ≤90s, via Mux com presigned URL — lock 90d pós-upload)
- Ficha Banco de Talentos: 18 áreas de atuação, CNH (`nao/A/B/AB`), modalidades aceitas (9 — clt/pj/estagio/temporario/folguista/diarista/freelancer/intermitente/outros), disponibilidade, NR certifications, **5 últimos vínculos profissionais** (D-099 Fase 11 — sobrescreve D-060/D-064; alterável via matriz funcional), pretensão salarial

### 2.6 Trial

**Não se aplica** (`personas-and-plans.md:70`). Morador Base é gratuito via vinculação pelo síndico; Morador Pagante paga imediatamente (sem trial) ou permanece Base.

### 2.7 Device policy

**1 dispositivo ativo por conta** (Rule 8).

### 2.8 Jornada canônica

**M1–M15** (`Inventario-Telas.md:88-107` + `_archive/content-discovery-2026-03/jornada-morador.md`): login → timeline → ata → responder enquete → ciência evento → votar assembleia → procuração → upload vídeo-currículo → Connect Me → Morador → Síndico → avaliar gestão → consumir Vizinhança.

---

## 3. Empresa Condominial (`role=enterprise`)

### 3.1 Perfil

Prestadora de serviço condominial (portaria, limpeza, manutenção, segurança, reformas, assessoria contábil/jurídica, paisagismo, desinsetização, etc.). **Seller do marketplace Connect Me**. Alvo primário: empresas verificáveis via CNPJ ativo Receita Federal. 31 categorias e ~180 subcategorias (`enterprise-catergories-subcategories-enum.schema.ts`).

Fontes canônicas: [[../00-product/sub-produtos/02-connect-me]] §4, [[../00-product/sub-produtos/03-reputacao]], [[../00-product/sub-produtos/04-conteudo-videos]], `commercial.md:16-39 Req 18`, `personas-and-plans.md:20`.

### 3.2 Sub-tipos operacionais (`user_role_type`)

- **`administrator`** — Admin da empresa na plataforma (representante legal, gestor de conta); gerencia plano, usuários, vídeos institucionais, deals.
- **`operator`** — Operador técnico (funcionário da empresa sem poder de gestão); executa ações operacionais (responder propostas, registrar execução) mas não altera plano nem adiciona/remove usuários.

### 3.3 Permissões por tier universal

> Empresa não tem tier `base`. Tiers operacionais: `trial` (7 dias) → `plus` → `pro`. Historicamente chamados "Plus / Pro" — os rótulos sobrevivem, mas o dado canônico é `plan_tier`.

| Ação | trial | plus | pro |
|---|---|---|---|
| Cadastrar empresa (CNPJ Receita Federal) | ✅ | ✅ | ✅ |
| Publicar vídeo institucional (12 tipos — enum `video_type`) | ⏳ | 8/mês | 12/mês |
| Lock 90d por vídeo publicado (GR-029) | — | ✅ | ✅ |
| Perfil público visível (destaques, casos) | ❌ (privado) | ✅ | ✅ |
| Responder Connect Me (Síndico→Empresa) | ✅ | ✅ | ✅ |
| Iniciar Connect Me Empresa→Empresa (E→E) | ❌ | ✅ | ✅ |
| Criar proposta em RFP | ✅ | ✅ | ✅ |
| Registrar execução pós-deal | ⏳ | ⏳ | ✅ |
| Banco de Talentos — buscar/ver currículos | ❌ | ✅ | ✅ |
| Gerenciar usuários da empresa (convidar operadores) | ⏳ | ⏳ | ✅ |
| Convidar Agência de Marketing (delegação) | ❌ | ✅ | ✅ |
| Criar curso na Academia (certificação) | ❌ | ❌ | ✅ |
| Receber avaliação pós-contrato (alimenta reputação) | ✅ | ✅ | ✅ |
| Editar compliance markers | ✅ | ✅ | ✅ |
| Publicar cupom Vizinhança | ❌ | ❌ | ❌ (exclusivo de `local_company`) |

**Hard paywalls**: validar execução (mínimo `pro`), criar curso Academia (mínimo `pro`).

### 3.4 Dados obrigatórios (cadastro)

Fonte: `commercial.md:16-39 Req 18` + `institutional.md:55-90 Req 6.3`.

- Razão social
- Nome fantasia (opcional no cadastro; preenche perfil)
- Email do representante legal (login)
- Senha
- CNPJ (14 dígitos; validação Receita Federal ativa — stub em M1, pleno M2+)
- Data de aniversário da empresa (constituição)
- Representante legal (nome completo, CPF, telefone)
- Email comercial
- Telefone comercial
- Endereço comercial com CEP (ViaCEP)
- Contato financeiro (nome/telefone/email)
- Categoria principal (1 de 31) + subcategorias de atuação (seleção múltipla da lista controlada `enterprise-catergories-subcategories-enum.schema.ts`)
- Estados / cidades de atuação

### 3.5 Dados de perfil opcionais

- Logo (≤5 MB, 300×300 px)
- Descrição (≤1000 chars)
- Certificações (ISO 9001, 14001, B Corp, ABRACS, NR-* etc.)
- Vídeos institucionais (12 tipos canônicos — ver [[../00-product/sub-produtos/04-conteudo-videos]] §4)
- Portfolio (fotos / case studies — até 10)
- 15 compliance markers + 25 governance markers autodeclarados

### 3.6 Trial

**7 dias** (`personas-and-plans.md:68`). Durante trial: tudo visível; bloqueia publicar vídeos, registrar execuções, gerenciar usuários adicionais.

### 3.7 Device policy

**1 dispositivo ativo por conta do representante legal**. Contas secundárias (`operator`) via feature flag — futuro M3+.

### 3.8 Verificação (reputação de entrada)

- Status: `pending_verification → active → suspended → blocked` (`CompanyStatus` enum `commercial/domain/enums/enums.go:7-12`).
- CNPJ ativo Receita Federal (M2+; stub em M1).
- Situação regular PGFN/CADIN (M4+).
- Selo "Verificada" visual quando todas as validações OK.
- Tier de Reputação inicial: **Bronze** após cadastro completo + CNPJ verificado ([[../00-product/sub-produtos/03-reputacao]] §4).

### 3.9 Jornada canônica

**E1–E16** (`Inventario-Telas.md:113-129`): hub operacional → oportunidades → propostas → deals → execução → avaliação recebida → inteligência/métricas → convidar agência de marketing (E14) → Marketing Link (E16).

---

## 4. Agência de Marketing (`role=marketing`)

### 4.1 Perfil

Agência terceirizada que **administra marketing digital institucional de múltiplas empresas condominiais clientes** dentro do ecossistema Master Síndico. **Sub-produto interno de primeira classe** (D-059 Fase 7 / D-060 Fase 8) — NÃO é produto externo, NÃO é impersonation/actor delegado.

Fontes canônicas: [[../00-product/sub-produtos/10-marketing-agencias]], `identity.md:52,91`, D-060 `STATE.md:363-373`.

> **Correção importante vs. legado**: versões antigas (`personas-and-plans.md:21`) descrevem agência como "Actor delegado (não é tenant), sem perfil próprio, vinculado a Empresa". Essa leitura foi **substituída por D-060**: agência **tem sim login próprio** via Zitadel (role `marketing`); "não é tenant" significa apenas que ela não opera tenant condominial próprio — ela administra no tenant da empresa cliente mediante **DelegationGrant ABAC explícito**.

### 4.2 Sub-tipos operacionais

Não possui `user_role_type` operacional diferenciado no MVP. Todos os usuários da agência operam com o mesmo escopo de grants.

### 4.3 Identidade + delegação (modelo canônico D-060)

- Login próprio via Zitadel (authorization code + PKCE, cookie httpOnly `.mastersindico.com.br`, fallback Bearer mobile).
- Cache Redis `ms:auth:{zitadelUserID}` TTL 5min.
- 1 dispositivo ativo por conta (Rule 8).
- **Não tem tenant próprio**: opera dentro do tenant da empresa cliente via aggregate `DelegationGrant` ([[../00-product/sub-produtos/10-marketing-agencias]] §7).
- Cada vínculo agência ↔ empresa é uma `DelegationGrant` com permissões granulares (`ManageVideos`, `ViewAnalytics`) — enum `MarketingAgencyLinkStatus ∈ {pending, active, revoked}` (código vivo `content/domain/entities/marketing_agency_link.go`).

### 4.4 Permissões por tier universal

> Agência **não consome plano próprio no MVP** — opera sob o tier da empresa cliente. Capacidade "Delegar Agência" é da empresa e exige `plus+` (`functional-matrix.md:87`).

| Ação | Permitido via grant ABAC? |
|---|---|
| Upload de vídeo em nome da empresa cliente | ✅ (se grant `ManageVideos=true`) |
| Editar metadados de vídeo | ✅ |
| Ver analytics de vídeos próprios da agência (cross-empresa consolidado) | ✅ (se grant `ViewAnalytics=true`) |
| Receber Marketing Link (formulário de interesse de síndico/empresa via vídeo) | ✅ |
| Visualizar deals da empresa | ❌ |
| Enviar/receber Connect Me | ❌ |
| Gerenciar usuários da empresa | ❌ |
| Acessar propostas/RFPs | ❌ |
| Acessar dados pessoais de síndicos/moradores | ❌ |
| Token de delegação pode ser revogado a qualquer momento pela empresa | ✅ |

### 4.5 Marketing Link (feature distinta de Connect Me)

Formulário de intenção que cola tags em vídeos institucionais da empresa; síndico/empresa interessados submetem formulário → armazenado em tabela `marketing_link_requests` (separada de `connect_me_requests`) → notificação para a agência. Sem cota, sem quebra de privacidade (não expõe contato do emissor até aceite explícito).

### 4.6 Dados obrigatórios (cadastro via convite da empresa)

- Razão social da agência
- CNPJ
- Representante legal
- Email comercial (login)
- Senha
- Endereço com CEP
- **Token de convite da empresa** (gerado em E14; agência não se auto-cadastra do zero comprando plano; entrada é sempre por convite)

### 4.7 Dados de perfil opcionais

- Logo
- Descrição
- Portfólio de cases audiovisuais
- Tipos de suporte oferecidos (enum `MarketingSupportType` — `Enums e Listas Mestre.md:84-89`)

### 4.8 Trial

**Não se aplica como entidade independente**. Agência herda status da empresa que a convidou. Se a empresa está em trial, a agência opera nos limites do trial da empresa.

### 4.9 Device policy

**1 dispositivo ativo por conta** (Rule 8).

### 4.10 Jornada canônica

**MK1–MK8** (`Inventario-Telas.md` §Marketing + `_archive/content-discovery-2026-03/jornada-empresa-marketing.md:28-271`): aceitar convite → dashboard cross-empresa → upload vídeo por empresa → editar metadados → analytics → Marketing Link pipeline → revogação.

---

## 5. Parceiro da Vizinhança / Comércio Local (`role=local_company`)

### 5.1 Perfil

Estabelecimento comercial hyperlocal (pizzaria, papelaria, farmácia, petshop, academia, salão, restaurante de bairro) buscando vínculo com moradores de condomínios próximos. **Não presta serviço ao condomínio** (isso é Empresa Condominial, sub-produto 2). Vende diretamente ao morador como consumidor final.

Fontes canônicas: [[../00-product/sub-produtos/08-vizinhanca]], `personas-and-plans.md:22,59`, `commercial.md Req 27-27.3`.

### 5.2 Sub-tipos operacionais

Não possui `user_role_type` diferenciado. Representante legal único (`administrator` implícito).

### 5.3 Permissões por tier universal

> Legado usa `local_company_standard` — slug **abolido** por ADR-0032. Parceiro opera com tiers universais; MVP pragmaticamente vive em um único tier funcional (`plus` equivalente) dentro do enum global.

| Ação | trial (30d) | plus (tier operacional MVP) | pro (gamificação plena) |
|---|---|---|---|
| Perfil público hyperlocal (raio default 2km) | ✅ | ✅ | ✅ |
| Criar promoção / cupom (lock 4h por estabelecimento) | ⏳ | ✅ | ✅ |
| Sistema de Cupons — formato `ID_LOJA(5)+SEQ(3)+PALAVRA(5)` (13 chars; D-059 Fase 8) | ⏳ | ✅ | ✅ |
| Palavra-chave do dia (gamificação Req 100) | ❌ | ⚠️ limitada | ✅ |
| Receber seal "Síndico-aprovado" (concedido por síndicos plus+) | ✅ | ✅ | ✅ |
| Métricas de engajamento próprias (views, ativações, resgates) | ⏳ | ✅ | ✅ |
| Criar campanha dedicada | ⏳ | ✅ | ✅ |
| Receber Connect Me do Síndico (Req 19.3 — `syndic_to_local_company`) | ✅ | ✅ | ✅ |
| Enviar Connect Me | ❌ | ❌ | ❌ (parceiro só RECEBE) |
| Acesso à Academia LMS | ❌ | ❌ | ❌ (Decision 7 — exclusão canônica) |
| Acesso ao Banco de Talentos | ❌ | ❌ | ❌ |
| Upload de vídeos institucionais | ❌ | ❌ | ❌ |
| Escopo de cupom: `aberta_bairro` ou `exclusiva_condominio` | ⏳ | ✅ | ✅ |

**Regra anti-spam**: parceiro **não envia Connect Me** — apenas recebe (reduz prospecção não solicitada a síndicos).

### 5.4 Dados obrigatórios (cadastro — self-service ou convite síndico)

Fonte: `registration-data.md` §Parceiro.

- Nome / razão social
- Email do representante
- Senha
- **CPF MEI OU CNPJ** (Decision 12 — CPF MEI aceito; CNPJ opcional para pequenos comércios)
- Data de aniversário (pessoa física ou empresa)
- Representante legal (nome, telefone)
- Financeiro (nome, telefone, email — pode ser o próprio representante)
- Endereço com CEP (ViaCEP — base do raio hyperlocal)
- Categoria de atuação (38 categorias hyperlocal — `Enums-Consolidados.md §12`)
- Raio de atendimento (default 2km; ajustável)

### 5.5 Dados de perfil opcionais

- Logo (≤5 MB, 300×300 px)
- Nome fantasia
- Email comercial
- Telefone comercial
- Até 3 fotos do estabelecimento
- Palavra-chave do dia configurada (se tier pro)

### 5.6 Trial

**30 dias** (`personas-and-plans.md:69`). Durante trial: tudo visível; bloqueia criar promoções, criar campanhas, ver métricas.

### 5.7 Device policy

**1 dispositivo ativo por conta** (Rule 8).

### 5.8 Jornada canônica

**VZ1–VZ6** (`Vizinhanca-4-jornadas.md` + `_archive/content-discovery-2026-03/jornada-parceiro-vizinhanca.md`): cadastro → definir raio → criar promoção → gerar cupom `ID_LOJA+SEQ+PALAVRA` → acompanhar métricas → ajustar campanha.

Outras 3 jornadas de Vizinhança ([[../00-product/sub-produtos/08-vizinhanca]]):
- **VS1-VS10** — Síndico indica parceiros, concede seal
- **VE1-VE6** — Empresa vê parceiros do bairro
- **VM1-VM7** — Morador consome benefícios

---

## 6. Admin Master Síndico (`role=admin`)

### 6.1 Perfil

Time interno da Master Síndico responsável por governança global, moderação, suporte operacional e financeiro. **Role privilegiada transversal** (D-058 Fase 7 / D-061 Fase 8) — NÃO é aplicação separada; telas admin vivem **dentro** das apps existentes (síndico-web, empresa-web, morador-app) gated por ABAC.

Fontes canônicas: [[../03-subprojects/admin-privileges]], `functional-matrix.md:129-142`, D-061 `STATE.md:375-385`, `identity.md:91` ("admin — provisionada manualmente no Zitadel, sem fluxo de auto-registro").

### 6.2 Sub-tipos operacionais (scoped permissions ABAC — Decision T8 / D-061)

Admin não tem `user_role_type` no enum institucional (`principal/titular/...`); seus sub-tipos são **scoped permissions ABAC** aplicadas sobre a role única `admin`:

- **`super_admin`** — Permissão total, admin_bypass hierarquia 1. ADR-0032 §Engine: `if user.Role == RoleAdmin { return nil }`.
- **`moderator`** — Aprova/rejeita vídeos, resolve harassment reports (enum `HarassmentAdminAction ∈ {warning, suspension, ban}`).
- **`support`** — Vê dados de tenant para resolver ticket; toda visualização gravada em `audit_log_entries` append-only (LGPD 5 anos).
- **`financial`** — Reconciliação Stripe, refunds, disputas, MRR dashboard.

### 6.3 Permissões (todas implícitas via admin_bypass, exceto gravações sensíveis que têm audit trail obrigatório)

| Ação | Admin |
|---|---|
| Dashboard financeiro (MRR, assinaturas, churn) | ✅ |
| Gestão global de tenants (listar, suspender, reativar, blocked) | ✅ |
| Gestão de usuários (listar, bloquear, editar role) | ✅ |
| Moderação cross-tenant de conteúdo (vídeos, fórum, Banco Talentos) | ✅ |
| Revisar denúncias de assédio (Connect Me HarassmentReport) — aplicar warning/suspension/ban | ✅ |
| Configurações da plataforma (categorias, parâmetros, matriz funcional) | ✅ |
| Broadcasts segmentados (email via Resend / SMS via Twilio) | ✅ |
| Aprovação manual de CNPJs (quando validação automática falha) | ✅ |
| Controle de inadimplentes (via Stripe webhooks) | ✅ |
| Refunds e disputas Stripe (financeiro) | ✅ |
| Ver audit trail completa (Timeline + `audit_log_entries` + `lgpd_logs`) | ✅ |
| Criar lives (D-097 Fase 11 revoga D-063/D-076 — admin + Empresa Pro + agência delegada; primeira live = apresentação da plataforma) | ✅ |
| Bypass ABAC em leitura cross-tenant (com audit obrigatório) | ✅ |
| `PATCH /companies/{id}/reputation_tier` direto | ❌ (GR-028 — retorna 403 mesmo para admin; reputação é determinística) |

### 6.4 Dados de cadastro

Internos; não passa por fluxo de cadastro público. Provisionado manualmente no Zitadel pelo team Master Síndico.

### 6.5 Device policy + MFA

- **1 dispositivo ativo** (Rule 8).
- **MFA obrigatório** (ADR-0026 admin-mfa-m1) — TOTP em M1; WebAuthn em M2+.
- **IP allowlist** para `super_admin` em M3+.

### 6.6 Trial

Não se aplica.

### 6.7 Jornada canônica

Sem jornada numérica própria. Painel admin vive embutido nas apps existentes:
- Moderação de vídeos → dentro da área de Conteúdo da app síndico/empresa com UI extra.
- Gestão de tenants → área admin na sidebar quando role=admin detectada.
- Broadcasts → central de comunicação cross-app.
- Financeiro → módulo Billing com view admin.

Detalhes completos em [[../03-subprojects/admin-privileges]].

---

## Sub-roles operacionais transversais (não são personas próprias)

Conceituais, implementados como `user_role_type` no enum interno (`personas-and-plans.md:133-143`) + scoped permissions ABAC:

- **`principal`** — síndico principal OU titular de unidade (ambos usam o mesmo valor do enum, desambiguação por contexto de tenant/role).
- **`auxiliary`** — subsíndico. Delegado do síndico titular; maioria das ações exceto encerrar mandato e designar novo síndico.
- **`assistant`** — assistente de síndico. Acesso operacional sem voto vinculante.
- **`titular`** — titular de unidade (morador principal). Vota em assembleia com peso = fração ideal.
- **`dependent`** — dependente (filho menor, cônjuge). Leitura apenas; sem voto.
- **`administrator`** — administrador de empresa (representante legal / gestor de conta).
- **`operator`** — operador técnico de empresa (funcionário sem poder de gestão).

Conselho / conselheiros fiscais-consultivos aparecem como scoped permissions ABAC sobre `auxiliary` ou `assistant` (votação colegiada em `supplier_vote_session` para contratos acima de threshold configurável — A-026 legado).

---

## Tabela-resumo consolidada — persona × tier × feature-chave

Tiers universais (ADR-0032). Coluna "Aplicável a role" só aparece quando a feature não é universal.

| Feature | trial | base | plus | pro | Role(s) aplicáveis |
|---|---|---|---|---|---|
| Criar condomínio | ⏳ | 1 | até 15 | até 15 | `syndic` |
| Timeline — criar atividade | ⏳ | ❌ | ✅ | ✅ | `syndic` |
| Timeline — ler | ✅ | ✅ | ✅ | ✅ | todas |
| Comunicado oficial — criar | ⏳ | ❌ | ✅ | ✅ | `syndic` |
| Publicar vídeo institucional (12 tipos) | ⏳ | ❌ | 4-8/mês | 12/mês | `syndic` (4/8), `enterprise` (8/12) |
| Vídeo-currículo 90s Banco Talentos | ❌ | ❌ | ✅ | ✅ | `resident` (pagante) |
| Ver/buscar currículos Banco Talentos | ❌ | ❌ | ✅ | ✅ | `enterprise`, `marketing` |
| Connect Me Síndico → Empresa | 0 | 2/ano | 4/ano | ilimitado | `syndic` |
| Connect Me Morador → Síndico | 0 | **0/ano** (D-058 Fase 8) | 4/ano | 4/ano | `resident` |
| Connect Me Empresa → Empresa (E→E) | ❌ | n/a | ✅ | ✅ | `enterprise` |
| Connect Me Síndico → Parceiro Vizinhança | 0 | ⚠️ | ✅ | ✅ | `syndic` |
| Votar em assembleia | ❌ | ❌ | ✅ | ✅ | `resident` (titular) |
| Procuração (conceder/receber) | ❌ | ❌ | ✅ | ✅ | `resident` (titular) |
| Convocar assembleia / painel presidente | ⏳ | ✅ | ✅ | ✅ | `syndic` |
| Criar live | — | — | — | — | `admin` only (D-063 Fase 8) |
| Validar execução de empresa | ✅ | ✅ | ✅ | ✅ | `syndic` |
| Registrar execução | ⏳ | — | ⏳ | ✅ | `enterprise` |
| Publicar cupom Vizinhança | ⏳ | ❌ | ✅ | ✅ | `local_company` |
| Palavra-chave do dia | ❌ | ❌ | ⚠️ | ✅ | `local_company` |
| Conceder seal "Síndico-aprovado" | ❌ | ❌ | ✅ | ✅ | `syndic` |
| Ver métricas agregadas próprias | ⏳ | ❌ | ✅ | ✅ | `enterprise`, `local_company`, `syndic` |
| Exportar PDF timeline | ⏳ | ⏳ | ✅ | ✅ | `syndic` |
| Academia LMS — certificado | ❌ | ❌ | ✅ (Morador Pagante / Empresa+) | ✅ | `resident`, `syndic`, `enterprise` |
| Academia LMS — criar curso | ❌ | ❌ | ❌ | ✅ | `enterprise` |
| Fórum LMS — criar tópico | ❌ | ❌ | ❌ | ❌ (ninguém) | — |
| Delegar Agência de Marketing | ❌ | n/a | ✅ | ✅ | `enterprise` |
| Admin bypass / audit trail leitura | ✅ | ✅ | ✅ | ✅ | `admin` |

Matriz completa detalhada em [[../04-requirements/matrix-functional]] (pendente reescrita pós-ADR-0032) e em `functional-matrix.md` (backend) após DT-004 fechar.

---

## Invariantes de identidade (transversais a todas as personas)

- **1 CPF = 1 User**; **1 CNPJ = 1 User** (aggregate `User` em `identity`).
- Persona é derivada do **membership ativo** em tenant específico, não do cadastro da identidade.
- O mesmo User pode operar como múltiplas personas em tenants diferentes (síndico no Condomínio X, morador no Y, representante de Empresa Z).
- **Cross-persona mobility** (ADR-0032 §Upgrade/downgrade entre personas): se Morador vira Síndico (eleição), `user.role` muda; `plan_id` permanece; quotas recalculadas pela nova matriz `(novo_role, plan_tier)`. Se novo role exige plano mínimo (ex: Síndico precisa pelo menos `plus`), backend aplica soft-block até upgrade voluntário.
- Role Zitadel é o dado autoritativo; sub-role (`user_role_type`) é lógica interna.
- Revogação de membership em cascade: titular revogado → dependentes bloqueados (Req 17).
- 1-device-policy transversal (Rule 8 / ADR-0014).

---

## Terminologia legada (tradução explícita para o canônico)

Quando consultar fontes legadas ou históricas, traduzir sempre:

- *"fonte usa rótulo 'Síndico N1' banido por D-067 Fase 7 — canônico aqui é `(role=syndic, plan_tier=base)`"*
- *"fonte usa rótulo 'Síndico N2' banido por D-067 — canônico é `(role=syndic, plan_tier=plus)`"*
- *"fonte usa rótulo 'Síndico N3' banido por D-067 — canônico é `(role=syndic, plan_tier=pro)`"*
- *"fonte usa slug `enterprise_plus` banido por ADR-0032 — canônico é `(role=enterprise, plan_tier=plus)`"*
- *"fonte usa slug `enterprise_pro` banido por ADR-0032 — canônico é `(role=enterprise, plan_tier=pro)`"*
- *"fonte usa slug `resident_base` banido por ADR-0032 — canônico é `(role=resident, plan_tier=base)`"*
- *"fonte usa slug `resident_paid` ou rótulo 'Morador Pagante' — canônico é `(role=resident, plan_tier ∈ {plus, pro})`"*
- *"fonte usa slug `local_company_standard` banido por ADR-0032 — canônico é `(role=local_company, plan_tier=plus)` no MVP"*
- *"fonte usa slug `marketing_standard` banido por ADR-0032 — agência não consome plano próprio no MVP; opera sob grant ABAC da empresa cliente em tier `plus+`"*

---

## Referências

- **ADR canônica**: [[../02-architecture/adr/0032-global-plans-stripe-style]]
- **Decisões vivas**: `STATE.md` Fase 8 §D-056..D-069 + Fase 7 §D-056..D-070 (ver aviso DT-008 sobre duplicação)
- **Matriz funcional canônica (backend)**: `Development/backend/.kiro/specs/master-sindico/requirements/functional-matrix.md` (pendente DT-004 — reescrita sem N1/N2/N3)
- **Personas e planos (backend)**: `Development/backend/.kiro/specs/master-sindico/requirements/personas-and-plans.md`
- **Sub-produtos (fonte autoritativa de coerência)**: [[../00-product/sub-produtos/_moc]]
- **Admin como role transversal**: [[../03-subprojects/admin-privileges]]
- **Enums canônicos**: `backend/internal/modules/institutional/domain/enums/enums.go`
- **Links internos**: [[vision]], [[portfolio-de-produtos]], [[business-model]], [[glossary]], [[../01-domain/bounded-contexts]]
