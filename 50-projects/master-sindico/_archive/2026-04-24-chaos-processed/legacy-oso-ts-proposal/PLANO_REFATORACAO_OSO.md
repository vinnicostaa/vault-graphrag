# Plano de Refatoração - Integração Oso Cloud

## 📋 Diagnóstico da Situação Atual

### Problemas Identificados

1. **Permissões no lugar errado**
   - Schema de `permissions` está em `modules/billing/` 
   - Deveria estar em `modules/auth/` ou em estrutura própria

2. **Fragmentação da lógica de autorização**
   - `PermissionService` em `billing/domain/services/`
   - `OsoCloudService` em `core/auth/`
   - Cliente Oso em `infrastructure/authorization/`
   - Nenhuma integração real entre eles

3. **Dependência de schema externo**
   - `PermissionService` usa `mastersindico-schemas` (pacote externo)
   - Não está usando o Oso Cloud para decisões de autorização

4. **Falta de plugin Fastify**
   - Oso não está integrado no ciclo de vida do Fastify
   - Não há decoradores na request para facilitar verificações

5. **Policy Polar incompleta**
   - Policy básica, mas não integrada com o código
   - Falta sincronização de fatos (facts) com o banco de dados

## 🎯 Objetivos da Refatoração

1. **Centralizar autorização no Oso Cloud**
2. **Criar estrutura modular e bem organizada**
3. **Integrar Oso com Fastify via plugin**
4. **Sincronizar dados do banco com Oso (facts)**
5. **Focar em permissões Sprint 1 (MVP)**

## 🏗️ Nova Arquitetura Proposta

```
src/
├── infrastructure/
│   ├── authorization/
│   │   ├── oso.client.ts           # Cliente Oso singleton
│   │   ├── policies/
│   │   │   └── main.polar          # Policy atualizada
│   │   └── sync/
│   │       ├── sync-service.ts     # Sincroniza facts com Oso
│   │       └── fact-mappers.ts     # Mapeia entidades → facts
│   │
│   └── database/
│       └── drizzle/
│           └── schema/
│               └── auth/
│                   ├── users.ts
│                   ├── roles.ts          # Novo
│                   └── user-roles.ts     # Novo (many-to-many)
│
├── modules/
│   └── auth/
│       ├── domain/
│       │   ├── entities/
│       │   │   └── permission-context.entity.ts
│       │   ├── services/
│       │   │   └── authorization.service.ts    # Novo service usando Oso
│       │   └── repositories/
│       │       └── role.repository.ts
│       │
│       └── infrastructure/
│           └── database/
│               └── drizzle/
│                   └── repositories/
│                       └── role.repository.impl.ts
│
└── shared/
    └── plugins/
        └── oso.plugin.ts           # Plugin Fastify p/ Oso

```

## 📝 Plano de Implementação

### Fase 1: Reestruturação do Schema (Database)

#### 1.1 Mover schema de permissões para auth

**Arquivo**: `src/infrastructure/database/drizzle/schema/auth/roles.ts`

```typescript
import { index, pgTable, text, timestamp, uuid } from "drizzle-orm/pg-core";

export const roles = pgTable(
  "roles",
  {
    id: uuid()
      .primaryKey()
      .defaultRandom(),

    name: text().notNull().unique(), // 'resident_base', 'syndic_n1', etc.
    displayName: text().notNull(),
    description: text(),
    
    // Metadata para configuração de quotas
    metadata: text({ mode: 'json' }).$type<{
      quotas?: Record<string, { limit: number; period: string }>;
      features?: string[];
    }>(),

    createdAt: timestamp().defaultNow().notNull(),
    updatedAt: timestamp().defaultNow().notNull()
      .$onUpdate(() => new Date()),
  },
  (t) => [
    index("roles_name_idx").using('btree', t.name),
  ],
);
```

**Arquivo**: `src/infrastructure/database/drizzle/schema/auth/user-roles.ts`

```typescript
import { index, pgTable, timestamp, uuid } from "drizzle-orm/pg-core";
import { users } from "../users/users";
import { roles } from "./roles";

export const userRoles = pgTable(
  "user_roles",
  {
    id: uuid()
      .primaryKey()
      .defaultRandom(),

    userId: uuid()
      .notNull()
      .references(() => users.id, { onDelete: "cascade" }),
      
    roleId: uuid()
      .notNull()
      .references(() => roles.id, { onDelete: "cascade" }),

    // Para roles temporárias ou com expiração
    expiresAt: timestamp(),
    
    createdAt: timestamp().defaultNow().notNull(),
  },
  (t) => [
    index("user_roles_user_id_idx").using('btree', t.userId),
    index("user_roles_role_id_idx").using('btree', t.roleId),
    index("user_roles_user_role_idx").using('btree', t.userId, t.roleId),
  ],
);
```

#### 1.2 Atualizar schema de users

Remover campo `role` de `users.ts` (vai usar tabela many-to-many agora):

```typescript
// Antes:
role: userRoles("role").default("none").notNull(),

// Depois: remover essa linha
```

### Fase 2: Plugin Fastify para Oso

**Arquivo**: `src/shared/plugins/oso.plugin.ts`

```typescript
import type { FastifyInstance, FastifyPluginAsync, FastifyRequest } from 'fastify';
import fp from 'fastify-plugin';
import { Oso } from 'oso-cloud';
import { env } from '../config';

// Tipos para decorar a request
declare module 'fastify' {
  interface FastifyRequest {
    oso: Oso;
    authorize: (action: string, resourceType: string, resourceId?: string) => Promise<boolean>;
    can: (action: string, resourceType: string, resourceId?: string) => Promise<boolean>;
    listResources: (action: string, resourceType: string) => Promise<string[]>;
  }
}

const osoPlugin: FastifyPluginAsync = async (fastify: FastifyInstance) => {
  // Criar cliente Oso singleton
  const osoClient = new Oso(env.OSO_URL, env.OSO_API_KEY);

  // Decorar instance com cliente
  fastify.decorate('oso', osoClient);

  // Decorar request com helpers
  fastify.decorateRequest('oso', osoClient);
  
  fastify.decorateRequest('authorize', async function (
    this: FastifyRequest,
    action: string,
    resourceType: string,
    resourceId?: string,
  ): Promise<boolean> {
    const userId = this.user?.id; // assumindo que `user` vem do authenticate hook
    
    if (!userId) {
      return false;
    }

    return await osoClient.authorize(
      { type: 'User', id: userId },
      action,
      {
        type: resourceType,
        id: resourceId || '_',
      },
    );
  });

  fastify.decorateRequest('can', async function (
    this: FastifyRequest,
    action: string,
    resourceType: string,
    resourceId?: string,
  ): Promise<boolean> {
    return await this.authorize(action, resourceType, resourceId);
  });

  fastify.decorateRequest('listResources', async function (
    this: FastifyRequest,
    action: string,
    resourceType: string,
  ): Promise<string[]> {
    const userId = this.user?.id;
    
    if (!userId) {
      return [];
    }

    return await osoClient.list(
      { type: 'User', id: userId },
      action,
      resourceType,
    );
  });

  fastify.log.info('Oso Cloud plugin registered successfully');
};

export default fp(osoPlugin, {
  name: 'oso-plugin',
  dependencies: [],
});
```

### Fase 3: Policy Polar Atualizada (Sprint 1)

**Arquivo**: `src/infrastructure/authorization/policies/main.polar`

```polar
# ===================================
# ACTORS
# ===================================
actor User {}

# ===================================
# RESOURCES
# ===================================

# Recursos globais (organizacionais)
resource Organization {
  permissions = [
    "videos:view",
    "videos:view:full",
    "videos:view:preview",
    "videos:publish",
    "videos:upload:institutional",
    "videos:upload:instructional",
    "forum:access",
    "profiles:view",
    "curriculums:create",
    "curriculums:view",
    "connect_me:use",
  ];
}

# Recursos específicos
resource Video {
  permissions = ["watch:full", "watch:preview", "edit", "delete"];
  relations = { author: User };
  
  "edit" if "author";
  "delete" if "author";
}

resource Profile {
  permissions = ["view", "edit"];
  relations = { owner: User };
  
  "edit" if "owner";
}

# ===================================
# PERMISSÕES POR ROLE (SPRINT 1)
# ===================================

# Base (não pagante)
has_permission(user: User, "videos:view:preview", _: Organization) if
  has_role(user, "resident_base");

has_permission(user: User, "profiles:view", _: Organization) if
  has_role(user, "resident_base");

# Morador Pagante
has_permission(user: User, "videos:view:full", _: Organization) if
  has_role(user, "resident_paid");

has_permission(user: User, "curriculums:create", _: Organization) if
  has_role(user, "resident_paid");

has_permission(user: User, "connect_me:use", _: Organization) if
  has_role(user, "resident_paid");

# Síndico N1 (Aprender)
has_permission(user: User, "videos:view:full", _: Organization) if
  role in ["syndic_n1", "syndic_n2", "syndic_n3"] and
  has_role(user, role);

has_permission(user: User, "forum:access", _: Organization) if
  role in ["syndic_n1", "syndic_n2", "syndic_n3"] and
  has_role(user, role);

# Síndico N2 (Atuar)
has_permission(user: User, "videos:publish", _: Organization) if
  role in ["syndic_n2", "syndic_n3"] and
  has_role(user, role);

has_permission(user: User, "profiles:view", _: Organization) if
  role in ["syndic_n2", "syndic_n3"] and
  has_role(user, role);

# Síndico N3 (Consolidar)
# Herda tudo de N2

# Empresas Plus
has_permission(user: User, "videos:upload:instructional", _: Organization) if
  has_role(user, "enterprise_plus");

has_permission(user: User, "curriculums:view", _: Organization) if
  role in ["enterprise_plus", "enterprise_pro"] and
  has_role(user, role);

# Empresas Pro
has_permission(user: User, "videos:upload:institutional", _: Organization) if
  has_role(user, "enterprise_pro");

# ===================================
# FACT DEFINITIONS
# ===================================
has_role(user: User, role: String);
```

### Fase 4: Serviço de Sincronização de Facts

**Arquivo**: `src/infrastructure/authorization/sync/sync-service.ts`

```typescript
import type { Oso } from 'oso-cloud';

export interface FactSyncService {
  syncUserRole(userId: string, roleName: string): Promise<void>;
  removeUserRole(userId: string, roleName: string): Promise<void>;
  syncAllUserRoles(userId: string, roles: string[]): Promise<void>;
}

export class OsoFactSyncService implements FactSyncService {
  constructor(private readonly oso: Oso) {}

  async syncUserRole(userId: string, roleName: string): Promise<void> {
    await this.oso.insert([
      'has_role',
      { type: 'User', id: userId },
      roleName,
    ]);
  }

  async removeUserRole(userId: string, roleName: string): Promise<void> {
    await this.oso.delete([
      'has_role',
      { type: 'User', id: userId },
      roleName,
    ]);
  }

  async syncAllUserRoles(userId: string, roles: string[]): Promise<void> {
    // Remove todos os roles existentes
    await this.oso.delete([
      'has_role',
      { type: 'User', id: userId },
      null, // wildcard para remover todos
    ]);

    // Insere novos roles
    const facts = roles.map(role => [
      'has_role',
      { type: 'User', id: userId },
      role,
    ]);

    await this.oso.bulk(facts);
  }
}
```

### Fase 5: Authorization Service (Substituindo PermissionService)

**Arquivo**: `src/modules/auth/domain/services/authorization.service.ts`

```typescript
import type { Oso } from 'oso-cloud';

export interface AuthorizationContext {
  userId: string;
  roles?: string[];
}

export class AuthorizationService {
  constructor(private readonly oso: Oso) {}

  /**
   * Verifica se o usuário pode realizar uma ação
   */
  async can(
    context: AuthorizationContext,
    action: string,
    resourceType: string,
    resourceId?: string,
  ): Promise<boolean> {
    return await this.oso.authorize(
      { type: 'User', id: context.userId },
      action,
      {
        type: resourceType,
        id: resourceId || '_',
      },
    );
  }

  /**
   * Lista todos os recursos do tipo especificado que o usuário pode acessar
   */
  async listResources(
    context: AuthorizationContext,
    action: string,
    resourceType: string,
  ): Promise<string[]> {
    return await this.oso.list(
      { type: 'User', id: context.userId },
      action,
      resourceType,
    );
  }

  /**
   * Lista todas as ações que o usuário pode realizar no recurso
   */
  async actions(
    context: AuthorizationContext,
    resourceType: string,
    resourceId: string,
  ): Promise<string[]> {
    return await this.oso.actions(
      { type: 'User', id: context.userId },
      {
        type: resourceType,
        id: resourceId,
      },
    );
  }

  /**
   * Verifica múltiplas permissões de uma vez
   */
  async canAll(
    context: AuthorizationContext,
    checks: Array<{ action: string; resourceType: string; resourceId?: string }>,
  ): Promise<boolean> {
    const results = await Promise.all(
      checks.map(check =>
        this.can(context, check.action, check.resourceType, check.resourceId),
      ),
    );

    return results.every(result => result);
  }
}
```

### Fase 6: Hook de Autorização para Rotas

**Arquivo**: `src/shared/hooks/authorize.hook.ts`

```typescript
import type { FastifyRequest } from 'fastify';
import { ForbiddenError } from '../errors';

interface AuthorizeOptions {
  action: string;
  resourceType: string;
  resourceId?: (request: FastifyRequest) => string;
}

export function authorize(options: AuthorizeOptions) {
  return async function (request: FastifyRequest) {
    const { action, resourceType, resourceId } = options;
    
    const resource = resourceId ? resourceId(request) : undefined;
    
    const allowed = await request.authorize(action, resourceType, resource);
    
    if (!allowed) {
      throw new ForbiddenError(
        `Você não tem permissão para realizar a ação: ${action} em ${resourceType}`,
      );
    }
  };
}

// Uso em rotas:
// fastify.get(
//   '/videos/:id',
//   {
//     preHandler: [
//       authenticate,
//       authorize({
//         action: 'videos:view:full',
//         resourceType: 'Organization',
//       }),
//     ],
//   },
//   handler,
// );
```

### Fase 7: Migração de Dados

**Script**: `scripts/migrate-permissions-to-oso.ts`

```typescript
import { drizzle } from 'drizzle-orm/node-postgres';
import { oso } from '../src/infrastructure/authorization/oso.client';
import { users } from '../src/infrastructure/database/drizzle/schema';

async function migratePermissionsToOso() {
  const db = drizzle(/* connection */);

  // 1. Buscar todos os usuários com suas roles/planos
  const allUsers = await db
    .select({
      id: users.id,
      role: users.role,
    })
    .from(users);

  // 2. Para cada usuário, sincronizar role com Oso
  const facts = allUsers.map(user => [
    'has_role',
    { type: 'User', id: user.id },
    user.role,
  ]);

  // 3. Inserir em lote no Oso Cloud
  await oso.bulk(facts);

  console.log(`Migrated ${facts.length} user roles to Oso Cloud`);
}

migratePermissionsToOso().catch(console.error);
```

## 🚀 Cronograma de Execução

### Sprint 1 - Fundação (Semana 1)
- [ ] Criar schemas de roles e user-roles
- [ ] Criar plugin Oso para Fastify
- [ ] Atualizar Policy Polar com permissões básicas
- [ ] Implementar AuthorizationService

### Sprint 2 - Sincronização (Semana 2)
- [ ] Implementar FactSyncService
- [ ] Criar hooks de autorização
- [ ] Migrar dados existentes para Oso
- [ ] Atualizar use cases para usar novo serviço

### Sprint 3 - Testes e Refinamento (Semana 3)
- [ ] Testes unitários do AuthorizationService
- [ ] Testes de integração com Oso Cloud
- [ ] Remover PermissionService antigo
- [ ] Documentação

## 📊 Matriz de Permissões - Sprint 1

### Permissões Básicas

| Ação | Base | Morador Pg | Síndico N1 | Síndico N2 | Síndico N3 | Empresa+ | Empresa Pro |
|------|------|------------|------------|------------|------------|----------|-------------|
| `videos:view:preview` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| `videos:view:full` | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| `videos:publish` | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ | ❌ |
| `videos:upload:instructional` | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| `videos:upload:institutional` | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| `forum:access` | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| `profiles:view` | ✅ | ✅ | ❌ | ✅ | ✅ | ✅ | ✅ |
| `curriculums:create` | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| `curriculums:view` | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| `connect_me:use` | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |

## 🔧 Configuração Necessária

### Variáveis de Ambiente

```env
# Oso Cloud
OSO_URL=https://cloud.osohq.com
OSO_API_KEY=your_api_key_here
```

### Instalação de Dependências

```bash
npm install oso-cloud@2.5.3
npm install fastify-plugin
```

## ⚠️ Pontos de Atenção

1. **Migration de Dados**: Executar script de migração antes de remover PermissionService
2. **Testes**: Garantir cobertura de testes antes de deploy
3. **Rollback**: Manter PermissionService antigo até validação completa
4. **Monitoramento**: Adicionar logs para verificar chamadas ao Oso
5. **Performance**: Oso Cloud tem rate limits - considerar caching se necessário

## 📚 Recursos

- [Oso Cloud Docs](https://www.osohq.com/docs)
- [Oso Cloud Node.js SDK](https://www.osohq.com/docs/reference/sdks/install.md)
- [Polar Language Reference](https://www.osohq.com/docs/reference/polar/introduction.md)
- [Fastify Plugins](https://fastify.dev/docs/latest/Reference/Plugins/)
