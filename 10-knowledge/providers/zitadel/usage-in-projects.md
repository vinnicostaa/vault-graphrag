---
title: Zitadel — Usage in Projects
type: note
tags: [provider, zitadel, identity, oidc, usage-in-projects]
source: https://zitadel.com/docs
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [Zitadel Usage]
---

# Zitadel — Usage in Projects

Mapa de onde Zitadel é aplicado nos projetos deste vault. Cada linha aponta para o ponto de integração canônico no projeto; detalhes de implementação vivem lá, não aqui.

## Tabela de uso

| Projeto | Uso | Aplicado em |
|---|---|---|
| Master Síndico | OIDC (Authorization Code + PKCE) para perfis **Síndico**, **Morador** e **Empresa**; ABAC com roles + plan_tier em custom claims; 1-device-per-account enforced via `Action` + tabela `user_devices`; machine user com JWT Profile para jobs de billing/integração | [[../../../50-projects/master-sindico/08-security/BEYOND_CORP]] |

## Convenções cruzadas

- Toda integração começa revisitando [[integration-guide]] e validando o checklist lá antes de abrir código.
- Decisão de cloud vs self-hosted é registrada como `D-###` no `STATE.md` do projeto, **não** nesta nota.
- Claims custom específicas do produto (`plan_tier`, `tenant_id`, `device_id`) são definidas no `Action` do Zitadel e documentadas no `08-security/` do projeto — esta nota apenas lista que existem.
- Rotação de chaves (machine user, `private_key_jwt`) é tarefa recorrente do projeto; entra em `05-tasks/` com cadência trimestral.

## Como adicionar um projeto novo aqui

1. Criar linha na tabela com `Projeto | Uso | Aplicado em`.
2. `Aplicado em` aponta para o arquivo de segurança do projeto (`08-security/BEYOND_CORP.md` ou equivalente).
3. Sem duplicar regra de negócio — manter descrição curta (≤ 2 linhas por projeto).
4. Registrar `D-###` no `STATE.md` do projeto quando a decisão de adotar Zitadel é tomada.

## Vizinhos no grafo

- [[_moc]]
- [[overview]]
- [[integration-guide]]
- [[patterns]]
- [[antipatterns]]
- [[../_moc]]
- [[../../security/beyond-corp]]
- [[../../security/_moc]]
