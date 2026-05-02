# ANÁLISE TÉCNICA COMPLETA - REFATORAÇÃO OSO CLOUD

## ÍNDICE
1. [Situação Atual - O Que Existe HOJE](#1-situação-atual)
2. [Arquitetura Proposta - RBAC vs ABAC](#2-arquitetura-proposta)
3. [Estrutura de Banco de Dados - Mudanças Detalhadas](#3-estrutura-de-banco-de-dados)
4. [Impactos no Código Backend](#4-impactos-no-código-backend)
5. [Integração com Awilix (DI)](#5-integração-com-awilix)
6. [Ordem de Plugins Fastify](#6-ordem-de-plugins-fastify)
7. [Por Que Serviço + Plugin](#7-por-que-serviço--plugin)
8. [Contextos Globais vs Unitários](#8-contextos-globais-vs-unitários)
9. [Adaptação Frontend (SolidJS)](#9-adaptação-frontend-solidjs)
10. [Migração Detalhada - Passo a Passo](#10-migração-detalhada)
11. [Exemplos Práticos Reais](#11-exemplos-práticos-reais)

---

## 1. SITUAÇÃO ATUAL

### 1.1 O que EXISTE hoje no projeto

#### **A. Tabelas em `modules/billing/`**

**`permissions` table:**
```typescript
// src/infrastructure/database/drizzle/schema/billing/permissions.ts
{
  id: uuid,
  key: text,        // Ex: "create_video"
  name: text,       // Ex: "Criar Vídeo"
  description: text,
  category: text,   // Ex: "content" | "assembly" | "connect_me"
  createdAt: timestamp
}
```

**`plan_permissions` table** (junction):
```typescript
{
  id: uuid,
  planId: uuid → references plans.id,
  permissionId: uuid → references permissions.id,
  metadata: jsonb,
  createdAt: timestamp
}
```

**Problema:** Essas tabelas existem mas **NÃO estão sendo usadas**!

#### **B. PermissionService**

Localização: `modules/billing/domain/services/permission.service.ts`

```typescript
export class PermissionService {
  async can(context: PermissionContext, permission: PermissionKey): Promise<PermissionCheckResult>
  async canAll(context: PermissionContext, permissions: PermissionKey[]): Promise<PermissionCheckResult>
  async canAny(context: PermissionContext, permissions: PermissionKey[]): Promise<PermissionCheckResult>
  async getPermissions(context: PermissionContext): Promise<PermissionKey[]>
  async checkQuota(context: PermissionContext, resource: QuotaResource): Promise<QuotaCheckResult>
}
```

**Como funciona HOJE:**
- Usa pacote externo `mastersindico-schemas` 
- Funções como `hasPermission()`, `getQuota()` vêm do pacote compartilhado
- Não consulta banco de dados
- Implementa **ABAC** (Attribute-Based Access Control)
- Context:
  ```typescript
  interface PermissionContext {
    userId: string;
    plan: PlanKey | null;  // "resident_base", "syndic_n2", etc
    role: "user" | "admin" | "super_admin";
    permissions?: PermissionKey[];  // override manual
  }
  ```

**Problemas:**
1. Depende de pacote externo (pode desatualizar)
2. Não integrado com Oso Cloud
3. Localização errada (billing ao invés de auth)
4. Não rastreia uso de quotas (apenas retorna configs)

#### **C. OsoCloudService (Básico)**

Localização: `core/auth/oso-cloud.service.ts`

```typescript
export class OsoCloudService {
  private client: Oso;
  
  async can(userId, action, resourceType, resourceId): Promise<boolean>
  async tellRole(userId, role): Promise<void>
  async listPermissions(userId, resourceType, resourceId)
}
```

**Status:** Criado mas **não integrado** com:
- AuthModule
- PermissionService
- Fastify lifecycle
- Sistema de DI (Awilix)

#### **D. Tabela `users`**

```typescript
{
  id: uuid,
  name: text,
  email: text,
  emailVerified: boolean,
  username: text,
  image: text,
  phone: text,
  role: userRoles,  // ENUM: "none" | "resident" | "syndic" | "enterprise" | "marketing" | "local_company" | "admin"
  banned: boolean,
  banReason: text,
  banExpires: timestamp,
  ...
}
```

**Problema:** `role` é um ENUM simples (não multi-role).

#### **E. AuthService e JwtService**

**AuthService** (`modules/auth/domain/services/auth.service.ts`):
- Gerencia sessões
- Cria tokens
- Envia notificações

**JwtService** (`modules/auth/domain/services/jwt.service.ts`):
```typescript
interface JwtPayload {
  sub: string;  // userId
  sid: string;  // sessionId
  iat?: number;
  exp?: number;
}
```

**Problema:** JWT **não contém permissões** (apenas userId e sessionId).

#### **F. Hook de Autenticação**

`shared/hooks/authenticate.hook.ts`:
```typescript
export async function authenticate(request, reply) {
  const { getSessionUseCase, jwtService, userRepository, sessionRepository } = request.diScope.cradle;
  
  const token = jwtService.extractToken(request);
  const { session, user } = await getSessionUseCase.execute({ token });
  
  request.user = user;      // Entity
  request.session = session; // Entity
}
```

**Problema:** Não injeta permissões/roles no request.

---

## 2. ARQUITETURA PROPOSTA

### 2.1 RBAC vs ABAC - Quando Usar Cada Um

#### **A. RBAC (Role-Based Access Control)**

**Quando usar:**
- Permissões **globais** (não dependem de recurso específico)
- Permissões por **cargo/função**
- Exemplos:
  - "Um Syndic N3 pode publicar vídeos" → Permissão global
  - "Um Enterprise Pro pode criar cursos certificados" → Permissão global
  - "Um usuário pode acessar o fórum" → Permissão global

**Como implementar no Oso:**
```polar
# Actor
actor User {}

# Resource
resource Organization {}

# Role-based permissions
has_role(user: User, role: String);

# Regras RBAC
allow(user: User, "videos:upload:instructional", _org: Organization) if
  has_role(user, "enterprise_plus") or
  has_role(user, "enterprise_pro");

allow(user: User, "videos:publish", _org: Organization) if
  has_role(user, "syndic_n2") or
  has_role(user, "syndic_n3");
```

**Vantagens RBAC:**
- ✅ Simples de entender
- ✅ Fácil de auditar
- ✅ Performance excelente
- ✅ Ideal para permissões gerais

#### **B. ABAC (Attribute-Based Access Control)**

**Quando usar:**
- Permissões dependem de **atributos do recurso**
- Ownership (dono do recurso)
- Relações específicas
- Exemplos:
  - "Apenas o autor pode deletar o vídeo" → Ownership
  - "Membros do condomínio podem votar na assembleia" → Relação
  - "Aprovador pode aprovar documentos do seu setor" → Contexto

**Como implementar no Oso:**
```polar
resource Video {
  roles = ["author", "editor"];
  permissions = ["watch:full", "edit", "delete"];
  
  # ABAC: apenas o autor pode deletar
  "delete" if "author";
  "edit" if "author" or "editor";
}

# Relação author
has_relation(video: Video, "author", user: User);

# Regra ABAC
allow(user: User, "delete", video: Video) if
  has_relation(video, "author", user);
```

**Vantagens ABAC:**
- ✅ Permissões granulares
- ✅ Contexto do recurso
- ✅ Ownership natural
- ✅ Delegação de acesso

#### **C. HÍBRIDO (RBAC + ABAC) - Nossa Abordagem**

**Estratégia:**
1. **RBAC** para permissões globais (acesso a features)
2. **ABAC** para ownership e permissões específicas

**Exemplo prático:**
```polar
# RBAC: Enterprise pode fazer upload
allow(user: User, "videos:upload:instructional", org: Organization) if
  has_role(user, "enterprise_plus");

# ABAC: Mas só pode deletar seus próprios vídeos
resource Video {
  permissions = ["delete"];
  "delete" if "author";
}

allow(user: User, "delete", video: Video) if
  has_relation(video, "author", user);
```

**Quando usar cada um:**

| Cenário | Abordagem | Motivo |
|---------|-----------|--------|
| Pode acessar feature X? | RBAC | Depende apenas do plano/role |
| Pode deletar este vídeo? | ABAC | Depende de ser o autor |
| Pode publicar vídeos? | RBAC | Depende do plano |
| Pode editar este perfil? | ABAC | Depende de ser o dono |
| Pode votar nesta assembleia? | ABAC | Depende de ser membro do condomínio |
| Pode acessar fórum? | RBAC | Depende do plano |

---

### 2.2 Arquitetura em Camadas

```
┌─────────────────────────────────────────────────────────────┐
│                         FRONTEND                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  useAuth()   │  │ usePermission│  │   Guards     │      │
│  │  Context     │  │              │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            ▲
                            │ HTTP
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    BACKEND - ROUTES                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ authenticate │→ │  authorize   │  │   handler    │      │
│  │    hook      │  │    hook      │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    USE CASES LAYER                           │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  AuthorizationService (centralizado)                 │   │
│  │  - can(), authorize(), listResources()               │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│              INFRASTRUCTURE - Oso Client                     │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  OsoCloudService (wrapper do SDK)                    │   │
│  │  - authorize(), insert(), delete(), bulk()           │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
                      Oso Cloud API
```

---

## 3. ESTRUTURA DE BANCO DE DADOS

### 3.1 O Que SAI de `billing/`

**Tabelas a REMOVER:**
```typescript
❌ billing/permissions.ts          // Vai para Oso Cloud (fatos)
❌ billing/plan_permissions.ts     // Vai para Oso Cloud (fatos)
```

**Por quê?**
- Oso Cloud armazena permissões como **fatos** (facts)
- Não precisamos de tabela local para permissões
- Fonte única da verdade = Oso Cloud
- Sincronização automática via API

### 3.2 O Que FICA em `billing/`

```typescript
✅ billing/plans.ts               // Mantém (configuração de planos)
✅ billing/plan_features.ts       // Mantém (features UI/UX)
✅ billing/subscriptions.ts       // Mantém (billing do Stripe)
✅ billing/invoices.ts            // Mantém (faturas)
✅ billing/transactions.ts        // Mantém (pagamentos)
✅ billing/payment_methods.ts     // Mantém (métodos de pagamento)
```

### 3.3 O Que VAI PARA `auth/`

#### **A. Nova tabela: `roles`**

```typescript
// src/infrastructure/database/drizzle/schema/auth/roles.ts

import { index, jsonb, pgTable, text, timestamp, uuid } from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";

export const roles = pgTable(
  "roles",
  {
    id: uuid().primaryKey().$defaultFn(() => uuidv7()),
    
    // Identificação
    name: text().unique().notNull(), // "resident_base", "syndic_n2", etc
    displayName: text().notNull(),   // "Morador Base", "Síndico N2"
    description: text(),
    
    // Metadados (quotas, features, etc)
    metadata: jsonb().$type<{
      quotas?: Record<string, { limit: number; period: string }>;
      features?: string[];
      color?: string;
    }>(),
    
    // Timestamps
    createdAt: timestamp().$defaultFn(() => new Date()).notNull(),
    updatedAt: timestamp().$defaultFn(() => new Date()).$onUpdate(() => new Date()).notNull(),
  },
  (t) => [
    index("roles_name_idx").on(t.name),
  ]
);
```

#### **B. Nova tabela: `user_roles` (Many-to-Many)**

```typescript
// src/infrastructure/database/drizzle/schema/auth/user-roles.ts

import { index, pgTable, timestamp, uuid } from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";
import { users } from "../users/users";
import { roles } from "./roles";

export const userRoles = pgTable(
  "user_roles",
  {
    id: uuid().primaryKey().$defaultFn(() => uuidv7()),
    
    // Relações
    userId: uuid().notNull().references(() => users.id, { onDelete: "cascade" }),
    roleId: uuid().notNull().references(() => roles.id, { onDelete: "cascade" }),
    
    // Controle
    expiresAt: timestamp(), // Opcional: role temporária
    deletedAt: timestamp(), // Soft delete
    
    // Timestamps
    createdAt: timestamp().$defaultFn(() => new Date()).notNull(),
    updatedAt: timestamp().$defaultFn(() => new Date()).$onUpdate(() => new Date()).notNull(),
  },
  (t) => [
    index("user_roles_user_id_idx").on(t.userId),
    index("user_roles_role_id_idx").on(t.roleId),
    index("user_roles_user_role_idx").on(t.userId, t.roleId),
  ]
);
```

**Por quê Many-to-Many?**
- Um usuário pode ter **múltiplas roles** (ex: Syndic + Enterprise)
- Roles temporárias (expiresAt)
- Histórico (soft delete)

### 3.4 Mudanças na Tabela `users`

**ANTES:**
```typescript
export const users = pgTable("users", {
  ...
  role: userRoles("role").default("none").notNull(), // ❌ ENUM ÚNICO
  ...
});
```

**DEPOIS:**
```typescript
export const users = pgTable("users", {
  ...
  // ❌ REMOVER campo role (agora vem da relação user_roles)
  ...
});
```

**Migração:**
```sql
-- 1. Criar tabelas novas
CREATE TABLE roles (...);
CREATE TABLE user_roles (...);

-- 2. Migrar roles existentes
INSERT INTO roles (name, display_name) VALUES
  ('resident', 'Morador'),
  ('syndic', 'Síndico'),
  ('enterprise', 'Empresa'),
  ('admin', 'Administrador');

-- 3. Migrar relações users → roles
INSERT INTO user_roles (user_id, role_id)
SELECT u.id, r.id
FROM users u
JOIN roles r ON r.name = u.role
WHERE u.role != 'none';

-- 4. Remover coluna antiga
ALTER TABLE users DROP COLUMN role;
```

### 3.5 Novo `relations.ts`

```typescript
// src/infrastructure/database/drizzle/schema/relations.ts

import { relations } from "drizzle-orm";
import * as schema from "./index";

export const userRelations = relations(schema.users, ({ many }) => ({
  // Many-to-many com roles
  userRoles: many(schema.userRoles),
  
  // Outras relações existentes
  sessions: many(schema.sessions),
  subscriptions: many(schema.userSubscriptions),
  videos: many(schema.videos),
  ...
}));

export const roleRelations = relations(schema.roles, ({ many }) => ({
  userRoles: many(schema.userRoles),
}));

export const userRoleRelations = relations(schema.userRoles, ({ one }) => ({
  user: one(schema.users, {
    fields: [schema.userRoles.userId],
    references: [schema.users.id],
  }),
  role: one(schema.roles, {
    fields: [schema.userRoles.roleId],
    references: [schema.roles.id],
  }),
}));

// ❌ REMOVER relações antigas de permissions
// ❌ REMOVER: planPermissions relations
// ❌ REMOVER: permissions relations
```

---

## 4. IMPACTOS NO CÓDIGO BACKEND

### 4.1 AuthService - O Que Muda

**ANTES:**
```typescript
// modules/auth/domain/services/auth.service.ts
export class AuthService {
  constructor(private deps: {
    sessionRepository: ISessionRepository;
    jwtService: JwtTokenService;
    emailProvider: IEmailProvider;
  }) {}
  
  async createAuthenticatedSession(user, metadata): Promise<{ session, accessToken, expiresAt }> {
    const session = this.createSession(user, metadata);
    const { accessToken, expiresAt } = await this.deps.jwtService.generateToken(
      user.getId(),
      session.getId(),
      session.getExpiresAt()
    );
    return { session, accessToken, expiresAt };
  }
}
```

**DEPOIS (SEM MUDANÇAS!):**
```typescript
// NÃO MUDA! AuthService continua focado em sessões e tokens
export class AuthService {
  // ... mesma interface
}
```

**Por quê não muda?**
- AuthService cuida de **autenticação** (quem você é)
- AuthorizationService cuida de **autorização** (o que você pode fazer)
- Princípio da Responsabilidade Única (SRP)

### 4.2 JwtService - O Que Muda

**ANTES:**
```typescript
interface JwtPayload {
  sub: string;  // userId
  sid: string;  // sessionId
  iat?: number;
  exp?: number;
}
```

**OPÇÃO 1: NÃO ADICIONAR ROLES NO JWT (Recomendado)**

```typescript
// JWT continua SLIM (apenas userId + sessionId)
interface JwtPayload {
  sub: string;  // userId
  sid: string;  // sessionId
  iat?: number;
  exp?: number;
}

// Roles vêm do banco a cada request via authenticate hook
```

**Vantagens:**
- ✅ JWT não precisa ser reemitido quando roles mudam
- ✅ Sempre sincronizado com o banco
- ✅ Sem cache de permissões desatualizadas
- ✅ Audit trail completo

**Desvantagens:**
- ⚠️ 1 query extra por request (SELECT roles WHERE userId)

**OPÇÃO 2: ADICIONAR ROLES NO JWT (Não recomendado)**

```typescript
interface JwtPayload {
  sub: string;
  sid: string;
  roles: string[]; // ["resident_base", "syndic_n2"]
  iat?: number;
  exp?: number;
}
```

**Vantagens:**
- ✅ Sem query extra

**Desvantagens:**
- ❌ Precisa reemitir JWT quando roles mudam
- ❌ Possível dessincronia entre JWT e banco
- ❌ JWT maior (+ banda)

**DECISÃO: Opção 1 (JWT slim)**

### 4.3 Hook `authenticate.ts` - O Que Muda

**ANTES:**
```typescript
export async function authenticate(request, reply) {
  const { getSessionUseCase, jwtService, userRepository, sessionRepository } = request.diScope.cradle;
  
  const token = jwtService.extractToken(request);
  const { session, user } = await getSessionUseCase.execute({ token });
  
  request.user = user;
  request.session = session;
}
```

**DEPOIS:**
```typescript
export async function authenticate(request, reply) {
  const { 
    getSessionUseCase, 
    jwtService, 
    userRepository, 
    sessionRepository,
    roleRepository,          // 🆕 NOVO
    syncService              // 🆕 NOVO (Oso sync)
  } = request.diScope.cradle;
  
  const token = jwtService.extractToken(request);
  if (!token) throw new UnauthorizedError("Token ausente");
  
  const { session, user } = await getSessionUseCase.execute({ token });
  
  // 🆕 Buscar roles do usuário
  const userRoles = await roleRepository.findByUserId(user.getId());
  
  // 🆕 Sincronizar roles com Oso (garantir consistência)
  // Isso é idempotente e rápido (Oso cacheia)
  await syncService.syncUserRoles(user.getId(), userRoles.map(r => r.name));
  
  // Injetar no request
  request.user = user;
  request.session = session;
  request.roles = userRoles;        // 🆕 NOVO
  request.permissions = undefined;  // 🆕 Lazy load via Oso
}
```

**Mudanças:**
1. ✅ Busca roles do banco
2. ✅ Sincroniza roles com Oso
3. ✅ Injeta roles no request
4. ✅ Permissions vêm lazy via Oso (só quando necessário)

### 4.4 Novo Hook `authorize.ts`

```typescript
// src/shared/hooks/authorize.hook.ts

import type { FastifyReply, FastifyRequest } from "fastify";
import { ForbiddenError } from "../../core/errors";

interface AuthorizeOptions {
  action: string;
  resourceType: string;
  resourceId?: string | ((req: FastifyRequest) => string);
}

export function authorize(options: AuthorizeOptions) {
  return async (request: FastifyRequest, _reply: FastifyReply) => {
    const { authorizationService } = request.diScope.cradle;
    
    // Extrair resourceId
    const resourceId = typeof options.resourceId === 'function'
      ? options.resourceId(request)
      : options.resourceId;
    
    // Verificar permissão
    const result = await authorizationService.authorize(
      { userId: request.user.getId() },
      options.action,
      options.resourceType,
      resourceId
    );
    
    if (!result.allowed) {
      throw new ForbiddenError(result.reason || "Acesso negado");
    }
  };
}

// Variante para múltiplas permissões (AND)
export function authorizeAll(checks: AuthorizeOptions[]) {
  return async (request: FastifyRequest, _reply: FastifyReply) => {
    const { authorizationService } = request.diScope.cradle;
    
    for (const check of checks) {
      const resourceId = typeof check.resourceId === 'function'
        ? check.resourceId(request)
        : check.resourceId;
        
      const result = await authorizationService.authorize(
        { userId: request.user.getId() },
        check.action,
        check.resourceType,
        resourceId
      );
      
      if (!result.allowed) {
        throw new ForbiddenError(result.reason || "Acesso negado");
      }
    }
  };
}

// Variante para múltiplas permissões (OR)
export function authorizeAny(checks: AuthorizeOptions[]) {
  return async (request: FastifyRequest, _reply: FastifyReply) => {
    const { authorizationService } = request.diScope.cradle;
    
    let lastError: string | undefined;
    
    for (const check of checks) {
      const resourceId = typeof check.resourceId === 'function'
        ? check.resourceId(request)
        : check.resourceId;
        
      const result = await authorizationService.authorize(
        { userId: request.user.getId() },
        check.action,
        check.resourceType,
        resourceId
      );
      
      if (result.allowed) return; // Sucesso!
      lastError = result.reason;
    }
    
    throw new ForbiddenError(lastError || "Acesso negado");
  };
}
```

### 4.5 Novo AuthorizationService

```typescript
// src/modules/auth/domain/services/authorization.service.ts

import type { Oso } from "oso-cloud";

interface AuthContext {
  userId: string;
  roles?: string[];
}

interface AuthResult {
  allowed: boolean;
  reason?: string;
}

export class AuthorizationService {
  constructor(
    private readonly oso: Oso,
  ) {}
  
  // ==================== PERMISSÕES ====================
  
  async can(
    context: AuthContext,
    action: string,
    resourceType: string,
    resourceId?: string
  ): Promise<boolean> {
    try {
      return await this.oso.authorize(
        { type: "User", id: context.userId },
        action,
        { type: resourceType, id: resourceId || "global" }
      );
    } catch (error) {
      // Log error mas retorna false (fail-safe)
      console.error("Oso authorization error:", error);
      return false;
    }
  }
  
  async authorize(
    context: AuthContext,
    action: string,
    resourceType: string,
    resourceId?: string
  ): Promise<AuthResult> {
    const allowed = await this.can(context, action, resourceType, resourceId);
    
    return {
      allowed,
      reason: allowed ? undefined : `Permissão negada: ${action} em ${resourceType}`
    };
  }
  
  async canAll(
    context: AuthContext,
    checks: Array<{ action: string; resourceType: string; resourceId?: string }>
  ): Promise<boolean> {
    for (const check of checks) {
      const allowed = await this.can(
        context,
        check.action,
        check.resourceType,
        check.resourceId
      );
      if (!allowed) return false;
    }
    return true;
  }
  
  async canAny(
    context: AuthContext,
    checks: Array<{ action: string; resourceType: string; resourceId?: string }>
  ): Promise<boolean> {
    for (const check of checks) {
      const allowed = await this.can(
        context,
        check.action,
        check.resourceType,
        check.resourceId
      );
      if (allowed) return true;
    }
    return false;
  }
  
  // ==================== RECURSOS ====================
  
  async listResources(
    context: AuthContext,
    action: string,
    resourceType: string
  ): Promise<string[]> {
    try {
      const result = await this.oso.list(
        { type: "User", id: context.userId },
        action,
        resourceType
      );
      return result;
    } catch (error) {
      console.error("Oso list error:", error);
      return [];
    }
  }
  
  async actions(
    context: AuthContext,
    resourceType: string,
    resourceId: string
  ): Promise<string[]> {
    try {
      return await this.oso.actions(
        { type: "User", id: context.userId },
        { type: resourceType, id: resourceId }
      );
    } catch (error) {
      console.error("Oso actions error:", error);
      return [];
    }
  }
}
```

---

## 5. INTEGRAÇÃO COM AWILIX

### 5.1 Mudanças no `auth.module.ts`

**ANTES:**
```typescript
// modules/auth/auth.module.ts

export class AuthModule implements Module {
  async register(app: FastifyInstance) {
    diContainer.register({
      // Repositórios
      accountRepository: asClass(AccountRepositoryImpl, { lifetime: Lifetime.SCOPED }),
      sessionRepository: asClass(SessionRepositoryImpl, { lifetime: Lifetime.SCOPED }),
      
      // Serviços
      authService: asClass(AuthService, { lifetime: Lifetime.SCOPED }),
      jwtService: asClass(JwtTokenService, { lifetime: Lifetime.SINGLETON }),
      
      // Use Cases
      signInUseCase: asClass(SignInUseCase, { lifetime: Lifetime.SCOPED }),
      ...
    });
  }
}
```

**DEPOIS:**
```typescript
// modules/auth/auth.module.ts

import { OsoCloudService } from "../../core/auth/oso-cloud.service";
import { AuthorizationService } from "./domain/services/authorization.service";
import { SyncService } from "../../infrastructure/authorization/sync/sync-service";
import { RoleRepositoryImpl } from "./infrastructure/database/drizzle/repositories/role.repository.impl";

export class AuthModule implements Module {
  async register(app: FastifyInstance) {
    diContainer.register({
      // ==================== REPOSITÓRIOS ====================
      accountRepository: asClass(AccountRepositoryImpl, { lifetime: Lifetime.SCOPED }),
      sessionRepository: asClass(SessionRepositoryImpl, { lifetime: Lifetime.SCOPED }),
      roleRepository: asClass(RoleRepositoryImpl, { lifetime: Lifetime.SCOPED }), // 🆕 NOVO
      
      // ==================== DOMAIN SERVICES ====================
      authService: asClass(AuthService, { lifetime: Lifetime.SCOPED }),
      jwtService: asClass(JwtTokenService, { lifetime: Lifetime.SINGLETON }),
      
      // 🆕 NOVOS SERVIÇOS DE AUTORIZAÇÃO
      osoCloudService: asClass(OsoCloudService, { lifetime: Lifetime.SINGLETON }), // SINGLETON!
      authorizationService: asClass(AuthorizationService, { lifetime: Lifetime.SCOPED }),
      syncService: asClass(SyncService, { lifetime: Lifetime.SCOPED }),
      
      // ==================== USE CASES ====================
      signInUseCase: asClass(SignInUseCase, { lifetime: Lifetime.SCOPED }),
      ...
    });
  }
}
```

**Mudanças:**
1. ✅ `roleRepository` → SCOPED (depende de UnitOfWork)
2. ✅ `osoCloudService` → **SINGLETON** (cliente compartilhado)
3. ✅ `authorizationService` → SCOPED (usa logger do request)
4. ✅ `syncService` → SCOPED (usa logger do request)

### 5.2 Por Que OsoCloudService é SINGLETON?

**Motivo:**
```typescript
export class OsoCloudService {
  private client: Oso;  // 👈 Cliente compartilhado
  
  constructor() {
    this.client = new Oso(env.OSO_URL, env.OSO_API_KEY);
    // 👆 Conexão HTTP reutilizável (connection pool interno)
  }
}
```

**Vantagens:**
- ✅ Uma única conexão HTTP (connection pooling)
- ✅ Menos overhead
- ✅ Sem estado (stateless)
- ✅ Thread-safe

**Alternativa (SCOPED):**
```typescript
// ❌ NÃO FAZER
osoCloudService: asClass(OsoCloudService, { lifetime: Lifetime.SCOPED })
// Problema: cria nova conexão HTTP a cada request!
```

### 5.3 Mudanças no `billing.module.ts`

**ANTES:**
```typescript
export class BillingModule implements Module {
  async register(app: FastifyInstance) {
    diContainer.register({
      permissionService: asClass(PermissionService, { lifetime: Lifetime.SCOPED }), // ❌ REMOVER
      ...
    });
  }
}
```

**DEPOIS:**
```typescript
export class BillingModule implements Module {
  async register(app: FastifyInstance) {
    diContainer.register({
      // ❌ REMOVER permissionService (agora está em auth!)
      
      // Manter apenas billing-specific services
      planRepository: asClass(PlanRepositoryImpl, { lifetime: Lifetime.SCOPED }),
      subscriptionRepository: asClass(SubscriptionRepositoryImpl, { lifetime: Lifetime.SCOPED }),
      paymentProvider: asClass(StripeProvider, { lifetime: Lifetime.SINGLETON }),
      ...
    });
  }
}
```

---

## 6. ORDEM DE PLUGINS FASTIFY

### 6.1 Por Que a Ordem Importa?

Fastify executa plugins **na ordem de registro**. A ordem afeta:
- Decorators disponíveis
- Hooks executados
- Error handlers

### 6.2 Ordem Atual vs Nova Ordem

**ATUAL:**
```typescript
// app.ts
export async function AppServer() {
  const app = Fastify({ logger: ... });
  
  // 1. FUNDAÇÃO
  await app.register(awilixPlugin);       // DI container
  await app.register(loggerPlugin);       // Logger
  await app.register(middlePlugin);       // Middleware support
  
  // 2. INFRAESTRUTURA
  await app.register(redisPlugin);        // Cache
  
  // 3. SEGURANÇA
  await app.register(corsPlugin);
  await app.register(helmetPlugin);
  await app.register(rateLimitPlugin);
  
  // 4. DOCUMENTAÇÃO
  await app.register(swaggerPlugin);
  
  // 5. PARSERS
  await app.register(rawBodyPlugin);
  await app.register(multipartPlugin);
  await app.register(cookiePlugin);
  await app.register(schedulePlugin);
  
  // 6. ERROR HANDLER
  app.setErrorHandler(errorHandler);
  
  // 7. MÓDULOS
  const appModule = new AppModule();
  await appModule.register(app);
  await appModule.bootstrap(app);
  
  return app;
}
```

**NOVA ORDEM (COM OSO PLUGIN):**
```typescript
// app.ts
export async function AppServer() {
  const app = Fastify({ logger: ... });
  
  // 1. FUNDAÇÃO
  await app.register(awilixPlugin);       // DI container (primeiro!)
  await app.register(loggerPlugin);       // Logger
  await app.register(middlePlugin);       // Middleware support
  
  // 2. INFRAESTRUTURA
  await app.register(redisPlugin);        // Cache
  // 🆕 NOVO: Plugin Oso ANTES dos módulos
  await app.register(osoPlugin);          // Oso Client (singleton)
  
  // 3. SEGURANÇA
  await app.register(corsPlugin);
  await app.register(helmetPlugin);
  await app.register(rateLimitPlugin);
  
  // 4. DOCUMENTAÇÃO
  await app.register(swaggerPlugin);
  
  // 5. PARSERS
  await app.register(rawBodyPlugin);
  await app.register(multipartPlugin);
  await app.register(cookiePlugin);
  await app.register(schedulePlugin);
  
  // 6. ERROR HANDLER
  app.setErrorHandler(errorHandler);
  
  // 7. MÓDULOS
  const appModule = new AppModule();
  await appModule.register(app);         // AuthModule registra serviços
  await appModule.bootstrap(app);
  
  return app;
}
```

**Mudança chave:** Plugin Oso registrado **ANTES** dos módulos (mas DEPOIS do Awilix).

### 6.3 Por Que Oso Plugin ANTES dos Módulos?

```
awilixPlugin → Cria diContainer
     ↓
osoPlugin → Registra osoClient no container
     ↓
AuthModule → Usa osoClient via DI
```

**Se inverter:**
```
awilixPlugin → Cria diContainer
     ↓
AuthModule → Tenta usar osoClient... ❌ NÃO EXISTE AINDA!
     ↓
osoPlugin → Registra osoClient (tarde demais)
```

---

## 7. POR QUE SERVIÇO + PLUGIN?

### 7.1 Arquitetura Proposta

```
┌─────────────────────────────────────────┐
│       shared/plugins/oso.plugin.ts       │  👈 Plugin Fastify
│  - Registra Oso client no DI            │
│  - Decora request com helpers           │
│  - Configuração global                  │
└─────────────────────────────────────────┘
              │ injeta
              ▼
┌─────────────────────────────────────────┐
│    modules/auth/domain/services/        │
│    authorization.service.ts              │  👈 Domain Service
│  - Lógica de negócio                    │
│  - can(), authorize(), listResources()  │
│  - Abstração sobre Oso                  │
└─────────────────────────────────────────┘
              │ usa
              ▼
┌─────────────────────────────────────────┐
│    core/auth/oso-cloud.service.ts       │  👈 Infrastructure
│  - Wrapper do SDK Oso                   │
│  - authorize(), insert(), delete()      │
│  - Cliente HTTP                         │
└─────────────────────────────────────────┘
```

### 7.2 Responsabilidades

#### **A. Plugin (`oso.plugin.ts`)**

**Responsabilidade:** Integração com Fastify lifecycle

```typescript
// src/shared/plugins/oso.plugin.ts

import fp from "fastify-plugin";
import type { FastifyPluginAsync } from "fastify";
import { Oso } from "oso-cloud";
import { env } from "../config";

const osoPlugin: FastifyPluginAsync = fp(async (fastify) => {
  // 1. Criar cliente Oso (singleton)
  const osoClient = new Oso(env.OSO_URL, env.OSO_API_KEY);
  
  // 2. Registrar no DI container
  fastify.diContainer.register({
    oso: asValue(osoClient)
  });
  
  // 3. Decorar request com helpers (opcional, conveniente)
  fastify.decorateRequest('authorize', null);
  fastify.decorateRequest('can', null);
  fastify.decorateRequest('listResources', null);
  fastify.decorateRequest('actions', null);
  
  // 4. Hook para injetar helpers em cada request
  fastify.addHook('onRequest', async (request) => {
    const { authorizationService } = request.diScope.cradle;
    
    // Helper: request.can(action, resourceType, resourceId)
    request.can = async (action: string, resourceType: string, resourceId?: string) => {
      if (!request.user) return false;
      return authorizationService.can(
        { userId: request.user.getId() },
        action,
        resourceType,
        resourceId
      );
    };
    
    // Helper: request.authorize(action, resourceType, resourceId)
    request.authorize = async (action: string, resourceType: string, resourceId?: string) => {
      if (!request.user) throw new UnauthorizedError();
      return authorizationService.authorize(
        { userId: request.user.getId() },
        action,
        resourceType,
        resourceId
      );
    };
    
    // Helper: request.listResources(action, resourceType)
    request.listResources = async (action: string, resourceType: string) => {
      if (!request.user) return [];
      return authorizationService.listResources(
        { userId: request.user.getId() },
        action,
        resourceType
      );
    };
    
    // Helper: request.actions(resourceType, resourceId)
    request.actions = async (resourceType: string, resourceId: string) => {
      if (!request.user) return [];
      return authorizationService.actions(
        { userId: request.user.getId() },
        resourceType,
        resourceId
      );
    };
  });
  
  // 5. Lifecycle: cleanup on close
  fastify.addHook('onClose', async () => {
    // Oso SDK não precisa de cleanup manual
  });
});

export default osoPlugin;
```

**Vantagens:**
- ✅ Integração com lifecycle do Fastify
- ✅ Helpers convenientes no request
- ✅ Cleanup automático
- ✅ Singleton garantido

#### **B. AuthorizationService**

**Responsabilidade:** Lógica de domínio de autorização

```typescript
// Já mostrado anteriormente
export class AuthorizationService {
  constructor(private readonly oso: Oso) {}
  
  async can(...) { ... }
  async authorize(...) { ... }
  async canAll(...) { ... }
  async canAny(...) { ... }
}
```

**Vantagens:**
- ✅ Testável (mock do Oso)
- ✅ Reutilizável (Use Cases, Jobs, etc)
- ✅ Lógica de negócio centralizada
- ✅ Abstração (oculta detalhes do Oso)

#### **C. OsoCloudService**

**Responsabilidade:** Wrapper do SDK Oso

```typescript
// Já mostrado anteriormente
export class OsoCloudService {
  private client: Oso;
  
  async can(...) { ... }
  async tellRole(...) { ... }
  async listPermissions(...) { ... }
}
```

**Vantagens:**
- ✅ Abstração sobre SDK externo
- ✅ Fácil trocar provider no futuro
- ✅ Tipos TypeScript
- ✅ Error handling centralizado

### 7.3 Por Que Não Só o Plugin?

**Problema com apenas plugin:**
```typescript
// ❌ BAD: Use Case dependendo de helpers do request
class PublishVideoUseCase {
  async execute(request: FastifyRequest, data: unknown) {
    // ❌ Acoplado ao Fastify!
    if (!await request.can('videos:publish', 'Organization')) {
      throw new ForbiddenError();
    }
  }
}
```

**Solução com AuthorizationService:**
```typescript
// ✅ GOOD: Use Case agnóstico de framework
class PublishVideoUseCase {
  constructor(
    private readonly authorizationService: AuthorizationService
  ) {}
  
  async execute(context: { userId: string }, data: unknown) {
    // ✅ Não depende de Fastify!
    if (!await this.authorizationService.can(context, 'videos:publish', 'Organization')) {
      throw new ForbiddenError();
    }
  }
}
```

**Vantagens:**
- ✅ Use Cases testáveis (sem mock de Fastify)
- ✅ Reutilizável em Jobs, CLIs, etc
- ✅ Clean Architecture

---

## 8. CONTEXTOS GLOBAIS VS UNITÁRIOS

### 8.1 O Que São?

#### **A. Contextos Globais**

**Definição:** Permissões que NÃO dependem de um recurso específico.

**Exemplos:**
- "Pode acessar o fórum?" → Depende do plano, não de um fórum específico
- "Pode fazer upload de vídeos instrucionais?" → Depende do plano
- "Pode usar Connect Me?" → Depende do plano

**Como verificar:**
```typescript
// Contexto global: resourceId = undefined ou "global"
const canAccessForum = await authService.can(
  { userId },
  'forum:access',
  'Organization'  // ← Tipo genérico, sem ID específico
);
```

**Polar:**
```polar
# Regra global: qualquer Organization
allow(user: User, "forum:access", _org: Organization) if
  has_role(user, "syndic_n1") or
  has_role(user, "syndic_n2") or
  has_role(user, "syndic_n3");
```

#### **B. Contextos Unitários**

**Definição:** Permissões que DEPENDEM de um recurso específico.

**Exemplos:**
- "Pode deletar ESTE vídeo?" → Precisa ser o autor
- "Pode editar ESTE perfil?" → Precisa ser o dono
- "Pode votar NESTA assembleia?" → Precisa ser membro do condomínio

**Como verificar:**
```typescript
// Contexto unitário: resourceId específico
const canDeleteVideo = await authService.can(
  { userId },
  'delete',
  'Video',
  videoId  // ← ID específico do recurso
);
```

**Polar:**
```polar
resource Video {
  permissions = ["delete"];
  roles = ["author"];
  
  # Apenas o autor pode deletar
  "delete" if "author";
}

# Relação author
has_relation(video: Video, "author", user: User);

# Regra unitária
allow(user: User, "delete", video: Video) if
  has_relation(video, "author", user);
```

### 8.2 Quando Usar Cada Um?

| Cenário | Tipo | Exemplo |
|---------|------|---------|
| Feature do plano | Global | `can('forum:access', 'Organization')` |
| Quota | Global | `can('videos:publish', 'Organization')` |
| Ownership | Unitário | `can('delete', 'Video', videoId)` |
| Membership | Unitário | `can('vote', 'Assembly', assemblyId)` |
| Delegação | Unitário | `can('edit', 'Profile', profileId)` |

### 8.3 Exemplo Prático Completo

**Caso de Uso:** Publicar vídeo

```typescript
class PublishVideoUseCase {
  constructor(
    private readonly authService: AuthorizationService,
    private readonly videoRepository: IVideoRepository,
    private readonly syncService: SyncService
  ) {}
  
  async execute(context: { userId: string }, data: CreateVideoDto) {
    // 1. CONTEXTO GLOBAL: Pode fazer upload de vídeos instrucionais?
    const canUpload = await this.authService.can(
      context,
      'videos:upload:instructional',
      'Organization'  // ← Global
    );
    
    if (!canUpload) {
      throw new ForbiddenError('Seu plano não permite upload de vídeos instrucionais');
    }
    
    // 2. Criar vídeo no banco
    const video = await this.videoRepository.create({
      ...data,
      authorUserId: context.userId
    });
    
    // 3. SINCRONIZAR OWNERSHIP com Oso (contexto unitário)
    await this.syncService.setOwnership(
      'Video',
      video.id,
      context.userId,
      'author'  // ← Relação
    );
    
    return video;
  }
}

// Depois, para deletar:
class DeleteVideoUseCase {
  async execute(context: { userId: string }, videoId: string) {
    // CONTEXTO UNITÁRIO: Pode deletar ESTE vídeo específico?
    const canDelete = await this.authService.can(
      context,
      'delete',
      'Video',
      videoId  // ← Unitário
    );
    
    if (!canDelete) {
      throw new ForbiddenError('Apenas o autor pode deletar este vídeo');
    }
    
    await this.videoRepository.delete(videoId);
    
    // Cleanup: remover ownership do Oso
    await this.syncService.removeOwnership('Video', videoId, context.userId, 'author');
  }
}
```

### 8.4 Como Estruturar Polar

```polar
# ===================================
# ACTORS
# ===================================
actor User {}

# ===================================
# RESOURCES GLOBAIS (sem ownership)
# ===================================
resource Organization {
  permissions = [
    "videos:upload:instructional",
    "videos:upload:institutional",
    "videos:publish",
    "forum:access",
    "connect_me:use"
  ];
}

# Permissões globais baseadas em role
allow(user: User, "videos:upload:instructional", _org: Organization) if
  has_role(user, "enterprise_plus") or
  has_role(user, "enterprise_pro");

allow(user: User, "forum:access", _org: Organization) if
  has_role(user, "syndic_n1") or
  has_role(user, "syndic_n2") or
  has_role(user, "syndic_n3");

# ===================================
# RESOURCES UNITÁRIOS (com ownership)
# ===================================
resource Video {
  permissions = ["watch:full", "watch:preview", "edit", "delete"];
  roles = ["author", "editor"];
  
  # Ownership rules
  "edit" if "author" or "editor";
  "delete" if "author";
}

# Relações
has_relation(video: Video, "author", user: User);
has_relation(video: Video, "editor", user: User);

# Regras unitárias
allow(user: User, action, video: Video) if
  has_permission(video, action, user);

# ===================================
# FACTS (sincronizados do backend)
# ===================================
has_role(user: User, role: String);  # Ex: has_role("user-123", "syndic_n2")
has_relation(video: Video, "author", user: User);  # Ex: has_relation("video-456", "author", "user-123")
```

---

## 9. ADAPTAÇÃO FRONTEND (SOLIDJS)

### 9.1 Situação Atual

O usuário usa **SolidJS** (não React!). Código atual:

```typescript
// auth-context.tsx (atual)
export interface AuthContextType {
  user: Accessor<User | null>;
  session: Accessor<SessionResponseDto | null>;
  isAuthenticated: Accessor<boolean>;
  isLoading: Accessor<boolean>;
  signIn: (input: SignInDto) => Promise<AuthResponseDto>;
  signOut: () => Promise<void>;
  refreshSession: () => Promise<void>;
}

// useAuth.ts (atual)
export function useAuthGuard() {
  const auth = useAuth();
  
  return {
    user: auth.user,
    session: auth.session,
    isAuthenticated: createMemo(() => !!auth.user()),
    isLoading: auth.isLoading,
    signIn: auth.signIn,
    signOut: auth.signOut,
    refresh: auth.refreshSession,
  };
}
```

### 9.2 Mudanças no Backend → DTO

**Novo `AuthResponseDto`:**
```typescript
// mastersindico-schemas/src/dtos/auth.dto.ts

export interface User {
  id: string;
  name: string;
  email: string;
  username: string;
  image?: string;
  emailVerified: boolean;
  
  // 🆕 NOVO: roles do usuário
  roles: string[];  // ["resident_base", "syndic_n2"]
  
  // 🆕 NOVO: permissões globais (cache)
  permissions?: string[];  // ["videos:publish", "forum:access"]
}

export interface AuthResponseDto {
  user: User;
  session: SessionResponseDto;
}
```

**Backend retorna:**
```typescript
// modules/auth/application/use-cases/get-session.use-case.ts

export class GetSessionUseCase {
  async execute({ token }: { token: string }) {
    const payload = await this.jwtService.verifyToken(token);
    const user = await this.userRepository.findById(payload.sub);
    const session = await this.sessionRepository.findOneById(payload.sid);
    
    // 🆕 Buscar roles
    const userRoles = await this.roleRepository.findByUserId(user.getId());
    
    // 🆕 Opcional: buscar permissões globais (cache para UI)
    const permissions = await this.authService.getPermissions({
      userId: user.getId()
    });
    
    return {
      user: {
        id: user.getId(),
        name: user.getName(),
        email: user.getEmail().getValue(),
        username: user.getUsername().getValue(),
        emailVerified: user.getEmailVerified(),
        roles: userRoles.map(r => r.name),  // 🆕
        permissions,                         // 🆕
      },
      session: {
        id: session.getId(),
        expiresAt: session.getExpiresAt(),
        ...
      }
    };
  }
}
```

### 9.3 Atualizar Frontend AuthContext

```typescript
// src/context/auth-context.tsx

import { useMutation, useQuery, useQueryClient } from "@tanstack/solid-query";
import type { AxiosError } from "axios";
import type {
  AuthResponseDto,
  SessionResponseDto,
  SignInDto,
  SignUpDto,
  SignUpResponseDto,
  User,
} from "mastersindico-schemas";
import * as Solid from "solid-js";
import { AxiosConfig } from "../lib/axios";

export interface AuthContextType {
  user: Solid.Accessor<User | null>;
  session: Solid.Accessor<SessionResponseDto | null>;
  isAuthenticated: Solid.Accessor<boolean>;
  isLoading: Solid.Accessor<boolean>;
  signIn: (input: SignInDto) => Promise<AuthResponseDto>;
  signUp: (input: SignUpDto) => Promise<SignUpResponseDto>;
  signOut: () => Promise<void>;
  refreshSession: () => Promise<void>;
  
  // 🆕 NOVOS MÉTODOS DE PERMISSÃO
  hasRole: (role: string) => boolean;
  hasAnyRole: (roles: string[]) => boolean;
  hasAllRoles: (roles: string[]) => boolean;
  hasPermission: (permission: string) => boolean;
  hasAnyPermission: (permissions: string[]) => boolean;
  can: (action: string) => boolean;  // Alias para hasPermission
}

export function AuthProvider(props: { children: Solid.JSX.Element }) {
  const queryClient = useQueryClient();

  const sessionQuery = useQuery(() => ({
    queryKey: ["auth-session"],
    queryFn: async () => {
      try {
        const { data } = await AxiosConfig.get<AuthResponseDto>(
          "/v1/auth/get-session",
          {},
        );
        return data;
      } catch (error) {
        const axiosError = error as AxiosError;
        if (
          axiosError.response?.status === 401 ||
          axiosError.response?.status === 404
        ) {
          return null;
        }
        throw error;
      }
    },
    retry: false,
    staleTime: 1000 * 60 * 5,
    gcTime: 1000 * 60 * 60,
    refetchOnWindowFocus: true,
  }));

  const signInMutation = useMutation(() => ({
    mutationFn: async (input: SignInDto) => {
      const { data } = await AxiosConfig.post<AuthResponseDto>(
        "/v1/auth/sign-in/email",
        input,
      );
      return data;
    },
    onSuccess: async () => {
      await queryClient.invalidateQueries({ queryKey: ["auth-session"] });
    },
  }));

  const user = Solid.createMemo(() => sessionQuery.data?.user || null);
  const session = Solid.createMemo(() => sessionQuery.data?.session || null);
  const isAuthenticated = Solid.createMemo(() => !!user());
  const isLoading = Solid.createMemo(() => sessionQuery.isLoading);

  const signIn = (input: SignInDto) => signInMutation.mutateAsync(input);

  const signOut = async () => {
    try {
      await AxiosConfig.post("/v1/auth/sign-out");
    } finally {
      queryClient.removeQueries({ queryKey: ["auth-session"] });
      queryClient.clear();
      window.location.href = "/login";
    }
  };

  const refreshSession = async () => {
    await queryClient.invalidateQueries({ queryKey: ["auth-session"] });
  };

  // 🆕 NOVOS HELPERS DE PERMISSÃO
  const hasRole = (role: string): boolean => {
    const currentUser = user();
    if (!currentUser) return false;
    return currentUser.roles?.includes(role) || false;
  };

  const hasAnyRole = (roles: string[]): boolean => {
    const currentUser = user();
    if (!currentUser) return false;
    return roles.some(role => currentUser.roles?.includes(role));
  };

  const hasAllRoles = (roles: string[]): boolean => {
    const currentUser = user();
    if (!currentUser) return false;
    return roles.every(role => currentUser.roles?.includes(role));
  };

  const hasPermission = (permission: string): boolean => {
    const currentUser = user();
    if (!currentUser) return false;
    return currentUser.permissions?.includes(permission) || false;
  };

  const hasAnyPermission = (permissions: string[]): boolean => {
    const currentUser = user();
    if (!currentUser) return false;
    return permissions.some(perm => currentUser.permissions?.includes(perm));
  };

  const can = (action: string): boolean => {
    return hasPermission(action);
  };

  return (
    <AuthContext.Provider
      value={{
        user,
        session,
        isAuthenticated,
        isLoading,
        signIn,
        signOut,
        refreshSession,
        hasRole,
        hasAnyRole,
        hasAllRoles,
        hasPermission,
        hasAnyPermission,
        can,
      }}
    >
      {props.children}
    </AuthContext.Provider>
  );
}
```

### 9.4 Atualizar useAuthGuard

```typescript
// src/hooks/useAuth.ts

import { createMemo } from "solid-js";
import { useAuth } from "~/context/auth-context";

export function useAuthGuard() {
  const auth = useAuth();

  return {
    // Acesso direto aos dados
    user: auth.user,
    session: auth.session,

    // Estado de autenticação
    isAuthenticated: createMemo(() => !!auth.user()),
    isLoading: auth.isLoading,

    // Ações
    signIn: auth.signIn,
    signOut: auth.signOut,
    refresh: auth.refreshSession,

    // 🆕 Utilitários de ROLE
    hasRole: (role: string) => createMemo(() => auth.hasRole(role)),
    hasAnyRole: (roles: string[]) => createMemo(() => auth.hasAnyRole(roles)),
    hasAllRoles: (roles: string[]) => createMemo(() => auth.hasAllRoles(roles)),

    // 🆕 Utilitários de PERMISSÃO
    hasPermission: (permission: string) => createMemo(() => auth.hasPermission(permission)),
    hasAnyPermission: (permissions: string[]) => createMemo(() => auth.hasAnyPermission(permissions)),
    can: (action: string) => createMemo(() => auth.can(action)),

    // 🆕 Verificações específicas
    canPublishVideos: createMemo(() => auth.can('videos:publish')),
    canAccessForum: createMemo(() => auth.can('forum:access')),
    canUseConnectMe: createMemo(() => auth.can('connect_me:use')),
    
    // 🆕 Roles específicas
    isResident: createMemo(() => auth.hasAnyRole(['resident_base', 'resident_paid'])),
    isSyndic: createMemo(() => auth.hasAnyRole(['syndic_n1', 'syndic_n2', 'syndic_n3'])),
    isEnterprise: createMemo(() => auth.hasAnyRole(['enterprise_plus', 'enterprise_pro'])),
  };
}
```

### 9.5 Componentes de Guard

```typescript
// src/components/guards/PermissionGuard.tsx

import { Show, type JSX } from "solid-js";
import { useAuthGuard } from "~/hooks/useAuth";

interface PermissionGuardProps {
  permission: string;
  fallback?: JSX.Element;
  children: JSX.Element;
}

export function PermissionGuard(props: PermissionGuardProps) {
  const auth = useAuthGuard();
  
  return (
    <Show
      when={auth.hasPermission(props.permission)()}
      fallback={props.fallback}
    >
      {props.children}
    </Show>
  );
}

// Uso:
// <PermissionGuard permission="videos:publish">
//   <PublishButton />
// </PermissionGuard>
```

```typescript
// src/components/guards/RoleGuard.tsx

import { Show, type JSX } from "solid-js";
import { useAuthGuard } from "~/hooks/useAuth";

interface RoleGuardProps {
  role?: string;
  anyRole?: string[];
  allRoles?: string[];
  fallback?: JSX.Element;
  children: JSX.Element;
}

export function RoleGuard(props: RoleGuardProps) {
  const auth = useAuthGuard();
  
  const hasAccess = () => {
    if (props.role) return auth.hasRole(props.role)();
    if (props.anyRole) return auth.hasAnyRole(props.anyRole)();
    if (props.allRoles) return auth.hasAllRoles(props.allRoles)();
    return false;
  };
  
  return (
    <Show when={hasAccess()} fallback={props.fallback}>
      {props.children}
    </Show>
  );
}

// Uso:
// <RoleGuard anyRole={["syndic_n2", "syndic_n3"]}>
//   <PublishVideoButton />
// </RoleGuard>
```

### 9.6 Exemplo de Uso em Componente

```typescript
// src/pages/videos/VideoCard.tsx

import { Show } from "solid-js";
import { useAuthGuard } from "~/hooks/useAuth";
import { PermissionGuard } from "~/components/guards/PermissionGuard";

interface VideoCardProps {
  video: Video;
}

export function VideoCard(props: VideoCardProps) {
  const auth = useAuthGuard();
  
  // Lógica condicional baseada em permissões
  const canWatchFull = () => auth.can('videos:watch:full')();
  const isOwner = () => auth.user()?.id === props.video.authorId;
  
  return (
    <div class="video-card">
      <h3>{props.video.title}</h3>
      
      {/* Preview sempre disponível */}
      <Show when={canWatchFull()}>
        <button onClick={() => playFullVideo(props.video)}>
          Assistir Completo
        </button>
      </Show>
      
      <Show when={!canWatchFull()}>
        <button onClick={() => playPreview(props.video)}>
          Preview (25%)
        </button>
        <UpgradeButton />
      </Show>
      
      {/* Botões de ação apenas para o autor */}
      <Show when={isOwner()}>
        <button onClick={() => editVideo(props.video)}>Editar</button>
        <button onClick={() => deleteVideo(props.video)}>Deletar</button>
      </Show>
      
      {/* Guard component para publicar */}
      <PermissionGuard 
        permission="videos:publish"
        fallback={<UpgradeButton />}
      >
        <button onClick={() => publishVideo(props.video)}>
          Publicar
        </button>
      </PermissionGuard>
    </div>
  );
}
```

### 9.7 Roteamento Protegido

```typescript
// src/routes/videos/publish.tsx

import { createRouteData, Navigate } from "@tanstack/solid-router";
import { useAuthGuard } from "~/hooks/useAuth";
import { PublishVideoForm } from "~/components/videos/PublishVideoForm";

export const Route = createRoute({
  path: "/videos/publish",
  component: PublishVideoPage,
});

function PublishVideoPage() {
  const auth = useAuthGuard();
  
  // Redirect se não tiver permissão
  if (!auth.can('videos:publish')()) {
    return <Navigate href="/upgrade" />;
  }
  
  return (
    <div>
      <h1>Publicar Vídeo</h1>
      <PublishVideoForm />
    </div>
  );
}
```

---

## 10. MIGRAÇÃO DETALHADA

### 10.1 Passo 1: Criar Novas Tabelas

```bash
# 1. Gerar migration
npm run db:generate

# Criar arquivo de migration manualmente
# db/migrations/0001_add_roles_tables.sql
```

```sql
-- migrations/0001_add_roles_tables.sql

BEGIN;

-- ===================================
-- 1. CRIAR TABELA roles
-- ===================================
CREATE TABLE roles (
  id UUID PRIMARY KEY,
  name TEXT UNIQUE NOT NULL,
  display_name TEXT NOT NULL,
  description TEXT,
  metadata JSONB,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX roles_name_idx ON roles(name);

-- ===================================
-- 2. CRIAR TABELA user_roles
-- ===================================
CREATE TABLE user_roles (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  role_id UUID NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
  expires_at TIMESTAMP,
  deleted_at TIMESTAMP,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX user_roles_user_id_idx ON user_roles(user_id);
CREATE INDEX user_roles_role_id_idx ON user_roles(role_id);
CREATE INDEX user_roles_user_role_idx ON user_roles(user_id, role_id);

-- ===================================
-- 3. POPULAR ROLES PADRÃO
-- ===================================
INSERT INTO roles (id, name, display_name, description, metadata) VALUES
  (gen_random_uuid(), 'resident_base', 'Morador Base', 'Plano gratuito para moradores', '{"quotas": {}}'),
  (gen_random_uuid(), 'resident_paid', 'Morador Premium', 'Plano pago para moradores', '{"quotas": {"connect_me:use": {"limit": 4, "period": "monthly"}}}'),
  (gen_random_uuid(), 'syndic_n1', 'Síndico N1', 'Síndico nível 1', '{"quotas": {}}'),
  (gen_random_uuid(), 'syndic_n2', 'Síndico N2', 'Síndico nível 2', '{"quotas": {"videos:publish": {"limit": 4, "period": "monthly"}}}'),
  (gen_random_uuid(), 'syndic_n3', 'Síndico N3', 'Síndico nível 3', '{"quotas": {"videos:publish": {"limit": -1, "period": "unlimited"}}}'),
  (gen_random_uuid(), 'enterprise_plus', 'Empresa Plus', 'Plano empresarial básico', '{"quotas": {}}'),
  (gen_random_uuid(), 'enterprise_pro', 'Empresa Pro', 'Plano empresarial avançado', '{"quotas": {}}'),
  (gen_random_uuid(), 'marketing', 'Marketing', 'Equipe de marketing', '{"quotas": {}}'),
  (gen_random_uuid(), 'local_commerce', 'Comércio Local', 'Empresas locais', '{"quotas": {}}');

-- ===================================
-- 4. MIGRAR ROLES EXISTENTES
-- ===================================
INSERT INTO user_roles (id, user_id, role_id)
SELECT 
  gen_random_uuid(),
  u.id,
  r.id
FROM users u
JOIN roles r ON (
  (u.role = 'resident' AND r.name = 'resident_base') OR
  (u.role = 'syndic' AND r.name = 'syndic_n1') OR
  (u.role = 'enterprise' AND r.name = 'enterprise_plus') OR
  (u.role = 'marketing' AND r.name = 'marketing') OR
  (u.role = 'local_company' AND r.name = 'local_commerce') OR
  (u.role = 'admin' AND r.name = 'syndic_n3')
)
WHERE u.role != 'none';

COMMIT;
```

### 10.2 Passo 2: Sincronizar Oso Cloud

```typescript
// scripts/sync-oso-initial.ts

import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';
import { Oso } from 'oso-cloud';
import * as schema from '../src/infrastructure/database/drizzle/schema';

async function syncOsoInitial() {
  const pool = new Pool({ connectionString: process.env.DATABASE_URL });
  const db = drizzle(pool, { schema });
  const oso = new Oso(process.env.OSO_URL!, process.env.OSO_API_KEY!);
  
  console.log('🔄 Sincronizando roles com Oso Cloud...');
  
  // 1. Buscar todos os user_roles
  const userRoles = await db.query.userRoles.findMany({
    with: {
      user: true,
      role: true,
    }
  });
  
  // 2. Criar fatos em batch
  const facts = userRoles.map(ur => ({
    predicate: 'has_role',
    args: [
      { type: 'User', id: ur.userId },
      ur.role.name
    ]
  }));
  
  console.log(`📊 Sincronizando ${facts.length} roles...`);
  
  // 3. Inserir em batch (bulk)
  await oso.bulk(facts.map(f => ({ insert: f })));
  
  console.log('✅ Sincronização completa!');
  
  await pool.end();
}

syncOsoInitial().catch(console.error);
```

```bash
# Rodar sincronização inicial
npx tsx scripts/sync-oso-initial.ts
```

### 10.3 Passo 3: Atualizar Código

#### **A. Criar RoleRepository**

```typescript
// src/modules/auth/infrastructure/database/drizzle/repositories/role.repository.impl.ts

import type { UseCaseDependencies } from "../../../../../core/contracts/base-use-case";
import * as schema from "../../../../../infrastructure/database/drizzle/schema";
import type { IRoleRepository } from "../../../domain/repositories/role.repository";
import { Role } from "../../../domain/entities/role.entity";
import { eq } from "drizzle-orm";

interface RoleRepositoryDependencies extends UseCaseDependencies {}

export class RoleRepositoryImpl implements IRoleRepository {
  constructor(private readonly deps: RoleRepositoryDependencies) {}
  
  async findByUserId(userId: string): Promise<Role[]> {
    const userRoles = await this.deps.unitOfWork.db.query.userRoles.findMany({
      where: eq(schema.userRoles.userId, userId),
      with: {
        role: true,
      }
    });
    
    return userRoles.map(ur => new Role(
      ur.role.id,
      ur.role.name,
      ur.role.displayName,
      ur.role.description,
      ur.role.metadata,
      ur.expiresAt
    ));
  }
  
  async findByName(name: string): Promise<Role | null> {
    const role = await this.deps.unitOfWork.db.query.roles.findFirst({
      where: eq(schema.roles.name, name)
    });
    
    if (!role) return null;
    
    return new Role(
      role.id,
      role.name,
      role.displayName,
      role.description,
      role.metadata
    );
  }
  
  async assignRole(userId: string, roleName: string): Promise<void> {
    const role = await this.findByName(roleName);
    if (!role) throw new Error(`Role ${roleName} not found`);
    
    await this.deps.unitOfWork.db.insert(schema.userRoles).values({
      userId,
      roleId: role.getId(),
    });
  }
  
  async removeRole(userId: string, roleName: string): Promise<void> {
    const role = await this.findByName(roleName);
    if (!role) return;
    
    await this.deps.unitOfWork.db.delete(schema.userRoles).where(
      eq(schema.userRoles.userId, userId),
      eq(schema.userRoles.roleId, role.getId())
    );
  }
}
```

#### **B. Atualizar GetSessionUseCase**

```typescript
// modules/auth/application/use-cases/get-session.use-case.ts

export class GetSessionUseCase extends BaseUseCase {
  async execute({ token }: { token: string }) {
    const payload = await this.deps.jwtService.verifyToken(token);
    const user = await this.deps.userRepository.findById(payload.sub);
    const session = await this.deps.sessionRepository.findOneById(payload.sid);
    
    if (!user || !session) {
      throw new UnauthorizedError();
    }
    
    // 🆕 Buscar roles
    const userRoles = await this.deps.roleRepository.findByUserId(user.getId());
    
    return {
      user: {
        id: user.getId(),
        name: user.getName(),
        email: user.getEmail().getValue(),
        username: user.getUsername().getValue(),
        emailVerified: user.getEmailVerified(),
        roles: userRoles.map(r => r.name),  // 🆕
      },
      session: {
        id: session.getId(),
        expiresAt: session.getExpiresAt(),
        ...
      }
    };
  }
}
```

### 10.4 Passo 4: Remover Código Antigo

```bash
# ❌ REMOVER após validação completa:
rm src/modules/billing/domain/services/permission.service.ts
rm src/modules/billing/domain/services/permission.example.ts
rm src/infrastructure/database/drizzle/schema/billing/permissions.ts
```

**Checklist antes de remover:**
- [ ] Todas as rotas testadas com novo sistema
- [ ] Frontend atualizado e testado
- [ ] Oso Cloud sincronizado
- [ ] Rollback plan preparado
- [ ] Monitoramento configurado

### 10.5 Passo 5: Configurar Polar no Oso Cloud

```bash
# Upload policy para Oso Cloud
oso-cloud policy upload src/infrastructure/authorization/policies/main.polar
```

```polar
# src/infrastructure/authorization/policies/main.polar

# ===================================
# ACTORS
# ===================================
actor User {}

# ===================================
# RESOURCES
# ===================================
resource Organization {
  permissions = [
    "videos:view:preview",
    "videos:view:full",
    "videos:publish",
    "videos:upload:instructional",
    "videos:upload:institutional",
    "forum:access",
    "profiles:view",
    "curriculums:create",
    "curriculums:view",
    "connect_me:use"
  ];
}

resource Video {
  permissions = ["watch:full", "watch:preview", "edit", "delete"];
  roles = ["author", "editor"];
  
  "edit" if "author" or "editor";
  "delete" if "author";
}

# ===================================
# FACTS
# ===================================
has_role(user: User, role: String);
has_relation(video: Video, relation: String, user: User);

# ===================================
# RULES - RBAC (Global Permissions)
# ===================================

# Preview: todos podem ver
allow(user: User, "videos:view:preview", _org: Organization) if
  has_role(user, _);

# Full: apenas planos pagos
allow(user: User, "videos:view:full", _org: Organization) if
  has_role(user, "resident_paid") or
  has_role(user, "syndic_n1") or
  has_role(user, "syndic_n2") or
  has_role(user, "syndic_n3") or
  has_role(user, "enterprise_plus") or
  has_role(user, "enterprise_pro");

# Publicar: Syndic N2+
allow(user: User, "videos:publish", _org: Organization) if
  has_role(user, "syndic_n2") or
  has_role(user, "syndic_n3");

# Upload instrucional: apenas Enterprise
allow(user: User, "videos:upload:instructional", _org: Organization) if
  has_role(user, "enterprise_plus") or
  has_role(user, "enterprise_pro");

# Upload institucional: apenas Enterprise Pro
allow(user: User, "videos:upload:institutional", _org: Organization) if
  has_role(user, "enterprise_pro");

# Fórum: Syndic N1+
allow(user: User, "forum:access", _org: Organization) if
  has_role(user, "syndic_n1") or
  has_role(user, "syndic_n2") or
  has_role(user, "syndic_n3") or
  has_role(user, "enterprise_plus") or
  has_role(user, "enterprise_pro");

# ===================================
# RULES - ABAC (Resource Permissions)
# ===================================

# Vídeos: apenas autor pode deletar
allow(user: User, action, video: Video) if
  has_permission(video, action, user);
```

---

## 11. EXEMPLOS PRÁTICOS REAIS

### 11.1 Rota de Publicar Vídeo

```typescript
// src/modules/videos/infrastructure/http/routes/videos.routes.ts

import { authenticate } from "../../../../../shared/hooks/authenticate.hook";
import { authorize } from "../../../../../shared/hooks/authorize.hook";

export async function videosRoutes(app: FastifyInstance) {
  // Publicar vídeo
  app.post('/videos/publish', {
    preHandler: [
      authenticate,  // 1. Autentica usuário
      authorize({    // 2. Verifica permissão global
        action: 'videos:publish',
        resourceType: 'Organization'
      })
    ],
    schema: {
      body: publishVideoSchema,
      response: {
        201: videoResponseSchema
      }
    }
  }, async (request, reply) => {
    const { publishVideoUseCase } = request.diScope.cradle;
    
    const video = await publishVideoUseCase.execute(
      { userId: request.user.getId() },
      request.body
    );
    
    return reply.status(201).send(video);
  });
  
  // Deletar vídeo
  app.delete('/videos/:videoId', {
    preHandler: [
      authenticate,
      authorize({    // Verifica ownership
        action: 'delete',
        resourceType: 'Video',
        resourceId: (req) => req.params.videoId  // 👈 Dynamic
      })
    ]
  }, async (request, reply) => {
    const { deleteVideoUseCase } = request.diScope.cradle;
    
    await deleteVideoUseCase.execute(
      { userId: request.user.getId() },
      request.params.videoId
    );
    
    return reply.status(204).send();
  });
}
```

### 11.2 Use Case com Quota

```typescript
// src/modules/videos/application/use-cases/publish-video.use-case.ts

export class PublishVideoUseCase extends BaseUseCase {
  async execute(context: { userId: string }, data: PublishVideoDto) {
    // 1. Verificar permissão global
    const canPublish = await this.deps.authorizationService.can(
      context,
      'videos:publish',
      'Organization'
    );
    
    if (!canPublish) {
      throw new ForbiddenError('Seu plano não permite publicar vídeos');
    }
    
    // 2. Verificar quota (via metadata da role)
    const roles = await this.deps.roleRepository.findByUserId(context.userId);
    const quotaConfig = roles
      .map(r => r.metadata?.quotas?.['videos:publish'])
      .find(q => q !== undefined);
    
    if (quotaConfig && quotaConfig.limit !== -1) {
      // Verificar uso atual (query no banco)
      const currentMonth = new Date().getMonth();
      const currentYear = new Date().getFullYear();
      
      const count = await this.deps.videoRepository.countPublished(
        context.userId,
        new Date(currentYear, currentMonth, 1),
        new Date(currentYear, currentMonth + 1, 0)
      );
      
      if (count >= quotaConfig.limit) {
        throw new BusinessError(`Você atingiu o limite de ${quotaConfig.limit} vídeos por mês`);
      }
    }
    
    // 3. Publicar vídeo
    const video = await this.deps.videoRepository.update(data.videoId, {
      status: 'published',
      publishedAt: new Date()
    });
    
    // 4. Sincronizar ownership com Oso
    await this.deps.syncService.setOwnership(
      'Video',
      video.id,
      context.userId,
      'author'
    );
    
    return video;
  }
}
```

### 11.3 Frontend - Botão Condicional

```typescript
// src/components/videos/PublishButton.tsx

import { Show } from "solid-js";
import { useAuthGuard } from "~/hooks/useAuth";
import { createQuery } from "@tanstack/solid-query";

interface PublishButtonProps {
  videoId: string;
}

export function PublishButton(props: PublishButtonProps) {
  const auth = useAuthGuard();
  
  // Verificar permissão
  const canPublish = auth.can('videos:publish');
  
  // Buscar quota (se tiver permissão)
  const quotaQuery = createQuery(() => ({
    queryKey: ['quota', 'videos:publish'],
    queryFn: async () => {
      const res = await axios.get('/v1/quotas/videos:publish');
      return res.data;
    },
    enabled: canPublish(),
  }));
  
  // Handler
  const handlePublish = async () => {
    try {
      await axios.post(`/v1/videos/${props.videoId}/publish`);
      alert('Vídeo publicado!');
    } catch (error) {
      alert('Erro ao publicar');
    }
  };
  
  return (
    <Show
      when={canPublish()}
      fallback={<UpgradeButton message="Assine para publicar vídeos" />}
    >
      <Show
        when={!quotaQuery.isLoading}
        fallback={<button disabled>Carregando...</button>}
      >
        <Show
          when={quotaQuery.data && quotaQuery.data.remaining > 0}
          fallback={
            <button disabled>
              Limite atingido ({quotaQuery.data?.used}/{quotaQuery.data?.limit})
            </button>
          }
        >
          <button onClick={handlePublish}>
            Publicar ({quotaQuery.data?.used}/{quotaQuery.data?.limit})
          </button>
        </Show>
      </Show>
    </Show>
  );
}
```

---

## CONCLUSÃO

Esta análise cobriu **TODOS** os aspectos que faltaram:

✅ **O que existe HOJE** (schemas, PermissionService, relations)  
✅ **RBAC vs ABAC** (quando usar cada um)  
✅ **Estrutura de banco** (mudanças detalhadas, migrations)  
✅ **Impactos no código** (AuthService, JwtService, hooks)  
✅ **Awilix (DI)** (como registrar serviços)  
✅ **Ordem de plugins** (por que importa, nova ordem)  
✅ **Por que serviço + plugin** (responsabilidades)  
✅ **Contextos globais vs unitários** (quando usar)  
✅ **Frontend SolidJS** (AuthContext, useAuth, Guards)  
✅ **Migração completa** (passo a passo)  
✅ **Exemplos práticos** (rotas, use cases, frontend)

**Próximos passos:**
1. Revisar este documento
2. Criar issues no projeto para cada fase
3. Implementar incrementalmente
4. Testar cada camada
5. Migrar gradualmente

**Dúvidas ou ajustes?** Posso detalhar qualquer seção específica.
