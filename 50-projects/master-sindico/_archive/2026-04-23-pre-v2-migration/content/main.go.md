Ótima pergunta! Vou te explicar como estruturar o `main.go` seguindo Clean Architecture e as melhores práticas para o projeto Master Síndico.

## 🎯 Estrutura do main.go (cmd/api/main.go)

O `main.go` deve ser **minimalista** e ter apenas 3 responsabilidades:

### 1. **Bootstrapping** (Inicialização)
- Carregar configurações (env vars, arquivos)
- Inicializar logger
- Conectar com dependências externas (DB, Redis, NATS, etc)

### 2. **Dependency Injection** (Wire)
- Usar Wire para injetar todas as dependências
- Criar instâncias de repositórios, services, use cases, handlers
- Montar a árvore de dependências

### 3. **Server Lifecycle** (Start/Stop)
- Iniciar servidor HTTP (Gin)
- Registrar rotas e middlewares
- Graceful shutdown (capturar sinais SIGINT/SIGTERM)

---

## 📋 Fluxo Recomendado

```
main()
  ↓
1. Load Config (pkg/config)
  ↓
2. Initialize Logger (pkg/logger)
  ↓
3. Connect Infrastructure
   - PostgreSQL (GORM)
   - Redis
   - NATS
   - OpenSearch
   - S3/MinIO
  ↓
4. Wire Dependency Injection
   - Repositories
   - Domain Services
   - Use Cases
   - HTTP Handlers
  ↓
5. Setup HTTP Server (Gin)
   - Middlewares (ordem importa!)
     * Recovery
     * Logger
     * CORS
     * Auth
     * Tenant
     * Policy
     * Rate Limit
     * Audit
   - Routes
     * Health checks
     * API v1 routes
     * WebSocket (assembly)
  ↓
6. Start Server
  ↓
7. Wait for Shutdown Signal
  ↓
8. Graceful Shutdown
   - Stop accepting new requests
   - Finish pending requests (timeout 30s)
   - Close DB connections
   - Close Redis connections
   - Close NATS connections
   - Flush logs
```

---

## 🔑 Conceitos Importantes

### **1. Ordem dos Middlewares (CRÍTICO!)**

A ordem importa muito! Deve ser exatamente assim:

```
Request
  ↓
Recovery (captura panics)
  ↓
Logger (loga tudo)
  ↓
CORS (valida origem)
  ↓
Auth (valida JWT, extrai user_id)
  ↓
Tenant (injeta tenant_id no contexto)
  ↓
Trust (cria TrustContext)
  ↓
Policy (valida permissões ABAC)
  ↓
Rate Limit (previne abuse)
  ↓
Audit (registra ação)
  ↓
Handler (executa use case)
```

### **2. Graceful Shutdown**

Quando receber SIGINT/SIGTERM:
1. Para de aceitar novas requisições
2. Aguarda requisições em andamento terminarem (max 30s)
3. Fecha conexões com DB, Redis, NATS
4. Flush de logs pendentes
5. Exit code 0

### **3. Health Checks**

Dois endpoints essenciais:
- `/health` - Básico (sempre retorna 200 OK)
- `/health/ready` - Readiness (verifica DB, Redis, NATS)

### **4. Configuração**

Usar `pkg/config` para centralizar:
- Variáveis de ambiente
- Valores default
- Validação de configs obrigatórias
- Secrets (nunca hardcode!)

---

## 🏗️ Estrutura de Pastas Relacionadas

```
cmd/api/
  └── main.go              # Entry point da API

pkg/
  ├── config/              # Carregamento de configs
  │   └── config.go
  ├── logger/              # Setup do logger
  │   └── logger.go
  └── server/              # Setup do servidor HTTP
      ├── server.go        # Struct Server
      ├── routes.go        # Registro de rotas
      └── middleware.go    # Middlewares globais

infrastructure/wire/
  ├── wire.go              # Definições Wire
  └── wire_gen.go          # Gerado pelo Wire
```

---

## 🎨 Padrão Recomendado

### **Separação de Responsabilidades:**

1. **main.go** → Apenas orquestração
2. **pkg/config** → Gerenciamento de configuração
3. **pkg/logger** → Setup de logging
4. **pkg/server** → Setup do servidor HTTP
5. **infrastructure/wire** → Dependency injection

### **Por que separar?**

- **Testabilidade**: Cada parte pode ser testada isoladamente
- **Reutilização**: `pkg/server` pode ser usado em outros entry points
- **Clareza**: main.go fica com ~100 linhas, fácil de entender
- **Manutenção**: Mudanças em configs não afetam o main

---

## 🔧 Wire (Dependency Injection)

O Wire vai gerar código que cria toda a árvore de dependências:

```
Database Connection
  ↓
Repositories (precisam do DB)
  ↓
Domain Services (precisam dos Repos)
  ↓
Use Cases (precisam dos Services)
  ↓
HTTP Handlers (precisam dos Use Cases)
  ↓
Router (precisa dos Handlers)
```

**Vantagem do Wire:**
- Compile-time (não runtime como outros DI)
- Zero reflection
- Erros de dependência detectados no build
- Performance máxima

---

## 🚦 Exemplo de Fluxo Completo

```
1. Usuario faz request: POST /api/timeline
   ↓
2. Recovery Middleware (captura panics)
   ↓
3. Logger Middleware (loga request)
   ↓
4. CORS Middleware (valida origem)
   ↓
5. Auth Middleware (valida JWT → extrai user_id)
   ↓
6. Tenant Middleware (busca tenant_id do user → injeta no ctx)
   ↓
7. Trust Middleware (cria TrustContext com user_id, tenant_id, role, plan)
   ↓
8. Policy Middleware (valida se user pode criar timeline)
   ↓
9. Rate Limit Middleware (verifica se não excedeu limite)
   ↓
10. Audit Middleware (registra tentativa de criação)
    ↓
11. TimelineHandler (chama PublishTimelineUseCase)
    ↓
12. PublishTimelineUseCase (lógica de negócio)
    ↓
13. TimelineRepository (persiste no DB)
    ↓
14. EventBus (publica TimelinePublished event)
    ↓
15. Response 201 Created
```

---

## 📝 Checklist do main.go

- [ ] Carregar configs (env vars)
- [ ] Validar configs obrigatórias
- [ ] Inicializar logger estruturado
- [ ] Conectar PostgreSQL (com retry)
- [ ] Conectar Redis (com retry)
- [ ] Conectar NATS (com retry)
- [ ] Conectar OpenSearch (opcional, pode falhar)
- [ ] Conectar S3/MinIO
- [ ] Executar Wire para DI
- [ ] Criar servidor Gin
- [ ] Registrar middlewares (ordem correta!)
- [ ] Registrar rotas
- [ ] Iniciar servidor em goroutine
- [ ] Capturar sinais de shutdown
- [ ] Implementar graceful shutdown
- [ ] Logar startup completo

---

## 🎯 Resumo

O `main.go` é o **maestro** que:
1. Prepara o palco (configs, logger, conexões)
2. Chama os músicos (Wire injeta dependências)
3. Organiza a orquestra (middlewares + rotas)
4. Inicia o concerto (start server)
5. Encerra com elegância (graceful shutdown)

**Regra de ouro**: main.go não deve ter lógica de negócio, apenas orquestração!

Quer que eu explique alguma parte específica com mais detalhes?