---
title: assembly — Web Requirements (SolidJS)
type: requirement
tags: [master-sindico, web, solidjs, assembly, feature-req, livekit, websocket]
project: master-sindico
stack: web
app: assembly
port: 3004
milestone: m3
status: not-started
created: 2026-04-24
---

# assembly — Web Requirements

## Escopo e marco

**Marco**: M3 (Sprint 12, 2026-07-20). Entregável principal: Assembleia Live completa — configuração, pauta, edital, publicação, **telão 2 áreas** (apresentação + painel operacional), voto, homologação, encerramento, relatório/ata. App **mais complexo** por WebSocket P95 < 200ms + Livekit + realtime voto.

**Depende de**:
- Backend assembly (Sprint M3) — endpoints REST + WSS.
- Livekit (ADR-0005) para áudio/vídeo.
- WebSocket bidirecional para broadcast voto/resultados.
- Cookie httpOnly compartilhado.

## Personas servidas

- `sindico` — cria/conduz assembleia (telão painel operacional), homologa ata.
- `secretario` (sub-role M3) — suporte ao síndico; redige ata paralela.
- `morador` — participa remoto, vê pauta, vota (se elegível).
- `procurador` (outorgado) — vota em nome de outorgante (com documento vigente).
- `admin_ms` — auditoria técnica (D-058) acesso read-only a logs.

## Telas cobertas

| ID | Título | Rota | UI catalog |
|---|---|---|---|
| ASM-CFG | Configurar assembleia | `/cfg/$id` | [[../ui-catalog/assembleia/_moc]] |
| ASM-PAUTA | Montar pauta | `/pauta/$id` | [[../ui-catalog/assembleia/_moc]] |
| ASM-EDITAL | Gerar edital | `/edital/$id` | [[../ui-catalog/assembleia/_moc]] |
| ASM-PUB | Publicar convocação | `/publicar/$id` | [[../ui-catalog/assembleia/_moc]] |
| ASM-TELAO | Telão 2 áreas (apresentação) | `/live/$id/telao` | [[../ui-catalog/assembleia/_moc]] |
| ASM-PAINEL-SIND | Painel operacional síndico | `/live/$id/painel` | [[../ui-catalog/assembleia/_moc]] |
| ASM-VOTO | Tela de voto (morador) | `/live/$id/voto` | [[../ui-catalog/assembleia/_moc]] |
| ASM-HOML | Homologar ata | `/homologar/$id` | [[../ui-catalog/assembleia/_moc]] |
| ASM-ENCER | Encerramento | `/encerrar/$id` | [[../ui-catalog/assembleia/_moc]] |
| ASM-REL | Relatório/ata final | `/relatorio/$id` | [[../ui-catalog/assembleia/_moc]] |

## Funcionais (FR-WEB-ASM-NNN)

### Pré-assembleia (CFG, PAUTA, EDITAL, PUB)

- **FR-WEB-ASM-001** — ASM-CFG wizard: tipo (ordinária/extraordinária), data, local (físico/híbrido/online), quórum mínimo, permite procuração (sim/não), prazo mínimo convocação (default 10 dias).
- **FR-WEB-ASM-002** — ASM-PAUTA: lista de itens drag-and-drop (reorder) — cada item com título, descrição, tipo voto (sim/não/abstenção, múltipla escolha), maioria exigida (simples/qualificada/unanimidade).
- **FR-WEB-ASM-003** — Voice input R2 no campo descrição do item de pauta.
- **FR-WEB-ASM-004** — Validação: total itens ≥ 1, cada item com título ≥ 5 chars, quórum coerente com LGPD assembleia.
- **FR-WEB-ASM-005** — ASM-EDITAL auto-gerado a partir de CFG + PAUTA; preview PDF + editor final rich-text.
- **FR-WEB-ASM-006** — ASM-PUB dispara notificação: email + push morador + anexo PDF edital + post na timeline do condomínio.

### Telão 2 áreas (ASM-TELAO)

- **FR-WEB-ASM-010** — Layout 2 colunas: **apresentação** (70%) e **painel operacional** (30%) toggle para fullscreen apresentação.
- **FR-WEB-ASM-011** — Área apresentação: item de pauta atual em destaque, slides/anexos, resultado de voto em tempo real (quando aberto), contador quórum.
- **FR-WEB-ASM-012** — Painel operacional: lista de presentes (check-in QR + manual), chat com secretário, cronômetro por item, botão "Abrir voto".
- **FR-WEB-ASM-013** — Vídeo Livekit embed (síndico + secretário streamando; participantes remotos como tiles).
- **FR-WEB-ASM-014** — Transição entre itens de pauta: broadcast WS `{type:"agenda_next",itemId}` — todas telas atualizam < 200ms.

### Voto (ASM-VOTO — morador)

- **FR-WEB-ASM-020** — Ao receber WS `{type:"vote_open",itemId,duration}`, cliente morador abre modal de voto.
- **FR-WEB-ASM-021** — Opções renderizadas conforme tipo (radio sim/não/abstenção ou checkbox múltipla).
- **FR-WEB-ASM-022** — Voto submetido via WS `{type:"vote_cast",itemId,choice,signature}` — assinatura cliente HMAC local (anti-replay).
- **FR-WEB-ASM-023** — Feedback imediato: "Voto registrado" (sem revelar conteúdo para outros, privacidade).
- **FR-WEB-ASM-024** — Timer visível (countdown voto); auto-submit abstenção ao esgotar (config).
- **FR-WEB-ASM-025** — Procurador: dropdown "votando em nome de:" lista outorgantes vigentes.
- **FR-WEB-ASM-026** — WS reconnect: manda `{type:"vote_status",itemId}` ao reconectar; backend retorna voto atual; não duplica.

### Painel síndico em vivo (ASM-PAINEL-SIND)

- **FR-WEB-ASM-030** — Quórum real-time: `presentes / total_elegiveis`; indicador verde/vermelho.
- **FR-WEB-ASM-031** — Lista de presentes + procurações validadas (tooltip outorgante).
- **FR-WEB-ASM-032** — Botões: Abrir voto / Encerrar voto / Próximo item / Pausar.
- **FR-WEB-ASM-033** — Resultado parcial durante voto (opcional, configurável — pode esconder até encerrar).
- **FR-WEB-ASM-034** — Exportar snapshot do momento (CSV votos + presenças) para ata.

### Pós-assembleia (HOML, ENCER, REL)

- **FR-WEB-ASM-040** — ASM-HOML: preview ata auto-gerada; síndico edita rich-text; double-confirm publicar.
- **FR-WEB-ASM-041** — Homologação exige assinatura digital (backend — frontend só coleta intenção + 4-eyes se M3+).
- **FR-WEB-ASM-042** — ASM-ENCER: fecha sessão, libera gravação Livekit, gera hash ata.
- **FR-WEB-ASM-043** — ASM-REL: PDF ata final + CSV votos + gravação + export auditado (timestamps, hashes).

## Não-funcionais (NFR-WEB-ASM)

- **Performance realtime**: **WebSocket P95 < 200ms** latência mensagem (canal voto crítico). Livekit latência < 300ms.
- **LCP**: < 2.0s desktop (telão usa desktop; não prioriza mobile).
- **INP**: < 150ms (interações do síndico precisam parecer instantâneas).
- **Bundle**: telão+painel lazy-loaded em rota `/live/$id`; budget < 400 KB (vídeo Livekit é pesado mas `@livekit/components-solid` ou wrapper).
- **Acessibilidade**: WCAG 2.1 **AAA** no voto (ação legal crítica). Teclado-only operável. Screen reader anuncia resultados.
- **Reliability**: WS auto-reconnect backoff; buffer de eventos durante disconnect; replay ao reconectar; mensagem "Reconectando..." ARIA live.
- **Segurança**: voto não falsificável (assinatura HMAC + backend nonce); auditoria completa; nenhum voto em localStorage.

## Rotas TanStack Router

```
src/routes/
├── __root.tsx
├── index.tsx                          # lista assembleias
├── cfg.$id.tsx                        # ASM-CFG
├── pauta.$id.tsx                      # ASM-PAUTA
├── edital.$id.tsx                     # ASM-EDITAL
├── publicar.$id.tsx                   # ASM-PUB
├── live.$id.telao.tsx                 # ASM-TELAO
├── live.$id.painel.tsx                # ASM-PAINEL-SIND (guard sindico/secretario)
├── live.$id.voto.tsx                  # ASM-VOTO (guard morador elegível)
├── homologar.$id.tsx                  # ASM-HOML
├── encerrar.$id.tsx                   # ASM-ENCER
└── relatorio.$id.tsx                  # ASM-REL
```

## Estado

- **WebSocket singleton** `assembly-ws.ts` — `ReconnectingWebSocket` wrapper com backoff 1→2→4→8→16→30s cap, heartbeat 30s, buffer replay, msgs tipadas Zod (`packages/schemas/src/assembly.ts`).
- **createStore** `assemblyStore`: `{currentItem, quorum, presences, myVote, ws_status}` — atualizado por handlers WS.
- **TanStack Solid Query** — dados estáticos (cfg, pauta inicial); uso moderado pois WS é fonte primária em live.
- **createSignal** — timers, toggle fullscreen, modal voto.
- **TanStack Solid Form + Zod** — CFG wizard, PAUTA itens, HOML ata.

## Integração backend

- **REST Base URL**: `http://localhost:8000/api/v1/assembly/*`.
- **WSS URL**: `wss://localhost:8000/ws/assembly/:id` (dev) / `wss://api.mastersindico.com.br/ws/assembly/:id` (prod).
- **Livekit URL**: `wss://livekit.mastersindico.com.br` (token emitido por backend, TTL 1h).
- **Cookie httpOnly compartilhado**.

### Endpoints consumidos

| Método | Path | Payload | Retorno |
|---|---|---|---|
| POST | `/assembly` | `{tipo,data,local,quorumMin,procuracao,prazo}` | 201 `{id}` |
| PUT | `/assembly/:id/pauta` | `{items[]}` | 200 |
| GET | `/assembly/:id/edital` | — | `{pdfUrl,rtf}` |
| POST | `/assembly/:id/publicar` | — | 204 |
| POST | `/assembly/:id/start` | — | `{livekitToken}` |
| WSS | `/ws/assembly/:id` | tipos Zod | stream |
| POST | `/assembly/:id/vote` | `{itemId,choice,signature,onBehalfOf?}` | 204 (eco via WS) |
| POST | `/assembly/:id/close-item` | `{itemId}` | 204 (broadcast via WS) |
| POST | `/assembly/:id/homologar` | `{ataFinal}` | 204 |
| GET | `/assembly/:id/relatorio` | — | `{pdfUrl,csvUrl,hash}` |

### WS Event types (packages/schemas Zod)

```
type AssemblyWSEvent =
  | { type:"agenda_next"; itemId:string }
  | { type:"vote_open"; itemId:string; duration:number }
  | { type:"vote_cast"; itemId:string; userId:string; onBehalfOf?:string }  // server-broadcast (não revela choice)
  | { type:"vote_result"; itemId:string; tally:{sim,nao,abs} }
  | { type:"quorum_update"; presentes:number; total:number }
  | { type:"presence_join"|"presence_leave"; userId:string }
  | { type:"chat_msg"; from:string; text:string }
  | { type:"heartbeat" }
```

## Design system

- **Telão**: layout customizado alta densidade informacional; texto grande (Michroma em headers), números gigantes para quórum.
- Cores status voto: verde (quórum ok), âmbar (voto aberto), vermelho (quórum insuficiente).
- `Dialog` modal voto custom com timer circular animado.
- `Toast` (Kobalte) para eventos WS (presença entrou, voto aberto).
- Livekit `<LiveKitRoom>` wrapper + tiles custom.
- Reduce-motion: animações do timer simplificadas para ring estático + número.

## Segurança

- **Voto integridade**: HMAC client (chave derivada do cookie assinado) + backend valida + nonce anti-replay + timestamp tolerance 30s.
- **CSP**: `connect-src` inclui `wss://*.mastersindico.com.br` + `wss://livekit.mastersindico.com.br`; `media-src` Livekit.
- **Procuração**: frontend apenas apresenta outorgantes do backend (validados vigência+assinatura); NÃO calcula elegibilidade.
- **Gravação**: consent banner antes de entrar (LGPD); gravação controlada pelo backend.
- **XSS chat**: DOMPurify + length limit 500 chars.
- **Auditoria**: toda ação do síndico (abrir voto, fechar item, homologar) logada server-side com IP+UA+hash.

## Testes

- **Vitest** — 95% WS schemas + vote signature + reducer de `assemblyStore` (camada crítica).
- **@solidjs/testing-library** — modal voto render, cfg wizard, pauta drag-drop.
- **MSW + mock WS** — cenários: vote_open → vote_cast → vote_result; reconnect durante voto aberto.
- **fast-check PBT** — sequências de eventos WS não levam store a estado inconsistente.
- **Playwright (E2E M3)** — síndico cria assembleia → publica → abre live → 3 moradores votam → homologa.
- **jest-axe** — telão, voto modal, painel.

## Status atual (real) — AUDIT A-FASE10-DEV-005

🔴 `apps/assembly/src/index.tsx` = stub `<div>Hello assembly!</div>`. Zero das 10 telas. Zero testes. `@livekit/*` ausente em `bun.lock`. WS client inexistente. Backend assembly + Livekit token service ainda não existem (Sprint M3). Não bloqueador M1.

## Gaps/decisões abertas

- **G1**: **Telão 2 áreas** — layout CSS grid fixo vs resizable via `Splitter` (Kobalte não tem — custom?).
- **G2**: Livekit Solid wrapper — não há oficial; `@livekit/client` direto + hook custom.
- **G3**: Assinatura HMAC client — chave derivada de `getTime()+cookieNonce` ou JWK do backend?
- **G4**: Gravação — armazenamento (Mux Asset? S3 direto?) — depende ADR-0005.
- **G5**: Procuração digital — e-CPF/ICP-Brasil integration — backlog M3+ ou M4?
- **G6**: Telão offline mode — se WS cair, o que mostrar? Cache último estado + badge "desconectado".
- **G7**: Secretário role — sub-role novo? ABAC config M3 — alinhar com Zitadel.

## Links

- [[_moc]]
- [[../../../CLAUDE]]
- [[../architecture]]
- [[../design-system]]
- [[../security]]
- [[../requirements]]
- [[../ui-catalog/assembleia/_moc]]
- [[../../../02-architecture/adr/0011-livekit-sfu]]
- [[../../../08-security/BEYOND_CORP]]
