---
title: ASM-TELAO — Telão (2 Áreas) Ao Vivo
type: ui-screen
tags: [master-sindico, ui-catalog, web, assembleia]
project: master-sindico
persona: sindico
secondary_persona: morador
category: ASM
screen_id: ASM-TELAO
sub_produto: assembleia-live
status: specification
stack: web
created: 2026-04-24
---

# ASM-TELAO — Telão Projetado Ao Vivo

## Finalidade

View de projeção (televisor/projetor na sala presencial + versão morador-web / mobile para participantes online futuros) dividida em **2 áreas**: (1) Apresentação de materiais de apoio e (2) Painel operacional com KPIs e estado da assembleia em tempo real.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/03-Modulos/Assembleia-Live-Funcional]] §5.7 (Apresentação) e §5.8 (Estrutura do telão)
- [[../../../../04-requirements/functional/assembly]] Req 59

## Persona e ABAC

- **Primária**: `syndic` ou operador da mesa (controla a apresentação).
- **Secundária**: morador presente (consome passivamente via projetor; opcional via app com view read-only).
- ABAC: `assembly.presentation.control` (operador).

## Fluxo da tela

1. **Área 1 — Apresentação** (50% da tela horizontal, topo): renderiza documentos de apoio conforme operador clica (PDF, imagem, vídeo). PowerPoint deve ser convertido para PDF antes do upload (§5.7).
2. **Área 2 — Painel operacional** (50% inferior):
   - Pauta completa com item em destaque
   - Unidades presentes
   - Unidades ausentes
   - ~~Unidades online~~ (desabilitada M1 — híbrida inativa)
   - Fila de mãos levantadas (timer visível)
   - Cronômetro da fala
   - Status dos itens (resolvido/adiado/prejudicado/não-resolvido)
   - Quórum presente + quórum necessário
   - Relógio da assembleia (tempo desde abertura)
3. Durante votação: layout muda para destacar votos por unidade + placar agregado.

## Componentes

- Layout grid 50/50.
- Embed PDF.js, image viewer, video player (Mux hls).
- KPI widgets com atualização WebSocket.
- Cronômetro/relógio high-contrast.

## Estados (loading/empty/error/success)

- **Pré-início**: logo Master Síndico + dados do condomínio + countdown.
- **Error**: banner degradado (desconexão temporária), apresentação fallback local.
- **Success**: transições suaves entre itens.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| WebSocket live | `/ws/assemblies/:id` | WS | — | stream de eventos |
| Apresentar | `/api/v1/assemblies/:id/presentation/show` | POST | `{asset_id}` | 204 |
| Upload PDF | `/api/v1/assemblies/:id/presentation/upload` | POST | multipart | `{asset_id}` |

## Regras de negócio críticas

- Formatos aceitos: PDF, imagem, vídeo (§5.7). **PowerPoint convertido para PDF antes**.
- WebSocket latência **< 200ms P95** (§11 NFR) — todos os eventos sincronizam.
- Apresentação **não** aparece em telão público para moradores online até híbrida ativa (M2+).
- Gravação da tela: se habilitada, respeita aceite de termo ([[ASM-TERM]] §4.17 autorização de gravação).

## Ligações

- Origem: [[ASM-PAINEL-SIND]].
- Destino: [[ASM-VOTO]] (telão muda durante votação).
- Relacionados: [[../../../../client-canvas/Fluxo Assembleia Ao Vivo]].

## Gaps/ressalvas

- Conversor PowerPoint → PDF: **não incluído** — operador deve converter antes.
- Suporte a múltiplos monitores (telão + segundo monitor do operador): backlog M2.

## Links

- [[_moc]]
- [[ASM-PAINEL-SIND]]
- [[ASM-VOTO]]
