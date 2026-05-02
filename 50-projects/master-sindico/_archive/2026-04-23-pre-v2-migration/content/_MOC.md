---
title: "_MOC — Map of Content (Master Síndico Obsidian Vault)"
updated: 2026-04-22
maintainer: agent-3 (analista)
type: index
---

# _MOC — Map of Content

Índice navegável do vault `Content/` do **Master Síndico**. Este vault NÃO é a fonte canônica de implementação — ele é o arquivo histórico/Obsidian do projeto. **Fonte canônica viva**: [`backend/.kiro/specs/master-sindico/`](../backend/.kiro/specs/master-sindico/).

## Hierarquia de fontes (ordem dura de consulta)

1. `backend/.kiro/SESSION_CHARTER.md` — ordens ativas da sessão
2. `backend/.kiro/specs/master-sindico/requirements/*.md` — recortes por bounded context (canônico)
3. `backend/.kiro/specs/master-sindico/design/`, `tasks/`, `reference/`
4. `backend/.kiro/steering/*.md` — regras duras
5. `backend/.kiro/STATE.md` — D-0XX decisões, DT-0XX dívidas
6. `backend/.kiro/AUDIT.md` — A-0XX bugs descobertos
7. **Este vault** (`Content/`) — discovery histórico, jornadas em PDF, design original
8. Obsidian externo do João em `~/Documentos/Obsidian Vault/Clients/MasterSindico/` — visualizações (canvas, tldraw)

## Entradas do vault (criadas/mantidas pelo agente analista)

### Handoffs entre agentes
- [[_handoffs/2026-04-22-relatorio-agente3-analista]] — relatório completo desta sessão (entrega para agente principal)

### Regras de negócio
- [[regras-de-negocio/regras-canonicas]] — 10 regras de ouro + 12 decisões resolvidas + invariantes técnicas (T1-T15)
- [[regras-de-negocio/quebras-detectadas]] — 18 inconsistências detectadas entre fontes (com severidade e fix proposto)

### Metodologia
- [[sdd-gsd-aplicado]] — síntese de SDD + GSD (GitHub Spec Kit) e como já estamos aplicando + recomendações pontuais

### Arquivos originais (intocados — preservar para evidência histórica)
- `ARCHITECTURE_ANALYSIS_REPORT.md` (67KB) — análise arquitetural inicial (Mar/2026)
- `requirements.md` (206KB) — requisitos monolíticos originais (recortados por módulo em backend/.kiro/specs/master-sindico/requirements/)
- `design.md` (84KB) — design monolítico original (em transição para backend/.kiro/specs/master-sindico/design/)
- `tasks.md` (68KB) — tasks monolíticas originais (recortadas em backend/.kiro/specs/master-sindico/tasks/sprint-N.md)
- `excopo-tecnico-antigo.txt` (31KB) — escopo técnico inicial
- `main.go.md` — esboço do main.go inicial
- `contents/` — 23 PDFs de jornadas (Síndico, Morador, Empresa, Marketing, Parceiro), matrizes funcionais e diagramas de assembleia
- `contents/master-sindico-domain-mapping.md` — Domain Mapping & Architecture (Mar/2026, **DESATUALIZADO** — usa pt-BR roles, TypeScript, Mercado Pago)
- `contents/master-sindico-arquitetura-solucoes.md` — Arquitetura de Soluções (Mar/2026, **PARCIALMENTE STALE** — assembly conflict marcado como pendente, já resolvido por Decision 1)

> Os arquivos originais ficam aqui como evidência. **Não modificar.** Para consultar regra atual, sempre ir em `backend/.kiro/specs/master-sindico/requirements/*.md`.

### Pastas

```
Content/
├── _MOC.md                                  # este arquivo
├── _handoffs/                               # entregas entre agentes (esta sessão multi-agente)
├── _inbox/                                  # docs novos sem classificação ainda
├── regras-de-negocio/                       # auditoria + canon
├── sdd-gsd-aplicado.md
├── ARCHITECTURE_ANALYSIS_REPORT.md          # original
├── design.md                                # original
├── requirements.md                          # original
├── tasks.md                                 # original
├── excopo-tecnico-antigo.txt                # original
├── main.go.md                               # original
├── .kiro/                                   # specs antigas duplicadas (stale)
└── contents/                                # PDFs e MDs de discovery
```

## Mapa de bounded contexts (atalhos)

| Domínio | Spec canônico | Entidades-chave |
|---|---|---|
| Identity | `backend/.kiro/specs/master-sindico/requirements/identity.md` | User, Session, AuthUser; 1-device fingerprint; PKCE OIDC; ABAC |
| Billing | `backend/.kiro/specs/master-sindico/requirements/billing.md` | Subscription (append-only), Plan, FeatureUsage; trial por persona; soft-block |
| Institutional | `backend/.kiro/specs/master-sindico/requirements/institutional.md` | Condominium, Unit, Mandate, Timeline (append-only), MasterPlan |
| Commercial | `backend/.kiro/specs/master-sindico/requirements/commercial.md` | Company, ConnectMe (4 fluxos), Proposal, Deal, ExecutionRecord, Amendment |
| Content | `backend/.kiro/specs/master-sindico/requirements/content.md` | Video (Mux), Search, LMS, MarketingAgency (token, não tenant) |
| Assembly | `backend/.kiro/specs/master-sindico/requirements/assembly.md` | Assembly, Agenda, Vote, Proxy, Minutes, Homologation; WebSocket |

## Regras de ouro (10) — resumo

1. Separação User/Profile/Subscription
2. Imutabilidade Timeline + Negócio Fechado + Votos
3. Connect Me ≠ Chat (unidirecional)
4. Vídeos empresa só visíveis em votação
5. Métricas (likes/comentários) privadas ao dono
6. Backend = verdade única
7. Trava trimestral (90 dias) p/ vídeos institucionais e currículo
8. 1 dispositivo por conta (fingerprint + IP)
9. Tenant isolation (cross-tenant só via grants)
10. LGPD by design (log de revelação)

Ver [[regras-de-negocio/regras-canonicas]] para detalhamento completo.

## Status do projeto (snapshot 2026-04-22)

- **Backend**: Sprints 0-9 completos. Marco 1 (MVP contratual) = 2026-05-08. Sprint 10 (frontend + Saga + Zitadel managed) próximo.
- **Web (SolidJS)**: scaffold (auth placeholder em 6 apps).
- **App (Flutter)**: shell auth feature.
- **Specs**: backend canônico, web/app sem `.kiro` próprio (decisão D-030, 2026-04-22).
- **Sessão atual**: 3 agentes em paralelo — agente 1 implementa, agente 2 revisa, agente 3 (este) é analista cross-doc/regras.

## Convenções deste vault

- Português brasileiro para textos; identificadores e paths em inglês.
- Wiki-links `[[arquivo]]` para navegação Obsidian.
- Frontmatter YAML em todo arquivo novo (title, updated, type).
- Sem emojis decorativos — apenas funcionais (🔴/🟡/✅ para status).
- Datas no formato ISO `YYYY-MM-DD`.
