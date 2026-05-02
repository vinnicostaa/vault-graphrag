---
title: Aggregate — Event
type: spec
tags: [domain, ddd, aggregate, institutional, master-sindico, v2]
bc: institutional
source: 90-ingested/.../specs/requirements/institutional.md + D-115 (2026-04-24)
created: 2026-04-23
updated: 2026-04-24
---

# Aggregate — `Event` (Evento Condominial)

**BC**: institutional · **Raiz**: ✅

Evento agendado no condomínio. Moradores confirmam ciência/presença. Enum canônico `EventType` tem **18 valores** via [[../../STATE#d-115|D-115]] (2026-04-24, Opção C).

**Escopo**: este aggregate **NÃO cobre** Reservas de espaço comum (churrasqueira, salão, piscina) — vão para sub-produto [[../../00-product/sub-produtos/12-reservas-espacos]] M3+. Também **NÃO cobre** Comunicados (enum `announcement_type` em [[Announcement]]) nem Enquetes ([[Poll]]).

## Entidade raiz

```go
type Event struct {
    id              EventID
    condominiumID   CondominiumID
    createdBy       UserID
    eventType       EventType               // 18 valores — ver §EventType
    equipmentTarget *EquipmentTarget        // opcional, subcategoria para manutencao_programada + inspecao_tecnica
    externalRefID   *string                 // para shadow Events (ex: Assembly.ID quando eventType=assembleia_*)
    title           string
    description     string
    scheduledAt     time.Time
    location        *string
    status          EventStatus             // scheduled | in_progress | finished | canceled
    responses       []EventResponse
    createdAt       time.Time
    updatedAt       time.Time
}

type EventResponse struct {
    eventID     EventID
    userID      UserID
    response    ResponseType                // ciente | confirmado
    respondedAt time.Time
}
```

## Value Objects

- `EventID` (UUIDv7)
- `EventType` (enum — 18 valores abaixo)
- `EquipmentTarget` (enum secundário — 14 valores abaixo)
- `EventStatus`, `ResponseType`

### EventType (18 valores canônicos — D-115 Opção C)

Ordem de uso típico no domínio (governança → institucional → compliance → operacional):

| # | Valor | pt-BR label | Shadow? | Subcategoria? |
|---|---|---|---|---|
| 1 | `assembleia_ordinaria` | Assembleia Ordinária | ✅ shadow (ver §Shadow) | — |
| 2 | `assembleia_extraordinaria` | Assembleia Extraordinária | ✅ shadow | — |
| 3 | `reuniao_conselho` | Reunião de Conselho | — | — |
| 4 | `reuniao_administrativa` | Reunião Administrativa | — | — |
| 5 | `prestacao_de_contas` | Prestação de Contas | — | — |
| 6 | `audiencia_juridica` | Audiência Jurídica | — | — |
| 7 | `apresentacao_projeto` | Apresentação de Projeto | — | — |
| 8 | `reuniao_com_fornecedor` | Reunião com Fornecedor | — | — |
| 9 | `manutencao_programada` | Manutenção Programada | — | ✅ `equipment_target` |
| 10 | `inspecao_tecnica` | Inspeção Técnica | — | ✅ `equipment_target` |
| 11 | `obra_programada` | Obra Programada | — | — |
| 12 | `interrupcao_programada_servicos` | Interrupção Programada de Serviços | — | — |
| 13 | `treinamento_equipe` | Treinamento de Equipe | — | — |
| 14 | `simulado_emergencia` | Simulado de Emergência | — | — |
| 15 | `campanha_institucional` | Campanha Institucional | — | — |
| 16 | `evento_comunitario` | Evento Comunitário | — | — |
| 17 | `atualizacao_normativa` | Atualização Normativa | — | — |
| 18 | `outros` | Outros | — | — |

### EquipmentTarget (14 valores — subcategoria de manutencao_programada + inspecao_tecnica)

| # | Valor | pt-BR label | Uso típico |
|---|---|---|---|
| 1 | `elevador` | Elevador | manutenção + inspeção |
| 2 | `piscina` | Piscina | manutenção |
| 3 | `jardim` | Jardim | manutenção |
| 4 | `gerador` | Gerador | manutenção + inspeção |
| 5 | `hidraulico` | Hidráulico | manutenção |
| 6 | `eletrico` | Elétrico | manutenção + inspeção |
| 7 | `portao_automatico` | Portão Automático | manutenção |
| 8 | `sistema_incendio` | Sistema de Incêndio (AVCB) | inspeção + manutenção |
| 9 | `cftv` | CFTV / Câmeras | manutenção |
| 10 | `climatizacao` | Climatização / Ar condicionado | manutenção |
| 11 | `fachada` | Fachada | inspeção |
| 12 | `telhado` | Telhado | inspeção |
| 13 | `estrutura` | Estrutura | inspeção |
| 14 | `outros` | Outros equipamentos | escape-hatch |

Admin cliente pode adicionar valores no enum secundário via painel admin plataforma ([[../../STATE#d-113|D-113]]) quando necessário.

## Invariantes

- **INV-EVT-001** — `scheduledAt > NOW()` na criação (não retroativa). Exceção: shadow Events derivados de Assembly (podem ter `scheduledAt` passado se Assembly já aconteceu historicamente).
- **INV-EVT-002** — `EventResponse` UPSERT único por (event_id, user_id) — morador pode trocar resposta `ciente ↔ confirmado` mas não duplica.
- **INV-EVT-003** — `eventType ∈ {manutencao_programada, inspecao_tecnica}` **pode** ter `equipmentTarget`; outros tipos devem ter `equipmentTarget = null`.
- **INV-EVT-004** — `eventType ∈ {assembleia_ordinaria, assembleia_extraordinaria}` **deve** ter `externalRefID = Assembly.ID` válido (shadow Event). Impossível criar manualmente via S16 — somente via emissão automática quando Assembly é publicada.
- **INV-EVT-005** — ao editar Event shadow, alterações não afetam `Assembly` original. CTA no UI redireciona para ASM-CFG.
- **INV-EVT-006** — `status.canceled` é terminal (não volta a `scheduled`).

## Factories

```go
// Criação manual via S16 (UI síndico)
func NewEvent(
    condoID CondominiumID, 
    createdBy UserID, 
    typ EventType, 
    equipment *EquipmentTarget,     // obrigatório se typ in {manutencao_programada, inspecao_tecnica}
    title, description string, 
    scheduledAt time.Time, 
    location *string,
) (*Event, error) {
    // Valida INV-EVT-001, INV-EVT-003, INV-EVT-004
    if typ == EventTypeAssembleiaOrdinaria || typ == EventTypeAssembleiaExtraordinaria {
        return nil, apperrors.ErrBusiness.WithMessage("eventos de assembleia são criados automaticamente; use ASM-CFG")
    }
    if (typ == EventTypeManutencaoProgramada || typ == EventTypeInspecaoTecnica) && equipment == nil {
        return nil, apperrors.ErrValidation.WithMessage("equipment_target obrigatório para este tipo")
    }
    // ...
}

// Shadow Event criado automaticamente quando Assembly publica
func NewShadowFromAssembly(
    assembly *Assembly,
) (*Event, error) {
    typ := EventTypeAssembleiaOrdinaria
    if assembly.Type() == AssemblyTypeExtraordinaria {
        typ = EventTypeAssembleiaExtraordinaria
    }
    assemblyID := string(assembly.ID())
    return &Event{
        id:              NewEventID(),
        condominiumID:   assembly.CondominiumID(),
        createdBy:       assembly.CreatedBy(),
        eventType:       typ,
        externalRefID:   &assemblyID,     // ✅ satisfaz INV-EVT-004
        title:           assembly.Title(),
        scheduledAt:     assembly.ScheduledFor(),
        status:          EventStatusScheduled,
    }, nil
}

func ReconstructEvent(...) *Event
```

## Métodos de domínio

- `RegisterResponse(userID UserID, response ResponseType) error` — UPSERT
- `Cancel(reason string) error` — `scheduled → canceled`
- `Start() error` / `Finish() error` — transições
- `IsShadow() bool` — retorna `externalRefID != nil`

## Repository interface

```go
type IEventRepository interface {
    Save(ctx context.Context, e *Event) error
    FindByID(ctx context.Context, id EventID) (*Event, error)
    FindByExternalRef(ctx context.Context, externalRefID string) (*Event, error)  // para sync com Assembly
    ListByCondominium(ctx context.Context, cid CondominiumID, futureOnly bool) ([]*Event, error)
    SaveResponse(ctx context.Context, r *EventResponse) error                       // UPSERT
}
```

## Eventos emitidos

- `EventScheduled(event_id, condominium_id, event_type, scheduled_at)` — fan-out notification morador
- `EventResponseRegistered(event_id, user_id, response)` — alimenta dashboard síndico
- `EventCanceled(event_id, reason)` — fan-out notification

## Shadow Events — fluxo automático

Quando `Assembly` é publicada (evento `AssemblyPublished`):

1. Subscriber `CreateShadowEventHandler` escuta
2. Invoca `Event.NewShadowFromAssembly(assembly)`
3. Persiste via `IEventRepository.Save`
4. Publica `EventScheduled` para trigger de notificações

Quando `Assembly` é cancelada (evento `AssemblyCanceled`):

1. Subscriber busca Event via `FindByExternalRef(assembly_id)`
2. Chama `event.Cancel(reason)`
3. Publica `EventCanceled`

Quando `Assembly` muda data (evento `AssemblyRescheduled`):

1. Subscriber busca Event shadow + atualiza `scheduledAt`
2. Publica `EventScheduled` novamente (re-notifica)

**UI S16 detecta shadow Events** pelo campo `externalRefID != nil`:
- Exibe badge "Assembleia"
- CTA "Gerenciar" redireciona para ASM-CFG daquele Assembly
- **Não permite edição local** de campos do shadow

## Links

- [[../bounded-contexts#3-institutional]]
- [[../../STATE#d-115|D-115]] — decisão Opção C
- [[Condominium]]
- [[Announcement]] — **enum disjunto** `announcement_type` (6 tipos que estavam em Bloco 9 ficam aqui)
- [[Assembly]] — shadow Events vêm daqui
- [[Poll]] — Enquete Consultiva (NÃO é Event)
- [[../../00-product/sub-produtos/12-reservas-espacos]] — Reservas (novo sub-produto M3+, NÃO Event)
- [[../../03-subprojects/web/ui-catalog/sindico/S16]] — UI Criar Evento
