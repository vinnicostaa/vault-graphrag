---
title: Reconciliação — Enums + Regras canônicas + 40 Quebras detectadas
type: note
tags: [reconciliation, enums, regras-canonicas, quebras, decisoes-pendentes, master-sindico, v2, fase-7]
source: Content/04-Listas-Mestre/Enums-Consolidados + enterprise-catergories-*.schema.ts + Content/regras-de-negocio/{regras-canonicas,quebras-detectadas} + handoffs
created: 2026-04-23
updated: 2026-04-23
---

# Reconciliação — Enums Canônicos + Regras + Quebras Detectadas

> **Origem**: Agent Explore (Fase 7f) leu `Enums-Consolidados.md` (687 linhas, 90+ enums em 19 categorias), `enterprise-catergories-subcategories-enum.schema.ts` (832 linhas, 32 categorias + 202 subcategorias), `regras-canonicas.md` (10 regras + 12 decisões + T1-T15 + stack), `quebras-detectadas.md` (40 quebras), `sdd-gsd-aplicado.md`, handoffs.

---

## § 1. Inventário — 19 categorias de Enums (90+ total)

### Categoria 1: Identidade e Acesso

- `Role` — 6 valores: `syndic | resident | enterprise | marketing | local_company | admin` (T7, inglês, Zitadel-native)
- `RoleType` — sub-contexto operacional (banco local, não Zitadel): `principal | auxiliary | assistant | titular | dependent | administrator | operator`
- `UserStatus` — `pending_verification | active | suspended | deleted`

### Categoria 2: Bilhetagem e Planos

- `PlanTier` — **4 valores canônicos**: `trial | base | plus | pro`
- `PlanSlug` — 8 slugs: `resident_base | resident_paid | syndic_n1 | syndic_n2 | syndic_n3 | enterprise_plus | enterprise_pro | marketing_standard | local_company_standard`
- `SubscriptionStatus` — `trial | active | past_due | canceled | paused | expired`
- `QuotaKey` — feature keys (connect_me, video_upload_institutional, video_upload_curriculum, live_session_minutes, etc.)
- `QuotaPeriod` — `monthly | annual | once | never`

### Categoria 3: Condomínio

- `CondominiumType` — 6 tipos: `residencial | comercial | misto | shopping_center | galeria_comercial | edificio_garagem`
- `MandateType` — 3: `sindico_eleito | sindico_profissional | sindico_interino`
- `MandateStatus` — `ativo | encerrado | transferido`
- `MandateTerminationReason` — 4: `fim_periodo | renuncia | destituicao | transferencia`

### Categoria 4: Unidade e Vínculo

- `UnitType` — 3 canônicos ou 6 expandidos: `apartamento | casa | sala_comercial | loja | garagem | outro` (v2 expandido; v1 só 3)
- `MembershipType` — 3: `proprietario_residente | proprietario_nao_residente | inquilino`
- `UnitStatus` — 3: `ocupada_proprietario | ocupada_inquilino | unidade_vazia`
- `ResidentRole` — 2: `titular | dependente`
- `MembershipTerminationReason` — 4: `mudanca_definitiva | venda_unidade | encerramento_contrato_locacao | erro_cadastro`

### Categoria 5: Plano Diretor

- `ImpactedArea` — **26** valores (estrutura_predial, sistema_hidraulico, sistema_eletrico, sistema_incendio, reservatorios, elevadores, garagem, fachada, cobertura, seguranca_patrimonial, administracao, portaria, playground, academia, salao_festas, corredores, jardins, casa_maquinas, rede_esgoto, rede_drenagem, sistema_gas, iluminacao, sistema_cameras, controle_acesso, areas_comuns, outros)
- `ActivityType` — **26** valores (manutencao_preventiva, manutencao_corretiva, inspecao_tecnica, vistoria_tecnica, contratacao_servico, execucao_servico, reparo_emergencial, obra_melhoria, adequacao_normativa, atualizacao_infraestrutura, monitoramento_ambiental, revisao_tecnica, auditoria_tecnica, limpeza_tecnica, controle_pragas, limpeza_reservatorios, treinamento_tecnico, adequacao_legal, revisao_equipamentos, atualizacao_administrativa, contratacao_fornecedor, encerramento_contrato, avaliacao_fornecedor, implantacao_sistema, atualizacao_normas, acao_emergencial)
- `RiskType` — 9 valores (sem_risco, risco_operacional, risco_estrutural, risco_sanitario, risco_eletrico, risco_hidraulico, risco_incendio, risco_juridico, risco_ambiental)
- `NextAction` — 8 valores
- `MasterPlanStatus` — 7: `planejada | em_contratacao | em_execucao | concluida | suspensa | reprogramada | atrasada`
- `Importance` — 5: `baixa | moderada | alta | critica | emergencial`
- `ExpectedImpact` — 8 valores

### Categoria 6: Timeline

- `TimelineEntryType` — 7: `atividade_da_gestao | registro_de_execucao | comunicado | evento | adendo | resultado_consulta_moradores | assembly_result`
- `TimelineVisibility` — 2: `gestao | publico_moradores`

### Categoria 7: Eventos

- `EventType` — 13 valores (assembleia_ordinaria, assembleia_extraordinaria, manutencao_programada, inspecao_tecnica, obra_programada, interrupcao_programada_servicos, treinamento_equipe, campanha_institucional, evento_comunitario, reuniao_administrativa, atualizacao_normativa, simulado_emergencia, outros)
- `EventStatus` — `scheduled | in_progress | completed | cancelled`
- `EventRSVP` — 3: `ciente | confirmado | nao_confirmado`

### Categoria 8: Comunicados

- `AnnouncementType` — **8 canônicos**: `aviso_operacional | interrupcao_programada | orientacao_moradores | aviso_seguranca | comunicado_institucional | conclusao_servico | recomendacao_manutencao | alerta_preventivo`
- `PostDealCommunicationType` — 5: `orientacao_tecnica | recomendacao_manutencao | alerta_preventivo | conclusao_servico | atualizacao_garantia`
- `AnnouncementPriority` — `low | medium | high | urgent`

### Categoria 9: Consultas/Enquetes

- `PollType` — 3: `sim_nao_nao_sei | multiple_choice | scale_1_to_5` (MVP pré-assembleia: só `sim_nao`)

### Categoria 10: Assembleia

- `AssemblyType` — 4: `ordinaria | extraordinaria | implantacao | ratificacao`
- `AssemblyStatus` — 5: `em_preparacao | publicada | em_andamento | encerrada | cancelada`
- `AssemblyModality` — 3: `presencial | hibrida | online` (MVP só presencial)
- `AgendaItemType` — 5: `informativo | votacao_simples | votacao_fracao_ideal | escolha_fornecedor | eleicao`
- `AgendaItemStatus` — 4: `resolvido | adiado | nao_resolvido | prejudicado`
- `QuorumType` — 3: `simple_majority | qualified_majority | unanimous`
- `VoteChoice` — binário: `favor | contra | abstencao`; escala: `escala_1_a_5 | abstencao`
- `VoteOrigin` — 3: `app | presencial_assistido | proxy`
- `CheckInMode` — 3: `app | qr_code | terminal_presencial`
- `AttendanceStatus` — 3: `presente | ausente_momentaneo | desconectado_saiu`
- `AssemblyParticipationIntent` — 4
- `SpeechDuration` — 2: `2_minutos | 3_minutos`
- `DefaultStatus` — 2: `apto_a_votar | inapto_a_votar`
- `ContingencyReason` — 3: `internet_failure | device_failure | digital_voting_unavailable`
- `HomologationStatus` — 3: `pending | approved | rejected`

### Categoria 11: Connect Me

- `ConnectMeFluxo` — 4: `sindico_empresa | morador_sindico | empresa_empresa | sindico_parceiro`
- `ConnectMeStatus` — 6: `open | interested | proposal_sent | em_negociacao | closed | expired`
- `DeclinationReason` — 6 (fora_area_atendimento, servico_fora_especialidade, agenda_indisponivel, conflito_servicos_agendados, orcamento_incompativel, outro_motivo)
- `MoradorSindicoConnectMeCategory` — 5 (duvida_gestao, solicitacao_manutencao, reclamacao, sugestao, outro)
- `Urgency` — 4: `baixa | media | alta | urgente`
- `PartnershipType` — 5 (subcontratacao, parceria_tecnica, fornecimento_materiais, indicacao_servicos, outro)
- `PromotionType` — 5

### Categoria 12: Propostas/Deals/Execução

- `ProposalStatus` — 6: `draft | sent | awaiting_validation | validated | rejected | accepted`
- `ClosedDealStatus` — 5: `draft | awaiting_company_signature | awaiting_sindico_signature | confirmed | cancelled`
- `ExecutionStatus` — 5: `draft | sent | awaiting_validation | validated | rejected`
- `SubmissionStatus` — 7 (rascunho, enviado, aguardando_validacao, solicitado_ajuste, aprovado, publicado, rejeitado)
- `RiskClassification` — 4: `baixo | moderado | elevado | critico`
- `ServiceNature` — 4: `preventiva | corretiva | emergencial | estrategica`
- `ServiceStatus` — 3
- `CompanyUserType` — 2: `administrador | operador_tecnico`
- `EvaluationCriteria` — 5 (qualidade_servico, cumprimento_prazo, atendimento, custo_beneficio, recomendacao)
- `GovernanceEvaluationCriteria` — 3 (satisfacao_gestao, transparencia_comunicacao, percepcao_evolucao; escala 1-10, reset bimestral)

### Categoria 13: Vizinhança

- `PartnerCategory` — **38 em 12 grupos** (alimentação 9, mercado 3, saúde 1, pet 3, academia 4, estética 3, serviços domésticos 3, profissionais 5, assistência técnica 2, automotivo 2, cursos 2, outros 1)
- `BenefitType` — 7
- `BenefitScope` — 2: `aberta_bairro | exclusiva_condominio`
- `PartnerInvitationStatus` — 3
- `CouponCode` (VO) — formato `ID_LOJA(5) + SEQ(3) + PALAVRA(5)` = 13 chars; trava 4h por estabelecimento. **⚠️ Q23: possivelmente removido**, confirmar com João.

### Categoria 14: Vídeo e Content

- `VideoType` — **12 tipos** (apresentacao_institucional, apresentacao_equipe_tecnica, demonstracao_servico, explicacao_tecnica, boas_praticas_condominiais, orientacao_preventiva, caso_real_atendimento, treinamento_tecnico, explicacao_legislacao_normas, dicas_manutencao, conteudo_educativo_sindicos, conteudo_educativo_moradores)
- `VideoState` — v1: 5 estados; v2 expandido: 9 (uploading, processing, moderation, approved, published, locked, unlocked, removed, rejected)
- `TokenScope` — agência: `videos:upload | videos:edit | analytics:read`

### Categoria 15: LMS/Academia

- `CourseCategory` — 10 (gestao_condominial, saude_ambiental, seguranca_predial, manutencao_predial, legislacao_condominial, mediacao_conflitos, tecnologia_condominios, sustentabilidade, administracao_financeira, outros)
- `CourseLevel` — 3: `basico | intermediario | avancado`
- `CourseAccessLevel` — 2: `gratuito | pago`
- `ForumCategory` — 7
- `LibraryMaterialType` — 6 (videos, guias_tecnicos, manuais, livros_digitais, ebooks, materiais_tecnicos)
- `LiveType` — 4 (debates_condominiais, lancamento_solucoes, palestras_tecnicas, entrevistas_especialistas)
- `LiveAccessLevel` — 2: `abertas | exclusivas`
- `CourseTrilha` — 3: `trilha_sindico | trilha_empresa | trilha_morador`

### Categoria 16: Banco de Talentos

- `TalentArea` — **18 áreas** (operacoes_condominiais, limpeza_conservacao_higienizacao, manutencao_predial, seguranca_patrimonial, obras_reformas_adequacoes, administracao_apoio_administrativo, atendimento_relacionamento_suporte, logistica_almoxarifado_suprimentos, gestao_coordenacao_lideranca, treinamento_qualidade_processos_compliance, tecnologia_sistemas_suporte_digital, comercial_vendas_parcerias, comunicacao_marketing_conteudo, financeiro_contabil_controladoria, juridico_contratos_compliance_legal, recursos_humanos_gestao_pessoas, apoio_operacional_externo, outras_areas)
- `ContractModality` — 9 (clt, pj, estagio, temporario, folguista, diarista, freelancer, intermitente, outros)
- `AvailabilitySchedule` — 4: `diurno | noturno | escala | finais_semana`
- `StartAvailability` — 3: `imediato | ate_15_dias | a_combinar`
- `TravelTime` — 3
- `CNHType` — 4: `nao | A | B | AB`
- `NRCertification` — 4: `NR-10 | NR-33 | NR-35 | outros`
- `EmploymentExitReason` — 4
- **Q39 pendente**: 3 vs 5 vínculos últimos (PDF diz 5; adotar 5 conforme Decision 6)

### Categoria 17: Marketing Link

- `MarketingSupportType` — 6
- `SupportPriority` — 3: `imediato | proximos_30_dias | sem_prazo_definido`
- `MarketingLinkStatus` — 6

### Categoria 18: Compliance

- `ComplianceStatus` — 3: `em_dia | parcial | pendente`
- `ComplianceRiskType` — 8 (operacional, financeiro, juridico, estrutural, reputacional, sanitario, trabalhista, tecnologia_seguranca_informacao)
- `ComplianceRiskLevel` — 4: `baixo | medio | alto | critico`
- `ComplianceCheckStatus` — 3
- `SignatureStatus` — 2: `assinado | pendente`
- **Gatilhos que tornam Compliance obrigatório** — 4: `encerrar_mandato | gerar_relatorio_final | marcar_negocio_fechado | exportar_dossie`

### Categoria 19: Sistema

- `TenantType` — 2: `condominium | company`
- `AuditAction` — 15+ (user_registered, user_authenticated, subscription_created, subscription_expired, condominium_created, timeline_published, master_plan_action_created, connect_me_request_created, connect_me_interest_shown, deal_confirmed, execution_validated, adendo_created, assembly_started, vote_cast, vote_homologated, assembly_finalized, declaration_submitted, signature_completed, conflict_declared, data_revealed, cross_tenant_access)
- `LogLevel` — 5: `debug | info | warn | error | fatal`

---

## § 2. Schema de Categorias de Empresa — 32 categorias + 202 subcategorias

**Fonte**: `enterprise-catergories-subcategories-enum.schema.ts` (832 linhas, Zod schema completo)

### 32 Categorias principais:

1. ADMINISTRACAO_GESTAO_GOVERNANCA (6 sub)
2. JURIDICO_DIREITO_IMOBILIARIO (6)
3. CONTABILIDADE_AUDITORIA_FINANCAS (8)
4. RH_PESSOAS_SAUDE_OCUPACIONAL (6)
5. CURSOS_EDUCACAO_DESENVOLVIMENTO (4)
6. CURSOS_ORATORIA_COMUNICACAO (4)
7. SEGUROS (5)
8. LAUDOS_AUTOVISTORIA_MANUTENCAO_PREDIAL (6)
9. SAUDE_AMBIENTAL (4)
10. ELETRICA_ENERGIA_SPDA (9)
11. HIDRAULICA_SANEAMENTO (9)
12. MEDICAO_INDIVIDUALIZACAO_CONSUMO (5)
13. GAS (6)
14. ELEVADORES_TRANSPORTE_VERTICAL (5)
15. CLIMATIZACAO_VENTILACAO_EXAUSTAO (5)
16. PREVENCAO_COMBATE_INCENDIOS (7)
17. SEGURANCA_CONTROLE_ACESSO (7)
18. LIMPEZA_ESPECIALIZADA (5)
19. TERCEIRIZACAO_SERVICOS (10)
20. JARDINS_AREAS_VERDES (5)
21. LAZER_ESPORTE_BEM_ESTAR (6)
22. ARQUITETURA_DESIGN (5)
23. ENGENHARIA (12)
24. CONSTRUCAO_REFORMAS_MANUTENCAO_ESTRUTURAL (11)
25. MANUTENCAO_CONSERVACAO_FACHADAS (8)
26. GARAGENS_MOBILIDADE_ACESSOS (5)
27. TECNOLOGIA_SISTEMAS_CERTIFICACOES (8)
28. SUSTENTABILIDADE_ESG (5)
29. MERCADO_CONDOMINIOS (7)
30. MATERIAIS_PRODUTOS_SUPRIMENTOS (10)
31. CONVENIENCIA_COMERCIO_LOCAL_COMUNIDADE (5)

### Taxonomia exemplo (ADMINISTRACAO_GESTAO_GOVERNANCA):
```
├── ADMINISTRADORAS_CONDOMINIOS
├── CONSULTORIA_GESTAO_CONDOMINIAL
├── GOVERNANCA_CONDOMINIAL
├── LGPD_PROTECAO_DADOS
├── COMPLIANCE_CONDOMINIAL
└── MEDIACAO_CONFLITOS
```

---

## § 3. 10 Regras Canônicas (RC-1..RC-10, invioláveis)

| # | Regra | Status |
|---|---|---|
| **RC-1** | Separação de identidade — nunca misture User, Profile e Subscription | ✅ |
| **RC-2** | Imutabilidade — Timeline, Deal Fechado, votos homologados nunca editados (só adendo Req 25) | ✅ |
| **RC-3** | Connect Me ≠ Chat — formulário unidirecional, sem reply/thread/histórico | ✅ |
| **RC-4** | Visibilidade vídeo empresa — moradores só durante votação (flag + grants + revogação automática via NATS) | ✅ |
| **RC-5** | Métricas privadas — likes/comentários só ao dono, sem ranking público | ✅ |
| **RC-6** | Backend como verdade única — toda regra de negócio na API (frontend é UI) | ✅ |
| **RC-7** | Trava trimestral — vídeos institucionais e vídeo-currículo: 1 atualização/90 dias | ✅ |
| **RC-8** | 1 dispositivo por conta — fingerprint + IP, login 2º device invalida sessão anterior | ✅ |
| **RC-9** | Tenant isolation — cross-tenant só via grants explícitos (`WHERE condominium_id = $tenantID`) | ✅ |
| **RC-10** | LGPD by design — log obrigatório de revelação (`lgpd_logs` append-only, retenção 5 anos) | ✅ |

---

## § 4. 12 Decisões Resolvidas (D1-D12, não revisitar)

| # | Decisão | Status |
|---|---|---|
| D1 | Assembleia Live + WebSocket + Telão (MVP só presencial; online/híbrida inativa) | ✅ |
| D2 | Planos: `trial/base/plus/pro` | ✅ |
| D3 | Onboarding por persona (Morador 4, Síndico 6, Empresa 7, Parceiro 4) | ✅ |
| D4 | Paywall: **soft block**, sem grace period | ✅ |
| D5 | Connect Me 4 direções: S→E, M→S, E→E (Plus/Pro), S→Parceiro | ✅ |
| D6 | Banco Talentos in-scope; 11 telas morador, vídeo 90s lock 90d | ✅ |
| D7 | Academia LMS in-app completa; parceiro SEM acesso | ✅ |
| D8 | Painel Admin MS in-scope | ✅ |
| D9 | Connect Me M→S unidirecional FRIO (sem reply) | ✅ |
| D10 | Quotas Connect Me ANUAL: MorBase 2 · MorPag 4 · SindN1 2 · N2 4 · N3 ∞ | ✅ |
| D11 | Limite 15 condomínios por síndico | ✅ |
| D12 | Parceiro Vizinhança persona com login; CPF ou CNPJ aceitos | ✅ |

---

## § 5. T1-T15 Decisões Técnicas (invariantes implementadas)

| # | Decisão |
|---|---|
| T1 | `AuthUser` em `internal/shared/types/` |
| T2 | Gin abstraído via `contracts.HTTPRouter` + `contracts.Context` |
| T3 | `AppError{Code, Message, Err}` + sentinels + `errors.Is/As` |
| T4 | UoW no `context.Context` via chave privada `txKey{}` + `database.DBFromContext(ctx, pool)` |
| T5 | goose v3 + `embed.FS` |
| T6 | zerolog wrapper interface `Logger` |
| T7 | Roles primárias inglês: `syndic, resident, enterprise, marketing, local_company, admin` |
| T8 | Role types sub-contexto operacional **só no banco** |
| T9 | Cookie `access_token` httpOnly, `Domain=.mastersindico.com.br`, Secure, SameSite=Lax |
| T10 | Money `INTEGER` centavos; `int64` Go; fração ideal `NUMERIC(5,4)` |
| T11 | OIDC Authorization Code + PKCE via `zitadel/oidc/v3` |
| T12 | Rotas públicas auth FORA de `/api/v1` |
| T13 | Cache introspection Redis `ms:auth:{zitadelUserID}` TTL 5min |
| T14 | Mobile: fallback `Authorization: Bearer` (RFC 6750) |
| T15 | Guard sem role → `403 NO_ROLE_ASSIGNED` antes do UPSERT |

---

## § 6. Stack Atual Confirmada (2026-04-22)

| Camada | Tecnologia |
|---|---|
| Backend | Go 1.26 + Gin abstraído (Clean Arch + DDD + CQRS + Vertical Slices) |
| Web | SolidJS · TanStack Router/Query/Form · Rsbuild · UnoCSS · Kobalte · Axios · Zod · Biome (Bun) |
| Mobile | Flutter/Dart FVM, Clean Arch, get_it+injectable, go_router |
| Auth | Zitadel v4 (OIDC PKCE) |
| Billing | Stripe (`IPaymentGateway`) |
| Vídeo VOD | Mux (`IVideoProvider`) |
| Vídeo Live | Livekit (`ILiveProvider`) |
| Email | **Resend** (não AWS SES ⚠️) |
| SMS | Twilio (planejado) |
| Storage | MinIO dev / R2 ou S3 prod (D-029) |
| Busca | **PostgreSQL tsvector** (não OpenSearch ⚠️) |
| DB | PostgreSQL 18 + pgx/v5 + sqlc v1.30 |
| Cache | Redis 7 |
| Mensageria | NATS JetStream |
| Migrations | goose v3 (embed.FS) |
| Observabilidade | OTel + Prometheus (Sprint 9) |
| Deploy | **Railway** (não AWS ECS ⚠️) |
| CI | GitHub Actions |

---

## § 7. 40 Quebras Detectadas

### 🔴 CRITICAL (10)

| # | Título | Fix |
|---|---|---|
| **Q1** | Stack desatualizada (SES/OpenSearch/ECS) | 3 substituições em `product-overview.md` |
| **Q2** | Quota Morador Base Connect Me diverge: 2/ano vs 0/ano | Adotar Decision 10 (Base 2/ano); **Bloqueia ABAC consistente** |
| **Q3** | Empresa "Base" mencionada onde não existe | Trocar "Base" por "trial/soft-block" em Req 18 |
| **Q4** | Agência: "sem login" vs role `marketing` no Zitadel | Clarificar: tem login, delegado via scoped token |
| **Q5** | Síndico N1 + certificado LMS: spec diverge | Alinhar com matriz (N2+ certificado) |
| **Q19** | Roles pt-BR (sindico, morador_titular...) vs inglês (T7) | Note no topo arquivo: T7 canônico |
| **Q20** | "agencia" vs "marketing"; sem "local_company" | Subconjunto de Q19 |
| **Q21** | Mercado Pago vs Stripe | Marcar docs antigos stale |
| **Q22** | bcrypt/Argon2 vs Zitadel-only | Arqueologia |
| **Q23** | **Sistema de Cupons (Promoção do Dia) DESAPARECEU** | **⚠️ Confirmar com João**: cancelado/substituído/pendente |

### 🟡 MEDIUM (15)

| # | Título | Severity |
|---|---|---|
| Q6 | Assembly marked PENDENTE quando Decision 1 resolveu | Doc stale |
| Q7 | master-sindico-domain-mapping.md desatualizado | Doc stale |
| Q8 | Connect Me E→E: 3 redações diferentes | Padronizar |
| Q9 | Vídeos institucionais: matriz mensal + trava trimestral não cruzadas | Nota explícita |
| Q10 | Visibilidade "Base" em matriz Empresa | Decorre Q3 |
| Q11 | deploy-config diz Railway, product-overview diz AWS | Decorre Q1 |
| Q12 | obsidian-mapping cita vault externo stale | Deprecar |
| Q13 | excopo-tecnico-antigo.txt + main.go.md sem classificação | Mover `_inbox/` |
| Q24 | Cardinalidade Tipos de Comunicado (8 vs 13+) | Adotar 8; subtipos em campo texto |
| Q25 | Score Governança 0-100 vs 1-10 + Compliance 0-100 | Alinhar com João |
| Q26 | 4 Sprints contratuais vs 9 atuais | Formalizar aditivo |
| Q27 | Assembleia ASSÍNCRONA (contrato) vs LIVE com WebSocket (atual) | Formalizar aditivo |
| Q28 | Lives restritas Admin MS vs Lives no LMS | Adicionar regra Req 98.1 |
| Q29 | Limite 12 (PDF) vs 15 (Decision 11) | Spec interna vence |
| Q30 | Comunicados pós-deal: 5 vs 7+ tipos | Padronizar |

### 🟢 LOW (15)

Q14-Q18 e Q31-Q40: divergências cosméticas ou confirmações já OK.

---

## § 8. 10 Decisões Pendentes com João (consolidadas)

| Prior | ID | Tema | Status |
|---|---|---|---|
| **P0** | Q2 | Quota Morador Base Connect Me: 2/ano ou 0? | Bloqueia ABAC |
| **P0** | Q23 | Sistema Cupons: in-scope/descartado/substituído? | Gap contratual |
| P1 | Q4 | Agência tem login? role marketing ou só token delegado? | Abre D-### |
| P1 | Q5 | Síndico N1 certificado LMS? (provavelmente não) | Req 98.1 |
| P2 | Q26 | Aditivo contratual formalizado (4→9 sprints)? | Legal |
| P2 | Q27 | Live Assembly aditivado (assíncrono→live)? | Legal |
| P2 | Q25 | Score Governança: 1 ou 2 scores? | Design |
| P3 | Q28 | Lives no LMS admin-only? | Req 98.1 |
| P3 | Q39 | BT: 3 ou 5 vínculos? | Decision 6 (5) |
| P3 | Q40 | Eleição gera avaliação escondida? | Req 49.1 novo |

---

## § 9. Divergências v2 vs Canônico de Enums

### v2 é SUPERIOR (expandido):
- `UnitType`: v2 6 valores vs v1 3 — **revisar v1**
- `VideoState`: v2 9 estados vs v1 5 — **revisar v1**
- `AssemblyStatus`: v2 adiciona `homologated` — **revisar v1**

### v2 FALTA (expandir conforme v1):
- `MasterPlanActivity` enums (26 tipos + 26 áreas + 9 riscos) — não destilados em v2 ubiquitous-language
- `AuditAction` (15+ ações) — v2 tem outline só
- `PartnerCategory` (38 em 12 grupos) — v2 menciona mas não lista

### v2 FALTA CATEGORIAS INTEIRAS:
- 32 categorias empresa + 202 subcategorias (enterprise-catergories schema) — não referenciadas em v2

---

## § 10. TL;DR

- **90+ enums** em 19 categorias canônicas
- **32 categorias empresa + 202 subcategorias** em schema TS
- **10 regras canônicas** invioláveis (todas implementadas)
- **12 decisões resolvidas** (D1-D12)
- **15 decisões técnicas** (T1-T15) todas implementadas
- **40 quebras detectadas** (10 🔴 + 15 🟡 + 15 🟢)
- **10 decisões pendentes** com João (P0 bloqueadores: Q2 + Q23)
- Stack stale em v2: SES→Resend, OpenSearch→tsvector, AWS→Railway
- Enums que v2 precisa expandir: UnitType (feito), VideoState (feito), MasterPlanActivity, AuditAction, PartnerCategory, 32 categorias empresa
