---
title: CLAUDE.md — vault automation-agents
type: agent-contract
tags:
  - claude
  - agent
  - vault-root
  - contract
created: 2026-04-22T00:00:00.000Z
updated: '2026-04-24'
---

# CLAUDE.md — Contrato do Agente no Vault `automation-agents`

Este arquivo é **carregado em toda sessão** que opera neste vault. Ele descreve **o que é esse vault**, **como o agente deve navegar**, **o que ele deve fazer antes de afirmar algo** e **onde escrever novas informações**.

> **Regra zero**: se você (agente) está em dúvida sobre qualquer coisa, pare, pesquise (ciclo §4), dual-check (§5) e só aí proceda. Nunca invente. Nunca assuma. Sempre linke a fonte.

> **Regra zero-B (sincronização)**: este vault tem um clone em filesystem que NÃO sincroniza automaticamente. Toda mutação vai via MCP Obsidian (`mcp__obsidian__*`). Edit/Write direto no filesystem do clone é proibido.

---

## 1. O que é este vault

Vault Obsidian dedicado ao projeto **automation-agents** — ferramenta multi-agente que usa o vault como **cérebro persistente** dos agentes Planner / Worker / Dual Reviewer / Orchestrator.

O vault serve **três públicos simultaneamente**:

| Público | Uso primário | Pasta-âncora |
|---|---|---|
| **Agentes LLM** (Planner/Worker/Reviewer) | Contexto durável entre sessões, contratos operacionais, memórias do projeto | `30-agents/`, `50-projects/<projeto>/CLAUDE.md` |
| **Dev humano** (operador) | Decisões, specs, roadmap, dívidas, auditorias | `50-projects/<projeto>/STATE.md`, `AUDIT.md`, `SESSION_CHARTER.md` |
| **Projetos concretos** (Master Síndico hoje, outros amanhã) | Specs vivas, cronogramas, execução, segurança | `50-projects/<projeto>/` |

**Invariante-chave**: o vault é **knowledge in markdown**. Nada de binários, nada de `.docx`, nada de `.canvas` crus — tudo destilado em `.md` com frontmatter YAML canônico.

**Filosofia de separação conhecimento global ↔ projeto**:
- Tudo que é **regra de negócio** ou **linguagem ubíqua** do produto → `50-projects/<slug>/`
- Tudo que é **atemporal, público, open-source, reutilizável** (providers, patterns, arquitetura, métodos de execução) → `10-knowledge/` ou `20-stacks/` ou `30-agents/`
- Projeto **referencia** o global via `[[links]]` em vez de duplicar

---

## 2. Mapa do vault (estrutura Johnny.Decimal)

```
vault/
├── _index.md                       # índice raiz (MOC do vault)
├── CLAUDE.md                       # este arquivo
├── 00-meta/                        # governança do vault
│   ├── vault-guide.md
│   └── _moc.md
├── 10-knowledge/                   # conhecimento técnico atemporal
│   ├── architecture/               # Clean Arch, Hexagonal, MVC, Event-Driven
│   ├── patterns/                   # DDD, SOLID, CQRS, SAGA, ACID, Factory...
│   ├── principles/                 # Clean Code, Code Calisthenics, Do/Don't
│   ├── methodology/                # SDD, GSD, TDD/BDD, graph-rag, legacy-as-ref
│   ├── antipatterns/               # AP-### catálogo Fowler (code smells cross-linguagem)
│   ├── runtime-antipatterns/       # OP-### catálogo operacional (concorrência, idempotência, cache, segurança runtime)
│   ├── security/                   # BeyondCorp, OWASP, CVE workflow
│   ├── testing/                    # pirâmide, PBT, mutation, property-based
│   ├── database/                   # transações, isolation, sharding
│   ├── observability/              # logs, metrics, traces, SLOs
│   └── providers/                  # SaaS/infra: stripe, mux, zitadel, livekit, railway (novo 2026-04-23)
├── 20-stacks/                      # templates por stack
│   ├── go/ typescript/ rust/ flutter/ postgres/
├── 30-agents/                      # operação do sistema multi-agente
│   ├── orchestrator/ planner/ worker/ reviewer/
│   ├── playbooks/                  # research-loop, dual-check, dep-audit, cve-scan, onboarding
│   ├── skills/                     # skills Claude Code reutilizáveis
│   ├── _memory/ _session/ _inbox/  # buffers do sistema multi-agente
│   ├── autonomous-execution.md
│   ├── mcp-integration.md
│   └── claude-code-plugins.md
├── 40-templates/
├── 50-projects/master-sindico/
├── 60-sources/
└── 90-archive/
```

**Regra de profundidade**: máximo 3 níveis. `10-knowledge/patterns/cqrs.md` OK; `10-knowledge/patterns/read/query/cqrs.md` NÃO.

**Regra de nomes**: `kebab-case.md`. Sem datas no nome, sem prefixos numéricos (exceto nas pastas raiz).

**Regra de frontmatter canônico** (expandida 2026-04-27 — D-vault-026):

```yaml
---
title: <título humano>
type: <ver lista canônica abaixo>
tags: [topic-1, topic-2]
source: <origem se destilado de fora>
created: 2026-04-22
updated: 2026-04-23
aliases: [sinônimo]
---
```

**Lista canônica de `type`** (25 valores):
- **Conhecimento atemporal**: `concept`, `pattern`, `antipattern`, `principle`
- **Containers / agregadores**: `moc`, `template`, `summary`, `note`, `guide`
- **Projeto / produto**: `project`, `source`, `aggregate`, `requirement`, `enum`
- **Especificações**: `spec`, `sprint-spec`, `ui-spec`, `ui-screen`, `ui-catalog-journey`, `canvas`
- **Decisões / governança**: `adr`, `state`, `audit`, `roadmap`, `playbook`
- **Operação multi-agente**: `agent-contract`

**Diretrizes**:
- `spec` é guarda-chuva — preferir `sprint-spec` / `ui-spec` / `aggregate`-as-spec quando cabível
- `audit-log` foi consolidado em `audit` (D-vault-026)
- `principle-applied` foi rebaixado a `note` com tag `principle-applied` (D-vault-026)
- One-offs (`feature-plan`, `task-list`, `gap`, `pendencias`, `traceability-matrix`, `sub-produto`, `design-system`, `feature-plan`, `ui-spec-index`, `steering`, `session-charter`) → `note`/`concept`/`moc`/`state` conforme semântica (D-vault-026)
- Tipos em `60-sources/` (`research-source`, `prompt`, `research-destilado`, `source-reference`, `index`, `execute`) são exceção pois zonas de raw/source mantêm tipologia da origem

**Regra de preenchimento lazy (`20-stacks/`)** — ver [[00-meta/STATE|D-vault-017]]:

- Pasta existe como contrato, mas **só é preenchida sob demanda**: quando 2+ projetos reais usarem a mesma stack **OU** quando um projeto descobrir padrão específico de linguagem que merece ficar global.
- Enquanto não houver demanda dupla, `20-stacks/<stack>/_moc.md` é um stub válido com status `⏳ não instrumentado` + referência a quem originou.
- Padrões de stack descobertos em projeto único ficam em `50-projects/<p>/07-quality/PATTERNS.md` até promoção.
- Promoção requer: (a) segundo projeto evidente, (b) research-loop (§4), (c) D-vault-### registrando motivo.

---

## 3. Fluxo canônico: onde o agente escreve cada coisa

| Tipo de informação | Vai em | Exemplo |
|---|---|---|
| Conceito técnico atemporal (pattern, princípio, método) | `10-knowledge/<subpasta>/` | "O que é CQRS" |
| Provider SaaS/infra (Stripe, Mux, Zitadel, etc.) | `10-knowledge/providers/<provider>/` | Stripe Connect patterns |
| Convenção por stack (idiomatic Go, idiomatic TS) | `20-stacks/<stack>/` | "Go error handling" |
| Como operar um agente específico | `30-agents/<role>/` | Worker charter |
| Playbook repetível pro agente | `30-agents/playbooks/` | Research loop |
| Template replicável | `40-templates/` | CLAUDE.md.template |
| Decisão viva de um projeto (D-###) | `50-projects/<p>/STATE.md` | "Decidimos Stripe vs Pagar.me" |
| Bug descoberto em review (A-###) | `50-projects/<p>/11-audit/AUDIT.md` | "TOCTOU em Vote" |
| Contrato da sessão (charter) | `50-projects/<p>/SESSION_CHARTER.md` | Ordens do usuário |
| Requisito funcional do projeto | `50-projects/<p>/04-requirements/functional/` | identity.md |
| Task individual | `50-projects/<p>/05-tasks/by-module/<módulo>.md` ou `by-sprint/sprint-N.md` | Sprint 1 |
| Artefato do pipeline (plan/output/review/verdict) | `30-agents/<role>/<sub>/<task-id>.md` — estado oficial | `planner/plans/42.md` |
| Ponteiro de fila (pipeline) | `30-agents/_inbox/<next-role>/<task-id>.md` — só frontmatter + link | `_inbox/worker/42.md` |
| Template replicável do pipeline | `40-templates/pipeline/` | `plan-template.md`, `output-template.md` |
| Padrão aplicado no projeto | `50-projects/<p>/07-quality/PATTERNS.md` | Quando usar Saga aqui |
| Ingestão de fonte externa | `60-sources/<fonte>/` + link em `10-knowledge/` | Spec Kit |

---

## 4. Research-Loop obrigatório (antes de afirmar/decidir)

**Nunca tratar nada como verdade absoluta até completar este ciclo** quando a decisão envolver arquitetura, segurança, escolha de lib, pattern ou contrato público. Playbook em [[30-agents/playbooks/research-loop]].

```
1. Analisar o que o vault já tem
   └── grep/read em 10-knowledge/, 50-projects/<p>/, 60-sources/
       STATE.md D-### anteriores podem já ter decidido isto

2. Pesquisar docs OFICIAIS e ATUALIZADAS
   └── Context7 MCP (preferido) → resolve-library-id → get-library-docs
   └── Provider MCP se existe (ex: mcp.stripe.com) — tem precedência sobre WebFetch
   └── WebFetch em site oficial (não tutorial aleatório)
   └── Anotar versão/data consultada

3. Ver como grandes empresas fazem
   └── Engineering blogs (Google/Meta/Netflix/Stripe/Shopify/Uber/Airbnb)
   └── Casos públicos (postmortems, ADRs publicados, conference talks)
   └── Repos OSS de referência

4. Analisar patterns recomendados pro contexto
   └── Mapear em 10-knowledge/patterns/ e linkar
   └── Se não existe, criar nota nova destilando

5. Dual-check (§5) — 2o agente revisa antes de tratar como verdade

6. Registrar a decisão
   └── STATE.md como D-### com: alternativas consideradas, fontes consultadas, rationale, data
   └── Se impacto arquitetural: ADR em 02-architecture/adr/ do projeto
```

**Saída obrigatória** em toda decisão não-trivial: D-### em STATE + (opcional) ADR + links pras fontes.

---

## 5. Dual-check (2o par de olhos obrigatório)

Playbook em [[30-agents/playbooks/dual-check]].

**Gatilhos** (situações que obrigam dual-check):

- Escolha de arquitetura ou lib crítica
- Contrato público (API, webhook, evento de domínio)
- Mudança de schema de banco
- Mudança de regra de segurança (ABAC, auth, cookies, CORS)
- Mudança de regra de negócio que passa por produção
- Conflito entre 2+ fontes consultadas
- Modo autônomo sem supervisão

**Formato**:

```
Agente-A (propositor):
  • Proposta: <decisão>
  • Fontes consultadas: <URLs + versões>
  • Alternativas consideradas: <N>
  • Rationale: <por quê essa>

Agente-B (revisor, contexto limpo — não viu o raciocínio de A):
  • Re-pesquisa as mesmas fontes
  • Avalia: concorda / discorda / ressalva
  • Se discorda: proposta alternativa com rationale

Consolidação:
  • Se A == B: decisão registrada
  • Se A ≠ B: terceiro ciclo com dados adicionais, usuário notificado se não resolver
```

Executar via `Agent` tool com `subagent_type=general-purpose` ou `Plan`, preservando contexto limpo (não reaproveitar conversa do agente-A).

---

## 6. Anti-alucinação: nunca assuma

O agente **nunca**:

- Inventa APIs, flags, funções, comandos que não viu na doc oficial
- Cita versão de lib sem checar `go.mod`/`package.json`/`Cargo.toml`
- Afirma "isso é padrão da indústria" sem linkar 2+ fontes
- Diz "funciona" sem rodar o teste / build / lint
- Preenche lacuna com "provavelmente" — vai e checa

Quando não sabe, **escreve explicitamente**:

> ⚠️ PENDÊNCIA: este ponto não foi validado em doc oficial. Bloqueia até pesquisa.

---

## 7. MCPs disponíveis e quando usar

Ver [[30-agents/mcp-integration]].

| MCP | Pra que | Quando usar |
|---|---|---|
| `obsidian` | Ler/escrever notas do vault (ÚNICO caminho pra mutação) | Sempre — qualquer leitura/escrita no vault passa por aqui |
| `stripe` (mcp.stripe.com) | API Stripe em tempo real | Antes de WebFetch ou código em integração Stripe |
| `context7` (plugin) | Doc oficial atualizada de libs | Sempre antes de usar lib externa |
| `postgres` (crystaldba) | Inspeção read-only de schema | Quando modelando/auditando DB real |
| `github` | Issues, PRs, comentários | Workflow de entrega |
| `aws-docs` (awslabs) | Doc oficial AWS | Infra decisions |
| `railway` / `figma` | Plataformas específicas | Só se o projeto usa |

---

## 8. Padrões de engenharia inegociáveis (constituição técnica)

Aplicam-se a **todos os projetos** neste vault, salvo exceção registrada em STATE D-###.

1. **DDD** — bounded contexts explícitos; ubiquitous language versionada; domain services sem deps de infra
2. **Clean Architecture** — dependências apontam para dentro (infra → app → domain); domínio **não conhece** framework
3. **CQRS** — command ≠ query; arquivos separados; handlers específicos
4. **SOLID** — especialmente SRP (use case faz uma coisa) e DIP (depende de interface)
5. **TDD/BDD** — testes antes ou ao lado do código; coverage thresholds em `.coverage.yml` (domain 95%, app 90%, infra 85%, http 75%)
6. **SDD** — ciclo 5-fase (Discuss → Plan → Execute → Verify → Ship); `<verify>` declarativo como contrato; spec vive em `specs/`
7. **GSD** — Get Shit Done como ergonomia: tasks pequenas, shipping incremental, nada de big-bang
8. **ACID + SAGA** — UoW dentro do mesmo bounded context; Saga com compensação quando cruza serviço externo
9. **Code Calisthenics** (adaptado): um nível de indent por método; sem `else` (guard clauses/early return); pequenos objetos; sem getters/setters sem comportamento; no caso Go: early return sempre; sem `else` bloco
10. **BeyondCorp adaptado** — zero trust, ABAC, tenant isolation estrito, device fingerprint, PII nunca em logs
11. **OpenAPI 3.1** — contrato gerado ou escrito antes do handler; mantido em `specs/contracts/openapi.yaml`
12. **No try/catch defensivo** — só captura quando tem ação útil (compensação, log com contexto, transformação de erro). Caso padrão: deixar propagar. Em Go: `if err != nil { return nil, err }` sem panic.
13. **No if/else hell** — extrair strategies, tabelas de dispatch, polymorphism; aninhamento ≥ 3 níveis é sinal de extração pendente
14. **Sem abstração prematura** — YAGNI. Abstrair quando há 2+ implementações reais, não antes

---

## 9. Fluxo de início de sessão (onboarding do agente)

**Passos obrigatórios** ao entrar em nova sessão sobre um projeto:

1. Ler `vault/CLAUDE.md` (este arquivo) — contrato do vault
2. Ler `vault/50-projects/<projeto>/CLAUDE.md` — contrato do projeto
3. Ler `vault/50-projects/<projeto>/STATE.md` — decisões vivas e dívidas
4. Ler `vault/50-projects/<projeto>/11-audit/AUDIT.md` — findings abertos (🔴🟡 bloqueiam task nova)
5. Ler `vault/50-projects/<projeto>/SESSION_CHARTER.md` — ordens da sessão corrente
6. Ler `vault/50-projects/<projeto>/05-tasks/by-sprint/sprint-<N>.md` — task atual

**Reportar em até 15 linhas**: sprint atual, próxima task, AUDIT 🔴🟡, D-### abertos, gates vermelhos, modo de operação.

**Se modo autônomo**: não pergunta — reporta e segue Gate 0 (AUDIT) → próxima task.

---

## 10. Fluxo de fim de sessão (checklist)

- [ ] STATE.md recebeu decisões novas (D-###)?
- [ ] AUDIT.md recebeu findings novos / fechados?
- [ ] SESSION_CHARTER.md registra o que foi feito?
- [ ] Notas criadas têm frontmatter + ≥ 2 `[[links]]`?
- [ ] MOCs impactados foram atualizados?
- [ ] Zero conteúdo específico de cliente em `10-knowledge/` (vai em `50-projects/`)?
- [ ] Testes passaram, coverage nos thresholds?
- [ ] Docs sincronizadas com código?

---

## 11. O que NÃO fazer neste vault

- ❌ Criar binário, PDF, docx, canvas cru — converter antes
- ❌ Duplicar conteúdo entre arquivos — linkar
- ❌ Copiar parágrafos de fonte externa — destilar em linguagem própria
- ❌ Nota solta sem links — ou linka 2+ ou não pertence ainda
- ❌ Conteúdo específico de cliente em `10-knowledge/` — vai em `50-projects/`
- ❌ Arquivo na raiz do vault que não seja `_index.md` ou `CLAUDE.md`
- ❌ Mudar decisão sem atualizar STATE
- ❌ Arquivo com conteúdo específico de MasterSindico fora de `50-projects/master-sindico/`
- ❌ Editar o clone filesystem direto (somente via MCP Obsidian)

---

## 12. Idioma

- Corpo de notas: **pt-BR**
- Frontmatter, `title` técnico, identifiers de código: **inglês**
- Docstrings de API pública bilíngue (EN em cima, pt-BR embaixo) — só em templates de código

---

## 13. Links essenciais

- [[_index]] — índice navegacional do vault
- [[00-meta/vault-guide]] — operação do dia-a-dia
- [[00-meta/STATE]] — **decisões vivas do vault global** (D-vault-###) — canal de governança estrutural
- [[00-meta/AUDIT]] — **findings estruturais do vault global** (A-vault-###) — gate 0 de sessão sobre knowledge/pipeline
- [[30-agents/autonomous-execution]] — modo autônomo e limites
- [[30-agents/mcp-integration]] — uso de MCPs
- [[30-agents/playbooks/research-loop]] — ciclo obrigatório de pesquisa
- [[30-agents/playbooks/dual-check]] — validação dupla
- [[30-agents/playbooks/community-summary]] — Graph-RAG Global Search (D-vault-003)
- [[30-agents/playbooks/gsd-resync]] — re-sincronização da gsd-library (D-vault-004)
- [[30-agents/gsd-library/_moc]] — biblioteca GSD destilada (33 cards + 6 famílias)
- [[10-knowledge/patterns/_moc]] — catálogo GoF completo (D-vault-001)
- [[10-knowledge/architecture/_moc]] — Clean Architecture + Canonical Five (D-vault-005)
- [[10-knowledge/antipatterns/_moc]] — catálogo AP-### Fowler (code smells clássicos)
- [[10-knowledge/runtime-antipatterns/_moc]] — catálogo OP-### operacional (D-vault-012)
- [[10-knowledge/providers/_moc]] — nós de providers SaaS/infra
- [[40-templates/pipeline/_moc]] — templates do pipeline multi-agente (D-vault-015.a)
- [[50-projects/_moc]] — inventário vivo de projetos (master-sindico, websocket-chat, ...)
- [[40-templates/CLAUDE_md_template]] — template pra novos projetos
