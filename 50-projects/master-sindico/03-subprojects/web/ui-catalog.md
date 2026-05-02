---
title: Web — UI Catalog (catálogo de telas)
type: spec
tags:
  - subproject
  - web
  - master-sindico
  - v2
  - ui-catalog
  - telas
  - personas
  - jornadas
  - apps
source: >-
  Development/Content/02-Jornadas/Inventario-Telas.md (371 linhas, 136+ telas) +
  Development/web/apps/* + 00-product/personas.md
created: 2026-04-23T00:00:00.000Z
updated: '2026-04-25'
aliases:
  - ui-catalog
  - telas
  - screen-catalog
---

# Web — UI Catalog (catálogo de telas)

Catálogo **exaustivo** das telas do sub-projeto `web/`, mapeado **por app × persona × rota × estados**. Fonte primária: `Development/Content/02-Jornadas/Inventario-Telas.md` (371 linhas; 136+ telas + assembleia + banco talentos).

Total deste catálogo: **≥ 141 telas** distribuídas entre as **7 apps** (`auth`, `cms`, `lms`, `forum`, `assembly`, `lp`, **`admin`** — D-134 2026-04-25).

> **⚠️ ATUALIZAÇÃO 2026-04-25 (D-134)**: admin owner Master Síndico **AGORA TEM app separada** `apps/admin` (revoga D-072 que dizia "role transversal embutida"). Trigger: viabilização Cloudflare Access SSO+MFA (D-117) + benchmark big tech (D-113).
>
> **Outros sub-roles** (administradora condominial D-114, agência marketing D-102, empresa-síndica D-073, auditor assembly D-132) **continuam em rotas internas dos produtos** (principalmente `cms`) — não criam apps separadas. Apenas o admin owner Master Síndico tem app dedicada.
>
> Ver [[security#10-admin-isolation]] e [[../admin-privilegios]] (banner superseded em parte por D-134).

Cada tela é catalogada com:
- **ID**: estável (M1-M15, S1-S31, E1-E16, MK1-MK8, VZ/VS/VE/VM, LMS1-LMS15, C1-C11, TEL1-TEL11, AS1-AS8, AD1-AD8, PUB1-PUB6, SYS1-SYS8).
- **App**: `auth | cms | lms | forum | assembly | lp`.
- **Persona alvo**: quem acessa.
- **Rota**: path relativo dentro da app.
- **Ações**: verbos primários.
- **Estados**: loading, empty, error, success, permission-denied.
- **Visibilidade / regras**: gates ABAC, quotas, trial.

Regras transversais (R1-R10 extraídas do inventário; aplicam a todas as telas relevantes):

- **R1 Confirmação de contexto** — modal "Publicar para Condomínio X?" antes de publish crítico.
- **R2 Voz + texto** — todo textarea aceita voice input (Web Speech API).
- **R3 Nada deletado** — edita ou oculta com trilha; INSERT-only em timeline.
- **R4 Origem externa pendente** — upload de empresa entra como "Aguardando validação".
- **R5 Pós-fechamento imutável** — Adendo Formal é único mecanismo de alteração.
- **R6 Continuidade institucional** — novo síndico herda histórico; antigo mantém leitura + export.
- **R7 Compliance não trava onboarding** — mas marcos exigem compliance ativo.
- **R8 Assinatura obrigatória** — compliance "Em dia" só com todos assinados.
- **R9 Notificações internas** — ação importante gera notificação in-app.
- **R10 LGPD + rastreabilidade** — compartilhar dado registra `{finalidade, ts, user, consent_version}`.

---

## Índice por app

- [§A Auth (8 telas)](#a-app-auth) — `auth.mastersindico.com.br`
- [§B CMS — painel multi-persona (~100 telas)](#b-app-cms)
  - [B.1 Morador (M1-M15)](#b1-morador-m1-m15) — 15 telas
  - [B.2 Síndico (S1-S31) + Compliance (C1-C11)](#b2-síndico-s1-s31--compliance-c1-c11) — 42 telas
  - [B.3 Empresa (E1-E16)](#b3-empresa-e1-e16) — 16 telas
  - [B.4 Agência de Marketing (MK1-MK8)](#b4-agência-marketing-mk1-mk8) — 8 telas
  - [B.5 Parceiro Vizinhança (VZ1-VZ6) + Síndico (VS1-VS10) + Empresa (VE1-VE6) + Morador (VM1-VM7)](#b5-vizinhanca) — 29 telas
  - [B.6 Banco de Talentos — Morador (TEL1-TEL11)](#b6-banco-talentos-tel1-tel11) — 11 telas
  - [B.7 Admin MS — app dedicada `apps/admin` (AD1-AD8)](#b7-admin-ms--app-dedicada-d-134) — 8 telas (D-134 2026-04-25)
  - [B.8 Conta + Sistema (SYS1-SYS8)](#b8-conta-sistema-sys1-sys8) — 8 telas
- [§C LMS (LMS1-LMS15)](#c-app-lms-lms1-lms15) — 15 telas
- [§D Forum (subset Vizinhança)](#d-app-forum) — reutiliza telas VZ/VS/VE/VM
- [§E Assembly (AS1-AS8)](#e-app-assembly-as1-as8) — 8 telas
- [§F LP (PUB1-PUB6)](#f-app-lp-pub1-pub6) — 6 telas

---

## A. App `auth`

Port **3000**. Domain `auth.mastersindico.com.br`. Entry-point Zitadel OIDC + PKCE para todas as personas.

### AUTH1 — Splash / Home auth

- **Rota**: `/`
- **Persona**: Público (não autenticado). Redireciona se já logado.
- **Conteúdo**: logo dourado, slogan "Governança condominial estruturada", [Entrar] → `/sign-in`, [Criar conta] → `/sign-up`.
- **Estados**: loading (splash logo), error (sem conexão backend).

### AUTH2 — Sign In

- **Rota**: `/sign-in` (montada sob layout `_auth`)
- **Persona**: Todas.
- **Campos**: Email, Senha; [Esqueci minha senha] link; [Entrar com Google/Apple] OIDC social (M3+).
- **Ações**: [Entrar] → backend `/api/v1/auth/login` → Zitadel PKCE → callback → cookie.
- **Estados**: loading (spinner btn), error (credenciais inválidas, tentativas excedidas), success (redirect).
- **Regras**: rate-limit 5 tentativas / 15min → CAPTCHA; 3 falhas consecutivas → temporary lockout 30min.

### AUTH3 — Sign Up (escolha de persona)

- **Rota**: `/sign-up`
- **Persona**: Público.
- **Conteúdo**: 5 cards de persona (Síndico, Morador, Empresa, Parceiro Vizinhança, Agência Marketing) — cada um com breve descrição + [Quero ser X].
- **Ações**: [Quero ser síndico] → `/sign-up/sindico` (primeira tela do onboarding síndico).
- **Estados**: idle.

### AUTH4 — Cadastro (forms por persona)

- **Rota**: `/sign-up/:persona` (persona = `sindico | morador | empresa | parceiro | agencia`)
- **Persona**: Público em fluxo.
- **Conteúdo**: form multi-step específico da persona. Dados obrigatórios do §00-product/personas.
- **Auto-save** em Redis backend a cada step.
- **Estados**: rascunho salvo, submit pendente, error (CPF duplicado etc).
- **Regras**: Zod sync + async validation; LGPD consent ao final.

### AUTH5 — OTP / Recuperação de senha

- **Rota**: `/recovery`
- **Persona**: Todas.
- **Fluxo**: informar email → OTP (SMS + email) → nova senha.
- **Estados**: OTP expirado, OTP inválido (max 3 tentativas), success → redirect `/sign-in`.

### AUTH6 — Callback Zitadel

- **Rota**: `/callback` (recebe `?code=X&state=Y`)
- **Persona**: Todas (pós-OIDC).
- **Conteúdo**: spinner + "Autenticando..."; backend resolve cookie → redirect app de origem.
- **Estados**: success (redirect), error (code inválido, state mismatch — muito raro).

### AUTH7 — Device Mismatch

- **Rota**: `/device-mismatch`
- **Persona**: Usuário logado em outro device.
- **Conteúdo**: "Você está conectado em outro dispositivo. Ao continuar aqui, o outro será desconectado." + [Continuar aqui] (force_login=true) / [Cancelar] volta.
- **Estados**: loading, error, success.

### AUTH8 — Logout / Sign Out

- **Rota**: `/logout` (POST)
- **Persona**: Usuário autenticado.
- **Fluxo**: backend revoga sessão + limpa cookie + end_session Zitadel → redirect `/`.
- **Estados**: loading, success.

---

## B. App `cms`

Port **3001**. Domain `app.mastersindico.com.br`. Painel principal operacional **multi-persona**. 100+ telas.

### B.1 Morador (M1-M15)

15 telas da jornada morador (fonte: `Inventario-Telas.md` §1). Persona: **Morador** (titular ou dependente; base ou pagante).

#### M1 — Painel do Morador (home)

- **Rota**: `/` (redirect baseado em persona ativa)
- **Conteúdo**: 4 cards — "Meus Condomínios" (M2), "Meu Banco de Talentos" (TEL1), "Vizinhança" (VM1), "Academia" (LMS1 — só morador pagante).
- **Header**: nome + avatar + notificações + seletor de condomínio ativo.
- **Estados**: empty (sem vínculo → CTA "Adicionar meu primeiro condomínio" → M3), loading (skeleton cards).

#### M2 — Selecionar Condomínio

- **Rota**: `/condominios`
- **Conteúdo**: lista de memberships ativas (cards com nome, endereço resumido, unidade, tipo, vínculo, papel).
- **Ações**: [Acessar] → M5; [Adicionar novo] → M3; [Gerenciar acessos] → M13.
- **Estados**: empty, loading (skeleton), error.
- **Regra**: múltiplos condomínios e múltiplas unidades permitidos.

#### M3 — Buscar Condomínio por ID

- **Rota**: `/condominios/adicionar`
- **Campo**: ID alfanumérico 8 chars.
- **Após inserir**: card de preview (nome, endereço completo, CEP) + [Confirmar] → M4.
- **Estados**: busca loading, not-found (mensagem clara), error.
- **Regra**: rate-limit anti-scraping (backend); confirmação visual reduz fraude.

#### M4 — Cadastro da Unidade

- **Rota**: `/condominios/:condominiumId/cadastro-unidade`
- **Campos**: número unidade, bloco (se aplicável), tipo unidade (residencial/comercial), vínculo (proprietário/inquilino/morador autorizado), papel (titular/dependente).
- **Ações**: [Confirmar] → gera pending_membership → síndico valida.
- **Estados**: submit loading, success (mensagem "Aguardando aprovação"), error (unidade já ocupada).

#### M5 — Home do Condomínio

- **Rota**: `/condominios/:condominiumId`
- **Conteúdo**: 6 cards — Linha do Tempo (M6), Plano Diretor (M8), Eventos (M10), Comunicados (M11), Avaliar Gestão (M12), Meu Vínculo (M13).
- **Header**: nome do condomínio + quick switch.
- **Estados**: loading skeleton, error.

#### M6 — Linha do Tempo

- **Rota**: `/condominios/:condominiumId/timeline`
- **Conteúdo**: feed infinito scroll de entradas publicadas (append-only). Cada entrada: título, data, autor, tipo (26 tipos), ícone de risco (9 níveis).
- **Filtros**: tipo, período, risco, autor.
- **Ações**: [Ver detalhe] → M7.
- **Estados**: empty ("Nenhuma atividade publicada ainda"), loading, error, eof.
- **Regras**: R3 (nada deletado), R2 (leitura — morador não publica).

#### M7 — Detalhe da Publicação

- **Rota**: `/condominios/:condominiumId/timeline/:entryId`
- **Conteúdo**: título, corpo rich-text (DOMPurify sanitized), mídia anexada (imagens, PDF via viewer), adendos linkados, histórico de edições.
- **Ações**: [Voltar], compartilhar (link copy).
- **Estados**: loading, error (404 IDOR se não for do condo do morador).

#### M8 — Plano Diretor (leitura)

- **Rota**: `/condominios/:condominiumId/plano-diretor`
- **Conteúdo**: lista de ações do PD (kanban por status — backlog / em andamento / concluído / adiado).
- **Filtros**: prazo, risco, responsável.
- **Ações**: [Ver detalhe da ação] → M9.
- **Regras**: PD atualizado apenas via evento TimelinePublished (nunca manual).

#### M9 — Detalhe da Ação PD

- **Rota**: `/condominios/:condominiumId/plano-diretor/:actionId`
- **Conteúdo**: título, descrição, prazo, status, risco, responsável, histórico de atualizações (lincado à timeline).
- **Estados**: loading, error.

#### M10 — Eventos

- **Rota**: `/condominios/:condominiumId/eventos`
- **Conteúdo**: 13 tipos de evento (reunião, assembleia, manutenção, visita, etc.). Cards com data + [Ciente] / [Confirmar presença].
- **Estados**: empty, loading, error.
- **Regra**: morador só responde — não cria.

#### M11 — Comunicados

- **Rota**: `/condominios/:condominiumId/comunicados`
- **Conteúdo**: lista cronológica; tracking de leitura (lido/não lido); filtro origem (síndico / empresa validada).
- **Ações**: [Marcar como lido].
- **Estados**: empty, loading, error.

#### M12 — Avaliar Gestão (bimestral)

- **Rota**: `/condominios/:condominiumId/avaliar-gestao`
- **Periodicidade**: obrigatória bimestral.
- **Campos**: 3 perguntas, escala 1-10, anônima.
- **Estados**: already_submitted (bloqueado até próximo ciclo), submit loading, success.
- **Regra**: R9 notifica backend → Timeline (resultado agregado).

#### M13 — Meu Vínculo

- **Rota**: `/conta/vinculos`
- **Conteúdo**: lista de memberships (ativos + encerrados com badge "Encerrado em DD/MM/AAAA").
- **Ações**: [Editar dados unidade] (limitado — só titular), [Gerenciar dependentes] → M14, [Encerrar vínculo] → M15.
- **Estados**: loading, error.
- **Regra**: R4 (dependente não pode alterar status nem remover titular).

#### M14 — Gerenciar Acessos (dependentes)

- **Rota**: `/conta/vinculos/:membershipId/dependentes`
- **Conteúdo**: lista até 2 dependentes por titular. Cards com nome, CPF, data nasc., [Remover].
- **Ações**: [Adicionar dependente] (modal com form).
- **Regra**: máx 2; dependente ≥ 13 anos (política — confirmar).

#### M15 — Encerrar Vínculo

- **Rota**: `/conta/vinculos/:membershipId/encerrar`
- **Conteúdo**: dialog double-confirm "Isso removerá seu acesso ao condomínio X. Você perderá histórico de participação, votações futuras, avaliações. Tem certeza?"
- **Ações**: [Cancelar] / [Encerrar vínculo].
- **Regra**: unidade vazia → remove dependentes automaticamente (backend); audit trail registra.

---

### B.2 Síndico (S1-S31) + Compliance (C1-C11)

42 telas. Persona: **Síndico N1/N2/N3** (+ Subsíndico + Conselho via ABAC).

#### S1 — Entrada da Governança (pós-login)

- **Rota**: `/` (quando role síndico ativa)
- **Conteúdo**: onboarding se primeira vez; senão redirect para S2.

#### S2 — Meus Condomínios

- **Rota**: `/condominios`
- **Conteúdo**: lista até **15 condomínios** (Decision 11). Ativos ordenados primeiro; encerrados com badge.
- **Ações**: [Acessar] → S6; [Cadastrar novo] → S3.
- **Regra**: máx 15 ativos (D-###).

#### S3 — Cadastrar Condomínio (Endereço)

- **Rota**: `/condominios/novo`
- **Step 1**: CEP (via ViaCEP) → preenche logradouro, bairro, cidade, estado; número; complemento; nome do condomínio.

#### S4 — Dados da Gestão (Mandato)

- **Rota**: `/condominios/novo/mandato`
- **Step 2**: data início mandato, data fim (estimada), tipo síndico (morador/profissional), documento de posse (upload PDF assinado).
- **Regra**: mandato define auto-lock após término (R6 continuidade).

#### S5 — Cadastro / Vínculo Empresa Síndica

- **Rota**: `/condominios/:id/empresa-sindica`
- **Conteúdo**: se síndico profissional → vincula empresa síndica (CNPJ + convite).
- **Toggles permissões** (5 parametrizáveis): visualizar publicações, editar, ocultar, validar execuções, validar comunicados.
- **Audit trail** obrigatório.

#### S6 — Home da Governança (central do síndico)

- **Rota**: `/condominios/:id`
- **Conteúdo**: **10 cards** (dashboard cross-módulo):
  1. Plano Diretor (S7)
  2. Linha do Tempo (S11)
  3. Eventos (S15)
  4. Connect Me (S18)
  5. Registros de Execução (S21)
  6. Comunicados (S23)
  7. Validações Pendentes (S26) — com badge de quantidade
  8. Dashboard (S27)
  9. Compliance (S28/C1)
  10. Modo Assembleia (link app `assembly`)
- **Indicadores em tempo real**: próximas assembleias, pendências, último comunicado, score compliance.
- **Regra**: tela-pivô do ecossistema.

#### S7-S10 — Plano Diretor (4 telas)

- **S7 Home PD** `/condominios/:id/plano-diretor` — kanban + filtros
- **S8 Cadastrar Ação PD** `/condominios/:id/plano-diretor/novo` — form (título, descrição, prazo, risco, responsável, dependências)
- **S9 Lista Ações PD** `/condominios/:id/plano-diretor/lista` — view tabela
- **S10 Detalhe Ação PD** `/condominios/:id/plano-diretor/:actionId` — editar (gera nova versão); histórico

#### S11-S14 — Linha do Tempo (4 telas)

- **S11 Timeline** `/condominios/:id/timeline` — feed + filtros
- **S12 Criar Atividade** `/condominios/:id/timeline/novo` — 26 tipos, 9 níveis de risco, voice input (R2)
- **S13 Editar Atividade** `/condominios/:id/timeline/:entryId/editar` — gera **nova versão** linkada (R3 nada deletado)
- **S14 Histórico LT** `/condominios/:id/timeline/:entryId/historico` — trilha de versões

#### S15-S17 — Eventos (3 telas)

- **S15 Eventos** `/condominios/:id/eventos` — calendário + lista
- **S16 Criar Evento** `/condominios/:id/eventos/novo` — form
- **S17 Lista Eventos** `/condominios/:id/eventos/lista` — filtros avançados

#### S18-S20 — Connect Me (3 telas)

- **S18 Connect Me Hub** `/condominios/:id/connect-me` — 4 fluxos em tabs (Síndico→Empresa, Morador→Síndico, Empresa→Empresa, Síndico→Parceiro)
- **S19 Criar Connect Me** `/condominios/:id/connect-me/novo` — modal LGPD + seleção destinatário + mensagem
- **S20 Connect Me Recebido** `/condominios/:id/connect-me/recebido/:cmId` — ver msg do morador + [Aceitar] / [Recusar motivo estruturado]

#### S21-S22 — Registros de Execução (2 telas)

- **S21 Registros** `/condominios/:id/execucoes` — lista de execuções submetidas por empresas
- **S22 Detalhe Registro** `/condominios/:id/execucoes/:executionId` — [Aprovar → publica Timeline] / [Rejeitar motivo] / [Solicitar ajuste]

#### S23-S25 — Comunicados (3 telas)

- **S23 Comunicados** `/condominios/:id/comunicados` — feed
- **S24 Criar Comunicado** `/condominios/:id/comunicados/novo` — rich-text + voice + destinatários
- **S25 Lista Comunicados** `/condominios/:id/comunicados/lista` — filtros + métricas leitura

#### S26 — Validações Pendentes (hub)

- **Rota**: `/condominios/:id/validacoes`
- **Conteúdo**: tudo que veio de empresa e precisa validação (execuções, comunicados, adendos). Badge de quantidade no S6.
- **Ações**: aprovar/rejeitar/solicitar ajuste em bulk.
- **Regra**: R4 (origem externa sempre pendente).

#### S27 — Dashboard Síndico

- **Rota**: `/condominios/:id/dashboard`
- **Conteúdo**: **13 indicadores** (eventos, participação morador, avaliação média, compliance score, execuções abertas, Connect Me, próximas assembleias, etc.).
- **Filtros**: período, tipo.
- **Exportação**: PDF (Sprint 6).

#### S28-S31 — Compliance Síndico (4 telas principais)

- **S28 Entrada** `/condominios/:id/compliance` — hub
- **S29 Termo de Responsabilidade** `/condominios/:id/compliance/termo` — aceite inicial
- **S30 Declaração Anual** `/condominios/:id/compliance/declaracao` — 1x/ano obrigatória
- **S31 Auditoria** `/condominios/:id/compliance/auditoria` — logs de ações críticas (append-only)

#### C1-C11 — Compliance (11 telas detalhadas)

Ver `Development/Content/03-Modulos/Compliance-Detalhado.md`. Rotas sob `/condominios/:id/compliance/*`:

- **C1 Painel** `/` — status (Em dia / Parcial / Pendente)
- **C2 Declaração Anual** `/declaracao`
- **C3 Assinaturas Obrigatórias** `/assinaturas` — tracking 3 usuários
- **C4 Matriz Conflito Interesse** `/conflito-interesse`
- **C5 Trilha de Auditoria** `/auditoria` — imutável, visível (somente leitura)
- **C6 Imutabilidade e Adendos** `/adendos`
- **C7 Mapa de Risco Consolidado** `/mapa-risco`
- **C8 Conformidade Contratual** `/contratos` — alertas garantia/prazo
- **C9 Registro LGPD** `/lgpd`
- **C10 Score Governança (privado)** `/score`
- **C11 Dossiê Exportável** `/dossie` — PDF gerado

### B.3 Empresa (E1-E16)

16 telas. Persona: **Empresa Plus / Pro** (admin + operador técnico).

#### E1 — Painel Empresarial (home)

- **Rota**: `/empresa` (quando persona empresa ativa)
- **Conteúdo**: **10 cards** + indicadores (Oportunidades, Registrar execução, Atividades técnicas, Comunicados técnicos, Status envios, Dashboard, Perfil, Usuários, Gestão Agência, Marketing Link).
- **Indicadores**: novas oportunidades, propostas aguardando, votações em curso, negócios ativos, execuções pendentes, garantias vencendo, avaliações recebidas.

#### E2-E4 — Connect Me Empresa (3 telas)

- **E2 Oportunidades** `/empresa/oportunidades` — lista (Síndico→Empresa entrando)
- **E3 Recusar Oportunidade** modal em E2 — motivo estruturado (6 opções + justificativa)
- **E4 Detalhe Oportunidade** `/empresa/oportunidades/:cmId` — após "Tenho interesse" revela contato do síndico + LGPD log

#### E5-E9 — Execução (5 telas)

- **E5 Registrar Execução** `/empresa/execucao/novo` — form (tipo, data, descrição, anexos, CEP confirmação)
- **E6 Atividade Técnica** `/empresa/atividades-tecnicas/novo` — variante técnica
- **E7 Comunicados Técnicos** `/empresa/comunicados-tecnicos` — para síndico, vai a validação
- **E8 Status de Envios** `/empresa/envios` — hub de tudo submetido; termos versionados (LGPD log)
- **E9 Detalhe Registro** `/empresa/envios/:executionId` — estados: `rascunho → enviado → aprovado → publicado → bloqueado`; alternativos: `solicitado_ajuste, rejeitado`

#### E10 — Dashboard Empresarial

- **Rota**: `/empresa/dashboard`
- **Indicadores**: conversão Connect Me, execuções aprovadas/total, avaliação média (5 critérios), ranking categoria/região, garantias próximas vencimento.

#### E11 — Perfil Institucional

- **Rota**: `/empresa/perfil`
- **Conteúdo**: logo, nome fantasia, descrição, categorias, certificações, vídeos institucionais, portfolio, áreas de atuação. Edita com trava 90d em vídeos.

#### E12 — Usuários da Empresa

- **Rota**: `/empresa/usuarios`
- **Conteúdo**: lista (admin + operador técnico). Convidar por email.
- **Permissões**: admin (tudo) · operador técnico (apenas registrar execução + comunicados).

#### E13-E16 — Marketing / Agência (4 telas)

- **E13 Gestão de Agência** `/empresa/agencia` — hub (delegação)
- **E14 Convidar Agência** `/empresa/agencia/convidar` — gera token convite
- **E15 Gerenciar Acessos Agência** `/empresa/agencia/acessos` — toggles permissões agência
- **E16 Formulário Marketing Link** `/empresa/marketing-link` — E→Agência expressa intenção (NÃO cria vínculo — só Gestão de Agência cria)

### B.4 Agência de Marketing (MK1-MK8)

8 telas. Persona: **Agência** (actor delegado).

- **MK1 Painel** `/agencia` — cards (Empresas atendidas, Publicar vídeos, Biblioteca, Dashboard, Marketing Link)
- **MK2 Empresas Atendidas** `/agencia/empresas` — lista (após aceitar convite)
- **MK3 Gerenciar Empresa** `/agencia/empresas/:empresaId` — switch de contexto
- **MK4 Publicar Vídeo** `/agencia/empresas/:empresaId/videos/novo` — 12 tipos; checkbox "Autorizar exibição em votação fornecedor"
- **MK5 Biblioteca de Vídeos** `/agencia/empresas/:empresaId/videos` — editar / ocultar (não remove histórico)
- **MK6 Dashboard Empresa** `/agencia/empresas/:empresaId/dashboard` — views, retenção, mais visto
- **MK7 Dashboard Geral** `/agencia/dashboard` — consolidação
- **MK8 Marketing Link Recebido** `/agencia/marketing-link` — solicitações pendentes

**Limitações ABAC (gates frontend + backend)**:
Agência **não acessa**: Connect Me, execuções, comunicados técnicos, Timeline, dados síndicos/moradores, dashboard governança.

### B.5 Vizinhança (VZ/VS/VE/VM — 29 telas)

Reutilizadas principalmente no app `forum` (§D), mas parte do catálogo CMS.

#### Parceiro Vizinhança (VZ1-VZ6)

- **VZ1 Cadastro do Parceiro** `/parceiro/cadastro` — persona `parceiro` (CPF ou CNPJ)
- **VZ2 Perfil do Parceiro** `/parceiro/perfil` — Promoções + Métricas
- **VZ3 Criar Promoção** `/parceiro/promocoes/novo` — 7 tipos (desconto %, desconto fixo, produto grátis, combo, avaliação grátis, brinde, benefício exclusivo) + segmentação (aberta_bairro | exclusiva_condomínio)
- **VZ4 Promoções Ativas** `/parceiro/promocoes` — editar/encerrar
- **VZ5 Métricas** `/parceiro/metricas` — views, ativações, cliques contato, gráficos engajamento
- **VZ6 Histórico** `/parceiro/historico` — promoções encerradas

#### Vizinhança Síndico (VS1-VS10, 10 telas)

- **VS1 Painel** `/vizinhanca` — 4 seções (Explorar, Exclusivas, Indicados, Histórico)
- **VS2 Explorar Parceiros** `/vizinhanca/explorar` — filtros categoria/bairro
- **VS3 Detalhe Parceiro** `/vizinhanca/parceiros/:id`
- **VS4 Detalhe Promoção** `/vizinhanca/promocoes/:id`
- **VS5 Ativar Promoção** modal
- **VS6 Indicar Parceiro Local** `/vizinhanca/indicar` — envia link cadastro
- **VS7 Criar Benefício Exclusivo** `/vizinhanca/beneficios/novo` — só moradores do condo
- **VS8 Benefícios do Condomínio** `/vizinhanca/beneficios` — editar/encerrar
- **VS9 Parceiros Indicados** `/vizinhanca/meus-indicados` — status (convidado/cadastrado/ativo)
- **VS10 Histórico Ativações** `/vizinhanca/historico`

#### Vizinhança Empresa (VE1-VE6)

- **VE1** Painel — só abertas (bairro), NÃO exclusivas
- **VE2-VE6** análogos a VS, sem acesso a exclusivas

#### Vizinhança Morador (VM1-VM7)

- **VM1** Painel/Feed bairro
- **VM2-VM4** detalhe / ativar
- **VM5** Benefícios do meu Condomínio (exclusivos)
- **VM6** Parceiros Indicados pelo Síndico (badge)
- **VM7** Histórico ativações

### B.6 Banco de Talentos (TEL1-TEL11)

11 telas. Persona: **Morador** (vídeo-currículo 90s, lock 90d).

- **TEL1 Boas-vindas** `/banco-talentos`
- **TEL2 Vídeo de Apresentação** `/banco-talentos/video` — upload Mux, 90s, lock 90d
- **TEL3 Perfil Profissional** `/banco-talentos/perfil` — 6 perguntas abertas
- **TEL4 Áreas de Interesse** `/banco-talentos/areas` — 18 áreas
- **TEL5 Dados Pessoais** `/banco-talentos/dados` — CEP (só bairro)
- **TEL6 Disponibilidade** `/banco-talentos/disponibilidade` — modalidade, horários, início
- **TEL7 Pretensão Salarial** `/banco-talentos/pretensao` — mínimo + ideal
- **TEL8 Experiência Profissional** `/banco-talentos/experiencia` — ⚠️ inconsistência 3 vs 5 vínculos (resolver M1)
- **TEL9 Formação + Autorrelato** `/banco-talentos/formacao`
- **TEL10 Consentimento LGPD** `/banco-talentos/lgpd`
- **TEL11 Confirmação** `/banco-talentos/confirmacao`

**Visualização Empresa** (7 funcionalidades sob `/empresa/banco-talentos/*`): Favoritos, Exportar PDF, Histórico busca, Funil, Tags Auto 7 cat, Tags manual, Busca avançada.

### B.7 Admin MS — app dedicada (D-134)

**D-134 (2026-04-25)**: admin owner Master Síndico em **app `apps/admin` dedicada** (porta dev 3010, subdomínio `admin.mastersindico.com.br` atrás de Cloudflare Access SSO+MFA — D-117). Revoga D-072 (admin role transversal). Bundle isolado de `cms`; auditoria forte (todo acesso gera audit log). Ver [[security#10-admin-isolation]] e [[../admin-privilegios]] (banner superseded).

**5 módulos do app**:
1. Dashboard Financeiro (revenue, MRR, churn)
2. Gestão de Usuários (suspender, banir, reset MFA, revisão LGPD)
3. Moderação (vídeos pendentes, denúncias, conteúdo flagged)
4. Configuração de Plataforma (planos, preços, feature flags, ADRs)
5. Broadcast (notificações cross-tenant, comunicados sistema)

**Migration plan incremental** (não big-bang):
- M1: esqueleto + auth Zitadel + Cloudflare Access + Dashboard Financeiro básico
- M2: módulos 2 e 3 (Usuários + Moderação)
- M3: módulos 4 e 5 (Config + Broadcast)

- **AD1 Admin Home** `/admin` — dashboard cross-tenant: métricas plataforma, alertas, filas de moderação
- **AD2 Gestão de Tenants** `/admin/tenants` — lista condomínios; suspender/reativar; audit
- **AD3 Moderação de Conteúdo** `/admin/moderation` — vídeos pendentes (M3+), harassment reports, denúncias Connect Me
- **AD4 Financeiro** `/admin/financial` — Stripe reconciliation; refunds; disputes (role `financial`)
- **AD5 Broadcasts** `/admin/broadcasts` — comunicado global plataforma (role `super_admin`)
- **AD6 Audit Trail** `/admin/audit` — read-only logs cross-tenant
- **AD7 ABAC Override** `/admin/abac` — grant temporário (4-eyes M3+)
- **AD8 Suporte (DSAR)** `/admin/support` — export/delete dados usuário (role `support`)

**Gates**: topbar banner "⚠ Modo Admin · {sub-role}"; toda ação destrutiva double-confirm; MFA obrigatório (M2+); 4-eyes em `AD7` e destrutivas (M3+).

### B.8 Conta + Sistema (SYS1-SYS8)

- **SYS1 Minha Conta** `/conta` — dados pessoais, foto, senha
- **SYS2 Assinatura** `/conta/assinatura` — tier Stripe-style (`trial | base | plus | pro`), trial countdown, upgrade → Stripe Customer Portal
- **SYS3 Dispositivos** `/conta/dispositivos` — lista ativos + [Desconectar]
- **SYS4 Segurança (MFA)** `/conta/seguranca` — TOTP setup (M2+), senha
- **SYS5 Device Mismatch** `/device-mismatch` — (duplicado AUTH7 mas dentro do cms)
- **SYS6 Dados Pessoais (LGPD)** `/conta/dados-pessoais` — exportar JSON + excluir conta (double-confirm)
- **SYS7 Notificações** `/conta/notificacoes` — preferências canal (push, email, SMS)
- **SYS8 Ajuda / Suporte** `/conta/suporte` — chat suporte + FAQ

---

## C. App `lms` (LMS1-LMS15)

Port **3002**. Domain `academia.mastersindico.com.br`. **M3 / pós-lançamento**.

Persona acesso: **Síndico, Empresa, Morador (pagante)**. Parceiro Vizinhança bloqueado (Decision 7).

- **LMS1 Entrada** `/` — 5 seções (Cursos, Treinamentos, Lives, Fórum, Biblioteca)
- **LMS2 Explorar Cursos** `/cursos` — filtros categoria/nível/tipo
- **LMS3 Página do Curso** `/cursos/:id`
- **LMS4 Aula do Curso** `/cursos/:id/aula/:aulaId` — player + progresso
- **LMS5 Conclusão** `/cursos/:id/certificado` — download/compartilhar PDF
- **LMS6 Treinamentos** `/treinamentos`
- **LMS7 Página Treinamento** `/treinamentos/:id`
- **LMS8 Agenda de Lives** `/lives`
- **LMS9 Sala da Live** `/lives/:id/sala` — vídeo + chat + Q&A
- **LMS10 Replay Lives** `/lives/:id/replay`
- **LMS11 Fórum** `/forum` — 7 categorias
- **LMS12 Criar Tópico** `/forum/novo` — empresas também podem
- **LMS13 Discussão Fórum** `/forum/topico/:id`
- **LMS14 Biblioteca** `/biblioteca` — filtros tipo/categoria
- **LMS15 Página Material** `/biblioteca/:id` — baixar/ler online

**Cursos pagos**: direcionam Stripe Checkout (não Mercado Pago).

---

## D. App `forum`

Port **3003**. Domain `vizinhanca.mastersindico.com.br`. **M2**.

Reutiliza telas VZ (Parceiro), VS (Síndico), VE (Empresa), VM (Morador) do §B.5. Roteamento por persona ativa.

**Rota principal**: `/` redireciona para VM1/VS1/VE1/VZ2 conforme persona ativa.

---

## E. App `assembly` (AS1-AS8)

Port **3004**. Domain `assembleia.mastersindico.com.br`. **M3+**.

Persona: **Síndico (presidir)**, **Morador (vota/checkin)**, **Empresa (fornecedor em votação)**.

- **AS1 Configuração** `/assembleias/:id/config` — tipo, modalidade (presencial MVP), quórum, tempo fala
- **AS2 Pauta** `/assembleias/:id/pauta` — drag-and-drop com lock jurídico após publicação
- **AS3 Edital + Procurações** `/assembleias/:id/edital` — gerador PDF + tracking 6m
- **AS4 Simulador de Quórum** `/assembleias/:id/simulador` — cálculo fração ideal
- **AS5 Check-in** `/assembleias/:id/checkin` — app / QR Code / terminal CPF+bloco+unidade
- **AS6 Votação Live** `/assembleias/:id/live` — WebSocket; 4 tipos (simples, fração ideal DECIMAL, fornecedor, eleição); auto-save Redis
- **AS7 Telão (2 áreas)** `/assembleias/:id/telao` — UI real-time < 200ms; modo apresentação (slides)
- **AS8 Ata + Relatórios** `/assembleias/:id/ata` — ata automática; 6 tipos relatório (CSV + PDF); score transparência 0-100; homologação → imutável

**Tela-pivô crítica**: AS7 telão latência < 200ms é hard requirement.

---

## F. App `lp` (PUB1-PUB6)

Port **3005**. Domain `mastersindico.com.br` (apex). **Público**.

- **PUB1 Home** `/` — hero com blob system (navy → gold), CTA "Continuar" → `auth/sign-up`
- **PUB2 Preço / Planos** `/preco` — tier universal (`trial/base/plus/pro`) — **sem slugs persona-specific**
- **PUB3 Sobre** `/sobre` — sobre a Master Síndico
- **PUB4 Blog** `/blog` — artigos SEO governança condominial
- **PUB5 Blog Post** `/blog/:slug`
- **PUB6 Docs / FAQ** `/docs` — perguntas frequentes

**SEO**: SSG via Rsbuild (PUB1-PUB6); meta tags únicas por rota; sitemap.xml; structured data Organization + Article.

---

## Total do catálogo

| App | Telas |
|---|---|
| `auth` | 8 |
| `cms` — Morador | 15 |
| `cms` — Síndico + Compliance | 42 |
| `cms` — Empresa | 16 |
| `cms` — Agência | 8 |
| `cms` — Vizinhança (subset) | 29 |
| `cms` — Banco Talentos | 11 |
| `admin` — Admin MS dedicada (D-134) | 8 |
| `cms` — Conta + Sistema | 8 |
| `lms` | 15 |
| `forum` | (reutiliza VZ/VS/VE/VM) |
| `assembly` | 8 |
| `lp` | 6 |
| **Total único catalogado** | **174** |

Observação: telas VZ/VS/VE/VM aparecem uma vez em `cms` + novamente como rotas canônicas em `forum` — não são telas novas, apenas canonical home varia. Total **≥ 141 telas estruturalmente distintas** (após deduplicação).

---

## Telas-pivô críticas

Telas com **maior acoplamento arquitetural** (merecem cuidado especial):

| Tela | App | Por que crítica |
|---|---|---|
| **S6** — Home Governança | `cms` | Dashboard cross-módulo único do síndico (10 cards + real-time) |
| **S26** — Validações Pendentes | `cms` | Hub centralizado; desacopla empresa/síndico |
| **E1** — Painel Empresarial | `cms` | Simétrica a S6 para empresa (10 cards) |
| **AS7** — Telão Assembleia | `assembly` | Latência < 200ms hard req; 2 monitores |
| **AS6** — Votação Live | `assembly` | Fração ideal DECIMAL; auto-save Redis; reconnect WS |
| **M5** — Home Condomínio | `cms` | Ponto de entrada real do morador |
| **AUTH2** — Sign In | `auth` | Rate-limit + 1-device + fingerprint |
| **C1** — Painel Compliance | `cms` | Gate de encerramento de mandato + dossiê |
| **AD1** — Admin Home | `admin` | D-134 app dedicada; SSO Cloudflare Access + MFA obrigatório |

---

## Regras de visibilidade críticas

Resumo de gates (ABAC + quotas + trial) que impactam telas:

1. **Vídeos de empresa para morador**: escondidos EXCETO durante votação de fornecedor ativa (`autorizar_exibicao_votacao=true`); revogados após votação fechar — sem cache stale.
2. **Connect Me quotas**: Morador base 2/ano, pagante 4/ano; Síndico N1 limitado, N2/N3 crescente; backend calcula, frontend exibe `remainingQuota`.
3. **Trial expiração**: bloqueia ações estratégicas por persona (Síndico 15d, Empresa 7d, Parceiro 30d) — `<TrialBlocker />` UI.
4. **Agência acesso**: bloqueada em Connect Me, execuções, comunicados técnicos, Timeline, dados síndico/morador, dashboard governança.
5. **Parceiro Vizinhança**: bloqueado em LMS (botão "Academia" oculto).
6. **Admin MS**: gates duplos (UI route guard + backend ABAC); MFA obrigatório M2+; 4-eyes em destrutivas M3+.
7. **Empresa Plus → Pro**: Pro abre Connect Me E→E; Plus apenas responde.
8. **Morador Base → Pagante**: Pagante vê vídeo empresa integral (Base apenas 25% preview).
9. **Síndico N2 → N3**: N3 ilimitado em publicação + Connect Me + dashboards; N2 quotas moderadas.
10. **Subsíndico / Conselho**: via scoped permissions ABAC; ocultam encerrar mandato.

---

## Mapa persona ↔ app ↔ grupo de telas

| Persona | Entry | Telas primárias |
|---|---|---|
| Síndico N1/N2/N3 | `auth` → `cms` | S1-S31 + C1-C11 + parte AD (se admin MS), VS1-VS10, AS* |
| Morador (base/pag) | `auth` → `cms` | M1-M15, TEL1-TEL11, VM1-VM7, LMS1-LMS15 (pagante/N1free), AS5-AS6 (vota) |
| Empresa Plus/Pro | `auth` → `cms` | E1-E16, VE1-VE6, LMS (consulta), AS (fornecedor em votação) |
| Agência | `auth` → `cms` | MK1-MK8 |
| Parceiro Vizinhança | `auth` → `cms` → `forum` | VZ1-VZ6 (sem LMS) |
| Admin MS | `auth` → **`apps/admin`** (D-134 — app dedicada, NÃO `cms/admin/*`) | AD1-AD8 + read cross-tenant nas outras rotas com flag (Cloudflare Access SSO+MFA) |
| Público | `lp` (+ `auth/sign-up`) | PUB1-PUB6 |

---

## Jornadas (UI specs por persona — Fase D 2026-04-25)

A partir de 9 PDFs do cliente foram destiladas **8 specs de jornada** que complementam este catálogo macro com **fluxos de tela completos** (estados, ações, campos, regras canônicas referenciadas):

- [[ui-catalog/jornadas/_moc|ui-catalog/jornadas/_moc]] — MOC das jornadas
- [[ui-catalog/jornadas/sindico|sindico]] (S1-S31), [[ui-catalog/jornadas/morador|morador]] (M1-M15), [[ui-catalog/jornadas/empresa|empresa]] (E1-E16), [[ui-catalog/jornadas/agencia-marketing|agencia-marketing]] (MK1-MK8), [[ui-catalog/jornadas/parceiro-vizinhanca|parceiro-vizinhanca]] (VZ+VM), [[ui-catalog/jornadas/onboarding|onboarding]] (4 perfis), [[ui-catalog/jornadas/curriculo-cadastro|curriculo-cadastro]] (M-CV-1 a M-CV-11), [[ui-catalog/jornadas/curriculo-empresa-view|curriculo-empresa-view]] (E-CV-*)

PDFs originais preservados em `60-sources/master-sindico-research/client-material/pdfs/`. Pendências/divergências consolidadas em [[ui-catalog/jornadas/_pendencias-fase-h|_pendencias-fase-h]].

## Links

- [[README]] — overview web
- [[architecture]] — routing TanStack + feature-sliced
- [[patterns]] — Solid primitives + Kobalte + shortcuts
- [[design-system]] — tokens consumidos por estas telas
- [[requirements]] — CWV + a11y + i18n + forms + upload + realtime
- [[security]] — XSS/CSP/IDOR/admin-isolation
- [[tasks]] — FE-### ordenados por sprint
- [[../mobile/ui-catalog]] — paridade mobile (subset M + S + AS + votação)
- [[../../00-product/personas]] — 7 personas
- Fonte origem: `Development/Content/02-Jornadas/Inventario-Telas.md`
- [[_moc]] — MOC do web
