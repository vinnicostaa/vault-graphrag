---
title: "11-administradoras-condominiais — Administradora Condominial (entidade + vínculo empresa síndica)"
type: spec
tags: [sub-produto, master-sindico, v2, fase-8, administradora, empresa-sindica, institutional, commercial, delegation]
source: Fase 8 — destilação exaustiva (código Go + .kiro + Content + Vault + schema TS)
created: 2026-04-23
updated: 2026-04-23
status: destilado
aliases: [administradora, empresa-síndica, syndic-company]
---

# 11 — Administradoras Condominiais

Empresa especializada em **administração condominial** (contábil, jurídica, prestação de contas, suporte operacional ao síndico). No MVP, **não é produto separado**: é um **tipo de Empresa** (role=`enterprise`, subcategoria `ADMINISTRADORAS_CONDOMINIOS` dentro da categoria `ADMINISTRACAO_GESTAO_GOVERNANCA`) que pode ser **vinculada** a um condomínio via aggregate `EmpresaSindicaLink` no bounded context `commercial`, com **permissões delegadas configuráveis**.

> **Nota de linguagem:** o projeto usa 3 termos que se sobrepõem parcialmente:
> 1. **Administradora** — termo de mercado (empresa de administração condominial) e termo usado em Assembleia (ex: "administradora presente na mesa").
> 2. **Empresa Síndica** — termo do aggregate/fluxo de delegação (`EmpresaSindicaLink`, `empresa_sindica_links`).
> 3. **Administradoras Condominiais** — rotulagem deste sub-produto + subcategoria no enum de empresas.
>
> No MVP, os três referem-se à **mesma entidade** (Empresa com subcategoria `ADMINISTRADORAS_CONDOMINIOS` que aceitou um convite de vínculo com um condomínio). No M5+, a feature `empresa-síndica` evolui para **tenant multi-condomínio próprio com dashboard consolidado** (escopo futuro — seção 17).

---

## 1. Fontes consultadas

| Fonte | Caminho | Função |
|---|---|---|
| .kiro Institutional | `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/backend/.kiro/specs/master-sindico/requirements/institutional.md` | Req 6.2 (`administradoras_vinculadas`), Req 7 (mandato + vínculo empresa síndica) |
| .kiro Commercial | `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/backend/.kiro/specs/master-sindico/requirements/commercial.md` | Req 18 (registro de empresa), status/role de empresa, regras de conteúdo |
| .kiro Identity | `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/backend/.kiro/specs/master-sindico/requirements/identity.md` | roles (`enterprise`, `admin`) e role_types (`administrator`, `operator`) |
| .kiro Personas/Plans | `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/backend/.kiro/specs/master-sindico/requirements/personas-and-plans.md` | Tiers `plus`/`pro` para role `enterprise` |
| Content requirements | `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/Content/requirements.md` | Req 93 AC1-10 (Empresa Síndica Registration and Linking), refs administradora em Assembly (Req 66+) |
| Content inventário | `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/Content/02-Jornadas/Inventario-Telas.md` | S5 (Cadastro / Vínculo Empresa Síndica), R6 sindic |
| Content regras | `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/Content/regras-de-negocio/regras-canonicas.md` | S5 na matriz de telas |
| Schema categorias | `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/enterprise-catergories-subcategories-enum.schema.ts` | Subcategoria `ADMINISTRADORAS_CONDOMINIOS` ∈ categoria `ADMINISTRACAO_GESTAO_GOVERNANCA` (primeira subcategoria da primeira categoria) |
| Go domain aggregate | `Development/backend/internal/modules/commercial/domain/entities/empresa_sindica_link.go` | Entidade raiz + permissões + state machine |
| Go domain enums | `Development/backend/internal/modules/commercial/domain/enums/enums.go` | `EmpresaSindicaLinkStatus` (pending/active/revoked) |
| Go repo interface | `Development/backend/internal/modules/commercial/domain/repositories/i_empresa_sindica_repository.go` | Create/FindByID/FindByToken/Save/ListByCondominium |
| Go use cases | `Development/backend/internal/modules/commercial/application/use_cases/empresa_sindica_use_cases.go` | Invite/Accept/UpdatePermissions/Revoke/List |
| Go HTTP handler | `Development/backend/internal/modules/commercial/infrastructure/http/routes/empresa_sindica_handler.go` | 5 endpoints REST (invite/accept/update-perm/revoke/list) |
| Go module wiring | `Development/backend/internal/modules/commercial/module.go` | Wiring dos 5 use cases + handler |
| Go models gerados | `Development/backend/internal/modules/institutional/infrastructure/database/generated/models.go` | Struct `EmpresaSindicaLink` (sqlc) em **módulo institutional** (cross-context via sqlc generation) |
| Migration | `Development/backend/internal/shared/providers/database/migrations/00307_create_empresa_sindica.sql` | Tabela `empresa_sindica_links` + enum `empresa_sindica_link_status` + 3 índices |
| Migration CNPJ | `Development/backend/internal/shared/providers/database/migrations/00310_normalize_empresa_sindica_cnpj.sql` | Normalização de CNPJ para 14 dígitos |
| Vault Institutional Arch | `/home/vinni/Documentos/Obsidian Vault/Clients/MasterSindico/01 - Arquitetura de Sistema/System Architecture - Institutional.md` | Entity `EmpresaSindica` listada em CondominiumModule (Use Case `LinkEmpresaSindica`) |
| Vault Identidade | `/home/vinni/Documentos/Obsidian Vault/Clients/MasterSindico/02 - Domínios e Requisitos/Domínio - Identidade.md` | Req 7 AC9-12 (vínculo + 5 permissões + token + audit) |
| Vault Assembleia | `/home/vinni/Documentos/Obsidian Vault/Clients/MasterSindico/02 - Domínios e Requisitos/Domínio - Assembleia.md` | Administradora na mesa, validação de procurações, emissão de inadimplentes |

---

## 2. Propósito

A **Administradora Condominial** é a empresa profissional que dá **suporte operacional e administrativo** ao síndico eleito/profissional. Ela cobre rotinas que o síndico, sozinho, não costuma ter tempo ou expertise para executar:

- **Contabilidade condominial** (categoria/subcategoria dedicada no enum).
- **Prestação de contas** mensal/anual.
- **Assessoria jurídica / cobrança extrajudicial e judicial**.
- **Emissão de lista de inadimplentes** (obrigatória até **1h antes da 1ª convocação** de assembleia — Vault Assembleia:56).
- **Gestão documental técnica** (laudos, AVCB, plantas, relatórios).
- **Suporte em Assembleia** (presidir mesa junto com síndico, validar procurações 24h antes, homologar votação, emitir edital — Content requirements:1313-1622).
- **Validação delegada** de registros de execução de outras empresas e de comunicados técnicos, quando o síndico autoriza (permissões 4 e 5 — ver seção 7).

No MVP (M1 — 2026-05-08), a Administradora **existe como tipo de Empresa** (role `enterprise` + subcategoria `ADMINISTRADORAS_CONDOMINIOS`). A relação condomínio ↔ administradora é modelada pelo aggregate `EmpresaSindicaLink` do bounded context `commercial`, iniciado pelo síndico na tela **S5** (Content Inventario-Telas:55).

---

## 3. O que NÃO é

- Não substitui o síndico eleito. O síndico é a figura legal (Código Civil art. 1.347) que responde pelo condomínio; a administradora apenas dá suporte operacional.
- Não tem poder decisório em assembleia. Em Assembleia, a administradora pode ocupar a mesa (`administradora_presente` — Content requirements:1553, 1636), homologar votação (junto com síndico — Content requirements:1622) e apresentar documentos no telão (Content requirements:1542, 1579), mas o **voto ponderado por fração ideal** pertence aos moradores e o voto de desempate é do presidente da mesa (síndico).
- Não é uma aplicação separada. É um **tipo de Empresa** no mesmo painel E1-E16 (painel empresarial, 16 telas — Inventario-Telas:113-129). Não tem login dedicado nem dashboard próprio de administradora no MVP.
- Não é Agência de Marketing. Agência (`marketing`) é outra role, vinculada a uma **Empresa** (não a um condomínio), que delega apenas publicação de vídeos — ver [[10-marketing-agencias]].
- Não é Empresa Prestadora comum. Empresa comum executa serviços pontuais (limpeza, elevadores, segurança), registra execuções e aguarda validação do síndico. A administradora é **delegatária de poderes de gestão**, com permissões configuráveis inclusive para **validar execuções das outras empresas** em nome do síndico.
- Não é Parceiro da Vizinhança. Parceiro (`local_company`) é comércio local para moradores; não participa de governança.
- Não confundir com Admin da plataforma. `admin` é role da equipe Master Síndico que opera via `apps/admin` dedicada no monorepo web (D-134 revoga D-058/D-072); nada a ver com Administradora Condominial.

---

## 4. Modelagem canônica (entidade + vinculação)

### 4.1. Administradora como Empresa (perfil)

A Administradora é **uma `Company` comum** (bounded context `commercial`, tabela `companies`), criada pelo fluxo padrão do Req 18 (commercial.md:16-39):

- `role` (Zitadel + PG) = `enterprise` (identity.md:52)
- `role_type` interno = `administrator` (admin da empresa) ou `operator` (operador técnico) — identity.md:56, personas-and-plans.md:141-142
- `categoria_principal` = `ADMINISTRACAO_GESTAO_GOVERNANCA` (schema TS:8)
- `subcategorias_atuacao` inclui `ADMINISTRADORAS_CONDOMINIOS` (schema TS:79, CATEGORIA_SUB_MAP linha 587-594)
- Tier disponível: `plus` ou `pro` (personas-and-plans.md:46-47) — mesma política das demais empresas
- Status (enum `CompanyStatus`): `pending_verification` → `active` → `suspended` → `blocked` (enums.go:7-12; commercial.md:24-27)
- Cadastro: Req 6.3 em institutional.md:55-90 — logo, razão social, CNPJ único (ViaCEP, Receita Federal), 25 compliance markers autodeclarados

### 4.2. Vinculação Condomínio ↔ Administradora via `EmpresaSindicaLink`

O vínculo é **aggregate próprio** no bounded context `commercial`, modelado em `empresa_sindica_link.go`:

- **Chave do vínculo**: `(condominium_id, company_id)` quando ativo; `(condominium_id, token)` na fase de convite.
- **Fluxo**: Síndico (`createdBy`) preenche S5 → backend gera UUIDv7 como `token` → envia email para a administradora → administradora clica, valida token (`FindByToken`), chama `Accept(company_id)` → `status` pula `pending` → `active` e `linked_at` é carimbado.
- **Campos canônicos** (empresa_sindica_link.go:32-46):
  - `id` (UUIDv7)
  - `condominium_id` (FK institutional.condominiums — cross-context)
  - `company_id` (nullable até aceitar; FK commercial.companies)
  - `company_name`, `cnpj` (14 dígitos normalizados via VO `CNPJ`), `email`
  - `token` (UUIDv7, UNIQUE, usado só enquanto `pending`)
  - `status: EmpresaSindicaLinkStatus`
  - `permissions: EmpresaSindicaPermissions` (5 booleans — ver §7)
  - `linked_at *time.Time` (preenchido no aceite)
  - `created_by` (user_id do síndico)
  - `created_at`, `updated_at`
- **Factory** `NewEmpresaSindicaLink`: valida `condominium_id` não-vazio, `company_name` ≥ 2 chars (trim), CNPJ pelo algoritmo da Receita Federal, `email` não-vazio, `token` não-vazio. Status inicial sempre `pending`, permissões default `false`.
- **Reconstruction** `ReconstructEmpresaSindicaLink`: hidratação sem validações (repo → aggregate).

### 4.3. Relação com `SyndicMandate` (institutional)

O mandato do síndico (`syndic_mandate.go` em institutional) é **independente** do `EmpresaSindicaLink`:

- Mandato: vínculo síndico ↔ condomínio com tipos `sindico_eleito | sindico_profissional | sindico_interino` (institutional.md:137, 329).
- Administradora: vínculo **adicional** condomínio ↔ empresa administradora — não substitui mandato.
- Req 6.2 permite `administradoras_vinculadas` (lista de empresa IDs) no perfil do Síndico para tier ≥ plus (institutional.md:45 — termo legado "N2/N3" na fonte precisa reescrita via DT-004; aqui aplicado canônico `plan_tier >= plus` por ADR-0032) — é **dado declarativo** no perfil do síndico (usado para vitrine/histórico), separado da vinculação operacional ativa via `EmpresaSindicaLink`.

### 4.4. Cross-context sqlc

Embora o aggregate `EmpresaSindicaLink` viva em `commercial/domain/entities`, o **struct gerado por sqlc** (`EmpresaSindicaLink`) está em `institutional/infrastructure/database/generated/models.go` (leitura do grep — models.go referencia `EmpresaSindicaLinkStatus`). A migração `00307_create_empresa_sindica.sql` cria a tabela no schema default (sem prefixo de contexto). **DT-###**: essa é uma pendência de organização de queries sqlc (código compila, mas a duplicação de location é confusa — gap de arquitetura, registrar).

---

## 5. Personas envolvidas

| Persona | Papel | Zitadel role | role_type |
|---|---|---|---|
| **Síndico Titular** | Quem **convida** a administradora, configura permissões, revoga. Criador (`created_by`) do link. Titular do mandato em institutional. | `syndic` | `principal` (ou `auxiliary` se autorizado em S10/Req 10) |
| **Administradora (representante)** | Usuário da empresa administradora que **aceita** o convite e opera as ações delegadas. | `enterprise` | `administrator` (admin da empresa) ou `operator` (operador técnico) |
| **Morador** | **Consumidor passivo** — lê publicações de timeline (síndico/administradora), participa de enquetes/eventos, vê relatórios. Não interage diretamente com o vínculo. | `resident` | `titular` ou `dependent` |
| **Admin Master Síndico** | Papel transversal (D-058). Pode auditar, revisar denúncias de abuso, suspender Company administradora (Req 19 anti-assédio). | `admin` | — |

### Quem NÃO participa

- **Agência de Marketing** (`marketing`) — out of scope (ver [[10-marketing-agencias]]).
- **Parceiro da Vizinhança** (`local_company`) — out of scope (ver [[08-vizinhanca]]).

---

## 6. Tiers × Role

Administradora = Empresa (`enterprise`). Logo segue a política global **Stripe-style Tier-Universal** (ADR-0032):

| Tier | Acesso típico de Administradora |
|---|---|
| `trial` (7 dias) | Full-visibility com bloqueios: não pode publicar vídeos, registrar execuções, gerenciar usuários (personas-and-plans.md:68) |
| `base` | N/A para `enterprise` — empresas não têm plano base free (personas-and-plans.md:46) |
| `plus` | Todas as features operacionais padrão da empresa + Connect Me E→E (Req 19.2 commercial.md:109) |
| `pro` | Plus + Connect Me ilimitado + cursos certificados criar (personas-and-plans.md:121) + perfil Pro visível para todos (institutional.md:88) |

**Importante:** a permissão para **operar como empresa síndica de um condomínio específico** é **ortogonal** ao plano. Qualquer empresa `active` (status operational — enums.go:27) pode ser convidada por um síndico, independentemente do tier. O tier afeta apenas **o que a empresa pode fazer no painel E1-E16** (publicar vídeos, quotas, visibilidade pública).

---

## 7. Permissões configuráveis delegadas

O síndico configura no **momento do convite** (e pode atualizar depois) **5 toggles booleanos** — mapeados 1:1 no código, schema SQL e Content requirements (Req 93 AC6 — requirements.md:3129; Req 7 AC10 — institutional.md ref, vault Identidade:331):

| # | Nome (Go) | Nome (JSON API) | Nome (SQL) | Função |
|---|---|---|---|---|
| 1 | `ViewPublications` | `view_publications` | `perm_view_publications` | Visualizar publicações da Timeline do condomínio (entradas `atividade_da_gestao`, `comunicado`, `evento`, `registro_de_execucao`, `adendo`, etc. — institutional.md:246-253) |
| 2 | `EditSyndicPosts` | `edit_syndic_posts` | `perm_edit_syndic_posts` | Editar publicações do síndico na Timeline — conflita com **Rule 2 / Invariante Timeline append-only**: a edição aqui é feita via **adendo** (Req 25 — commercial.md:316-338), nunca UPDATE direto |
| 3 | `HidePublications` | `hide_publications` | `perm_hide_publications` | Ocultar publicações (visibilidade, não deleção — Inventario-Telas:94 "nada é deletado, apenas editar/ocultar com trilha") |
| 4 | `ValidateExecutions` | `validate_executions` | `perm_validate_executions` | Validar **registros de execução** de outras empresas (commercial.md:241-266, Req 23). No MVP essa validação é do síndico; com permissão ativa, pode ser feita pela administradora em nome dele. Audit trail obrigatório registra quem validou |
| 5 | `ValidateCommunications` | `validate_communications` | `perm_validate_communications` | Validar **comunicados técnicos** emitidos por empresas (ex: Comunicados Pós-Deal — Req 24, commercial.md:290-312) |

**Default** (novo link): todos `false`. Síndico escolhe no momento do convite quais habilitar (campo `permissions` do body do POST — empresa_sindica_handler.go:55-61).

**Nomenclatura divergente** entre fontes: o termo **`validar_comunicados_empresas`** aparece no Vault Identidade:331 e `validar_comunicados_tecnicos` em Content requirements:3129 — o código canoniza como `ValidateCommunications` / `perm_validate_communications`. Registrar como **A-### findings** (divergência doc ↔ doc, não bloqueante).

### 7.1. UpdatePermissions (substituição completa)

A função `UpdatePermissions(p EmpresaSindicaPermissions)` (empresa_sindica_link.go:143-147) faz **substituição total** (replace, não merge) — o síndico sempre envia as 5 flags no PUT, não um patch parcial (handler:117-137). Isso simplifica o front e garante atomicidade.

### 7.2. Regras de atualização

- Link com status `revoked` **não pode ter permissões atualizadas** (guard em `UpdateEmpresaSindicaPermissionsUseCase.Execute` — use_cases:115-117, erro `ErrConflict` "cannot update permissions of a revoked link").
- Link `pending` pode ter permissões atualizadas (útil caso o síndico mude de ideia antes da empresa aceitar — é o caso que motiva `UpdatePermissions` aceitar pending também).
- Não há limite de quantas vezes o síndico atualiza as permissões.

---

## 8. Aggregates

### 8.1. `EmpresaSindicaLink` (aggregate root — bounded context `commercial`)

- **Arquivo**: `Development/backend/internal/modules/commercial/domain/entities/empresa_sindica_link.go`
- **Identidade**: `id` (UUIDv7)
- **Invariantes próprias**:
  - Status só transita pelas transições legais (ver §14).
  - `company_id` só é populado em `Accept` (nunca no construtor).
  - `linked_at` é carimbado uma única vez, na transição `pending → active`.
  - CNPJ armazenado em **14 dígitos** (via `value_objects.CNPJ.Digits()` — empresa_sindica_link.go:74), consistente com `companies.cnpj`.
- **Métodos de domínio**: `Accept(companyID)`, `Revoke()`, `UpdatePermissions(p)`.
- **Errors tipados**: `ErrEmpresaSindicaEmptyCondominiumID`, `ErrEmpresaSindicaInvalidCompanyName`, `ErrEmpresaSindicaEmptyEmail`, `ErrEmpresaSindicaEmptyToken`, `ErrEmpresaSindicaNotPending`, `ErrEmpresaSindicaAlreadyRevoked` (empresa_sindica_link.go:13-19).

### 8.2. `EmpresaSindicaPermissions` (value object — parte do aggregate)

- Struct com 5 booleans (empresa_sindica_link.go:22-28).
- Sem identidade, sem ciclo de vida próprio — substituído integralmente via `UpdatePermissions`.
- Serializado na API em minúsculas `snake_case` (handler:55-61) e no banco como 5 colunas booleanas `perm_*` (migration:16-20).

### 8.3. Repository interface

`IEmpresaSindicaRepository` (commercial/domain/repositories/i_empresa_sindica_repository.go):

```go
Create(ctx, link) error
FindByID(ctx, id) (*EmpresaSindicaLink, error)
FindByToken(ctx, token) (*EmpresaSindicaLink, error)
Save(ctx, link) error
ListByCondominium(ctx, condominium_id) ([]*EmpresaSindicaLink, error)
```

Implementação em `commercial/infrastructure/database/empresa_sindica_repository_impl.go`.

---

## 9. Enums

### 9.1. `EmpresaSindicaLinkStatus` (enum Postgres + enum Go)

- Arquivo Go: `commercial/domain/enums/enums.go:270-289`.
- Valores: `pending | active | revoked`.
- Helpers: `IsValid()`, `IsPending()`, `IsActive()`.
- Postgres: `CREATE TYPE empresa_sindica_link_status AS ENUM ('pending', 'active', 'revoked')` (migration 00307:3).

### 9.2. `EmpresaSindicaPermissions` (5 flags booleanos)

Não é enum, é struct de VO (§8.2). Listado aqui para completude:

```
view_publications       bool
edit_syndic_posts       bool
hide_publications       bool
validate_executions     bool
validate_communications bool
```

### 9.3. Enums relacionados (reuso de outros contextos)

- `CompanyStatus` (enums.go:6-27): `pending_verification | active | suspended | blocked` — aplica-se à empresa administradora como à qualquer outra.
- `CompanyRole` (enums.go:30-43): `admin | operator` — role interna da empresa (não confundir com `user_role` Zitadel).
- `user_role` (Zitadel + PG): `syndic | resident | enterprise | marketing | local_company | admin`.
- `user_role_type` (só local, decisão T8): `principal | auxiliary | assistant | titular | dependent | administrator | operator`.
- `plan_tier`: `trial | base | plus | pro`.

---

## 10. Tela S5 detalhada (Cadastro / Vínculo Empresa Síndica)

### 10.1. Identificação na malha de telas

- **ID**: S5
- **Nome** (Inventario-Telas:55): "Cadastro / Vínculo Empresa Síndica"
- **Módulo**: Registry
- **Jornada**: Síndico (S1-S31) — é a **5ª tela do fluxo de cadastro** (S1 Entrada Governança → S2 Meus Condomínios → S3 Cadastrar Condomínio Endereço → S4 Dados da Gestão/Mandato → **S5 Vínculo Empresa Síndica** → S6 Home da Governança).
- **Também referenciada em**: regras-canonicas.md:55 (mesma listagem).
- **Aparição no onboarding**: Síndico (6 telas) — "Dados pessoais → Tipo síndico → Governance markers → Criar condo → Mandato → **Empresa síndica**" (vault Identidade:229; billing.md:80).

### 10.2. Fluxo resumido

**Passo 1 — Formulário de convite** (ação do Síndico em S5):
- Inputs: `condominium_id` (pré-preenchido, vem do context selector/S2), `company_name`, `cnpj`, `email`.
- Toggles de 5 permissões (todos default `false`).
- Validação client-side: CNPJ com dígitos verificadores.
- Submit → `POST /api/v1/empresa-sindica` (handler:41-87).

**Passo 2 — Backend gera convite**:
- `InviteEmpresaSindicaUseCase.Execute` (use_cases:34-56):
  - Gera `token = utils.NewID()` (UUIDv7).
  - Cria aggregate com `NewEmpresaSindicaLink` (validações).
  - Aplica `permissions` do command.
  - Persiste via `repo.Create`.
  - Loga (campos sem PII — compliant com BeyondCorp `no-PII-logs`): `link_id`, `condominium_id`, `company_name`, `created_by`.
- **Gap**: no código atual não há disparo de email no use case (seria `IEmailGateway.Send(email, token)` via port). Registrar como **A-### finding** — handler devolve o token no response, front provavelmente faz o envio ou é TODO do Sprint 2+.

**Passo 3 — Administradora recebe email**:
- Email contém link com `token`. (Integração de email ainda não no código — ver gap acima.)

**Passo 4 — Administradora aceita**:
- Empresa administradora precisa **já estar cadastrada** como Company (`active`) na plataforma (company_id existe).
- Empresa clica "Aceitar" → front envia `POST /api/v1/empresa-sindica/accept` com `{ token, company_id }`.
- `AcceptEmpresaSindicaInviteUseCase.Execute` (use_cases:74-92):
  - `FindByToken(token)`.
  - `link.Accept(company_id)` — valida status `pending`, seta `company_id`, `linked_at = now`, status → `active`.
  - `repo.Save(link)`.
  - Loga aceite.
- **Gap (pentester)**: não há validação de que o `company_id` do body corresponde ao email do convite. Um atacante com o token poderia aceitar com outra empresa. Registrar como **A-### pentester finding** (IDOR / token replay).

**Passo 5 — Síndico atualiza permissões a qualquer momento**:
- `PUT /api/v1/empresa-sindica/:link_id/permissions` (handler:112-144).
- Substitui os 5 booleans.
- Bloqueado se `status == revoked`.

**Passo 6 — Síndico revoga**:
- `DELETE /api/v1/empresa-sindica/:link_id` (handler:146-156).
- `RevokeEmpresaSindicaUseCase.Execute` (use_cases:141-158).
- Status → `revoked`. Acesso da administradora ao condomínio é bloqueado **no próximo request** (ABAC middleware checa status do link ativo).
- **Revogação imediata** exigida por Req 93 AC8-9 (requirements.md:3131-3132, vault Identidade:333) — implementação real depende de invalidar cache ABAC Redis (D-001 — decisão em aberto sobre purge instantâneo vs staleness 5min — identity.md:80). Ver gap seção 19.

**Passo 7 — Listagem**:
- `GET /api/v1/empresa-sindica?condominium_id=...` (handler:158-169).
- Retorna todos os links do condomínio (pending + active + revoked) — útil para histórico S5.

### 10.3. Regras adjacentes da tela (Inventario-Telas:83-106)

- **R6 do síndico (Inventario-Telas:89)**: "Empresa síndica vinculada pode: visualizar publicações, editar publicações do síndico, ocultar publicações, validar registros de empresas, validar comunicados de empresas" — **fonte de verdade funcional das 5 permissões**.
- **Bloco 4 — Usuários e Permissões (Inventario-Telas:103-106)**: menciona perfis de convite "Síndico Preposto / Administradora / Conselho (leitura ampliada)". **Inconsistência**: esse bloco fala de "usuários do condomínio" (Req 10, até 3: principal/auxiliar/assistente — institutional.md:208-238), **não** do vínculo com Empresa Síndica. Não misturar: "Administradora" como rótulo de user interno != Empresa Administradora vinculada via S5.

### 10.4. Anti-erro de contexto (Req 94)

Síndico com 2+ condomínios **precisa confirmar contexto antes de publicar** (Req 94, requirements.md:3135-3148) — aplica também a S5 (convite é ação de publicação; deve mostrar modal "Convidar empresa síndica para Condomínio X?" antes de efetivar).

---

## 11. Audit trail obrigatório

Req 93 AC7 (requirements.md:3130) e AC10 (3133) — "THE Empresa_Sindica_Manager SHALL log all empresa síndica actions in audit trail" e "SHALL maintain empresa síndica action history per condominium".

### 11.1. Ações a serem logadas

Toda ação executada **pela administradora vinculada** em nome do síndico (ou pelo próprio síndico sobre o vínculo) deve emitir audit log:

| Ação | Ator | Resource | Metadata |
|---|---|---|---|
| `empresa_sindica.invited` | syndic user | EmpresaSindicaLink | link_id, company_name, permissions snapshot |
| `empresa_sindica.accepted` | enterprise user | EmpresaSindicaLink | link_id, company_id, linked_at |
| `empresa_sindica.permissions_updated` | syndic user | EmpresaSindicaLink | link_id, diff de permissions |
| `empresa_sindica.revoked` | syndic user | EmpresaSindicaLink | link_id, reason (não há campo no código — **gap**) |
| `empresa_sindica.timeline_post_edited` | enterprise user (via EditSyndicPosts) | TimelineEntry (adendo) | entry_id, original_id |
| `empresa_sindica.publication_hidden` | enterprise user (via HidePublications) | TimelineEntry | entry_id |
| `empresa_sindica.execution_validated` | enterprise user (via ValidateExecutions) | ExecutionRecord | execution_record_id, decision |
| `empresa_sindica.communication_validated` | enterprise user (via ValidateCommunications) | PostDealComunicado | comunicado_id, decision |

### 11.2. Estado atual no código

- Os **use cases logam** via `logger.Info` com structured fields (empresa_sindica_use_cases.go:49-54, 86-90, 123-127, 153-157) — é **log operacional**, não audit trail LGPD-compliant.
- Audit logger formal existe na cadeia de middleware (identity.md:69 "AuditLogger (Sprint 3)") — **ainda não aplicado aos endpoints `/empresa-sindica`**. Registrar como **A-### finding** (obrigatório para Req 93 AC7).
- Sem tabela `empresa_sindica_audit_log` dedicada (migração 00307 não cria).

### 11.3. Compliance transversal

Toda ação de administradora logada precisa carregar:
- `timestamp`, `actor_user_id`, `actor_role`, `link_id`, `condominium_id`, `action`, `resource_id`, `ip_address`, `user_agent`.
- LGPD log se envolve dados de terceiros (Req 19 commercial.md:60 — padrão extensível).
- Retenção: regra do domínio `cross-domain` (Req 33-47) — mínimo 5 anos para atos de gestão.

---

## 12. Regras de negócio

| ID local | Regra | Fonte |
|---|---|---|
| BR-AC-01 | 1 administradora ativa por condomínio por vez (§13) | Req 93 AC3, convenção — "WHEN empresa síndica accepts invitation, SHALL link to condominium" (singular) |
| BR-AC-02 | Administradora pode operar em múltiplos condomínios; **cada condomínio exige convite separado** (link próprio) | Consequência do modelo: `ListByCondominium` scope por condo, `company_id` repetível em múltiplos links active |
| BR-AC-03 | Síndico pode editar as 5 permissões **a qualquer momento** em link `pending` ou `active` | use_cases:115-117 |
| BR-AC-04 | Revogação é **imediata** e **terminal** — não existe "re-ativar" um link revoked | state machine §14; revogação bloqueia `UpdatePermissions` |
| BR-AC-05 | Para re-vincular após revogação, síndico cria **novo convite** (novo `id`, novo `token`) | Consequência de BR-AC-04 |
| BR-AC-06 | CNPJ é armazenado em 14 dígitos, normalizado via VO | empresa_sindica_link.go:74 |
| BR-AC-07 | Token é UUIDv7 gerado backend, UNIQUE na tabela, usado só enquanto `pending` | migration:12, 29 (index parcial) |
| BR-AC-08 | Empresa administradora precisa existir como Company `active` antes do aceite | use case Accept usa `company_id` do body — dependência externa |
| BR-AC-09 | Usuário da administradora precisa ter role `enterprise` + role_type `administrator` ou `operator` | identity.md:56 |
| BR-AC-10 | Link ativo NÃO impede o síndico de publicar/validar diretamente | Permissões são **delegação**, não **substituição** exclusiva |
| BR-AC-11 | Timeline permanece append-only mesmo via `EditSyndicPosts` — edição acontece via **adendo** (Req 25) | institutional.md:421 (Rule 2); commercial.md:316-338 |
| BR-AC-12 | Administradora sob `trial` expirado entra em soft-block — link não é revogado, mas ações ficam `403 PLAN_REQUIRED` | personas-and-plans.md:74 (Decision 4) |
| BR-AC-13 | Admin Master Síndico pode suspender Company administradora por abuso (Req 19 anti-assédio — commercial.md:62); `CompanyStatus=suspended` bloqueia operação, mas link permanece para trilha | enums.go:25 (`IsOperational` só para `active`) |

---

## 13. Invariantes

**INV-AC-01** — Em qualquer momento, para cada par `(condominium_id, status=active)` existe **no máximo 1 `EmpresaSindicaLink`** ativo. (Regra lógica; **não há constraint SQL** que garanta isso — migration 00307 não tem UNIQUE parcial. Registrar como **A-### finding** — race condition: dois convites aceitos quase simultaneamente geram dois links active para o mesmo condo. Sugerir `CREATE UNIQUE INDEX ... WHERE status = 'active'`.)

**INV-AC-02** — Token é único enquanto link `pending`. Tabela tem `UNIQUE` em `token` global (migration:12), reforçado por index parcial em `WHERE status = 'pending'`.

**INV-AC-03** — `company_id` é obrigatório em links `active` e `revoked` (carimbado no `Accept`) e NULL em links `pending`. Garantido pela state machine; SQL permite `company_id` NULL em qualquer status (migration:8) — registrar como soft invariant.

**INV-AC-04** — `linked_at` é imutável pós-Accept. Garantido pelo aggregate (só setado em `Accept`, não há setter público).

**INV-AC-05** — CNPJ sempre 14 dígitos no banco (CHECK `length(cnpj) = 18` no migration original 00307:10 usa **formato mascarado**; migration 00310 normaliza para 14 dígitos). Verificar versão final do schema.

**INV-AC-06** — Toda ação delegada (toggles 2-5 ligados) gera audit entry. Invariante **aspiracional**, pendente de implementação (§11.2).

**INV-AC-07** — `EmpresaSindicaLink.status` transita apenas pelas transições válidas da state machine §14.

---

## 14. State machine

```
         INVITE (Invite UC)
              │
              ▼
         ┌─────────┐
         │ pending │──────────────────┐
         └────┬────┘                  │
              │ Accept(company_id)    │ Revoke()
              ▼                       ▼
         ┌─────────┐            ┌─────────┐
         │ active  │──Revoke()─▶│ revoked │  (terminal)
         └─────────┘            └─────────┘
```

### 14.1. Transições legais

| De | Para | Gatilho | Método | Efeito |
|---|---|---|---|---|
| (nada) | `pending` | Síndico cria convite em S5 | `NewEmpresaSindicaLink` | Link criado com permissões e token |
| `pending` | `active` | Empresa aceita com token válido | `Accept(companyID)` | Seta `company_id` + `linked_at` |
| `pending` | `revoked` | Síndico cancela antes do aceite | `Revoke()` | Link fica no histórico; token invalida |
| `active` | `revoked` | Síndico revoga vínculo | `Revoke()` | Administradora perde acesso no próximo request |
| `revoked` | (nada) | — | — | Terminal |

### 14.2. Guards no código

- `Accept`: `if !l.status.IsPending() { return ErrEmpresaSindicaNotPending }` (empresa_sindica_link.go:121-123).
- `Revoke`: `if l.status == Revoked { return ErrEmpresaSindicaAlreadyRevoked }` (empresa_sindica_link.go:134-136).
- `UpdatePermissions`: sem guard no aggregate, mas use case barra em `revoked` (use_cases:115-117).

### 14.3. Transições inválidas

- `active → pending`: impossível (sem método).
- `revoked → active/pending`: impossível (terminal).
- `pending → pending` (re-convite mesmo link): impossível — síndico cria **novo** link.

---

## 15. Domain events

### 15.1. Estado atual

**Gap**: o aggregate `EmpresaSindicaLink` **não emite domain events** no código atual. Use cases fazem apenas `logger.Info` (empresa_sindica_use_cases.go:49-54 etc.). Não há struct/interface de domain event em `commercial/domain/events/` para esse aggregate (só existe diretório `institutional/domain/events/` que não referencia empresa síndica).

Content requirements:3127 especifica: "THE Empresa_Sindica_Manager SHALL emit `EmpresaSindicaLinked` event" (Req 93 AC4). **Implementação pendente.**

### 15.2. Eventos canônicos a emitir (spec para Sprint 2+)

| Nome | Quando | Payload |
|---|---|---|
| `commercial.empresa_sindica.invited` | Síndico cria convite | `link_id, condominium_id, company_name, cnpj, email, permissions, created_by, created_at` |
| `commercial.empresa_sindica.accepted` | Empresa aceita convite | `link_id, condominium_id, company_id, linked_at` |
| `commercial.empresa_sindica.permissions_updated` | Síndico atualiza toggles | `link_id, old_permissions, new_permissions, updated_by, updated_at` |
| `commercial.empresa_sindica.revoked` | Síndico revoga | `link_id, condominium_id, company_id (se tinha), revoked_at, revoked_by` |
| `commercial.empresa_sindica.delegated_action` | Administradora executa ação com permissão | `link_id, action (edit_post/hide/validate_exec/validate_comm), resource_id, actor_user_id, timestamp` |

Distribuição via **NATS** (padrão do projeto — institutional.md:267 "timeline.entry.created" etc.).

### 15.3. Consumidores esperados

- **Timeline (institutional)**: `commercial.empresa_sindica.accepted` → publica entrada tipo `comunicado` de "Nova empresa síndica vinculada".
- **Audit log (cross-domain)**: todos os eventos acima.
- **Notification service**: `invited` → email para empresa; `revoked` → email para empresa + síndico.
- **ABAC cache invalidation**: `revoked` → purge de `ms:auth:{userID}` dos usuários da empresa (D-001).

---

## 16. Dependências

### 16.1. Entradas (bounded contexts dos quais depende)

- **Identity**: usuários (role `enterprise`, role_type `administrator`/`operator`); ABAC para autorização de cada request; cache Redis para revogação instantânea (§15.3).
- **Institutional**: `Condominium` (FK `condominium_id`); `SyndicMandate` (contexto do síndico titular); `TimelineEntry` (resource impactado pelas permissões 2-3); `MasterPlan` (status muda via Timeline).
- **Commercial — Company**: `Company` aggregate (FK `company_id` pós-aceite); status `active` exigido.
- **Commercial — ExecutionRecord / PostDealComunicado**: resources validados via permissões 4-5.

### 16.2. Saídas (bounded contexts que dependem dele)

- **Assembly**: administradora na mesa (`administradora_presente` — Content requirements:1553, 1636), validar procurações (requirements:1418), homologar votação (requirements:1622), emitir lista de inadimplentes 1h antes (vault Assembleia:56). Link ativo é precondição para esses papéis.
- **Institutional — Timeline**: publicações editadas por administradora (via `EditSyndicPosts`) exigem adendo com autor-administradora (`actor_user_id`) e `on_behalf_of` = `created_by` do link.
- **Cross-Domain — Audit**: consome todos os domain events (§15.2).
- **Billing**: tier da Company afeta capacidade de operação (§6).

### 16.3. Infraestrutura

- **Postgres**: tabela `empresa_sindica_links` + enum `empresa_sindica_link_status` + 3 índices (migration 00307).
- **Redis**: cache ABAC (`ms:auth:{userID}`) — purge no revoke (pendente).
- **NATS**: eventos futuros (§15.2).
- **Email provider (Resend ou adapter)**: envio de convite com token — **pendente**.

---

## 17. Escopo futuro M5+ (feature `empresa-síndica` como tenant próprio)

Do briefing: **"M5+ (backlog): administradora como tenant multi-condomínio com dashboard consolidado (feature empresa-síndica)"**.

### 17.1. Diferença conceitual

| Dimensão | MVP (atual) | M5+ (backlog) |
|---|---|---|
| Modelagem | Empresa comum + `EmpresaSindicaLink` por condomínio | **Tenant próprio** `empresa_sindica` com tipo `tenant_type='syndic_company'` |
| Painel | E1-E16 genérico (painel empresarial) | **Dashboard consolidado** próprio: visão cross-condomínio de validações pendentes, inadimplência, timeline agregada, compliance |
| Permissões | 5 toggles por link | Perfil "Administradora" com matriz de ações + escopo multi-condo |
| Faturamento | Plano Plus/Pro da empresa | Plano próprio "Administradora" (tiers específicos por nº de condomínios atendidos) |
| Assembleia | Papel `administradora_presente` (hoje nullable, sem UI dedicada) | Pipeline dedicado para administradora presidir/apoiar assembleia com funcionalidades específicas |
| Onboarding | Cadastro padrão de empresa | Onboarding próprio de administradora (documentos corporativos, CNAE específico, responsável técnico) |

### 17.2. Motivadores do evolutivo

- Content requirements seção CROSS-DOMAIN:3116-3133 já documenta a delegação profissional como peça separada.
- Mercado brasileiro: administradoras grandes gerenciam dezenas/centenas de condomínios — painel consolidado é exigência competitiva.
- Vault `System Architecture - Institutional.md`:44 reconhece `EmpresaSindica` como **entity de primeira classe** do CondominiumModule (não apenas vínculo transitório).

### 17.3. Decisões dependentes (pendentes)

- D-### (futuro): tenant dedicado ou super-usuário da Empresa com scopo multi-condo?
- D-### (futuro): plano `syndic_company_*` no Stripe ou extensão do `enterprise_*`?
- D-### (futuro): como reconciliar `EmpresaSindicaLink` legado (M1-M4) com tenant novo?
- ADR futura: topologia de banco (schema-per-tenant vs RLS).

---

## 18. Estado no código (Abril/2026)

### 18.1. Implementado

| Item | Arquivo | Status |
|---|---|---|
| Aggregate `EmpresaSindicaLink` | commercial/domain/entities/empresa_sindica_link.go | completo (factory + reconstruction + state machine + 5 permissões) |
| Enum `EmpresaSindicaLinkStatus` | commercial/domain/enums/enums.go:270-289 | completo |
| Repo interface | commercial/domain/repositories/i_empresa_sindica_repository.go | completo (5 métodos) |
| Repo impl | commercial/infrastructure/database/empresa_sindica_repository_impl.go | presente |
| Use cases (5) | commercial/application/use_cases/empresa_sindica_use_cases.go | Invite/Accept/UpdatePermissions/Revoke/List |
| HTTP handler (5 rotas) | commercial/infrastructure/http/routes/empresa_sindica_handler.go | completo |
| Migration | migrations/00307_create_empresa_sindica.sql | completo |
| Normalização CNPJ | migrations/00310_normalize_empresa_sindica_cnpj.sql | completo |
| Wiring no módulo | commercial/module.go:23, 53, 112-119 | completo |
| Testes | commercial/domain/entities/empresa_sindica_link_test.go + use_cases test | presentes (coverage não auditado aqui) |
| Models sqlc | institutional/infrastructure/database/generated/models.go (duplicado) | location estranha — DT-### |

### 18.2. Não implementado / gaps

| Item | Gap | Severidade |
|---|---|---|
| Envio de email com token | sem adapter de email no use case Invite | Alta (sem isso, fluxo S5 não fecha ponta-a-ponta em prod) |
| Validação token ↔ email ↔ company_id no Accept | atacante com token pode aceitar com qualquer `company_id` | **Alta — pentester** |
| Domain events | nenhum evento emitido (`EmpresaSindicaLinked`, etc.) | Alta (Req 93 AC4 exige) |
| Audit trail formal (tabela + middleware) | só há `logger.Info` | Alta (Req 93 AC7, AC10) |
| UNIQUE constraint `(condominium_id) WHERE status='active'` | sem proteção em SQL | Média — race condition (pentester) |
| Purge instantâneo de cache ABAC no Revoke | D-001 ainda em aberto; staleness 5min hoje | Média (Req 93 AC9 exige "immediately block login") |
| Reason obrigatório em Revoke | sem campo `revoke_reason` no aggregate | Baixa (recomendável para audit) |
| UI S5 completa | front ainda não implementado (scope de Sprint 2+ presumivelmente) | Ambiental |

---

## 19. Gaps e perguntas abertas

### 19.1. Arquitetura / design

- **Q1**: `EmpresaSindicaLink` vive em `commercial`, mas **faz sentido estar em `institutional`** conceitualmente (é vínculo condomínio ↔ …). System Architecture Institutional:44-45 lista `EmpresaSindica` como entity de `CondominiumModule`. Revisar em ADR.
- **Q2**: Falta de telas dedicadas para Administradora. Hoje ela usa E1-E16 genérico. Briefing explicitamente pede: **"verificar se tem admin dashboard próprio"** — resposta: **NÃO** no MVP (só no M5+, §17). Registrar como gap ambiental.
- **Q3**: Quem audita o aceite? Se `company_id` é passado pela empresa, backend precisa validar que o usuário fazendo o POST pertence àquela company. ABAC `RequirePermission(accept, empresa_sindica_link)` + check `user.company_id == body.company_id`.
- **Q4**: Como lidar com **troca de mandato** (Req 17 institutional.md:401) enquanto há link ativo? O síndico novo herda o link? Precisa re-confirmar? **Não coberto hoje.**

### 19.2. Segurança (pentester)

- **P1**: Token enumeration — UUIDv7 tem timestamp em prefixo, não ideal para bearer. Avaliar cryptographically random token (32 bytes) para email-token.
- **P2**: Token reuse após revogação `pending → revoked` — migration 00307 não invalida token explicitamente; `FindByToken` não filtra por status. Um link `pending → revoked` ainda é encontrável por `FindByToken`, mas `Accept` bloqueia via `IsPending` guard — OK semanticamente, porém expõe link revogado via `FindByToken`. Sugerir `FindByToken` filtrar `status='pending'`.
- **P3**: Rate limiting no endpoint de convite — síndico malicioso pode flodar empresas (spam). Já há middleware RateLimiter (identity.md:68) — verificar se aplicado a `POST /empresa-sindica`.
- **P4**: LGPD — email e CNPJ da empresa convidada são PII. Confirmar que `logger.Info` não loga esses campos (verificar linhas 49-54 do use case: só loga `company_name` e `created_by` — OK, não loga email/cnpj).

### 19.3. Regras / produto

- **R1**: 1 administradora por condomínio — **invariante assumida pelo briefing** mas não expressa em SQL nem em guard de Invite (síndico pode convidar 2 empresas simultaneamente, ambas em `pending`; se ambas aceitam antes de o síndico revogar uma, vira race).
- **R2**: Administradora pode ter **múltiplos usuários** (admin + N operators) — todos ganham acesso delegado via link? Sim (ABAC grants por company_id). Validar em code review.
- **R3**: Quando a empresa administradora sai do status `active` (vira `suspended/blocked`), os links `active` **devem bloquear as ações delegadas** mesmo sem revogação explícita. Implementação: ABAC checa `company.status.IsOperational()` além do link status.

### 19.4. Documentação

- **D1**: Divergência de nome entre `validar_comunicados_empresas` (vault) e `validar_comunicados_tecnicos` (Content requirements:3129). Código canoniza como `validate_communications` / `ValidateCommunications`.
- **D2**: Não há `Domínio - Institucional.md` no vault (apenas Comercial/Assembleia/Identidade/Conteúdo/Cross-Domain) — seção institucional fica parcialmente descrita em System Architecture e nos requirements .kiro.
- **D3**: Terminologia mista "empresa síndica" × "administradora" × "empresa administradora" — recomendar glossário oficial no `01-domain/ubiquitous-language.md`.

---

## 20. Referências cruzadas

### 20.1. Sub-produtos relacionados (v2)

- [[01-governanca-institucional]] — Condomínio, Timeline, Plano Diretor (surface afetada pelas permissões 1-3).
- [[02-connect-me]] — Canal unidirecional; administradora não participa diretamente, mas empresas podem se comunicar via Connect Me E→E.
- [[03-reputacao]] — Reputação da Company administradora segue mesma matriz das demais empresas.
- [[05-assembleia-live]] — Administradora na mesa, valida procurações, homologa votação, emite lista de inadimplentes.
- [[09-compliance]] — Audit trail das ações delegadas + Req 93 AC7.
- [[10-marketing-agencias]] — Outro modelo de delegação (por empresa, não por condomínio). Comparar padrões.

### 20.2. Domínios e requisitos

- **Req 6.2** (institutional.md:16) — `administradoras_vinculadas` no perfil Síndico com `plan_tier >= plus` (termo legado "N2/N3" na fonte; canônico via ADR-0032).
- **Req 6.3** (institutional.md:55) — Perfil Company genérico.
- **Req 7** (institutional.md:117) — Registro de condomínio + mandato + empresa síndica.
- **Req 18** (commercial.md:16) — Registro de empresa.
- **Req 23** (commercial.md:241) — Registro de execução (alvo de `ValidateExecutions`).
- **Req 24** (commercial.md:290) — Comunicados pós-deal (alvo de `ValidateCommunications`).
- **Req 25** (commercial.md:316) — Adendo (mecanismo para `EditSyndicPosts`).
- **Req 93** (requirements.md:3118) — Empresa Síndica Registration and Linking (10 ACs — fonte de verdade funcional).
- **Req 94** (requirements.md:3135) — Context Confirmation (aplica a S5).

### 20.3. ADRs e decisões (v2)

- **ADR-0032** (`02-architecture/adr/0032-global-plans-stripe-style.md`) — tier universal aplica a empresas administradoras.
- **D-056** — Agnosticismo de provedor (email provider para envio de token).
- **D-134** (revoga D-058/D-072) — Admin = `apps/admin` dedicada no monorepo web (não confundir com Administradora Condominial).
- **D-066** — Personas carregam matriz de permissão.
- **D-001** (em aberto, .kiro/STATE.md) — Purge instantâneo de cache Redis via webhook Zitadel vs staleness 5min — **crítico para revogação instantânea de administradora** (Req 93 AC9).

### 20.4. Telas no inventário

- **S5** — Cadastro / Vínculo Empresa Síndica (foco deste sub-produto).
- **S4** — Dados da Gestão/Mandato (tela anterior no onboarding).
- **S6** — Home da Governança (tela posterior; pós-S5, síndico vê estado do vínculo em card).
- **E1** — Painel Empresarial (administradora loga aqui como empresa comum no MVP).
- **Assembly telas** (Content requirements:1313-1671) — participação da administradora em assembleia.

### 20.5. Código

- Aggregate: `backend/internal/modules/commercial/domain/entities/empresa_sindica_link.go`
- Use cases: `backend/internal/modules/commercial/application/use_cases/empresa_sindica_use_cases.go`
- HTTP: `backend/internal/modules/commercial/infrastructure/http/routes/empresa_sindica_handler.go`
- Repo impl: `backend/internal/modules/commercial/infrastructure/database/empresa_sindica_repository_impl.go`
- Migration: `backend/internal/shared/providers/database/migrations/00307_create_empresa_sindica.sql`
- Wiring: `backend/internal/modules/commercial/module.go:23, 53, 112-119`

---

## Checklist de destilação (Fase 8 — método)

- [x] Leitura arquivo-por-arquivo das fontes listadas em §1
- [x] Catálogo fiel com trechos citados (arquivo + linha)
- [x] Modelagem canônica (aggregate + VO + repo + state machine)
- [x] Enums mapeados 1:1 (Go ↔ Postgres ↔ JSON API)
- [x] Tela S5 detalhada (7 passos ponta-a-ponta)
- [x] Regras, invariantes, state machine, domain events previstos
- [x] Gaps / findings listados (arquitetura, segurança, regras, docs)
- [x] Dependências inter-sub-produto mapeadas
- [x] Estado no código (implementado × gap)
- [x] Escopo futuro M5+ documentado como backlog
- [ ] **Dual-check obrigatório** — agent B em contexto limpo revisa (pendente — PRÓXIMO PASSO)
- [ ] Findings migrados para `AUDIT.md` como A-### (pendente)
- [ ] DTs migradas para `AUDIT.md` como DT-### (pendente)
- [ ] Glossário ubíquo `administradora / empresa síndica / admin` no `01-domain/ubiquitous-language.md` (pendente)
