---
title: ASM-CIEN — Ciência Obrigatória
type: ui-screen
tags: [master-sindico, ui-catalog, web, assembleia]
project: master-sindico
persona: morador
secondary_persona: sindico
category: ASM
screen_id: ASM-CIEN
sub_produto: assembleia-live
status: specification
stack: web
created: 2026-04-24
---

# ASM-CIEN — Ciência Obrigatória da Convocação

## Finalidade

Tela do morador (web/mobile) exibida **imediatamente após login** quando há assembleia pendente de ciência. **Bloqueia o app** para qualquer outra função até que o morador registre ciência (§4.8). Também é a visão do síndico para monitorar % ciência, lista nominal de pendentes e quem sequer abriu a notificação.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/03-Modulos/Assembleia-Live-Funcional]] §4.8 (Ciência obrigatória)
- [[../../../../04-requirements/functional/assembly]] Req 53
- [[../../../../client-canvas/Fluxo Assembleia Ao Vivo]]

## Persona e ABAC

- **Primária morador**: `resident` vê modal bloqueante próprio.
- **Síndico/administradora**: `assembly.cience.read` — vê dashboard agregado.
- Morador dependente: herda modalidade do titular (role_type dependent).

## Fluxo da tela

**Visão morador**:
1. Modal fullscreen inescapável ao abrir app.
2. Título, data, horários, modalidade.
3. Link "Ler edital completo" (abre PDF).
4. Lista de itens da pauta resumida.
5. 5 checkboxes dos **Termos obrigatórios** ([[ASM-TERM]]) — aceite pré-ciência.
6. Botão **[Dou ciência]** (habilita após todos os checkboxes).
7. Após clique: registra e-mail/IP/timestamp; libera app.

**Visão síndico**:
- Progresso geral (% ciência).
- Tabela nominal: Unidade · Morador · Status (deu ciência / abriu mas não deu / não abriu).
- Filtro + export CSV.
- CTA "Enviar lembrete individual".

## Componentes

- Modal bloqueante (morador) com `role="alertdialog"`.
- Tabela com status colorido (verde/amarelo/vermelho).
- Barra de progresso (% ciência).

## Estados (loading/empty/error/success)

- **Empty síndico**: "Ainda ninguém deu ciência" + CTA lembrete em massa.
- **Error morador**: tentar novamente (idempotente server-side).
- **Success morador**: toast "Ciência registrada" + libera nav.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Registrar ciência | `/api/v1/assemblies/:id/cience` | POST | `{terms_accepted:[...]}` | `{cience_id, at}` |
| Dashboard síndico | `/api/v1/assemblies/:id/cience/report` | GET | — | `{total, accepted, opened, untouched, list[]}` |
| Lembrete | `/api/v1/assemblies/:id/cience/remind` | POST | `{user_ids[]?}` | 202 |

## Regras de negócio críticas

- **App travado** até ciência — gate no router (morador-web + mobile): se existe `pending_cience > 0`, força modal.
- Retenção **6 meses** dos dados de ciência (§4.8).
- Síndico vê literalmente: quem deu, quando, quem ainda não, quem não abriu a comunicação.
- Ciência inclui aceite dos 5 termos ([[ASM-TERM]]) — LGPD (§4.17) + declaração de identidade.
- Append-only em `audit_trail` + `lgpd_logs` (INV-088).

## Ligações

- Origem: [[ASM-PUB]] (notificação → morador entra aqui).
- Destino: [[ASM-TERM]], [[ASM-PROC-CAD]] (morador pode cadastrar procuração após ciência).
- Relacionados: [[../../../../client-canvas/Fluxo Assembleia Ao Vivo]].

## Gaps/ressalvas

- Ciência de morador dependente: fluxo explícito ainda não documentado pelo cliente — assume que titular dá ciência por unidade, dependente vê read-only.
- Retenção 6 meses conflita com retenção 5 anos do `audit_trail` — registro mais granular (browser, IP) expira em 6 meses, entrada resumida em `audit_trail` persiste 5 anos.

## Links

- [[_moc]]
- [[ASM-TERM]]
- [[ASM-PROC-CAD]]
