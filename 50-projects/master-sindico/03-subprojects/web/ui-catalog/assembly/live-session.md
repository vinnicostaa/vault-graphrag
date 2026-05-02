---
title: Live Session — UI Spec
type: ui-spec
tags: [ui-catalog, assembly, master-sindico, frontend, livekit, telao]
bc: assembly
source: _chaos/ARQUITETURA-ASSEMBLEIA.pdf §5 + DIAGRAMA-VISUAL.pdf §3-4 + MÓDULO-TELAS.pdf TELA 9-10
absorbed_in: 2026-04-25 — Fase E
status: absorbed
---

# Live Session — Assembleia ao Vivo (Conferência Livekit)

> **Origem**: 3 PDFs Master Síndico — destilados em Fase E (2026-04-25).
> **D-133**: Assembleia = **conferência tipo Meet** via Livekit (presencial/híbrida/online). NÃO confundir com Lives broadcast (content BC + Cloudflare Stream/Mux Live — ver [[../content/live-streaming|content/live-streaming]]).

> Sprint 3-4 · App `assembly` · Rotas: `/assembleias/:id/live`, `/assembleias/:id/live/painel`, `/assembleias/:id/live/telao`
> Perfis: Síndico (presidente), Administradora, Auditor (D-132 efêmero), Morador (presencial/online), Telão público.

---

## Layout — Painel ao vivo do Síndico

**Rota**: `/assembleias/:id/live/painel` (Síndico)

```
┌────────────────────────────────────────────────────────────┐
│ ASSEMBLEIA ORDINÁRIA 2026/1 • 🔴 AO VIVO • Item 3/8       │
│ ⏱ Iniciada às 19:02 (35 min)                              │
├────────────────────────────────────────────────────────────┤
│                                                            │
│ ITEM ATUAL                                                  │
│ ▶ 3. Escolha do Fornecedor para Fachada                    │
│   [Discussão] [Iniciar votação] [Adiar] [Prejudicado]     │
│                                                            │
│ ┌─PRESENÇA (real-time)───┬─FILA DE FALA───────────────┐  │
│ │ Presentes:    72/120   │ 1. Apt A-302 ⏳ aguardando │  │
│ │ Online ativos: —       │ 2. Apt B-105 ⏳            │  │
│ │ Ausentes:     48/120   │ 3. Apt C-201 ⏳            │  │
│ │ Frações pres: 68%      │ [Chamar próximo]  [Remover]│  │
│ └────────────────────────┴────────────────────────────┘  │
│                                                            │
│ ┌─QUÓRUM─────────────────┬─CONTROLE FALA──────────────┐  │
│ │ Necessário item: 50%   │ Falando: Apt A-302         │  │
│ │ Atingido: ✅ 68%       │ ⏱ 02:15 / 03:00           │  │
│ │                        │ [+ Prorrogar 2 min] [Cortar│  │
│ └────────────────────────┴────────────────────────────┘  │
│                                                            │
│ ┌─AÇÕES PRESIDÊNCIAIS────────────────────────────────────┐│
│ │ [▶ Apresentar material] [📊 Telão] [⚙ Modo conting.] ││
│ │ [🎯 Iniciar votação] [✅ Homologar] [🔚 Encerrar]    ││
│ └────────────────────────────────────────────────────────┘│
└────────────────────────────────────────────────────────────┘
```

### Cabeçalho

- Status: `🔴 AO VIVO` quando `Assembly.status = em_andamento`.
- Relógio da assembleia (`startedAt → now()`).
- Indicador de item ativo na pauta.

### Painel central

- 4 cards real-time (WebSocket `/ws/assemblies/:id/state` — latência < 200ms):
  - **Presença**: presentes + ausentes + frações.
  - **Fila de fala**: FIFO, ações de aprovação.
  - **Quórum**: por item atual.
  - **Controle de fala**: cronômetro do morador atual.

### Ações presidenciais (botões grandes)

- Apresentar material (ASM-### Fase E novo).
- Telão (abre `/live/telao` em nova janela/monitor).
- Modo contingência (ativa fallback offline — ASM-022).
- Iniciar votação / Homologar / Encerrar.

---

## Tela — Cadastro Mesa Diretora (pré-iniciar)

**Rota**: `/assembleias/:id/live/mesa-diretora` (Síndico — bloqueante antes de "Iniciar")

```
┌────────────────────────────────────────────────────────┐
│  CADASTRO DA MESA DIRETORA                             │
│  Obrigatório antes de iniciar a assembleia             │
│                                                        │
│  PRESIDENTE DA MESA *                                  │
│  Nome     [Carlos Pereira_____________]               │
│  CPF      [123.456.789-00]                            │
│                                                        │
│  SECRETÁRIOS (até 3) *                                 │
│  ┌────────────────────────────────────────────┐       │
│  │ 1. Ana Lima         CPF: ***.***.***-11    │ [✗]  │
│  │ 2. Pedro Costa      CPF: ***.***.***-22    │ [✗]  │
│  │ 3. + Adicionar secretário                  │      │
│  └────────────────────────────────────────────┘      │
│                                                        │
│  ADMINISTRADORA (presente) *                           │
│  Empresa  Admin XYZ Ltda  Nome [Maria Souza] CPF[..] │
│                                                        │
│  AUDITOR (opcional, D-132)                             │
│  Empresa  [____]  Nome [____] CPF [____]              │
│  ⓘ Cria Membership efêmera role=assembly_auditor      │
│                                                        │
│  FORNECEDORES PRESENTES (opcional)                     │
│  ┌────────────────────────────────────────────┐       │
│  │ 1. Empresa Alpha Fachadas — João Pereira    │ [✗] │
│  │ 2. Beta Recuperação — Marcos Lima           │ [✗] │
│  │ + Adicionar fornecedor                      │     │
│  └────────────────────────────────────────────┘      │
│                                                        │
│  [Cancelar]                  [Salvar e iniciar →]    │
└────────────────────────────────────────────────────────┘
```

### Regras (NOVO ASM-### Fase E)

- Composição da mesa diretora obrigatória antes de `Assembly.Start()`.
- Síndico não vota (INV-081 imparcialidade).
- Auditor: D-132 — Membership efêmera `role=assembly_auditor` ABAC, válida apenas durante a assembleia (TTL = `Assembly.ended_at`).
- Fornecedores presentes: registro adicional ao acervo já carregado em pré-assembleia.

### Mensagem institucional

> "A identificação dos responsáveis pela condução da assembleia garante transparência e legitimidade ao processo deliberativo."

---

## Tela — Vídeo Institucional Pre-Roll (Iniciar)

**Rota**: `/assembleias/:id/live/init` (todos os participantes)

```
┌────────────────────────────────────────────────────┐
│  ▶ VÍDEO INSTITUCIONAL MASTER SÍNDICO              │
│                                                    │
│  ┌──────────────────────────────────────────────┐ │
│  │                                              │ │
│  │   [Player Mux 16:9 — autoplay obrigatório]  │ │
│  │                                              │ │
│  │   Regras de participação, fala, votação,    │ │
│  │   deliberação                                │ │
│  │                                              │ │
│  └──────────────────────────────────────────────┘ │
│                                                    │
│  ⏱ 1:30 / 2:00                                    │
│                                                    │
│  Mensagem institucional:                          │
│  "A assembleia é o espaço de diálogo e decisão    │
│   coletiva do condomínio. Respeito, escuta e      │
│   responsabilidade ajudam a construir soluções    │
│   que beneficiam toda a comunidade."              │
│                                                    │
│  [Ir para painel ao vivo →] (após terminar)       │
└────────────────────────────────────────────────────┘
```

### Regras (NOVO ASM-### Fase E)

- Pre-roll obrigatório quando `Assembly.Start()` (R48 §12).
- Asset gerenciado pela Master Síndico (CNT-029 broadcast asset).
- Player Mux com autoplay.
- Não pode ser pulado (`skippable=false`).

---

## Tela — Telão da Assembleia (2 áreas) — ASM-018

**Rota**: `/assembleias/:id/live/telao` (público — tela cheia, sem auth ABAC token-only)

```
┌──────────────────────────────────────────────────────────┐
│ MASTER SÍNDICO • ASSEMBLEIA ORDINÁRIA 2026/1 • 🔴 AO VIVO│
├──────────────────────────────────┬────────────────────────┤
│                                  │ PAUTA                  │
│  ÁREA 1 — APRESENTAÇÃO            │ ✓ 1. Cont. fachada   │
│  (60% da tela)                   │ ✓ 2. Paisagismo      │
│                                  │ ▶ 3. Forn. fachada    │
│  ┌────────────────────────────┐ │   4. Eleição           │
│  │                            │ │   5. Inform. obras     │
│  │   [PDF / Imagem / Vídeo /   │ │                        │
│  │    Resultado de votação]    │ ├────────────────────────┤
│  │                            │ │ PRESENÇA               │
│  │   Síndico apresenta         │ │ Presentes: 72          │
│  │   sem sair da plataforma   │ │ Online: —              │
│  │                            │ │ Ausentes: 48           │
│  └────────────────────────────┘ │ Frações: 68%           │
│                                  ├────────────────────────┤
│                                  │ FILA DE FALA           │
│                                  │ 1. Apt A-302 ⏱ 02:15  │
│                                  │ 2. Apt B-105           │
│                                  │ 3. Apt C-201           │
│                                  ├────────────────────────┤
│                                  │ QUÓRUM ITEM 3          │
│                                  │ Necess: 50%            │
│                                  │ Atingido: ✅ 68%       │
│                                  ├────────────────────────┤
│                                  │ ⏱ RELÓGIO ASSEMBLEIA  │
│                                  │ 00:35:42               │
└──────────────────────────────────┴────────────────────────┘
```

### Regras (ASM-018)

- 2 áreas:
  - **Área 1 — Apresentação** (60%): PDF / imagem / vídeo / resultado de votação. Síndico/administradora/auditor podem apresentar. PowerPoint deve ser convertido em PDF.
  - **Área 2 — Painel operacional** (40%): pauta com item destacado, presença, fila de fala, cronômetro, quórum, relógio.
- Layout responsivo (otimizado projetor 1920x1080+).
- WebSocket mesma conexão, múltiplos listeners.
- Atualização < 200ms.
- Token de acesso público (signed URL gerado pelo síndico).

### Estados durante votação

Quando votação ativa, Área 1 muda para mostrar:
```
┌──────────────────────────────────────┐
│ VOTAÇÃO EM ANDAMENTO                 │
│ Pauta 3: Escolha Fornecedor          │
│                                      │
│ | Unidade | Voto      |              │
│ | A-302   | Empresa A |              │
│ | B-105   | Empresa B |              │
│ | C-201   | Pendente  |              │
│ ...                                  │
│                                      │
│ Total votos: 38/72 (53%)            │
│ Empresa A: ████████░░ 60%           │
│ Empresa B: ████░░░░░░ 25%           │
│ Empresa C: ██░░░░░░░░ 15%           │
└──────────────────────────────────────┘
```

---

## Fila de Fala Cronometrada (ASM-019)

### Tela do Morador — Levantar mão

**Rota**: `/assembleias/:id/live` (Morador)

```
┌────────────────────────────────────┐
│ Sua posição na fila: 3º            │
│                                    │
│  ┌──────────────────────────────┐ │
│  │   ✋ LEVANTAR MÃO            │ │
│  │   (toque pra entrar fila)    │ │
│  └──────────────────────────────┘ │
│                                    │
│  Tempo de fala definido: 3 min    │
│  Prorrogação possível: +2 min     │
│                                    │
│  Falando agora: Apt A-302         │
│  ⏱ 02:15 / 03:00                  │
└────────────────────────────────────┘
```

### Quando autorizado pelo síndico

```
┌────────────────────────────────────┐
│  🎤 SUA FALA AGORA                 │
│                                    │
│  ⏱  02:30 / 03:00                  │
│  ████████████████░░░░ 83%          │
│                                    │
│  Microfone: 🟢 ATIVO               │
│  (alternado para captação ótima)   │
│                                    │
│  Lembrete: somente falas           │
│  microfonadas serão transcritas    │
│  para a ata.                       │
│                                    │
│  [✋ Encerrar minha fala]          │
└────────────────────────────────────┘
```

### Regras (ASM-019 + Fase E novos)

- Fila FIFO; síndico/presidente aprova chamada.
- Cronômetro inicia ao chamar.
- **NOVO Fase E**: microfone alternado (sistema gerencia mute/unmute para captação ótima e transcrição) — Livekit API.
- **NOVO Fase E**: somente falas microfonadas vão para a ata (lembrete UX bem visível pré-assembleia).
- Alerta 30s antes de fim.
- Síndico pode prorrogar +2min (1x).
- Timeout → Livekit corta áudio automaticamente.
- Transcrição pode usar Whisper/Speechmatics (job async pós-fala) — pendência ADR.

---

## Fluxo de Votação ao Vivo (ASM-014)

### Tipos de voto suportados (NOVO Fase E — diagrama visual)

- **SIM / NÃO / ABSTENÇÃO** (binário com abstenção)
- **ESCALA 1-5 + ABSTENÇÃO** (Likert pra qualidade)

### Tela do Morador — Votar

**Rota**: `/assembleias/:id/live/votar/:itemId` (Morador apto)

```
┌──────────────────────────────────────┐
│ PAUTA 3: Escolha Fornecedor Fachada │
│ Tipo de votação: ESCOLHA_FORNECEDOR │
│                                      │
│ ⏰ Votação aberta — encerra ao clique│
│   do síndico                         │
│                                      │
│ Termo de voto:                       │
│ "Declaro que exerço este voto em    │
│  nome da unidade vinculada ao meu    │
│  acesso, sob minha responsabilidade."│
│ [✓ Aceito]                           │
│                                      │
│ Selecione 1 opção:                   │
│ ┌──────────────────────────────┐    │
│ │ ○ Empresa Alpha               │    │
│ │   📹 [Ver vídeo durante voto] │    │
│ │   R$45.000 • 60 dias          │    │
│ ├──────────────────────────────┤    │
│ │ ● Empresa Beta ← selecionado  │    │
│ │   R$52.000 • 45 dias          │    │
│ └──────────────────────────────┘    │
│                                      │
│ ⚠️ Voto vinculado à unidade.        │
│ Irreversível após confirmar.         │
│                                      │
│ [Confirmar voto →]                   │
└──────────────────────────────────────┘
```

### Estados do voto

- `pending` (votação aberta, ainda não votou)
- `cast` (votou — exibe "✅ voto registrado")
- `proxy_used` (procurador votou em nome — replicado automaticamente)
- `presencial_assistido` (síndico/administradora lançou em nome do morador via terminal — R57 §6)
- `auto_abstention` (timeout 15min sem voto após votação aberta)

### Painel do Síndico — Controle de votação

```
┌──────────────────────────────────────────┐
│ VOTAÇÃO PAUTA 3 • aberta há 2:34        │
│                                          │
│ ┌─Status real-time────────────────────┐ │
│ │ Total aptos: 72  Votaram: 38        │ │
│ │ Faltam: 34                          │ │
│ │ Empresa A: ████████░░ 60% (16 unid) │ │
│ │ Empresa B: ████░░░░░░ 25% (10 unid) │ │
│ │ Empresa C: ██░░░░░░░░ 15% (6 unid)  │ │
│ │ Abstenção: 6 unidades               │ │
│ │ Quórum atingido: ✅ sim             │ │
│ └─────────────────────────────────────┘ │
│                                          │
│ [🔄 Zerar e refazer]  [⏹ Encerrar voto]│
│                                          │
│ Voto presencial assistido:               │
│ [+ Lançar voto manual por unidade]      │
└──────────────────────────────────────────┘
```

### Regras (ASM-014, ASM-015, ASM-016)

- 1 voto por (agenda_item, voter) — UNIQUE DB constraint (INV-071).
- Voto imutável pós-envio (INV-039/INV-071).
- Auto-save Redis real-time (R57 §19-§20) — recovery < 1min em crash.
- Reabertura permitida pré-homologação (R57 §13-§14): síndico zera + redoes; nova `voting_session_id` distinta (preserva histórico).
- Procuração: voto replicado automaticamente para unidade representada (peso fração) — INV-077.
- Ausência momentânea 15min → abstenção automática (`auto_abstention=true` flag).

---

## Voto Presencial Assistido (ASM-016 + R57 §6)

**Rota**: `/assembleias/:id/live/voto-presencial-assistido` (Síndico/Administradora)

```
┌──────────────────────────────────────────┐
│ VOTO PRESENCIAL ASSISTIDO                │
│ Pauta 3: Escolha Fornecedor              │
│                                          │
│ Para uso quando morador entrou pelo     │
│ terminal de apoio (sem app).             │
│                                          │
│ Unidade: [Bloco A] [Apt 105] 🔍         │
│ ✓ Encontrado: João Silva (apto)         │
│                                          │
│ Voto: ( ) Empresa A  (●) Empresa B  ( ) C│
│                                          │
│ Termo de voto assinado pelo morador:     │
│ ┌────────────────────────────────────┐  │
│ │ "Declaro que exerço este voto em   │  │
│ │  nome da unidade vinculada ao meu  │  │
│ │  acesso, sob minha                 │  │
│ │  responsabilidade."                │  │
│ │ [✓] Morador confirma (assinatura   │  │
│ │     digital biométrica/PIN local)  │  │
│ └────────────────────────────────────┘  │
│                                          │
│ Operador: Carlos (síndico)               │
│ Terminal: TERM-001-LOBBY                 │
│ Horário: 19:42:13                        │
│                                          │
│ [Cancelar]            [Lançar voto →]   │
└──────────────────────────────────────────┘
```

### Regras (R57 §6-§8 absorção Fase C)

- `origin = presencial_assistido` no Vote.
- Campos extra: `{operator_id, vote_type, unit, terminal_session_id}`.
- `termo_de_voto` obrigatório, assinado pelo morador (HMAC-keyed, audit_trail).
- Audit trail completo (ASM-### auditoria 20+ eventos — Fase E).

---

## Status dos Itens (ASM-### Fase E)

Estados terminais do item de pauta após discussão/votação:

- `RESOLVIDO` — votação encerrada com resultado válido (aprovado/rejeitado).
- `ADIADO` — síndico decide adiar para próxima assembleia.
- `NÃO_RESOLVIDO` — quórum não atingido / votação inconclusiva.
- `PREJUDICADO` — perdeu sentido (ex: já resolvido em outro item).

Cada item registra:
- `hora_abertura`
- `hora_entrada_votacao`
- `hora_encerramento`
- `duracao_total_discussao`

---

## Componentes

`LiveAssemblyPanel`, `MesaDiretoraForm`, `AuditorRegistration`, `InstitutionalPreRollPlayer`, `TelaoDualArea`, `TelaoPresentationArea`, `TelaoOperationalPanel`, `SpeechQueueWidget`, `SpeechTimer`, `RaiseHandButton`, `LiveVotingForm`, `VoteAssistedForm`, `VotingControlPanel`, `ItemStatusBadge`, `LivekitRoomProvider`, `LivekitMicController`.

## Cross-links

- [[./management|management]]
- [[./voting|voting]] — formulário simplificado de voto
- [[./check-in|check-in]] — entrada na sessão
- [[./pre-assembly|pre-assembly]] — preparação anterior
- [[./post-assembly|post-assembly]] — encerramento e ata
- [[./contingency-mode|contingency-mode]] — fallback offline
- [[../../../../04-requirements/functional/assembly|functional/assembly]] — ASM-014/015/016/017/018/019/020
- [[../../../../01-domain/aggregates/Assembly|Assembly]]
- [[../../../../01-domain/aggregates/AgendaItem|AgendaItem]]
- [[../../../../01-domain/aggregates/Vote|Vote]] (D-130 entity)
- [[../../../../01-domain/aggregates/AssemblyAdministratorLink|AssemblyAdministratorLink]] (D-127)
- [[../../../../STATE]] — D-127, D-130, D-132, D-133

## Pendências UX/técnicas

- ⚠️ **Microfone alternado**: Livekit API permite mute/unmute programático, mas algoritmo de "captação ótima" precisa de spec adicional (sugerir: VAD + RMS).
- ⚠️ **Transcrição**: Whisper local vs Speechmatics cloud — ADR pendente (custo).
- ⚠️ **Auditor role efêmera (D-132)**: criação automática de Membership ao iniciar assembleia + auto-revogação em End().
- ⚠️ **Voto presencial assistido**: spec UX detalhado da assinatura biométrica do morador no terminal (provider local: Touch ID nativo do tablet?).
