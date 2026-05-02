---
title: Incident Communication Template
type: template
tags:
  - template
  - master-sindico
  - incident-response
  - compliance
  - stub
status: stub
created: '2026-04-27'
updated: '2026-04-27'
---

# Incident Communication Template

> 🟡 **Stub** criado em 2026-04-27. Template de comunicação para incidentes de segurança (LGPD art. 48 — comunicação de vazamento à ANPD em prazo razoável + titulares afetados).

## Quando usar

- Vazamento de dados pessoais (LGPD art. 48).
- CVE crítico não-mitigável em tempo hábil.
- Indisponibilidade > 4h (SLA M1).

## Canais

- **Status page** (público): updates em tempo real
- **Email transacional** aos afetados (LGPD art. 48 §1º)
- **ANPD** (LGPD art. 48): formulário oficial em até 2 dias úteis
- **In-app banner** durante incident

## Estrutura da mensagem (template)

```
Assunto: Incidente em <Master Síndico> — <data> — Comunicação obrigatória

Prezado(a) <nome>,

Em <data>, identificamos <descrição factual do incidente, sem floreio>. 

**O que aconteceu**: <fato>
**Dados afetados**: <categorias específicas — nunca genérico>
**Risco para você**: <específico — risco real, mitigações user-side>
**O que fizemos**: <ações tomadas — contenção, remediação, notificação ANPD>
**O que você pode fazer**: <ações concretas — trocar senha, ativar 2FA, monitorar>
**Próximos passos**: <cronograma de updates>
**Contato DPO**: <email + formulário>

Atenciosamente,
DPO Master Síndico
```

## Princípios

- **Honestidade total** — nunca minimizar, nunca terceirizar culpa
- **Específico** — categorias e impacto reais; "pode ter sido afetado" só se realmente desconhecido
- **Acionável** — leitor sabe o que fazer
- **Audit-trail** — toda comunicação (email, in-app, ANPD) registra timestamp + recipient + delivery status

## Links

- [[../08-security/cve-process]]
- [[../08-security/lgpd]]
- [[../06-execution/playbooks/incident-response]]
- [[../11-audit/postmortems/_moc]]
