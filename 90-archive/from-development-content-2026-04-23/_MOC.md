---
title: "_MOC — Map of Content (Master Síndico Obsidian Vault)"
updated: 2026-04-22
maintainer: agent-3 (analista)
type: index
---

# _MOC — Map of Content

Índice navegável do vault `Content/` do **Master Síndico**. Construído na sessão multi-agente de 2026-04-22 a partir de leitura completa de **23 arquivos originais** (PDFs + monolíticos + .ini + .md). Este vault é **arquivo histórico organizado** — a fonte canônica viva é `backend/.kiro/specs/master-sindico/`.

## Hierarquia de fontes (ordem dura de consulta)

1. `backend/.kiro/SESSION_CHARTER.md` — ordens ativas da sessão
2. `backend/.kiro/specs/master-sindico/requirements/*.md` — recortes por bounded context (canônico)
3. `backend/.kiro/specs/master-sindico/design/`, `tasks/`, `reference/`
4. `backend/.kiro/steering/*.md` — regras duras
5. `backend/.kiro/STATE.md` — D-0XX decisões, DT-0XX dívidas
6. `backend/.kiro/AUDIT.md` — A-0XX bugs descobertos
7. **Este vault** (`Content/`) — discovery histórico, jornadas em PDF, design original
8. Obsidian externo do João em `~/Documentos/Obsidian Vault/Clients/MasterSindico/` — visualizações

---

## Estrutura do vault (criada/mantida pelo agente analista)

```
Content/
├── _MOC.md                                       ← este arquivo
├── _handoffs/
│   ├── 2026-04-22-relatorio-agente3-analista.md  ← handoff inicial (1ª rodada)
│   └── 2026-04-22-relatorio-final-agente3.md     ← handoff final (2ª rodada — 2026-04-22)
├── _inbox/                                        (vazio — recomendações pendentes)
├── 00-Visao/                                      (a expandir conforme produto)
├── 01-Arquitetura/
│   └── Stack-e-Modulos.md                        ← stack atual + bounded contexts + mapa módulo↔BC
├── 02-Jornadas/
│   ├── Inventario-Telas.md                       ← 136+ telas catalogadas (M, S, E, MK, VS, VE, VM, VZ, LMS, C)
│   └── Onboarding-por-Persona.md                 ← 4 fluxos (Morador 4, Síndico 6, Empresa 7, Parceiro 4)
├── 03-Modulos/
│   ├── Assembleia-Live-Funcional.md              ← módulo Assembleia detalhado (5 etapas, 30+ regras)
│   ├── Compliance-Detalhado.md                   ← C1-C11 + 4 gatilhos
│   ├── Connect-Me-4-direcoes.md                  ← 4 fluxos + LGPD + Marketing Link
│   └── Vizinhanca-4-jornadas.md                  ← VZ + VS + VE + VM (29 telas)
├── 04-Listas-Mestre/
│   └── Enums-Consolidados.md                     ← TODOS os enums (19 categorias)
├── regras-de-negocio/
│   ├── regras-canonicas.md                       ← 10 regras + 12 decisões + T1-T15 + stack atual
│   └── quebras-detectadas.md                     ← 40 quebras cross-fonte (10 critical, 15 medium, 15 low)
├── sdd-gsd-aplicado.md                           ← metodologia SDD + GSD aplicada + recomendações
└── (arquivos originais intocados):
    ├── ARCHITECTURE_ANALYSIS_REPORT.md           ← 67KB — análise arquitetural inicial Mar/2026
    ├── design.md                                 ← 84KB — design monolítico original
    ├── requirements.md                           ← 206KB — requisitos monolíticos (104 Reqs)
    ├── tasks.md                                  ← 67KB — tasks monolíticas
    ├── excopo-tecnico-antigo.txt                 ← 31KB — escopo técnico v3.0 Janeiro 2026
    ├── main.go.md                                ← esboço inicial do main
    ├── .kiro/                                    ← specs antigas duplicadas (stale)
    └── contents/
        ├── master-sindico-arquitetura-solucoes.md ← 35KB — Mar/2026, parcialmente stale
        ├── master-sindico-domain-mapping.md       ← 21KB — Mar/2026, stack legacy
        ├── 6 PDFs de jornadas (Síndico, Morador, Empresa x2, Marketing, Parceiro)
        ├── 4 PDFs de arquitetura (Governança, Assembleia Funcional, Diagrama Visual, Telas Assembleia)
        ├── 2 PDFs de matrizes (Funcional Geral, Trial)
        ├── 3 PDFs de Banco de Talentos (Currículo, Cadastro)
        ├── 1 PDF de Onboarding
        ├── 1 PDF de Vizinhança Morador
        ├── 4 .ini (LMS, Academia, Vizinhança Empresa, Tela Morador/Síndico/Empresa)
        └── 1 .md (Vizinhança Síndico)
```

---

## Atalhos por necessidade

### "Quero implementar a feature X — onde leio?"
1. Spec viva: `backend/.kiro/specs/master-sindico/requirements/{domain}.md`
2. Tela do produto: [[02-Jornadas/Inventario-Telas]]
3. Enum do domínio: [[04-Listas-Mestre/Enums-Consolidados]]
4. Verificar quebras conhecidas: [[regras-de-negocio/quebras-detectadas]]
5. Ver módulos relevantes em `03-Modulos/`

### "Quero entender a arquitetura"
1. [[01-Arquitetura/Stack-e-Modulos]] — visão completa
2. `backend/.kiro/steering/sdd-layers.md` — DDD/Clean por camada
3. `backend/AGENTS.md` — contrato do agente
4. [[regras-de-negocio/regras-canonicas]] §5 + §8 — T1-T15 + middleware BeyondCorp

### "Quero entender as regras de negócio"
1. [[regras-de-negocio/regras-canonicas]] — 10 regras + 12 decisões
2. [[03-Modulos/Connect-Me-4-direcoes]] — fluxo unidirecional + LGPD
3. [[03-Modulos/Compliance-Detalhado]] — C1-C11 + gatilhos
4. [[03-Modulos/Assembleia-Live-Funcional]] — Live Assembly + telão + procurações
5. [[03-Modulos/Vizinhanca-4-jornadas]] — promoções abertas vs exclusivas

### "Quero ver as inconsistências entre fontes"
[[regras-de-negocio/quebras-detectadas]] — 40 quebras catalogadas com fix proposto.

### "Quero entender SDD/GSD"
[[sdd-gsd-aplicado]] — síntese + 4 polishes recomendados.

### "Quero o handoff completo do agente 3 para o principal"
[[_handoffs/2026-04-22-relatorio-final-agente3]] — entrega consolidada.

---

## Mapa: Bounded Context Go ↔ Módulo do Produto ↔ Telas

| BC Go | Módulos do produto | Telas (IDs) |
|---|---|---|
| `identity` | Auth, Onboarding, ABAC | (não tem telas próprias — middleware) |
| `billing` | Planos, Trial, Soft-block | (paywall integrado) |
| `institutional` | Governança · Linha do Tempo · Plano Diretor · Eventos · Comunicados · Compliance | M1-M15, S1-S31, C1-C11 |
| `commercial` | Connect Me · Propostas · Deals · Execução · Vizinhança · Banco de Talentos | E1-E16, S18-S26, VZ1-VZ6, VS1-VS10, VE1-VE6, VM1-VM7, BT 01-11 |
| `content` | Vídeos · Marketing · Academia LMS · Ebooks | MK1-MK8, LMS1-LMS15 |
| `assembly` | Assembleia Live (5 etapas) | (~30+ telas, ver [[03-Modulos/Assembleia-Live-Funcional]]) |

---

## Status do projeto (snapshot 2026-04-22)

- **Backend Go**: Sprints 0-9 completos. Marco 1 (MVP contratual) = 2026-05-08. 33 features fechadas (F1-F33). 12 itens AUDIT abertos para Sprint 10 (0 críticos). PBT 4/6 regras críticas. Coverage ≥85% global.
- **Web (SolidJS)**: scaffold (auth placeholder em 6 apps).
- **App (Flutter)**: shell auth feature.
- **Specs**: backend canônico, web/app sem `.kiro` próprio (decisão D-030, 2026-04-22).
- **Sessão atual**: 3 agentes em paralelo — agente 1 implementa, agente 2 revisa, agente 3 (este) é analista cross-doc/regras.

---

## Inventário de quebras (atualizado)

| Severidade | Quantidade | Quem resolve |
|---|---|---|
| 🔴 Critical | 10 | Principal + João |
| 🟡 Medium | 15 | Principal (mudanças de doc) |
| 🟢 Low | 15 | Confirmações + arqueologia |
| **Total** | **40** | |

10 decisões pendentes com João — ver [[regras-de-negocio/quebras-detectadas]] §"Decisões pendentes".

---

## Convenções deste vault

- Português brasileiro para textos; identificadores e paths em inglês.
- Wiki-links `[[arquivo]]` para navegação Obsidian.
- Frontmatter YAML em todo arquivo novo (title, updated, type, source, related-spec).
- Sem emojis decorativos — apenas funcionais (🔴/🟡/🟢/✅ para status).
- Datas no formato ISO `YYYY-MM-DD`.

---

## Estatísticas do vault

| Métrica | Valor |
|---|---|
| Arquivos originais lidos (PDFs + MDs + .ini + .txt) | 23/23 (100%) |
| Páginas de PDF processadas | 80+ |
| Notas criadas pelo analista | 8 (este MOC + regras-canonicas + quebras-detectadas + sdd-gsd-aplicado + Inventario-Telas + Onboarding-por-Persona + Assembleia-Live-Funcional + Compliance-Detalhado + Stack-e-Modulos + Connect-Me-4-direcoes + Vizinhanca-4-jornadas + Enums-Consolidados = 12 notas) |
| Telas catalogadas | 136+ |
| Enums catalogados | 90+ (em 19 categorias) |
| Quebras de regra detectadas | 40 |
| Decisões pendentes com João | 10 |
| Tamanho médio das notas | ~7-15KB cada (densas, com refs cruzadas) |
