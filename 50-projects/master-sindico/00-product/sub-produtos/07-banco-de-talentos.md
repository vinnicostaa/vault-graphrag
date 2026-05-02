---
title: "07-banco-de-talentos — Sub-produto (destilação Fase 8)"
type: note
tags: [sub-produto, master-sindico, v2, fase-8, banco-de-talentos, content, commercial, talento, vídeo-currículo, lgpd]
created: 2026-04-23
updated: 2026-04-23
status: destilado
aliases: ["Banco de Talentos", "Talent Bank", "Vídeo-currículo"]
---

# 07 — Banco de Talentos

> Destilação exaustiva Fase 8. Catálogo fiel com trechos citados por `arquivo (seção/linha)`. Governo da destilação: `CLAUDE.md` v2 §7.

---

## 1. Fontes consultadas

Fontes canônicas efetivamente lidas nesta destilação (leitura arquivo-por-arquivo, sem grep-only):

### 1.1 Requisitos (.kiro — legado ingerido em 90-ingested)

- `master-sindico-v2/90-ingested/from-vault-50-projects-master-sindico/specs/requirements/commercial.md` (versão 1.0 stable; frontmatter tag `talent-bank`)
  - Sem bloco `Req 28` dedicado — a spec `.kiro` de `commercial` declara o escopo Banco de Talentos no título e no frontmatter (`tags: [... talent-bank ...]`) mas não detalha os 11 requisitos; isso é preenchido por `client-vault/.../System Architecture - Content.md` + Decision 6 + `novo escopo-7.md §1.4, §4.7`.
- `master-sindico-v2/90-ingested/from-vault-50-projects-master-sindico/specs/requirements/content.md` (versão 1.0 stable)
  - Req 29 Vídeos Institucionais — fornece a base de upload Mux + trava trimestral 90d; Banco de Talentos reutiliza a infra (`IVideoProvider`, HLS, MinIO/R2).
- `master-sindico-v2/90-ingested/from-vault-50-projects-master-sindico/specs/requirements/functional-matrix.md` linhas 41-42 e tabela Empresa Plus/Pro: matriz autoritativa Banco Talentos × persona × plano.

### 1.2 System Architecture (client-vault)

- `master-sindico-v2/90-ingested/from-vault-50-projects-master-sindico/client-vault/01 - Arquitetura de Sistema/System Architecture - Content.md` — seção **TalentBankModule (NOVO — Decision 6)** com 11 telas + 7 funcionalidades empresa + schema `curricula`, `curriculum_videos`, `curriculum_professional_profiles`, `curriculum_areas_of_interest`, `curriculum_work_experience`, `curriculum_education`, `company_curriculum_favorites`, `company_curriculum_tags`, `curriculum_search_logs`.
- `master-sindico-v2/90-ingested/from-vault-50-projects-master-sindico/client-vault/01 - Arquitetura de Sistema/System Architecture - Data Model.md` — detalhamento de colunas PostgreSQL para Banco de Talentos.
- `master-sindico-v2/90-ingested/from-source-material-masterSindico-extracted/MasterSindico/01 - Arquitetura de Sistema/Decisões e Pendências.md` — resolução de Q "60s vs 90s" (prevaleceu 90s) + pendência Decision 6 "Banco de Talentos entra em Sprint 1/2? Gap 3 vs 5 vínculos."

### 1.3 Domínios e Requisitos (client-vault)

- `master-sindico-v2/90-ingested/from-vault-50-projects-master-sindico/client-vault/02 - Domínios e Requisitos/Dominio Conteudo.md` — acoplamento de Banco de Talentos em `content` (infra vídeo) + eventos `content.talent.*`.
- `master-sindico-v2/90-ingested/from-vault-50-projects-master-sindico/client-vault/02 - Domínios e Requisitos/Visibilidade Videos.md` — regras de visibilidade (inspiração para privacidade de currículo).
- `master-sindico-v2/90-ingested/from-vault-50-projects-master-sindico/client-vault/02 - Domínios e Requisitos/Dominio Comercial.md` — Banco de Talentos aparece como feature do lado empresa (busca + favoritos + tags).

### 1.4 Enums e Listas Mestre

- `master-sindico-v2/90-ingested/from-vault-50-projects-master-sindico/client-vault/03 - Modelagem e Dados/Enums e Listas Mestre.md` linha 184: `Modalidades de Contratação (Banco de Talentos)` = 9 valores (`clt, pj, estagio, temporario, folguista, diarista, freelancer, intermitente, outros`).
- `master-sindico-v2/90-ingested/from-vault-50-projects-master-sindico/client-vault/03 - Modelagem e Dados/Enums e Listas Mestre.md` linha 38: `CNH` = 4 valores (`nao, A, B, AB`).
- `master-sindico-v2/90-ingested/_reconciliacao-dev/05-enums-regras-quebras.md` Categoria 16 (Banco de Talentos): reconciliação canônica dos 8 enums (TalentArea 18, ContractModality 9, AvailabilitySchedule 4, StartAvailability 3, TravelTime 3, CNHType 4, NRCertification 4, EmploymentExitReason 4).

### 1.5 Reconciliação v2

- `master-sindico-v2/90-ingested/_reconciliacao-dev/04-telas-jornadas.md` §9 "Família BT — Banco de Talentos (BT 01-11)" — catálogo canônico de **11 telas BT01..BT11** com descrições de uma linha + cobertura v2 **0%** + empresa view (busca + favoritos + tags + convite via Connect Me especial).
- `master-sindico-v2/90-ingested/_reconciliacao-dev/05-enums-regras-quebras.md` §1 categoria 16 + **Q39** "3 vs 5 vínculos (PDF diz 5; adotar 5 conforme Decision 6)" — **FECHADA 2026-04-24 via [[../../STATE#d-099-—-banco-de-talentos-5-vínculos-profissionais-fecha-q39-sobrescreve-d-060-d-064|D-099 Fase 11]]: 5 vínculos**. PDF original do cliente vence — D-060/D-064 sobrescritas.

### 1.6 Código Go (backend/internal/modules)

- **Não existe código Go neste repositório** (`master-sindico-v2/` é pasta de **remontagem pré-migração** — sem código, só specs+docs+domínio; ver `master-sindico-v2/CLAUDE.md §1` "O que NÃO é: não tem código"). A menção do prompt ao `backend/internal/modules/commercial/` e `backend/internal/modules/content/` refere-se ao backend vivo fora do escopo deste vault. Tratado como **gap** em §18.

### 1.7 STATE v2 — decisões vinculantes

- `master-sindico-v2/STATE.md`:
  - ~~**D-060 — Banco de Talentos: 3 vínculos profissionais**~~ **SOBRESCRITA por D-099 Fase 11 = 5 vínculos** (2026-04-24).
  - ~~**D-064 — Banco de Talentos com 3 vínculos**~~ **SOBRESCRITA por D-099 Fase 11 = 5 vínculos** (2026-04-24). PDF original do cliente "ESTRUTURA DE CADASTRO DE CURRÍCULO" TELA 08 canônico.
  - **D-099 — Banco de Talentos: 5 vínculos profissionais (Q39 fechada na direção PDF do cliente)** — Data 2026-04-24. Alteração futura ainda via matriz funcional + seed.
  - **D-056 — Agnosticismo de provedor** — Mux atrás de `IVideoProvider` (aplica-se ao vídeo-currículo).
  - **D-134 (revoga D-058/D-072) — Admin = `apps/admin` dedicada** — admin modera Banco de Talentos (aprovar/remover currículos sinalizados) via app separada com Cloudflare Access SSO+MFA.
  - **D-066 — Planos globais Stripe-style sem slugs** — matriz funcional Banco Talentos apoia em `PlanTier ∈ {trial, base, plus, pro}` + role.

### 1.8 Content/ (Obsidian do cliente) e Inventario-Telas.md

- `Content/contents/` **não existe neste repositório** (caminho citado pelo prompt é do projeto cliente externo; o material equivalente já foi ingerido em `master-sindico-v2/90-ingested/from-source-material-masterSindico-extracted/MasterSindico/**` + `client-vault/**`).
- `Content/02-Jornadas/Inventario-Telas.md` **não existe** como arquivo autônomo neste repositório (equivalente canônico está em `master-sindico-v2/90-ingested/_reconciliacao-dev/04-telas-jornadas.md` §9 — usado como fonte da seção §8 abaixo). Marcado como **gap de ingestão** em §18.
- `Content/requirements.md` **não existe neste repositório** (ingerido em `90-ingested/from-vault-50-projects-master-sindico/specs/requirements/*.md` particionado por BC). Banco de Talentos aparece em `commercial.md` (título+tags) e `content.md` (Req 29 infra).

---

## 2. Propósito

Banco de Talentos é o sub-produto que **conecta morador-candidato a empresa condominial contratante** dentro da plataforma Master Síndico, via **vídeo-currículo curto (90s)** e **busca estruturada por área/região**.

**Tese central**: o segmento de talentos condominiais é **sub-atendido** pelo mercado de recrutamento tradicional:

- **Doméstica** buscando diarista/mensalista em condomínios do bairro.
- **Porteiro** migrando de um prédio pra outro sem precisar mandar currículo PDF via WhatsApp.
- **Técnico de manutenção** (elétrico, hidráulico, obras) apresentando case em vídeo curto.
- **Zeladoria, jardinagem, limpeza de reservatório, brigada de incêndio (NR-23/35), controle de pragas** — ocupações condominiais específicas sem portal vertical.

Plataformas generalistas (LinkedIn, Infojobs, Catho) não capturam este público: preferem texto+conexões+endorsements, têm atrito de cadastro alto, exigem perfil profissional extenso. Classificados informais (WhatsApp do bloco, Facebook Grupos, sites paroquiais) são **sem triagem, sem histórico auditável, sem verificação mínima**.

**Solução Master Síndico**: o morador pagante publica um vídeo-currículo de **90 segundos** (alto-impacto, baixo esforço) + ficha estruturada (18 áreas de atuação condominial, disponibilidade, CNH, NR, experiência condominial, pretensão salarial). Empresas Plus/Pro buscam por filtros objetivos (área + região + CEP + raio + modalidade + horário). Match.

**Quem atende**: empresas de **categoria condominial** já presentes na plataforma via `commercial` (administradoras, empresas de limpeza, manutenção, segurança, obras, jardinagem). A busca é **B2B2C**: empresa contrata; morador trabalha no condomínio dela (ou de outros condomínios da carteira).

**O que não é**:
- ❌ **Plataforma de emprego aberta** — só morador **pagante** publica; só empresa **Plus/Pro/Marketing/Vizinhança** busca. Parceiro da Vizinhança **não acessa** (matriz funcional linha Parceiro Vizinhança).
- ❌ **LinkedIn condominial** — sem endorsements, sem likes públicos, sem rede social, sem feed. Assimétrico (morador publica ↔ empresa busca; nenhum feed público lista currículos).
- ❌ **Catho/Infojobs com cara de condomínio** — foco em **vídeo-currículo** como mídia primária, não em CV texto estilo Europass.
- ❌ **Sistema de contratação completo** — não fecha contrato de trabalho, não gera CLT, não emite holerite. Apenas **match inicial**; contratação formal acontece fora da plataforma (ou reaproveitando Connect Me Empresa→Morador fluxo especial; ver §15).

---

## 3. Diferencial

| Eixo | Master Síndico (Banco de Talentos) | Mercado generalista |
|---|---|---|
| **Mídia primária** | Vídeo-currículo **90s** | Texto + foto (CV PDF) |
| **Lock pós-upload** | **90 dias** no vídeo (igual Req 29) | Sem lock |
| **Domínio da vaga** | 18 áreas condominiais-específicas (TalentArea enum) | Genérico/horizontal |
| **Endorsements** | **Proibido** (anti-fraude) | Centrais (LinkedIn) |
| **Likes públicos** | **Proibido** (anti-fraude) | Comuns |
| **Auto-tags empresa** | 7 categorias calculadas | Tag manual apenas |
| **Audit trail** | `curriculum_search_logs` (LGPD) registra cada busca | Opaco |
| **Convite formal** | Via Connect Me Empresa→Morador fluxo especial (§15) | Mensagem direta |
| **Raio geográfico** | Filtro CEP + raio (morador em bairro do condomínio) | Cidade grande |
| **Segmento** | **Candidato sub-atendido** (doméstica, porteiro, técnico) | Knowledge worker |

**Por que vídeo 90s?**
- **Impacto emocional**: empresa enxerga a pessoa (dicção, postura, aparência profissional) em 90s — alinhado à realidade de cargos presenciais (recepção, portaria, limpeza).
- **Barreira baixa de criação**: morador filma uma vez no celular; não precisa de software de edição, templates, fontes, portfólio.
- **Anti-spam**: criar currículo estruturado + gravar vídeo tem esforço real; filtra "spray and pray".
- **Lock 90d**: compromisso com a própria imagem; não dá pra publicar → apagar → republicar com discurso contraditório.
- **Mesma infra Mux do Req 29**: zero custo incremental de integração.

**Anti-fraude by design**:
- Sem endorsements (não existe tabela `curriculum_endorsements`).
- Sem likes públicos no currículo (não existe `curriculum_likes`).
- Busca por empresa é **logada** em `curriculum_search_logs` (LGPD trilha auditável).
- Admin transversal (D-058) pode suspender/deletar currículos sinalizados.

---

## 4. Personas

### 4.1 Morador Pagante — **publica**

Persona `resident` com `PlanTier='base'`-pagante (morador_paid conforme matriz). Única persona que publica vídeo-currículo e cria ficha estruturada.

**Fonte**: `functional-matrix.md` linha 41 "Banco Talentos | Upload vídeo-currículo | Pagante ✓" — demais colunas (Base, N1, N2, N3, Plus, Pro, Marketing, Vizinhança) ✗.

**Motivação**: visibilidade para empresas do ecossistema condominial sem ter que criar perfil LinkedIn, pagar Catho Premium ou depender de indicação informal.

**Jornada curta**: BT01 Home → BT03 Vídeo → BT02 Perfil → BT04 Áreas → BT05 Modalidade → BT06 Disponibilidade → BT07 Pretensão → BT08 Experiências → BT09 Formação+CNH+NR → BT10 LGPD → BT11 Confirmação (ordem sujeita a UX do cliente; see §8).

### 4.2 Empresa Plus/Pro — **busca**

Persona `enterprise` com `PlanTier ∈ {plus, pro}`. Vê + busca + favorita + tagueia + exporta PDF de currículos.

**Fonte**: `functional-matrix.md` tabela Empresa Plus/Pro "Banco Talentos | Ver/buscar currículos | Plus ✓ | Pro ✓"; tabela Empresa Trial: ✗.

**Motivação**: recrutamento segmentado (mão de obra condominial), reduzindo custo e tempo vs agências de RH tradicionais.

**Operações Plus × Pro**:
- Plus: busca + ver vídeo + favoritar + tagueio manual.
- Pro: tudo de Plus + analytics funil (buscas → views → favoritos → contatos) + auto-tags (7 categorias calculadas) + exportação PDF em lote + histórico completo de buscas.

> Nota: granularidade Plus vs Pro detalhada em `System Architecture - Content.md` §TalentBankModule "7 Funcionalidades" — na matriz agregada ambos têm ✓, mas pacote Pro é superset.

### 4.3 Parceiro da Vizinhança — **sem acesso**

`functional-matrix.md` tabela "Parceiro da Vizinhança (Local Company)": `Banco Talentos | Acesso | ✗`. Explícito. Parceiro de vizinhança não contrata via BT.

### 4.4 Admin Master Síndico — **modera** (D-058)

Role `admin` (Zitadel). Transversal, ABAC admin-bypass. Ações:
- Ver lista completa de currículos (sem filtro de tenant).
- Suspender currículo por violação de conteúdo (discriminação, dados falsos, imagens impróprias).
- Recalcular flags anti-spam.
- Responder a reports (morador denuncia empresa que mandou oferta abusiva; empresa denuncia currículo com conteúdo falso).
- Auditar `curriculum_search_logs` para investigações LGPD.

### 4.5 Agência Marketing, Síndico, Morador Base — **sem acesso**

- Agência: escopo delegado restrito a vídeos institucionais da empresa cliente (Req 30) — não toca Banco Talentos.
- Síndico (qualquer `plan_tier`): matriz "Banco Talentos | Não tem acesso | —". Mesmo `plan_tier=pro` não navega BT. Separação limpa entre governança institucional e contratação individual. (Fonte usa label legado "N1/N2/N3" banido por D-067/ADR-0032.)
- Morador Base (free): sem upload, sem busca.

---

## 5. Tiers × Role — matriz funcional canônica

Fonte: `functional-matrix.md` linhas 41-42 + tabelas Empresa Trial/Plus/Pro + Parceiro Vizinhança + Agência Marketing.

| Persona \ Ação | Publicar currículo | Ver/buscar currículos | Favoritar | Auto-tags | Exportar PDF | Moderar |
|---|---|---|---|---|---|---|
| Morador Base (free) | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| **Morador Pagante** | **✓** (1 currículo) | ✗ | ✗ | ✗ | ✗ | ✗ |
| Síndico (qualquer `plan_tier`) | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| Empresa Trial | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| **Empresa Plus** | ✗ | **✓** | **✓** | ✗ | ✗ | ✗ |
| **Empresa Pro** | ✗ | **✓** | **✓** | **✓** | **✓** | ✗ |
| Agência Marketing | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| Parceiro Vizinhança | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| **Admin** | ✗ | **✓** (global) | — | — | **✓** (auditoria) | **✓** |

**Chave da matriz funcional** (D-066 + D-058): `{persona, plan_tier, action}` → decisão ABAC. Admin tem bypass. Reconfiguração (ex: liberar Empresa Plus para auto-tags) é mudança de seed, não de código.

---

## 6. Aggregates

Contexto: `content` (infra vídeo + aggregate Curriculum) + `commercial` (ações do lado empresa — favoritos, tags, search logs).

### 6.1 `Curriculum` (Aggregate Root)

**Tabela raiz**: `curricula` (ver schema em `System Architecture - Content.md` §TalentBankModule)

```
curricula:
  id UUID PK
  morador_id UUID UNIQUE  -- 1 currículo por morador (invariante)
  status curriculum_status  -- draft | submitted | published | archived (ver §12)

  -- Dados pessoais
  full_name, phone, email, cep, age, marital_status

  -- Documentação operacional condominial
  cnh_type CNHType
  nr_courses NRCertification[]

  -- Preferências
  contract_types ContractModality[]
  available_schedules AvailabilitySchedule[]
  start_availability StartAvailability
  travel_time_tolerance TravelTime
  desired_salary_min NUMERIC
  desired_salary_ideal NUMERIC

  -- LGPD
  lgpd_consent_version TEXT   -- versionado (invariante §11)
  lgpd_consented_at TIMESTAMPTZ

  created_at, updated_at, deleted_at
```

**Regra crítica**: `UNIQUE(morador_id)` — um morador **não pode** ter dois currículos. Alterações viram UPDATE do mesmo registro (soft-delete via `archived` se morador decidir remover).

### 6.2 `CurriculumVideo` (Entidade interna — mesmo aggregate)

```
curriculum_videos:
  id UUID PK
  curriculum_id UUID FK (CASCADE)
  video_id UUID                 -- ref Video (Content Video aggregate, Mux-backed)
  duration_seconds INTEGER CHECK (duration_seconds <= 90)
  locked_until TIMESTAMPTZ       -- upload_ts + INTERVAL '90 days'
  created_at TIMESTAMPTZ
```

**Invariante**: `duration_seconds <= 90` (DB CHECK). **Lock**: `locked_until = created_at + 90 days`; re-upload só após `locked_until < now()`.

### 6.3 `TalentArea` + `DesiredFunction` (entidade)

```
curriculum_areas_of_interest:
  id, curriculum_id
  area TalentArea     -- enum 18 valores (§7.1)
  desired_function TEXT  -- cargo específico dentro da área (livre)
  created_at
```

Morador pode ter **múltiplas** áreas (ex: `manutencao_predial` + `apoio_operacional_externo`). Cada entry carrega `desired_function` livre (ex: "Zelador Noturno", "Auxiliar de Jardinagem").

### 6.4 `EmploymentHistory` (entidade — **5 vínculos — D-099 Fase 11** sobrescreve D-060/D-064)

```
curriculum_work_experience:
  id, curriculum_id
  company_name, position, activities, tenure
  exit_reason EmploymentExitReason
  display_order INTEGER          -- 1..5 (CHECK — D-099 Fase 11 sobrescreve 3 anterior)
  created_at

  UNIQUE(curriculum_id, display_order)
  CHECK (display_order BETWEEN 1 AND 5)  -- D-099 Fase 11 (sobrescreve 3 anterior)
```

**D-099 Fase 11 (STATE v2, sobrescreve D-060/D-064)**: **5 últimos vínculos**. Q39 fechada na direção do PDF original do cliente. Overriding: `display_order ≤ 5`. Alteração futura **via matriz funcional + seed** (não hard-code; trocar CHECK exige migration).

### 6.5 `ProfessionalProfile` (entidade)

```
curriculum_professional_profiles:
  id, curriculum_id
  question_number INTEGER (1-6)   -- 6 perguntas abertas fixas
  question_text TEXT
  answer TEXT
  created_at, updated_at
```

Ver `System Architecture - Content.md` BT02. 6 perguntas abertas (ex: "Conte um episódio em que você resolveu um problema técnico sob pressão"). Catálogo das 6 perguntas é seed, não código.

### 6.6 `Education` (entidade)

```
curriculum_education:
  id, curriculum_id
  course_name, institution, professional_relevance
  created_at
```

BT09. Formação sem limite rígido de N entries (morador pode listar ensino médio + curso técnico + curso sindical).

### 6.7 `CompanyCurriculumFavorite` (lado empresa — BC `commercial`)

```
company_curriculum_favorites:
  id, company_id, curriculum_id
  created_at
  UNIQUE(company_id, curriculum_id)
```

Empresa Plus/Pro favorita. Privado (outras empresas não veem quem favoritou quem).

### 6.8 `CompanyCurriculumTag` (lado empresa)

```
company_curriculum_tags:
  id, company_id, curriculum_id, tag TEXT
  created_at
```

Tags **manuais privadas** (não compartilhadas entre empresas). Ex: empresa A taga `candidato_X` como "A chamar semana que vem"; empresa B não enxerga.

**Auto-tags (Pro only)**: 7 categorias calculadas pelo sistema (ex: "alta-disponibilidade", "certificações-NR-completas", "experiência-condominial-direta", etc.) — lista exata em `System Architecture - Content.md` §TalentBankModule "7 Funcionalidades empresa — Auto-tags". Armazenadas na mesma tabela com `source='auto'`.

### 6.9 `CurriculumSearchLog` (LGPD audit trail)

```
curriculum_search_logs:
  id, company_id
  filters_used JSONB
  results_count INTEGER
  profiles_viewed INTEGER DEFAULT 0
  searched_at TIMESTAMP
```

Cada busca e cada view registrada. LGPD obriga trilha; morador tem direito a solicitar relatório de quem viu seu currículo (invariante §11).

---

## 7. Enums

Fonte canônica: `master-sindico-v2/90-ingested/_reconciliacao-dev/05-enums-regras-quebras.md` §1 categoria 16 + cross-check com `Enums e Listas Mestre.md` (linhas 38, 184).

### 7.1 `TalentArea` — 18 áreas condominiais

```
operacoes_condominiais
limpeza_conservacao_higienizacao
manutencao_predial
seguranca_patrimonial
obras_reformas_adequacoes
administracao_apoio_administrativo
atendimento_relacionamento_suporte
logistica_almoxarifado_suprimentos
gestao_coordenacao_lideranca
treinamento_qualidade_processos_compliance
tecnologia_sistemas_suporte_digital
comercial_vendas_parcerias
comunicacao_marketing_conteudo
financeiro_contabil_controladoria
juridico_contratos_compliance_legal
recursos_humanos_gestao_pessoas
apoio_operacional_externo
outras_areas
```

**Regras**:
- Morador escolhe **uma ou mais** (array). Tabela `curriculum_areas_of_interest`.
- Cada entry opcionalmente acompanha `desired_function` (texto livre) pra refinar.
- Busca empresa filtra `WHERE area = ANY($1)` (Postgres array overlap).

### 7.2 `ContractModality` — 9 modalidades

```
clt
pj
estagio
temporario
folguista
diarista
freelancer
intermitente
outros
```

Fonte: `Enums e Listas Mestre.md` linha 184 (linha literal: "`clt`, `pj`, `estagio`, `temporario`, `folguista`, `diarista`, `freelancer`, `intermitente`, `outros`"). Morador escolhe múltiplas (disposto a CLT **ou** diária).

### 7.3 `AvailabilitySchedule` — 4 horários

```
diurno
noturno
escala
finais_semana
```

Morador marca múltiplas. Empresa filtra (ex: "preciso porteiro noturno" → `AvailabilitySchedule = 'noturno'`).

### 7.4 `StartAvailability` — 3 janelas

```
imediato
ate_15_dias
a_combinar
```

Única (não múltipla).

### 7.5 `TravelTime` — 3 tolerâncias de deslocamento

3 faixas (valores específicos em seed; padrão provável `ate_30_min | ate_1h | acima_1h` — confirmar no cliente; marcado como **gap §18**).

### 7.6 `CNHType` — 4 valores

```
nao
A
B
AB
```

Fonte direta: `Enums e Listas Mestre.md` linha 38. Corresponde a: sem CNH, moto, carro, carro+moto.

### 7.7 `NRCertification` — 4 valores

```
NR-10        -- Segurança em Instalações e Serviços em Eletricidade
NR-33        -- Segurança em Espaços Confinados (ex: cisternas)
NR-35        -- Trabalho em Altura (fachadas, telhado, caixa d'água)
outros       -- NR-6 EPI, NR-23 Incêndio, NR-32 Saúde Hospitalar etc.
```

Morador marca múltiplas (`NRCertification[]`).

### 7.8 `EmploymentExitReason` — 4 motivos

```
termino_contrato
nova_oportunidade
ajuste_horario
outros
```

Usado em `curriculum_work_experience.exit_reason`. Domina saída respeitosa (sem "demissão por justa causa" como opção — alinhado à postura anti-preconceito-de-seleção).

---

## 8. Telas BT 01–11 (catálogo fiel)

Fonte: `master-sindico-v2/90-ingested/_reconciliacao-dev/04-telas-jornadas.md` §9 + `System Architecture - Content.md` §TalentBankModule "11 Telas de Cadastro".

Ordem lógica do fluxo de cadastro (UX final pode reordenar BT02..BT09 conforme design). BT01, BT10, BT11 são pontas fixas (home → LGPD → confirmação).

### BT01 — Home Banco de Talentos

**Objetivo**: landing do morador pagante; explica o que é + CTA "Criar currículo" (ou "Editar meu currículo" se existente).

**Conteúdo**:
- Headline ("Sua carreira condominial em 90 segundos")
- Explicação breve (quem vê? empresas Plus/Pro; como funciona? vídeo + ficha; pode editar? sim, após 90d do vídeo).
- CTA primário: "Criar/Editar currículo"
- CTA secundário: "Como funciona?" (modal de FAQ ou link)
- Status de publicação se currículo já existe: `draft | submitted | published | archived`.

**ABAC**: requer `role='resident' AND plan_tier IN ('base' com flag pagante) [...] AND NOT curriculum.archived`. Morador Base redireciona pra upgrade.

### BT02 — Perfil profissional (6 perguntas abertas)

**Objetivo**: 6 perguntas abertas que formam a base textual do currículo.

**Campos** (schema `curriculum_professional_profiles`):
- 6 pares `(question_number, question_text, answer)` — perguntas fixas no seed (não cliente).
- Cada `answer` com limite de caracteres (sugestão ~500 chars; confirmar com cliente).

**Exemplos de pergunta** (indicativos, finais no seed): "Qual seu maior diferencial?"; "Me conte um episódio profissional marcante"; "Como você lida com feedback difícil?"; etc. (catálogo completo é seed de produto — gap §18).

### BT03 — Vídeo-currículo (90s, lock 90d)

**Objetivo**: gravar/enviar vídeo de **até 90 segundos** apresentando-se profissionalmente.

**Fluxo técnico**:
1. Frontend solicita upload presigned via `POST /videos?type=resume` (GR-019: backend NÃO faz proxy de bytes).
2. Cliente envia bytes direto pro Mux.
3. Webhook Mux `video.asset.ready` entrega `duration_seconds` + `playback_id`.
4. Backend valida `duration_seconds <= 90`. Se > 90, marca `Video.state='rejected'` com motivo `duration_exceeded`.
5. Se válido, cria `curriculum_videos(curriculum_id, video_id, duration_seconds, locked_until=now()+INTERVAL '90 days')`.
6. Evento `content.talent.curriculum-updated` publicado.

**Lock 90d**: próxima atualização do vídeo-currículo só é aceita após `locked_until < now()`. Motivador: compromisso com a própria imagem + anti-spam + alinhamento com trava trimestral de vídeos institucionais (content.md Req 29 Rule 7).

**Copy de UX**: "Seu vídeo ficará travado por 90 dias após o envio. Prepare-se para gravá-lo com cuidado."

### BT04 — Áreas de Interesse (18 opções)

**Objetivo**: morador seleciona 1 ou mais das 18 `TalentArea` + escreve função desejada dentro de cada.

**UX**:
- Lista das 18 áreas (§7.1). Checkbox múltiplo.
- Para cada área marcada, campo livre "Função desejada" (ex: dentro de `manutencao_predial` → "Auxiliar de manutenção elétrica").
- Validação: mínimo 1 área.

**Gera** `curriculum_areas_of_interest[]`.

### BT05 — Modalidade de Contratação (9 opções)

**Objetivo**: morador marca modalidades aceitas.

**UX**: 9 checkboxes `ContractModality` (§7.2). Mínimo 1.

Persiste em `curricula.contract_types ContractModality[]`.

### BT06 — Disponibilidade (4 horários + janela início + deslocamento)

**Objetivo**: coletar disponibilidade tripla.

**Campos**:
1. `available_schedules AvailabilitySchedule[]` — 4 opções (diurno/noturno/escala/finais_semana); mínimo 1.
2. `start_availability StartAvailability` — 3 opções (imediato/até 15 dias/a combinar); uma.
3. `travel_time_tolerance TravelTime` — 3 faixas; uma.

### BT07 — Pretensão salarial

**Objetivo**: coletar faixa salarial pretendida.

**Campos**:
- `desired_salary_min NUMERIC(12,2)` — piso aceitável.
- `desired_salary_ideal NUMERIC(12,2)` — ideal.
- Moeda BRL fixa.
- Validação: `desired_salary_min <= desired_salary_ideal`.

**Copy**: "Defina piso e ideal. Ajuda a filtrar ofertas alinhadas com sua realidade."

### BT08 — Experiências (5 vínculos — D-099 Fase 11 sobrescreve D-060/D-064)

**Objetivo**: últimos **5** vínculos profissionais (`EmploymentHistory`).

**Campos por vínculo**:
- `company_name TEXT` (opcional — pode marcar "não desejo divulgar")
- `position TEXT` (cargo)
- `activities TEXT` (principais atividades)
- `tenure TEXT` (período, formato livre "jan/2022 – nov/2023")
- `exit_reason EmploymentExitReason`
- `display_order INTEGER 1..5` (UNIQUE por currículo — D-099 Fase 11)

**Invariante**: `count(work_experience WHERE curriculum_id=X) <= 5` (D-099 Fase 11).

**Nota histórica**: docs legadas (`System Architecture - Content.md`, `04-telas-jornadas.md`) mencionam "3-5" ou "5" vínculos; PDF original do cliente "ESTRUTURA DE CADASTRO DE CURRÍCULO" TELA 08 = **5**. D-060/D-064 Fase 8 disseram "3" erroneamente; **D-099 Fase 11 (2026-04-24) fechou em 5**. Alteração futura via matriz funcional (parâmetro), não hard-code.

### BT09 — Formação + CNH + NR

**Objetivo**: consolidar formação educacional, CNH, NRs.

**Sub-blocos**:
1. Formação — N entries em `curriculum_education(course_name, institution, professional_relevance)`.
2. CNH — `cnh_type CNHType` (enum 4).
3. NR — `nr_courses NRCertification[]` (enum 4, múltiplo).

**Sem validação cruzada de documento** (sem upload de PDF de NR — MVP). Trust-by-default + trilha de auditoria (admin pode exigir comprovação em caso de denúncia).

### BT10 — Termo LGPD

**Objetivo**: consentimento explícito versionado.

**Campos**:
- Checkbox obrigatório "Li e aceito o termo LGPD vigente na versão X".
- `curricula.lgpd_consent_version TEXT` (ex: `2026-Q2-v1`).
- `curricula.lgpd_consented_at TIMESTAMPTZ`.

**Regras**:
- Obrigatório (invariante §11).
- Sem aceite → `Curriculum` permanece `draft` (não pode transicionar para `submitted`).
- Quando termo é atualizado pela plataforma, morador é notificado e precisa **reconsentir** para manter `published`.

### BT11 — Confirmação

**Objetivo**: revisão final + submissão.

**Fluxo**:
1. Tela resume todos os dados preenchidos (leitura).
2. Botão "Publicar currículo" → transiciona `Curriculum.status = draft → submitted`.
3. Admin review (síncrono manual M3; automação M4+) — moderação mínima (texto ofensivo, imagem imprópria no vídeo).
4. Aprovação → `submitted → published` → evento `content.talent.curriculum-published` → indexa em busca.
5. Rejeição → `submitted → draft` com motivo; morador reabre.

**Empresa view** (apêndice nas telas — não é "BT12"):
- Dashboard busca (filtros area + região + CEP + raio + modalidade + horário).
- Lista resultados (cards com foto + nome + área primária + score de match se Pro).
- Detalhe: player do vídeo + ficha completa + botões favoritar / tagar / exportar PDF (Pro) / "convidar via Connect Me".

---

## 9. Empresa view — busca, favoritos, tags, raio geográfico

Fonte: `System Architecture - Content.md` §TalentBankModule "Lado Empresa — 7 Funcionalidades".

1. **Busca avançada** (Plus + Pro) — filtros por:
   - `TalentArea[]` (overlap).
   - CEP + raio (geo: km ou deslocamento estimado).
   - `ContractModality[]` (overlap).
   - `AvailabilitySchedule[]` (overlap).
   - `StartAvailability`.
   - `CNHType` (ex: "apenas candidatos com carro").
   - `NRCertification[]` (ex: "exige NR-35").
   - Faixa salarial (`desired_salary_min` entre X..Y).
   - Palavra-chave (full-text search em `position`, `activities`, `desired_function`, `answer` perguntas abertas).
2. **Sistema de favoritos** (Plus + Pro) — salvar perfil; lista privada por empresa.
3. **Exportação PDF** (Pro) — um PDF por currículo ou lote (ZIP).
4. **Histórico de buscas** (Pro) — relatório com filtros usados + resultados + CTR.
5. **Analytics de funil** (Pro) — buscas → views → favoritos → contatos (tudo com privacidade cross-tenant; empresa X não vê números de empresa Y).
6. **Auto-tags** (Pro) — 7 categorias calculadas (lista exata em seed; gap §18).
7. **Tags manuais** (Plus + Pro) — string livre, privadas por empresa.

**Busca tecnologia** — `ISearchProvider` real (OpenSearch ou Meilisearch — ADR-0042 `proposed pending dual-check`). **D-120 Fase 14 revoga D-067**: tsvector vigente é stub temporário. Reindexação em `content.talent.curriculum-published` e `curriculum-updated`.

**Privacidade cross-tenant**: empresa A nunca vê quem empresa B favoritou; quem empresa B tagou como X; quais currículos empresa B exportou. ABAC tenant-strict.

**Invariante da busca**: toda busca grava em `curriculum_search_logs` (§6.9) — LGPD trilha auditável, empresa não pode zerar histórico.

---

## 10. Regras de negócio

Consolidação das regras invocáveis no código/validação.

1. **Vídeo 90s obrigatório** — sem `curriculum_videos` válido, `Curriculum` não sai de `draft`. DB CHECK `duration_seconds <= 90`.
2. **Lock 90 dias pós-upload** — `locked_until = created_at + 90 days`; re-upload **rejeitado** (409) enquanto `now() < locked_until`.
3. **5 vínculos máx** (D-099 Fase 11 — sobrescreve D-060/D-064) — `count(curriculum_work_experience) <= 5`; DB CHECK `display_order BETWEEN 1 AND 5` + UNIQUE.
4. **Consentimento LGPD obrigatório e versionado** — sem `lgpd_consent_version` + `lgpd_consented_at`, `Curriculum` fica em `draft`.
5. **1 currículo por morador** — `UNIQUE(morador_id)` em `curricula`.
6. **Só morador pagante publica** — ABAC `resident AND plan_tier=base com flag pagante`.
7. **Só Empresa Plus/Pro busca** — ABAC `enterprise AND plan_tier IN ('plus','pro')`; admin bypass.
8. **Parceiro Vizinhança sem acesso** — matriz funcional explícita ✗.
9. **Sem endorsements, sem likes públicos** — não existem tabelas `curriculum_endorsements` ou `curriculum_likes_public` (anti-fraude).
10. **Toda busca logada** — `curriculum_search_logs` com `filters_used JSONB` (LGPD).
11. **Privacidade cross-tenant** — favorites/tags/logs privados por empresa.
12. **Moderação admin pode suspender** — admin via `apps/admin` dedicada (D-134 revoga D-058) transiciona `published → archived` com motivo.
13. **Archived não aparece em busca** — filter `WHERE status='published'`.
14. **Cancelamento do plano pagante** — morador deixa de ser pagante → currículo vai para `archived` (soft-hide). Mantém dados por X anos conforme LGPD (gap §18 — política de retenção).
15. **Convite via Connect Me Empresa→Morador** — fluxo especial (§15), não rompe unidirecionalidade do Connect Me.
16. **Busca só retorna `published`** — `draft | submitted | archived` nunca expostos.

---

## 11. Invariantes

Formalização para testes PBT + DB CHECKs.

1. **INV-BT-01** — `Curriculum.video.duration_seconds ≤ 90` (DB CHECK).
2. **INV-BT-02** — `Curriculum.video.locked_until = video.created_at + 90 days` (regra de domínio; re-upload só se `now() > locked_until`).
3. **INV-BT-03** — `|Curriculum.work_experience| ≤ 5` (D-099 Fase 11; DB CHECK `display_order BETWEEN 1 AND 5` + UNIQUE`(curriculum_id, display_order)`).
4. **INV-BT-04** — `Curriculum.lgpd_consent_version IS NOT NULL AND Curriculum.lgpd_consented_at IS NOT NULL` quando `status ∈ {submitted, published}` (DB CHECK parcial).
5. **INV-BT-05** — `UNIQUE(morador_id)` em `curricula`.
6. **INV-BT-06** — `Curriculum.status` transições válidas (§12); qualquer outra combinação rejeitada.
7. **INV-BT-07** — `CurriculumSearchLog` insert em toda busca; sem `DELETE` permitido (append-only LGPD).
8. **INV-BT-08** — Cross-tenant: `SELECT * FROM company_curriculum_favorites WHERE company_id = $caller_tenant` sempre (never leak).
9. **INV-BT-09** — `|Curriculum.areas_of_interest| ≥ 1` quando `status = 'published'`.
10. **INV-BT-10** — `Curriculum.desired_salary_min ≤ Curriculum.desired_salary_ideal` (CHECK).

---

## 12. State machine — `Curriculum`

```
      [criação pelo morador]
             │
             ▼
         ┌───────┐
         │ draft │◀──────┐
         └───┬───┘       │ (rejeição admin)
             │           │
  (BT11 envia)           │
             │           │
             ▼           │
        ┌─────────┐      │
        │submitted│──────┘
        └────┬────┘
             │
  (admin aprova)
             │
             ▼
        ┌─────────┐
        │published│──────┐
        └────┬────┘      │ (admin suspende / morador arquiva / plano cai)
             │           │
             └──────────▶┌────────┐
                         │archived│
                         └────────┘
```

Transições válidas:
- `draft → submitted` — ao finalizar BT11 (LGPD aceita, video ≤ 90s, ≥ 1 área, ≥ 1 modalidade, formação preenchida).
- `submitted → published` — admin aprova (ou automação M4+).
- `submitted → draft` — admin rejeita com motivo; morador reabre e corrige.
- `published → archived` — morador arquiva deliberadamente; admin suspende; plano pagante expira.
- `archived → draft` — morador reativa (condicional a plano ativo + LGPD vigente).

Proibidas: `draft → published` direto, `archived → published` direto, `archived → submitted`.

---

## 13. Domain events

Fonte: `System Architecture - Content.md` §TalentBankModule "Events".

| Evento | Publisher | Consumers | Payload chave |
|---|---|---|---|
| `content.talent.curriculum-created` | `content` | `commercial` (indexação), audit | `curriculum_id, morador_id, ts` |
| `content.talent.curriculum-updated` | `content` | search reindex, audit | `curriculum_id, changed_fields[], ts` |
| `content.talent.curriculum-published` | `content` | search indexer, notification (empresas que monitoram área/região? M4+), audit | `curriculum_id, areas[], cep, ts` |
| `content.talent.curriculum-archived` | `content` | search reindex (remove), audit | `curriculum_id, reason, ts` |
| `content.talent.curriculum-viewed` | `commercial` (empresa abriu detalhe) | LGPD log, analytics Pro | `company_id, curriculum_id, ts` |
| `content.talent.curriculum-favorited` | `commercial` | analytics Pro | `company_id, curriculum_id, ts` |
| `content.talent.curriculum-unfavorited` | `commercial` | analytics | idem |
| `content.talent.curriculum-exported` | `commercial` | LGPD log, analytics Pro | `company_id, curriculum_ids[], format, ts` |
| `content.talent.curriculum-search-performed` | `commercial` | LGPD log (`curriculum_search_logs`), analytics | `company_id, filters_used, results_count, ts` |
| `content.talent.curriculum-connect-me-invited` | `commercial` | notificação morador (§15), audit | `company_id, curriculum_id, connect_me_request_id, ts` |

Eventos fluem via bus (NATS JetStream quando implementado; síncrono in-process até lá). **INV-BT-07** garante append-only em `curriculum_search_logs`.

---

## 14. Dependências

### 14.1 Infra (D-056 — agnosticismo)

- **`IVideoProvider` (Mux atual)** — upload presigned, HLS, webhooks `asset.ready/errored`, duration extraction. Sem SDK Mux vazando pra `domain/` (ver vault `CLAUDE.md` master-sindico §4).
- **Storage** — `IStorageProvider` (MinIO dev / R2 ou S3 prod) para thumbnails, storyboard.
- **Busca** — `ISearchProvider` real (OpenSearch ou Meilisearch — ADR-0042 pending; D-120 revoga D-067 PG tsvector). Índice por `areas_of_interest`, `position`, `activities`, `desired_function`, `answer`.

### 14.2 Bounded contexts

- **`identity`** — autenticação + ABAC; confirmar `role='resident' AND plan_tier pagante` para publish; `role='enterprise' AND plan_tier IN ('plus','pro')` para search.
- **`commercial`** — aggregate `Company` (empresa que busca); Connect Me Empresa→Morador (§15).
- **`content`** — aggregate `Video` (Mux-backed); reuso do fluxo de upload Req 29.
- **`billing`** — `plan_tier` flag `paid` (para morador); trial vs plus/pro (para empresa).
- **`cross-domain`** — `IAuditLogger` (DT-010 legado) para trilha LGPD + admin moderação.

### 14.3 Quotas & billing

- **Morador pagante**: 1 vídeo-currículo publicado ativo (N=1 currículo por morador, INV-BT-05). `QuotaKey = video_upload_curriculum` (existe em `ubiquitous-language.md`), `QuotaPeriod = once` conceitual — execução é pela INV-BT-05 mesmo.
- **Empresa Plus**: buscas ilimitadas? (gap §18 — confirmar com cliente se há quota/dia).
- **Empresa Pro**: ilimitado + exportação PDF.

---

## 15. Convite via Connect Me — fluxo especial Empresa → Morador

**Problema**: Connect Me canônico (`commercial.md` Req 19, 19.1, 19.2, 19.3) é unidirecional **fluxos fixos** — não prevê Empresa→Morador diretamente. Mas o Banco de Talentos precisa formalizar oferta da empresa ao candidato.

**Solução (formalizada na destilação)**: o botão "Convidar" no currículo dispara um fluxo especial de Connect Me:

1. Empresa Plus/Pro abre currículo (`curriculum-viewed`).
2. Clica em "Convidar para entrevista / oferecer oportunidade".
3. Preenche **formulário estruturado** (não chat):
   - Tipo da oportunidade (ex: `vaga_fixa | servico_temporario | entrevista`).
   - Categoria (TalentArea).
   - Descrição (texto limitado).
   - Salário ofertado ou faixa.
   - Local (CEP do condomínio onde a vaga se aplica).
   - Modalidade (ContractModality).
   - Prazo para resposta (dias).
4. Evento `content.talent.curriculum-connect-me-invited` dispara notificação ao morador.
5. Morador recebe: **dados da empresa visíveis** no card (não há "reveal-after-interest" como nos fluxos clássicos — aqui a empresa já se identificou de propósito). Morador clica "Tenho Interesse" ou "Não Tenho Interesse" ou "Ignorar".
6. Se interesse → dados de contato do morador são revelados à empresa (telefone/email conforme consentimento LGPD da ficha).
7. Log LGPD: timestamp, company_id, morador_id, IP, terms_version (igual padrão Connect Me clássico — `commercial.md` Req 19 "Log LGPD").
8. Auto-close em N dias sem resposta (padrão Connect Me: 48h clássico; aqui talvez 7 dias — confirmar; gap §18).

**Por que isso não rompe unidirecionalidade?**
- Cada Connect Me é **um ato estruturado**, não uma thread.
- Morador não "responde" em texto livre; só clica Interesse/Sem Interesse.
- Empresa não "replica". Se quiser novo convite, abre novo Connect Me (conta contra quota).
- Não há histórico de conversa — cada convite é iniciativa separada.

**Quota**: conta contra cota de Connect Me da empresa (ou cota separada? gap §18).

**Anti-assédio**:
- Botão "Denunciar" no convite (igual Req 19).
- Admin pode banir empresas que spammam convites sem relevância.
- Morador pode bloquear empresa (não recebe mais convites dela — futuro).

---

## 16. Anti-fraude

Diferencial estrutural vs LinkedIn/Catho:

- **Sem endorsements** — não existe `curriculum_endorsements`. Cliente decidiu que endorsements são manipuláveis (amigos endossam amigos). Foco em **fatos verificáveis** (experiência descrita, NR, CNH).
- **Sem likes públicos** — não existe `curriculum_likes`. Currículo não tem contador social visível. Evita beauty-contest e gamificação tóxica.
- **Toda busca logada** — empresas não podem fazer "search & ghost" anonimamente. `curriculum_search_logs` é append-only.
- **Favoritos e tags privados** — empresa A não vê tags de empresa B. Sem mercado negro de "shortlists".
- **Admin pode pausar** — currículos com conteúdo suspeito (dados falsos, vídeo ofensivo) vão para `archived` por ato admin.
- **LGPD versionado** — mudanças no termo obrigam reconsent. Sem "aceite perpétuo silencioso".
- **Lock 90d no vídeo** — elimina currículo-fantasma ("crio, tiro, crio de novo com outro discurso").

---

## 17. Estado no código

**Não aplicável a este repositório**. `master-sindico-v2/` é pasta de **remontagem pré-migração sem código** (ver `CLAUDE.md` v2 §1). Cobertura de implementação:

- **Telas BT01–11**: reconciliação `04-telas-jornadas.md` §12 reporta `BT 01-11 | 11 | 0 | 11 | RED` (11 faltando de 11).
- **Aggregates**: inexistentes em `01-domain/aggregates/` da v2 (sem `Curriculum.md`, `CurriculumVideo.md`, `EmploymentHistory.md` etc.) — lacuna a endereçar na Fase 9 (ver gap §18).
- **Enums canônicos**: catalogados em `05-enums-regras-quebras.md` §1 categoria 16 + `Enums e Listas Mestre.md`.
- **Referências commercial**: `04-requirements/functional/commercial.md` tem COM-028 (empresa busca) + COM-029 (morador publica vídeo) — esboço sem detalhar os 11 requisitos granulares.
- **Backend Go real** (fora do vault, `backend/internal/modules/commercial/` e `/content/` citados pelo prompt): status de implementação **fora do alcance desta destilação** — registrar em Fase 9 como tarefa cross-repo.

---

## 18. Gaps

Itens pendentes que bloqueiam/exigem esclarecimento:

| # | Gap | Ação sugerida |
|---|---|---|
| G-BT-01 | `TravelTime` — 3 faixas sem valores literais catalogados (enum existe, valores não). | Confirmar com cliente: `ate_30_min / ate_1h / acima_1h` ou outros? |
| G-BT-02 | 6 perguntas abertas do BT02 — catálogo textual definitivo. | Pedir PDF/seed ao cliente; registrar em `04-requirements/functional/banco-talentos.md`. |
| G-BT-03 | 7 auto-tags calculadas (Pro) — lista literal. | Recuperar de `System Architecture - Content.md` ou cliente. |
| G-BT-04 | Quota Connect Me Empresa→Morador (convite do Banco Talentos) — mesma cota do clássico ou separada? | Decisão D-### + registrar em STATE. |
| G-BT-05 | Auto-close do convite Empresa→Morador — 48h (clássico) ou maior? | Decisão D-### + registrar em STATE. |
| G-BT-06 | Política de retenção LGPD pós-archived — quanto tempo manter `curricula.deleted_at IS NOT NULL`? | Confirmar com consultor LGPD (DPO). |
| G-BT-07 | Moderação M3 manual vs M4+ automação — ML pipeline? heurísticas? | Roadmap M4+. |
| G-BT-08 | Quota de buscas Empresa Plus — ilimitada ou capada? | Cliente. |
| G-BT-09 | Arquivo `04-requirements/functional/banco-talentos.md` não existe na v2 — só COM-028/029 no commercial. | Criar arquivo dedicado com BT-001..BT-0NN EARS. |
| G-BT-10 | Aggregates `Curriculum`, `CurriculumVideo`, `EmploymentHistory`, etc. ausentes em `01-domain/aggregates/` v2. | Criar arquivos aggregate na Fase 9. |
| G-BT-11 | Status de implementação backend Go (fora do vault) — real state of `internal/modules/commercial/` e `/content/`. | Cross-repo audit com agente com acesso ao backend. |
| G-BT-12 | Admin mod flow — telas BT-Admin (lista global, ações) — não catalogadas. | UX para Admin. |
| G-BT-13 | Fluxo de conversão morador-base-free → morador-pagante no contexto BT — CTA dentro de BT01? | UX + billing cross-check. |
| G-BT-14 | Morador bloquear empresa — recebe mais convites? Não especificado. | Backlog M4+. |
| G-BT-15 | `Content/02-Jornadas/Inventario-Telas.md` e `Content/contents/` (citados pelo prompt) não ingeridos como arquivos autônomos. | Confirmar com usuário se há material novo a ingerir. |

---

## 19. Referências cruzadas

### 19.1 STATE v2

- ~~[[../../STATE#d-060]] — Banco de Talentos 3 vínculos~~ **sobrescrita por D-099 (5 vínculos)**.
- ~~[[../../STATE#d-064]] — Reforço 3 vínculos~~ **sobrescrita por D-099 (5 vínculos)**.
- [[../../STATE#d-099-—-banco-de-talentos-5-vínculos-profissionais-fecha-q39-sobrescreve-d-060-d-064|D-099]] — Banco de Talentos 5 vínculos (Fase 11, Q39 fechada).
- [[../../STATE#d-056]] — Agnosticismo de provedor (Mux atrás de `IVideoProvider`).
- [[../../STATE#d-134]] — Admin = `apps/admin` dedicada (revoga D-058/D-072) — modera BT.
- [[../../STATE#d-080]] — Planos globais Stripe-style (matriz funcional).
- [[../../STATE#d-120]] — Search real M1 (revoga D-067 PG tsvector — ADR-0042 OpenSearch vs Meilisearch pending dual-check).

### 19.2 Domain v2

- [[../../01-domain/ubiquitous-language]] §4 `commercial` — `Curriculum` Aggregate Root registrado.
- [[../../01-domain/business-rules]] — "Apenas empresas Plus/Pro podem visualizar currículos do Banco de Talentos. Convite via Connect Me fluxo especial (Empresa → Morador/Síndico)".
- [[../../01-domain/bounded-contexts#commercial]] — "Banco de Talentos (busca de morador por empresa)".

### 19.3 Requirements v2

- [[../../04-requirements/functional/commercial#com-028]] — empresa busca currículos.
- [[../../04-requirements/functional/commercial#com-029]] — morador publica vídeo-currículo.
- [[../../04-requirements/matrix-functional]] — matriz canônica.
- [[../../04-requirements/compliance-lgpd]] — política LGPD (aplicar versionamento).
- [[../../04-requirements/registration-data]] — dados de cadastro morador (reuso).

### 19.4 Sub-produtos correlatos

- [[02-connect-me]] — fluxo especial Empresa→Morador BT (§15) reutiliza o padrão estrutural.
- [[04-conteudo-videos]] — infra Mux + lock 90d (Req 29) reaproveitada.
- [[03-reputacao]] — reputação da empresa Plus/Pro (search/favorite stats podem futuramente alimentar tier — M4+).
- [[11-administradoras-condominiais]] — administradoras Plus/Pro também podem buscar candidatos (recurso compartilhado).

### 19.5 Fontes ingeridas

- `../../90-ingested/from-vault-50-projects-master-sindico/specs/requirements/commercial.md` (título + tags `talent-bank`)
- `../../90-ingested/from-vault-50-projects-master-sindico/specs/requirements/content.md` (Req 29 — infra Mux)
- `../../90-ingested/from-vault-50-projects-master-sindico/specs/requirements/functional-matrix.md` (linhas 41-42, tabelas Empresa/Vizinhança)
- `../../90-ingested/from-vault-50-projects-master-sindico/client-vault/01 - Arquitetura de Sistema/System Architecture - Content.md` (§TalentBankModule — canônico)
- `../../90-ingested/from-vault-50-projects-master-sindico/client-vault/01 - Arquitetura de Sistema/System Architecture - Data Model.md` (schema SQL)
- `../../90-ingested/from-source-material-masterSindico-extracted/MasterSindico/01 - Arquitetura de Sistema/Decisões e Pendências.md` (Decision 6 + resolução 60→90s)
- `../../90-ingested/from-vault-50-projects-master-sindico/client-vault/03 - Modelagem e Dados/Enums e Listas Mestre.md` (linhas 38, 184)
- `../../90-ingested/_reconciliacao-dev/04-telas-jornadas.md` §9 (BT01–BT11)
- `../../90-ingested/_reconciliacao-dev/05-enums-regras-quebras.md` §1 categoria 16 (enums + Q39)

---

> Este arquivo é o **canônico Fase 8** para Banco de Talentos. Próxima fase deve:
> (1) criar `04-requirements/functional/banco-talentos.md` com EARS BT-001..BT-0NN;
> (2) criar aggregates `Curriculum.md`, `CurriculumVideo.md`, `EmploymentHistory.md`, `ProfessionalProfile.md`, `Education.md` em `01-domain/aggregates/`;
> (3) endereçar gaps G-BT-01..G-BT-15.
