Master Síndico Core Context

    Este arquivo é a Fonte Única da Verdade (Single Source of Truth). Todas as IAs (GLM-4.7, Kimi k2.5) devem seguir estritamente estas diretrizes arquiteturais e de negócio.

1. Visão Geral do Produto

O Master Síndico é uma plataforma de governança condominial aplicada (não é ERP). Foca em capacitação, reconhecimento do síndico, vídeos verticais (TikTok-style) e integração com comércio local.

    Stack: Node.js (Fastify), TypeScript, Drizzle ORM, PostgreSQL, Clean Architecture, DDD.

    Infra: Mux (Vídeos), Stripe (Pagamentos).

2. Regras de Ouro (Arquitetura & Negócio)
2.1. O Princípio da Separação (CRÍTICO)

Nunca misture quem o usuário É com o que ele PAGA.

    User (Identity): Credenciais, login, role imutável (ex: syndic).

    Profile (Dados): Dados cadastrais. Define se o perfil está completo (incomplete vs active).

        Regra: Existe apenas UMA classe base BaseProfile. Não existem entidades separadas para "Pagos".

    Subscription (Acesso): Define o plano (syndic_n2, resident_base) e status financeiro.

    PermissionService (Lógica): Cruza o Plano da Subscription com a Matriz Funcional para autorizar ações.

2.2. Arquitetura de Entidades

    Unified Profile: Todos os perfis (Resident, Syndic, Enterprise, etc.) herdam de BaseProfile.

    Status Unificado: Todo perfil tem status incomplete, pending_payment ou active.

    Soft Delete: Obrigatório em todas as entidades (deletedAt).

2.3. Regras de Negócio Invioláveis

    Onboarding: Fluxo "Stripe-like". Cadastro inicial -> Escolha de Perfil -> Pagamento/Ativação.

    Upload Trimestral: Vídeos institucionais e currículos têm trava sistêmica de 90 dias para atualização.

    Connect Me: Não é chat. É formulário unidirecional com cotas anuais definidas pelo plano.

    Anti-Spam: Perfis de Marketing não possuem Connect Me ativo.

    Reconhecimento: O status do síndico (Bronze...Diamante) é calculado por métricas, não comprado.

3. Estrutura de Domínio (DDD)
Módulos Principais
Plaintext

src/modules/
├── auth/           # Login, Sessions (Cookie-based), Verifications
├── users/          # Entidade User agnóstica
├── profiles/       # Resident, Syndic, Enterprise, etc. (Herdam de BaseProfile)
├── subscription/   # Plans, Subscriptions, PermissionService (ABAC)
├── content/        # Videos (Provider Agnostic), Lives
├── assembly/       # Assembleias, Votação, Atas, Tokens legíveis
├── engagement/     # Forum, Cupons (Logic 4h lock), Connect Me
└── infrastructure/ # Webhooks, Audit Logs, Rate Limits

Mapeamento de Planos (SubscriptionPlan)

    Morador: resident_base (Grátis), resident_paid

    Síndico: syndic_n1 (Aprender), syndic_n2 (Atuar), syndic_n3 (Consolidar)

    Empresa: enterprise_plus, enterprise_pro

    Outros: marketing_standard, local_company_standard

4. Diretrizes para IAs (Prompt System)
🤖 Para o GLM-4.7 (Coding)

    Estilo: Use TypeScript estrito. Drizzle ORM com migrations separadas.

    Repositories: Sempre implemente interfaces (IResidentProfileRepository). Use BaseRepository genérico.

    Patterns: Use Cases transactionais (this.runInTransaction).

    Proibido: Nunca criar lógica de "PaidProfile". Sempre use PermissionService para checar acesso.

    Schema: Siga os schemas definidos em infrastructure/database/drizzle/schema/.

🧠 Para o Kimi k2.5 (Reasoning)

    Contexto: Ao analisar uma feature, consulte a Matriz Funcional para saber as permissões, todos documentos devem e estão em uma pasta chamada contextos na raiz do projeto.

    Validação: Antes de sugerir um fluxo, verifique se ele viola as "Regras de Ouro" (ex: permitir chat em tempo real, o que é proibido).

    Segurança: Sempre sugira validação no Backend, nunca confie apenas no Frontend.

5. Referência Rápida de Tabelas (Schema Drizzle)

    users: Core identity.

    profiles_*: Tabelas separadas (1:1 com users). Ex: resident_profile.

    plans & subscriptions: Controle financeiro.

    videos: Abstração de provider (Mux/Cloudflare).

    assemblies: Tokens legíveis (bjr-fgvo-yhd), controle de acesso.