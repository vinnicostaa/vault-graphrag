# 13 — ENGINE DE CUPONS ("PROMOÇÃO DO DIA")

> Sprint 3 · Rotas: /vizinhanca/cupons, /vizinhanca/cupons/:id, /vizinhanca/minhas-promocoes (lojista)
> Sistema de geração de códigos promocionais para comércio físico local. NÃO É Helpdesk/Chamado.

---

## REGRAS DE NEGÓCIO

### Geração de Código
- Formato rígido: `ID_LOJA(5) + SEQUENCIAL(3) + PALAVRA(5)`
- Exemplo: Loja ID 00123 + Seq 055 + Palavra PADAO = `00123055PADAO`
- ID_LOJA: 5 dígitos (ID único do comércio na plataforma, zero-padded)
- SEQUENCIAL: 3 dígitos (número sequencial do cupom gerado naquele estabelecimento)
- PALAVRA: 5 letras maiúsculas (palavra-chave do dia definida pelo lojista)

### Trava de Tempo
- Morador só pode gerar 1 cupom a cada **4 horas** para o MESMO estabelecimento
- Pode gerar cupons de estabelecimentos DIFERENTES simultaneamente
- Lógica: Verificar `lastCouponAt` para o par (userId, lojaId) — se < 4h → bloqueia

### Palavra do Dia
- Definida pelo lojista (perfil Vizinhança)
- Máximo 5 letras, apenas letras MAIÚSCULAS, sem números/especiais
- Validação regex: `/^[A-Z]{1,5}$/`
- Lojista pode trocar a palavra a qualquer momento (afeta novos cupons, não retroativo)

### Promoções Exclusivas
- Síndico pode negociar com comerciante local descontos exclusivos para seus condomínios
- Comerciante ao cadastrar promoção escolhe:
  - **Exclusiva a condomínio** (via número de inscrição, negociado com síndico)
  - **Regional** (aberta para todos os usuários da região/CEP)
- Promoção tem: Título, Regra (texto), Validade (data início/fim)

### Permissões (Matriz Funcional)
| Ação | Base | Morador Pg | Síndicos | Emp. Plus/Pro | Vizinhança |
|------|------|-----------|----------|---------------|------------|
| Ver benefícios locais | ✅ | ✅ | ✅ | ✅ | ✅ |
| Ativar promoções do dia | ✅ | ✅ | ✅ | — | — |
| Cadastrar promoções | — | — | — | — | ✅ |
| Palavra-chave do dia | — | — | — | — | ✅ |
| Ativar promo exclusiva condo | — | — | ✅ N2/N3 | — | ✅ (aceitar) |

---

## LAYOUT — LISTAGEM DE PROMOÇÕES (Morador/Síndico)

```
┌──────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                    │
│          │  CLUBE DE BENEFÍCIOS              🔍 [Buscar...]  │
│          │  h2 Michroma 20px                                  │
│          │                                                    │
│          │  📍 Seu CEP: 01310-100 • São Paulo     [Alterar]  │
│          │                                                    │
│          │  [Todos] [Alimentação] [Serviços] [Lazer] [Saúde] │
│          │   ← pills horizontais scroll                       │
│          │                                                    │
│          │  ┌──────────────┐ ┌──────────────┐ ┌────────────┐│
│          │  │ 🏪           │ │ 🍕           │ │ 💈         ││
│          │  │ Padaria Sol  │ │ Pizza Nostra │ │ Barbearia  ││
│          │  │              │ │              │ │ Classic    ││
│          │  │ 🎟️ 10% OFF  │ │ 🎟️ 2por1    │ │ 🎟️ R$15OFF ││
│          │  │ toda loja    │ │ terça-feira  │ │ 1° corte   ││
│          │  │              │ │              │ │            ││
│          │  │ ⏰ Válido    │ │ ⏰ Válido    │ │ ⏰ Válido  ││
│          │  │ até 28/02    │ │ até 15/03    │ │ até 30/03  ││
│          │  │              │ │              │ │            ││
│          │  │ [Pegar Cupom]│ │ [Pegar Cupom]│ │[Pegar Cupom]│
│          │  └──────────────┘ └──────────────┘ └────────────┘│
│          │                                                    │
│          │  ┌──────────────┐ ┌──────────────┐               │
│          │  │ 🏋️           │ │ 🧹           │               │
│          │  │ Gym Power    │ │ LimpaJá      │               │
│          │  │              │ │              │               │
│          │  │ 🔒 EXCLUSIVO │ │ 🎟️ 20% OFF  │               │
│          │  │ Cond. Vila   │ │ 1ª limpeza   │               │
│          │  │ Park         │ │              │               │
│          │  │              │ │              │               │
│          │  │ [Pegar Cupom]│ │ [Pegar Cupom]│               │
│          │  └──────────────┘ └──────────────┘               │
└──────────────────────────────────────────────────────────────┘
```

### Grid
- Desktop: 3 colunas `grid-template-columns: repeat(3, 1fr)` gap 16px
- Tablet (768px–1023px): 2 colunas
- Mobile (<768px): 1 coluna, cards full-width

### CSS — Promo Card

```css
.promo-card {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 12px;
  overflow: hidden;
  transition: all 200ms ease;
}
.promo-card:hover {
  border-color: var(--primary);
  box-shadow: 0 4px 12px oklch(0.715 0.120 84.2 / 0.1);
}

/* Imagem ou ícone placeholder do estabelecimento */
.promo-card-banner {
  height: 120px;
  background: var(--muted);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 40px; /* emoji ou ícone */
}

.promo-card-body { padding: 16px; }

.promo-card-name {
  font-family: 'Manrope';
  font-size: 16px;
  font-weight: 700;
  color: var(--foreground);
  margin-bottom: 8px;
}

.promo-card-offer {
  display: flex;
  align-items: center;
  gap: 6px;
  font-family: 'Manrope';
  font-size: 14px;
  font-weight: 600;
  color: var(--primary);
  margin-bottom: 4px;
}

.promo-card-desc {
  font-family: 'Manrope';
  font-size: 13px;
  color: var(--muted-foreground);
  margin-bottom: 12px;
  line-height: 1.4;
}

.promo-card-validity {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 12px;
  color: var(--muted-foreground);
  margin-bottom: 16px;
}

/* Badge para promoção exclusiva de condomínio */
.promo-exclusive-badge {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 2px 8px;
  border-radius: 9999px;
  font-size: 11px;
  font-weight: 600;
  background: oklch(0.715 0.120 84.2 / 0.1);
  color: var(--primary);
  margin-bottom: 8px;
}

/* Botão Pegar Cupom */
.promo-btn-coupon {
  width: 100%;
  padding: 10px 16px;
  background: var(--primary);
  color: var(--primary-foreground);
  border: none;
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 200ms;
}
.promo-btn-coupon:hover {
  background: oklch(0.65 0.120 84.2);
}
.promo-btn-coupon:disabled {
  background: var(--muted);
  color: var(--muted-foreground);
  cursor: not-allowed;
}
```

---

## LAYOUT — MODAL DE CUPOM GERADO

Quando morador clica em "Pegar Cupom" e é aprovado (sem bloqueio 4h):

```
┌─────────────────────────────────────────┐
│           ✅ CUPOM GERADO!              │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │                                 │    │
│  │     00123 055 PADAO             │    │
│  │     ─────  ───  ─────           │    │
│  │     Loja   Seq  Palavra         │    │
│  │                                 │    │
│  └─────────────────────────────────┘    │
│                                         │
│  Padaria Sol                            │
│  10% OFF em toda loja                   │
│                                         │
│  ⏰ Válido até: 28/02/2026             │
│  📍 Rua Augusta, 1200 • SP             │
│                                         │
│  ⚠️ Apresente este código no           │
│     estabelecimento para validar        │
│     seu desconto.                       │
│                                         │
│  [📋 Copiar Código]  [✕ Fechar]        │
└─────────────────────────────────────────┘
```

### CSS — Modal Cupom

```css
.coupon-modal-overlay {
  position: fixed;
  inset: 0;
  background: oklch(0 0 0 / 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 50;
  animation: fadeIn 200ms ease;
}

.coupon-modal {
  background: var(--card);
  border-radius: 16px;
  padding: 32px;
  max-width: 420px;
  width: 90%;
  text-align: center;
  animation: fadeInUp 300ms ease;
}

.coupon-modal-icon {
  font-size: 48px;
  margin-bottom: 8px;
}

.coupon-modal-title {
  font-family: 'Michroma';
  font-size: 18px;
  color: var(--success);
  letter-spacing: 0.05em;
  margin-bottom: 24px;
}

/* Container do código */
.coupon-code-box {
  background: var(--surface);
  border: 2px dashed var(--primary);
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 20px;
}

.coupon-code {
  font-family: 'Michroma';
  font-size: 28px;
  letter-spacing: 0.15em;
  color: var(--primary);
  word-spacing: 12px;
}

.coupon-code-labels {
  display: flex;
  justify-content: center;
  gap: 24px;
  margin-top: 8px;
}

.coupon-code-label {
  font-family: 'Manrope';
  font-size: 10px;
  text-transform: uppercase;
  color: var(--surface-foreground);
  opacity: 0.6;
  letter-spacing: 0.08em;
}

.coupon-store-name {
  font-family: 'Manrope';
  font-size: 16px;
  font-weight: 700;
  color: var(--foreground);
  margin-bottom: 4px;
}

.coupon-offer-text {
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--primary);
  font-weight: 600;
  margin-bottom: 16px;
}

.coupon-meta {
  font-size: 13px;
  color: var(--muted-foreground);
  line-height: 1.8;
  margin-bottom: 16px;
}

.coupon-warning {
  background: oklch(0.769 0.165 70.1 / 0.1);
  border-radius: 8px;
  padding: 12px;
  font-size: 13px;
  color: var(--warning);
  line-height: 1.5;
  margin-bottom: 20px;
}

.coupon-actions {
  display: flex;
  gap: 12px;
}

.coupon-btn-copy {
  flex: 1;
  padding: 10px 16px;
  background: var(--primary);
  color: var(--primary-foreground);
  border: none;
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  transition: all 200ms;
}
.coupon-btn-copy:hover { background: oklch(0.65 0.120 84.2); }

/* Estado pós-cópia */
.coupon-btn-copy.copied {
  background: var(--success);
  color: var(--success-foreground);
}

.coupon-btn-close {
  padding: 10px 16px;
  background: var(--muted);
  color: var(--muted-foreground);
  border: none;
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 200ms;
}
.coupon-btn-close:hover { background: var(--border); }
```

---

## LAYOUT — ESTADO BLOQUEADO (Trava 4h)

Quando morador tenta gerar cupom antes de 4h:

```
┌─────────────────────────────────────────┐
│           ⏳ AGUARDE                     │
│                                         │
│  Você já possui um cupom ativo para     │
│  este estabelecimento.                  │
│                                         │
│  Próximo cupom disponível em:           │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │     02h : 34min : 15s           │    │
│  └─────────────────────────────────┘    │
│                                         │
│  Cupom ativo: 00123055PADAO             │
│  📋 [Copiar código ativo]              │
│                                         │
│  [Voltar às promoções]                  │
└─────────────────────────────────────────┘
```

### CSS — Countdown Timer

```css
.coupon-blocked-modal {
  background: var(--card);
  border-radius: 16px;
  padding: 32px;
  max-width: 420px;
  width: 90%;
  text-align: center;
}

.coupon-blocked-icon {
  font-size: 48px;
  margin-bottom: 8px;
}

.coupon-blocked-title {
  font-family: 'Michroma';
  font-size: 18px;
  color: var(--warning);
  letter-spacing: 0.05em;
  margin-bottom: 12px;
}

.coupon-blocked-text {
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--muted-foreground);
  line-height: 1.5;
  margin-bottom: 20px;
}

.coupon-countdown {
  background: var(--muted);
  border-radius: 12px;
  padding: 16px 24px;
  margin-bottom: 20px;
}

.coupon-countdown-time {
  font-family: 'Michroma';
  font-size: 32px;
  color: var(--warning);
  letter-spacing: 0.1em;
}

/* Separadores ":" piscando */
.coupon-countdown-sep {
  animation: blink 1s step-start infinite;
}
@keyframes blink {
  50% { opacity: 0; }
}

.coupon-active-code {
  font-family: 'Michroma';
  font-size: 16px;
  color: var(--primary);
  letter-spacing: 0.1em;
  margin-bottom: 16px;
}
```

---

## LAYOUT — PAINEL DO LOJISTA (Vizinhança)

Rota: `/vizinhanca/minhas-promocoes`

```
┌──────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                    │
│          │  MINHAS PROMOÇÕES             [+ Nova Promoção]   │
│          │  h2 Michroma 20px                                  │
│          │                                                    │
│          │  ┌─── PALAVRA DO DIA ─────────────────────────┐   │
│          │  │                                             │   │
│          │  │  Palavra atual:  [P] [A] [D] [A] [O]       │   │
│          │  │                                             │   │
│          │  │  [Alterar Palavra]                          │   │
│          │  │                                             │   │
│          │  └─────────────────────────────────────────────┘   │
│          │                                                    │
│          │  ┌─── MÉTRICAS ───────────────────────────────┐   │
│          │  │  Cupons Gerados                             │   │
│          │  │                                             │   │
│          │  │  [12]  Hoje     [78]  Semana   [342]  Mês  │   │
│          │  │                                             │   │
│          │  └─────────────────────────────────────────────┘   │
│          │                                                    │
│          │  PROMOÇÕES ATIVAS                                  │
│          │  ┌─────────────────────────────────────────────┐   │
│          │  │  🎟️ 10% OFF toda loja                       │   │
│          │  │  Regional • Até 28/02 • 45 cupons           │   │
│          │  │  [Editar] [Encerrar]                        │   │
│          │  └─────────────────────────────────────────────┘   │
│          │  ┌─────────────────────────────────────────────┐   │
│          │  │  🎟️ 15% Exclusivo Cond. Villa Park          │   │
│          │  │  Exclusiva • Até 15/03 • 12 cupons          │   │
│          │  │  [Editar] [Encerrar]                        │   │
│          │  └─────────────────────────────────────────────┘   │
│          │                                                    │
│          │  PROMOÇÕES ENCERRADAS                              │
│          │  ┌─────────────────────────────────────────────┐   │
│          │  │  🎟️ 2por1 às terças                         │   │
│          │  │  Regional • Encerrada • 203 cupons          │   │
│          │  └─────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────┘
```

### CSS — Palavra do Dia

```css
.word-of-day-card {
  background: var(--card);
  border: 2px solid var(--primary);
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 24px;
}

.word-of-day-title {
  font-family: 'Michroma';
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 0.12em;
  color: var(--primary);
  margin-bottom: 16px;
}

.word-of-day-letters {
  display: flex;
  gap: 8px;
  margin-bottom: 16px;
}

/* Cada letra individual — similar a OTP field */
.word-letter {
  width: 48px;
  height: 56px;
  background: var(--surface);
  border: 2px solid var(--primary);
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: 'Michroma';
  font-size: 24px;
  color: var(--primary);
  letter-spacing: 0;
}

/* Quando editando — estilo input */
.word-letter-input {
  width: 48px;
  height: 56px;
  background: var(--background);
  border: 2px solid var(--ring);
  border-radius: 8px;
  font-family: 'Michroma';
  font-size: 24px;
  color: var(--foreground);
  text-align: center;
  text-transform: uppercase;
  outline: none;
}
.word-letter-input:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 3px oklch(0.715 0.120 84.2 / 0.15);
}
```

### CSS — Métricas

```css
.metrics-card {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 24px;
}

.metrics-title {
  font-family: 'Michroma';
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: var(--muted-foreground);
  margin-bottom: 16px;
}

.metrics-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
}

.metric-item { text-align: center; }

.metric-value {
  font-family: 'Michroma';
  font-size: 28px;
  color: var(--primary);
  letter-spacing: 0.05em;
}

.metric-label {
  font-family: 'Manrope';
  font-size: 12px;
  color: var(--muted-foreground);
  margin-top: 4px;
}
```

### CSS — Promo List Item (Lojista)

```css
.promo-list-item {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 16px;
  margin-bottom: 12px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.promo-list-info { flex: 1; }

.promo-list-title {
  font-family: 'Manrope';
  font-size: 15px;
  font-weight: 600;
  color: var(--foreground);
  margin-bottom: 4px;
}

.promo-list-meta {
  font-family: 'Manrope';
  font-size: 13px;
  color: var(--muted-foreground);
}

.promo-list-meta .tag-exclusive {
  color: var(--primary);
  font-weight: 600;
}

.promo-list-meta .tag-regional {
  color: var(--muted-foreground);
}

.promo-list-actions {
  display: flex;
  gap: 8px;
}

.promo-list-btn-edit,
.promo-list-btn-end {
  padding: 6px 12px;
  border-radius: 6px;
  font-family: 'Manrope';
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  border: 1px solid var(--border);
  background: transparent;
  color: var(--foreground);
  transition: all 200ms;
}
.promo-list-btn-edit:hover { border-color: var(--primary); color: var(--primary); }
.promo-list-btn-end:hover { border-color: var(--destructive); color: var(--destructive); }
```

---

## FORM — NOVA PROMOÇÃO (Lojista)

```
┌─────────────────────────────────────────┐
│  NOVA PROMOÇÃO                          │
│                                         │
│  Título *                               │
│  ┌─────────────────────────────────┐    │
│  │ 10% OFF em toda loja            │    │
│  └─────────────────────────────────┘    │
│                                         │
│  Descrição da regra *                   │
│  ┌─────────────────────────────────┐    │
│  │ Válido para compras acima de    │    │
│  │ R$30. Apresente o cupom no cai- │    │
│  │ xa antes do pagamento.          │    │
│  └─────────────────────────────────┘    │
│  0/300 caracteres                       │
│                                         │
│  Tipo de promoção *                     │
│  ○ Regional (todos da região/CEP)       │
│  ○ Exclusiva a condomínio               │
│                                         │
│  [Se exclusiva:]                        │
│  Nº de Inscrição do Condomínio *        │
│  ┌─────────────────────────────────┐    │
│  │ 12345-ABC                       │    │
│  └─────────────────────────────────┘    │
│                                         │
│  Validade *                             │
│  ┌───────────┐  ┌───────────────┐       │
│  │ 01/02/2026│  │ 28/02/2026    │       │
│  │ Início    │  │ Fim           │       │
│  └───────────┘  └───────────────┘       │
│                                         │
│  [Cancelar]           [Publicar →]      │
└─────────────────────────────────────────┘
```

### Validação (Zod)

```ts
import { z } from "zod";

export const PromoSchema = z.object({
  titulo: z.string().min(3, "Mín. 3 caracteres").max(80),
  descricao: z.string().min(10, "Descreva a regra").max(300),
  tipo: z.enum(["regional", "exclusiva"]),
  condominioInscricao: z.string().optional()
    .refine((val, ctx) => {
      if (ctx?.parent?.tipo === "exclusiva" && !val) return false;
      return true;
    }, "Obrigatório para promoção exclusiva"),
  validadeInicio: z.date(),
  validadeFim: z.date(),
  palavraDoDia: z.string()
    .regex(/^[A-Z]{1,5}$/, "Apenas letras maiúsculas, máx. 5")
    .optional(),
}).refine(
  (data) => data.validadeFim > data.validadeInicio,
  { message: "Fim deve ser posterior ao início", path: ["validadeFim"] }
);
```

### CSS — Form

```css
.promo-form {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 32px;
  max-width: 560px;
}

.promo-form-title {
  font-family: 'Michroma';
  font-size: 20px;
  color: var(--foreground);
  letter-spacing: 0.05em;
  margin-bottom: 24px;
}

.promo-form-field { margin-bottom: 20px; }

.promo-form-label {
  display: block;
  font-family: 'Manrope';
  font-size: 13px;
  font-weight: 600;
  color: var(--foreground);
  margin-bottom: 6px;
}

.promo-form-input {
  width: 100%;
  padding: 10px 14px;
  background: var(--background);
  border: 1px solid var(--input);
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--foreground);
  outline: none;
  transition: border-color 200ms;
}
.promo-form-input:focus {
  border-color: var(--ring);
  box-shadow: 0 0 0 3px oklch(0.715 0.120 84.2 / 0.15);
}

.promo-form-textarea {
  width: 100%;
  min-height: 80px;
  padding: 10px 14px;
  background: var(--background);
  border: 1px solid var(--input);
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--foreground);
  resize: vertical;
  outline: none;
}
.promo-form-textarea:focus {
  border-color: var(--ring);
  box-shadow: 0 0 0 3px oklch(0.715 0.120 84.2 / 0.15);
}

.promo-form-charcount {
  font-size: 12px;
  color: var(--muted-foreground);
  text-align: right;
  margin-top: 4px;
}

/* Radio tipo */
.promo-form-radio-group {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.promo-form-radio-item {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
}

.promo-form-radio {
  width: 18px;
  height: 18px;
  border-radius: 50%;
  border: 2px solid var(--border);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 200ms;
}
.promo-form-radio.selected {
  border-color: var(--primary);
}
.promo-form-radio.selected::after {
  content: '';
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: var(--primary);
}

.promo-form-radio-label {
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--foreground);
}

/* Datas lado a lado */
.promo-form-date-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

/* Botões do form */
.promo-form-actions {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  margin-top: 24px;
}

.promo-form-btn-cancel {
  padding: 10px 20px;
  background: transparent;
  border: 1px solid var(--border);
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 14px;
  font-weight: 500;
  color: var(--muted-foreground);
  cursor: pointer;
  transition: all 200ms;
}
.promo-form-btn-cancel:hover { border-color: var(--foreground); color: var(--foreground); }

.promo-form-btn-submit {
  padding: 10px 24px;
  background: var(--primary);
  color: var(--primary-foreground);
  border: none;
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 200ms;
}
.promo-form-btn-submit:hover { background: oklch(0.65 0.120 84.2); }
```

---

## MOBILE — AJUSTES

### Cards em coluna única
- Cards ocupam 100% width
- Banner do card reduz para 100px
- Padding interno: 12px

### Botão flutuante "Pegar Cupom" (mobile)
- Quando no detalhe de uma promoção, sticky bottom bar com o botão

```css
@media (max-width: 767px) {
  .promo-grid { grid-template-columns: 1fr; }

  .promo-card-banner { height: 100px; }
  .promo-card-body { padding: 12px; }

  .promo-sticky-cta {
    position: fixed;
    bottom: 72px; /* acima do bottom-nav */
    left: 0; right: 0;
    padding: 12px 16px;
    background: var(--card);
    border-top: 1px solid var(--border);
    z-index: 30;
  }
  .promo-sticky-cta .promo-btn-coupon {
    width: 100%;
    padding: 14px;
    font-size: 16px;
  }
}
```

---

## COMPONENTES

| Componente | Tipo | Dependência |
|---|---|---|
| `PromoCard` | Card de promoção na grid | — |
| `CouponModal` | Modal com código gerado | Kobalte Dialog |
| `BlockedModal` | Modal com countdown | Kobalte Dialog |
| `WordOfDayEditor` | Input estilo OTP para palavra | @corvu/otp-field adaptado |
| `MetricsCard` | 3 métricas (hoje/semana/mês) | — |
| `PromoForm` | Form de nova promoção | TanStack Form + Zod |
| `PromoListItem` | Item na lista do lojista | — |
| `CEPFilter` | Input CEP com máscara | — |
| `CategoryPills` | Horizontal scroll de categorias | — |
| `CountdownTimer` | Timer regressivo 4h | setInterval SolidJS |

---

## ESTADOS

| Estado | Comportamento |
|---|---|
| Loading | Skeleton: 3 cards com shimmer |
| Empty (nenhuma promo no CEP) | Ilustração + "Nenhum benefício na sua região" |
| Cupom gerado com sucesso | Modal verde com código + botão copiar |
| Bloqueado 4h | Modal warning com countdown regressivo |
| Promo exclusiva (morador não é do condo) | Card visível mas botão disabled + tooltip "Exclusivo para moradores do Cond. X" |
| Sem CEP cadastrado | Banner topo: "Cadastre seu CEP para ver benefícios da sua região" + [Configurar] |

---

## CHECKLIST

- [ ] Grid responsiva 3→2→1 colunas
- [ ] Cards com hover gold border
- [ ] Botão "Pegar Cupom" com validação 4h no backend
- [ ] Modal de sucesso com código formatado (Michroma 28px gold)
- [ ] Modal bloqueado com countdown animado
- [ ] Botão "Copiar Código" com feedback visual (ícone muda, cor verde)
- [ ] Filtro por CEP (input com máscara 5+3)
- [ ] Pills de categorias com horizontal scroll
- [ ] Painel lojista: Palavra do Dia (input estilo OTP, 5 chars)
- [ ] Métricas: hoje/semana/mês com números Michroma
- [ ] Form nova promoção: validação Zod, tipo regional/exclusiva
- [ ] Badge "Exclusivo" em cards de promo vinculada a condomínio
- [ ] Promo-sticky CTA em mobile
- [ ] Dark mode: todos os tokens oklch resolvem corretamente
