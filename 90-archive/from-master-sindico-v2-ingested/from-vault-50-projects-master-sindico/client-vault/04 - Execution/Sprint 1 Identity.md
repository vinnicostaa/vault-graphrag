# 🚀 Backlog da Sprint 1: Foundation & Identity

> **Período:** 23/03/2026 - 05/04/2026
> **Objetivo:** Estabelecer a fundação técnica em Go, integrar Zitadel para Auth e Casbin para Authz, e realizar o Onboarding das 4 personas.
> **Status:** 🏗️ Em Progresso

---

## 🛠️ 1. Infraestrutura & Setup (Back-end Go)
*Foco: Boilerplate profissional e conexão com serviços core.*

- [ ] **Setup do Repositório (Hexagonal):** Criar estrutura de pastas `/cmd`, `/internal`, `/pkg`.
- [ ] **Uber-FX Integration:** Configurar Injeção de Dependência global.
- [ ] **Gin Server + Middleware:** Configurar Logger, CORS, Recovery e o `AuthMiddleware` (Zitadel).
- [ ] **Drizzle/Ent Schema:** Criar tabelas base: `users`, `accounts`, `tenants`, `plans`.
- [ ] **NATS JetStream Connection:** Criar o `NATSAdapter` em `/internal/infra/messaging`.

---

## 🔐 2. Identity Module (Zitadel + Casbin)
*Foco: Segurança e Isolamento de Tenants.*

- [ ] **Zitadel OIDC Setup:** Configurar Application, Roles e Key Path.
- [ ] **Casbin Policy:** Implementar o arquivo `rbac_model.conf` e carregar as políticas iniciais (ABAC).
- [ ] **Identity UseCases:**
    - `ValidateToken`: Validar JWT do Zitadel e extrair `userId` + `tenantId`.
    - `SyncUser`: Sincronizar dados do Zitadel com o banco local (Postgres) via Webhook ou On-demand.
    - `CheckPermission`: Wrapper do Casbin para validar `(user, resource, action)`.

---

## 📝 3. Onboarding & Personas (Vertical Slice)
*Foco: Primeira funcionalidade funcional E2E.*

- [ ] **Onboarding API:**
    - `POST /v1/onboarding/start`: Identifica persona via SearchParams.
    - `PATCH /v1/onboarding/progress`: Salva estado parcial no Redis/DB.
    - `POST /v1/onboarding/complete`: Ativa o tenant e emite evento `Identity.TenantActivated` no NATS.
- [ ] **Front-end (SolidJS):** Criar fluxos de telas para Morador (4 telas), Síndico (6 telas), Empresa (7 telas).
- [ ] **Mobile (Flutter):** Implementar Login OIDC e tela inicial de Onboarding.

---

## ✅ Definição de Pronto (DoD) para S1
- [ ] Código em Go com 80%+ de cobertura nos UseCases.
- [ ] Endpoint `/auth/me` retornando as roles corretas do Casbin.
- [ ] Evento de `TenantActivated` capturado pelo NATS.
- [ ] Onboarding funcionando 100% na Web e Mobile.

**Voltar para:** [[00 - Índice Master Síndico|Portal do Arquiteto]]
