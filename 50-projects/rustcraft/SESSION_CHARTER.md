---
title: SESSION_CHARTER — rustcraft — normalizacao do vault
type: note
tags:
  - session
  - rustcraft
  - vault
  - governance
created: '2026-05-27'
updated: '2026-05-27'
---
# SESSION_CHARTER — rustcraft — normalizacao do vault

## Escopo da sessao

Analisar o vault de forma sistematica e minuciosa para entender como usa-lo corretamente no projeto `rustcraft`, corrigir o uso raso anterior e registrar audits reais no caminho canonico.

## Ordens do usuario

### Contexto importado do Zed

> "analise de forma sistematica e minuciosa, revise tudo, analise de forma de dupla checagem e trate meus inputs dos contextos como se tambem fosse pra voce e melhore, pois achei o agente muito raso e tambem use o MCP do obsidan para trabalhar"

### Correcao desta sessao

> "percebi que voce nao criou audits no vault, nao estudou ele a fundo para entender como trabalhar com ele, preciso que analise de forma sistematica e minuciosa o vault pra entender como usa-lo e melhora-lo"

## Principios inegociaveis para esta sessao

- Usar o MCP Obsidian para ler e escrever no vault.
- Ler o contrato raiz `[[../../CLAUDE]]` antes de assumir o funcionamento do vault.
- Comparar `rustcraft` com `[[../../40-templates/project-scaffold/_moc]]` e projetos existentes.
- Registrar findings em `[[11-audit/AUDIT]]`, nao apenas em nota solta.
- Manter o projeto tutor-first: o agente estrutura, pesquisa, revisa e decompoe; o usuario codifica salvo pedido explicito.

## Fontes de verdade consultadas

1. `[[../../CLAUDE]]` — contrato raiz do vault.
2. `[[../../_index]]` — indice raiz.
3. `[[../../40-templates/project-scaffold/_moc]]` — scaffold canonico.
4. `[[../../30-agents/playbooks/onboarding-novo-projeto]]` — onboarding obrigatorio.
5. `[[../../30-agents/playbooks/research-loop]]` — pesquisa obrigatoria antes de decidir.
6. `[[../../30-agents/playbooks/dual-check]]` — validacao por segundo olhar.
7. `[[../master-sindico/_moc]]` e `[[../master-sindico/11-audit/AUDIT]]` — exemplo completo.
8. `[[../websocket-chat/_moc]]` e `[[../websocket-chat/SESSION_CHARTER]]` — exemplo menor.

## Trabalho executado

- Mapeada a raiz do vault e a taxonomia Johnny.Decimal.
- Identificado que `rustcraft` estava como scaffold parcial.
- Criado caminho canonico `11-audit/` para audits.
- Criada esta `SESSION_CHARTER` para rastrear a ordem do usuario.
- Criado `STEERING.md` com regras especificas do projeto.
- Criados MOCs faltantes para aproximar `rustcraft` do scaffold 00-12.

## Links

- [[_moc]]
- [[STATE]]
- [[11-audit/AUDIT]]
- [[STEERING]]
- [[../../CLAUDE]]
