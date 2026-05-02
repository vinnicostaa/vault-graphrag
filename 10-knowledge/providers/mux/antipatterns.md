---
title: Mux — antipatterns
type: antipattern
tags: [provider, mux, video, antipatterns]
source: https://docs.mux.com
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
---

# Mux — antipatterns

Catálogo de erros recorrentes em integrações Mux, com a correção canônica. Cada item aponta para o padrão oposto em [[patterns]] ou para referência em [[integration-guide]].

---

## AP-MUX-001 — Usar `playback_id` como chave primária no DB

**Sintoma**: tabela `videos` com `PK playback_id`. Esperança de "é um UUID, serve como chave".

**Por que quebra**:
- Um Asset pode ter **N** Playback IDs (ex: um `public` + um `signed` para uso interno)
- Rotação de policy requer **criar novo** Playback ID, então o ID muda
- `playback_id` é **público** (vai na URL HLS) — usar como PK vaza o mesmo ID que identifica recurso sensível em logs de erro, analytics, BI

**Correto**: PK é `asset_id` (UUID privado, estável). Manter `playback_ids JSONB` ou tabela filha `video_playback_ids(asset_id FK, playback_id, policy)`.

Referência: [[overview]] → Arquitetura conceitual.

---

## AP-MUX-002 — Playback URL sem signing para conteúdo privado

**Sintoma**: criar Asset com `playback_policy: ['public']` por praticidade; "depois a gente fecha".

**Por que quebra**: Qualquer pessoa com a URL HLS acessa. Engine web, crawler, screenshot de QA em canal público, link compartilhado em grupo errado — todos bypassam auth. HLS segments são cacheados em CDN; revogar é meia-verdade (cache local sobrevive).

**Correto**: `playback_policy: ['signed']` desde o `POST /assets`. Viewer recebe JWT gerado por request autenticado com TTL curto. Ver [[patterns]] → Signed playback URLs.

Se precisa de dois modos (preview público curto + integral privado): criar **dois Playback IDs** no mesmo Asset, um `public` outro `signed`.

---

## AP-MUX-003 — Confiar em webhook sem verificar assinatura HMAC

**Sintoma**: `POST /webhooks/mux` aceita qualquer payload JSON, processa e muda estado.

**Por que quebra**: endpoint público recebível por atacante. Payload forjado libera vídeo que nunca foi enviado, apaga asset real, ou dispara notificação falsa.

**Correto**: validar header `Mux-Signature: t=<ts>,v1=<hmac>` com `MUX_WEBHOOK_SECRET`:

1. Parse `t` (timestamp) e `v1` (hex HMAC)
2. Checar `|now - t| ≤ 300s` (anti-replay)
3. Recomputar `HMAC-SHA256(secret, f"{t}.{raw_body}")` e comparar em **constant time**
4. Se falha → `400` e **não** processar

SDKs oficiais têm helper (`mux.webhooks.verifySignature`). Usar em vez de roll-your-own. Ver [[integration-guide]] → Verificação de assinatura.

---

## AP-MUX-004 — Esperar `asset.ready` sincronamente na requisição

**Sintoma**: handler HTTP chama `POST /assets`, faz polling em loop `GET /assets/{id}` até `status=ready`, devolve URL no response.

**Por que quebra**:
- Encoding é **async**. Vídeos longos (> 30 min) podem levar muitos minutos
- Worker/goroutine/connection HTTP fica preso por tempo indefinido
- Client-side timeout expira antes de retornar
- Escala quebra: 100 uploads concorrentes = 100 workers travados

**Correto**: fluxo event-driven.
1. Handler cria o upload/asset e **retorna 202 Accepted** com `asset_id`
2. Cliente escuta via WebSocket/SSE ou poll leve do próprio domínio (não do Mux)
3. Webhook `video.asset.ready` atualiza estado no DB e notifica cliente

Ver [[patterns]] → Webhook lifecycle.

---

## AP-MUX-005 — Re-upload do arquivo quando encoding falha

**Sintoma**: webhook `video.asset.errored` chega; código re-envia o mesmo arquivo esperando encoding diferente.

**Por que quebra**:
- Erro de encoding do Mux é determinístico para o mesmo input (codec não suportado, arquivo corrompido, bitrate fora do range). Re-upload reproduz o erro.
- Custa banda e tempo duas vezes
- Polui dashboard com assets errored duplicados

**Correto**: inspecionar `errors` no payload do webhook. Classificar:
- **Input inválido** (`invalid_input`, `unsupported_codec`) → expor ao usuário "arquivo não suportado, reenvie em MP4/H.264"
- **Falha transitória de sistema** (raríssimo) → Mux já faz retry interno antes de marcar errored
- **Policy violation** → alertar admin

Nunca retry automático. Exceção: se erro veio de `uploads` (não `assets`) por falha de rede no PUT — aí sim, retry do PUT com backoff.

---

## AP-MUX-006 — Armazenar JWT de playback no DB

**Sintoma**: ao criar Asset, gera JWT com TTL de 30 dias, persiste em coluna `video.playback_token`. Cliente pega e usa até expirar.

**Por que quebra**:
- TTL longo anula propósito de signed playback (token vaza = acesso por dias)
- Revogação é impossível sem rotacionar Signing Key (quebra todos os tokens vivos)
- Token no DB vaza em backup, log, BI query, screenshot de debug

**Correto**: gerar JWT **on-demand** no handler que serve o player. TTL curto (10-60 min). Player refresca via endpoint autenticado quando expira. Nunca persistir.

Ver [[patterns]] → Signed playback URLs, seção "Estratégia de TTL".

---

## AP-MUX-007 — Usar `Development` environment em produção

**Sintoma**: "está funcionando no dev, só promove". Produção fica com credenciais `Development`.

**Por que quebra**:
- Assets saem com **watermark Mux** visível
- Quota mensal grátis estoura e começa a retornar 402
- TTL de storage é reduzido (Mux limpa assets antigos de dev)
- Métricas não chegam ao dashboard de produção

**Correto**: environments separados (`Development`, `Staging`, `Production`), cada um com seu token. Toggle de env via variável, não hardcoded. Ver [[patterns]] → Environment separation.

---

## AP-MUX-008 — Handler de webhook sem idempotência

**Sintoma**: webhook `video.asset.ready` chega, handler cria registro, dispara email. Mux re-envia (timeout de 10s, replay), handler processa de novo, segundo email enviado.

**Por que quebra**: Mux **garante at-least-once** delivery de webhooks. Duplicatas são esperadas. Sem dedup, efeitos colaterais repetem.

**Correto**: tabela `processed_events(event_id PK, received_at)`. Handler faz `INSERT ... ON CONFLICT DO NOTHING`; se não inseriu, já foi processado, retorna 200 sem side effect.

```sql
INSERT INTO processed_events(event_id) VALUES ($1)
ON CONFLICT (event_id) DO NOTHING
RETURNING event_id;
-- se RETURNING vazio: duplicata, pular
```

---

## AP-MUX-009 — Polling de `GET /assets` em vez de webhook

**Sintoma**: cronjob a cada 30s listando todos os assets para sincronizar estado local.

**Por que quebra**:
- Gasta rate limit
- Latência: mudança de estado demora até 30s pra refletir
- Não escala com volume de assets

**Correto**: webhook é a **source of truth** de estado. Polling só como fallback de reconciliação (ex.: cronjob diário que detecta drift).

---

## AP-MUX-010 — Expor `MUX_TOKEN_SECRET` ao cliente

**Sintoma**: SPA embute token secret em `window.MUX_KEY` para "não precisar de backend".

**Por que quebra**: secret no bundle JS = secret público. Qualquer um deleta assets, cria live streams, gera JWT de playback arbitrário. Vazamento equivalente a conta comprometida.

**Correto**: todas as operações privilegiadas (create asset, create live, generate JWT) passam pelo backend. Cliente só recebe URLs já assinadas e IDs públicos. Direct Upload é a única operação "cliente fala com Mux direto", e mesmo assim via URL pré-assinada gerada no backend.

---

## Links

- [[_moc]]
- [[../_moc]]
- [[overview]]
- [[integration-guide]]
- [[patterns]]
- [[../../antipatterns/_moc]] — catálogo global de antipatterns
- [[../../security/_moc]] — signing, HMAC, secret management
