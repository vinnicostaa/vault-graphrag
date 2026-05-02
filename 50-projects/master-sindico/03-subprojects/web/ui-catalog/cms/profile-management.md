---
title: Profile Management — UI Spec
type: ui-spec
tags: [ui-catalog, master-sindico, cms, profile, identity, cross]
bc: identity
source: _chaos/04-PROFILE-MANAGEMENT.md (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# 04 — Perfil & Gestão de Perfis

> **Origem**: `_chaos/04-PROFILE-MANAGEMENT.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Tradução aplicada**: `N1/N2/N3` → `trial/base/plus/pro` (ADR-0032 / D-081 / D-096); "Morador Pagante" removido (D-126); "My Síndico" → "Master Síndico".

> Sprint 1 · App `cms` · Rotas: `/perfil/:id`, `/perfil/editar`, `/meu-perfil`, `/meu-status`, `/meu-plano`
> Referências: Instagram Profile + Genaigurus Badges + Flavor Cover.

---

## Regras de negócio

- Cada **persona** tem layout de perfil diferente (Síndico, Morador, Empresa, Vizinhança).
- **Visibilidade ABAC**: perfil só renderiza quando matriz `(viewer_role × viewer_plan_tier × target_role)` permitir (ver [[../../../../00-product/business-model#41-matriz-consolidada]]).
- **Morador é gratuito** (`plan_tier=base`). Quando opta pelo addon **Banco de Talentos** (D-099, M3), seu vídeo-currículo e ficha tornam-se visíveis/indexáveis para empresas `plus`+ que tenham essa permissão.
- **Sem dados de contato público** (telefone, email, WhatsApp, site). Liberação só via fluxo Connect Me com consent LGPD.
- **Status do Síndico** (Bronze / Prata / Ouro / Diamante — ver [[../../../../00-product/sub-produtos/03-reputacao]]) baseado em critérios objetivos determinísticos (D-051; reputação imutável via PATCH — GR-028).

## Layout — Perfil Desktop

```
┌──────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                    │
│          │  COVER BANNER (300px, full-width)                  │
│          │  ┌──────────────────────────────────────────────┐ │
│          │  │   gradient navy → navy-variant               │ │
│          │  │                                              │ │
│          │  │   [Avatar 120px]                             │ │
│          │  │   border 4px gold (se Pro)                   │ │
│          │  └──────────────────────────────────────────────┘ │
│          │                                                    │
│          │  INFO                                              │
│          │  ┌──────────────────────────────────────────────┐ │
│          │  │ Nome Completo          [Editar Perfil] btn   │ │
│          │  │ @handle • Síndico Pro • 🥇 Ouro             │ │
│          │  │                                              │ │
│          │  │ Bio: Síndico profissional há 8 anos...       │ │
│          │  │ 📍 São Paulo, SP                             │ │
│          │  │ 🏢 3 condomínios • 📹 47 vídeos • 📚 5 mód │ │
│          │  │                                              │ │
│          │  │ [Connect Me] [Compartilhar]                  │ │
│          │  └──────────────────────────────────────────────┘ │
│          │                                                    │
│          │  TABS                                              │
│          │  [Vídeos] [Cursos] [Sobre] [Gestão]               │
│          │   ═══════                                          │
│          │  Grid de Video Cards (4 cols)                      │
│          │  [Card] [Card] [Card] [Card]                      │
└──────────────────────────────────────────────────────────────┘
```

## Layout — Perfil Mobile

```
┌─────────────────────────┐
│  COVER (180px)          │
│  [← Back]    [⚙ Edit]  │
│                         │
│  [Avatar 80px]          │
│  overlap cover -40px    │
├─────────────────────────┤
│  Nome Completo          │
│  @handle • 🥇 Ouro     │
│                         │
│  Stats Row (horizontal):│
│  3 Condos │ 47 Vídeos  │
│  5 Módulos│             │
│                         │
│  [Connect Me] [Share]   │
│                         │
│  [Vídeos][Cursos][Sobre]│
│   ═══════               │
│                         │
│  Grid cols-2 gap-2      │
│  [VC] [VC]              │
│  [VC] [VC]              │
└─────────────────────────┘
```

## CSS

```css
.profile-cover {
  width: 100%; height: 300px; /* 180px mobile */
  background: linear-gradient(135deg, var(--secondary), var(--surface-variant));
  position: relative;
}
.profile-avatar-wrapper {
  position: absolute; bottom: -48px; left: 24px;
}
.profile-avatar {
  width: 120px; height: 120px; /* 80px mobile */
  border-radius: 50%; border: 4px solid var(--card);
  object-fit: cover;
}
.profile-avatar[data-plan-tier="pro"] {
  border-color: var(--primary);
  box-shadow: 0 0 16px oklch(0.715 0.120 84.2 / 0.3);
}
.profile-info { padding: 56px 24px 24px; }
.profile-name { font-family: 'Michroma'; font-size: 24px; color: var(--foreground); }
.profile-handle {
  font-family: 'Manrope'; font-size: 14px; color: var(--muted-foreground);
  display: flex; align-items: center; gap: 8px; margin-top: 4px;
}
.profile-bio {
  font-family: 'Manrope'; font-size: 14px; color: var(--foreground);
  line-height: 1.6; margin: 12px 0; max-width: 600px;
}
.profile-stats { display: flex; gap: 24px; margin: 12px 0; }
.profile-stat { text-align: center; }
.profile-stat-value { font-family: 'Manrope'; font-size: 18px; font-weight: 700; color: var(--foreground); }
.profile-stat-label { font-family: 'Manrope'; font-size: 12px; color: var(--muted-foreground); }
.profile-actions { display: flex; gap: 8px; margin-top: 16px; }
```

## Variações por persona

### Perfil do Síndico

- Status badge (Bronze / Prata / Ouro / Diamante) ao lado do nome.
- Stats: nº condomínios, vídeos assistidos (> 70 %), módulos concluídos.
- Tabs: Vídeos (`plus`/`pro`), Gestão (timeline), Sobre.
- **Connect Me** visível conforme [[../../../../00-product/business-model#41-matriz-consolidada]].
- `trial`/`base`: sem perfil público, sem vídeos publicados.

### Perfil do Morador

- Aba **Banco de Talentos** (D-099 — opcional, M3): vídeo-currículo 90 s fixo no topo (player embed se existir).
- Badge "Disponível" se currículo ativo.
- Tabs: Banco de Talentos (vídeo + ficha), Sobre.
- Sem Banco de Talentos ativo: perfil mínimo, sem currículo.

> "Morador Pagante" não existe mais (D-126). Morador é sempre `plan_tier=base` gratuito; Banco de Talentos é feature opcional gratuita.

### Perfil da Empresa

- Logo como avatar (quadrado com `border-radius: var(--radius-lg)`).
- Até 5 categorias técnicas como tags.
- Badge de plano `Plus` ou `Pro`.
- Vídeo institucional fixo (se `pro`).
- Tabs: Vídeos, Cursos (se `pro`), Sobre.
- **SEM dados de contato visíveis** (Connect Me-only).

### Perfil do Comércio (Vizinhança / Parceiro Local)

- Logo + nome + endereço + CEP.
- Promoção do dia (se ativa).
- Tabs: Promoções, Sobre.

## Tabs

```css
.profile-tabs {
  display: flex; border-bottom: 1px solid var(--border); margin-top: 24px;
}
.profile-tab {
  padding: 12px 20px; font-family: 'Manrope'; font-size: 14px; font-weight: 500;
  color: var(--muted-foreground); cursor: pointer; position: relative;
  transition: color var(--duration-moderate) var(--ease-out);
}
.profile-tab:hover { color: var(--foreground); }
.profile-tab.active { color: var(--primary); font-weight: 600; }
.profile-tab.active::after {
  content: ''; position: absolute; bottom: -1px; left: 0; right: 0;
  height: 2px; background: var(--primary); border-radius: 1px;
}
```

## Status Dashboard (`/meu-status`)

```
┌────────────────────────────────────────┐
│  Meu Status                            │
│                                        │
│  [🥇] Ouro ← badge grande 48px        │
│                                        │
│  ┌──────┐ ┌──────┐ ┌──────┐          │
│  │  47  │ │   3  │ │   5  │          │
│  │Vídeos│ │Condos│ │Módulo│          │
│  └──────┘ └──────┘ └──────┘          │
│                                        │
│  Para Diamante 💎:                     │
│  ████████████░░░ 80%                   │
│  Faltam: 3 vídeos + 2 módulos         │
│                                        │
│  Histórico:                            │
│  ● Jan/2026 — Atingiu Ouro            │
│  ● Out/2025 — Atingiu Prata           │
│  ● Jul/2025 — Início Bronze           │
└────────────────────────────────────────┘
```

> Status Bronze→Diamante é determinístico (GR-028 / D-051). Detalhes da fórmula em [[../../../../00-product/sub-produtos/03-reputacao]].

## Edição de perfil (`/perfil/editar`)

```
┌────────────────────────────────────────┐
│  Editar Perfil                         │
│                                        │
│  [Foto ___] [Upload] (circular 120px) │
│                                        │
│  [Nome _________________________]     │
│  [Bio __________________________]     │
│  [Localização __________________]     │
│                                        │
│  Para Síndico:                         │
│  [Condomínios vinculados] +Adicionar  │
│                                        │
│  Para Empresa:                         │
│  Categorias: [tag][tag][tag] +Editar  │
│                                        │
│  [Salvar Alterações →] primary        │
│  [Cancelar] ghost                     │
└────────────────────────────────────────┘
```

## Componentes

`ProfilePage`, `ProfileCover`, `ProfileAvatar`, `ProfileInfo`, `ProfileStats`, `ProfileTabs`, `ProfileVideoGrid`, `StatusBadge`, `StatusDashboard`, `EditProfileForm`, `PlanCard`.

## Links

- [[../../../web/ui-catalog#b8-conta-sistema-sys1-sys8]] — telas SYS1-SYS8 (Conta + Sistema)
- [[../../../../00-product/sub-produtos/03-reputacao]] — fórmula Bronze→Diamante
- [[../../../../00-product/business-model]] — `plan_tier` e quotas
- [[../../patterns/ui-states|patterns/ui-states]] — estados obrigatórios
- [[../../../../02-architecture/design-tokens|design-tokens]] — tokens
