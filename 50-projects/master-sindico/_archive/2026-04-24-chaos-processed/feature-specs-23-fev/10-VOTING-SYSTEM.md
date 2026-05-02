# 10 — SISTEMA DE VOTAÇÃO ASSÍNCRONA

> Sprint 2 · Rota: /assembleias/:id/votar/:pautaId
> Vinculado à pauta da assembleia. 1 voto por unidade.

---

## REGRAS DE NEGÓCIO

- Votação vinculada a pauta de assembleia
- Tipos: Aprovar/Reprovar OU Múltipla Escolha (Fornecedor A, B, C)
- 1 voto por unidade (validação backend por unidade habitacional)
- Janela de tempo: só vota se evento estiver "Aberto"
- Após confirmar, voto IRREVERSÍVEL
- Feedback visual imediato

## LAYOUT — VOTAÇÃO

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

## APÓS VOTAR

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
  padding: 16px; border: 2px solid var(--border); border-radius: 12px;
  cursor: pointer; transition: all 200ms; margin-bottom: 12px;
}
.vote-option:hover { border-color: var(--primary) / 0.5; background: oklch(0.715 0.120 84.2 / 0.03); }
.vote-option.selected { border-color: var(--primary); border-left-width: 4px; background: oklch(0.715 0.120 84.2 / 0.05); }
.vote-option-radio { width: 20px; height: 20px; border-radius: 50%; border: 2px solid var(--border); margin-right: 12px; display: inline-flex; align-items: center; justify-content: center; }
.vote-option.selected .vote-option-radio { border-color: var(--primary); }
.vote-option.selected .vote-option-radio::after { content: ''; width: 10px; height: 10px; border-radius: 50%; background: var(--primary); }
.vote-option-name { font-family: 'Manrope'; font-size: 16px; font-weight: 600; }
.vote-option-details { font-size: 13px; color: var(--muted-foreground); margin-top: 4px; display: flex; gap: 12px; }

.vote-confirm-modal { text-align: center; padding: 32px; }
.vote-confirm-icon { width: 64px; height: 64px; color: var(--success); margin: 0 auto 16px; }
.vote-confirm-title { font-family: 'Michroma'; font-size: 20px; margin-bottom: 8px; }

.vote-timer {
  display: flex; align-items: center; gap: 8px;
  padding: 8px 16px; background: var(--warning) / 0.08;
  border-radius: var(--radius); font-size: 13px; color: var(--warning);
}
.vote-warning { font-size: 12px; color: var(--muted-foreground); margin-top: 16px; display: flex; align-items: center; gap: 8px; }
```

## TIPO APROVAR/REPROVAR

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
.vote-approve-btn { border: 2px solid var(--success); color: var(--success); border-radius: 12px; padding: 16px 32px; font-weight: 600; }
.vote-approve-btn:hover, .vote-approve-btn.selected { background: var(--success); color: white; }
.vote-reject-btn { border: 2px solid var(--destructive); color: var(--destructive); border-radius: 12px; padding: 16px 32px; font-weight: 600; }
.vote-reject-btn:hover, .vote-reject-btn.selected { background: var(--destructive); color: white; }
```

## COMPONENTES

VotingPage, VoteOption, VoteRadioGroup, VoteApproveReject, VoteConfirmation, VoteTimer, VoteWarning
