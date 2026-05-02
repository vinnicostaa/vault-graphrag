---
title: Coupon Engine — UI Spec
type: ui-spec
tags: [ui-catalog, master-sindico, commercial, coupon, vizinhanca]
bc: commercial
source: _chaos/13-COUPON-ENGINE.md (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# 13 — Engine de Cupons ("Promoção do Dia")

> **Origem**: `_chaos/13-COUPON-ENGINE.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Tradução aplicada**: `N1/N2/N3` → `trial/base/plus/pro` (ADR-0032 / D-081 / D-096); "Morador Pagante" removido (D-126).
> **Nota**: spec prescritiva (formato `ID_LOJA(5)+SEQ(3)+PALAVRA(5)`, lock 4 h) **a virar ADR ou subitem de ADR coupon (D-059 / D-078)**.

> Sprint 3 · App `forum` / `cms` · Rotas: `/vizinhanca/cupons`, `/vizinhanca/cupons/:id`, `/vizinhanca/minhas-promocoes` (lojista)
> Sistema de geração de códigos promocionais para comércio físico local. **NÃO É** Helpdesk / Chamado.

---

## Regras de negócio

### Geração de código
- **Formato rígido**: `ID_LOJA(5) + SEQUENCIAL(3) + PALAVRA(5)` (D-059 / D-078).
- **Exemplo**: Loja `00123` + Seq `055` + Palavra `PADAO` = `00123055PADAO`.
- **ID_LOJA**: 5 dígitos (ID único do comércio na plataforma, zero-padded).
- **SEQUENCIAL**: 3 dígitos (número sequencial do cupom gerado naquele estabelecimento).
- **PALAVRA**: 5 letras maiúsculas (palavra-chave do dia definida pelo lojista).

### Trava de tempo
- Morador só pode gerar 1 cupom a cada **4 horas** para o MESMO estabelecimento.
- Pode gerar cupons de estabelecimentos DIFERENTES simultaneamente.
- Lógica: verificar `lastCouponAt` para o par `(userId, lojaId)` — se < 4 h → bloqueia (Redis lock + DB `coupon_issuance.expires_at`).

### Palavra do Dia
- Definida pelo lojista (perfil Vizinhança / `role=local_company`).
- Máximo 5 letras, apenas letras MAIÚSCULAS, sem números / especiais.
- Validação regex: `/^[A-Z]{1,5}$/`.
- Lojista pode trocar a palavra a qualquer momento (afeta novos cupons, não retroativo).

### Promoções exclusivas
- **Síndico** pode negociar com comerciante local descontos exclusivos para seus condomínios.
- Comerciante ao cadastrar promoção escolhe:
  - **Exclusiva a condomínio** (via número de inscrição, negociado com síndico)
  - **Regional** (aberta para todos os usuários da região / CEP)
- Promoção tem: Título, Regra (texto), Validade (data início / fim).

### Permissões (matriz canônica)

| Ação | Morador `base` | Síndico | Empresa `plus`/`pro` | Parceiro `local_company` |
|---|---|---|---|---|
| Ver benefícios locais | ✅ | ✅ | ✅ | ✅ |
| Ativar promoções do dia | ✅ | ✅ | — | — |
| Cadastrar promoções | — | — | — | ✅ (`plus`+) |
| Palavra-chave do dia | — | — | — | ✅ (`pro` ideal — D-099) |
| Ativar promo exclusiva condo | — | ✅ (`plus`+, emissor seal) | — | ✅ (aceitar) |

> Ver matriz completa em [[../../../../00-product/business-model#41-matriz-consolidada]].

---

## Layout — Listagem de promoções (Morador / Síndico)

```
┌──────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                    │
│          │  CLUBE DE BENEFÍCIOS              🔍 [Buscar...]  │
│          │                                                    │
│          │  📍 Seu CEP: 01310-100 • São Paulo     [Alterar]  │
│          │                                                    │
│          │  [Todos] [Alimentação] [Serviços] [Lazer] [Saúde] │
│          │                                                    │
│          │  ┌──────────────┐ ┌──────────────┐ ┌────────────┐│
│          │  │ 🏪 Padaria   │ │ 🍕 Pizza     │ │ 💈 Barbear.││
│          │  │ Sol          │ │ Nostra       │ │ Classic    ││
│          │  │ 🎟️ 10% OFF  │ │ 🎟️ 2por1    │ │ 🎟️ R$15OFF ││
│          │  │ ⏰ até 28/02 │ │ ⏰ até 15/03 │ │ ⏰ até 30/03│
│          │  │ [Pegar Cupom]│ │ [Pegar Cupom]│ │[Pegar Cupom]│
│          │  └──────────────┘ └──────────────┘ └────────────┘│
└──────────────────────────────────────────────────────────────┘
```

Grid responsivo: Desktop 3 col, tablet 2 col, mobile 1 col.

### CSS — Promo Card

```css
.promo-card {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: var(--radius-xl);
  overflow: hidden;
  transition: all var(--duration-moderate) var(--ease-out);
}
.promo-card:hover {
  border-color: var(--primary);
  box-shadow: 0 4px 12px oklch(0.715 0.120 84.2 / 0.1);
}
.promo-card-banner {
  height: 120px; background: var(--muted);
  display: flex; align-items: center; justify-content: center;
  font-size: 40px;
}
.promo-card-body { padding: 16px; }
.promo-card-name {
  font-family: 'Manrope'; font-size: 16px; font-weight: 700;
  color: var(--foreground); margin-bottom: 8px;
}
.promo-card-offer {
  display: flex; align-items: center; gap: 6px;
  font-family: 'Manrope'; font-size: 14px; font-weight: 600;
  color: var(--primary); margin-bottom: 4px;
}
.promo-card-validity {
  display: flex; align-items: center; gap: 4px;
  font-size: 12px; color: var(--muted-foreground); margin-bottom: 16px;
}
.promo-exclusive-badge {
  display: inline-flex; align-items: center; gap: 4px;
  padding: 2px 8px; border-radius: var(--radius-full);
  font-size: 11px; font-weight: 600;
  background: oklch(0.715 0.120 84.2 / 0.1); color: var(--primary);
  margin-bottom: 8px;
}
.promo-btn-coupon {
  width: 100%; padding: 10px 16px;
  background: var(--primary); color: var(--primary-foreground);
  border: none; border-radius: var(--radius-md);
  font-family: 'Manrope'; font-size: 14px; font-weight: 600;
  cursor: pointer; transition: all var(--duration-moderate) var(--ease-out);
}
.promo-btn-coupon:hover { background: var(--primary-active); }
.promo-btn-coupon:disabled {
  background: var(--muted); color: var(--muted-foreground); cursor: not-allowed;
}
```

---

## Modal — Cupom gerado

```
┌─────────────────────────────────────────┐
│           ✅ CUPOM GERADO!              │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │     00123 055 PADAO             │    │
│  │     ─────  ───  ─────           │    │
│  │     Loja   Seq  Palavra         │    │
│  └─────────────────────────────────┘    │
│                                         │
│  Padaria Sol — 10% OFF em toda loja     │
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

```css
.coupon-code-box {
  background: var(--surface);
  border: 2px dashed var(--primary);
  border-radius: var(--radius-xl);
  padding: 20px; margin-bottom: 20px;
}
.coupon-code {
  font-family: 'Michroma'; font-size: 28px;
  letter-spacing: 0.15em; color: var(--primary); word-spacing: 12px;
}
.coupon-code-labels { display: flex; justify-content: center; gap: 24px; margin-top: 8px; }
.coupon-code-label {
  font-family: 'Manrope'; font-size: 10px;
  text-transform: uppercase; color: var(--surface-foreground); opacity: 0.6;
  letter-spacing: 0.08em;
}
```

---

## Estado bloqueado (trava 4 h)

```
┌─────────────────────────────────────────┐
│           ⏳ AGUARDE                     │
│                                         │
│  Você já possui um cupom ativo para     │
│  este estabelecimento.                  │
│                                         │
│  Próximo cupom disponível em:           │
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

```css
.coupon-countdown-time {
  font-family: 'Michroma'; font-size: 32px;
  color: var(--warning); letter-spacing: 0.1em;
}
.coupon-countdown-sep { animation: blink 1s step-start infinite; }
@keyframes blink { 50% { opacity: 0; } }
```

---

## Painel do Lojista (Vizinhança)

Rota: `/vizinhanca/minhas-promocoes`.

Componentes principais:

- **Palavra do Dia**: input estilo OTP de 5 letras maiúsculas.
- **Métricas**: Cupons gerados (Hoje / Semana / Mês — Michroma 28 px gold).
- **Promoções ativas / encerradas**: lista com [Editar] / [Encerrar].

### CSS — Word of Day editor

```css
.word-letter {
  width: 48px; height: 56px;
  background: var(--surface);
  border: 2px solid var(--primary); border-radius: var(--radius-md);
  display: flex; align-items: center; justify-content: center;
  font-family: 'Michroma'; font-size: 24px; color: var(--primary);
}
.word-letter-input {
  width: 48px; height: 56px;
  background: var(--background); border: 2px solid var(--ring);
  border-radius: var(--radius-md);
  font-family: 'Michroma'; font-size: 24px;
  color: var(--foreground); text-align: center; text-transform: uppercase;
  outline: none;
}
.word-letter-input:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 3px oklch(0.715 0.120 84.2 / 0.15);
}
```

---

## Form — Nova promoção (Lojista)

Validação Zod:

```ts
import { z } from "zod";

export const PromoSchema = z
  .object({
    titulo: z.string().min(3, "Mín. 3 caracteres").max(80),
    descricao: z.string().min(10, "Descreva a regra").max(300),
    tipo: z.enum(["regional", "exclusiva"]),
    condominioInscricao: z.string().optional(),
    validadeInicio: z.date(),
    validadeFim: z.date(),
    palavraDoDia: z
      .string()
      .regex(/^[A-Z]{1,5}$/, "Apenas letras maiúsculas, máx. 5")
      .optional(),
  })
  .refine((data) => data.validadeFim > data.validadeInicio, {
    message: "Fim deve ser posterior ao início",
    path: ["validadeFim"],
  })
  .refine((data) => data.tipo === "regional" || !!data.condominioInscricao, {
    message: "Obrigatório para promoção exclusiva",
    path: ["condominioInscricao"],
  });
```

---

## Componentes

| Componente | Dependência |
|---|---|
| `PromoCard` | — |
| `CouponModal` | Kobalte Dialog |
| `BlockedModal` (countdown 4 h) | Kobalte Dialog |
| `WordOfDayEditor` | `@corvu/otp-field` adaptado |
| `MetricsCard` | — |
| `PromoForm` | TanStack Form + Zod |
| `PromoListItem` | — |
| `CEPFilter` | — |
| `CategoryPills` | — |
| `CountdownTimer` | `setInterval` SolidJS |

## Estados

Padrão (loading / empty / error / success / permission-denied) seguem [[../../patterns/ui-states|patterns/ui-states]].

| Estado específico | Comportamento |
|---|---|
| Cupom gerado | Modal verde com código formatado + copiar |
| Bloqueado 4 h | Modal warning + countdown |
| Promo exclusiva (morador não-membro) | Card visível, botão disabled, tooltip "Exclusivo Cond. X" |
| Sem CEP | Banner topo "Cadastre seu CEP" + [Configurar] |

## Links

- [[../../../../STATE]] — D-059 / D-078 spec ADR cupom
- [[../../../../00-product/sub-produtos/08-vizinhanca]]
- [[../../../../00-product/business-model#41-matriz-consolidada]]
- [[./benefits-club|benefits-club]] — hub Vizinhança (14)
- [[../../patterns/ui-states|patterns/ui-states]]
- [[../../../../02-architecture/design-tokens|design-tokens]]
