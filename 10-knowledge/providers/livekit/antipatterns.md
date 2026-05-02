---
title: Livekit — Antipatterns (o que não fazer)
type: note
tags:
  - provider
  - livekit
  - webrtc
  - realtime
  - antipatterns
source: 'https://docs.livekit.io'
source_date: '2026-04-24'
created: '2026-04-24'
updated: '2026-04-24'
---

# Livekit — Antipatterns

Erros recorrentes em integrações Livekit. Cada item descreve o **problema**, a **consequência concreta** e a **correção**. Contraparte positiva em [[patterns]].

> Fonte: https://docs.livekit.io — consultada 2026-04-24.

---

## ❌ Token com TTL longo (horas ou dias)

**O que parece**: "o usuário fica horas na sala, dá trabalho renovar, vou emitir token válido por 24h."

**Consequência**: o token é um **bearer credential** JWT. Se vaza (extensão de browser, log, print de network tab, screenshare), qualquer pessoa com ele entra na Room **até expirar**. Com TTL de 24h, a janela de ataque é 24h. Revogar exige trocar `LIVEKIT_API_SECRET` (invalida todos os tokens) ou esperar expirar.

**Correção**: TTL ≤ 15min + refresh server-driven (ver [[patterns#Token TTL curto + refresh server-driven]]). No Livekit Cloud o SFU reemite tokens internamente para clients já conectados, então TTL curto **não força reconnect**.

---

## ❌ Trust no client para permissions

**O que parece**: frontend tem lógica `if (user.role === 'viewer') { disablePublishButton() }` e o backend emite token com `CanPublish=true` para todos.

**Consequência**: atacante abre DevTools, remove o `disabled`, publica tracks. Ou usa um client custom. Ou usa o CLI do Livekit diretamente com o token.

**Correção**: **toda permission vive no JWT**, validada pelo SFU. Backend decide em tempo de emissão o que cada identidade pode ou não. Client side é pura UI — nunca fonte de autoridade. Ver [[patterns#Permissions granulares no JWT]].

---

## ❌ Room name como input livre do cliente

**O que parece**: `POST /join` recebe `{ roomName }` do body e emite token direto pra essa room.

**Consequências múltiplas**:

1. **Colisão**: dois usuários podem pedir `"my-meeting"` e cair na mesma Room sem relação.
2. **DoS por criação infinita**: atacante cria 1 milhão de rooms com names aleatórios → consumo de memória + poluição em `ListRooms`.
3. **Enumeration**: atacante tenta `"admin-review"`, `"board-2026"`, etc, e entra em rooms que não deveriam ser dele se o nome for adivinhável.

**Correção**: server-side deriva o room name do agregado de domínio (ex: `meeting:<uuid>`) e valida o input do cliente contra recursos que **ele possui** (tem permissão nesse `meeting_id`). Nunca aceitar o nome cru. Ver [[patterns#Room naming determinístico]].

---

## ❌ Skip cleanup de egress

**O que parece**: dispara `StartRoomCompositeEgress`, nunca escuta `egress_ended`, não copia arquivo para storage final.

**Consequências**:

- Arquivos ficam em bucket temporário ou de staging
- Sem verificação de integridade/tamanho pós-upload
- Se egress falha (`status=FAILED`), ninguém sabe — usuário pede a gravação dias depois e não existe
- Custo S3 cresce sem controle (arquivos orfãos sem tagging)

**Correção**: escutar webhook `egress_ended` como parte do ciclo de vida (ver [[patterns#Egress assíncrono com webhooks como fonte da verdade]]). Se `status=COMPLETE`, persistir URL + metadata + mover pra bucket com object lock. Se `status=FAILED`, alertar.

---

## ❌ Room metadata para dados sensíveis

**O que parece**: `Metadata: '{"attendees_emails":["a@x.com","b@y.com"],"pin":"1234"}'`.

**Consequência**: **toda metadata de Room é distribuída para todos os participants** via signaling. Qualquer um conectado lê. Mesmo `participant.metadata` (do próprio participant) é público para todos os outros participants da mesma Room.

**Correção**:

- Metadata só para dados **não confidenciais** que genuinamente precisam estar no client (ex: kind da sala, tema, layout hint)
- Dados sensíveis ficam no backend, acessados via API autenticada separada
- Para comunicação privada entre dois participants, usar data channel com payload criptografado end-to-end

---

## ❌ Identity baseada em email (ou qualquer coisa mutável)

**O que parece**: `at.SetIdentity(user.Email)`.

**Consequências**:

- Usuário troca email → é uma identidade nova no Livekit. Histórico de moderation, mute state, kick lists — tudo perde correlação.
- Email é PII; vai pros logs do SFU, webhook payloads, gravações — violação LGPD/GDPR em muitos contextos.
- Dois projetos diferentes podem ter o mesmo email → colisão entre tenants.

**Correção**: identity = `user_id` estável (UUID de domínio). Display name separado em `SetName()` pode ser email ou nome. Ver [[patterns#Participant identity estável e imutável]].

---

## ❌ Retry de connect sem backoff

**O que parece**: `while (!connected) { await room.connect(url, token); }` num loop apertado.

**Consequências**:

- Flood de WebSocket handshakes no SFU — potencial rate-limit ou ban de IP
- Thundering herd quando um outage curto termina (milhares de clients martelando)
- Token pode ter expirado mid-loop — todas as tentativas seguintes falham pelo mesmo motivo sem o backend saber

**Correção**: exponential backoff (1s, 2s, 4s, 8s, cap 30s) + jitter aleatório de ±20% + limit de 10 tentativas antes de exigir ação do usuário + buscar token fresh entre tentativas. Ver [[patterns#Reconnect com exponential backoff + resync]].

---

## ❌ `empty_timeout: 0` ou omitir

**O que parece**: "a sala é pra ficar disponível o dia todo, deixo sem timeout."

**Consequência**: após o último participant sair, a Room persiste no SFU indefinidamente. Em cenários de retry/reconnect fantasma, acumula memória. `ListRooms` retorna centenas de rooms mortas. Sem webhook `room_finished` nunca disparado → backend nunca sabe que pode arquivar estado.

**Correção**: sempre setar `empty_timeout` ≥ 60s. Se precisa de sala "sempre on", recriar programaticamente no primeiro reentry em vez de manter viva eternamente.

---

## ❌ Egress compositing em loop com mesmo template

**O que parece**: disparar 10 egress RoomComposite simultâneos com layouts diferentes "para o usuário escolher".

**Consequência**: cada RoomCompositeEgress spawna um Chrome headless completo. 10 × Chrome = CPU/RAM massivos no egress node + 10× o custo. Livekit Cloud cobra por minuto de egress — multiplicador direto na fatura.

**Correção**: um único RoomCompositeEgress com layout default + TrackEgress paralelo para sources brutas (sem Chrome, sem transcode). Composição alternativa é feita em **post-processing** (ffmpeg) se realmente necessário.

---

## ❌ Processar webhook sem verificar assinatura

**O que parece**: endpoint `POST /hooks/livekit` lê `req.body` e processa.

**Consequência**: qualquer atacante que conhece a URL pública injeta eventos falsos (`egress_ended` fake → backend marca gravação como pronta, aponta pra URL controlada pelo atacante). Integridade do estado de domínio fica à mercê de quem descobriu a URL.

**Correção**: todo webhook Livekit traz `Authorization: Bearer <jwt>` assinado com a API secret. SDK tem `webhook.ReceiveWebhookEvent(body, headers, keyProvider)` que valida SHA256 do payload e assinatura do JWT antes de deserializar. Rejeitar com 401 se falhar.

---

## ❌ Guardar `LIVEKIT_API_SECRET` no frontend

**O que parece**: "vou gerar o token no client pra não precisar de backend."

**Consequência**: o secret está no bundle JS público. Qualquer visitante extrai, gera token pra qualquer Room com qualquer permission (`RoomAdmin: true` inclusive). Comprometimento total do projeto Livekit. Único mitigation: **trocar o secret**, invalidando todos os tokens emitidos.

**Correção**: `LIVEKIT_API_SECRET` **nunca** sai do backend. Client pede token ao backend autenticado, que decide permissions e assina. Esta é a linha intransigente — não há caso de uso legítimo onde o secret precisa estar no client.

---

## Vizinhos no grafo

- [[_moc]]
- [[patterns]]
- [[overview]]
- [[integration-guide]]
- [[../_moc]]
- [[../mux/_moc]]
