# 🔍 ANÁLISE COMPLETA: Modelagem vs Escopo Validado

**Data:** 2026-02-01  
**Documento Base:** `novo escopo-7.pdf` (Escopo Técnico Validado - SOW)  
**Modelagem Atual:** `modelagem.txt`

---

## ⚠️ RESUMO EXECUTIVO

### Status Geral: 🟡 **85% Completo - Ajustes Necessários**

| Categoria | Status | Observações |
|-----------|--------|-------------|
| **Enums** | ✅ 95% | Falta apenas `categoria_empresa` e `subcategoria_empresa` |
| **Usuários e Auth** | ✅ 100% | Completo e correto |
| **Leads** | ✅ 100% | Completo (extra - não está no escopo) |
| **Planos e Assinaturas** | ⚠️ 70% | Falta separação clara role vs plan |
| **Perfis** | ⚠️ 80% | Faltam campos críticos em cada perfil |
| **Condomínios** | ✅ 95% | Quase completo |
| **Categorias** | ❌ 40% | Falta migrar para banco (ainda está enum) |
| **Vídeos** | ✅ 90% | Faltam regras de trava trimestral |
| **Lives** | ✅ 100% | Completo |
| **Cursos (LMS)** | ✅ 100% | Completo |
| **Assembleias** | ⚠️ 75% | Faltam campos e regras |
| **Fórum** | ✅ 95% | Quase completo |
| **Cupons** | ❌ 0% | **FALTA COMPLETAMENTE** |
| **Connect Me** | ❌ 0% | **FALTA COMPLETAMENTE** |
| **Infraestrutura** | ⚠️ 50% | Faltam audit logs e rate limiting |

---

## 1. 🔴 CRÍTICO - TABELAS COMPLETAMENTE AUSENTES

### 1.1 Sistema de Cupons (Sprint 3)

**Conforme Escopo:** Sprint 3, item 3.1 - "Engine de Cupons ('Promoção do Dia')"

**❌ FALTA CRIAR:**

```typescript
// ===================================
// CUPONS E PROMOÇÕES (Sprint 3)
// ===================================

export const promocoes = pgTable(
	"promocoes",
	{
		id: uuid("id").primaryKey().$defaultFn(() => uuidv7()),
		
		// FK para vizinhança
		vizinhancaUserId: uuid("vizinhanca_user_id")
			.notNull()
			.references(() => users.id, { onDelete: "cascade" }),
		
		// Dados da promoção
		titulo: text("titulo").notNull(),
		descricao: text("descricao").notNull(),
		regra: text("regra"), // "10% de desconto", "Compre 1 leve 2"
		
		// Palavra do dia (geração de cupom)
		palavraDia: text("palavra_dia").notNull(), // máximo 5 letras maiúsculas
		
		// Exclusividade (Escopo: "Funcionalidade Adicional para Síndico")
		tipo: text("tipo").default("regional").notNull(), // 'regional' ou 'exclusiva'
		condominioExclusivoId: uuid("condominio_exclusivo_id")
			.references(() => buildings.id, { onDelete: "set null" }),
		
		// Validade
		validoDe: timestamp("valido_de").notNull(),
		validoAte: timestamp("valido_ate").notNull(),
		ativo: boolean("ativo").default(true),
		
		createdAt: timestamp("created_at").$defaultFn(() => new Date()).notNull(),
		updatedAt: timestamp("updated_at")
			.$defaultFn(() => new Date())
			.$onUpdate(() => new Date())
			.notNull(),
	},
	(t) => [
		index("promocoes_vizinhanca_idx").on(t.vizinhancaUserId),
		index("promocoes_condominio_idx").on(t.condominioExclusivoId),
		index("promocoes_validade_idx").on(t.validoDe, t.validoAte),
		index("promocoes_ativo_idx").on(t.ativo),
	]
);

export const cupons = pgTable(
	"cupons",
	{
		id: uuid("id").primaryKey().$defaultFn(() => uuidv7()),
		
		// Código no formato: ID_LOJA(5) + SEQ(3) + PALAVRA(5)
		codigo: text("codigo").notNull().unique(), // "00123055PAO"
		
		// FKs
		promocaoId: uuid("promocao_id")
			.notNull()
			.references(() => promocoes.id, { onDelete: "cascade" }),
		vizinhancaUserId: uuid("vizinhanca_user_id")
			.notNull()
			.references(() => users.id, { onDelete: "cascade" }),
		usuarioId: uuid("usuario_id")
			.notNull()
			.references(() => users.id, { onDelete: "cascade" }),
		
		// Status
		status: text("status").default("gerado").notNull(), // gerado, usado, expirado
		usadoEm: timestamp("usado_em"),
		
		createdAt: timestamp("created_at").$defaultFn(() => new Date()).notNull(),
		expiresAt: timestamp("expires_at").notNull(), // createdAt + 4 horas
	},
	(t) => [
		index("cupons_codigo_idx").on(t.codigo),
		index("cupons_usuario_idx").on(t.usuarioId),
		index("cupons_vizinhanca_idx").on(t.vizinhancaUserId),
		index("cupons_status_idx").on(t.status),
		// Índice para validar regra de 4h
		index("cupons_usuario_vizinhanca_created_idx").on(
			t.usuarioId,
			t.vizinhancaUserId,
			t.createdAt
		),
	]
);
```

**Regras de Negócio (Escopo 3.1):**
- ✅ Formato do cupom: `ID_LOJA(5) + SEQ(3) + PALAVRA(5)`
- ✅ Trava de 4 horas: Usuário só pode gerar 1 cupom a cada 4h para mesma loja
- ✅ Palavra do dia: Máximo 5 letras maiúsculas
- ✅ Promoções exclusivas por condomínio (negociadas com síndico)
- ✅ Promoções regionais (abertas para todos do CEP)

---

### 1.2 Sistema de Connect Me (Sprint 1)

**Conforme Escopo:** Sprint 1, item 1.6 - "Funcionalidade 'Connect Me' (Formulário de Contato Unidirecional)"

**❌ FALTA CRIAR:**

```typescript
// ===================================
// CONNECT ME (Sprint 1, item 1.6)
// ===================================

export const connectMe = pgTable(
	"connect_me",
	{
		id: uuid("id").primaryKey().$defaultFn(() => uuidv7()),
		
		// Direção do contato
		remetenteId: uuid("remetente_id")
			.notNull()
			.references(() => users.id, { onDelete: "cascade" }),
		destinatarioId: uuid("destinatario_id")
			.notNull()
			.references(() => users.id, { onDelete: "cascade" }),
		
		// Mensagem do formulário
		assunto: text("assunto").notNull(),
		mensagem: text("mensagem").notNull(),
		
		// Dados de contato (preenchidos pelo remetente)
		nomeContato: text("nome_contato").notNull(),
		emailContato: text("email_contato").notNull(),
		telefoneContato: text("telefone_contato"),
		
		// Status
		status: text("status").default("enviado").notNull(), // enviado, lido
		lidoEm: timestamp("lido_em"),
		
		// Email disparado (Escopo 2.2)
		emailEnviado: boolean("email_enviado").default(false),
		emailEnviadoEm: timestamp("email_enviado_em"),
		
		createdAt: timestamp("created_at").$defaultFn(() => new Date()).notNull(),
	},
	(t) => [
		index("connect_me_remetente_idx").on(t.remetenteId),
		index("connect_me_destinatario_idx").on(t.destinatarioId),
		index("connect_me_status_idx").on(t.status),
		index("connect_me_created_at_idx").on(t.createdAt),
	]
);

// Log para validar cotas (Matriz Funcional)
export const connectMeLog = pgTable(
	"connect_me_log",
	{
		id: uuid("id").primaryKey().$defaultFn(() => uuidv7()),
		
		userId: uuid("user_id")
			.notNull()
			.references(() => users.id, { onDelete: "cascade" }),
		destinatarioId: uuid("destinatario_id")
			.notNull()
			.references(() => users.id, { onDelete: "cascade" }),
		
		// Para calcular cotas anuais
		ano: integer("ano").notNull(),
		
		createdAt: timestamp("created_at").$defaultFn(() => new Date()).notNull(),
	},
	(t) => [
		index("connect_me_log_user_ano_idx").on(t.userId, t.ano),
		index("connect_me_log_user_idx").on(t.userId),
	]
);
```

**Regras de Negócio (Escopo 1.6 e Matriz Funcional):**
- ✅ **NÃO é chat** (não tem histórico, reply, ou conversa)
- ✅ Formulário unidirecional: Input → Email disparado
- ✅ Cotas por plano:
  - Morador Base: 2/ano
  - Morador Pagante: 4/ano
  - Síndico N2: 2/ano
  - Síndico N3: 4/ano
- ✅ Direção controlada:
  - Morador → Síndico
  - Síndico → Empresa
  - Empresa Plus/Pro → Empresa (entre empresas)
- ❌ **Marketing NÃO tem Connect Me** (anti-spam)

---

## 2. 🟡 AJUSTES NECESSÁRIOS - SEPARAÇÃO ROLE vs PLAN

### 2.1 Problema Crítico: Confusão entre Role e Plan

**Escopo Seção 6.1:** Existe diferença entre ROLE (o que a pessoa É) e PLAN (nível de pagamento).

**Exemplo:**
- **Role:** `sindico`
- **Plans:** `n1`, `n2`, `n3`

**Situação Atual:**
```typescript
// users.ts
export const users = pgTable("users", {
	role: userRoles("role").default("none").notNull(), // ✅ CORRETO
	// ❌ FALTA: Campo "plan"
});
```

**❌ PROBLEMA:** Não há como diferenciar um Síndico N1 de um Síndico N3.

**✅ SOLUÇÃO:**

```typescript
// 1. Adicionar enum de plans
export const userPlans = pgEnum("user_plans", [
	"none",
	"base",           // Morador Base (free)
	"pagante",        // Morador Pagante (R$ 9,90)
	"n1",             // Síndico N1
	"n2",             // Síndico N2
	"n3",             // Síndico N3
	"plus",           // Empresa Plus
	"pro",            // Empresa Pro
	"marketing_standard", // Marketing
	"vizinhanca_standard" // Vizinhança
]);

// 2. Adicionar campo na tabela users
export const users = pgTable("users", {
	// ... campos existentes
	role: userRoles("role").default("none").notNull(),
	plan: userPlans("plan").default("none").notNull(), // ⬅️ ADICIONAR
	// ...
});

// 3. Adicionar índice
index("users_plan_idx").on(t.plan),
index("users_role_plan_idx").on(t.role, t.plan),
```

**Impacto:** Esse campo é **CRÍTICO** para todo o sistema ABAC (Attribute-Based Access Control).

---

## 3. 🟡 PERFIS - CAMPOS FALTANDO

### 3.1 Perfil Morador

**Status Atual:** ✅ 85% Completo

**❌ FALTA (Escopo Sprint 1, item 1.3):**

```typescript
export const moradorProfiles = pgTable("morador_profiles", {
	// ... campos existentes ✅
	
	// ❌ ADICIONAR:
	cpf: text("cpf").unique(),
	birthDate: timestamp("birth_date"),
	
	// Endereço (opcional)
	street: text("street"),
	number: text("number"),
	block: text("block"),
	district: text("district"),
	city: text("city"),
	state: text("state"),
	zipCode: text("zip_code"),
	
	// Ajustar referência de building para condomínio
	// ✅ JÁ TEM: buildingId
	
	// ✅ JÁ TEM: curriculumVideoUrl, lastCurriculumUpload
	// ✅ JÁ TEM: connectMeQuota, connectMeUsed, connectMeResetsAt
});
```

**Regra Trimestral (Escopo 2.6):**
- ✅ Vídeo-currículo: 90 segundos
- ✅ Trava de 90 dias: `lastCurriculumUpload`
- ⚠️ **Falta validação no backend:** `lastCurriculumUpload + 90 dias`

---

### 3.2 Perfil Síndico

**Status Atual:** ✅ 90% Completo

**❌ FALTA (Escopo Sprint 1 e Cap. 8):**

```typescript
export const sindicoProfiles = pgTable("sindico_profiles", {
	// ... campos existentes ✅
	
	// ❌ ADICIONAR:
	cpf: text("cpf").unique().notNull(),
	phone: text("phone"),
	tempoCarreiraAnos: integer("tempo_carreira_anos").default(0),
	
	// Vídeo institucional (pitch de venda)
	videoInstitucionalUrl: text("video_institucional_url"),
	lastInstitucionalUpload: timestamp("last_institucional_upload"),
	
	// ✅ JÁ TEM: recognitionLevel, recognitionScore
	// ✅ JÁ TEM: videosWatched, modulesCompleted, buildingsManaged
	
	// ⚠️ AJUSTAR: Campo "buildingsManaged" deve ser calculado, não armazenado
	// (contar registros em sindicoBuildings)
	
	// ✅ JÁ TEM: videosPublishedThisMonth, videoQuotaResetsAt
});
```

**Regra de Reconhecimento (Cap. 8):**
- ✅ Status calculado por: `videosWatched` (≥70%), `modulesCompleted`, `buildingsManaged`
- ⚠️ **Falta:** Job/CRON para recalcular `recognitionLevel` periodicamente
- ✅ Visível para moradores no perfil público

---

### 3.3 Perfil Empresa

**Status Atual:** ✅ 75% Completo

**❌ FALTA (Escopo Sprint 1, item 1.3):**

```typescript
export const empresaProfiles = pgTable("empresa_profiles", {
	// ... campos existentes ✅
	
	// ❌ ADICIONAR:
	phone: text("phone"),
	
	// Endereço COMPLETO (obrigatório)
	street: text("street").notNull(),
	number: text("number").notNull(),
	block: text("block"),
	district: text("district").notNull(),
	city: text("city").notNull(),
	state: text("state").notNull(), // VARCHAR(2)
	zipCode: text("zip_code").notNull(), // 8 dígitos
	
	// Seguro de responsabilidade civil (campo crítico)
	possuiSeguroResponsabilidadeCivil: boolean("possui_seguro_responsabilidade_civil")
		.default(false),
	
	// ✅ JÁ TEM: companyName, cnpj, bio, description
	// ✅ JÁ TEM: institutionalVideoUrl, lastInstitutionalUpload
	// ✅ JÁ TEM: videosPublishedThisMonth, videoQuotaResetsAt
	// ✅ JÁ TEM: totalStoredVideos
});
```

**Categorias (Manual Cap. 6):**
- ⚠️ **PROBLEMA:** Categorias ainda estão em tabela separada `empresaCategories`
- ✅ **CORRETO:** 1 categoria principal + até 5 subcategorias
- ❌ **FALTA:** Relacionamento com `serviceCategories` não permite distinção entre "principal" e "secundária"

**✅ SOLUÇÃO:**

```typescript
export const empresaProfiles = pgTable("empresa_profiles", {
	// ... campos existentes
	
	// Categoria principal (obrigatória)
	categoriaPrincipalId: uuid("categoria_principal_id")
		.notNull()
		.references(() => serviceCategories.id),
});

// Subcategorias (máximo 5)
export const empresaSubcategorias = pgTable(
	"empresa_subcategorias",
	{
		id: uuid("id").primaryKey().$defaultFn(() => uuidv7()),
		
		empresaUserId: uuid("empresa_user_id")
			.notNull()
			.references(() => users.id, { onDelete: "cascade" }),
		subcategoriaId: uuid("subcategoria_id")
			.notNull()
			.references(() => serviceCategories.id, { onDelete: "cascade" }),
		
		ordem: integer("ordem").default(0),
		
		createdAt: timestamp("created_at").$defaultFn(() => new Date()).notNull(),
	},
	(t) => [
		index("empresa_subcategorias_empresa_idx").on(t.empresaUserId),
		uniqueIndex("empresa_subcategorias_unique").on(t.empresaUserId, t.subcategoriaId),
	]
);
```

**Validação Backend:**
- Máximo 5 subcategorias por empresa
- Subcategorias devem pertencer à categoria principal (validar via `CATEGORIA_SUB_MAP`)

---

### 3.4 Perfil Marketing

**Status Atual:** ✅ 70% Completo

**❌ FALTA (Escopo Sprint 4, item 4.4):**

```typescript
export const marketingProfiles = pgTable("marketing_profiles", {
	// ... campos existentes ✅
	
	// ❌ ADICIONAR:
	phone: text("phone"),
	
	// Endereço (cidade/estado para sistema de "cadeiras")
	city: text("city").notNull(),
	state: text("state").notNull(), // VARCHAR(2)
	
	// Sistema de "cadeiras" (leilão BNI-style)
	slotsContratados: integer("slots_contratados").default(0),
	
	// ✅ JÁ TEM: agencyName, cnpj, bio
	// ⚠️ REMOVER: totalStoredVideos (Marketing não publica vídeos, apenas tags)
});
```

**Sistema de Anúncios (Escopo 4.4):**

**❌ FALTA CRIAR:**

```typescript
export const anunciosMarketing = pgTable(
	"anuncios_marketing",
	{
		id: uuid("id").primaryKey().$defaultFn(() => uuidv7()),
		
		marketingUserId: uuid("marketing_user_id")
			.notNull()
			.references(() => users.id, { onDelete: "cascade" }),
		
		// Tag em vídeo específico (opcional)
		videoId: uuid("video_id")
			.references(() => videos.id, { onDelete: "cascade" }),
		
		// Estado (limite: 3 empresas/estado)
		estado: text("estado").notNull(), // VARCHAR(2)
		
		// Posição no carrossel (leilão)
		posicaoCarrossel: integer("posicao_carrossel"),
		
		ativo: boolean("ativo").default(true),
		
		createdAt: timestamp("created_at").$defaultFn(() => new Date()).notNull(),
		expiresAt: timestamp("expires_at"),
	},
	(t) => [
		index("anuncios_marketing_user_idx").on(t.marketingUserId),
		index("anuncios_marketing_estado_idx").on(t.estado),
		index("anuncios_marketing_video_idx").on(t.videoId),
		// Regra: Apenas 1 empresa por posição no carrossel
		uniqueIndex("anuncios_marketing_estado_posicao_unique").on(
			t.estado,
			t.posicaoCarrossel
		).where(sql`ativo = true`),
	]
);
```

**Regras de Negócio:**
- ✅ Limite: 3 empresas/estado
- ✅ Sistema de leilão por "cadeira" (posição no carrossel)
- ✅ Tags em vídeos (popup após 30s para usuários Base)
- ❌ **NÃO tem Connect Me** (anti-spam)

---

### 3.5 Perfil Vizinhança

**Status Atual:** ✅ 80% Completo

**❌ FALTA (Escopo Sprint 3):**

```typescript
export const vizinhancaProfiles = pgTable("vizinhanca_profiles", {
	// ... campos existentes ✅
	
	// ❌ ADICIONAR:
	cnpj: text("cnpj").unique(), // Opcional (pode ser MEI sem CNPJ)
	phone: text("phone"),
	
	// Endereço completo
	street: text("street"),
	number: text("number"),
	district: text("district"),
	// ✅ JÁ TEM: city, cep
	state: text("state").notNull(), // VARCHAR(2)
	
	// Galeria de fotos (máximo 3)
	fotos: text("fotos").array(), // text[] no Postgres
	
	// Descrição
	descricao: text("descricao"),
	
	// ✅ JÁ TEM: businessName, businessType, address, cep
	// ✅ JÁ TEM: currentKeyword (palavra do dia)
});
```

**Validação Backend:**
- Máximo 3 fotos: `CHECK (array_length(fotos, 1) <= 3)`

---

## 4. 🟡 CATEGORIAS - MIGRAÇÃO PARA BANCO

**❌ PROBLEMA CRÍTICO:** Categorias e subcategorias ainda estão como **ENUMS hardcoded** no código.

**Conforme Escopo (Nota futura):** "Categorias devem ser migradas do enum estático para banco de dados com API."

**Status Atual:**
```typescript
// enums.schema.ts (Frontend)
export const categoriaEmpresaSchema = z.enum([
	"ADMINISTRACAO_GESTAO_GOVERNANCA",
	"JURIDICO_DIREITO_IMOBILIARIO",
	// ... 32 categorias
]);

export const subcategoriaEmpresaSchema = z.enum([
	"ADMINISTRADORAS_CONDOMINIOS",
	"CONSULTORIA_GESTAO_CONDOMINIAL",
	// ... centenas de subcategorias
]);
```

**✅ JÁ TEM na modelagem:**
```typescript
export const serviceCategories = pgTable("service_categories", {
	id, slug, name, description, parentId, order, isActive
});
```

**❌ FALTA:**
1. Seed/Migration para popular `serviceCategories` com todas as 32 categorias + subcategorias
2. Atualizar frontend para buscar categorias da API (não mais enum)
3. Relacionamento correto entre empresa e categorias (categoria principal + 5 subcategorias)

**✅ AÇÃO:**

```typescript
// Migration para popular categorias
// 1. Criar categorias principais (parentId = null)
INSERT INTO service_categories (slug, name, parent_id) VALUES
  ('ADMINISTRACAO_GESTAO_GOVERNANCA', 'Administração, Gestão e Governança', NULL),
  ('JURIDICO_DIREITO_IMOBILIARIO', 'Jurídico e Direito Imobiliário', NULL),
  // ... todas as 32 categorias

// 2. Criar subcategorias (parentId = categoria pai)
INSERT INTO service_categories (slug, name, parent_id) VALUES
  ('ADMINISTRADORAS_CONDOMINIOS', 'Administradoras de Condomínios', (SELECT id FROM service_categories WHERE slug = 'ADMINISTRACAO_GESTAO_GOVERNANCA')),
  // ... todas as subcategorias
```

---

## 5. 🟡 VÍDEOS - REGRAS DE TRAVA TRIMESTRAL

**Status Atual:** ✅ 90% Completo

**❌ FALTA Implementar Regra (Escopo 2.6):**

**Campo já existe:**
```typescript
export const videos = pgTable("videos", {
	// ✅ JÁ TEM:
	canBeReplacedAfter: timestamp("can_be_replaced_after"),
	replacesVideoId: uuid("replaces_video_id").references(() => videos.id),
});
```

**❌ FALTA Lógica de Validação (Backend):**

```typescript
// Antes de permitir upload de vídeo institucional/currículo
async function validateTrimestraUpload(userId: string, videoType: VideoType) {
	const ultimoUpload = await db
		.select()
		.from(videos)
		.where(
			and(
				eq(videos.authorUserId, userId),
				eq(videos.type, videoType),
				or(
					eq(videoType, 'institucional'),
					eq(videoType, 'curriculo')
				)
			)
		)
		.orderBy(desc(videos.createdAt))
		.limit(1);
	
	if (ultimoUpload.length > 0) {
		const diasDesdeUltimo = dayjs().diff(ultimoUpload[0].createdAt, 'days');
		
		if (diasDesdeUltimo < 90) {
			const diasRestantes = 90 - diasDesdeUltimo;
			throw new Error(
				`Você pode atualizar este vídeo novamente em ${diasRestantes} dias.`
			);
		}
	}
	
	// Se passou dos 90 dias, setar canBeReplacedAfter
	const canBeReplaced = dayjs().add(90, 'days').toDate();
	return canBeReplaced;
}
```

**Aplicável a:**
- ✅ Vídeo institucional (Empresa): 2min, trava 90 dias
- ✅ Vídeo-currículo (Morador Pagante): 90s, trava 90 dias
- ❌ Vídeo institucional (Síndico): **FALTA** na modelagem atual

---

## 6. 🟡 ASSEMBLEIAS - CAMPOS E REGRAS

**Status Atual:** ✅ 75% Completo

**❌ FALTA (Escopo Sprint 2, item 2.1):**

### 6.1 Tabela `assemblies`

```typescript
export const assemblies = pgTable("assemblies", {
	// ✅ JÁ TEM: id, buildingId, createdByUserId, title, description
	// ✅ JÁ TEM: editalUrl, editalText
	// ✅ JÁ TEM: accessToken, tokenExpiresAt
	// ✅ JÁ TEM: requiresApproval, allowedUserIds
	// ✅ JÁ TEM: scheduledDate, openVotingAt, closeVotingAt
	// ✅ JÁ TEM: status
	
	// ❌ ADICIONAR:
	local: text("local"), // Local físico da assembleia
});
```

### 6.2 Tabela `agendaItems` (Pautas)

```typescript
export const agendaItems = pgTable("agenda_items", {
	// ✅ JÁ TEM: id, assemblyId, order, title, description
	// ✅ JÁ TEM: sindicoVideoUrl, sindicoAudioUrl
	
	// ✅ CORRETO - Anexos de empresas em tabela separada
});

export const agendaItemAttachments = pgTable("agenda_item_attachments", {
	// ✅ JÁ TEM: id, agendaItemId, empresaUserId, videoUrl
	// ✅ JÁ TEM: title, description
	
	// ✅ CORRETO
});
```

### 6.3 Sistema de Votação

**❌ PROBLEMA:** Regra "1 voto por unidade" não está clara.

**Escopo:** "1 voto por unidade (exige regra de negócio para validar unidade)"

**Status Atual:**
```typescript
export const userVotes = pgTable("user_votes", {
	voteId, userId, apartmentNumber, answer
});

// ⚠️ Constraint:
uniqueIndex("user_votes_vote_apartment_unique").on(t.voteId, t.apartmentNumber)
```

**✅ CORRETO**, mas:

**❌ FALTA:** Relacionamento com `vinculos_unidade` para validar que:
1. Usuário está vinculado àquela unidade
2. Unidade pertence ao condomínio da assembleia

**✅ SOLUÇÃO:**

```typescript
export const userVotes = pgTable("user_votes", {
	id: uuid("id").primaryKey().$defaultFn(() => uuidv7()),
	
	voteId: uuid("vote_id")
		.notNull()
		.references(() => votes.id, { onDelete: "cascade" }),
	
	// ❌ REMOVER: userId (redundante)
	// ❌ REMOVER: apartmentNumber (string não é confiável)
	
	// ✅ ADICIONAR: FK para vínculo de unidade
	vinculoUnidadeId: uuid("vinculo_unidade_id")
		.notNull()
		.references(() => moradorProfiles.id), // ou criar tabela vinculos_unidade
	
	answer: text("answer").notNull(),
	
	createdAt: timestamp("created_at").$defaultFn(() => new Date()).notNull(),
},
(t) => [
	index("user_votes_vote_idx").on(t.voteId),
	// Regra: 1 voto por unidade
	uniqueIndex("user_votes_vote_unidade_unique").on(t.voteId, t.vinculoUnidadeId),
]);
```

**Isso requer:**

**❌ FALTA CRIAR:**

```typescript
export const vinculosUnidade = pgTable(
	"vinculos_unidade",
	{
		id: uuid("id").primaryKey().$defaultFn(() => uuidv7()),
		
		moradorId: uuid("morador_id")
			.notNull()
			.references(() => moradorProfiles.id, { onDelete: "cascade" }),
		buildingId: uuid("building_id")
			.notNull()
			.references(() => buildings.id, { onDelete: "cascade" }),
		
		numeroUnidade: text("numero_unidade").notNull(),
		bloco: text("bloco"),
		tipoVinculo: text("tipo_vinculo").default("proprietario").notNull(), 
		// proprietario, inquilino, dependente
		
		ativo: boolean("ativo").default(true),
		dataInicio: timestamp("data_inicio"),
		dataFim: timestamp("data_fim"),
		
		createdAt: timestamp("created_at").$defaultFn(() => new Date()).notNull(),
	},
	(t) => [
		index("vinculos_unidade_morador_idx").on(t.moradorId),
		index("vinculos_unidade_building_idx").on(t.buildingId),
		uniqueIndex("vinculos_unidade_unique").on(
			t.moradorId,
			t.buildingId,
			t.numeroUnidade
		),
	]
);
```

---

## 7. ✅ TABELAS COMPLETAS E CORRETAS

### 7.1 Usuários e Auth
- ✅ `users` - Completo (falta apenas `plan`)
- ✅ `accounts` - Completo
- ✅ `sessions` - Completo
- ✅ `verifications` - Completo

### 7.2 Leads
- ✅ `leads` - Completo (extra - não está no escopo, mas é útil)

### 7.3 Planos
- ✅ `plans` - Completo
- ✅ `planFeatures` - Completo
- ✅ `userSubscriptions` - Completo
- ⚠️ Falta apenas separar `role` vs `plan` na tabela `users`

### 7.4 Condomínios
- ✅ `buildings` - Completo
- ✅ `sindicoBuildings` - Completo (relação N:N)

### 7.5 Vídeos
- ✅ `videos` - Completo (falta apenas lógica de trava trimestral)
- ✅ `videoViews` - Completo
- ✅ `videoLikes` - Completo
- ✅ `videoComments` - Completo

### 7.6 Lives
- ✅ `lives` - Completo
- ✅ `liveMessages` - Completo

### 7.7 Cursos (LMS)
- ✅ `courses` - Completo
- ✅ `courseModules` - Completo
- ✅ `courseLessons` - Completo
- ✅ `courseEnrollments` - Completo
- ✅ `lessonProgress` - Completo

### 7.8 Fórum
- ✅ `forumCategories` - Completo
- ✅ `forumTopics` - Completo
- ✅ `forumReplies` - Completo
- ✅ `forumLikes` - Completo (incompleto no arquivo, mas estrutura está correta)

---

## 8. 🔴 INFRAESTRUTURA - TABELAS FALTANDO

### 8.1 Audit Logs

**❌ FALTA CRIAR:**

```typescript
export const auditLogs = pgTable(
	"audit_logs",
	{
		id: uuid("id").primaryKey().$defaultFn(() => uuidv7()),
		
		// Quem fez a ação
		userId: uuid("user_id")
			.references(() => users.id, { onDelete: "set null" }),
		
		// Ação
		action: text("action").notNull(), // 'create', 'update', 'delete'
		entity: text("entity").notNull(), // 'video', 'user', 'assembly'
		entityId: uuid("entity_id"),
		
		// Dados
		oldData: text("old_data"), // JSONB
		newData: text("new_data"), // JSONB
		
		// Metadata
		ipAddress: text("ip_address"),
		userAgent: text("user_agent"),
		
		createdAt: timestamp("created_at").$defaultFn(() => new Date()).notNull(),
	},
	(t) => [
		index("audit_logs_user_idx").on(t.userId),
		index("audit_logs_entity_idx").on(t.entity, t.entityId),
		index("audit_logs_action_idx").on(t.action),
		index("audit_logs_created_at_idx").on(t.createdAt),
	]
);
```

### 8.2 Rate Limiting

**❌ FALTA CRIAR:**

```typescript
export const rateLimits = pgTable(
	"rate_limits",
	{
		id: uuid("id").primaryKey().$defaultFn(() => uuidv7()),
		
		// Identificador (userId, IP, etc.)
		identifier: text("identifier").notNull(),
		
		// Endpoint/Action
		action: text("action").notNull(),
		
		// Contador
		count: integer("count").default(1),
		
		// Janela de tempo
		windowStart: timestamp("window_start").$defaultFn(() => new Date()).notNull(),
		expiresAt: timestamp("expires_at").notNull(),
	},
	(t) => [
		uniqueIndex("rate_limits_unique").on(t.identifier, t.action, t.windowStart),
		index("rate_limits_expires_at_idx").on(t.expiresAt),
	]
);
```

### 8.3 Webhooks (Stripe, etc.)

**❌ FALTA CRIAR:**

```typescript
export const webhookEvents = pgTable(
	"webhook_events",
	{
		id: uuid("id").primaryKey().$defaultFn(() => uuidv7()),
		
		// Provedor
		provider: text("provider").notNull(), // 'stripe', 'mercadopago'
		eventType: text("event_type").notNull(),
		eventId: text("event_id").unique().notNull(),
		
		// Payload
		payload: text("payload").notNull(), // JSONB
		
		// Status
		processed: boolean("processed").default(false),
		processedAt: timestamp("processed_at"),
		error: text("error"),
		
		createdAt: timestamp("created_at").$defaultFn(() => new Date()).notNull(),
	},
	(t) => [
		index("webhook_events_provider_idx").on(t.provider),
		index("webhook_events_processed_idx").on(t.processed),
		index("webhook_events_created_at_idx").on(t.createdAt),
	]
);
```

---

## 9. 📋 CHECKLIST FINAL - AÇÕES NECESSÁRIAS

### 🔴 CRÍTICO (Sprint 1)

- [ ] **1. Adicionar campo `plan` na tabela `users`**
  - Criar enum `user_plans`
  - Adicionar campo `plan: userPlans("plan").default("none").notNull()`
  - Criar índice `users_plan_idx`

- [ ] **2. Criar tabela `conectMe`**
  - Tabela principal
  - Tabela de log (`connectMeLog`)

- [ ] **3. Criar tabelas de Cupons**
  - `promocoes`
  - `cupons`

- [ ] **4. Criar tabela `vinculosUnidade`**
  - Relação morador ↔ unidade ↔ condomínio

- [ ] **5. Ajustar perfis:**
  - `moradorProfiles`: Adicionar cpf, birthDate, endereço
  - `sindicoProfiles`: Adicionar cpf, phone, tempoCarreiraAnos, videoInstitucionalUrl
  - `empresaProfiles`: Adicionar phone, endereço completo, possuiSeguroResponsabilidadeCivil
  - `marketingProfiles`: Adicionar phone, city, state, slotsContratados
  - `vizinhancaProfiles`: Adicionar cnpj, phone, street, number, district, state, fotos, descricao

- [ ] **6. Criar tabela `anunciosMarketing`**
  - Sistema de "cadeiras" por estado

### 🟡 IMPORTANTE (Sprint 2-3)

- [ ] **7. Ajustar relacionamento Empresa ↔ Categorias**
  - Campo `categoriaPrincipalId` em `empresaProfiles`
  - Tabela `empresaSubcategorias` (máximo 5)

- [ ] **8. Migrar Categorias para Banco**
  - Criar migration/seed para popular `serviceCategories`
  - Atualizar frontend para buscar da API

- [ ] **9. Implementar lógica de trava trimestral**
  - Validação backend antes de upload
  - Setar `canBeReplacedAfter` automaticamente

- [ ] **10. Ajustar sistema de votação**
  - Trocar `apartmentNumber` (string) por FK para `vinculosUnidade`
  - Validar unidade pertence ao condomínio da assembleia

### 🟢 DESEJÁVEL (Sprint 4)

- [ ] **11. Criar tabelas de infraestrutura**
  - `auditLogs`
  - `rateLimits`
  - `webhookEvents`

- [ ] **12. Criar job/CRON para recalcular `recognitionLevel`**
  - Calcular periodicamente (diário/semanal)
  - Atualizar `sindicoProfiles.recognitionLevel`

---

## 10. 🎯 RESUMO DE COMPLIANCE COM O ESCOPO

| Sprint | Funcionalidade | Status | Pendências |
|--------|----------------|--------|------------|
| **Sprint 1** | Auth e Onboarding | ✅ 95% | Falta campo `plan` |
| **Sprint 1** | Perfis | ⚠️ 80% | Faltam campos em todos perfis |
| **Sprint 1** | Vídeos | ✅ 90% | Falta lógica trava trimestral |
| **Sprint 1** | Busca e Catálogo | ⚠️ 70% | Categorias ainda são enums |
| **Sprint 1** | Connect Me | ❌ 0% | **Tabela não existe** |
| **Sprint 2** | Assembleias | ⚠️ 75% | Falta ajustar votação |
| **Sprint 2** | Votação | ⚠️ 60% | Falta vincular unidades |
| **Sprint 3** | Cupons | ❌ 0% | **Tabelas não existem** |
| **Sprint 3** | Vizinhança | ✅ 80% | Faltam campos no perfil |
| **Sprint 4** | LMS | ✅ 100% | Completo ✅ |
| **Sprint 4** | Lives | ✅ 100% | Completo ✅ |
| **Sprint 4** | Fórum | ✅ 95% | Quase completo |
| **Sprint 4** | Marketing | ⚠️ 70% | Falta tabela anúncios |
| **Infra** | Audit/Webhooks | ⚠️ 50% | Faltam tabelas |

**Compliance Geral: 🟡 75%**

---

## 11. 🚀 PRÓXIMOS PASSOS RECOMENDADOS

### Fase 1: Corrigir Fundação (1-2 dias)
1. Adicionar campo `plan` na tabela `users`
2. Criar tabelas de Connect Me
3. Criar tabelas de Cupons
4. Criar tabela `vinculosUnidade`
5. Adicionar campos faltantes nos perfis

### Fase 2: Categorias e Relações (1 dia)
6. Criar migration para popular categorias
7. Ajustar relacionamento empresa ↔ categorias

### Fase 3: Regras de Negócio (1 dia)
8. Implementar validação de trava trimestral
9. Ajustar sistema de votação com unidades
10. Criar tabela `anunciosMarketing`

### Fase 4: Infraestrutura (1 dia)
11. Criar audit logs
12. Criar rate limiting
13. Criar webhooks

### Fase 5: Jobs e Automações (1 dia)
14. Job para recalcular reconhecimento de síndicos
15. Job para expirar cupons (4h)
16. Job para expirar tokens de assembleia

---

**Documento gerado por:** Droid (Factory AI)  
**Versão:** 1.0  
**Status:** ✅ Análise completa e detalhada  
**Nível de Criticidade:** 🔴 Alta - Ações imediatas necessárias
