---
title: Auth & Onboarding — UI Spec
type: ui-spec
tags: [ui-catalog, master-sindico, auth, identity, onboarding]
bc: identity
source: _chaos/01-AUTH-ONBOARDING.md (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# 01 — Autenticação & Onboarding

> **Origem**: `_chaos/01-AUTH-ONBOARDING.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Tradução aplicada**: `N1/N2/N3` → `trial/base/plus/pro` (ADR-0032 / D-081 / D-096); "Morador Pagante" removido (D-126); "My Síndico" → "Master Síndico".
> **Mapeamento de telas canônicas**: AUTH1-AUTH8 (ver [[../../../web/ui-catalog#a-app-auth]]).

> Sprint 1 · App `auth` (porta 3000) · Rotas: `/login`, `/cadastro`, `/verificar-email`, `/definir-senha`, `/bem-vindo`
> Deps: `@corvu/otp-field`, `@tanstack/solid-form`, `zod`

---

## Regras de negócio

- **Cadastro parametrizado via URL**: `mastersindico.com.br/cadastro?role=sindico&plan=pro` (ver [[../../../../00-product/business-model|business-model]] para mapeamento de `plan_tier`)
- **Fluxo**: Formulário → Email OTP → Definir Senha → Welcome Screen
- **NÃO há OCR, biometria ou validação de documento** — apenas email/token
- **JWT cookie httpOnly** emitido pelo backend após ativação (Zitadel OIDC + PKCE — ver [[../../../web/README#43-auth-flow-zitadel-oidc--pkce]])
- **Role e plan_tier** travados no backend; query string apenas pré-popula UI (ABAC determina permissões finais)
- **Telas de auth são públicas** (sem middleware de sessão)

> Vocabulário canônico: usar `plan_tier ∈ {trial, base, plus, pro}` — labels UI podem ser "Síndico Pro", "Empresa Plus", etc., mas o dado persistido segue ADR-0032.

## Rotas

| Rota | Componente | Descrição |
|---|---|---|
| `/login` | `LoginPage` | Formulário email + senha |
| `/cadastro` | `SignupPage` | Formulário com `role` e `plan` de URL params |
| `/verificar-email` | `VerifyEmailPage` | OTP 6 dígitos |
| `/definir-senha` | `SetPasswordPage` | Criar senha com requisitos |
| `/bem-vindo` | `WelcomePage` | Sucesso + features do plano |

## Layout — Login (Desktop split 50/50)

```
┌───────────────────────────────────────────────────────────────┐
│  ┌──────────────────────┐  ┌─────────────────────────────────┐│
│  │  PAINEL VISUAL       │  │  [🛡 Logo Master Síndico]       ││
│  │  bg: var(--secondary)│  │                                 ││
│  │  skyline pattern 5%  │  │  Acessar sua conta              ││
│  │                      │  │  h2 Michroma 24px               ││
│  │  Michroma 20px gold: │  │                                 ││
│  │  "Governança começa  │  │  [Email ____________________]  ││
│  │   com método."       │  │  [Senha ____________________]  ││
│  │                      │  │  [Esqueceu a senha?] link       ││
│  │                      │  │                                 ││
│  │                      │  │  [Entrar →] btn primary xl      ││
│  │                      │  │  full-width                     ││
│  │                      │  │                                 ││
│  │                      │  │  Não tem conta? [Criar conta →] ││
│  └──────────────────────┘  └─────────────────────────────────┘│
└───────────────────────────────────────────────────────────────┘
```

Mobile: painel vira header de 120px (logo + frase), formulário ocupa o restante full-width.

## Layout — Cadastro

Mesmo split 50/50. Texto do painel adapta ao role detectado:

- **Síndico**: "Sua gestão merece registro."
- **Empresa**: "Autoridade se prova com conhecimento."
- **Morador**: "Participação responsável."

Badge de plano visível: ex. **"Síndico Pro"** em badge `variant=default` abaixo do título.

### Campos por role

| Role | Campos |
|---|---|
| `resident` (Morador) | Nome, Email, CPF |
| `syndic` (Síndico) | Nome, Email, CPF, Telefone |
| `enterprise` (Empresa) | Razão Social, CNPJ, Email, Responsável, Telefone |
| `local_company` (Vizinhança / Parceiro Local) | Nome estabelecimento, CNPJ, Email, CEP, Endereço |

> Morador é **gratuito por padrão** (`plan_tier=base`). O addon "Banco de Talentos" (D-099 — vídeo-currículo 90s, 5 vínculos profissionais visíveis, M3) é separado e não exige plano pago. Fluxo de cadastro morador NUNCA cobra cartão. Ver [[../../../../00-product/business-model#32-morador-roleresident]].

## OTP (6 dígitos)

```
┌────────────────────────────────────┐
│  Confirme seu email                │
│  Código enviado para               │
│  joao@email.com (bold, primary)    │
│                                    │
│  ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐  │
│  │ 4│ │ 2│ │ 8│ │  │ │  │ │  │  │
│  └──┘ └──┘ └──┘ └──┘ └──┘ └──┘  │
│                                    │
│  [Reenviar código] timer 60s       │
│  [Confirmar →]                     │
└────────────────────────────────────┘
```

```css
.otp-field {
  width: 48px; height: 56px;
  font-family: 'Manrope'; font-size: 24px; font-weight: 700;
  text-align: center;
  background: var(--card); border: 2px solid var(--border); border-radius: var(--radius-md);
  transition: all var(--duration-moderate) var(--ease-out);
}
.otp-field:focus { border-color: var(--primary); box-shadow: 0 0 0 3px oklch(0.715 0.120 84.2 / 0.2); }
.otp-field[data-filled] { border-color: var(--primary); }
```

## Definição de senha

- Requisitos: ≥ 8 chars, 1 maiúscula, 1 número, 1 especial
- Validação real-time: ✅ verde (met) ou ○ muted (unmet) por requisito
- Botão **Ativar** habilitado somente quando 100 % válido

## Welcome Screen (`/bem-vindo`)

```
┌────────────────────────────────────┐
│  🎉 confetti overlay sutil         │
│  [Avatar 80px] border 3px gold     │
│  box-shadow: 0 0 20px gold/20%    │
│                                    │
│  Bem-vindo, João!                  │
│  h1 Michroma 28px                  │
│                                    │
│  Sua conta Síndico Pro está ativa. │
│                                    │
│  ✓ Publicar até 12 vídeos/mês     │
│  ✓ Criar assembleias              │
│  ✓ Exportar gestão em PDF         │
│  ✓ Acesso a cursos certificados   │
│                                    │
│  [Explorar Plataforma →] primary   │
│  [Completar meu perfil] ghost      │
└────────────────────────────────────┘
```

Features dinâmicas por `plan_tier` — exibir 4-6 itens mais relevantes do plano ativado (consultar `plans.features` e `plans.quotas` no backend; ver [[../../../../00-product/business-model#9-procedimento-canônico-para-instanciar-plano-novo]]).

## Seleção de categorias (Empresa)

Pós-ativação, tela para selecionar até **5 categorias técnicas** (Manual Cap. 6).
Tags com **+** (não selecionada) e **✕** (selecionada). Progress: "3/5 selecionadas".

```css
.interest-tag {
  display: inline-flex; align-items: center; gap: 6px;
  padding: 8px 16px; border-radius: var(--radius-full);
  border: 1px solid var(--border); background: var(--card);
  font-family: 'Manrope'; font-size: 14px; font-weight: 500;
  cursor: pointer; transition: all var(--duration-moderate) var(--ease-out);
}
.interest-tag:hover { border-color: var(--primary); background: oklch(0.715 0.120 84.2 / 0.05); }
.interest-tag.selected { background: var(--primary); color: var(--primary-foreground); border-color: var(--primary); }
```

## Componentes

`AuthLayout`, `LoginForm`, `SignupForm`, `OTPField`, `PasswordForm`, `PasswordRequirement`, `WelcomeScreen`, `InterestTagSelector`.

## Validação (Zod)

```ts
const loginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(1),
});

const signupSchema = z.object({
  name: z.string().min(3),
  email: z.string().email(),
  cpf: z.string().regex(/^\d{11}$/),
});

const passwordSchema = z
  .object({
    password: z
      .string()
      .min(8)
      .regex(/[A-Z]/)
      .regex(/[0-9]/)
      .regex(/[^a-zA-Z0-9]/),
    confirm: z.string(),
  })
  .refine((d) => d.password === d.confirm, { path: ["confirm"] });
```

## Estados

| Estado | Comportamento |
|---|---|
| Loading submit | Spinner + disabled + "Criando conta..." |
| Email duplicado | Toast `destructive` "Email já em uso" |
| OTP expirado | Toast `warning` "Código expirado" |
| Conta ativada | Toast `success` → redirect `/bem-vindo` |
| URL sem params | Fallback `role=resident, plan_tier=base` |

> Estados padrão (loading / empty / error / success / permission-denied) seguem [[../../patterns/ui-states|patterns/ui-states]].

## Links

- [[../../../web/ui-catalog#a-app-auth]] — telas AUTH1-AUTH8 do catálogo macro
- [[../../../web/README#43-auth-flow-zitadel-oidc--pkce]] — fluxo OIDC + PKCE
- [[../../../../00-product/business-model]] — `plan_tier` canônico
- [[../../../../01-domain/bounded-contexts|bounded-contexts]] — BC `identity`
- [[../../../../08-security/BEYOND_CORP|BEYOND_CORP]] — zero-trust
- [[../../../../02-architecture/design-tokens|design-tokens]] — OKLCH + tipografia
- [[../../patterns/ui-states|patterns/ui-states]] — estados obrigatórios
