---
title: ONB-S5 — Síndico Presença Profissional (N2/N3)
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding, sindico]
project: master-sindico
persona: sindico
category: ONB
screen_id: ONB-S5
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-S5 — Síndico · Presença Profissional (Tela 5, N2/N3 apenas)

## Finalidade

Tela **exclusiva N2/N3** — síndicos dos planos Plus/Pro — para cadastro de mini-bio, vídeo institucional (com lock 90d), formação/certificações, administradoras vinculadas. Para N1 esta tela é opcional/pulada.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] — TELA 5 Síndico
- [[../../../../04-requirements/registration-data]] §2.2

## Persona e ABAC

- **Primária**: `syndic` com `plan_tier in {syndic_n2, syndic_n3}`.
- ABAC: `institutional.syndic.profile.write`.

## Fluxo da tela

1. Mensagem principal (literal): **"Mostre quem está por trás da gestão."**
2. Campos (N2/N3):
   - **Mini bio profissional** (≤ 500 chars).
   - **Vídeo institucional** (upload até 90s, **trava trimestral de 90 dias** — §2.4 excopo, CNT-004).
   - **Formação e certificações** (JSON array `[{certificacao, data_obtencao, orgao}]`).
   - **Administradoras vinculadas** (seleção de empresas cadastradas — ou solicita cadastro).
3. Nota: _"N1 não tem perfil público — campos opcionais para N1."_
4. Botão **[Continuar]** → [[ONB-S6]].

## Componentes

- Uploader vídeo com validação de duração (90s max).
- Form dinâmico para formações.
- Busca de administradoras.

## Estados (loading/empty/error/success)

- **Error**: vídeo > 90s · upload falha · ultima atualização < 90d (trava).
- **Success**: vídeo processado (via Mux provider agnóstico, D-056).

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Upload vídeo institucional | `/api/v1/videos/institutional/upload` | POST | multipart | `{video_id, processing:true}` |
| Verificar trava 90d | `/api/v1/videos/institutional/lock-status` | GET | — | `{locked_until?}` |
| Salvar draft | `/api/v1/onboarding/draft` | PATCH | `{step:5, bio, video_id, formacoes, administradoras[]}` | 204 |

## Regras de negócio críticas

- **Trava trimestral 90 dias** para vídeo institucional (§2.4 excopo + CNT-004): se já upado em < 90d, bloqueia (admin bypassa com 4-eyes via ADR-0024/D-054).
- Duração vídeo ≤ 90s (hard-coded).
- Mini bio: sem contato/preço/CTA comercial (regra similar ao termo empresa).

## Ligações

- Origem: [[ONB-S4]].
- Destino: [[ONB-S6]].
- Relacionados: [[../../../admin-privilegios]] §9 (moderação).

## Gaps/ressalvas

- N1 pode **pular totalmente** a tela; UX decisão: mostrar card "atualize para N2/N3 para ativar perfil público".

## Links

- [[_moc]]
- [[ONB-S4]]
- [[ONB-S6]]
