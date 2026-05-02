---
title: Settings — Configurações do Usuário (UI Spec)
type: ui-spec
tags: [ui-catalog, master-sindico, cross, settings, account, identity]
bc: cross
source: _chaos/22-SETTINGS.md (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# 22 — Configurações do Usuário (Settings)

> **Origem**: `_chaos/22-SETTINGS.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Tradução aplicada**: `N1/N2/N3` → `trial/base/plus/pro` (ADR-0032 / D-081 / D-096); "Morador Pagante" removido (D-126); "My Síndico" → "Master Síndico".

> Sprint 1-2 · App `cms` · Todos os perfis autenticados.
> Dependências: [[../../../../02-architecture/design-tokens|design-tokens]] · [[../shell/app-shell|app-shell]] · [[../auth/onboarding|auth/onboarding]].

---

## 1. Contexto

Settings agrupa todas as opções de personalização, segurança e gerenciamento da conta. Design segue referência **Meta** (Instagram / Facebook): navegação por categorias na lateral esquerda (desktop) ou lista empilhada (mobile).

### Acesso
- Todas as personas autenticadas.
- Cada persona vê apenas opções relevantes ao seu `role` / `plan_tier`.
- Acessível via avatar dropdown na topbar → "Configurações" ou via sidebar / bottom nav.

---

## 2. Layout

### Desktop (≥ 1024 px) — Meta-style

```
┌────────────┬────────────────────────────────────────────────┐
│ SIDEBAR    │  SETTINGS CONTENT                              │
│ SETTINGS   │  ┌──────────────────────────────────────────┐ │
│ (280px)    │  │  Título da Seção                         │ │
│            │  │  Descrição breve                         │ │
│ Conta      │  ├──────────────────────────────────────────┤ │
│ Notific.   │  │  (Campos da seção ativa)                 │ │
│ Aparência  │  └──────────────────────────────────────────┘ │
│ Privacid.  │                                                │
│ Segurança  │                                                │
│ Plano      │                                                │
└────────────┴────────────────────────────────────────────────┘
```

### Mobile (< 768 px) — Lista empilhada
Lista de categorias → push navigation para detalhe.

### Sidebar Settings

Item ativo: `--muted` background, border-left 3 px `--primary` (Gold).

```css
.settings-sidebar-item {
  display: flex; align-items: center; gap: 12px;
  padding: 12px 16px;
  font-family: 'Manrope'; font-size: 14px;
  color: var(--muted-foreground); cursor: pointer;
  transition: all var(--duration-moderate) var(--ease-out);
}
.settings-sidebar-item:hover { background: oklch(var(--muted) / 0.5); }
.settings-sidebar-item.active {
  background: var(--muted); color: var(--foreground);
  border-left: 3px solid var(--primary);
  font-weight: 600;
}
```

---

## 3. Seções

### 3.1 Conta — `/configuracoes/conta`

- **Foto de perfil**: avatar 80 px circular + [Alterar foto] / [Remover].
- **Informações pessoais**: Nome, Email (verificado, alterável com revalidação), Telefone, Bio (300 chars).
- **Dados específicos do perfil** (varia conforme `role`):
  - Síndico: condomínios vinculados (read-only) + Status Bronze→Diamante.
  - Empresa: CNPJ (read-only), Razão Social, Categorias técnicas (até 5) [Editar].
  - Morador: Unidade (read-only), Condomínio (read-only).
- **Zona de Perigo**: Desativar conta (modal com input de senha).

### 3.2 Notificações — `/configuracoes/notificacoes`

Grupos de toggles (Kobalte Switch):

- **Comunicados da Plataforma** (sempre ativos — não desativável).
- **Conta e Segurança** (sempre ativo — confirmação de email, redefinição de senha, novo login).
- **Assembleias e Eventos** (Morador / Síndico): Nova assembleia, Votação aberta, Resultado, Avaliação disponível.
- **Conteúdo e Cursos** (Síndico `plus`+, Empresa `plus`+): Novo módulo, Certificado gerado, Live ao vivo.
- **Oportunidades** (Morador com Banco Talentos): Empresa demonstrou interesse.
- **Cupons e Benefícios**: Promoção exclusiva do condomínio, Cupom expirado.

Salvamento automático (cada toggle dispara mutation imediatamente). Sem botão "Salvar".

```css
/* Kobalte Switch */
.kb-switch__control {
  width: 40px; height: 22px; border-radius: 11px;
  background: var(--muted);
  transition: background var(--duration-moderate) ease;
}
.kb-switch__control[data-checked] { background: var(--primary); }
```

### 3.3 Aparência — `/configuracoes/aparencia`

**Tema**: 3 cards selecionáveis — Claro / Escuro / Sistema. Integração via Kobalte `ColorModeProvider` (já em `src/index.tsx`). Persistência em `localStorage`.

**Idioma** (futuro): select desabilitado com "Português (Brasil)".

### 3.4 Privacidade — `/configuracoes/privacidade`

- **Visibilidade do perfil**: toggles "Perfil visível na busca", "Mostrar bio", "Mostrar região".
- **Vídeo-Currículo** (Morador com Banco Talentos): toggle "Visível para empresas".
- **Dados pessoais (LGPD)**:
  - [Solicitar meus dados] — Art. 18 LGPD; export JSON / PDF por email.
  - [Solicitar exclusão] — outline `--destructive` + modal confirmação severo + input senha.

### 3.5 Segurança — `/configuracoes/seguranca`

- **Alterar senha**: senha atual + nova + confirmar + indicador de força (4 segmentos).
- **Sessões ativas**: lista de sessões com [Encerrar sessão] + botão "Encerrar todas as outras".
- **Dispositivo único**: box informativo (regra ABAC `1-device-policy` — ADR-0014).

```css
.password-strength-bar {
  display: flex; gap: 4px; height: 4px; margin-top: 8px;
}
.password-strength-segment {
  flex: 1; background: var(--muted); border-radius: 2px;
  transition: background var(--duration-moderate);
}
.password-strength-segment.weak     { background: var(--destructive); }
.password-strength-segment.medium   { background: var(--warning); }
.password-strength-segment.strong   { background: oklch(0.6 0.15 149); }
.password-strength-segment.very-strong { background: var(--success); }
```

### 3.6 Meu Plano — `/configuracoes/plano`

- **Plano atual** (card destaque): nome (Michroma 20 px gold) + Status + Desde + Próxima cobrança + Valor.
- **O que seu plano inclui** (lista com ✅ verde).
- **Histórico de pagamentos** (mini-tabela: Data / Valor / Status).
- **Gerenciar assinatura**: [Alterar Plano] (redireciona Stripe Customer Portal) / [Cancelar Assinatura] (modal de retenção).

> Pricing variável (não fixar no spec) — lê de `plans` no backend. Ver [[../../../../00-product/business-model#8-pricing—variáveis-de-negócio-fora-do-canônico]].

---

## 4. Validações Zod

```ts
const AccountSchema = z.object({
  fullName: z.string().min(3).max(100),
  email: z.string().email(),
  phone: z.string().optional(),
  bio: z.string().max(300).optional(),
});

const ChangePasswordSchema = z
  .object({
    currentPassword: z.string().min(1, "Senha atual obrigatória"),
    newPassword: z
      .string()
      .min(8, "Mínimo 8 caracteres")
      .regex(/[A-Z]/, "Uma letra maiúscula")
      .regex(/[0-9]/, "Um número")
      .regex(/[^A-Za-z0-9]/, "Um caractere especial"),
    confirmPassword: z.string(),
  })
  .refine((d) => d.newPassword === d.confirmPassword, {
    message: "Senhas não coincidem",
    path: ["confirmPassword"],
  });

const CompanyCategoriesSchema = z.object({
  categories: z.array(z.string()).min(1).max(5),
});
```

## 5. Componentes

`SettingsShell`, `SettingsSidebar`, `SettingsSection`, `ToggleRow`, `ThemeSelector`, `PasswordStrength`, `SessionCard`, `PlanCard`, `PaymentHistory`, `DangerZone`, `PhotoUpload`, `CancellationModal`.

## 6. Estados

Padrão (loading / empty / error / success / permission-denied) seguem [[../../patterns/ui-states|patterns/ui-states]].

## Links

- [[../../../web/ui-catalog#b8-conta-sistema-sys1-sys8]] — telas SYS1-SYS8
- [[../../../../00-product/business-model]]
- [[../../../../08-security/lgpd|08-security/lgpd]]
- [[../auth/onboarding|auth/onboarding]]
- [[../../patterns/ui-states|patterns/ui-states]]
- [[../../../../02-architecture/design-tokens|design-tokens]]
