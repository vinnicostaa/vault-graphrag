# DIAGRAMAS DE ARQUITETURA - REFATORAÇÃO OSO CLOUD

## 1. FLUXO COMPLETO DE AUTORIZAÇÃO

### 1.1 Fluxo Request → Response

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENTE (Frontend)                       │
│                                                                  │
│  const canPublish = await axios.post('/v1/videos/publish', {...})│
└───────────────────────────────┬─────────────────────────────────┘
                                │ HTTP POST
                                │ Authorization: Bearer <JWT>
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                      FASTIFY MIDDLEWARE CHAIN                    │
│                                                                  │
│  1. cookiePlugin → extrai JWT do cookie                         │
│  2. authenticate hook → valida JWT + busca user + roles         │
│  3. authorize hook → verifica permissão via Oso                 │
│  4. handler → executa use case                                  │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│              AUTHENTICATE HOOK (shared/hooks/)                   │
│                                                                  │
│  const token = jwtService.extractToken(request)                 │
│  ├─> Extrair JWT                                                │
│  │                                                               │
│  const { user, session } = getSessionUseCase.execute({ token }) │
│  ├─> Validar JWT                                                │
│  ├─> Buscar User do banco                                       │
│  ├─> Buscar Session do banco                                    │
│  │                                                               │
│  const roles = roleRepository.findByUserId(user.id)             │
│  ├─> Buscar Roles do banco                                      │
│  │                                                               │
│  syncService.syncUserRoles(user.id, roles)                      │
│  ├─> Sincronizar roles com Oso Cloud (idempotente)              │
│  │                                                               │
│  request.user = user        ✅                                  │
│  request.session = session  ✅                                  │
│  request.roles = roles      ✅                                  │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│               AUTHORIZE HOOK (shared/hooks/)                     │
│                                                                  │
│  const result = authorizationService.authorize(                 │
│    { userId: request.user.id },                                 │
│    'videos:publish',                                            │
│    'Organization'                                               │
│  )                                                               │
│  ├─> Chama AuthorizationService                                 │
│  │                                                               │
│  if (!result.allowed) throw ForbiddenError()                    │
│  └─> Se negado, retorna 403                                     │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│         AUTHORIZATION SERVICE (domain/services/)                 │
│                                                                  │
│  async authorize(context, action, resourceType, resourceId?) {  │
│    const allowed = await this.oso.authorize(                    │
│      { type: 'User', id: context.userId },                      │
│      action,                                                    │
│      { type: resourceType, id: resourceId || 'global' }         │
│    )                                                             │
│    return { allowed, reason }                                   │
│  }                                                               │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    OSO CLOUD API (SAAS)                         │
│                                                                  │
│  1. Buscar fatos do usuário                                     │
│     has_role(User:123, "syndic_n2") → TRUE                      │
│                                                                  │
│  2. Executar policy (main.polar)                                │
│     allow(user, "videos:publish", _org) if                      │
│       has_role(user, "syndic_n2") or                            │
│       has_role(user, "syndic_n3")                               │
│                                                                  │
│  3. Retornar decisão                                            │
│     → TRUE                                                      │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                  HANDLER → USE CASE                             │
│                                                                  │
│  const video = await publishVideoUseCase.execute(...)           │
│  ├─> Lógica de negócio                                          │
│  ├─> Verificar quota (se aplicável)                             │
│  ├─> Criar/Atualizar vídeo no banco                             │
│  └─> Sincronizar ownership com Oso                              │
│                                                                  │
│  return { id, title, status: 'published' }                      │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    RESPONSE → CLIENTE                            │
│                                                                  │
│  HTTP 201 Created                                               │
│  { id: 'xxx', title: 'My Video', status: 'published' }          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. ARQUITETURA EM CAMADAS

```
┌────────────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER (Frontend)                    │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐         │
│  │  Components   │  │  AuthContext  │  │  useAuthGuard │         │
│  │  (Guards)     │  │               │  │               │         │
│  └───────────────┘  └───────────────┘  └───────────────┘         │
└────────────────────────────────────┬───────────────────────────────┘
                                     │ HTTP (REST)
                                     ▼
┌────────────────────────────────────────────────────────────────────┐
│                    APPLICATION LAYER (API Routes)                   │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐         │
│  │  authenticate │→ │  authorize    │→ │   handlers    │         │
│  │  (hook)       │  │  (hook)       │  │               │         │
│  └───────────────┘  └───────────────┘  └───────────────┘         │
└────────────────────────────────────┬───────────────────────────────┘
                                     │
                                     ▼
┌────────────────────────────────────────────────────────────────────┐
│                    DOMAIN LAYER (Use Cases)                         │
│  ┌───────────────────────────────────────────────────────┐         │
│  │  PublishVideoUseCase                                  │         │
│  │  ├─> authorizationService.can()                       │         │
│  │  ├─> videoRepository.create()                         │         │
│  │  └─> syncService.setOwnership()                       │         │
│  └───────────────────────────────────────────────────────┘         │
│                                                                     │
│  ┌───────────────────────────────────────────────────────┐         │
│  │  Domain Services                                      │         │
│  │  ├─> AuthService (sessões/tokens)                     │         │
│  │  ├─> AuthorizationService (permissões)               │         │
│  │  └─> JwtService (JWT)                                 │         │
│  └───────────────────────────────────────────────────────┘         │
└────────────────────────────────────┬───────────────────────────────┘
                                     │
                                     ▼
┌────────────────────────────────────────────────────────────────────┐
│               INFRASTRUCTURE LAYER                                  │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐         │
│  │  Repositories │  │ OsoCloudService│  │  SyncService  │         │
│  │  (Drizzle ORM)│  │  (Oso SDK)    │  │  (Facts Sync) │         │
│  └───────────────┘  └───────────────┘  └───────────────┘         │
└────────────────────────────────────┬───────────────────────────────┘
                                     │
                    ┌────────────────┴────────────────┐
                    ▼                                 ▼
         ┌──────────────────────┐        ┌──────────────────────┐
         │   PostgreSQL         │        │   Oso Cloud API      │
         │   (User Data)        │        │   (Permissions)      │
         └──────────────────────┘        └──────────────────────┘
```

---

## 3. BANCO DE DADOS - ANTES E DEPOIS

### ANTES (Estrutura Antiga)

```
┌──────────────┐
│    users     │
├──────────────┤
│ id           │
│ name         │
│ email        │
│ role ◄───────┼──── ENUM: 'resident' | 'syndic' | 'enterprise'
└──────────────┘      (❌ ÚNICO, não suporta múltiplas roles)

┌──────────────────┐
│  permissions     │  ❌ EXISTE mas NÃO É USADA
├──────────────────┤
│ id               │
│ key              │  Ex: "create_video"
│ name             │  Ex: "Criar Vídeo"
│ category         │
└──────────────────┘

┌──────────────────────┐
│  plan_permissions    │  ❌ EXISTE mas NÃO É USADA
├──────────────────────┤
│ id                   │
│ plan_id              │
│ permission_id        │
│ metadata             │
└──────────────────────┘
```

### DEPOIS (Nova Estrutura)

```
┌──────────────┐
│    users     │
├──────────────┤
│ id           │
│ name         │
│ email        │
│ ❌ role      │  ← REMOVIDO (agora vem de user_roles)
└──────┬───────┘
       │
       │ 1:N
       ▼
┌──────────────┐      N:1      ┌──────────────┐
│  user_roles  │◄───────────────┤    roles     │
├──────────────┤                ├──────────────┤
│ id           │                │ id           │
│ user_id      │                │ name         │  "resident_base"
│ role_id      │                │ display_name │  "Morador Base"
│ expires_at   │                │ description  │
│ deleted_at   │                │ metadata     │  { quotas, features }
└──────────────┘                └──────────────┘

                                ┌──────────────────────────┐
                                │   Oso Cloud (Facts)      │
                                ├──────────────────────────┤
                                │ has_role(User:123,       │
                                │   "resident_base")       │
                                │                          │
                                │ has_role(User:123,       │
                                │   "syndic_n2")           │
                                │                          │
                                │ has_relation(Video:456,  │
                                │   "author", User:123)    │
                                └──────────────────────────┘
                                       ▲
                                       │ sincronizado por
                                       │ SyncService
                                       │
```

---

## 4. DEPENDENCY INJECTION (AWILIX)

### Container Global

```
┌──────────────────────────────────────────────────────────────┐
│                   Fastify App Instance                        │
│                                                               │
│  app.diContainer (GLOBAL CONTAINER)                          │
│  ├─ logger: Logger (SINGLETON)                               │
│  ├─ oso: Oso (SINGLETON) ◄──── Plugin registra              │
│  ├─ jwtService: JwtTokenService (SINGLETON)                  │
│  └─ osoCloudService: OsoCloudService (SINGLETON)             │
│                                                               │
│  app.decorateRequest('diScope') ◄──── Request-scoped DI      │
└──────────────────────────────────────────────────────────────┘
           │
           │ onRequest hook cria
           ▼
┌──────────────────────────────────────────────────────────────┐
│           Request-Scoped Container (diScope)                  │
│                                                               │
│  request.diScope.cradle                                      │
│  ├─ logger: Logger (contexto do request, com requestId)     │
│  ├─ unitOfWork: UnitOfWork (transação do banco)             │
│  │                                                            │
│  ├─ userRepository: UserRepositoryImpl (SCOPED)              │
│  ├─ sessionRepository: SessionRepositoryImpl (SCOPED)        │
│  ├─ roleRepository: RoleRepositoryImpl (SCOPED) ◄── NOVO    │
│  │                                                            │
│  ├─ authService: AuthService (SCOPED)                        │
│  ├─ authorizationService: AuthorizationService (SCOPED) ◄─┐  │
│  │                          └─> usa oso (SINGLETON)      │  │
│  ├─ syncService: SyncService (SCOPED)                    │  │
│  │                          └─> usa oso (SINGLETON)      │  │
│  │                                                        │  │
│  └─ publishVideoUseCase: PublishVideoUseCase (SCOPED)   │  │
│                          └─> usa authorizationService ───┘  │
└──────────────────────────────────────────────────────────────┘
```

### Lifetime Strategy

```typescript
// SINGLETON (uma instância para toda a app)
✅ Oso client (connection pool interno)
✅ JwtService (stateless, apenas cripto)
✅ Logger global (app.log)

// SCOPED (nova instância por request)
✅ Repositories (dependem de UnitOfWork)
✅ AuthorizationService (usa logger do request)
✅ SyncService (usa logger do request)
✅ Use Cases (toda lógica de negócio)
```

---

## 5. PLUGIN OSO - LIFECYCLE

```
┌──────────────────────────────────────────────────────────────┐
│                    APP STARTUP SEQUENCE                       │
└──────────────────────────────────────────────────────────────┘

1️⃣  const app = Fastify({ logger: ... })
     │
     ▼
2️⃣  app.register(awilixPlugin)
     │ ├─> Cria diContainer (global)
     │ └─> Decora app com 'diContainer'
     ▼
3️⃣  app.register(osoPlugin)  ◄─── NOVO
     │ ├─> Cria Oso client
     │ │   const oso = new Oso(OSO_URL, OSO_API_KEY)
     │ │
     │ ├─> Registra no DI
     │ │   diContainer.register({ oso: asValue(oso) })
     │ │
     │ └─> Decora request com helpers
     │     request.can = async (action, type, id) => { ... }
     │     request.authorize = async (action, type, id) => { ... }
     │
     ▼
4️⃣  AppModule.register(app)
     │ ├─> AuthModule.register()
     │ │   ├─> roleRepository: SCOPED
     │ │   ├─> authorizationService: SCOPED (usa oso do container)
     │ │   └─> syncService: SCOPED
     │ │
     │ └─> BillingModule.register()
     │     └─> (permissionService removido)
     │
     ▼
5️⃣  app.listen(3000)

┌──────────────────────────────────────────────────────────────┐
│                    REQUEST LIFECYCLE                          │
└──────────────────────────────────────────────────────────────┘

1️⃣  Request chega
     │
     ▼
2️⃣  onRequest hook (do osoPlugin)
     │ ├─> Injeta helpers no request
     │ │   request.can = () => { ... }
     │ │   request.authorize = () => { ... }
     │ │
     │ └─> Cria diScope (request-scoped container)
     │
     ▼
3️⃣  authenticate hook
     │ ├─> Extrai JWT
     │ ├─> Busca user/session
     │ ├─> Busca roles
     │ ├─> Sincroniza roles com Oso
     │ └─> Injeta no request: user, session, roles
     │
     ▼
4️⃣  authorize hook
     │ └─> authorizationService.authorize(...)
     │     └─> oso.authorize(...)
     │         └─> Oso Cloud API
     │
     ▼
5️⃣  Handler executa
     │
     ▼
6️⃣  Response enviada
```

---

## 6. RBAC vs ABAC - COMPARAÇÃO VISUAL

### RBAC (Permissões Globais)

```
┌─────────────┐         ┌──────────────────┐
│   User      │  tem    │   Role           │
│  (João)     │────────►│ "syndic_n2"      │
└─────────────┘         └──────────────────┘
                              │
                              │ permite
                              ▼
                        ┌──────────────────┐
                        │   Permissão      │
                        │ "videos:publish" │
                        └──────────────────┘
                              │
                              │ em qualquer
                              ▼
                        ┌──────────────────┐
                        │  Organization    │
                        │  (global)        │
                        └──────────────────┘

Polar:
allow(user, "videos:publish", _org: Organization) if
  has_role(user, "syndic_n2");

Verificação:
✅ João tem role "syndic_n2"?
   → SIM → ALLOW
```

### ABAC (Permissões Unitárias)

```
┌─────────────┐         ┌──────────────────┐
│   User      │  autor  │   Video          │
│  (João)     │────────►│ (id: video-123)  │
└─────────────┘         └──────────────────┘
                              │
                              │ relação
                              ▼
                        ┌──────────────────┐
                        │   Permissão      │
                        │   "delete"       │
                        └──────────────────┘

Polar:
resource Video {
  permissions = ["delete"];
  roles = ["author"];
  "delete" if "author";
}

has_relation(video, "author", user);

allow(user, "delete", video) if
  has_permission(video, "delete", user);

Verificação:
✅ João é author de video-123?
   → SIM → ALLOW
```

---

## 7. SINCRONIZAÇÃO OSO - QUANDO E COMO

### Eventos que Disparam Sincronização

```
┌────────────────────────────────────────────────────────────┐
│                  EVENTOS DE SINCRONIZAÇÃO                   │
└────────────────────────────────────────────────────────────┘

1️⃣  USER SIGNUP / LOGIN
    └─> syncService.syncUserRole(userId, 'resident_base')
        └─> Oso: insert has_role(User:123, "resident_base")

2️⃣  MUDANÇA DE PLANO (upgrade/downgrade)
    └─> syncService.syncUserRole(userId, newRole)
        └─> Oso: insert has_role(User:123, "resident_paid")

3️⃣  CRIAÇÃO DE RECURSO (ex: vídeo)
    └─> syncService.setOwnership('Video', videoId, userId, 'author')
        └─> Oso: insert has_relation(Video:456, "author", User:123)

4️⃣  DELEGAÇÃO DE ACESSO (ex: adicionar editor)
    └─> syncService.setOwnership('Video', videoId, editorId, 'editor')
        └─> Oso: insert has_relation(Video:456, "editor", User:789)

5️⃣  REMOÇÃO DE ACESSO
    └─> syncService.removeOwnership('Video', videoId, userId, 'author')
        └─> Oso: delete has_relation(Video:456, "author", User:123)

6️⃣  USER DELETION
    └─> syncService.deleteUserFacts(userId)
        └─> Oso: delete ALL facts for User:123
```

### Fluxo de Sincronização

```
┌────────────────────────────────────────────────────────────┐
│                  BACKEND (Use Case)                         │
│                                                             │
│  1. Executar lógica de negócio                             │
│     video = await videoRepository.create(...)              │
│                                                             │
│  2. Sincronizar com Oso                                    │
│     await syncService.setOwnership(                        │
│       'Video',                                             │
│       video.id,                                            │
│       userId,                                              │
│       'author'                                             │
│     )                                                       │
└────────────────────────┬───────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────┐
│                  SYNC SERVICE                               │
│                                                             │
│  async setOwnership(type, id, userId, relation) {          │
│    await this.oso.insert([                                 │
│      'has_relation',                                       │
│      { type, id },                                         │
│      relation,                                             │
│      { type: 'User', id: userId }                          │
│    ])                                                       │
│  }                                                          │
└────────────────────────┬───────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────┐
│                  OSO CLOUD API                              │
│                                                             │
│  POST /v1/facts                                            │
│  {                                                          │
│    "predicate": "has_relation",                            │
│    "args": [                                               │
│      { "type": "Video", "id": "video-123" },               │
│      "author",                                             │
│      { "type": "User", "id": "user-456" }                  │
│    ]                                                        │
│  }                                                          │
│                                                             │
│  ✅ Fact armazenado                                        │
└────────────────────────────────────────────────────────────┘
```

---

## 8. CONTEXTOS GLOBAIS VS UNITÁRIOS

### Contexto Global

```
┌────────────────────────────────────────────────────────────┐
│              PERGUNTA: "Posso publicar vídeos?"            │
└────────────────────────────────────────────────────────────┘
                         │
                         ▼
         ┌───────────────────────────────┐
         │  Depende do MEU plano?        │
         │  ✅ SIM                       │
         │                               │
         │  Depende de um vídeo          │
         │  específico?                  │
         │  ❌ NÃO                       │
         └───────────────┬───────────────┘
                         │
                         ▼
              ✅ CONTEXTO GLOBAL

Código:
const can = await authService.can(
  { userId },
  'videos:publish',
  'Organization'  ◄─── Tipo genérico, sem ID específico
);

Polar:
allow(user, "videos:publish", _org: Organization) if
  has_role(user, "syndic_n2");
        ▲
        └─── "_org" ignora o recurso específico
```

### Contexto Unitário

```
┌────────────────────────────────────────────────────────────┐
│         PERGUNTA: "Posso deletar ESTE vídeo?"             │
└────────────────────────────────────────────────────────────┘
                         │
                         ▼
         ┌───────────────────────────────┐
         │  Depende do MEU plano?        │
         │  ❌ NÃO                       │
         │                               │
         │  Depende de ser autor DESTE   │
         │  vídeo específico?            │
         │  ✅ SIM                       │
         └───────────────┬───────────────┘
                         │
                         ▼
              ✅ CONTEXTO UNITÁRIO

Código:
const can = await authService.can(
  { userId },
  'delete',
  'Video',
  videoId  ◄─── ID ESPECÍFICO do recurso
);

Polar:
allow(user, "delete", video: Video) if
  has_relation(video, "author", user);
        ▲
        └─── "video" é uma instância específica
```

---

## 9. FRONTEND - COMPONENTES DE GUARD

### PermissionGuard (Global)

```typescript
// Componente
<PermissionGuard permission="videos:publish">
  <PublishButton />
</PermissionGuard>

// Fluxo
┌─────────────────────────────┐
│  PermissionGuard            │
│  permission="videos:publish"│
└──────────┬──────────────────┘
           │
           ▼
    ┌──────────────┐
    │  useAuth()   │
    └──────┬───────┘
           │
           ▼
    ┌────────────────────────┐
    │ user.permissions[]     │
    │ ├─ "videos:publish" ✅ │
    │ └─ "forum:access"      │
    └──────┬─────────────────┘
           │
           ▼
    ┌──────────────┐
    │  RENDERIZA   │
    │ PublishButton│
    └──────────────┘
```

### Ownership Check (Unitário)

```typescript
// Componente
function VideoCard({ video }) {
  const auth = useAuth();
  const isOwner = auth.user().id === video.authorId;
  
  return (
    <Show when={isOwner}>
      <DeleteButton />
    </Show>
  );
}

// Fluxo
┌─────────────────────────────┐
│  VideoCard                  │
│  video.authorId = "user-123"│
└──────────┬──────────────────┘
           │
           ▼
    ┌──────────────┐
    │  useAuth()   │
    │ user.id      │
    │ = "user-123" │
    └──────┬───────┘
           │
           ▼
    ┌───────────────────┐
    │ user.id ===       │
    │ video.authorId?   │
    │ ✅ SIM           │
    └──────┬────────────┘
           │
           ▼
    ┌──────────────┐
    │  RENDERIZA   │
    │ DeleteButton │
    └──────────────┘
```

---

## 10. TROUBLESHOOTING - DECISÕES NEGADAS

### Debug Flow

```
❌ Authorization denied: "videos:publish"

1️⃣  Verificar se user está autenticado
    └─> request.user existe? → ✅ Sim

2️⃣  Verificar roles do user
    └─> request.roles = ["resident_base"]

3️⃣  Verificar facts no Oso
    curl https://cloud.osohq.com/v1/facts
    └─> has_role(User:123, "resident_base") ✅ EXISTS

4️⃣  Verificar policy (main.polar)
    allow(user, "videos:publish", _org) if
      has_role(user, "syndic_n2") or  ❌ NÃO TEM
      has_role(user, "syndic_n3");    ❌ NÃO TEM
    
5️⃣  Usar Oso Explain (debug tool)
    oso-cloud explain User:123 "videos:publish" Organization:global
    
    Output:
    ❌ DENY
    Reason: User does not have required role
    Required: syndic_n2 OR syndic_n3
    Actual: resident_base

6️⃣  Solução: User precisa fazer upgrade de plano
    resident_base → syndic_n2
```

---

## CONCLUSÃO

Estes diagramas cobrem:
✅ Fluxo completo de request → response  
✅ Arquitetura em camadas  
✅ Estrutura de banco antes/depois  
✅ Dependency Injection (Awilix)  
✅ Plugin Oso lifecycle  
✅ RBAC vs ABAC visual  
✅ Sincronização de fatos  
✅ Contextos globais vs unitários  
✅ Frontend guards  
✅ Troubleshooting  

Use estes diagramas junto com o documento de análise técnica para ter visão completa da arquitetura.
