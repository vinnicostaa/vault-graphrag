---
title: "Sub-produto 1 — Governança Institucional"
type: spec
tags: [sub-produto, institucional, governanca, master-sindico, v2, fase-8]
source: specs/institutional.md + specs/cross-domain.md + institutional/domain/** + migrations 00200-00210 + vault Content/02-Jornadas + vault Content/03-Modulos
created: 2026-04-23
updated: 2026-04-23
status: destilado
dual_check: pending
---

# Sub-produto 1 — Governança Institucional

Núcleo do MVP. Todo o resto do ecossistema master-sindico pluga aqui: Connect Me publica via validação → Timeline; Assembleia persiste ata como TimelineEntry; Compliance bloqueia encerramento de mandato; Billing controla criação de condomínio; Identity gera Memberships; Content hospeda vídeos que o síndico cola no perfil institucional.

---

## § 1. Fontes consultadas

| Arquivo | Linhas | Data | Papel |
|---|---|---|---|
| `backend/.kiro/specs/master-sindico/requirements/institutional.md` | 1-437 | 2026-04-22 | Requisitos 6.2–17 (canônicos) |
| `backend/.kiro/specs/master-sindico/requirements/cross-domain.md` | 1-501 | 2026-04-22 | Req 33-42 compliance, 46-47 mandato |
| `backend/.kiro/specs/master-sindico/requirements/functional-matrix.md` | 1-229 | 2026-04-22 | Matriz feature × persona × tier |
| `backend/internal/modules/institutional/domain/entities/condominium.go` | 1-123 | 2026-04-21 | Agregado raiz |
| `backend/internal/modules/institutional/domain/entities/unit.go` | 1-84 | 2026-04-21 | **GAP: sem unit_type / fracao_ideal** |
| `backend/internal/modules/institutional/domain/entities/membership.go` | 1-139 | 2026-04-21 | Vínculo titular/dependente |
| `backend/internal/modules/institutional/domain/entities/syndic_mandate.go` | 1-122 | 2026-04-21 | Mandato |
| `backend/internal/modules/institutional/domain/entities/timeline_entry.go` | 1-138 | 2026-04-21 | INSERT-only + adendo |
| `backend/internal/modules/institutional/domain/entities/master_plan_activity.go` | 1-213 | 2026-04-21 | Plano Diretor reativo |
| `backend/internal/modules/institutional/domain/entities/announcement.go` | 1-122 | 2026-04-21 | Comunicados 8 tipos × 4 prioridades |
| `backend/internal/modules/institutional/domain/entities/event.go` | 1-106 | 2026-04-21 | Eventos 13 tipos |
| `backend/internal/modules/institutional/domain/entities/poll.go` | 1-215 | 2026-04-21 | Consulta 3 tipos |
| `backend/internal/modules/institutional/domain/entities/validation_item.go` | 1-162 | 2026-04-21 | Hub de validações |
| `backend/internal/modules/institutional/domain/entities/compliance.go` | 1-201 | 2026-04-21 | Termo + declaração + audit log |
| `backend/internal/modules/institutional/domain/entities/management_assessment_response.go` | 1-95 | 2026-04-21 | Avaliação bimestral 1-10 |
| `backend/internal/modules/institutional/domain/enums/enums.go` | 1-327 | 2026-04-21 | Enums canônicos |
| `backend/internal/modules/institutional/domain/enums/master_plan.go` | 1-164 | 2026-04-21 | Catálogos 26/26/9/8 |
| `backend/internal/modules/institutional/domain/events/events.go` | 1-48 | 2026-04-21 | Domain events publicados |
| `backend/internal/modules/institutional/application/use_cases/` | 36 arquivos | 2026-04-21 | Casos de uso (F-###) |
| `backend/internal/shared/providers/database/migrations/00200-00210*.sql` | 11 arquivos | 2026-04-21 | DDL canônico |
| `Obsidian Vault/.../System Architecture - Institutional.md` | 1-207 | 2026-03 | Visão de arquitetura (6 módulos) |
| `Development/Content/03-Modulos/Compliance-Detalhado.md` | 1-284 | 2026-04-22 | Telas C1-C11 + gatilhos |
| `Development/Content/02-Jornadas/Inventario-Telas.md` | 1-371 | 2026-04-22 | M1-M15 + S1-S31 |

---

## § 2. Propósito

> "Domínio de gestão condominial: estrutura de condomínios, unidades residenciais, timeline imutável, plano diretor e mandatos de síndicos."
> — `institutional.md:12`

> "O domínio Institutional é o coração da plataforma — a governança condominial. Controla a Timeline imutável, o Plano Diretor reativo, validações centralizadas, compliance, e avaliações. É o backbone que conecta as ações comerciais ao registro institucional."
> — `System Architecture - Institutional.md:25`

Governança Institucional é **onde tudo vira histórico**. Cada ação relevante do mandato — reunião, contratação, reforma, votação, decisão — vira um `TimelineEntry` imutável. O Plano Diretor se atualiza **exclusivamente por reação** a entradas da Timeline (Req 12 AC8). Compliance bloqueia transições críticas (encerrar mandato) quando houver pendências. O síndico é o único ator que publica; moradores leem, respondem (enquetes, eventos, avaliação) e dão ciência.

Tier universal Stripe-style (ADR-0032):
- `trial` — 15 dias síndico; acesso completo.
- `base` — síndico eleito/profissional com 1 condomínio ativo; cria atividade Timeline e comunicados limitados.
- `plus` — até 15 condomínios; dashboards + export + perfil público.
- `pro` — 15 condomínios (hard limit), recursos completos.

Admin MS tem privilégio transversal (audit trail completo, broadcasts, aprovações manuais).

---

## § 3. O que NÃO é

Guard-rails que impedem escopo vazar para outros sub-produtos:

- **Não é Connect Me.** Comunicação empresa↔síndico é Sub-produto 2. Governança só consome o evento `commercial.deal.confirmed` → publica TimelineEntry tipo `registro_de_execucao`.
- **Não é Reputação.** Markers autodeclarados (15 governance + 25 compliance) aparecem no Perfil do Síndico (Req 6.2) mas o scoring calculado vira Sub-produto 3.
- **Não é Academia / LMS.** Vídeos institucionais do síndico (trava trimestral Req 29) vivem no Sub-produto 4 (Conteúdo) — Governança só linka o video_id.
- **Não é Assembleia ao vivo.** Votação síncrona, conferência, painel presidente → Sub-produto 5. Governança recebe apenas o resultado (`assembly_result` TimelineEntry).
- **Não é gestão financeira.** Não há ledger, boleto, pagamento. Billing é Stripe-only e externo.
- **Não é Banco de Talentos.** Vídeo-currículo do morador → Sub-produto 7.
- **Não é Vizinhança.** Parceiros locais + benefícios → Sub-produto 8.
- **Não é moderação de fórum.** Fórum LMS → Sub-produto 6.
- **Não é emissão de contratos.** Compliance contratual C8 é apenas **listagem + alertas** de deals fechados em Connect Me, não gera contrato.

---

## § 4. Personas envolvidas × tiers

Conforme `functional-matrix.md:55-98` + `Inventario-Telas.md:88-107`.

### Síndico

| Tier | Condomínios ativos | Cria atividade Timeline | Cria comunicado | Export PDF/CSV | Perfil público |
|---|---|---|---|---|---|
| `trial` | 1 (⏳ soft-block) | ⏳ | ⏳ | ⏳ | ❌ |
| `base` | 1 | ❌ | ❌ | ❌ | Limitado |
| `plus` | até 15 | ✅ | ✅ | ✅ | ✅ |
| `pro` | até 15 | ✅ | ✅ | ✅ | ✅ |

Hard limit universal: **15 condomínios por síndico** — `create_condominium_use_case.go:23` `MaxCondominiumsPerSyndic = 15`.

### Morador (titular / dependente)

| Tier | Ler Timeline | Ler comunicados | Responder enquete | Ciência evento | Avaliar gestão | Votar assembleia |
|---|---|---|---|---|---|---|
| `base` (free) | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| `pagante` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

Dependente: leitura apenas; sem voto em Assembleia (Req 9 + Req 16). Revogação do titular **cascateia** bloqueio dos dependentes (Req 17 — motivos: moved_out/sold_unit/contract_ended/removed_by_syndic).

### Subsíndico / Conselho (sub-roles ABAC)

`RoleType` em `enums.go:70-85`: `principal`, `auxiliary`, `assistant`. Permissões granulares via toggle configurável (`Inventario-Telas.md:103-107`): criar PD, registrar atividade LT, publicar, criar/encerrar evento, criar/encerrar votação, validar proposta, gerar link externo, marcar/confirmar negócio, criar adendo, exportar relatório, assinar compliance.

Invariante Postgres: `uq_syndic_mandates_active_principal` — 1 principal ativo por condomínio (migr. 00201:72-74).

### Administradora (via EmpresaSindicaLink)

Persona separada (sub-produto 11). Em Governança aparece como ator delegado:
> "Empresa síndica vinculada pode: visualizar publicações, editar publicações do síndico, ocultar publicações, validar registros de empresas, validar comunicados de empresas" — `Inventario-Telas.md:89`

Token por email com permissões configuráveis + audit log (Req 7).

### Admin MS

Privilégio transversal: gestão global de tenants, moderação, broadcasts, aprovação manual de CNPJs, controle de inadimplência, **audit trail completo** (`functional-matrix.md:132-142`). Não é tier de plano — é role privilegiada (D-058).

---

## § 5. Matriz funcional resumida (sem slugs)

Recorte institucional da `functional-matrix.md`:

| Feature | Ação | Morador base | Morador pagante | Síndico trial | Síndico base | Síndico plus | Síndico pro | Admin MS |
|---|---|---|---|---|---|---|---|---|
| Timeline | Ler | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Timeline | Criar atividade | ❌ | ❌ | ⏳ | ❌ | ✅ | ✅ | ✅ |
| Timeline | Criar adendo | ❌ | ❌ | ⏳ | ❌ | ✅ | ✅ | ✅ |
| Plano Diretor | Ver | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Plano Diretor | Criar atividade | ❌ | ❌ | ⏳ | ❌ | ✅ | ✅ | ✅ |
| Comunicados | Ler | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Comunicados | Criar | ❌ | ❌ | ⏳ | ❌ | ✅ | ✅ | ✅ |
| Eventos | Ciência / RSVP | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Eventos | Criar | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Enquete | Responder | ✅ | ✅ | n/a | n/a | n/a | n/a | n/a |
| Enquete | Criar | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Avaliação gestão | Responder | ✅ | ✅ | n/a | n/a | n/a | n/a | n/a |
| Avaliação gestão | Ver agregado | n/a | n/a | ✅ | ✅ | ✅ | ✅ | ✅ |
| Validações (hub) | Ver pendentes | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Validações | Decidir (validate/reject/adjust) | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Compliance | Declarar anual | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Compliance | Ver auditoria | ❌ | ❌ | ✅ principal | ✅ principal | ✅ principal | ✅ principal | ✅ total |
| Condomínio | Criar | ❌ | ❌ | ⏳ | base=1 | plus=até 15 | pro=até 15 | ✅ |
| Condomínio | Limite universal | — | — | 15 | 15 | 15 | 15 | — |
| Mandato | Encerrar | ❌ | ❌ | ⏳ | ✅ se compliance em dia | ✅ | ✅ | ✅ |
| Mandato | Transferir | ❌ | ❌ | ⏳ | ✅ | ✅ | ✅ | ✅ |
| Membership | Criar titular (via convite) | n/a | n/a | ✅ | ✅ | ✅ | ✅ | ✅ |
| Membership | Adicionar dependente (titular) | ✅ (titular próprio) | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Membership | Encerrar vínculo | ✅ (próprio) | ✅ | ✅ (moradores do condo) | ✅ | ✅ | ✅ | ✅ |
| Relatório | Exportar PDF/CSV | ❌ | ❌ | ⏳ | ❌ | ✅ | ✅ | ✅ |

Legenda: ⏳ trial (soft-block), ❌ 403 `PLAN_REQUIRED`, ✅ permitido, n/a não aplicável.

---

## § 6. Aggregates (código real)

### 6.1 Condominium — `condominium.go:29-42`

Agregado raiz. 9 campos.

```go
type Condominium struct {
    id             string
    publicID       string
    name           string
    condoType      enums.CondominiumType
    address        value_objects.Address
    totalUnits     int
    blocksOrTowers int
    createdBy      string
    createdAt, updatedAt time.Time
    deletedAt      *time.Time
}
```

Invariantes de construção (`NewCondominium` linhas 46-81):
- `publicID` obrigatoriamente 8 chars `[A-Z0-9]` (regex `publicIDPattern`).
- `name` trim 2–200 chars.
- `condoType` IsValid sobre os 6 valores.
- `totalUnits` 1–10000.
- `blocksOrTowers` 0–500.
- `createdBy` é o user_id do síndico autenticado.

Mandates **não** pertencem ao agregado: "vivem em entidade própria (SyndicMandate) referenciando este ID — evita agregado inflado + permite tx isoladas quando síndico renova mandato" (linhas 29-30).

**6 tipos** (`CondominiumType`, enum PG em migr. 00200 + `enums.go:11-18`):
1. `residencial`
2. `comercial`
3. `misto`
4. `shopping_center`
5. `galeria_comercial`
6. `edificio_garagem`

Address (VO em `value_objects/`, campos desnormalizados em `condominiums` table — migr. 00201:21-27): `cep` regex `^[0-9]{8}$`, `street` 2-200, `number` 1-20, `complement` opcional, `neighborhood` 2-100, `city` 2-100, `state` exato 2 chars.

publicID é o "identificador humano" — aparece no QR e é como morador busca o condomínio (M3). Gerado pela aplicação via interface `PublicIDGenerator` com rand criptográfico.

### 6.2 Unit — `unit.go:17-29` — **GAPS CRÍTICOS M1**

```go
type Unit struct {
    id            string
    condominiumID string
    unitNumber    string    // "101", "Apto 501"
    block         string    // "" = sem bloco
    floor         *int
    active        bool
    createdAt, updatedAt time.Time
}
```

**GAP 1 — `unit_type` AUSENTE.** Spec Req 9 (`institutional.md:192`) prevê enum `apartamento | casa | sala_comercial | loja | garagem | outro`. Nem a entidade nem a migration 00202 declaram essa coluna. Bloqueia condomínios `shopping_center` / `galeria_comercial` / `edificio_garagem` na prática (não dá para diferenciar "loja" de "apartamento" no front).

**GAP 2 — `fracao_ideal` AUSENTE.** Spec Req 9 (`institutional.md:194`) + Req 57 (Assembly) exigem `NUMERIC(5,4)` para ponderar voto. Sem isso, votação por fração ideal na Assembleia **não funciona**: degrada para voto por cabeça. Marca M1 em risco.

Ambos documentados em `§ 15. Gaps`.

Integridade única da Unit (migr. 00202:42):
```sql
UNIQUE (condominium_id, unit_number, block)
```

### 6.3 Membership — `membership.go:24-39`

Vínculo morador↔unit com 3 status e 4 motivos de encerramento.

```go
type Membership struct {
    id       string
    unitID   string
    userID   string
    roleType enums.RoleType            // titular | dependent
    status   enums.MembershipStatus    // pending_confirmation | active | ended
    consentSignedAt *time.Time
    endedAt         *time.Time
    endReason       enums.MembershipEndReason
    endNote         string              // texto livre
    createdAt, updatedAt time.Time
}
```

`const MaxDependentsPerUnit = 2` (linha 11). Hard-coded no domínio.

Status (`enums.go:104-118`):
1. `pending_confirmation` — criado, falta assinar termo. Já permite **leitura** (`IsActive() → true`).
2. `active` — assinatura feita via `ConfirmConsent()`.
3. `ended` — encerrado (histórico preservado; nunca DELETE).

End reasons (`enums.go:129-135` + migration 00202:17-22):
- `moved_out`
- `sold_unit`
- `contract_ended`
- `removed_by_syndic`

Invariantes PG (migr. 00202:74-76):
```sql
CREATE UNIQUE INDEX uq_memberships_active_titular
    ON memberships(unit_id)
    WHERE status IN ('pending_confirmation', 'active') AND role_type = 'titular';
```
→ **1 titular ativo por unit** (INV-009). Dependentes não ganham esse lock — o limite de 2 é aplicado no domínio/use case (`MaxDependentsPerUnit`).

### 6.4 SyndicMandate — `syndic_mandate.go:18-29`

```go
type SyndicMandate struct {
    id            string
    condominiumID string
    userID        string
    mandateType   enums.MandateType   // eleito | profissional | interino
    status        enums.MandateStatus // scheduled | active | ended
    roleType      enums.RoleType      // principal | auxiliary | assistant
    startDate     time.Time
    endDate       *time.Time          // nil = indeterminado
    createdAt, updatedAt time.Time
}
```

3 `MandateType` (`enums.go:38-42`):
- `sindico_eleito`
- `sindico_profissional`
- `sindico_interino`

3 `MandateStatus` (`enums.go:55-59`): `scheduled`, `active`, `ended`. Status auto-calculado no construtor: se `startDate > now` → `scheduled`, senão `active` (linhas 50-54).

Regra temporal: `endDate` opcional; se presente deve ser `>= startDate` (linha 46).

Invariante PG (migr. 00201:72-74):
```sql
CREATE UNIQUE INDEX uq_syndic_mandates_active_principal
    ON syndic_mandates(condominium_id)
    WHERE status = 'active' AND role_type = 'principal';
```
→ **1 principal ativo por condomínio** (INV-008).

Auxiliares + assistentes podem ter múltiplos ativos. Limite final de 3 usuários síndicos por condomínio (principal + 2 auxiliares) é reforçado no use case (Req 10).

### 6.5 TimelineEntry — `timeline_entry.go:25-36`

Coração imutável da governança.

```go
type TimelineEntry struct {
    id             string
    condominiumID  string
    authorID       string
    entryType      enums.TimelineEntryType
    title          string
    body           string
    bodyJSON       []byte         // payload específico do tipo (JSONB)
    parentEntryID  string         // "" = entry original; != "" = adendo
    publishedAt    *time.Time     // nil = draft
    createdAt      time.Time
}
```

**INSERT-only (INV-022).** Regra de produto canônica:
> "Append-only, imutável após publicação — nunca edit, nunca delete (Rule 2)" — `institutional.md:259`

> "SEM deleted_at (Req 11 + Req 2.5): entradas não podem ser apagadas." — `migrations/00203_create_timeline.sql:16`

No código: a entidade não oferece setter de mutação; só `Publish()` que erra se já publicada (`ErrTimelineAlreadyPublished`, linha 19).

**7 TimelineEntryType** (`enums.go:150-158` + enum PG migr. 00203):
1. `atividade_da_gestao` — gestão planeja/executa algo.
2. `registro_de_execucao` — empresa registra serviço realizado.
3. `comunicado` — aviso síndico→moradores.
4. `evento` — assembleia, reunião, festa, manutenção programada.
5. `adendo` — **único mecanismo de retratação/modificação**.
6. `resultado_consulta_moradores` — fechamento de Poll.
7. `assembly_result` — resultado homologado de Assembleia.

**Invariante adendo ↔ parent** (migration 00203:39-42):
```sql
CHECK (
    (entry_type = 'adendo' AND parent_entry_id IS NOT NULL)
    OR (entry_type <> 'adendo' AND parent_entry_id IS NULL)
)
```
→ Reforçada no domínio: `ErrAdendoRequiresParent` + `ErrNonAdendoHasParent` (linhas 17-18).

Visibilidade: não há campo `visibility` explícito na entidade/migração — segmentação `gestao` vs `publico_moradores` deriva do `entry_type` + contexto ABAC. **Gap documental**: spec pede segmentação explícita; código não possui campo dedicado (G-GI-03).

### 6.6 MasterPlanActivity — `master_plan_activity.go:26-43`

Plano Diretor reativo: status é consequência de evento Timeline, nunca UPDATE manual (Req 12 AC8).

```go
type MasterPlanActivity struct {
    id                     string
    condominiumID          string
    areaCode               string    // catálogo 26
    activityTypeCode       string    // catálogo 26
    riskCode               string    // catálogo 9
    nextActionCode         string    // catálogo 8
    title                  string
    description            string
    status                 enums.MasterPlanStatus
    deadline               *time.Time
    lastTriggeringEntryID  string    // qual Timeline entry mudou o status
    createdBy              string
    metadata               []byte    // JSONB livre
    createdAt, updatedAt   time.Time
    deletedAt              *time.Time
}
```

**26 áreas** (`enums/master_plan.go:49-76` — lista exaustiva):
`fachada · telhado · playground · piscina · areas_comuns · elevadores · hidraulica · eletrica · seguranca · iluminacao · jardinagem · limpeza · portaria · sistemas_incendio · acessibilidade · estacionamento · academia · salao_festas · quadras · pets · sauna_spa · coworking · espaco_gourmet · brinquedoteca · lavanderia · administracao`

**26 tipos de atividade** (`master_plan.go:79-106`):
`manutencao_preventiva · manutencao_corretiva · inspecao_tecnica · laudo_tecnico · obra · reforma · substituicao_equipamento · aquisicao · contratacao_servico · renovacao_contrato · certificacao · treinamento_equipe · auditoria · limpeza_profunda · dedetizacao · descupinizacao · impermeabilizacao · pintura · troca_luminarias · modernizacao_sistema · instalacao_equipamento · ajuste_contrato · campanha_conscientizacao · consulta_moradores · assembleia_extraordinaria · documentacao_legal`

**9 riscos** (`master_plan.go:109-119`):
`saude_seguranca · patrimonial · financeiro · juridico · reputacional · operacional · ambiental · regulatorio · conforto`

**8 próximas ações** (`master_plan.go:122-131`):
`solicitar_orcamento · agendar_inspecao · contratar_fornecedor · iniciar_obra · comunicar_moradores · submeter_assembleia · aguardar_condicao_climatica · obter_licenca`

> Nota: spec `institutional.md:274` fala "8 próximas ações" mas em seções resumidas aparece "7". Código: 8 no slice `MasterPlanNextActions` — canônico.

**7 status** (`enums/master_plan.go:7-15`):
1. `planejada` — criada, aguardando ação.
2. `em_contratacao` — entrou em fluxo de orçamento/negociação.
3. `em_execucao` — obra/serviço em andamento.
4. `concluida` — **terminal**.
5. `suspensa` — **terminal** até ação explícita.
6. `reprogramada` — **terminal** até nova decisão.
7. `atrasada` — calculado por cron noturno (deadline passou & ainda em progress).

Transições automáticas (`mapEntryTypeToStatus` linhas 200-213):
- TimelineEntry `registro_de_execucao` → se status atual ∈ {planejada, em_contratacao} → `em_execucao`.
- TimelineEntry `atividade_da_gestao` → se `planejada` → `em_contratacao`.
- Terminal bloqueia advance: `IsTerminal() → {concluida, suspensa, reprogramada}`.

`MarkConcluida(triggeringEntryID)` → só se não terminal.
`MarkAtrasada()` → só se `IsInProgress() ∈ {planejada, em_contratacao, em_execucao}`; idempotente para simplificar cron.

### 6.7 Announcement — `announcement.go:21-34`

```go
type Announcement struct {
    id               string
    condominiumID    string
    authorID         string
    announcementType enums.AnnouncementType
    priority         enums.AnnouncementPriority
    title            string
    body             string
    publishedAt      *time.Time   // nil = draft
    expiresAt        *time.Time
    createdAt, updatedAt time.Time
    deletedAt        *time.Time
}
```

**Imutável pós-publish (INV-024).** `Publish()` falha se `publishedAt != nil` (linhas 96-104).

**8 AnnouncementType** (`enums.go:226-235`):
1. `urgencia`
2. `manutencao`
3. `evento`
4. `financeiro`
5. `regimento_interno`
6. `regra_convivencia`
7. `decisao_assembleia`
8. `informativo_geral`

**4 AnnouncementPriority** (`enums.go:252-257`):
`baixa · media · alta · critica`

> "'critica' força push notification + highlight persistente no feed" — migr. 00206:21

Tracking de leitura via `announcement_reads` (migr. 00206:52-59) — UNIQUE(announcement_id, user_id), idempotente.

Constraint temporal curiosa (migr. 00206:42):
```sql
CHECK (expires_at IS NULL OR expires_at >= published_at OR published_at IS NULL)
```
→ permite setar `expires_at` em draft antes de publicar.

### 6.8 Event — `event.go:19-32`

```go
type Event struct {
    id            string
    condominiumID string
    authorID      string
    eventType     enums.EventType
    title         string
    description   string
    location      string
    startsAt      time.Time
    endsAt        *time.Time
    createdAt, updatedAt time.Time
    deletedAt     *time.Time
}
```

**13 EventType** (`enums.go:183-196`):
1. `assembleia_ordinaria`
2. `assembleia_extraordinaria`
3. `reuniao_conselho`
4. `reuniao_moradores`
5. `festa`
6. `confraternizacao`
7. `atividade_esportiva`
8. `manutencao_programada`
9. `dedetizacao`
10. `limpeza_caixa_agua`
11. `inspecao_bombeiros`
12. `aniversario`
13. `evento_especial`

**Regra temporal**: `endsAt` opcional; se presente `>= startsAt` (`ErrInvalidEventPeriod` linha 15).

**EventResponse** (migr. 00205:57-67 + `event_response.go`):
- UNIQUE(event_id, user_id) — 1 resposta por morador por evento.
- `EventResponseType` ∈ {`ciente`, `confirmado`} (`enums.go:214-221`).
- UPDATE permitido (morador pode migrar de `ciente` → `confirmado`).

Spec (`institutional.md:307-310`): notificações automáticas 3d/1d/3h antes, tracking de ciência 6 meses. **Não-implementado** no código atual (G-GI-04).

### 6.9 Poll — `poll.go:23-37` (Consulta Moradores / pré-assembleia)

```go
type Poll struct {
    id              string
    condominiumID   string
    authorID        string
    pollType        enums.PollType
    status          enums.PollStatus   // open | closed
    question        string
    description     string
    closesAt        *time.Time
    closedAt        *time.Time
    timelineEntryID string             // populado ao Close — linka resultado na Timeline
    createdAt, updatedAt time.Time
    deletedAt       *time.Time
}
```

**3 PollType** (`enums.go:271-275`):
1. `yes_no_dk` — "Sim / Não / Não sei" (opções **fixas** geradas pelo sistema via `FixedOptionsFor`).
2. `multiple_choice` — 2-10 opções custom (`ErrInvalidOptionCount`).
3. `scale_1_to_5` — escala Likert 1-5 (opções fixas).

**Recorte MVP M1**: segundo instrução de produto, apenas `yes_no_dk` vale como consulta pré-assembleia. Spec permite os 3.

`HasFixedOptions()` → true para `yes_no_dk` e `scale_1_to_5` (linha 287).

Entidades relacionadas:
- `PollOption` — UNIQUE(poll_id, label), sort_order.
- `PollResponse` — UNIQUE(poll_id, user_id), UPDATE permitido enquanto `status=open`.

Auto-fechamento: `IsOpen()` retorna false se `closesAt < now` mesmo antes do cron transicionar (soft-close, linhas 114-122).

`Close(timelineEntryID)` publica resultado na Timeline (`resultado_consulta_moradores`) e persiste o link. Idempotência falha: segunda chamada → `ErrPollAlreadyClosed`.

### 6.10 ValidationItem — `validation_item.go:20-35`

Hub genérico de validações (Req 3.3). Sprint 4+ Commercial popula via eventos `commercial.execution.submitted`, `commercial.proposal.sent`.

```go
type ValidationItem struct {
    id              string
    condominiumID   string
    targetType      string   // "execution_record" | "proposal" | "technical_activity" | "company_communication" | ...
    targetID        string
    title           string
    payload         []byte   // JSONB snapshot
    submittedBy     string
    status          enums.ValidationStatus
    decidedBy       string
    decidedAt       *time.Time
    decisionReason  string
    timelineEntryID string   // populado em Validate (link → Timeline)
    createdAt, updatedAt time.Time
}
```

**4 ValidationStatus** (`enums.go:306-311`):
1. `pending`
2. `validated` — **terminal**
3. `rejected` — **terminal**
4. `adjustment_requested` — não-terminal; submitter pode ajustar e resubmeter (volta para `pending`).

Métodos (linhas 112-161):
- `Validate(decidedBy, reason, timelineEntryID)` — só se não terminal.
- `Reject(decidedBy, reason)` — idem.
- `RequestAdjustment(decidedBy, reason)` — idem.

`IsPending()` → `pending OR adjustment_requested` (linha 105).

### 6.11 Compliance — `compliance.go`

3 entidades coabitam neste arquivo + a avaliação de gestão em arquivo próprio:

**ResponsibilityTermSignature** (linhas 24-33):
- `termVersion` + `termHash` (SHA-256 hex 64 chars) + `signedIP` + `userAgent` + `signedAt`.
- Nova versão do termo → nova assinatura obrigatória (append-only).

**AnnualDeclaration** (linhas 90-98):
- `declarationYear` 2020–2100; `declarationText` 10–20000 chars; `signedIP`; `submittedAt`.
- UNIQUE(condominium_id, user_id, declaration_year) — 1/ano (migr. 00209:36).
- **Bloqueia encerramento de mandato se ano corrente não declarado** (Req 34 + `institutional.md:62`).

**AuditLogEntry** (linhas 146-156):
- `actorID` (nullable — eventos sistêmicos), `actorIP`, `action`, `targetType`, `targetID`, `condominiumID`, `payload` JSONB.
- Retenção 5 anos; `TRUNCATE` só por job de compliance autorizado. Nunca UPDATE/DELETE por aplicação.
- Índices: `idx_audit_actor`, `idx_audit_target`, `idx_audit_condominium`, `idx_audit_action` (migr. 00209:64-67).

**ManagementAssessmentResponse** — `management_assessment_response.go:20-31`:
- Bimestral (janela `[period_start, period_end)` derivada por domain service `CurrentBimonthlyPeriod`).
- 3 perguntas escala 1-10 em colunas `rating_q1/q2/q3`; comentário opcional ≤ 2000 chars.
- Anonimato para síndico: lê **agregados**; user_id só para dedup.
- UNIQUE(user_id, condominium_id, period_start) — 1 resposta por morador/período.
- Bloqueio: se avaliação atrasada (do lado do morador respondendo) → síndico não exporta relatório nem cria atividade (Req 15).

> **Compliance (link §9 sub-produto 9)** — este sub-produto contém as **entidades jurídicas** (termo, declaração, audit, avaliação). O sub-produto 9 define **telas + gatilhos + política** consolidada (C1-C11, incluindo Matriz de Conflito/C4, Risk Map/C7, Dossiê/C11 que hoje não têm entidade dedicada aqui).

---

## § 7. Enums canônicos

Contagem autoritativa (do código + migrations):

| Enum | Qty | Valores |
|---|---|---|
| `CondominiumType` | 6 | residencial, comercial, misto, shopping_center, galeria_comercial, edificio_garagem |
| `MandateType` | 3 | sindico_eleito, sindico_profissional, sindico_interino |
| `MandateStatus` | 3 | scheduled, active, ended |
| `RoleType` (síndico) | 3 | principal, auxiliary, assistant |
| `RoleType` (membership) | 2 | titular, dependent |
| `MembershipStatus` | 3 | pending_confirmation, active, ended |
| `MembershipEndReason` | 4 | moved_out, sold_unit, contract_ended, removed_by_syndic |
| `TimelineEntryType` | 7 | atividade_da_gestao, registro_de_execucao, comunicado, evento, adendo, resultado_consulta_moradores, assembly_result |
| `MasterPlanStatus` | 7 | planejada, em_contratacao, em_execucao, concluida, suspensa, reprogramada, atrasada |
| `MasterPlan ImpactedArea` (catálogo) | 26 | ver § 6.6 |
| `MasterPlan ActivityType` (catálogo) | 26 | ver § 6.6 |
| `MasterPlan RiskType` (catálogo) | 9 | ver § 6.6 |
| `MasterPlan NextAction` (catálogo) | 8 | ver § 6.6 |
| `EventType` | 13 | assembleia_ordinaria…evento_especial |
| `EventResponseType` | 2 | ciente, confirmado |
| `AnnouncementType` | 8 | urgencia, manutencao, evento, financeiro, regimento_interno, regra_convivencia, decisao_assembleia, informativo_geral |
| `AnnouncementPriority` | 4 | baixa, media, alta, critica |
| `PollType` | 3 | yes_no_dk, multiple_choice, scale_1_to_5 |
| `PollStatus` | 2 | open, closed |
| `ValidationStatus` | 4 | pending, validated, rejected, adjustment_requested |

---

## § 8. Telas — catálogo

Total institucional: **15 telas Morador (M1-M15) + 31 telas Síndico (S1-S31) + 11 telas Compliance (C1-C11)** = 57. Fonte: `Inventario-Telas.md` + `Compliance-Detalhado.md`.

### Família M (Morador) — 15 telas (recorte institucional)

| ID | Nome | Rota (proposta) | Persona | Ações principais | Estados |
|---|---|---|---|---|---|
| M1 | Painel do Morador | `/m` | titular/dependente | ler notificações, trocar condomínio, entrar em M5 | default |
| M2 | Selecionar Condomínio | `/m/condominiums` | multi-condo morador | listar; selecionar (persiste sessão) | vazio, 1, N |
| M3 | Buscar Condomínio por ID | `/m/condominiums/search` | morador novo | input 8-char publicID → preview → confirmar | busca, achou, not found |
| M4 | Cadastro da Unidade | `/m/units/new` | morador novo | escolher unit_number + bloco + upload foto + termo LGPD | preencher, enviar, aguardando confirmação do titular |
| M5 | Home do Condomínio | `/m/condominiums/{id}` | morador | cards: timeline, eventos, comunicados, enquetes, avaliação, vínculo | plan_tier varia |
| M6 | Linha do Tempo | `/m/condominiums/{id}/timeline` | morador | scroll infinito + filtros (tipo, data); ler-apenas | paginado; filtro aplicado |
| M7 | Detalhe da Publicação | `/m/timeline/{entryId}` | morador | ler title/body/bodyJSON; mostra adendos linkados | publicado, com_adendo, adendo |
| M8 | Plano Diretor | `/m/condominiums/{id}/master-plan` | morador | lista atividades com status + áreas + deadline | 7 status filtráveis |
| M9 | Detalhe da Ação PD | `/m/master-plan/{activityId}` | morador | ver área, tipo, risco, next_action, histórico Timeline | 7 status |
| M10 | Eventos | `/m/condominiums/{id}/events` | morador | lista futuros/passados; RSVP (ciente/confirmado) | 13 tipos; ciente/confirmado |
| M11 | Comunicados | `/m/condominiums/{id}/announcements` | morador | lista; badge "não lido"; abrir marca read_at | 8 tipos × 4 prioridades |
| M12 | Avaliar Gestão | `/m/condominiums/{id}/assessment` | morador | 3 perguntas 1-10 + comment opcional; 1x/período bimestral | aberta, respondida no período |
| M13 | Meu Vínculo | `/m/membership` | morador | ver unit, role_type, consent_signed_at, data início | pending_confirmation, active |
| M14 | Gerenciar Acessos (dependentes) | `/m/membership/dependents` | titular | adicionar até 2; revogar → bloqueia login imediatamente | 0-2 dependentes |
| M15 | Encerrar Vínculo | `/m/membership/end` | titular/dependente | escolher motivo (4) + nota; logout imediato | pending_end, ended |

**Regras estruturais Morador** — `Inventario-Telas.md:38-44`: morador NÃO publica; unidade obrigatória; dependentes leitura-apenas; unit vazia → dependentes removidos.

### Família S (Síndico) — 31 telas

| ID | Nome | Rota | Ações-chave | Estados |
|---|---|---|---|---|
| S1 | Entrada da Governança | `/s/onboarding` | wizard inicial pós-login | trial, assinou termo, não |
| S2 | Meus Condomínios | `/s/condominiums` | listar até 15; ordenar ativos primeiro | 0-15 |
| S3 | Cadastrar Condomínio (Endereço) | `/s/condominiums/new` | ViaCEP lookup; campos addr + tipo (6) | draft, cep_validado |
| S4 | Dados da Gestão (Mandato) | `/s/condominiums/new/mandate` | mandate_type, datas, role_type | scheduled/active |
| S5 | Cadastro/Vínculo Empresa Síndica | `/s/condominiums/new/empresa-sindica` | token email + permissions toggles + audit log | vinculado, pendente |
| S6 | Home da Governança | `/s/condominiums/{id}` | 10 cards: PD · LT · Eventos · Connect Me · Registros exec · Comunicados · Validações Pendentes · Dashboard · Compliance · Modo Assembleia | — |
| S7 | Plano Diretor | `/s/condominiums/{id}/master-plan` | visualizar com filtros por área/status/risco | 7 status × 26 áreas |
| S8 | Cadastrar Ação PD | `/s/master-plan/new` | form area+type+risk+next_action+title+desc+deadline | draft, criada (status=planejada) |
| S9 | Lista Ações PD | `/s/master-plan` | filtros + ordenar por deadline | idem S7 |
| S10 | Detalhe Ação PD | `/s/master-plan/{id}` | linked TimelineEntries + status history + markConcluida manual | 7 status |
| S11 | Linha do Tempo | `/s/condominiums/{id}/timeline` | scroll + filtros; ações: criar, criar adendo | published / draft |
| S12 | Criar Atividade | `/s/timeline/new` | form: type, title, body, bodyJSON | draft → publish |
| S13 | Editar Atividade (→ gera adendo) | `/s/timeline/{id}/adendo` | preencher justificativa + novo body | publicado + adendos vinculados |
| S14 | Histórico LT | `/s/timeline/{id}/history` | ver chain de adendos seguindo parent_entry_id | — |
| S15 | Eventos | `/s/condominiums/{id}/events` | calendário | futuros / passados |
| S16 | Criar Evento | `/s/events/new` | type (13), starts_at, ends_at, location, description | draft, published |
| S17 | Lista Eventos | `/s/events` | tabela + RSVP counts | — |
| S18 | Connect Me (hub) | `/s/connect-me` | (sub-produto 2) | — |
| S19 | Criar Connect Me | `/s/connect-me/new` | (sub-produto 2) | — |
| S20 | Connect Me Recebido (M→S) | `/s/connect-me/received` | (sub-produto 2) | — |
| S21 | Registros de Execução | `/s/validations/execution-records` | lista de ValidationItem target_type=execution_record | pending, validated, rejected, adjustment_requested |
| S22 | Detalhe Registro Execução | `/s/validations/{id}` | aprovar/rejeitar/pedir ajuste + comentário | idem |
| S23 | Comunicados | `/s/condominiums/{id}/announcements` | listar | draft, published |
| S24 | Criar Comunicado | `/s/announcements/new` | type (8) + priority (4) + title + body + expires_at | draft → publish (imutável após) |
| S25 | Lista Comunicados | `/s/announcements` | filtros | — |
| S26 | Validações Pendentes (hub) | `/s/validations` | lista cross-source agrupada por target_type | — |
| S27 | Dashboard | `/s/dashboard` | agregados: total moradores, score, taxa PD, satisfação, próximos marcos | — |
| S28 | Compliance (entrada) | `/s/compliance` | card navegador para C1-C11 | em_dia / parcial / pendente |
| S29 | Termo de Responsabilidade | `/s/compliance/term` | ler termo versionado + SHA-256 + assinar | assinado / pendente |
| S30 | Declaração Anual | `/s/compliance/annual-declaration` | formulário anual + checkbox + submeter | em dia / atrasada |
| S31 | Auditoria | `/s/compliance/audit` | lista audit_log_entries com filtros actor/date/action | — |

**Regras estruturais Síndico** — `Inventario-Telas.md:83-107`:
- LT é memória oficial; tudo passa por lá.
- PD não atualiza manualmente (só via publicação na LT).
- Empresa não publica direto — passa por validação do síndico.
- Edição preserva histórico (versão anterior + trilha).
- Até **15 condomínios** por síndico (Decision 11).
- Confirmação de contexto antes de publicar ("Publicar para Condomínio X?").
- Pós-fechamento é imutável (alteração só via Adendo Formal).
- Compliance não trava onboarding; trava encerramento de mandato.
- Compliance fica "em dia" só quando TODOS usuários assinaram.
- Todo compartilhamento registrado (LGPD/C9).

### Família C (Compliance Síndico) — 11 telas

Derivadas de `Compliance-Detalhado.md:24-230`. Alinhadas com entities: `ResponsibilityTermSignature`, `AnnualDeclaration`, `AuditLogEntry`, `ValidationItem`.

| ID | Nome | Mapeia |
|---|---|---|
| C1 | Painel de Compliance | Status geral Em dia / Parcial / Pendente + pendências |
| C2 | Declaração Anual | `AnnualDeclaration` (1x/ano, Req 34) |
| C3 | Assinaturas Obrigatórias | `ResponsibilityTermSignature` por user do condomínio (Req 35) — "só em dia quando TODOS assinaram" |
| C4 | Matriz de Conflito de Interesse | Req 36, sem entidade atual (G-GI-05) |
| C5 | Trilha de Auditoria Visível | `AuditLogEntry` + filtros user/date/action (Req 37) |
| C6 | Imutabilidade e Adendos | TimelineEntry type=`adendo` + parent_entry_id (ponteiro, não tela standalone) |
| C7 | Mapa de Risco Consolidado | Agregado sobre `master_plan_activities.risk_code` (Req 38) |
| C8 | Conformidade Contratual | Agregado sobre deals fechados Sub-produto 2 + alertas de garantia |
| C9 | Registro LGPD (Connect Me) | `lgpd_logs` (sub-produto 2) — leitura-apenas aqui |
| C10 | Score Interno de Governança | Sub-produto 3 (Reputação) — reflete agregado |
| C11 | Dossiê Exportável (PDF) | Req 40 — job gera PDF assinado; armazena MinIO (G-GI-06) |

### Tela crítica S6 — Home da Governança

`Inventario-Telas.md:332-338`: **única tela cross-módulo** do síndico. 10 cards + indicadores real-time:

1. Plano Diretor (atividades abertas + atrasadas)
2. Linha do Tempo (últimas publicações)
3. Eventos (próximos 7 dias)
4. Connect Me (oportunidades não lidas)
5. Registros de Execução (pending validations)
6. Comunicados (últimos publicados + non-read rate)
7. Validações Pendentes (total count cross-source)
8. Dashboard (atalho)
9. Compliance (status Em dia / Parcial / Pendente)
10. Modo Assembleia (atalho para live)

---

## § 9. Regras de negócio

- **BR-GI-001** — Síndico pode ter no máximo **15 condomínios ativos** — `create_condominium_use_case.go:23` (`MaxCondominiumsPerSyndic = 15`). Enforced pre-insert; retorna `ErrCondominiumLimitReached` → 409.
- **BR-GI-002** — Um condomínio tem no máximo **1 síndico principal ativo** — Postgres UNIQUE `uq_syndic_mandates_active_principal`.
- **BR-GI-003** — Um condomínio tem no máximo **3 usuários síndicos** (principal + 2 auxiliares + assistentes) — Req 10; validado em use case `InviteCondominiumUser` (Sprint futuro).
- **BR-GI-004** — Uma unit tem no máximo **1 titular ativo** — UNIQUE `uq_memberships_active_titular`.
- **BR-GI-005** — Uma unit tem no máximo **2 dependentes ativos** por titular — `MaxDependentsPerUnit = 2` (`membership.go:11`); valida no use case `AddDependentUseCase`.
- **BR-GI-006** — `end_date` de mandato (quando presente) deve ser `>= start_date` — CHECK migr. 00201:64 + `ErrMandateEndBeforeStart`.
- **BR-GI-007** — `ends_at` de evento (quando presente) deve ser `>= starts_at` — CHECK migr. 00205:47 + `ErrInvalidEventPeriod`.
- **BR-GI-008** — `period_end > period_start` para Avaliação de Gestão — migr. 00207:36 + `ErrInvalidPeriod`.
- **BR-GI-009** — 1 resposta de Avaliação por morador/período/condomínio — UNIQUE(user_id, condominium_id, period_start) migr. 00207:38.
- **BR-GI-010** — 1 declaração anual por ano por usuário por condomínio — UNIQUE migr. 00209:36.
- **BR-GI-011** — 1 resposta de Evento por (event_id, user_id) — UNIQUE migr. 00205:66; UPDATE permitido (pode mudar de `ciente` → `confirmado`).
- **BR-GI-012** — 1 resposta de Poll por (poll_id, user_id) — UNIQUE migr. 00208:65; UPDATE permitido enquanto `status=open`.
- **BR-GI-013** — 1 leitura de Comunicado por (announcement_id, user_id) — UNIQUE migr. 00206:58; idempotente.
- **BR-GI-014** — `publicID` de Condomínio é UNIQUE globalmente e sempre 8 chars `[A-Z0-9]`.
- **BR-GI-015** — Adendo DEVE linkar `parent_entry_id`; qualquer outro tipo NÃO pode ter parent — CHECK migr. 00203:39-42.
- **BR-GI-016** — `termHash` de assinatura de termo é SHA-256 hex (64 chars) — CHECK migr. 00209:13 + `isValidSHA256Hex`.
- **BR-GI-017** — Encerrar mandato exige compliance em dia — Req 34 + Req 46; caso contrário bloqueia com mensagem "Para prosseguir, finalize as pendências de compliance" (`Compliance-Detalhado.md:244`).
- **BR-GI-018** — Dependente sem poder de voto em Assembleia — Req 9 + Req 16.
- **BR-GI-019** — Revogação de vínculo titular → revoga automaticamente dependentes da mesma unit — Req 17 + R5 Morador (`Inventario-Telas.md:43`).
- **BR-GI-020** — Votação em pauta onde síndico declarou conflito é marcada como "abstém obrigatória" — Req 36 C4.
- **BR-GI-021** — Morador Base não vota em assembleia; paywall hard `PLAN_REQUIRED` — `functional-matrix.md:39`.
- **BR-GI-022** — Status de Plano Diretor NUNCA é setado por UPDATE manual — muda exclusivamente via `AdvanceStatusFromTimeline` ou `MarkAtrasada` cron — Req 12 Rule 2.
- **BR-GI-023** — Síndico cujos moradores não respondem avaliação bimestral tem certas ações bloqueadas (export, criar atividade) — Req 15.
- **BR-GI-024** — Validação `pending`/`adjustment_requested` pode ser alterada; `validated`/`rejected` são terminais.
- **BR-GI-025** — Mudança de condomínio no seletor (S2) invalida cache ABAC local — Req 8.
- **BR-GI-026** — Comunicado publicado é imutável — `Publish()` retorna erro se já publicado (`ErrAnnouncementAlreadyPublished`).
- **BR-GI-027** — Timeline entry publicada é imutável — mesmo princípio; retratação só via adendo.
- **BR-GI-028** — Audit log nunca UPDATE/DELETE por aplicação; TRUNCATE apenas via job de compliance autorizado (retenção 5 anos + archive pós-1 ano em MinIO).

---

## § 10. Invariantes

- **INV-001** — `Condominium.publicID` casa `^[A-Z0-9]{8}$` — construtor + CHECK PG.
- **INV-002** — `Condominium.totalUnits ∈ [1, 10000]`.
- **INV-003** — `Condominium.blocksOrTowers ∈ [0, 500]`.
- **INV-004** — `Condominium.name` trim length ∈ [2, 200].
- **INV-005** — `SyndicMandate.endDate`, se presente, `>= startDate`.
- **INV-006** — `SyndicMandate.roleType ∈ {principal, auxiliary, assistant}`.
- **INV-007** — `SyndicMandate.mandateType ∈ {sindico_eleito, sindico_profissional, sindico_interino}`.
- **INV-008** — 1 mandato principal ativo por condomínio (UNIQUE PG).
- **INV-009** — Membership `titular` é único ativo por unit (UNIQUE PG).
- **INV-010** — Até 2 dependentes por unit (enforced em use case, não PG).
- **INV-011** — `Membership.endReason ∈ {moved_out, sold_unit, contract_ended, removed_by_syndic}` quando encerrado.
- **INV-012** — `Membership.status` de "pending_confirmation" vira "active" via `ConfirmConsent`; nunca retorna para pending.
- **INV-013** — `Membership.status=ended` é terminal; linha nunca é DELETE.
- **INV-014** — TimelineEntry body length ≤ 20000 chars.
- **INV-015** — TimelineEntry title length ∈ [2, 200].
- **INV-016** — TimelineEntry `bodyJSON` é JSON válido (default `{}`).
- **INV-017** — Unit `unit_number` length ∈ [1, 20]; `block` ∈ [1, 20] se presente; `floor >= 0`.
- **INV-018** — Unit UNIQUE(condominium_id, unit_number, block).
- **INV-019** — Announcement title ∈ [2, 200]; body ∈ [1, 20000].
- **INV-020** — Announcement expires_at, se presente, `>= published_at` (ou draft sem published_at).
- **INV-021** — Event title ∈ [2, 200]; description ≤ 10000.
- **INV-022** — TimelineEntry é **append-only**: sem `deleted_at`, sem UPDATE após publish; adendo é único mecanismo de retratação.
- **INV-023** — Adendo tem parent_entry_id ≠ "" (obrigatório); não-adendo não tem parent (proibido) — CHECK PG.
- **INV-024** — Announcement publicado é imutável (`Publish()` falha na 2ª chamada).
- **INV-025** — Poll question ∈ [2, 500]; description ≤ 5000; multiple_choice tem 2-10 options; option label ∈ [1, 200].
- **INV-026** — Poll UPDATE de resposta só enquanto status=open.
- **INV-027** — ValidationItem status terminal (`validated|rejected`) não pode ser re-decidido.
- **INV-028** — MasterPlanActivity status terminal (`concluida|suspensa|reprogramada`) não recebe advance automático via Timeline.
- **INV-029** — MasterPlanActivity status **nunca** é UPDATE direto — só via `AdvanceStatusFromTimeline` ou `MarkAtrasada`.
- **INV-030** — AuditLogEntry é append-only; `action` length ∈ [2, 100]; `target_type` idem.
- **INV-031** — ResponsibilityTermSignature `termHash` é SHA-256 hex (64 chars lowercase).
- **INV-032** — AnnualDeclaration `declarationYear ∈ [2020, 2100]`; `declarationText` ∈ [10, 20000].
- **INV-033** — ManagementAssessmentResponse `ratingQ1/Q2/Q3 ∈ [1, 10]`; comment ≤ 2000.
- **INV-034** — `Membership.status=pending_confirmation` é tratado como "ativo para leitura" (`IsActive() → true`); só falta publicar coisas.
- **INV-035** — `CondominiumType.IsValid()` sobre os 6 valores exatos; qualquer string fora → rejeitada em handler/construtor.

---

## § 11. State machines

### MasterPlanActivity (7 estados)

```
        ┌───────────┐
        │ planejada │
        └─────┬─────┘
              │ Timeline{atividade_da_gestao}
              ▼
    ┌──────────────────┐
    │  em_contratacao  │
    └────┬───────┬─────┘
         │       │ Timeline{registro_de_execucao}
         │       ▼
         │  ┌──────────────┐
         │  │ em_execucao  │──────────────┐
         │  └──────┬───────┘              │
         │         │ MarkConcluida        │
         │         ▼                      │
         │  ┌──────────────┐              │
         │  │  concluida*  │              │  cron MarkAtrasada
         │  └──────────────┘              │  (se deadline < now)
         │                                ▼
         │                          ┌───────────┐
         └─────────────────────────▶│ atrasada  │
                                    └───────────┘

Terminais (bloqueiam advance): concluida, suspensa, reprogramada
```

Status `suspensa` e `reprogramada` são setados manualmente pelo síndico (endpoint dedicado — não implementado ainda).

### Announcement (draft → published terminal)

```
  (não existe)
     │ NewAnnouncement
     ▼
  draft (publishedAt = nil)
     │ Publish()
     ▼
  published (imutável) ◀─── sem transição de volta
     │
     │ expires_at atinge now
     ▼
  expired (lógica UI; linha PG continua published)
```

Soft-delete via `deleted_at` existe na tabela mas **não há método no domínio** — provavelmente usado por admin MS via SQL direto.

### Membership (pending → active → ended)

```
  (new)
    │ NewMembership
    ▼
  pending_confirmation ◀──┐
    │ ConfirmConsent      │ (não há retorno)
    ▼                     │
  active                  │
    │ End(reason, note)   │
    ▼                     │
  ended ────────(terminal; nunca DELETE)
```

### SyndicMandate (scheduled/active → ended)

```
  (new)
    │ NewSyndicMandate
    │   startDate > now → scheduled
    │   startDate <= now → active
    ▼
  scheduled ──(cron ou startDate atingida)──▶ active
                                                  │
                                                  │ End(closingDate)
                                                  ▼
                                                ended (terminal)
```

Idempotência: `End()` em mandato já ended → silencioso (linha 115-117).

### Poll (open → closed)

```
  (new)
    │ NewPoll (pollType, options)
    ▼
  open ◀─── respostas UPDATE permitido
    │ Close(timelineEntryID)  OU  soft-close (closesAt < now)
    ▼
  closed (timeline_entry_id populated com resultado)
```

### ValidationItem (pending ⇄ adjustment_requested, → validated/rejected terminais)

```
  (new) ─▶ pending
           │       │
           │       └─▶ adjustment_requested
           │                 │
           │   (submitter ajusta)
           │                 ▼
           │          volta para pending
           │
           ├─▶ validated (terminal, publica Timeline)
           └─▶ rejected  (terminal)
```

---

## § 12. Domain events emitidos/consumidos

Declarados em `domain/events/events.go` (subset implementado):

| Event | Origem | Payload principal |
|---|---|---|
| `institutional.timeline.published` | `TimelineEntry.Publish()` via use case `PublishTimelineEntryUseCase` | EntryID, CondominiumID, AuthorID, EntryType, ParentEntryID, PublishedAtTs |
| `institutional.condominium.created` | `CreateCondominiumUseCase` | CondominiumID, CreatedBy, PublicID |
| `institutional.membership.ended` | `EndMembershipUseCase` | MembershipID, UnitID, UserID, EndReason |

Declarados na arquitetura (`System Architecture - Institutional.md:49-59, 93-105`) — **não todos implementados** no código atual:
- `institutional.condominium.mandate-ended`
- `institutional.condominium.mandate-transferred`
- `institutional.condominium.unit-registered`
- `institutional.governance.masterplan-created`
- `institutional.governance.masterplan-status-changed`
- `institutional.governance.event-created`
- `institutional.governance.comunicado-created`
- `institutional.governance.question-created`
- `institutional.governance.question-closed`
- `institutional.governance.adendo-created`
- `institutional.validation.validated`
- `institutional.validation.rejected`
- `institutional.compliance.declaration-submitted`
- `institutional.compliance.signature-completed`
- `institutional.compliance.conflict-declared`
- `institutional.compliance.governance-score-updated`
- `institutional.compliance.issue-detected`
- `institutional.evaluation.management-submitted`
- `institutional.evaluation.company-submitted`

Eventos consumidos (declarados):
- `identity.auth.user-registered` — cria estrutura inicial.
- `identity.user.onboarding-completed` — libera trial.
- `commercial.deal.confirmed` — auto-publica Timeline.
- `commercial.execution.submitted` — cria ValidationItem.
- `commercial.proposal.sent` — idem.
- `commercial.technical-activity.sent` — idem.
- `commercial.comunicado.sent` — idem.
- `institutional.governance.timeline-published` (loop-back) — recalcula MasterPlan.
- `assembly.voting.closed` — revoga acesso temporário a vídeos empresa (Req 103 cross-domain).

Implementação: `IEventPublisher` (`domain/events/events.go:45-48`) com `Publish(event)` + `Subscribe(eventType, handler)`. Adapter canônico em `infrastructure/events/InMemoryPublisher` (dev) → NATS na produção.

---

## § 13. Dependências inter-sub-produto

| Sub-produto | Direção | Contrato |
|---|---|---|
| **Identity** | in | Authenticate middleware popula `selected_condominium_id`; ABAC valida `RequirePermission(action, resource)`. Memberships são criadas aqui quando morador aceita convite. |
| **Billing** | in | Tier (`trial/base/plus/pro`) gating feature matrix (§ 5). Hard limit 15 condomínios. Soft paywall no criar comunicado (plus+) e export PDF (plus+). |
| **Connect Me** (Sub-produto 2) | bi | Out: eventos `validation.validated` viram TimelineEntry `registro_de_execucao`. In: consumo `commercial.execution.submitted` → cria ValidationItem. |
| **Reputação** (Sub-produto 3) | out | Governance Score (Req 33) consome agregados Timeline + Master Plan + Compliance + Assessment. |
| **Content** (Sub-produto 4) | out | Perfil síndico linka `video_id` do vídeo institucional (trava trimestral — Req 29). Governança só armazena referência. |
| **Assembleia** (Sub-produto 5) | in | Ata homologada vira TimelineEntry `assembly_result` (imutável). Votação consulta fração ideal (**GAP M1** — hoje não existe). |
| **Compliance** (Sub-produto 9) | in/self | §6.11 aqui centraliza entidades. Gatilhos: encerrar mandato → exige declaração anual em dia + todas assinaturas de usuários. |
| **Marketing** (Sub-produto 10) | n/a | Sem dependência direta. |
| **Administradoras** (Sub-produto 11) | in | EmpresaSindicaLink concede acesso delegado (visualizar/editar publicações, validar registros empresa). Token por email + permissions toggles + audit log. |

Eventos cross-domain críticos:
- `commercial.deal.confirmed` → `TimelineEntry{registro_de_execucao}` → cascata `timeline.published` → MasterPlan `AdvanceStatusFromTimeline`.
- `assembly.voting.closed` → `video.visibility.revoked` (Req 103).
- `mandate.ended` → trigger Dossiê PDF (Req 40; job async).

---

## § 14. Estado no código

Features implementadas (F-###) — mapeadas do diretório `application/use_cases/`:

| F-GI-### | Use Case | Arquivo | Status |
|---|---|---|---|
| F-GI-01 | Criar Condomínio | `create_condominium_use_case.go` | ✅ implementado; limite 15 enforced |
| F-GI-02 | Registrar Unit | `create_unit_use_case.go` | ⚠️ **sem unit_type + fracao_ideal** |
| F-GI-03 | Registrar Membership (titular/dep) | `register_membership_use_case.go` | ✅ |
| F-GI-04 | Encerrar Membership | `end_membership_use_case.go` | ✅ com 4 end_reasons |
| F-GI-05 | Encerrar Mandato | `end_mandate_use_case.go` | ✅ |
| F-GI-06 | Transferir Mandato | `transfer_mandate_use_case.go` | ✅ (Req 47) |
| F-GI-07 | Criar TimelineEntry | `create_timeline_entry_use_case.go` | ✅ com adendo + INSERT-only |
| F-GI-08 | Listar Timeline | `list_timeline_use_case.go` | ✅ com filtros |
| F-GI-09 | Criar MasterPlanActivity | `create_master_plan_activity_use_case.go` | ✅ com 26+26+9+8 catálogos |
| F-GI-10 | Listar MasterPlan | `list_master_plan_use_case.go` | ✅ com status filter |
| F-GI-11 | Criar Announcement + Publish | `announcement_use_cases.go` | ✅ 8 tipos × 4 prioridades |
| F-GI-12 | Criar Event + RSVP | `event_use_cases.go` | ✅ 13 tipos |
| F-GI-13 | Criar Poll + vote + close | `poll_use_cases.go` | ✅ 3 tipos; MVP recomenda apenas yes_no_dk |
| F-GI-14 | Avaliação bimestral (responder + métricas) | `management_assessment_use_cases.go` | ✅ agregação anônima |
| F-GI-15 | Validation Hub (list + decide) | `validation_use_cases.go` | ✅ 4 status; linka TimelineEntry ao validar |
| F-GI-16 | Compliance (assinar termo + declaração anual + audit) | `compliance_use_cases.go` | ✅ 3 entidades implementadas |
| F-GI-17 | Dashboard síndico | `dashboard_use_case.go` | ✅ agregados básicos |

**Gaps críticos no código**:

- 🔴 **Unit sem `unit_type`** — entidade `unit.go` + migração 00202 não declaram. Spec Req 9 exige 6 valores (`apartamento/casa/sala_comercial/loja/garagem/outro`).
- 🔴 **Unit sem `fracao_ideal`** — idem. Spec Req 9 + Req 57 exigem `NUMERIC(5,4)`. Bloqueia votação ponderada na Assembleia.
- 🟡 **TimelineEntry sem campo `visibility`** — spec sugere `gestao` vs `publico_moradores`; código infere via `entry_type`/ABAC. Segmentação explícita ausente.
- 🟡 **Eventos auto-notificação 3d/1d/3h** — Req 13 exige; código não tem job/scheduler ainda.
- 🟡 **Conflito de Interesse (Req 36 / C4)** — sem entidade nem tabela; declaração inexistente.
- 🟡 **Risk Map (Req 38 / C7)** — derivado de agregações sobre `master_plan_activities.risk_code`; sem entidade dedicada (aceitável como view).
- 🟡 **Dossiê PDF assinado (Req 40 / C11)** — job ausente.
- 🟡 **Governance Score (Req 33)** — sub-produto 3; aqui apenas coleta eventos que o score consome.

---

## § 15. Gaps / pendências

| ID | Gap | Severidade | Bloqueia M1? | Ação |
|---|---|---|---|---|
| **G-GI-01** | `Unit.unit_type` ausente em entidade e migração | 🔴 crítico | parcial (shopping/galeria sem categorização) | Criar enum `unit_type` (6 valores), adicionar coluna + CHECK, atualizar entidade |
| **G-GI-02** | `Unit.fracao_ideal` (NUMERIC(5,4)) ausente | 🔴 crítico | **SIM** — Assembleia votação ponderada | Adicionar coluna + validação (`>=0 AND <=1`); ajustar cálculo de quórum no Sub-produto 5 |
| **G-GI-03** | `TimelineEntry.visibility` não existe | 🟡 médio | não | Adicionar enum `visibility ∈ {gestao, publico_moradores}` default pelo entry_type |
| **G-GI-04** | Eventos de notificação pré-evento (3d/1d/3h) | 🟡 médio | não | Job scheduler (Sprint 3+) |
| **G-GI-05** | Matriz de Conflito de Interesse (C4) | 🟡 médio | não | Entidade + tabela + UX C4 |
| **G-GI-06** | Dossiê PDF assinado (Req 40 / C11) | 🟡 médio | não | Job assíncrono pós-end_mandate |
| **G-GI-07** | Audit log archive MinIO após 1 ano | 🟢 baixo | não | Job cron compliance |
| **G-GI-08** | Eventos de domain `institutional.*` implementados parcialmente (3/20) | 🟡 médio | não | Publicar nos use cases restantes |
| **G-GI-09** | Inconsistência Score: C10 fala em 2 scores (Governança 1-10 + Compliance 0-100) vs Req 33 (1 score 0-100) | 🟢 baixo | não | Decisão de produto — canônico Req 33 |
| **G-GI-10** | Inconsistência PDF Síndico: "12 condomínios" vs Decision 11 "15" (atual) | 🟢 baixo | não | Canônico: 15 (enforced no código) |
| **G-GI-11** | Revogação cascata titular → dependentes não está explícita no `end_membership_use_case.go` | 🟡 médio | talvez | Auditar use case; garantir cascata |
| **G-GI-12** | Tipos Poll `multiple_choice` e `scale_1_to_5` fora do recorte MVP (apenas `yes_no_dk` para pré-assembleia) | 🟢 baixo | não | Flag de produto — hide nos forms não-MVP |
| **G-GI-13** | Compliance Declaration não dispara publicação automática na Timeline (spec C2: "Publicação automática LT tipo administrativo → conformidade_e_governanca") | 🟡 médio | não | Adicionar emissão no use case |
| **G-GI-14** | `functional-matrix.md` ainda usa slugs legados `N1/N2/N3` — deve migrar para Stripe-style `base/plus/pro` universal (ADR-0032) | 🟡 médio | não | Rewrite matriz + docs |
| **G-GI-15** | Empresa Síndica (EmpresaSindicaLink) permissões toggles não-implementadas — só audit log existe | 🟡 médio | não | Entidade + tabela + UX S5 |

---

## § 15.5 Telas & Endpoints

Rastreabilidade ponta-a-ponta entre UI (web/mobile) e backend está consolidada em [[../../03-subprojects/traceability]] §2.1.

**Escopo** deste sub-produto na matriz de traceability:
- **Telas web**: 24 (S1-S17 · S21-S27) + 15 (M1-M15) = **39 superfícies**
- **Telas mobile**: 15 (M1-M15) + 5 (S1 · S2 · S6 · S7 · S8) = **20 superfícies**
- **Endpoints backend**: ~48 rotas sob `/api/v1/condominiums/*`, `/api/v1/institutional/*`, `/api/v1/syndic/*`

**Endpoints-chave**:
- Timeline: `/api/v1/condominiums/{id}/timeline` · `/api/v1/institutional/timeline/{id}` · `/amend` · `/history`
- Plano Diretor: `/api/v1/condominiums/{id}/master-plan/activities` · `/{aid}`
- Comunicados: `/api/v1/condominiums/{id}/announcements`
- Eventos: `/api/v1/condominiums/{id}/events`
- Memberships: `/api/v1/institutional/memberships/*`
- Validações: `/api/v1/condominiums/{id}/validations`
- Dashboard: `/api/v1/condominiums/{id}/dashboard/*` (🔴 agregador M1-blocker)
- Compliance transversal: `/api/v1/condominiums/{id}/compliance/*` (ver sub-produto 9)

**Invariantes observáveis em UI**: INV-021 (fração Σ≤100) · INV-024 (timeline append-only) · INV-025 (announcement imutável) · INV-026 (membership única) · INV-027 (≤15 condos) · INV-029 (≤2 dep) · INV-032 (adendo) · INV-119 (DB-level immutability).

**Gaps apontados pela matriz** (ver §5.1 da traceability): S6 dashboard agregador · S27 agregador plus+ · C10 governance score endpoint ausente · S1 first-access inferido.

**Sprints**: backend Sprint 2 (base) + Sprint 3 (completa) ✅ · UI Sprint 10 (M1 blocker) 🟡.

---

## § 16. Referências

**Specs canônicas**:
- `backend/.kiro/specs/master-sindico/requirements/institutional.md` — Req 6.2–17 (fonte primária Governança)
- `backend/.kiro/specs/master-sindico/requirements/cross-domain.md` — Req 33-47 (Compliance + Mandato)
- `backend/.kiro/specs/master-sindico/requirements/functional-matrix.md` — matriz tier×persona

**Código**:
- `backend/internal/modules/institutional/domain/entities/*.go` (12 entidades)
- `backend/internal/modules/institutional/domain/enums/*.go` (enums + catálogos)
- `backend/internal/modules/institutional/domain/events/events.go` (domain events)
- `backend/internal/modules/institutional/application/use_cases/*.go` (36 arquivos — 17 use cases + testes)
- `backend/internal/shared/providers/database/migrations/00200_*.sql` a `00210_*.sql` (11 DDL files)

**Obsidian Vault**:
- `Clients/MasterSindico/01 - Arquitetura de Sistema/System Architecture - Institutional.md` — visão de arquitetura (6 módulos)
- `Clients/MasterSindico/02 - Domínios e Requisitos/Domínio - Identidade.md` — ABAC + roles (cross-reference)

**Content (Development/Content)**:
- `02-Jornadas/Inventario-Telas.md` — M1-M15 + S1-S31 (canônico de telas)
- `02-Jornadas/Onboarding-por-Persona.md` — fluxo 4 personas
- `03-Modulos/Compliance-Detalhado.md` — C1-C11 (detalhe tela a tela)

**Cruzamentos**:
- Sub-produto 2 (Connect Me) — valida registros empresa → ValidationItem → TimelineEntry
- Sub-produto 3 (Reputação) — consome agregados governance
- Sub-produto 4 (Conteúdo) — vídeo institucional do perfil síndico
- Sub-produto 5 (Assembleia) — ata → Timeline `assembly_result`; depende de fração ideal (GAP G-GI-02)
- Sub-produto 9 (Compliance) — gatilhos bloqueiam encerrar mandato, exportar dossiê
- Sub-produto 11 (Administradoras) — EmpresaSindicaLink, permissões delegadas

**Dívidas técnicas relacionadas**:
- DT-010 — `IAuditLogger` cross-módulo (Sprint 10) — `Compliance-Detalhado.md:273`
- D-0XX para telas ausentes — `backend/.kiro/STATE.md`

**ADRs relevantes**:
- ADR-0032 — Planos globais Stripe-style (trial/base/plus/pro) — elimina slugs N1/N2/N3
- D-056 — Agnosticismo de provedor
- D-134 (revoga D-058/D-072) — Admin = `apps/admin` dedicada no monorepo web (Cloudflare Access SSO+MFA)

---

*Fim do sub-produto 1 — Governança Institucional.*
