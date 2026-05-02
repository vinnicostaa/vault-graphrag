---
title: Post-Assembly — UI Spec
type: ui-spec
tags: [ui-catalog, assembly, master-sindico, frontend, ata, relatorios, timeline]
bc: assembly
source: _chaos/ARQUITETURA-ASSEMBLEIA.pdf §6-7 + DIAGRAMA-VISUAL.pdf §5-6 + MÓDULO-TELAS.pdf
absorbed_in: 2026-04-25 — Fase E
status: absorbed
---

# Post-Assembly — Encerramento, Ata, Relatórios e Histórico

> **Origem**: 3 PDFs Master Síndico — destilados em Fase E (2026-04-25).
> **Backend**: ASM-023 (Encerramento), ASM-024 (Ata gerada), ASM-025 (Imutabilidade), ASM-026 (Relatórios), ASM-027 (Homologação), ASM-028 (Score), ASM-029 (Notificações pós), ASM-030 (Widget).

> Sprint 4 · App `assembly` · Rotas: `/assembleias/:id/encerrar`, `/assembleias/:id/ata`, `/assembleias/:id/relatorios`, `/assembleias/:id/historico`
> Perfis: Síndico (encerramento, homologação), Administradora (homologação alternativa), Morador (visualização ata + recibo de voto).

---

## Tela 1 — Encerrar Assembleia (Síndico/Administradora)

**Rota**: `/assembleias/:id/encerrar` (modal a partir do painel ao vivo)

```
┌──────────────────────────────────────────────────┐
│  ENCERRAR ASSEMBLEIA                             │
│                                                  │
│  Verificações pré-encerramento:                  │
│  ✅ Todos itens com status final (8/8)           │
│  ✅ Nenhuma votação aberta                       │
│  ✅ Presidente cadastrado: Carlos Pereira       │
│  ✅ Secretários cadastrados (2/3)                │
│                                                  │
│  ⚠️ Itens não-resolvidos:                       │
│  • Pauta 5 — Status: ADIADO                     │
│  • Pauta 7 — Status: PREJUDICADO                │
│                                                  │
│  Mensagem que será enviada aos moradores:       │
│  ┌──────────────────────────────────────────┐   │
│  │ "A assembleia foi encerrada em            │   │
│  │  15/03/2026, das 19:00 às 21:32. Os      │   │
│  │  resultados serão disponibilizados após   │   │
│  │  conferência. Obrigado pela participação." │  │
│  └──────────────────────────────────────────┘   │
│                                                  │
│  [✗ Cancelar]            [Encerrar definitivo →]│
└──────────────────────────────────────────────────┘
```

### Cascata pós-clique "Encerrar definitivo"

1. `Assembly.End()` → status `closed`.
2. **D-127**: cascade invalida todos `AssemblyAdministratorLink` (status=expired).
3. **D-132**: cascade invalida todas Memberships efêmeras `assembly_auditor`.
4. Procurações: ainda válidas por +24h (ASM-010 / INV-078).
5. Job async gera ata PDF (ASM-024).
6. Push/email/SMS notificação encerramento (ASM-029).
7. Evento NATS `assembly.closed` → handler institucional cria TimelineEntry (INS-011).
8. Job calcula score de transparência (ASM-028).
9. Abre votação de homologação (ASM-027).

---

## Tela 2 — Registro Final (Pós-Encerramento)

**Rota**: `/assembleias/:id/registro-final` (read-only, gerada automaticamente)

```
┌────────────────────────────────────────────────────────┐
│  REGISTRO FINAL — Assembleia Ordinária 2026/1          │
│  Status: ENCERRADA • 15/03/2026                        │
│                                                        │
│  ┌─DATAS E DURAÇÃO──────────────────────────────────┐ │
│  │ Início:    15/03/2026 19:02                      │ │
│  │ Término:   15/03/2026 21:32                      │ │
│  │ Duração:   2h 30min                              │ │
│  └──────────────────────────────────────────────────┘ │
│                                                        │
│  ┌─MESA DIRETORA────────────────────────────────────┐ │
│  │ Presidente: Carlos Pereira                       │ │
│  │ Secretários: Ana Lima, Pedro Costa               │ │
│  │ Administradora: Admin XYZ Ltda — Maria Souza     │ │
│  │ Auditor: (não designado)                         │ │
│  │ Fornecedores: Empresa Alpha, Beta Recuperação    │ │
│  └──────────────────────────────────────────────────┘ │
│                                                        │
│  ┌─PRESENÇA─────────────────────────────────────────┐ │
│  │ Total presentes:        72                       │ │
│  │ Presenciais:            68                       │ │
│  │ Online:                 — (M4+)                  │ │
│  │ Ausências/desconexões:  4 (após check-in)        │ │
│  │ Lista CPF/Bloco/Unidade: [Ver detalhe →]         │ │
│  └──────────────────────────────────────────────────┘ │
│                                                        │
│  ┌─VOTOS─────────────────────────────────────────────┐│
│  │ Total válidos:          312                      ││
│  │ Por procuração:         18                       ││
│  │ Presenciais assistidos: 4 (terminal)             ││
│  │ Bloqueados (inadimplência): 12                  ││
│  │ Liberações manuais:     2                        ││
│  │ Por item: [Ver detalhe →]                        ││
│  └──────────────────────────────────────────────────┘│
│                                                        │
│  ┌─HOMOLOGAÇÕES─────────────────────────────────────┐ │
│  │ Realizadas: 5/8                                  │ │
│  │ Pendentes:  3 (aguardando síndico)               │ │
│  └──────────────────────────────────────────────────┘ │
│                                                        │
│  ┌─DURAÇÃO POR ITEM─────────────────────────────────┐ │
│  │ Pauta 1: 25min                                   │ │
│  │ Pauta 2: 18min                                   │ │
│  │ Pauta 3: 35min                                   │ │
│  │ ...                                              │ │
│  └──────────────────────────────────────────────────┘ │
│                                                        │
│  [📊 Score de Transparência: 87/100]                  │
│  [📄 Baixar ata PDF]  [📥 Exportar relatórios]       │
└────────────────────────────────────────────────────────┘
```

### Regras (ASM-### Fase E novo + R6.2)

Campos canônicos do registro final: data início/término, duração, presidente, secretários, administradora, auditor, fornecedores, total presentes (online/presenciais), ausências/desconexões, lista CPF/bloco/unidade, total votos válidos, votos por item, votos por procuração, votos presenciais assistidos, unidades bloqueadas inadimplência, liberações manuais, homologações realizadas/pendentes, duração de cada item.

---

## Tela 3 — Homologação (Votação obrigatória pós-ata) — ASM-027

**Rota**: `/assembleias/:id/homologar` (Síndico OU Administradora — D-127)

```
┌────────────────────────────────────────────────────┐
│  HOMOLOGAR VOTAÇÃO                                 │
│  Pauta 3: Escolha Fornecedor Fachada               │
│                                                    │
│  Resultado oficial:                                │
│  ✅ Aprovado: Empresa Alpha (60% — 16 unidades)   │
│  ⚪ Empresa Beta: 25% — 10 unidades               │
│  ⚪ Empresa Gamma: 15% — 6 unidades               │
│  ⚫ Abstenções: 6                                 │
│  Quórum atingido: ✅ 68% (mín 50%)               │
│                                                    │
│  Aprova esta votação como oficial e final?        │
│                                                    │
│  Quem pode homologar (D-127):                      │
│  • Síndico (você)                                  │
│  • Administradora com permissão                    │
│    `pg.assembly.administrator_delegation`          │
│                                                    │
│  ⚠️ Após homologar:                               │
│  • Resultado torna-se oficial e imutável           │
│  • Ata para este item é congelada                  │
│  • Notificação enviada aos moradores               │
│                                                    │
│  [✗ Reabrir votação]      [✓ Homologar votação →] │
└────────────────────────────────────────────────────┘
```

### Estados pós-homologação

- Item congela em `RESOLVIDO` (ou `NÃO_RESOLVIDO` se quórum não atingido).
- `Vote.isHomologated=true` em todos os votos do item (D-130 entity).
- Ata global pode ser homologada após todos os itens (ASM-027).

---

## Tela 4 — Visualização da Ata Homologada (Morador)

**Rota**: `/assembleias/:id/ata` (Morador — read-only)

```
┌──────────────────────────────────────────────────┐
│  ATA — Assembleia Ordinária 2026/1               │
│  ✅ HOMOLOGADA em 16/03/2026 às 14:23            │
│  🔒 IMUTÁVEL (correção via adendo)               │
│                                                  │
│  📄 Ata Oficial (PDF assinado digitalmente)      │
│  └ Assinatura síndico: ✅ ICP-Brasil (Lei 14.063)│
│  └ Assinatura secretário: ✅                     │
│  └ Hash SHA-256: 3a7b...e9f1                    │
│  └ Data carimbo: 2026-03-16T14:23:01Z            │
│                                                  │
│  [📥 Baixar PDF]   [👁 Ver online]              │
│                                                  │
│  CONTEÚDO RESUMIDO:                              │
│  ┌────────────────────────────────────────────┐ │
│  │ Data: 15/03/2026 19:00-21:32               │ │
│  │ Local: Salão festas Bloco A                │ │
│  │ Quórum atingido: ✅                        │ │
│  │                                            │ │
│  │ DELIBERAÇÕES:                              │ │
│  │ 1. Cont. fachada → APROVADO               │ │
│  │ 2. Paisagismo    → APROVADO               │ │
│  │ 3. Forn. fachada → Empresa Alpha          │ │
│  │ 4. Eleição       → ADIADO                 │ │
│  │ ...                                        │ │
│  └────────────────────────────────────────────┘ │
│                                                  │
│  [📊 Score Transparência: 87/100]               │
│                                                  │
│  Seu voto pessoal:                              │
│  • Pauta 1: APROVAR ✓                          │
│  • Pauta 2: APROVAR ✓                          │
│  • Pauta 3: Empresa Beta                        │
│  • Pauta 4: Abstenção (auto após 15min)         │
│  [Recibo individual de voto →]                  │
└──────────────────────────────────────────────────┘
```

### Regras (ASM-024, ASM-025)

- Ata gerada automaticamente após `Assembly.End()` (job async).
- Conteúdo: data, hora, local, presença anonimizada, pauta + resultado, falas (texto/resumo), procurações usadas, assinatura digital síndico (M4+).
- Imutável após homologação (`homologated_at != NULL` → lock).
- **NOVO Fase E**: hash SHA-256 + carimbo do tempo (Lei 14.063 ICP-Brasil M4+).
- Correção pós-homologação: TimelineEntry adendo (INS-016) — nunca edita ata.

### Recibo individual de voto

```
┌──────────────────────────────────────┐
│  RECIBO DE VOTO                      │
│                                      │
│  Assembleia: Ordinária 2026/1        │
│  Unidade: Bloco A — Apt 302          │
│  Morador: João Silva (CPF *.789-00)  │
│                                      │
│  Pauta 1: APROVAR (peso 0.0083)      │
│  Pauta 2: APROVAR (peso 0.0083)      │
│  Pauta 3: Empresa Beta (peso 0.0083) │
│  Pauta 4: Abstenção (auto)           │
│                                      │
│  Origem: app                         │
│  IP: 192.168.x.x                     │
│  Timestamp: 15/03 19:32              │
│  Device: iPhone 14 Pro               │
│                                      │
│  Hash do voto: a3f...c91             │
└──────────────────────────────────────┘
```

---

## Tela 5 — Relatórios Exportáveis (ASM-026)

**Rota**: `/assembleias/:id/relatorios` (Síndico + Administradora)

```
┌──────────────────────────────────────────────────────┐
│  RELATÓRIOS DA ASSEMBLEIA                            │
│                                                      │
│  Selecione um relatório:                             │
│                                                      │
│  ┌────────────────────────────────────────────────┐ │
│  │ 📋 Presença                                    │ │
│  │ Lista anonimizada (unidade + fração + horário) │ │
│  │ [Baixar CSV]  [Baixar PDF]                     │ │
│  ├────────────────────────────────────────────────┤ │
│  │ 🗳 Votos                                       │ │
│  │ Resultado detalhado por item, votos por opção  │ │
│  │ [Baixar CSV]  [Baixar PDF]                     │ │
│  ├────────────────────────────────────────────────┤ │
│  │ 📜 Procurações                                 │ │
│  │ Lista das proxies usadas + status validação    │ │
│  │ [Baixar CSV]  [Baixar PDF]                     │ │
│  ├────────────────────────────────────────────────┤ │
│  │ 🎤 Falas                                       │ │
│  │ Registro, tempo por morador, tópicos          │ │
│  │ [Baixar CSV]  [Baixar PDF]                     │ │
│  ├────────────────────────────────────────────────┤ │
│  │ 💸 Inadimplência e liberações                  │ │
│  │ Bloqueios + liberações manuais (audit trail)   │ │
│  │ [Baixar CSV]  [Baixar PDF]                     │ │
│  ├────────────────────────────────────────────────┤ │
│  │ 🔍 Trilha de Auditoria                         │ │
│  │ 20+ eventos: criação → encerramento            │ │
│  │ [Baixar CSV]  [Baixar PDF]                     │ │
│  ├────────────────────────────────────────────────┤ │
│  │ ✅ Decisões / Consolidado executivo            │ │
│  │ Resumo 1-2 páginas                             │ │
│  │ [Baixar CSV]  [Baixar PDF]                     │ │
│  ├────────────────────────────────────────────────┤ │
│  │ 📊 Score de Transparência                      │ │
│  │ 10 componentes detalhados (0-100)              │ │
│  │ [Baixar CSV]  [Baixar PDF]                     │ │
│  └────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────┘
```

### Regras (ASM-026)

- 6+ tipos canônicos (PDFs PDFs §7.1 lista CSVs): presença, votos, procurações, item-por-item, inadimplência+liberações, trilha auditoria, consolidado.
- **Fase E NOVO**: relatório de **falas** (registrado de fala por morador, tempo, tópicos — derivado do controle de fala ASM-019).
- Anonimização configurável: lista de presença sem nomes (só unidade/fração) por padrão.
- Acesso: síndico + administradora (D-127).
- Exportação CSV ou PDF.

---

## Tela 6 — Linha do Tempo / Histórico (ASM-### NOVO)

**Rota**: `/condominios/:id/historico-assembleias` (Síndico + Administradora + Morador read-only)

```
┌──────────────────────────────────────────────────────┐
│  HISTÓRICO DE ASSEMBLEIAS — Condomínio Aurora       │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │ 📅 15/03/2026 — Ordinária 2026/1            │   │
│  │ 🟢 ENCERRADA • Score: 87/100                 │   │
│  │ 📄 Edital  📋 Pauta  📊 Relatórios          │   │
│  │ Deliberações: 8 itens (5 aprovados, 1 adiado)│   │
│  │ [Detalhes →]                                 │   │
│  ├──────────────────────────────────────────────┤   │
│  │ 📅 10/01/2026 — Extraordinária Fachada       │   │
│  │ 🟢 ENCERRADA • Score: 92/100                 │   │
│  │ ...                                          │   │
│  ├──────────────────────────────────────────────┤   │
│  │ 📅 12/03/2025 — Implantação                  │   │
│  │ 🟢 ENCERRADA • Score: 75/100                 │   │
│  │ ...                                          │   │
│  └──────────────────────────────────────────────┘   │
│                                                      │
│  📊 Evolução do score (12 últimas):                 │
│  ╭────────────────────────────────╮                │
│  │ 100 ┤    ●                     │                │
│  │  90 ┤  ● │ ●                   │                │
│  │  80 ┤●─┘   ● ─●─●              │                │
│  │  70 ┤        ╲ ●               │                │
│  │  60 ┤         ●                │                │
│  │     └────────────────────       │                │
│  ╰────────────────────────────────╯                │
└──────────────────────────────────────────────────────┘
```

### Regras

- Linha do tempo de cada condomínio (R7.3): assembleia, edital, pauta, relatórios, exportações, status final, presença, deliberações.
- Alimentada via TimelineEntry handler institucional (INS-011, INS-038).
- Score de transparência exibido com gráfico evolutivo (12 últimas — ASM-030).

---

## Mensagem automática pós-encerramento (R6.3)

Push + email + SMS opt-in disparados ao encerrar:

> "A assembleia foi encerrada em [data], das [hora início] às [hora término]. Os resultados serão disponibilizados após conferência. Obrigado pela participação."

E após homologação completa (ASM-029):

> "Ata homologada disponível. Score de transparência: X/100. Baixe a ata em [link]."

---

## Componentes

`AssemblyEndModal`, `EndCheckList`, `RegistroFinalReadOnly`, `HomologationVotingModal`, `MinutesPDFViewer`, `MinutesIntegritySignature`, `IndividualVoteReceipt`, `ReportsList`, `ReportDownloadButton`, `AssemblyTimelineList`, `TransparencyScoreEvolutionChart`.

## Cross-links

- [[./management|management]]
- [[./live-session|live-session]]
- [[./transparency-score|transparency-score]] — detalhamento do score
- [[./administrator-panel|administrator-panel]]
- [[../../../../04-requirements/functional/assembly|functional/assembly]] — ASM-023/024/025/026/027/029
- [[../../../../01-domain/aggregates/Assembly|Assembly]]
- [[../../../../01-domain/aggregates/Minutes|Minutes]]
- [[../institutional/transparency|institutional/transparency]] — Painel Transparência (linha do tempo)

## Pendências

- ⚠️ Template jurídico de ata canônico (revisão advogado BR — pendência ASM-024).
- ⚠️ ICP-Brasil provider (Bry, ITI, AC SERPRO) — ADR M4+ assinatura digital.
- ⚠️ Hash SHA-256 + carimbo do tempo (RFC 3161 TSA) — spec técnica para M4+.
