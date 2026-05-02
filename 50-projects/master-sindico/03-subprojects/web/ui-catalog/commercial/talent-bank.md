---
title: Banco de Talentos — UI Spec
type: ui-spec
tags: [ui-catalog, master-sindico, commercial, talent-bank, content]
bc: commercial
source: _chaos/21-TALENT-BANK.md (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# 21 — Banco de Talentos (Currículos em Vídeo)

> **Origem**: `_chaos/21-TALENT-BANK.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Tradução aplicada**: `N1/N2/N3` → `trial/base/plus/pro` (ADR-0032 / D-081 / D-096); **"Morador Pagante" removido (D-126)** — Banco de Talentos agora é **addon livre** disponível para morador `base` (gratuito), conforme D-099 confirmado por João 2026-04-25 (5 vínculos profissionais visíveis, M3).

> **Sprint 4 / M3** · Web & Mobile · Acesso conforme matriz canônica
> Dependências: [[../content/video-player|video-player]] · [[../content/video-upload|video-upload]] · [[../cross/private-feedback|private-feedback]] · [[../shell/app-shell|app-shell]] · [[../../../../02-architecture/design-tokens|design-tokens]]

---

## 1. Contexto e regras de negócio

### Definição
Banco de Talentos = **morador opta por exibir um vídeo-currículo de até 90 segundos** + ficha profissional, fixos no perfil, visível para empresas / síndicos com permissão.

> "Morador Pagante" foi descontinuado em D-126 (confirmado João 2026-04-25). **Morador é gratuito por padrão** (`plan_tier=base`). Banco de Talentos é addon livre opt-in. Spec original de fev/2026 mencionava "Morador Pagante" — substituído por "Morador (com Banco de Talentos ativo)" nesta absorção.

### Quem cadastra currículo
- **Morador** com Banco de Talentos ativado (M3+).
- **Sem Banco de Talentos**: morador comum sem visibilidade profissional.

### Quem visualiza (matriz)

| Perfil | Visualiza? |
|---|---|
| Morador `base` (sem Banco Talentos) | ❌ |
| Morador (com Banco Talentos) | ❌ (apenas o próprio) |
| Síndico (qualquer) | ❌ |
| Empresa `plus` | ✅ |
| Empresa `pro` | ✅ |
| Marketing | ✅ (em nome da empresa cliente) |
| Parceiro Vizinhança | ❌ (D-062 — bloqueado também aqui) |

### Trava trimestral (90 dias)
- Atualização do **vídeo** a cada 90 dias (GR-029).
- Ficha profissional (texto): editável a qualquer momento.

### Vínculos profissionais
**5 vínculos** profissionais visíveis no vídeo-currículo (D-099 confirmado 2026-04-25, M3). _A spec original mencionava "3 últimos" (D-064 obsoleto)._

### Regra de exibição no perfil
- Vídeo aparece como **item fixo** em perfil de morador com Banco Talentos ativo.
- **Não é galeria** — único vídeo institucional.
- Para outros moradores / visitantes sem permissão: vídeo NÃO aparece.

### "Demonstrar Interesse"
- Empresas com permissão clicam "Demonstrar Interesse" no perfil / card.
- Sistema dispara email / notificação para o morador com dados da empresa.
- **Não é chat. Não é Connect Me.** Sinal unidirecional.

---

## 2. Fluxo do morador — Cadastro

### 2.1 Aba "Oportunidade"

Rotas: `/oportunidade`, `/oportunidade/curriculo`, `/oportunidade/curriculo/editar`.

### 2.2 Tela "Meu Currículo"

Estrutura: Vídeo-currículo player + barra de progresso 90 dias + Ficha profissional + Métricas privadas (visualizações / interesses / empresas — visível APENAS ao dono — ver [[../cross/private-feedback|private-feedback]]).

```css
.curriculum-quarterly-bar {
  height: 8px; background: var(--muted);
  border-radius: 4px; margin: 8px 0;
}
.curriculum-quarterly-fill {
  height: 100%; background: var(--primary);
  border-radius: 4px;
  transition: width var(--duration-slower) var(--ease-out);
}
.curriculum-quarterly-fill.complete { background: var(--success); }
```

### 2.3 Estado vazio — Primeiro cadastro

Ilustração + "Cadastre seu vídeo-currículo e conecte-se a oportunidades. Grave um vídeo de até 90 segundos apresentando suas habilidades." + [Começar Cadastro] (Gold).

### 2.4 Formulário (2 steps)

**Step 1 — Ficha Profissional**:
- Nome completo, Área de atuação (31 categorias Cap. 6), Subcategorias (multi-select max 3), Anos de experiência, Estado / Cidade / Bairro, Sobre mim (50–500 chars), Email contato (pré-preenchido), Telefone (opcional).

**Step 2 — Vídeo-Currículo**:
- Drop zone (reutiliza padrão [[../content/video-upload|video-upload]]).
- Validação backend ≤ 90 s (não confiar no front).
- Formatos: MP4 / MOV / WebM, max 100 MB.
- Dicas de gravação em box `--muted` com border-left gold.

### Validação Zod

```ts
const CurriculumSchema = z.object({
  fullName: z.string().min(3).max(100),
  mainArea: z.string().min(1, "Selecione uma área"),
  subAreas: z.array(z.string()).max(3).optional(),
  yearsExperience: z.number().int().min(0).max(60),
  state: z.string().length(2),
  city: z.string().min(2).max(100),
  neighborhood: z.string().max(100).optional(),
  aboutMe: z.string().min(50).max(500),
  contactEmail: z.string().email(),
  contactPhone: z.string().optional(),
});

const VideoResumeSchema = z.object({
  file: z
    .custom<File>()
    .refine((f) => f.size <= 100 * 1024 * 1024, "Máximo 100MB")
    .refine(
      (f) => ["video/mp4", "video/quicktime", "video/webm"].includes(f.type),
      "Formato inválido",
    )
    .refine(async (f) => {
      const duration = await getVideoDuration(f);
      return duration <= 90;
    }, "Duração máxima: 90 segundos"),
});
```

---

## 3. Fluxo da Empresa — Visualização

Rotas: `/talentos`, `/talentos/:moradorId`.

### 3.1 Listagem de currículos

Grid responsivo (3 / 2 / 1 cols) com filtros: Área, Região (Estado + Cidade), Experiência (range), palavra-chave (busca em "Sobre mim" + nome).

```css
.curriculum-card {
  background: var(--card); border: 1px solid var(--border);
  border-radius: var(--radius-md); padding: 16px;
  transition: all var(--duration-moderate) var(--ease-out);
}
.curriculum-card:hover {
  box-shadow: var(--shadow-sm);
}
```

### 3.2 Detalhe do currículo

Header com avatar + nome + área + experiência + região + Player vídeo 16:9 + Sobre mim + Especialidades (pills) + **Botão "💛 Demonstrar Interesse"** (`--primary` Gold, full-width max 400 px, centralizado).

### 3.3 Modal "Demonstrar Interesse"

```
┌────────────────────────────────────────┐
│  DEMONSTRAR INTERESSE                  │
│                                        │
│  Você está demonstrando interesse no   │
│  perfil profissional de:               │
│                                        │
│  João Silva da Costa                   │
│  Elétrica · 8 anos · SP               │
│                                        │
│  O profissional receberá:              │
│  · Nome da sua empresa                 │
│  · Categoria da sua empresa            │
│  · Email de contato empresarial        │
│                                        │
│  Mensagem opcional (max 300):          │
│  [textarea]                            │
│                                        │
│  [Cancelar]      [Confirmar Interesse] │
└────────────────────────────────────────┘
```

- Cooldown: empresa não pode demonstrar interesse no mesmo profissional > 1× / mês.

---

## 4. Visualização no perfil do morador

| Quem visualiza | Vídeo | Ficha |
|---|---|---|
| O próprio morador | ✅ + métricas | ✅ + edição |
| Outro morador | ❌ | ❌ |
| Síndico | ❌ | ❌ |
| Empresa `plus`/`pro` | ✅ | ✅ |
| Marketing | ✅ | ✅ |
| Parceiro Vizinhança | ❌ | ❌ |

Layout horizontal desktop (vídeo 60 % + dados 40 %); vertical mobile.

---

## 5. Componentes

`CurriculumCard`, `CurriculumDetail`, `VideoResumePlayer`, `CurriculumForm`, `QuarterlyLockBar`, `InterestModal`, `CurriculumMetrics`, `SpecialtyPills`, `TalentFilters`, `EmptyCurriculum`.

## 6. Estados

Padrão (loading / empty / error / success / permission-denied) seguem [[../../patterns/ui-states|patterns/ui-states]].

## 7. Notas de implementação

1. **Upload**: backend valida duração ≤ 90 s server-side (não confiar apenas no frontend).
2. **Trava trimestral**: backend é a verdade única.
3. **"Demonstrar Interesse"**: NÃO é Connect Me. Disparo simples (email / notificação).
4. **Privacidade**: email / telefone do morador SÓ visível à empresa após "Demonstrar Interesse".
5. **Métricas**: visíveis APENAS ao dono ([[../cross/private-feedback|private-feedback]]).
6. **Contagem de visualização**: ≥ 5 s no detalhe ou ≥ 50 % do vídeo (evita contagens acidentais).

## Links

- [[../../../../STATE]] — D-126 Morador Pagante removido / D-099 5 vínculos / D-064 obsoleto
- [[../../../web/ui-catalog#b6-banco-talentos-tel1-tel11]] — telas TEL1-TEL11 (catálogo macro)
- [[../content/video-upload|content/video-upload]] — upload base
- [[../cross/private-feedback|cross/private-feedback]] — métricas privadas
- [[../../../../00-product/business-model#41-matriz-consolidada]] — matriz "Banco Talentos buscar currículos"
- [[../../patterns/ui-states|patterns/ui-states]]
- [[../../../../02-architecture/design-tokens|design-tokens]]
