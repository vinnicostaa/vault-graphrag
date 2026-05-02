# 01 — AUTENTICAÇÃO & ONBOARDING

> Sprint 1 · Rotas: /login, /cadastro, /verificar-email, /definir-senha, /bem-vindo
> Deps: @corvu/otp-field, @tanstack/solid-form, zod

---

## REGRAS DE NEGÓCIO

- Cadastro parametrizado via URL: mastersindico.com/cadastro?role=sindico&plan=pro
- Fluxo: Formulário → Email OTP → Definir Senha → Welcome Screen
- NÃO há OCR, biometria ou validação de documento — apenas email/token
- JWT emitido após ativação, role travada no backend
- Telas de auth são públicas (sem middleware)

## ROTAS

| Rota | Componente | Descrição |
|---|---|---|
| /login | LoginPage | Formulário email+senha |
| /cadastro | SignupPage | Formulário com role/plan de URL params |
| /verificar-email | VerifyEmailPage | OTP 6 dígitos |
| /definir-senha | SetPasswordPage | Criar senha com requisitos |
| /bem-vindo | WelcomePage | Sucesso + features do plano |

## LAYOUT — LOGIN (Desktop split 50/50)

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

Mobile: Painel vira header 120px (logo+frase), formulário ocupa restante full-width.

## LAYOUT — CADASTRO

Mesmo split 50/50. Texto do painel adapta ao role detectado:
- Síndico: "Sua gestão merece registro."
- Empresa: "Autoridade se prova com conhecimento."
- Morador: "Participação responsável."

Badge de plano visível: "Síndico Pro" em badge variant=default abaixo do título.

### Campos por Role

Morador: Nome, Email, CPF
Síndico: Nome, Email, CPF, Telefone
Empresa: Razão Social, CNPJ, Email, Responsável, Telefone
Vizinhança: Nome estabelecimento, CNPJ, Email, CEP, Endereço

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
  background: var(--card); border: 2px solid var(--border); border-radius: var(--radius);
  transition: all 200ms;
}
.otp-field:focus { border-color: var(--primary); box-shadow: 0 0 0 3px oklch(0.715 0.120 84.2 / 0.2); }
.otp-field[data-filled] { border-color: var(--primary); }
```

## DEFINIÇÃO DE SENHA

Requisitos: ≥8 chars, 1 maiúscula, 1 número, 1 especial
Validação real-time: ✅ verde (met) ou ○ muted (unmet) por requisito
Botão "Ativar" enabled somente quando 100% válido

## WELCOME SCREEN (/bem-vindo)

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

Features dinâmicas por plano — exibir 4-6 itens mais relevantes do plano ativado.

## SELEÇÃO DE CATEGORIAS (Empresa)

Pós-ativação, tela para selecionar até 5 categorias técnicas (Manual Cap. 6).
Tags com "+" (não selecionada) e "✕" (selecionada). Progress "3/5 selecionadas".

```css
.interest-tag {
  display: inline-flex; align-items: center; gap: 6px;
  padding: 8px 16px; border-radius: 9999px;
  border: 1px solid var(--border); background: var(--card);
  font-family: 'Manrope'; font-size: 14px; font-weight: 500;
  cursor: pointer; transition: all 200ms;
}
.interest-tag:hover { border-color: var(--primary); background: oklch(0.715 0.120 84.2 / 0.05); }
.interest-tag.selected { background: var(--primary); color: var(--primary-foreground); border-color: var(--primary); }
```

## COMPONENTES

AuthLayout, LoginForm, SignupForm, OTPField, PasswordForm, PasswordRequirement, WelcomeScreen, InterestTagSelector

## VALIDAÇÃO (Zod)

```ts
const loginSchema = z.object({ email: z.string().email(), password: z.string().min(1) });
const signupSchema = z.object({ name: z.string().min(3), email: z.string().email(), cpf: z.string().regex(/^\d{11}$/) });
const passwordSchema = z.object({
  password: z.string().min(8).regex(/[A-Z]/).regex(/[0-9]/).regex(/[^a-zA-Z0-9]/),
  confirm: z.string()
}).refine(d => d.password === d.confirm, { path: ["confirm"] });
```

## ESTADOS

Loading submit: spinner + disabled + "Criando conta..."
Email duplicado: toast destructive "Email já em uso"
OTP expirado: toast warning "Código expirado"
Conta ativada: toast success → redirect /bem-vindo
URL sem params: fallback role=morador, plan=base
