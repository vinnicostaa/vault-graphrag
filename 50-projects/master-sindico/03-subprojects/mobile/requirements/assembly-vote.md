---
title: Assembly Vote — Mobile Flutter
type: requirement
tags: [master-sindico, mobile, flutter, assembly, feature-req]
project: master-sindico
stack: mobile
priority: m3
status: pending
bounded_context: assembly
created: 2026-04-24
---

# Assembly Vote — Mobile

Votação **real-time** durante assembleia live via **WebSocket**. Peso do voto = fração ideal da unidade (aplicado server-side). Resultado stream em tempo real para todos participantes.

## Escopo M1/M2/M3

- **M1/M2**: fora do escopo.
- **M3**: voto Sim/Não/Abstenção + stream resultado + timeout + reconnect.

## Telas envolvidas

- `ASM-VOTO` (tela principal — item em votação + 3 botões)
- `ASM-FILA-FALA` (levantar mão para falar)
- `ASM-PROC-CAD` (cadastrar procuração simples — subset mobile; completo = web)
- `ASM-CIEN` (ciência de item não-votável)

## Funcionais (FR-MOB-ASM-VOTE-N)

- **FR-MOB-ASM-VOTE-1** Conectar WebSocket `/ws/assembly/{id}` com `Authorization: Bearer ...` no query (fallback header se proxy suportar).
- **FR-MOB-ASM-VOTE-2** Receber evento `item.voting.opened { item_id, question, opened_at, timeout_seconds }` → renderizar tela com timer countdown.
- **FR-MOB-ASM-VOTE-3** 3 botões full-width: Sim / Não / Abstenção — altura ≥ 56dp; cores alto contraste.
- **FR-MOB-ASM-VOTE-4** Tap em botão → `POST /api/v1/assembly/{id}/items/{item_id}/vote { choice }` com idempotency key `{user_id}_{item_id}`.
- **FR-MOB-ASM-VOTE-5** Após voto, UI trava (enable=false) + exibe "Voto registrado" + peso aplicado (fração ideal).
- **FR-MOB-ASM-VOTE-6** Receber evento `item.voting.tally.updated { yes, no, abstain, fracao_total }` → atualizar gráfico barra ao vivo.
- **FR-MOB-ASM-VOTE-7** Receber `item.voting.closed { result }` → tela resultado + transição para próximo item.
- **FR-MOB-ASM-VOTE-8** Reconnect com backoff exponencial (1s, 2s, 4s, 8s — max 30s); após 3 falhas → modal "Sem conexão" + oferecer reconexão manual.
- **FR-MOB-ASM-VOTE-9** Durante voto, tela bloqueia pull-to-refresh e gestos que saiam (AppBar back desabilitado).
- **FR-MOB-ASM-VOTE-10** `ASM-FILA-FALA`: botão "Levantar mão" → `POST /api/v1/assembly/{id}/speaker-queue/raise` (envia fila ao síndico).
- **FR-MOB-ASM-VOTE-11** `freeRASP integrity != ok` em modo **gated M3** → bloquear voto (modal "dispositivo não seguro para votar"); ciência ainda permitida.

## Não-funcionais (NFR-MOB)

- **NFR-MOB-PERF-1** Latência evento → UI update < 500ms (WebSocket + render).
- **NFR-MOB-SEC-1** Voto idempotente server-side; cliente envia idempotency key.
- **NFR-MOB-OFFLINE-1** Offline durante assembleia = NÃO permitido votar; banner "conecte-se para votar".
- **NFR-MOB-A11Y-1** Botão de voto com `Semantics(label: 'Votar Sim', button: true, onTapHint: 'voto registrado imediatamente')`.

## Dados locais

- **Hive `assembly_{id}_votes`**: `{ item_id: choice }` cache local (UI optimistic + server-confirm).
- **NÃO hydrated_bloc** para state de sessão ativa (volátil).
- **Invalidação**: push `assembly.closed` → limpar tudo.

## Integração backend

- WebSocket `wss://api.mastersindico.com.br/ws/assembly/{id}` (eventos JSON).
- `POST /api/v1/assembly/{id}/items/{item_id}/vote`
- `POST /api/v1/assembly/{id}/items/{item_id}/acknowledge`
- `POST /api/v1/assembly/{id}/speaker-queue/raise`
- `DELETE /api/v1/assembly/{id}/speaker-queue/me`

## Padrões Flutter aplicáveis

- `VoteBloc` + events `Connected / ItemOpened / VoteSubmitted / TallyReceived / ItemClosed / Disconnected`.
- `bloc_concurrency.droppable()` **obrigatório** no `VoteSubmitted` (double-tap = catástrofe jurídica).
- `freezed` em `VoteChoice` enum + `AssemblyItem` entity + `TallySnapshot`.
- `web_socket_channel` package + reconnect wrapper customizado.
- `StreamSubscription` gerenciado no Bloc (`close()` em `close()`).

## Gaps/decisões abertas

- **Q-MOB-ASM-VOTE-01** Voto secreto vs aberto — D-070 (assembly websocket) define mas config por item; UI precisa ler flag `is_secret` para esconder próprio voto após cast.
- **Q-MOB-ASM-VOTE-02** Procuração (voto em nome de terceiro) mobile subset: apenas visualizar poderes outorgados; cadastro = web (Q28 aberta).
- **Q-MOB-ASM-VOTE-03** Empate em voto aberto: server-side decide desempate (síndico); UI só mostra resultado.

## Links

- [[../../../_moc]]
- [[../../../00-product/sub-produtos/05-assembleia-live]]
- [[assembly-check-in]]
- [[../../../STATE#D-070|D-070 WebSocket assembleia]]
- [[../../../STATE#D-050|D-050 Integrity gated M3]]
