---
title: ONB-E6 — Empresa Conteúdo Institucional
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding, empresa]
project: master-sindico
persona: empresa
category: ONB
screen_id: ONB-E6
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-E6 — Empresa · Conteúdo Institucional (Tela 6)

## Finalidade

Cadastro do conteúdo público da empresa: descrição, vídeo institucional, vídeos técnicos, portfólio. Base da vitrine na plataforma. Regra dura: **sem conteúdo comercial direto** — contato ocorre via Connect Me.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] — TELA 6 Empresa
- [[../../../../04-requirements/registration-data]] §3.2

## Persona e ABAC

- `enterprise` pré-ativação.
- ABAC: `content.institutional.write`.

## Fluxo da tela

1. Campos:
   - **Descrição institucional** (≤ 2000 chars, sem contato/preço/CTA comercial).
   - **Vídeo institucional** (upload ≤ 90s, trava 90d §2.4 excopo).
   - **Vídeos técnicos** (lista — quota do plano Plus/Pro).
   - **Portfólio**: fotos (até 10, 5MB cada) + cases texto.
2. **Regra dura (literal)**: _"Sem chamadas comerciais, sem preços, sem dados de contato no conteúdo público — direcionar via Connect Me."_
3. Botão **[Continuar]** → [[ONB-E7]].

## Componentes

- Rich text editor com contador.
- Uploader vídeo/imagens com validação.
- Linter que destaca ocorrências de telefone/e-mail/preço no texto (bloqueia).

## Estados (loading/empty/error/success)

- **Error**: vídeo > 90s · trava 90d ativa · descrição com contato detectado.
- **Success**: vídeos entram em moderação (48h SLA manual M1 — ADR-0024).

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Upload vídeo institucional | `/api/v1/videos/institutional/upload` | POST | multipart | `{video_id, moderation_state:"pending"}` |
| Upload vídeo técnico | `/api/v1/videos/technical/upload` | POST | multipart | `{video_id}` |
| Upload portfólio | `/api/v1/portfolios/upload` | POST | multipart | `{photo_id}` |
| Salvar draft | `/api/v1/onboarding/draft` | PATCH | `{step:6, desc, video_inst_id, videos_tec_ids, portfolio_ids, cases}` | 204 |

## Regras de negócio críticas

- **Trava 90 dias** vídeo institucional (§2.4 excopo, CNT-004, ADR-D-054 admin bypass via 4-eyes).
- Descrição sem contato (COM-003).
- Moderação manual M1 (ADR-0024); cascade Mux + Rekognition + OpenAI Moderation em M2+.
- Quota vídeos técnicos depende do plano (Plus=5, Pro=ilimitado — config).

## Ligações

- Origem: [[ONB-E5]].
- Destino: [[ONB-E7]].
- Relacionados: [[../../../admin-privilegios]] §9.

## Gaps/ressalvas

- Linter de conteúdo comercial: regex simples M1; ML/NLP M3+.

## Links

- [[_moc]]
- [[ONB-E5]]
- [[ONB-E7]]
