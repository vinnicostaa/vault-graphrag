---
title: Mux — usage across projects
type: note
tags: [provider, mux, cross-project]
created: 2026-04-24
updated: 2026-04-24
---

# Mux — usage across projects

Mapa de onde o Mux é consumido dentro do vault. Serve como índice reverso: quando um pattern ou antipattern do Mux mudar, esta página mostra quem recebe o impacto.

---

## Tabela de uso

| Projeto | Uso | Aplicado em |
|---|---|---|
| master-sindico | Live streaming de assembleia condominial (RTMP ingest + HLS delivery) + playback signed (JWT HS256, TTL curto) + replay VOD automático via `video.asset.ready` | [[../../../50-projects/master-sindico/01-domain/bounded-contexts]] (contexto Assembly), [[../../../50-projects/master-sindico/STATE]] (decisão de provider) |

---

## Padrões aplicados por projeto

### master-sindico

- **Signed playback** ([[patterns]] §1) — todo playback de assembleia é privado, restrito a condôminos autenticados. JWT `aud=v`, TTL 15 min, refresh pelo Mux Player.
- **Webhook lifecycle** ([[patterns]] §2) — `video.live_stream.active` → notifica condôminos que assembleia começou. `video.asset.ready` do replay → marca assembleia como arquivada.
- **Reconnect window** ([[patterns]] §4) — configurado em 90s: síndico transmitindo de rede instável (wifi de garagem, 4G) recupera sem encerrar a sessão.
- **Passthrough idempotency** ([[patterns]] §6) — `passthrough` carrega `tenant:<condominio>;assembly:<id>` para correlação domínio ↔ asset.
- **Environment separation** ([[patterns]] §10) — envs Mux `Development`, `Staging`, `Production` espelham envs do app.

### Antipatterns que o projeto evita

- [[antipatterns]] AP-MUX-001 — `asset_id` é PK; `playback_ids` é lista filha.
- [[antipatterns]] AP-MUX-003 — handler de webhook valida HMAC via SDK antes de tocar DB.
- [[antipatterns]] AP-MUX-006 — JWT gerado por request; nunca persistido.

---

## Projetos candidatos (não decididos ainda)

Vazio — adicionar quando novo projeto do vault declarar uso de Mux em STATE.md D-###.

---

## Links

- [[_moc]]
- [[../_moc]]
- [[overview]]
- [[integration-guide]]
- [[patterns]]
- [[antipatterns]]
