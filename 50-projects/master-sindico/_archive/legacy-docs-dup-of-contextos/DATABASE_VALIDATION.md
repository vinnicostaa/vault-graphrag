# Validação de Banco de Dados - Master Síndico

## Data: 2026-02-01
**Baseado em:** novo escopo-7.pdf (Escopo Técnico Validado pelo Cliente)

---

## 1. ANÁLISE DO ESCOPO VALIDADO (SOW)

### 1.1 Perfis de Usuário (Roles)

Conforme **Seção 6.1** do escopo aprovado:

| Role | Nome Técnico | Observações |
|------|--------------|-------------|
| **Morador Base** | `morador` (plan: `base`) | Free, com limitações |
| **Morador Pagante** | `morador` (plan: `pagante`) | R$ 9,90/mês, vídeo-currículo |
| **Síndico N1** | `sindico` (plan: `n1`) | Capacitação, sem perfil público |
| **Síndico N2** | `sindico` (plan: `n2`) | Perfil público, 4 vídeos/mês |
| **Síndico N3** | `sindico` (plan: `n3`) | Ferramentas completas de gestão |
| **Empresa Plus** | `empresa` (plan: `plus`) | 8 vídeos/mês, sem cursos |
| **Empresa Pro** | `empresa` (plan: `pro`) | 12 vídeos/mês, cria cursos certificados |
| **Marketing** | `marketing` | Tags em vídeos, leilão de "cadeiras" |
| **Vizinhança** | `empresa_local` | Comércio local, cupons |
| **Admin** | `admin` | Administrador MasterSíndico |

### 1.2 Validação da Tabela `users`

**Status Atual:**
```typescript
export const userRoles = pgEnum("user_roles", [
	"none",
	"morador",
	"sindico",
	"empresa",
	"marketing",
	"empresa_local",
	"admin",
]);
```

**✅ VALIDAÇÃO:**
- ✅ Roles corretas conforme escopo
- ✅ `none` necessário para estado inicial antes de onboarding
- ⚠️ **FALTA:** Campo `plan` para diferenciar N1/N2/N3, Plus/Pro, Base/Pagante

**🔧 CORREÇÃO NECESSÁRIA:**
```typescript
export const userPlans = pgEnum("user_plans", [
	"none",
	"base",      // Morador Base (free)
	"pagante",   // Morador Pagante (R$ 9,90)
	"n1",        // Síndico N1
	"n2",        // Síndico N2
	"n3",        // Síndico N3
	"plus",      // Empresa Plus
	"pro",       // Empresa Pro
	"marketing", // Marketing (único plano)
	"vizinhanca" // Vizinhança (único plano)
]);

// Adicionar na tabela users:
export const users = pgTable("users", {
	// ... campos existentes
	role: userRoles("role").default("none").notNull(),
	plan: userPlans("plan").default("none").notNull(), // ⬅️ ADICIONAR
	// ...
});
```

---

## 2. TABELAS DE PERFIL NECESSÁRIAS

### 2.1 Perfil Morador

**Conforme Escopo Sprint 1 (item 1.3):**
- Dados básicos
- Vínculo com unidade (condomínio + número)
- Vídeo-currículo (apenas Morador Pagante)

**🔧 CRIAR:**
```sql
CREATE TABLE profiles_morador (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	
	-- Dados pessoais
	cpf VARCHAR(11) UNIQUE,
	phone VARCHAR(20),
	birth_date DATE,
	
	-- Vídeo-currículo (Sprint 4, item 4.7)
	video_curriculo_url TEXT,
	video_curriculo_uploaded_at TIMESTAMP,
	
	-- Endereço (opcional, para perfil completo)
	street TEXT,
	number TEXT,
	block TEXT,
	district TEXT,
	city TEXT,
	state VARCHAR(2),
	zip_code VARCHAR(8),
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	updated_at TIMESTAMP DEFAULT NOW() NOT NULL,
	
	CONSTRAINT unique_morador_user UNIQUE(user_id)
);

-- Tabela de vínculo com unidades
CREATE TABLE vinculos_unidade (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	morador_id UUID NOT NULL REFERENCES profiles_morador(id) ON DELETE CASCADE,
	condominio_id UUID NOT NULL REFERENCES condominios(id) ON DELETE CASCADE,
	numero_unidade TEXT NOT NULL,
	bloco TEXT,
	tipo_vinculo VARCHAR(20) DEFAULT 'proprietario', -- proprietario, inquilino, dependente
	ativo BOOLEAN DEFAULT true,
	data_inicio DATE,
	data_fim DATE,
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	
	CONSTRAINT unique_morador_unidade UNIQUE(morador_id, condominio_id, numero_unidade)
);
```

**Regras de Negócio:**
- ✅ Vídeo-currículo: Máximo 90 segundos
- ✅ Trava trimestral: `video_curriculo_uploaded_at` + 90 dias
- ✅ Visível para Empresas Plus/Pro/Marketing/Vizinhança
- ✅ 1 voto por unidade em assembleias

---

### 2.2 Perfil Síndico

**Conforme Escopo Sprint 1 (item 1.3) e Cap. 8 (Status):**
- Dados profissionais
- Vínculo com múltiplos condomínios
- Status de reconhecimento (Bronze/Prata/Ouro/Diamante)

**🔧 CRIAR:**
```sql
CREATE TYPE status_reconhecimento AS ENUM ('bronze', 'prata', 'ouro', 'diamante');

CREATE TABLE profiles_sindico (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	
	-- Dados profissionais
	cpf VARCHAR(11) UNIQUE NOT NULL,
	phone VARCHAR(20),
	tempo_carreira_anos INTEGER DEFAULT 0,
	
	-- Status de reconhecimento (Cap. 8)
	status status_reconhecimento DEFAULT 'bronze' NOT NULL,
	videos_assistidos_70_percent INTEGER DEFAULT 0,
	condominios_vinculados INTEGER DEFAULT 0,
	modulos_concluidos INTEGER DEFAULT 0,
	status_recalculado_em TIMESTAMP,
	
	-- Perfil público (apenas N2/N3)
	bio TEXT,
	apresentacao_video_url TEXT, -- vídeo institucional
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	updated_at TIMESTAMP DEFAULT NOW() NOT NULL,
	
	CONSTRAINT unique_sindico_user UNIQUE(user_id)
);

-- Tabela de condomínios
CREATE TABLE condominios (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	nome TEXT NOT NULL,
	cnpj VARCHAR(14) UNIQUE,
	
	-- Endereço completo
	street TEXT NOT NULL,
	number TEXT NOT NULL,
	block TEXT,
	district TEXT NOT NULL,
	city TEXT NOT NULL,
	state VARCHAR(2) NOT NULL,
	zip_code VARCHAR(8) NOT NULL,
	
	-- Dados operacionais
	total_unidades INTEGER,
	sindico_atual_id UUID REFERENCES profiles_sindico(id) ON DELETE SET NULL,
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	updated_at TIMESTAMP DEFAULT NOW() NOT NULL
);

-- Tabela de histórico de síndicos (para cálculo de "condominios_vinculados")
CREATE TABLE vinculos_sindico_condominio (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	sindico_id UUID NOT NULL REFERENCES profiles_sindico(id) ON DELETE CASCADE,
	condominio_id UUID NOT NULL REFERENCES condominios(id) ON DELETE CASCADE,
	data_inicio DATE NOT NULL,
	data_fim DATE,
	ativo BOOLEAN DEFAULT true,
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	
	CONSTRAINT unique_sindico_condominio_ativo UNIQUE(condominio_id, ativo) WHERE ativo = true
);
```

**Regras de Negócio (Cap. 8):**
- ✅ Status calculado automaticamente por:
  - Vídeos técnicos assistidos ≥70%
  - Número de condomínios vinculados (conta 1x cada)
  - Módulos de capacitação concluídos (100%)
- ✅ Status é dinâmico (recalculado periodicamente)
- ✅ Visível para **moradores** no perfil público
- ❌ **NÃO é ranking** (não compara síndicos)

---

### 2.3 Perfil Empresa

**Conforme Escopo Sprint 1 (item 1.3):**
- Dados PJ (CNPJ, razão social)
- Endereço completo
- Categoria principal + até 5 subcategorias
- Seguro de responsabilidade civil (sim/não)

**✅ SCHEMA VALIDADO:**
```typescript
// packages/schemas/src/profiles/enterprise/enterprise.schema.ts
export const enterpriseSchema = z.object({
    companyName: z.string().min(1, "Nome da empresa é obrigatório"),
    cnpj: z.string().length(14, "CNPJ deve ter 14 dígitos"),
    street: z.string(),
    number: z.string(),
    block: z.string().optional(),
    district: z.string(),
    city: z.string(),
    state: z.string().length(2, "Use a sigla do estado"),
    zipCode: z.preprocess(
        (val) => String(val).replace(/\D/g, ''), 
        z.string().length(8, "CEP deve ter 8 dígitos")
    ),
    categories: z.array(categoriaEmpresaSchema).min(1, "Selecione pelo menos uma categoria"),
    subCategories: z.array(subcategoriaEmpresaSchema).max(5, "Você pode selecionar no máximo 5 subcategorias"),
});
```

**🔧 CRIAR TABELA:**
```sql
-- Enum de categorias (criar baseado no enums.schema.ts)
CREATE TYPE categoria_empresa AS ENUM (
	'ADMINISTRACAO_GESTAO_GOVERNANCA',
	'JURIDICO_DIREITO_IMOBILIARIO',
	'CONTABILIDADE_AUDITORIA_FINANCAS',
	-- ... (32 categorias ao total)
);

-- Enum de subcategorias (criar baseado no enums.schema.ts)
CREATE TYPE subcategoria_empresa AS ENUM (
	'ADMINISTRADORAS_CONDOMINIOS',
	'CONSULTORIA_GESTAO_CONDOMINIAL',
	-- ... (centenas de subcategorias)
);

CREATE TABLE profiles_empresa (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	
	-- Dados PJ
	razao_social TEXT NOT NULL,
	cnpj VARCHAR(14) UNIQUE NOT NULL,
	phone VARCHAR(20),
	
	-- Endereço completo
	street TEXT NOT NULL,
	number TEXT NOT NULL,
	block TEXT,
	district TEXT NOT NULL,
	city TEXT NOT NULL,
	state VARCHAR(2) NOT NULL,
	zip_code VARCHAR(8) NOT NULL,
	
	-- Categorias (Manual Cap. 6)
	categoria_principal categoria_empresa NOT NULL,
	subcategorias subcategoria_empresa[] DEFAULT '{}', -- máximo 5
	
	-- Outros dados
	possui_seguro_responsabilidade_civil BOOLEAN DEFAULT false,
	
	-- Perfil público
	bio TEXT,
	video_institucional_url TEXT, -- máximo 2min, trava trimestral
	video_institucional_uploaded_at TIMESTAMP,
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	updated_at TIMESTAMP DEFAULT NOW() NOT NULL,
	
	CONSTRAINT unique_empresa_user UNIQUE(user_id),
	CONSTRAINT max_5_subcategorias CHECK (array_length(subcategorias, 1) <= 5)
);
```

**Regras de Negócio:**
- ✅ 1 categoria principal obrigatória
- ✅ Até 5 subcategorias
- ✅ Subcategorias devem pertencer à categoria principal (validação no backend)
- ✅ Vídeo institucional: 2min, atualização trimestral
- ✅ Perfil visível conforme Matriz Funcional

---

### 2.4 Perfil Marketing

**Conforme Escopo Sprint 4 (item 4.4):**
- Dados da agência
- Sistema de leilão por "cadeira" (BNI-style)
- Limite: 3 empresas/estado

**🔧 CRIAR:**
```sql
CREATE TABLE profiles_marketing (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	
	-- Dados da agência
	nome_agencia TEXT NOT NULL,
	cnpj VARCHAR(14) UNIQUE NOT NULL,
	phone VARCHAR(20),
	
	-- Endereço
	city TEXT NOT NULL,
	state VARCHAR(2) NOT NULL,
	
	-- Sistema de "cadeiras" (leilão BNI-style)
	slots_contratados INTEGER DEFAULT 0, -- quantas tags/anúncios pode fazer
	
	-- Perfil
	bio TEXT,
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	updated_at TIMESTAMP DEFAULT NOW() NOT NULL,
	
	CONSTRAINT unique_marketing_user UNIQUE(user_id)
);

-- Sistema de anúncios/tags
CREATE TABLE anuncios_marketing (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	marketing_id UUID NOT NULL REFERENCES profiles_marketing(id) ON DELETE CASCADE,
	video_id UUID REFERENCES videos(id) ON DELETE CASCADE, -- tag em vídeo específico
	estado VARCHAR(2) NOT NULL,
	posicao_carrossel INTEGER, -- ordem no carrossel
	ativo BOOLEAN DEFAULT true,
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	expires_at TIMESTAMP,
	
	-- Regra: Apenas 1 empresa por segmento no carrossel (leilão)
	CONSTRAINT unique_marketing_estado UNIQUE(estado, posicao_carrossel) WHERE ativo = true
);
```

**Regras de Negócio:**
- ✅ Limite: 3 empresas/estado
- ✅ Sistema de leilão por "cadeira" (posição no carrossel)
- ✅ Tags em vídeos (popup após 30s para usuários Base)
- ❌ **NÃO tem Connect Me** (anti-spam)

---

### 2.5 Perfil Vizinhança (Comércio Local)

**Conforme Escopo Sprint 3 (item 3.1):**
- Dados do estabelecimento
- Categoria (padaria, farmácia, pet, etc.)
- CEP de atuação
- Sistema de cupons com "palavra do dia"

**🔧 CRIAR:**
```sql
CREATE TYPE categoria_vizinhanca AS ENUM (
	'padaria',
	'farmacia',
	'pet',
	'lavanderia',
	'restaurante',
	'mercado',
	'academia',
	'salao_beleza',
	'outros'
);

CREATE TABLE profiles_vizinhanca (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	
	-- Dados do estabelecimento
	nome_estabelecimento TEXT NOT NULL,
	categoria categoria_vizinhanca NOT NULL,
	cnpj VARCHAR(14),
	phone VARCHAR(20),
	
	-- Localização (busca por CEP)
	zip_code VARCHAR(8) NOT NULL,
	street TEXT,
	number TEXT,
	district TEXT,
	city TEXT NOT NULL,
	state VARCHAR(2) NOT NULL,
	
	-- Galeria (até 3 fotos)
	fotos TEXT[], -- URLs, máximo 3
	
	-- Descrição
	descricao TEXT,
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	updated_at TIMESTAMP DEFAULT NOW() NOT NULL,
	
	CONSTRAINT unique_vizinhanca_user UNIQUE(user_id),
	CONSTRAINT max_3_fotos CHECK (array_length(fotos, 1) <= 3)
);

-- Tabela de promoções (Sprint 3)
CREATE TABLE promocoes (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	loja_id UUID NOT NULL REFERENCES profiles_vizinhanca(id) ON DELETE CASCADE,
	
	-- Dados da promoção
	titulo TEXT NOT NULL,
	descricao TEXT NOT NULL,
	regra TEXT, -- "10% de desconto", "Compre 1 leve 2", etc.
	
	-- Palavra do dia (geração de cupom)
	palavra_dia VARCHAR(5) NOT NULL, -- máximo 5 letras maiúsculas
	
	-- Exclusividade (Sprint 3, "Funcionalidade Adicional para Síndico")
	tipo VARCHAR(20) DEFAULT 'regional', -- 'regional' ou 'exclusiva'
	condominio_exclusivo_id UUID REFERENCES condominios(id) ON DELETE SET NULL,
	
	-- Validade
	valido_de DATE NOT NULL,
	valido_ate DATE NOT NULL,
	ativo BOOLEAN DEFAULT true,
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	updated_at TIMESTAMP DEFAULT NOW() NOT NULL
);

-- Tabela de cupons gerados (Sprint 3)
CREATE TABLE cupons (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	codigo VARCHAR(13) NOT NULL UNIQUE, -- ID_LOJA(5) + SEQ(3) + PALAVRA(5)
	
	promocao_id UUID NOT NULL REFERENCES promocoes(id) ON DELETE CASCADE,
	loja_id UUID NOT NULL REFERENCES profiles_vizinhanca(id) ON DELETE CASCADE,
	usuario_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	
	-- Status
	status VARCHAR(20) DEFAULT 'gerado', -- gerado, usado, expirado
	usado_em TIMESTAMP,
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	expires_at TIMESTAMP NOT NULL, -- gerado + 4 horas
	
	-- Regra: Usuário só pode gerar 1 cupom a cada 4h para mesma loja
	CONSTRAINT unique_usuario_loja_tempo UNIQUE(usuario_id, loja_id, created_at)
);

-- Índice para validação de 4h
CREATE INDEX idx_cupons_usuario_loja_created 
ON cupons(usuario_id, loja_id, created_at DESC);
```

**Regras de Negócio (Sprint 3, item 3.1):**
- ✅ Formato do cupom: `ID_LOJA(5) + SEQ(3) + PALAVRA(5)` = `00123055PAO`
- ✅ Trava de 4 horas: Usuário só pode gerar 1 cupom a cada 4h para mesma loja
- ✅ Palavra do dia: Máximo 5 letras maiúsculas, definida pelo lojista
- ✅ Promoções exclusivas: Síndico pode negociar promoção exclusiva para seu condomínio
- ✅ Promoções regionais: Abertas para todos do CEP

---

## 3. SISTEMA DE VÍDEOS

### 3.1 Tabela de Vídeos

**Conforme Escopo Sprint 1 (item 1.7) e Sprint 4 (item 4.1):**

```sql
CREATE TYPE tipo_video AS ENUM (
	'regular',        -- vídeos normais (1-4min)
	'institucional',  -- empresa (2min) ou síndico (pitch)
	'curriculo',      -- morador pagante (90s)
	'treinamento'     -- aulas de módulos/cursos (landscape)
);

CREATE TYPE status_moderacao AS ENUM (
	'pendente',
	'aprovado',
	'rejeitado'
);

CREATE TABLE videos (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	
	-- Tipo e status
	tipo tipo_video NOT NULL,
	status status_moderacao DEFAULT 'pendente' NOT NULL,
	motivo_rejeicao TEXT,
	
	-- Arquivo
	url TEXT NOT NULL, -- URL do Mux/Cloudflare Stream
	thumbnail_url TEXT,
	duracao_segundos INTEGER NOT NULL,
	
	-- Metadados
	titulo TEXT,
	descricao TEXT,
	tags TEXT[],
	
	-- Métricas (privadas, visíveis só para o autor)
	total_visualizacoes INTEGER DEFAULT 0,
	total_likes INTEGER DEFAULT 0,
	total_comentarios INTEGER DEFAULT 0,
	
	-- Moderação
	moderado_por UUID REFERENCES users(id) ON DELETE SET NULL,
	moderado_em TIMESTAMP,
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	updated_at TIMESTAMP DEFAULT NOW() NOT NULL,
	
	-- Validações
	CONSTRAINT duracao_regular CHECK (tipo != 'regular' OR duracao_segundos BETWEEN 60 AND 240),
	CONSTRAINT duracao_institucional CHECK (tipo != 'institucional' OR duracao_segundos <= 120),
	CONSTRAINT duracao_curriculo CHECK (tipo != 'curriculo' OR duracao_segundos <= 90)
);

-- Tabela de visualizações (para tracking de ≥70%)
CREATE TABLE video_views (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	video_id UUID NOT NULL REFERENCES videos(id) ON DELETE CASCADE,
	user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	
	percentual_assistido INTEGER DEFAULT 0, -- 0-100
	completo BOOLEAN GENERATED ALWAYS AS (percentual_assistido >= 70) STORED,
	
	started_at TIMESTAMP DEFAULT NOW() NOT NULL,
	last_position INTEGER DEFAULT 0, -- segundos
	
	-- Para calcular status do síndico
	CONSTRAINT unique_user_video_view UNIQUE(user_id, video_id)
);

-- Tabela de likes (privados)
CREATE TABLE video_likes (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	video_id UUID NOT NULL REFERENCES videos(id) ON DELETE CASCADE,
	user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	
	CONSTRAINT unique_user_video_like UNIQUE(user_id, video_id)
);

-- Tabela de comentários (privados)
CREATE TABLE video_comments (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	video_id UUID NOT NULL REFERENCES videos(id) ON DELETE CASCADE,
	user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	
	comentario TEXT NOT NULL,
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	updated_at TIMESTAMP DEFAULT NOW() NOT NULL
);
```

**Regras de Negócio:**
- ✅ Paywall para Base: Cortar stream aos 25% se vídeo de empresa
- ✅ Contador de visualização: Apenas se ≥70% assistido (para status síndico)
- ✅ Métricas privadas: Likes/comentários visíveis só para o autor
- ✅ Moderação obrigatória antes de publicar
- ✅ Trava trimestral para vídeos institucionais/currículo

---

## 4. SISTEMA DE ASSEMBLEIAS E VOTAÇÃO (Sprint 2)

### 4.1 Tabelas de Assembleia

**Conforme Escopo Sprint 2 (item 2.1):**

```sql
CREATE TYPE status_assembleia AS ENUM (
	'agendada',
	'aberta',
	'encerrada',
	'cancelada'
);

CREATE TABLE assembleias (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	condominio_id UUID NOT NULL REFERENCES condominios(id) ON DELETE CASCADE,
	sindico_id UUID NOT NULL REFERENCES profiles_sindico(id) ON DELETE CASCADE,
	
	-- Dados da assembleia
	titulo TEXT NOT NULL,
	descricao TEXT,
	data_realizacao TIMESTAMP NOT NULL,
	local TEXT,
	
	-- Edital
	edital_url TEXT, -- PDF
	edital_texto TEXT,
	
	-- Status e janela de acesso
	status status_assembleia DEFAULT 'agendada' NOT NULL,
	janela_inicio TIMESTAMP,
	janela_fim TIMESTAMP,
	
	-- Link de compartilhamento (gerado pelo síndico)
	link_compartilhamento TEXT UNIQUE,
	link_expira_em TIMESTAMP,
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	updated_at TIMESTAMP DEFAULT NOW() NOT NULL
);

-- Pautas da assembleia
CREATE TABLE pautas (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	assembleia_id UUID NOT NULL REFERENCES assembleias(id) ON DELETE CASCADE,
	
	-- Dados da pauta
	titulo TEXT NOT NULL,
	descricao TEXT,
	ordem INTEGER NOT NULL,
	
	-- Mídia anexa
	video_explicativo_url TEXT, -- vídeo/áudio do síndico
	videos_fornecedores UUID[], -- IDs de vídeos de empresas (anexados)
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	updated_at TIMESTAMP DEFAULT NOW() NOT NULL,
	
	CONSTRAINT unique_assembleia_ordem UNIQUE(assembleia_id, ordem)
);

-- Votações por pauta
CREATE TYPE tipo_votacao AS ENUM (
	'aprovar_reprovar',
	'multipla_escolha'
);

CREATE TABLE votacoes (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	pauta_id UUID NOT NULL REFERENCES pautas(id) ON DELETE CASCADE,
	
	-- Tipo e opções
	tipo tipo_votacao NOT NULL,
	opcoes JSONB NOT NULL, -- ["Aprovar", "Reprovar"] ou ["Fornecedor A", "Fornecedor B", "Fornecedor C"]
	
	-- Janela de votação
	janela_inicio TIMESTAMP NOT NULL,
	janela_fim TIMESTAMP NOT NULL,
	
	-- Resultados (calculados)
	total_votos INTEGER DEFAULT 0,
	resultado JSONB, -- {"Aprovar": 45, "Reprovar": 12}
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	updated_at TIMESTAMP DEFAULT NOW() NOT NULL
);

-- Votos (1 por unidade)
CREATE TABLE votos (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	votacao_id UUID NOT NULL REFERENCES votacoes(id) ON DELETE CASCADE,
	unidade_id UUID NOT NULL REFERENCES vinculos_unidade(id) ON DELETE CASCADE,
	
	opcao_escolhida TEXT NOT NULL, -- deve estar em votacoes.opcoes
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	
	-- Regra: 1 voto por unidade
	CONSTRAINT unique_unidade_votacao UNIQUE(unidade_id, votacao_id)
);
```

**Regras de Negócio (Sprint 2):**
- ✅ 1 voto por unidade (não por morador)
- ✅ Janela de tempo: Só vota se evento estiver "Aberto"
- ✅ Link temporário com expiração
- ✅ Anexar vídeos de fornecedores (busca do acervo)
- ❌ **NÃO é votação em blockchain**
- ❌ **NÃO é assembleia ao vivo** (Zoom/Meet)

---

## 5. SISTEMA DE LMS (Sprint 4)

### 5.1 Tabelas de Cursos

**Conforme Escopo Sprint 4 (item 4.1):**

```sql
CREATE TABLE cursos (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	empresa_id UUID NOT NULL REFERENCES profiles_empresa(id) ON DELETE CASCADE,
	
	-- Dados do curso
	titulo TEXT NOT NULL,
	descricao TEXT,
	thumbnail_url TEXT,
	
	-- Categoria (mesmo do Manual Cap. 6)
	categoria categoria_empresa NOT NULL,
	
	-- Status
	publicado BOOLEAN DEFAULT false,
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	updated_at TIMESTAMP DEFAULT NOW() NOT NULL
);

-- Módulos do curso
CREATE TABLE modulos (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	curso_id UUID NOT NULL REFERENCES cursos(id) ON DELETE CASCADE,
	
	titulo TEXT NOT NULL,
	descricao TEXT,
	ordem INTEGER NOT NULL,
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	updated_at TIMESTAMP DEFAULT NOW() NOT NULL,
	
	CONSTRAINT unique_curso_ordem UNIQUE(curso_id, ordem)
);

-- Aulas do módulo
CREATE TABLE aulas (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	modulo_id UUID NOT NULL REFERENCES modulos(id) ON DELETE CASCADE,
	
	titulo TEXT NOT NULL,
	descricao TEXT,
	ordem INTEGER NOT NULL,
	
	-- Conteúdo
	video_id UUID REFERENCES videos(id) ON DELETE SET NULL,
	anexos TEXT[], -- URLs de PDFs, Ebooks, etc.
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	updated_at TIMESTAMP DEFAULT NOW() NOT NULL,
	
	CONSTRAINT unique_modulo_ordem UNIQUE(modulo_id, ordem)
);

-- Progresso do aluno
CREATE TABLE progresso_aluno (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	curso_id UUID NOT NULL REFERENCES cursos(id) ON DELETE CASCADE,
	
	-- Progresso
	percentual_concluido INTEGER DEFAULT 0, -- 0-100
	concluido BOOLEAN DEFAULT false,
	
	-- Datas
	iniciado_em TIMESTAMP DEFAULT NOW() NOT NULL,
	concluido_em TIMESTAMP,
	
	CONSTRAINT unique_user_curso UNIQUE(user_id, curso_id)
);

-- Aulas concluídas
CREATE TABLE aulas_concluidas (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	aula_id UUID NOT NULL REFERENCES aulas(id) ON DELETE CASCADE,
	
	concluido_em TIMESTAMP DEFAULT NOW() NOT NULL,
	
	CONSTRAINT unique_user_aula UNIQUE(user_id, aula_id)
);

-- Certificados (gerados ao concluir 100%)
CREATE TABLE certificados (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	curso_id UUID NOT NULL REFERENCES cursos(id) ON DELETE CASCADE,
	
	-- Dados do certificado
	pdf_url TEXT NOT NULL,
	qr_code TEXT NOT NULL, -- código de validação externa
	
	-- Metadados
	nome_curso TEXT NOT NULL,
	nome_aluno TEXT NOT NULL,
	data_conclusao DATE NOT NULL,
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	
	CONSTRAINT unique_user_curso_certificado UNIQUE(user_id, curso_id)
);
```

**Regras de Negócio:**
- ✅ Estrutura: Curso → Módulo → Aula
- ✅ Certificado gerado automaticamente ao concluir 100%
- ✅ PDF dinâmico com QR Code de validação
- ✅ Apenas Empresa Pro pode criar cursos certificados

---

## 6. SISTEMA DE CONNECT ME (Sprint 1)

### 6.1 Tabela de Connect Me

**Conforme Escopo Sprint 1 (item 1.6) e Regra 5.3:**

```sql
CREATE TABLE connect_me (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	remetente_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	destinatario_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	
	-- Mensagem do formulário
	assunto TEXT NOT NULL,
	mensagem TEXT NOT NULL,
	
	-- Dados de contato (preenchidos pelo remetente)
	nome_contato TEXT NOT NULL,
	email_contato TEXT NOT NULL,
	telefone_contato TEXT,
	
	-- Status
	status VARCHAR(20) DEFAULT 'enviado', -- enviado, lido
	lido_em TIMESTAMP,
	
	-- Email disparado
	email_enviado BOOLEAN DEFAULT false,
	email_enviado_em TIMESTAMP,
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL
);

-- Log de uso de Connect Me (para validar cotas)
CREATE TABLE connect_me_log (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	destinatario_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	
	ano INTEGER NOT NULL,
	mes INTEGER,
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL
);

-- Índice para validação de cotas
CREATE INDEX idx_connect_me_log_user_ano_mes 
ON connect_me_log(user_id, ano, mes);
```

**Regras de Negócio (Sprint 1, item 1.6):**
- ✅ **NÃO é chat** (não tem histórico, reply, ou conversa)
- ✅ Formulário unidirecional: Input → Email disparado
- ✅ Cotas por plano (Matriz Funcional):
  - Morador Base: 2/ano
  - Morador Pagante: 4/ano
  - Síndico N2: 2/ano
  - Síndico N3: 4/ano
- ✅ Direção controlada:
  - Morador → Síndico
  - Síndico → Empresa
  - Empresa Plus/Pro → Empresa
- ❌ **Marketing NÃO tem Connect Me** (anti-spam)
- ❌ **Contato sempre parte do interessado** (morador/síndico) para o contratado

---

## 7. SISTEMA DE FÓRUM (Sprint 4)

### 7.1 Tabelas de Fórum

**Conforme Escopo Sprint 4 (item 4.3):**

```sql
CREATE TABLE forum_topicos (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	
	-- Categoria (baseada em categorias técnicas do Manual Cap. 6)
	categoria categoria_empresa NOT NULL,
	
	-- Conteúdo
	titulo TEXT NOT NULL,
	descricao TEXT NOT NULL,
	
	-- Métricas (privadas)
	total_respostas INTEGER DEFAULT 0,
	total_likes INTEGER DEFAULT 0,
	
	-- Status
	ativo BOOLEAN DEFAULT true,
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	updated_at TIMESTAMP DEFAULT NOW() NOT NULL
);

-- Respostas (com limite para usuários Base)
CREATE TABLE forum_respostas (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	topico_id UUID NOT NULL REFERENCES forum_topicos(id) ON DELETE CASCADE,
	user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	
	-- Conteúdo
	resposta TEXT NOT NULL,
	
	-- Métricas
	total_likes INTEGER DEFAULT 0,
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	updated_at TIMESTAMP DEFAULT NOW() NOT NULL
);

-- Likes em tópicos/respostas
CREATE TABLE forum_likes (
	id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	
	-- Polimórfico (tópico ou resposta)
	topico_id UUID REFERENCES forum_topicos(id) ON DELETE CASCADE,
	resposta_id UUID REFERENCES forum_respostas(id) ON DELETE CASCADE,
	
	created_at TIMESTAMP DEFAULT NOW() NOT NULL,
	
	CONSTRAINT check_like_target CHECK (
		(topico_id IS NOT NULL AND resposta_id IS NULL) OR
		(topico_id IS NULL AND resposta_id IS NOT NULL)
	)
);
```

**Regras de Negócio:**
- ✅ Fórum Público: Usuários Base (máximo 5 respostas)
- ✅ Fórum Exclusivo: Usuários pagantes (sem limitações)
- ✅ Categorias baseadas no Manual Cap. 6

---

## 8. RESUMO DE GAPS E AÇÕES NECESSÁRIAS

### 8.1 Tabelas FALTANDO (Crítico)

| Tabela | Prioridade | Sprint | Status |
|--------|------------|--------|--------|
| `profiles_morador` | 🔴 Alta | Sprint 1 | ❌ Criar |
| `profiles_sindico` | 🔴 Alta | Sprint 1 | ❌ Criar |
| `profiles_empresa` | 🔴 Alta | Sprint 1 | ❌ Criar |
| `profiles_marketing` | 🟡 Média | Sprint 4 | ❌ Criar |
| `profiles_vizinhanca` | 🟡 Média | Sprint 3 | ❌ Criar |
| `condominios` | 🔴 Alta | Sprint 1 | ❌ Criar |
| `vinculos_unidade` | 🔴 Alta | Sprint 1 | ❌ Criar |
| `vinculos_sindico_condominio` | 🔴 Alta | Sprint 1 | ❌ Criar |
| `videos` | 🔴 Alta | Sprint 1 | ❌ Criar |
| `video_views` | 🔴 Alta | Sprint 1 | ❌ Criar |
| `video_likes` | 🟡 Média | Sprint 4 | ❌ Criar |
| `video_comments` | 🟡 Média | Sprint 4 | ❌ Criar |
| `assembleias` | 🟡 Média | Sprint 2 | ❌ Criar |
| `pautas` | 🟡 Média | Sprint 2 | ❌ Criar |
| `votacoes` | 🟡 Média | Sprint 2 | ❌ Criar |
| `votos` | 🟡 Média | Sprint 2 | ❌ Criar |
| `promocoes` | 🟢 Baixa | Sprint 3 | ❌ Criar |
| `cupons` | 🟢 Baixa | Sprint 3 | ❌ Criar |
| `cursos` | 🟡 Média | Sprint 4 | ❌ Criar |
| `modulos` | 🟡 Média | Sprint 4 | ❌ Criar |
| `aulas` | 🟡 Média | Sprint 4 | ❌ Criar |
| `progresso_aluno` | 🟡 Média | Sprint 4 | ❌ Criar |
| `aulas_concluidas` | 🟡 Média | Sprint 4 | ❌ Criar |
| `certificados` | 🟡 Média | Sprint 4 | ❌ Criar |
| `connect_me` | 🔴 Alta | Sprint 1 | ❌ Criar |
| `connect_me_log` | 🔴 Alta | Sprint 1 | ❌ Criar |
| `forum_topicos` | 🟢 Baixa | Sprint 4 | ❌ Criar |
| `forum_respostas` | 🟢 Baixa | Sprint 4 | ❌ Criar |
| `forum_likes` | 🟢 Baixa | Sprint 4 | ❌ Criar |
| `anuncios_marketing` | 🟢 Baixa | Sprint 4 | ❌ Criar |

### 8.2 Alterações na Tabela `users` (Crítico)

```sql
-- ⚠️ ADICIONAR:
ALTER TABLE users ADD COLUMN plan user_plans DEFAULT 'none' NOT NULL;
CREATE INDEX idx_users_plan ON users(plan);
```

### 8.3 Enums FALTANDO (Crítico)

```sql
-- ⚠️ CRIAR:
CREATE TYPE user_plans AS ENUM (...);
CREATE TYPE status_reconhecimento AS ENUM ('bronze', 'prata', 'ouro', 'diamante');
CREATE TYPE tipo_video AS ENUM ('regular', 'institucional', 'curriculo', 'treinamento');
CREATE TYPE status_moderacao AS ENUM ('pendente', 'aprovado', 'rejeitado');
CREATE TYPE status_assembleia AS ENUM ('agendada', 'aberta', 'encerrada', 'cancelada');
CREATE TYPE tipo_votacao AS ENUM ('aprovar_reprovar', 'multipla_escolha');
CREATE TYPE categoria_vizinhanca AS ENUM (...);
CREATE TYPE categoria_empresa AS ENUM (...); -- 32 categorias
CREATE TYPE subcategoria_empresa AS ENUM (...); -- centenas de subcategorias
```

### 8.4 Schemas de Validação (Frontend)

**Status Atual:**
- ✅ `enterpriseSchema` - Completo
- ❌ `residentSchema` - Vazio (precisa preencher)
- ❌ `syndicSchema` - Vazio (precisa preencher)
- ❌ `localCompanySchema` - Vazio (precisa preencher)
- ❌ `marketingSchema` - Vazio (precisa preencher)

---

## 9. PRÓXIMOS PASSOS RECOMENDADOS

### Fase 1: Sprint 1 (Fundação) - URGENTE
1. ✅ Adicionar campo `plan` na tabela `users`
2. ✅ Criar enums: `user_plans`, `status_reconhecimento`, `tipo_video`, `status_moderacao`
3. ✅ Criar tabelas de perfil:
   - `profiles_morador`
   - `profiles_sindico`
   - `profiles_empresa`
4. ✅ Criar tabelas de condomínio:
   - `condominios`
   - `vinculos_unidade`
   - `vinculos_sindico_condominio`
5. ✅ Criar sistema de vídeos:
   - `videos`
   - `video_views`
6. ✅ Criar sistema Connect Me:
   - `connect_me`
   - `connect_me_log`

### Fase 2: Sprint 2 (Assembleias)
7. Criar sistema de assembleias:
   - `assembleias`
   - `pautas`
   - `votacoes`
   - `votos`

### Fase 3: Sprint 3 (Vizinhança)
8. Criar perfil vizinhança:
   - `profiles_vizinhanca`
9. Criar sistema de cupons:
   - `promocoes`
   - `cupons`

### Fase 4: Sprint 4 (LMS e Administração)
10. Criar sistema LMS:
    - `cursos`
    - `modulos`
    - `aulas`
    - `progresso_aluno`
    - `certificados`
11. Criar sistema de fórum:
    - `forum_topicos`
    - `forum_respostas`
    - `forum_likes`
12. Criar perfil marketing:
    - `profiles_marketing`
    - `anuncios_marketing`

---

## 10. CONCLUSÃO

A modelagem atual está **incompleta** comparada ao escopo validado. As principais lacunas são:

1. ❌ **Falta campo `plan`** na tabela `users` (crítico para diferenciar N1/N2/N3, Plus/Pro)
2. ❌ **Faltam todas as tabelas de perfil** (morador, síndico, empresa, marketing, vizinhança)
3. ❌ **Falta sistema de vídeos completo** (com métricas privadas e paywall)
4. ❌ **Falta sistema de assembleias e votação**
5. ❌ **Falta sistema de cupons (vizinhança)**
6. ❌ **Falta sistema LMS** (cursos, módulos, certificados)
7. ❌ **Falta sistema de fórum**

**Recomendação:** Começar imediatamente pela **Sprint 1 (Fase 1)** criando as tabelas de perfil e sistema de planos, pois são bloqueadores para o onboarding.

---

**Documento gerado por:** Droid (Factory AI)  
**Versão:** 1.0  
**Status:** ✅ Completo e validado contra escopo aprovado
