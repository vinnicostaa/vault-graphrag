---
title: Voting System — UI Spec
type: ui-spec
tags: [ui-catalog, master-sindico, assembly, voting]
bc: assembly
source: _chaos/10-VOTING-SYSTEM.md (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# 10 — Sistema de Votação

> **Origem**: `_chaos/10-VOTING-SYSTEM.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Tradução aplicada**: vocabulário neutro; "Morador Pagante" removido (D-126).

> Sprint 2 · App `assembly` · Rota: `/assembleias/:id/votar/:pautaId`
> Vinculado à pauta da assembleia. **1 voto por unidade** (validação backend).

---

## Regras de negócio

- **Votação vinculada a pauta** de assembleia.
- **Tipos** (canônicos M3+):
  - Aprovar / Reprovar (binária)
  - Múltipla Escolha (Fornecedor A, B, C)
  - **Fração ideal DECIMAL** (votação proporcional — peso por unidade)
  - **Eleição** (síndico, conselho)
- **1 voto por unidade** (validação backend por unidade habitacional).
- **Hard paywall**: Morador `base` **NÃO vota** (D-058 / D-126). Para votar exige addon de plano (M3 — atualmente: morador é `base` gratuito → exige upgrade explícito quando feature liberada).

> Atenção: o doc original de fev/2026 dizia "Morador Pagante 4/ano". Hoje (D-126) `Morador Pagante` não existe mais — morador é sempre `base` gratuito. A "votação" como feature paga só está prevista no addon de tier futuro. No M3 atual a votação é gated por matriz `(role=resident, plan_tier=plus|pro)` quando produto evoluir; até lá, morador `base` vê tela de paywall com mensagem "Funcionalidade disponível em plano premium".

- **Janela de tempo**: só vota se evento estiver "Aberto".
- **Após confirmar, voto IRREVERSÍVEL** (audit trail imutável — ADR-0033).
- **Feedback visual imediato**.

## Layout — Votação

```
┌────────────────────────────────────────────────────┐
│  ASSEMBLEIA: Ordinária 2026/1                      │
│  ⏰ Votação aberta até 20/03/2026 às 23:59        │
│                                                    │
│  PAUTA 3: Escolha do Fornecedor para Fachada      │
│  📹 [Player: Vídeo explicativo do Síndico]        │
│                                                    │
│  Opções de voto:                                   │
│                                                    │
│  ┌──────────────────────────────────────────────┐ │
│  │ ○ Empresa Alpha Fachadas                     │ │
│  │   📹 [Ver vídeo] • R$45.000 • 60 dias       │ │
│  └──────────────────────────────────────────────┘ │
│  ┌──────────────────────────────────────────────┐ │
│  │ ○ Beta Recuperação Predial                   │ │
│  │   📹 [Ver vídeo] • R$52.000 • 45 dias       │ │
│  └──────────────────────────────────────────────┘ │
│  ┌──────────────────────────────────────────────┐ │
│  │ ● Gamma Engenharia ← selecionado            │ │
│  │   📹 [Ver vídeo] • R$48.500 • 50 dias       │ │
│  └──────────────────────────────────────────────┘ │
│                                                    │
│  [Confirmar Voto →] primary                       │
│                                                    │
│  ⚠️ Voto vinculado à unidade. Irreversível.       │
└────────────────────────────────────────────────────┘
```

## Após votar

```
┌──────────────────────────────┐
│  ✅ Voto registrado!         │
│                              │
│  Pauta: Escolha Fornecedor   │
│  Sua escolha: Gamma Eng.     │
│  Unidade: Apt 302 - Bloco A  │
│                              │
│  [← Voltar às pautas]       │
└──────────────────────────────┘
```

## CSS

```css
.vote-option {
  padding: 16px; border: 2px solid var(--border); border-radius: var(--radius-xl);
  cursor: pointer; transition: all var(--duration-moderate) var(--ease-out); margin-bottom: 12px;
}
.vote-option:hover {
  border-color: oklch(0.715 0.120 84.2 / 0.5);
  background: oklch(0.715 0.120 84.2 / 0.03);
}
.vote-option.selected {
  border-color: var(--primary); border-left-width: 4px;
  background: oklch(0.715 0.120 84.2 / 0.05);
}
.vote-option-radio {
  width: 20px; height: 20px; border-radius: 50%;
  border: 2px solid var(--border); margin-right: 12px;
  display: inline-flex; align-items: center; justify-content: center;
}
.vote-option.selected .vote-option-radio { border-color: var(--primary); }
.vote-option.selected .vote-option-radio::after {
  content: ''; width: 10px; height: 10px; border-radius: 50%; background: var(--primary);
}
.vote-option-name { font-family: 'Manrope'; font-size: 16px; font-weight: 600; }
.vote-option-details {
  font-size: 13px; color: var(--muted-foreground); margin-top: 4px;
  display: flex; gap: 12px;
}

.vote-confirm-modal { text-align: center; padding: 32px; }
.vote-confirm-icon { width: 64px; height: 64px; color: var(--success); margin: 0 auto 16px; }
.vote-confirm-title { font-family: 'Michroma'; font-size: 20px; margin-bottom: 8px; }

.vote-timer {
  display: flex; align-items: center; gap: 8px;
  padding: 8px 16px; background: oklch(0.769 0.165 70.1 / 0.08);
  border-radius: var(--radius-md); font-size: 13px; color: var(--warning);
}
.vote-warning {
  font-size: 12px; color: var(--muted-foreground); margin-top: 16px;
  display: flex; align-items: center; gap: 8px;
}
```

## Tipo Aprovar / Reprovar

```
┌────────────────────────────────┐
│ Pauta 2: Renovar contrato X   │
│                                │
│ [👍 APROVAR]  [👎 REPROVAR]   │
│  btn outline   btn outline     │
│  hover:success hover:destructive│
│                                │
│ [Confirmar →]                  │
└────────────────────────────────┘
```

```css
.vote-approve-btn {
  border: 2px solid var(--success); color: var(--success);
  border-radius: var(--radius-xl); padding: 16px 32px; font-weight: 600;
}
.vote-approve-btn:hover, .vote-approve-btn.selected {
  background: var(--success); color: white;
}
.vote-reject-btn {
  border: 2px solid var(--destructive); color: var(--destructive);
  border-radius: var(--radius-xl); padding: 16px 32px; font-weight: 600;
}
.vote-reject-btn:hover, .vote-reject-btn.selected {
  background: var(--destructive); color: white;
}
```

## Componentes

`VotingPage`, `VoteOption`, `VoteRadioGroup`, `VoteApproveReject`, `VoteConfirmation`, `VoteTimer`, `VoteWarning`.

## Atualizações Fase E (2026-04-25)

> Os 3 PDFs Master Síndico introduziram especializações importantes:

- **2 tipos canônicos de voto** (em adição aos 4 tipos de votação): `SIM/NÃO/ABSTENÇÃO` ou `ESCALA 1-5 + ABSTENÇÃO`. Ver detalhes em [[./live-session#fluxo-de-votação-ao-vivo-asm-014|live-session]].
- **Voto presencial assistido (R57 §6)**: voto lançado pelo síndico/administradora em nome de morador que entrou via terminal. Termo de voto + operator_id obrigatórios. Ver [[./live-session#voto-presencial-assistido-asm-016--r57-§6|live-session]].
- **Procuração 100% digital (ASM-010)**: NÃO há upload de PDF físico. Lei 14.063 (ICP-Brasil). Validade 24h pós-encerramento. Ver [[./pre-assembly#tela-6-cadastro-de-procuração-morador-—-100-digital-asm-010|pre-assembly]].
- **Reabertura de votação (R57 §13-§14)**: pré-homologação síndico pode zerar votação; nova `voting_session_id` distinta preserva histórico.
- **Auto-save Redis (R57 §19-§20)**: votação tem backup contínuo; recovery < 1min em crash. Ver [[./contingency-mode|contingency-mode]].

## Links

### Telas Fase E relacionadas

- [[./pre-assembly|pre-assembly]] — cadastro pauta + tipos votação por item
- [[./live-session|live-session]] — voto durante assembleia ao vivo + telão 2 áreas + voto presencial assistido
- [[./post-assembly|post-assembly]] — homologação + recibo individual de voto
- [[./contingency-mode|contingency-mode]] — voto em modo offline

### Outros

- [[./management|management]] — gestão de assembleias (09)
- [[../../../web/ui-catalog#e-app-assembly-as1-as8]] — AS6 (Votação Live), AS7 (Telão)
- [[../../../../00-product/business-model#41-matriz-consolidada]] — gate "Votar em assembleia"
- [[../../../../04-requirements/functional/assembly|functional/assembly]] — ASM-014/015/016
- [[../../../../01-domain/aggregates/Vote|Vote]] (D-130 entity de AgendaItem)
- [[../../patterns/ui-states|patterns/ui-states]]
- [[../../../../02-architecture/design-tokens|design-tokens]]
