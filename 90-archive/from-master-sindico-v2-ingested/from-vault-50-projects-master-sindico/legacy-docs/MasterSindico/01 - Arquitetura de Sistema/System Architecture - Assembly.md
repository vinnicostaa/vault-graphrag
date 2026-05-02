---
title: System Architecture — Assembly Domain
parent: System Architecture
domain: ASSEMBLY
status: Not Started
tags:
  - mastersindico
  - architecture
  - assembly
  - websocket
  - realtime
  - ddd
---

# HIGH-DOMAIN: Assembly

> **Status:** Não iniciado — Sprint 5+ (módulo mais complexo)
> **Tenant Isolation:** `condominium_id`
> **Módulos:** 1 externamente, 3 fases internamente
> **Source of Truth:** Requirements 48-60.12, Decision 1 (RESOLVED)
> **Infraestrutura:** WebSocket + NATS JetStream + Redis Pub/Sub

---

## Visão Geral

O Assembly é o HIGH-DOMAIN mais complexo do sistema. Cobre todo o ciclo de vida de assembleias condominiais: preparação, execução ao vivo com votação real-time, e pós-assembleia com relatórios e transparência.

> [!WARNING]
> **Divergência corrigida:** O blueprint v2.0 tratava como "MVP presencial com polling 5s". A Decision 1 foi RESOLVED como **Live Assembly (Option B)** com WebSocket real-time, telão, fila de fala, e votação ao vivo.

> Veja o canvas visual: [[Assembly Engine.canvas]]

```
ASSEMBLY (Tenant: condominium_id)
├── Pré-Assembleia    — Config, pauta, edital, ciência, polls, procuração
├── Live-Day          — Check-in, votação, fala, telão, contingência
└── Pós-Assembleia    — Finalização, relatórios, transparência, homologação
```

---

## Fase 1: Pré-Assembleia

### Configurador (Req 48)
| Campo | Detalhe |
|-------|---------|
| Tipo | ordinária, extraordinária, implantação |
| Modalidade | presencial (MVP), híbrida, online (futuro) |
| Tempo de fala | 2 ou 3 minutos (configurável) |
| Extensão de fala | Permitida ou não (toggle) |
| Quorum | simple_majority, qualified_majority, unanimous |
| Administradora | Opcional, com acesso delegado |

### Pauta / Agenda (Req 49)
| Aspecto | Detalhe |
|---------|---------|
| **Entities** | AgendaItem |
| **Tipos de item** | informativo, votacao_simples, votacao_fracao_ideal, escolha_fornecedor, eleicao |
| **Ordenação** | drag-and-drop, display_order |
| **Lock** | Itens trancados após publicação do edital |
| **Attachments** | Documentos anexos via R2 |

### Edital (Req 50.1)
- Geração automática a partir da pauta configurada
- Distribuição via email + push para todos os moradores
- Prazo legal de antecedência

### Ciência e Participação (Req 50.2, 50.3)
- Morador confirma ciência do edital
- Registro de participação (presencial/online/delegação)
- Tracking de quem leu vs quem não leu

### Polls Preliminares (Req 51)
- Consultas prévias sobre itens da pauta
- Resultado informativo (não vinculante)
- Ajuda síndico a calibrar expectativas

### Procuração / Proxy (Req 52)
- Morador delega voto a outro morador
- Validação: 1 procurador pode representar máx N moradores
- Registro com aceite do procurador

### Análise de Fornecedor (Req 53)
- Exibe propostas validadas vinculadas a itens da pauta
- Vídeos de fornecedores (se autorizados)
- Dados de avaliações anteriores

### Painel Administradora (Req 54)
- Acesso delegado para administradora do condomínio
- Visualização da pauta, documentos, configurações
- Sem poder de voto

### Simulador de Quorum (Req 55)
- Calcula quorum necessário por tipo de votação
- Baseado em unidades totais e frações ideais
- Mostra cenários: "se X moradores comparecerem..."

---

## Fase 2: Live-Day (Real-Time)

### Infraestrutura WebSocket

```
Client (Telão / App Morador / Presidente)
    │ WSS + JWT auth
    ▼
Fastify WebSocket (@fastify/websocket)
    │ Room: assembly:{id}
    │
    ├── assembly:{id}:telao     → presença, quorum, item atual, fila
    ├── assembly:{id}:voting    → contagem votos, status
    ├── assembly:{id}:speech    → fila de fala, timer
    ├── assembly:{id}:checkin   → presença real-time
    └── assembly:{id}:control   → ações do presidente
```

**Fluxo de eventos:**
1. Ação ocorre (voto, check-in, fala)
2. Persiste PostgreSQL + outbox (TX atômica)
3. Outbox publisher → NATS `assembly.live.*`
4. Consumer `assembly-broadcaster` → Redis pub/sub
5. Fastify instances → push para WebSocket clients

### Check-in (Req 56)
| Método | Detalhe |
|--------|---------|
| **App** | Botão no app com geolocation |
| **QR Code** | Escaneado na entrada |
| **Terminal** | Para assembleias maiores, terminal dedicado |
| **Status** | presente, ausente_momentaneo, desconectado_saiu |

### Votação em Tempo Real (Req 57)

| Tipo | Peso | Detalhe |
|------|------|---------|
| **Simples** | 1 voto/unidade | Maioria simples |
| **Fração Ideal** | fracao_ideal_weight | Ponderado por fração |
| **Fornecedor** | 1 voto/unidade | Entre propostas validadas |
| **Eleição** | 1 voto/unidade | Candidatos a cargo |

**Opções:** favor, contra, abstenção (+ numeração para eleição)

**Regras:**
- Proxy: vota em nome do delegante (`is_proxy = true`)
- Presencial assistido: operador registra para morador sem app (`is_presencial_assistido = true`)
- Homologação: presidente confirma resultado (`is_homologated`)
- Unique constraint: `(assembly_id, agenda_item_id, morador_id)`

### Fila de Fala (Req 60.5)
- Timer configurável (2 ou 3 min)
- Extensão permitida ou não
- Ordem FIFO com prioridade para presidente
- Visual no telão com countdown

### Telão (Req 60.3)
**Layout 2 áreas:**
- **Área Informativa**: presença, quorum, item atual, fila de fala
- **Área de Conteúdo**: slides, PDFs, vídeos de fornecedor, resultado parcial

### Controles do Presidente (Req 60.2)
- Abrir/fechar votação
- Avançar item da pauta
- Pausar assembleia
- Ativar contingência
- Encerrar fala
- Homologar resultado

### Contingência (Req 60.10)
- Ativada quando: queda de internet, falha de sistema
- Registra votos em modo offline
- Sincroniza quando conexão restaurada
- Logging completo para auditoria

### Clock (Req 60.11)
- Relógio da assembleia visível no telão
- Duração total em andamento
- Timer de fala individual

### Termos Legais (Req 60.12)
- Checkbox de autorização de gravação
- Termos de participação
- Aceite obrigatório antes do check-in

---

## Fase 3: Pós-Assembleia

### Finalização (Req 58)
- Presidente encerra assembleia
- Sistema gera ata automática baseada nos registros
- Auto-publica resultado na Timeline (`assembly_result`)
- Emite event `institutional.governance.timeline-published`

### Relatórios (Req 59)
6 tipos de relatório:
1. **Presença**: Lista de check-ins com horário e método
2. **Votação**: Resultado por item com breakdown
3. **Procurações**: Delegações ativas e utilizadas
4. **Fala**: Registro de falas com duração
5. **Consolidado**: Visão geral completa
6. **Transparência**: Score detalhado

Exportação: CSV e PDF

### Indicador de Transparência (Req 60.9)
Score 0-100 calculado com 10 componentes:
1. Edital publicado com antecedência legal
2. Pauta disponível previamente
3. Quorum atingido
4. Votação registrada digitalmente
5. Procurações formalizadas
6. Tempo de fala respeitado
7. Ata gerada automaticamente
8. Relatórios disponíveis em 24h
9. Resultados publicados na Timeline
10. Conformidade com regimento

### Homologação (Req 60.6)
- Presidente homologa cada resultado de votação
- Após homologação, resultado é imutável
- Não-homologação gera pendência para resolver

### Notificações Automáticas (Req 60.8)
- Após encerramento: resumo para todos os moradores
- Ausentes: notificação com link para ata
- Síndico: relatório completo por email

---

## Eventos do Domínio (30+)

### Pré-Assembleia
- `assembly.config.created`
- `assembly.config.published` (edital)
- `assembly.config.agenda-item-added`
- `assembly.config.proxy-registered`
- `assembly.config.ciencia-confirmed`

### Live-Day
- `assembly.live.started`
- `assembly.checkin.morador-checked-in`
- `assembly.live.item-opened`
- `assembly.live.item-closed`
- `assembly.vote.cast`
- `assembly.vote.homologated`
- `assembly.speech.started`
- `assembly.speech.ended`
- `assembly.speech.extended`
- `assembly.live.contingency-activated`
- `assembly.live.contingency-resolved`
- `assembly.live.paused`
- `assembly.live.resumed`

### Pós-Assembleia
- `assembly.live.finalized`
- `assembly.config.report-generated`
- `assembly.config.transparency-calculated`
- `assembly.config.ata-published`

---

## Schema Core

```
assemblies:
  id, condominium_id, created_by
  title, assembly_type, scheduled_date/time, location
  quorum_type, speech_time_minutes, speech_extension_allowed
  modalidade (presencial/hibrida/online)
  status (em_preparacao/publicada/em_andamento/encerrada/cancelada)
  administradora_id (nullable)
  started_at, ended_at, total_duration_minutes
  contingency_active, transparency_score

assembly_agenda_items:
  id, assembly_id, title, description
  item_type, voting_required, quorum_required
  display_order, item_status
  opened_at, closed_at, attachments, is_locked

assembly_checkins:
  id, assembly_id, morador_id, unit_id
  check_in_method (app/qr_code/terminal)
  attendance_status, checked_in_at
  UNIQUE(assembly_id, morador_id)

assembly_votes:
  id, assembly_id, agenda_item_id, morador_id, unit_id
  vote_value ('favor'/'contra'/'abstencao'/'1'-'5')
  is_proxy, proxy_id
  is_presencial_assistido, operator_id
  fracao_ideal_weight (numeric, nullable)
  voted_at, is_homologated
  UNIQUE(assembly_id, agenda_item_id, morador_id)
```

---

## Dependências

```
AuthModule ← autenticação WebSocket
CondominiumModule ← tenant context + unidades
BillingModule ← plano permite assembleia?
ProposalModule ← análise de fornecedor na pauta
GovernanceModule ← publica resultados na Timeline
```

---

## Estimativa

| Fase | Sprints |
|------|---------|
| Pré-Assembleia | 1-2 |
| Live-Day (infra + check-in + votação) | 2 |
| Live-Day (fala + telão + contingência) | 1 |
| Pós-Assembleia | 1 |
| **Total** | **4-6** |

---

## Navegação

← [[System Architecture - Content]] | [[System Architecture]] | [[System Architecture - Infrastructure]] →
