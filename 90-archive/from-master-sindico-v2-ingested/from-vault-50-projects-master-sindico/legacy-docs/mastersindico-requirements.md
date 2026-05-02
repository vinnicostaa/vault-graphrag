---
title: "mastersindico-requirements"
type: source
tags: [pdf, converted]
source: 50-projects/master-sindico/legacy-docs/mastersindico-requirements.pdf
converted: 2026-04-22
---

# mastersindico-requirements

> PDF convertido via pdftotext. Layout original perdido; texto preservado.

Requirements Document — Master Síndico
Platform

Introduction

Master Síndico é um ecossistema digital de governança condominial aplicada, composto por
Backend (API/Core), Frontend Web (SolidJS) e Mobile (iOS/Android). A plataforma opera sobre
arquitetura multi-tenant com dois tipos principais: Condomínio e Empresa, permitindo gestão
profissional, rastreabilidade completa e blindagem institucional para síndicos, moradores e
empresas prestadoras de serviço.

O sistema implementa governança baseada em:

       •Linha do Tempo imutável (append-only, memória institucional)

       •Plano Diretor reativo (status automático via Timeline)

       •Validações centralizadas (empresa → síndico → publicação)

       •Compliance robusto (declarações, auditoria, LGPD)

       •Marketplace unidirecional (Connect Me com LGPD by design)

       •Event-driven architecture para comunicação cross-tenant



Glossary

       •System: Master Síndico Platform
       •Backend: API/Core com regras de negócio, banco de dados e integrações
       •Frontend: SPA responsiva em SolidJS
       •Mobile: Aplicativo iOS/Android
       •Tenant: Unidade de isolamento de dados (Condomínio ou Empresa)
       •Condominium_Tenant: Tenant tipo condomínio (tenant_type='condominium')
       •Company_Tenant: Tenant tipo empresa (tenant_type='company')
       •Sindico: Síndico, gestor do condomínio (até 15 condomínios por síndico)
       •Morador: Morador vinculado a unidade (1 titular + até 2 dependentes)
       •Empresa: Empresa prestadora de serviços
       •Agencia: Agência de marketing (actor delegado, não é tenant)
       •Timeline: Linha do Tempo, memória institucional imutável
       •Master_Plan: Plano Diretor, ações estratégicas com status automático
       •Connect_Me: Marketplace unidirecional de contato
       •Validation_Hub: Hub centralizado de validações pendentes
       •Compliance_Module: Módulo de compliance e auditoria
       •Assembly_Engine: Motor de assembleias (escopo pendente: ao vivo vs assíncrona)
       •External_Service: Serviço de terceiro (Mux, Mercado Pago, AWS SES, etc)
       •Adapter_Layer: Camada anti-corrupção para external services
       •Event_Bus: Barramento de eventos para comunicação entre domínios
       •ABAC: Attribute-Based Access Control
       •LGPD: Lei Geral de Proteção de Dados (Lei 13.709/2018)




HIGH-DOMAIN: Identity


Requirement 1: User Authentication

User Story: As a user, I want to authenticate securely, so that I can access the platform with my
credentials.



Acceptance Criteria

       1.THE Authentication_Service SHALL implement JWT-based authentication with Passport

       2.WHEN a user provides valid credentials, THE Authentication_Service SHALL generate a JWT
       token with expiration

       3.WHEN a user provides invalid credentials, THE Authentication_Service SHALL return an
       authentication error

       4.THE Authentication_Service SHALL support password reset via email verification
       5.THE Authentication_Service SHALL implement rate limiting to prevent brute force attacks

       6.WHEN a JWT token expires, THE Authentication_Service SHALL require re-authentication

       7.THE Authentication_Service SHALL log all authentication attempts with timestamp and IP
       address



Requirement 2: User Authorization (ABAC)

User Story: As a system administrator, I want attribute-based access control, so that users can only
access resources they are authorized for.



Acceptance Criteria

       1.THE Authorization_Service SHALL implement ABAC (Attribute-Based Access Control)

       2.WHEN checking authorization, THE Authorization_Service SHALL evaluate: userId, tenantId,
       role, plan, and action

       3.THE Authorization_Service SHALL return: allowed (boolean), role, plan, and available
       features

       4.THE Authorization_Service SHALL support roles: sindico, morador_titular,
       morador_dependente, empresa_administrador, empresa_operador, agencia

       5.THE Authorization_Service SHALL enforce tenant isolation (users can only access their
       tenant data)

       6.WHEN a user attempts unauthorized action, THE Authorization_Service SHALL deny access
       and log the attempt

       7.THE Authorization_Service SHALL support cross-tenant operations via explicit grants
       (Connect Me, validations)



Requirement 3: User Registry

User Story: As a new user, I want to register on the platform, so that I can create my account.



Acceptance Criteria

       1.THE User_Registry SHALL support registration for: sindico, morador, empresa, agencia
       2.WHEN registering, THE User_Registry SHALL require: email, password, name, phone,
       user_type

       3.THE User_Registry SHALL validate email uniqueness before registration

       4.THE User_Registry SHALL send email verification after registration

       5.THE User_Registry SHALL hash passwords using bcrypt or argon2

       6.THE User_Registry SHALL create user record with status: pending_verification, active,
       suspended, deleted

       7.WHEN email is verified, THE User_Registry SHALL activate the account

       8.THE User_Registry SHALL emit UserRegistered event after successful registration



Requirement 4: Billing and Subscriptions

User Story: As a user, I want to subscribe to a plan, so that I can access premium features.



Acceptance Criteria

       1.THE Billing_Service SHALL support plans: trial, base, plus, pro

       2.THE Billing_Service SHALL provide 7-day trial with full Pro access

       3.WHEN trial expires, THE Billing_Service SHALL downgrade to base plan if no payment

       4.THE Billing_Service SHALL support payment frequencies: monthly, annual

       5.THE Billing_Service SHALL integrate with Mercado Pago via Adapter_Layer

       6.WHEN subscription expires, THE Billing_Service SHALL block premium features retroactively
       (data remains, access blocked)

       7.THE Billing_Service SHALL emit SubscriptionExpired event when subscription ends

       8.THE Billing_Service SHALL maintain billing_history with: date, amount, plan, status,
       payment_method

       9.THE Billing_Service SHALL calculate subscription renewal date from initial subscription date

       10.THE Billing_Service SHALL support plan upgrades and downgrades with prorated billing



Requirement 5: Onboarding

User Story: As a new user, I want guided onboarding, so that I can set up my account correctly.
Acceptance Criteria

       1.THE Onboarding_Service SHALL identify user type via SearchParams or user selection

       2.WHEN user_type is sindico, THE Onboarding_Service SHALL guide through: condominium
       creation, mandate data, empresa_sindica link (optional)

       3.WHEN user_type is morador, THE Onboarding_Service SHALL guide through: condominium
       search by ID, unit registration, termo de ciência, photo upload

       4.WHEN user_type is empresa, THE Onboarding_Service SHALL guide through: company
       profile, CNPJ validation, service categories, institutional data

       5.WHEN user_type is agencia, THE Onboarding_Service SHALL guide through: agency profile,
       service types, portfolio

       6.THE Onboarding_Service SHALL validate required fields before allowing progression

       7.THE Onboarding_Service SHALL save progress to allow resuming onboarding

       8.WHEN onboarding is complete, THE Onboarding_Service SHALL mark user as onboarded
       and redirect to main dashboard



Requirement 6: Plan-Based Feature Access

User Story: As a system, I want to enforce plan-based access, so that users can only use features
included in their plan.



Acceptance Criteria

       1.THE Feature_Access_Service SHALL maintain feature matrix mapping plans to features

       2.WHEN checking feature access, THE Feature_Access_Service SHALL evaluate: userId, plan,
       feature_name

       3.THE Feature_Access_Service SHALL return: allowed (boolean), quota (used, limit, resetDate)
       if applicable

       4.THE Feature_Access_Service SHALL enforce quotas for: Connect Me requests, video uploads,
       storage limits

       5.WHEN quota is exceeded, THE Feature_Access_Service SHALL deny access and return quota
       information

       6.THE Feature_Access_Service SHALL reset quotas based on plan rules (monthly, annual, per-
       action)
       7.THE Feature_Access_Service SHALL support feature flags for gradual rollout




HIGH-DOMAIN: Institutional


Requirement 7: Condominium Registry

User Story: As a sindico, I want to register a condominium, so that I can start managing it.



Acceptance Criteria

       1.THE Condominium_Registry SHALL create tenant with tenant_type='condominium'

       2.WHEN registering, THE Condominium_Registry SHALL require: address, CEP,
       condominium_type, total_units, blocks_or_towers

       3.THE Condominium_Registry SHALL validate CEP via ViaCEP API through Adapter_Layer

       4.THE Condominium_Registry SHALL support condominium_types: residencial, comercial,
       misto, shopping_center, galeria_comercial, edificio_garagem

       5.THE Condominium_Registry SHALL generate unique alphanumeric ID (8 characters)

       6.THE Condominium_Registry SHALL generate QR Code containing condominium ID

       7.THE Condominium_Registry SHALL create mandate record with: mandate_type, start_date,
       end_date, status

       8.THE Condominium_Registry SHALL support mandate_types: sindico_eleito,
       sindico_profissional, sindico_interino

       9.THE Condominium_Registry SHALL allow linking to empresa_sindica (optional)

       10.WHEN condominium is created, THE Condominium_Registry SHALL emit
       CondominiumCreated event

       11.THE Condominium_Registry SHALL allow sindico to manage up to 15 condominiums

       12.WHEN sindico attempts to create 16th condominium, THE Condominium_Registry SHALL
       deny and return limit error
Requirement 8: Condominium Selector

User Story: As a sindico, I want to select which condominium I'm managing, so that I can switch
between my condominiums.



Acceptance Criteria

       1.THE Condominium_Selector SHALL list all condominiums managed by sindico

       2.THE Condominium_Selector SHALL display: name, address, status (active, ended),
       mandate_end_date

       3.THE Condominium_Selector SHALL filter condominiums by status: active, ended

       4.WHEN sindico selects a condominium, THE Condominium_Selector SHALL set active tenant
       context

       5.THE Condominium_Selector SHALL persist selected condominium in session

       6.THE Condominium_Selector SHALL display up to 15 condominiums per sindico



Requirement 9: Unit Registry

User Story: As a morador, I want to register my unit, so that I can link to my condominium.



Acceptance Criteria

       1.THE Unit_Registry SHALL require: condominium_id, block_or_tower, unit_number,
       owner_name, owner_cpf

       2.THE Unit_Registry SHALL validate condominium_id exists before registration

       3.THE Unit_Registry SHALL create unit record with status: pending_approval, active, inactive

       4.THE Unit_Registry SHALL link morador as titular (owner)

       5.THE Unit_Registry SHALL allow 1 titular + up to 2 dependentes per unit

       6.WHEN unit is registered, THE Unit_Registry SHALL emit UnitRegistered event

       7.THE Unit_Registry SHALL require photo upload (selfie or document)

       8.THE Unit_Registry SHALL require acceptance of termo de ciência
Requirement 10: User Management (Condominium)

User Story: As a sindico, I want to manage users in my condominium, so that I can control access.



Acceptance Criteria

       1.THE User_Management SHALL allow sindico to add up to 3 users per condominium

       2.THE User_Management SHALL support roles: sindico_principal, sindico_auxiliar, assistente

       3.THE User_Management SHALL implement permission matrix with toggles for each user

       4.THE User_Management SHALL support permissions: view_timeline, publish_timeline,
       manage_master_plan, manage_events, manage_proposals, manage_votations,
       manage_closed_deals, manage_compliance, manage_users, end_mandate

       5.WHEN adding user, THE User_Management SHALL send invitation email

       6.WHEN user accepts invitation, THE User_Management SHALL activate user with assigned
       permissions

       7.THE User_Management SHALL allow sindico_principal to modify permissions of other users

       8.THE User_Management SHALL prevent sindico_principal from removing themselves

       9.THE User_Management SHALL log all permission changes with timestamp and author



Requirement 11: Timeline (Linha do Tempo)

User Story: As a sindico, I want to publish to the timeline, so that I can maintain institutional memory.



Acceptance Criteria

       1.THE Timeline SHALL implement append-only, immutable log

       2.THE Timeline SHALL support entry_types: master_plan_action, event, comunicado,
       closed_deal, service_execution, assembly_result

       3.WHEN publishing, THE Timeline SHALL require: title, description, entry_type, date, author

       4.THE Timeline SHALL allow attaching: photos, documents, videos

       5.THE Timeline SHALL generate unique entry_id for each publication

       6.THE Timeline SHALL set published_at timestamp on publication

       7.WHEN entry is published, THE Timeline SHALL become immutable (no edits, no deletions)
       8.THE Timeline SHALL allow adendo (addendum) as only modification mechanism

       9.WHEN adendo is created, THE Timeline SHALL link to original entry and mark as amended

       10.THE Timeline SHALL emit TimelinePublished event after publication

       11.THE Timeline SHALL update Master_Plan status automatically when related action is
       published

       12.THE Timeline SHALL be visible to all moradores in read-only mode



Requirement 12: Master Plan (Plano Diretor)

User Story: As a sindico, I want to manage the master plan, so that I can track strategic actions.



Acceptance Criteria

       1.THE Master_Plan SHALL support creating actions with: title, description, area_impactada,
       tipo_atividade, risco_associado, proxima_acao, estimated_date, estimated_cost

       2.THE Master_Plan SHALL support 26 areas_impactadas: estrutura_predial,
       sistema_hidraulico, sistema_eletrico, sistema_incendio, reservatorios, elevadores, garagem,
       fachada, cobertura, seguranca_patrimonial, administracao, portaria, playground, academia,
       salao_festas, corredores, jardins, casa_maquinas, rede_esgoto, rede_drenagem, sistema_gas,
       iluminacao, sistema_cameras, controle_acesso, areas_comuns, outros

       3.THE Master_Plan SHALL support 26 tipos_atividade: manutencao_preventiva,
       manutencao_corretiva, inspecao_tecnica, vistoria_tecnica, contratacao_servico,
       execucao_servico, reparo_emergencial, obra_melhoria, adequacao_normativa,
       atualizacao_infraestrutura, monitoramento_ambiental, revisao_tecnica, auditoria_tecnica,
       limpeza_tecnica, controle_pragas, limpeza_reservatorios, treinamento_tecnico,
       adequacao_legal, revisao_equipamentos, atualizacao_administrativa, contratacao_fornecedor,
       encerramento_contrato, avaliacao_fornecedor, implantacao_sistema, atualizacao_normas,
       acao_emergencial

       4.THE Master_Plan SHALL support 9 riscos_associados: sem_risco, risco_operacional,
       risco_estrutural, risco_sanitario, risco_eletrico, risco_hidraulico, risco_incendio, risco_juridico,
       risco_ambiental

       5.THE Master_Plan SHALL support 8 proximas_acoes: monitorar_evolucao, nova_inspecao,
       contratar_fornecedor, executar_manutencao, atualizar_plano_diretor, registrar_conclusao,
       acompanhar_garantia, reavaliar_necessidade

       6.THE Master_Plan SHALL support 7 status: planejada, em_contratacao, em_execucao,
       concluida, suspensa, reprogramada, atrasada
        7.THE Master_Plan SHALL NOT allow manual status updates

        8.WHEN related entry is published to Timeline, THE Master_Plan SHALL update status
        automatically

        9.THE Master_Plan SHALL calculate risk_score based on risco_associado and days_overdue

        10.THE Master_Plan SHALL emit MasterPlanActionCreated event when action is created

        11.THE Master_Plan SHALL allow filtering by: status, area_impactada, risco_associado,
        date_range



Requirement 13: Events

User Story: As a sindico, I want to create events, so that I can notify moradores about scheduled
activities.



Acceptance Criteria

        1.THE Event_Manager SHALL support creating events with: title, description, event_type,
        start_date, end_date, location, notify_moradores

        2.THE Event_Manager SHALL support 13 event_types: assembleia_ordinaria,
        assembleia_extraordinaria, manutencao_programada, inspecao_tecnica, obra_programada,
        interrupcao_programada_servicos, treinamento_equipe, campanha_institucional,
        evento_comunitario, reuniao_administrativa, atualizacao_normativa, simulado_emergencia,
        outros

        3.WHEN event is created, THE Event_Manager SHALL emit EventCreated event

        4.WHEN notify_moradores is true, THE Event_Manager SHALL send notifications via email and
        push

        5.THE Event_Manager SHALL allow attaching documents and images to events

        6.THE Event_Manager SHALL display events in calendar view

        7.THE Event_Manager SHALL allow filtering events by: event_type, date_range, status

        8.THE Event_Manager SHALL support event status: scheduled, in_progress, completed,
        cancelled

        9.WHEN event is completed, THE Event_Manager SHALL allow publishing to Timeline
Requirement 14: Comunicados

User Story: As a sindico, I want to send comunicados, so that I can inform moradores about
important matters.



Acceptance Criteria

       1.THE Comunicado_Manager SHALL support creating comunicados with: title, message,
       comunicado_type, priority, notify_moradores

       2.THE Comunicado_Manager SHALL support 8 comunicado_types: aviso_operacional,
       interrupcao_programada, orientacao_moradores, aviso_seguranca, comunicado_institucional,
       conclusao_servico, recomendacao_manutencao, alerta_preventivo

       3.THE Comunicado_Manager SHALL support priority levels: low, medium, high, urgent

       4.WHEN comunicado is created, THE Comunicado_Manager SHALL emit ComunicadoCreated
       event

       5.WHEN notify_moradores is true, THE Comunicado_Manager SHALL send notifications via
       email and push

       6.THE Comunicado_Manager SHALL allow attaching documents and images

       7.THE Comunicado_Manager SHALL track read status per morador

       8.THE Comunicado_Manager SHALL allow filtering by: comunicado_type, priority, date_range,
       read_status



Requirement 15: Morador Evaluation (Avaliação de Gestão)

User Story: As a morador, I want to evaluate the management, so that I can provide feedback.



Acceptance Criteria

       1.THE Evaluation_Service SHALL require moradores to evaluate management every 2 months

       2.THE Evaluation_Service SHALL support 5 evaluation criteria: comunicacao, transparencia,
       manutencao, financeiro, atendimento

       3.THE Evaluation_Service SHALL use scale 1-10 for each criterion

       4.THE Evaluation_Service SHALL allow optional text feedback
       5.WHEN evaluation period arrives, THE Evaluation_Service SHALL send notification to
       moradores

       6.WHEN morador submits evaluation, THE Evaluation_Service SHALL emit
       EvaluationSubmitted event

       7.THE Evaluation_Service SHALL calculate average score per criterion

       8.THE Evaluation_Service SHALL display aggregated results to sindico (anonymous)

       9.THE Evaluation_Service SHALL block access to certain features if evaluation is overdue

       10.THE Evaluation_Service SHALL maintain evaluation history per morador



Requirement 16: Morador Access Management

User Story: As a morador titular, I want to manage dependentes, so that I can control who has access
to my unit.



Acceptance Criteria

       1.THE Access_Manager SHALL allow titular to add up to 2 dependentes

       2.WHEN adding dependente, THE Access_Manager SHALL require: name, email, phone,
       relationship

       3.THE Access_Manager SHALL send invitation to dependente via email

       4.WHEN dependente accepts, THE Access_Manager SHALL activate access with role
       morador_dependente

       5.THE Access_Manager SHALL allow titular to revoke dependente access

       6.WHEN access is revoked, THE Access_Manager SHALL immediately block dependente login

       7.THE Access_Manager SHALL log all access changes with timestamp and author



Requirement 17: Morador Vinculo Management

User Story: As a morador, I want to manage my vinculo, so that I can update my unit information or
end my relationship.
Acceptance Criteria

       1.THE Vinculo_Manager SHALL display current vinculo: condominium, unit, role
       (titular/dependente), start_date

       2.THE Vinculo_Manager SHALL allow titular to update: phone, email, emergency_contact

       3.THE Vinculo_Manager SHALL allow titular to end vinculo with required reason

       4.WHEN vinculo is ended, THE Vinculo_Manager SHALL set end_date and status=inactive

       5.WHEN vinculo is ended, THE Vinculo_Manager SHALL revoke access for titular and all
       dependentes

       6.THE Vinculo_Manager SHALL emit VinculoEnded event when vinculo ends

       7.THE Vinculo_Manager SHALL maintain vinculo history per morador




HIGH-DOMAIN: Commercial


Requirement 18: Company Registry

User Story: As a company, I want to register on the platform, so that I can offer my services to
condominiums.



Acceptance Criteria

       1.THE Company_Registry SHALL create tenant with tenant_type='company'

       2.WHEN registering, THE Company_Registry SHALL require: company_name, cnpj, address,
       phone, email, service_categories

       3.THE Company_Registry SHALL validate CNPJ format and uniqueness

       4.THE Company_Registry SHALL support multiple service_categories per company

       5.THE Company_Registry SHALL create company profile with status: pending_verification,
       active, suspended, blocked

       6.WHEN company is created, THE Company_Registry SHALL emit CompanyCreated event

       7.THE Company_Registry SHALL allow company to upload: logo, institutional_photos,
       certifications, licenses
       8.THE Company_Registry SHALL require acceptance of terms of service



Requirement 19: Connect Me (Marketplace)

User Story: As a sindico, I want to request contact from companies, so that I can find service
providers.



Acceptance Criteria

       1.THE Connect_Me SHALL implement unidirectional contact request (sindico → company)

       2.WHEN sindico creates request, THE Connect_Me SHALL require: service_category,
       description, urgency, budget_range

       3.THE Connect_Me SHALL NOT reveal sindico contact data until company shows interest

       4.THE Connect_Me SHALL send notification to matching companies

       5.WHEN company clicks "Tenho Interesse", THE Connect_Me SHALL reveal sindico contact
       data

       6.THE Connect_Me SHALL log data revelation with: timestamp, company_id, sindico_id,
       ip_address, terms_version (LGPD compliance)

       7.THE Connect_Me SHALL enforce plan-based quotas for requests

       8.THE Connect_Me SHALL support request status: open, interested, proposal_sent, closed,
       expired

       9.WHEN request expires (configurable days), THE Connect_Me SHALL auto-close request

       10.THE Connect_Me SHALL emit ConnectMeRequestCreated and ConnectMeInterestShown
       events



Requirement 20: Proposal Management

User Story: As a company, I want to send proposals, so that I can offer my services to condominiums.



Acceptance Criteria

       1.THE Proposal_Manager SHALL allow company to create proposal with: title, description,
       scope, estimated_cost, estimated_duration, attachments
       2.THE Proposal_Manager SHALL link proposal to: condominium_id, connect_me_request_id
       (optional)

       3.THE Proposal_Manager SHALL set proposal status: draft, sent, awaiting_validation,
       validated, rejected, accepted

       4.WHEN proposal is sent, THE Proposal_Manager SHALL emit ProposalSent event

       5.THE Proposal_Manager SHALL require sindico validation before proposal becomes visible to
       moradores

       6.WHEN sindico validates, THE Proposal_Manager SHALL change status to validated

       7.WHEN sindico rejects, THE Proposal_Manager SHALL change status to rejected and require
       rejection_reason

       8.THE Proposal_Manager SHALL allow attaching: technical_documents, photos, videos,
       certifications

       9.THE Proposal_Manager SHALL maintain proposal version history

       10.THE Proposal_Manager SHALL allow company to update proposal only while status=draft



Requirement 21: Supplier Voting (Votação de Fornecedor)

User Story: As a sindico, I want to create supplier voting, so that moradores can choose between
proposals.



Acceptance Criteria

       1.THE Voting_Manager SHALL allow sindico to create voting with: title, description,
       voting_type, start_date, end_date, proposals[]

       2.THE Voting_Manager SHALL support voting_types: simple_majority, qualified_majority,
       unanimous

       3.THE Voting_Manager SHALL link 2 or more validated proposals to voting

       4.THE Voting_Manager SHALL calculate required quorum based on voting_type and
       total_units

       5.WHEN voting starts, THE Voting_Manager SHALL notify all moradores

       6.THE Voting_Manager SHALL allow each morador to vote once

       7.THE Voting_Manager SHALL track votes with: morador_id, proposal_id, timestamp

       8.WHEN voting ends, THE Voting_Manager SHALL calculate results and declare winner
       9.WHEN voting ends, THE Voting_Manager SHALL become immutable (no changes allowed)

       10.THE Voting_Manager SHALL emit VotingCreated, VotingStarted, VotingEnded events

       11.THE Voting_Manager SHALL display real-time voting progress to sindico

       12.THE Voting_Manager SHALL support early closure if quorum and majority are reached



Requirement 22: Closed Deal (Negócio Fechado)

User Story: As a sindico, I want to formalize a closed deal, so that I can register the contract with the
company.



Acceptance Criteria

       1.THE Deal_Manager SHALL allow sindico to create closed deal with: company_id, proposal_id
       (optional), service_description, agreed_cost, start_date, end_date, payment_terms

       2.THE Deal_Manager SHALL generate termo_empresa (company term) requiring company
       signature

       3.THE Deal_Manager SHALL generate termo_sindico (sindico term) requiring sindico signature

       4.THE Deal_Manager SHALL require double confirmation: company signs first, then sindico
       signs

       5.WHEN both signatures are complete, THE Deal_Manager SHALL set status=confirmed and
       lock record (immutable)

       6.THE Deal_Manager SHALL emit DealCreated, DealConfirmed events

       7.THE Deal_Manager SHALL allow attaching: contract_documents, technical_specifications,
       insurance_certificates

       8.WHEN deal is confirmed, THE Deal_Manager SHALL publish to Timeline automatically

       9.THE Deal_Manager SHALL NOT allow edits after confirmation (only adendo)

       10.THE Deal_Manager SHALL support deal status: draft, awaiting_company_signature,
       awaiting_sindico_signature, confirmed, cancelled



Requirement 23: Service Execution Registry

User Story: As a company, I want to register service execution, so that I can document my work.
Acceptance Criteria

       1.THE Execution_Registry SHALL allow company to create execution record with: deal_id,
       execution_date, description, photos, documents

       2.THE Execution_Registry SHALL support execution status: draft, sent, awaiting_validation,
       validated, rejected

       3.WHEN company sends record, THE Execution_Registry SHALL emit ExecutionRecordSent
       event

       4.THE Execution_Registry SHALL require sindico validation before publication

       5.WHEN sindico validates, THE Execution_Registry SHALL publish to Timeline automatically

       6.WHEN sindico rejects, THE Execution_Registry SHALL require rejection_reason and notify
       company

       7.THE Execution_Registry SHALL allow company to update record only while status=draft

       8.THE Execution_Registry SHALL link execution records to closed deal

       9.THE Execution_Registry SHALL maintain execution history per deal



Requirement 24: Post-Deal Comunicados

User Story: As a company, I want to send comunicados after deal closure, so that I can inform about
service progress.



Acceptance Criteria

       1.THE Post_Deal_Comunicado SHALL allow company to create comunicado linked to deal_id

       2.THE Post_Deal_Comunicado SHALL require sindico validation before publication

       3.WHEN company sends comunicado, THE Post_Deal_Comunicado SHALL emit
       ComunicadoSent event

       4.WHEN sindico validates, THE Post_Deal_Comunicado SHALL publish to Timeline and notify
       moradores

       5.WHEN sindico rejects, THE Post_Deal_Comunicado SHALL require rejection_reason

       6.THE Post_Deal_Comunicado SHALL allow attaching: photos, documents, videos

       7.THE Post_Deal_Comunicado SHALL maintain comunicado history per deal
Requirement 25: Adendo (Addendum)

User Story: As a sindico, I want to create an adendo, so that I can modify immutable records formally.



Acceptance Criteria

       1.THE Adendo_Manager SHALL allow creating adendo for: timeline_entry, closed_deal,
       master_plan_action

       2.WHEN creating adendo, THE Adendo_Manager SHALL require: original_record_id, reason,
       description, changes_summary

       3.THE Adendo_Manager SHALL link adendo to original record

       4.THE Adendo_Manager SHALL mark original record as amended

       5.THE Adendo_Manager SHALL publish adendo to Timeline

       6.THE Adendo_Manager SHALL maintain adendo chain (adendo of adendo)

       7.THE Adendo_Manager SHALL emit AdendoCreated event

       8.THE Adendo_Manager SHALL require sindico authorization for all adendos

       9.THE Adendo_Manager SHALL display adendo history in chronological order



Requirement 26: Company Evaluation

User Story: As a sindico, I want to evaluate companies, so that I can provide feedback on their
services.



Acceptance Criteria

       1.THE Company_Evaluation SHALL allow sindico to evaluate company after deal completion

       2.THE Company_Evaluation SHALL support 5 evaluation criteria: qualidade_servico,
       cumprimento_prazo, atendimento, custo_beneficio, recomendacao

       3.THE Company_Evaluation SHALL use scale 1-10 for each criterion

       4.THE Company_Evaluation SHALL allow optional text feedback

       5.WHEN evaluation is submitted, THE Company_Evaluation SHALL emit CompanyEvaluated
       event

       6.THE Company_Evaluation SHALL calculate average score per criterion per company
       7.THE Company_Evaluation SHALL display aggregated results to company (anonymous)

       8.THE Company_Evaluation SHALL display company ratings to other sindicos (public)

       9.THE Company_Evaluation SHALL maintain evaluation history per company



Requirement 27: Parceiro da Vizinhança (Neighborhood Partner)

User Story: As a sindico, I want to register neighborhood partners, so that moradores can access
local services.



Acceptance Criteria

       1.THE Partner_Manager SHALL allow sindico to register partner with: name, category,
       address, phone, description, discount_info

       2.THE Partner_Manager SHALL generate unique partner_link with QR Code

       3.THE Partner_Manager SHALL track partner_link usage with: morador_id, timestamp,
       ip_address

       4.THE Partner_Manager SHALL display partners to moradores in directory

       5.THE Partner_Manager SHALL allow filtering partners by category

       6.THE Partner_Manager SHALL emit PartnerRegistered event when partner is added

       7.THE Partner_Manager SHALL allow sindico to deactivate partners

       8.WHEN partner is deactivated, THE Partner_Manager SHALL hide from morador directory



Requirement 28: Validation Hub

User Story: As a sindico, I want a centralized validation hub, so that I can review all pending
validations.



Acceptance Criteria

       1.THE Validation_Hub SHALL aggregate all pending validations: proposals, execution_records,
       post_deal_comunicados

       2.THE Validation_Hub SHALL display: item_type, company_name, submission_date, priority

       3.THE Validation_Hub SHALL allow filtering by: item_type, company, date_range, priority
       4.THE Validation_Hub SHALL allow bulk actions: validate_all, reject_all

       5.WHEN sindico validates item, THE Validation_Hub SHALL update item status and emit event

       6.WHEN sindico rejects item, THE Validation_Hub SHALL require rejection_reason

       7.THE Validation_Hub SHALL display validation count badge in navigation

       8.THE Validation_Hub SHALL send daily digest email if pending validations exist




HIGH-DOMAIN: Content


Requirement 29: Video Library (Institutional Videos)

User Story: As a company, I want to upload institutional videos, so that I can showcase my services.



Acceptance Criteria

       1.THE Video_Library SHALL allow company to upload videos with: title, description,
       video_type, thumbnail

       2.THE Video_Library SHALL support 12 video_types: apresentacao_institucional,
       apresentacao_equipe_tecnica, demonstracao_servico, explicacao_tecnica,
       boas_praticas_condominiais, orientacao_preventiva, caso_real_atendimento,
       treinamento_tecnico, explicacao_legislacao_normas, dicas_manutencao,
       conteudo_educativo_sindicos, conteudo_educativo_moradores

       3.THE Video_Library SHALL integrate with video streaming service (Mux or Cloudflare Stream)
       via Adapter_Layer

       4.WHEN video is uploaded, THE Video_Library SHALL process video and generate streaming
       URLs

       5.THE Video_Library SHALL enforce plan-based quotas for video uploads and storage

       6.THE Video_Library SHALL support video status: processing, ready, failed, deleted

       7.THE Video_Library SHALL emit VideoUploaded event when video is ready

       8.THE Video_Library SHALL allow filtering videos by: video_type, company, date_range

       9.THE Video_Library SHALL track video views with: viewer_id, timestamp, watch_duration
       10.THE Video_Library SHALL display video analytics to company: views, avg_watch_time,
       completion_rate



Requirement 30: Marketing Agency Management

User Story: As a marketing agency, I want to manage company videos, so that I can help companies
with content.



Acceptance Criteria

       1.THE Agency_Manager SHALL allow agency to link to multiple companies (delegated access)

       2.THE Agency_Manager SHALL allow agency to upload videos on behalf of company

       3.THE Agency_Manager SHALL require company authorization before agency can manage
       content

       4.WHEN agency uploads video, THE Agency_Manager SHALL attribute video to company (not
       agency)

       5.THE Agency_Manager SHALL display dashboard with: companies_managed,
       videos_uploaded, total_views

       6.THE Agency_Manager SHALL allow filtering by company

       7.THE Agency_Manager SHALL emit AgencyVideoUploaded event

       8.THE Agency_Manager SHALL maintain audit log of agency actions per company



Requirement 31: Content Search and Discovery

User Story: As a sindico, I want to search for companies and content, so that I can find relevant
services.



Acceptance Criteria

       1.THE Search_Engine SHALL support searching: companies, videos, service_categories

       2.THE Search_Engine SHALL implement full-text search with relevance ranking

       3.THE Search_Engine SHALL support filters: service_category, location, rating, video_type

       4.THE Search_Engine SHALL display results with: title, description, thumbnail, rating,
       view_count
       5.THE Search_Engine SHALL track search queries for analytics

       6.THE Search_Engine SHALL emit SearchPerformed event

       7.THE Search_Engine SHALL support autocomplete for search queries

       8.THE Search_Engine SHALL display trending searches and popular content



Requirement 32: Content Visibility and Recommendations

User Story: As a sindico, I want personalized recommendations, so that I can discover relevant
content.



Acceptance Criteria

       1.THE Recommendation_Engine SHALL analyze sindico behavior: searches, views,
       connect_me_requests

       2.THE Recommendation_Engine SHALL recommend: companies, videos, service_categories

       3.THE Recommendation_Engine SHALL display recommendations on dashboard

       4.THE Recommendation_Engine SHALL update recommendations based on new interactions

       5.THE Recommendation_Engine SHALL emit RecommendationShown event

       6.THE Recommendation_Engine SHALL track recommendation click-through rate




CROSS-DOMAIN: Compliance and Governance


Requirement 33: Compliance Dashboard

User Story: As a sindico, I want a compliance dashboard, so that I can monitor governance health.



Acceptance Criteria

       1.THE Compliance_Dashboard SHALL display compliance status: em_dia, pendente, atrasado

       2.THE Compliance_Dashboard SHALL aggregate: declaracao_anual_status,
       assinaturas_pendentes, conflito_interesse_status, auditoria_status
       3.THE Compliance_Dashboard SHALL calculate governance_score (0-100) based on
       compliance metrics

       4.THE Compliance_Dashboard SHALL display pending actions with priority

       5.THE Compliance_Dashboard SHALL emit ComplianceDashboardViewed event

       6.THE Compliance_Dashboard SHALL allow filtering by: compliance_area, status, date_range



Requirement 34: Annual Declaration (Declaração Anual)

User Story: As a sindico, I want to submit annual declaration, so that I can comply with governance
requirements.



Acceptance Criteria

       1.THE Annual_Declaration SHALL require submission once per year or at new mandate start

       2.THE Annual_Declaration SHALL require: financial_summary, major_actions, pending_issues,
       next_year_plan

       3.THE Annual_Declaration SHALL support attaching: financial_reports, audit_reports,
       assembly_minutes

       4.WHEN declaration is submitted, THE Annual_Declaration SHALL emit DeclarationSubmitted
       event

       5.THE Annual_Declaration SHALL set compliance status to em_dia

       6.WHEN declaration is overdue, THE Annual_Declaration SHALL block: mandate_end,
       final_report_export, dossie_export

       7.THE Annual_Declaration SHALL maintain declaration history per mandate

       8.THE Annual_Declaration SHALL send reminder notifications 30, 15, 7 days before deadline



Requirement 35: Mandatory Signatures (Assinatura Obrigatória)

User Story: As a user, I want to sign compliance terms, so that I can acknowledge governance rules.



Acceptance Criteria

       1.THE Signature_Manager SHALL require all condominium users to sign compliance terms
       2.THE Signature_Manager SHALL track signatures with: user_id, timestamp, ip_address,
       terms_version

       3.WHEN all users have signed, THE Signature_Manager SHALL set compliance status to
       em_dia

       4.WHEN any user has not signed, THE Signature_Manager SHALL set compliance status to
       pendente

       5.THE Signature_Manager SHALL send reminder notifications to users with pending
       signatures

       6.THE Signature_Manager SHALL block certain actions until signature is complete

       7.THE Signature_Manager SHALL emit SignatureCompleted event when user signs

       8.THE Signature_Manager SHALL maintain signature history per user



Requirement 36: Conflict of Interest Matrix

User Story: As a sindico, I want to declare conflicts of interest, so that I can maintain transparency.



Acceptance Criteria

       1.THE Conflict_Matrix SHALL require sindico to declare relationships with: companies,
       suppliers, service_providers

       2.THE Conflict_Matrix SHALL support relationship_types: familiar, societario, financeiro,
       profissional, nenhum

       3.THE Conflict_Matrix SHALL require annual update or when new relationship emerges

       4.WHEN conflict is declared, THE Conflict_Matrix SHALL emit ConflictDeclared event

       5.THE Conflict_Matrix SHALL display conflicts in compliance dashboard

       6.THE Conflict_Matrix SHALL flag deals with companies where conflict exists

       7.THE Conflict_Matrix SHALL maintain conflict history per sindico



Requirement 37: Audit Trail (Trilha de Auditoria)

User Story: As a system, I want to maintain audit trail, so that all actions are traceable.
Acceptance Criteria

       1.THE Audit_Trail SHALL log all critical actions: timeline_publications, deal_confirmations,
       validations, signature_events, compliance_submissions

       2.THE Audit_Trail SHALL record: user_id, action_type, timestamp, ip_address, user_agent,
       before_state, after_state

       3.THE Audit_Trail SHALL be immutable (append-only, no deletions)

       4.THE Audit_Trail SHALL support filtering by: user, action_type, date_range, entity_type

       5.THE Audit_Trail SHALL allow exporting audit logs in CSV and JSON formats

       6.THE Audit_Trail SHALL emit AuditLogCreated event for each logged action

       7.THE Audit_Trail SHALL retain logs for minimum 5 years (configurable)

       8.THE Audit_Trail SHALL support search by entity_id to trace all actions on specific record



Requirement 38: Risk Map (Mapa de Risco)

User Story: As a sindico, I want to view risk map, so that I can prioritize actions.



Acceptance Criteria

       1.THE Risk_Map SHALL aggregate risks from Master_Plan actions

       2.THE Risk_Map SHALL calculate risk_score based on: risco_associado, days_overdue,
       estimated_cost

       3.THE Risk_Map SHALL display risks in matrix: probability vs impact

       4.THE Risk_Map SHALL support filtering by: area_impactada, risco_associado, status

       5.THE Risk_Map SHALL highlight critical risks (score > threshold)

       6.THE Risk_Map SHALL emit RiskMapViewed event

       7.THE Risk_Map SHALL allow exporting risk map as PDF



Requirement 39: Contractual Compliance (Conformidade Contratual)

User Story: As a sindico, I want to track contract compliance, so that I can ensure companies fulfill
obligations.
Acceptance Criteria

       1.THE Contract_Compliance SHALL track: deal_id, expected_deliverables, actual_deliverables,
       compliance_status

       2.THE Contract_Compliance SHALL calculate compliance_percentage per deal

       3.THE Contract_Compliance SHALL flag deals with compliance issues

       4.THE Contract_Compliance SHALL emit ComplianceIssueDetected event when issue is found

       5.THE Contract_Compliance SHALL display compliance dashboard per company

       6.THE Contract_Compliance SHALL allow sindico to mark deliverables as: pending, completed,
       delayed, not_delivered



Requirement 40: LGPD Registry (Registro LGPD)

User Story: As a system, I want to maintain LGPD registry, so that data sharing is compliant.



Acceptance Criteria

       1.THE LGPD_Registry SHALL log all data sharing events: connect_me_revelations,
       partner_link_usage, external_integrations

       2.THE LGPD_Registry SHALL record: data_subject_id, data_recipient_id, data_types_shared,
       timestamp, ip_address, consent_version, purpose

       3.THE LGPD_Registry SHALL be immutable (append-only)

       4.THE LGPD_Registry SHALL support data subject access requests (export all data for user)

       5.THE LGPD_Registry SHALL support data deletion requests (anonymize user data)

       6.THE LGPD_Registry SHALL emit LGPDEventLogged event

       7.THE LGPD_Registry SHALL maintain registry for minimum 5 years

       8.THE LGPD_Registry SHALL allow exporting LGPD logs per user



Requirement 41: Governance Score

User Story: As a sindico, I want to see governance score, so that I can measure management quality.
Acceptance Criteria

       1.THE Governance_Score SHALL calculate score (0-100) based on: compliance_status,
       timeline_activity, master_plan_completion, morador_evaluations, transparency_metrics

       2.THE Governance_Score SHALL update score daily

       3.THE Governance_Score SHALL display score trend over time

       4.THE Governance_Score SHALL emit GovernanceScoreUpdated event when score changes
       significantly

       5.THE Governance_Score SHALL provide recommendations to improve score

       6.THE Governance_Score SHALL compare score with platform average (anonymized)



Requirement 42: Exportable Dossier (Dossiê Exportável)

User Story: As a sindico, I want to export dossier, so that I can provide complete mandate
documentation.



Acceptance Criteria

       1.THE Dossier_Exporter SHALL generate comprehensive PDF with: mandate_info,
       timeline_entries, master_plan_actions, closed_deals, compliance_records, evaluations,
       governance_score

       2.THE Dossier_Exporter SHALL require compliance status=em_dia before export

       3.WHEN compliance is not em_dia, THE Dossier_Exporter SHALL block export and list pending
       items

       4.THE Dossier_Exporter SHALL include digital signatures and timestamps

       5.THE Dossier_Exporter SHALL emit DossierExported event

       6.THE Dossier_Exporter SHALL support filtering by date_range

       7.THE Dossier_Exporter SHALL generate unique dossier_id for traceability
CROSS-DOMAIN: Indicators and Analytics


Requirement 43: Indicators Dashboard (Sindico)

User Story: As a sindico, I want to view indicators dashboard, so that I can monitor condominium
metrics.



Acceptance Criteria

       1.THE Indicators_Dashboard SHALL display: total_moradores, active_units,
       timeline_entries_count, master_plan_completion_rate, pending_validations_count,
       governance_score, morador_satisfaction_avg

       2.THE Indicators_Dashboard SHALL support date_range filters: last_7_days, last_30_days,
       last_90_days, last_year, custom_range

       3.THE Indicators_Dashboard SHALL display charts: timeline_activity_over_time,
       master_plan_status_distribution, evaluation_trends

       4.THE Indicators_Dashboard SHALL allow exporting dashboard as PDF

       5.THE Indicators_Dashboard SHALL emit DashboardViewed event

       6.THE Indicators_Dashboard SHALL update metrics in real-time

       7.THE Indicators_Dashboard SHALL compare current period with previous period (% change)



Requirement 44: Indicators Dashboard (Company)

User Story: As a company, I want to view indicators dashboard, so that I can monitor business
metrics.



Acceptance Criteria

       1.THE Company_Dashboard SHALL display: connect_me_requests_received, proposals_sent,
       deals_closed, avg_evaluation_score, total_revenue, video_views_count

       2.THE Company_Dashboard SHALL support date_range filters: last_7_days, last_30_days,
       last_90_days, last_year, custom_range

       3.THE Company_Dashboard SHALL display charts: proposals_conversion_rate,
       evaluation_trends, revenue_over_time
          4.THE Company_Dashboard SHALL allow exporting dashboard as PDF

          5.THE Company_Dashboard SHALL emit CompanyDashboardViewed event

          6.THE Company_Dashboard SHALL compare current period with previous period (% change)



Requirement 45: Indicators Dashboard (Agency)

User Story: As an agency, I want to view indicators dashboard, so that I can monitor content
performance.



Acceptance Criteria

          1.THE Agency_Dashboard SHALL display: companies_managed, videos_uploaded, total_views,
          avg_watch_time, top_performing_videos

          2.THE Agency_Dashboard SHALL support filtering by company

          3.THE Agency_Dashboard SHALL display charts: views_over_time, watch_time_distribution,
          video_type_performance

          4.THE Agency_Dashboard SHALL allow exporting dashboard as PDF

          5.THE Agency_Dashboard SHALL emit AgencyDashboardViewed event




CROSS-DOMAIN: Mandate Management


Requirement 46: Mandate End (Encerrar Mandato)

User Story: As a sindico, I want to end my mandate, so that I can formally close my management
period.



Acceptance Criteria

          1.THE Mandate_Manager SHALL allow sindico to end mandate with required reason

          2.THE Mandate_Manager SHALL support end_reasons: fim_periodo, renuncia, destituicao,
          transferencia
       3.THE Mandate_Manager SHALL require compliance status=em_dia before allowing mandate
       end

       4.WHEN compliance is not em_dia, THE Mandate_Manager SHALL block mandate end and list
       pending items

       5.THE Mandate_Manager SHALL set mandate end_date and status=ended

       6.THE Mandate_Manager SHALL generate final report automatically

       7.THE Mandate_Manager SHALL publish mandate end to Timeline

       8.THE Mandate_Manager SHALL emit MandateEnded event

       9.WHEN mandate ends, THE Mandate_Manager SHALL revoke access for all condominium
       users

       10.THE Mandate_Manager SHALL maintain mandate history per condominium



Requirement 47: Mandate Transfer

User Story: As a sindico, I want to transfer mandate, so that the next sindico can continue
management.



Acceptance Criteria

       1.THE Mandate_Transfer SHALL allow current sindico to invite new sindico via email

       2.WHEN new sindico accepts, THE Mandate_Transfer SHALL create new mandate record

       3.THE Mandate_Transfer SHALL transfer: condominium_data, timeline_history,
       master_plan_actions, closed_deals, compliance_records

       4.THE Mandate_Transfer SHALL NOT transfer: user_accounts, personal_evaluations

       5.THE Mandate_Transfer SHALL set previous mandate status=transferred

       6.THE Mandate_Transfer SHALL emit MandateTransferred event

       7.THE Mandate_Transfer SHALL publish transfer to Timeline

       8.THE Mandate_Transfer SHALL require compliance status=em_dia before transfer
CROSS-DOMAIN: Assembly Module (                                🔴 PENDING SCOPE
DECISION)


Requirement 48: Assembly Pre-Configuration

User Story: As a sindico, I want to configure an assembly, so that I can prepare for the meeting.



Acceptance Criteria

       1.THE Assembly_Configurator SHALL allow creating assembly with: title, assembly_type, date,
       time, location, quorum_type

       2.THE Assembly_Configurator SHALL support assembly_types: ordinaria, extraordinaria

       3.THE Assembly_Configurator SHALL support quorum_types: simple_majority,
       qualified_majority, unanimous

       4.THE Assembly_Configurator SHALL allow linking administradora via token

       5.THE Assembly_Configurator SHALL emit AssemblyCreated event

       6.THE Assembly_Configurator SHALL support assembly status: draft, scheduled, in_progress,
       completed, cancelled



Requirement 49: Assembly Agenda (Pauta)

User Story: As a sindico, I want to create assembly agenda, so that I can organize discussion topics.



Acceptance Criteria

       1.THE Agenda_Manager SHALL allow creating agenda items with: title, description, item_type,
       voting_required, quorum_required, attachments

       2.THE Agenda_Manager SHALL support item_types: informativo, deliberativo, votacao

       3.THE Agenda_Manager SHALL allow reordering agenda items

       4.THE Agenda_Manager SHALL calculate total estimated duration

       5.THE Agenda_Manager SHALL emit AgendaItemCreated event

       6.THE Agenda_Manager SHALL allow attaching: documents, proposals, financial_reports
Requirement 50: Assembly Notifications

User Story: As a sindico, I want to notify moradores about assembly, so that they can prepare and
attend.



Acceptance Criteria

          1.THE Assembly_Notifier SHALL send notifications to all moradores with: date, time, location,
          agenda, attachments

          2.THE Assembly_Notifier SHALL send notifications via: email, push, SMS (optional)

          3.THE Assembly_Notifier SHALL send reminders: 7_days_before, 3_days_before, 1_day_before,
          2_hours_before

          4.THE Assembly_Notifier SHALL track notification delivery status

          5.THE Assembly_Notifier SHALL emit AssemblyNotificationSent event



Requirement 51: Preliminary Polls (Enquetes Preliminares)

User Story: As a sindico, I want to create preliminary polls, so that I can gauge morador opinions
before assembly.



Acceptance Criteria

          1.THE Poll_Manager SHALL allow creating polls with: question, options[], end_date

          2.THE Poll_Manager SHALL allow each morador to vote once

          3.THE Poll_Manager SHALL display real-time poll results to sindico

          4.WHEN poll ends, THE Poll_Manager SHALL publish results

          5.THE Poll_Manager SHALL emit PollCreated, PollVoted, PollEnded events



Requirement 52: Proxy Registration (Procuração)

User Story: As a morador, I want to register a proxy, so that someone can vote on my behalf.
Acceptance Criteria

       1.THE Proxy_Manager SHALL allow morador to register proxy with: proxy_holder_name,
       proxy_holder_cpf, assembly_id, scope (all_items or specific_items[])

       2.THE Proxy_Manager SHALL require proxy document upload

       3.THE Proxy_Manager SHALL require administradora validation before proxy is active

       4.WHEN proxy is validated, THE Proxy_Manager SHALL emit ProxyValidated event

       5.THE Proxy_Manager SHALL allow morador to revoke proxy before assembly starts

       6.THE Proxy_Manager SHALL replicate proxy_holder vote to morador vote during assembly



Requirement 53: Supplier Analysis (Análise de Fornecedores)

User Story: As a sindico, I want to analyze suppliers before assembly, so that moradores can make
informed decisions.



Acceptance Criteria

       1.THE Supplier_Analyzer SHALL display: company_profile, proposals, evaluations,
       certifications, videos

       2.THE Supplier_Analyzer SHALL calculate comparison matrix for multiple suppliers

       3.THE Supplier_Analyzer SHALL allow attaching analysis to agenda item

       4.THE Supplier_Analyzer SHALL emit SupplierAnalysisCreated event



Requirement 54: Administradora Panel

User Story: As an administradora, I want to manage assembly data, so that I can support the sindico.



Acceptance Criteria

       1.THE Administradora_Panel SHALL allow importing: unit_list, fracao_ideal,
       inadimplencia_status

       2.THE Administradora_Panel SHALL allow validating proxies

       3.THE Administradora_Panel SHALL display assembly dashboard with: registered_units,
       proxies_pending, quorum_projection
       4.THE Administradora_Panel SHALL emit AdministradoraActionPerformed event



Requirement 55: Quorum Simulator

User Story: As a sindico, I want to simulate quorum, so that I can predict assembly viability.



Acceptance Criteria

       1.THE Quorum_Simulator SHALL calculate: total_units, registered_moradores,
       confirmed_presences, proxies_validated, projected_quorum_percentage

       2.THE Quorum_Simulator SHALL display quorum requirement vs projected quorum

       3.THE Quorum_Simulator SHALL highlight risk if projected quorum is below requirement

       4.THE Quorum_Simulator SHALL update in real-time as confirmations arrive



Requirement 56: Assembly Check-in (                🔴 PENDING: Live vs Async)
User Story: As a morador, I want to check-in to assembly, so that my presence is registered.



Acceptance Criteria

       1.THE Checkin_Manager SHALL support check-in methods: app, qr_code, terminal

       2.WHEN morador checks in, THE Checkin_Manager SHALL register: morador_id, timestamp,
       check_in_method

       3.THE Checkin_Manager SHALL validate morador is authorized for assembly

       4.THE Checkin_Manager SHALL update quorum count in real-time

       5.THE Checkin_Manager SHALL emit MoradorCheckedIn event

       6.THE Checkin_Manager SHALL prevent duplicate check-ins



Requirement 57: Assembly Voting (               🔴 PENDING: Live vs Async)
User Story: As a morador, I want to vote on agenda items, so that I can participate in decisions.
Acceptance Criteria

       1.THE Assembly_Voting SHALL allow voting on agenda items with voting_required=true

       2.THE Assembly_Voting SHALL support vote_options: favor, contra, abstencao

       3.THE Assembly_Voting SHALL allow each morador to vote once per item

       4.THE Assembly_Voting SHALL replicate proxy votes automatically

       5.THE Assembly_Voting SHALL calculate results in real-time

       6.THE Assembly_Voting SHALL emit VoteCast event

       7.THE Assembly_Voting SHALL display results on telao (big screen)

       8.WHEN voting ends, THE Assembly_Voting SHALL lock votes (immutable)



Requirement 58: Assembly Finalization

User Story: As a sindico, I want to finalize assembly, so that I can close the meeting and publish
results.



Acceptance Criteria

       1.THE Assembly_Finalizer SHALL generate final report with: attendance, votes_per_item,
       decisions, minutes

       2.THE Assembly_Finalizer SHALL require sindico confirmation before finalization

       3.WHEN assembly is finalized, THE Assembly_Finalizer SHALL set status=completed

       4.THE Assembly_Finalizer SHALL publish assembly results to Timeline

       5.THE Assembly_Finalizer SHALL emit AssemblyFinalized event

       6.THE Assembly_Finalizer SHALL generate PDF report with digital signatures

       7.THE Assembly_Finalizer SHALL send final report to all moradores via email



Requirement 59: Assembly Reports

User Story: As a sindico, I want to generate assembly reports, so that I can document the meeting.
Acceptance Criteria

       1.THE Assembly_Reporter SHALL generate reports: attendance_report, voting_report,
       proxy_report, audit_report

       2.THE Assembly_Reporter SHALL support exporting reports in PDF and CSV formats

       3.THE Assembly_Reporter SHALL include: morador_list, presence_status, votes_cast,
       proxy_usage, quorum_achieved

       4.THE Assembly_Reporter SHALL emit ReportGenerated event



Requirement 60: Assembly Mode (Presentation)

User Story: As a sindico, I want to present assembly in slide mode, so that I can conduct the meeting
professionally.



Acceptance Criteria

       1.THE Assembly_Presenter SHALL display assembly in full-screen slide mode

       2.THE Assembly_Presenter SHALL navigate through agenda items sequentially

       3.THE Assembly_Presenter SHALL display: current_item, voting_results, quorum_status, timer

       4.THE Assembly_Presenter SHALL support presenter notes (visible only to sindico)

       5.THE Assembly_Presenter SHALL display institutional video before assembly starts

       6.THE Assembly_Presenter SHALL emit PresentationStarted event




CROSS-DOMAIN: External Integrations


Requirement 61: Payment Integration (Mercado Pago)

User Story: As a user, I want to pay via Mercado Pago, so that I can subscribe to plans.



Acceptance Criteria

       1.THE Payment_Adapter SHALL integrate with Mercado Pago API via Adapter_Layer
       2.THE Payment_Adapter SHALL support payment methods: credit_card, debit_card, pix, boleto

       3.WHEN payment is initiated, THE Payment_Adapter SHALL create payment_intent with:
       amount, currency, description, customer_id

       4.WHEN payment succeeds, THE Payment_Adapter SHALL emit PaymentSucceeded event

       5.WHEN payment fails, THE Payment_Adapter SHALL emit PaymentFailed event with
       failure_reason

       6.THE Payment_Adapter SHALL handle webhooks for payment status updates

       7.THE Payment_Adapter SHALL implement retry logic for failed payments

       8.THE Payment_Adapter SHALL log all payment transactions with: transaction_id, amount,
       status, timestamp



Requirement 62: Email Service Integration

User Story: As a system, I want to send emails, so that I can notify users.



Acceptance Criteria

       1.THE Email_Adapter SHALL integrate with email service (AWS SES or SendGrid) via
       Adapter_Layer

       2.THE Email_Adapter SHALL support email types: transactional, notification, marketing

       3.WHEN sending email, THE Email_Adapter SHALL require: recipient, subject, body, email_type

       4.THE Email_Adapter SHALL use email templates for consistent branding

       5.THE Email_Adapter SHALL track email delivery status: sent, delivered, bounced, opened,
       clicked

       6.THE Email_Adapter SHALL emit EmailSent event

       7.THE Email_Adapter SHALL implement rate limiting to prevent spam

       8.THE Email_Adapter SHALL support batch sending for bulk notifications



Requirement 63: SMS Service Integration

User Story: As a system, I want to send SMS, so that I can notify users via text message.
Acceptance Criteria

       1.THE SMS_Adapter SHALL integrate with SMS service (Twilio or Zenvia) via Adapter_Layer

       2.WHEN sending SMS, THE SMS_Adapter SHALL require: phone_number, message, sms_type

       3.THE SMS_Adapter SHALL validate phone number format before sending

       4.THE SMS_Adapter SHALL track SMS delivery status: sent, delivered, failed

       5.THE SMS_Adapter SHALL emit SMSSent event

       6.THE SMS_Adapter SHALL implement rate limiting per user

       7.THE SMS_Adapter SHALL support SMS templates for common notifications



Requirement 64: Video Streaming Integration

User Story: As a system, I want to stream videos, so that users can watch institutional content.



Acceptance Criteria

       1.THE Video_Adapter SHALL integrate with video streaming service (Mux or Cloudflare
       Stream) via Adapter_Layer

       2.WHEN video is uploaded, THE Video_Adapter SHALL process video and generate:
       streaming_url, thumbnail_url, duration, resolution

       3.THE Video_Adapter SHALL support adaptive bitrate streaming

       4.THE Video_Adapter SHALL track video playback events: play, pause, seek, complete

       5.THE Video_Adapter SHALL emit VideoProcessed event when video is ready

       6.THE Video_Adapter SHALL implement video access control (signed URLs)

       7.THE Video_Adapter SHALL support video deletion and cleanup



Requirement 65: Storage Integration (S3)

User Story: As a system, I want to store files, so that users can upload documents and images.
Acceptance Criteria

          1.THE Storage_Adapter SHALL integrate with object storage (AWS S3 or equivalent) via
          Adapter_Layer

          2.THE Storage_Adapter SHALL support file types: images, documents, videos

          3.WHEN file is uploaded, THE Storage_Adapter SHALL generate: file_url, file_size, mime_type,
          upload_timestamp

          4.THE Storage_Adapter SHALL implement file access control (signed URLs with expiration)

          5.THE Storage_Adapter SHALL enforce file size limits per file type

          6.THE Storage_Adapter SHALL emit FileUploaded event

          7.THE Storage_Adapter SHALL support file deletion and cleanup

          8.THE Storage_Adapter SHALL organize files by tenant_id for isolation



Requirement 66: CEP Lookup Integration

User Story: As a user, I want to auto-fill address from CEP, so that I can register faster.



Acceptance Criteria

          1.THE CEP_Adapter SHALL integrate with ViaCEP API via Adapter_Layer

          2.WHEN CEP is provided, THE CEP_Adapter SHALL return: street, neighborhood, city, state

          3.WHEN CEP is invalid, THE CEP_Adapter SHALL return error

          4.THE CEP_Adapter SHALL implement caching to reduce API calls

          5.THE CEP_Adapter SHALL emit CEPLookupPerformed event



Requirement 67: Push Notification Integration

User Story: As a system, I want to send push notifications, so that mobile users receive real-time
alerts.



Acceptance Criteria

          1.THE Push_Adapter SHALL integrate with push notification service (Firebase Cloud
          Messaging or equivalent) via Adapter_Layer
       2.WHEN sending push, THE Push_Adapter SHALL require: user_id, title, body,
       notification_type, deep_link

       3.THE Push_Adapter SHALL support platforms: iOS, Android

       4.THE Push_Adapter SHALL track notification delivery status: sent, delivered, opened

       5.THE Push_Adapter SHALL emit PushNotificationSent event

       6.THE Push_Adapter SHALL support notification badges and sounds

       7.THE Push_Adapter SHALL implement notification preferences per user




CROSS-DOMAIN: Data Parsers and Serializers


Requirement 68: JSON Parser and Serializer

User Story: As a system, I want to parse and serialize JSON, so that I can exchange data with clients
and external services.



Acceptance Criteria

       1.WHEN a valid JSON string is provided, THE JSON_Parser SHALL parse it into internal data
       structures

       2.WHEN an invalid JSON string is provided, THE JSON_Parser SHALL return a descriptive error
       with line and column information

       3.THE JSON_Serializer SHALL format internal data structures into valid JSON strings

       4.FOR ALL valid data structures, parsing then serializing then parsing SHALL produce an
       equivalent structure (round-trip property)

       5.THE JSON_Parser SHALL support all JSON data types: object, array, string, number, boolean,
       null

       6.THE JSON_Serializer SHALL produce properly escaped strings and valid number formats



Requirement 69: CSV Parser and Serializer

User Story: As a sindico, I want to import/export data in CSV format, so that I can work with
spreadsheets.
Acceptance Criteria

       1.WHEN a valid CSV file is provided, THE CSV_Parser SHALL parse it into structured records

       2.WHEN an invalid CSV file is provided, THE CSV_Parser SHALL return a descriptive error with
       row and column information

       3.THE CSV_Serializer SHALL format structured records into valid CSV files

       4.FOR ALL valid record sets, parsing then serializing then parsing SHALL produce equivalent
       records (round-trip property)

       5.THE CSV_Parser SHALL support: custom delimiters, quoted fields, escaped quotes, headers

       6.THE CSV_Serializer SHALL properly escape special characters and quote fields containing
       delimiters



Requirement 70: PDF Generator

User Story: As a system, I want to generate PDFs, so that users can export reports and documents.



Acceptance Criteria

       1.THE PDF_Generator SHALL generate PDFs from: HTML templates, structured data, images

       2.THE PDF_Generator SHALL support: headers, footers, page numbers, table of contents

       3.THE PDF_Generator SHALL embed: images, fonts, digital signatures

       4.THE PDF_Generator SHALL generate accessible PDFs (PDF/UA compliance)

       5.THE PDF_Generator SHALL emit PDFGenerated event

       6.THE PDF_Generator SHALL support custom page sizes and orientations




CROSS-DOMAIN: Security and Data Protection


Requirement 71: Data Encryption at Rest

User Story: As a system, I want to encrypt sensitive data at rest, so that data is protected from
unauthorized access.
Acceptance Criteria

       1.THE Encryption_Service SHALL encrypt sensitive fields: passwords, cpf, cnpj, financial_data,
       personal_documents

       2.THE Encryption_Service SHALL use industry-standard encryption algorithms (AES-256)

       3.THE Encryption_Service SHALL manage encryption keys securely (key rotation, secure
       storage)

       4.THE Encryption_Service SHALL decrypt data only when authorized user requests it

       5.THE Encryption_Service SHALL log all decryption operations for audit



Requirement 72: Data Encryption in Transit

User Story: As a system, I want to encrypt data in transit, so that communication is secure.



Acceptance Criteria

       1.THE System SHALL enforce HTTPS for all API endpoints

       2.THE System SHALL use TLS 1.2 or higher

       3.THE System SHALL reject unencrypted HTTP requests

       4.THE System SHALL implement certificate pinning for mobile apps

       5.THE System SHALL use secure WebSocket connections (WSS) if real-time features are
       implemented



Requirement 73: Input Validation and Sanitization

User Story: As a system, I want to validate and sanitize inputs, so that I can prevent injection attacks.



Acceptance Criteria

       1.THE Validation_Service SHALL validate all user inputs against defined schemas (Zod)

       2.THE Validation_Service SHALL sanitize inputs to prevent: SQL injection, XSS, command
       injection

       3.WHEN input is invalid, THE Validation_Service SHALL return descriptive validation errors
       4.THE Validation_Service SHALL enforce: data types, string lengths, numeric ranges, regex
       patterns

       5.THE Validation_Service SHALL validate file uploads: file type, file size, file content



Requirement 74: Rate Limiting and DDoS Protection

User Story: As a system, I want to implement rate limiting, so that I can prevent abuse and DDoS
attacks.



Acceptance Criteria

       1.THE Rate_Limiter SHALL enforce rate limits per: IP address, user_id, API endpoint

       2.THE Rate_Limiter SHALL support configurable limits: requests_per_minute,
       requests_per_hour, requests_per_day

       3.WHEN rate limit is exceeded, THE Rate_Limiter SHALL return 429 Too Many Requests error

       4.THE Rate_Limiter SHALL implement exponential backoff for repeated violations

       5.THE Rate_Limiter SHALL emit RateLimitExceeded event

       6.THE Rate_Limiter SHALL whitelist trusted IPs (admin, monitoring)



Requirement 75: Session Management

User Story: As a system, I want to manage user sessions securely, so that unauthorized access is
prevented.



Acceptance Criteria

       1.THE Session_Manager SHALL generate secure session tokens (JWT)

       2.THE Session_Manager SHALL set session expiration (configurable, default 24 hours)

       3.THE Session_Manager SHALL support session refresh without re-authentication

       4.THE Session_Manager SHALL invalidate sessions on: logout, password change, suspicious
       activity

       5.THE Session_Manager SHALL track active sessions per user

       6.THE Session_Manager SHALL allow users to view and revoke active sessions
       7.THE Session_Manager SHALL implement concurrent session limits per user



Requirement 76: Two-Factor Authentication (Optional)

User Story: As a user, I want to enable 2FA, so that my account is more secure.



Acceptance Criteria

       1.WHERE 2FA is enabled, THE Authentication_Service SHALL require second factor after
       password

       2.THE Authentication_Service SHALL support 2FA methods: TOTP (authenticator app), SMS

       3.WHEN enabling 2FA, THE Authentication_Service SHALL generate backup codes

       4.THE Authentication_Service SHALL allow disabling 2FA with backup code or account
       recovery

       5.THE Authentication_Service SHALL emit TwoFactorEnabled, TwoFactorDisabled events




CROSS-DOMAIN: Performance and Scalability


Requirement 77: Caching Strategy

User Story: As a system, I want to implement caching, so that performance is optimized.



Acceptance Criteria

       1.THE Cache_Manager SHALL cache frequently accessed data: user_profiles,
       condominium_data, plan_features

       2.THE Cache_Manager SHALL support cache invalidation strategies: TTL, manual invalidation,
       event-based invalidation

       3.THE Cache_Manager SHALL use distributed cache (Redis or equivalent) for multi-instance
       deployments

       4.THE Cache_Manager SHALL implement cache warming for critical data

       5.THE Cache_Manager SHALL emit CacheHit, CacheMiss events for monitoring
Requirement 78: Database Query Optimization

User Story: As a system, I want optimized database queries, so that response times are fast.



Acceptance Criteria

       1.THE Database_Layer SHALL use indexes on frequently queried fields: tenant_id, user_id,
       status, created_at

       2.THE Database_Layer SHALL implement query pagination for large result sets

       3.THE Database_Layer SHALL use database connection pooling

       4.THE Database_Layer SHALL log slow queries (threshold configurable, default 1 second)

       5.THE Database_Layer SHALL implement read replicas for read-heavy operations



Requirement 79: Asynchronous Processing

User Story: As a system, I want to process heavy tasks asynchronously, so that API responses are fast.



Acceptance Criteria

       1.THE Job_Queue SHALL process asynchronous tasks: email_sending, video_processing,
       report_generation, notification_dispatch

       2.THE Job_Queue SHALL support job priorities: low, normal, high, urgent

       3.THE Job_Queue SHALL implement retry logic with exponential backoff for failed jobs

       4.THE Job_Queue SHALL track job status: queued, processing, completed, failed

       5.THE Job_Queue SHALL emit JobQueued, JobCompleted, JobFailed events

       6.THE Job_Queue SHALL support scheduled jobs (cron-like)



Requirement 80: Event-Driven Architecture

User Story: As a system, I want event-driven communication, so that domains are decoupled.
Acceptance Criteria

       1.THE Event_Bus SHALL publish domain events: UserRegistered, TimelinePublished,
       DealConfirmed, etc.

       2.THE Event_Bus SHALL support event subscribers across domains

       3.THE Event_Bus SHALL guarantee at-least-once delivery for critical events

       4.THE Event_Bus SHALL implement dead letter queue for failed event processing

       5.THE Event_Bus SHALL log all published events for audit and replay

       6.THE Event_Bus SHALL support event versioning for backward compatibility




CROSS-DOMAIN: Monitoring and Observability


Requirement 81: Application Logging

User Story: As a developer, I want comprehensive logging, so that I can troubleshoot issues.



Acceptance Criteria

       1.THE Logger SHALL log events with levels: debug, info, warn, error, fatal

       2.THE Logger SHALL include context: timestamp, user_id, tenant_id, request_id, ip_address,
       user_agent

       3.THE Logger SHALL support structured logging (JSON format)

       4.THE Logger SHALL integrate with log aggregation service (CloudWatch, Datadog, or
       equivalent)

       5.THE Logger SHALL implement log sampling for high-volume events

       6.THE Logger SHALL sanitize sensitive data before logging (passwords, tokens, PII)



Requirement 82: Error Tracking

User Story: As a developer, I want to track errors, so that I can fix bugs quickly.
Acceptance Criteria

          1.THE Error_Tracker SHALL capture unhandled exceptions and errors

          2.THE Error_Tracker SHALL include: stack_trace, request_context, user_context, environment

          3.THE Error_Tracker SHALL group similar errors for easier analysis

          4.THE Error_Tracker SHALL integrate with error tracking service (Sentry, Rollbar, or equivalent)

          5.THE Error_Tracker SHALL emit alerts for critical errors

          6.THE Error_Tracker SHALL track error frequency and trends



Requirement 83: Performance Monitoring

User Story: As a developer, I want to monitor performance, so that I can identify bottlenecks.



Acceptance Criteria

          1.THE Performance_Monitor SHALL track: API response times, database query times, external
          service latency

          2.THE Performance_Monitor SHALL calculate percentiles: p50, p95, p99

          3.THE Performance_Monitor SHALL emit alerts when thresholds are exceeded

          4.THE Performance_Monitor SHALL integrate with APM service (New Relic, Datadog, or
          equivalent)

          5.THE Performance_Monitor SHALL track resource usage: CPU, memory, disk, network



Requirement 84: Health Checks

User Story: As a system, I want health check endpoints, so that monitoring systems can verify service
status.



Acceptance Criteria

          1.THE Health_Check SHALL expose /health endpoint returning service status

          2.THE Health_Check SHALL verify: database connectivity, cache connectivity, external service
          availability

          3.THE Health_Check SHALL return HTTP 200 when healthy, 503 when unhealthy
       4.THE Health_Check SHALL include detailed status per dependency

       5.THE Health_Check SHALL implement liveness and readiness probes for Kubernetes



Requirement 85: Metrics Collection

User Story: As a developer, I want to collect metrics, so that I can monitor system behavior.



Acceptance Criteria

       1.THE Metrics_Collector SHALL track: request_count, error_rate, active_users, tenant_count,
       event_count

       2.THE Metrics_Collector SHALL support custom business metrics:
       timeline_publications_per_day, deals_closed_per_month

       3.THE Metrics_Collector SHALL integrate with metrics service (Prometheus, CloudWatch, or
       equivalent)

       4.THE Metrics_Collector SHALL support metric aggregation: sum, avg, min, max, count

       5.THE Metrics_Collector SHALL enable metric-based alerting




CROSS-DOMAIN: Testing and Quality Assurance


Requirement 86: Automated Testing

User Story: As a developer, I want automated tests, so that I can ensure code quality.



Acceptance Criteria

       1.THE Test_Suite SHALL include: unit tests, integration tests, end-to-end tests

       2.THE Test_Suite SHALL achieve minimum 80% code coverage for critical paths

       3.THE Test_Suite SHALL run automatically on: commit, pull request, deployment

       4.THE Test_Suite SHALL fail build if tests fail

       5.THE Test_Suite SHALL generate test reports with coverage metrics
Requirement 87: Property-Based Testing for Parsers

User Story: As a developer, I want property-based tests for parsers, so that edge cases are covered.



Acceptance Criteria

       1.FOR ALL valid JSON objects, parsing then serializing then parsing SHALL produce equivalent
       objects (round-trip property)

       2.FOR ALL valid CSV records, parsing then serializing then parsing SHALL produce equivalent
       records (round-trip property)

       3.THE Property_Tests SHALL generate random valid inputs to test parser robustness

       4.THE Property_Tests SHALL verify parser error handling with invalid inputs

       5.THE Property_Tests SHALL run as part of automated test suite




CROSS-DOMAIN: Deployment and DevOps


Requirement 88: Continuous Integration/Continuous Deployment

User Story: As a developer, I want CI/CD pipeline, so that deployments are automated and reliable.



Acceptance Criteria

       1.THE CI_CD_Pipeline SHALL run on: commit to main branch, pull request creation

       2.THE CI_CD_Pipeline SHALL execute: linting, type checking, tests, build

       3.THE CI_CD_Pipeline SHALL deploy to: staging (automatic), production (manual approval)

       4.THE CI_CD_Pipeline SHALL implement blue-green deployment or canary deployment

       5.THE CI_CD_Pipeline SHALL support rollback to previous version

       6.THE CI_CD_Pipeline SHALL emit deployment notifications to team
Requirement 89: Environment Configuration

User Story: As a developer, I want environment-specific configuration, so that services behave
correctly per environment.



Acceptance Criteria

       1.THE Config_Manager SHALL support environments: development, staging, production

       2.THE Config_Manager SHALL load configuration from: environment variables, config files,
       secrets manager

       3.THE Config_Manager SHALL validate required configuration on startup

       4.THE Config_Manager SHALL support configuration hot-reload for non-critical settings

       5.THE Config_Manager SHALL never log sensitive configuration values



Requirement 90: Database Migrations

User Story: As a developer, I want database migrations, so that schema changes are versioned and
reproducible.



Acceptance Criteria

       1.THE Migration_Manager SHALL version all database schema changes

       2.THE Migration_Manager SHALL support: up migrations, down migrations (rollback)

       3.THE Migration_Manager SHALL run migrations automatically on deployment

       4.THE Migration_Manager SHALL track applied migrations in database

       5.THE Migration_Manager SHALL fail deployment if migration fails

       6.THE Migration_Manager SHALL support data migrations in addition to schema migrations
CROSS-DOMAIN: Accessibility and Internationalization


Requirement 91: Web Accessibility (WCAG Compliance)

User Story: As a user with disabilities, I want accessible interface, so that I can use the platform.



Acceptance Criteria

       1.THE Frontend SHALL implement semantic HTML with proper heading hierarchy

       2.THE Frontend SHALL provide alt text for all images

       3.THE Frontend SHALL support keyboard navigation for all interactive elements

       4.THE Frontend SHALL maintain sufficient color contrast (WCAG AA minimum)

       5.THE Frontend SHALL provide focus indicators for keyboard navigation

       6.THE Frontend SHALL support screen readers with ARIA labels and roles

       7.THE Frontend SHALL allow text resizing without breaking layout



Requirement 92: Internationalization (i18n)

User Story: As a user, I want interface in my language, so that I can understand the platform.



Acceptance Criteria

       1.THE Frontend SHALL support multiple languages: pt-BR (primary), en-US, es-ES

       2.THE Frontend SHALL load translations from language files

       3.THE Frontend SHALL detect user language from: browser settings, user preference

       4.THE Frontend SHALL allow users to change language in settings

       5.THE Frontend SHALL format dates, numbers, and currency according to locale

       6.THE Frontend SHALL support RTL languages (future-proofing)
🔴 CRITICAL PENDING DECISIONS
Decision 1: Assembly Module Scope

Context: Assembly module has two possible implementations with drastically different complexity
and infrastructure requirements.

Options:

Option A: Asynchronous Assembly (Recommended for MVP)

       •Pre-assembly: configuration, agenda, notifications, polls, proxies

       •Voting: asynchronous voting period (e.g., 7 days)

       •Post-assembly: results, reports, timeline publication

       •Infrastructure: Standard web stack, no real-time requirements

       •Complexity: Medium

       •Timeline: 2-3 sprints

Option B: Live Assembly (Full Scope)

       •All features from Option A, plus:

       •Real-time check-in (QR code, terminal)

       •Live voting with real-time results

       •Telão (big screen) display

       •Speaker queue management

       •Live quorum tracking

       •Infrastructure: WebSocket server, real-time database, high availability

       •Complexity: High

       •Timeline: 5-7 sprints

Required Decision: Which option to implement? When to implement (MVP or Phase 2)?



Decision 2: Plan Matrix per Persona

Context: Current plan structure (trial, base, plus, pro) needs detailed feature mapping per persona.
Required Information:

       •Which features are available in each plan for sindico?

       •Which features are available in each plan for morador?

       •Which features are available in each plan for empresa?

       •Which features are available in each plan for agencia?

       •What are the quotas per plan (Connect Me requests, video uploads, storage)?



Decision 3: Onboarding Flows

Context: Each persona has different onboarding requirements.

Required Information:

       •Detailed onboarding steps for sindico (with/without empresa_sindica)

       •Detailed onboarding steps for morador (with/without condominium pre-registration)

       •Detailed onboarding steps for empresa (with/without agencia)

       •Detailed onboarding steps for agencia

       •Which steps are mandatory vs optional?

       •Which steps can be skipped and completed later?



Decision 4: Paywall Implementation

Context: Need to define how features are blocked when plan limits are reached.

Required Information:

       •Hard block vs soft block (warning + grace period)?

       •Which features are completely blocked vs degraded?

       •How to handle retroactive blocking when subscription expires?

       •Grace period duration before blocking?



Decision 5: Connect Me Scope

Context: Original documents mention additional Connect Me flows.
Required Information:

       •Morador → Sindico Connect Me: Still in scope?

       •Empresa → Empresa Connect Me: Still in scope?

       •If yes, what are the use cases and rules?



Decision 6: Banco de Talentos / Vídeo-Currículo

Context: Mentioned in original documents but not detailed.

Required Information:

       •Still in scope?

       •If yes, what are the requirements?

       •Which personas can access?

       •How does it integrate with other modules?



Decision 7: LMS (Learning Management System)

Context: Mentioned in architecture document but not detailed.

Required Information:

       •Still in scope?

       •If yes, what are the requirements?

       •Which personas can access?

       •Course creation, enrollment, progress tracking?



Decision 8: Admin Panel (Master Síndico)

Context: Platform needs admin panel for Master Síndico team.

Required Information:

       •What features are needed in admin panel?

       •User management, tenant management, billing management?
       •Analytics and reporting?

       •Support tools?



Decision 9: Vizinhança/Comércio Local

Context: Parceiro da Vizinhança is partially defined, but full scope unclear.

Required Information:

       •Is this a separate persona with login?

       •Or just a directory managed by sindico?

       •Payment integration for partner services?

       •Commission model?



Decision 10: Mobile App Scope

Context: Mobile apps (iOS/Android) are mentioned but scope is unclear.

Required Information:

       •Which features are available on mobile?

       •Mobile-first features (push notifications, camera, QR scanner)?

       •Offline capabilities?

       •Which personas have mobile access?




Technical Stack Summary


Frontend

       •Framework: SolidJS ^1.9.10

       •Build: Rsbuild ^1.7.1

       •Router: @tanstack/solid-router ^1.157.16

       •Data Fetching: @tanstack/solid-query ^5.90.23
     •Forms: @tanstack/solid-form + zod-adapter

     •Components: @kobalte/core ^0.13.11

     •CSS: UnoCSS

     •Validation: Zod ^4.3.6

     •HTTP: Axios ^1.13.4

     •TypeScript ^5.9.3



Design System (IMMUTABLE)

     •Primary: oklch(0.715 0.120 84.2) - Gold CTA

     •Secondary: oklch(0.218 0.055 256.1) - Navy

     •Accent: oklch(0.871 0.129 82.0) - Gold Light

     •Surface: oklch(0.218 0.055 256.1) - Navy

     •Typography: Manrope (body), Michroma (headings)

     •Border-radius: 0.5rem



Backend (To Be Defined)

     •Architecture: Erlang OTP + DNS Architecture pattern

     •Language: Node.js/TypeScript or Elixir (to be decided)

     •Database: PostgreSQL (recommended for multi-tenancy)

     •Cache: Redis

     •Queue: Bull/BullMQ or RabbitMQ

     •Event Bus: Internal or external (Kafka, RabbitMQ, AWS EventBridge)



External Services

     •Video Streaming: Mux or Cloudflare Stream

     •Payment: Mercado Pago

     •Email: AWS SES or SendGrid

     •SMS: Twilio or Zenvia
     •Storage: AWS S3 or equivalent

     •Push Notifications: Firebase Cloud Messaging

     •CEP Lookup: ViaCEP

     •Error Tracking: Sentry or Rollbar

     •APM: New Relic or Datadog

     •Logging: CloudWatch or Datadog



Mobile (To Be Defined)

     •iOS: Swift or React Native

     •Android: Kotlin or React Native




Enums and Master Lists


Condominium Types (6 items)

     1.residencial

     2.comercial

     3.misto

     4.shopping_center

     5.galeria_comercial

     6.edificio_garagem



Mandate Types (3 items)

     1.sindico_eleito

     2.sindico_profissional

     3.sindico_interino
Mandate End Reasons (4 items)

     1.fim_periodo

     2.renuncia

     3.destituicao

     4.transferencia



Area Impactada (26 items)

     1.estrutura_predial

     2.sistema_hidraulico

     3.sistema_eletrico

     4.sistema_incendio

     5.reservatorios

     6.elevadores

     7.garagem

     8.fachada

     9.cobertura

     10.seguranca_patrimonial

     11.administracao

     12.portaria

     13.playground

     14.academia

     15.salao_festas

     16.corredores

     17.jardins

     18.casa_maquinas

     19.rede_esgoto

     20.rede_drenagem

     21.sistema_gas
     22.iluminacao

     23.sistema_cameras

     24.controle_acesso

     25.areas_comuns

     26.outros



Tipos de Atividade (26 items)

     1.manutencao_preventiva

     2.manutencao_corretiva

     3.inspecao_tecnica

     4.vistoria_tecnica

     5.contratacao_servico

     6.execucao_servico

     7.reparo_emergencial

     8.obra_melhoria

     9.adequacao_normativa

     10.atualizacao_infraestrutura

     11.monitoramento_ambiental

     12.revisao_tecnica

     13.auditoria_tecnica

     14.limpeza_tecnica

     15.controle_pragas

     16.limpeza_reservatorios

     17.treinamento_tecnico

     18.adequacao_legal

     19.revisao_equipamentos

     20.atualizacao_administrativa

     21.contratacao_fornecedor

     22.encerramento_contrato
     23.avaliacao_fornecedor

     24.implantacao_sistema

     25.atualizacao_normas

     26.acao_emergencial



Risco Associado (9 items)

     1.sem_risco

     2.risco_operacional

     3.risco_estrutural

     4.risco_sanitario

     5.risco_eletrico

     6.risco_hidraulico

     7.risco_incendio

     8.risco_juridico

     9.risco_ambiental



Próxima Ação (8 items)

     1.monitorar_evolucao

     2.nova_inspecao

     3.contratar_fornecedor

     4.executar_manutencao

     5.atualizar_plano_diretor

     6.registrar_conclusao

     7.acompanhar_garantia

     8.reavaliar_necessidade



Status do Plano Diretor (7 items)

     1.planejada
     2.em_contratacao

     3.em_execucao

     4.concluida

     5.suspensa

     6.reprogramada

     7.atrasada



Tipos de Evento (13 items)

     1.assembleia_ordinaria

     2.assembleia_extraordinaria

     3.manutencao_programada

     4.inspecao_tecnica

     5.obra_programada

     6.interrupcao_programada_servicos

     7.treinamento_equipe

     8.campanha_institucional

     9.evento_comunitario

     10.reuniao_administrativa

     11.atualizacao_normativa

     12.simulado_emergencia

     13.outros



Tipos de Comunicado (8 items)

     1.aviso_operacional

     2.interrupcao_programada

     3.orientacao_moradores

     4.aviso_seguranca

     5.comunicado_institucional
     6.conclusao_servico

     7.recomendacao_manutencao

     8.alerta_preventivo



Tipos de Vídeo Institucional (12 items)

     1.apresentacao_institucional

     2.apresentacao_equipe_tecnica

     3.demonstracao_servico

     4.explicacao_tecnica

     5.boas_praticas_condominiais

     6.orientacao_preventiva

     7.caso_real_atendimento

     8.treinamento_tecnico

     9.explicacao_legislacao_normas

     10.dicas_manutencao

     11.conteudo_educativo_sindicos

     12.conteudo_educativo_moradores



Timeline Entry Types (6 items)

     1.master_plan_action

     2.event

     3.comunicado

     4.closed_deal

     5.service_execution

     6.assembly_result



User Roles (6 items)

     1.sindico
     2.morador_titular

     3.morador_dependente

     4.empresa_administrador

     5.empresa_operador

     6.agencia



Plans (4 items)

     1.trial (7 days, full Pro access)

     2.base

     3.plus

     4.pro



Assembly Types (2 items)

     1.ordinaria

     2.extraordinaria



Quorum Types (3 items)

     1.simple_majority

     2.qualified_majority

     3.unanimous



Vote Options (3 items)

     1.favor

     2.contra

     3.abstencao
Compliance Status (3 items)

       1.em_dia

       2.pendente

       3.atrasado



Evaluation Criteria (Morador → Sindico) (5 items)

       1.comunicacao

       2.transparencia

       3.manutencao

       4.financeiro

       5.atendimento



Evaluation Criteria (Sindico → Empresa) (5 items)

       1.qualidade_servico

       2.cumprimento_prazo

       3.atendimento

       4.custo_beneficio

       5.recomendacao




Document Status

Status: Requirements Phase Complete - Pending User Review

Next Steps:

       1.User reviews requirements document

       2.User provides feedback or requests changes

       3.User clicks "Proceed to Design" button to move to next phase

       4.If user responds with "Skip to Implementation Plan", proceed directly to task creation
Coverage:

      •✅    92 detailed requirements across all domains

      •✅    All 4 HIGH-DOMAINS covered (Identity, Institutional, Commercial, Content)

      •✅    Cross-domain requirements (Compliance, Analytics, Assembly, Integrations, Security,
      Performance, Testing, DevOps, Accessibility)

      •✅    All enums and master lists documented

      •✅    Technical stack defined

      •✅    10 critical pending decisions documented

      •✅    EARS patterns followed for all requirements

      •✅    INCOSE quality rules applied

Pending:

      •🔴    10 critical decisions requiring client input

      •🔴    Plan matrix per persona

      •🔴    Detailed onboarding flows

      •🔴    Assembly module scope decision (Live vs Async)

      •🔴    Mobile app detailed scope

      •🔴    Admin panel requirements
