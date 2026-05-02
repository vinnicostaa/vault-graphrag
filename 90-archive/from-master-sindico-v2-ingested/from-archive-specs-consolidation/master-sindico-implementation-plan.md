# Plano de Implementação - Master Síndico

## Visão Geral

Este documento descreve o plano de implementação dos módulos faltantes do Master Síndico, seguindo estritamente as diretrizes arquiteturais definidas no [`CONTEXT_BRAIN.md`](../contextos/CONTEXT_BRAIN.md).

## Princípios Arquiteturais

### 1. Princípio da Separação (CRÍTICO)

- **User (Identity)**: Credenciais, login, role imutável (ex: syndic)
- **Profile (Dados)**: Dados cadastrais. Define se o perfil está completo (incomplete vs active)
- **Subscription (Acesso)**: Define o plano (syndic_n2, resident_base) e status financeiro
- **PermissionService (Lógica)**: Cruza o Plano da Subscription com a Matriz Funcional para autorizar ações

### 2. Regras de Ouro

- Existe apenas UMA classe base [`BaseProfile`](../apps/api/src/modules/profile/domain/entities/base-profile.entity.ts). Não existem entidades separadas para "Pagos"
- Use [`PermissionService`](../apps/api/src/modules/subscription/domain/services/permission.service.ts) para checar acesso
- Soft Delete obrigatório em todas as entidades (deletedAt)
- Use Cases transactionais via `runInTransaction`
- Drizzle ORM com migrations separadas por entidade

## Estado Atual

### ✅ Implementado

- **Módulo Auth**: Login, Sessions, Verifications
- **Módulo User**: Entidade User agnóstica
- **Módulo Profile**: Entidades de domínio (BaseProfile, ResidentProfile, SyndicProfile, EnterpriseProfile, LocalCompanyProfile, MarketingProfile)
- **Schemas do Banco**: users, accounts, sessions, verifications, resident_profile, syndic_profile, enterprise_profile, local_company_profile, marketing_profile, videos
- **Repositórios**: Todos os repositórios de perfil implementados

### ❌ Não Implementado

- **Módulo Subscription**: Planos, Subscriptions, PermissionService
- **Use Cases de Profile**: CRUD para todos os perfis
- **Rotas HTTP de Profile**: Endpoints REST
- **Módulo Content**: Vídeos (entidades, use cases, rotas)
- **Módulo Assembly**: Assembleias, Votação, Atas
- **Módulo Engagement**: Fórum, Cupons, Connect Me
- **Módulo LMS**: Módulos, Cursos, Certificados
- **Webhooks Stripe**: Integração com pagamentos
- **Sistema de Reconhecimento**: Status do síndico (Bronze/Prata/Ouro/Diamante)

## Fases de Implementação

### Fase 1: Módulo de Subscription (Planos e Pagamentos)

#### 1.1 Schema do Banco

```typescript
// apps/api/src/infrastructure/database/drizzle/schema/subscriptions/subscriptions.ts
export const subscriptions = pgTable("subscriptions", {
  id: uuid("id").primaryKey().$defaultFn(() => uuidv7()),
  userId: uuid("user_id").notNull().references(() => users.id, { onDelete: "cascade" }),
  plan: subscriptionPlan("plan").notNull(),
  status: subscriptionStatus("status").default("trial").notNull(),
  stripeCustomerId: text("stripe_customer_id"),
  stripeSubscriptionId: text("stripe_subscription_id"),
  currentPeriodStart: timestamp("current_period_start"),
  currentPeriodEnd: timestamp("current_period_end"),
  cancelAtPeriodEnd: boolean("cancel_at_period_end").default(false),
  createdAt: timestamp("created_at").$defaultFn(() => new Date()).notNull(),
  updatedAt: timestamp("updated_at").$defaultFn(() => new Date()).$onUpdate(() => new Date()).notNull(),
  deletedAt: timestamp("deleted_at"),
});
```

#### 1.2 Entidades de Domínio

```typescript
// apps/api/src/modules/subscription/domain/entities/subscription.entity.ts
export class Subscription {
  constructor(
    private readonly id: string,
    private readonly userId: string,
    private plan: SubscriptionPlan,
    private status: SubscriptionStatus,
    private readonly createdAt: Date,
    private updatedAt: Date,
    private deletedAt: Date | null = null,
  ) {}

  isActive(): boolean {
    return this.status === "active";
  }

  cancel(): void {
    if (this.status === "canceled") {
      throw new BusinessError("Subscription already canceled", "SUBSCRIPTION_ALREADY_CANCELED");
    }
    this.status = "canceled";
    this.updatedAt = new Date();
  }
}
```

#### 1.3 PermissionService (ABAC)

```typescript
// apps/api/src/modules/subscription/domain/services/permission.service.ts
export class PermissionService {
  constructor(
    private subscriptionRepository: ISubscriptionRepository,
    private functionalMatrix: FunctionalMatrix,
  ) {}

  async can(userId: string, feature: Feature, action: Action): Promise<boolean> {
    const subscription = await this.subscriptionRepository.findByUserId(userId);
    if (!subscription || !subscription.isActive()) {
      return false;
    }

    return this.functionalMatrix.hasPermission(subscription.getPlan(), feature, action);
  }
}
```

#### 1.4 Use Cases

```typescript
// apps/api/src/modules/subscription/application/use-cases/create-subscription.use-case.ts
export class CreateSubscriptionUseCase extends TransactionalUseCase<CreateSubscriptionInput, SubscriptionDto> {
  constructor(
    deps: UseCaseDependencies,
    private subscriptionRepository: ISubscriptionRepository,
    private userRepository: IUserRepository,
  ) {
    super(deps);
  }

  async execute(input: CreateSubscriptionInput): Promise<SubscriptionDto> {
    return this.runInTransaction(async () => {
      const user = await this.userRepository.findById(input.userId);
      if (!user) throw new NotFoundError("User not found");

      const subscription = Subscription.create({
        id: generateId(),
        userId: input.userId,
        plan: input.plan,
        status: "trial",
        createdAt: new Date(),
        updatedAt: new Date(),
      });

      await this.subscriptionRepository.save(subscription);
      return subscription.toDto();
    });
  }
}
```

#### 1.5 Rotas HTTP

```
POST   /v1/subscriptions           - Criar subscription
GET    /v1/subscriptions/:id       - Obter subscription
PUT    /v1/subscriptions/:id       - Atualizar subscription
DELETE /v1/subscriptions/:id       - Cancelar subscription
GET    /v1/subscriptions/user/:userId - Obter subscription por usuário
```

---

### Fase 2: Módulo de Profile (Use Cases e Rotas)

#### 2.1 Use Cases para ResidentProfile

```typescript
// apps/api/src/modules/profile/application/use-cases/create-resident-profile.use-case.ts
export class CreateResidentProfileUseCase extends TransactionalUseCase<CreateResidentProfileInput, ResidentProfileDto> {
  constructor(
    deps: UseCaseDependencies,
    private residentProfileRepository: IResidentProfileRepository,
    private userRepository: IUserRepository,
  ) {
    super(deps);
  }

  async execute(input: CreateResidentProfileInput): Promise<ResidentProfileDto> {
    return this.runInTransaction(async () => {
      // Validações de negócio
      const user = await this.userRepository.findById(input.userId);
      if (!user) throw new NotFoundError("User not found");

      // Criar perfil
      const profile = ResidentProfile.create({
        id: generateId(),
        userId: input.userId,
        birthDate: input.birthDate,
        cpf: input.cpf,
        address: input.address,
        buildingNameOrCode: input.buildingNameOrCode,
        status: "incomplete",
        createdAt: new Date(),
        updatedAt: new Date(),
      });

      await this.residentProfileRepository.save(profile);
      return profile.toDto();
    });
  }
}
```

#### 2.2 Validações Zod

```typescript
// apps/api/src/modules/profile/application/validators/create-resident-profile.validator.ts
export const createResidentProfileSchema = z.object({
  userId: z.string().uuid(),
  birthDate: z.coerce.date(),
  cpf: z.string().length(11),
  address: z.object({
    street: z.string().min(1),
    number: z.string().min(1),
    block: z.string().nullable(),
    district: z.string().min(1),
    city: z.string().min(1),
    state: z.string().length(2),
    zipCode: z.string().length(8),
  }),
  buildingNameOrCode: z.string().nullable(),
});
```

#### 2.3 Rotas HTTP

```
# Resident Profile
POST   /v1/profiles/resident          - Criar perfil
GET    /v1/profiles/resident/:id      - Obter perfil
PUT    /v1/profiles/resident/:id      - Atualizar perfil
DELETE /v1/profiles/resident/:id      - Deletar perfil (soft delete)

# Syndic Profile
POST   /v1/profiles/syndic            - Criar perfil
GET    /v1/profiles/syndic/:id        - Obter perfil
PUT    /v1/profiles/syndic/:id        - Atualizar perfil
DELETE /v1/profiles/syndic/:id        - Deletar perfil (soft delete)

# Enterprise Profile
POST   /v1/profiles/enterprise        - Criar perfil
GET    /v1/profiles/enterprise/:id    - Obter perfil
PUT    /v1/profiles/enterprise/:id    - Atualizar perfil
DELETE /v1/profiles/enterprise/:id    - Deletar perfil (soft delete)

# Local Company Profile
POST   /v1/profiles/local-company     - Criar perfil
GET    /v1/profiles/local-company/:id - Obter perfil
PUT    /v1/profiles/local-company/:id - Atualizar perfil
DELETE /v1/profiles/local-company/:id - Deletar perfil (soft delete)

# Marketing Profile
POST   /v1/profiles/marketing        - Criar perfil
GET    /v1/profiles/marketing/:id    - Obter perfil
PUT    /v1/profiles/marketing/:id    - Atualizar perfil
DELETE /v1/profiles/marketing/:id    - Deletar perfil (soft delete)
```

---

### Fase 3: Módulo de Content (Vídeos)

#### 3.1 Entidades de Domínio

```typescript
// apps/api/src/modules/content/domain/entities/video.entity.ts
export class Video {
  constructor(
    private readonly id: string,
    private readonly authorUserId: string,
    private title: string,
    private description: string | null,
    private type: VideoType,
    private status: VideoStatus,
    private provider: string,
    private providerVideoId: string | null,
    private streamUrl: string | null,
    private thumbnailUrl: string | null,
    private durationSeconds: number | null,
    private canBeReplacedAfter: Date | null,
    private readonly createdAt: Date,
    private updatedAt: Date,
    private deletedAt: Date | null = null,
  ) {}

  approve(moderatorId: string): void {
    this.status = "ready";
    this.moderatedAt = new Date();
    this.moderatedByUserId = moderatorId;
    this.updatedAt = new Date();
  }

  reject(reason: string, moderatorId: string): void {
    this.status = "failed";
    this.moderationNote = reason;
    this.moderatedAt = new Date();
    this.moderatedByUserId = moderatorId;
    this.updatedAt = new Date();
  }
}
```

#### 3.2 Trava Trimestral (90 dias)

```typescript
// apps/api/src/modules/content/domain/services/video-upload.service.ts
export class VideoUploadService {
  constructor(private videoRepository: IVideoRepository) {}

  async canUploadInstitutionalVideo(userId: string): Promise<boolean> {
    const lastUpload = await this.videoRepository.findLastInstitutionalByUserId(userId);
    if (!lastUpload) return true;

    const daysSinceLastUpload = dayjs().diff(lastUpload.getCreatedAt(), "days");
    return daysSinceLastUpload >= 90;
  }
}
```

#### 3.3 Use Cases

```typescript
// apps/api/src/modules/content/application/use-cases/moderate-video.use-case.ts
export class ModerateVideoUseCase extends TransactionalUseCase<ModerateVideoInput, VideoDto> {
  constructor(
    deps: UseCaseDependencies,
    private videoRepository: IVideoRepository,
  ) {
    super(deps);
  }

  async execute(input: ModerateVideoInput): Promise<VideoDto> {
    return this.runInTransaction(async () => {
      const video = await this.videoRepository.findById(input.videoId);
      if (!video) throw new NotFoundError("Video not found");

      // Validações de conteúdo
      this.validateContent(video);

      // Atualizar status
      if (input.approved) {
        video.approve(input.moderatorId);
      } else {
        video.reject(input.rejectionReason, input.moderatorId);
      }

      await this.videoRepository.save(video);
      return video.toDto();
    });
  }

  private validateContent(video: Video): void {
    // Verificar se contém informações de contato
    // Verificar se tem chamadas diretas para venda
    // Verificar se tem ofertas promocionais
  }
}
```

#### 3.4 Rotas HTTP

```
# Vídeos
POST   /v1/videos/upload            - Upload de vídeo
GET    /v1/videos/:id               - Obter vídeo
GET    /v1/videos/feed              - Feed de vídeos
POST   /v1/videos/:id/like          - Curtir vídeo
POST   /v1/videos/:id/comment       - Comentar vídeo
POST   /v1/videos/:id/moderate      - Moderar vídeo (admin)
```

---

### Fase 4: Módulo de Assembly (Assembleias e Votação)

#### 4.1 Schema do Banco

```typescript
// apps/api/src/infrastructure/database/drizzle/schema/assembly/assemblies.ts
export const assemblies = pgTable("assemblies", {
  id: uuid("id").primaryKey().$defaultFn(() => uuidv7()),
  userId: uuid("user_id").notNull().references(() => users.id, { onDelete: "cascade" }),
  title: text("title").notNull(),
  description: text("description"),
  accessToken: text("access_token").notNull().unique(),
  startsAt: timestamp("starts_at").notNull(),
  endsAt: timestamp("ends_at").notNull(),
  createdAt: timestamp("created_at").$defaultFn(() => new Date()).notNull(),
  updatedAt: timestamp("updated_at").$defaultFn(() => new Date()).$onUpdate(() => new Date()).notNull(),
  deletedAt: timestamp("deleted_at"),
});
```

#### 4.2 Tokens Legíveis

```typescript
// apps/api/src/modules/assembly/domain/services/token-generator.service.ts
export class TokenGeneratorService {
  generate(): string {
    // Gera token legível: bjr-fgvo-yhd
    const adjectives = ['bjr', 'fgvo', 'yhd', 'kzm', 'qwx'];
    const nouns = ['token', 'vote', 'access', 'key'];
    
    return `${adjectives[Math.floor(Math.random() * adjectives.length)]}-${nouns[Math.floor(Math.random() * nouns.length)]}-${Math.random().toString(36).substring(2, 5)}`;
  }
}
```

#### 4.3 Use Cases

```typescript
// apps/api/src/modules/assembly/application/use-cases/vote.use-case.ts
export class VoteUseCase extends TransactionalUseCase<VoteInput, VoteDto> {
  constructor(
    deps: UseCaseDependencies,
    private assemblyRepository: IAssemblyRepository,
    private voteRepository: IVoteRepository,
  ) {
    super(deps);
  }

  async execute(input: VoteInput): Promise<VoteDto> {
    return this.runInTransaction(async () => {
      const assembly = await this.assemblyRepository.findById(input.assemblyId);
      if (!assembly) throw new NotFoundError("Assembly not found");

      if (!assembly.isVotingOpen()) {
        throw new BusinessError("Voting is closed", "VOTING_CLOSED");
      }

      const existingVote = await this.voteRepository.findByAssemblyAndUser(input.assemblyId, input.userId);
      if (existingVote) {
        throw new BusinessError("User already voted", "ALREADY_VOTED");
      }

      const vote = Vote.create({
        id: generateId(),
        assemblyId: input.assemblyId,
        userId: input.userId,
        option: input.option,
        votedAt: new Date(),
      });

      await this.voteRepository.save(vote);
      return vote.toDto();
    });
  }
}
```

#### 4.4 Rotas HTTP

```
# Assembleias
POST   /v1/assemblies              - Criar assembleia
GET    /v1/assemblies/:id          - Obter assembleia
POST   /v1/assemblies/:id/agendas   - Adicionar pauta
POST   /v1/assemblies/:id/vote      - Votar
GET    /v1/assemblies/:id/results   - Resultados da votação
```

---

### Fase 5: Módulo de Engagement (Fórum, Cupons, Connect Me)

#### 5.1 Engine de Cupons

```typescript
// apps/api/src/modules/engagement/application/use-cases/generate-coupon.use-case.ts
export class GenerateCouponUseCase extends TransactionalUseCase<GenerateCouponInput, CouponDto> {
  constructor(
    deps: UseCaseDependencies,
    private couponRepository: ICouponRepository,
  ) {
    super(deps);
  }

  async execute(input: GenerateCouponInput): Promise<CouponDto> {
    return this.runInTransaction(async () => {
      // Validar trava de 4 horas
      const lastCoupon = await this.couponRepository.findLastByUserAndStore(input.userId, input.storeId);
      if (lastCoupon) {
        const hoursSinceLast = dayjs().diff(lastCoupon.getCreatedAt(), "hours");
        if (hoursSinceLast < 4) {
          throw new BusinessError(`Aguarde ${4 - hoursSinceLast} horas`, "COUPON_COOLDOWN");
        }
      }

      // Gerar código do cupom
      const couponCode = this.generateCouponCode(input.storeId);

      const coupon = Coupon.create({
        id: generateId(),
        userId: input.userId,
        storeId: input.storeId,
        code: couponCode,
        createdAt: new Date(),
      });

      await this.couponRepository.save(coupon);
      return coupon.toDto();
    });
  }

  private generateCouponCode(storeId: string): string {
    // Formato: ID_LOJA(5) + SEQUENCIAL(3) + PALAVRA(5)
    // Exemplo: 00123055PAO
    const storeIdPadded = storeId.padStart(5, '0');
    const sequential = Math.floor(Math.random() * 999).toString().padStart(3, '0');
    const word = this.getRandomWord();

    return `${storeIdPadded}${sequential}${word}`;
  }
}
```

#### 5.2 Connect Me (Cotas Anuais)

```typescript
// apps/api/src/modules/engagement/domain/services/connect-me.service.ts
export class ConnectMeService {
  constructor(
    private subscriptionRepository: ISubscriptionRepository,
    private connectMeRepository: IConnectMeRepository,
  ) {}

  async canSendConnectMe(userId: string, targetUserId: string): Promise<boolean> {
    const subscription = await this.subscriptionRepository.findByUserId(userId);
    if (!subscription || !subscription.isActive()) {
      return false;
    }

    const annualQuota = this.getAnnualQuota(subscription.getPlan());
    const usedQuota = await this.connectMeRepository.countByUserAndYear(userId, new Date().getFullYear());

    return usedQuota < annualQuota;
  }

  private getAnnualQuota(plan: SubscriptionPlan): number {
    const quotas: Record<SubscriptionPlan, number> = {
      resident_base: 2,
      resident_paid: 4,
      syndic_n1: 0,
      syndic_n2: 2,
      syndic_n3: 4,
      enterprise_plus: 10,
      enterprise_pro: 20,
      marketing_standard: 0,
      local_company_standard: 0,
    };

    return quotas[plan] || 0;
  }
}
```

#### 5.3 Rotas HTTP

```
# Fórum
POST   /v1/forum/topics              - Criar tópico
GET    /v1/forum/topics              - Listar tópicos
GET    /v1/forum/topics/:id          - Obter tópico
POST   /v1/forum/topics/:id/reply    - Responder tópico
POST   /v1/forum/topics/:id/like     - Curtir tópico

# Cupons
POST   /v1/coupons/generate          - Gerar cupom
GET    /v1/coupons/my                - Listar meus cupons

# Connect Me
POST   /v1/connect-me/send           - Enviar formulário de contato
GET    /v1/connect-me/received       - Listar contatos recebidos
GET    /v1/connect-me/sent           - Listar contatos enviados
```

---

### Fase 6: Módulo de LMS (Módulos e Cursos)

#### 6.1 Certificados

```typescript
// apps/api/src/modules/lms/application/use-cases/generate-certificate.use-case.ts
export class GenerateCertificateUseCase extends TransactionalUseCase<GenerateCertificateInput, CertificateDto> {
  constructor(
    deps: UseCaseDependencies,
    private courseRepository: ICourseRepository,
    private userRepository: IUserRepository,
    private certificateRepository: ICertificateRepository,
  ) {
    super(deps);
  }

  async execute(input: GenerateCertificateInput): Promise<CertificateDto> {
    return this.runInTransaction(async () => {
      const course = await this.courseRepository.findById(input.courseId);
      if (!course) throw new NotFoundError("Curso não encontrado");

      const user = await this.userRepository.findById(input.userId);
      if (!user) throw new NotFoundError("Usuário não encontrado");

      // Validar se usuário completou 100% do curso
      const progress = await this.progressRepository.findByUserAndCourse(input.userId, input.courseId);
      if (!progress || progress.getPercentage() < 100) {
        throw new BusinessError("Curso não foi concluído", "COURSE_NOT_COMPLETED");
      }

      // Gerar certificado
      const certificate = Certificate.create({
        id: generateId(),
        userId: input.userId,
        courseId: input.courseId,
        qrCode: this.generateQRCode(input.userId, input.courseId),
        issuedAt: new Date(),
      });

      await this.certificateRepository.save(certificate);
      return certificate.toDto();
    });
  }

  private generateQRCode(userId: string, courseId: string): string {
    // Gera QR code para validação externa
    return `https://mastersindico.com/certificates/validate?u=${userId}&c=${courseId}`;
  }
}
```

#### 6.2 Rotas HTTP

```
# Cursos
GET    /v1/courses                  - Listar cursos
GET    /v1/courses/:id              - Obter curso
POST   /v1/courses/:id/modules      - Adicionar módulo
POST   /v1/courses/:id/modules/:moduleId/lessons - Adicionar aula
POST   /v1/courses/:id/complete     - Completar aula
GET    /v1/courses/:id/certificate  - Obter certificado
```

---

### Fase 7: Integração com Webhooks (Stripe)

#### 7.1 Handler de Webhooks

```typescript
// apps/api/src/modules/subscription/infrastructure/webhooks/stripe-webhook.handler.ts
export class StripeWebhookHandler {
  constructor(
    private subscriptionRepository: ISubscriptionRepository,
    private logger: FastifyBaseLogger,
  ) {}

  async handle(event: Stripe.Event): Promise<void> {
    switch (event.type) {
      case 'customer.subscription.created':
        await this.handleSubscriptionCreated(event);
        break;
      case 'customer.subscription.updated':
        await this.handleSubscriptionUpdated(event);
        break;
      case 'customer.subscription.deleted':
        await this.handleSubscriptionDeleted(event);
        break;
      case 'invoice.payment_succeeded':
        await this.handlePaymentSucceeded(event);
        break;
      case 'invoice.payment_failed':
        await this.handlePaymentFailed(event);
        break;
      default:
        this.logger.warn(`Unhandled event type: ${event.type}`);
    }
  }

  private async handlePaymentSucceeded(event: Stripe.Event): Promise<void> {
    const invoice = event.data.object as Stripe.Invoice;
    // Atualizar status da subscription para active
    // Ativar perfil do usuário se estiver pending_payment
  }
}
```

#### 7.2 Rotas HTTP

```
POST   /v1/webhooks/stripe          - Webhook do Stripe
```

---

### Fase 8: Sistema de Reconhecimento do Síndico

#### 8.1 Cálculo de Status

```typescript
// apps/api/src/modules/profile/domain/services/syndic-status-calculator.service.ts
export class SyndicStatusCalculatorService {
  constructor(
    private syndicProfileRepository: ISyndicProfileRepository,
    private videoRepository: IVideoRepository,
    private courseRepository: ICourseRepository,
  ) {}

  async calculateStatus(userId: string): Promise<SyndicStatus> {
    const profile = await this.syndicProfileRepository.findByUserId(userId);
    if (!profile) throw new NotFoundError("Syndic profile not found");

    // 1. Consumo de conteúdo qualificado (30%)
    const qualifiedVideosWatched = await this.videoRepository.countQualifiedWatchedByUser(userId);
    const contentScore = Math.min(qualifiedVideosWatched * 3, 30);

    // 2. Atuação prática (40%)
    const buildingsCount = profile.getBuildingsCount();
    const practiceScore = Math.min(buildingsCount * 10, 40);

    // 3. Capacitação formal (30%)
    const completedCourses = await this.courseRepository.countCompletedByUser(userId);
    const educationScore = Math.min(completedCourses * 5, 30);

    const totalScore = contentScore + practiceScore + educationScore;

    return this.determineStatus(totalScore);
  }

  private determineStatus(score: number): SyndicStatus {
    if (score >= 80) return 'diamante';
    if (score >= 60) return 'ouro';
    if (score >= 40) return 'prata';
    return 'bronze';
  }
}
```

#### 8.2 Job Agendado

```typescript
// apps/api/src/modules/profile/infrastructure/jobs/recalculate-syndic-status.job.ts
export class RecalculateSyndicStatusJob extends BaseTask {
  constructor(
    deps: BaseTaskDependencies,
    private syndicStatusCalculator: SyndicStatusCalculatorService,
    private syndicProfileRepository: ISyndicProfileRepository,
    private logger: FastifyBaseLogger,
  ) {
    super(deps);
  }

  async execute(): Promise<void> {
    this.logger.info('Iniciando recálculo de status de síndicos...');

    const syndics = await this.syndicProfileRepository.findAllActive();
    
    for (const syndic of syndics) {
      try {
        const newStatus = await this.syndicStatusCalculator.calculateStatus(syndic.getUserId());
        
        // Atualizar status no perfil
        syndic.updateRecognitionStatus(newStatus);
        await this.syndicProfileRepository.save(syndic);
        
        this.logger.info(`Status atualizado para usuário ${syndic.getUserId()}: ${newStatus}`);
      } catch (error) {
        this.logger.error(`Erro ao atualizar status para usuário ${syndic.getUserId()}`, error);
      }
    }

    this.logger.info('Recálculo de status concluído');
  }
}
```

#### 8.3 Rotas HTTP

```
GET    /v1/syndic/status/:userId     - Obter status de síndico (público)
```

---

## 4. Configuração do TypeScript

### 4.1 tsconfig.json

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "moduleResolution": "node",
    "target": "ES2022",
    "module": "ESNext",
    "lib": ["ES2022"]
  }
}
```

---

## 5. Referências

- [`CONTEXT_BRAIN.md`](../contextos/CONTEXT_BRAIN.md) - Fonte única da verdade
- [`TECHNICAL_ANALYSIS.md`](../contextos/TECHNICAL_ANALYSIS.md) - Análise técnica completa
- [`PROFILE_MODULE_DOCS.md`](../contextos/PROFILE_MODULE_DOCS.md) - Documentação do módulo de perfil

---

## 6. Próximos Passos

1. **Implementar Fase 1 (Subscription)** - Prioridade máxima
2. **Implementar Fase 2 (Profile Use Cases)** - Prioridade alta
3. **Implementar Fase 3 (Content)** - Prioridade média
4. **Implementar Fase 4 (Assembly)** - Prioridade média
5. **Implementar Fase 5 (Engagement)** - Prioridade baixa
6. **Implementar Fase 6 (LMS)** - Prioridade baixa
7. **Implementar Fase 7 (Webhooks)** - Prioridade alta
8. **Implementar Fase 8 (Reconhecimento)** - Prioridade baixa

---

## 7. Notas Importantes

- **Sempre** seguir o Princípio da Separação (User ≠ Profile ≠ Subscription)
- **Nunca** criar entidades "PaidProfile" - usar PermissionService
- **Sempre** implementar soft delete em todas as entidades
- **Sempre** usar Unit of Work para transações
- **Sempre** validar lógica de negócio no backend
- **Nunca** confiar apenas no frontend para validações
- **Sempre** usar TypeScript strict mode
- **Sempre** seguir os padrões DDD e Clean Architecture
