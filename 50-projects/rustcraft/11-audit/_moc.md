---
title: MOC — rustcraft 11-audit
type: moc
tags:
  - moc
  - audit
  - rustcraft
created: '2026-05-27'
updated: '2026-05-27'
---
# MOC — 11-audit

Area canonica de auditoria do projeto `rustcraft`, alinhada ao scaffold `[[../../../40-templates/project-scaffold/_moc]]`.

## Entrada obrigatoria

- [[AUDIT]] — findings vivos A-###.
- [[audit-log/_moc]] — audits datados e evidencias.
- [[dual-check-log/_moc]] — registros de dual-check.
- [[postmortems/_moc]] — incidentes blameless, se houver.

## Regra de uso

1. Abrir `[[AUDIT]]` no inicio da sessao.
2. Se houver finding High/Critical aberto, resolver ou planejar antes de task nova.
3. Registrar novos findings como A-###.
4. Criar audit datado em `audit-log/` quando a revisao for maior que uma linha.
5. Atualizar `[[../STATE]]` se o finding gerar decisao ou divida tecnica.

## Links

- [[../_moc]]
- [[../STATE]]
- [[../SESSION_CHARTER]]
- [[../../../40-templates/audit-template]]
