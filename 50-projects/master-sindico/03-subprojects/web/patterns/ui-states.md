---
title: UI States — Estados, Feedback & Micro-interações
type: pattern
tags: [patterns, master-sindico, ui-states, feedback, accessibility, cross]
bc: cross
source: _chaos/24-STATES-FEEDBACK.md (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# 24 — Estados de UI, Feedback & Micro-interações

> **Origem**: `_chaos/24-STATES-FEEDBACK.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Tradução aplicada**: `N1/N2/N3` → `trial/base/plus/pro` (ADR-0032 / D-081 / D-096); "Morador Pagante" removido (D-126).
> **Tipo**: PATTERN cross-app (não ui-catalog) — aplicado a TODAS as telas do `web/`.

> Sprint: **Transversal** (aplica a todas as sprints).
> Prioridade: **Crítica** — todo componente DEVE ter seus estados definidos.
> Dependências: [[../../../02-architecture/design-tokens|design-tokens]] · [[../ui-catalog/shell/app-shell|shell/app-shell]].
> Referências visuais: YouTube (skeleton shimmer), Instagram (pull-to-refresh), Meta (error boundaries), Stripe (toast feedback).

---

## 1. Filosofia de estados

### 1.1 Princípio fundamental

Nenhuma tela do Master Síndico pode existir em estado "indefinido". Todo componente visual deve ter resposta clara para **6 estados** possíveis:

| Estado | Quando ocorre |
|---|---|
| **Loading** | Dados sendo buscados |
| **Skeleton** | Carregamento inicial da tela |
| **Success** | Dados carregados com sucesso |
| **Empty** | Sem dados para exibir |
| **Error** | Falha na operação |
| **Restricted** | Sem permissão (ABAC / Paywall) |

### 1.2 Regra de Ouro

> **"O usuário nunca deve olhar para uma tela em branco."**
> Se algo está carregando, mostre skeleton. Se deu erro, mostre o que aconteceu e o que fazer. Se não tem dados, mostre o caminho. Se não tem permissão, mostre por quê.

### 1.3 Hierarquia de feedback

| Janela | Componentes |
|---|---|
| **Imediato** (< 100 ms) | Hover, focus rings, active states, toggle switches |
| **Rápido** (100 ms – 1 s) | Toast, Like / Unlike, Form validation inline, Dropdown open/close |
| **Demorado** (> 1 s) | Skeleton, Progress bars, Spinner contextual, Upload progress |

---

## 2. Skeleton Loading

### 2.1 Conceito

Skeletons são placeholders visuais que replicam a estrutura exata do conteúdo que será carregado. Comunicam ao usuário o que esperar, reduzem percepção de espera e evitam layout shift (CLS).

### 2.2 SkeletonPrimitive

```ts
type SkeletonProps = {
  variant?: "text" | "circle" | "rect" | "card";
  width?: string;     // CSS
  height?: string;    // CSS
  borderRadius?: string;
  animated?: boolean; // default true
};
```

```css
.skeleton {
  background: var(--muted);
  background-image: linear-gradient(
    90deg,
    var(--muted) 25%,
    rgba(255, 255, 255, 0.08) 50%,
    var(--muted) 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s ease-in-out infinite;
  border-radius: var(--radius-md);
}
@keyframes shimmer {
  0%   { background-position: -200% 0; }
  100% { background-position:  200% 0; }
}
@media (prefers-reduced-motion: reduce) {
  .skeleton { animation: none; }
}
```

### 2.3 Variantes por contexto

| Componente | Quantidade típica |
|---|---|
| `SkeletonVideoCard` | 8 cards na Home (4×2 desktop / 2×4 tablet) |
| `SkeletonProfileHeader` | 1 (banner + avatar overlap + stats) |
| `SkeletonForumPost` | 5 posts |
| `SkeletonAssemblyCard` | 3 cards |
| `SkeletonCourseCard` | 4 cards |
| `SkeletonDashboard` | 4 KPI + chart + 5 rows table |
| `SkeletonNotificationItem` | 6 items |

### 2.4 Regras de implementação

1. **Nunca usar spinner sozinho em tela cheia.** Spinners só em botões / ações pontuais.
2. **Skeleton reflete layout real**: avatar + 2 linhas + thumbnail = mesmas shapes.
3. **Shimmer LTR**: `linear-gradient(90deg, ...)`.
4. **Duração 1.5 s**: < 1 s causa ansiedade; > 2.5 s parece travado.
5. **Quantidade**: apenas o que cabe na viewport.
6. **Border-radius**: todos `var(--radius-md)`.
7. **Transição → conteúdo real**: fade `opacity 0 → 1` em 200 ms via `<Show when={!isLoading()} fallback={<Skeleton />}>`.

---

## 3. Empty States

### 3.1 Anatomia

```
┌──────────────────────────────────────────────────┐
│              [ILUSTRAÇÃO 80–120 px]              │
│              monocromática --muted-foreground    │
│                                                   │
│           Título do Estado Vazio                 │ ← Michroma 16-18 px
│        Descrição explicativa curta que           │ ← Manrope 13-14 px muted
│        orienta o próximo passo do usuário        │   max-width 320 px center
│                                                   │
│              [Ação Primária]                     │ ← Botão gold (se houver)
│              Link secundário →                   │ ← (opcional)
└──────────────────────────────────────────────────┘
```

Container: `flex col`, `items-center`, `justify-center`, `min-height: 400px`, `padding: 48px 24px`.

### 3.2 Catálogo de empty states

| Contexto | Ícone | Título | Ação |
|---|---|---|---|
| Home sem conteúdo | Play circle riscado | "Nenhum conteúdo disponível" | — |
| Busca sem resultados | Lupa com X | "Nenhum resultado encontrado" | [Limpar filtros] |
| Perfil sem vídeos (dono) | Câmera + | "Você ainda não publicou vídeos" | [Publicar meu primeiro vídeo] |
| Perfil sem vídeos (visitante) | Play circle vazio | "Nenhum vídeo publicado" | — |
| Assembleias (Síndico) | Calendário + | "Nenhuma assembleia criada" | [Criar assembleia] |
| Assembleias (Morador) | Calendário vazio | "Sem assembleias agendadas" | — |
| Votação | Cédula vazia | "Nenhuma votação aberta" | — |
| Fórum sem posts | Balão vazio | "Nenhuma discussão por aqui" | [Criar novo tópico] |
| Cursos sem inscritos | Livro vazio | "Você ainda não iniciou nenhum curso" | [Explorar cursos] |
| Notificações | Sino com check | "Tudo em dia!" | — |
| Connect Me sem histórico | Envelope vazio | "Nenhum formulário enviado" | [Buscar profissionais] |
| Vizinhança sem promo no CEP | Pin com ? | "Nenhuma promoção na sua região" | [Alterar CEP] |
| Vizinhança sem CEP | Pin com ! | "Informe seu CEP" | [Informar CEP] |
| Currículos (Empresa) | Pessoa + | "Nenhum currículo disponível" | — |
| Métricas privadas | Gráfico vazio | "Ainda sem interações" | — |
| Transparência (Síndico) | Timeline pontilhada | "Sua linha do tempo está em branco" | [Adicionar primeiro registro] |
| Admin sem denúncias | Escudo + check | "Nenhuma denúncia pendente" | — |
| Lives | Câmera com linha | "Nenhuma live agendada" | — |
| Podcast | Headphones vazio | "Nenhum episódio disponível" | — |

### 3.3 Regras de implementação

1. **Ilustrações**: SVG monocromáticos Iconify; cor `var(--muted-foreground)` com 60 % opacity; 80 px mobile / 120 px desktop.
2. **Título**: Michroma 16 px mobile / 18 px desktop; `var(--foreground)`; max 40 chars.
3. **Descrição**: Manrope 13 px mobile / 14 px desktop; `var(--muted-foreground)`; line-height 1.5; max 2 linhas (80 chars); text-align center; max-width 320 px.
4. **Botão**: só se existe ação prática; gold primary (principal) ou outline (secundária).
5. **ABAC**: botão só renderiza se permissão. Ex: "Criar assembleia" só para Síndico `pro`.
6. **Tom profissional**: nunca "Ops!", "Oopsie!", emojis tristes.

---

## 4. Error States

### 4.1 Tipos de erro

| Tipo | HTTP | Tratamento |
|---|---|---|
| Rede indisponível | 0 | Banner + Retry |
| Não autorizado | 401 | Redirect Login |
| Sem permissão | 403 | Paywall / Restricted |
| Não encontrado | 404 | Empty state custom |
| Validação | 422 | Inline nos campos |
| Rate limit | 429 | Toast + cooldown |
| Erro servidor | 500 | Error boundary |
| Timeout | 408 | Retry com backoff |
| Serviço externo | 502 / 503 | Toast informativo |

### 4.2 Error Boundary (falha geral)

Wrapper SolidJS captura erros não tratados e exibe fallback amigável:

```
                    ⚠ (var(--destructive))
        Algo não saiu como esperado
   Ocorreu um erro ao carregar esta página.
        Nossa equipe foi notificada.
   [Tentar de novo] [Voltar ao início]
   Detalhes técnicos ▾  (collapsible)
```

```tsx
import { ErrorBoundary } from "solid-js";
<ErrorBoundary fallback={(err, reset) => <ErrorPage error={err} reset={reset} />}>
  {children}
</ErrorBoundary>
```

### 4.3 Network Error Banner

Sticky no topo (abaixo da topbar) quando `!navigator.onLine`:

```css
.network-banner {
  position: sticky; top: 64px; left: 0; right: 0;
  height: 40px; padding: 0 16px;
  background: var(--destructive);
  color: var(--destructive-foreground);
  display: flex; align-items: center; gap: 8px;
  font-family: 'Manrope'; font-size: 13px;
  z-index: var(--z-banner);
}
.network-banner.online {
  background: var(--success);
  /* auto-dismiss 3s */
}
```

Detecção: `navigator.onLine` + listeners `online`/`offline` + ping `/api/health` a cada 30 s quando offline.

### 4.4 Inline form errors

```css
.input-error {
  border: 2px solid var(--destructive);
  box-shadow: 0 0 0 3px oklch(0.568 0.200 26.4 / 0.15);
}
.field-error-message {
  display: flex; align-items: center; gap: 4px;
  font-family: 'Manrope'; font-size: 12px;
  color: var(--destructive);
  margin-top: 4px;
}
```

Mensagens Zod → amigáveis:
- `z.string().min(1)` → "Este campo é obrigatório"
- `z.string().email()` → "Informe um e-mail válido"
- `z.string().min(6)` → "Mínimo de 6 caracteres"
- `z.string().max(N)` → "Máximo de N caracteres"
- `z.string().regex(CEP)` → "CEP inválido. Formato: 00000-000"
- `z.string().regex(CNPJ)` → "CNPJ inválido"

**Timing**: validação inline `onBlur` + tempo real ao corrigir. Não validar `onChange` por tecla.

### 4.5 404 — Página não encontrada

Catch-all no TanStack Router. Layout: "404" Michroma 72 px muted + "Página não encontrada" + descrição + [Voltar ao início].

### 4.6 Timeout & Retry

```
0–10 s:  Skeleton normal
10–20 s: Skeleton + "Isso está demorando mais que o normal..."
> 20 s:  Erro: "Não foi possível carregar os dados" + [Tentar novamente]
```

TanStack Query:
```ts
{
  retry: 3,
  retryDelay: (attempt) => Math.min(1000 * 2 ** attempt, 8000),
  staleTime: 5 * 60 * 1000,
}
```

---

## 5. Estados Restritos (Paywall & ABAC)

### 5.1 Estratégias

| Estratégia | Quando usar |
|---|---|
| **Não renderizar** (ABAC silent) | Menu / botão que o role / tier não tem |
| **Paywall overlay** | Conteúdo parcialmente visível (ex: vídeo 25 %) |
| **Gate page** | Rota inteira bloqueada |
| **Upgrade prompt** | Feature existe mas plano limita (ex: cota Connect Me esgotada) |

### 5.2 Não renderizar (ABAC silent)

Elementos sem permissão **não existem** na UI. Sem placeholder, sem cadeado.

```tsx
<PermissionGate requires={["assembly:create"]} planTiers={["plus", "pro"]}>
  <CreateAssemblyButton />
</PermissionGate>
```

### 5.3 Paywall overlay (vídeo 25 %)

Para vídeos de Empresa quando morador `base` assiste — corta em 25 %, overlay navy 95 % com blur 4 px:

```css
.paywall-overlay {
  position: absolute; inset: 0;
  background: linear-gradient(
    to top,
    oklch(0.183 0.031 263.4 / 0.95) 0%,
    oklch(0.183 0.031 263.4 / 0.7) 70%,
    transparent 100%
  );
  backdrop-filter: blur(4px);
  display: flex; flex-direction: column;
  align-items: center; justify-content: center;
  padding: 32px;
}
```

Backend envia `isPreview: true` + `previewEndTime`. Player para no timestamp e renderiza overlay; seek bloqueado além do ponto.

### 5.4 Gate page (rota bloqueada)

```
🔒
Funcionalidade não disponível no seu plano atual
O acesso a {nome} requer o plano {tier_minimo}.
[Conhecer planos]   ← Voltar
```

Middleware de rota (TanStack `beforeLoad`):

```ts
beforeLoad: ({ context }) => {
  if (!hasPermission(context.auth, requiredPermission)) {
    throw new GatePageError({ feature, minTier });
  }
}
```

### 5.5 Trava temporal 90 dias (upload trimestral)

Card informativo + countdown boxes (dias / horas / min) Michroma 28 px + botão "Substituir vídeo" disabled (opacity 0.4, cursor not-allowed, tooltip "Disponível em DD/MM/YYYY").

```css
.countdown-box {
  width: 72px; height: 64px;
  background: var(--surface-variant);
  border-radius: var(--radius-md);
  display: flex; flex-direction: column;
  align-items: center; justify-content: center;
}
.countdown-number {
  font-family: 'Michroma'; font-size: 28px; color: var(--foreground);
}
.countdown-label {
  font-family: 'Manrope'; font-size: 11px;
  color: var(--muted-foreground); text-transform: uppercase;
}
```

### 5.6 Trava cooldown 4 h (cupons)

Card warning com countdown "Xh YYmin restantes" Michroma 20 px + progress bar + [Ver meu cupom ativo].

### 5.7 Cota Connect Me esgotada

```
Cotas de contato esgotadas
Você já utilizou suas {X} cotas anuais de Connect Me para síndicos.
Visualização: bolhas preenchidas (Base: 2/2, plus: 4/4) (D-126: morador base = 0/ano)
Suas cotas serão renovadas em janeiro de {ano+1}.
[Fazer upgrade] (se base)
```

> Após D-126: morador `base` tem **0 Connect Me** anual (hard paywall). Mensagem ajusta: "Connect Me não está disponível no seu plano."

---

## 6. Toast Notifications (solid-sonner)

### 6.1 Posicionamento

| Breakpoint | Posição | Margem |
|---|---|---|
| Desktop ≥ 1024 px | bottom-right | 24 px |
| Tablet 768-1023 | bottom-center | 16 px |
| Mobile < 768 px | top-center | 16 px (evita conflito com bottom nav) |

Stack: máximo 3 simultâneos.

### 6.2 Variantes

```css
.toast {
  width: 360px; padding: 14px 16px;
  background: var(--card); border: 1px solid var(--border);
  border-radius: var(--radius-md);
  box-shadow: var(--shadow-lg);
  font-family: 'Manrope';
}
.toast.success     { border-left: 4px solid var(--success); }
.toast.error       { border-left: 4px solid var(--destructive); }
.toast.warning     { border-left: 4px solid var(--warning); }
.toast.info        { border-left: 4px solid var(--ring); }

.toast-title       { font-size: 14px; font-weight: 600; color: var(--foreground); }
.toast-description { font-size: 13px; color: var(--muted-foreground); }
.toast-action      { font-size: 13px; font-weight: 600; color: var(--primary); cursor: pointer; }
```

### 6.3 Duração

| Tipo | Duração |
|---|---|
| Success | 3 s |
| Info | 4 s |
| Warning | 5 s |
| Error | 6 s (ou persistente se ação) |
| Upload progress | persistente |

Dismiss: swipe (mobile) ou click no ×.

### 6.4 Catálogo de mensagens

**Success**: "Perfil atualizado", "Vídeo publicado", "Assembleia criada", "Voto registrado", "Avaliação enviada", "Cupom gerado", "Connect Me enviado", "Curso iniciado", "Certificado disponível", "Post publicado", "Senha alterada", "Foto atualizada", "Promoção cadastrada", "Denúncia registrada", "Comunicado agendado".

**Error**: "Não foi possível salvar...", "Erro ao publicar vídeo. Verifique o formato.", "Falha ao enviar formulário. Verifique sua conexão.", "Erro ao registrar voto.", "Não foi possível gerar o cupom.", "Formato não suportado.", "Arquivo excede o tamanho máximo".

**Warning**: "Você atingiu o limite mensal de vídeos ({N}/{max})", "Cupom ativo detectado. Aguarde {tempo}.", "Cotas Connect Me esgotadas para este ano.", "Upload bloqueado. Próxima atualização em {data}.", "Janela de votação encerrada.", "Limite de caracteres atingido".

**Info**: "Live ao vivo agora!", "Novo episódio do podcast.", "Sua avaliação é importante.", "Novo comunicado do Master Síndico".

---

## 7. Loading States Pontuais

### 7.1 Button Loading

```tsx
<button disabled={isLoading()} aria-busy={isLoading()}>
  <Show when={isLoading()} fallback="Publicar">
    <Spinner size="sm" /> Publicando...
  </Show>
</button>
```

```css
button[aria-busy="true"] {
  opacity: 0.7; cursor: not-allowed; min-width: 120px;
}
```

### 7.2 Upload Progress

Progress bar 6 px gold durante upload; verde 100 % concluído; vermelho em erro.

```css
.upload-progress-bar {
  height: 6px; background: var(--muted);
  border-radius: 3px; overflow: hidden;
}
.upload-progress-fill {
  height: 100%; background: var(--primary);
  transition: width var(--duration-moderate);
}
.upload-progress-fill.complete { background: var(--success); }
.upload-progress-fill.error    { background: var(--destructive); }
```

### 7.3 Pull to Refresh (mobile)

Threshold 80 px; max pull 120 px (rubber band); spinner gold 24 px; haptic `navigator.vibrate(10)`.

### 7.4 Infinite Scroll

`IntersectionObserver` no sentinel; threshold 0.5; root margin 200 px (pre-fetch). TanStack `useInfiniteQuery`. Estado `!hasNextPage`: "Você chegou ao fim".

---

## 8. Micro-interações

### 8.1 Like animation

```css
.like-button.liked .icon {
  color: var(--primary);
  animation: heartBounce var(--duration-slow) ease-out;
}
@keyframes heartBounce {
  0%   { transform: scale(1); }
  30%  { transform: scale(1.3); }
  60%  { transform: scale(0.9); }
  100% { transform: scale(1); }
}
```

> Lembrete: número de likes NÃO é exibido para visitantes (R-PRIV-MX). Apenas o toggle visual aparece para todos. O dono vê contador no painel privado.

### 8.2 Vote confirmation

1. Botão selecionado: `background → var(--primary)` + scale `1 → 1.05 → 1`.
2. Checkmark fade-in 200 ms.
3. Botões não selecionados: `opacity → 0.4`, `pointer-events: none`.
4. Toast success: "Voto registrado".
5. Locked (sem mudança).

### 8.3 Form field focus

```css
.input { border: 1px solid var(--input); }
.input:focus {
  border: 2px solid var(--ring);
  box-shadow: 0 0 0 3px oklch(0.715 0.120 84.2 / 0.15);
  transition: border-color var(--duration-normal), box-shadow var(--duration-normal);
}
.input.error  { border: 2px solid var(--destructive); }
.input.success { border: 2px solid var(--success); }
```

### 8.4 Card hover (desktop)

```css
.card { transition: transform var(--duration-moderate), box-shadow var(--duration-moderate); }
.card:hover {
  transform: translateY(-2px);
  box-shadow: var(--shadow-lg);
}
.card:active { transform: translateY(0); transition: transform var(--duration-quick); }
```

### 8.5 Toggle switch / Tab indicator / Accordion / Page transitions / Notification badge pulse

Detalhamento abaixo (mantém durações canônicas: 150 ms rápido, 200 ms padrão, 300 ms lento).

```css
@keyframes notificationPulse {
  0%   { box-shadow: 0 0 0 0 oklch(0.568 0.200 26.4 / 0.6); }
  70%  { box-shadow: 0 0 0 6px oklch(0.568 0.200 26.4 / 0); }
  100% { box-shadow: 0 0 0 0 oklch(0.568 0.200 26.4 / 0); }
}
.notification-dot { animation: notificationPulse 2s infinite; }
```

---

## 9. Estados Especiais

### 9.1 Maintenance Mode

Tela full sem nav / sidebar / footer. Centralizado: logo Master Síndico + 🔧 (64 px muted) + "Estamos em manutenção" (Michroma 24 px) + descrição + "Previsão de retorno: DD/MM/YYYY HH:MM".

### 9.2 Session Expired

Modal sobreposto não-fechável (sem ×, sem click fora) quando JWT expira:
- Backdrop: `oklch(0 0 0 / 0.6)` + `backdrop-filter: blur(8px)`.
- Modal 400 px (desktop) / full-width 32 px margin (mobile).
- "Sessão expirada" + descrição + [Fazer login novamente] (gold).

### 9.3 Force Update (mobile)

"Atualização necessária" — não dismissível, link para loja.

---

## 10. Acessibilidade

### 10.1 Screen readers

- Skeleton: `aria-busy="true"` no container, `aria-label="Carregando conteúdo"`.
- Empty state: `role="status"`, `aria-label` descritivo.
- Error: `role="alert"`, `aria-live="assertive"` para críticos.
- Toast: `role="status"` + `aria-live="polite"` (success/info) ou `aria-live="assertive"` (error).
- Loading button: `aria-disabled="true"`, `aria-busy="true"`.

### 10.2 Focus management

- Error boundary: foco no título do erro.
- Modal session expired: focus trap.
- Toast: NÃO rouba foco.
- Empty state com ação: foco no botão.

### 10.3 Motion preferences

```css
@media (prefers-reduced-motion: reduce) {
  .skeleton { animation: none; }
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## 11. Implementação técnica (SolidJS)

### 11.1 Componentes base

```
packages/ui/src/states/
├── skeleton.tsx           // SkeletonPrimitive + variants
├── empty-state.tsx        // EmptyState genérico
├── error-boundary.tsx     // Wrapper com fallback
├── error-page.tsx         // 404 / 500
├── gate-page.tsx          // Paywall / restrição plano
├── network-banner.tsx
├── loading-button.tsx
├── upload-progress.tsx
├── countdown.tsx          // 90 dias / 4 h
├── permission-gate.tsx    // ABAC conditional render
└── paywall-overlay.tsx
```

### 11.2 Padrão TanStack Query + SolidJS

```tsx
const query = createQuery(() => ({
  queryKey: ["resource", id()],
  queryFn: () => fetchResource(id()),
}));

return (
  <Switch>
    <Match when={query.isLoading}>
      <SkeletonVariant />
    </Match>
    <Match when={query.isError}>
      <ErrorState error={query.error} retry={() => query.refetch()} />
    </Match>
    <Match when={query.data && query.data.length === 0}>
      <EmptyState variant="no-videos" />
    </Match>
    <Match when={query.data}>
      {(data) => <ContentList items={data()} />}
    </Match>
  </Switch>
);
```

### 11.3 Toaster config (solid-sonner)

```tsx
<Toaster
  position="bottom-right"
  toastOptions={{
    classes: {
      success: "border-l-4 border-l-success",
      error: "border-l-4 border-l-destructive",
      warning: "border-l-4 border-l-warning",
      info: "border-l-4 border-l-ring",
    },
    duration: { success: 3000, error: 6000, warning: 5000, info: 4000 },
  }}
/>
```

---

## 12. Checklist de implementação

Para cada tela:

- [ ] Skeleton loading definido (shimmer, shapes corretas)
- [ ] Empty state com texto e ação contextual
- [ ] Error state com retry
- [ ] Restricted state (ABAC / paywall) tratado
- [ ] Toast de sucesso para ações mutativas
- [ ] Toast de erro para falhas
- [ ] Button loading para ações assíncronas
- [ ] Inline validation nos formulários
- [ ] Focus management para a11y
- [ ] `aria-*` attributes em estados dinâmicos
- [ ] Responsivo mobile testado em < 768 px
- [ ] Dark mode com tokens OKLCH
- [ ] `prefers-reduced-motion` respeitado

---

## 13. Notas finais

1. **Skeleton antes de tudo**: ao criar tela nova, skeleton é o PRIMEIRO componente.
2. **Empty states como oportunidade**: cada um é chance de guiar o usuário.
3. **Erros honestos**: claros, sem jargão técnico ("Não foi possível carregar" > "Error 500").
4. **Consistência de timing**: 150 ms rápido / 200 ms padrão / 300 ms lento (não inventar).
5. **Toast é confirmação, não informação primária**: para info crítica use modais / banners inline.
6. **ABAC silencioso**: elementos sem permissão NÃO existem na UI — exceto paywall explícito (vídeo 25 %, gate page).
7. **Performance**: shimmer usa `background-position` (GPU-accelerated). Nunca animar `width/height/margin`.
8. **Testes visuais**: cada estado deve ser testável isoladamente (props que simulam o estado).

## Links

- [[../../../02-architecture/design-tokens|design-tokens]] — tokens consumidos
- [[../ui-catalog/shell/app-shell|shell/app-shell]]
- [[../ui-catalog/_moc|ui-catalog/_moc]]
- [[../patterns|patterns.md]] — patterns gerais (SolidJS primitives, TanStack)
- [[../README|web/README]]
