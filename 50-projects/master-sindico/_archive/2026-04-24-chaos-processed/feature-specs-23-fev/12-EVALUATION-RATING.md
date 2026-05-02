# 12 — AVALIAÇÃO OBJETIVA DA GESTÃO

> Sprint 2 · Rota: /avaliar/:sindicoId
> Moradores avaliam Síndico com perguntas objetivas geradas pela plataforma

---

## REGRAS DE NEGÓCIO

- Perguntas FIXAS do sistema — síndico NÃO cria perguntas personalizadas
- Avaliação disponível ANTES de atividades/eventos definidos pelo MasterSíndico (janela de acesso)
- Média ponderada
- Privacidade: resultado consolidado visível para o Síndico. Moradores veem apenas "avaliação enviada", não veem avaliações de outros.
- 5 estrelas + tags descritivas opcionais

## LAYOUT — MORADOR AVALIANDO

```
┌──────────────────────────────────────────┐
│  AVALIAR GESTÃO DO SÍNDICO               │
│  João Silva • Cond. Villa Park           │
│                                          │
│  Pergunta 1 de 5:                        │
│  "Como você avalia a transparência das   │
│   decisões neste período?"               │
│                                          │
│  ☆ ☆ ☆ ☆ ☆                             │
│  1  2  3  4  5                           │
│                                          │
│  ────────────────────                    │
│                                          │
│  Pergunta 2 de 5:                        │
│  "A comunicação foi clara?"              │
│                                          │
│  ★ ★ ★ ☆ ☆                             │
│                                          │
│  [← Anterior]      [Próxima →]          │
│                                          │
│  ████████░░░░░░ 2/5                     │
└──────────────────────────────────────────┘
```

## CSS

```css
.star-rating { display: flex; gap: 8px; }
.star {
  width: 32px; height: 32px; cursor: pointer;
  color: var(--muted-foreground); transition: all 200ms;
}
.star:hover, .star.preview { color: var(--primary); transform: scale(1.1); }
.star.filled { color: var(--primary); }
.star.filled:active { animation: heartBounce 300ms ease; }

.eval-progress { margin-top: 24px; }
.eval-progress-bar { height: 4px; background: var(--muted); border-radius: 2px; }
.eval-progress-fill { height: 100%; background: var(--primary); border-radius: 2px; transition: width 300ms; }
.eval-progress-text { font-size: 12px; color: var(--muted-foreground); margin-top: 4px; }

.eval-question { font-family: 'Manrope'; font-size: 16px; font-weight: 500; color: var(--foreground); line-height: 1.5; margin-bottom: 16px; }
.eval-question-number { font-size: 13px; color: var(--muted-foreground); margin-bottom: 8px; }
```

## RESULTADO (Síndico vê)

```
┌────────────────────────────────────┐
│  Resultado da Avaliação            │
│                                    │
│  [4.2] ← Michroma 48px gold       │
│  /5.0                              │
│                                    │
│  Transparência:  ★★★★☆ 4.1       │
│  Comunicação:    ★★★★★ 4.8       │
│  Execução:       ★★★☆☆ 3.5       │
│  Atendimento:    ★★★★☆ 4.3       │
│  Satisfação:     ★★★★☆ 4.2       │
│                                    │
│  23 avaliações recebidas           │
└────────────────────────────────────┘
```

```css
.eval-result-score { font-family: 'Michroma'; font-size: 48px; color: var(--primary); }
.eval-result-total { font-size: 24px; color: var(--muted-foreground); }
.eval-result-breakdown { margin-top: 24px; }
.eval-result-item { display: flex; align-items: center; gap: 12px; padding: 8px 0; }
.eval-result-label { width: 140px; font-family: 'Manrope'; font-size: 14px; }
.eval-result-stars { display: flex; gap: 2px; }
.eval-result-value { font-weight: 600; font-size: 14px; min-width: 30px; }
```

## COMPONENTES

EvaluationPage, StarRating, EvalQuestion, EvalProgress, EvalResult, EvalBreakdown
