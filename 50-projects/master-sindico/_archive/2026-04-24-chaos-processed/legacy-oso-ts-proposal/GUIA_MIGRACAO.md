# Guia de Migração - Sistema de Autorização com Oso Cloud

## 📋 Visão Geral

Este guia detalha o processo de migração do sistema de permissões antigo (baseado em `mastersindico-schemas`) para o novo sistema baseado em **Oso Cloud**.

## 🎯 Objetivos da Migração

1. ✅ Centralizar autorização no Oso Cloud
2. ✅ Remover dependência de `mastersindico-schemas`
3. ✅ Integrar Oso com Fastify via plugin
4. ✅ Sincronizar dados do banco com Oso (facts)
5. ✅ Mover permissões de `billing` para `auth`

## 📦 Pré-requisitos

### 1. Instalar Dependências

```bash
npm install oso-cloud@2.5.3
npm install fastify-plugin
```

### 2. Configurar Variáveis de Ambiente

```env
# .env
OSO_URL=https://cloud.osohq.com
OSO_API_KEY=your_api_key_here
```

## 🗄️ Migração do Banco de Dados

### 1. Criar Novas Tabelas

Execute as migrations para criar as tabelas `roles` e `user_roles`:

```bash
# Gerar migration
npm run db:generate

# Aplicar migration
npm run db:push
```

### 2. Popular Roles Iniciais

Execute o script de seed para criar os roles do sistema:

```bash
npx tsx scripts/seed-roles.ts
```

Isso criará os seguintes roles:
- `resident_base` (Morador Base)
- `resident_paid` (Morador Pagante)
- `syndic_n1`, `syndic_n2`, `syndic_n3` (Síndicos)
- `enterprise_plus`, `enterprise_pro` (Empresas)
- `marketing` (Marketing)
- `local_commerce` (Comércio Local)

### 3. Migrar Dados Existentes

Se você já tem usuários com roles no sistema antigo, execute:

```bash
npx tsx scripts/migrate-user-roles.ts
```

Este script:
1. Lê os roles existentes da coluna `users.role`
2. Cria registros em `user_roles`
3. Sincroniza os dados com Oso Cloud

## 🔧 Configuração do Código

### 1. Registrar Plugin Oso

Em `src/app.ts`, registre o plugin **antes** de qualquer rota:

```typescript
import osoPlugin from './shared/plugins/oso.plugin';

// ... outras configurações

await app.register(osoPlugin);

// Depois registrar rotas
```

### 2. Atualizar Imports

**Antes:**
```typescript
import { PermissionService } from '../modules/billing/domain/services/permission.service';
```

**Depois:**
```typescript
import { AuthorizationService } from '../modules/auth/domain/services/authorization.service';
```

### 3. Usar Hook de Autorização em Rotas

**Antes:**
```typescript
fastify.get('/videos/:id', 
  { preHandler: [authenticate] },
  async (request, reply) => {
    // Lógica manual de verificação
    // ...
  }
);
```

**Depois:**
```typescript
import { authorize } from '../shared/hooks/authorize.hook';

fastify.get('/videos/:id',
  {
    preHandler: [
      authenticate,
      authorize({
        action: 'videos:view:full',
        resourceType: 'Organization',
      }),
    ],
  },
  handler,
);
```

### 4. Usar AuthorizationService em Use Cases

**Antes:**
```typescript
const result = await this.permissionService.can(context, 'videos:publish');
if (!result.allowed) {
  throw new ForbiddenError(result.reason);
}
```

**Depois:**
```typescript
const context = { userId: user.id };
const canPublish = await this.authService.can(
  context,
  'videos:publish',
  'Organization'
);

if (!canPublish) {
  throw new ForbiddenError('Você não tem permissão para publicar vídeos');
}
```

## 🔄 Sincronização de Facts

### Quando Sincronizar

O sistema deve sincronizar facts com Oso Cloud nos seguintes eventos:

1. **Criação de Usuário**
```typescript
// Após criar usuário no banco
await syncService.syncUserRole(userId, 'resident_base');
```

2. **Mudança de Plano**
```typescript
// Quando usuário muda de plano
await syncService.syncUserRole(userId, newRole);
// Ou, se quiser substituir completamente:
await syncService.syncAllUserRoles(userId, [newRole]);
```

3. **Criação de Recurso com Ownership**
```typescript
// Após criar vídeo
await syncService.setOwnership('Video', videoId, userId, 'author');
```

4. **Deleção de Usuário**
```typescript
// Antes de deletar usuário
await syncService.deleteUserFacts(userId);
```

### Exemplo Completo - Use Case de Sign Up

```typescript
export class SignUpUseCase {
  constructor(
    private readonly userRepo: IUserRepository,
    private readonly syncService: OsoFactSyncService,
  ) {}

  async execute(dto: SignUpDTO): Promise<User> {
    // 1. Criar usuário no banco
    const user = await this.userRepo.create({
      email: dto.email,
      name: dto.name,
      // ...
    });

    // 2. Atribuir role inicial
    const roleId = await this.getRoleId('resident_base');
    await this.userRoleRepo.create({
      userId: user.id,
      roleId,
    });

    // 3. Sincronizar com Oso
    await this.syncService.syncUserRole(user.id, 'resident_base');

    return user;
  }
}
```

## 📝 Exemplos de Uso

### Verificar Permissão Simples

```typescript
const canAccess = await request.can('forum:access', 'Organization');
if (!canAccess) {
  throw new ForbiddenError('Acesso negado ao fórum');
}
```

### Verificar Múltiplas Permissões

```typescript
const context = { userId: user.id };
const canDoAll = await authService.canAll(context, [
  { action: 'videos:publish', resourceType: 'Organization' },
  { action: 'forum:access', resourceType: 'Organization' },
]);
```

### Listar Recursos Acessíveis

```typescript
const videoIds = await request.listResources('watch:full', 'Video');
// Use videoIds para filtrar query no banco
```

### Verificar Ações Disponíveis

```typescript
const actions = await request.actions('Video', videoId);
// Retorna: ['watch:full', 'edit', 'delete'] ou subconjunto
```

## 🧪 Testes

### Testar Permissões Localmente

Antes de fazer deploy, teste as permissões localmente:

```typescript
import { Oso } from 'oso-cloud';

const oso = new Oso(process.env.OSO_URL!, process.env.OSO_API_KEY!);

// Sincronizar role de teste
await oso.insert(['has_role', { type: 'User', id: 'test-user' }, 'syndic_n2']);

// Testar permissão
const allowed = await oso.authorize(
  { type: 'User', id: 'test-user' },
  'videos:publish',
  { type: 'Organization', id: '_' }
);

console.log('Can publish:', allowed); // true
```

### Usar Oso Dev Server

Para desenvolvimento local, você pode usar o Oso Dev Server:

```bash
# Instalar CLI
curl -L https://cloud.osohq.com/install.sh | bash

# Iniciar dev server
oso-cloud dev-server --policy ./src/infrastructure/authorization/policies/main.polar
```

## 🚨 Troubleshooting

### Erro: "User not found in Oso"

**Problema:** Facts não foram sincronizados corretamente.

**Solução:**
```bash
# Re-sincronizar todos os usuários
npx tsx scripts/sync-all-users-to-oso.ts
```

### Erro: "Permission denied" inesperado

**Problema:** Policy Polar pode estar incorreta.

**Solução:**
1. Verificar policy no Oso Cloud Dashboard
2. Usar a aba "Explain" para debugar
3. Verificar se facts foram sincronizados:

```typescript
const facts = await oso.list({ type: 'User', id: userId }, 'has_role', '_');
console.log('User roles:', facts);
```

### Performance Lenta

**Problema:** Muitas chamadas ao Oso Cloud.

**Solução:**
- Use `canAll` ou `canAny` para verificar múltiplas permissões de uma vez
- Considere caching para permissões que não mudam frequentemente
- Use Local Authorization (futura implementação) para queries no banco

## 📚 Recursos Adicionais

- [Oso Cloud Docs](https://www.osohq.com/docs)
- [Polar Language Reference](https://www.osohq.com/docs/reference/polar/introduction.md)
- [Matriz Funcional](./PLANO_REFATORACAO_OSO.md#matriz-de-permissões---sprint-1)

## ✅ Checklist de Migração

- [ ] Instalar dependências
- [ ] Configurar variáveis de ambiente
- [ ] Executar migrations do banco
- [ ] Popular roles iniciais (seed)
- [ ] Migrar dados existentes
- [ ] Registrar plugin Oso no app
- [ ] Atualizar imports e código
- [ ] Adicionar hooks de autorização em rotas
- [ ] Implementar sincronização de facts
- [ ] Testar permissões
- [ ] Remover código antigo (PermissionService)
- [ ] Deploy da policy no Oso Cloud
- [ ] Monitorar logs em produção

## 🎉 Conclusão

Após completar esta migração, você terá:

✅ Sistema de autorização centralizado e escalável
✅ Permissões gerenciadas de forma declarativa (Polar)
✅ Sincronização automática com Oso Cloud
✅ Código mais limpo e manutenível
✅ Estrutura preparada para crescimento
