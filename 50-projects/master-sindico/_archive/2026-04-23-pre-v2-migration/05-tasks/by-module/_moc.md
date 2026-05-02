---
title: MOC — Tasks by Module
type: moc
tags: [moc, master-sindico, tasks, module]
project: master-sindico
created: 2026-04-22
---

# Tasks — by Module

Cada bounded context tem sua pilha de tasks. Tasks podem referenciar múltiplos sprints; cross-module fica em [[../global]].

## Módulos

- [[identity]] — auth, sessão, 1-device, onboarding, ABAC engine
- [[billing]] — planos, Stripe, trial, quotas, feature_usage, webhooks
- [[institutional]] — condomínio, unidade, membership, timeline, plano diretor, eventos, compliance
- [[commercial]] — Connect Me, empresas, contratos, marketplace, reputação, empresa-sindica, harassment-reports
- [[content]] — vídeos Mux, trava 90d, busca, moderação
- [[assembly]] — assembleias, votação, atas, fila de fala, live Livekit
- [[cross-domain]] — integrações cross-module, eventos de domínio, IAuditLogger

## Vizinhos

- [[../_moc]]
- [[../by-sprint/_moc]]
- [[../../04-requirements/_moc]] — requirements por módulo
