# Resumo Executivo - Refatoração Sistema de Autorização com Oso Cloud

## 🎯 Objetivo

Refatorar o sistema de permissões atual para usar **Oso Cloud** como solução centralizada de autorização, seguindo a Matriz Funcional do projeto e preparando o sistema para Sprint 1.

## 📊 Diagnóstico

### Problemas Encontrados

1. **Localização Incorreta**: Schema de permissions está em `modules/billing/` em vez de `modules/auth/`
2. **Fragmentação**: Três implementações diferentes de autorização sem integração real
3. **Dependência Externa**: PermissionService usa `mastersindico-schemas` (pacote externo)
4. **Falta de Integração**: Oso não está integrado no ciclo de vida do Fastify
5. **Sincronização Manual**: Não há sincronização automática de dados com Oso

## ✅ Solução Implementada

### Nova Arquitetura

```
src/
├── infrastructure/
│   ├── authorization/
│   │   ├── oso.client.ts
│   │   ├── policies/main.polar (atualizado)
│   │   └── sync/sync-service.ts (novo)
│   │
│   └── database/drizzle/schema/auth/
│       ├── roles.ts (novo)
│       └── user-roles.ts (novo)
│
├── modules/auth/domain/services/
│   └── authorization.service.ts (novo)
│
└── shared/
    ├── plugins/oso.plugin.ts (novo)
    └── hooks/authorize.hook.ts (novo)
```

### Componentes Criados

#### 1. **Schemas de Banco de Dados** (Nova Sintaxe Drizzle)

- `roles.ts`: Tabela de roles/planos do sistema
- `user-roles.ts`: Relacionamento many-to-many User ↔ Roles

#### 2. **Plugin Fastify - Oso**

Integra Oso Cloud no ciclo de vida do Fastify, adicionando helpers na request:

```typescript
// Uso em rotas
const allowed = await request.can('videos:publish', 'Organization');
const videos = await request.listResources('watch:full', 'Video');
```

#### 3. **Serviço de Sincronização de Facts**

Sincroniza dados do banco com Oso Cloud:

```typescript
await syncService.syncUserRole(userId, 'resident_paid');
await syncService.setOwnership('Video', videoId, userId, 'author');
```

#### 4. **Authorization Service**

Service principal para verificação de permissões:

```typescript
const canPublish = await authService.can(
  { userId },
  'videos:publish',
  'Organization'
);
```

#### 5. **Hook de Autorização**

Hook reutilizável para proteger rotas:

```typescript
fastify.get('/videos/:id', {
  preHandler: [
    authenticate,
    authorize({
      action: 'videos:view:full',
      resourceType: 'Organization',
    }),
  ],
}, handler);
```

#### 6. **Policy Polar Atualizada**

Policy completa com todas as permissões da Sprint 1 conforme Matriz Funcional.

## 📋 Permissões Implementadas (Sprint 1)

### Matriz de Permissões

| Ação | Base | Morador Pg | Síndico N1 | Síndico N2 | Empresa+ | Empresa Pro |
|------|------|------------|------------|------------|----------|-------------|
| videos:view:preview | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| videos:view:full | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| videos:publish | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ |
| videos:upload:instructional | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| videos:upload:institutional | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| forum:access | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |
| profiles:view | ✅ | ✅ | ❌ | ✅ | ✅ | ✅ |
| curriculums:create | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |
| curriculums:view | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| connect_me:use | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |

## 🚀 Passos para Implementação

### 1. Instalação

```bash
npm install oso-cloud@2.5.3 fastify-plugin
```

### 2. Configuração

```env
OSO_URL=https://cloud.osohq.com
OSO_API_KEY=your_api_key_here
```

### 3. Migração do Banco

```bash
# Gerar e aplicar migrations
npm run db:generate
npm run db:push

# Popular roles
npx tsx scripts/seed-roles.ts

# Migrar dados existentes
npx tsx scripts/migrate-user-roles.ts
```

### 4. Atualizar Código

```typescript
// app.ts
import osoPlugin from './shared/plugins/oso.plugin';
await app.register(osoPlugin);
```

### 5. Atualizar Use Cases

```typescript
// Antes
const result = await permissionService.can(context, 'videos:publish');

// Depois
const canPublish = await authService.can(
  { userId },
  'videos:publish',
  'Organization'
);
```

### 6. Proteger Rotas

```typescript
import { authorize } from '../shared/hooks/authorize.hook';

fastify.post('/videos', {
  preHandler: [
    authenticate,
    authorize({
      action: 'videos:upload:instructional',
      resourceType: 'Organization',
    }),
  ],
}, handler);
```

## 📦 Arquivos Criados

### Código-Fonte

1. `/src/infrastructure/database/drizzle/schema/auth/roles.ts`
2. `/src/infrastructure/database/drizzle/schema/auth/user-roles.ts`
3. `/src/infrastructure/database/drizzle/schema/auth/index.ts`
4. `/src/shared/plugins/oso.plugin.ts`
5. `/src/infrastructure/authorization/sync/sync-service.ts`
6. `/src/modules/auth/domain/services/authorization.service.ts`
7. `/src/shared/hooks/authorize.hook.ts`
8. `/src/infrastructure/authorization/policies/main.polar`

### Scripts e Documentação

9. `/scripts/seed-roles.ts`
10. `/examples/authorization-usage.example.ts`
11. `PLANO_REFATORACAO_OSO.md`
12. `GUIA_MIGRACAO.md`
13. `RESUMO_EXECUTIVO.md` (este arquivo)

## 🎓 Benefícios da Nova Arquitetura

### Técnicos

✅ **Centralização**: Uma única fonte de verdade para autorização
✅ **Escalabilidade**: Oso Cloud é horizontalmente escalável
✅ **Manutenibilidade**: Policy declarativa (Polar) vs código imperativo
✅ **Testabilidade**: Fácil testar permissões isoladamente
✅ **Auditoria**: Logs centralizados no Oso Cloud

### Negócio

✅ **Segurança**: Autorização consistente em todo o sistema
✅ **Flexibilidade**: Mudanças de permissões sem deploy de código
✅ **Compliance**: Auditoria completa de acessos
✅ **Time-to-market**: Menos código para manter

## ⚠️ Pontos de Atenção

1. **Sincronização**: Garantir que facts sejam sempre sincronizados
2. **Migration**: Executar scripts de migração antes de remover código antigo
3. **Testes**: Validar todas as permissões antes de deploy
4. **Rollback**: Manter código antigo até validação completa
5. **Monitoramento**: Adicionar logs e métricas para chamadas ao Oso

## 📈 Próximos Passos (Sprints Futuras)

- **Sprint 2**: Assembleias, Votações (Síndico N3)
- **Sprint 3**: Cursos Certificados (Empresa Pro)
- **Sprint 4**: Clube de Benefícios (Comércio Local)
- **Sprint 5**: Lives (MasterSíndico Admin)
- **Sprint 6**: Local Authorization (performance)

## 🔗 Recursos

- [Documentação Oso Cloud](https://www.osohq.com/docs)
- [Guia de Migração Completo](./GUIA_MIGRACAO.md)
- [Plano de Refatoração Detalhado](./PLANO_REFATORACAO_OSO.md)

## 💡 Conclusão

Esta refatoração estabelece uma base sólida para o sistema de autorização do My Síndico, alinhada com:

- ✅ Matriz Funcional do projeto
- ✅ Best practices de Clean Architecture
- ✅ Requisitos da Sprint 1
- ✅ Escalabilidade para sprints futuras

Todos os componentes foram implementados seguindo os padrões do projeto (DDD, Drizzle ORM beta, TypeScript) e estão prontos para integração.
