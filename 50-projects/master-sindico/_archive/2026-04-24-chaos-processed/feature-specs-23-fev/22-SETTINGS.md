# 22 — CONFIGURAÇÕES DO USUÁRIO (SETTINGS)

> **Sprint 1-2** · Web & Mobile · Todos os perfis autenticados
> Dependências: 00-DESIGN-TOKENS · 02-APP-SHELL · 01-AUTH-ONBOARDING

---

## 22.1 CONTEXTO E REGRAS DE NEGÓCIO

### 22.1.1 Definição
A área de Configurações agrupa todas as opções de personalização, segurança e gerenciamento da conta do usuário. Conforme SOW 1.3: "Configurações de Usuário: Alterar senha, gerenciar notificações (on/off)."

O design segue a referência da Meta (Instagram/Facebook) conforme solicitado pelo cliente: navegação por categorias na lateral esquerda (desktop) ou lista empilhada (mobile), com conteúdo da categoria selecionada à direita.

### 22.1.2 Acesso
- Todas as personas autenticadas têm acesso às configurações.
- Cada persona vê apenas as opções relevantes ao seu perfil/plano.
- Acessível via: avatar dropdown no topbar → "Configurações" ou via sidebar/bottom nav.

---

## 22.2 LAYOUT GERAL

### 22.2.1 Desktop (≥1024px) — Layout Meta-Style

```
┌───────────────────────────────────────────────────────────┐
│  TOPBAR (global, conforme doc 02)                          │
├────────────┬────────────────────────────────────────────────┤
│            │                                                │
│  SIDEBAR   │  SETTINGS CONTENT                              │
│  SETTINGS  │                                                │
│  (280px)   │  ┌──────────────────────────────────────────┐ │
│            │  │  Título da Seção                         │ │
│  ┌────────┐│  │  Descrição breve                         │ │
│  │Conta   ││  ├──────────────────────────────────────────┤ │
│  │        ││  │                                          │ │
│  │Notific.││  │  (Campos e opções da seção ativa)        │ │
│  │        ││  │                                          │ │
│  │Aparênc.││  │                                          │ │
│  │        ││  │                                          │ │
│  │Privac. ││  │                                          │ │
│  │        ││  │                                          │ │
│  │Seguranç││  │                                          │ │
│  │        ││  │                                          │ │
│  │Plano   ││  │                                          │ │
│  └────────┘│  │                                          │ │
│            │  └──────────────────────────────────────────┘ │
│            │                                                │
└────────────┴────────────────────────────────────────────────┘
```

### 22.2.2 Mobile (<768px) — Lista Empilhada

```
Tela 1: Lista de categorias
┌─────────────────────────────┐
│  ← Configurações            │
│                             │
│  ┌─────────────────────────┐│
│  │ 👤 Conta             → ││
│  ├─────────────────────────┤│
│  │ 🔔 Notificações     → ││
│  ├─────────────────────────┤│
│  │ 🎨 Aparência        → ││
│  ├─────────────────────────┤│
│  │ 🔒 Privacidade      → ││
│  ├─────────────────────────┤│
│  │ 🛡️ Segurança        → ││
│  ├─────────────────────────┤│
│  │ 💳 Meu Plano        → ││
│  └─────────────────────────┘│
│                             │
│  ─────────────────────────  │
│  [Sair da Conta]            │
│  (--destructive, text only) │
│                             │
└─────────────────────────────┘

Tela 2: Detalhe da categoria (push navigation)
┌─────────────────────────────┐
│  ← Conta                   │
│                             │
│  (conteúdo da seção)        │
│                             │
└─────────────────────────────┘
```

### 22.2.3 Sidebar de Settings (Desktop)

```
SIDEBAR SETTINGS (280px, background --card, border-right --border)
─────────────────────────────
CONFIGURAÇÕES
Michroma 14px --foreground, letter-spacing 0.05em

─── CATEGORIAS ───

👤  Conta
🔔  Notificações
🎨  Aparência
🔒  Privacidade
🛡️  Segurança
💳  Meu Plano

─── ─── ─── ───

🚪  Sair da Conta
    (--destructive text)
─────────────────────────────
```

- Item ativo: background `--muted`, text `--foreground`, borda esquerda 3px `--primary` (Gold)
- Item inativo: text `--muted-foreground`
- Hover: background `--muted` opacity 0.5
- Ícones: 18px, à esquerda do label
- Gap entre ícone e label: 12px
- Padding vertical por item: 12px
- Font: Manrope 14px

---

## 22.3 SEÇÃO: CONTA

### 22.3.1 Rota
```
/configuracoes/conta
```

### 22.3.2 Layout

```
┌──────────────────────────────────────────────────────┐
│  CONTA                                                │
│  Michroma 16px --foreground                           │
│  "Gerencie suas informações pessoais"                 │
│  Manrope 13px --muted-foreground                      │
│                                                        │
│  ┌────────────────────────────────────────────────┐   │
│  │  FOTO DE PERFIL                                │   │
│  │                                                │   │
│  │  ┌──────┐                                      │   │
│  │  │      │  [Alterar foto]  [Remover]           │   │
│  │  │ 80px │                                      │   │
│  │  │      │  JPG, PNG ou WebP. Máximo 5MB.       │   │
│  │  └──────┘  Recomendado: 400x400px              │   │
│  │                                                │   │
│  └────────────────────────────────────────────────┘   │
│                                                        │
│  ┌────────────────────────────────────────────────┐   │
│  │  INFORMAÇÕES PESSOAIS                          │   │
│  │                                                │   │
│  │  Nome completo                                 │   │
│  │  ┌────────────────────────────────────────┐    │   │
│  │  │ João Silva da Costa                    │    │   │
│  │  └────────────────────────────────────────┘    │   │
│  │                                                │   │
│  │  Email                                         │   │
│  │  ┌────────────────────────────────────────┐    │   │
│  │  │ joao.silva@email.com       🔒 verific. │   │   │
│  │  └────────────────────────────────────────┘    │   │
│  │  ⚠️ Alteração de email requer nova validação   │   │
│  │                                                │   │
│  │  Telefone                                      │   │
│  │  ┌────────────────────────────────────────┐    │   │
│  │  │ (11) 99999-0000                        │    │   │
│  │  └────────────────────────────────────────┘    │   │
│  │                                                │   │
│  │  Bio (opcional)                                │   │
│  │  ┌────────────────────────────────────────┐    │   │
│  │  │ "Síndico profissional há 5 anos..."    │    │   │
│  │  │                                        │    │   │
│  │  │ (max 300 chars)                        │    │   │
│  │  └────────────────────────────────────────┘    │   │
│  │  189/300                                       │   │
│  │                                                │   │
│  │  [Salvar Alterações]                           │   │
│  │  (--primary Gold, disabled se nada mudou)      │   │
│  │                                                │   │
│  └────────────────────────────────────────────────┘   │
│                                                        │
│  ┌────────────────────────────────────────────────┐   │
│  │  DADOS ESPECÍFICOS DO PERFIL                   │   │
│  │  (varia conforme persona)                      │   │
│  │                                                │   │
│  │  [Para Síndico]:                               │   │
│  │  · Condomínios vinculados (read-only, lista)   │   │
│  │  · Status: Ouro ⭐                             │   │
│  │                                                │   │
│  │  [Para Empresa]:                               │   │
│  │  · CNPJ (read-only)                            │   │
│  │  · Razão Social                                │   │
│  │  · Categorias técnicas (até 5) [Editar]        │   │
│  │                                                │   │
│  │  [Para Morador]:                               │   │
│  │  · Unidade vinculada (read-only)               │   │
│  │  · Condomínio (read-only)                      │   │
│  │                                                │   │
│  └────────────────────────────────────────────────┘   │
│                                                        │
│  ┌────────────────────────────────────────────────┐   │
│  │  ZONA DE PERIGO                                │   │
│  │  (border --destructive, background sutil red)  │   │
│  │                                                │   │
│  │  Desativar conta                               │   │
│  │  Sua conta será desativada e seu perfil não    │   │
│  │  será mais visível. Você pode reativá-la a     │   │
│  │  qualquer momento fazendo login novamente.     │   │
│  │                                                │   │
│  │  [Desativar Minha Conta]                       │   │
│  │  (--destructive outline, not filled)           │   │
│  │                                                │   │
│  └────────────────────────────────────────────────┘   │
│                                                        │
└──────────────────────────────────────────────────────┘
```

**Cards/Seções:**
- Cada grupo de campos é envolvido em card com background `--card`, border `--border`, border-radius `--radius`, padding 24px
- Título da seção dentro do card: Manrope 14px semibold `--foreground`
- Gap entre cards: 24px

**Upload de Foto:**
- Avatar circular 80px com preview atual
- Botão "Alterar foto": secondary, abre file picker (aceita JPG/PNG/WebP, max 5MB)
- Botão "Remover": text-only `--destructive`
- Crop/resize: feito no backend após upload, ou usar biblioteca como `solid-image-crop` se necessário

**Zona de Perigo:**
- Border: 1px `--destructive`
- Background: `--destructive` opacity 0.05
- Título: Manrope 14px semibold `--destructive`
- Botão: outline variant `--destructive` (borda vermelha, texto vermelho, fundo transparente)
- Ação abre modal de confirmação com input de senha

---

## 22.4 SEÇÃO: NOTIFICAÇÕES

### 22.4.1 Rota
```
/configuracoes/notificacoes
```

### 22.4.2 Layout

```
┌──────────────────────────────────────────────────────┐
│  NOTIFICAÇÕES                                         │
│  "Escolha como e quando ser notificado"               │
│                                                        │
│  ┌────────────────────────────────────────────────┐   │
│  │  COMUNICADOS DA PLATAFORMA                     │   │
│  │                                                │   │
│  │  Comunicados oficiais do Master Síndico        │   │
│  │  (não é possível desativar)                    │   │
│  │  Email: ✅ Sempre ativo    Push: ✅ Sempre ativo│   │
│  │                                                │   │
│  └────────────────────────────────────────────────┘   │
│                                                        │
│  ┌────────────────────────────────────────────────┐   │
│  │  CONTA E SEGURANÇA                             │   │
│  │                                                │   │
│  │  Confirmação de email        [────●] on        │   │
│  │  Redefinição de senha        [────●] on        │   │
│  │  Novo login detectado        [────●] on        │   │
│  │  (não é possível desativar)                    │   │
│  │                                                │   │
│  └────────────────────────────────────────────────┘   │
│                                                        │
│  ┌────────────────────────────────────────────────┐   │
│  │  ASSEMBLEIAS E EVENTOS (Morador/Síndico)       │   │
│  │                                                │   │
│  │  Nova assembleia publicada   [────●] on        │   │
│  │  Votação aberta              [────●] on        │   │
│  │  Resultado de votação        [────●] on        │   │
│  │  Avaliação disponível        [────●] on        │   │
│  │                                                │   │
│  └────────────────────────────────────────────────┘   │
│                                                        │
│  ┌────────────────────────────────────────────────┐   │
│  │  CONTEÚDO E CURSOS (Síndico/Empresa)           │   │
│  │                                                │   │
│  │  Novo módulo disponível      [●────] off       │   │
│  │  Certificado gerado          [────●] on        │   │
│  │  Live ao vivo agora          [────●] on        │   │
│  │                                                │   │
│  └────────────────────────────────────────────────┘   │
│                                                        │
│  ┌────────────────────────────────────────────────┐   │
│  │  OPORTUNIDADES (Morador Pagante)               │   │
│  │                                                │   │
│  │  Empresa demonstrou interesse [────●] on       │   │
│  │                                                │   │
│  └────────────────────────────────────────────────┘   │
│                                                        │
│  ┌────────────────────────────────────────────────┐   │
│  │  CUPONS E BENEFÍCIOS                           │   │
│  │                                                │   │
│  │  Promoção exclusiva do cond. [────●] on        │   │
│  │  Cupom expirado              [●────] off       │   │
│  │                                                │   │
│  └────────────────────────────────────────────────┘   │
│                                                        │
│  Alterações salvas automaticamente.                    │
│  Manrope 12px --muted-foreground                       │
│                                                        │
└──────────────────────────────────────────────────────┘
```

**Toggle Switch (Kobalte Switch):**
- Track: 40px × 22px, border-radius 11px
- On: background `--primary` (Gold), thumb branco à direita
- Off: background `--muted`, thumb `--muted-foreground` à esquerda
- Transição: 200ms ease
- Disabled (obrigatórios): opacity 0.7, cursor not-allowed, tooltip "Esta notificação não pode ser desativada"

**Regras de visibilidade das seções:**
- "Assembleias e Eventos": visível para Moradores e Síndicos
- "Conteúdo e Cursos": visível para Síndicos e Empresas com acesso a cursos
- "Oportunidades": visível apenas para Morador Pagante
- "Cupons e Benefícios": visível para todos que têm acesso ao Clube de Benefícios

**Salvamento automático:**
- Cada toggle salva imediatamente via mutation (TanStack Query)
- Sem botão "Salvar" — feedback via micro-animação do toggle + texto "Salvo" breve

---

## 22.5 SEÇÃO: APARÊNCIA

### 22.5.1 Rota
```
/configuracoes/aparencia
```

### 22.5.2 Layout

```
┌──────────────────────────────────────────────────────┐
│  APARÊNCIA                                            │
│  "Personalize a interface ao seu gosto"               │
│                                                        │
│  ┌────────────────────────────────────────────────┐   │
│  │  TEMA                                          │   │
│  │                                                │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐       │   │
│  │  │  ☀️ Claro │ │  🌙 Escuro│ │  💻 Sistema│      │   │
│  │  │          │ │          │ │          │       │   │
│  │  │ ┌──────┐ │ │ ┌──────┐ │ │ ┌──────┐ │       │   │
│  │  │ │ Mini │ │ │ │ Mini │ │ │ │ Mini │ │       │   │
│  │  │ │previe│ │ │ │previe│ │ │ │previe│ │       │   │
│  │  │ │ w    │ │ │ │ w    │ │ │ │ w    │ │       │   │
│  │  │ └──────┘ │ │ └──────┘ │ │ └──────┘ │       │   │
│  │  │  ● ativo │ │  ○       │ │  ○       │       │   │
│  │  └──────────┘ └──────────┘ └──────────┘       │   │
│  │                                                │   │
│  └────────────────────────────────────────────────┘   │
│                                                        │
│  ┌────────────────────────────────────────────────┐   │
│  │  IDIOMA (futuro)                               │   │
│  │                                                │   │
│  │  🌐 Português (Brasil)              [▾]       │   │
│  │                                                │   │
│  │  ℹ️ Outros idiomas estarão disponíveis         │   │
│  │  em breve.                                     │   │
│  │                                                │   │
│  └────────────────────────────────────────────────┘   │
│                                                        │
└──────────────────────────────────────────────────────┘
```

**Seletor de Tema:**
- 3 cards selecionáveis lado a lado
- Cada card: 160px × 180px, border `--border`, border-radius `--radius`
- Card ativo: border 2px `--primary` (Gold), radio dot preenchido
- Card inativo: border 1px `--border`, radio dot vazio
- Mini preview: representação miniatura do tema (fundo claro/escuro/split)
- Integração: Kobalte `ColorModeProvider` (já no `src/index.tsx`)
- Persistência: `localStorage` via Kobalte (já implementado)
- Transição de tema: `transition: background-color 200ms ease, color 200ms ease`

**Idioma (preparado para futuro):**
- Select desabilitado com "Português (Brasil)" como única opção
- Info icon com tooltip: "Outros idiomas em breve"

---

## 22.6 SEÇÃO: PRIVACIDADE

### 22.6.1 Rota
```
/configuracoes/privacidade
```

### 22.6.2 Layout

```
┌──────────────────────────────────────────────────────┐
│  PRIVACIDADE                                          │
│  "Controle quem pode ver suas informações"            │
│                                                        │
│  ┌────────────────────────────────────────────────┐   │
│  │  VISIBILIDADE DO PERFIL                        │   │
│  │                                                │   │
│  │  Perfil visível na busca      [────●] on       │   │
│  │  Manrope 12px: "Quando ativo, seu perfil       │   │
│  │  aparece nos resultados de busca conforme      │   │
│  │  seu plano permite."                           │   │
│  │                                                │   │
│  │  Mostrar bio no perfil        [────●] on       │   │
│  │                                                │   │
│  │  Mostrar região no perfil     [────●] on       │   │
│  │                                                │   │
│  └────────────────────────────────────────────────┘   │
│                                                        │
│  ┌────────────────────────────────────────────────┐   │
│  │  VÍDEO-CURRÍCULO (Morador Pagante)             │   │
│  │                                                │   │
│  │  Currículo visível para       [────●] on       │   │
│  │  empresas                                      │   │
│  │  Manrope 12px: "Quando desativado, seu         │   │
│  │  vídeo-currículo não será exibido no           │   │
│  │  Banco de Talentos."                           │   │
│  │                                                │   │
│  └────────────────────────────────────────────────┘   │
│                                                        │
│  ┌────────────────────────────────────────────────┐   │
│  │  DADOS PESSOAIS                                │   │
│  │                                                │   │
│  │  [Solicitar meus dados]                        │   │
│  │  Receba uma cópia de todos os dados            │   │
│  │  armazenados sobre você na plataforma.         │   │
│  │  (LGPD — Art. 18)                              │   │
│  │                                                │   │
│  │  [Solicitar exclusão de dados]                 │   │
│  │  Solicite a remoção completa dos seus dados.   │   │
│  │  (--destructive outline)                       │   │
│  │  ⚠️ Esta ação é irreversível e resultará       │   │
│  │  na exclusão permanente da sua conta.          │   │
│  │                                                │   │
│  └────────────────────────────────────────────────┘   │
│                                                        │
│  Alterações salvas automaticamente.                    │
│                                                        │
└──────────────────────────────────────────────────────┘
```

**Toggles de Privacidade:**
- Mesma especificação visual dos toggles de Notificações
- Cada toggle tem descrição auxiliar em Manrope 12px `--muted-foreground` abaixo

**LGPD:**
- "Solicitar meus dados": botão secondary, dispara request ao backend que gera export em JSON/PDF e envia por email
- "Solicitar exclusão": botão outline `--destructive`, abre modal com aviso severo + input de senha para confirmar
- Texto LGPD: Manrope 12px `--muted-foreground`, referência ao artigo da lei

---

## 22.7 SEÇÃO: SEGURANÇA

### 22.7.1 Rota
```
/configuracoes/seguranca
```

### 22.7.2 Layout

```
┌──────────────────────────────────────────────────────┐
│  SEGURANÇA                                            │
│  "Proteja sua conta"                                  │
│                                                        │
│  ┌────────────────────────────────────────────────┐   │
│  │  ALTERAR SENHA                                 │   │
│  │                                                │   │
│  │  Senha atual *                                 │   │
│  │  ┌──────────────────────────────┐              │   │
│  │  │ ●●●●●●●●             [👁️]   │              │   │
│  │  └──────────────────────────────┘              │   │
│  │                                                │   │
│  │  Nova senha *                                  │   │
│  │  ┌──────────────────────────────┐              │   │
│  │  │ ●●●●●●●●●●●●         [👁️]   │              │   │
│  │  └──────────────────────────────┘              │   │
│  │  Força: ████████░░ Forte                       │   │
│  │  ✅ Mínimo 8 caracteres                        │   │
│  │  ✅ Uma letra maiúscula                         │   │
│  │  ✅ Um número                                   │   │
│  │  ❌ Um caractere especial                       │   │
│  │                                                │   │
│  │  Confirmar nova senha *                        │   │
│  │  ┌──────────────────────────────┐              │   │
│  │  │ ●●●●●●●●●●●●         [👁️]   │              │   │
│  │  └──────────────────────────────┘              │   │
│  │                                                │   │
│  │  [Alterar Senha]                               │   │
│  │  (--primary Gold)                              │   │
│  │                                                │   │
│  └────────────────────────────────────────────────┘   │
│                                                        │
│  ┌────────────────────────────────────────────────┐   │
│  │  SESSÕES ATIVAS                                │   │
│  │                                                │   │
│  │  🖥️ Chrome · Windows 11                        │   │
│  │  São Paulo, SP · Agora (esta sessão)           │   │
│  │                                                │   │
│  │  📱 Safari · iPhone 15                         │   │
│  │  São Paulo, SP · Há 3 horas                    │   │
│  │  [Encerrar sessão]                             │   │
│  │                                                │   │
│  │  ───────────────────────────                   │   │
│  │                                                │   │
│  │  [Encerrar todas as outras sessões]            │   │
│  │  (--destructive outline)                       │   │
│  │                                                │   │
│  └────────────────────────────────────────────────┘   │
│                                                        │
│  ┌────────────────────────────────────────────────┐   │
│  │  DISPOSITIVO ÚNICO                             │   │
│  │                                                │   │
│  │  ℹ️ Sua conta está limitada a 1 dispositivo     │   │
│  │  ativo por vez (conforme regra da plataforma). │   │
│  │                                                │   │
│  │  Dispositivo atual: Chrome · Windows 11        │   │
│  │  IP: 189.xxx.xxx.xxx                           │   │
│  │                                                │   │
│  └────────────────────────────────────────────────┘   │
│                                                        │
└──────────────────────────────────────────────────────┘
```

**Indicador de Força da Senha:**
- Barra horizontal: 4 segmentos
- Fraca (1/4): `--destructive` (vermelho)
- Média (2/4): `--warning` (amarelo)
- Forte (3/4): oklch(0.6 0.15 149) (verde médio)
- Muito forte (4/4): `--success` (verde pleno)
- Critérios listados abaixo com ✅ (atendido, verde) ou ❌ (pendente, vermelho)

**Toggle de visibilidade da senha (👁️):**
- Ícone olho à direita do input
- Toggle: `type="password"` ↔ `type="text"`
- Ícone muda para olho cortado quando visível

**Sessões Ativas:**
- Lista de sessões com: ícone dispositivo (🖥️ desktop / 📱 mobile), browser + OS, localização aproximada, tempo
- Sessão atual: badge "Esta sessão" em `--success`
- Botão "Encerrar sessão": text-only `--destructive` por sessão individual
- Botão "Encerrar todas": outline `--destructive`, com confirmação

**Dispositivo Único (Conforme Matriz Funcional):**
- Box informativo com background `--muted`, border-left 3px `--primary`
- Exibe dispositivo atual e IP (mascarado parcialmente)
- Não é editável — apenas informativo

---

## 22.8 SEÇÃO: MEU PLANO

### 22.8.1 Rota
```
/configuracoes/plano
```

### 22.8.2 Layout

```
┌──────────────────────────────────────────────────────┐
│  MEU PLANO                                            │
│  "Gerencie sua assinatura"                            │
│                                                        │
│  ┌────────────────────────────────────────────────┐   │
│  │  PLANO ATUAL                                   │   │
│  │                                                │   │
│  │  ┌──────────────────────────────────────────┐  │   │
│  │  │                                          │  │   │
│  │  │  🏆 SÍNDICO PRO (N3)                    │  │   │
│  │  │  Michroma 20px --primary (Gold)          │  │   │
│  │  │                                          │  │   │
│  │  │  Status: Ativo                           │  │   │
│  │  │  Desde: 15/03/2025                       │  │   │
│  │  │  Próxima cobrança: 15/03/2026            │  │   │
│  │  │  Valor: R$ XX,XX / mês                   │  │   │
│  │  │                                          │  │   │
│  │  └──────────────────────────────────────────┘  │   │
│  │                                                │   │
│  └────────────────────────────────────────────────┘   │
│                                                        │
│  ┌────────────────────────────────────────────────┐   │
│  │  O QUE SEU PLANO INCLUI                        │   │
│  │                                                │   │
│  │  ✅ Perfil institucional público                │   │
│  │  ✅ Publicação de vídeos (4/mês)               │   │
│  │  ✅ Connect Me ativo                            │   │
│  │  ✅ Cursos com certificado                      │   │
│  │  ✅ Fórum e Lives                               │   │
│  │  ✅ Gestão completa (assembleias, avaliação,   │   │
│  │     plano de reeleição, exportação PDF)         │   │
│  │  ✅ Link temporário de eventos                  │   │
│  │                                                │   │
│  └────────────────────────────────────────────────┘   │
│                                                        │
│  ┌────────────────────────────────────────────────┐   │
│  │  HISTÓRICO DE PAGAMENTOS                       │   │
│  │                                                │   │
│  │  15/02/2026  R$ XX,XX  ✅ Pago                 │   │
│  │  15/01/2026  R$ XX,XX  ✅ Pago                 │   │
│  │  15/12/2025  R$ XX,XX  ✅ Pago                 │   │
│  │                                                │   │
│  │  [Ver todos os pagamentos]                     │   │
│  └────────────────────────────────────────────────┘   │
│                                                        │
│  ┌────────────────────────────────────────────────┐   │
│  │  GERENCIAR ASSINATURA                          │   │
│  │                                                │   │
│  │  [Alterar Plano]  (secondary)                  │   │
│  │  Redireciona para página de planos/upgrade     │   │
│  │                                                │   │
│  │  [Cancelar Assinatura]  (--destructive outline)│   │
│  │  ⚠️ Ao cancelar, seu plano será rebaixado       │   │
│  │  para Base ao final do período pago.           │   │
│  │                                                │   │
│  └────────────────────────────────────────────────┘   │
│                                                        │
└──────────────────────────────────────────────────────┘
```

**Card do Plano Atual:**
- Destaque visual: borda `--primary` (Gold) 2px, ou gradiente sutil gold no topo do card
- Ícone do plano: 🏆 para Pro, ⭐ para Plus, etc.
- Nome do plano: Michroma 20px `--primary` (Gold)
- Status, datas e valor: Manrope 14px `--foreground`

**Lista de funcionalidades:**
- Checkmarks: `--success` (verde)
- Texto: Manrope 14px `--foreground`
- Lista simples sem bullets, apenas ✅ prefix

**Histórico de Pagamentos:**
- Mini tabela: Data, Valor, Status
- Status "Pago": badge green
- Status "Pendente": badge yellow
- Status "Falhou": badge red
- Link "Ver todos": expande ou navega para lista completa

**Cancelamento:**
- Modal de confirmação com retenção:
  - "Tem certeza que deseja cancelar?"
  - Lista do que será perdido ao cancelar
  - Botão "Manter meu plano" (primary, destaque)
  - Botão "Cancelar assinatura" (text-only destructive, discreto)

---

## 22.9 ROTAS COMPLETAS

```typescript
/configuracoes                  → Redirect para /configuracoes/conta
/configuracoes/conta            → Informações pessoais, foto, dados do perfil
/configuracoes/notificacoes     → Toggles de notificação por categoria
/configuracoes/aparencia        → Tema (claro/escuro/sistema), idioma futuro
/configuracoes/privacidade      → Visibilidade, currículo, LGPD
/configuracoes/seguranca        → Alterar senha, sessões, dispositivo único
/configuracoes/plano            → Plano atual, funcionalidades, pagamentos, cancelar
```

---

## 22.10 COMPONENTES

| Componente | Descrição | Props |
|---|---|---|
| `SettingsShell` | Layout com sidebar de categorias | `activeSection`, `children` |
| `SettingsSidebar` | Lista de categorias (desktop) | `items`, `active` |
| `SettingsSection` | Card wrapper para cada grupo | `title`, `description?`, `children` |
| `ToggleRow` | Linha com label + descrição + toggle switch | `label`, `description?`, `value`, `onChange`, `disabled?` |
| `ThemeSelector` | 3 cards selecionáveis de tema | `value`, `onChange` |
| `PasswordStrength` | Indicador de força + critérios | `password` |
| `SessionCard` | Linha de sessão ativa | `session`, `isCurrent`, `onRevoke` |
| `PlanCard` | Card do plano atual com destaque | `plan`, `status`, `since`, `nextBilling` |
| `PaymentHistory` | Mini-tabela de pagamentos | `payments`, `onViewAll` |
| `DangerZone` | Wrapper vermelho para ações destrutivas | `children` |
| `PhotoUpload` | Avatar com botões alterar/remover | `currentUrl`, `onUpload`, `onRemove` |
| `CancellationModal` | Modal de retenção para cancelamento | `planName`, `onKeep`, `onCancel` |

---

## 22.11 VALIDAÇÕES ZOD

```typescript
// Informações da Conta
const AccountSchema = z.object({
  fullName: z.string().min(3).max(100),
  email: z.string().email(),
  phone: z.string().optional(),
  bio: z.string().max(300).optional(),
})

// Alterar Senha
const ChangePasswordSchema = z.object({
  currentPassword: z.string().min(1, 'Senha atual obrigatória'),
  newPassword: z.string()
    .min(8, 'Mínimo 8 caracteres')
    .regex(/[A-Z]/, 'Uma letra maiúscula')
    .regex(/[0-9]/, 'Um número')
    .regex(/[^A-Za-z0-9]/, 'Um caractere especial'),
  confirmPassword: z.string(),
}).refine(d => d.newPassword === d.confirmPassword, {
  message: 'Senhas não coincidem',
  path: ['confirmPassword'],
})

// Categorias da Empresa (edição via settings)
const CompanyCategoriesSchema = z.object({
  categories: z.array(z.string()).min(1).max(5),
})
```

---

## 22.12 ESTADOS E FEEDBACK

### Toasts (solid-sonner)
- Salvar conta: toast success "Dados atualizados com sucesso"
- Alterar senha: toast success "Senha alterada com sucesso"
- Alterar foto: toast success "Foto de perfil atualizada"
- Encerrar sessão: toast info "Sessão encerrada"
- Toggle notificação: sem toast (feedback visual inline)
- Erro de salvamento: toast error "Erro ao salvar. Tente novamente."
- Cancelar assinatura: toast warning "Sua assinatura será encerrada em DD/MM/YYYY"

### Loading
- Formulários: botão "Salvar" mostra spinner inline e texto "Salvando..."
- Sessões: skeleton list (3 linhas pulsantes)
- Plano: skeleton card + skeleton lista

---

## 22.13 ACESSIBILIDADE

- Todos os toggles: `role="switch"`, `aria-checked`, `aria-label` descritivo
- Navegação por keyboard: Tab entre categorias na sidebar, Enter para selecionar
- Formulários: labels `htmlFor` associados, erros com `aria-describedby`
- Zona de perigo: `aria-live="polite"` no aviso
- Modal de cancelamento: focus trap, `aria-modal="true"`
- Contraste em toggles: estado on/off claramente distinguível sem depender apenas de cor
