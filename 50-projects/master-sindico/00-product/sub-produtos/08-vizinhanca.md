---
title: "Sub-produto 08 — Vizinhança (hyperlocal condominial)"
type: spec
tags: [sub-produto, master-sindico, v2, vizinhanca, hyperlocal, comercio-local, cupons, fase-8]
source: destilação exaustiva Fase 8 (2026-04-23)
created: 2026-04-23
updated: 2026-04-23
status: destilado
aliases: [vizinhanca, neighborhood, comercio-local, sub-produto-vizinhanca]
---

# Sub-produto 08 — Vizinhança

Rede hyperlocal que conecta **comércio local (Parceiro)** a **moradores** dos condomínios da região, com **curadoria do síndico** (seal "Síndico-aprovado") e **empresas plus/pro** como participantes secundários. **4 jornadas integradas, 29 telas** (VZ-6, VS-10, VE-6, VM-7). **Sistema de Cupons ATIVO** com formato canônico `ID_LOJA(5)+SEQ(3)+PALAVRA(5)` (13 chars) + lock 4h/estabelecimento (D-064 / Q23 fechada).

> Diferença crítica: **Parceiro NÃO presta serviço ao condomínio**. Serviço-ao-condomínio é jornada B2B formal (Connect Me → Proposta → Deal → Execução) operada pelo bounded context Commercial via sub-produto 02 Connect Me. Aqui o Parceiro vende ao **morador como consumidor final**.

---

## 1. Fontes consultadas

| # | Caminho absoluto | Papel na destilação |
|---|---|---|
| 1 | `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/backend/.kiro/specs/master-sindico/requirements/commercial.md` | Canônico Req 27–27.3 + Req 19.3 (Connect Me Síndico→Parceiro) |
| 2 | `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/backend/.kiro/specs/master-sindico/requirements/personas-and-plans.md` | Role `local_company`, trial 30d, quotas |
| 3 | `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/backend/.kiro/specs/master-sindico/requirements/functional-matrix.md` | Matriz tier×role para features Vizinhança |
| 4 | `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/backend/internal/modules/commercial/` | Código Go (verificado — **sem aggregates Partner/NeighborhoodBenefit/Coupon implementados ainda**) |
| 5 | `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/backend/internal/modules/commercial/domain/enums/enums.go` | Enums atuais (CompanyStatus, ConnectMeFlow — `syndic_to_local_company` já existe) |
| 6 | `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/Content/03-Modulos/Vizinhanca-4-jornadas.md` | **Canônico principal das 4 jornadas** |
| 7 | `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/Content/02-Jornadas/Inventario-Telas.md` §5–§8 | 29 telas VZ1-VZ6 / VS1-VS10 / VE1-VE6 / VM1-VM7 |
| 8 | `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/Content/04-Listas-Mestre/Enums-Consolidados.md` §12 | 38 categorias, 7 benefícios, scope, status convite |
| 9 | `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/Content/contents/*MÓDULO VIZINHANÇA*.md` | Jornada Síndico completa (VS1-VS10) |
| 10 | `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/Content/contents/MÓDULO VIZINHANÇA.ini` | Jornada Empresa (VE1-VE6) |
| 11 | `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/_archive/content-discovery-2026-03/modulo-vizinhanca-morador.md` | Jornada Morador (VM1-VM7) |
| 12 | `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/_archive/content-discovery-2026-03/jornada-parceiro-vizinhanca.md` | Jornada Parceiro (VZ1-VZ6) |
| 13 | `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/Content/requirements.md` Req 27/27.1/27.2/27.3, Req 99, Req 100 | v3.0 — Requirements EARS + Sistema de Cupons / Palavra-Chave do Dia |
| 14 | `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/client-vault/02 - Domínios e Requisitos/Domínio - Comercial.md` §Req 27 | Seção canônica Obsidian vault cliente |
| 15 | `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/client-vault/02 - Domínios e Requisitos/Jornadas Vizinhanca.md` | Canvas convertido (nodes + edges VZ↔VS↔VE↔VM) |
| 16 | `master-sindico-v2/STATE.md` D-064 | Confirmação Sistema de Cupons ativo |
| 17 | `master-sindico-v2/STATE.md` D-066/D-067 | Planos globais Stripe-style (trial/base/plus/pro) — sem slugs tipo `local_company_standard` |

---

## 2. Propósito

Fortalecer o **ecossistema hyperlocal** em torno do condomínio: o síndico atua como **conector de confiança** entre moradores e comércio de bairro; parceiros locais (CPF MEI ou CNPJ pequeno) ganham canal direto com público cativo; moradores economizam e descobrem estabelecimentos via curadoria de quem eles já confiam.

Mensagem institucional canônica (transversal às 4 jornadas):

> "O condomínio faz parte do bairro. A comunidade começa no condomínio e se estende pelo bairro."
>
> "A Master Síndico conecta você diretamente com moradores, síndicos e empresas da região."

---

## 3. O que Vizinhança NÃO é

| Confusão comum | Não é porque |
|---|---|
| ❌ Groupon / cupons genéricos | Escopo **hyperlocal** (raio default 2km + filtro por bairro); curadoria do síndico; relação direta parceiro↔morador, não mercado aberto. |
| ❌ iFood / delivery | Não processa pagamento; não gerencia entrega; **ativação do benefício é presencial** ("apresente esta tela no estabelecimento"). |
| ❌ Connect Me formal B2B | Connect Me (sub-produto 02) é jornada formal: síndico contrata empresa para **prestar serviço ao condomínio** (execução, proposta, deal, dupla assinatura). Vizinhança é relação **parceiro ↔ consumidor final (morador)**, sem serviço contratado pelo condomínio. |
| ❌ Catálogo estático de parceiros | Feed é dinâmico com prioridade por proximidade + seal síndico + promoção ativa. Parceiro tem jornada de gestão (criar/editar/encerrar promoção, métricas, histórico). |
| ❌ Sistema de avaliação de parceiro | Avaliação (5 critérios 1-10) é do sub-produto 03 Reputação, aplicada a **empresas formais** (Req 26). Parceiro recebe métricas de engajamento (views, ativações), mas não rating Bronze→Diamante. |

---

## 4. Diferenciais

1. **Hyperlocal com raio configurável** — default 2km a partir do condomínio, ajustável por admin. Combinado com filtro de bairro.
2. **Mini-seal "Síndico-aprovado"** — parceiro ganha badge "Indicado pelo Síndico" via votação informal de síndicos **tier plus+ role=syndic** (reformulação canônica sob D-066/D-067: acabaram os slugs `syndic_n2/n3`; agora é `(role=syndic, plan_tier ∈ {plus, pro})`).
3. **Sistema de Cupons com lock temporal** — formato `ID_LOJA(5)+SEQ(3)+PALAVRA(5)` (13 chars, ex: `00123055PAO`). Um cupom ativo por estabelecimento a cada 4h. Previne spam e cria escassez genuína.
4. **Escopo duplo** — `aberta_bairro` (todo o ecossistema local) vs `exclusiva_condominio` (só moradores daquele condo). Morador pagante vê ambos; empresa só vê aberta_bairro; síndico vê tudo do próprio condomínio.
5. **Gamification via "Palavra-chave do dia"** — parceiro tier pro pode definir palavra diária; morador usa palavra para acessar promoção exclusiva gamificada (Req 100).
6. **Parceiro sem Connect Me aberto** — parceiro **recebe** Connect Me do síndico (Req 19.3 — `syndic_to_local_company`), não envia. Mantém atrito baixo e elimina spam a síndicos.

---

## 5. Personas envolvidas

### 5.1 Parceiro (role `local_company`)

- **Quem é**: comércio local, profissional autônomo (CPF MEI aceito), prestador de bairro. **CNPJ opcional** (Decision 12 do Content canônico).
- **Tenant próprio**: sim — `tenant_type='company'` com marcador `is_neighborhood=true` (ou aggregate `Partner` separado — ver §8).
- **Como entra**: (a) self-service em `App → Vizinhança → Quero ser parceiro` (VZ1); (b) convite do síndico (VS6) com link de cadastro.
- **Trial**: **30 dias** (Req 4.1 / personas-and-plans). Durante trial: tudo visível; **bloqueia** criar promoções, ver métricas, criar campanhas.
- **Planos**: sob D-066/D-067 — tiers globais `trial | base | plus | pro`. Matriz funcional restringe features.
- **Restrição cross-módulo (Decision 7)**: Parceiro **NÃO tem acesso à Academia LMS** — botão oculto na navegação.
- **Pode**: criar/editar/encerrar promoções; emitir cupons (dentro do lock 4h); acessar métricas próprias; receber Connect Me do síndico.
- **Não pode**: publicar comunicados em condomínios; registrar execução de serviços; acessar painel empresarial; enviar Connect Me.

### 5.2 Morador Pagante (role `resident`, `plan_tier=plus`+)

- **Quem é**: morador de condomínio que pagou upgrade (Morador Pagante) para destravar features premium.
- **Acesso a Vizinhança**: matriz funcional confirma — Morador Base **não vê** parceiros (`❌`); **Morador Pagante sim** (`✅`). Linha `Vizinhança | Ver parceiros (bairro) | Pagante ✅` e `Vizinhança | Benefícios exclusivos | Pagante ✅`.
- **Pode**: visualizar parceiros, visualizar promoções, ativar promoções (resgatar cupom), ver parceiros indicados pelo síndico, ver benefícios exclusivos do próprio condomínio.
- **Não pode**: publicar promoções, cadastrar parceiros.

### 5.3 Síndico tier plus+ (role `syndic`, `plan_tier ∈ {plus, pro}`)

- **Emissor de seal "Síndico-aprovado"** — indica parceiros confiáveis a moradores (VS6 → gera convite; aceite cria seal).
- **Cria benefícios exclusivos** (VS7) — promoções em parceria com estabelecimento, visíveis **apenas** para moradores do próprio condomínio.
- **Ativa promoções** (VS5) em nome próprio — gera métrica `actor_type=sindico` para o parceiro.
- **Base** (plan_tier=base) tem acesso somente-leitura (explorar parceiros já cadastrados) — detalhe a confirmar no dual-check da matriz D-066/D-067.

### 5.4 Empresa Plus/Pro (role `enterprise`, `plan_tier ∈ {plus, pro}`)

- **Papel secundário**. Empresa **não cria** promoções nem cadastra parceiros.
- **Pode**: explorar parceiros do bairro (VE2), ver detalhes (VE3), ativar promoções abertas (VE4-VE5), histórico próprio (VE6), entrar em contato com parceiro.
- **Visibilidade dura**: empresa **NÃO vê** promoções `exclusiva_condominio` — apenas `aberta_bairro`.

### 5.5 Admin (role `admin` — privilégio transversal D-058)

- **Métricas de plataforma**: parceiros cadastrados, promoções ativas, promoções por bairro, engajamento médio, ativações separadas por actor_type (morador/síndico/empresa — 3 contadores).
- **Moderação**: suspender parceiro, revogar seal, configurar raio geográfico.
- **Painel Admin MS** é gap aberto (ver STATE D-068).

---

## 6. Tiers × role — matriz funcional de Vizinhança

> Sob D-066/D-067: tier é enum universal `trial | base | plus | pro`. Personas carregam matriz de permissão; não existem slugs tipo `syndic_n1`, `enterprise_plus`, `local_company_standard`.

| Funcionalidade | Trial | Base | Plus | Pro | Admin |
|---|---|---|---|---|---|
| **Morador** ver parceiros do bairro | — | ❌ | ✅ | ✅ | ✅ |
| **Morador** ver benefícios exclusivos do condomínio | — | ❌ | ✅ | ✅ | ✅ |
| **Morador** ativar promoção (resgatar cupom) | — | ❌ | ✅ | ✅ | ✅ |
| **Morador** ver parceiros "Indicados pelo Síndico" | — | ❌ | ✅ | ✅ | ✅ |
| **Morador** ver histórico de ativações próprias | — | ❌ | ✅ | ✅ | ✅ |
| **Síndico** explorar parceiros | ❌ (leitura) | ✅ (leitura) | ✅ | ✅ | ✅ |
| **Síndico** indicar parceiro (VS6) | ❌ | ❌ | ✅ | ✅ | ✅ |
| **Síndico** criar benefício exclusivo (VS7) | ❌ | ❌ | ✅ | ✅ | ✅ |
| **Síndico** emitir seal "Aprovado" | ❌ | ❌ | ✅ | ✅ | ✅ |
| **Síndico** ativar promoção (VS5) | ❌ | ✅ | ✅ | ✅ | ✅ |
| **Empresa Plus** ver parceiros abertas | ❌ | — | ✅ | ✅ | ✅ |
| **Empresa Plus** ativar promoção aberta | ❌ | — | ✅ | ✅ | ✅ |
| **Parceiro** criar perfil (VZ1) | ✅ (leitura) | ✅ | ✅ | ✅ | ✅ |
| **Parceiro** criar promoção | ❌ | ❌ | ✅ | ✅ | ✅ |
| **Parceiro** emitir cupom (lock 4h) | ❌ | ❌ | ✅ | ✅ | ✅ |
| **Parceiro** palavra-chave do dia | ❌ | ❌ | ❌ | ✅ | ✅ |
| **Parceiro** métricas completas | ❌ | limitado | ✅ | ✅ | ✅ |
| **Admin MS** métricas agregadas | — | — | — | — | ✅ |

> **DT aberto**: celular interface de `(role=resident, plan_tier=base)` ↔ Vizinhança na matriz D-067 ainda incompleta (a matriz original 228 linhas usa colunas N1/N2/N3 — reescrita pendente na DT-004). Este quadro acima é **projeção** coerente com D-064/D-066/D-067; dual-check pendente.

---

## 7. Jornadas detalhadas (29 telas)

### 7.1 VZ — Parceiro (6 telas)

Acesso: **App → Vizinhança → "Quero ser parceiro"** OU aceite de convite do síndico (VS6 → link).

| ID | Nome | Conteúdo canônico |
|---|---|---|
| **VZ1** | **Cadastro do Parceiro** | Campos obrigatórios: `nome_estabelecimento`, `categoria_principal`, `subcategoria`, `telefone`, `whatsapp`, `email`, `cep`, `endereco`, `numero`, `bairro`, `cidade`, `descricao`. Opcionais: `instagram`, `logo`. **CNPJ não obrigatório** — CPF MEI basta. Mensagem institucional: "Cadastre seu estabelecimento e ofereça benefícios para quem vive e trabalha no bairro." |
| **VZ2** | **Perfil do Parceiro** | Seções: **Promoções** + **Métricas**. Botões: `Criar promoção` · `Ver métricas` · `Editar perfil`. Exibe: nome, categoria, endereço, contato, descrição. |
| **VZ3** | **Criar Promoção** (Promoção 4h / cupom) | Campos: `titulo`, `descricao`, `tipo` (7 opções), `validade` (data início/término), `termos`. **Segmentação obrigatória**: `aberta_bairro` OU `exclusiva_condominio` (se exclusiva, lista de condomínios registrados). Ao publicar, sistema **emite cupom** no formato `ID_LOJA(5)+SEQ(3)+PALAVRA(5)` e aplica **lock 4h** para aquele `partner_id`. |
| **VZ4** | **Promoções Ativas** | Lista com: `titulo`, `tipo`, `validade`, `visualizacoes`, `ativacoes`. Ações: `Editar promoção` · `Encerrar promoção`. |
| **VZ5** | **Palavra do Dia / Métricas do Parceiro** | Indicadores: `visualizacoes_perfil`, `visualizacoes_promocoes`, `ativacoes`, `cliques_em_contato`. Gráficos: `engajamento_semanal`, `promocoes_mais_ativadas`. **Palavra-chave do dia** (apenas tier pro — Req 100.2): campo editável 1×/dia; desbloqueia acesso à promoção gamificada. Métricas segmentadas por `actor_type: morador | sindico | empresa` (3 contadores — Req 27.3 §21). |
| **VZ6** | **Histórico de Promoções** | Lista imutável: `promocao`, `data_publicacao`, `data_encerramento`, `numero_ativacoes`. Auditoria de redempções. |

**Regras operacionais do Parceiro**:
- ✅ Pode: criar/editar/encerrar promoções; emitir cupom (respeitando lock 4h); editar perfil; ver métricas próprias.
- ❌ Não pode: publicar comunicados em condomínios; registrar execução de serviços; acessar painel empresarial; enviar Connect Me; acessar LMS/Academia.

### 7.2 VS — Síndico (10 telas)

Acesso: **Governança → Vizinhança**.

| ID | Nome | Conteúdo canônico |
|---|---|---|
| **VS1** | **Painel da Vizinhança** | 4 seções: *explorar parceiros do bairro*, *promoções exclusivas do condomínio*, *parceiros indicados*, *histórico de promoções ativadas*. Botões: `Explorar parceiros` · `Indicar parceiro local` · `Criar benefício exclusivo`. Mensagem: "A comunidade começa no condomínio e se estende pelo bairro." |
| **VS2** | **Explorar Parceiros** | Feed de parceiros cadastrados. Card: `nome_estabelecimento`, `categoria`, `bairro`, `promocao_ativa`. Filtros: `categoria`, `bairro`. Botão: `Ver parceiro`. |
| **VS3** | **Detalhe do Parceiro** | Info: nome, categoria, endereço, telefone, descrição, promoções disponíveis. Botões: `Ver promoção` · `Entrar em contato` · `Indicar para moradores`. |
| **VS4** | **Detalhe da Promoção** | Info: titulo, descricao, tipo, validade. Mensagem: "Esta promoção faz parte da rede de vizinhança da Master Síndico." Botão: `Ativar promoção`. |
| **VS5** | **Ativar Promoção** | Tela de apresentação no estabelecimento. Elementos: `nome_estabelecimento`, `nome_promocao`, `validade`. Mensagem: "Apresente esta tela no estabelecimento para ativar seu benefício." **A ativação gera métrica `actor_type=sindico` para o parceiro.** |
| **VS6** | **Indicar Parceiro Local** (Convites) | Formulário para síndico convidar estabelecimento: `nome_estabelecimento`, `categoria`, `telefone`, `endereco`, `observacao`. Botão: `Enviar convite`. Sistema envia link → flow VZ1 de cadastro. Emite `PartnerInvitationCreated` com status inicial `convite_enviado`. |
| **VS7** | **Criar Benefício Exclusivo para Condomínio** | Síndico cria promoção em parceria com estabelecimento (só parceiros ativos). Campos: `selecionar_parceiro`, `titulo`, `descricao`, `tipo_beneficio` (7 opções), `validade` (data_inicio, data_termino). Botão: `Publicar benefício`. **Regra**: promoção aparece **APENAS** para moradores daquele condomínio (`scope=exclusiva_condominio`). |
| **VS8** | **Benefícios do Condomínio** (Moderação) | Lista de promoções exclusivas ativas: `promocao`, `parceiro`, `validade`, `ativacoes`. Ações: `Editar benefício` · `Encerrar benefício`. |
| **VS9** | **Parceiros Indicados pelo Síndico** (Seal) | Lista com status dos convites. Status possíveis: `convite_enviado` → `parceiro_cadastrado` → `parceiro_ativo`. É aqui que o seal "Síndico-aprovado" se materializa (parceiro_ativo é o gatilho do badge visível ao morador). |
| **VS10** | **Histórico de Promoções Ativadas / Métricas Síndico** | Lista: `promocao`, `estabelecimento`, `data_ativacao`. Indicadores: `numero_parceiros_indicados`, `promocoes_exclusivas_ativas`, `ativacoes_beneficios`. Controle de promoções utilizadas. |

**Regras do Síndico**:
- ✅ Pode: explorar; ativar; indicar; criar benefícios exclusivos; emitir/revogar seal.
- ❌ Não pode: editar promoções criadas pelo parceiro (só as exclusivas criadas por si em VS7).

### 7.3 VE — Empresa (6 telas)

Acesso: **Painel Empresarial → Vizinhança**. Limitado a `plan_tier ∈ {plus, pro}`.

| ID | Nome | Conteúdo canônico |
|---|---|---|
| **VE1** | **Painel da Vizinhança para Empresas** | 3 seções: *explorar parceiros*, *promoções disponíveis*, *histórico*. Botões: `Explorar parceiros` · `Ver promoções`. Mensagem: "A comunidade também é formada por empresas que prestam serviços nos condomínios. Explore parceiros locais e benefícios disponíveis na região." |
| **VE2** | **Explorar Parceiros** | Feed com filtros `categoria`, `bairro`. |
| **VE3** | **Detalhe do Parceiro** | Info: nome, categoria, endereço, telefone, descrição, promoções. Botões: `Ver promoção` · `Entrar em contato`. |
| **VE4** | **Detalhe da Promoção** | Info: titulo, descricao, tipo, validade. Botão: `Ativar promoção`. |
| **VE5** | **Ativar Promoção** | Padrão VS5/VM4 — **métrica `actor_type=empresa`**. |
| **VE6** | **Histórico de Promoções Utilizadas** | Lista: promoção, estabelecimento, data_ativacao. |

**Regras da Empresa**:
- ✅ Pode: explorar, visualizar, ativar promoções, entrar em contato.
- ❌ Não pode: criar/editar promoções, cadastrar parceiros.
- ⚠️ **Regra crítica de visibilidade**: Empresa **NÃO vê** promoções `exclusiva_condominio` — só `aberta_bairro`.

### 7.4 VM — Morador (7 telas)

Acesso: **Painel do Morador → Vizinhança**. Requer `plan_tier=plus`+ (Morador Pagante).

| ID | Nome | Conteúdo canônico |
|---|---|---|
| **VM1** | **Painel / Feed do Bairro** | Feed priorizado por `same_neighborhood` + `active_promotions` + `sindico_indicated_partners`. Card: nome, categoria, bairro, promoção ativa. Filtro: `categoria`. Mensagem: "Conheça estabelecimentos do seu bairro que oferecem benefícios para moradores da comunidade." |
| **VM2** | **Detalhe do Parceiro** | Info: nome, categoria, endereço, telefone, descrição, promoções. Botões: `Ver promoção` · `Entrar em contato` · `Ir até local` (integração maps). |
| **VM3** | **Detalhe da Promoção** | Info: titulo, descricao, tipo, validade. Mensagem: "Esta promoção faz parte da rede de vizinhança da Master Síndico." Botão: `Ativar promoção`. |
| **VM4** | **Ativar Promoção** | Padrão. Tracking: `morador_id`, `promocao_id`, `timestamp`. Métrica `actor_type=morador`. |
| **VM5** | **Benefícios do meu Condomínio** | Promoções **exclusivas** criadas pelo síndico em VS7 (parceria com estabelecimento). Mensagem: "Alguns parceiros oferecem benefícios exclusivos para moradores do seu condomínio." Visibilidade: **apenas** moradores do condomínio específico. |
| **VM6** | **Parceiros Indicados pelo Síndico** | Lista com **badge "Indicado pelo Síndico"** (seal). Mensagem: "Estabelecimentos indicados pelo seu síndico para a comunidade." |
| **VM7** | **Histórico de Ativações** | Lista: `promocao`, `estabelecimento`, `data_ativacao`. Permite ao morador revisar cupons resgatados. |

**Regras do Morador**:
- ✅ Pode: visualizar parceiros, visualizar promoções, ativar promoções, ver parceiros indicados, ver benefícios exclusivos do próprio condomínio.
- ❌ Não pode: publicar promoções, cadastrar parceiros.

---

## 8. Aggregates (DDD)

> **Estado atual no código Go** (`backend/internal/modules/commercial/`): **nenhum** dos aggregates abaixo está implementado. Só existe `Company`, `CompanyUser`, `ConnectMeRequest`, `ClosedDeal`, `ExecutionRecord`, `Proposal`, `SupplierVoteSession`, `EmpresaSindicaLink`, `CompanyEvaluation`, `HarassmentReport`, `PostDealComunicado`. Vizinhança é gap arquitetural (ver §19 Gaps).

### 8.1 `Partner` (aggregate root)

Representa o comércio local cadastrado. Sucessor conceitual de `companies` + `is_neighborhood=true` — **decisão aberta** se vira aggregate novo (`Partner`) ou reuso de `Company` com flag. Recomendação desta destilação: **aggregate separado** `Partner` porque:

- Parceiro **não** precisa validação CNPJ (Company exige — Req 18).
- Parceiro **não** tem fluxos de proposta/deal/execução (Company sim).
- Parceiro tem perfil orientado a vitrine (categoria/subcategoria/logo/Instagram) distinto de `company_profiles` (compliance markers, apresentação B2B).

**Campos essenciais**:
- `partner_id` (UUID)
- `tenant_id` (FK identity)
- `nome_estabelecimento`, `categoria_principal` (enum 38), `subcategoria`
- `cpf_or_cnpj` (optional CNPJ, sempre CPF)
- `cep`, `endereco`, `numero`, `bairro`, `cidade`
- `geolocation` (PostGIS `geography(Point, 4326)` OU `latitude NUMERIC(10,7), longitude NUMERIC(10,7)`)
- `telefone`, `whatsapp`, `email`, `instagram` (opt), `logo_url` (opt)
- `descricao` (text)
- `status` (enum `PartnerStatus`: `pending | active | suspended`)
- `is_sindico_approved` (bool — derivado de `Seal` ativo)
- `created_at`, `updated_at`

**Invariantes**:
- CEP válido obrigatório.
- `categoria_principal` ∈ enum 38.
- Um `tenant_id` só pode ter 1 `Partner`.

### 8.2 `NeighborhoodBenefit` (aggregate)

Promoção publicada por Parceiro (`aberta_bairro`) ou pelo Síndico em parceria com Parceiro (`exclusiva_condominio`).

**Campos**:
- `benefit_id` (UUID)
- `partner_id` (FK Partner)
- `created_by_user_id` (author — pode ser parceiro OU síndico)
- `scope` (enum `BenefitScope`: `aberta_bairro | exclusiva_condominio`)
- `condominium_id` (FK, NOT NULL se `scope=exclusiva_condominio`)
- `titulo`, `descricao`, `termos`
- `tipo` (enum `BenefitType`: 7 valores)
- `valido_de`, `valido_ate` (timestamp)
- `status` (enum `BenefitStatus`: `draft | active | ended | expired`)
- `visualizacoes_count`, `ativacoes_count` (contadores desnormalizados)

**Invariantes**:
- `valido_ate > valido_de`.
- Se `scope=exclusiva_condominio` → `condominium_id` NOT NULL e autor tem role=syndic naquele condomínio.
- Se `scope=exclusiva_condominio` e autor=syndic → requer `partner_id` ativo na plataforma.

### 8.3 `Coupon` (aggregate) — **Sistema de Cupons ATIVO (D-064)**

Representa o **código concreto** emitido para cada promoção ativa, com lock temporal e rastreabilidade imutável.

**Campos**:
- `coupon_id` (UUID — chave interna)
- `coupon_code` (VARCHAR(13) — formato canônico, único no mundo)
- `benefit_id` (FK NeighborhoodBenefit)
- `partner_id` (FK Partner — denormalizado para lock)
- `palavra_chave` (VARCHAR(5) — última palavra do code, uppercase)
- `sequence_number` (INT — SEQ dentro do partner)
- `status` (enum `CouponStatus`: `active | redeemed | expired`)
- `issued_at`, `expires_at`
- `redeemed_at`, `redeemed_by_user_id`, `redeemed_by_actor_type` (enum `morador | sindico | empresa`)
- Impl. `unique(partner_id, status='active')` parcial (índice) — garante **1 cupom ativo por estabelecimento**.

**Formato canônico do `coupon_code`**:
```
ID_LOJA(5) + SEQ(3) + PALAVRA(5) = 13 chars

Exemplo: "00123055PAO" + 2 chars padding → "00123055PAO12"
         └─┬──┘└┬─┘└─┬─┘
          store seq  word
```

> ⚠️ **Ambiguidade a dual-check**: exemplo original `00123055PAO` tem 11 chars, não 13. Interpretação mais coerente com a spec: `ID_LOJA=00123` (5), `SEQ=055` (3), `PALAVRA=PAO` + padding para 5 → `PAOXX` OU a palavra é livre 3-5 chars e o total é ≤13. **Registrar como A-### na AUDIT para usuário confirmar**: ou (a) PALAVRA é exatamente 5 chars (upper-case, padding com X se shorter), ou (b) total máx 13 mas PALAVRA é livre 3-5 chars.

### 8.4 `PartnerInvitation` (aggregate)

Convite emitido pelo síndico em VS6 para cadastrar novo parceiro.

**Campos**:
- `invitation_id` (UUID)
- `issued_by_syndic_id` (FK)
- `condominium_id` (FK — contexto de quem indica)
- `nome_estabelecimento`, `categoria`, `telefone`, `endereco`, `observacao`
- `invitation_link_token` (único, TTL ex: 30 dias)
- `status` (enum `PartnerInvitationStatus`: `convite_enviado | parceiro_cadastrado | parceiro_ativo`)
- `accepted_at`, `resulting_partner_id` (FK Partner, nullable)

**Invariantes**:
- Apenas `role=syndic` e `plan_tier ∈ {plus,pro}` podem criar.
- `status` progride monotonicamente.
- Ao criar `Partner` via link → `status='parceiro_cadastrado'`; ao ativar Partner → `status='parceiro_ativo'`.

### 8.5 `Seal` (aggregate) — "Síndico-aprovado"

Badge exibido aos moradores indicando que o síndico do **seu** condomínio endossa o parceiro.

**Campos**:
- `seal_id` (UUID)
- `partner_id` (FK Partner)
- `granted_by_syndic_id` (FK)
- `condominium_id` (FK — seal é *por condomínio*)
- `granted_at`, `revoked_at` (nullable)
- `reason_revoked` (text, nullable)
- `status` (enum `SealStatus`: `granted | revoked`)

**Invariantes**:
- Apenas `role=syndic ∧ plan_tier ∈ {plus, pro}` pode conceder (D-067).
- Rastreável: quem concedeu, quando, motivo se revogado.
- **Revogável**: `revoked_at` preenchido → seal não exibe mais.
- `unique(partner_id, condominium_id, status='granted')` (um seal ativo por pair).

---

## 9. Enums canônicos

### 9.1 `PartnerCategory` — 38 em 12 grupos

Fonte canônica: `Content/04-Listas-Mestre/Enums-Consolidados.md §12`.

| Grupo | Qtd | Valores |
|---|---|---|
| **Alimentação** | 9 | `padaria`, `restaurante`, `lanchonete`, `cafeteria`, `pizzaria`, `hamburgueria`, `sorveteria`, `doceria`, `acai` |
| **Mercado** | 3 | `supermercado`, `hortifruti`, `adega` |
| **Saúde** | 1 | `farmacia` |
| **Pet** | 3 | `pet_shop`, `clinica_veterinaria`, `banho_tosa` |
| **Academia** | 4 | `academia`, `crossfit`, `pilates`, `yoga` |
| **Estética** | 3 | `salao_beleza`, `barbearia`, `spa` |
| **Serviços Domésticos** | 3 | `lavanderia`, `costureira`, `sapateiro` |
| **Profissionais** | 5 | `eletricista`, `encanador`, `chaveiro`, `marceneiro`, `pintor` |
| **Assistência Técnica** | 2 | `assistencia_celular`, `assistencia_computadores` |
| **Automotivo** | 2 | `oficina`, `lava_jato` |
| **Cursos** | 2 | `cursos_idiomas`, `reforco_escolar` |
| **Outros** | 1 | `outros_servicos` |
| **Total** | **38** | |

### 9.2 `BenefitType` — 7

```
desconto_percentual · desconto_fixo · produto_gratuito · combo_promocional
avaliacao_gratuita · brinde · beneficio_exclusivo
```

### 9.3 `BenefitScope` — 2

```
aberta_bairro | exclusiva_condominio
```

### 9.4 `PartnerInvitationStatus` — 3

```
convite_enviado → parceiro_cadastrado → parceiro_ativo
```

### 9.5 Demais enums novos (propostos)

- `PartnerStatus`: `pending | active | suspended`
- `BenefitStatus`: `draft | active | ended | expired`
- `CouponStatus`: `active | redeemed | expired`
- `SealStatus`: `granted | revoked`
- `ActivationActorType`: `morador | sindico | empresa` (para métricas segmentadas — Req 27.3 §21)

---

## 10. Sistema de Cupons (seção dedicada — ATIVO D-064)

> Q23 foi reaberta e confirmada **ATIVO** pelo usuário (STATE D-064). Especificação completa abaixo.

### 10.1 Formato

```
coupon_code = ID_LOJA(5) + SEQ(3) + PALAVRA(5)  — total 13 chars
```

| Componente | Tamanho | Semântica |
|---|---|---|
| `ID_LOJA` | 5 chars | Identificador numérico zero-padded do `Partner` (ex: `00123`). Derivado de `partner.numeric_id` (sequence PG por tenant). |
| `SEQ` | 3 chars | Sequência zero-padded do cupom dentro do partner (ex: `055`). Reseta a cada 999 → necessita estratégia de rollover (a definir). |
| `PALAVRA` | 5 chars | Palavra-chave mnemônica **uppercase** definida pelo parceiro ao criar promoção (ex: `PAO`, `PIZZA`). Se < 5 chars, padding com `X`. |

**Exemplo**: `00123055PAOXX` (parceiro 123, cupom 55, palavra "PAO" padded).

> ⚠️ **Gap de consistência**: o exemplo original no briefing (`00123055PAO`) tem 11 chars — **a ser confirmado com usuário** se palavra é sempre exatamente 5 (com padding) OU se total é variável até 13. Registrar como A-### no AUDIT pro dual-check.

### 10.2 Trava temporal (lock 4h)

**Regra dura**:
> Cada `partner_id` pode ter **no máximo 1 cupom com status=active por vez**. Enquanto houver cupom ativo, tentativas de emitir novo cupom retornam `409 Conflict` com `retry_after` indicando tempo restante.

**Implementação sugerida**:
- Índice parcial único: `UNIQUE (partner_id) WHERE status='active'`.
- Redis lock adicional: `ms:coupon:lock:{partner_id}` com TTL 4h — confirma janela mesmo se cupom foi `redeemed` antes.
- `expires_at = issued_at + 4h` default (configurável por admin; pode ser reduzido pelo parceiro mas nunca estendido).

### 10.3 Palavra-chave do dia (gamification — tier pro)

- Feature exclusiva `plan_tier=pro` do Parceiro (Req 100.2).
- Parceiro define palavra 1×/dia (rotação diária obrigatória).
- Morador digita palavra correta na tela VM3 para desbloquear promoção exclusiva oculta.
- Emite evento `coupon.daily_keyword.guessed` quando morador acerta.
- Previne abuso: rate-limit por morador (ex: 5 tentativas/dia em qualquer parceiro).

### 10.4 Histórico imutável de redempções

Tabela `coupon_redemptions` (append-only):

| Campo | Tipo |
|---|---|
| `redemption_id` | UUID PK |
| `coupon_id` | FK Coupon |
| `coupon_code` | VARCHAR(13) — denormalizado para auditoria |
| `redeemed_at` | TIMESTAMPTZ |
| `redeemed_by_user_id` | UUID |
| `redeemed_by_actor_type` | `morador | sindico | empresa` |
| `partner_id`, `benefit_id` | FK desnormalizados |

- **Sem UPDATE, sem DELETE**. Conforme invariante de imutabilidade do projeto (herdada da Timeline — CLAUDE.md §4).
- Queries de BI usam `redemptions` como source of truth para ativações.
- Rede com evento NATS `neighborhood.coupon.redeemed`.

---

## 11. Seal "Síndico-aprovado"

### 11.1 Emissão

- Gatilho implícito: síndico convida parceiro em VS6 e parceiro aceita + ativa (`PartnerInvitation.status=parceiro_ativo`). Ao virar ativo, sistema **sugere** criar Seal.
- Gatilho explícito: em VS3 (Detalhe do Parceiro), botão "Indicar para moradores" cria Seal.
- **Guardrails**: `role=syndic ∧ plan_tier ∈ {plus, pro}` (D-066/D-067). Base não emite.
- **Escopo**: 1 Seal por par `(partner_id, condominium_id)`. Um parceiro pode ter múltiplos seals (um por condomínio onde é endossado).

### 11.2 Revogação

- Síndico pode revogar a qualquer momento via VS9 (ou tela dedicada futura).
- Requer `reason_revoked` (text não vazio).
- Cria evento `neighborhood.seal.revoked` para auditoria.
- Após revogado, parceiro **deixa de exibir badge** no feed dos moradores daquele condomínio.

### 11.3 Exibição (UX)

- Morador em VM6 vê lista de parceiros com seal ativo do **próprio condomínio**.
- Morador em VM1 (feed) vê badge "Indicado pelo Síndico" nos parceiros com Seal ativo.
- Feed da Vizinhança prioriza parceiros com seal ativo na ordenação (§12).

### 11.4 "Votação informal" (refinamento conceitual)

> O briefing menciona "votação informal síndicos N2+". Sob D-067, **N2/N3 são abolidos**; usar `(role=syndic, plan_tier ∈ {plus, pro})`.
>
> Interpretação canônica: "votação informal" significa que o seal é **unilateralmente concedido por cada síndico para seu condomínio** (não há votação formal entre múltiplos síndicos). O efeito agregado (parceiro tem seal em N condomínios = "popularidade" entre síndicos) pode ser exposto no Painel Admin MS como indicador, mas não altera a semântica do seal individual.

---

## 12. Geolocalização

### 12.1 Opção canônica recomendada

**PostgreSQL + PostGIS** (extensão `postgis`) com coluna `geolocation geography(Point, 4326)`.

- Índice: `CREATE INDEX ON partners USING GIST (geolocation);`
- Query de proximidade: `ST_DWithin(partner.geolocation, condominium.geolocation, :raio_metros)`.
- Default raio: **2000 metros** (2km). Configurável em admin.

### 12.2 Alternativa simplificada (se PostGIS não for aceito)

Colunas `latitude NUMERIC(10,7)`, `longitude NUMERIC(10,7)` + filtro bounding-box em SQL + haversine no domain service. Menos eficiente, mas elimina dependência externa.

> Decisão da extensão fica em ADR futura (gap). Recomendação desta destilação: PostGIS — alinhado ao uso de PostgreSQL 18 no projeto.

### 12.3 Priorização do feed (VM1)

Ordenação composta:
1. `is_sindico_approved=true` **E** condomínio do morador → topo absoluto (VM6 / badge).
2. `has_active_promotion=true` **E** dentro do raio de 2km.
3. Dentro do raio de 2km (sem promoção).
4. Mesmo bairro (mas fora do raio estrito).

---

## 13. Regras de negócio

| # | Regra | Enforcement |
|---|---|---|
| R-VZ-01 | 1 cupom ativo por `partner_id` por vez (lock 4h) | Índice parcial único + Redis lock |
| R-VZ-02 | `coupon_code` formato estrito 13 chars `ID_LOJA(5)+SEQ(3)+PALAVRA(5)` | Domain VO com regex + check constraint |
| R-VZ-03 | Promoção `exclusiva_condominio` visível **apenas** a moradores daquele condomínio | Query ABAC `commercial.exclusive_benefit.view` com `unit.condominium_id == benefit.condominium_id` |
| R-VZ-04 | Promoção `aberta_bairro` visível a moradores + síndicos + empresas dentro do raio | ABAC + ST_DWithin |
| R-VZ-05 | Empresa **nunca** vê `exclusiva_condominio` | ABAC `commercial.exclusive_benefit.view` rejeita `role=enterprise` |
| R-VZ-06 | Parceiro não envia Connect Me (só recebe) | ABAC `commercial.connect_me.create` rejeita `role=local_company` |
| R-VZ-07 | Parceiro não acessa LMS (Decision 7) | Navegação oculta botão + ABAC `lms.access` rejeita `role=local_company` |
| R-VZ-08 | Seal concedível só por `role=syndic ∧ plan_tier ∈ {plus, pro}` | ABAC `commercial.seal.grant` |
| R-VZ-09 | Seal revogável a qualquer momento pelo emissor | ABAC `commercial.seal.revoke` |
| R-VZ-10 | `PartnerInvitation.status` monotônico (não regride) | State machine na entity (guard clauses) |
| R-VZ-11 | Benefício `exclusiva_condominio` requer `condominium_id` NOT NULL | CHECK constraint SQL + invariante de domínio |
| R-VZ-12 | Ativação gera métrica segmentada por `actor_type` | Evento `neighborhood.promotion.activated` carrega `actor_type` |
| R-VZ-13 | Parceiro em trial (30d) não cria promoções nem emite cupom | Trial rules service bloqueia |
| R-VZ-14 | `categoria_principal` ∈ enum 38 valores | CHECK constraint + domain enum |
| R-VZ-15 | Palavra-chave do dia: 1×/dia por parceiro, só tier pro | Domain service + quota Redis |
| R-VZ-16 | Redempção é append-only (sem update/delete) | Invariante imutável da tabela `coupon_redemptions` |

---

## 14. Invariantes de domínio

1. **Cupom único ativo por Partner** — nunca 2 cupons `status=active` para mesmo `partner_id`.
2. **Formato de cupom rígido** — `coupon_code` sempre 13 chars; regex validado em VO; rejeitar qualquer input diferente.
3. **Lock de 4h é duro** — não existe bypass. Admin pode **forçar expirar** um cupom (mudando status para `expired`), mas não cria outro dentro da janela.
4. **Seal requer tier plus+** — impossível conceder seal estando em base/trial.
5. **Escopo de benefício imutável após publicação** — `scope` não muda entre `aberta_bairro` e `exclusiva_condominio` após `status=active`.
6. **Redempção nunca desfaz** — cupom `redeemed` não volta a `active`.
7. **Partner sem plano ativo não emite cupom** — bloqueio em billing service.
8. **Empresa não vê exclusivas** — regra dura, testável em ABAC matrix.

---

## 15. State machines

### 15.1 `Coupon`

```
         (publish promo)
draft ──────────────▶ active
                       │
                       ├─(redeemed_by user)──▶ redeemed  [terminal]
                       │
                       └─(expires_at passes OR admin force-expire)──▶ expired  [terminal]
```

- De `active`: transições só para `redeemed` ou `expired`. Terminais.

### 15.2 `Partner`

```
         (VZ1 complete)
draft ──────────────▶ active
                       │
                       ├─(admin suspend)──▶ suspended
                       │                      │
                       │                      └─(admin reactivate)──▶ active
                       │
                       └─(admin block)──▶ blocked  [terminal]
```

### 15.3 `Seal`

```
         (VS6 partner_ativo OR VS3 action)
(none) ──────────────▶ granted
                       │
                       └─(syndic revoke)──▶ revoked  [terminal]
```

### 15.4 `PartnerInvitation`

```
(VS6 submit) ──▶ convite_enviado ──(parceiro aceita link)──▶ parceiro_cadastrado ──(parceiro ativa perfil)──▶ parceiro_ativo  [terminal]
```

### 15.5 `NeighborhoodBenefit`

```
draft ──(publish)──▶ active ──(end manually OR valido_ate passes)──▶ ended/expired  [terminal]
```

---

## 16. Domain events

Convenção NATS: `neighborhood.<aggregate>.<verb>` (JetStream, stream `commercial`).

| Evento | Payload-chave | Consumidores |
|---|---|---|
| `neighborhood.partner.registered` | `partner_id, tenant_id, categoria, bairro, city, geolocation` | Search index, métrica admin |
| `neighborhood.partner.activated` | `partner_id` | Notificação síndicos do bairro |
| `neighborhood.partner.suspended` | `partner_id, reason` | Search index (remove) |
| `neighborhood.benefit.created` | `benefit_id, partner_id, scope, condominium_id?` | Notificação push moradores elegíveis |
| `neighborhood.benefit.ended` | `benefit_id` | Search index, analytics |
| `neighborhood.coupon.issued` | `coupon_id, coupon_code, partner_id, expires_at` | Lock tracker, métrica, push |
| `neighborhood.coupon.redeemed` | `coupon_id, coupon_code, partner_id, redeemed_by_user_id, actor_type` | Métrica do parceiro, histórico, analytics |
| `neighborhood.coupon.expired` | `coupon_id, partner_id` | Lock release |
| `neighborhood.exclusive_benefit.created` | `benefit_id, syndic_id, condominium_id, partner_id` | Notificação restrita aos moradores daquele condo |
| `neighborhood.promotion.activated` | `promotion_id, actor_type, actor_id` | Métrica separada por actor_type |
| `neighborhood.invitation.sent` | `invitation_id, syndic_id, email` | Email provider (envio de link) |
| `neighborhood.invitation.accepted` | `invitation_id, resulting_partner_id` | Atualiza VS9, notifica síndico |
| `neighborhood.seal.granted` | `seal_id, partner_id, syndic_id, condominium_id` | Feed morador, badge |
| `neighborhood.seal.revoked` | `seal_id, reason` | Feed morador (remove badge), auditoria |
| `neighborhood.daily_keyword.set` | `partner_id, palavra, day` | Rotação diária |
| `neighborhood.daily_keyword.guessed` | `morador_id, partner_id, day` | Gamification/analytics |

---

## 17. Dependências cruzadas

| Módulo / Sub-produto | Dependência |
|---|---|
| **01 Governança Institucional** | `Condominium` (tenant do condomínio); `Unit` para determinar se morador pertence ao condomínio (ABAC em VM5); sobreposição para "benefício exclusivo". |
| **02 Connect Me** | Req 19.3 `syndic_to_local_company` (promoção exclusiva via formulário) — alternativa ao VS7 para síndico propor benefício. Enum `ConnectMeFlow.SyndicToLocalCompany` já existe no Go. |
| **03 Reputação** | Seal é **diferente** da reputação Bronze→Diamante. Seal é por-síndico-por-condomínio e booleano (ativo/revogado). Reputação é agregada plataforma-inteira. Parceiro **não tem** reputação formal. |
| **Identity** | `User` com role `local_company` (Zitadel); trial 30d em billing. |
| **Billing** | `Subscription` com `plan_tier` controla: criar promoção (≥ plus), palavra-chave do dia (= pro). Parceiro em trial bloqueia criar promoção. |
| **Shared / Cross-cutting** | NATS JetStream (eventos); Redis (lock 4h + quota daily keyword); PostGIS (geo). |

---

## 18. Estado atual no código Go

### 18.1 Implementado

- `internal/modules/commercial/domain/entities/company.go` — `Company` aggregate (usado hoje pra B2B formal).
- `internal/modules/commercial/domain/entities/connect_me_request.go` — `ConnectMeRequest` cobre os 4 fluxos.
- `internal/modules/commercial/domain/enums/enums.go` — `ConnectMeFlowSyndicToLocalCompany = "syndic_to_local_company"` já está no enum (linha 52). **Único vestígio** de Vizinhança no código.

### 18.2 NÃO implementado (gaps)

Busca `grep -r "neighborhood\|vizinhan\|coupon\|Partner"` em `backend/internal/modules/commercial/` retornou **zero ocorrências** além de:
- `infrastructure/database/generated/models.go` (apenas como token gerado por sqlc — sem tabela correspondente ainda).

**Não existem**:
- Aggregate `Partner`
- Aggregate `NeighborhoodBenefit`
- Aggregate `Coupon`
- Aggregate `PartnerInvitation`
- Aggregate `Seal`
- Enums `PartnerCategory`, `BenefitType`, `BenefitScope`, `PartnerInvitationStatus`, `PartnerStatus`, `BenefitStatus`, `CouponStatus`, `SealStatus`
- Repositories, use cases, mappers, handlers
- Migrations SQL para `partners`, `neighborhood_benefits`, `coupons`, `coupon_redemptions`, `partner_invitations`, `seals`, `partner_visibility`
- Integração PostGIS

### 18.3 Conclusão do §18

Vizinhança é **greenfield no backend**. Todo o aggregate model desta destilação é especificação, não código-existente. Implementação cai em sprint futura (fora de M1 2026-05-08; provavelmente M2/M3).

---

## 19. Gaps identificados

| # | Gap | Severidade | Ação sugerida |
|---|---|---|---|
| G-VZ-01 | Código Go de Vizinhança não existe — toda §8/§9 é spec | 🟡 média | Priorizar em sprint pós-M1; registrar em 05-tasks |
| G-VZ-02 | Ambiguidade no formato do `coupon_code` (exemplo 11 vs spec 13 chars) | 🔴 alta | **Dual-check urgente com usuário** — A-### pendente. Eu tratei como 13 com padding `X` nesta destilação. |
| G-VZ-03 | Matriz funcional oficial usa `syndic_n1/n2/n3` que D-067 aboliu | 🔴 alta | Reescrever matriz (DT-004 aberta) antes de codificar ABAC |
| G-VZ-04 | PostGIS vs lat/long — decisão pendente (ADR) | 🟡 média | ADR-XXXX-postgis-neighborhood.md |
| G-VZ-05 | Rollover de `SEQ` (3 chars = 999 cupons/partner) | 🟢 baixa | Definir estratégia: (a) recicla após 999 + 4h passados; (b) expande para 4 chars (quebra formato) |
| G-VZ-06 | "Votação informal síndicos N2+" — conceito ambíguo | 🟡 média | Redefinido como "seal unilateral por síndico" nesta destilação — dual-check usuário |
| G-VZ-07 | `plan_tier=base` no Morador — Vizinhança acessível? Matriz funcional diz Base=❌ | 🟡 média | Confirmar em DT-004 |
| G-VZ-08 | Empresa em tier `base` (existe?) acessa Vizinhança? | 🟢 baixa | Matriz funcional confirma Empresa tem só Plus/Pro — resolver em reescrita matriz |
| G-VZ-09 | Integração com mapas externos (VM2 "Ir até local") | 🟢 baixa | Decisão entre Google Maps / Mapbox / OSM (provider-agnostic D-056) |
| G-VZ-10 | Mini-seal agregado entre condomínios ("X síndicos aprovam") | 🟢 baixa | Painel Admin MS futuro (D-068) |
| G-VZ-11 | Moderação de promoções ofensivas / SPAM por parceiro | 🟡 média | Reuso infra de `HarassmentReport`? ou novo `BenefitReport`? |
| G-VZ-12 | Parceiro trial 30d X Decision 7 (sem LMS) — conflito com marketing de trial abundante? | 🟢 baixa | Rever UX trial parceiro |
| G-VZ-13 | Raio 2km configurável — por condomínio? por plataforma? | 🟢 baixa | Default plataforma, override admin por condo |

---

## 20. Referências cruzadas

### Documentos v2 (este workspace)
- [[master-sindico-v2/00-product/sub-produtos/02-connect-me]] — especificar Connect Me `syndic_to_local_company` (Req 19.3)
- [[master-sindico-v2/00-product/sub-produtos/01-governanca-institucional]] — condominium + unit (contexto morador)
- [[master-sindico-v2/00-product/sub-produtos/03-reputacao]] — não-dependência (seal ≠ reputação)
- [[master-sindico-v2/STATE]] D-064 (cupom ativo), D-066/D-067 (tiers globais), D-068 (Obsidian vault cliente como fonte)
- [[master-sindico-v2/AUDIT]] — registrar A-### novos: G-VZ-02 (formato cupom), G-VZ-03 (matriz stale), G-VZ-06 (seal votação)

### Vault raiz
- [[../../../../10-knowledge/patterns/transaction-patterns#saga]] — emitir cupom + lock cross-concern cabe Saga?
- `PostGIS` *(extensão Postgres — knowledge a destilar)* (a criar) — geo
- [[vault/10-knowledge/patterns/cqrs]] — comandos vs queries de Vizinhança
- [[vault/50-projects/master-sindico/client-vault/02 - Domínios e Requisitos/Jornadas Vizinhanca]] — canvas original
- [[vault/50-projects/master-sindico/client-vault/02 - Domínios e Requisitos/Domínio - Comercial]] — seção Vizinhança Obsidian vault

### Código e specs .kiro
- `backend/.kiro/specs/master-sindico/requirements/commercial.md` §Req 19.3, §Req 27–27.3 (canônico invariantes)
- `backend/.kiro/specs/master-sindico/requirements/personas-and-plans.md` §Parceiro da Vizinhança, §Trial
- `backend/.kiro/specs/master-sindico/requirements/functional-matrix.md` — tabela tier × role (stale — DT-004)
- `backend/internal/modules/commercial/domain/enums/enums.go` — `ConnectMeFlowSyndicToLocalCompany`

### Content canônico
- `Content/03-Modulos/Vizinhanca-4-jornadas.md` — canônico das 4 jornadas
- `Content/02-Jornadas/Inventario-Telas.md` §5–§8 — 29 telas
- `Content/04-Listas-Mestre/Enums-Consolidados.md` §12 — enums
- `Content/contents/*MÓDULO VIZINHANÇA*.md`, `MÓDULO VIZINHANÇA.ini`, `MÓDULO VIZINHANÇA - MORADOR.pdf`, `Jornada do Parceiro da Vizinhança.pdf` — fontes cliente
- `Content/requirements.md` Req 27–27.3 (EARS v3.0), Req 99 (Benefits Club), Req 100 (Palavra-Chave do Dia)

---

## 21. Checklist de aceite desta destilação

- [x] Todas as 29 telas VZ/VS/VE/VM listadas com conteúdo
- [x] Sistema de Cupons (formato, lock 4h, palavra-chave do dia, histórico imutável) detalhado
- [x] Seal "Síndico-aprovado" reformulado sob D-067 (tier plus+ role=syndic, não N2+)
- [x] Raio geográfico 2km + PostGIS documentado
- [x] Diferença com Connect Me explicitada (consumidor final vs B2B formal)
- [x] Tabela tier × role projetada sob planos globais D-066
- [x] 5 aggregates (Partner, NeighborhoodBenefit, Coupon, PartnerInvitation, Seal) com campos e invariantes
- [x] 5 enums mestres + 4 enums novos propostos
- [x] 16 regras de negócio + 8 invariantes + 5 state machines
- [x] 15 domain events NATS mapeados
- [x] Estado no código Go levantado (Vizinhança = greenfield)
- [x] 13 gaps catalogados para dual-check
- [x] Fontes cruzadas (17 arquivos absolutos) listadas
