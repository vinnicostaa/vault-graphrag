---
title: "Domínio - Identidade"
version: "3.0"
date: 2026-03-10
status: "em-revisão"
high_domain: "Identidade"
requisitos: "Req 1–17"
total_requisitos: 17
tags:
  - mastersindico
  - dominio
  - identidade
  - iam
  - billing
  - onboarding
---

# 🔐 Domínio Principal: Identidade

> **High-Domain:** Identidade | **Requisitos:** 1–17 | **Telas:** M1–M15, S1–S6
> O IAM do ecossistema. Todo mundo resolve identidade passando por aqui primeiro.
> **Breadcrumbs:** [[00 - Índice Master Síndico|🏠 Início]] > [[Arquitetura do Ecossistema|🏛️ Arquitetura]] > 📄 Domínio - Identidade

---

## Visão do Domínio

![[Dominio Identidade.canvas]]

---

## Requisito 1: Autenticação de Usuário

> **História do Usuário:** Como usuário, eu quero me autenticar de forma segura, para que eu possa acessar a plataforma com minhas credenciais.

### Critérios de Aceite

| # | Tipo | Critério |
|---|------|----------|
| 1 | OBRIGATÓRIO | O `Authentication_Service` DEVE implementar autenticação baseada em JWT com Passport |
| 2 | EVENTO | QUANDO um usuário fornecer credenciais válidas, O `Authentication_Service` DEVE gerar um token JWT com expiração |
| 3 | EVENTO | QUANDO um usuário fornecer credenciais inválidas, O `Authentication_Service` DEVE retornar um erro de autenticação |
| 4 | OBRIGATÓRIO | O `Authentication_Service` DEVE suportar redefinição de senha via verificação de email |
| 5 | SEGURANÇA | O `Authentication_Service` DEVE implementar rate limiting para prevenir ataques de força bruta |
| 6 | EVENTO | QUANDO um token JWT expirar, O `Authentication_Service` DEVE exigir re-autenticação |
| 7 | AUDITORIA | O `Authentication_Service` DEVE registrar todas as tentativas de autenticação com timestamp e endereço IP |
| 8 | SEGURANÇA | O `Authentication_Service` DEVE limitar acesso do usuário a **1 dispositivo por conta** |
| 9 | SEGURANÇA | O `Authentication_Service` DEVE rastrear fingerprint do dispositivo e endereço IP por sessão |
| 10 | EVENTO | QUANDO um usuário tentar login de um segundo dispositivo, O `Authentication_Service` DEVE invalidar a sessão anterior |
| 11 | SEGURANÇA | O `Authentication_Service` DEVE prevenir múltiplas contas do mesmo endereço IP para evitar burla de plano |

### Requisitos Não-Funcionais (Req 1)

| Categoria | Requisito |
|-----------|-----------|
| **Segurança** | JWT com expiração + refresh token |
| **Segurança** | Limite de 1 dispositivo ativo por conta |
| **Segurança** | Rate limiting contra brute force |
| **Auditoria** | Log de todas as tentativas de autenticação |
| **Anti-fraude** | Detecção de múltiplas contas por IP |

### Rotas de API (Req 1)

```
POST   /v1/auth/login              → Login (email + senha) → JWT
POST   /v1/auth/register           → Registro novo usuário
POST   /v1/auth/refresh            → Refresh token
POST   /v1/auth/forgot-password    → Solicitar reset de senha
POST   /v1/auth/reset-password     → Confirmar reset com token
POST   /v1/auth/oauth/:provider    → OAuth (Google/Apple)
GET    /v1/auth/me                 → Sessão atual do usuário
POST   /v1/auth/logout             → Logout (invalidar sessão)
```

### Eventos Emitidos

| Evento | Quando |
|--------|--------|
| `UserLoggedIn` | Login bem-sucedido |
| `UserLoggedOut` | Logout ou invalidação de sessão |
| `LoginFailed` | Tentativa de login com credenciais inválidas |
| `SessionInvalidated` | Login de outro dispositivo |

---

## Requisito 2: Autorização de Usuário (ABAC)

> **História do Usuário:** Como administrador do sistema, eu quero controle de acesso baseado em atributos, para que os usuários só acessem os recursos autorizados.

### Critérios de Aceite

| # | Tipo | Critério |
|---|------|----------|
| 1 | OBRIGATÓRIO | O `Authorization_Service` DEVE implementar ABAC (Attribute-Based Access Control) |
| 2 | OBRIGATÓRIO | QUANDO verificar autorização, O `Authorization_Service` DEVE avaliar: `userId`, `tenantId`, `role`, `plan`, `action` |
| 3 | OBRIGATÓRIO | O `Authorization_Service` DEVE retornar: `allowed` (boolean), `role`, `plan` e features disponíveis |
| 4 | OBRIGATÓRIO | O `Authorization_Service` DEVE suportar roles: `sindico`, `morador_titular`, `morador_dependente`, `empresa_administrador`, `empresa_operador`, `agencia` |
| 5 | SEGURANÇA | O `Authorization_Service` DEVE garantir isolamento de tenant (usuários só acessam dados do seu tenant) |
| 6 | AUDITORIA | QUANDO um usuário tentar ação não autorizada, O `Authorization_Service` DEVE negar acesso e registrar a tentativa |
| 7 | EXCEÇÃO | O `Authorization_Service` DEVE suportar operações cross-tenant via grants explícitos (Connect Me, validações) |

### Modelo ABAC

![[Modelo ABAC.canvas]]

### Roles do Sistema

| Role | Contexto | Permissões Resumidas |
|------|----------|---------------------|
| `sindico` | Tenant Condomínio | Owner — tudo no condomínio |
| `morador_titular` | Tenant Condomínio | Leitura + avaliação + Connect Me |
| `morador_dependente` | Tenant Condomínio | Leitura limitada |
| `empresa_administrador` | Tenant Empresa | Owner — tudo na empresa |
| `empresa_operador` | Tenant Empresa | Registros e atividades técnicas |
| `agencia` | Delegado da Empresa | Apenas conteúdo institucional |

---

## Requisito 3: Registro de Usuário

> **História do Usuário:** Como novo usuário, eu quero me registrar na plataforma, para que eu possa criar minha conta.

### Critérios de Aceite

| # | Tipo | Critério |
|---|------|----------|
| 1 | OBRIGATÓRIO | O `User_Registry` DEVE suportar registro para: `sindico`, `morador`, `empresa`, `agencia` |
| 2 | OBRIGATÓRIO | QUANDO registrar, O `User_Registry` DEVE exigir: `email`, `password`, `name`, `phone`, `user_type` |
| 3 | VALIDAÇÃO | O `User_Registry` DEVE validar unicidade do email antes do registro |
| 4 | OBRIGATÓRIO | O `User_Registry` DEVE enviar verificação de email após registro |
| 5 | SEGURANÇA | O `User_Registry` DEVE fazer hash de senhas usando `bcrypt` ou `argon2` |
| 6 | OBRIGATÓRIO | O `User_Registry` DEVE criar registro com status: `pending_verification`, `active`, `suspended`, `deleted` |
| 7 | EVENTO | QUANDO email for verificado, O `User_Registry` DEVE ativar a conta |
| 8 | EVENTO | O `User_Registry` DEVE emitir evento `UserRegistered` após registro bem-sucedido |

### Fluxo de Registro

![[Fluxo Registro.canvas]]

---

## Requisito 4: Billing e Assinaturas

> **História do Usuário:** Como usuário, eu quero assinar um plano, para que eu possa acessar funcionalidades premium.

### Critérios de Aceite

| # | Tipo | Critério |
|---|------|----------|
| 1 | OBRIGATÓRIO | O `Billing_Service` DEVE suportar planos: `trial`, `base`, `plus`, `pro` |
| 2 | OBRIGATÓRIO | O `Billing_Service` DEVE fornecer períodos de trial: **15 dias** para Síndico, **7 dias** para Empresa, **30 dias** para Parceiro da Vizinhança |
| 3 | OBRIGATÓRIO | O `Billing_Service` DEVE fornecer trial com acesso limitado (visualização habilitada, ações estratégicas bloqueadas) |
| 4 | EVENTO | QUANDO o trial expirar, O `Billing_Service` DEVE rebaixar para plano base se não houver pagamento |
| 5 | OBRIGATÓRIO | O `Billing_Service` DEVE suportar frequências de pagamento: `mensal`, `anual` |
| 6 | INTEGRAÇÃO | O `Billing_Service` DEVE integrar com **Mercado Pago** via `Adapter_Layer` |
| 7 | REGRA | QUANDO a assinatura expirar, O `Billing_Service` DEVE bloquear funcionalidades premium retroativamente (dados permanecem, acesso bloqueado) |
| 8 | EVENTO | O `Billing_Service` DEVE emitir evento `SubscriptionExpired` quando a assinatura terminar |
| 9 | OBRIGATÓRIO | O `Billing_Service` DEVE manter `billing_history` com: `date`, `amount`, `plan`, `status`, `payment_method` |
| 10 | REGRA | O `Billing_Service` DEVE calcular data de renovação a partir da data da assinatura inicial |
| 11 | OBRIGATÓRIO | O `Billing_Service` DEVE suportar upgrade e downgrade de plano com billing proporcional |

### Modelo de Planos

![[Modelo Planos.canvas]]

> [!WARNING]
> **Decision 4 — PENDENTE:** Quando o plano expira, o bloqueio é imediato (hard block) ou tem período de graça (soft block com aviso)? Impacta a lógica de frontend inteira.

### Rotas de API (Req 4)

```
GET    /v1/billing/plans           → Listar planos disponíveis
POST   /v1/billing/subscribe       → Criar assinatura
PUT    /v1/billing/upgrade         → Upgrade de plano
PUT    /v1/billing/downgrade       → Downgrade de plano
DELETE /v1/billing/cancel          → Cancelar assinatura
GET    /v1/billing/history         → Histórico de pagamentos
GET    /v1/billing/status          → Status atual da assinatura
```

---

## Requisito 4.1: Módulo Trial (Regras Detalhadas)

> **História do Usuário:** Como usuário trial, eu quero explorar a plataforma, para que eu possa entender seu valor antes de assinar.

### Regras Gerais do Trial

| # | Regra |
|---|-------|
| 1 | O Trial DEVE tornar todas as funcionalidades da plataforma **visíveis** |
| 2 | O Trial DEVE **bloquear** ações estratégicas/operacionais durante o período |
| 3 | Mensagem de bloqueio: _"Essa funcionalidade está disponível nos planos ativos da plataforma Master Síndico. Ative seu plano para desbloquear esta função."_ |
| 4 | O Trial DEVE exibir mensagens contextuais de conversão |
| 5 | O Trial DEVE rastrear ações do usuário para personalizar mensagens |

### Trial por Persona

| Persona | Duração | Ações Permitidas | Ações Bloqueadas |
|---------|---------|-------------------|------------------|
| **Síndico** | 15 dias | Criar perfil, 1 condomínio, plano diretor, assistir vídeos, buscar empresas, Connect Me, perguntas moradores, dashboard, ver módulo assembleia | Criar atividades timeline, comunicados, eventos, exportar relatórios, cadastrar parceiro, validar registros, compliance, múltiplos condos |
| **Empresa** | 7 dias | Perfil institucional, segmento/subcategoria, logo/descrição, perfil público, assistir vídeos, buscar síndicos/empresas, marketing link, dashboard, receber Connect Me | Publicar vídeos, registro execução, atividades técnicas, comunicados técnicos, gerenciar usuários, convidar agência |
| **Parceiro** | 30 dias | Perfil estabelecimento, endereço/CEP, logo/descrição, ver condomínios região, ver promoções locais | Criar promoção, palavra-chave promoção, métricas de promoção, promoções exclusivas, campanhas locais |

---

## Requisito 5: Onboarding

> **História do Usuário:** Como novo usuário, eu quero um onboarding guiado, para que eu possa configurar minha conta corretamente.

### Critérios de Aceite

| # | Tipo | Critério |
|---|------|----------|
| 1 | OBRIGATÓRIO | O `Onboarding_Service` DEVE identificar tipo de usuário via SearchParams ou seleção |
| 2 | FLUXO | QUANDO `user_type` for **síndico**, guiar por: criação de condomínio, dados de mandato, link empresa síndica (opcional) |
| 3 | FLUXO | QUANDO `user_type` for **morador**, guiar por: busca de condomínio por ID, registro de unidade, termo de ciência, upload de foto |
| 4 | FLUXO | QUANDO `user_type` for **empresa**, guiar por: perfil da empresa, validação CNPJ, categorias de serviço, dados institucionais |
| 5 | FLUXO | QUANDO `user_type` for **agência**, guiar por: perfil da agência, tipos de serviço, portfólio |
| 6 | VALIDAÇÃO | O `Onboarding_Service` DEVE validar campos obrigatórios antes de permitir progressão |
| 7 | UX | O `Onboarding_Service` DEVE salvar progresso para permitir retomada |
| 8 | EVENTO | QUANDO onboarding for completo, O `Onboarding_Service` DEVE marcar usuário como onboarded e redirecionar ao dashboard |

### Telas de Onboarding por Persona

| Persona | Qtd Telas | Fluxo |
|---------|-----------|-------|
| **Morador** | 4 telas | Buscar condo → Registrar unidade → Termo de ciência → Foto |
| **Síndico** | 6 telas | Dados pessoais → Tipo síndico → Governance markers → Criar condo → Mandato → Empresa síndica |
| **Empresa** | 7 telas | Dados empresa → CNPJ → Categorias → Compliance markers → Logo/fotos → Termos → Dashboard |
| **Parceiro** | 4 telas | Dados estabelecimento → Endereço/CEP → Logo/descrição → Dashboard |

---

## Requisito 6: Acesso Baseado em Plano

> **História do Usuário:** Como sistema, eu quero impor acesso baseado em plano, para que os usuários só usem funcionalidades incluídas no seu plano.

### Critérios de Aceite

| # | Tipo | Critério |
|---|------|----------|
| 1 | OBRIGATÓRIO | O `Feature_Access_Service` DEVE manter matriz de features mapeando planos para funcionalidades |
| 2 | OBRIGATÓRIO | QUANDO verificar acesso, DEVE avaliar: `userId`, `plan`, `feature_name` |
| 3 | OBRIGATÓRIO | DEVE retornar: `allowed` (boolean), `quota` (`used`, `limit`, `resetDate`) se aplicável |
| 4 | OBRIGATÓRIO | DEVE impor quotas para: requisições Connect Me, uploads de vídeo, limites de armazenamento |
| 5 | EVENTO | QUANDO a quota for excedida, DEVE negar acesso e retornar informações da quota |
| 6 | REGRA | DEVE resetar quotas conforme regras do plano (mensal, anual, por ação) |
| 7 | DEVOPS | DEVE suportar feature flags para rollout gradual |

---

## Requisitos 6.2–6.4: Perfis Profissionais

### Req 6.2 — Perfil do Síndico

| Campo | Obrigatório | Detalhe |
|-------|-------------|---------|
| `professional_photo` | ✅ | Foto profissional |
| `full_name` | ✅ | Nome completo |
| `birth_date` | ✅ | Data de nascimento |
| `professional_email` | ✅ | Email profissional |
| `phone` | ✅ | Telefone |
| `cpf` | ✅ | CPF |
| `professional_address` | ✅ | Endereço de atuação |
| `city_state_of_operation` | ✅ | Cidade/Estado |
| `sindico_type` | ✅ | `morador_sindico` ou `sindico_profissional` |
| `years_of_experience` | ✅ | Anos de experiência |
| `condominiums_under_management` | ✅ | Condomínios gerenciados |

**15 Governance Markers (autodeclarados):**

`membro_abracs` · `seguro_responsabilidade_civil` · `assessoria_juridica` · `assessoria_contabil` · `assessoria_seguranca_trabalho` · `conformidade_lgpd` · `conformidade_nr1` · `programa_compliance` · `selo_reclame_aqui` · `outras_certificacoes` · `premiacoes`

> **Disclaimer obrigatório:** _"Marcadores são informativos, baseados em autodeclaração, e não representam certificação oficial da plataforma."_

**Campos extras para N2/N3:** `mini_bio_profissional`, `video_institucional`, `formacao_certificacoes`, `administradoras_vinculadas`

**Termos obrigatórios:** `termo_registro_gestao`, `termo_indicadores_selos`, `termo_links_temporarios`

### Req 6.3 — Perfil da Empresa

| Campo | Obrigatório | Detalhe |
|-------|-------------|---------|
| `logo` | ✅ | Logo da empresa |
| `razao_social` | ✅ | Razão social |
| `nome_fantasia` | ✅ | Nome fantasia |
| `cnpj` | ✅ | CNPJ válido |
| `data_aniversario` | ✅ | Fundação |
| `representante_legal` | ✅ | Nome do representante |
| `email_comercial` | ✅ | Email comercial |
| `telefone_comercial` | ✅ | Telefone |
| `endereco_comercial_cep` | ✅ | CEP do endereço |
| `categoria_principal` | ✅ | Categoria de serviço |
| `subcategorias_atuacao` | ✅ | Subcategorias |

**25 Compliance Markers (autodeclarados):** lista completa incluindo ISOs, ESG, selos, certificações.

**Regras de conteúdo:** sem chamadas comerciais, sem preços, sem dados de contato no conteúdo.

### Req 6.4 — Perfil do Parceiro da Vizinhança

| Campo | Obrigatório | Detalhe |
|-------|-------------|---------|
| `logo` | ✅ | Logo |
| `razao_social_nome` | ✅ | Razão social ou nome |
| `nome_fantasia` | ✅ | Nome fantasia |
| `cnpj_cpf` | ✅ | CNPJ ou CPF (sem exigência de CNPJ) |
| `endereco_cep` | ✅ | CEP |
| `whatsapp` | Opcional | WhatsApp |
| `instagram` | Opcional | Instagram |
| Até 3 fotos | Opcional | Fotos do estabelecimento |

---

## Requisitos 7–10: Condomínio e Unidades

### Req 7 — Registro de Condomínio

| # | Critério |
|---|----------|
| 1 | DEVE criar tenant com `tenant_type='condominium'` |
| 2 | Campos obrigatórios: `address`, `CEP`, `condominium_type`, `total_units`, `blocks_or_towers` |
| 3 | DEVE validar CEP via ViaCEP API através do `Adapter_Layer` |
| 4 | Tipos de condomínio: `residencial`, `comercial`, `misto`, `shopping_center`, `galeria_comercial`, `edificio_garagem` |
| 5 | DEVE gerar ID alfanumérico único (8 caracteres) |
| 6 | DEVE gerar QR Code contendo o ID do condomínio |
| 7 | DEVE criar registro de mandato: `mandate_type`, `start_date`, `end_date`, `status` |
| 8 | Tipos de mandato: `sindico_eleito`, `sindico_profissional`, `sindico_interino` |
| 9 | DEVE permitir vincular `empresa_sindica` (opcional) com permissões configuráveis |
| 10 | Permissões da empresa síndica: `visualizar_publicacoes`, `editar_publicacoes_sindico`, `ocultar_publicacoes`, `validar_registros_execucao`, `validar_comunicados_empresas` |
| 11 | Token via email para validação da empresa síndica |
| 12 | Log de todas as ações da empresa síndica em trilha de auditoria |
| 13 | Emitir evento `CondominiumCreated` |
| 14 | Limite: **até 15 condomínios por síndico** |
| 15 | Negar criação do 16º condomínio com erro de limite |

> [!WARNING]
> **Gap:** O PDF possui inconsistência interna — regra 5 diz 12 condomínios, tela S2 diz 15. Requirements usa 15. Precisa confirmação do cliente.

### Req 8 — Seletor de Condomínio

O síndico troca de contexto entre condomínios. Exibe nome, endereço, status (ativo/encerrado), data fim do mandato. Persiste seleção na sessão.

### Req 9 — Registro de Unidade

Morador registra unidade: bloco/torre + número + nome proprietário + CPF. **1 titular + até 2 dependentes por unidade.** Requer upload de foto e termo de ciência.

### Req 10 — Gestão de Usuários (Condomínio)

Síndico pode adicionar até 3 usuários no condomínio. Roles: `sindico_principal`, `sindico_auxiliar`, `assistente`. Matriz de permissões com toggles individuais.

---

## Requisitos 11–14.1: Governança

### Req 11 — Timeline (Linha do Tempo)

> A **entidade central** de todo o sistema de governança. Tudo converge para ela.

| # | Critério |
|---|----------|
| 1 | DEVE implementar log append-only, **imutável** |
| 2 | Tipos de entrada: `atividade_da_gestao`, `registro_de_execucao`, `comunicado`, `evento`, `adendo`, `resultado_consulta_moradores`, `assembly_result` |
| 3 | Campos: `title`, `description`, `entry_type`, `date`, `author` |
| 4 | Anexos: fotos, documentos, vídeos |
| 5 | ID único para cada publicação |
| 6 | Timestamp `published_at` na publicação |
| 7 | **Após publicação: IMUTÁVEL** (sem edição, sem exclusão) |
| 8 | Único mecanismo de modificação: **adendo** |
| 9 | Adendo linkado à entrada original, marcado como emendado |
| 10 | Emitir evento `TimelinePublished` |
| 11 | **Atualizar Plano Diretor automaticamente** quando ação relacionada é publicada |
| 12 | Visível para todos os moradores em **modo leitura** |

### Req 12 — Plano Diretor

> **Domínio reativo** — status só muda quando atividade vinculada é publicada na Timeline. NÃO permite atualização manual.

**26 áreas impactadas** · **26 tipos de atividade** · **9 riscos associados** · **8 próximas ações** · **7 status**

Status: `planejada` → `em_contratacao` → `em_execucao` → `concluida` | `suspensa` | `reprogramada` | `atrasada`

> [!WARNING]
> **Gap:** O PDF lista 7 status, o .ini listava 11. Requirements usa 7 (alinhado com PDF). Pendente confirmação do cliente.

### Req 13 — Eventos

**13 tipos de evento** (assembleia ordinária, manutenção programada, inspeção, etc.)
Suporta confirmação de participação e ciência por morador. Publicável na Timeline após conclusão.

### Req 14 — Comunicados

**8 tipos de comunicado** · **4 níveis de prioridade** (low, medium, high, urgent)
Rastreia status de leitura por morador.

### Req 14.1 — Perguntas aos Moradores

Síndico cria perguntas para moradores. **3 tipos:** `sim_nao_nao_sei`, `multiple_choice`, `scale_1_to_5`. Resultado consolidado é publicado na Timeline.

---

## Requisitos 15–17: Morador

### Req 15 — Avaliação de Gestão

Moradores avaliam gestão do síndico a cada **2 meses**. **3 perguntas** com escala 1-10:

1. _"Qual seu nível de satisfação com a gestão do síndico?"_
2. _"Você considera que a gestão tem sido transparente na comunicação com os moradores?"_
3. _"Você percebe evolução na organização do condomínio durante esta gestão?"_

Resultados agregados e **anônimos** para o síndico. Bloqueia certas funcionalidades se avaliação estiver atrasada.

### Req 16 — Gestão de Acesso do Morador

Titular pode adicionar **até 2 dependentes**. Convite por email. Revogação imediata bloqueia login do dependente.

### Req 17 — Gestão de Vínculo do Morador

Morador pode encerrar vínculo com motivo obrigatório. Encerramento revoga acesso para titular + todos os dependentes. Mantém histórico.

**Motivos de encerramento:** `mudanca_definitiva`, `venda_unidade`, `encerramento_contrato_locacao`, `erro_cadastro`

---

**Breadcrumbs:** [[00 - Índice Master Síndico|🏠 Início]] · **Próximo:** [[Domínio - Institucional]]
