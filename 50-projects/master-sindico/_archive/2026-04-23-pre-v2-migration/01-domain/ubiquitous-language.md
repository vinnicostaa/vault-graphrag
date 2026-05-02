---
title: Ubiquitous Language — Master Síndico
type: project
tags: [master-sindico, ddd, ubiquitous-language]
project: master-sindico
created: 2026-04-22
---

# Ubiquitous Language — Master Síndico

Glossário **vinculante** entre negócio e código. Se o termo está aqui, é assim que usamos. Mudança requer PR + consentimento explícito.

Glossário leve ao leitor em [[../00-product/glossary]].

---

## Convenção de idioma

| Categoria | Idioma | Exemplo |
|---|---|---|
| **Domínio legal/condominial brasileiro** (termos com significado jurídico) | pt-BR | `Assembleia`, `Sindico`, `Convencao`, `PlanoD iretor`, `FracaoIdeal`, `Condomino`, `Ata` |
| **Conceitos técnicos universais** | inglês | `User`, `Session`, `Subscription`, `Plan`, `Webhook`, `Event`, `Role` |
| **API paths** | inglês | `/api/v1/condominiums`, `/api/v1/assemblies` |
| **Enums de estado técnico** | inglês | `state: active \| paused \| canceled` |
| **Enums de domínio pt-BR** | pt-BR | `ReputacaoTier: Bronze \| Prata \| Ouro \| Platina \| Diamante` |

**Regra**: privilegia consistência dentro do contexto; não misture idiomas em mesmo nome.

---

## Termos obrigatórios (não alterar sem ADR)

### Identity
| Termo | Tipo | Descrição |
|---|---|---|
| `User` | Aggregate Root | Identidade única por CPF/CNPJ |
| `Session` | Entity | Sessão ativa com device fingerprint |
| `Email` | VO | Imutável, validação RFC 5322 lowercase |
| `CPF` | VO | 11 dígitos; `Masked()` pra log |
| `CNPJ` | VO | 14 dígitos; `Masked()` pra log |
| `Password` | VO | Nunca armazenado plain |
| `Phone` | VO | E.164; validação BR |
| `DeviceFingerprint` | VO | Hash opaco; não reverse |
| `Role` | enum pt-BR | `Sindico \| Morador \| Subsindico \| Conselho \| Admin \| Empresa \| Parceiro \| Agencia` |

### Billing
| Termo | Tipo | Descrição |
|---|---|---|
| `Subscription` | Aggregate Root | Assinatura Stripe mapeada |
| `Plan` | Entity | `tier`, features, preço, quotas |
| `PlanTier` | enum | `Trial \| Basic \| Pro \| Enterprise \| EmpresaBasico \| EmpresaPro \| EmpresaEnterprise \| Parceiro` |
| `TrialDuration` | VO | Depende da persona |
| `Money` | VO | int64 centavos BRL |
| `FeatureUsage` | Aggregate Root | Consumo vs quota |
| `Quota` | VO | `limit` + `unlimited` flag |
| `StripeWebhookEvent` | Entity | Idempotency ledger |

### Institutional
| Termo | Tipo | Descrição |
|---|---|---|
| `Condominium` | Aggregate Root | Tenant primário |
| `Unit` | Entity (dentro de Condominium) | Apartamento/casa; tem fração ideal |
| `FracaoIdeal` | VO | Percentual 0..100 com 4 casas decimais |
| `Membership` | Entity | Vínculo user↔condo↔unit com role |
| `TimelineEntry` | Entity imutável | Sem UPDATE; append-only |
| `MasterPlan` (Plano Diretor) | Aggregate Root | Multi-ano; atividades com status |
| `Announcement` | Aggregate Root | Comunicado oficial; após publish imutável |
| `AnnouncementType` | enum pt-BR | `InformativoGeral \| Aviso \| Urgente \| Festa` |
| `Event` | Aggregate Root | Reunião, obra, festa |
| `EventType` | enum pt-BR | `ReuniaoMoradores \| Obra \| Festa \| Assembleia` |
| `EventResponse` | Entity | `ciente \| confirmado` (UPSERT) |
| `Poll` | Aggregate Root | Enquete informal (diferente de votação formal) |
| `SyndicMandate` | Entity | Período em que alguém é síndico |
| `ValidationItem` / `ComplianceAssessment` | Aggregates | Checklists legais |

### Commercial
| Termo | Tipo | Descrição |
|---|---|---|
| `Company` | Aggregate Root | Empresa com CNPJ |
| `CompanyUser` | Entity | user↔company link com role interno |
| `ConnectMeRequest` | Aggregate Root | RFP aberto pelo síndico |
| `ConnectMeFluxo` | enum pt-BR | `Vizinhanca \| Academia \| Marketplace \| Direto` |
| `Proposal` | Entity | Resposta de empresa |
| `ClosedDeal` | Entity | Contrato fechado |
| `ExecutionRecord` | Entity | Histórico de execução do contrato |
| `CompanyEvaluation` | Entity | Avaliação pós-contrato |
| `ReputacaoTier` | VO (enum) | `Bronze \| Prata \| Ouro \| Platina \| Diamante` |
| `SupplierVoteSession` | Aggregate Root | Votação colegiada do conselho |
| `SupplierVoteCast` | Entity | Voto individual |
| `EmpresaSindicaLink` | Entity | Convite síndico↔empresa |
| `HarassmentReport` | Entity | Denúncia |

### Content
| Termo | Tipo | Descrição |
|---|---|---|
| `Video` | Aggregate Root | Mux-backed; metadata local |
| `VideoState` | enum | `uploading \| processing \| moderation \| approved \| published \| locked \| removed` |
| `LockedUntil` | VO | Timestamp 90d pós publish |
| `SearchIndex` | Aggregate Root | OpenSearch representation |
| `MediaProvider` | enum | `Mux \| S3` (só Mux ativo) |

### Assembly
| Termo | Tipo | Descrição |
|---|---|---|
| `Assembly` (Assembleia) | Aggregate Root | Contém Agenda, Attendance, Votes, Minutes |
| `AssemblyType` | enum pt-BR | `Ordinaria \| Extraordinaria \| Instalacao` |
| `AgendaItem` | Entity (dentro de Assembly) | Item de pauta |
| `AgendaItemState` | enum | `Pending \| InDebate \| Voting \| Passed \| Rejected \| Withdrawn` |
| `Vote` | Entity | Voto individual |
| `VoteChoice` | enum pt-BR | `Aprovar \| Rejeitar \| Abster` |
| `Minutes` (Ata) | Aggregate Root | Imutável após publish |
| `LiveSession` | Aggregate Root | Sessão Livekit |
| `ProxyGrant` (Procuração) | Entity | Outorga de voto |
| `AttendanceRecord` (Presença) | Entity | Quem compareceu |
| `ScienceRecord` (Ciência) | Entity | Registro de ciência sobre decisão |
| `QuorumThreshold` | VO | Mínimo de fração para deliberar |

### Cross-domain
| Termo | Tipo | Descrição |
|---|---|---|
| `AuditEntry` | Entity | IAuditLogger output; DT-010 |
| `AbacDecision` | VO | `Permit \| Deny(reason)` |
| `DomainEvent` | Interface | Evento publicado; payload imutável |

---

## Termos **proibidos** (por ambiguidade ou desuso)

- ❌ `Tenant` no código de domínio — usar `Condominium` (é o conceito do negócio)
- ❌ `Building` — usar `Condominium`
- ❌ `Customer` — usar `User` ou `Company`
- ❌ `Invoice` solto — está em Stripe, não é agregado nosso
- ❌ `Resident` — usar `Morador` (role) ou `User` (identidade)
- ❌ `Manager` — ambíguo; usar `Sindico` ou role explícita
- ❌ `Permission` solto — usar `AbacDecision` ou `Policy`
- ❌ `Account` — usar `User` ou `Subscription` conforme contexto
- ❌ `Admin` como role geral — usar `SysAdmin` (global) ou `role: Admin` (no contexto do módulo)

---

## Observação: termos pt-BR em código Go

Manter quando tem **semântica jurídica única** que inglês não captura:

```go
// OK — jurídico brasileiro preciso
type Assembleia struct { ... }
type FracaoIdeal struct { ... }
type Sindico struct { ... }

// OK — enum legível
type ConnectMeFluxo string
const (
    FluxoVizinhanca ConnectMeFluxo = "vizinhanca"
    FluxoAcademia   ConnectMeFluxo = "academia"
)

// RUIM — inglês sem ganho
type Assembly struct { ... }  // pt-BR é mais preciso pro contexto legal
```

Em API pública (OpenAPI paths, JSON fields), preferir inglês por compat internacional:

```json
{
  "assembly_type": "ordinaria",  // path em inglês, valor pt-BR
  "agenda_items": [...],
  "quorum_threshold": 0.50
}
```

---

## Revisão

- Termo novo: PR com justificativa + link para conceito de origem
- Termo que mudou: ADR + atualização de todos os usos
- Termo que saiu: deprecation em 1 sprint; remoção em 2

## Links

- [[_moc]]
- [[bounded-contexts]]
- [[../00-product/glossary]]
- [[../CLAUDE]]
- [[../STEERING]]
