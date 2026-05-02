---
title: MOC — 11-audit/postmortems
type: moc
tags: [moc, master-sindico, v2, audit, postmortems]
created: 2026-04-23
updated: 2026-04-23
---

# MOC — 11-audit/postmortems

Postmortems blameless de incidents > 15min (obrigatório P0/P1; opcional P2 se aprendizado relevante).

**Regra blameless**: foco no sistema + processo. Ninguém demitido por incident bem-investigado.

## Arquivos nesta pasta

- [[template]] — template canônico (copiar pra `YYYY-MM-DD-<slug>.md` por incident)

## Processo

1. Incident P0/P1 resolvido → autor gera draft em 48h
2. Review por dev sênior + PO + (se LGPD) DPO
3. Status `final` — apresentar stand-up sprint seguinte + postar em #announcement
4. Ações T-### tracked no backlog até implementadas

## Histórico de incidents

Vazio — ainda não houve incident em prod sob remontagem v2. Legado vivo não registrou postmortems blameless formalmente (lacuna identificada, T-705 no backlog pra dry-run primeiro).

## Vizinhos

- [[../_moc]]
- [[../AUDIT]]
- [[../audit-log/_moc]]
- [[../dual-check-log/_moc]]
- [[../../06-execution/incident]]
- [[../../06-execution/rollback]]
