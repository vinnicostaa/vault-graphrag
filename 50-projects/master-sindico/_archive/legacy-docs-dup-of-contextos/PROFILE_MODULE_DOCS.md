# Profile Module - Documentação

## Resumo da Implementação

Este módulo implementa o gerenciamento de perfis de usuários seguindo os padrões DDD, SOLID e Clean Architecture estabelecidos no projeto.

## Arquitetura

### Estrutura de Diretórios

```
modules/profile/
├── profile.module.ts                    # Registro DI e rotas
├── domain/
│   ├── entities/
│   │   ├── base-profile.entity.ts       # Classe base ÚNICA para todos os perfis
│   │   ├── resident.entity.ts           # Perfil de Morador
│   │   ├── syndic.entity.ts             # Perfil de Síndico
│   │   ├── enterprise.entity.ts         # Perfil de Empresa
│   │   ├── local-company.entity.ts      # Perfil de Comércio Local
│   │   └── marketing.entity.ts          # Perfil de Marketing
│   ├── repositories/
│   │   ├── resident.repository.ts       # Interface
│   │   ├── syndic.repository.ts
│   │   ├── enterprise.repository.ts
│   │   ├── local-company.repository.ts
│   │   └── marketing.repository.ts
│   └── enums/
│       ├── profile-status.ts            # Status UNIFICADO
│       └── syndic-type.ts               # morador | profissional
├── application/
│   └── use-cases/                       # TODO: Implementar
└── infrastructure/
    ├── database/drizzle/repositories/
    │   ├── resident.repository.impl.ts
    │   ├── syndic.repository.impl.ts
    │   ├── enterprise.repository.impl.ts
    │   ├── local-company.repository.impl.ts
    │   └── marketing.repository.impl.ts
    └── http/routes/                     # TODO: Implementar
```

## Decisões Arquiteturais

### 1. Unificação de Status

**Antes**: Existiam dois enums separados (`profileStatus` e `paidProfileStatus`)

**Depois**: Um único `profileStatus` para TODOS os perfis:
- `incomplete` - Perfil criado mas faltam dados obrigatórios
- `pending_payment` - Dados completos, aguardando primeira ativação
- `active` - Perfil ativo e completo

### 2. Separação Profile vs Subscription

| Conceito | Responsabilidade | Exemplo |
|----------|------------------|---------|
| **Profile** | Armazena DADOS do perfil | CPF, endereço, CNPJ |
| **Subscription** | Controla PLANO e PAGAMENTO | `resident_base` (R$0), `enterprise_plus` (R$149) |
| **PermissionService** | Define O QUE pode fazer | `canPublishVideos()` |

### 3. Soft Delete

Todos os perfis suportam soft delete via campo `deletedAt`:
- Queries filtram automaticamente `deletedAt IS NULL`
- Métodos `softDelete()` e `restore()` na entidade
- Repositório implementa `softDelete()`, `restore()` e `hardDelete()`

## Schemas do Banco

### Profile Status (Unificado)

```typescript
export const profileStatus = pgEnum("profile_status", [
  "incomplete",
  "pending_payment", 
  "active",
]);
```

### Subscription Plan

```typescript
export const subscriptionPlan = pgEnum("subscription_plan", [
  "resident_base",      // Gratuito
  "resident_paid",      // Pago
  "syndic_n1",          // Aprender
  "syndic_n2",          // Atuar
  "syndic_n3",          // Consolidar
  "enterprise_plus",    // Demonstrar
  "enterprise_pro",     // Influenciar
  "marketing_standard",
  "local_company_standard",
]);
```

## Entidades

### BaseProfile (Classe Abstrata)

Campos comuns a todos os perfis:
- `id`, `userId`, `status`
- `createdAt`, `updatedAt`, `deletedAt`

Métodos:
- `isActive()`, `isIncomplete()`, `isPendingPayment()`, `isDeleted()`
- `activate()`, `markAsPendingPayment()`
- `softDelete()`, `restore()`
- `toBaseDto()`

### Perfis Específicos

| Perfil | Campos Específicos |
|--------|-------------------|
| **ResidentProfile** | birthDate, cpf, address, buildingNameOrCode |
| **SyndicProfile** | birthDate, cpf, address, type, experienceYears, buildingsCount, certifications, miniBio |
| **EnterpriseProfile** | cnpj, foundationDate, legalRep, commercialContact, financeContact, address, certifications, isos, seals |
| **LocalCompanyProfile** | document (CPF/CNPJ), foundationDate, contacts, address, photos |
| **MarketingProfile** | cnpj, foundationDate, commercialContact, address |

## Repositórios

### Interface Padrão

```typescript
interface IProfileRepository<T> {
  findById(id: string): Promise<T | null>;
  findByUserId(userId: string): Promise<T | null>;
  findBy[UniqueField](value: string): Promise<T | null>;
  
  save(profile: T): Promise<void>;
  softDelete(id: string): Promise<void>;
  restore(id: string): Promise<void>;
  hardDelete(id: string): Promise<void>;
}
```

### Implementação

- Estendem `BaseRepository`
- Usam `this.db.query.*` para queries (Drizzle v1 beta)
- Usam `this.db.insert().onConflictDoUpdate()` para upsert
- Mappers `toDomain()` e `toPersistence()`

## Fluxos de Uso

### Criação de Perfil

```typescript
// No Use Case
const profile = ResidentProfile.create({
  id: generateId(),
  userId: user.getId(),
  birthDate,
  cpf,
  address: { ... },
  status: "incomplete",
  createdAt: new Date(),
  updatedAt: new Date(),
});

await this.residentProfileRepository.save(profile);
```

### Ativação de Perfil

```typescript
// Quando dados estão completos
profile.markAsPendingPayment(); // Se necessário
await repository.save(profile);

// Após confirmação de pagamento (via webhook)
profile.activate();
await repository.save(profile);
```

### Soft Delete

```typescript
profile.softDelete();
await repository.save(profile);

// Ou direto no repositório
await repository.softDelete(profileId);
```

## Próximos Passos

1. **Use Cases**: Implementar CRUD para cada tipo de perfil
2. **Rotas HTTP**: Expor endpoints REST
3. **Validações Zod**: Schemas de input/output
4. **Subscription Module**: Gerenciar planos e pagamentos
5. **PermissionService**: Controle de acesso baseado no plano

## Padrões Seguidos

- **DDD**: Entidades ricas, Value Objects implícitos (address como objeto)
- **SOLID**: 
  - SRP: Cada entidade tem uma responsabilidade
  - OCP: BaseProfile extensível
  - LSP: Todas as entidades podem ser usadas onde BaseProfile é esperado
  - ISP: Interfaces de repositório específicas
  - DIP: Use cases dependem de interfaces, não implementações
- **Clean Architecture**: Domínio não conhece infraestrutura
- **Unit of Work**: Transações gerenciadas pelo UoW via `this.unitOfWork.run()`
