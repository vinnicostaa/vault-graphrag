---
title: ASM-PUB — Publicar Edital
type: ui-screen
tags: [master-sindico, ui-catalog, web, assembleia]
project: master-sindico
persona: sindico
secondary_persona: administradora
category: ASM
screen_id: ASM-PUB
sub_produto: assembleia-live
status: specification
stack: web
created: 2026-04-24
---

# ASM-PUB — Publicar Edital

## Finalidade

Confirma a publicação oficial da assembleia, dispara notificações multicanal (e-mail + push + SMS), gera QR code da assembleia e **aciona LOCK_Pauta** (§4.5 funcional). Ponto de não-retorno — a partir daqui, alterações substantivas exigem assembleia nova.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/03-Modulos/Assembleia-Live-Funcional]] §4.5 (Lock) e §4.7 (Publicação)
- [[../../../../04-requirements/functional/assembly]] Req 52
- [[../../../../client-canvas/Arquitetura Assembleia]]

## Persona e ABAC

- **Primária**: `syndic` com step-up MFA em M1 (ADR-0026 — rota equiparada a "publish").
- **Secundária**: `administradora` com `assembly.publish` quando delegada.
- Audit obrigatório: `audit_trail.action = assembly.published` com `reason` e `pdf_sha256`.

## Fluxo da tela

1. Resumo da assembleia em cartão único (tipo, data, convocações, modalidade).
2. Preview do edital gerado/anexado ([[ASM-EDITAL]]).
3. Checklist de pré-requisitos verde:
   - [x] Dados estruturais (ASM-CFG)
   - [x] ≥1 item de pauta (ASM-PAUTA)
   - [x] Edital válido
   - [x] Fração ideal importada (opcional antes, obrigatório até 1h antes do início — [[ASM-FRAC]])
   - [x] ≥ 8 dias até 1ª convocação
4. Campo **Canais de notificação**: email (obrigatório), push (obrigatório), SMS (obrigatório se telefone disponível).
5. Botão **[Publicar agora]** com confirmação dupla ("Esta ação aciona LOCK_Pauta").
6. Após publicar: exibe **QR code único** + link de compartilhamento + CTA para [[ASM-CIEN]] (monitorar ciência).

## Componentes

- Checklist-guard (bloqueia publicação se algum requisito vermelho).
- Modal de confirmação com contador "3 segundos" anti-clique-acidental.
- QR code SVG (biblioteca qrcode.js).
- Contador regressivo até 1ª convocação (live).

## Estados (loading/empty/error/success)

- **Loading**: barra de progresso "Publicando... disparando notificações".
- **Error**: rollback automático + mensagem ("falha no provider de e-mail; tente novamente").
- **Success**: confete + QR code + CTA para dashboard [[ASM-DASH]].

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Publicar | `/api/v1/assemblies/:id/publish` | POST | `{channels:[email,push,sms]}` + header X-Step-Up-Token | `{status:"published", qr_url, published_at}` |
| Re-disparar | `/api/v1/assemblies/:id/notify` | POST | `{channel}` | 202 |

## Regras de negócio críticas

- **LOCK_Pauta (§4.5)** ativado no commit: nenhum item pode ter título/descrição/quórum/anexos alterados.
- Dispara **e-mail + push + SMS** (§4.7) — SMS respeita opt-in LGPD (IEmailProvider/IPushProvider/ISMSProvider agnósticos, D-056).
- QR code gerado **único por assembleia** (vinculado a `assembly_id`).
- Lembretes automáticos agendados (§4.10): **3 dias, 1 dia, 3h antes** (cron + fila).
- Step-up MFA obrigatório: sem `X-Step-Up-Token` válido ≤ 5min, recusado 401.

## Ligações

- Origem: [[ASM-EDITAL]].
- Destino: [[ASM-CIEN]] (monitoramento ciência).
- Relacionados: [[../../../admin-privilegios]] (telas admin gated), [[../../../../client-canvas/Fluxo Assembleia Ao Vivo]].

## Gaps/ressalvas

- SMS opt-in: campo explícito no cadastro (IDN-024) precisa estar verdadeiro; moradores sem opt-in são omitidos silenciosamente.
- Custo de SMS é responsabilidade do cliente (§2.2 excopo-tecnico-antigo).

## Links

- [[_moc]]
- [[ASM-CIEN]]
- [[ASM-EDITAL]]
