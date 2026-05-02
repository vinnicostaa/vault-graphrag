---
title: Functional Requirements — Institutional BC
type: requirement
tags:
  - requirements
  - functional
  - institutional
  - condominium
  - timeline
  - master-plan
  - announcement
  - master-sindico
  - v2
  - ears
source: >-
  legacy specs/requirements/institutional + cross-domain + contextos/telas +
  _chaos/requirements(6).md (2026-04-13) + _chaos/mastersindico-requirements.pdf
  (2026-03-09)
created: 2026-04-23T00:00:00.000Z
updated: '2026-04-25'
fase_c_absorvido: true
---

# Functional Requirements — Institutional BC

> **Banner Fase C (2026-04-25)**: enriquecido com deltas absorvidos do `_chaos/requirements(6).md` (R11/R12/R15/R36/R42/R47) e `mastersindico-requirements.pdf`. Termos legados (N1/N2/N3, "Morador Pagante", Mercado Pago, Mandate-aggregate) já traduzidos para canônico vigente (D-094..D-135). Telas (S1-S31) NÃO processadas — vão pra Fase D em `03-subprojects/web/ui-catalog/`.

Requisitos funcionais do **bounded context `institutional`**. Responsabilidade: condomínio (cadastro/tenant), unidades com fração ideal, memberships, timeline imutável, plano diretor, comunicados, eventos, enquetes, governance score.

Formato **EARS**. Prioridade: M (M1), S (M2), C (M3+), W (backlog).

> Destilado de `../../90-ingested/from-vault-50-projects-master-sindico/specs/requirements/institutional.md` + `cross-domain.md` + `novo escopo-7.md` + `telas.md` + `_chaos/requirements(6).md` (2026-04-13) + `_chaos/mastersindico-requirements.pdf` (2026-03-09, absorvido em Fase C 2026-04-25).

---

## INS-001 — Criar condomínio (tenant)

**Prioridade**: M  
**EARS**: *Quando* síndico emite `POST /condominiums`, *o sistema deve* validar CEP via ViaCEP, criar tenant com `tenant_type='condominium'`, gerar ID alfanumérico único 8 chars (ex: `CD2024AB`), gerar QR Code, criar `mandate` inicial, criar membership do síndico-criador como `role_type=principal`.

**Campos obrigatórios**: `address`, `cep` (validado), `condominium_type ∈ {residencial, comercial, misto, shopping_center, galeria_comercial, edificio_garagem}`, `total_units`, `blocks_or_towers` (opcional), `mandate_type ∈ {sindico_eleito, sindico_profissional, sindico_interino}`, `mandate_start_date`, `mandate_end_date`.

**Dependências**: GR-003, BIL-018 (quota 15/conta).

**Critério de aceite**:
- Hard limit 15 condomínios por síndico (BIL-018).
- Validação `mandate_end_date > mandate_start_date`.
- Audit trail: `institutional.condominium.created`.

**Fonte**: legado `institutional.md` Req 7.

---

## INS-002 — Transferência de mandato (síndico → síndico)

**Prioridade**: S  
**EARS**: *Quando* síndico emite `POST /condominiums/{id}/transfer-mandate {to_email, reason}`, *o sistema deve*: (1) validar `compliance_status = em_dia` (R47 §8 — bloqueia se atrasado, lista pendências); (2) gerar token invite TTL 7d ao email do novo síndico; (3) **transferir** `condominium_data, timeline_history, master_plan_actions, closed_deals, compliance_records`; (4) **NÃO transferir** `user_accounts` nem `personal_evaluations` (R47 §4 — ressetadas); (5) marcar Membership atual `mandate_ended_at = now()`, `mandate_end_reason = 'transferred'`, `transferred_to_user_id` (D-129 flags); (6) ao aceite do novo síndico → criar nova Membership com `is_active_mandate = true`, `mandate_started_at = now()`; (7) emitir `MandateTransferred` event; (8) publicar entrada Timeline tipo `atividade_da_gestao`.

**Critério de aceite**:
- Histórico Timeline visível pro novo síndico (read-only — append-only INS-012).
- Avaliações de gestão (INS-031) **NÃO** herdadas; novo mandato gera score zerado.
- Avaliações ESCONDIDAS (D-104) anteriores **nunca acessadas** pelo novo síndico (membership_id mudou; HMAC ADR-0028 diferente).
- `AssemblyAdministratorLink` (D-127) **não herda automaticamente** durante assembleia ativa — novo síndico precisa convidar de novo.
- Audit trail completo (`audit_trail` + `lgpd_logs`).
- Compliance check: bloqueio se compliance ≠ em_dia, retorna lista de pendências.

**Fonte**: legado `cross-domain.md` Req 47, `institutional.md` Req 17; **R47 absorção Fase C**; D-129 (Mandate consolidado em Membership flags); D-104 (avaliação escondida); D-127 (AssemblyAdministratorLink).

---

## INS-003 — Encerramento de mandato

**Prioridade**: S  
**EARS**: *Quando* síndico principal inicia encerramento, *o sistema deve* validar `compliance = em_dia` (IDN-024, declaração anual), gerar dossiê PDF (Req 40), marcar condomínio como "sem síndico".

**Critério de aceite**:
- Bloqueio se compliance atrasada.
- Dossiê armazenado em S3.

**Fonte**: legado `cross-domain.md` Req 46.

---

## INS-004 — Empresa síndica (token por email)

**Prioridade**: C (M5+)  
**EARS**: *Quando* síndico vincula empresa síndica ao condomínio, *o sistema deve* gerar token por email com permissões configuráveis + audit log.

**Critério de aceite**:
- Ver backlog (M5+).

**Fonte**: legado `institutional.md` Req 7, [[00-product/portfolio-de-produtos]] anti-catálogo.

---

## INS-005 — Unidade CRUD com fração ideal

**Prioridade**: M  
**EARS**: *Quando* síndico emite `POST /condominiums/{id}/units`, *o sistema deve* criar unidade com `{condominium_id, unit_identifier, unit_type, fracao_ideal NUMERIC(5,4)}`. *Quando* soma de frações > 1.0000 → `422 validation_error code=fraction_sum_exceeded`.

**Dependências**: GR-033.

**Critério de aceite**:
- DB CHECK `fracao_ideal > 0 AND fracao_ideal <= 1.0000`.
- Trigger/validação: Σ unidades ativas ≤ 1.0000 por condomínio.
- `unit_identifier` UNIQUE por condomínio.
- Tipos: `apartamento`, `casa`, `sala_comercial`, `loja`, `garagem`, `outro`.

**Fonte**: legado `institutional.md` Req 9.

---

## INS-006 — Unidade: upload de foto + termo

**Prioridade**: M  
**EARS**: *Quando* unidade é criada, *o sistema deve* exigir upload de foto e aceitação de termo LGPD (rule 10 legado).

**Critério de aceite**:
- Foto 5MB max, JPG/PNG/WebP.
- Termo versionado (GR-023).

**Fonte**: legado `institutional.md` Req 9.

---

## INS-007 — Unidade: membership titular + dependentes (DTO estrito)

**Prioridade**: M  
**EARS**: *Quando* titular é vinculado à unidade, *o sistema deve* criar membership `role=resident, role_type=titular`. *Quando* titular adiciona dependente, *o sistema deve* criar membership `role_type=dependent` (máximo 2 por unidade).

**DTO hardening (pós-[[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-012]])**:

- **`PATCH /api/v1/memberships/:id`** aceita **apenas** `{ role_type?, active?, metadata? }` no body.
- **Proibidos** no body: `user_id`, `condominium_id`, `tenant_id`, `company_id`, `role`, `is_admin`.
- `user_id` vem do **token do caller** (self) ou do **invite token** (IDN-032); nunca do body.
- `condominium_id` vem do **path** (`/condominiums/:cond_id/memberships/:id`) e ABAC valida membership ativo do caller no `cond_id`.
- PBT: body `{user_id: "<outro>", condominium_id: "<outro>"}` → sempre 422.
- INV-107 é a invariante.

**Dependências**: IDN-013, IDN-032, IDN-033, IDN-039 (DTO estrito), INV-107.

**Critério de aceite**:
- Invariante: máx 1 titular + 2 dependentes por unidade.
- Dependente: acesso leitura, sem voto.
- DTO `UpdateMembershipRequest` strict (OpenAPI `additionalProperties: false`).
- CI grep bloqueia handlers aceitando `user_id` / `condominium_id` em body de PATCH.

**Fonte**: legado `institutional.md` Req 9, Req 16; DTO hardening pós-[[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-012]].

---

## INS-008 — Encerramento de vínculo (morador)

**Prioridade**: S  
**EARS**: *Quando* síndico emite encerramento de membership morador, *o sistema deve* exigir `reason ∈ {transferencia, inadimplencia, saida_voluntaria, outro}`, revogar acesso titular + dependentes, preservar histórico (sem hard delete).

**Critério de aceite**:
- Campo `unit_members.revoked_at`.
- Logout imediato (IDN-010 adaptado).
- Reativação possível em X dias (configurável).

**Fonte**: legado `institutional.md` Req 17.

---

## INS-009 — Seletor de condomínio (header)

**Prioridade**: M  
**EARS**: *Quando* síndico com > 1 condomínio acessa app, *o sistema deve* expor seletor no header; escolha persiste em sessão; mudança invalida cache ABAC local e impacta filtros padrão (Timeline, Assembly, relatórios).

**Dependências**: IDN-014.

**Critério de aceite**:
- Dropdown visual.
- `GET /me/condominiums` retorna lista.

**Fonte**: legado `institutional.md` Req 8.

---

## INS-010 — Gestão de usuários do condomínio (síndicos co-admins)

**Prioridade**: S  
**EARS**: *Quando* síndico principal convida co-admin, *o sistema deve* gerar token invite + permissões por role_type conforme matriz:

| Permissão | principal | auxiliar | assistente |
|---|---|---|---|
| Criar atividade timeline | ✓ | ✓ | ✗ |
| Validar execução empresa | ✓ | ✓ | ✓ |
| Enviar comunicado | ✓ | ✓ | ✗ |
| Presidir assembleia | ✓ | ✗ | ✗ |
| Editar perfil condomínio | ✓ | ✗ | ✗ |
| Convidar novos usuários | ✓ | ✓ | ✗ |

**Dependências**: IDN-032.

**Critério de aceite**:
- Limites: 1 principal + 2 auxiliares + 3 assistentes.
- Audit: todas mudanças registradas.

**Fonte**: legado `institutional.md` Req 10.

---

## INS-011 — Timeline: 7 tipos de entrada

**Prioridade**: M  
**EARS**: *Quando* síndico emite POST em qualquer fonte de timeline, *o sistema deve* persistir em `timeline_entries` com `type ∈ {atividade_da_gestao, registro_de_execucao, comunicado, evento, adendo, resultado_consulta_moradores, assembly_result}`.

**Critério de aceite**:
- Tabela append-only (ver INS-012).
- Campos: `{id, condominium_id, type, title, description, metadata_json, previous_entry_id, created_by, created_at, signature_hash}`.

**Fonte**: legado `institutional.md` Req 11.

---

## INS-012 — Timeline imutável (INSERT-only)

**Prioridade**: M  
**EARS**: *Quando* tentativa de `UPDATE timeline_entries` ou `DELETE` → DB trigger rejeita com exception. *Quando* correção necessária → nova entry com `type=adendo` + `previous_entry_id`.

**Dependências**: GR-030.

**Critério de aceite**:
- DB trigger `prevent_mutation_timeline`.
- Sem coluna `deleted_at`.
- PBT: qualquer sequência de requests nunca remove row.

**Fonte**: legado `institutional.md` Req 11, [[STEERING]] §41.

---

## INS-013 — Timeline → Master Plan sync automático

**Prioridade**: M  
**EARS**: *Quando* entry tipo `atividade_da_gestao` ou `registro_de_execucao` é inserida, *o sistema deve* publicar evento `timeline.entry.created` via NATS. Listener em `institutional.master_plan` consome e atualiza `master_plan_activities` correspondente.

**Dependências**: GR-035.

**Critério de aceite**:
- Status master_plan NUNCA muda via UPDATE manual — só via timeline event.
- Outbox pattern pra garantir entrega.

**Fonte**: legado `institutional.md` Req 11, Req 12.

---

## INS-014 — Timeline visibility

**Prioridade**: M  
**EARS**: *Quando* morador emite `GET /condominiums/{id}/timeline`, *o sistema deve* retornar entries em modo read-only (sem actions). *Quando* usuário não é membership ativo do condomínio → 403.

**Critério de aceite**:
- Paginação cursor (GR-016).
- Filtros por type, período.

**Fonte**: legado `institutional.md` Req 11.

---

## INS-015 — Timeline: assinatura digital (M3+)

**Prioridade**: C (M3+)  
**EARS**: *Quando* entry é criada, *o sistema deve* gerar `signature_hash = HMAC(prev_hash + entry_data, secret)` pra integridade tipo blockchain.

**Critério de aceite**:
- Hash chain verificável via `GET /condominiums/{id}/timeline/verify`.
- Integrado com assinatura digital BR (MP 2.200-2) quando aplicável.

**Fonte**: legado `institutional.md` Req 11 notas.

---

## INS-016 — Adendo: chain de modificações

**Prioridade**: M  
**EARS**: *Quando* adendo é criado, *o sistema deve* vincular `previous_entry_id` ao original. *Quando* adendo sobre adendo → chain válida (ex: A → B → C).

**Critério de aceite**:
- `previous_entry_id` FK em `timeline_entries`.
- Endpoint `GET /timeline_entries/{id}/chain` retorna árvore de adendos.

**Fonte**: legado `commercial.md` Req 25.

---

## INS-017 — Master Plan: 26 áreas pré-definidas

**Prioridade**: M  
**EARS**: *Quando* condomínio é criado, *o sistema deve* inicializar master_plan com 26 áreas default. *Quando* síndico cria atividade no master_plan, *o sistema deve* exigir `area_impactada` em enum fixo (config seed, não editável).

**Áreas canônicas (26 itens — Fase C absorção R12 §2 do `_chaos/requirements(6).md`)**:

```
estrutura_predial, sistema_hidraulico, sistema_eletrico, sistema_incendio,
reservatorios, elevadores, garagem, fachada, cobertura, seguranca_patrimonial,
administracao, portaria, playground, academia, salao_festas, corredores,
jardins, casa_maquinas, rede_esgoto, rede_drenagem, sistema_gas, iluminacao,
sistema_cameras, controle_acesso, areas_comuns, outros
```

**Tipos de atividade canônicos (26 itens — R12 §3)**:

```
manutencao_preventiva, manutencao_corretiva, inspecao_tecnica, vistoria_tecnica,
contratacao_servico, execucao_servico, reparo_emergencial, obra_melhoria,
adequacao_normativa, atualizacao_infraestrutura, monitoramento_ambiental,
revisao_tecnica, auditoria_tecnica, limpeza_tecnica, controle_pragas,
limpeza_reservatorios, treinamento_tecnico, adequacao_legal, revisao_equipamentos,
atualizacao_administrativa, contratacao_fornecedor, encerramento_contrato,
avaliacao_fornecedor, implantacao_sistema, atualizacao_normas, acao_emergencial
```

**Critério de aceite**:
- Seed: 26 áreas + 26 tipos atividade em migration `master_plan_taxonomies` ou Postgres ENUM.
- Glossário em [[../../01-domain/ubiquitous-language]] (a destilar) + [[../../01-domain/business-rules]].
- `outros` aceito como fallback (livre texto curto).
- Localização: enums em pt-BR (sem i18n M1).

**Fonte**: legado `institutional.md` Req 12; **R12 §2-3 absorção Fase C (`_chaos/requirements(6).md` + `mastersindico-requirements.pdf`)**.

---

## INS-018 — Master Plan: 7 status (transições controladas)

**Prioridade**: M  
**EARS**: *Quando* atividade master_plan muda estado, *o sistema deve* transicionar entre `{planejada, em_contratacao, em_execucao, concluida, suspensa, reprogramada, atrasada}` exclusivamente via **eventos de Timeline** (INS-013) — UPDATE manual rejeitado por trigger DB. *Quando* há tentativa de UPDATE direto em `master_plan_activities.status` → trigger rejeita com exception.

**Status canônicos (7 itens — R12 §6 absorção Fase C)**:

```
planejada → em_contratacao → em_execucao → concluida
              ↓                  ↓
         suspensa | reprogramada | atrasada (estados de desvio)
```

**Critério de aceite**:
- Enum Postgres `master_plan_status` validado.
- Trigger `prevent_manual_status_update_master_plan` (DB-level, não app-level).
- State machine documentada em `01-domain/state-machines.md` (a criar/atualizar).
- `concluida` é terminal (cancelamento via `suspensa`).

**Fonte**: legado `institutional.md` Req 12; **R12 §6-§8 absorção Fase C**; INS-013 (sync via Timeline).

---

## INS-019 — Master Plan: 9 riscos + 8 próximas ações (taxonomia fixa)

**Prioridade**: S  
**EARS**: *Quando* síndico cria atividade master_plan, *o sistema deve* aceitar **um** `risco_associado` (enum 9 itens) e **uma** `proxima_acao` (enum 8 itens). *Quando* `risco_associado ∈ {risco_estrutural, risco_incendio, risco_juridico, risco_eletrico, risco_ambiental}` E status = `atrasada` → calcular `risk_score` alto (formula em ADR pendente; baseline R38 `score = f(risco_associado, days_overdue, estimated_cost)`).

**Riscos associados (9 itens — R12 §4)**:

```
sem_risco, risco_operacional, risco_estrutural, risco_sanitario,
risco_eletrico, risco_hidraulico, risco_incendio, risco_juridico,
risco_ambiental
```

**Próximas ações (8 itens — R12 §5)**:

```
monitorar_evolucao, nova_inspecao, contratar_fornecedor,
executar_manutencao, atualizar_plano_diretor, registrar_conclusao,
acompanhar_garantia, reavaliar_necessidade
```

**Critério de aceite**:
- Enum Postgres `master_plan_risk` (9), `master_plan_next_action` (8).
- Sugestão de `proxima_acao` automática baseada em `(status, risco_associado)` — função pura `SuggestNextAction(activity)`.
- Risk score formula: ADR-pendente; baseline `score = (risk_weight × days_overdue) + cost_factor` (R38 §2).

**Fonte**: legado `institutional.md` Req 12; **R12 §4-§5 absorção Fase C**; R38 (Risk Map) consolidado em INS-036.

---

## INS-020 — Master Plan: export PDF (`plan_tier ∈ {plus, pro}`)

**Prioridade**: S  
**EARS**: *Quando* síndico com `plan_tier ∈ {plus, pro}` (D-067/D-081/D-096) emite `GET /condominiums/{id}/master-plan/export?format=pdf`, *o sistema deve* gerar PDF consolidado.

**Dependências**: BIL-021 feature flag.

**Critério de aceite**:
- N1 → 403.
- Async job se > 100 atividades.
- Template HTML → PDF via wkhtmltopdf.

**Fonte**: legado `novo escopo-7.md` §2.3, `institutional.md` Req 12.

---

## INS-021 — Comunicado: 8 tipos × 4 prioridades

**Prioridade**: M  
**EARS**: *Quando* síndico cria comunicado, *o sistema deve* aceitar `type` em enum 8 itens (R-_chaos canônico) × `priority ∈ {baixa, media, alta, urgente}`.

**Tipos canônicos de comunicado (8 itens — R-_chaos enums absorção Fase C)**:

```
aviso_operacional, interrupcao_programada, orientacao_moradores,
aviso_seguranca, comunicado_institucional, conclusao_servico,
recomendacao_manutencao, alerta_preventivo
```

> **Nota Fase C**: lista anterior `{aviso_geral, informe, solicitacao, regra, evento, emergencia, financeiro, manutencao}` foi **revisada** com base em `_chaos/requirements(6).md` (mais alinhada à terminologia condominial técnica). Migrar enum se DB já populado (expand-contract).

**Critério de aceite**:
- Tabela `communications` com enum Postgres `communication_type` (8) + `communication_priority` (4).
- Notificações diferenciadas por prioridade (urgente → push imediato, SMS opt-in; baixa → feed).

**Fonte**: legado `institutional.md` Req 14; **R-_chaos enums absorção Fase C**.

---

## INS-022 — Comunicado: publicação + lock (imutável pós-publish)

**Prioridade**: M  
**EARS**: *Quando* síndico publica comunicado (`POST /communications/{id}/publish`), *o sistema deve* travar `published_at` e tornar `title`, `content`, `priority` read-only. Correção via adendo.

**Critério de aceite**:
- Campo `published_at` NOT NULL triggera lock.
- Timeline entry criada automaticamente (`type=comunicado`).

**Fonte**: legado `institutional.md` Req 14, [[STEERING]] §41.

---

## INS-023 — Comunicado: tracking de leitura

**Prioridade**: M  
**EARS**: *Quando* morador visualiza comunicado, *o sistema deve* registrar `communication_reads {user_id, communication_id, read_at}`. *Quando* síndico consulta → retorna % de leitura.

**Critério de aceite**:
- UNIQUE `(user_id, communication_id)`.
- Badge no app: "N comunicados não lidos".

**Fonte**: legado `institutional.md` Req 14.

---

## INS-024 — Comunicado: retenção 1 ano mínimo

**Prioridade**: M  
**EARS**: *Quando* comunicado é publicado, *o sistema deve* manter ativo 1+ ano. Após arquiva em `communications_archive`.

**Critério de aceite**:
- Job cron mensal.
- Acesso via `GET /communications?archived=true`.

**Fonte**: legado `institutional.md` Req 14.

---

## INS-025 — Evento do condomínio: 13 tipos canônicos

**Prioridade**: S  
**EARS**: *Quando* síndico cria evento, *o sistema deve* aceitar `event_type` em enum 13 itens (R-_chaos canônico). *Quando* `notify_moradores=true`, *o sistema deve* enviar notificação por canal apropriado ao `event_type` (assembleia → push+email+SMS; outros → push).

**Tipos canônicos (13 itens — R13 + R-_chaos enums Fase C)**:

```
assembleia_ordinaria, assembleia_extraordinaria, manutencao_programada,
inspecao_tecnica, obra_programada, interrupcao_programada_servicos,
treinamento_equipe, campanha_institucional, evento_comunitario,
reuniao_administrativa, atualizacao_normativa, simulado_emergencia, outros
```

> **Nota Fase C**: lista anterior genérica ("data comercial, feriado, plantão emergência") foi **substituída** por enum literal canônico. Ver D-115 STATE (18 tipos com subcategorias é a versão expandida — manter 13 como base, 18 como expansão M2+).

**Critério de aceite**:
- Tabela `condominium_events` com enum Postgres `event_type` (13).
- `outros` aceito como fallback livre; demais 12 são fixos pra agregação/relatório.
- Validação: `start_date ≥ now()` na criação; transições `scheduled → in_progress → completed | cancelled` (state machine).

**Fonte**: legado `institutional.md` Req 13; **R13 + R-_chaos enums absorção Fase C**; D-115 (expansão 18 tipos M2+).

---

## INS-026 — Evento: confirmação de participação

**Prioridade**: S  
**EARS**: *Quando* morador recebe notificação de evento, *o sistema deve* permitir resposta `sim/nao/talvez`. Registra `event_confirmations`.

**Critério de aceite**:
- Única resposta por `(user_id, event_id)`.
- Alteração permitida até T-3h.

**Fonte**: legado `institutional.md` Req 13.

---

## INS-027 — Evento: ciência obrigatória (tracking 6 meses)

**Prioridade**: S  
**EARS**: *Quando* morador precisa dar ciência de evento importante, *o sistema deve* registrar `acknowledged_at` com tracking 6 meses.

**Critério de aceite**:
- Integra com assembleia ciência obrigatória (ASM-020).

**Fonte**: legado `institutional.md` Req 13.

---

## INS-028 — Evento: notificações T-3d, T-1d, T-3h

**Prioridade**: S  
**EARS**: *Quando* evento é confirmado, *o sistema deve* agendar notificações push/email em T-3 dias, T-1 dia, T-3 horas.

**Critério de aceite**:
- Scheduler (cron horário).
- Template por evento type.

**Fonte**: legado `institutional.md` Req 13.

---

## INS-029 — Consulta/enquete interna: 3 tipos

**Prioridade**: S  
**EARS**: *Quando* síndico cria consulta, *o sistema deve* aceitar `survey_type ∈ {sim_nao_nao_sei, multiple_choice_ate_5, scale_1_a_5}`.

**Critério de aceite**:
- Tabelas `surveys`, `survey_responses`.
- Prazo de votação configurável (ex: 3 dias).

**Fonte**: legado `institutional.md` Req 14.1.

---

## INS-030 — Consulta: anônima para síndico

**Prioridade**: S  
**EARS**: *Enquanto* consulta está aberta, *o sistema deve* exibir apenas contagem (não nomes). Resultado final → entrada Timeline tipo `resultado_consulta_moradores`.

**Critério de aceite**:
- Síndico vê `{option, count}`, não `user_id`.
- Notificação automática convidando participação.

**Fonte**: legado `institutional.md` Req 14.1.

---

## INS-031 — Avaliação de gestão pelo morador (bimestral, 3 perguntas escala 1-10)

**Prioridade**: S  
**EARS**: *Quando* passam-se 61 dias (≈2 meses) desde última avaliação, *o sistema deve* abrir `governance_assessments` para o morador com **3 perguntas fixas escala 1-10** (não customizáveis). *Enquanto* avaliação está atrasada (tracking 30+ dias após disponibilização), *o sistema deve* bloquear ações estratégicas do síndico (export de relatório, criação de atividade master_plan tipo `obra_melhoria`). *Quando* morador submete → grava `morador_evaluations` anônima (síndico vê só média + breakdown). *Quando* mandato encerra → fatia de 25% em INV-GOV-001 (D-103 governance score).

**Critérios canônicos (3 itens — R15 absorção Fase C)**:

| Critério | Texto canônico (pt-BR) | Escala |
|---|---|---|
| `satisfacao_gestao` | "Qual seu nível de satisfação com a gestão do síndico?" | 1-10 |
| `transparencia_comunicacao` | "Você considera que a gestão tem sido transparente na comunicação com os moradores?" | 1-10 |
| `percepcao_evolucao` | "Você percebe evolução na organização do condomínio durante esta gestão?" | 1-10 |

**Critério de aceite**:
- Anônima para síndico (síndico vê apenas `{criterion, avg_score, response_count}`).
- Perguntas FIXAS do sistema (sem customização — D-110 ratifica imutabilidade do texto pra comparabilidade cross-platform).
- Tabela `morador_evaluations {id, mandate_id_AKA_membership_id, criterion, score, anonymous_hash, submitted_at}` — `anonymous_hash = HMAC(membership_id, user_id)` per ADR-0028; síndico nunca consegue rastrear morador individual.
- Notificação automática T-7d, T-1d quando vence.
- Avaliação ESCONDIDA de eleição (D-104) é **distinta** dessa — ver ASM-ELE-AVAL em [[assembly]].

> **Nota Fase C — D-129**: refs anteriores a `mandate_id` traduzidas para `membership_id` (D-129 consolidou Mandate em flags na Membership). Subsíndicos com `is_active_mandate=true` também são avaliados.

**Fonte**: legado `institutional.md` Req 15, `novo escopo-7.md` §2.4; **R15 + R-_chaos Evaluation Criteria absorção Fase C**; D-103 (governance score); D-110 (texto fixo); D-129 (mandate_id → membership_id).

---

## INS-032 — Governance score (cross-domain)

**Prioridade**: S  
**EARS**: *Quando* job noturno roda, *o sistema deve* calcular `governance_score (0-100)` baseado em 10 componentes (ver [[../compliance-lgpd]] e Req 33 legado). *Quando* score < 60 → notificação ao síndico.

**Dependências**: GR-026, audit trail.

**Critério de aceite**:
- 10 componentes: LGPD, declarações, assinaturas, audit trail, reclamações, fundo reserva, assembleias, rotinas, avaliação, política.
- Histórico 12 meses em `governance_score_history`.

**Fonte**: legado `cross-domain.md` Req 33.

---

## INS-033 — Declaração anual de compliance

**Prioridade**: S  
**EARS**: *Quando* síndico atinge aniversário anual, *o sistema deve* solicitar declaração anual de conformidade (checklist). *Enquanto* atrasada → bloqueia export de dossiê.

**Critério de aceite**:
- Tabela `compliance_declarations`.
- Histórico 5 anos (LGPD).

**Fonte**: legado `cross-domain.md` Req 34.

---

## INS-034 — Dossiê PDF exportável (fim de mandato — bloqueio compliance)

**Prioridade**: S  
**EARS**: *Quando* síndico emite `POST /mandates/:membership_id/dossier/export`, *o sistema deve*: (1) **validar `compliance_status = em_dia`** — se NÃO em_dia, responde `409 compliance_blocked` com lista de `pending_items` (R42 §2-§3); (2) gerar job async que consolida `mandate_info, timeline_entries, master_plan_actions, closed_deals, compliance_records, evaluations (anônimas agregadas), governance_score (D-103 ambos), compliance_score`; (3) gerar PDF com assinatura digital (ICP-Brasil — Lei 14.063, M4+ pleno; M1-M3 stub digital signature timestamp) + `dossier_id` único; (4) armazenar em MinIO/R2 permanente; (5) emitir `DossierExported` event + `lgpd_logs` (acesso a dados pessoais agregados).

**Critério de aceite**:
- Validação compliance: `compliance_declarations` anual + assinaturas pendentes obrigatórias = 0 antes de liberar.
- Bloqueio retorna mensagem estruturada: `{blocked: true, missing: ["annual_declaration_2026", "term_acceptance_v3"], next_steps: [...]}`.
- Filtragem opcional `?date_range=...` (R42 §6).
- `dossier_id` UUIDv7 único pra rastreabilidade (R42 §7 + GR-024).
- TTL link presigned 48h (download); regerável.
- Síndico pode baixar pós-encerramento (acesso preservado via Membership.mandate_ended_at flag).

**Fonte**: legado `cross-domain.md` Req 40, `institutional.md` Req 40; **R42 absorção Fase C** (bloqueio compliance + assinatura digital + dossier_id); D-103 (governance + compliance scores).

---

## INS-035 — Fundo de Reserva tracking (manual M1, automático backlog)

**Prioridade**: C  
**EARS**: *Quando* síndico registra contribuição ao fundo de reserva, *o sistema deve* armazenar em `reserve_fund_contributions`.

**Critério de aceite**:
- Input manual (não integra ERP).
- Componente governance score.

**Fonte**: legado `cross-domain.md` Req 33 componente 6.

---

## INS-036 — Mapa de riscos

**Prioridade**: C  
**EARS**: *Quando* síndico atualiza mapa de riscos (anual), *o sistema deve* persistir em `risk_maps` + `risk_items` com tipos: financeiro, compliance, estrutural, social, segurança.

**Critério de aceite**:
- Integração com master_plan (atividades ligadas a riscos).

**Fonte**: legado `cross-domain.md` Req 38.

---

## INS-037 — Perfil público do condomínio

**Prioridade**: S  
**EARS**: *Quando* usuário autenticado acessa `GET /condominiums/{id}/public`, *o sistema deve* retornar perfil público: nome, endereço parcial, governance score, badge transparência, total de mandatos.

**Critério de aceite**:
- Dados públicos (sem PII de moradores).
- Endpoint autenticado (não anon).

**Fonte**: legado `cross-domain.md` Req 60.9 (indicador transparência).

---

## INS-038 — Widget indicador de transparência

**Prioridade**: C  
**EARS**: *Quando* condomínio tem ≥ 1 assembleia homologada, *o sistema deve* exibir `TransparencyBadge` com score e histórico 12 assembleias.

**Critério de aceite**:
- Componente SolidJS.
- Benchmark: "Acima da média" / "Média" / "Abaixo" (percentil).

**Fonte**: legado `assembly.md` Req 60.9, `cross-domain.md` Req 33.

---

## INS-039 — Perfil do síndico (extensão institutional)

**Prioridade**: M  
**EARS**: *Quando* síndico completa onboarding, *o sistema deve* persistir `SyndicProfile {user_id, professional_photo, full_name, birth_date, professional_email, phone, cpf, professional_address, city_state_of_operation, sindico_type, years_of_experience, condominiums_under_management}`. Campos N2/N3: `mini_bio_profissional, video_institucional_id, formacao_certificacoes (JSON), administradoras_vinculadas`.

**Dependências**: IDN-017, [[../registration-data]].

**Critério de aceite**:
- Foto profissional mín 400×400.
- `mini_bio_profissional` ≤ 500 chars.
- Vídeo com lock 90d (GR-029).
- Ver [[../registration-data]] Síndico.

**Fonte**: legado `institutional.md` Req 6.2, `DADOS PARA CADASTRO.md`.

---

## INS-040 — Governance markers (15 autodeclarados)

**Prioridade**: M  
**EARS**: *Quando* síndico edita governance markers, *o sistema deve* aceitar checkbox list: `membro_abracs, seguro_responsabilidade_civil, assessoria_juridica, assessoria_contabil, assessoria_seguranca_trabalho, conformidade_lgpd, conformidade_nr1, programa_compliance, selo_reclame_aqui, outras_certificacoes, premiacoes` (total 15 com campo específico outras/premiações em JSON).

**Critério de aceite**:
- Disclaimer fixo: _"Marcadores são informativos, baseados em autodeclaração, e não representam certificação oficial da plataforma."_
- Sem validação externa.

**Fonte**: legado `institutional.md` Req 6.2, `DADOS PARA CADASTRO.md`.

---

## INS-041 — Perfil do morador (extensão institutional)

**Prioridade**: M  
**EARS**: *Quando* morador completa onboarding, *o sistema deve* persistir `ResidentProfile {user_id, full_name, birth_date, email, phone, cpf, cep, condominium_id, endereco, photo (optional), video_curriculum_id (optional morador pagante)}`.

**Dependências**: [[../registration-data]].

**Critério de aceite**:
- `cpf` imutável (IDN-029).
- Validação CPF via algoritmo.
- Vídeo-currículo (pagante): BIL-019, GR-029.

**Fonte**: legado `institutional.md` profiles, `DADOS PARA CADASTRO.md`, `ONBOARDING_CADASTRO_VS_PERFIL.md`.

---

## INS-042 — Busca de condomínios (por ID, nome, CEP)

**Prioridade**: M  
**EARS**: *Quando* morador novo precisa se vincular, *o sistema deve* expor `GET /condominiums/search?q=<string>` com busca por ID alfanumérico, nome ou CEP.

**Critério de aceite**:
- Full-text via `ISearchProvider` real (OpenSearch ou Meilisearch — ADR-0042 pending; D-120 Fase 14 revoga D-067 PG tsvector — GR-011).
- Só retorna condomínios públicos (não privados).

**Fonte**: legado `institutional.md` Req 7, `novo escopo-7.md` §1.5.

---

## INS-043 — Paywall Base Morador: perfil empresa visível a Síndico Gestor+

**Prioridade**: M  
**EARS**: *Quando* morador Base tenta acessar perfil detalhado de empresa, *o sistema deve* retornar preview limitado (2 fotos + 1 vídeo 60s — `novo escopo-7.md` §1.3 matriz). Full visibility só Síndico Gestor (N2) / Plus / Pro.

**Dependências**: [[../matrix-functional]].

**Critério de aceite**:
- Ver matriz Perfil e Visibilidade.

**Fonte**: legado matriz funcional, `content.md` Req 29 Rule 4.

---

## Fase C — Deltas absorvidos 2026-04-25

### INS-044 — Conflict of Interest Matrix (síndico declara relações com fornecedores)

**Prioridade**: S (M2)  
**EARS**: *Quando* síndico completa onboarding, *o sistema deve* exigir declaração de conflitos de interesse (matriz). *Quando* aniversário anual da última declaração se aproxima (T-30d), *o sistema deve* notificar pra atualização. *Quando* síndico cria deal (COM-019) com empresa onde há conflito declarado, *o sistema deve* flaggear no UI ("⚠️ Conflito de interesse declarado") + emitir `ConflictDealFlagged` event + exigir confirmação dupla. *Quando* assembleia tem `votacao_fornecedor` (ASM-021) envolvendo empresa com conflito do síndico → síndico marcado abstenção obrigatória (IDN-034 já existente).

**Estrutura da declaração** (R36 absorção Fase C):
- Entidade alvo: `target_type ∈ {company, supplier, service_provider}` + `target_id` (FK).
- Tipo de relação: `relationship_type ∈ {familiar, societario, financeiro, profissional, nenhum}`.
- Atualização: anual obrigatória OU quando nova relação emerge.
- Histórico append-only: `conflict_declarations {id, syndic_id, target_type, target_id, relationship_type, declared_at, valid_until, revoked_at}`.

**Critério de aceite**:
- Tabela `conflict_declarations` append-only.
- Display em `/syndic/{id}/compliance-dashboard` (visível ao próprio síndico + admin auditor).
- Evento `ConflictDeclared` emitido em qualquer mudança.
- Deal flow (COM-019) consulta antes de finalizar — não bloqueia, mas flagga + força confirmação dupla.
- Componente do governance_score (D-103) — peso negativo se há deals com conflito não-flagged.

**Dependências**: IDN-034 (declaração no onboarding já cobre escopo mínimo), COM-019, ASM-021, D-103.

**Fonte**: **R36 absorção Fase C** (`_chaos/requirements(6).md` + `mastersindico-requirements.pdf`).

---

### INS-045 — Mandate End (encerramento de mandato — bloqueio compliance)

**Prioridade**: S  
**EARS**: *Quando* síndico emite `POST /mandates/:membership_id/end {reason}`, *o sistema deve*: (1) **validar `compliance_status = em_dia`** (R46 §3-§4 — bloqueia se atrasado, lista pendências, retorna `409 compliance_blocked`); (2) aceitar `reason ∈ {fim_periodo, renuncia, destituicao, transferencia}` (R-_chaos enums); (3) setar Membership `mandate_ended_at = now()`, `mandate_end_reason = reason`, `is_active_mandate = false`, `mandate_end_document_url` (URL ata homologada ou descrição texto); (4) gerar relatório final automático (job async); (5) publicar entrada Timeline tipo `atividade_da_gestao` (R46 §7); (6) emitir `MandateEnded` event; (7) revogar acessos: todos os usuários do condomínio recebem notificação + login ressincroniza ABAC (cache invalidate); (8) preservar `compliance_records` no histórico (não deletar); (9) calcular agregação D-104 (avaliação escondida) sobre conjunto deste mandato (D-103 INV-GOV-001 fatia 25%).

**Mandate End Reasons (4 itens — R-_chaos absorção Fase C)**:

```
fim_periodo, renuncia, destituicao, transferencia
```

**Critério de aceite**:
- Validação compliance: bloqueia se `compliance_declarations` ano vigente faltando OU `compliance_status ≠ em_dia`.
- Histórico mandato preservado (D-129: query `SELECT * FROM memberships WHERE condominium_id=X AND role='syndic' ORDER BY mandate_started_at DESC`).
- Acessos do mandato revogados imediatamente (Membership.End() cascade).
- Diferente de INS-002 (transferência ativa novo síndico) — INS-045 é encerramento sem sucessor designado (condomínio fica "sem síndico" até nova eleição/contratação).

**Dependências**: D-129 (Mandate flags), D-103 (governance score), D-104 (avaliação escondida), INS-033 (declaração anual), INS-034 (dossier).

**Fonte**: legado `cross-domain.md` Req 46; **R46 absorção Fase C**; D-129.

---

### INS-046 — Master Plan: timeline auto-update (invariante explicitada)

**Prioridade**: M  
**EARS**: *Quando* entrada Timeline tipo `registro_de_execucao` é publicada com link a `master_plan_action_id`, *o sistema deve* (via NATS listener — INS-013) atualizar `master_plan_activities.status` automaticamente conforme regra:
- Execução concluída → `status = concluida` + `completed_at = now()`.
- Execução em andamento → `status = em_execucao`.
- Execução parcial / atraso → `status = atrasada`.

*Quando* há tentativa de UPDATE manual em `master_plan_activities.status` via API ou DB direto → trigger DB rejeita (INV-INS-046 nova).

**Invariante INS-046 (NEW Fase C)**: status de `master_plan_activities` muda APENAS via Timeline event consumido por handler — UPDATE direto rejeitado por trigger.

**Critério de aceite**:
- Trigger DB `prevent_master_plan_status_manual_update` (R12 §7).
- Listener NATS idempotente (consume event_id único).
- Outbox pattern (XD-002) garante entrega.

**Fonte**: **R11 §11 + R12 §7-§8 absorção Fase C** (formaliza como INVARIANTE — antes só ressalva no INS-013).

---

### INS-047 — Master Síndico Plataform (Adendo R29 markers expandidos)

**Prioridade**: M  
**EARS**: confirmação de cobertura — INS-040 (15 governance markers de síndico) e COM-002 (25 compliance markers de empresa) já cobrem o escopo de R29.

**Cobertura confirmada Fase C**: ABRACS, Seguro RC, ISOs (9001/14001/45001/37001/19600/37301), ESG, Selo Verde, Selo Carbon Free, Selo EuReciclo, Reclame Aqui, CIPA, NRs, certificação própria, premiações — todos presentes em INS-040 + COM-002.

**Adições Fase C** (se não já em INS-040):
- `iso_19600` (Compliance gerencial) — verificar; senão adicionar ao seed migration.
- `programa_compliance` — verificar.
- `selo_amiga_crianca` — verificar.

**Critério de aceite**:
- Disclaimer obrigatório (autodeclaração — não validação plataforma) já em INS-040 e COM-002.
- Sem mudança de código; apenas garantir seed migration cobre todos os 15 (síndico) + 25 (empresa) markers.

**Fonte**: **R29 absorção Fase C** (cobertura confirmada).

---

## Invariantes de institutional

1. Timeline INSERT-only — nunca DELETE, nunca UPDATE.
2. Master Plan status muda APENAS via timeline event (INV-INS-046 — DB-trigger enforced — Fase C).
3. Condomínio: hard limit 15 por conta de síndico.
4. Síndico no condomínio: 1 principal + 2 auxiliares + 3 assistentes.
5. Fração ideal: NUMERIC(5,4), Σ ≤ 1.0000 por Condominium (não por Block — D-128 ratifica).
6. Mandato: `mandate_started_at < mandate_ended_at` (D-129 flags em Membership; não aggregate).
7. Unidade: 1 titular + máx 2 dependentes.
8. Comunicado imutável pós-publish.
9. Adendo vinculado via `previous_entry_id` (cadeia profundidade ≤ 5 — INV-ADJ-006 D-131).
10. Conflito de interesse (INS-044): declaração anual obrigatória; deals com conflito flagged.
11. Mandate End/Transfer (INS-002, INS-045): bloqueio se `compliance_status ≠ em_dia`.
12. Dossier (INS-034): bloqueio compliance + assinatura digital.

---

## Pendências

- ⚠️ **PENDÊNCIA INS-017**: lista exata das 26 áreas do master plan — dual-check com stakeholder.
- ⚠️ **PENDÊNCIA INS-025**: lista exata dos 13 tipos de evento — dual-check.
- ⚠️ **PENDÊNCIA INS-040**: se certificações "outras" têm estrutura livre ou vocabulary controlado — dual-check Fase 3.
- ⚠️ **PENDÊNCIA INS-035**: se Fundo de Reserva integra com ERP externo ou só manual — ADR M3+.

---

## Referências

- Legado: `specs/requirements/institutional.md`, `cross-domain.md` Req 33-40, 46-47, `novo escopo-7.md` §2.
- [[../global]] GR-030 (timeline immutable), GR-033 (fração ideal).
- [[../../01-domain/bounded-contexts#institutional]]
- [[../registration-data]] — Síndico, Morador.
- [[../matrix-functional]] — Gestão Condominial.
- [[_moc]]
