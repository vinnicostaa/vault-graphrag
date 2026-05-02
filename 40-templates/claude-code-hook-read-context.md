---
title: Hook template — "Read Context Before Task"
type: template
tags: [template, claude-code, hook, sdd]
source: .kiro/hooks/read-context-before-task.kiro.hook
created: 2026-04-22
---

# Hook template — Read Context Before Task

Hook Claude Code que **força o agente a ler contexto do módulo antes de implementar uma task**. Instala como `<projeto>/.kiro/hooks/read-context-before-task.kiro.hook`.

## Arquivo

```json
{
  "enabled": true,
  "name": "Read Module Context Before Task",
  "description": "Antes de executar uma spec task, lê os arquivos relevantes do módulo para garantir contexto correto antes de implementar",
  "version": "1",
  "when": {
    "type": "preTaskExecution"
  },
  "then": {
    "type": "askAgent",
    "prompt": "Antes de implementar esta task, leia os arquivos relevantes do módulo afetado em `<path-dos-módulos>/` para entender o que já existe. Verifique:\n1. Estrutura atual do módulo (domain, application, infrastructure)\n2. Interfaces já definidas (repositories, providers)\n3. Migrations existentes em `<migrations-path>/`\n4. Queries de codegen existentes no módulo\nSó então prossiga com a implementação seguindo a ordem: migration → codegen → domain → application → infrastructure → routes."
  }
}
```

## Parametrização

Trocar nos placeholders ao copiar pra projeto novo:

- `<path-dos-módulos>` — ex: `backend/internal/modules/`, `src/modules/`, `apps/api/src/modules/`
- `<migrations-path>` — ex: `backend/internal/shared/providers/database/migrations/`, `apps/api/drizzle/migrations/`

## Por que existe

Sem este hook, agentes (especialmente em sessão longa autônoma) tendem a:

- Implementar antes de ler código existente → duplicação
- Criar padrões divergentes de módulos irmãos → drift
- Esquecer de checar migrations já feitas → quebrar schema
- Re-criar interfaces que já existiam

O hook injeta um **preTaskExecution** que força a fase Discuss do SDD. Ver [[../10-knowledge/methodology/sdd-workflow]].

## Outros hooks úteis (a criar)

- **`audit-gate.kiro.hook`** — bloqueia task se `AUDIT.md` tem itens 🔴/🟡 abertos (Gate 0 do SDD)
- **`verify-before-commit.kiro.hook`** — roda build + lint + test antes de permitir `git commit`
- **`state-update-reminder.kiro.hook`** — avisa se sessão terminou sem atualizar STATE
- **`ship-readme-reminder.kiro.hook`** — avisa que README deve ser atualizado em mudança feature-level

## Ativação

Os hooks em `.kiro/hooks/*.hook` são lidos automaticamente pelo Claude Code se a feature estiver habilitada no cliente. Ver docs oficiais Claude Code sobre hooks (pode ter mudado de nome/formato em versão mais recente).

## Links

- [[_moc]]
- [[../10-knowledge/methodology/sdd-workflow]] — Fase Discuss (leitura de contexto)
- [[../30-agents/autonomous-execution]] — Gate 0 do AUDIT
- [[state-template]]
- [[audit-template]]
