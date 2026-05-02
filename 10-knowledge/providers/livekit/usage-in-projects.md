---
title: Livekit — Usage in Projects
type: note
tags:
  - provider
  - livekit
  - webrtc
  - realtime
  - usage
  - projects
source: 'https://docs.livekit.io'
source_date: '2026-04-24'
created: '2026-04-24'
updated: '2026-04-24'
---

# Livekit — Usage in Projects

Mapa de quais projetos deste vault consomem Livekit, em qual contexto de domínio, e com qual configuração-chave. Para arquitetura geral do provider, ver [[_moc]]; para padrões, ver [[patterns]].

> Este arquivo é **um índice**, não a fonte da verdade. Para detalhe de cada projeto, seguir o link do bounded context correspondente.

---

## Tabela de uso

| Projeto | Bounded Context | Caso de uso | Room naming | Participants | Egress target |
|---|---|---|---|---|---|
| Master Síndico | Assembly | Assembleia condominial ao vivo com gravação + ata imutável | `assembly:<assembly_id>:<occurrence>` | Residentes convocados (subscribe por default; publish = a palavra é concedida pelo síndico) | S3 bucket com object lock → ata PDF gerada pós-evento |

---

## Master Síndico — Assembleia ao vivo

**Bounded context**: `Assembly` — ver [[../../../50-projects/master-sindico/01-domain/bounded-contexts]].

**Caso de uso**: assembleia condominial com vídeo/áudio ao vivo, controle de palavra, e **gravação imutável** obrigatória para fins legais (ata em formato auditável).

**Configuração-chave**:

- **1 Room por ocorrência de assembleia** — nome determinístico `assembly:<uuid>:<data>`, criado via `RoomServiceClient.CreateRoom` no momento do agendamento (não auto-create), com `empty_timeout=600` (10min de tolerância entre saídas/entradas).
- **Participants = residentes convocados** — `identity = unit_id + user_id` (permite rastrear por unidade habitacional); `display_name = nome da unidade`.
- **Permissions por papel**:
  - Síndico/administração: `CanPublish=true, CanSubscribe=true, RoomAdmin=true`
  - Residentes: `CanPublish=false, CanSubscribe=true` por default
  - Concessão de palavra: `UpdateParticipant` muda `CanPublish=true` temporariamente
- **Egress**: `RoomCompositeEgress` com layout `grid` → MP4 em S3 bucket com **object lock** (imutabilidade 5 anos por compliance). Após `egress_ended{COMPLETE}`, pipeline gera ata em PDF com hash do arquivo MP4 referenciado.
- **Webhooks consumidos**: `room_started`, `participant_joined` (registro de presença para quórum), `participant_left`, `egress_ended`, `room_finished`.
- **Quórum**: calculado em tempo real pelo backend observando `participant_joined` / `participant_left` vs. lista de unidades aptas a votar.

**Decisão de arquitetura**: Livekit Cloud na fase inicial; considerar self-host quando custo mensal > 1× SRE (ver critério em [[overview#Cloud vs self-hosted]]).

**Por que Livekit e não [[../mux/_moc|Mux]]**: assembleia é **interativa** (palavra aberta, reação em tempo real, votação observável). Mux é broadcast 1→N sem interatividade nativa — perde o modelo de participação.

Para detalhe completo de requisitos, ver:
- [[../../../50-projects/master-sindico/01-domain/bounded-contexts]] — bounded context `Assembly`
- (especificação funcional e fluxo de voto/quórum vivem no projeto)

---

## Adicionando um projeto novo

Ao integrar Livekit em um projeto novo deste vault:

1. Atualizar esta tabela com uma linha nova
2. Seção detalhada abaixo no formato do Master Síndico
3. Linkar o bounded context do projeto
4. Registrar D-### em `STATE.md` do projeto com rationale da escolha Livekit vs. concorrentes

Concorrentes a considerar antes de confirmar Livekit:

- [[../mux/_moc|Mux]] — se o caso de uso for broadcast 1→N para 10k+ espectadores passivos
- Daily.co — se time pequeno priorizar SDK turnkey com UI kit
- Twilio Video — **não escolher** (produto em EOL)
- Pion (lib Go) — só se construir SFU próprio é diferencial de negócio

---

## Vizinhos no grafo

- [[_moc]]
- [[overview]]
- [[integration-guide]]
- [[patterns]]
- [[antipatterns]]
- [[../_moc]]
- [[../mux/_moc]]
