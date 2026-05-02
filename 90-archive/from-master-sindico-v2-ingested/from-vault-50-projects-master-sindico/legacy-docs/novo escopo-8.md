---
title: "novo escopo-8"
type: source
tags: [pdf, converted]
source: 50-projects/master-sindico/legacy-docs/novo escopo-8.pdf
converted: 2026-04-22
---

# novo escopo-8

> PDF convertido via pdftotext. Layout original perdido; texto preservado.

Master Síndico: Especificação de
Requisitos e Roadmap de Execução
1. Visão Geral do Ecossistema
Plataforma multi-tenant de governança condominial imutável baseada em evidências.

   • Atores: Síndico (Gestor), Morador (Titular + 2 dependentes), Empresa (Prestador)
     e Agência.

   • Core: Timeline append-only (Memória Institucional) e Plano Diretor reativo.

2. Requisitos Funcionais (RF)
2.1. Gestão de Identidade e Acesso (Identity)
   • RF01 - Autenticação OIDC: Login centralizado via Zitadel com suporte a MFA.

   • RF02 - Onboarding de Personas: Fluxo de cadastro diferenciado para as 4
     personas (Síndico, Morador, Empresa, Agência).

   • RF03 - Autorização ABAC: Controle de acesso baseado em atributos (Tenant ID +
     Role + Status do Plano).

2.2. Governança e Compliance (Core)
   • RF04 - Timeline Imutável: Registro de eventos (comunicados, assembleias,
     manutenções) sem permissão de exclusão ou edição retroativa.

   • RF05 - Plano Diretor: Dashboard de status automático derivado das ações
     registradas na Timeline.

   • RF06 - Registro de Unidades: Cadastro de unidades vinculando moradores
     titulares e dependentes.

2.3. Marketplace e Expansão (Connect Me)
   • RF07 - Busca Georeferenciada: Localização de empresas/parceiros por
     proximidade.

   • RF08 - Blindagem LGPD: O contato entre Síndico e Empresa só ocorre após
     "match" ou interesse explícito.
2.4. Monetização (Billing)
     • RF09 - Gestão de Assinaturas: Planos com limites de uso (quotas) e bloqueio
       imediato por inadimplência.



3. Requisitos Não Funcionais (RNF)
     • RNF01 - Performance: Backend em Go (Golang) para alta concorrência e baixa
       latência.

     • RNF02 - Integridade: Persistência em PostgreSQL com camada de leitura (Read
       Model) no OpenSearch para buscas complexas.

     • RNF03 - Arquitetura: Desenvolvimento orientado a Vertical Slices para garantir
       entregas funcionais a cada sprint.

     • RNF04 - Mobile: Aplicativo nativo (iOS/Android) via Flutter.



4. Fatias de Desenvolvimento (Vertical Slices)
     1. Slice 1 (Foundation): Auth (Zitadel) + Tenant Isolation + Setup de Infra.

     2. Slice 2 (Billing): Engine de Planos + Paywall + Integração Financeira.

     3. Slice 3 (Registry): Domínio Condominial (Prédios, Unidades e Membros).

     4. Slice 4 (Governance): Timeline + Plano Diretor + Notificações.

     5. Slice 5 (Connect Me): Busca Global (OpenSearch) + Perfis de Parceiros.



5. Cronograma de Sprints
As datas abaixo seguem rigorosamente o cronograma contratual de 16 semanas:

         Período
Sprint                Foco da Entrega (Mobile + Web + Backend)
         (Ref.)
         01/04 –      Setup & Identity: Flutter Setup, OIDC Zitadel e Arquitetura de
S1
         14/04        Módulos Go.

S2       15/04 –      Onboarding: Auth UI, Cadastro das 4 personas e Engine de
         Período
Sprint                Foco da Entrega (Mobile + Web + Backend)
         (Ref.)
         28/04        Planos/Trial.

         29/04 –
S3                    Core Business: Home, Leitura da Timeline e Plano Diretor Inicial.
         12/05

         13/05 –      Registry: Painel do Morador (Unidades/Dependentes) e
S4
         26/05        Comunicados.

         27/05 –
S5                    Marketplace: Painel Empresa, Connect Me e Geoprocessamento.
         09/06

         10/06 –      Search & Media: Busca Global (OpenSearch), Video Player e
S6
         23/06        Paywall.

         24/06 –
S7                    Connectivity: Notificações Push e Sincronização Web/Mobile.
         07/07

         08/07 –
S8                    Finalization: Módulo de Assembleia, QA e Entrega da Versão 1.0.
         21/07



6. Stack Tecnológica de Referência
     • Backend: Go (Golang) - Core API.

     • Frontend Web: SolidJS (Contratação 50% Vinicius / 50% Jorge).

     • Mobile: Flutter (100% Jorge Maurilio).

     • Infra: PostgreSQL (Relacional), OpenSearch (Search/Analytics), Zitadel (IAM).
