---
title: Dependency Audit — Master Síndico (referência)
type: note
tags: [master-sindico, reference, dependencies,security,playbook]
project: master-sindico
updated: 2026-04-23
---

# Dependency Audit — no Master Síndico

Este projeto **segue** o padrão atemporal canônico do vault:

→ **[[../../../../30-agents/playbooks/dependency-audit]]**

Qualquer especialização (extensão, exceção, configuração específica) está documentada abaixo e em [[../../../30-agents/playbooks/_moc]].

## Aplicação em Master Síndico

Backend: `go mod tidy && go mod verify`. Web: `bun pm ls`. Mobile: `flutter pub outdated`.

## Links

- [[../../../../30-agents/playbooks/dependency-audit]] — referência canônica (atemporal)
- [[../_moc]]
- [[../../CLAUDE]]
