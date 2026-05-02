---
title: "Domínio - Comercial"
version: "3.0"
date: 2026-03-10
status: "em-revisão"
high_domain: "Comercial"
requisitos: "Req 18–27.3"
total_requisitos: 16
tags:
  - mastersindico
  - dominio
  - comercial
  - connect-me
  - servicos
  - vizinhanca
---

# 💼 Domínio Principal: Comercial

> **High-Domain:** Comercial | **Requisitos:** 18–27.3 | **Telas:** E1–E16, VS1–VS10, VE1–VE6, VZ1–VZ6, VM1–VM7
> A camada cross-tenant. Tudo que cruza fronteiras entre condomínio e empresa vive aqui.
> **Breadcrumbs:** [[00 - Índice Master Síndico|🏠 Início]] > [[Arquitetura do Ecossistema|🏛️ Arquitetura]] > 📄 Domínio - Comercial

---

## Visão do Domínio

![[Dominio Comercial.canvas]]

---

## Requisito 18: Registro de Empresa

> **História do Usuário:** Como empresa, eu quero me registrar na plataforma, para que eu possa oferecer meus serviços a condomínios.

### Critérios de Aceite

| # | Critério |
|---|----------|
| 1 | DEVE criar tenant com `tenant_type='company'` |
| 2 | Campos obrigatórios: `company_name`, `cnpj`, `address`, `phone`, `email`, `service_categories` |
| 3 | DEVE validar formato e unicidade do CNPJ |
| 4 | DEVE suportar múltiplas `service_categories` por empresa |
| 5 | Status: `pending_verification`, `active`, `suspended`, `blocked` |
| 6 | Emitir evento `CompanyCreated` |
| 7 | Upload: logo, fotos institucionais, certificações, licenças |
| 8 | Limite Base: máx. 2 fotos + 1 vídeo (60s máx.) |
| 9 | Sem limite de conteúdo para Plus e Pro |
| 10 | Aceite obrigatório dos termos de serviço |
| 11 | Regras de visibilidade: Base visível apenas para Síndico Gestor/Plus/Pro; Plus/Pro visível para todos |
| 12 | Configuração: `visible_to_moradores` (boolean), `visible_to_empresas` (boolean) |

### Rotas de API (Req 18)

```
POST   /v1/companies                  → Criar empresa
GET    /v1/companies/:id              → Detalhe da empresa
PUT    /v1/companies/:id              → Atualizar dados
POST   /v1/companies/:id/media        → Upload de mídia
GET    /v1/companies/search           → Buscar empresas (filtros)
```

---

## Requisitos 19–19.3: Connect Me (4 Direções)

> **PRINCÍPIO FUNDAMENTAL:** Connect Me é **marketplace unidirecional** com LGPD by design. NÃO é chat, NÃO salva histórico de conversas, NÃO tem reply (exceção: Decision 11 pendente para Morador→Síndico).

### Fluxo Geral

![[Fluxo Connect Me.canvas]]

### Req 19 — Síndico → Empresa (Principal)

| # | Critério |
|---|----------|
| 1 | Formulário unidirecional síndico → empresa |
| 2 | Campos: `service_category`, `description`, `urgency`, `budget_range` |
| 3 | NÃO revela dados do síndico até empresa aceitar |
| 4 | Notifica empresas que atendem a categoria |
| 5 | Ao clicar "Tenho Interesse": revela `nome_sindico`, `telefone`, `email`, `descricao_completa` |
| 6 | Log LGPD: `timestamp`, `company_id`, `sindico_id`, `ip_address`, `terms_version` |
| 7 | Quotas por plano |
| 8 | Status: `open` → `interested` → `proposal_sent` → `em_negociacao` → `closed` / `expired` |
| 9 | Auto-fecha após 48h sem interesse |
| 10 | Emite `ConnectMeRequestCreated`, `ConnectMeInterestShown`, `ConnectMeStatusUpdated` |

**Quotas Connect Me Síndico→Empresa:**

| Plano | Quota |
|-------|-------|
| Síndico N1 | 2/ano |
| Síndico N2 | 4/ano |
| Síndico N3 | Ilimitado |

**Funcionalidade Anti-Assédio:**
- Botão "Denunciar Assédio" para síndicos
- Log: `timestamp`, `company_id`, `sindico_id`, `description`, `evidence`
- Notifica administradores da plataforma
- Ações possíveis: `warning`, `suspension`, `ban`

### Req 19.1 — Morador → Síndico

| Campo | Detalhe |
|-------|---------|
| Campos | `subject`, `description`, `urgency`, `category` |
| Categorias | `duvida_gestao`, `solicitacao_manutencao`, `reclamacao`, `sugestao`, `outro` |
| Urgência | `baixa`, `media`, `alta`, `urgente` |
| Quota Base | 2/ano |
| Quota Pagante | 4/ano |

> [!WARNING]
> **Decision 11 — PENDENTE:** O Req 19.1 descreve: resposta do síndico com `message` e `status_update`, status `em_analise`/`respondido`/`resolvido`, histórico de requisições. Isso é um **mini ticket system**, não formulário unidirecional. **Conflita com o princípio "Connect Me NÃO é chat".** Aguardando decisão do cliente.

### Req 19.2 — Empresa → Empresa

| Campo | Detalhe |
|-------|---------|
| Campos | `service_category`, `description`, `partnership_type`, `urgency` |
| Partnership types | `subcontratacao`, `parceria_tecnica`, `fornecimento_materiais`, `indicacao_servicos`, `outro` |
| Restrição | Apenas planos Plus e Pro |
| Padrão | Igual ao Síndico→Empresa (unidirecional, LGPD, 48h auto-close) |

### Req 19.3 — Síndico → Parceiro da Vizinhança

| Campo | Detalhe |
|-------|---------|
| Campos | `promotion_type`, `description`, `target_audience_size`, `desired_discount`, `urgency` |
| Promotion types | `desconto_exclusivo`, `promocao_dia`, `campanha_sazonal`, `parceria_continua`, `outro` |
| Padrão | Unidirecional, LGPD, 48h auto-close |
| Métrica extra | Conversão: `requests → proposals → activated_promotions` |

### Rotas de API (Req 19)

```
POST   /v1/connect-me                        → Criar requisição
GET    /v1/connect-me/:id                     → Detalhe
POST   /v1/connect-me/:id/interest            → Mostrar interesse
PUT    /v1/connect-me/:id/status              → Atualizar status
POST   /v1/connect-me/:id/report              → Denunciar assédio
GET    /v1/connect-me/opportunities            → Listar oportunidades (empresa)
GET    /v1/connect-me/my-requests              → Minhas requisições
```

> [!WARNING]
> **Gap Crítico:** Quotas são por **/ano** (PDF Matriz Funcional) ou por **/mês**? Algumas seções do requirements mencionam `/month`. Precisa confirmação do cliente.

---

## Requisitos 20–25: Ciclo de Serviço

### Fluxo Completo

![[Fluxo Ciclo Servico.canvas]]

### Req 20 — Propostas

Empresa cria proposta: `title`, `description`, `scope`, `estimated_cost`, `estimated_duration`. Status: `draft` → `sent` → `awaiting_validation` → `validated`/`rejected` → `accepted`. Requer validação do síndico. Imutável após status `sent` (edita apenas em `draft`).

### Req 21 — Votação de Fornecedor

Síndico vincula 2+ propostas validadas a uma votação. Tipos: `simple_majority`, `qualified_majority`, `unanimous`. Quórum calculado por tipo + total de unidades. Cada morador vota 1 vez. Resultado em tempo real para síndico. Fechamento antecipado se quórum + maioria atingidos.

### Req 22 — Negócio Fechado

**Dupla confirmação:** empresa assina primeiro, depois síndico assina. Após ambas assinaturas: `status=confirmed`, registro **imutável**, publicação automática na Timeline. Apenas adendo como modificação.

### Req 23 — Registro de Execução de Serviço

Empresa registra: `condominio_atendido`, `tipo_servico`, `descricao`, `area_impactada`, `natureza_atividade` (preventiva/corretiva/emergencial/estratégica), `status_servico`, `garantia`, orientações, materiais, fotos, vídeos. **Validação obrigatória do síndico** → Timeline.

### Req 23.1 — Atividade Técnica

Empresa contribui para gestão preventiva: `tipo_atividade`, `descricao`, `area_impactada`, `nivel_importancia` (baixo/médio/alto/crítico), `risco_identificado`, `impacto_esperado`, `recomendacao_tecnica`. **Validação obrigatória do síndico** → Timeline.

### Req 24 — Comunicados Pós-Deal

Tipos: `orientacao_tecnica`, `recomendacao_manutencao`, `alerta_preventivo`, `conclusao_servico`, `atualizacao_garantia`. Vinculado a `deal_id`. **Validação obrigatória do síndico** → Timeline + notifica moradores.

### Req 25 — Adendo

Único mecanismo de modificação de registros imutáveis. Campos: `original_record_id`, `reason`, `description`, `changes_summary`. Suporta chain de adendos (adendo de adendo). Autorização do síndico obrigatória.

### Req 26 — Avaliação de Empresa

Síndico avalia empresa após conclusão do serviço. **5 critérios** com escala 1–10:

| Critério | Descrição |
|----------|-----------|
| `qualidade_servico` | Qualidade do serviço prestado |
| `cumprimento_prazo` | Cumprimento de prazos |
| `atendimento` | Qualidade do atendimento |
| `custo_beneficio` | Relação custo-benefício |
| `recomendacao` | Nível de recomendação |

Resultados: agregados e **anônimos** para a empresa, **públicos** para outros síndicos.

---

## Requisitos 27–27.3: Vizinhança (Comércio Local)

> Módulo completo com **4 jornadas** e **29 telas** mapeadas.

### Mapa de Jornadas

![[Jornadas Vizinhanca.canvas]]

### Req 27 — Síndico (VS1–VS10)

| Tela | Nome | Função |
|------|------|--------|
| VS1 | Painel da Vizinhança | Dashboard: explorar, exclusivas, indicados, histórico |
| VS2 | Explorar Parceiros | Feed com filtros categoria/bairro |
| VS3 | Detalhe do Parceiro | Info + promoções + ações |
| VS4 | Detalhe da Promoção | Título, descrição, tipo, validade |
| VS5 | Ativar Promoção | Apresentar no estabelecimento |
| VS6 | Indicar Parceiro | Formulário de convite |
| VS7 | Criar Benefício Exclusivo | Promoção exclusiva para condomínio |
| VS8 | Benefícios do Condomínio | Lista de promoções exclusivas ativas |
| VS9 | Parceiros Indicados | Status dos convites enviados |
| VS10 | Histórico de Promoções | Controle de ativações |

**30+ categorias de parceiro** organizadas: Alimentação (9), Mercado (3), Farmácia, Pet (2), Academia (4), Estética (3), Serviços Domésticos (3), Profissionais (5), Assistência Técnica (2), Automotivo (2), Cursos (2), Outros.

**7 tipos de benefício:** `desconto_percentual`, `desconto_fixo`, `produto_gratuito`, `combo_promocional`, `avaliacao_gratuita`, `brinde`, `beneficio_exclusivo`.

### Req 27.1 — Empresa (VE1–VE6)

Empresa pode: explorar parceiros, visualizar/ativar promoções, contatar parceiros.
Empresa **NÃO pode:** criar, editar ou cadastrar promoções.
Empresa **NÃO vê** promoções exclusivas de condomínio.

### Req 27.2 — Morador (VM1–VM7)

Feed prioriza: mesmo bairro → promoções ativas → parceiros indicados pelo síndico.
Badge "Indicado pelo Síndico" nos parceiros indicados.
Seção "Benefícios do meu condomínio" (promoções exclusivas).
Morador **NÃO pode** publicar ou cadastrar.

### Req 27.3 — Parceiro (VZ1–VZ6)

| Tela | Nome | Função |
|------|------|--------|
| VZ1 | Cadastro | Nome, categoria, telefone, WhatsApp, email, CEP, endereço, bairro, descrição. Instagram e logo opcionais. **CNPJ não obrigatório** — CPF basta |
| VZ2 | Perfil | Dados do estabelecimento, promoções ativas |
| VZ3 | Criar Promoção | Título, descrição, tipo, validade. **Segmentação:** aberta (bairro) ou exclusiva (condomínio) |
| VZ4 | Promoções Ativas | Lista com filtros e status |
| VZ5 | Métricas | Visualizações perfil, views promoções, ativações, cliques contato, engajamento semanal |
| VZ6 | Histórico | Data publicação, encerramento, ativações |

**Regras do Parceiro:**
- ✅ Pode criar/editar/encerrar promoções
- ❌ NÃO pode publicar comunicados em condomínios
- ❌ NÃO pode registrar execução de serviços
- ❌ NÃO pode acessar painel empresarial

---

**Breadcrumbs:** [[00 - Índice Master Síndico|🏠 Início]] · **Anterior:** [[Domínio - Identidade]] · **Próximo:** [[Domínio - Conteúdo]]
