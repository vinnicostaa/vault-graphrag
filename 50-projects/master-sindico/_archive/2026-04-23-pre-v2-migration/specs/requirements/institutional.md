---
title: Institutional — Condomínios, Unidades, Timeline, Mandato e Gestão
version: 1.0
status: stable
type: requirement
domain: institutional
tags: [condominium, unit, timeline, mandate, governance, resident, resident-profile, syndic-profile]
---

# Institutional — Condomínios, Unidades, Timeline, Mandato e Gestão

Domínio de gestão condominial: estrutura de condomínios, unidades residenciais, timeline imutável, plano diretor e mandatos de síndicos.

---

## Req 6.2 — Perfil do Síndico

Campos de cadastro, governance markers autodeclarados e campos N2/N3 adicionais.

### Campos obrigatórios de cadastro

- `professional_photo` (upload)
- `full_name`
- `birth_date`
- `professional_email`
- `phone`
- `cpf` (validado, único no escopo de síndicos)
- `professional_address`
- `city_state_of_operation`
- `sindico_type` (enum: `eleito`, `profissional`, `interino`)
- `years_of_experience` (int)
- `condominiums_under_management` (número inicial)

### Governance Markers (15 autodeclarados)

Checkbox list: `membro_abracs`, `seguro_responsabilidade_civil`, `assessoria_juridica`, `assessoria_contabil`, `assessoria_seguranca_trabalho`, `conformidade_lgpd`, `conformidade_nr1`, `programa_compliance`, `selo_reclame_aqui`, `outras_certificacoes`, `premiacoes`.

Disclaimer obrigatório: _"Marcadores são informativos, baseados em autodeclaração, e não representam certificação oficial da plataforma."_

### Campos extras N2/N3

- `mini_bio_profissional` (texto até 500 chars)
- `video_institucional` (upload, trava trimestral — Req 29)
- `formacao_certificacoes` (JSON: `[{certificacao, data_obtencao}]`)
- `administradoras_vinculadas` (lista de empresa IDs)

### Requisitos MUST

- Perfil imutável após onboarding (edição limitada a campos não-críticos)
- Foto profissional em alta qualidade (mín. 400×400)
- Visibilidade: público para N2/N3, oculto para N1

---

## Req 6.3 — Perfil da Empresa

Campos de cadastro, compliance markers autodeclarados e restrições de conteúdo.

### Campos obrigatórios

- `logo` (upload)
- `razao_social`
- `nome_fantasia`
- `cnpj` (validado, único no escopo de empresas)
- `data_aniversario`
- `representante_legal`
- `email_comercial`
- `telefone_comercial`
- `endereco_comercial_cep` (validado via ViaCEP)
- `categoria_principal` (enum: lista definida em config)
- `subcategorias_atuacao` (até 5, múltipla seleção)

### Compliance Markers (25 autodeclarados)

ISOs (9001, 14001, 45001, 37001, 37301), ESG, selos, certificações, NRs. Checkbox list com disclaimer igual ao de Síndico.

### Regras de conteúdo

- ❌ Sem chamadas comerciais óbvias em vídeos públicos
- ❌ Sem preços no conteúdo público
- ❌ Sem dados de contato (telefone, WhatsApp) no conteúdo público — direcionar via Connect Me

### Requisitos MUST

- Unicidade de CNPJ (validação Receita Federal — adapter Sprint 2+)
- Logo em alta qualidade (mín. 200×200)
- Categoria principal obrigatória, subcategorias desejadas
- Visibilidade: Base visível apenas para Síndico Gestor/Plus/Pro; Plus/Pro visível para todos

---

## Req 6.4 — Perfil do Parceiro da Vizinhança

Campos mínimos para comércio local (CPF aceito, CNPJ não obrigatório).

### Campos obrigatórios

- `logo` (upload)
- `razao_social_nome`
- `nome_fantasia`
- `cnpj_cpf` (CPF aceito — validação CPF via algoritmo, CNPJ via Receita)
- `endereco_cep` (validado via ViaCEP)

### Campos opcionais

- `whatsapp`
- `instagram`
- Até 3 fotos (galeria)

### Requisitos MUST

- Cadastro simplificado (menos campos que Empresa)
- Visibilidade: público para comunidades locais

---

## Req 7 — Registro de Condomínio

Estrutura de tenant com `tenant_type='condominium'`, mandato e vinculação de empresa síndica.

### Campos de registro

- `address` (rua, número, complemento)
- `cep` (validado via ViaCEP adapter)
- `condominium_type` (enum: `residencial`, `comercial`, `misto`, `shopping_center`, `galeria_comercial`, `edificio_garagem`)
- `total_units` (int)
- `blocks_or_towers` (int, opcional)

### Identidade do condomínio

- **ID alfanumérico único**: 8 caracteres (ex: `CD2024AB`) — gerado via algoritmo
- **QR Code**: gerado automaticamente para identificação de unidades

### Mandato e Empresa Síndica

- `mandate_type` (enum: `sindico_eleito`, `sindico_profissional`, `sindico_interino`)
- `mandate_start_date`, `mandate_end_date`
- Empresa síndica: token por email + permissões configuráveis + audit log

### Requisitos MUST

- Validação CEP obrigatória (via ViaCEP adapter)
- Tenant criado automaticamente no registro (multi-tenancy)
- Limite: **15 condomínios por síndico** (Decision 11)
- Mandato: obrigatório, datas validadas (end > start)
- Audit log de todos os acessos/modificações

### Requisitos SHOULD

- Suporte a transferência de mandato (motivo obrigatório — Req 17)
- Integração com prefeitura para CEP/endereço (Sprint 4+)

### Notas de implementação

- Implementação atual: `internal/modules/institutional/`
- Tabelas: `condominiums`, `mandates`, `syndic_companies`
- Geração de ID: utilizar UUID + algoritmo de encode para 8 chars alfanuméricos

---

## Req 8 — Seletor de Condomínio

Síndico com múltiplos condomínios troca contexto entre eles.

### Requisitos MUST

- Dropdown/seletor de condomínios no header
- Persiste seleção na sessão (cookie/localStorage)
- Mudança de contexto: invalidar cache ABAC local
- Impacto em: filtros padrão de Timeline, Assembleia, relatórios

### Notas de implementação

- Sessão carrega `selected_condominium_id` via middleware Authenticate
- ABAC valida se user tem acesso ao condo selecionado

---

## Req 9 — Registro de Unidade

Estrutura de unidades habitacionais com titular e dependentes.

### Estrutura

- **1 titular** (morador principal com poder de voto)
- **Até 2 dependentes** (familiares, sem voto na assembleia mas acesso leitura)

### Dados de unidade

- `condominium_id` (FK)
- `unit_identifier` (ex: "101", "Apto 501", "Bloco B - 302")
- `unit_type` (enum: `apartamento`, `casa`, `sala_comercial`, `loja`, `garagem`, `outro`)
- Fração ideal (DECIMAL para votação — T10, Req 57)

### Requisitos MUST

- Upload de foto + termo de ciência obrigatório (LGPD — rule 10)
- Validação de unidade única por condomínio
- Revogação de dependente: imediata bloqueia login daquele perfil

### Notas de implementação

- Tabelas: `units`, `unit_members`
- Fração ideal: `NUMERIC(5,4)` no PostgreSQL (nunca float)

---

## Req 10 — Gestão de Usuários (Condomínio)

Limite de 3 usuários síndicos por condomínio com matriz de permissões.

### Roles por condomínio

- `sindico_principal` (1 obrigatório)
- `sindico_auxiliar` (até 2)
- `assistente` (até 3, sem poder de voto)

### Matriz de permissões (toggles)

| Permissão | Principal | Auxiliar | Assistente |
|-----------|-----------|----------|-----------|
| Criar atividade timeline | ✅ | ✅ | ❌ |
| Validar execução empresa | ✅ | ✅ | ✅ |
| Enviar comunicados | ✅ | ✅ | ❌ |
| Presidir assembleia | ✅ | ❌ | ❌ |
| Editar perfil condomínio | ✅ | ❌ | ❌ |
| Convidar novos usuários | ✅ | ✅ | ❌ |

### Requisitos MUST

- Convite por email com token de aceitação
- Revogação imediata invalida sessão ativa
- Audit log de todas as mudanças (quem, quando, o quê)

### Notas de implementação

- Tabelas: `condominium_users`, `user_permissions`
- Enum: `user_role_type` com `principal`, `auxiliary`, `assistant` (T8)

---

## Req 11 — Timeline (Linha do Tempo)

Estrutura imutável de eventos do condomínio com 7 tipos.

### Tipos de entrada

1. `atividade_da_gestao` — atividade planejada/executada do Plano Diretor
2. `registro_de_execucao` — empresa registra serviço realizado
3. `comunicado` — notícia/aviso do síndico
4. `evento` — assembleias, reuniões, datas importantes
5. `adendo` — modificação de registro anterior (único mecanismo — Req 25)
6. `resultado_consulta_moradores` — resultado de enquete/consulta
7. `assembly_result` — resultado de assembleia

### Requisitos MUST

- **Append-only, imutável após publicação** — nunca edit, nunca delete (Rule 2)
- Único mecanismo de modificação: adendo linkado ao original
- Auto-update de Plano Diretor via evento NATS (Req 12)
- Visibilidade moradores: modo leitura (sem ações, apenas consulta)
- Timestamp, autor, assinatura digital (Sprint 3+)

### Notas de implementação

- Tabelas: `timeline_entries`
- Campo `previous_entry_id` para adendos (chain de modificações)
- Evento NATS: `timeline.entry.created` → dispara cálculo de Plano Diretor

---

## Req 12 — Plano Diretor

Status sincronizado com Timeline. 26 áreas impactadas, 9 riscos, 8 próximas ações.

### Status (7 possíveis)

- `planejada` → `em_contratacao` → `em_execucao` → `concluida`
- `suspensa`, `reprogramada`, `atrasada` (desvios)

### Requisitos MUST

- **Status NUNCA muda por UPDATE manual** — muda **exclusivamente** via evento Timeline (Rule 2)
- Cálculo automático: entrada Timeline de tipo `atividade_da_gestao` cria/atualiza entrada Plano Diretor
- 26 áreas: predefinidas em configuração (condomínio pode mapear atividades a áreas)
- 9 riscos: matriz associada a cada atividade
- 8 próximas ações: sugestão automática baseada em status anterior

### Notas de implementação

- Tabelas: `master_plans`, `master_plan_activities`, `master_plan_risks`, `master_plan_next_actions`
- Evento NATS: listener em `timeline.entry.created` → `UpdateMasterPlan`
- Nunca usar UPDATE direto; transição via Timeline

---

## Req 13 — Eventos do Condomínio

13 tipos de evento com confirmação de participação e ciência.

### Tipos (exemplos)

Assembleia, reunião de síndico, manutenção programada, data comercial, feriado, plantão de emergência, etc.

### Requisitos MUST

- 13 tipos predefinidos configuráveis
- Confirmação de participação por morador (sim/não/talvez)
- Ciência obrigatória (tracking 6 meses — Req 50.2)
- Notificações automáticas: 3 dias, 1 dia, 3 horas antes

### Notas de implementação

- Tabelas: `condominium_events`, `event_confirmations`

---

## Req 14 — Comunicados

8 tipos, 4 níveis de prioridade, com tracking de leitura.

### Tipos × Prioridades

| Tipo | Exemplo | Níveis de prioridade |
|------|---------|---|
| Avisos gerais | Manutenção de água | Baixa, Média, Alta, Urgente |
| Informes | Resultados auditoria | Baixa, Média, Alta, Urgente |
| Solicitações | Pagar taxa extra | Baixa, Média, Alta, Urgente |
| Regras | Mudança regimento interno | Baixa, Média, Alta, Urgente |

### Requisitos MUST

- Tracking de leitura por morador (timestamp)
- Notificações push/email por prioridade
- Arquivo de comunicados (mín. 1 ano de retenção)

### Notas de implementação

- Tabelas: `communications`, `communication_reads`
- Badge no app: "N comunicados não lidos"

---

## Req 14.1 — Consultas aos Moradores

3 tipos: sim/não, múltipla escolha, escala 1-5. Resultado → Timeline.

### Tipos

1. `sim_nao_nao_sei` — pergunta binária com "não sei"
2. `multiple_choice` — até 5 opções
3. `scale_1_to_5` — escala Likert

### Requisitos MUST

- Anônima para síndico (só vê contagem, não nomes)
- Resultado automático → entrada Timeline tipo `resultado_consulta_moradores`
- Prazo de votação configurável (ex: 3 dias)
- Notificação automática convidando participação

### Notas de implementação

- Tabelas: `surveys`, `survey_responses`

---

## Req 15 — Avaliação de Gestão

Pesquisa a cada 2 meses com 3 perguntas escala 1-10.

### Requisitos MUST

- **Anônima para síndico** (não vê nome do avaliador)
- 3 perguntas fixas escala 1-10 (ou Likert customizável)
- Bloqueio: se atrasada, síndico não consegue fazer certas ações (export relatórios, criar atividade)
- Frequência: a cada 2 meses (365 ÷ 6 ≈ 61 dias)
- Notificação obrigatória quando vence

### Notas de implementação

- Tabelas: `governance_assessments`, `assessment_responses`

---

## Req 16 — Gestão de Dependentes

Titular adiciona até 2 dependentes com revogação imediata.

### Requisitos MUST

- Dependente criado pelo titular via convite ou adicionar direto
- Revogação imediata: bloqueia login e acesso leitura
- Dependente: acesso leitura-apenas (não pode votar em assembleia)

### Notas de implementação

- Tabelas: `unit_members` com `member_type` (titular/dependent)

---

## Req 17 — Encerramento de Vínculo

Revoga acesso de titular e dependentes com motivo obrigatório.

### Requisitos MUST

- Motivo obrigatório (enum: transferência, inadimplência, saída voluntária, outro)
- Revoga acesso: titular + dependentes → logout imediato
- Mantém histórico: dados permanecem para auditoria (hard delete nunca)
- Possibilidade de reativar em `X` dias (configurável)

### Notas de implementação

- Campo `revoked_at` em `unit_members`
- Audit log detalhado

---

## Invariantes de institutional

1. Timeline é append-only — nunca DELETE, nunca UPDATE diretamente
2. Plano Diretor status muda apenas via Timeline
3. Um condomínio tem 15 síndicos no máximo (por conta de síndico)
4. Um síndico pode ter até 3 usuários (principal + 2 auxiliares + assistentes)
5. Fração ideal é NUMERIC, nunca float
6. Mandato: end_date > start_date
7. Dependente: máximo 2 por unidade

---

## Referências cruzadas

- **Req 29** (Content): trava trimestral de vídeos institucionais
- **Req 48-60** (Assembly): relacionamento de eventos, votação, homologação
- **Req 25** (Commercial): adendo como mecanismo de modificação
- **Personas e Planos**: `personas-and-plans.md` — Síndico × N1/N2/N3
- **Cross-Domain**: Req 33-47 compliance e governance score
