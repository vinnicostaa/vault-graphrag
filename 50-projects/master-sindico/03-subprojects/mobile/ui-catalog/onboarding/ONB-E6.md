---
title: ONB-E6 — Documentos (Mobile, Empresa)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, onboarding, empresa]
project: master-sindico
persona: empresa
screen_id: ONB-E6
sub_produto: banco-talentos
plan_requirement: trial
status: specification
stack: mobile
created: 2026-04-24
---

# ONB-E6 — Documentos (Mobile, Empresa)

## Finalidade

Upload docs: contrato social, CND federal, CND estadual, certidão municipal (opcional).

## Persona e ABAC

- Empresa.

## Fluxo mobile

1. Lista de doc slots (4 itens).
2. Tap slot → picker (foto/PDF).
3. Upload presigned.
4. Quando todos obrigatórios carregados → habilita próximo.

## Widget Flutter principal

`OnbE6DocsPage` com `ListView` de `DocumentSlot`.

## Platform-specific notes

- Permissions camera + storage.
- Progress por slot.

## Estados

- AllEmpty / PartiallyFilled / AllFilled / Uploading / Error.

## Offline behavior

- Picker livre; upload fila.

## Push notification trigger

Nenhum.

## Gaps/ressalvas

- Validação automática CND via parse PDF — M3.

## Links

- [[_moc]] · [[ONB-E7]]
