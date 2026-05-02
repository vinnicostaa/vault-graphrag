---
title: MOC — 40-templates
type: moc
tags:
  - moc
  - templates
created: 2026-04-22T00:00:00.000Z
updated: '2026-04-24'
---

# 40-templates — Templates replicáveis

Arquivos que se copiam pra **qualquer projeto novo**, com placeholders `<...>` pra preencher. Núcleo da proposta do automation-agents: "ferramenta genérica que gera vault por projeto".

## Templates disponíveis

- [[CLAUDE_md_template]] — template **canônico** de `CLAUDE.md` para novos projetos em `50-projects/<slug>/`. Usado pelo [[../30-agents/playbooks/onboarding-novo-projeto]]
- [[AGENTS_SPEC]] — template de `AGENTS.md` (coding guidelines + antipatterns catalogados + padrões cross-módulo)
- [[CLAUDE_SPEC]] — template de `CLAUDE.md` focado em Claude Code
- [[README_SPEC]] — template de `README.md` por tipo de projeto (monorepo, backend, frontend, mobile)
- [[state-template]] — STATE.md (D-### e bloqueadores)
- [[audit-template]] — AUDIT.md (A-### priorizados)
- [[session-charter-template]] — briefing de sessão
- [[client-vault-mapping]] — mapa de vault Obsidian externo do cliente
- [[claude-code-hook-read-context]] — hook Claude Code pra leitura de contexto

## Project scaffold (instanciável)

- [[project-scaffold/_moc]] — esqueleto completo de projeto (00-product até 12-docs) pronto para copiar via [[../30-agents/playbooks/onboarding-novo-projeto]]

## Pipeline multi-agente (runtime)

- [[pipeline/_moc]] — templates do pipeline Planner / Worker / Reviewer (D-vault-015.a Fase 1 — D-vault-015.b/c pendentes)
  - [[pipeline/plan-template]] — briefing + plan do Planner
  - [[pipeline/output-template]] — output do Worker

## Como usar um template

1. **Via playbook** (recomendado): `onboarding-novo-projeto` dispara scaffold automaticamente
2. **Manual**:
   - Copiar o arquivo/scaffold pro repo-alvo
   - Grepar por `<...>` / `<TODO>` e substituir cada placeholder com valor real
   - Rodar o checklist de qualidade de cada SPEC
   - Registrar D-000 de kick-off em STATE.md

## Vizinhos no grafo

- [[../_index]] — raiz
- [[../10-knowledge/_moc]] — teoria que os SPECs aplicam
- [[../20-stacks/_moc]] — templates específicos por stack
- [[../50-projects/_moc]] — inventário vivo de projetos instanciados
