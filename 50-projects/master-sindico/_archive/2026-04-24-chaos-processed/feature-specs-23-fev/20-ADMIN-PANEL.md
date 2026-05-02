# 20 — PAINEL ADMINISTRATIVO GLOBAL (MASTERSÍNDICO)

> **Sprint 4** · Exclusivo Web · Acesso: Apenas Administrador MasterSíndico (Dono da Plataforma)
> Dependências: 00-DESIGN-TOKENS · 02-APP-SHELL · 16-PRIVATE-FEEDBACK

---

## 20.1 CONTEXTO E REGRAS DE NEGÓCIO

### 20.1.1 Definição
O Painel Administrativo é a central de controle global da plataforma Master Síndico. É o único ponto de acesso para gestão financeira, moderação de conteúdo, gerenciamento de usuários, configurações da plataforma e envio de comunicados oficiais (broadcast).

### 20.1.2 Restrição Crítica de Acesso
- **EXCLUSIVO WEB**: Este painel NÃO existe no aplicativo mobile.
- **Apenas o perfil "MasterSíndico" (Administrador/Dono da Plataforma)** tem acesso.
- Se qualquer outro perfil tentar acessar `/admin/*`, recebe **HTTP 403** e é redirecionado para a home.
- O middleware ABAC do TanStack Router deve bloquear a renderização completa da rota — **nenhum componente do admin carrega** para perfis não autorizados.

### 20.1.3 Escopo Funcional (5 Módulos)
Conforme SOW item 4.6:
1. **Dashboard Financeiro** (4.6.1)
2. **Gestão de Usuários** (4.6.2)
3. **Moderação de Conteúdo** (4.6.3)
4. **Configurações da Plataforma** (4.6.4)
5. **Comunicados Oficiais — Broadcast** (4.6.5)

### 20.1.4 Princípio de Design
O admin segue a mesma identidade visual da plataforma (Navy + Gold), porém com densidade de informação maior. O layout prioriza **eficiência operacional** sobre estética emocional — é uma ferramenta de trabalho, não uma vitrine.

---

## 20.2 LAYOUT GERAL DO ADMIN

### 20.2.1 Estrutura da Página

```
┌─────────────────────────────────────────────────────────────┐
│ TOPBAR ADMIN (Navy --surface)                                │
│ ┌──────┐  Master Síndico · Admin    [🔔] [Avatar ▾]        │
│ └──────┘                                                     │
├────────────┬────────────────────────────────────────────────┤
│            │                                                  │
│  SIDEBAR   │  CONTENT AREA                                   │
│  ADMIN     │                                                  │
│  (240px)   │  ┌──────────────────────────────────────────┐  │
│            │  │  Breadcrumb: Admin > Dashboard            │  │
│  Dashboard │  ├──────────────────────────────────────────┤  │
│  Usuários  │  │                                          │  │
│  Moderação │  │  (Conteúdo dinâmico do módulo ativo)     │  │
│  Config.   │  │                                          │  │
│  Comunic.  │  │                                          │  │
│            │  │                                          │  │
│            │  │                                          │  │
│            │  └──────────────────────────────────────────┘  │
│            │                                                  │
└────────────┴────────────────────────────────────────────────┘
```

### 20.2.2 Sidebar Admin

A sidebar do admin é **separada da sidebar principal da plataforma**. Quando o usuário está em `/admin/*`, a sidebar muda completamente.

```
SIDEBAR ADMIN (240px fixo, Navy --surface)
─────────────────────────────
Logo Master Síndico (versão compacta)
"Painel Administrativo" (Manrope 11px, --muted-foreground, uppercase, letter-spacing 0.1em)

─── NAVEGAÇÃO ───

📊 Dashboard
   └─ Financeiro
   └─ Visão Geral

👥 Usuários
   └─ Listar Todos
   └─ Por Perfil/Plano
   └─ Histórico

🛡️ Moderação
   └─ Vídeos Denunciados
   └─ Posts do Fórum
   └─ Conteúdo Removido

⚙️ Configurações
   └─ Termos e Políticas
   └─ Banners da Home
   └─ Categorias Técnicas
   └─ Parâmetros Globais

📢 Comunicados
   └─ Novo Comunicado
   └─ Histórico de Envios
   └─ Agendados

─── FOOTER SIDEBAR ───
← Voltar à Plataforma (link para /)
v2.0.0 (Manrope 10px muted)
─────────────────────────────
```

**Especificações visuais:**
- Background: `--surface` (Navy oklch(0.218 0.055 256.1))
- Texto ativo: `--primary` (Gold) com barra lateral 3px gold à esquerda
- Texto inativo: `--surface-foreground` (cinza claro)
- Hover: `--surface-variant` como background
- Ícones: 18px, Iconify `lucide:*` ou `tabler:*`
- Sub-itens: indent 16px à esquerda, font-size 13px
- Transição hover: `transition: background 150ms ease`

### 20.2.3 Topbar Admin

```
TOPBAR (height: 56px, Navy --surface, border-bottom 1px --border)
┌─────────────────────────────────────────────────────────────┐
│ [☰ toggle]  Admin · Master Síndico         [🔔 badge] [AV]│
└─────────────────────────────────────────────────────────────┘
```

- O toggle `☰` colapsa a sidebar em telas ≤1280px
- Badge de notificação mostra contagem de itens pendentes de moderação
- Avatar dropdown com: "Minha Conta", "Voltar à Plataforma", "Sair"
- Breadcrumb aparece abaixo da topbar na content area, não dentro dela

### 20.2.4 Responsividade Admin

| Breakpoint | Sidebar | Comportamento |
|---|---|---|
| ≥1280px | Fixa 240px, sempre visível | Layout padrão |
| 1024–1279px | Colapsável, inicia fechada | Toggle no topbar abre overlay |
| <1024px | **NÃO SUPORTADO** — exibe mensagem | "O painel administrativo requer tela ≥1024px" |

**Mensagem de tela pequena:**
```
┌──────────────────────────────────────┐
│          🖥️ TELA NECESSÁRIA          │
│                                      │
│  O Painel Administrativo requer      │
│  uma tela de pelo menos 1024px.      │
│                                      │
│  Acesse de um computador para        │
│  gerenciar a plataforma.             │
│                                      │
│  [← Voltar à Plataforma]            │
└──────────────────────────────────────┘
```
- Centralizado vertical e horizontalmente
- Fundo `--background`, texto `--foreground`
- Botão gold `--primary`

---

## 20.3 MÓDULO 1: DASHBOARD FINANCEIRO (4.6.1)

### 20.3.1 Rota
```
/admin/dashboard
/admin/dashboard/financeiro
```

### 20.3.2 Visão Geral — Cards de Resumo

```
┌───────────────────────────────────────────────────────────┐
│  DASHBOARD FINANCEIRO                                      │
│  Manrope 12px uppercase --muted-foreground                 │
│                                                             │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌──────┐│
│  │ MRR          │ │ Assinaturas │ │ Cancelam.   │ │Churn ││
│  │              │ │ Ativas      │ │ (mês)       │ │ Rate ││
│  │ R$ 45.230    │ │ 1.284       │ │ 47          │ │ 3.6% ││
│  │ ↑ 12.3%      │ │ ↑ 8.1%      │ │ ↓ 2.1%      │ │↓0.4%││
│  │ vs mês ant.  │ │ vs mês ant. │ │ vs mês ant. │ │      ││
│  └─────────────┘ └─────────────┘ └─────────────┘ └──────┘│
│                                                             │
└───────────────────────────────────────────────────────────┘
```

**Cards de Resumo (KPI Cards):**
- Grid: 4 colunas (`grid-template-columns: repeat(4, 1fr)`, gap 16px)
- Card: `--card` background, border `--border`, border-radius `--radius`
- Padding: 20px
- Label: Manrope 12px uppercase `--muted-foreground`, letter-spacing 0.05em
- Valor principal: **Michroma 28px** `--foreground`
- Variação positiva: `--success` (verde) com ícone ↑
- Variação negativa: `--destructive` (vermelho) com ícone ↓
- Subtexto: Manrope 11px `--muted-foreground` "vs mês anterior"

### 20.3.3 Gráficos de Receita

```
┌───────────────────────────────────────────────────────────┐
│                                                             │
│  RECEITA RECORRENTE (MRR)            Período: [▾ 12 meses]│
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │                                                     │  │
│  │         📈 Line Chart (MRR ao longo do tempo)       │  │
│  │                                                     │  │
│  │  Y: R$ valores                                      │  │
│  │  X: Meses (Jan, Fev, Mar...)                        │  │
│  │                                                     │  │
│  │  Linha principal: --primary (Gold) 2px              │  │
│  │  Área preenchida: --primary com opacity 0.1         │  │
│  │  Grid lines: --border com opacity 0.5               │  │
│  │  Tooltip on hover: Card com valor exato             │  │
│  │                                                     │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
└───────────────────────────────────────────────────────────┘
```

**Especificações do gráfico:**
- Biblioteca sugerida: `chart.js` ou `unovis` (leve, tree-shakeable)
- Linha MRR: cor `--primary` (Gold), stroke-width 2px, pontos circulares 4px
- Área abaixo da linha: gradiente de `--primary` opacity 0.15 → transparent
- Eixo Y: Manrope 11px `--muted-foreground`, valores em R$ abreviados (45K)
- Eixo X: Manrope 11px `--muted-foreground`, meses abreviados
- Grid horizontal: `--border` opacity 0.3, dashed
- Tooltip no hover: card `--card` com sombra, mostra "Mês: R$ valor_exato"
- Seletor de período: dropdown Kobalte com opções "3 meses", "6 meses", "12 meses", "Todo período"

### 20.3.4 Distribuição por Plano

```
┌──────────────────────────┐ ┌──────────────────────────┐
│  DISTRIBUIÇÃO POR PLANO  │ │ RECEITA POR PLANO        │
│                          │ │                          │
│  ┌────────────────────┐  │ │  Síndico N3    R$ 12.400 │
│  │                    │  │ │  ████████████████  27.4%  │
│  │   🍩 Donut Chart   │  │ │                          │
│  │                    │  │ │  Emp. Pro       R$ 10.800 │
│  │   Total: 1.284     │  │ │  █████████████░░  23.9%  │
│  │                    │  │ │                          │
│  └────────────────────┘  │ │  Síndico N2    R$ 8.200  │
│                          │ │  ██████████░░░░░  18.1%  │
│  ● Base        412 32%  │ │                          │
│  ● Morador Pg  234 18%  │ │  Emp. Plus     R$ 6.100  │
│  ● Síndico N1  189 15%  │ │  ████████░░░░░░░  13.5%  │
│  ● Síndico N2  156 12%  │ │                          │
│  ● Síndico N3   98  8%  │ │  (demais planos...)      │
│  ● Emp. Plus    87  7%  │ │                          │
│  ● Emp. Pro     63  5%  │ │                          │
│  ● Marketing    28  2%  │ │                          │
│  ● Vizinhança   17  1%  │ │                          │
└──────────────────────────┘ └──────────────────────────┘
```

**Donut Chart:**
- Cores por plano (paleta derivada do design system, nunca sair do espectro Navy-Gold):
  - Base: `--muted` (cinza)
  - Morador Pg: oklch(0.6 0.08 84) (gold escuro)
  - Síndico N1: oklch(0.5 0.06 256) (navy médio)
  - Síndico N2: oklch(0.6 0.07 256) (navy claro)
  - Síndico N3: `--primary` (Gold pleno)
  - Emp. Plus: oklch(0.55 0.09 84) (gold médio)
  - Emp. Pro: oklch(0.72 0.12 84) (gold brilhante)
  - Marketing: oklch(0.65 0.1 200) (azul intermediário)
  - Vizinhança: oklch(0.7 0.05 84) (gold pálido)
- Centro do donut: número total em Michroma 24px
- Legenda abaixo: bullets coloridos + label + count + percentual
- Legenda font: Manrope 13px

**Barras de Receita por Plano:**
- Barras horizontais, preenchimento proporcional
- Barra preenchida: cor do plano correspondente
- Barra vazia: `--muted` background
- Border-radius: 4px
- Valor: Manrope 14px bold `--foreground`
- Percentual: Manrope 12px `--muted-foreground`

### 20.3.5 Integração com Gateway de Pagamento
- O dashboard consome dados da API de integração com **Mercado Pago** (ou outro gateway escolhido)
- Os dados são cacheados no backend e servidos via endpoints próprios
- O frontend **não** se comunica diretamente com o Mercado Pago
- Período de refresh: polling a cada 5 minutos ou manual com botão "Atualizar dados"

---

## 20.4 MÓDULO 2: GESTÃO DE USUÁRIOS (4.6.2)

### 20.4.1 Rota
```
/admin/usuarios
/admin/usuarios/:userId
```

### 20.4.2 Listagem de Usuários

```
┌───────────────────────────────────────────────────────────┐
│  GESTÃO DE USUÁRIOS                                        │
│                                                             │
│  ┌─────────────────────────────────────────┐ ┌───────────┐│
│  │ 🔍 Buscar por nome, email, CPF/CNPJ... │ │ Filtros ▾ ││
│  └─────────────────────────────────────────┘ └───────────┘│
│                                                             │
│  Filtros ativos: [Síndico N2 ×] [Ativo ×]   Limpar todos  │
│                                                             │
│  ┌─────────────────────────────────────────────────────────┐
│  │ ☐ │ Usuário          │ Perfil      │ Plano    │ Status │
│  ├───┼──────────────────┼─────────────┼──────────┼────────┤
│  │ ☐ │ 🟢 João Silva    │ Síndico     │ N3 Pro   │ Ativo  │
│  │   │ joao@email.com   │             │          │        │
│  ├───┼──────────────────┼─────────────┼──────────┼────────┤
│  │ ☐ │ 🟢 Maria Santos  │ Empresa     │ Plus     │ Ativo  │
│  │   │ maria@emp.com.br │             │          │        │
│  ├───┼──────────────────┼─────────────┼──────────┼────────┤
│  │ ☐ │ 🔴 Pedro Lima    │ Morador     │ Pagante  │Bloqueado│
│  │   │ pedro@gmail.com  │             │          │        │
│  └───┴──────────────────┴─────────────┴──────────┴────────┘
│                                                             │
│  Mostrando 1-25 de 1.284    [← Anterior] [1] [2] [3] [→]  │
└───────────────────────────────────────────────────────────┘
```

**Tabela de Usuários:**
- Componente: Tabela HTML semântica (`<table>`) estilizada, não grid CSS
- Header: `--muted` background, Manrope 12px uppercase `--muted-foreground`, letter-spacing 0.05em
- Rows: background `--card`, border-bottom `--border`, hover `--muted` opacity 0.5
- Checkbox coluna: para ações em lote (bloquear/desbloquear)
- Avatar coluna "Usuário": círculo 32px com iniciais ou foto
- Indicador online: dot 8px `--success` (ativo) ou `--destructive` (bloqueado) ao lado do nome
- Email: Manrope 12px `--muted-foreground`, abaixo do nome
- Perfil: Badge com cor do perfil (mesmo mapeamento do donut chart)
- Plano: Texto simples Manrope 13px
- Status: Badge — "Ativo" (green), "Bloqueado" (red), "Pendente" (yellow)

**Filtros (Dropdown Kobalte):**
- Por Perfil: Base, Morador Pg, Síndico N1/N2/N3, Emp. Plus/Pro, Marketing, Vizinhança
- Por Plano: Todos os planos disponíveis
- Por Status: Ativo, Bloqueado, Pendente
- Por Data de Cadastro: Range de datas
- Filtros ativos aparecem como pills removíveis abaixo do input de busca

**Paginação:**
- 25 itens por página (fixo)
- Botões de navegação: `← Anterior` `→ Próximo` com números de página
- Texto "Mostrando X-Y de Z": Manrope 12px `--muted-foreground`
- Estilo botões: border `--border`, hover `--muted`, active `--primary` text

### 20.4.3 Detalhe do Usuário

```
┌───────────────────────────────────────────────────────────┐
│  ← Voltar para lista                                       │
│                                                             │
│  ┌──────────┐                                              │
│  │          │  JOÃO SILVA DA COSTA                         │
│  │  Avatar  │  joao.silva@email.com                        │
│  │  80x80   │  CPF: ***.***.***-34                         │
│  │          │  Cadastro: 15/03/2025                        │
│  └──────────┘  Último acesso: há 2 horas                   │
│                                                             │
│  Perfil: [Síndico]  Plano: [N3 Pro]  Status: [🟢 Ativo]   │
│                                                             │
│  ┌─────────┬──────────┬──────────┬──────────┬────────────┐│
│  │ Dados   │Condomínios│Atividade │ Conteúdo │  Ações     ││
│  └─────────┴──────────┴──────────┴──────────┴────────────┘│
│                                                             │
│  (Conteúdo da tab ativa)                                    │
│                                                             │
└───────────────────────────────────────────────────────────┘
```

**Tabs do Detalhe (Kobalte Tabs):**

**Tab "Dados":**
- Formulário read-only com todos os dados cadastrais
- Campos específicos por persona (conforme SOW 1.3):
  - Morador: dados básicos + unidade vinculada
  - Síndico: dados profissionais + condomínios vinculados + status Bronze/Prata/Ouro/Diamante
  - Empresa: dados PJ + até 5 categorias técnicas selecionadas
- Botão "Editar Dados" (abre formulário editável, caso excepcional conforme SOW 4.6.2)
- Botão "Editar" com ícone de cadeado e tooltip: "Edição administrativa — use com cautela"

**Tab "Condomínios" (apenas para Síndicos):**
- Lista de condomínios vinculados
- Para cada: Nome, Endereço, Qtd moradores, Data de vínculo
- Badge de status calculado (Bronze/Prata/Ouro/Diamante)

**Tab "Atividade":**
- Timeline vertical com últimas ações do usuário:
  - Login, Upload de vídeo, Votação, Avaliação, Connect Me enviado, etc.
- Formato: `[Data/Hora] — [Ação] — [Detalhe]`
- Timeline line: 2px `--border`, dots 8px `--primary`
- Max: últimas 50 ações, com scroll

**Tab "Conteúdo":**
- Grid de cards com vídeos publicados pelo usuário
- Para empresas: vídeos instrucionais + cursos criados
- Para síndicos: assembleias criadas + vídeos de pauta
- Card mini: thumbnail 160x90, título truncado 2 linhas, data

**Tab "Ações":**
- **Bloquear Usuário**: Botão `--destructive`, abre modal de confirmação
  - Motivo obrigatório (textarea 100-500 chars)
  - Checkbox "Enviar email notificando bloqueio"
  - Modal: "Tem certeza que deseja bloquear [Nome]? Esta ação impedirá o acesso à plataforma."
- **Desbloquear Usuário**: Botão `--success`, confirmação simples
- **Resetar Senha**: Envia email de redefinição
- **Alterar Plano**: Dropdown com planos disponíveis para aquele perfil
  - Aviso: "Alterar o plano afeta imediatamente as permissões do usuário."

---

## 20.5 MÓDULO 3: MODERAÇÃO DE CONTEÚDO (4.6.3)

### 20.5.1 Rota
```
/admin/moderacao
/admin/moderacao/videos
/admin/moderacao/forum
/admin/moderacao/historico
```

### 20.5.2 Fila de Moderação — Visão Geral

```
┌───────────────────────────────────────────────────────────┐
│  MODERAÇÃO DE CONTEÚDO                                     │
│                                                             │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐             │
│  │ 🎬 Vídeos  │ │ 💬 Fórum   │ │ 📋 Histór. │             │
│  │ 12 pending │ │ 5 pending  │ │ 847 total  │             │
│  └────────────┘ └────────────┘ └────────────┘             │
│                                                             │
│  VÍDEOS DENUNCIADOS (12)                                   │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ ┌─────────┐  "Como resolver infiltração..."         │  │
│  │ │ Thumb   │  Por: Empresa XYZ · Emp. Pro            │  │
│  │ │ 120x68  │  Denúncias: 3 · Motivo: Conteúdo       │  │
│  │ │         │  comercial / Propaganda                  │  │
│  │ └─────────┘  Há 2 horas                             │  │
│  │              [▶ Assistir] [✓ Aprovar] [✗ Reprovar]  │  │
│  ├─────────────────────────────────────────────────────┤  │
│  │ (próximo item...)                                    │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
└───────────────────────────────────────────────────────────┘
```

**Cards de Contagem (topo):**
- 3 cards horizontais, cada um com ícone + label + contagem pendente
- Badge de contagem: se > 0, usa `--destructive` background com texto branco
- Se zero: usa `--muted` background

**Lista de Itens Denunciados:**
- Cada item: thumbnail (vídeo) ou preview (post), título, autor, perfil/plano, qtd denúncias, motivo principal, tempo
- Motivos possíveis de denúncia (pré-definidos, conforme doc 17-FORUM):
  - Conteúdo comercial / Propaganda
  - Linguagem inadequada
  - Informação falsa / Enganosa
  - Dados de contato no conteúdo (telefone, WhatsApp, email)
  - Fora do tema da categoria
  - Outro (com texto livre)
- **Ações rápidas inline:**
  - `[▶ Assistir/Ler]`: abre modal com player do vídeo ou texto do post
  - `[✓ Aprovar]`: remove da fila (conteúdo permanece publicado), badge green
  - `[✗ Reprovar]`: abre modal de confirmação com motivo

### 20.5.3 Modal de Reprovação/Remoção

```
┌────────────────────────────────────────┐
│  REMOVER CONTEÚDO                      │
│                                        │
│  Conteúdo: "Como resolver infiltra..." │
│  Autor: Empresa XYZ                    │
│                                        │
│  Motivo da remoção:                    │
│  ┌────────────────────────────────┐    │
│  │ [▾ Selecionar motivo]         │    │
│  └────────────────────────────────┘    │
│  Opções:                               │
│  - Violação de diretrizes de conteúdo  │
│  - Conteúdo comercial/propaganda       │
│  - Dados de contato expostos           │
│  - Conteúdo ofensivo                   │
│  - Informação perigosa/falsa           │
│                                        │
│  Observações adicionais:               │
│  ┌────────────────────────────────┐    │
│  │                                │    │
│  │ (textarea opcional, max 500)   │    │
│  │                                │    │
│  └────────────────────────────────┘    │
│                                        │
│  ☐ Suspender funcionalidades do autor  │
│    (empresa reincidente)               │
│                                        │
│  ☐ Notificar autor por email           │
│                                        │
│  [Cancelar]          [Remover Conteúdo]│
│                      (--destructive)   │
└────────────────────────────────────────┘
```

- Select do motivo: Kobalte Select, obrigatório
- Textarea: opcional, borda `--input`, focus ring `--ring`
- Checkbox "Suspender funcionalidades": visível apenas se autor tem histórico de remoções
- Checkbox "Notificar autor": checked por padrão
- Botão "Remover": `--destructive`, disabled até motivo selecionado

### 20.5.4 Moderação do Fórum
- Mesma estrutura da fila de vídeos, mas com preview de texto
- Card mostra: título do post, 3 primeiras linhas do conteúdo, categoria, autor, qtd denúncias
- Ação adicional: "Remover Resposta" (para respostas específicas, não o post inteiro)
- Conforme SOW 4.3: moderação reativa, sem IA automática

### 20.5.5 Histórico de Moderação
- Tabela com todas as ações tomadas
- Colunas: Data, Tipo (Vídeo/Post/Resposta), Conteúdo, Autor, Ação (Aprovado/Removido/Suspenso), Moderador
- Filtros por: tipo, ação, período
- Exportável em CSV (botão no topo direito)

---

## 20.6 MÓDULO 4: CONFIGURAÇÕES DA PLATAFORMA (4.6.4)

### 20.6.1 Rota
```
/admin/configuracoes
/admin/configuracoes/termos
/admin/configuracoes/banners
/admin/configuracoes/categorias
/admin/configuracoes/parametros
```

### 20.6.2 Termos de Uso e Política de Privacidade

```
┌───────────────────────────────────────────────────────────┐
│  TERMOS E POLÍTICAS                                        │
│                                                             │
│  ┌──────────────────┐ ┌──────────────────┐                │
│  │ Termos de Uso    │ │ Política de      │                │
│  │ Última edição:   │ │ Privacidade      │                │
│  │ 15/01/2026       │ │ Última edição:   │                │
│  │ [Editar]         │ │ 15/01/2026       │                │
│  └──────────────────┘ │ [Editar]         │                │
│                        └──────────────────┘                │
│                                                             │
│  EDITOR:                                                   │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ [B] [I] [H1] [H2] [Link] [Lista] [—]               │  │
│  ├─────────────────────────────────────────────────────┤  │
│  │                                                     │  │
│  │  (Rich text editor - conteúdo dos termos)           │  │
│  │                                                     │  │
│  │  Altura mínima: 500px                               │  │
│  │  Suporte a headings, parágrafos, listas, links      │  │
│  │                                                     │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  [Cancelar]                    [Salvar e Publicar]         │
│                                                             │
│  ⚠️ Ao publicar, todos os usuários serão notificados       │
│  sobre a atualização dos termos no próximo login.          │
└───────────────────────────────────────────────────────────┘
```

- Editor rich text: toolbar básica (bold, italic, headings, links, listas ordenadas/não-ordenadas, separador)
- Não precisa ser WYSIWYG completo — pode ser Markdown com preview
- Salvar gera versão (versionamento simples: data + snapshot do conteúdo)
- Aviso amarelo `--warning` sobre notificação automática aos usuários

### 20.6.3 Banners da Home

```
┌───────────────────────────────────────────────────────────┐
│  BANNERS DA HOME                         [+ Novo Banner]  │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ ┌──────────────────┐                                │  │
│  │ │                  │  Banner Principal               │  │
│  │ │   Preview        │  Status: 🟢 Ativo              │  │
│  │ │   320x120        │  Link: /cursos                 │  │
│  │ │                  │  Desde: 01/02/2026              │  │
│  │ └──────────────────┘                                │  │
│  │  [☰ drag]              [Editar] [Desativar] [🗑️]    │  │
│  ├─────────────────────────────────────────────────────┤  │
│  │ (próximo banner, arrastável para reordenar)         │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
└───────────────────────────────────────────────────────────┘
```

- Lista de banners com drag & drop para reordenação (drag handle `☰` à esquerda)
- Preview da imagem: thumbnail 320x120 com border-radius `--radius`
- Cada banner: imagem, título (opcional), link destino, status (ativo/inativo), data de início
- Upload: aceita PNG/JPG/WebP, max 2MB, ratio recomendado 16:5
- Limite: máximo 5 banners ativos simultâneos
- Botão "Novo Banner": abre modal com upload + campos

### 20.6.4 Categorias Técnicas

```
┌───────────────────────────────────────────────────────────┐
│  CATEGORIAS TÉCNICAS                   [+ Nova Categoria] │
│  (Capítulo 6 do Manual Técnico)                           │
│                                                             │
│  🔍 Buscar categoria...                                    │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ 1. Administração, Gestão e Governança        [▾]   │  │
│  │    ├─ Administradoras de condomínios                │  │
│  │    ├─ Consultoria em gestão condominial             │  │
│  │    ├─ Governança condominial                        │  │
│  │    ├─ LGPD e proteção de dados                      │  │
│  │    ├─ Compliance condominial                        │  │
│  │    └─ Mediação de conflitos                         │  │
│  │    [+ Adicionar subcategoria] [Editar] [Desativar]  │  │
│  ├─────────────────────────────────────────────────────┤  │
│  │ 2. Jurídico e Direito Imobiliário            [▾]   │  │
│  │    ├─ Assessoria jurídica condominial               │  │
│  │    ├─ Direito imobiliário                           │  │
│  │    └─ ...                                           │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
└───────────────────────────────────────────────────────────┘
```

- Lista colapsável (accordion Kobalte) das 31 categorias do Cap. 6
- Cada categoria: expandir/colapsar para ver subcategorias
- Ações por categoria: Editar nome, Adicionar subcategoria, Desativar (soft delete)
- Ações por subcategoria: Editar, Remover, Reordenar (drag & drop)
- Busca filtra categorias e subcategorias simultaneamente
- Aviso ao desativar: "X empresas estão vinculadas a esta categoria. Desativar a removerá dos filtros de busca."

### 20.6.5 Parâmetros Globais

```
┌───────────────────────────────────────────────────────────┐
│  PARÂMETROS GLOBAIS                                        │
│                                                             │
│  Tempo padrão de janela de votação                         │
│  ┌──────────┐ dias                                         │
│  │ 7        │                                              │
│  └──────────┘                                              │
│                                                             │
│  Limite de vídeos por mês (Síndico N2)                     │
│  ┌──────────┐ vídeos/mês                                   │
│  │ 4        │                                              │
│  └──────────┘                                              │
│                                                             │
│  Limite de vídeos por mês (Emp. Plus)                      │
│  ┌──────────┐ vídeos/mês                                   │
│  │ 8        │                                              │
│  └──────────┘                                              │
│                                                             │
│  Limite de vídeos por mês (Emp. Pro)                       │
│  ┌──────────┐ vídeos/mês                                   │
│  │ 12       │                                              │
│  └──────────┘                                              │
│                                                             │
│  Trava trimestral de vídeo institucional                   │
│  ┌──────────┐ dias                                         │
│  │ 90       │ ⚠️ Alteração afeta regra hard-coded          │
│  └──────────┘                                              │
│                                                             │
│  Limite de Connect Me (Morador Base)                       │
│  ┌──────────┐ por ano                                      │
│  │ 2        │                                              │
│  └──────────┘                                              │
│                                                             │
│  Limite de Connect Me (Morador Pg)                         │
│  ┌──────────┐ por ano                                      │
│  │ 4        │                                              │
│  └──────────┘                                              │
│                                                             │
│  Trava de cupom por estabelecimento                        │
│  ┌──────────┐ horas                                        │
│  │ 4        │                                              │
│  └──────────┘                                              │
│                                                             │
│  Mínimo de visualização para contagem (Síndico)            │
│  ┌──────────┐ %                                            │
│  │ 70       │                                              │
│  └──────────┘                                              │
│                                                             │
│  [Restaurar Padrões]             [Salvar Alterações]       │
│                                                             │
│  Última alteração: 15/01/2026 por Admin                    │
└───────────────────────────────────────────────────────────┘
```

- Cada parâmetro: label descritivo + input numérico + unidade
- Inputs: border `--input`, focus ring `--ring`, Manrope 14px
- Aviso `--warning` em parâmetros sensíveis (ex: trava trimestral 90 dias)
- Botão "Restaurar Padrões": secondary, abre confirmação
- Botão "Salvar": `--primary` (Gold)
- Timestamp da última alteração: Manrope 11px `--muted-foreground`
- Validação Zod: todos os valores devem ser números inteiros positivos

---

## 20.7 MÓDULO 5: COMUNICADOS OFICIAIS — BROADCAST (4.6.5)

### 20.7.1 Rota
```
/admin/comunicados
/admin/comunicados/novo
/admin/comunicados/historico
/admin/comunicados/agendados
```

### 20.7.2 Criação de Comunicado

```
┌───────────────────────────────────────────────────────────┐
│  NOVO COMUNICADO OFICIAL                                   │
│                                                             │
│  Destinatários:                                            │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ [▾ Selecionar segmentação]                          │  │
│  │  ○ Todos os usuários                                │  │
│  │  ○ Por perfil: [☑ Síndicos] [☑ Empresas] [☐ Mor.] │  │
│  │  ○ Por plano: [▾ Selecionar planos]                 │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  Assunto:                                                  │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ Atualização importante sobre a plataforma           │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  Mensagem:                                                 │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ [B] [I] [Link] [Lista]                              │  │
│  ├─────────────────────────────────────────────────────┤  │
│  │                                                     │  │
│  │ (Editor de mensagem com formatação básica)          │  │
│  │                                                     │  │
│  │ Mínimo: 20 caracteres                               │  │
│  │ Máximo: 5.000 caracteres                            │  │
│  │                                                     │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  Call to Action (opcional):                                │
│  ┌──────────────────┐  ┌──────────────────────────────┐  │
│  │ Texto do botão   │  │ URL de destino               │  │
│  │ "Saiba mais"     │  │ https://mastersindico.com/... │  │
│  └──────────────────┘  └──────────────────────────────┘  │
│                                                             │
│  Agendamento:                                              │
│  ○ Enviar agora                                            │
│  ○ Agendar para: [📅 Data] [🕐 Hora]                      │
│                                                             │
│  ─────────────────────────────────────────────────────     │
│  Preview do Email:                                         │
│  ┌─────────────────────────────────────────────────────┐  │
│  │  ┌──────────────┐                                   │  │
│  │  │ Logo Master  │                                   │  │
│  │  │ Síndico      │                                   │  │
│  │  └──────────────┘                                   │  │
│  │                                                     │  │
│  │  Atualização importante sobre a plataforma          │  │
│  │                                                     │  │
│  │  (Preview do conteúdo formatado)                    │  │
│  │                                                     │  │
│  │  [Saiba mais]                                       │  │
│  │                                                     │  │
│  │  ─────────────────────                              │  │
│  │  Master Síndico · Comunicado Oficial                │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  [Salvar Rascunho]              [Enviar / Agendar]        │
│                                                             │
│  Estimativa: ~1.284 destinatários                          │
└───────────────────────────────────────────────────────────┘
```

**Segmentação:**
- Radio buttons Kobalte para tipo de segmentação
- "Todos os usuários": envia para toda a base
- "Por perfil": checkboxes por tipo (Síndico, Empresa, Morador, Marketing, Vizinhança)
- "Por plano": multi-select com todos os planos disponíveis
- Estimativa de destinatários atualiza em tempo real conforme seleção

**Editor de Mensagem:**
- Toolbar mínima: Bold, Italic, Link, Lista
- Preview do email: renderização live abaixo do editor
- Template do email: header com logo, corpo com conteúdo, CTA button gold, footer institucional
- O preview usa fundo branco com borda sutil para simular email

**CTA (Call to Action):**
- Campos opcionais: texto do botão (max 30 chars) + URL destino
- Se preenchido, aparece como botão gold no preview do email
- Se vazio, email vai sem botão

**Agendamento:**
- "Enviar agora": despacha imediatamente (com confirmação modal)
- "Agendar para": date picker + time picker Kobalte
- Data mínima: agora + 1 hora
- Timezone: horário de Brasília (GMT-3), exibido explicitamente

**Confirmação de envio (modal):**
```
┌────────────────────────────────────────┐
│  CONFIRMAR ENVIO                       │
│                                        │
│  Você está prestes a enviar um         │
│  comunicado para ~1.284 destinatários. │
│                                        │
│  Segmentação: Todos os usuários        │
│  Assunto: "Atualização importante..."  │
│                                        │
│  ⚠️ Esta ação não pode ser desfeita.    │
│                                        │
│  [Cancelar]      [Confirmar Envio]     │
│                     (--primary)        │
└────────────────────────────────────────┘
```

### 20.7.3 Histórico de Comunicados

```
┌───────────────────────────────────────────────────────────┐
│  HISTÓRICO DE COMUNICADOS                                  │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ Data       │ Assunto          │ Dest. │ Status      │  │
│  ├────────────┼──────────────────┼───────┼─────────────┤  │
│  │ 15/02/2026 │ Atualização da.. │ 1.284 │ ✅ Enviado  │  │
│  │ 01/02/2026 │ Novos termos de..│   947 │ ✅ Enviado  │  │
│  │ 20/02/2026 │ Manutenção prog..│ 1.300 │ ⏰ Agendado │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
└───────────────────────────────────────────────────────────┘
```

- Tabela com: Data, Assunto (truncado 40 chars), Destinatários (contagem), Status
- Status: "Enviado" (green check), "Agendado" (clock icon yellow), "Rascunho" (cinza)
- Clicar em uma linha abre detalhe com: conteúdo completo, segmentação, logs de entrega
- Comunicados agendados podem ser editados ou cancelados antes da data

---

## 20.8 ROTAS DO ADMIN

```typescript
// TanStack Router — Admin Routes
/admin                          → Redirect para /admin/dashboard
/admin/dashboard                → Dashboard Financeiro (Visão Geral)
/admin/dashboard/financeiro     → Dashboard detalhado MRR + gráficos
/admin/usuarios                 → Listagem de Usuários
/admin/usuarios/:userId         → Detalhe do Usuário (tabs)
/admin/moderacao                → Visão Geral Moderação
/admin/moderacao/videos         → Fila de Vídeos Denunciados
/admin/moderacao/forum          → Fila de Posts Denunciados
/admin/moderacao/historico      → Histórico de Ações de Moderação
/admin/configuracoes            → Redirect para /admin/configuracoes/termos
/admin/configuracoes/termos     → Editor Termos de Uso / Política
/admin/configuracoes/banners    → Gestão de Banners da Home
/admin/configuracoes/categorias → Categorias Técnicas (31 categorias)
/admin/configuracoes/parametros → Parâmetros Globais
/admin/comunicados              → Redirect para /admin/comunicados/novo
/admin/comunicados/novo         → Formulário Novo Comunicado
/admin/comunicados/historico    → Histórico de Envios
/admin/comunicados/agendados    → Comunicados Agendados
```

### 20.8.1 Middleware de Proteção

```typescript
// Todas as rotas /admin/* devem ter:
beforeLoad: ({ context }) => {
  if (context.auth.user?.role !== 'MASTER_ADMIN') {
    throw redirect({ to: '/' })
  }
}
```

---

## 20.9 COMPONENTES ADMIN

| Componente | Descrição | Props Principais |
|---|---|---|
| `AdminShell` | Layout wrapper com sidebar + topbar admin | `children` |
| `AdminSidebar` | Navegação lateral admin 240px | `activeRoute`, `collapsed` |
| `AdminTopbar` | Barra superior admin 56px | `onToggleSidebar`, `pendingCount` |
| `KpiCard` | Card de métrica com variação | `label`, `value`, `change`, `changeType` |
| `ChartLine` | Gráfico de linha MRR | `data`, `period` |
| `ChartDonut` | Gráfico donut distribuição | `data`, `centerLabel` |
| `UserTable` | Tabela de usuários paginada | `data`, `filters`, `onPageChange` |
| `UserDetail` | Detalhe completo do usuário | `userId` |
| `ModerationQueue` | Fila de itens para moderar | `type: 'video' \| 'post'`, `items` |
| `ModerationAction` | Modal de ação de moderação | `item`, `onApprove`, `onReject` |
| `RichTextEditor` | Editor de texto formatado | `value`, `onChange`, `toolbar` |
| `BannerManager` | Lista drag & drop de banners | `banners`, `onReorder` |
| `CategoryTree` | Accordion de categorias + subcategorias | `categories`, `onEdit` |
| `ParamInput` | Input numérico com label e unidade | `label`, `value`, `unit`, `warning?` |
| `BroadcastForm` | Formulário completo de comunicado | `onSubmit`, `onSaveDraft` |
| `EmailPreview` | Preview renderizado do email | `subject`, `body`, `cta?` |
| `MinScreenGuard` | Bloqueia acesso em telas < 1024px | `minWidth`, `redirectTo` |

---

## 20.10 VALIDAÇÕES ZOD (ADMIN)

```typescript
// Parâmetros Globais
const GlobalParamsSchema = z.object({
  votingWindowDays: z.number().int().min(1).max(90),
  videoLimitSindicoN2: z.number().int().min(1).max(50),
  videoLimitEmpPlus: z.number().int().min(1).max(50),
  videoLimitEmpPro: z.number().int().min(1).max(50),
  quarterlyLockDays: z.number().int().min(30).max(365),
  connectMeLimitBase: z.number().int().min(0).max(20),
  connectMeLimitPg: z.number().int().min(0).max(20),
  couponCooldownHours: z.number().int().min(1).max(48),
  minViewPercentage: z.number().int().min(50).max(100),
})

// Comunicado Broadcast
const BroadcastSchema = z.object({
  targeting: z.discriminatedUnion('type', [
    z.object({ type: z.literal('all') }),
    z.object({ type: z.literal('byProfile'), profiles: z.array(z.string()).min(1) }),
    z.object({ type: z.literal('byPlan'), plans: z.array(z.string()).min(1) }),
  ]),
  subject: z.string().min(5).max(200),
  body: z.string().min(20).max(5000),
  cta: z.object({
    text: z.string().max(30),
    url: z.string().url(),
  }).optional(),
  schedule: z.discriminatedUnion('type', [
    z.object({ type: z.literal('now') }),
    z.object({ type: z.literal('scheduled'), dateTime: z.string().datetime() }),
  ]),
})

// Bloqueio de Usuário
const BlockUserSchema = z.object({
  reason: z.string().min(100).max(500),
  notifyByEmail: z.boolean(),
})

// Remoção de Conteúdo
const RemoveContentSchema = z.object({
  reason: z.enum([
    'content_violation',
    'commercial_content',
    'exposed_contact_data',
    'offensive_content',
    'dangerous_information',
  ]),
  notes: z.string().max(500).optional(),
  suspendAuthor: z.boolean(),
  notifyAuthor: z.boolean(),
})

// Banner
const BannerSchema = z.object({
  title: z.string().max(100).optional(),
  imageUrl: z.string().url(),
  linkTo: z.string().max(500),
  isActive: z.boolean(),
  order: z.number().int().min(0),
})
```

---

## 20.11 ESTADOS E FEEDBACK

### 20.11.1 Loading States
- Dashboard: skeleton dos KPI cards (4 retângulos pulsantes) + skeleton do gráfico (área cinza pulsante)
- Tabela de usuários: skeleton rows (8 linhas com colunas pulsantes)
- Fila de moderação: skeleton cards (3 cards pulsantes)

### 20.11.2 Empty States
- Moderação sem pendências: ilustração ✅ "Nenhum conteúdo pendente de moderação. Tudo limpo!"
- Histórico de comunicados vazio: "Nenhum comunicado enviado ainda."
- Busca de usuários sem resultado: "Nenhum usuário encontrado para os filtros aplicados."

### 20.11.3 Toasts (solid-sonner)
- Salvar parâmetros: toast success "Parâmetros atualizados com sucesso"
- Bloquear usuário: toast warning "Usuário [Nome] bloqueado"
- Remover conteúdo: toast destructive "Conteúdo removido"
- Enviar comunicado: toast success "Comunicado enviado para ~X destinatários"
- Agendar comunicado: toast info "Comunicado agendado para DD/MM/YYYY às HH:MM"
- Erro de salvamento: toast error "Erro ao salvar. Tente novamente."

### 20.11.4 Confirmações Críticas
Toda ação destrutiva (bloquear, remover, enviar broadcast) requer modal de confirmação com:
- Descrição clara da ação
- Dados contextuais (nome do usuário, título do conteúdo, qtd destinatários)
- Aviso "Esta ação não pode ser desfeita" quando aplicável
- Dois botões: "Cancelar" (secondary) e "Confirmar [Ação]" (primary ou destructive)

---

## 20.12 ACESSIBILIDADE (ADMIN)

- Todas as tabelas com `role="table"` e headers semânticos `<th scope="col">`
- Modais com focus trap (Kobalte Dialog)
- Ações com keyboard shortcuts:
  - `Ctrl+S` / `Cmd+S`: Salvar no contexto atual
  - `Escape`: Fechar modal/overlay
- ARIA labels em todos os botões de ação (especialmente ícones sem texto)
- Contraste adequado mantido mesmo em density mode (textos menores devem ter contraste ≥ 4.5:1)
- Skip navigation link para pular sidebar e ir direto ao conteúdo

---

## 20.13 NOTAS DE IMPLEMENTAÇÃO

1. **O Admin Shell é um layout completamente separado** do App Shell principal (doc 02). Quando o usuário navega para `/admin/*`, o layout muda inteiro (sidebar diferente, topbar diferente).

2. **Gráficos**: Recomenda-se `chart.js` pela leveza e boa integração com SolidJS via wrapper reativo. Alternativa: `unovis` para charts mais complexos. Os dados vêm cacheados do backend, não se comunica direto com Mercado Pago.

3. **Rich Text Editor**: Para edição de termos e comunicados, avaliar `tiptap` (headless, integrável) ou solução mais simples com Markdown + preview. O output deve ser HTML sanitizado para emails.

4. **Drag & Drop**: Para banners e reordenação de categorias, usar `@thisbeyond/solid-dnd` ou implementação nativa com HTML5 Drag API.

5. **Paginação**: Todas as listagens (usuários, históricos) usam paginação server-side (cursor-based) via TanStack Query com `keepPreviousData: true` para UX fluida.

6. **Exports CSV**: O botão de exportar no histórico de moderação dispara download de CSV gerado pelo backend (não processa no frontend).

7. **Conforme SOW 4.6**: "Funcionalidades administrativas não estão disponíveis no Mobile." — o `MinScreenGuard` garante isso no frontend, mas o backend também deve rejeitar requests de admin vindos de user-agents mobile como camada extra de segurança.
