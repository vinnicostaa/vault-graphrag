---
title: Contingency Mode — UI Spec
type: ui-spec
tags: [ui-catalog, assembly, master-sindico, frontend, contingencia, offline, fallback]
bc: assembly
source: _chaos/ARQUITETURA-ASSEMBLEIA.pdf §11 + DIAGRAMA-VISUAL.pdf
absorbed_in: 2026-04-25 — Fase E
status: absorbed
---

# Contingency Mode — Modo de Contingência (Fallback Offline)

> **Origem**: 3 PDFs Master Síndico §11 — destilados em Fase E (2026-04-25).
> **Backend**: ASM-022 (modo offline + auto-sync), ASM-031 (Auto-save Redis).

> Sprint 4 · App `assembly` · Rotas: `/assembleias/:id/contingencia`
> Perfis: Síndico (ativador), Administradora (ativador alternativo), Morador (visualização degradada).

---

## Filosofia

O Modo de Contingência **reduz risco operacional** quando o ambiente digital falha. Garante que a assembleia possa continuar mesmo em:

- Queda de internet
- Falha de dispositivo
- Impossibilidade temporária de votação digital

A assembleia **não é cancelada** — o registro continua, mas alguns mecanismos digitais são substituídos por voto presencial assistido + reconciliação posterior.

---

## Tela 1 — Ativação do Modo Contingência

**Rota**: `/assembleias/:id/contingencia/ativar` (modal a partir do painel ao vivo)

**Quem pode ativar**: Síndico OU Administradora (D-127 com capability `assembly.contingency.activate`).

```
┌──────────────────────────────────────────────────┐
│  ⚠️ ATIVAR MODO DE CONTINGÊNCIA                  │
│                                                  │
│  Esse modo é para situações operacionais        │
│  graves. Use apenas se necessário.               │
│                                                  │
│  Motivo *                                        │
│  ( ) Queda de internet                           │
│  ( ) Falha de dispositivo                        │
│  ( ) Impossibilidade votação digital             │
│  (●) Outro:                                      │
│      [Descreva o motivo...]                      │
│                                                  │
│  Item afetado *                                  │
│  [Pauta 3: Escolha Fornecedor   ▼]              │
│  ⓘ Você pode entrar em contingência por item    │
│    específico ou para a assembleia inteira.      │
│                                                  │
│  ☑ Estou ciente de que:                          │
│    • Será registrado em audit trail              │
│    • Aparecerá em relatórios e ata               │
│    • Voto continuará via voto presencial         │
│      assistido (síndico/admin lança no painel)   │
│                                                  │
│  Responsável: Carlos Pereira (síndico)           │
│  Horário: 19:48:23                               │
│                                                  │
│  [✗ Cancelar]      [⚠️ Ativar contingência →]   │
└──────────────────────────────────────────────────┘
```

### Regras (ASM-022 + R11)

- Ativação manual obrigatória (não automática — escolha consciente do operador).
- Motivo obrigatório (registrado em audit trail).
- Item-específico OU assembleia-inteira.
- Audit trail event: `ContingencyModeActivated {responsible_id, reason, item_id?, activated_at}`.

---

## Tela 2 — Painel Durante Contingência

**Rota**: `/assembleias/:id/live/painel` (banner adicional)

```
┌──────────────────────────────────────────────────────┐
│ 🔴 MODO DE CONTINGÊNCIA ATIVO                        │
│ Pauta 3 • Ativado por Carlos às 19:48               │
│ Motivo: Queda de internet detectada                  │
│ [Ver detalhes →]   [⚠️ Desativar contingência]      │
├──────────────────────────────────────────────────────┤
│  (resto do painel ao vivo normal — degraded mode)    │
│                                                      │
│  Votação atual:                                      │
│  ⚠️ Voto digital pelo app: indisponível              │
│  ✅ Voto presencial assistido: disponível            │
│                                                      │
│  ┌─Lançar voto manual─────────────────────────────┐│
│  │ Unidade: [A-302]  Voto: [Empresa Beta ▼]      ││
│  │ Termo de voto pelo morador: [Captura digital] ││
│  │ Operador: Carlos                               ││
│  │ [+ Lançar voto manual]                         ││
│  └────────────────────────────────────────────────┘│
└──────────────────────────────────────────────────────┘
```

### Regras

- Banner visível e persistente (não pode ser dismissado durante contingência).
- Voto pelo app: bloqueado (UX cleansed) — morador vê tela "Modo contingência ativo, dirija-se ao terminal".
- Voto presencial assistido: único caminho de voto durante contingência.
- Continuação da assembleia normal: fila de fala, telão (em modo cache local), status de itens.

---

## Tela 3 — Tela do Morador em Contingência

**Rota**: `/assembleias/:id/live` (Morador — modal/banner)

```
┌──────────────────────────────────────────┐
│  ⚠️ MODO DE CONTINGÊNCIA                 │
│                                          │
│  A assembleia está com problemas         │
│  técnicos temporários.                   │
│                                          │
│  Suas ações:                             │
│  • Para votar: dirija-se ao terminal     │
│    de apoio na entrada                   │
│  • A administradora ou síndico           │
│    registrará seu voto em seu nome,      │
│    com sua confirmação digital.          │
│                                          │
│  Sua presença está mantida.              │
│  Outros itens continuam normalmente      │
│  quando contingência for desativada.     │
│                                          │
│  [OK, entendi]                           │
└──────────────────────────────────────────┘
```

---

## Tela 4 — Auto-Save Redis (NFR — invisível ao usuário)

> **Backend**: ASM-031 — auto-save em Redis local quando estado da assembly muda (votação aberta, presença, voto).

```
Keys padrão:
  assembly:{id}:state:current_item
  assembly:{id}:state:active_voting
  assembly:{id}:offline_votes:{user_id}
  assembly:{id}:speech_queue
  assembly:{id}:contingency_active
```

### Recuperação após crash

1. Cliente perdeu conexão WebSocket.
2. Estado salvo em Redis durante 1h pós-encerramento.
3. Backend reconnect handler restaura state.
4. Cliente atualiza UI < 1 min.
5. Conflict resolution: último voto válido (LWW timestamp).

---

## Tela 5 — Histórico de Contingências (Pós-assembleia)

**Rota**: `/assembleias/:id/historico-contingencia` (Síndico + Administradora read-only)

```
┌────────────────────────────────────────────────────┐
│  HISTÓRICO DE CONTINGÊNCIAS                        │
│  Assembleia Ordinária 2026/1                       │
│                                                    │
│  ┌──────────────────────────────────────────────┐ │
│  │ Pauta 3: Escolha Fornecedor                  │ │
│  │ Status: ✅ Resolvido em modo contingência   │ │
│  │ Ativado: 19:48:23 por Carlos (síndico)      │ │
│  │ Motivo: Queda de internet detectada          │ │
│  │ Normalização: 19:54:11 (06:48 minutos)       │ │
│  │ Votos lançados: 12 (presencial assistido)    │ │
│  │ Quórum atingido: ✅ sim                      │ │
│  │ Resultado: Empresa Alpha aprovada            │ │
│  │ [Ver detalhe completo →]                     │ │
│  └──────────────────────────────────────────────┘ │
│                                                    │
│  ⓘ Marcação de contingência aparece no relatório   │
│  individual do item (R11.3 transparência)          │
└────────────────────────────────────────────────────┘
```

### Regras (R11 — Transparência)

Cada contingência é registrada com:
- Que houve contingência
- Horário de início
- Horário de normalização (se houver)
- Quem ativou
- Motivo
- Itens afetados
- Votos lançados em modo contingência

Marcação aparece em:
- Ata oficial homologada
- Relatório de auditoria (ASM-026)
- Histórico de cada item

---

## Tela 6 — Desativação de Contingência

**Rota**: `/assembleias/:id/contingencia/desativar` (modal)

```
┌──────────────────────────────────────────────────┐
│  DESATIVAR MODO DE CONTINGÊNCIA                  │
│                                                  │
│  Pauta atual: 3 (Escolha Fornecedor)             │
│  Tempo em contingência: 6 min 48s                │
│                                                  │
│  Verificações:                                   │
│  ✅ Conexão de internet estabilizada             │
│  ✅ WebSocket reconectado                        │
│  ✅ Auto-sync de estado pendente                 │
│                                                  │
│  Votos lançados em contingência: 12              │
│  • Todos serão preservados                       │
│  • Marcação `origin=presencial_assistido`        │
│  • Aparecerão no relatório como tal              │
│                                                  │
│  [Cancelar]              [Desativar contingência]│
└──────────────────────────────────────────────────┘
```

---

## Componentes

`ContingencyActivateModal`, `ContingencyBanner`, `ContingencyResidentInfo`, `ContingencyHistoryList`, `ContingencyEventCard`, `OfflineVoteSyncIndicator`.

## Cross-links

- [[./live-session|live-session]] — painel ao vivo (banner contingência integrado)
- [[./check-in|check-in]] — terminal de apoio (caminho durante contingência)
- [[./post-assembly|post-assembly]] — relatórios pós (histórico contingência)
- [[../../../../04-requirements/functional/assembly|functional/assembly]] — ASM-022, ASM-031
- [[../../../../01-domain/aggregates/Assembly|Assembly]]
- [[../../../../STATE]] — D-127

## Pendências técnicas

- ⚠️ **Detecção automática de problemas**: heartbeat WebSocket vs polling — definir threshold (ex: 30s sem heartbeat = sugerir contingência).
- ⚠️ **Sync conflict resolution**: votos em offline_votes Redis se conflito com votos pelo app — política LWW vs reject (definir).
- ⚠️ **Capability `assembly.contingency.activate`**: confirmar inclusão no `pg.assembly.administrator_delegation` (D-116 IAM).
- ⚠️ **Notificação ao morador**: push notification quando contingência ativada/desativada (UX nice-to-have).
- ⚠️ **Mode degradation**: se a contingência for muito longa (> 30 min), sugerir adiar item — modal automático.
