---
title: Playbook — Onboarding de Novo Projeto (handshake agente↔dev)
type: playbook
tags: [playbook, onboarding, setup, new-project]
created: 2026-04-22
---

# Playbook — Onboarding de Novo Projeto

Ritual **obrigatório** quando o dev inicia um novo projeto neste vault. Preenche as lacunas de contexto que o agente precisa pra operar com autonomia sem alucinar.

**Saída**: novo `vault/50-projects/<slug>/` totalmente instanciado com CLAUDE.md, MOC, STATE, AUDIT, SESSION_CHARTER, pastas 00-12 esqueleto, e D-000 "kick-off" registrado em STATE.

## Princípio

O agente **não** começa a escrever código/arquitetura antes desse onboarding. Tentar pular = alucinação garantida. É barato fazer (30 min estruturados) e paga pra sempre.

## Quando disparar

- `/loop onboarding <slug-do-projeto>` ou comando equivalente
- Dev cria pasta vazia em `50-projects/<slug>/` e pede "setup"
- Dev diz "quero começar um projeto novo"

## Etapas

### Etapa 1 — Entrevista estruturada com o dev

Agente **faz** as perguntas abaixo. Se dev disse "depois eu respondo", agente **para** — não preenche com suposição. Ordem importa: as primeiras respostas guiam as próximas.

#### 1.1 Visão (o que é?)
- Nome do projeto: <slug kebab-case>?
- Em 2 frases: o que faz e pra quem?
- Problema que resolve (1 frase)?
- Por quê agora (timing)?
- Como saberemos que deu certo em 6 meses (1-3 métricas north-star)?

#### 1.2 Domínio e escopo
- Personas principais (quem usa)?
- Fluxos principais (top 5 user stories)?
- O que **não** é escopo (guard contra scope-creep)?
- Há projeto predecessor / legado a referenciar? Onde?

#### 1.3 Stack e plataformas
- Sub-projetos esperados (backend? web? mobile? CLI? ML?)
- Linguagem/framework por sub-projeto (se já decidido) — ou "a decidir" (entra em D-001)
- Hospedagem (Railway? AWS? Vercel? on-prem?)
- Banco de dados (Postgres? MySQL? Mongo?)
- Auth (Zitadel? Auth0? Clerk? próprio?)
- Principais integrações externas (Stripe? Resend? Twilio? Mux? etc.)

#### 1.4 Time e modo de operação
- É solo ou tem time?
- Vai usar agente em modo autônomo? Que limites?
- Existe deadline/cronograma-âncora?
- Orçamento de tempo/pessoas?

#### 1.5 Padrões e não-negociáveis
- Há preferência de arquitetura (Clean? Hexagonal? MVC?)
- Há padrões que o dev **proíbe** (no `else`? sem `any`? sem mock em integration?)
- Requisitos regulatórios (LGPD? GDPR? HIPAA? PCI?)
- Idioma de doc e código (pt-BR default)

#### 1.6 Qualidade e segurança
- Qual nível de cobertura mínima aceitável?
- Security posture (zero-trust? BeyondCorp? VPN? rede aberta?)
- SLA/SLO se aplicável (p99 latency? uptime?)
- Estratégia de deploy (blue-green? canary? rolling?)

Se dev não sabe, agente **não decide por conta própria** — registra como decisão em aberto (D-### "a decidir", sprint alvo).

### Etapa 2 — Instanciar scaffold

Copiar `vault/40-templates/project-scaffold/` para `vault/50-projects/<slug>/`:

```
50-projects/<slug>/
├── CLAUDE.md                       # contrato do projeto (preenchido com respostas)
├── _moc.md                         # MOC do projeto
├── overview.md                     # destilado das respostas
├── STATE.md                        # D-000: kick-off | D-001..: decisões pendentes
├── SESSION_CHARTER.md              # ordens da sessão de kick-off
├── STEERING.md                     # não-negociáveis declarados pelo dev
├── ROADMAP.md                      # cronograma macro (milestones)
├── DoR-DoD.md                      # Definition of Ready / Done
├── 00-product/
├── 01-domain/
├── 02-architecture/
├── 03-subprojects/
├── 04-requirements/
├── 05-tasks/
├── 06-execution/
├── 07-quality/
├── 08-security/
├── 09-testing/
├── 10-schedule/
├── 11-audit/
│   └── AUDIT.md
├── 12-docs/
└── _handoffs/
```

Cada pasta começa com `_moc.md` que lista o que **vai** morar ali.

### Etapa 3 — Preencher CLAUDE.md do projeto

Usar `vault/40-templates/CLAUDE.md.template` como base. Preencher com respostas da entrevista. Campos críticos:

- Stack declarada (ou "a decidir" com D-###)
- Padrões obrigatórios / proibidos
- MCPs relevantes (ex: stripe se billing, figma se UI-heavy)
- Estrutura de sub-projetos declarada
- Research-loop e dual-check mantidos como padrão

### Etapa 4 — Registrar decisões pendentes

Em `STATE.md`:

```markdown
### D-000 — Kick-off do projeto <slug> — 2026-MM-DD
Status: ✅ Concluído
Descrição: Onboarding com dev executado via playbook. Respostas arquivadas em SESSION_CHARTER.

### D-001 — <primeira decisão em aberto> — 2026-MM-DD
Status: ⏳ Em aberto
Contexto: <...>
Alternativas: ...
Sprint alvo pra decidir: Sprint 0

### D-002 — <segunda decisão>
...
```

### Etapa 5 — Aplicar Research-Loop em decisões críticas não resolvidas

Para cada D-### não óbvio do kick-off (stack, hospedagem, arquitetura, pattern principal), agente dispara [[research-loop]] e [[dual-check]] **antes de Sprint 1 começar**.

### Etapa 6 — Sprint 0 (preparação) cria base

Tasks canônicas do Sprint 0:

- [ ] Setup repo + CI mínimo (lint + test)
- [ ] Setup branch strategy
- [ ] Instalar ferramentas de dep audit do stack
- [ ] Configurar `.claude/settings.json` local com plugins e permissões apropriadas
- [ ] Primeiros ADRs (em `02-architecture/adr/`)
- [ ] `08-security/threat-model.md` inicial
- [ ] `09-testing/test-strategy.md`
- [ ] `06-execution/STEPS.md` com caminho zero→prod
- [ ] `10-schedule/cronograma-macro.md` com milestones
- [ ] `12-docs/README.md` stub

### Etapa 7 — Handshake final

Agente apresenta resumo de 15 linhas ao dev:

```
Projeto: <slug>
Visão: <1 linha>
Sub-projetos: <lista>
Stack confirmada: <lista>
Decisões abertas (precisam pesquisa): D-001..D-00N
Principais riscos detectados: <lista curta>
Primeiro milestone: <data>
Modo de operação acordado: <autônomo/supervisionado>
Próxima ação do agente: <research sobre D-001>
Próxima ação do dev: <ex: confirmar domínio DNS>
```

Dev aprova ou ajusta. Aprovado → onboarding fechado, Sprint 0 liberado.

## Checklist do fim do onboarding

- [ ] Pasta `50-projects/<slug>/` instanciada completa (pastas 00-12 com MOCs)
- [ ] `CLAUDE.md` preenchido (sem placeholder `<TODO>` não justificado)
- [ ] `STATE.md` com D-000 fechado e D-### para cada pendência
- [ ] `SESSION_CHARTER.md` com o diálogo da entrevista
- [ ] `STEERING.md` com não-negociáveis
- [ ] `ROADMAP.md` com milestones preliminares
- [ ] `DoR-DoD.md` com critérios mínimos
- [ ] `_index.md` raiz do vault atualizado (se novo projeto merece entrar no índice)
- [ ] `CLAUDE.md` raiz do vault **não** precisa mudar
- [ ] Research-loop disparado pras decisões críticas em aberto

## Anti-padrões a evitar

- ❌ Agente "adivinha" stack porque achou exemplo parecido — registra como D-### em aberto
- ❌ Sprint 1 começa antes do Sprint 0 fechar
- ❌ Pular entrevista porque "já conversamos"
- ❌ Copiar arquitetura de outro projeto sem validar contexto
- ❌ Instanciar scaffold **vazio** (sem preencher nada) e chamar de onboarding

## Links

- [[_moc]]
- [[research-loop]]
- [[dual-check]]
- [[../../40-templates/CLAUDE_md_template]]
- [[../../40-templates/project-scaffold/_moc]] *(será criado)*
- [[../../CLAUDE]]
