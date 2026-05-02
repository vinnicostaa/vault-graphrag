# 🏗️ Master Síndico - Arquitetura Moderna do Zero

> **Documento de Referência Arquitetural**  
> Baseado na análise técnica completa de 276 arquivos  
> Data: 2026-02-26  
> Status: Proposta de Evolução Arquitetural

---

## 📑 ÍNDICE RÁPIDO

### 🎯 Visão Geral
- [Objetivos da Evolução](#objetivos-da-evolução)
- [O que Mantém vs O que Muda](#o-que-mantém-vs-o-que-muda)

### 🏛️ Decisões Arquiteturais
- [Stack Tecnológica](#stack-tecnológica)
- [Estrutura de Código](#estrutura-de-código)
- [Padrões de Projeto](#padrões-de-projeto-implementados)

### 🔧 Ferramentas e Práticas
- [Observabilidade](#observabilidade)
- [Testing Strategy](#testing-strategy)
- [DevOps e CI/CD](#devops-e-cicd)

### 🚀 Implementação
- [Roadmap por Sprints](#roadmap-de-implementação)
- [Priorização](#matriz-de-priorização)

### 📚 Referências
- [Recursos de Estudo](#recursos-de-estudo)
- [Decisões Arquiteturais (ADRs)](#adrs)

---

## 🎯 OBJETIVOS DA EVOLUÇÃO

### O que Mantém (Pontos Fortes) ✅

1. **Domain-Driven Design (DDD)**
   - Separação clara: Domain / Application / Infrastructure
   - Entities com lógica de negócio encapsulada
   - Value Objects imutáveis
   - Repositories abstratos

2. **Attribute-Based Access Control (ABAC)**
   - CASL para permissões granulares
   - Ownership rules com conditions
   - Matriz funcional de permissões

3. **Clean Architecture**
   - Dependency Inversion Principle
   - Use Cases isolados
   - Infraestrutura plugável

4. **Unit of Work Pattern**
   - Transações automáticas
   - Rollback em caso de erro
   - Abstração do banco de dados

5. **Dependency Injection**
   - Awilix para DI container
   - Scoped containers por request
   - Auto-wiring de dependências

### O que Corrige (Problemas Críticos) ❌

1. **Falta de Domain Events**
   - `changeEmail()` não invalida sessões
   - `ban()` não notifica usuário
   - `changePlan()` não atualiza cache de permissions

2. **Race Conditions**
   - `incrementUsage()` em quotas não usa locks
   - Múltiplas requests podem exceder limite

3. **Authorization Silencioso**
   - `BaseUseCase.authorize()` apenas loga warn se não injetado
   - Risco de use case esquecer de validar

4. **Falta de Observabilidade**
   - Apenas logs, sem traces ou métricas
   - Difícil rastrear fluxo de requests
   - Sem visibilidade de performance

5. **Validações Apenas na Aplicação**
   - Banco aceita dados inválidos
   - Sem constraints de formato
   - Pode causar inconsistências

### O que Adiciona (Ferramentas Modernas) 🆕

1. **Effect-TS** - Error handling funcional e type-safe
2. **OpenTelemetry** - Traces, métricas e logs estruturados
3. **Temporal.io** - Workflows duráveis com retry automático
4. **WebSockets** - Real-time para notificações e chat
5. **Testcontainers** - Testes de integração com banco real
6. **Property-Based Testing** - Expandir uso para validar invariantes

---

## 🏛️ STACK TECNOLÓGICA

### Backend (Mantém + Evolui)

| Categoria | Atual | Proposta | Justificativa |
|-----------|-------|----------|---------------|
| **Runtime** | Node.js | ✅ Mantém | Performance adequada |
| **Framework** | Fastify | ✅ Mantém | Excelente performance |
| **Language** | TypeScript 5.x | ✅ Mantém | Type safety |
| **ORM** | Drizzle | ✅ Mantém | Type-safe, performático |
| **Validation** | Zod | ✅ Expandir | Runtime + compile-time |
| **Testing** | Vitest | 🆕 Adicionar | Mais rápido que Jest |
| **Error Handling** | Try-catch | 🆕 Effect-TS | Erros tipados |

### Infraestrutura

| Categoria | Atual | Proposta | Justificativa |
|-----------|-------|----------|---------------|
| **Database** | PostgreSQL 16+ | ✅ Mantém | Robusto e confiável |
| **Cache** | Redis 7+ | ✅ Mantém | Performance |
| **Workflows** | Cron jobs | 🆕 Temporal.io | Durabilidade + retry |
| **Observability** | Logs | 🆕 OpenTelemetry | Traces + métricas |
| **Monitoring** | - | 🆕 Grafana Stack | Dashboards + alertas |

### Pagamentos

| Categoria | Atual | Proposta | Justificativa |
|-----------|-------|----------|---------------|
| **Provider** | Stripe | ✅ Mantém | API excelente |
| **Webhooks** | Manual | 🆕 Svix | Retry + dashboard |

---

## 📁 ESTRUTURA DE CÓDIGO

### Opção Recomendada: Core Interno (Não Pacote Externo)

```
apps/api/src/
├── core/                          # ✅ Fundação (não é pacote!)
│   ├── contracts/
│   │   ├── base-use-case.ts      # BaseUseCase, AuthorizedUseCase
│   │   ├── base-repository.ts    # Repository base
│   │   ├── base-task.ts          # Cron jobs base
│   │   └── unit-of-work.interface.ts
│   │
│   ├── decorators/
│   │   ├── traced.decorator.ts   # @Traced() OpenTelemetry
│   │   └── validated.decorator.ts # @Validated() Zod
│   │
│   ├── domain/
│   │   ├── aggregate-root.ts     # 🆕 Domain Events
│   │   └── domain-event.ts       # 🆕 Base Event
│   │
│   ├── errors/
│   │   └── index.ts              # DomainError hierarchy
│   │
│   ├── value-objects/
│   │   ├── email.vo.ts
│   │   ├── phone.vo.ts
│   │   ├── username.vo.ts
│   │   └── money.vo.ts
│   │
│   └── types/
│       ├── execution-context.ts  # Request context
│       └── result.ts             # Result<T, E>
│
├── modules/                       # Módulos de negócio
│   ├── auth/
│   ├── user/
│   ├── billing/
│   └── ...
│
├── infrastructure/                # Infraestrutura compartilhada
│   ├── database/
│   ├── cache/
│   ├── telemetry/                # 🆕 OpenTelemetry
│   └── providers/
│
└── shared/                        # Utilitários
    ├── hooks/
    ├── plugins/
    └── utils/
```

**Por que não usar `packages/shared-kernel`?**

| Aspecto | Pacote Externo | Core Interno |
|---------|----------------|--------------|
| Complexidade | ⚠️ Alta | ✅ Baixa |
| Velocidade dev | ⚠️ Lenta (rebuild) | ✅ Rápida (hot reload) |
| Reuso | ✅ Frontend + Backend | ⚠️ Apenas backend |
| Manutenção | ⚠️ Difícil | ✅ Fácil |
| Type safety | ⚠️ Pode quebrar | ✅ Sempre sincronizado |

**Conclusão:** Use `apps/api/src/core` para backend-only.

---

## 🎯 PADRÕES DE PROJETO IMPLEMENTADOS

### 1. SAGA PATTERN (CRÍTICO para Billing)

**Problema:** Criar subscription no Stripe e falhar ao salvar no DB deixa inconsistente.

**Solução:** Saga com compensating transactions

```typescript
// apps/api/src/core/saga/saga-orchestrator.ts
export abstract class Saga<TContext> {
  private steps: SagaStep<TContext>[] = [];
  
  async execute(context: TContext): Promise<Result<TContext, SagaError>> {
    const executedSteps: SagaStep<TContext>[] = [];
    
    try {
      // Forward execution
      for (const step of this.steps) {
        await step.execute(context);
        executedSteps.push(step);
      }
      return Result.ok(context);
    } catch (error) {
      // Compensating transactions (rollback)
      for (const step of executedSteps.reverse()) {
        await step.compensate(context);
      }
      return Result.fail(new SagaError('Saga failed', error));
    }
  }
}
```

**Quando usar:**
- ✅ Operações distribuídas (Stripe + DB)
- ✅ Precisa de rollback automático
- ✅ Múltiplos sistemas externos

**Quando NÃO usar:**
- ❌ Operações simples (1 DB query)
- ❌ Sem side effects externos

---

### 2. CQRS (Command Query Responsibility Segregation)

**Problema:** Use cases misturam reads e writes, dificultando otimização.

**Solução:** Separar Commands (writes) e Queries (reads)

```typescript
// Commands usam repositories + entities
export class ChooseUserRoleCommand implements ICommand {
  readonly commandName = 'user.choose-role';
  
  constructor(
    public readonly userId: string,
    public readonly role: UserRole,
    public readonly planId: string,
  ) {}
}

// Queries usam SQL direto (otimizado)
export class GetUserByIdQuery implements IQuery<UserDTO> {
  readonly queryName = 'user.get-by-id';
  
  constructor(public readonly userId: string) {}
}
```

**Vantagens:**
- ✅ Queries otimizadas (sem carregar entity completa)
- ✅ Escalabilidade (read replicas)
- ✅ Testabilidade (handlers isolados)

---

### 3. SPECIFICATION PATTERN (Queries Complexas)

**Problema:** Queries complexas espalhadas nos repositories.

**Solução:** Specifications compostas

```typescript
const spec = new ActiveUserSpec()
  .and(new EmailVerifiedSpec())
  .and(new HasRoleSpec('syndic'));

// No repository
const users = await db
  .select()
  .from(usersTable)
  .where(spec.toSql());

// Ou em memória (testes)
const isValid = spec.isSatisfiedBy(user);
```

**Vantagens:**
- ✅ Reutilizável
- ✅ Testável
- ✅ Composição (and/or/not)

---

### 4. DOMAIN EVENTS (CRÍTICO - estava faltando!)

**Problema:** Mudanças em entities não propagam side effects.

**Solução:** Aggregate Root + Domain Events

```typescript
export class User extends AggregateRoot<User> {
  changeEmail(newEmail: EmailVO): void {
    const oldEmail = this.email.getValue();
    this.email = newEmail;
    this.emailVerified = false;
    
    // 🆕 Dispara evento
    this.addDomainEvent(
      new UserEmailChangedEvent(this.id, oldEmail, newEmail.getValue())
    );
  }
}

// Event handler invalida sessões
export class InvalidateSessionsOnEmailChange {
  async handle(event: UserEmailChangedEvent): Promise<void> {
    await this.sessionRepo.invalidateAllByUserId(event.aggregateId);
    await this.cacheProvider.del(`session:*:${event.aggregateId}`);
  }
}
```

**Resolve:**
- ✅ `changeEmail()` → invalida sessões
- ✅ `ban()` → invalida sessões + notifica
- ✅ `changePlan()` → atualiza cache de permissions

---

### 5. ADAPTER PATTERN com Chain of Responsibility

**Problema:** Adapters simples, sem fallback.

**Solução:** Chain com fallback automático

```typescript
// Tenta Resend, se falhar usa Nodemailer
const emailService = new ResendEmailAdapter(resend);
emailService.setNext(new NodemailerEmailAdapter(transporter));

const result = await emailService.handle({
  from: 'noreply@mastersindico.com',
  to: user.email,
  subject: 'Welcome',
  html: '<h1>Welcome!</h1>',
});
```

**Vantagens:**
- ✅ Fallback automático
- ✅ Resiliente a falhas
- ✅ Testável (mock de adapters)

---

## 🔍 OBSERVABILIDADE

### OpenTelemetry Integration

**Problema:** Apenas logs, sem traces ou métricas.

**Solução:** Traces + Métricas + Logs estruturados

```typescript
// Decorator para auto-instrumentação
export class LoginUseCase {
  @Traced('login.execute')
  async execute(email: string, password: string) {
    // Automaticamente cria span com timing
    // Registra métricas (counter, histogram)
    // Loga erros com contexto
  }
}
```

**Stack:**
- OpenTelemetry Collector
- Grafana (dashboards)
- Prometheus (métricas)
- Loki (logs)
- Tempo (traces)

---

## 🧪 TESTING STRATEGY

### Pirâmide de Testes

```
        /\
       /  \  E2E (5%)
      /____\
     /      \  Integration (15%)
    /________\
   /          \  Unit (80%)
  /__________\
```

### Ferramentas

1. **Vitest** - Unit tests (rápido)
2. **Testcontainers** - Integration tests (banco real)
3. **Property-Based Testing** - Invariantes (fast-check)

```typescript
// Property-based test
test.prop([fc.integer({ min: 0, max: 1000000 })])(
  'Money.add is commutative',
  (cents) => {
    const a = Money.fromCents(cents, 'BRL');
    const b = Money.fromCents(cents, 'BRL');
    
    expect(a.add(b).equals(b.add(a))).toBe(true);
  }
);
```

---

## 🌐 REST vs ALTERNATIVAS

### Decisão: Abordagem Híbrida

**95% REST + 5% WebSockets**

| Protocolo | Use Cases | Justificativa |
|-----------|-----------|---------------|
| **REST** | CRUD, queries, uploads, webhooks | Stateless, cacheable, escalável |
| **WebSockets** | Notificações, chat, live updates | Real-time, baixa latência |

### Por que NÃO migrar 100% para WebSockets?

1. ❌ **Stateful** - Dificulta escalabilidade horizontal
2. ❌ **Sem HTTP Cache** - 16x mais carga no servidor
3. ❌ **Segurança** - WAF e rate limiting complexos
4. ❌ **Mobile** - Drena bateria
5. ❌ **Custo** - 4.5x mais caro (precisa mais RAM)

### Quando usar cada um?

**REST para:**
- ✅ CRUD operations (95%)
- ✅ Queries complexas
- ✅ File uploads
- ✅ Webhooks (Stripe, Mux)

**WebSockets para:**
- ✅ Notificações real-time (5%)
- ✅ Chat/Comments
- ✅ Live updates (admin dashboard)
- ✅ Presence (online/offline)

---

## 🚀 ROADMAP DE IMPLEMENTAÇÃO

### Sprint 1-2: Fundação
- [ ] Setup monorepo com Turborepo
- [ ] Migrar para Effect-TS (error handling)
- [ ] Adicionar OpenTelemetry
- [ ] Setup Testcontainers

### Sprint 3-4: Domain Events
- [ ] Implementar DomainEvent base
- [ ] Refatorar User entity para Aggregate Root
- [ ] Adicionar event handlers (invalidar cache, notificações)
- [ ] Testes de integração

### Sprint 5-6: Background Jobs
- [ ] Setup Temporal.io
- [ ] Migrar cron jobs para workflows
- [ ] Implementar subscription renewal workflow
- [ ] Dashboard de workflows

### Sprint 7-8: Database Hardening
- [ ] Adicionar constraints no banco
- [ ] Migrar validações para Zod schemas
- [ ] Adicionar índices faltantes
- [ ] Performance testing

### Sprint 9-10: Observabilidade
- [ ] Setup Grafana stack
- [ ] Criar dashboards principais
- [ ] Configurar alertas críticos
- [ ] Runbooks operacionais

### Sprint 11-12: WebSockets
- [ ] Setup Socket.IO server
- [ ] Implementar autenticação via JWT
- [ ] Criar namespaces (/notifications, /video-chat, /admin)
- [ ] Integrar com Domain Events
- [ ] Frontend integration

---

## 📊 MATRIZ DE PRIORIZAÇÃO

| Feature | Impacto | Esforço | Prioridade | Sprint |
|---------|---------|---------|------------|--------|
| Domain Events | 🔥 Alto | Médio | P0 | 3-4 |
| WebSockets | 🔥 Alto | Médio | P0 | 11-12 |
| OpenTelemetry | Alto | Baixo | P1 | 1-2 |
| Temporal.io | Alto | Alto | P1 | 5-6 |
| Database Constraints | Médio | Baixo | P1 | 7-8 |
| Effect-TS | Médio | Alto | P2 | 1-2 |
| CQRS | Baixo | Alto | P3 | Futuro |
| Event Sourcing | Baixo | Muito Alto | P4 | Futuro |

---

## 📚 RECURSOS DE ESTUDO

### Documentação Oficial
- [Effect-TS](https://effect.website) - Error handling funcional
- [Temporal.io](https://docs.temporal.io) - Workflows duráveis
- [OpenTelemetry](https://opentelemetry.io/docs/languages/js/) - Observabilidade
- [Testcontainers](https://testcontainers.com) - Integration testing
- [Fast-Check](https://fast-check.dev) - Property-based testing

### Livros Recomendados
- "Domain-Driven Design" - Eric Evans
- "Implementing Domain-Driven Design" - Vaughn Vernon
- "Building Microservices" - Sam Newman
- "Release It!" - Michael Nygard

### Artigos e Talks
- Martin Fowler - CQRS Pattern
- Greg Young - Event Sourcing
- Udi Dahan - Saga Pattern

---

## 🎯 DECISÕES ARQUITETURAIS (ADRs)

### ADR-001: Manter Drizzle ORM
**Status:** Aceito  
**Contexto:** Avaliar se mantém Drizzle ou migra para Prisma/TypeORM  
**Decisão:** Manter Drizzle  
**Consequências:**
- ✅ Type safety excelente
- ✅ Performance superior
- ✅ Raw SQL quando necessário
- ⚠️ Migrations menos maduras que Prisma

### ADR-002: Não usar pacote shared-kernel
**Status:** Aceito  
**Contexto:** Avaliar se cria pacote externo para código compartilhado  
**Decisão:** Usar `apps/api/src/core` interno  
**Consequências:**
- ✅ Desenvolvimento mais rápido
- ✅ Hot reload funciona
- ✅ Type safety garantido
- ⚠️ Não reutilizável no frontend

### ADR-003: Abordagem Híbrida REST + WebSockets
**Status:** Aceito  
**Contexto:** Avaliar se migra 100% para WebSockets  
**Decisão:** 95% REST + 5% WebSockets  
**Consequências:**
- ✅ Escalabilidade mantida
- ✅ HTTP cache funciona
- ✅ Custo controlado
- ⚠️ Complexidade de 2 protocolos

### ADR-004: Implementar Domain Events
**Status:** Aceito  
**Contexto:** Resolver problema de side effects não propagados  
**Decisão:** Aggregate Root + Domain Events  
**Consequências:**
- ✅ Desacoplamento
- ✅ Auditoria completa
- ✅ Testabilidade
- ⚠️ Complexidade adicional

---

## 🏁 CONCLUSÃO

Esta proposta mantém os pontos fortes da arquitetura atual (DDD, ABAC, Clean Architecture) enquanto corrige problemas críticos identificados na análise técnica.

**Próximos Passos:**
1. Revisar e aprovar ADRs
2. Criar specs detalhadas para cada sprint
3. Começar implementação por Domain Events (maior impacto)

**Dúvidas ou Sugestões:**
- Abrir issue no repositório
- Discutir em reunião de arquitetura
- Consultar time de desenvolvimento

---

**Última atualização:** 2026-02-26  
**Versão:** 1.0.0  
**Autor:** Equipe de Arquitetura Master Síndico
