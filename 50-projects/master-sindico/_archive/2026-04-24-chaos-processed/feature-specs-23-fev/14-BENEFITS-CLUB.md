# 14 — CLUBE DE BENEFÍCIOS / VIZINHANÇA

> Sprint 3 · Rotas: /vizinhanca, /vizinhanca/estabelecimento/:id, /vizinhanca/meu-negocio (lojista)
> Hub de integração com comércio local. Gera uso diário e pertencimento.

---

## REGRAS DE NEGÓCIO

### Conceito
- Aba "Vizinhança" é o hub de comércio local na plataforma
- Busca filtrada por CEP — morador vê apenas benefícios da sua região
- Estabelecimentos são perfis do tipo "Vizinhança" na plataforma
- Promoções vivem no doc 13 (Coupon Engine). Este doc cobre: navegação, perfil do estabelecimento, listagem, descoberta por mapa/CEP

### Permissões (Matriz Funcional)
| Ação | Base | Morador Pg | Síndicos | Emp. Plus/Pro | Vizinhança |
|------|------|-----------|----------|---------------|------------|
| Ver benefícios locais | ✅ | ✅ | ✅ | ✅ | ✅ |
| Visualizar currículos | — | — | — | — | ✅ |
| Perfil público | — | — | — | — | ✅ |
| Cadastrar promoções | — | — | — | — | ✅ |

### CEP como critério principal
- Morador define seu CEP no cadastro/configurações
- Busca retorna estabelecimentos dentro do raio do CEP (definido pelo backend)
- Estabelecimento cadastra CEP de atuação (pode ter mais de um, conforme configuração)
- Se morador não tem CEP: banner pedindo para configurar

### Perfil do Estabelecimento
- Nome, Logo/foto, Descrição, Endereço, CEP, Categoria, Horário de funcionamento
- Promoção ativa do dia (se houver)
- Conteúdo limitado: até 2 fotos + 1 vídeo (60s) conforme Matriz Funcional

---

## LAYOUT — HUB VIZINHANÇA (Página Principal)

```
┌──────────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                        │
│          │  VIZINHANÇA                                            │
│          │  h2 Michroma 20px                                      │
│          │  Descubra o melhor da sua região                       │
│          │                                                        │
│          │  📍 CEP: 01310-100 • Bela Vista, SP       [Alterar]   │
│          │                                                        │
│          │  🔍 ┌──────────────────────────────────────────────┐   │
│          │     │ Buscar estabelecimento...                    │   │
│          │     └──────────────────────────────────────────────┘   │
│          │                                                        │
│          │  ─── CATEGORIAS ───────────────────────────────── →    │
│          │  [🍕Alimentação] [💈Serviços] [🏋️Saúde] [🎭Lazer]    │
│          │  [🛒Mercado] [🧹Limpeza] [🔧Reparos] [📦Outros]      │
│          │                                                        │
│          │  ─── PROMOÇÕES DO DIA 🔥 ──────────────────────────   │
│          │  ← scroll horizontal                                   │
│          │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐   │
│          │  │ 🏪 Pad.  │ │ 🍕 Pizza │ │ 💈 Barb. │ │ ...    │   │
│          │  │ Sol      │ │ Nostra   │ │ Classic  │ │        │   │
│          │  │ 10% OFF  │ │ 2por1    │ │ R$15 OFF │ │        │   │
│          │  │[Pegar →] │ │[Pegar →] │ │[Pegar →] │ │        │   │
│          │  └──────────┘ └──────────┘ └──────────┘ └────────┘   │
│          │                                                        │
│          │  ─── PERTO DE VOCÊ ────────────────────────────────   │
│          │                                                        │
│          │  ┌───────────────────┐ ┌───────────────────┐          │
│          │  │ [foto]            │ │ [foto]            │          │
│          │  │ Mercadinho Dona   │ │ Lavanderia Express│          │
│          │  │ Maria             │ │                   │          │
│          │  │ ⭐ 4.8 • 200m    │ │ ⭐ 4.5 • 350m    │          │
│          │  │ Alimentação       │ │ Serviços          │          │
│          │  └───────────────────┘ └───────────────────┘          │
│          │  ┌───────────────────┐ ┌───────────────────┐          │
│          │  │ [foto]            │ │ [foto]            │          │
│          │  │ Gym Power         │ │ Pet Shop Amigo    │          │
│          │  │ ⭐ 4.6 • 500m    │ │ ⭐ 4.9 • 150m    │          │
│          │  │ Saúde/Fitness     │ │ Pet               │          │
│          │  └───────────────────┘ └───────────────────┘          │
│          │                                                        │
│          │  [Ver todos →]                                         │
└──────────────────────────────────────────────────────────────────┘
```

### Seções da página
1. **Header**: Título + subtítulo + CEP do usuário
2. **Busca**: Input full-width
3. **Categorias**: Grid de ícones 4 colunas (mobile) ou horizontal scroll (desktop)
4. **Promoções do Dia**: Horizontal scroll de mini-cards com promo ativa
5. **Perto de Você**: Grid 2 colunas com estabelecimentos ordenados por distância

### CSS — Hub Layout

```css
.vizinhanca-hub {
  max-width: 900px;
  margin: 0 auto;
  padding: 0 16px;
}

.vizinhanca-header {
  margin-bottom: 24px;
}

.vizinhanca-title {
  font-family: 'Michroma';
  font-size: 20px;
  color: var(--foreground);
  letter-spacing: 0.05em;
  margin-bottom: 4px;
}

.vizinhanca-subtitle {
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--muted-foreground);
}

.vizinhanca-cep-bar {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 14px;
  background: var(--muted);
  border-radius: 8px;
  margin-bottom: 20px;
}

.vizinhanca-cep-icon {
  font-size: 16px;
}

.vizinhanca-cep-text {
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--foreground);
  font-weight: 500;
  flex: 1;
}

.vizinhanca-cep-change {
  font-family: 'Manrope';
  font-size: 13px;
  color: var(--primary);
  font-weight: 600;
  cursor: pointer;
  background: none;
  border: none;
}
.vizinhanca-cep-change:hover { text-decoration: underline; }
```

### CSS — Search Bar

```css
.vizinhanca-search {
  position: relative;
  margin-bottom: 24px;
}

.vizinhanca-search-input {
  width: 100%;
  padding: 12px 16px 12px 44px;
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 12px;
  font-family: 'Manrope';
  font-size: 15px;
  color: var(--foreground);
  outline: none;
  transition: border-color 200ms;
}
.vizinhanca-search-input:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 3px oklch(0.715 0.120 84.2 / 0.1);
}
.vizinhanca-search-input::placeholder {
  color: var(--muted-foreground);
}

.vizinhanca-search-icon {
  position: absolute;
  left: 14px;
  top: 50%;
  transform: translateY(-50%);
  color: var(--muted-foreground);
  font-size: 18px;
  pointer-events: none;
}
```

### CSS — Category Grid

```css
.vizinhanca-categories {
  margin-bottom: 28px;
}

.vizinhanca-section-label {
  font-family: 'Michroma';
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 0.12em;
  color: var(--muted-foreground);
  margin-bottom: 12px;
}

.vizinhanca-category-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
}

@media (min-width: 768px) {
  .vizinhanca-category-grid {
    grid-template-columns: repeat(8, 1fr);
  }
}

.vizinhanca-category-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  padding: 12px 8px;
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 12px;
  cursor: pointer;
  transition: all 200ms;
}
.vizinhanca-category-item:hover {
  border-color: var(--primary);
  background: oklch(0.715 0.120 84.2 / 0.05);
}
.vizinhanca-category-item.active {
  border-color: var(--primary);
  background: oklch(0.715 0.120 84.2 / 0.1);
}

.vizinhanca-category-icon {
  font-size: 24px;
}

.vizinhanca-category-label {
  font-family: 'Manrope';
  font-size: 11px;
  font-weight: 500;
  color: var(--foreground);
  text-align: center;
  line-height: 1.2;
}
```

### CSS — Promoções do Dia (Scroll Horizontal)

```css
.vizinhanca-promos-section {
  margin-bottom: 28px;
}

.vizinhanca-promos-scroll {
  display: flex;
  gap: 12px;
  overflow-x: auto;
  scroll-snap-type: x mandatory;
  -webkit-overflow-scrolling: touch;
  padding-bottom: 8px;
}
.vizinhanca-promos-scroll::-webkit-scrollbar { display: none; }

.vizinhanca-promo-mini {
  min-width: 160px;
  max-width: 160px;
  scroll-snap-align: start;
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 12px;
  flex-shrink: 0;
  transition: all 200ms;
}
.vizinhanca-promo-mini:hover {
  border-color: var(--primary);
}

.vizinhanca-promo-mini-icon {
  font-size: 28px;
  margin-bottom: 8px;
}

.vizinhanca-promo-mini-name {
  font-family: 'Manrope';
  font-size: 13px;
  font-weight: 600;
  color: var(--foreground);
  margin-bottom: 4px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.vizinhanca-promo-mini-offer {
  font-family: 'Manrope';
  font-size: 12px;
  font-weight: 600;
  color: var(--primary);
  margin-bottom: 10px;
}

.vizinhanca-promo-mini-btn {
  display: block;
  width: 100%;
  padding: 6px 10px;
  background: var(--primary);
  color: var(--primary-foreground);
  border: none;
  border-radius: 6px;
  font-family: 'Manrope';
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
  text-align: center;
  transition: all 200ms;
}
.vizinhanca-promo-mini-btn:hover { background: oklch(0.65 0.120 84.2); }
```

### CSS — Grid "Perto de Você"

```css
.vizinhanca-nearby { margin-bottom: 32px; }

.vizinhanca-nearby-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 16px;
}

@media (min-width: 1024px) {
  .vizinhanca-nearby-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

@media (max-width: 767px) {
  .vizinhanca-nearby-grid {
    grid-template-columns: 1fr;
  }
}

.vizinhanca-store-card {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 12px;
  overflow: hidden;
  cursor: pointer;
  transition: all 200ms;
}
.vizinhanca-store-card:hover {
  border-color: var(--primary);
  box-shadow: 0 4px 12px oklch(0 0 0 / 0.06);
}

.vizinhanca-store-photo {
  width: 100%;
  height: 130px;
  object-fit: cover;
  background: var(--muted);
}

.vizinhanca-store-body {
  padding: 12px;
}

.vizinhanca-store-name {
  font-family: 'Manrope';
  font-size: 15px;
  font-weight: 700;
  color: var(--foreground);
  margin-bottom: 4px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.vizinhanca-store-meta {
  display: flex;
  align-items: center;
  gap: 8px;
  font-family: 'Manrope';
  font-size: 13px;
  color: var(--muted-foreground);
}

.vizinhanca-store-rating {
  display: flex;
  align-items: center;
  gap: 3px;
  color: var(--primary);
  font-weight: 600;
}

.vizinhanca-store-distance {
  color: var(--muted-foreground);
}

.vizinhanca-store-category {
  font-family: 'Manrope';
  font-size: 12px;
  color: var(--muted-foreground);
  margin-top: 4px;
}

/* Badge de promo ativa sobreposta no canto do card */
.vizinhanca-store-promo-badge {
  position: absolute;
  top: 8px;
  right: 8px;
  background: var(--primary);
  color: var(--primary-foreground);
  padding: 2px 8px;
  border-radius: 6px;
  font-family: 'Manrope';
  font-size: 11px;
  font-weight: 600;
}
```

---

## LAYOUT — PERFIL DO ESTABELECIMENTO

Rota: `/vizinhanca/estabelecimento/:id`

```
┌──────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                    │
│          │  ┌─────────────────────────────────────────────┐   │
│          │  │                                             │   │
│          │  │          [FOTO BANNER / FACHADA]            │   │
│          │  │                     300px height            │   │
│          │  │                                             │   │
│          │  └─────────────────────────────────────────────┘   │
│          │                                                    │
│          │  ┌────────┐  Padaria Sol                           │
│          │  │ [logo] │  ⭐ 4.8                                │
│          │  │  80px  │  📍 Rua Augusta, 1200 • 200m          │
│          │  └────────┘  🕐 Seg-Sex 6h-20h • Sáb 6h-14h      │
│          │              📂 Alimentação • Padaria              │
│          │                                                    │
│          │  ┌── PROMOÇÃO ATIVA 🔥 ───────────────────────┐   │
│          │  │  10% OFF em toda loja                       │   │
│          │  │  Válido para compras acima de R$30          │   │
│          │  │  ⏰ Até 28/02/2026                          │   │
│          │  │                                             │   │
│          │  │  [🎟️ Pegar Cupom]                           │   │
│          │  └─────────────────────────────────────────────┘   │
│          │                                                    │
│          │  SOBRE                                              │
│          │  "Padaria artesanal no coração da Bela Vista..."   │
│          │                                                    │
│          │  GALERIA                                            │
│          │  ┌────────┐ ┌────────┐                             │
│          │  │ foto 1 │ │ foto 2 │   (máx 2 fotos)            │
│          │  └────────┘ └────────┘                             │
│          │                                                    │
│          │  VÍDEO                                              │
│          │  ┌─────────────────────────────────────────────┐   │
│          │  │  ▶ Vídeo institucional (máx 60s)            │   │
│          │  │            [player embed]                    │   │
│          │  └─────────────────────────────────────────────┘   │
│          │                                                    │
│          │  [← Voltar para Vizinhança]                       │
└──────────────────────────────────────────────────────────────┘
```

### CSS — Perfil Estabelecimento

```css
.store-profile { max-width: 800px; margin: 0 auto; }

.store-banner {
  width: 100%;
  height: 300px;
  object-fit: cover;
  border-radius: 16px;
  background: var(--muted);
}

@media (max-width: 767px) {
  .store-banner {
    height: 200px;
    border-radius: 0;
  }
}

.store-header {
  display: flex;
  gap: 16px;
  align-items: flex-start;
  margin-top: -40px;
  margin-left: 24px;
  position: relative;
  z-index: 2;
  margin-bottom: 16px;
}

.store-logo {
  width: 80px;
  height: 80px;
  border-radius: 16px;
  border: 3px solid var(--card);
  background: var(--card);
  object-fit: cover;
  box-shadow: 0 4px 12px oklch(0 0 0 / 0.1);
}

.store-info {
  padding-top: 44px; /* alinha com abaixo do logo */
}

.store-name {
  font-family: 'Michroma';
  font-size: 20px;
  color: var(--foreground);
  letter-spacing: 0.05em;
  margin-bottom: 4px;
}

.store-rating {
  display: flex;
  align-items: center;
  gap: 4px;
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--primary);
  font-weight: 600;
  margin-bottom: 6px;
}

.store-detail-line {
  display: flex;
  align-items: center;
  gap: 6px;
  font-family: 'Manrope';
  font-size: 13px;
  color: var(--muted-foreground);
  margin-bottom: 4px;
}

/* Promoção ativa destaque */
.store-promo-highlight {
  background: oklch(0.715 0.120 84.2 / 0.08);
  border: 1px solid var(--primary);
  border-radius: 12px;
  padding: 20px;
  margin: 24px 0;
}

.store-promo-label {
  font-family: 'Michroma';
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 0.12em;
  color: var(--primary);
  margin-bottom: 8px;
  display: flex;
  align-items: center;
  gap: 6px;
}

.store-promo-title {
  font-family: 'Manrope';
  font-size: 18px;
  font-weight: 700;
  color: var(--foreground);
  margin-bottom: 4px;
}

.store-promo-desc {
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--muted-foreground);
  margin-bottom: 8px;
  line-height: 1.5;
}

.store-promo-validity {
  font-size: 13px;
  color: var(--muted-foreground);
  margin-bottom: 16px;
}

/* Seções: Sobre, Galeria, Vídeo */
.store-section {
  margin-bottom: 28px;
}

.store-section-title {
  font-family: 'Michroma';
  font-size: 13px;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: var(--muted-foreground);
  margin-bottom: 12px;
}

.store-about-text {
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--foreground);
  line-height: 1.7;
}

.store-gallery {
  display: flex;
  gap: 12px;
}

.store-gallery-img {
  width: 200px;
  height: 150px;
  object-fit: cover;
  border-radius: 8px;
  background: var(--muted);
  cursor: pointer;
  transition: all 200ms;
}
.store-gallery-img:hover {
  transform: scale(1.02);
  box-shadow: 0 4px 12px oklch(0 0 0 / 0.1);
}

@media (max-width: 767px) {
  .store-gallery {
    flex-direction: column;
  }
  .store-gallery-img {
    width: 100%;
    height: 200px;
  }
}

.store-video-container {
  width: 100%;
  aspect-ratio: 16/9;
  background: var(--surface);
  border-radius: 12px;
  overflow: hidden;
}
```

---

## LAYOUT — PAINEL MEU NEGÓCIO (Lojista Vizinhança)

Rota: `/vizinhanca/meu-negocio`

```
┌──────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                    │
│          │  MEU NEGÓCIO                                       │
│          │  h2 Michroma 20px                                  │
│          │                                                    │
│          │  [Perfil] [Promoções] [Métricas]                  │
│          │   ← tabs Kobalte                                   │
│          │                                                    │
│          │  ═══ TAB PERFIL ════════════════════════════════   │
│          │                                                    │
│          │  Foto do estabelecimento                           │
│          │  ┌────────────────┐  [Alterar foto]               │
│          │  │    [preview]   │                                │
│          │  └────────────────┘                                │
│          │                                                    │
│          │  Nome *                                             │
│          │  [Padaria Sol                              ]       │
│          │                                                    │
│          │  Descrição *                                        │
│          │  [Padaria artesanal no coração...          ]       │
│          │  0/500                                              │
│          │                                                    │
│          │  Endereço *                                         │
│          │  [Rua Augusta, 1200                        ]       │
│          │                                                    │
│          │  CEP *                                              │
│          │  [01310-100                                ]       │
│          │                                                    │
│          │  Categoria *                                        │
│          │  [▼ Alimentação                            ]       │
│          │                                                    │
│          │  Horário de Funcionamento                           │
│          │  Seg-Sex: [06:00] às [20:00]                       │
│          │  Sáb:     [06:00] às [14:00]                       │
│          │  Dom:     [Fechado ▼]                              │
│          │                                                    │
│          │  Galeria (máx 2 fotos)                             │
│          │  ┌──────┐ ┌──────┐ ┌──────────┐                   │
│          │  │foto 1│ │foto 2│ │ + Upload │                   │
│          │  └──────┘ └──────┘ └──────────┘                   │
│          │                                                    │
│          │  Vídeo Institucional (máx 60s)                     │
│          │  ┌──────────────────────────────┐                  │
│          │  │ ▶ video-preview.mp4          │                  │
│          │  │ [Substituir]                 │                  │
│          │  └──────────────────────────────┘                  │
│          │                                                    │
│          │  [Salvar Alterações]                               │
│          │                                                    │
│          │  ═══ TAB PROMOÇÕES ═════════════════════════════   │
│          │  (Reutiliza doc 13 — Coupon Engine panel)          │
│          │                                                    │
│          │  ═══ TAB MÉTRICAS ══════════════════════════════   │
│          │  Cupons gerados: [342] este mês                    │
│          │  Visualizações do perfil: [1.2k] este mês          │
│          │  Promoções ativas: [2]                             │
└──────────────────────────────────────────────────────────────┘
```

### CSS — Tabs

```css
.meu-negocio-tabs {
  display: flex;
  gap: 0;
  border-bottom: 1px solid var(--border);
  margin-bottom: 24px;
}

.meu-negocio-tab {
  padding: 12px 20px;
  font-family: 'Manrope';
  font-size: 14px;
  font-weight: 500;
  color: var(--muted-foreground);
  background: none;
  border: none;
  border-bottom: 2px solid transparent;
  cursor: pointer;
  transition: all 200ms;
}
.meu-negocio-tab:hover {
  color: var(--foreground);
}
.meu-negocio-tab[data-selected] {
  color: var(--primary);
  border-bottom-color: var(--primary);
  font-weight: 600;
}
```

### CSS — Form Fields (reutiliza padrão do doc 13)

```css
.meu-negocio-form { max-width: 640px; }

.meu-negocio-field { margin-bottom: 20px; }

.meu-negocio-photo-preview {
  width: 200px;
  height: 140px;
  object-fit: cover;
  border-radius: 12px;
  background: var(--muted);
  margin-bottom: 8px;
}

.meu-negocio-photo-change {
  font-family: 'Manrope';
  font-size: 13px;
  color: var(--primary);
  cursor: pointer;
  background: none;
  border: none;
  font-weight: 600;
}

/* Horário grid */
.horario-grid {
  display: grid;
  grid-template-columns: 80px 1fr 30px 1fr;
  gap: 8px;
  align-items: center;
}

.horario-day-label {
  font-family: 'Manrope';
  font-size: 13px;
  font-weight: 500;
  color: var(--foreground);
}

.horario-separator {
  text-align: center;
  font-size: 13px;
  color: var(--muted-foreground);
}

.horario-input {
  padding: 8px 12px;
  background: var(--background);
  border: 1px solid var(--input);
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--foreground);
  text-align: center;
}

/* Galeria upload */
.gallery-upload-grid {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
}

.gallery-upload-item {
  width: 140px;
  height: 100px;
  border-radius: 8px;
  overflow: hidden;
  position: relative;
}

.gallery-upload-item img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.gallery-upload-remove {
  position: absolute;
  top: 4px;
  right: 4px;
  width: 24px;
  height: 24px;
  background: oklch(0 0 0 / 0.6);
  color: white;
  border: none;
  border-radius: 50%;
  font-size: 12px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.gallery-upload-add {
  width: 140px;
  height: 100px;
  border: 2px dashed var(--border);
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 4px;
  cursor: pointer;
  transition: all 200ms;
  font-family: 'Manrope';
  font-size: 12px;
  color: var(--muted-foreground);
}
.gallery-upload-add:hover {
  border-color: var(--primary);
  color: var(--primary);
}
```

---

## BANNER — SEM CEP

Quando o morador não tem CEP cadastrado:

```
┌──────────────────────────────────────────────────────────────┐
│  📍 Cadastre seu CEP para ver os benefícios da sua região    │
│     Acesse configurações e defina seu endereço.              │
│                                          [Configurar →]      │
└──────────────────────────────────────────────────────────────┘
```

```css
.vizinhanca-no-cep-banner {
  background: oklch(0.715 0.120 84.2 / 0.08);
  border: 1px solid var(--primary);
  border-radius: 12px;
  padding: 16px 20px;
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 24px;
}

.vizinhanca-no-cep-text {
  flex: 1;
}

.vizinhanca-no-cep-title {
  font-family: 'Manrope';
  font-size: 14px;
  font-weight: 600;
  color: var(--foreground);
  margin-bottom: 2px;
}

.vizinhanca-no-cep-desc {
  font-family: 'Manrope';
  font-size: 13px;
  color: var(--muted-foreground);
}

.vizinhanca-no-cep-btn {
  padding: 8px 16px;
  background: var(--primary);
  color: var(--primary-foreground);
  border: none;
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  white-space: nowrap;
}
```

---

## COMPONENTES

| Componente | Tipo | Dependência |
|---|---|---|
| `VizinhancaHub` | Page principal | — |
| `CEPBar` | Barra de CEP com editar | — |
| `CategoryGrid` | Grid de ícones-categorias | — |
| `PromoScroll` | Horizontal scroll de promoções do dia | — |
| `StoreCard` | Card de estabelecimento | — |
| `StoreProfile` | Página de detalhe | — |
| `StorePromoHighlight` | Box de promo ativa no perfil | — |
| `MeuNegocioForm` | Form de edição do lojista | TanStack Form + Zod |
| `GalleryUpload` | Upload de até 2 fotos | — |
| `HorarioGrid` | Grid de horários de funcionamento | — |
| `NoCEPBanner` | Banner para quem não tem CEP | — |

---

## ESTADOS

| Estado | Comportamento |
|---|---|
| Loading | Skeleton: CEP bar + 8 category circles + 3 promo cards shimmer |
| Empty (sem CEP) | Banner NoCEPBanner topo + conteúdo genérico/nacional |
| Empty (CEP sem resultados) | Ilustração + "Nenhum estabelecimento encontrado na sua região. Novos comércios estão sendo cadastrados!" |
| Store com promo ativa | Badge "🔥 Promo" sobreposta no card |
| Store sem promo | Card normal sem badge |
| Perfil com conteúdo limitado | Mostra até 2 fotos + 1 vídeo (conforme Matriz) |
| Lojista sem promo cadastrada | Tab Promoções com empty state + botão [Criar Primeira Promoção] |

---

## CHECKLIST

- [ ] Hub page: busca + categorias + promoções scroll + grid "perto de você"
- [ ] CEP bar com botão alterar
- [ ] Grid categorias 4 col mobile / 8 col desktop
- [ ] Scroll horizontal promoções do dia com snap
- [ ] Store cards 2 col desktop / 1 col mobile
- [ ] Perfil estabelecimento: banner + logo overlap + info + promo highlight
- [ ] Galeria limitada a 2 fotos + 1 vídeo 60s
- [ ] Painel "Meu Negócio" com tabs (Perfil/Promoções/Métricas)
- [ ] Horário de funcionamento: grid editável
- [ ] Banner NoCEP para usuários sem CEP
- [ ] Badge de promo ativa nos cards
- [ ] Dark mode tokens oklch
- [ ] Responsivo completo
