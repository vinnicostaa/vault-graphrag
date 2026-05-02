---
title: Rollback — Master Síndico (referência)
type: note
tags: [master-sindico, reference, rollback,playbook]
project: master-sindico
updated: 2026-04-23
---

# Rollback — no Master Síndico

Este projeto **segue** o padrão atemporal canônico do vault:

→ **[[../../../../30-agents/playbooks/rollback]]**

Qualquer especialização (extensão, exceção, configuração específica) está documentada abaixo e em [[../../../30-agents/playbooks/_moc]].

## Aplicação em Master Síndico

Backend: Railway oferece redeploy atômico da build anterior. DB: migrations via goose `down` (avaliar reversibilidade antes). Stripe/Mux/Livekit: reconciliação via webhooks (ver runbook DR).

## Links

- [[../../../../30-agents/playbooks/rollback]] — referência canônica (atemporal)
- [[../_moc]]
- [[../../CLAUDE]]
