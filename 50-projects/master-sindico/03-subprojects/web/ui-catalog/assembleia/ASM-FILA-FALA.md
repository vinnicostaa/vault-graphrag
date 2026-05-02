---
title: ASM-FILA-FALA — Fila de Fala
type: ui-screen
tags: [master-sindico, ui-catalog, web, assembleia]
project: master-sindico
persona: morador
secondary_persona: sindico
category: ASM
screen_id: ASM-FILA-FALA
sub_produto: assembleia-live
status: specification
stack: web
created: 2026-04-24
---

# ASM-FILA-FALA — Fila de Fala

## Finalidade

Mecanismo de ordenação de falas durante a assembleia: morador levanta a mão, entra na fila, síndico/presidente autoriza, cronômetro dispara (2 ou 3 min conforme [[ASM-CFG]], com prorrogação de +2 min se habilitada). Apenas falas microfonadas vão para a ata (§5.10).

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/03-Modulos/Assembleia-Live-Funcional]] §5.10 (Controle de fala)
- [[../../../../04-requirements/functional/assembly]] Req 58.2

## Persona e ABAC

- **Primária morador**: levantar/abaixar mão.
- **Primária síndico**: autorizar, liberar microfone, parar cronômetro.
- ABAC: `assembly.speech.raise` / `assembly.speech.authorize`.

## Fluxo da tela

**Morador**:
1. Botão grande **[Levantar mão]** em app durante discussão aberta.
2. Mostra posição na fila e tempo estimado.
3. Pode abaixar a mão a qualquer momento.

**Síndico**:
1. Painel com fila ordenada (FIFO): posição · unidade · nome · hora do levantamento.
2. CTA por linha: **[Chamar para fala]** → ativa cronômetro.
3. Controles: **[Prorrogar +2min]**, **[Encerrar fala]**.
4. Microfone deve ser habilitado de **forma alternada** (captação/gravação — §5.10).

## Componentes

- Botão mão grande + animação.
- Fila list com posições.
- Cronômetro destacado no telão ([[ASM-TELAO]]).

## Estados (loading/empty/error/success)

- **Empty síndico**: "Fila vazia".
- **Success**: transições fluidas ao chamar próximo.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Levantar mão | `/api/v1/assemblies/:id/queue/raise` | POST | — | `{position}` |
| Abaixar mão | `/api/v1/assemblies/:id/queue/lower` | POST | — | 204 |
| Chamar fala | `/api/v1/assemblies/:id/queue/:user_id/call` | POST | — | `{started_at, duration_s}` |
| Encerrar fala | `/api/v1/assemblies/:id/queue/:user_id/end` | POST | — | `{ended_at}` |

## Regras de negócio críticas

- Tempo máximo de fala definido em [[ASM-CFG]] (2 ou 3 min) + botão prorrogação +2 min se habilitada.
- **Microfone alternado**: backend registra em metadata `mic_session_id` para captura de áudio separada (integração com serviço de transcrição futura).
- **Apenas falas microfonadas** vão para a ata (§5.10) — orientar síndico a comunicar antes.
- Audit por evento (levantar, abaixar, chamar, prorrogar, encerrar).
- Latência WebSocket < 200ms P95.

## Ligações

- Origem: [[ASM-PAINEL-SIND]].
- Destino: [[ASM-TELAO]] (mostra fila).
- Relacionados: [[../../../../client-canvas/Fluxo Assembleia Ao Vivo]].

## Gaps/ressalvas

- Transcrição automática (falas microfonadas → ata): **fora do escopo M1** — backlog M2/M3 via Whisper/Gemini adapter.
- Moderação de fala (ex: desrespeito): UX pendente para "expulsar da fila" com registro.

## Links

- [[_moc]]
- [[ASM-PAINEL-SIND]]
- [[ASM-TELAO]]
