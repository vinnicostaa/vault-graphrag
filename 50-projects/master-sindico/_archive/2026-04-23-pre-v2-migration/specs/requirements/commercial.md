---
title: Commercial — Connect Me, Contratos, Marketplace e Banco de Talentos
version: 1.0
status: stable
type: requirement
domain: commercial
tags: [connect-me, proposal, contract, deal, talent-bank, neighborhood, company-registry]
---

# Commercial — Connect Me, Contratos, Marketplace e Banco de Talentos

Domínio de relações comerciais entre síndicos, empresas, moradores e comércio local. Inclui Connect Me (formulário unidirecional), propostas, negócios fechados, registro de empresas e vizinhança.

---

## Req 18 — Registro de Empresa

Criação de tenant com `tenant_type='company'` para prestadores de serviço.

### Campos de cadastro

Ver **`institutional.md` Req 6.3** — Perfil da Empresa (logo, razão social, CNPJ, categorias, compliance markers).

### Status de empresa

- `pending_verification` → `active` → `suspended` → `blocked`

### Requisitos MUST

- Validação e unicidade de CNPJ (via Receita Federal adapter — Sprint 2+)
- Visibilidade por plano: Base visível apenas para Síndico Gestor/Plus/Pro; Plus/Pro visível para todos
- Status: obrigatório, auditável (log de mudanças)
- Regras de conteúdo (sem chamadas comerciais óbvias, sem preços, sem contato — Req 6.3)

### Notas de implementação

- Tabelas: `companies`, `company_profiles`
- Enum: `company_status`
- Implementação atual: `internal/modules/commercial/`

---

## Req 19 — Connect Me: Síndico → Empresa

Formulário unidirecional com dados revelados após interesse da empresa.

### Fluxo

1. Síndico preenche formulário (nome, email, condomínio, descrição da demanda)
2. Empresa recebe notificação "Novo Connect Me"
3. Empresa clica "Tenho Interesse"
4. Dados do síndico são **revelados** apenas neste momento
5. Log LGPD: timestamp, company_id, sindico_id, ip, terms_version

### Requisitos MUST

- Formulário unidirecional (não é chat)
- Dados do síndico ocultos até empresa clicar "Tenho Interesse"
- Auto-close após 48h sem interesse
- Log LGPD obrigatório (rule 10): timestamp, IP, finalidade, versão dos termos
- Anti-assédio: botão "Denunciar" em cada Connect Me + ações admin (warning/suspension/ban)
- Quotas por plano (Req 10): N1: 2/ano, N2: 4/ano, N3: ilimitado
- Quota reseta 1º de janeiro (anual)

### Requisitos SHOULD

- Rascunho salvo em Redis durante preenchimento
- Sugestão de empresas (recomendações baseadas em categoria de serviço)

### Notas de implementação

- Tabelas: `connect_me_requests`, `lgpd_logs`
- Redis: `ms:quota:{userID}:connect_me:{year}`
- Evento NATS: `connect_me.request.created` → notificação à empresa
- Evento: `connect_me.request.interested` → reveal dados + log LGPD

---

## Req 19.1 — Connect Me: Morador → Síndico

Formulário unidirecional **frio** (sem reply, sem histórico).

### Categorias

- `duvida_gestao` — dúvida sobre gestão condominial
- `solicitacao_manutencao` — solicitação de reparo/manutenção
- `reclamacao` — reclamação (ex: barulho, vizinho)
- `sugestao` — sugestão de melhoria
- `outro` — livre

### Urgência

- `baixa`, `media`, `alta`, `urgente`

### Requisitos MUST

- **Frio**: sem reply, sem histórico de conversa, sem thread
- Síndico recebe notificação, pode responder via comunicado (público) ou denegar
- Quotas: Base 2/ano, Pagante 4/ano (anual, reseta 1º janeiro)
- Morador não vê histórico — cada novo Connect Me é iniciativa separada

### Notas de implementação

- Tabelas: `connect_me_resident_requests`
- Sem tabela de respostas (é assimétrico)

---

## Req 19.2 — Connect Me: Empresa → Empresa

Apenas para Plus/Pro. Parceria comercial entre empresas (não é venda).

### Partnership types

- `subcontratacao` — empresa A subcontratar empresa B
- `parceria_tecnica` — colaboração técnica
- `fornecimento_materiais` — empresa B fornece materiais a A
- `indicacao_servicos` — indicação de terceiros
- `outro` — livre

### Requisitos MUST

- Limitado a Plus/Pro (validação de plan_tier)
- Fluxo similar ao Síndico→Empresa (dados ocultos até interesse)
- Quota: conforme contrato (ilimitado para Pro, configurável para Plus)

### Notas de implementação

- Tabelas: `connect_me_enterprise_requests`

---

## Req 19.3 — Connect Me: Síndico → Parceiro da Vizinhança

Promoções e benefícios para condomínio.

### Promotion types

- `desconto_exclusivo` — desconto só para condomínio
- `promocao_dia` — promoção por tempo limitado
- `campanha_sazonal` — natal, ano novo, etc
- `parceria_continua` — relacionamento contínuo
- `outro` — livre

### Requisitos MUST

- Síndico cria benefício exclusivo para seu condomínio
- Parceiro responde com aceite ou contra-proposta
- Visibilidade: apenas para moradores do condomínio (Req 27.2)

### Notas de implementação

- Tabelas: `connect_me_neighborhood_requests`, `neighborhood_benefits`

---

## Req 20 — Propostas

Documento formal com mudança de status imutável após envio.

### Status

- `draft` — rascunho, edições livres
- `sent` — enviada, imutável daí em diante
- `awaiting_validation` — síndico validando (período de análise)
- `validated` — síndico aprovou
- `accepted` / `rejected` — resultado final (imutável — rule 2)

### Requisitos MUST

- **Imutável após `sent`** (rule 2) — único mecanismo de mudança: adendo (Req 25)
- Timestamp de envio, assinatura digital (Sprint 3+)
- Versionamento: caso síndico rejeite, empresa cria nova proposta (v2)
- Validação de data limite (ex: válida por 30 dias)

### Notas de implementação

- Tabelas: `proposals`, `proposal_versions`
- Campo `status`, `sent_at`, `validated_at`
- Adendo: `previous_proposal_id` para rastreabilidade

---

## Req 21 — Votação de Fornecedor

Seleção entre 2+ propostas validadas com quorum e resultado real-time.

### Tipos de quorum

- `simple_majority` — > 50%
- `qualified_majority` — ≥ 2/3
- `unanimous` — 100%

### Requisitos MUST

- Mínimo 2 propostas validadas (status `validated`)
- Votação é parte de Assembleia (Req 57)
- Resultado em tempo real (WebSocket)
- Fração ideal usada como peso de voto (NUMERIC, nunca float — T10)
- Resultado é imutável → adendo para questionar (Req 25)

### Notas de implementação

- Tabelas: `assembly_vendor_votes`, `assembly_voting_results`
- Evento NATS: `assembly.vendor_vote.created` → atualiza contagem real-time

---

## Req 22 — Negócio Fechado (Deal)

Contrato confirmado com dupla assinatura (empresa primeiro, síndico depois).

### Fluxo de assinatura

1. Empresa assina digitalmente (Sprint 3+)
2. Síndico recebe notificação de assinatura pendente
3. Síndico assina digitalmente
4. **Após confirmação: imutável** (rule 2)
5. Auto-publicação na Timeline tipo `registro_de_execucao` (Req 11)

### Requisitos MUST

- Dupla assinatura obrigatória
- Imutável após assinatura completa (rule 2)
- Auto-geração de número único de deal (ex: DL-20240522-001)
- Contato automático de ambas as partes (email Resend)
- Histórico de assinaturas (quem, quando, IP)

### Requisitos SHOULD

- PDF assinado digitalmente armazenado em MinIO (Sprint 3)
- Notificações push/SMS sobre status de assinatura pendente

### Notas de implementação

- Tabelas: `deals`, `deal_signatures`, `deal_status_history`
- Evento NATS: `deal.signed_complete` → auto-publish Timeline

---

## Req 23 — Registro de Execução

Empresa registra serviço realizado; síndico **obrigatoriamente** valida.

### Fluxo

1. Empresa registra: fotos, descrição, datas, custos, métricas
2. Síndico recebe notificação
3. Síndico **DEVE** validar (aceitar ou rejeitar com motivo)
4. Validação → Timeline tipo `registro_de_execucao` (Req 11)
5. Validação desatualiza Plano Diretor status (Req 12)

### Requisitos MUST

- Upload de fotos/vídeos (via Mux para vídeos — Req 29)
- Validação obrigatória — síndico não consegue ignorar
- Motivo obrigatório em caso de rejeição
- Bloqueia pagamento até validação (Sprint 3+)
- Audit trail completo (quem, quando, mudanças)

### Notas de implementação

- Tabelas: `execution_records`, `execution_validations`
- Enum: `validation_status` (pending, approved, rejected)
- Evento NATS: `execution.validated` → update master_plan

---

## Req 23.1 — Atividade Técnica

Empresa contribui para gestão preventiva (diferente de execução sob contrato).

### Exemplos

- Consultoria
- Auditoria
- Treinamento

### Requisitos MUST

- Fluxo similar a Req 23 (registro + validação obrigatória)
- Impacto em Plano Diretor: marca como atividade executada
- Empresa não recebe pagamento direto (não é deal)

### Notas de implementação

- Tabelas: `technical_activities`

---

## Req 24 — Comunicados Pós-Deal

Notificação sobre execução vinculada a deal, com 5 tipos.

### Tipos

1. `execucao_iniciada` — obra começou
2. `execucao_pausada` — obra suspensa
3. `execucao_retomada` — obra resumida
4. `execucao_concluida` — obra finalizada
5. `atraso_informado` — comunicado de atraso com justificativa

### Requisitos MUST

- Automático quando deal entra em certos status (Sprint 3+)
- **Validação obrigatória do síndico**
- Impacta Timeline e Master Plan (Req 11, 12)
- Síndico pode rejeitar com motivo (sem cumprir ciclo de notificação)

### Notas de implementação

- Tabelas: `deal_communications`

---

## Req 25 — Adendo

Único mecanismo de modificação de registros imutáveis.

### Requisitos MUST

- Adendo vinculado a registro anterior (`previous_record_id`)
- Chain de adendos suportado (adendo sobre adendo)
- Timestamp, autor, motivo obrigatórios
- Ambas as partes assinam (empresa + síndico)
- Impacta Timeline (Req 11)

### Exemplos de uso

- Alterar data de entrega de deal
- Corrigir custos de execução
- Adicionar observações a proposta rejeitada

### Notas de implementação

- Tabelas: `amendments`
- Enum: `amendment_type` (deal, execution_record, proposal, etc)
- Evento NATS: `amendment.signed` → update timeline

---

## Req 26 — Avaliação de Empresa

Pesquisa anônima do síndico sobre empresa (5 critérios escala 1-10).

### Critérios

1. Cumprimento de prazos
2. Qualidade do trabalho
3. Comunicação
4. Profissionalismo
5. Adequação de custos

### Requisitos MUST

- **Anônima para empresa** (síndico anônimo)
- Pública para outros síndicos (ver média de avaliação)
- Acionada pós-deal (Sprint 3+)
- Impacta reputação da empresa (rating público)

### Notas de implementação

- Tabelas: `company_assessments`, `assessment_responses`

---

## Req 27 — Vizinhança: Síndico (VS1–VS10)

Dashboard para síndico explorar e criar parcerias com comércio local.

### Funcionalidades

1. **Dashboard** — visão geral de parcerias ativas
2. **Explorar** — busca de parceiros por localização/categoria
3. **Indicar** — síndico recomenda parceiro para comunidade
4. **Criar benefício** — desconto exclusivo, promoção, etc
5. **Histórico** — histórico de todas as parcerias

### Requisitos MUST

- Busca por localização (bairro)
- Filtro por categoria
- Widget de indicação para moradores
- Métricas: cliques, uso de benefício, satisfação

### Notas de implementação

- Tabelas: `neighborhood_partnerships`, `neighborhood_metrics`

---

## Req 27.1 — Vizinhança: Empresa (VE1–VE6)

Dashboard para empresa explorar e **ativar** promoções (não criar).

### Funcionalidades

1. Explorar parcerias já criadas
2. Ativar benefício para período
3. Ver métricas de uso

### Requisitos MUST

- Empresa não pode criar benefícios (síndico cria)
- Apenas ativa/desativa períodos

---

## Req 27.2 — Vizinhança: Morador (VM1–VM7)

Feed de parceiros por bairro com badge de indicação.

### Funcionalidades

1. **Feed por bairro** — listagem de parceiros próximos
2. **Badge "Indicado pelo Síndico"** — visual diferenciador
3. **Seção "Benefícios do meu condomínio"** — promoções exclusivas
4. **Filtro por categoria**
5. **Detalhes do parceiro** — contato, fotos, horários

### Requisitos MUST

- Visibilidade apenas para moradores do condomínio
- Badge diferencia parceiros recomendados vs descobertos

### Notas de implementação

- Tabelas: `neighborhood_visibility` (controle de quem vê o quê)

---

## Req 27.3 — Vizinhança: Parceiro (VZ1–VZ6)

Cadastro simplificado, criação e gestão de promoções, métricas.

### Tipos de benefício (7)

- `desconto_percentual` — ex: 10% de desconto
- `desconto_fixo` — ex: R$ 5 de desconto
- `produto_gratuito` — ex: 1ª consulta grátis
- `combo_promocional` — ex: leve 3 pague 2
- `avaliacao_gratuita` — ex: diagnóstico grátis
- `brinde` — cortesia
- `beneficio_exclusivo` — genérico

### Escopo de promoção

- **Aberta (bairro)** — válida para moradores de vários condomínios do bairro
- **Exclusiva (condomínio)** — válida apenas para um condomínio

### Requisitos MUST

- Parceiro cria promoção (texto + data válida + escopo)
- Métricas: views, cliques, resgates (tracking de código QR/link — Sprint 3)
- Promoção é publicitária (mostra em feed morador)
- Histórico de todas as promoções (ativas + archived)

### Notas de implementação

- Tabelas: `neighborhood_benefits`, `benefit_metrics`

---

## Invariantes de commercial

1. Connect Me é unidirecional — nunca thread/reply
2. Propostas são imutáveis após `sent` — único mecanismo: adendo
3. Negócio Fechado é imutável após dupla assinatura
4. Execução de serviço exige validação obrigatória do síndico
5. Fração ideal em votação usa NUMERIC, nunca float
6. Avaliações são anônimas ao avaliado

---

## Referências cruzadas

- **Req 10** (Identity): quotas Connect Me por persona
- **Req 11** (Institutional): Timeline registra eventos de deal/execução
- **Req 12** (Institutional): Plano Diretor atualiza com execuções validadas
- **Req 29** (Content): vídeos de execução via Mux
- **Req 57** (Assembly): votação de fornecedor
- **Personas e Planos**: `personas-and-plans.md` — Empresa Plus/Pro, quotas
