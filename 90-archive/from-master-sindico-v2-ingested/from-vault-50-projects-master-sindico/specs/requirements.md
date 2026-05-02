# Requirements — Master Síndico (Nova Arquitetura)

> **⚠️ Em migração**: este arquivo monolítico é **fonte canônica atual** para os Reqs completos. Arquivos em `requirements/*.md` são recortes focados por domínio, criados incrementalmente. Priorize esta visão global quando precisar do escopo completo; use os recortes quando o escopo for específico. Quando todos os recortes estiverem completos, este arquivo será deprecated.
>
> **Recortes já criados**:
> - [`requirements/product-overview.md`](requirements/product-overview.md) — decisões 1-12 + T1-T15 + regras de ouro + marcos
> - [`requirements/personas-and-plans.md`](requirements/personas-and-plans.md) — personas × planos × trials × quotas
> - [`requirements/identity-mirror-pattern.md`](requirements/identity-mirror-pattern.md) — decisão User/Session ↔ Zitadel
> - [`requirements/README.md`](requirements/README.md) — índice completo


## Visão Geral

O Master Síndico é um ecossistema condominial completo — não um app único, mas múltiplos domínios integrados. Um mesmo CPF pode ser síndico no Condomínio X, morador no Y, e ter empresa cadastrada. A identidade é única; os contextos de operação são múltiplos.

**Stack confirmada:**
- **Backend:** Go (Gin) — Clean Architecture + DDD + CQRS, monolito modular por bounded context
- **Frontend Web:** SolidJS · TanStack Router/Query/Form · Rsbuild · UnoCSS · Axios · Kobalte
- **Mobile:** Dart (Flutter) — Clean Architecture
- **Auth/IAM:** Zitadel (OIDC + ABAC)
- **Billing:** Stripe (trial, subscriptions, Customer Portal)
- **Busca:** OpenSearch
- **Vídeo:** Mux (upload VOD, encoding, HLS streaming)
- **Conferência ao vivo:** Livekit (WebRTC, salas para assembleia online/híbrida)
- **Storage de objetos:** MinIO (self-hosted em Railway; Cloudflare R2 em prod)
- **Email transacional:** Resend (templates pt-BR via HTML)
- **SMS:** Twilio
- **Mensageria:** NATS JetStream (eventos assíncronos — Sprint 10+)
- **Banco de dados:** PostgreSQL 18 (pgx + sqlc + goose)
- **Cache:** Redis 7
- **Infra de deploy:** Railway (containers, PostgreSQL managed, Redis managed)

**Marcos de entrega:** Marco 1: 08/05/2026 · Marco 2: 20/06/2026 · Marco 3: 20/07/2026

---

## Decisões Resolvidas (fonte de verdade)

| # | Decisão | Resolução |
|---|---------|-----------|
| 1 | Escopo da Assembleia | Live Assembly com WebSocket real-time, Telão 2 áreas, fila de fala cronometrada. Opção online/híbrida inativa no MVP. |
| 2 | Níveis de plano | Trial · Base · Plus · Pro para todas as personas comerciais. Moradores: Base e Pagante. |
| 3 | Onboarding por persona | Sim — fluxos separados: Morador (4 telas), Síndico (6 telas), Empresa (7 telas), Parceiro (4 telas). |
| 4 | Paywall | Soft block — dados permanecem, acesso a funcionalidades premium bloqueado. Sem grace period. |
| 5 | Connect Me — direções | 4 fluxos: Síndico→Empresa, Morador→Síndico, Empresa→Empresa (Plus/Pro), Síndico→Parceiro. |
| 6 | Banco de Talentos | In-scope. 11 telas para morador, 7 funcionalidades para empresa. Vídeo-currículo 90s, lock 90 dias. |
| 7 | Academia LMS | In-app completa. 5 áreas, 3 trilhas. Parceiro da Vizinhança sem acesso. |
| 8 | Painel Admin MS | In-scope. Gestão global de tenants, moderação, financeiro, broadcasts. |
| 9 | Connect Me Morador→Síndico | Formulário unidirecional frio (sem reply, sem histórico). |
| 10 | Quotas Connect Me | Por **ano**. N1: 2/ano, N2: 4/ano, N3: ilimitado. Morador Base: 2/ano, Pagante: 4/ano. |
| 11 | Limite de condomínios | **15** por síndico. |
| 12 | Parceiro da Vizinhança | Persona com login. CPF aceito (CNPJ não obrigatório). |

---

## Regras de Ouro (invioláveis)

1. **Separação de identidade:** Nunca misture quem o usuário É (User) com o que ele PAGA (Subscription) com seus DADOS (Profile).
2. **Imutabilidade:** Timeline, Negócio Fechado confirmado, votos homologados — nunca editados, nunca deletados. Único mecanismo de modificação: adendo formal.
3. **Connect Me ≠ Chat:** Formulário unidirecional. Sem histórico, sem reply, sem chat.
4. **Visibilidade de vídeos de empresa:** Moradores só acessam durante votação de fornecedor em assembleia, se `autorizar_exibicao_votacao = true`. Acesso revogado após votação.
5. **Métricas privadas:** Likes e comentários visíveis apenas ao dono do conteúdo.
6. **Backend como verdade única:** Toda regra de negócio reside na API. Frontend é interface de consumo.
7. **Trava trimestral:** Vídeos institucionais (empresa) e vídeo-currículo (morador) — 1 atualização a cada 90 dias.
8. **1 dispositivo por conta:** Fingerprint + IP tracking. Login em segundo device invalida sessão anterior.
9. **Tenant isolation:** Usuários só acessam dados do seu tenant. Cross-tenant apenas via grants explícitos (Connect Me, validações).
10. **LGPD by design:** Dados revelados no Connect Me apenas após aceite. Log com timestamp, IP, finalidade, versão dos termos.

---

## Decisões Técnicas Implementadas (Sprint 0 + Sprint 1)

Estas decisões foram tomadas durante a implementação e são fonte de verdade para os sprints seguintes:

| # | Decisão | Resolução |
|---|---------|-----------|
| T1 | Localização do `AuthUser` | `internal/shared/types/` — `shared/middleware/` não pode importar de `modules/` |
| T2 | Framework HTTP | Gin, mas abstraído via `contracts.HTTPRouter` + `contracts.Context` — módulos não importam Gin |
| T3 | Error handling | `AppError{Code, Message, Err}` com sentinels + `errors.Is/As`; ErrorHandler global intercepta `c.Error()` |
| T4 | UnitOfWork | Tx injetada no `context.Context` via chave privada `txKey{}` — repositories extraem com `database.ExtractTx(ctx)` |
| T5 | Migrations | goose v3 com `embed.FS` — migrations dentro do binário; `database.Migrate(ctx, pool)` chamado no startup |
| T6 | Logger | zerolog wrapper com interface `Logger` — sem `SetGlobalLevel()` (mutação global quebra testes paralelos) |
| T7 | Roles no Zitadel | Roles primárias em inglês: `syndic`, `resident`, `enterprise`, `marketing`, `local_company`, `admin` |
| T8 | Role types | Sub-contexto operacional apenas no banco (`user_role_type` enum) — Zitadel não conhece role_type |
| T9 | Cookie auth | `access_token` httpOnly, `Domain=.mastersindico.com.br`, `Secure`, `SameSite=Lax` |
| T10 | Valores monetários | `INTEGER` centavos no PostgreSQL, `int64` em Go — nunca `float` |
| T11 | Fluxo OIDC | Authorization Code Flow com PKCE via `zitadel/oidc/v3` (`rp.NewRelyingPartyOIDC`) — não implicit flow |
| T12 | Rotas públicas de auth | `GET /auth/login`, `GET /auth/callback`, `GET /auth/logout` fora do grupo `/api/v1` (sem middleware Authenticate) |
| T13 | Cache de introspection | Redis `ms:auth:{zitadelUserID}` com TTL 5min — evita chamada ao Zitadel em cada request |
| T14 | Auth mobile | Fallback `Authorization: Bearer` (RFC 6750) além do cookie httpOnly — mobile não compartilha cookie de browser |
| T15 | Guard clause sem role | Tokens sem role retornam `403 NO_ROLE_ASSIGNED` antes do UPSERT — evita sincronizar usuário com `role=""` |

---

### Req 1 — Autenticação
- Authorization Code Flow com PKCE via Zitadel OIDC (implementado — T11)
- Refresh token gerenciado pelo Zitadel; backend valida via introspection com cache Redis TTL 5min (T13)
- Cookie `access_token` httpOnly no domínio raiz `.mastersindico.com.br`; fallback `Authorization: Bearer` para mobile (T9, T14)
- 1 dispositivo ativo por conta (fingerprint + IP)
- Rate limiting contra brute force
- Audit log de todas as tentativas (timestamp + IP)
- Anti-fraude: bloqueia múltiplas contas do mesmo IP
- OAuth Google e Apple: pós-lançamento (habilitação no painel Zitadel — sem desenvolvimento adicional)

### Req 2 — Autorização ABAC
- Engine própria (sem Casbin) — avalia `(userId, tenantId, role, roleType, planTier, action)` em cada request
- Implementada no middleware `RequirePermission(action, resource string)` — Sprint 1 (task 1.2)
- Retorna: `allowed` (boolean), `role`, `planTier`, features disponíveis
- Roles primárias (mapeadas 1:1 com Zitadel): `syndic`, `resident`, `enterprise`, `marketing`, `local_company`, `admin`
- Role types (sub-contexto operacional, lógica interna — não existe no Zitadel): `principal`, `auxiliary`, `assistant` (syndic) · `titular`, `dependent` (resident) · `administrator`, `operator` (enterprise)
- Cross-tenant via grants explícitos

**Cadeia de middleware BeyondCorp (ordem de execução):**
```
Recovery → RequestID → Logger → CORS → ErrorHandler
  → [rotas protegidas]:
      → RateLimiter (por IP + userId)
      → Authenticate (JWT → Zitadel introspection → cache Redis → user sync → 1 device check)
      → RequirePermission(action, resource)
  → Handler
  → AuditLogger (Sprint 3)
```

### Req 3 — Registro
- Suporta auto-registro: `syndic`, `resident`, `enterprise`, `local_company`
- Role `marketing` (Agência de Marketing) **não se registra diretamente** — é convidada por uma empresa via token (Sprint 5)
- Role `admin` é provisionada manualmente no Zitadel — sem fluxo de auto-registro
- Campos obrigatórios: `email`, `password`, `name`, `phone`, `role`
- Verificação de email obrigatória (responsabilidade do Zitadel)
- Hash de senha: gerenciado pelo Zitadel (argon2 internamente)
- Status: `pending_verification` → `active` → `suspended` → `deleted`
- User sync: UPSERT em `users` no primeiro login via Zitadel callback (zitadel_id como chave de sincronização)

### Req 4 — Billing e Assinaturas
- Planos: `trial` → `base` → `plus` → `pro`
- Trials: Síndico 15d, Empresa 7d, Parceiro 30d
- Trial: tudo visível, ações estratégicas bloqueadas
- Mensagem de bloqueio: _"Essa funcionalidade está disponível nos planos ativos da plataforma Master Síndico."_
- Integração Stripe via `IPaymentGateway` (agnóstico — troca de provider = nova implementação)
- Frequências: mensal e anual
- Upgrade/downgrade com billing proporcional
- Evento `SubscriptionExpired` → bloqueia premium, dados permanecem

### Req 4.1 — Trial por Persona

| Persona | Duração | Bloqueado |
|---------|---------|-----------|
| Síndico | 15 dias | Criar atividades timeline, comunicados, exportar relatórios, múltiplos condos |
| Empresa | 7 dias | Publicar vídeos, registros de execução, gerenciar usuários |
| Parceiro | 30 dias | Criar promoções, métricas, campanhas |

### Req 5 — Onboarding
- Auto-save de progresso (Redis)
- Identificação de persona via SearchParams ou seleção
- Fluxos:
  - **Morador (4 telas):** Buscar condo por ID → Registrar unidade → Termo LGPD → Foto
  - **Síndico (6 telas):** Dados pessoais → Tipo síndico → Governance markers → Criar condo → Mandato → Empresa síndica
  - **Empresa (7 telas):** Dados → CNPJ → Categorias → Compliance markers → Logo/fotos → Termos → Dashboard
  - **Parceiro (4 telas):** Dados estabelecimento → Endereço/CEP → Logo/descrição → Dashboard

### Req 6 — Acesso Baseado em Plano
- Matriz de features mapeando planos → funcionalidades
- Quotas: Connect Me requests, uploads de vídeo, storage
- Reset de quotas: anual (Connect Me), mensal (uploads)
- Feature flags para rollout gradual

### Req 6.2 — Perfil do Síndico
Campos obrigatórios de cadastro: `professional_photo`, `full_name`, `birth_date`, `professional_email`, `phone`, `cpf`, `professional_address`, `city_state_of_operation`, `sindico_type`, `years_of_experience`, `condominiums_under_management`

15 Governance Markers (autodeclarados): `membro_abracs`, `seguro_responsabilidade_civil`, `assessoria_juridica`, `assessoria_contabil`, `assessoria_seguranca_trabalho`, `conformidade_lgpd`, `conformidade_nr1`, `programa_compliance`, `selo_reclame_aqui`, `outras_certificacoes`, `premiacoes`

Disclaimer obrigatório: _"Marcadores são informativos, baseados em autodeclaração, e não representam certificação oficial da plataforma."_

Campos extras N2/N3: `mini_bio_profissional`, `video_institucional`, `formacao_certificacoes`, `administradoras_vinculadas`

### Req 6.3 — Perfil da Empresa
Campos obrigatórios: `logo`, `razao_social`, `nome_fantasia`, `cnpj`, `data_aniversario`, `representante_legal`, `email_comercial`, `telefone_comercial`, `endereco_comercial_cep`, `categoria_principal`, `subcategorias_atuacao` (até 5)

25 Compliance Markers (autodeclarados): ISOs (9001, 14001, 45001, 37001, 37301), ESG, selos, certificações, NRs

Regras de conteúdo: sem chamadas comerciais, sem preços, sem dados de contato no conteúdo público.

### Req 6.4 — Perfil do Parceiro da Vizinhança
Campos obrigatórios: `logo`, `razao_social_nome`, `nome_fantasia`, `cnpj_cpf` (CPF aceito), `endereco_cep`
Opcionais: `whatsapp`, `instagram`, até 3 fotos

### Req 7 — Registro de Condomínio
- Tenant com `tenant_type='condominium'`
- Campos: `address`, `CEP`, `condominium_type`, `total_units`, `blocks_or_towers`
- Validação CEP via ViaCEP (adapter)
- Tipos: `residencial`, `comercial`, `misto`, `shopping_center`, `galeria_comercial`, `edificio_garagem`
- ID alfanumérico único 8 chars + QR Code
- Mandato: `mandate_type`, `start_date`, `end_date`, `status`
- Tipos de mandato: `sindico_eleito`, `sindico_profissional`, `sindico_interino`
- Empresa síndica: token por email + permissões configuráveis + audit log
- Limite: **15 condomínios por síndico**

### Req 8 — Seletor de Condomínio
Síndico troca contexto entre condomínios. Persiste seleção na sessão.

### Req 9 — Registro de Unidade
1 titular + até 2 dependentes por unidade. Upload de foto + termo de ciência.

### Req 10 — Gestão de Usuários (Condomínio)
Até 3 usuários. Roles: `sindico_principal`, `sindico_auxiliar`, `assistente`. Matriz de permissões com toggles.

### Req 11 — Timeline (Linha do Tempo)
- Append-only, **imutável** após publicação
- 7 tipos: `atividade_da_gestao`, `registro_de_execucao`, `comunicado`, `evento`, `adendo`, `resultado_consulta_moradores`, `assembly_result`
- Único mecanismo de modificação: adendo (linkado ao original)
- Atualiza Plano Diretor automaticamente via evento NATS
- Visível para moradores em modo leitura

### Req 12 — Plano Diretor
- Status só muda via evento da Timeline — **NUNCA atualização manual**
- 26 áreas impactadas, 26 tipos de atividade, 9 riscos, 8 próximas ações
- 7 status: `planejada` → `em_contratacao` → `em_execucao` → `concluida` | `suspensa` | `reprogramada` | `atrasada`

### Req 13 — Eventos do Condomínio
13 tipos. Confirmação de participação e ciência por morador.

### Req 14 — Comunicados
8 tipos, 4 níveis de prioridade. Tracking de leitura por morador.

### Req 14.1 — Consultas aos Moradores
3 tipos: `sim_nao_nao_sei`, `multiple_choice`, `scale_1_to_5`. Resultado → Timeline.

### Req 15 — Avaliação de Gestão
A cada 2 meses. 3 perguntas escala 1-10. Anônima para síndico. Bloqueia funcionalidades se atrasada.

### Req 16 — Gestão de Dependentes
Titular adiciona até 2 dependentes. Revogação imediata bloqueia login.

### Req 17 — Encerramento de Vínculo
Motivo obrigatório. Revoga acesso titular + dependentes. Mantém histórico.

---

## Domínio 2 — Comercial (Req 18–27.3)

### Req 18 — Registro de Empresa
- Tenant com `tenant_type='company'`
- Validação e unicidade de CNPJ
- Status: `pending_verification` → `active` → `suspended` → `blocked`
- Visibilidade: Base visível apenas para Síndico Gestor/Plus/Pro; Plus/Pro visível para todos

### Req 19 — Connect Me: Síndico → Empresa
- Formulário unidirecional
- Dados do síndico revelados apenas após empresa clicar "Tenho Interesse"
- Log LGPD: timestamp, company_id, sindico_id, ip, terms_version
- Auto-close após 48h sem interesse
- Anti-assédio: botão denúncia + ações admin (warning/suspension/ban)
- Quotas: N1: 2/ano, N2: 4/ano, N3: ilimitado

### Req 19.1 — Connect Me: Morador → Síndico
- Formulário unidirecional frio (sem reply)
- Categorias: `duvida_gestao`, `solicitacao_manutencao`, `reclamacao`, `sugestao`, `outro`
- Urgência: `baixa`, `media`, `alta`, `urgente`
- Quotas: Base 2/ano, Pagante 4/ano

### Req 19.2 — Connect Me: Empresa → Empresa
- Apenas Plus/Pro
- Partnership types: `subcontratacao`, `parceria_tecnica`, `fornecimento_materiais`, `indicacao_servicos`, `outro`

### Req 19.3 — Connect Me: Síndico → Parceiro
- Promotion types: `desconto_exclusivo`, `promocao_dia`, `campanha_sazonal`, `parceria_continua`, `outro`

### Req 20 — Propostas
Status: `draft` → `sent` → `awaiting_validation` → `validated` → `accepted`/`rejected`. Imutável após `sent`.

### Req 21 — Votação de Fornecedor
2+ propostas validadas. Quorum: `simple_majority`, `qualified_majority`, `unanimous`. Resultado em tempo real.

### Req 22 — Negócio Fechado
Dupla assinatura: empresa primeiro, síndico depois. Após confirmação: imutável. Auto-publicação na Timeline.

### Req 23 — Registro de Execução
Empresa registra → Validação obrigatória do síndico → Timeline.

### Req 23.1 — Atividade Técnica
Empresa contribui para gestão preventiva. Validação obrigatória do síndico → Timeline.

### Req 24 — Comunicados Pós-Deal
5 tipos. Vinculado a `deal_id`. Validação obrigatória do síndico → Timeline.

### Req 25 — Adendo
Único mecanismo de modificação de registros imutáveis. Chain de adendos suportado.

### Req 26 — Avaliação de Empresa
5 critérios escala 1-10. Anônima para empresa. Pública para outros síndicos.

### Req 27 — Vizinhança: Síndico (VS1–VS10)
Dashboard, explorar parceiros, indicar, criar benefícios exclusivos, histórico.

### Req 27.1 — Vizinhança: Empresa (VE1–VE6)
Explorar e ativar promoções. Não pode criar.

### Req 27.2 — Vizinhança: Morador (VM1–VM7)
Feed por bairro. Badge "Indicado pelo Síndico". Seção "Benefícios do meu condomínio".

### Req 27.3 — Vizinhança: Parceiro (VZ1–VZ6)
Cadastro (CPF aceito), criar/editar promoções, métricas, histórico.
- 7 tipos de benefício: `desconto_percentual`, `desconto_fixo`, `produto_gratuito`, `combo_promocional`, `avaliacao_gratuita`, `brinde`, `beneficio_exclusivo`
- Promoção aberta (bairro) ou exclusiva (condomínio)

---

## Domínio 3 — Conteúdo (Req 29–32, 97–98.1)

### Req 29 — Vídeos Institucionais
- 12 tipos de vídeo
- Quotas mensais: N1: 4, N2: 8, N3: 12, Plus: 8, Pro: 12
- Trava trimestral: 90 dias entre atualizações
- Visibilidade moradores: SOMENTE durante votação de fornecedor se `autorizar_exibicao_votacao = true`
- Analytics: views, avg_watch_time, completion_rate, retention_rate
- Paywall Base: cortar stream aos 25%

### Req 30 — Agência de Marketing
- Ator delegado (não é tenant)
- Upload em nome da empresa
- Vídeo atribuído à empresa, não à agência
- Restrições: SÓ gerencia vídeos e analytics

### Req 31 — Busca e Descoberta
Full-text search. Filtros: categoria, localização, rating, tipo de vídeo. Autocomplete.

### Req 32 — Recomendações
Engine baseada em comportamento. Click-through tracking.

### Req 97 — Ebooks e Conteúdo Digital
PDF, EPUB. Acesso por plano premium.

### Req 98.1 — Academia LMS
- 5 áreas: Cursos, Treinamentos, Lives, Fórum, Biblioteca
- 10 categorias de curso, 3 níveis
- 3 trilhas: Síndico, Empresa, Morador
- Parceiro: sem acesso (botão escondido)
- Lives: 4 tipos, abertas/exclusivas, replay
- Fórum: 7 categorias, empresas podem criar tópicos
- Certificados automáticos ao completar 100%
- Integração cross-domain: recomenda treinamentos baseados em registros de atividade

---

## Domínio 4 — Assembleia (Req 48–60.12)

### Fase 1: Pré-Assembleia
- **Req 48:** Configuração — tipo, modalidade (presencial MVP), tempo de fala (2 ou 3 min), quorum
- **Req 49:** Pauta — 5 tipos de item, fração ideal via upload Excel (soma = 100%), lock após publicação
- **Req 50:** Notificações — 3 dias antes, 1 dia antes, 3 horas antes
- **Req 50.1:** Edital — geração automática PDF ou upload
- **Req 50.2:** Ciência obrigatória — morador perde acesso ao app até dar ciência. Tracking 6 meses.
- **Req 51:** Enquetes preliminares — consultivas, sem valor jurídico
- **Req 52:** Procurações — validação, prazo 24h pós-assembleia, proxy replica voto exato
- **Req 55:** Simulador de quorum

### Fase 2: Live-Day
- **Req 56:** Check-in — app, QR Code, terminal (CPF + bloco + unidade). Ausente momentâneo ≠ ausente definitivo.
- **Req 57:** Votação real-time — simples, fração ideal, fornecedor, eleição. Voto presencial assistido com `termo_de_voto`. Ausente momentâneo = abstenção automática.
- **Req 60.2:** Painel do Presidente — controle absoluto
- **Req 60.3:** Telão 2 áreas — Presentation Area + Operational Panel
- **Req 60.5:** Fila de fala — timer, extensão, alerta 30s
- **Req 60.10:** Modo contingência — votos offline + sync
- **Req 60.12:** Termos legais — aceite obrigatório antes do check-in

### Fase 3: Pós-Assembleia
- **Req 58:** Encerramento — ata automática, auto-publicação na Timeline
- **Req 59:** 6 tipos de relatório — presença, votação, procurações, fala, consolidado, transparência. Export CSV e PDF.
- **Req 60.6:** Homologação obrigatória por votação — após homologação, imutável
- **Req 60.7:** Score de transparência 0-100 (10 componentes)
- **Req 60.8:** Notificações automáticas pós-encerramento
- **Req 60.9:** Indicador de transparência

**NFR críticos:**
- WebSocket latência < 200ms
- `DECIMAL/NUMERIC` para fração ideal (nunca float)
- Append-only para todos os eventos de voto, check-in, homologação
- Auto-save de estado no Redis (recuperação de crash)

---

## Domínio 5 — Cross-Domain & Integrações (Req 33–47, 61–67, 102–103)

### Req 33–42 — Compliance
- Governance Score 0-100 (calculado diariamente)
- Declaração anual obrigatória — bloqueia exportação de dossiê se atrasada
- Assinaturas obrigatórias de todos os usuários cadastrados
- Matriz de conflitos de interesse
- Trilha de auditoria append-only — retenção 5 anos (LGPD)
- Mapa de riscos
- LGPD Registry — tracker de revelação de dados no Connect Me
- Dossiê PDF assinado digitalmente no fim do mandato

### Req 43–45 — Analytics
- Dashboard Síndico: total moradores, governance_score, taxa conclusão Plano Diretor, satisfação média
- Dashboard Empresa: condomínios conectados, CTR Connect Me, registros aprovados/rejeitados, avaliação média
- Dashboard Agência: engajamento cruzado das empresas vinculadas

### Req 46–47 — Gestão de Mandato
- Encerramento exige `compliance = em_dia`
- Transferência: passa histórico do condomínio, NÃO passa avaliações pessoais do síndico

### Req 61–68 — Integrações (Adapter Layer)

| Req | Serviço (atual) | Adapter |
|-----|-----------------|---------|
| 61 | Stripe | `IPaymentGateway` (billing/domain/providers) |
| 62 | Resend | `IEmailProvider` (shared/providers/email) |
| 63 | Twilio / Zenvia | `ISMSProvider` |
| 64 | Mux | `IVideoProvider` (content/domain/providers) |
| 65 | MinIO / Cloudflare R2 | `IStorageProvider` (shared/providers/storage) |
| 66 | ViaCEP | `ICEPLookup` |
| 67 | Firebase Cloud Messaging | `IPushProvider` |
| 68 | Livekit | `ILiveProvider` (assembly/domain/providers) |

### Req 102 — Antifraude
- 1 device ativo por conta
- CAPTCHA para tentativas suspeitas
- Bloqueia múltiplos cadastros do mesmo IP

### Req 103 — Visibilidade Estrita
- Vídeos de empresa blindados a moradores
- Liberados momentaneamente durante assembleia (escolha de fornecedor)
- Revogados após fim da votação

---

## Banco de Talentos (Decision 6)

### Lado Morador — 11 telas
Vídeo-currículo (90s, lock 90 dias), perfil profissional (6 perguntas), 18 áreas de interesse, dados pessoais, disponibilidade, pretensão salarial, experiência (últimos 3-5 vínculos), formação, consentimento LGPD, revisão.

### Lado Empresa — 7 funcionalidades
Busca avançada (Meilisearch), favoritos, exportação PDF, histórico de buscas, analytics de funil, auto-tags (7 categorias), tags manuais privadas.

Acesso: Empresas Plus/Pro apenas.

---

## Painel Admin Master Síndico

- Dashboard financeiro (MRR, assinaturas ativas/canceladas)
- Gestão de usuários (listar, bloquear, editar)
- Moderação de conteúdo (vídeos, fórum)
- Configurações da plataforma (categorias, parâmetros globais)
- Broadcasts oficiais (email/SMS segmentado)
- Aprovação manual de CNPJs
- Controle de inadimplentes Mercado Pago
- Gestão global de tenants

---

## Matriz Funcional (resumo)

| Funcionalidade | Base | Morador Pg | N1 | N2 | N3 | Plus | Pro | Marketing | Vizinhança |
|---|---|---|---|---|---|---|---|---|---|
| Busca geral | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Vídeos empresa (preview 25%) | ✅ | ✅ | ✅ | ✅ | — | — | — | — | — |
| Vídeos empresa (integral) | — | — | — | — | ✅ | — | — | — | — |
| Publicar vídeos | — | — | — | 4/mês | 8/mês | 8/mês | 12/mês | — | — |
| Fórum | — | — | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | — |
| Lives | — | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | — |
| Cursos certificados | — | — | — | ✅ | ✅ | — | criar | — | — |
| Connect Me | — | 2/ano | — | 2/ano | 4/ano | — | ✅ | — | — |
| Vídeo-currículo | — | ✅ | — | — | — | — | — | — | — |
| Visualizar currículos | — | — | — | — | — | ✅ | ✅ | ✅ | ✅ |
| Criar promoções | — | — | — | — | — | — | — | — | ✅ |
| Perfil público | — | — | — | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
