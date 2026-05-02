# 04 — PERFIL & GESTÃO DE PERFIS

> Sprint 1 · Rotas: /perfil/:id, /perfil/editar, /meu-perfil, /meu-status, /meu-plano
> Referências: Instagram Profile + Genaigurus Badges + Flavor Cover

---

## REGRAS DE NEGÓCIO

- Cada persona tem layout de perfil diferente (Síndico, Morador, Empresa, Vizinhança)
- Perfil visível apenas se Matriz Funcional permitir (middleware de visibilidade)
- Morador Pagante agora visível/indexável para empresas
- Sem dados de contato público (telefone, email, WhatsApp, site)
- Status do Síndico (Bronze/Prata/Ouro/Diamante) baseado em critérios objetivos

## LAYOUT — PERFIL DESKTOP

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
│          │  │ @handle • Síndico N3 • 🥇 Ouro              │ │
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

## LAYOUT — PERFIL MOBILE

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
  position: absolute; bottom: -48px; left: 24px; /* center em mobile */
}
.profile-avatar {
  width: 120px; height: 120px; /* 80px mobile */
  border-radius: 50%; border: 4px solid var(--card);
  object-fit: cover;
}
.profile-avatar[data-plan="pro"],
.profile-avatar[data-plan="n3"] {
  border-color: var(--primary);
  box-shadow: 0 0 16px oklch(0.715 0.120 84.2 / 0.3);
}
.profile-info { padding: 56px 24px 24px; /* compensar avatar overlap */ }
.profile-name { font-family: 'Michroma'; font-size: 24px; color: var(--foreground); }
.profile-handle { font-family: 'Manrope'; font-size: 14px; color: var(--muted-foreground); display: flex; align-items: center; gap: 8px; margin-top: 4px; }
.profile-bio { font-family: 'Manrope'; font-size: 14px; color: var(--foreground); line-height: 1.6; margin: 12px 0; max-width: 600px; }
.profile-stats { display: flex; gap: 24px; margin: 12px 0; }
.profile-stat { text-align: center; }
.profile-stat-value { font-family: 'Manrope'; font-size: 18px; font-weight: 700; color: var(--foreground); }
.profile-stat-label { font-family: 'Manrope'; font-size: 12px; color: var(--muted-foreground); }
.profile-actions { display: flex; gap: 8px; margin-top: 16px; }
```

## VARIAÇÕES POR PERSONA

### Perfil do Síndico
- Status badge (Bronze/Prata/Ouro/Diamante) ao lado do nome
- Stats: nº condomínios, vídeos assistidos (>70%), módulos concluídos
- Tabs: Vídeos (N2/N3), Gestão (timeline), Sobre
- Connect Me visível conforme Matriz Funcional
- N1: sem perfil público, sem vídeos publicados

### Perfil do Morador Pagante
- Vídeo-Currículo fixo no topo (player embed se existir)
- Badge "Disponível" se currículo ativo
- Tabs: Currículo (vídeo+ficha), Sobre
- Base: perfil mínimo, sem currículo

### Perfil da Empresa
- Logo como avatar (quadrado com rounded-lg)
- Até 5 categorias técnicas como tags
- Badge de plano "Plus" ou "Pro"
- Vídeo institucional fixo (se Pro)
- Tabs: Vídeos, Cursos (se Pro), Sobre
- SEM dados de contato visíveis

### Perfil do Comércio (Vizinhança)
- Logo + nome + endereço + CEP
- Promoção do dia (se ativa)
- Tabs: Promoções, Sobre

## TABS

```css
.profile-tabs { display: flex; border-bottom: 1px solid var(--border); margin-top: 24px; }
.profile-tab {
  padding: 12px 20px; font-family: 'Manrope'; font-size: 14px; font-weight: 500;
  color: var(--muted-foreground); cursor: pointer; position: relative;
  transition: color 200ms;
}
.profile-tab:hover { color: var(--foreground); }
.profile-tab.active { color: var(--primary); font-weight: 600; }
.profile-tab.active::after {
  content: ''; position: absolute; bottom: -1px; left: 0; right: 0;
  height: 2px; background: var(--primary); border-radius: 1px;
}
```

## STATUS DASHBOARD (/meu-status)

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

## EDIÇÃO DE PERFIL (/perfil/editar)

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

## COMPONENTES

ProfilePage, ProfileCover, ProfileAvatar, ProfileInfo, ProfileStats, ProfileTabs, ProfileVideoGrid, StatusBadge, StatusDashboard, EditProfileForm, PlanCard
