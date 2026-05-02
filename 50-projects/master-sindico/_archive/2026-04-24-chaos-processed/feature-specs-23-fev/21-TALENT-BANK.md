# 21 — BANCO DE TALENTOS (CURRÍCULOS EM VÍDEO)

> **Sprint 4** · Web & Mobile · Acesso conforme Matriz Funcional (SOW 4.7)
> Dependências: 00-DESIGN-TOKENS · 02-APP-SHELL · 05-VIDEO-PLAYER · 06-VIDEO-UPLOAD · 16-PRIVATE-FEEDBACK

---

## 21.1 CONTEXTO E REGRAS DE NEGÓCIO

### 21.1.1 Definição
O Banco de Talentos é a funcionalidade que permite ao **Morador Pagante** cadastrar um vídeo-currículo de até 90 segundos, fixo no perfil, para ser visualizado por empresas e síndicos com planos que permitam acesso a currículos.

Conforme SOW 2.6: "O vídeo-currículo do Morador Pagante é um vídeo fixo exibido no perfil, sujeito à trava trimestral de 90 dias. Não é uma galeria ou lista de vídeos, mas um único vídeo institucional de apresentação profissional."

### 21.1.2 Quem Cadastra Currículo
- **Morador Pagante**: Único perfil que pode criar vídeo-currículo e ficha profissional.
- **Morador Base**: Não tem acesso a esta funcionalidade.

### 21.1.3 Quem Visualiza Currículos (Matriz Funcional)
| Perfil | Pode visualizar currículos? |
|---|---|
| Base | ❌ Não |
| Morador Pg | ❌ Não (apenas cadastra o próprio) |
| Síndico N1 | ❌ Não |
| Síndico N2 | ❌ Não |
| Síndico N3 | ❌ Não |
| Empresa Plus | ✅ Sim |
| Empresa Pro | ✅ Sim |
| Marketing | ✅ Sim |
| Vizinhança | ✅ Sim |

### 21.1.4 Trava Trimestral (90 Dias)
- O morador só pode **atualizar** o vídeo-currículo a cada 90 dias (hard-coded).
- Lógica: mesma regra do vídeo institucional de empresas (doc 06-VIDEO-UPLOAD, regra temporal).
- Se tentar atualizar antes dos 90 dias → bloqueio com mensagem informando dias restantes.
- A ficha cadastral profissional (texto) pode ser editada a qualquer momento — a trava aplica-se apenas ao vídeo.

### 21.1.5 Regra de Exibição no Perfil
- O vídeo-currículo aparece como **item fixo** no perfil do morador (doc 04-PROFILE-MANAGEMENT).
- **Não é galeria**. É um único vídeo.
- Visível para empresas que acessam o perfil do morador (se a Matriz Funcional permitir).
- Para outros moradores ou visitantes que acessem o perfil: o vídeo-currículo **não aparece** (privacidade).

### 21.1.6 Funcionalidade "Demonstrar Interesse"
Conforme SOW 4.7: "Botão 'Demonstrar Interesse' (dispara notificação ou email para o morador — sujeito às regras de notificação)."
- Empresas com acesso a currículos podem clicar em "Demonstrar Interesse" no perfil/card do morador.
- O sistema dispara um email/notificação para o morador com os dados da empresa interessada.
- **Não é chat. Não é Connect Me.** É um sinal unidirecional.
- Sujeito às regras de notificação do SOW (não é notificação social, mas notificação de oportunidade).

---

## 21.2 FLUXO DO MORADOR: CADASTRO DE CURRÍCULO

### 21.2.1 Aba "Oportunidade" (Morador Pagante)

A aba "Oportunidade" é acessível via sidebar/bottom nav do morador pagante.

```
ROTA: /oportunidade
ROTA: /oportunidade/curriculo
ROTA: /oportunidade/curriculo/editar
```

### 21.2.2 Tela Principal — "Meu Currículo"

```
┌───────────────────────────────────────────────────────────┐
│  OPORTUNIDADE                                              │
│  Michroma 18px --foreground                                │
│  "Conecte-se a oportunidades no seu entorno"               │
│  Manrope 14px --muted-foreground                           │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │                                                     │  │
│  │            SEU VÍDEO-CURRÍCULO                      │  │
│  │                                                     │  │
│  │  ┌───────────────────────────────────────────┐      │  │
│  │  │                                           │      │  │
│  │  │         Player do Vídeo                   │      │  │
│  │  │         (16:9 ratio)                      │      │  │
│  │  │         max 90 segundos                   │      │  │
│  │  │                                           │      │  │
│  │  │         [▶]                               │      │  │
│  │  │                                           │      │  │
│  │  └───────────────────────────────────────────┘      │  │
│  │                                                     │  │
│  │  Próxima atualização disponível em: 47 dias         │  │
│  │  ████████████████░░░░░░░░░░░  48%                   │  │
│  │  (barra de progresso dos 90 dias)                   │  │
│  │                                                     │  │
│  │  [Atualizar Vídeo] (disabled se < 90 dias)          │  │
│  │                                                     │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │  FICHA PROFISSIONAL                                 │  │
│  │                                                     │  │
│  │  Nome: João Silva da Costa                          │  │
│  │  Área de atuação: Elétrica e Manutenção             │  │
│  │  Experiência: 8 anos                                │  │
│  │  Região: Zona Sul - São Paulo/SP                    │  │
│  │  Sobre mim: "Eletricista com certificação NR-10..." │  │
│  │                                                     │  │
│  │  [Editar Ficha]                                     │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │  📊 MÉTRICAS DO SEU CURRÍCULO (privadas)            │  │
│  │                                                     │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐          │  │
│  │  │ 👁️ 34    │  │ 💛 12    │  │ 🏢 5     │          │  │
│  │  │ Visual.  │  │ Interess.│  │ Empresas │          │  │
│  │  └──────────┘  └──────────┘  └──────────┘          │  │
│  │  (visível apenas para o dono — doc 16)              │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
└───────────────────────────────────────────────────────────┘
```

**Player do Vídeo-Currículo:**
- Proporção 16:9, largura máxima 640px
- Se não há vídeo cadastrado: placeholder com ícone de câmera e "Grave seu vídeo-currículo"
- Player reutiliza o componente do doc 05-VIDEO-PLAYER
- Sem controles de velocidade (vídeo é curto, 90s máx.)
- Badge "90s" no canto superior direito do player

**Barra de Progresso Trimestral:**
- Barra horizontal mostrando quantos dias se passaram desde o último upload
- Preenchimento: `--primary` (Gold) proporcional ao tempo decorrido
- Background: `--muted`
- Texto acima: "Próxima atualização disponível em: X dias"
- Quando zerado (90 dias completos): barra 100% verde `--success`, texto "Atualização disponível!"
- Border-radius: 4px, height: 8px

**Botão "Atualizar Vídeo":**
- Se < 90 dias: disabled, `--muted` background, cursor not-allowed
- Se ≥ 90 dias: `--primary` (Gold), ativo
- Se nunca enviou: botão muda para "Enviar Vídeo-Currículo" com ícone de upload

**Métricas Privadas:**
- 3 mini-cards conforme doc 16-PRIVATE-FEEDBACK
- Visualizações: quantas vezes o vídeo foi visto
- Interessados: quantas empresas clicaram "Demonstrar Interesse"
- Empresas: quantas empresas únicas visualizaram o perfil
- Valores em Michroma 20px `--primary` (Gold)
- Labels em Manrope 11px `--muted-foreground`

### 21.2.3 Estado Vazio — Primeiro Cadastro

```
┌───────────────────────────────────────────────────────────┐
│  OPORTUNIDADE                                              │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │                                                     │  │
│  │            🎬                                        │  │
│  │                                                     │  │
│  │    Cadastre seu vídeo-currículo                      │  │
│  │    e conecte-se a oportunidades                      │  │
│  │                                                     │  │
│  │    Grave um vídeo de até 90 segundos                 │  │
│  │    apresentando suas habilidades                     │  │
│  │    profissionais.                                    │  │
│  │                                                     │  │
│  │    Empresas e síndicos poderão                       │  │
│  │    encontrar seu perfil na busca.                    │  │
│  │                                                     │  │
│  │    [Começar Cadastro]                               │  │
│  │    (--primary Gold)                                  │  │
│  │                                                     │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
└───────────────────────────────────────────────────────────┘
```

- Ícone: emoji 🎬 em 48px ou ícone Iconify `lucide:video` 48px `--muted-foreground`
- Título: Michroma 16px `--foreground`
- Descrição: Manrope 14px `--muted-foreground`, text-align center
- Botão CTA: `--primary` (Gold), full-width max 300px

### 21.2.4 Formulário de Cadastro/Edição

```
┌───────────────────────────────────────────────────────────┐
│  CADASTRO DE CURRÍCULO                                     │
│  Michroma 18px --foreground                                │
│                                                             │
│  PASSO 1 — Ficha Profissional                              │
│                                                             │
│  Nome completo *                                           │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ João Silva da Costa                                 │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  Área de atuação *                                         │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ [▾ Selecionar área]                                 │  │
│  │  · Elétrica, Energia e SPDA                         │  │
│  │  · Hidráulica e Saneamento                          │  │
│  │  · Construção, Reformas e Manutenção                │  │
│  │  · (baseado nas 31 categorias do Cap. 6)            │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  Subcategoria / Especialidade                              │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ [▾ Selecionar subcategorias] (multi-select, max 3)  │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  Anos de experiência *                                     │
│  ┌──────────┐                                              │
│  │ 8        │                                              │
│  └──────────┘                                              │
│                                                             │
│  Região de atuação *                                       │
│  ┌────────────────┐  ┌────────────────┐                    │
│  │ Estado [▾ SP]  │  │ Cidade [▾ SP]  │                    │
│  └────────────────┘  └────────────────┘                    │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ Bairro/Região (texto livre)                         │  │
│  │ "Zona Sul"                                          │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  Sobre mim *                                               │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ Apresente-se brevemente. Descreva suas              │  │
│  │ habilidades, certificações e diferenciais.          │  │
│  │                                                     │  │
│  │ (textarea, min 50 chars, max 500 chars)             │  │
│  └─────────────────────────────────────────────────────┘  │
│  234/500 caracteres                                        │
│                                                             │
│  Contato preferencial (visível apenas para empresas        │
│  que demonstrarem interesse)                               │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ Email: joao@email.com (pré-preenchido da conta)     │  │
│  └─────────────────────────────────────────────────────┘  │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ Telefone: (11) 99999-0000 (opcional)                │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  [Próximo: Enviar Vídeo →]                                 │
│                                                             │
│  PASSO 2 — Vídeo-Currículo                                 │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │                                                     │  │
│  │  📤 Arraste seu vídeo aqui ou clique para           │  │
│  │     selecionar                                      │  │
│  │                                                     │  │
│  │  Formatos: MP4, MOV, WebM                           │  │
│  │  Duração máxima: 90 segundos                        │  │
│  │  Tamanho máximo: 100MB                              │  │
│  │  Resolução recomendada: 720p ou 1080p               │  │
│  │                                                     │  │
│  │  💡 Dicas:                                          │  │
│  │  · Fale diretamente para a câmera                   │  │
│  │  · Boa iluminação faz diferença                     │  │
│  │  · Apresente-se, cite experiência e habilidades     │  │
│  │  · Seja objetivo — você tem 90 segundos             │  │
│  │                                                     │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  (Após upload, preview do vídeo aparece aqui)              │
│  ┌─────────────────────────────────────────────────────┐  │
│  │                                                     │  │
│  │  [Player Preview do vídeo enviado]                  │  │
│  │  Duração: 1:23 ✅                                   │  │
│  │                                                     │  │
│  │  [Trocar vídeo]                                     │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  [← Voltar]                 [Publicar Currículo]           │
│                                                             │
└───────────────────────────────────────────────────────────┘
```

**Campos do Formulário (Validação Zod):**

```typescript
const CurriculumSchema = z.object({
  fullName: z.string().min(3).max(100),
  mainArea: z.string().min(1, 'Selecione uma área'), // categoria Cap. 6
  subAreas: z.array(z.string()).max(3).optional(),
  yearsExperience: z.number().int().min(0).max(60),
  state: z.string().length(2),
  city: z.string().min(2).max(100),
  neighborhood: z.string().max(100).optional(),
  aboutMe: z.string().min(50).max(500),
  contactEmail: z.string().email(),
  contactPhone: z.string().optional(),
})

const VideoResumeSchema = z.object({
  file: z.custom<File>()
    .refine(f => f.size <= 100 * 1024 * 1024, 'Máximo 100MB')
    .refine(f => ['video/mp4', 'video/quicktime', 'video/webm'].includes(f.type), 'Formato inválido')
    .refine(async f => {
      const duration = await getVideoDuration(f)
      return duration <= 90
    }, 'Duração máxima: 90 segundos'),
})
```

**Área de Upload:**
- Drop zone reutiliza padrão do doc 06-VIDEO-UPLOAD
- Borda dashed 2px `--border`, border-radius `--radius`
- Hover/drag-over: borda `--primary` (Gold), background `--primary` opacity 0.05
- Ícone upload: 48px `--muted-foreground`
- Após upload: preview inline do vídeo com player
- Validação de duração: se > 90s, mostra erro "Seu vídeo tem X segundos. O limite é 90 segundos."
- Progress bar durante upload: `--primary` (Gold)

**Dicas de Gravação:**
- Box com background `--muted` opacity 0.5, border-left 3px `--primary` (Gold)
- Ícone lâmpada 💡 Manrope 13px `--muted-foreground`
- Dicas curtas, práticas, em lista simples

---

## 21.3 FLUXO DA EMPRESA: VISUALIZAÇÃO DE CURRÍCULOS

### 21.3.1 Rota
```
/talentos
/talentos/:moradorId
```

### 21.3.2 Listagem de Currículos

```
┌───────────────────────────────────────────────────────────┐
│  BANCO DE TALENTOS                                         │
│  Michroma 18px --foreground                                │
│  "Encontre profissionais qualificados"                     │
│  Manrope 14px --muted-foreground                           │
│                                                             │
│  ┌──────────────────────────────────────────────────┐     │
│  │ 🔍 Buscar por área, nome ou palavra-chave...     │     │
│  └──────────────────────────────────────────────────┘     │
│                                                             │
│  Filtros:                                                  │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐      │
│  │ Área [▾]     │ │ Região [▾]   │ │ Experiên.[▾] │      │
│  └──────────────┘ └──────────────┘ └──────────────┘      │
│                                                             │
│  Filtros ativos: [Elétrica ×] [São Paulo ×]  Limpar       │
│                                                             │
│  48 profissionais encontrados                              │
│                                                             │
│  ┌─────────────────────┐  ┌─────────────────────┐        │
│  │ ┌──────┐            │  │ ┌──────┐            │        │
│  │ │Avatar│ João Silva  │  │ │Avatar│ Maria Lima │        │
│  │ │ 56px │            │  │ │ 56px │            │        │
│  │ └──────┘            │  │ └──────┘            │        │
│  │                     │  │                     │        │
│  │ Elétrica · 8 anos   │  │ Hidráulica · 5 anos│        │
│  │ Zona Sul - SP       │  │ Centro - RJ        │        │
│  │                     │  │                     │        │
│  │ ┌─────────────────┐ │  │ ┌─────────────────┐ │        │
│  │ │ 🎬 Video thumb  │ │  │ │ 🎬 Video thumb  │ │        │
│  │ │ [▶ 1:12]        │ │  │ │ [▶ 0:45]        │ │        │
│  │ └─────────────────┘ │  │ └─────────────────┘ │        │
│  │                     │  │                     │        │
│  │ "Eletricista com    │  │ "Encanadora com     │        │
│  │  certificação..."   │  │  experiência em..." │        │
│  │                     │  │                     │        │
│  │ [Ver Perfil]        │  │ [Ver Perfil]        │        │
│  │ [💛 Demonstrar      │  │ [💛 Demonstrar      │        │
│  │  Interesse]         │  │  Interesse]         │        │
│  └─────────────────────┘  └─────────────────────┘        │
│                                                             │
│  (grid continua...)                                        │
│                                                             │
│  [Carregar mais]                                           │
└───────────────────────────────────────────────────────────┘
```

**Grid de Cards de Currículo:**
- Desktop ≥1024px: 3 colunas
- Tablet 768-1023px: 2 colunas
- Mobile <768px: 1 coluna
- Gap: 16px

**Card de Currículo:**
- Background: `--card`
- Border: `--border`
- Border-radius: `--radius` (0.5rem)
- Padding: 16px
- Hover: sutil elevation (box-shadow 0 2px 8px rgba(0,0,0,0.08))

**Conteúdo do Card:**
- Avatar: 56px circular, com iniciais se sem foto
- Nome: Manrope 16px semibold `--foreground`
- Área + Experiência: Manrope 13px `--muted-foreground`, separados por "·"
- Região: Manrope 12px `--muted-foreground`
- Thumbnail do vídeo: 16:9 ratio dentro do card, border-radius 4px, com overlay play button e duração
- "Sobre mim" truncado: Manrope 13px `--muted-foreground`, max 2 linhas com ellipsis
- Botão "Ver Perfil": secondary, borda `--border`, texto `--foreground`
- Botão "Demonstrar Interesse": `--primary` (Gold), ícone 💛

**Filtros:**
- Área: Kobalte Select com as 31 categorias do Cap. 6
- Região: Estado + Cidade (cascading selects)
- Experiência: ranges ("0-2 anos", "3-5 anos", "6-10 anos", "10+ anos")
- Palavra-chave: busca no campo "Sobre mim" e nome
- Filtros ativos: pills removíveis abaixo do input de busca
- Contagem de resultados atualiza em tempo real

**Paginação:**
- Infinite scroll com "Carregar mais" (botão, não scroll automático)
- 12 cards por página
- TanStack Query com `useInfiniteQuery`

### 21.3.3 Detalhe do Currículo

```
┌───────────────────────────────────────────────────────────┐
│  ← Voltar para Banco de Talentos                           │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐ │
│  │                                                      │ │
│  │  ┌──────┐   JOÃO SILVA DA COSTA                     │ │
│  │  │Avatar│   Elétrica, Energia e SPDA                 │ │
│  │  │ 80px │   8 anos de experiência                    │ │
│  │  └──────┘   Zona Sul - São Paulo/SP                  │ │
│  │                                                      │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐ │
│  │                                                      │ │
│  │             VÍDEO-CURRÍCULO                          │ │
│  │                                                      │ │
│  │  ┌──────────────────────────────────────────────┐   │ │
│  │  │                                              │   │ │
│  │  │         Player do Vídeo                      │   │ │
│  │  │         (16:9 ratio, max 640px)              │   │ │
│  │  │         1:12 / 1:30                          │   │ │
│  │  │                                              │   │ │
│  │  └──────────────────────────────────────────────┘   │ │
│  │                                                      │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐ │
│  │  SOBRE MIM                                           │ │
│  │                                                      │ │
│  │  "Eletricista com certificação NR-10 e mais de      │ │
│  │   8 anos de experiência em manutenção predial.      │ │
│  │   Especialista em instalações elétricas de baixa    │ │
│  │   e média tensão, detecção de falhas e sistemas     │ │
│  │   de iluminação de emergência..."                   │ │
│  │                                                      │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐ │
│  │  ESPECIALIDADES                                      │ │
│  │                                                      │ │
│  │  [Instalações elétricas] [Manutenção elétrica]      │ │
│  │  [Para-raios (SPDA)]                                │ │
│  │                                                      │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐ │
│  │                                                      │ │
│  │  [💛 Demonstrar Interesse neste Profissional]       │ │
│  │  (--primary Gold, full-width max 400px, centraliz.) │ │
│  │                                                      │ │
│  │  Ao demonstrar interesse, o profissional receberá   │ │
│  │  seus dados de contato empresarial.                 │ │
│  │  Manrope 12px --muted-foreground                    │ │
│  │                                                      │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                             │
└───────────────────────────────────────────────────────────┘
```

**Especialidades:**
- Exibidas como badges/pills
- Background: `--muted`, text: `--foreground`
- Border-radius: 16px (pill shape)
- Padding: 4px 12px
- Manrope 13px

**Botão "Demonstrar Interesse":**
- Full-width, max 400px, centralizado
- `--primary` (Gold) background, text `--primary-foreground` (branco)
- Height: 48px, border-radius `--radius`
- Ícone 💛 à esquerda
- Ao clicar: abre modal de confirmação

### 21.3.4 Modal "Demonstrar Interesse"

```
┌────────────────────────────────────────┐
│  DEMONSTRAR INTERESSE                  │
│                                        │
│  Você está demonstrando interesse      │
│  no perfil profissional de:            │
│                                        │
│  João Silva da Costa                   │
│  Elétrica · 8 anos · SP               │
│                                        │
│  O profissional receberá:              │
│  · Nome da sua empresa                 │
│  · Categoria da sua empresa            │
│  · Email de contato empresarial        │
│                                        │
│  Mensagem opcional:                    │
│  ┌────────────────────────────────┐    │
│  │ "Temos interesse em conhecer   │    │
│  │  seu trabalho para um projeto  │    │
│  │  em andamento..."              │    │
│  │                                │    │
│  │ (max 300 chars)                │    │
│  └────────────────────────────────┘    │
│  189/300                               │
│                                        │
│  [Cancelar]      [Confirmar Interesse] │
│                     (--primary Gold)   │
└────────────────────────────────────────┘
```

- Lista transparente do que será compartilhado com o profissional
- Textarea opcional para mensagem personalizada (max 300 chars)
- Após confirmação: toast success "Interesse demonstrado com sucesso. João receberá seus dados."
- Cooldown: empresa não pode demonstrar interesse no mesmo profissional mais de 1 vez por mês

---

## 21.4 VISUALIZAÇÃO NO PERFIL DO MORADOR

Conforme doc 04-PROFILE-MANAGEMENT, o vídeo-currículo aparece como seção fixa no perfil do morador pagante.

### 21.4.1 Regras de Visibilidade no Perfil

| Quem visualiza | Vê vídeo-currículo? | Vê ficha profissional? |
|---|---|---|
| O próprio morador | ✅ Sim + métricas | ✅ Sim + edição |
| Outro morador | ❌ Não | ❌ Não |
| Morador Base | ❌ Não | ❌ Não |
| Síndico (qualquer) | ❌ Não | ❌ Não |
| Empresa Plus | ✅ Sim | ✅ Sim |
| Empresa Pro | ✅ Sim | ✅ Sim |
| Marketing | ✅ Sim | ✅ Sim |
| Vizinhança | ✅ Sim | ✅ Sim |

### 21.4.2 Seção no Perfil (Quando Visível)

```
(dentro da página de perfil do morador)

┌──────────────────────────────────────────────────────┐
│  VÍDEO-CURRÍCULO                                     │
│                                                      │
│  ┌───────────────────────────────┐                   │
│  │                               │                   │
│  │    Player 16:9                │   Elétrica · 8a   │
│  │    [▶ 1:12]                   │   Zona Sul - SP   │
│  │                               │                   │
│  │                               │   "Eletricista    │
│  └───────────────────────────────┘    com certif..."  │
│                                                      │
│  [💛 Demonstrar Interesse]                           │
│                                                      │
└──────────────────────────────────────────────────────┘
```

- Layout horizontal em desktop: vídeo à esquerda (60%), dados à direita (40%)
- Layout vertical em mobile: vídeo em cima, dados embaixo
- Se perfil não tem vídeo-currículo: seção não aparece (sem placeholder para visitantes)

---

## 21.5 RESPONSIVIDADE

| Breakpoint | Grid Talentos | Card Layout | Player |
|---|---|---|---|
| ≥1024px | 3 colunas | Vertical card | Max 640px |
| 768-1023px | 2 colunas | Vertical card | Full-width |
| <768px | 1 coluna | Vertical card full-width | Full-width |

### 21.5.1 Mobile — Bottom Sheet Filtros
Em mobile (<768px), os filtros abrem em bottom sheet (Kobalte Dialog/Sheet) em vez de dropdowns inline.
- Trigger: botão "Filtros" com ícone de funil + contagem de filtros ativos
- Sheet: slide up from bottom, max-height 70vh, com scroll interno
- Botão "Aplicar Filtros" no rodapé do sheet

---

## 21.6 COMPONENTES

| Componente | Descrição | Props |
|---|---|---|
| `CurriculumCard` | Card de currículo na listagem | `curriculum`, `onInterest` |
| `CurriculumDetail` | Página completa do currículo | `moradorId` |
| `VideoResumePlayer` | Player específico do vídeo-currículo | `videoUrl`, `duration` |
| `CurriculumForm` | Formulário de cadastro/edição em 2 steps | `initialData?`, `onSubmit` |
| `QuarterlyLockBar` | Barra de progresso da trava trimestral | `lastUploadDate`, `lockDays` |
| `InterestModal` | Modal de demonstrar interesse | `professional`, `onConfirm` |
| `CurriculumMetrics` | 3 mini-cards métricas privadas | `views`, `interests`, `companies` |
| `SpecialtyPills` | Pills de especialidades | `specialties: string[]` |
| `TalentFilters` | Painel de filtros (ou bottom sheet mobile) | `onFilter`, `activeFilters` |
| `EmptyCurriculum` | Estado vazio primeiro cadastro | `onStart` |

---

## 21.7 ESTADOS E FEEDBACK

### 21.7.1 Loading
- Grid de currículos: 6 skeleton cards (thumbnail pulsante + linhas de texto pulsantes)
- Detalhe: skeleton do player + linhas de texto

### 21.7.2 Empty States
- Busca sem resultados: "Nenhum profissional encontrado para os filtros aplicados. Tente ampliar a busca."
- Morador sem currículo (visto pela empresa no perfil): seção simplesmente não aparece

### 21.7.3 Toasts (solid-sonner)
- Currículo publicado: toast success "Seu currículo foi publicado com sucesso! Empresas já podem encontrá-lo."
- Vídeo atualizado: toast success "Vídeo-currículo atualizado. Próxima atualização em 90 dias."
- Interesse demonstrado: toast success "Interesse demonstrado. [Nome] receberá seus dados."
- Upload bloqueado (< 90 dias): toast warning "Aguarde mais X dias para atualizar seu vídeo."
- Erro de upload: toast error "Erro ao enviar vídeo. Verifique o formato e tente novamente."

### 21.7.4 Paywall
- Se Morador Base tenta acessar `/oportunidade`: tela de paywall conforme doc 24 (futuro)
  - "Cadastre seu currículo e conecte-se a oportunidades. Disponível para Moradores Pagantes."
  - Botão "Conhecer plano"
- Se empresa sem permissão tenta acessar `/talentos`: HTTP 403, tela não carrega

---

## 21.8 ACESSIBILIDADE

- Cards de currículo: `role="article"`, com `aria-label` descritivo ("Currículo de João Silva, Elétrica, 8 anos, São Paulo")
- Player do vídeo: controles acessíveis por keyboard (play/pause com Space, seek com arrows)
- Formulário: labels associados a inputs, erro messages com `aria-describedby`
- Modal de interesse: focus trap, `aria-modal="true"`, close com Escape
- Filtros: `aria-expanded` nos selects, `aria-selected` nas opções

---

## 21.9 NOTAS DE IMPLEMENTAÇÃO

1. **Upload de vídeo**: Reutilizar o fluxo do doc 06-VIDEO-UPLOAD com integração Mux/Cloudflare Stream. O backend valida duração ≤ 90s server-side (não confiar apenas no frontend).

2. **Trava trimestral**: O backend é a verdade única. O frontend exibe a barra de progresso e desabilita o botão, mas a API deve rejeitar uploads dentro dos 90 dias independentemente.

3. **"Demonstrar Interesse"**: Não é Connect Me. É um disparo simples (email/notificação) que envia dados da empresa para o morador. Sem reply, sem chat, sem histórico de conversa.

4. **Privacidade**: O email/telefone do morador é compartilhado APENAS quando a empresa clica "Demonstrar Interesse" — não é visível publicamente na listagem ou no detalhe do currículo.

5. **Métricas do currículo**: Seguem a regra do doc 16 (Private Feedback) — visíveis apenas para o dono. Empresas não sabem quantas outras empresas viram o mesmo currículo.

6. **Categorias do formulário**: As áreas de atuação correspondem às 31 categorias do Capítulo 6 do Manual Técnico. As subcategorias são as subcategorias de cada categoria. Dados vêm do backend (mesma fonte que alimenta a busca de empresas).

7. **Contagem de visualização**: Para contabilizar no metrics do morador, a empresa precisa ter ficado ≥5 segundos no detalhe do currículo ou assistido ≥50% do vídeo (evitar contagens acidentais).
