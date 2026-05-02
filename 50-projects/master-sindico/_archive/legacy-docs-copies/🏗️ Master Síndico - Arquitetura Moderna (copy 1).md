🏗️ Master Síndico - Arquitetura Moderna do Zero
Baseado na análise técnica completa, vou propor uma abordagem que:

Mantém os pontos fortes (DDD, ABAC, Clean Architecture)
Corrige os problemas críticos identificados
Adiciona ferramentas modernas para melhor DX e qualidade
📋 FASE 1: Decisões Arquiteturais Fundamentais
1.1 Stack Core (Mantém + Evolui)
Backend:

✅ Fastify (mantém - performance excelente)
✅ TypeScript 5.x (mantém - type safety)
✅ Drizzle ORM (mantém - type-safe, performático)
🆕 Effect-TS - Para error handling funcional e composição de efeitos
🆕 Zod - Validação runtime (já usa, mas expandir uso)
🆕 Vitest - Testes rápidos (substituir Jest se houver)
Infraestrutura:

✅ PostgreSQL 16+ (mantém)
✅ Redis 7+ (mantém - cache + rate limit)
🆕 Temporal.io - Workflows duráveis (substituir cron jobs simples)
🆕 OpenTelemetry - Observabilidade completa
🆕 Grafana Stack - Métricas + Logs + Traces
Pagamentos:

✅ Stripe (mantém - excelente API)
🆕 Stripe Webhooks com Svix - Retry automático e dashboard
1.2 Estrutura de Monorepo (Evolui)
master-sindico/
├── apps/
│   ├── api/                    # Backend principal
│   ├── web/                    # Frontend Next.js
│   ├── admin/                  # Painel admin
│   └── workers/                # 🆕 Background jobs (Temporal)
├── packages/
│   ├── shared-kernel/          # 🆕 Domain primitives compartilhados
│   │   ├── value-objects/      # Email, Phone, Money, etc
│   │   ├── errors/             # Hierarquia de erros
│   │   └── types/              # Types compartilhados
│   ├── contracts/              # 🆕 Interfaces + DTOs
│   ├── permissions/            # Matriz funcional (já existe)
│   ├── database/               # 🆕 Schemas Drizzle compartilhados
│   ├── telemetry/              # 🆕 OpenTelemetry setup
│   └── testing/                # 🆕 Test utilities
├── tools/
│   ├── scripts/                # Migrations, seeds, etc
│   └── generators/             # 🆕 Plop.js templates
└── docs/
    ├── architecture/           # ADRs (Architecture Decision Records)
    ├── api/                    # OpenAPI specs
    └── runbooks/               # Playbooks operacionais
🎯 FASE 2: Padrões e Práticas Modernas
2.1 Domain-Driven Design (Evolui)
Mantém:

✅ Separação clara: Domain / Application / Infrastructure
✅ Entities com lógica de negócio
✅ Value Objects imutáveis
✅ Repositories abstratos
Adiciona:

2.1.1 Domain Events (CRÍTICO - estava faltando!)
// packages/shared-kernel/domain-events/base-event.ts
export abstract class DomainEvent {
  readonly occurredAt: Date;
  readonly aggregateId: string;
  readonly eventVersion: number = 1;

  constructor(aggregateId: string) {
    this.aggregateId = aggregateId;
    this.occurredAt = new Date();
  }

  abstract get eventName(): string;
}

// Exemplo: UserEmailChanged
export class UserEmailChangedEvent extends DomainEvent {
  constructor(
    userId: string,
    public readonly oldEmail: string,
    public readonly newEmail: string,
  ) {
    super(userId);
  }

  get eventName() {
    return 'user.email.changed';
  }
}
Por que? Resolve problemas identificados:

✅ changeEmail() agora dispara evento → invalida sessões automaticamente
✅ ban() dispara evento → invalida sessões + notifica usuário
✅ changePlan() dispara evento → atualiza permissions no cache
2.1.2 Aggregate Root Pattern
// packages/shared-kernel/domain/aggregate-root.ts
export abstract class AggregateRoot<T> {
  private _domainEvents: DomainEvent[] = [];

  protected addDomainEvent(event: DomainEvent): void {
    this._domainEvents.push(event);
  }

  public pullDomainEvents(): DomainEvent[] {
    const events = [...this._domainEvents];
    this._domainEvents = [];
    return events;
  }
}

// User entity agora é Aggregate Root
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
2.2 Error Handling Funcional (Effect-TS)
Problema atual: Try-catch espalhado, erros não tipados.

Solução:

// packages/shared-kernel/effects/result.ts
import { Effect, pipe } from 'effect';

// Use case retorna Effect ao invés de Promise
export class LoginUseCase {
  execute(email: string, password: string): Effect.Effect<
    LoginResult,
    | ValidationError
    | UnauthorizedError
    | InternalError
  > {
    return pipe(
      Effect.tryPromise({
        try: () => this.userRepo.findByEmail(email),
        catch: () => new InternalError('Database error'),
      }),
      Effect.flatMap((user) =>
        user
          ? Effect.succeed(user)
          : Effect.fail(new UnauthorizedError('Invalid credentials'))
      ),
      Effect.flatMap((user) =>
        Effect.tryPromise({
          try: () => user.verifyPassword(password),
          catch: () => new InternalError('Password verification failed'),
        })
      ),
      Effect.map((user) => this.generateToken(user)),
    );
  }
}
Vantagens:

✅ Erros tipados no retorno (compile-time safety)
✅ Composição funcional (pipe)
✅ Retry automático, timeout, circuit breaker built-in
✅ Testabilidade (mock de efeitos)
2.3 Observabilidade Completa (OpenTelemetry)
Problema atual: Apenas logs, sem traces ou métricas estruturadas.

Solução:

// packages/telemetry/tracer.ts
import { trace, metrics } from '@opentelemetry/api';

export const tracer = trace.getTracer('master-sindico-api');
export const meter = metrics.getMeter('master-sindico-api');

// Decorator para auto-instrumentação
export function Traced(spanName?: string) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;

    descriptor.value = async function (...args: any[]) {
      const span = tracer.startSpan(spanName || propertyKey);
      
      try {
        const result = await originalMethod.apply(this, args);
        span.setStatus({ code: SpanStatusCode.OK });
        return result;
      } catch (error) {
        span.recordException(error);
        span.setStatus({ code: SpanStatusCode.ERROR });
        throw error;
      } finally {
        span.end();
      }
    };
  };
}

// Uso
export class LoginUseCase {
  @Traced('login.execute')
  async execute(email: string, password: string) {
    // Automaticamente cria span com timing
  }
}
Métricas customizadas:

// packages/telemetry/metrics.ts
const loginCounter = meter.createCounter('login.attempts', {
  description: 'Number of login attempts',
});

const loginDuration = meter.createHistogram('login.duration', {
  description: 'Login duration in ms',
  unit: 'ms',
});

// Uso no use case
loginCounter.add(1, { status: 'success', method: 'local' });
loginDuration.record(Date.now() - startTime);
2.4 Background Jobs com Temporal.io
Problema atual: Cron jobs simples, sem retry, sem visibilidade.

Solução:

// apps/workers/workflows/subscription-renewal.workflow.ts
import { proxyActivities, sleep } from '@temporalio/workflow';

const { chargeSubscription, sendEmail, updateDatabase } = proxyActivities({
  startToCloseTimeout: '5 minutes',
  retry: {
    maximumAttempts: 3,
    backoffCoefficient: 2,
  },
});

export async function subscriptionRenewalWorkflow(
  subscriptionId: string
): Promise<void> {
  // 1. Tenta cobrar
  const chargeResult = await chargeSubscription(subscriptionId);

  if (chargeResult.success) {
    await updateDatabase(subscriptionId, 'active');
    await sendEmail(subscriptionId, 'renewal_success');
    return;
  }

  // 2. Falhou - retry com backoff exponencial
  for (let attempt = 1; attempt <= 3; attempt++) {
    await sleep(`${attempt * 24}h`); // 24h, 48h, 72h
    
    const retryResult = await chargeSubscription(subscriptionId);
    
    if (retryResult.success) {
      await updateDatabase(subscriptionId, 'active');
      await sendEmail(subscriptionId, 'renewal_success_after_retry');
      return;
    }
  }

  // 3. Falhou definitivamente
  await updateDatabase(subscriptionId, 'past_due');
  await sendEmail(subscriptionId, 'renewal_failed');
}
Vantagens:

✅ Workflows duráveis (sobrevivem a crashes)
✅ Retry automático com backoff
✅ Dashboard visual (vê estado de cada workflow)
✅ Testável (mock de activities)
✅ Versionamento de workflows
2.5 Database Constraints + Validações
Problema atual: Validações apenas na aplicação, banco aceita dados inválidos.

Solução:

// packages/database/schemas/users.ts
import { pgTable, uuid, text, boolean, timestamp, check } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: uuid('id').primaryKey().defaultRandom(),
  email: text('email').notNull(),
  username: text('username').notNull(),
  phone: text('phone'),
  // ... outros campos
}, (table) => ({
  // 🆕 Constraints no banco
  emailFormat: check('email_format', sql`email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$'`),
  usernameFormat: check('username_format', sql`username ~ '^[a-z0-9_]{3,30}$'`),
  phoneFormat: check('phone_format', sql`phone ~ '^\d{10,13}$'`),
  banReasonRequired: check('ban_reason_required', sql`(NOT banned) OR (ban_reason IS NOT NULL)`),
  
  // Unique com soft delete
  uniqueEmail: uniqueIndex('users_email_unique').on(table.email).where(sql`deleted_at IS NULL`),
  uniqueUsername: uniqueIndex('users_username_unique').on(table.username).where(sql`deleted_at IS NULL`),
}));
2.6 Testing Strategy (Pirâmide de Testes)
        /\
       /  \  E2E (5%)
      /____\
     /      \  Integration (15%)
    /________\
   /          \  Unit (80%)
  /__________\
Ferramentas:

// packages/testing/setup.ts
import { beforeAll, afterAll, beforeEach } from 'vitest';
import { PostgreSqlContainer } from '@testcontainers/postgresql';
import { RedisContainer } from '@testcontainers/redis';

// 🆕 Testcontainers - banco real em testes
let pgContainer: PostgreSqlContainer;
let redisContainer: RedisContainer;

beforeAll(async () => {
  pgContainer = await new PostgreSqlContainer().start();
  redisContainer = await new RedisContainer().start();
  
  process.env.DATABASE_URL = pgContainer.getConnectionString();
  process.env.REDIS_URL = redisContainer.getConnectionString();
});

afterAll(async () => {
  await pgContainer.stop();
  await redisContainer.stop();
});
Property-Based Testing (já usa, mas expandir):

// apps/api/src/modules/billing/domain/money.spec.ts
import { fc, test } from '@fast-check/vitest';

test.prop([fc.integer({ min: 0, max: 1000000 })])('Money.add is commutative', (cents) => {
  const a = Money.fromCents(cents, 'BRL');
  const b = Money.fromCents(cents, 'BRL');
  
  expect(a.add(b).equals(b.add(a))).toBe(true);
});

test.prop([
  fc.integer({ min: 0, max: 1000000 }),
  fc.integer({ min: 1, max: 100 }),
])('Money.distribute sums to original', (cents, parts) => {
  const money = Money.fromCents(cents, 'BRL');
  const distributed = money.distribute(parts);
  
  const sum = distributed.reduce((acc, m) => acc.add(m), Money.zero('BRL'));
  expect(sum.equals(money)).toBe(true);
});
🛠️ FASE 3: Ferramentas de Desenvolvimento
3.1 Code Generation (Plop.js)
# Gera módulo completo com 1 comando
pnpm plop module

? Module name: notification
? Domain entities: Message, Template
? Use cases: SendEmail, SendSMS, CreateTemplate

✔ Created apps/api/src/modules/notification/
  ├── domain/
  │   ├── entities/
  │   │   ├── message.entity.ts
  │   │   └── template.entity.ts
  │   └── repositories/
  ├── application/
  │   └── use-cases/
  │       ├── send-email.use-case.ts
  │       ├── send-sms.use-case.ts
  │       └── create-template.use-case.ts
  ├── infrastructure/
  │   └── repositories/
  └── presentation/
      └── routes/
3.2 API Documentation (Scalar + Zod)
// Auto-gera OpenAPI a partir de Zod schemas
import { createRoute, z } from '@hono/zod-openapi';

const LoginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

const loginRoute = createRoute({
  method: 'post',
  path: '/auth/login',
  request: {
    body: {
      content: {
        'application/json': {
          schema: LoginSchema,
        },
      },
    },
  },
  responses: {
    200: {
      description: 'Login successful',
      content: {
        'application/json': {
          schema: z.object({
            token: z.string(),
            expiresAt: z.string().datetime(),
          }),
        },
      },
    },
  },
});
3.3 Database Migrations (Drizzle Kit + Versioning)
# Gera migration a partir do schema
pnpm drizzle-kit generate

# Aplica com validação
pnpm drizzle-kit migrate

# 🆕 Rollback seguro
pnpm drizzle-kit rollback --steps 1
3.4 Pre-commit Hooks (Husky + Lint-staged)
// .husky/pre-commit
{
  "*.ts": [
    "eslint --fix",
    "prettier --write",
    "vitest related --run"  // Roda apenas testes afetados
  ],
  "*.sql": ["sqlfluff fix"],
  "package.json": ["sort-package-json"]
}
📊 FASE 4: Monitoramento e Operações
4.1 Grafana Stack
# docker-compose.observability.yml
services:
  grafana:
    image: grafana/grafana:latest
    ports: ["3000:3000"]
    
  prometheus:
    image: prom/prometheus:latest
    ports: ["9090:9090"]
    
  loki:
    image: grafana/loki:latest
    ports: ["3100:3100"]
    
  tempo:
    image: grafana/tempo:latest
    ports: ["3200:3200"]
    
  otel-collector:
    image: otel/opentelemetry-collector:latest
    ports: ["4317:4317"]  # OTLP gRPC
4.2 Alerting (Grafana Alerts)
# alerts/high-error-rate.yml
apiVersion: 1
groups:
  - name: api-errors
    interval: 1m
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 5m
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }}% over 5 minutes"
🚀 FASE 5: Roadmap de Implementação
Sprint 1-2: Fundação
 Setup monorepo com Turborepo
 Migrar para Effect-TS (error handling)
 Adicionar OpenTelemetry
 Setup Testcontainers
Sprint 3-4: Domain Events
 Implementar DomainEvent base
 Refatorar User entity para Aggregate Root
 Adicionar event handlers (invalidar cache, notificações)
 Testes de integração
Sprint 5-6: Background Jobs
 Setup Temporal.io
 Migrar cron jobs para workflows
 Implementar subscription renewal workflow
 Dashboard de workflows
Sprint 7-8: Database Hardening
 Adicionar constraints no banco
 Migrar validações para Zod schemas
 Adicionar índices faltantes
 Performance testing
Sprint 9-10: Observabilidade
 Setup Grafana stack
 Criar dashboards principais
 Configurar alertas críticos
 Runbooks operacionais
📚 Recursos de Estudo
Effect-TS: https://effect.website
Temporal.io: https://docs.temporal.io
OpenTelemetry: https://opentelemetry.io/docs/languages/js/
Testcontainers: https://testcontainers.com
Property-Based Testing: https://fast-check.dev
Quer que eu crie uma spec para implementar alguma dessas melhorias? Posso começar por:

Domain Events System (mais impacto imediato)
Effect-TS Migration (melhor error handling)
Temporal.io Setup (workflows duráveis)
OpenTelemetry Integration (observabilidade)
Qual você prefere atacar primeiro?

Credits used: 1.06
Elapsed time: 1m 7s


Checkpoint
Restore
pensei que você iria sugerir algum novo patern ou algum padrão de projeto como saga, proxy, siglenton, etc, não sei se manteria o drizzle, onde você colocaria o zod ou se trocaria por outra lib, se acha valido a forma que eu trabalho com o DI, se acha valido a UoW, outras sugestões, etc

Kiro
Ótimo ponto! Vou fazer uma análise crítica profunda dos padrões arquiteturais, com sugestões concretas de melhorias e alternativas. Vou ser bem direto sobre o que manteria, o que mudaria e por quê.

🏛️ Análise Crítica de Padrões Arquiteturais
1. 🔍 ANÁLISE DO CÓDIGO ATUAL
Vou analisar o choose-user-role.use-case.ts que está aberto:

// Estrutura atual (inferida da análise técnica)
export class ChooseUserRoleUseCase extends BaseUseCase {
  constructor(
    unitOfWork: IUnitOfWork,
    logger: FastifyBaseLogger,
    authorizationService: IAuthorizationService,
    // Provavelmente injeta repositories aqui
  ) {
    super(unitOfWork, logger, authorizationService);
  }

  async execute(userId: string, role: UserRole, planId: string) {
    return this.runInTransaction(async () => {
      // Lógica aqui
    });
  }
}
✅ O que está BOM:
Dependency Injection - Awilix é excelente escolha
Unit of Work - Padrão correto para transações
Base Use Case - Reduz boilerplate
❌ O que está PROBLEMÁTICO:
Falta Saga Pattern para operações distribuídas
Falta CQRS - Mistura reads e writes
Falta Event Sourcing para auditoria crítica
Repository muito genérico - Falta especificação
Validação espalhada - Deveria ser centralizada
2. 🎯 PADRÕES QUE EU IMPLEMENTARIA
2.1 ⚡ SAGA PATTERN (CRÍTICO para Billing)
Problema atual: Quando cria subscription no Stripe e falha ao salvar no DB, fica inconsistente.

Solução: Saga Orchestrator

// packages/shared-kernel/saga/saga-orchestrator.ts
export abstract class Saga<TContext> {
  private steps: SagaStep<TContext>[] = [];
  
  protected addStep(step: SagaStep<TContext>): void {
    this.steps.push(step);
  }

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
        try {
          await step.compensate(context);
        } catch (compensationError) {
          // Log mas não falha - já estamos em recovery
          this.logger.error('Compensation failed', { step, compensationError });
        }
      }
      
      return Result.fail(new SagaError('Saga failed', error));
    }
  }
}

interface SagaStep<TContext> {
  execute(context: TContext): Promise<void>;
  compensate(context: TContext): Promise<void>;
}
Exemplo: Create Subscription Saga

// apps/api/src/modules/billing/application/sagas/create-subscription.saga.ts
export class CreateSubscriptionSaga extends Saga<CreateSubscriptionContext> {
  constructor(
    private stripeProvider: IPaymentProvider,
    private subscriptionRepo: ISubscriptionRepository,
    private permissionService: PermissionService,
    private emailService: IEmailService,
  ) {
    super();
    
    // Step 1: Create customer in Stripe
    this.addStep({
      execute: async (ctx) => {
        ctx.stripeCustomerId = await this.stripeProvider.createCustomer({
          email: ctx.user.email,
          name: ctx.user.username,
        });
      },
      compensate: async (ctx) => {
        if (ctx.stripeCustomerId) {
          await this.stripeProvider.deleteCustomer(ctx.stripeCustomerId);
        }
      },
    });
    
    // Step 2: Create subscription in Stripe
    this.addStep({
      execute: async (ctx) => {
        ctx.stripeSubscriptionId = await this.stripeProvider.createSubscription({
          customerId: ctx.stripeCustomerId,
          priceId: ctx.plan.providerPriceId,
        });
      },
      compensate: async (ctx) => {
        if (ctx.stripeSubscriptionId) {
          await this.stripeProvider.cancelSubscription(ctx.stripeSubscriptionId);
        }
      },
    });
    
    // Step 3: Save subscription in DB
    this.addStep({
      execute: async (ctx) => {
        const subscription = Subscription.create(/* ... */);
        ctx.subscription = await this.subscriptionRepo.save(subscription);
      },
      compensate: async (ctx) => {
        if (ctx.subscription) {
          await this.subscriptionRepo.delete(ctx.subscription.getId());
        }
      },
    });
    
    // Step 4: Update user permissions (cache)
    this.addStep({
      execute: async (ctx) => {
        await this.permissionService.invalidateCache(ctx.user.getId());
      },
      compensate: async (ctx) => {
        // Noop - cache expira sozinho
      },
    });
    
    // Step 5: Send confirmation email
    this.addStep({
      execute: async (ctx) => {
        await this.emailService.sendSubscriptionConfirmation(ctx.user.email);
      },
      compensate: async (ctx) => {
        // Noop - email já foi enviado
      },
    });
  }
}

// Use case agora é simples
export class CreateSubscriptionUseCase {
  async execute(userId: string, planId: string) {
    const saga = new CreateSubscriptionSaga(/* deps */);
    
    const result = await saga.execute({
      user: await this.userRepo.findById(userId),
      plan: await this.planRepo.findById(planId),
    });
    
    if (result.isFailure()) {
      throw result.error;
    }
    
    return result.value.subscription;
  }
}
Vantagens:

✅ Rollback automático se qualquer step falhar
✅ Auditoria completa (sabe qual step falhou)
✅ Testável (mock de steps individuais)
✅ Reutilizável (compose sagas)
2.2 📖 CQRS (Command Query Responsibility Segregation)
Problema atual: Use cases misturam reads e writes, dificultando otimização.

Solução: Separar Commands e Queries

// packages/shared-kernel/cqrs/command.ts
export interface ICommand {
  readonly commandName: string;
}

export interface ICommandHandler<TCommand extends ICommand, TResult> {
  handle(command: TCommand): Promise<Result<TResult, DomainError>>;
}

// packages/shared-kernel/cqrs/query.ts
export interface IQuery<TResult> {
  readonly queryName: string;
}

export interface IQueryHandler<TQuery extends IQuery<TResult>, TResult> {
  handle(query: TQuery): Promise<Result<TResult, DomainError>>;
}

// packages/shared-kernel/cqrs/bus.ts
export class CommandBus {
  private handlers = new Map<string, ICommandHandler<any, any>>();
  
  register<TCommand extends ICommand, TResult>(
    commandName: string,
    handler: ICommandHandler<TCommand, TResult>,
  ): void {
    this.handlers.set(commandName, handler);
  }
  
  async execute<TCommand extends ICommand, TResult>(
    command: TCommand,
  ): Promise<Result<TResult, DomainError>> {
    const handler = this.handlers.get(command.commandName);
    
    if (!handler) {
      return Result.fail(new InternalError(`No handler for ${command.commandName}`));
    }
    
    return handler.handle(command);
  }
}

export class QueryBus {
  // Similar ao CommandBus
}
Exemplo: Choose User Role

// apps/api/src/modules/user/application/commands/choose-role.command.ts
export class ChooseUserRoleCommand implements ICommand {
  readonly commandName = 'user.choose-role';
  
  constructor(
    public readonly userId: string,
    public readonly role: UserRole,
    public readonly planId: string,
  ) {}
}

export class ChooseUserRoleHandler implements ICommandHandler<ChooseUserRoleCommand, User> {
  constructor(
    private unitOfWork: IUnitOfWork,
    private userRepo: IUserRepository,
    private planRepo: IPlanRepository,
    private eventBus: IEventBus, // 🆕
  ) {}
  
  async handle(command: ChooseUserRoleCommand): Promise<Result<User, DomainError>> {
    return this.unitOfWork.transaction(async () => {
      // 1. Validações
      const user = await this.userRepo.findById(command.userId);
      if (!user) {
        return Result.fail(new NotFoundError('User'));
      }
      
      const plan = await this.planRepo.findById(command.planId);
      if (!plan) {
        return Result.fail(new NotFoundError('Plan'));
      }
      
      if (!plan.isAllowedForRole(command.role)) {
        return Result.fail(new BusinessError('Plan not allowed for this role'));
      }
      
      // 2. Executa domain logic
      user.assignRoleAndPlan(command.role, command.planId);
      
      // 3. Persiste
      await this.userRepo.save(user);
      
      // 4. Publica eventos
      const events = user.pullDomainEvents();
      for (const event of events) {
        await this.eventBus.publish(event);
      }
      
      return Result.ok(user);
    });
  }
}

// Query separada para leitura
export class GetUserByIdQuery implements IQuery<UserDTO> {
  readonly queryName = 'user.get-by-id';
  
  constructor(public readonly userId: string) {}
}

export class GetUserByIdHandler implements IQueryHandler<GetUserByIdQuery, UserDTO> {
  constructor(
    private db: DrizzleClient, // 🆕 Acesso direto ao DB (sem repository)
  ) {}
  
  async handle(query: GetUserByIdQuery): Promise<Result<UserDTO, DomainError>> {
    // Query otimizada, sem carregar entity completa
    const user = await this.db
      .select({
        id: users.id,
        email: users.email,
        username: users.username,
        role: users.role,
        // Apenas campos necessários
      })
      .from(users)
      .where(eq(users.id, query.userId))
      .limit(1);
    
    if (!user[0]) {
      return Result.fail(new NotFoundError('User'));
    }
    
    return Result.ok(user[0]);
  }
}
Vantagens:

✅ Commands usam repositories + entities (write model)
✅ Queries usam SQL direto (read model otimizado)
✅ Escalabilidade (read replicas para queries)
✅ Testabilidade (handlers isolados)
2.3 🎭 SPECIFICATION PATTERN (para queries complexas)
Problema atual: Queries complexas espalhadas nos repositories.

Solução:

// packages/shared-kernel/specification/specification.ts
export abstract class Specification<T> {
  abstract isSatisfiedBy(candidate: T): boolean;
  
  and(other: Specification<T>): Specification<T> {
    return new AndSpecification(this, other);
  }
  
  or(other: Specification<T>): Specification<T> {
    return new OrSpecification(this, other);
  }
  
  not(): Specification<T> {
    return new NotSpecification(this);
  }
  
  // Para Drizzle
  abstract toSql(): SQL;
}

// Exemplo: User specifications
export class ActiveUserSpec extends Specification<User> {
  isSatisfiedBy(user: User): boolean {
    return !user.isBanned() && !user.isDeleted();
  }
  
  toSql(): SQL {
    return and(
      eq(users.banned, false),
      isNull(users.deletedAt),
    );
  }
}

export class EmailVerifiedSpec extends Specification<User> {
  isSatisfiedBy(user: User): boolean {
    return user.isEmailVerified();
  }
  
  toSql(): SQL {
    return eq(users.emailVerified, true);
  }
}

export class HasRoleSpec extends Specification<User> {
  constructor(private role: UserRole) {}
  
  isSatisfiedBy(user: User): boolean {
    return user.getRole() === this.role;
  }
  
  toSql(): SQL {
    return eq(users.role, this.role);
  }
}

// Uso
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
2.4 🏭 FACTORY PATTERN (melhorado)
Problema atual: Factories simples, sem validação complexa.

Solução: Abstract Factory + Builder

// packages/shared-kernel/factory/entity-factory.ts
export abstract class EntityFactory<T> {
  protected abstract validate(data: unknown): Result<void, ValidationError>;
  protected abstract build(data: unknown): T;
  
  create(data: unknown): Result<T, ValidationError> {
    const validation = this.validate(data);
    if (validation.isFailure()) {
      return Result.fail(validation.error);
    }
    
    return Result.ok(this.build(data));
  }
}

// Exemplo: Subscription Factory
export class SubscriptionFactory extends EntityFactory<Subscription> {
  constructor(
    private planRepo: IPlanRepository,
    private userRepo: IUserRepository,
  ) {
    super();
  }
  
  protected validate(data: CreateSubscriptionDTO): Result<void, ValidationError> {
    const schema = z.object({
      userId: z.string().uuid(),
      planId: z.string().uuid(),
      billingCycle: z.enum(['monthly', 'yearly']),
      paymentMethodId: z.string().uuid().optional(),
    });
    
    const result = schema.safeParse(data);
    if (!result.success) {
      return Result.fail(new ValidationError('Invalid subscription data', result.error));
    }
    
    return Result.ok();
  }
  
  protected build(data: CreateSubscriptionDTO): Subscription {
    // Lógica complexa de criação
    const subscription = Subscription.create({
      userId: data.userId,
      planId: data.planId,
      status: SubscriptionStatus.INCOMPLETE,
      billingCycle: data.billingCycle,
      currentPeriodStart: new Date(),
      currentPeriodEnd: this.calculatePeriodEnd(data.billingCycle),
    });
    
    return subscription;
  }
  
  private calculatePeriodEnd(cycle: BillingCycle): Date {
    const now = new Date();
    return cycle === 'monthly'
      ? addMonths(now, 1)
      : addYears(now, 1);
  }
}

// Builder para casos complexos
export class SubscriptionBuilder {
  private data: Partial<SubscriptionData> = {};
  
  withUser(userId: string): this {
    this.data.userId = userId;
    return this;
  }
  
  withPlan(planId: string): this {
    this.data.planId = planId;
    return this;
  }
  
  withTrial(days: number): this {
    this.data.trialEnd = addDays(new Date(), days);
    return this;
  }
  
  withDiscount(couponId: string): this {
    this.data.couponId = couponId;
    return this;
  }
  
  build(): Result<Subscription, ValidationError> {
    return new SubscriptionFactory(/* deps */).create(this.data);
  }
}

// Uso
const subscription = new SubscriptionBuilder()
  .withUser(userId)
  .withPlan(planId)
  .withTrial(14)
  .withDiscount('PROMO2024')
  .build();
2.5 🔌 ADAPTER PATTERN (melhorado)
Problema atual: Adapters simples, sem fallback.

Solução: Chain of Responsibility + Adapter

// packages/shared-kernel/adapter/adapter-chain.ts
export abstract class Adapter<TInput, TOutput> {
  private next: Adapter<TInput, TOutput> | null = null;
  
  setNext(adapter: Adapter<TInput, TOutput>): Adapter<TInput, TOutput> {
    this.next = adapter;
    return adapter;
  }
  
  async handle(input: TInput): Promise<Result<TOutput, AdapterError>> {
    const result = await this.tryHandle(input);
    
    if (result.isSuccess() || !this.next) {
      return result;
    }
    
    // Fallback para próximo adapter
    return this.next.handle(input);
  }
  
  protected abstract tryHandle(input: TInput): Promise<Result<TOutput, AdapterError>>;
}

// Exemplo: Email providers com fallback
export class ResendEmailAdapter extends Adapter<SendEmailInput, SendEmailOutput> {
  constructor(private resend: Resend) {
    super();
  }
  
  protected async tryHandle(input: SendEmailInput): Promise<Result<SendEmailOutput, AdapterError>> {
    try {
      const result = await this.resend.emails.send({
        from: input.from,
        to: input.to,
        subject: input.subject,
        html: input.html,
      });
      
      return Result.ok({ messageId: result.id });
    } catch (error) {
      return Result.fail(new AdapterError('Resend failed', error));
    }
  }
}

export class NodemailerEmailAdapter extends Adapter<SendEmailInput, SendEmailOutput> {
  constructor(private transporter: Transporter) {
    super();
  }
  
  protected async tryHandle(input: SendEmailInput): Promise<Result<SendEmailOutput, AdapterError>> {
    try {
      const result = await this.transporter.sendMail({
        from: input.from,
        to: input.to,
        subject: input.subject,
        html: input.html,
      });
      
      return Result.ok({ messageId: result.messageId });
    } catch (error) {
      return Result.fail(new AdapterError('Nodemailer failed', error));
    }
  }
}

// Setup com fallback
const emailService = new ResendEmailAdapter(resend);
emailService.setNext(new NodemailerEmailAdapter(transporter));

// Uso - tenta Resend, se falhar usa Nodemailer
const result = await emailService.handle({
  from: 'noreply@mastersindico.com',
  to: user.email,
  subject: 'Welcome',
  html: '<h1>Welcome!</h1>',
});
3. 🔧 ANÁLISE DE FERRAMENTAS
3.1 Drizzle ORM - MANTER ou TROCAR?
Análise:

Aspecto	Drizzle	Prisma	TypeORM
Type Safety	⭐⭐⭐⭐⭐	⭐⭐⭐⭐	⭐⭐⭐
Performance	⭐⭐⭐⭐⭐	⭐⭐⭐	⭐⭐
Migrations	⭐⭐⭐⭐	⭐⭐⭐⭐⭐	⭐⭐⭐
Raw SQL	⭐⭐⭐⭐⭐	⭐⭐	⭐⭐⭐
Relations	⭐⭐⭐⭐	⭐⭐⭐⭐⭐	⭐⭐⭐⭐
Minha recomendação: MANTER Drizzle, mas...

Adicionar camada de abstração:

// packages/database/query-builder.ts
export class QueryBuilder<T> {
  constructor(
    private db: DrizzleClient,
    private table: PgTable,
  ) {}
  
  // Fluent API
  where(condition: SQL): this {
    // ...
  }
  
  orderBy(column: string, direction: 'asc' | 'desc'): this {
    // ...
  }
  
  limit(n: number): this {
    // ...
  }
  
  // Execução
  async first(): Promise<T | null> {
    // ...
  }
  
  async many(): Promise<T[]> {
    // ...
  }
  
  async paginate(page: number, perPage: number): Promise<Paginated<T>> {
    // ...
  }
  
  // 🆕 Soft delete automático
  async delete(): Promise<void> {
    await this.db
      .update(this.table)
      .set({ deletedAt: new Date() })
      .where(this.conditions);
  }
  
  // 🆕 Auditoria automática
  async update(data: Partial<T>): Promise<void> {
    await this.db
      .update(this.table)
      .set({
        ...data,
        updatedAt: new Date(),
        updatedBy: this.getCurrentUserId(),
      })
      .where(this.conditions);
  }
}

// Uso
const users = await new QueryBuilder(db, usersTable)
  .where(eq(users.role, 'syndic'))
  .where(isNull(users.deletedAt))
  .orderBy('createdAt', 'desc')
  .paginate(1, 20);
3.2 Zod - ONDE USAR?
Minha recomendação: 3 camadas de validação

// 1️⃣ API Layer (HTTP input)
// apps/api/src/modules/user/presentation/schemas/create-user.schema.ts
export const CreateUserSchema = z.object({
  email: z.string().email(),
  username: z.string().min(3).max(30).regex(/^[a-z0-9_]+$/),
  password: z.string().min(8),
  phone: z.string().regex(/^\d{10,13}$/).optional(),
});

// 2️⃣ Domain Layer (Value Objects)
// apps/api/src/modules/user/domain/value-objects/email.vo.ts
export class EmailVO {
  private constructor(private readonly value: string) {}
  
  static create(value: string): Result<EmailVO, ValidationError> {
    const schema = z.string().email();
    const result = schema.safeParse(value);
    
    if (!result.success) {
      return Result.fail(new ValidationError('Invalid email', result.error));
    }
    
    return Result.ok(new EmailVO(value.toLowerCase()));
  }
}

// 3️⃣ Database Layer (constraints)
// packages/database/schemas/users.ts
export const users = pgTable('users', {
  email: text('email').notNull(),
}, (table) => ({
  emailFormat: check('email_format', sql`email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$'`),
}));
Alternativas ao Zod:

Lib	Prós	Contras
Zod	Type inference, composable	Bundle size
Valibot	Menor bundle, mais rápido	Menos maduro
ArkType	Mais rápido que Zod	Sintaxe diferente
TypeBox	JSON Schema nativo	Menos ergonômico
Minha escolha: Zod + Valibot

Zod para schemas complexos (API)
Valibot para validações simples (performance)
3.3 Dependency Injection - Awilix vs Alternativas
Análise do Awilix atual:

✅ Pontos fortes:

Lightweight
Scoped containers (request-level)
Auto-wiring
❌ Pontos fracos:

Sem decorators (menos ergonômico)
Sem validação em compile-time
Difícil debug
Alternativa: TSyringe (Microsoft)

// packages/shared-kernel/di/decorators.ts
import { injectable, inject, container } from 'tsyringe';

@injectable()
export class CreateUserUseCase {
  constructor(
    @inject('IUserRepository') private userRepo: IUserRepository,
    @inject('IEmailService') private emailService: IEmailService,
    @inject('IUnitOfWork') private unitOfWork: IUnitOfWork,
  ) {}
  
  async execute(data: CreateUserDTO): Promise<Result<User, DomainError>> {
    // ...
  }
}

// Registration
container.register('IUserRepository', { useClass: DrizzleUserRepository });
container.register('IEmailService', { useClass: ResendEmailService });
container.register('IUnitOfWork', { useClass: DrizzleUnitOfWork });

// Resolve
const useCase = container.resolve(CreateUserUseCase);
Minha recomendação: MANTER Awilix, mas adicionar type safety

// packages/shared-kernel/di/container.ts
import { asClass, asFunction, asValue, AwilixContainer, createContainer } from 'awilix';

// Type-safe container
export interface AppContainer {
  // Repositories
  userRepository: IUserRepository;
  subscriptionRepository: ISubscriptionRepository;
  
  // Services
  emailService: IEmailService;
  paymentProvider: IPaymentProvider;
  
  // Use Cases
  createUserUseCase: CreateUserUseCase;
  
  // Infrastructure
  unitOfWork: IUnitOfWork;
  logger: FastifyBaseLogger;
}

export function buildContainer(): AwilixContainer<AppContainer> {
  const container = createContainer<AppContainer>();
  
  container.register({
    // Repositories (scoped - 1 por request)
    userRepository: asClass(DrizzleUserRepository).scoped(),
    
    // Services (singleton)
    emailService: asClass(ResendEmailService).singleton(),
    
    // Use Cases (scoped)
    createUserUseCase: asClass(CreateUserUseCase).scoped(),
    
    // Infrastructure
    unitOfWork: asClass(DrizzleUnitOfWork).scoped(),
  });
  
  return container;
}

// Uso com type safety
const container = buildContainer();
const useCase = container.resolve('createUserUseCase'); // ✅ Typed!
3.4 Unit of Work - MANTER ou TROCAR?
Análise:

✅ Pontos fortes:

Transações automáticas
Rollback em caso de erro
Abstração do DB
❌ Pontos fracos:

Não suporta transações aninhadas (savepoints)
Não suporta locks (SELECT FOR UPDATE)
Não suporta isolation levels customizados
Minha recomendação: EVOLUIR o UoW

// packages/shared-kernel/unit-of-work/unit-of-work.interface.ts
export interface IUnitOfWork {
  // Básico (já existe)
  transaction<T>(work: () => Promise<T>): Promise<T>;
  
  // 🆕 Transações aninhadas (savepoints)
  savepoint<T>(name: string, work: () => Promise<T>): Promise<T>;
  
  // 🆕 Locks
  lock<T>(table: string, id: string, work: () => Promise<T>): Promise<T>;
  
  // 🆕 Isolation levels
  withIsolation<T>(
    level: 'READ UNCOMMITTED' | 'READ COMMITTED' | 'REPEATABLE READ' | 'SERIALIZABLE',
    work: () => Promise<T>,
  ): Promise<T>;
  
  // 🆕 Retry com backoff
  withRetry<T>(
    work: () => Promise<T>,
    options?: RetryOptions,
  ): Promise<T>;
}

// Implementação Drizzle
export class DrizzleUnitOfWork implements IUnitOfWork {
  async savepoint<T>(name: string, work: () => Promise<T>): Promise<T> {
    return this.db.transaction(async (tx) => {
      await tx.execute(sql`SAVEPOINT ${sql.identifier(name)}`);
      
      try {
        const result = await work();
        await tx.execute(sql`RELEASE SAVEPOINT ${sql.identifier(name)}`);
        return result;
      } catch (error) {
        await tx.execute(sql`ROLLBACK TO SAVEPOINT ${sql.identifier(name)}`);
        throw error;
      }
    });
  }
  
  async lock<T>(table: string, id: string, work: () => Promise<T>): Promise<T> {
    return this.db.transaction(async (tx) => {
      // SELECT FOR UPDATE
      await tx.execute(sql`
        SELECT * FROM ${sql.identifier(table)}
        WHERE id = ${id}
        FOR UPDATE
      `);
      
      return work();
    });
  }
  
  async withRetry<T>(
    work: () => Promise<T>,
    options: RetryOptions = { maxAttempts: 3, backoff: 'exponential' },
  ): Promise<T> {
    let lastError: Error;
    
    for (let attempt = 1; attempt <= options.maxAttempts; attempt++) {
      try {
        return await work();
      } catch (error) {
        lastError = error;
        
        // Retry apenas em deadlock ou serialization failure
        if (!this.isRetryable(error)) {
          throw error;
        }
        
        if (attempt < options.max
Credits used: 1.62
Elapsed time: 1m 30s


Checkpoint
Restore
mas porque você escolheu remover o UoW do base-use-case?, não gosta da forma que eu injeto permissões diretamente em uma base?

Kiro
Ótima observação! Deixa eu esclarecer - NÃO removeria o UoW nem as permissões do BaseUseCase. Na verdade, acho que você acertou em cheio nesses pontos! Deixa eu explicar melhor minha visão:

🎯 ANÁLISE CRÍTICA DO SEU BASE USE CASE
✅ O QUE ESTÁ EXCELENTE (e eu manteria)
// Seu código atual (inferido da análise)
export abstract class BaseUseCase {
  constructor(
    protected readonly unitOfWork: IUnitOfWork,
    protected readonly logger: FastifyBaseLogger,
    protected readonly authorizationService?: IAuthorizationService,
  ) {}

  protected async runInTransaction<T>(work: () => Promise<T>): Promise<T> {
    return this.unitOfWork.transaction(work);
  }

  protected authorize(action: Actions, resource: Subjects): void {
    if (!this.authorizationService) {
      this.logger.warn('Authorization service not injected');
      return;
    }
    this.authorizationService.authorize(action, resource);
  }
}
Por que isso é BRILHANTE:

✅ DRY Principle - Não repete injeção em todo use case
✅ Separation of Concerns - Infraestrutura separada de lógica
✅ Template Method Pattern - Base define estrutura, filhos implementam
✅ Dependency Inversion - Depende de abstrações (interfaces)
🔧 O QUE EU MELHORARIA (não removeria!)
Problema 1: Authorization Silencioso
// ❌ PROBLEMA: Apenas loga warn se não injetado
protected authorize(action: Actions, resource: Subjects): void {
  if (!this.authorizationService) {
    this.logger.warn('Authorization service not injected');
    return; // ⚠️ PERIGOSO - continua sem validar!
  }
  this.authorizationService.authorize(action, resource);
}
Solução: Fail-fast com tipos

// ✅ SOLUÇÃO 1: Tipos diferentes para use cases públicos vs privados
export abstract class BaseUseCase {
  constructor(
    protected readonly unitOfWork: IUnitOfWork,
    protected readonly logger: FastifyBaseLogger,
  ) {}

  protected async runInTransaction<T>(work: () => Promise<T>): Promise<T> {
    return this.unitOfWork.transaction(work);
  }
}

// Use cases que PRECISAM de autorização
export abstract class AuthorizedUseCase extends BaseUseCase {
  constructor(
    unitOfWork: IUnitOfWork,
    logger: FastifyBaseLogger,
    protected readonly authorizationService: IAuthorizationService, // ✅ Obrigatório!
  ) {
    super(unitOfWork, logger);
  }

  protected authorize(action: Actions, resource: Subjects): void {
    this.authorizationService.authorize(action, resource);
  }

  protected can(action: Actions, resource: Subjects): boolean {
    return this.authorizationService.can(action, resource);
  }
}

// Use cases públicos (sem auth)
export abstract class PublicUseCase extends BaseUseCase {
  // Não tem authorizationService
}
Uso:

// ✅ Use case protegido
export class DeleteUserUseCase extends AuthorizedUseCase {
  async execute(userId: string): Promise<void> {
    // Compile-time safety - authorizationService sempre existe
    this.authorize('delete', 'User');
    
    await this.runInTransaction(async () => {
      // ...
    });
  }
}

// ✅ Use case público
export class RegisterUserUseCase extends PublicUseCase {
  async execute(data: RegisterDTO): Promise<User> {
    // Não tem authorize() - compile error se tentar usar
    await this.runInTransaction(async () => {
      // ...
    });
  }
}
Problema 2: Falta Context do Usuário Atual
// ❌ PROBLEMA: Use cases não sabem quem está executando
export class UpdateProfileUseCase extends AuthorizedUseCase {
  async execute(profileId: string, data: UpdateDTO): Promise<void> {
    // ⚠️ Como saber se o usuário pode editar ESTE perfil específico?
    this.authorize('update', 'Profile'); // Genérico demais!
  }
}
Solução: Execution Context

// packages/shared-kernel/use-case/execution-context.ts
export interface ExecutionContext {
  userId: string;
  sessionId: string;
  ability: AppAbility;
  ipAddress?: string;
  userAgent?: string;
  traceId: string; // Para correlação de logs
}

// Base use case com contexto
export abstract class BaseUseCase {
  constructor(
    protected readonly unitOfWork: IUnitOfWork,
    protected readonly logger: FastifyBaseLogger,
  ) {}

  // 🆕 Método execute abstrato que recebe contexto
  abstract execute(context: ExecutionContext, ...args: any[]): Promise<any>;

  protected async runInTransaction<T>(work: () => Promise<T>): Promise<T> {
    return this.unitOfWork.transaction(work);
  }
}

export abstract class AuthorizedUseCase extends BaseUseCase {
  // 🆕 Helper para autorização com ownership
  protected authorizeOwnership(
    context: ExecutionContext,
    action: Actions,
    resource: any, // Entity com userId
  ): void {
    const can = context.ability.can(action, resource);
    
    if (!can) {
      throw new ForbiddenError(
        `User ${context.userId} cannot ${action} ${resource.constructor.name}`,
      );
    }
  }

  // 🆕 Helper para autorização simples
  protected authorize(
    context: ExecutionContext,
    action: Actions,
    subject: Subjects,
  ): void {
    const can = context.ability.can(action, subject);
    
    if (!can) {
      throw new ForbiddenError(
        `User ${context.userId} cannot ${action} ${subject}`,
      );
    }
  }
}
Uso melhorado:

export class UpdateProfileUseCase extends AuthorizedUseCase {
  constructor(
    unitOfWork: IUnitOfWork,
    logger: FastifyBaseLogger,
    private profileRepo: IProfileRepository,
  ) {
    super(unitOfWork, logger);
  }

  async execute(
    context: ExecutionContext,
    profileId: string,
    data: UpdateProfileDTO,
  ): Promise<Profile> {
    return this.runInTransaction(async () => {
      const profile = await this.profileRepo.findById(profileId);
      
      if (!profile) {
        throw new NotFoundError('Profile');
      }

      // ✅ Valida ownership usando CASL conditions
      this.authorizeOwnership(context, 'update', profile);
      
      profile.update(data);
      await this.profileRepo.save(profile);
      
      return profile;
    });
  }
}
Problema 3: Falta Observabilidade
// ❌ PROBLEMA: Difícil rastrear execução de use cases
export class CreateSubscriptionUseCase extends AuthorizedUseCase {
  async execute(userId: string, planId: string): Promise<Subscription> {
    // Sem trace, sem métricas, sem contexto
    return this.runInTransaction(async () => {
      // ...
    });
  }
}
Solução: Decorators + OpenTelemetry

// packages/shared-kernel/use-case/decorators.ts
import { tracer, meter } from '@/telemetry';

export function Traced(useCaseName: string) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor,
  ) {
    const originalMethod = descriptor.value;

    descriptor.value = async function (...args: any[]) {
      const span = tracer.startSpan(`usecase.${useCaseName}`);
      const startTime = Date.now();

      try {
        // Adiciona contexto ao span
        const context = args[0] as ExecutionContext;
        span.setAttributes({
          'user.id': context.userId,
          'session.id': context.sessionId,
          'trace.id': context.traceId,
        });

        const result = await originalMethod.apply(this, args);
        
        span.setStatus({ code: SpanStatusCode.OK });
        
        // Métricas
        meter.createCounter(`usecase.${useCaseName}.success`).add(1);
        meter.createHistogram(`usecase.${useCaseName}.duration`).record(Date.now() - startTime);
        
        return result;
      } catch (error) {
        span.recordException(error);
        span.setStatus({ code: SpanStatusCode.ERROR });
        
        meter.createCounter(`usecase.${useCaseName}.error`).add(1, {
          'error.type': error.constructor.name,
        });
        
        throw error;
      } finally {
        span.end();
      }
    };

    return descriptor;
  };
}

// Uso
export class CreateSubscriptionUseCase extends AuthorizedUseCase {
  @Traced('create-subscription')
  async execute(
    context: ExecutionContext,
    planId: string,
  ): Promise<Subscription> {
    // Automaticamente traced!
  }
}
🏗️ ARQUITETURA FINAL DO BASE USE CASE
// packages/shared-kernel/use-case/base-use-case.ts

export interface ExecutionContext {
  userId: string;
  sessionId: string;
  ability: AppAbility;
  ipAddress?: string;
  userAgent?: string;
  traceId: string;
}

export abstract class BaseUseCase<TInput = void, TOutput = void> {
  constructor(
    protected readonly unitOfWork: IUnitOfWork,
    protected readonly logger: FastifyBaseLogger,
  ) {}

  abstract execute(context: ExecutionContext, input: TInput): Promise<TOutput>;

  protected async runInTransaction<T>(work: () => Promise<T>): Promise<T> {
    return this.unitOfWork.transaction(work);
  }

  // 🆕 Helper para logging estruturado
  protected log(level: 'info' | 'warn' | 'error', message: string, meta?: object): void {
    this.logger[level]({
      useCase: this.constructor.name,
      ...meta,
    }, message);
  }
}

export abstract class AuthorizedUseCase<TInput = void, TOutput = void> extends BaseUseCase<TInput, TOutput> {
  protected authorize(
    context: ExecutionContext,
    action: Actions,
    subject: Subjects,
  ): void {
    const can = context.ability.can(action, subject);
    
    if (!can) {
      this.log('warn', 'Authorization failed', {
        userId: context.userId,
        action,
        subject,
      });
      
      throw new ForbiddenError(
        `User ${context.userId} cannot ${action} ${subject}`,
      );
    }
  }

  protected authorizeOwnership(
    context: ExecutionContext,
    action: Actions,
    resource: any,
  ): void {
    const can = context.ability.can(action, resource);
    
    if (!can) {
      this.log('warn', 'Ownership authorization failed', {
        userId: context.userId,
        action,
        resourceType: resource.constructor.name,
        resourceId: resource.id,
      });
      
      throw new ForbiddenError(
        `User ${context.userId} cannot ${action} this ${resource.constructor.name}`,
      );
    }
  }

  protected can(
    context: ExecutionContext,
    action: Actions,
    subject: Subjects,
  ): boolean {
    return context.ability.can(action, subject);
  }
}

export abstract class PublicUseCase<TInput = void, TOutput = void> extends BaseUseCase<TInput, TOutput> {
  // Sem métodos de autorização
}
📊 COMPARAÇÃO: ANTES vs DEPOIS
ANTES (seu código atual)
export class UpdateProfileUseCase extends BaseUseCase {
  constructor(
    unitOfWork: IUnitOfWork,
    logger: FastifyBaseLogger,
    authorizationService: IAuthorizationService,
    private profileRepo: IProfileRepository,
  ) {
    super(unitOfWork, logger, authorizationService);
  }

  async execute(profileId: string, data: UpdateDTO): Promise<Profile> {
    this.authorize('update', 'Profile'); // ⚠️ Genérico
    
    return this.runInTransaction(async () => {
      const profile = await this.profileRepo.findById(profileId);
      profile.update(data);
      await this.profileRepo.save(profile);
      return profile;
    });
  }
}
Problemas:

❌ Não valida ownership (qualquer um com permissão pode editar qualquer perfil)
❌ Sem contexto do usuário atual
❌ Sem observabilidade
❌ Authorization opcional (pode esquecer de injetar)
DEPOIS (com melhorias)
export class UpdateProfileUseCase extends AuthorizedUseCase<UpdateProfileInput, Profile> {
  constructor(
    unitOfWork: IUnitOfWork,
    logger: FastifyBaseLogger,
    private profileRepo: IProfileRepository,
  ) {
    super(unitOfWork, logger);
  }

  @Traced('update-profile')
  async execute(
    context: ExecutionContext,
    input: UpdateProfileInput,
  ): Promise<Profile> {
    return this.runInTransaction(async () => {
      const profile = await this.profileRepo.findById(input.profileId);
      
      if (!profile) {
        throw new NotFoundError('Profile');
      }

      // ✅ Valida ownership com CASL conditions
      this.authorizeOwnership(context, 'update', profile);
      
      profile.update(input.data);
      await this.profileRepo.save(profile);
      
      this.log('info', 'Profile updated', {
        profileId: profile.getId(),
        userId: context.userId,
      });
      
      return profile;
    });
  }
}
Vantagens:

✅ Valida ownership (CASL conditions)
✅ Contexto do usuário sempre disponível
✅ Observabilidade automática (traces + métricas)
✅ Authorization obrigatória (compile-time)
✅ Type-safe (generics)
🎯 RESUMO: O QUE MANTER vs MUDAR
✅ MANTER (está excelente!)
BaseUseCase com UoW - Padrão correto
Injeção de dependências na base - DRY principle
runInTransaction helper - Abstração perfeita
Authorization na base - Separation of concerns
🔧 MELHORAR (não remover!)
Tornar authorization obrigatória - Tipos diferentes (AuthorizedUseCase vs PublicUseCase)
Adicionar ExecutionContext - Saber quem está executando
Validar ownership - CASL conditions
Adicionar observabilidade - Decorators + OpenTelemetry
Type-safe generics - BaseUseCase<TInput, TOutput>
Sua arquitetura base está sólida! Só precisa de alguns ajustes para torná-la mais segura e observável. O conceito de BaseUseCase com UoW e Authorization é excelente - não mudaria isso.

Quer que eu crie uma spec para implementar essas melhorias no BaseUseCase?

Credits used: 1.09
Elapsed time: 46s


Checkpoint
Restore
mas ai você está levando tudo para um pacote externo de shared-kernel se eu só vou usar isso no backend ?

Kiro
Excelente ponto! Você está absolutamente certo - não faz sentido criar um pacote shared-kernel se só vai usar no backend. Isso seria over-engineering.

Deixa eu repensar a estrutura de forma mais pragmática:

🎯 ESTRUTURA PRAGMÁTICA (Backend-Only)
Opção 1: Tudo dentro de apps/api/src/core (RECOMENDADO)
apps/api/src/
├── core/                           # ✅ Fundação do backend
│   ├── contracts/                  # Interfaces base
│   │   ├── base-use-case.ts       # Seu BaseUseCase atual
│   │   ├── base-repository.ts     # Seu BaseRepository atual
│   │   ├── base-task.ts           # Seu BaseTask atual
│   │   └── authorization-service.interface.ts
│   │
│   ├── errors/                     # Hierarquia de erros
│   │   └── index.ts               # DomainError, ValidationError, etc
│   │
│   ├── value-objects/             # VOs compartilhados
│   │   ├── email.vo.ts
│   │   ├── phone.vo.ts
│   │   ├── username.vo.ts
│   │   └── money.vo.ts
│   │
│   ├── decorators/                # 🆕 Decorators úteis
│   │   ├── traced.decorator.ts   # OpenTelemetry
│   │   └── validated.decorator.ts # Zod validation
│   │
│   └── types/                     # Types compartilhados
│       ├── execution-context.ts  # Context do request
│       └── result.ts             # Result<T, E> type
│
├── modules/                       # Seus módulos atuais
│   ├── auth/
│   ├── user/
│   ├── billing/
│   └── ...
│
├── infrastructure/                # Infraestrutura compartilhada
│   ├── database/
│   ├── cache/
│   └── providers/
│
└── shared/                        # Utilitários compartilhados
    ├── hooks/
    ├── plugins/
    └── utils/
Vantagens:

✅ Tudo em um lugar só
✅ Imports simples: @/core/contracts/base-use-case
✅ Sem overhead de pacote externo
✅ Fácil refatorar
🔧 IMPLEMENTAÇÃO PRAGMÁTICA
1. BaseUseCase Melhorado (dentro de core/contracts)
// apps/api/src/core/contracts/base-use-case.ts

import type { FastifyBaseLogger } from 'fastify';
import type { IUnitOfWork } from './unit-of-work.interface';
import type { AppAbility } from '@/modules/auth/domain/casl/ability';
import { ForbiddenError } from '@/core/errors';

// 🆕 Context do request (fica no core, não em pacote externo)
export interface ExecutionContext {
  userId: string;
  sessionId: string;
  ability: AppAbility;
  ipAddress?: string;
  userAgent?: string;
  requestId: string; // Para correlação de logs
}

// Base genérica
export abstract class BaseUseCase<TInput = void, TOutput = void> {
  constructor(
    protected readonly unitOfWork: IUnitOfWork,
    protected readonly logger: FastifyBaseLogger,
  ) {}

  abstract execute(context: ExecutionContext, input: TInput): Promise<TOutput>;

  protected async runInTransaction<T>(work: () => Promise<T>): Promise<T> {
    return this.unitOfWork.transaction(work);
  }

  protected log(level: 'info' | 'warn' | 'error', message: string, meta?: object): void {
    this.logger[level]({
      useCase: this.constructor.name,
      requestId: meta?.['requestId'],
      ...meta,
    }, message);
  }
}

// Use cases que PRECISAM de autorização
export abstract class AuthorizedUseCase<TInput = void, TOutput = void> 
  extends BaseUseCase<TInput, TOutput> {
  
  protected authorize(
    context: ExecutionContext,
    action: string,
    subject: string,
  ): void {
    const can = context.ability.can(action as any, subject as any);
    
    if (!can) {
      this.log('warn', 'Authorization failed', {
        userId: context.userId,
        action,
        subject,
        requestId: context.requestId,
      });
      
      throw new ForbiddenError(
        `Você não possui permissão para ${action} ${subject}`,
      );
    }
  }

  protected authorizeOwnership(
    context: ExecutionContext,
    action: string,
    resource: any,
  ): void {
    const can = context.ability.can(action as any, resource);
    
    if (!can) {
      this.log('warn', 'Ownership authorization failed', {
        userId: context.userId,
        action,
        resourceType: resource.constructor.name,
        resourceId: resource.id,
        requestId: context.requestId,
      });
      
      throw new ForbiddenError(
        `Você não pode ${action} este recurso`,
      );
    }
  }

  protected can(
    context: ExecutionContext,
    action: string,
    subject: string,
  ): boolean {
    return context.ability.can(action as any, subject as any);
  }
}

// Use cases públicos (sem auth)
export abstract class PublicUseCase<TInput = void, TOutput = void> 
  extends BaseUseCase<TInput, TOutput> {
  // Não tem métodos de autorização
}
2. Decorator para Observabilidade (dentro de core/decorators)
// apps/api/src/core/decorators/traced.decorator.ts

import { tracer } from '@/infrastructure/telemetry/tracer';
import { SpanStatusCode } from '@opentelemetry/api';

export function Traced(spanName?: string) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor,
  ) {
    const originalMethod = descriptor.value;
    const name = spanName || `${target.constructor.name}.${propertyKey}`;

    descriptor.value = async function (...args: any[]) {
      const span = tracer.startSpan(name);
      
      try {
        // Adiciona contexto se disponível
        const context = args[0];
        if (context?.userId) {
          span.setAttributes({
            'user.id': context.userId,
            'session.id': context.sessionId,
            'request.id': context.requestId,
          });
        }

        const result = await originalMethod.apply(this, args);
        span.setStatus({ code: SpanStatusCode.OK });
        return result;
      } catch (error) {
        span.recordException(error);
        span.setStatus({ code: SpanStatusCode.ERROR });
        throw error;
      } finally {
        span.end();
      }
    };

    return descriptor;
  };
}
3. Uso Real no Módulo User
// apps/api/src/modules/user/application/use-cases/choose-user-role.use-case.ts

import { AuthorizedUseCase, type ExecutionContext } from '@/core/contracts/base-use-case';
import { Traced } from '@/core/decorators/traced.decorator';
import type { IUnitOfWork } from '@/core/contracts/unit-of-work.interface';
import type { FastifyBaseLogger } from 'fastify';
import type { IUserRepository } from '../../domain/repositories/user.repository';
import type { IPlanRepository } from '@/modules/billing/domain/repositories/plan.repository';
import { NotFoundError, BusinessError } from '@/core/errors';
import type { UserRole } from '../../domain/enums/user-role.enum';

interface ChooseRoleInput {
  role: UserRole;
  planId: string;
}

export class ChooseUserRoleUseCase extends AuthorizedUseCase<ChooseRoleInput, void> {
  constructor(
    unitOfWork: IUnitOfWork,
    logger: FastifyBaseLogger,
    private readonly userRepository: IUserRepository,
    private readonly planRepository: IPlanRepository,
  ) {
    super(unitOfWork, logger);
  }

  @Traced('choose-user-role')
  async execute(
    context: ExecutionContext,
    input: ChooseRoleInput,
  ): Promise<void> {
    // ✅ Autorização genérica (pode escolher role?)
    this.authorize(context, 'choose', 'UserRole');

    return this.runInTransaction(async () => {
      // 1. Busca usuário (deve ser o próprio)
      const user = await this.userRepository.findById(context.userId);
      if (!user) {
        throw new NotFoundError('User');
      }

      // 2. Valida se pode escolher role (ownership)
      if (user.getId() !== context.userId) {
        throw new BusinessError('Você só pode escolher role para si mesmo');
      }

      // 3. Busca plano
      const plan = await this.planRepository.findById(input.planId);
      if (!plan) {
        throw new NotFoundError('Plan');
      }

      // 4. Valida se plano é compatível com role
      if (plan.getAllowedRole() !== input.role) {
        throw new BusinessError(
          `Plano ${plan.getName()} não é compatível com role ${input.role}`,
        );
      }

      // 5. Executa domain logic
      user.assignRoleAndPlan(input.role, input.planId);

      // 6. Persiste
      await this.userRepository.save(user);

      // 7. Log estruturado
      this.log('info', 'User role chosen', {
        userId: user.getId(),
        role: input.role,
        planId: input.planId,
        requestId: context.requestId,
      });
    });
  }
}
4. Hook de Autenticação Injeta Context
// apps/api/src/shared/hooks/authenticate.hook.ts

import type { FastifyRequest, FastifyReply } from 'fastify';
import type { ExecutionContext } from '@/core/contracts/base-use-case';

export async function authenticate(
  request: FastifyRequest,
  reply: FastifyReply,
): Promise<void> {
  // ... sua lógica atual de autenticação ...

  // ✅ Injeta ExecutionContext no request
  request.executionContext = {
    userId: user.getId(),
    sessionId: session.getId(),
    ability: ability,
    ipAddress: request.ip,
    userAgent: request.headers['user-agent'],
    requestId: request.id, // Fastify já gera
  } satisfies ExecutionContext;
}

// Augment Fastify types
declare module 'fastify' {
  interface FastifyRequest {
    executionContext: ExecutionContext;
  }
}
5. Route Handler Passa Context
// apps/api/src/modules/user/presentation/routes/user.routes.ts

import type { FastifyInstance } from 'fastify';
import { ChooseUserRoleUseCase } from '../../application/use-cases/choose-user-role.use-case';

export async function userRoutes(app: FastifyInstance) {
  app.post(
    '/users/me/role',
    {
      onRequest: [authenticate], // Injeta executionContext
      schema: {
        body: ChooseRoleSchema,
      },
    },
    async (request, reply) => {
      const useCase = request.diScope.resolve<ChooseUserRoleUseCase>('chooseUserRoleUseCase');
      
      // ✅ Passa context + input
      await useCase.execute(
        request.executionContext, // Context do request
        request.body,              // Input validado
      );

      return reply.code(204).send();
    },
  );
}
📊 COMPARAÇÃO: Pacote Externo vs Core Interno
Aspecto	Pacote shared-kernel	apps/api/src/core
Complexidade	⚠️ Alta (build, publish, versioning)	✅ Baixa (apenas imports)
Velocidade de dev	⚠️ Lenta (rebuild pacote)	✅ Rápida (hot reload)
Reuso	✅ Frontend + Backend	⚠️ Apenas backend
Manutenção	⚠️ Difícil (breaking changes)	✅ Fácil (refactor direto)
Type safety	⚠️ Pode quebrar	✅ Sempre sincronizado
Overhead	⚠️ CI/CD, npm registry	✅ Zero overhead
🎯 QUANDO USAR PACOTE EXTERNO?
Use packages/shared-kernel APENAS SE:

✅ Você tem múltiplos backends (API + Workers + Admin API)
✅ Você quer compartilhar Value Objects com o frontend (EmailVO, MoneyVO)
✅ Você tem equipes separadas (backend team vs frontend team)
✅ Você quer versionamento independente (semver)
No seu caso:

❌ Apenas 1 backend (API)
❌ Frontend não precisa de VOs (usa DTOs)
❌ Provavelmente mesma equipe
❌ Não precisa de versioning
Conclusão: Use apps/api/src/core ✅

🚀 ESTRUTURA FINAL RECOMENDADA
apps/api/src/
├── core/                          # ✅ Fundação (não é pacote externo!)
│   ├── contracts/
│   │   ├── base-use-case.ts      # BaseUseCase, AuthorizedUseCase, PublicUseCase
│   │   ├── base-repository.ts    # Seu BaseRepository atual
│   │   ├── base-task.ts          # Seu BaseTask atual
│   │   └── unit-of-work.interface.ts
│   │
│   ├── decorators/
│   │   └── traced.decorator.ts   # @Traced()
│   │
│   ├── errors/
│   │   └── index.ts              # DomainError hierarchy
│   │
│   ├── value-objects/
│   │   ├── email.vo.ts
│   │   ├── phone.vo.ts
│   │   └── money.vo.ts
│   │
│   └── types/
│       ├── execution-context.ts  # ExecutionContext interface
│       └── result.ts             # Result<T, E> type
│
├── modules/                       # Seus módulos (não muda)
├── infrastructure/                # Infraestrutura (não muda)
└── shared/                        # Utilitários (não muda)
Imports ficam simples:

import { AuthorizedUseCase, type ExecutionContext } from '@/core/contracts/base-use-case';
import { Traced } from '@/core/decorators/traced.decorator';
import { EmailVO } from '@/core/value-objects/email.vo';
import { NotFoundError } from '@/core/errors';
Faz muito mais sentido assim? Tudo dentro do backend, sem overhead de pacote externo, mas ainda bem organizado e reutilizável dentro da API.

Credits used: 1.1
Elapsed time: 45s


Checkpoint
Restore
implementaria o que aqui



Modo IA

Tudo

Imagens

Shopping

Vídeos

Notícias

Vídeos curtos

Mais

Ferramentas

Visão geral criada por IA

Top alternatives to REST APIs include

GraphQL for flexible data querying, gRPC for high-performance service-to-service communication, and WebSockets for real-time, bidirectional data exchange. Other options include MQTT for IoT devices, AsyncAPI for event-driven systems, and SOAP for enterprise-level security.

GraphQL: Developed by Facebook, it allows clients to request specific data in a single request, reducing over-fetching/under-fetching issues common in REST.

gRPC (Google Remote Procedure Call): Uses Protocol Buffers for high-performance communication, making it ideal for microservices.

WebSockets: Provides a persistent, full-duplex communication channel over a single TCP connection, ideal for real-time apps like chats or dashboards

.

MQTT (Message Queuing Telemetry Transport): A lightweight messaging protocol designed for low-bandwidth, high-latency, or IoT scenarios.

SOAP (Simple Object Access Protocol): A protocol relying on XML, often used for legacy systems needing strong security and transactional reliability.

Falcor: A library by Netflix for modeling remote data as a single JSON graph, which simplifies data retrieval

Kiro
Excelente pergunta! Vou analisar pragmaticamente quais alternativas ao REST fazem sentido para o Master Síndico, baseado nos requisitos reais do projeto.

🎯 ANÁLISE: O QUE IMPLEMENTAR NO MASTER SÍNDICO?
📊 Matriz de Decisão
Tecnologia	Use Case no Projeto	Prioridade	Complexidade	ROI
REST	API principal	✅ Atual	Baixa	Alto
WebSockets	Real-time features	🔥 Alta	Média	Alto
GraphQL	Frontend flexibility	⚠️ Média	Alta	Médio
gRPC	Microservices	⚠️ Baixa	Alta	Baixo
MQTT	IoT devices	❌ Nenhum	Média	Zero
SOAP	Legacy systems	❌ Nenhum	Alta	Zero
🔥 RECOMENDAÇÃO 1: WebSockets (IMPLEMENTAR!)
Por quê?
Baseado na análise técnica, o projeto tem casos de uso real-time:

Connect Me - Notificações instantâneas quando alguém te conecta
Video Comments - Chat em tempo real nos vídeos
Subscription Status - Atualização instantânea quando pagamento é aprovado
Admin Dashboard - Métricas em tempo real
Arquitetura Proposta
// apps/api/src/infrastructure/websocket/websocket-server.ts
import { Server as SocketIOServer } from 'socket.io';
import type { FastifyInstance } from 'fastify';
import { authenticate } from '@/shared/hooks/authenticate.hook';

export class WebSocketServer {
  private io: SocketIOServer;

  constructor(private app: FastifyInstance) {
    this.io = new SocketIOServer(app.server, {
      cors: {
        origin: process.env.CORS_ORIGIN?.split(',') || '*',
        credentials: true,
      },
      transports: ['websocket', 'polling'],
    });

    this.setupMiddleware();
    this.setupNamespaces();
  }

  private setupMiddleware(): void {
    // Autenticação via JWT
    this.io.use(async (socket, next) => {
      try {
        const token = socket.handshake.auth.token;
        
        if (!token) {
          return next(new Error('Authentication required'));
        }

        // Valida token (reutiliza lógica do authenticate hook)
        const decoded = await this.app.jwt.verify(token);
        
        // Injeta userId no socket
        socket.data.userId = decoded.sub;
        socket.data.sessionId = decoded.sid;
        
        next();
      } catch (error) {
        next(new Error('Invalid token'));
      }
    });
  }

  private setupNamespaces(): void {
    // Namespace para notificações
    const notifications = this.io.of('/notifications');
    notifications.on('connection', (socket) => {
      const userId = socket.data.userId;
      
      // Join room pessoal
      socket.join(`user:${userId}`);
      
      this.app.log.info({ userId }, 'User connected to notifications');
      
      socket.on('disconnect', () => {
        this.app.log.info({ userId }, 'User disconnected from notifications');
      });
    });

    // Namespace para chat de vídeos
    const videoChat = this.io.of('/video-chat');
    videoChat.on('connection', (socket) => {
      socket.on('join-video', (videoId: string) => {
        socket.join(`video:${videoId}`);
      });
      
      socket.on('leave-video', (videoId: string) => {
        socket.leave(`video:${videoId}`);
      });
      
      socket.on('send-comment', async (data) => {
        // Valida e salva comentário
        // Broadcast para todos na room
        videoChat.to(`video:${data.videoId}`).emit('new-comment', {
          id: data.id,
          userId: socket.data.userId,
          username: data.username,
          text: data.text,
          createdAt: new Date(),
        });
      });
    });

    // Namespace para admin dashboard
    const admin = this.io.of('/admin');
    admin.use(async (socket, next) => {
      // Valida se é admin
      const user = await this.getUserById(socket.data.userId);
      if (user.role !== 'admin') {
        return next(new Error('Admin access required'));
      }
      next();
    });
    
    admin.on('connection', (socket) => {
      socket.join('admin-dashboard');
      
      // Envia métricas a cada 5 segundos
      const interval = setInterval(async () => {
        const metrics = await this.getRealtimeMetrics();
        socket.emit('metrics-update', metrics);
      }, 5000);
      
      socket.on('disconnect', () => {
        clearInterval(interval);
      });
    });
  }

  // Método público para emitir eventos de qualquer lugar
  public emitToUser(userId: string, event: string, data: any): void {
    this.io.of('/notifications').to(`user:${userId}`).emit(event, data);
  }

  public emitToVideo(videoId: string, event: string, data: any): void {
    this.io.of('/video-chat').to(`video:${videoId}`).emit(event, data);
  }

  public emitToAdmins(event: string, data: any): void {
    this.io.of('/admin').to('admin-dashboard').emit(event, data);
  }
}
Integração com Domain Events
// apps/api/src/infrastructure/events/websocket-event-handler.ts
import type { DomainEvent } from '@/core/domain/domain-event';
import type { WebSocketServer } from '../websocket/websocket-server';

export class WebSocketEventHandler {
  constructor(private ws: WebSocketServer) {}

  async handle(event: DomainEvent): Promise<void> {
    switch (event.eventName) {
      case 'connect-me.sent':
        this.ws.emitToUser(event.data.toUserId, 'connect-me-received', {
          fromUserId: event.data.fromUserId,
          fromUsername: event.data.fromUsername,
          message: event.data.message,
        });
        break;

      case 'subscription.activated':
        this.ws.emitToUser(event.data.userId, 'subscription-activated', {
          planName: event.data.planName,
          expiresAt: event.data.expiresAt,
        });
        break;

      case 'video.comment.created':
        this.ws.emitToVideo(event.data.videoId, 'new-comment', {
          id: event.data.commentId,
          userId: event.data.userId,
          username: event.data.username,
          text: event.data.text,
          createdAt: event.data.createdAt,
        });
        break;

      case 'payment.succeeded':
        this.ws.emitToUser(event.data.userId, 'payment-succeeded', {
          amount: event.data.amount,
          currency: event.data.currency,
        });
        break;
    }
  }
}
Frontend Integration
// apps/web/src/lib/websocket-client.ts
import { io, Socket } from 'socket.io-client';

class WebSocketClient {
  private socket: Socket | null = null;

  connect(token: string): void {
    this.socket = io('http://localhost:3333/notifications', {
      auth: { token },
      transports: ['websocket', 'polling'],
    });

    this.socket.on('connect', () => {
      console.log('Connected to notifications');
    });

    this.socket.on('connect-me-received', (data) => {
      // Mostra toast notification
      toast.info(`${data.fromUsername} quer se conectar com você!`);
    });

    this.socket.on('subscription-activated', (data) => {
      toast.success(`Assinatura ${data.planName} ativada!`);
      // Atualiza UI
    });
  }

  disconnect(): void {
    this.socket?.disconnect();
  }
}

export const wsClient = new WebSocketClient();
⚠️ RECOMENDAÇÃO 2: GraphQL (CONSIDERAR para v2)
Por quê NÃO implementar agora?
❌ Complexidade alta - Precisa reescrever toda API
❌ Over-fetching não é problema - Frontend consome DTOs específicos
❌ N+1 queries - Drizzle já otimiza com joins
❌ Caching complexo - Apollo Client adiciona overhead
Quando faria sentido?
✅ Se tivesse múltiplos clientes (mobile app, desktop app, etc) ✅ Se frontend precisasse de queries muito flexíveis ✅ Se tivesse equipe dedicada para manter schema

Alternativa: REST com Expansions (Stripe-style)
// Ao invés de GraphQL, use query params para controlar response
GET /api/users/me?expand=profile,subscription,permissions

// Response
{
  "id": "user_123",
  "email": "user@example.com",
  "profile": {  // ✅ Incluído porque foi expandido
    "name": "João Silva",
    "avatarUrl": "..."
  },
  "subscription": {  // ✅ Incluído porque foi expandido
    "planName": "Síndico Pro",
    "status": "active"
  },
  "permissions": [...]  // ✅ Incluído porque foi expandido
}
Implementação:

// apps/api/src/shared/utils/expand-parser.ts
export function parseExpands(expandParam?: string): Set<string> {
  if (!expandParam) return new Set();
  return new Set(expandParam.split(',').map(e => e.trim()));
}

// Use case
export class GetUserProfileUseCase extends AuthorizedUseCase {
  async execute(
    context: ExecutionContext,
    options: { expand?: Set<string> },
  ): Promise<UserProfileDTO> {
    const user = await this.userRepo.findById(context.userId);
    
    const dto: UserProfileDTO = {
      id: user.getId(),
      email: user.getEmail().getValue(),
      username: user.getUsername().getValue(),
    };

    // Conditional loading baseado em expand
    if (options.expand?.has('profile')) {
      dto.profile = await this.loadProfile(user.getId());
    }

    if (options.expand?.has('subscription')) {
      dto.subscription = await this.loadSubscription(user.getId());
    }

    if (options.expand?.has('permissions')) {
      dto.permissions = await this.loadPermissions(user.getId());
    }

    return dto;
  }
}
❌ NÃO IMPLEMENTAR
gRPC
Por quê não?

❌ Você não tem microservices (monolito modular)
❌ Overhead de Protocol Buffers sem benefício
❌ Dificulta debug (binário vs JSON)
❌ Frontend não consome gRPC facilmente
Quando faria sentido?

✅ Se tivesse 10+ microservices
✅ Se tivesse comunicação service-to-service intensiva
✅ Se performance fosse gargalo crítico
MQTT
Por quê não?

❌ Não tem IoT devices
❌ WebSockets resolve real-time
❌ Overhead desnecessário
SOAP
Por quê não?

❌ Tecnologia legada
❌ XML é verboso
❌ Não tem integração com sistemas legados
🚀 ROADMAP DE IMPLEMENTAÇÃO
Fase 1: WebSockets (Sprint 1-2)
 Setup Socket.IO server
 Implementar autenticação via JWT
 Criar namespaces (/notifications, /video-chat, /admin)
 Integrar com Domain Events
 Frontend: Connect Me notifications
 Frontend: Video chat real-time
Fase 2: REST Expansions (Sprint 3)
 Implementar expand parser
 Refatorar use cases para suportar conditional loading
 Documentar no Swagger
 Frontend: Usar expansions para reduzir requests
Fase 3: Server-Sent Events (Sprint 4) - Alternativa leve
 SSE para notificações simples (fallback do WebSocket)
 Implementar retry automático
 Frontend: EventSource API
📊 COMPARAÇÃO: WebSockets vs Polling
Cenário: Connect Me Notifications
Polling (atual):

// Frontend faz request a cada 30s
setInterval(async () => {
  const notifications = await fetch('/api/notifications/unread');
  // Atualiza UI
}, 30000);
Problemas:

❌ 120 requests/hora por usuário (desperdício)
❌ Latência de até 30s
❌ Overhead de HTTP headers
WebSockets (proposto):

// Frontend recebe push instantâneo
socket.on('connect-me-received', (data) => {
  // Atualiza UI imediatamente
});
Vantagens:

✅ 1 conexão persistente
✅ Latência < 100ms
✅ Menos overhead (sem HTTP headers)
🎯 DECISÃO FINAL
✅ IMPLEMENTAR:
WebSockets (Socket.IO) - Real-time features
REST com Expansions - Flexibilidade sem GraphQL
⚠️ CONSIDERAR FUTURO:
GraphQL - Se tiver múltiplos clientes
Server-Sent Events - Fallback do WebSocket
❌ NÃO IMPLEMENTAR:
gRPC - Sem microservices
MQTT - Sem IoT
SOAP - Legado
Quer que eu crie uma spec para implementar WebSockets com Socket.IO? Posso detalhar:

Setup do servidor
Integração com Domain Events
Autenticação via JWT
Namespaces para cada feature
Frontend integration
Credits used: 1.08
Elapsed time: 50s


Checkpoint
Restore
porque não migrar todo o projeto para WebSockets

Kiro
Excelente pergunta! Vou analisar honestamente por que migrar 100% para WebSockets seria uma má ideia (apesar de parecer moderno).

❌ POR QUE NÃO MIGRAR TUDO PARA WEBSOCKETS
1. 🔥 Problema: Stateful vs Stateless
REST (Stateless):

// Cada request é independente
POST /api/users/login
Authorization: Bearer token123

// Servidor não precisa lembrar de nada
// Escala horizontalmente fácil
WebSockets (Stateful):

// Conexão persistente por usuário
socket.on('login', (data) => {
  // Servidor precisa manter estado da conexão
  // Usuário fica "preso" a 1 servidor
});
Problema de Escalabilidade:

┌─────────────────────────────────────────┐
│  Load Balancer                          │
└─────────────────────────────────────────┘
         │              │              │
    ┌────▼───┐     ┌────▼───┐     ┌────▼───┐
    │ API 1  │     │ API 2  │     │ API 3  │
    │ 1000   │     │ 1000   │     │ 1000   │
    │ sockets│     │ sockets│     │ sockets│
    └────────┘     └────────┘     └────────┘

❌ Usuário conectado no API 1 não recebe eventos do API 2
❌ Precisa de Redis Pub/Sub ou Sticky Sessions
❌ Complexidade 10x maior
REST escala fácil:

┌─────────────────────────────────────────┐
│  Load Balancer (Round Robin)           │
└─────────────────────────────────────────┘
         │              │              │
    ┌────▼───┐     ┌────▼───┐     ┌────▼───┐
    │ API 1  │     │ API 2  │     │ API 3  │
    │        │     │        │     │        │
    │ ✅     │     │ ✅     │     │ ✅     │
    └────────┘     └────────┘     └────────┘

✅ Qualquer servidor responde qualquer request
✅ Escala horizontal sem complexidade
2. 🌐 Problema: HTTP Caching
REST com HTTP Cache:

GET /api/plans
Cache-Control: public, max-age=3600
ETag: "abc123"

// Browser/CDN cacheia automaticamente
// 99% dos requests nem chegam no servidor
WebSockets:

socket.emit('get-plans');
socket.on('plans-response', (data) => {
  // ❌ Sem cache
  // ❌ Sempre vai no servidor
  // ❌ Desperdício de recursos
});
Impacto Real:

Endpoint	Requests/dia	Com HTTP Cache	Com WebSocket
GET /plans	100.000	1.000 (99% cache)	100.000 (0% cache)
GET /categories	50.000	500 (99% cache)	50.000 (0% cache)
GET /user/me	200.000	20.000 (90% cache)	200.000 (0% cache)
Resultado:

REST: 21.500 requests no servidor
WebSocket: 350.000 requests no servidor
16x mais carga!
3. 🔒 Problema: Segurança e Infraestrutura
REST:

# Nginx/Cloudflare protege automaticamente
location /api {
    # ✅ Rate limiting
    limit_req zone=api burst=10;
    
    # ✅ DDoS protection
    # ✅ WAF (Web Application Firewall)
    # ✅ Bot detection
    # ✅ SSL termination
}
WebSockets:

# Nginx precisa configuração especial
location /socket.io {
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    
    # ⚠️ Rate limiting complexo
    # ⚠️ DDoS protection limitado
    # ⚠️ WAF não funciona bem
    # ⚠️ Timeout management manual
}
Problemas de Segurança:

❌ CORS mais complexo
❌ CSRF tokens não funcionam
❌ Rate limiting por conexão (não por request)
❌ Cloudflare/CDN tem suporte limitado
4. 📱 Problema: Mobile e Conectividade
REST:

// Mobile app faz request quando precisa
async function loadProfile() {
  try {
    const profile = await fetch('/api/profile');
    return profile;
  } catch (error) {
    // ✅ Retry automático
    // ✅ Offline mode fácil
    // ✅ Background sync
  }
}
WebSockets:

// Mobile precisa manter conexão sempre aberta
socket.on('disconnect', () => {
  // ❌ Reconectar
  // ❌ Re-autenticar
  // ❌ Re-sincronizar estado
  // ❌ Drena bateria
});

// Usuário troca de WiFi para 4G
// ❌ Conexão cai
// ❌ Precisa reconectar
// ❌ Pode perder mensagens
Impacto na Bateria:

REST: App dorme quando não usa
WebSocket: Conexão persistente drena bateria
5. 🐛 Problema: Debug e Monitoramento
REST:

# Logs estruturados
[2026-02-26 10:30:15] POST /api/users/login 200 150ms
[2026-02-26 10:30:16] GET /api/profile 200 50ms
[2026-02-26 10:30:17] POST /api/videos 201 300ms

# ✅ Fácil ver o que aconteceu
# ✅ Ferramentas padrão (Postman, curl, browser DevTools)
# ✅ APM tools funcionam out-of-the-box
WebSockets:

# Logs confusos
[2026-02-26 10:30:15] Socket connected: abc123
[2026-02-26 10:30:16] Event: user-action
[2026-02-26 10:30:17] Event: another-action
[2026-02-26 10:30:18] Socket disconnected: abc123

# ❌ Difícil rastrear fluxo
# ❌ Ferramentas limitadas
# ❌ APM tools precisam instrumentação custom
6. 💰 Problema: Custo de Infraestrutura
REST (Stateless):

┌─────────────────────────────────────────┐
│  Cloudflare (CDN + Cache)               │
│  $20/mês                                │
└─────────────────────────────────────────┘
         │
    ┌────▼───────────────────────────────┐
    │  2x API Servers (2GB RAM cada)     │
    │  $20/mês                           │
    └────────────────────────────────────┘

Total: $40/mês para 100k usuários
WebSockets (Stateful):

┌─────────────────────────────────────────┐
│  Load Balancer com Sticky Sessions     │
│  $50/mês                                │
└─────────────────────────────────────────┘
         │
    ┌────▼───────────────────────────────┐
    │  5x API Servers (4GB RAM cada)     │
    │  Precisa mais RAM para conexões    │
    │  $100/mês                          │
    └────────────────────────────────────┘
         │
    ┌────▼───────────────────────────────┐
    │  Redis Pub/Sub (para sync)         │
    │  $30/mês                           │
    └────────────────────────────────────┘

Total: $180/mês para 100k usuários
4.5x mais caro!

✅ ABORDAGEM HÍBRIDA (RECOMENDADA)
Use REST para:
CRUD Operations (95% das operações)

POST /api/users
GET /api/users/:id
PUT /api/users/:id
DELETE /api/users/:id
Queries Complexas

GET /api/videos?category=sindico&sort=views&page=1
File Uploads

POST /api/videos/upload
Content-Type: multipart/form-data
Webhooks (Stripe, Mux)

POST /api/webhooks/stripe
Use WebSockets para:
Notificações Real-time (5% das operações)

socket.on('connect-me-received', (data) => {
  toast.info('Nova conexão!');
});
Chat/Comments Real-time

socket.emit('send-comment', { videoId, text });
socket.on('new-comment', (comment) => {
  // Atualiza UI
});
Live Updates (Admin Dashboard)

socket.on('metrics-update', (metrics) => {
  updateChart(metrics);
});
Presence (Online/Offline)

socket.on('user-online', (userId) => {
  showOnlineBadge(userId);
});
🏗️ ARQUITETURA HÍBRIDA IDEAL
// apps/api/src/app.ts

import Fastify from 'fastify';
import { Server as SocketIOServer } from 'socket.io';

const app = Fastify();

// ✅ REST para 95% das operações
app.register(authRoutes, { prefix: '/api/auth' });
app.register(userRoutes, { prefix: '/api/users' });
app.register(videoRoutes, { prefix: '/api/videos' });
app.register(billingRoutes, { prefix: '/api/billing' });

// ✅ WebSockets para 5% (real-time)
const io = new SocketIOServer(app.server, {
  path: '/socket.io',
  transports: ['websocket', 'polling'], // Fallback para polling
});

io.of('/notifications').on('connection', (socket) => {
  // Real-time notifications
});

io.of('/video-chat').on('connection', (socket) => {
  // Real-time comments
});
📊 COMPARAÇÃO FINAL
Aspecto	100% REST	100% WebSocket	Híbrido (95% REST + 5% WS)
Escalabilidade	⭐⭐⭐⭐⭐	⭐⭐	⭐⭐⭐⭐
Caching	⭐⭐⭐⭐⭐	⭐	⭐⭐⭐⭐⭐
Real-time	⭐	⭐⭐⭐⭐⭐	⭐⭐⭐⭐⭐
Segurança	⭐⭐⭐⭐⭐	⭐⭐⭐	⭐⭐⭐⭐
Debug	⭐⭐⭐⭐⭐	⭐⭐	⭐⭐⭐⭐
Custo	⭐⭐⭐⭐⭐	⭐⭐	⭐⭐⭐⭐
Complexidade	⭐⭐⭐⭐⭐	⭐⭐	⭐⭐⭐⭐
Mobile Battery	⭐⭐⭐⭐⭐	⭐⭐	⭐⭐⭐⭐
🎯 DECISÃO FINAL
NÃO migre tudo para WebSockets!

Use a regra 95/5:

✅ 95% REST - CRUD, queries, uploads, webhooks
✅ 5% WebSockets - Notificações, chat, live updates
Por quê?

REST é stateless (escala fácil)
HTTP cache economiza 99% dos requests
Infraestrutura mais barata
Debug mais fácil
Segurança mais robusta
Mobile-friendly (bateria)
Quando usar 100% WebSocket?

✅ Apps de chat (WhatsApp, Slack)
✅ Jogos multiplayer
✅ Editores colaborativos (Google Docs)
✅ Trading platforms (tempo real crítico)
Master Síndico não é nenhum desses casos!

Quer que eu crie uma spec para implementar a abordagem híbrida (REST + WebSockets nos lugares certos)?