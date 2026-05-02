---
title: MOC — Claude Code Skills
type: moc
tags: [moc, skills, claude-code, agents]
created: 2026-04-22
---

# Claude Code Skills — catálogo genérico

> **Escopo desta pasta (distinção hub-doc vs skills executáveis)** — resolve [[../../00-meta/AUDIT|A-vault-012]].
>
> - **Esta pasta** (`30-agents/skills/`) é **hub de documentação** descrevendo padrões de skills. **Nada aqui é executado** pelo Claude Code.
> - **Skills executáveis reais** vivem em:
>   - `~/.claude/skills/*.md` — skills globais do usuário
>   - `<projeto>/.claude/skills/*.md` — skills do projeto (invocadas via `/<nome>` no Claude Code)
>   - Plugins Claude Code (pode empacotar múltiplas skills)
> - Este MOC cataloga **padrões genéricos destiláveis** (como escrever uma skill bem). Instâncias concretas em `50-projects/<proj>/.claude/skills/`.
> - Agente que busca "como funciona a skill X" consulta esta pasta; agente que **invoca** uma skill depende do Claude Code runtime carregar do path correto.

Skills Claude Code são **slash commands customizados** (`/<nome>`) definidos em `.claude/skills/*.md` no projeto. Cada skill é um markdown com instruções que o agente obedece quando o comando é invocado.

Este MOC cataloga **padrões genéricos** de skills úteis em qualquer projeto que adota SDD. Instâncias concretas (adaptadas ao stack) estão em `50-projects/<proj>/.claude/skills/`.

## Skills genéricas (templates destiláveis)

### `/task-executor` — executor de task SDD
Implementa ciclo 5-fase [[../../10-knowledge/methodology/sdd-workflow|Discuss → Plan → Execute → Verify → Ship]] + Gate 0 (AUDIT bloqueia task nova).

**Responsabilidades**:
1. Gate 0 — abre `AUDIT.md` no início de qualquer sessão. Se há `🔴/🟡`, trata como micro-task antes de pegar nova.
2. Fase Discuss — lê task + requirements + design + antipatterns + STATE + AUDIT + steering relevante + código existente. Se ambíguo: **pausa** e atualiza spec antes de codar.
3. Fase Plan — declara arquivos (criar/modificar/não-tocar), docs consultadas via Context7, `<verify>` (testes que vão passar, coverage alvo).
4. Fase Execute — segue ordem canônica (migration → codegen → domain → application → infra → routes → wire-up → testes).
5. Fase Verify — build + vet + test + coverage + security scanner + vuln check.
6. Fase Ship — commit pt-BR imperativo + README update (se feature-level) + PR com Release Notes.

**Integra com**: [[../autonomous-execution]] (modo não-supervisionado), [[sprint-auditor|`/sprint-auditor`]] (fim de sprint), [[../mcp-integration]] (Context7 obrigatório).

Instância MS: [[../../50-projects/master-sindico/.claude/skills/task-executor]]

### `/dual-validate` — validação dual-agent Opus + Opus

Skill pra decisões de **alto impacto** em modo autônomo. Implementa pattern do [[../autonomous-execution#Fase 2 — Dual-agent validation]] em formato operacional.

**Quando usar** (alto impacto):
- Escolha arquitetural cross-bounded-context
- Padrão de segurança (auth, crypto, webhook, rate limit)
- Schema novo / migration irreversível
- Contrato público novo ou alterado
- Integração SDK externo (como modelar interface)
- Interpretação ambígua de requisito

**Quando NÃO usar** (baixo/médio):
- Nome de variável, ordem de args
- Escolha entre 2 impls equivalentes
- Refactor interno a arquivo
- Finding low/medium em AUDIT
- Bug óbvio com fix único

**Protocolo**:
1. **Briefing compartilhado** — problema + contexto (arquivo:linha) + restrições duras + perguntas numeradas + não-escopo
2. **Spawn paralelo** — 1 Analista + 1 Revisor, **ambos Opus**, **mesmo** briefing, **nunca** sequencial
3. **Formato de saída obrigatório**: Recomendação + Fundamento por fonte (5 fontes) + Alternativas rejeitadas + Riscos + Confiança
4. **Regras do Revisor**: listar hipóteses descartadas, explicitar divergências, **confiança do Revisor pesa mais** no tiebreak
5. **Orquestrador resolve**:
   - Convergência → executa + registra D-0XX
   - Divergência → identifica ponto específico + 1 pesquisa adicional + tiebreak (consistência > modernidade) + registra **ambos** pontos de vista em STATE
   - Ambos com baixa confiança → pausa + `spec-gap` em AUDIT severidade high

**Budget**: 10-20min wall clock. Se 3+ dual-validates no mesmo sprint → sinal de spec insuficiente.

Instância MS: [[../../50-projects/master-sindico/.claude/skills/dual-validate]]

### `/sprint-auditor` — auditoria pós-sprint

Skill acionada ao fim de sprint (ou início do próximo) pra revisar sistematicamente o entregue, abrir findings em AUDIT, e **bloquear** `task-executor` enquanto houver `🔴/🟡`.

**Ciclo virtuoso**:
```
task-executor (implementa) → fim do sprint
                             ↓
                       sprint-auditor (revisa, abre A-XXX)
                             ↓
                       task-executor (resolve items, marca ✅)
                             ↓
                       sprint-auditor (valida fixes + move ✅ → Histórico)
                             ↓
                       task-executor (próximo sprint)
```

**Protocolo** (5 fases):
- **A — Levantamento**: ler AUDIT + STATE + `git log --since=X` + `git diff --stat` + `tasks.md` do sprint
- **B — Revisão sistemática por módulo tocado** (ordem importa; vermelho short-circuita):
  1. Dependency rule
  2. Error handling (`_ = fn()`, `errors.Is/As`)
  3. Padrões da linguagem (early return, tipos corretos, context)
  4. DDD / Clean Arch
  5. Segurança (webhook HMAC, validação server-side, PII mascarado)
  6. Antipatterns do catálogo (AP-XXX — ver [[../../10-knowledge/antipatterns/_moc]])
  7. Threshold de coverage
  8. Spec alignment
  9. Cross-module isolation
- **C — Materializar findings** em AUDIT com ID `A-XXX` (nunca reutilizar), severidade, raiz (não sintoma), fix proposto em 1 frase, status 🔴
- **D — Validação (leitura só)**: rodar gates; gate vermelho vira finding `critical`
- **E — Síntese de relatório** — commits analisados, arquivos revisados, findings por severidade, bloqueadores pro próximo sprint

**Regras duras**:
- **Nunca implementa** fix — aponta, não corrige
- **Nunca apaga** entrada histórica
- **Nunca duplica** item já em STATE como DT-XXX
- **Nunca libera** task-executor com gate vermelho

**Delegação**: módulos grandes (>10 arquivos) → spawn `feature-dev:code-reviewer` subagent por camada, retorna só resumo (não polui contexto do orquestrador).

Instância MS: [[../../50-projects/master-sindico/.claude/skills/sprint-auditor]]

## Skills stack-specific (não-genéricas)

Skills altamente acopladas a um stack — ficam em `50-projects/<proj>/.claude/skills/` da instância + `20-stacks/<stack>/` se padrão genérico existir.

### `/go-review` (Go backend)
Revisão de código Go por prioridade (Dependency Rule → Errors → Padrões Go → DDD → Segurança → Antipatterns). Curta-circuita em crítico.

Instância MS: [[../../50-projects/master-sindico/.claude/skills/go-review]]

### `/migration-writer` (goose + PostgreSQL)
Cria migration goose com numeração por módulo, CHECK constraints, índices padrão (soft delete, tenant, FK, status).

Instância MS: [[../../50-projects/master-sindico/.claude/skills/migration-writer]]

### `/sqlc-writer` (sqlc + PostgreSQL)
Escreve query sqlc com colunas listadas, `deleted_at IS NULL` em soft delete, overrides de tipo correto.

Instância MS: [[../../50-projects/master-sindico/.claude/skills/sqlc-writer]]

## Padrão de arquivo skill

Estrutura canônica de `.claude/skills/<nome>.md`:

```markdown
# Skill: <Nome> (<escopo>)

<Descrição em 1-2 linhas: quando invocar, para que serve.>

## Referências obrigatórias

- Steering, outras skills, docs que o agente **deve** consultar.

## Gatilhos de invocação

1. <Situação A>
2. <Situação B>
3. Quando usuário pede `/<nome>` explicitamente.

## Protocolo de execução

### Fase A — <nome>
1. Passo 1 com arquivo:linha específico quando aplicável
2. Passo 2

### Fase B — <nome>
...

## Regras duras

- ✅ Sempre...
- ❌ Nunca...

## O que esta skill NÃO faz

- <delimitação de escopo>
```

## Como copiar skills pra projeto novo

1. Copiar skill de `50-projects/<outro-proj>/.claude/skills/<nome>.md` pra `<novo-proj>/.claude/skills/`
2. Grepar por referências específicas do projeto anterior (paths, stacks, nomes de módulos)
3. Substituir por placeholders ou valores reais do novo projeto
4. Testar invocação (`/<nome>` no Claude Code)
5. Registrar decisão de adoção em `<novo-proj>/STATE.md` como D-0XX

## Links

- [[../_moc]] — agentes
- [[../autonomous-execution]] — modo não-supervisionado + protocolo de decisão
- [[../mcp-integration]] — Context7 obrigatório em Fase 1 do protocolo
- [[../claude-code-plugins]] — plugins que complementam skills
- [[../../10-knowledge/methodology/sdd-workflow]] — ciclo 5-fase
- [[../../10-knowledge/antipatterns/_moc]] — catálogo AP-XXX referenciado em reviews
- [[../../50-projects/master-sindico/.claude/skills/_moc]] *(após mv)* — instâncias concretas
