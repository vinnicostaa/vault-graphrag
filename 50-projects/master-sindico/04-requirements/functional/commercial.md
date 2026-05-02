---
title: Functional Requirements — Commercial BC
type: requirement
tags:
  - requirements
  - functional
  - commercial
  - connect-me
  - reputation
  - talent-bank
  - neighborhood
  - master-sindico
  - v2
  - ears
source: >-
  legacy specs/requirements/commercial + cross-domain + contextos +
  _chaos/requirements(6).md (2026-04-13) + _chaos/mastersindico-requirements.pdf
  (2026-03-09)
created: 2026-04-23T00:00:00.000Z
updated: '2026-04-25'
fase_c_absorvido: true
---

# Functional Requirements — Commercial BC

> **Banner Fase C (2026-04-25)**: enriquecido com deltas R22/R27/R53 absorvidos do `_chaos/requirements(6).md` e `mastersindico-requirements.pdf`. Termos legados traduzidos para canônico vigente (D-126: 2 tenant kinds; D-131: Adjustment cross-domain). Telas (E1-E16, MK1-MK8, VS/VM/VE/VZ) → Fase D.

Requisitos funcionais do **bounded context `commercial`**. Responsabilidade: Empresas (cadastro + CNPJ), Connect Me (4 fluxos), propostas, deals, avaliações, reputação Bronze→Diamante, SupplierVoteSession, Banco de Talentos, Vizinhança (comércio local + cupons + seal síndico), harassment report.

Formato **EARS**. Prioridade: M (M1), S (M2), C (M3+), W (backlog).

> Destilado de `../../90-ingested/from-vault-50-projects-master-sindico/specs/requirements/commercial.md` + `cross-domain.md` + `novo escopo-7.md` §1-3.

---

## COM-001 — Criar Empresa (tenant_type=company)

**Prioridade**: M  
**EARS**: *Quando* empresa completa onboarding, *o sistema deve* criar tenant `tenant_type='company'` + `EnterpriseProfile {user_id, razao_social, nome_fantasia, cnpj, data_aniversario, representante_legal, email_comercial, telefone_comercial, endereco_comercial_cep, categoria_principal, subcategorias_atuacao (até 5)}`.

**Campos obrigatórios**: ver [[../registration-data]] Empresa.

**Critério de aceite**:
- CNPJ UNIQUE (algoritmo de dígito verificador validado; integração Receita stub M1, pleno M2+).
- Logo mín 200×200.
- `subcategorias_atuacao`: até 5, múltipla seleção controlada.
- Status: `pending_verification → active → suspended → blocked`.

**Fonte**: legado `commercial.md` Req 18, `institutional.md` Req 6.3, `DADOS PARA CADASTRO.md`.

---

## COM-002 — Compliance markers empresa (25 autodeclarados)

**Prioridade**: M  
**EARS**: *Quando* empresa edita compliance markers, *o sistema deve* aceitar lista de 25 checkboxes: `seguro_responsabilidade_civil, assessoria_juridica, assessoria_contabil, assessoria_seguranca_trabalho, responsavel_tecnico, regularidade_conselhos, conformidade_lgpd, conformidade_nr1, programa_compliance, iso_9001, iso_14001, iso_45001, iso_37001, iso_37301 (19600), esg, selo_verde_sustentavel, selo_amiga_crianca, selo_carbon_free, selo_eureciclo, selo_reclame_aqui, cipa, nrs, outras_certificacoes, certificacao_propria, premiacoes`.

**Critério de aceite**:
- Disclaimer fixo: _"Certificações são autodeclaradas, não certificadas por terceiro."_
- Sem validação externa M1.

**Fonte**: legado `institutional.md` Req 6.3, `DADOS PARA CADASTRO.md`.

---

## COM-003 — Empresa: regras de conteúdo público

**Prioridade**: M  
**EARS**: *Quando* empresa publica vídeo/texto público, *o sistema deve* rejeitar conteúdo com: chamadas comerciais óbvias, preços, dados de contato (telefone, WhatsApp). Moderação editorial (manual M1-M2).

**Critério de aceite**:
- Moderação pipeline (ver CNT-006 em [[content]]).
- Redirecionamento via Connect Me (não contato direto).

**Fonte**: legado `institutional.md` Req 6.3, `DADOS PARA CADASTRO.md` Termo 10.

---

## COM-004 — Visibilidade de empresa por plano

**Prioridade**: M  
**EARS**: *Enquanto* empresa tem `plan=base/trial`, *o sistema deve* exibir perfil apenas a Síndico Gestor/Plus/Pro. *Enquanto* Plus/Pro → perfil público a todos autenticados conforme matriz.

**Dependências**: [[../matrix-functional]].

**Critério de aceite**:
- Middleware ABAC valida.
- Ver matriz Perfil e Visibilidade.

**Fonte**: legado `commercial.md` Req 18.

---

## COM-005 — Validação de CNPJ (Receita Federal)

**Prioridade**: S (stub M1, pleno M2+)  
**EARS**: *Quando* empresa submete CNPJ, *o sistema deve* (M1) validar algoritmo dígito verificador; (M2+) chamar adapter Receita Federal para situação cadastral.

**Critério de aceite**:
- Interface `ICNPJValidator` com impl stub + impl Receita.
- Cache Redis TTL 30d.
- Selo "Verificada" quando situação=ativa.

**Fonte**: legado `commercial.md` Req 18, [[00-product/personas]].

---

## COM-006 — Connect Me: fluxo Síndico → Empresa

**Prioridade**: M  
**EARS**: *Quando* síndico emite `POST /connect-me/syndic-to-company` com `{category, description, orcamento_teto_cents, prazo, entregaveis, anexos[]}`, *o sistema deve* (1) validar quota (BIL-016), (2) criar ConnectMeRequest em status `sent`, (3) notificar empresas qualificadas na categoria.

**Critério de aceite**:
- Formulário estruturado (não texto livre).
- Dados do síndico ocultos até empresa clicar "Tenho Interesse".
- Log LGPD obrigatório (GR-026, cross-domain Req 39).

**Fonte**: legado `commercial.md` Req 19, [[STEERING]] §38.

---

## COM-007 — Connect Me: empresa manifesta interesse

**Prioridade**: M  
**EARS**: *Quando* empresa clica "Tenho Interesse", *o sistema deve* (1) revelar dados do síndico à empresa, (2) gravar `lgpd_logs {timestamp, company_id, syndic_id, ip, terms_version, purpose}`.

**Critério de aceite**:
- Reveal 1x por request (idempotência).
- Log append-only.

**Fonte**: legado `commercial.md` Req 19, cross-domain Req 39.

---

## COM-008 — Connect Me: auto-close 48h sem interesse

**Prioridade**: S  
**EARS**: *Quando* ConnectMeRequest fica 48h sem empresa interessada → `status = expired`, notificação ao síndico.

**Critério de aceite**:
- Job cron a cada 1h.
- Quota NÃO é devolvida (síndico escolheu gastar).

**Fonte**: legado `commercial.md` Req 19.

---

## COM-009 — Connect Me: Morador → Síndico (frio, sem reply)

**Prioridade**: M  
**EARS**: *Quando* morador emite `POST /connect-me/resident-to-syndic` com `{category, urgency, message}`, *o sistema deve* validar quota (2/ano Base, 4/ano Pagante), criar `connect_me_resident_requests`, notificar síndico. **Sem reply, sem histórico, sem thread.**

**Categorias**: `duvida_gestao, solicitacao_manutencao, reclamacao, sugestao, outro`.  
**Urgência**: `baixa, media, alta, urgente`.

**Critério de aceite**:
- Síndico responde via comunicado público (INS-022) ou ignora.
- Morador não vê histórico anterior.

**Fonte**: legado `commercial.md` Req 19.1.

---

## COM-010 — Connect Me: Empresa → Empresa (Plus/Pro only)

**Prioridade**: S  
**EARS**: *Quando* empresa Plus/Pro emite `POST /connect-me/company-to-company`, *o sistema deve* validar plan_tier, aceitar `partnership_type ∈ {subcontratacao, parceria_tecnica, fornecimento_materiais, indicacao_servicos, outro}`.

**Critério de aceite**:
- Quota: Plus limitado, Pro ilimitado.
- Dados ocultos até "Tenho Interesse".

**Fonte**: legado `commercial.md` Req 19.2.

---

## COM-011 — Connect Me: Síndico → Parceiro Vizinhança

**Prioridade**: S  
**EARS**: *Quando* síndico emite `POST /connect-me/syndic-to-partner`, *o sistema deve* aceitar `promotion_type ∈ {desconto_exclusivo, promocao_dia, campanha_sazonal, parceria_continua, outro}`. Parceiro responde com aceite ou contra-proposta.

**Critério de aceite**:
- Visibilidade: apenas moradores do condomínio.
- Registra em `neighborhood_benefits`.

**Fonte**: legado `commercial.md` Req 19.3.

---

## COM-012 — Connect Me: limitação anti-prospecção

**Prioridade**: M  
**EARS**: *Enquanto* usuário tem `role=marketing`, *o sistema deve* bloquear acesso ao botão "Connect Me" (anti-spam). *Quando* empresa tenta Connect Me pra morador → bloqueia (contato parte do contratante).

**Critério de aceite**:
- ABAC regra explícita.
- UI não renderiza botão.

**Fonte**: legado `novo escopo-7.md` §1.6 "Anti-Prospecção".

---

## COM-013 — Connect Me: botão "Denunciar" (harassment report)

**Prioridade**: S  
**EARS**: *Quando* usuário clica "Denunciar" em Connect Me, *o sistema deve* criar `HarassmentReport {reporter_id, reported_id, connect_me_id, reason, description, created_at}`, notificar admin. Admin pode aplicar: warning, suspension temporária, ban permanente.

**Critério de aceite**:
- Audit trail completo.
- Contexto: texto do Connect Me preservado como evidência.
- Impacto reputação empresa (suspende tier — GR-028).

**Fonte**: legado `commercial.md` Req 19 notas, matriz funcional observação.

---

## COM-014 — Rascunho de Connect Me (Redis auto-save)

**Prioridade**: S  
**EARS**: *Enquanto* usuário preenche Connect Me, *o sistema deve* auto-salvar em Redis `ms:connect_me_draft:{user_id}` com TTL 24h.

**Critério de aceite**:
- Debounce 500ms.
- Limpa ao publicar.

**Fonte**: legado `commercial.md` Req 19 "SHOULD".

---

## COM-015 — Proposta: status e imutabilidade

**Prioridade**: M  
**EARS**: *Quando* proposta é criada, *o sistema deve* iniciar em `status=draft`. *Quando* envio (`POST /proposals/{id}/send`) → `status=sent`, **imutável**. Status sucessivos: `awaiting_validation, validated, accepted, rejected`.

**Critério de aceite**:
- Draft editável livremente.
- Sent → lock (apenas adendo — INS-016).
- Validade de data limite (ex: 30 dias).

**Fonte**: legado `commercial.md` Req 20.

---

## COM-016 — Proposta: versionamento (síndico rejeita)

**Prioridade**: S  
**EARS**: *Quando* síndico rejeita proposta, *o sistema deve* permitir à empresa criar nova versão (v2) vinculada à original.

**Critério de aceite**:
- Tabela `proposal_versions` com `previous_proposal_id`.

**Fonte**: legado `commercial.md` Req 20.

---

## COM-017 — Proposta: assinatura digital

**Prioridade**: C (M3+)  
**EARS**: *Quando* proposta é enviada, *o sistema deve* gerar timestamp + assinatura digital (MP 2.200-2 / Lei 14.063 compatível).

**Fonte**: legado `commercial.md` Req 20.

---

## COM-018 — Votação de fornecedor (SupplierVoteSession)

**Prioridade**: S (M2+)  
**EARS**: *Quando* síndico abre votação de fornecedor durante assembleia, *o sistema deve* exigir mínimo 2 propostas validadas, suportar tipos de quorum (`simple_majority > 50%`, `qualified_majority ≥ 2/3`, `unanimous = 100%`), usar fração ideal como peso, publicar resultado real-time via WebSocket.

**Dependências**: ASM-021, GR-033.

**Critério de aceite**:
- Vote único por (session, voter) — UNIQUE DB (A-025/A-026 legado).
- Resultado imutável.

**Fonte**: legado `commercial.md` Req 21, `assembly.md` Req 57.

---

## COM-019 — Negócio Fechado (Deal) — dupla assinatura termo_empresa + termo_sindico

**Prioridade**: S  
**EARS**: *Quando* proposta é aceita, *o sistema deve* criar `Deal {id, proposal_id, company_id, condominium_id, syndic_id, value_cents, service_description, agreed_cost_cents, start_date, end_date, payment_terms, status='draft', created_at}`. Fluxo de dupla assinatura (R22 absorção Fase C):
1. Síndico (ou empresa) cria deal a partir de proposta aceita → `status = draft`.
2. Sistema gera **`termo_empresa`** (institutional text + escopo + custo) → empresa assina → `status = awaiting_sindico_signature`.
3. Sistema gera **`termo_sindico`** (institutional text + responsabilidade do contratante) → síndico assina → `status = confirmed`.
4. Após ambas assinaturas → **imutável** (lock); `confirmed_at = now()`.
5. Auto-publica entrada Timeline tipo `registro_de_execucao` (síndico) + entrada Timeline tipo `closed_deal` (institucional).
6. Reputação recalculada (COM-025).

**Critério de aceite**:
- Status enum: `draft, awaiting_company_signature, awaiting_sindico_signature, confirmed, cancelled`.
- Dupla assinatura obrigatória — empresa assina **primeiro**, síndico **depois** (R22 §4 ordem fixa: "company signs first, then sindico signs").
- `termo_empresa` e `termo_sindico` versionados (GR-023) — texto institucional canônico imutável.
- Anexos opcionais: `contract_documents, technical_specifications, insurance_certificates`.
- Após confirmed → edição **só via adendo** (D-131; INV-ADJ-006 profundidade ≤ 5).
- Auto-gen número único `DL-{YYYYMMDD}-{SEQ}` (ex: `DL-20260508-001`).
- Evento NATS `deal.signed_complete` → timeline + reputation recalc + `audit_trail` + `lgpd_logs`.
- Cancelamento permitido só pré-confirmed (status ∈ {draft, awaiting_*}).

**Texto canônico do termo de execução** (E8 Fase D — versionado, mas canônico spec):

> "Declaro executar o serviço conforme escopo validado, observando normas técnicas aplicáveis, legislação vigente e regras de segurança do trabalho, assumindo responsabilidade por vícios de execução, falhas técnicas e danos comprovadamente decorrentes da minha atuação. Declaro possuir capacidade técnica, operacional e regularidade jurídica compatível com o objeto contratado."

**Fonte**: legado `commercial.md` Req 22; **R22 absorção Fase C** (termo_empresa + termo_sindico + status enum 5); D-131 (Adjustment cross-domain).

---

## COM-020 — Registro de Execução

**Prioridade**: S  
**EARS**: *Quando* empresa registra execução (`POST /execution-records`), *o sistema deve* aceitar `{fotos, descricao, data_inicio, data_fim, custos_cents, metricas}`. Notificar síndico pra validação.

**Critério de aceite**:
- Upload de fotos (GR-018) e vídeos via Mux (GR-019).
- Status: `pending, approved, rejected`.

**Fonte**: legado `commercial.md` Req 23.

---

## COM-021 — Validação obrigatória de execução

**Prioridade**: S  
**EARS**: *Quando* síndico recebe execução, *o sistema deve* exigir validação (aceitar ou rejeitar com motivo). *Enquanto* pendente → bloqueia pagamento (M3+).

**Critério de aceite**:
- Rejeição: motivo obrigatório.
- Aprovação → Timeline + master_plan update (INS-013).
- Auditoria completa.

**Fonte**: legado `commercial.md` Req 23.

---

## COM-022 — Atividade técnica (contribuição preventiva)

**Prioridade**: S  
**EARS**: *Quando* empresa contribui com atividade técnica (consultoria, auditoria, treinamento) sem deal, *o sistema deve* permitir registro similar a Req 23, sem fluxo de pagamento.

**Critério de aceite**:
- Tabela `technical_activities`.
- Marca master_plan como atividade executada.

**Fonte**: legado `commercial.md` Req 23.1.

---

## COM-023 — Comunicado pós-deal (5 tipos)

**Prioridade**: C (M3+)  
**EARS**: *Quando* deal entra em status de execução, *o sistema deve* permitir empresa publicar comunicado tipo: `execucao_iniciada, execucao_pausada, execucao_retomada, execucao_concluida, atraso_informado`. Exige validação síndico.

**Fonte**: legado `commercial.md` Req 24.

---

## COM-024 — Avaliação de empresa (5 critérios)

**Prioridade**: S  
**EARS**: *Quando* deal termina, *o sistema deve* acionar avaliação do síndico sobre a empresa com 5 critérios escala 1-10: `cumprimento_prazos, qualidade_trabalho, comunicacao, profissionalismo, adequacao_custos`. **Anônima** pra empresa; pública a outros síndicos (média).

**Critério de aceite**:
- Tabela `company_assessments`.
- Pontuação alimenta reputação (COM-025).

**Fonte**: legado `commercial.md` Req 26.

---

## COM-025 — Reputação Bronze → Diamante (determinística)

**Prioridade**: S  
**EARS**: *Quando* evento afeta reputação (`deal.signed_complete`, `assessment.submitted`, `video.approved`, `harassment.confirmed`), *o sistema deve* chamar `RecalculateReputation(company_id)` (função pura). Tiers canônicos:

| Tier | Requisito acumulado |
|---|---|
| Bronze | Default ao criar empresa (cadastro + CNPJ verificado) |
| Prata | ≥ 3 contratos + avaliação média ≥ 3/5 |
| Ouro | ≥ 10 contratos + ≥ 3.5/5 + ≥ 3 condomínios distintos |
| Platina | ≥ 30 contratos + ≥ 4/5 + ≥ 10 condomínios + ≥ 1 vídeo institucional aprovado + ≥ 1 case em vídeo |
| Diamante | ≥ 100 contratos + ≥ 4.5/5 + ≥ 30 condomínios + ≥ 3 vídeos + múltiplos cases + tempo plataforma ≥ 12 meses |

**Dependências**: GR-028.

**Critério de aceite**:
- Função pura (PBT: inputs → mesmo tier).
- Operador Master Síndico NÃO pode alterar tier diretamente.
- Degradação permitida (contratos cancelados, avaliações pioraram).

**Fonte**: legado `institutional.md` Req 6.2, [[00-product/portfolio-de-produtos]] Produto 3, [[STATE]] D-010.

---

## COM-026 — Reputação: suspensão temporária por harassment

**Prioridade**: S  
**EARS**: *Quando* harassment report é confirmado, *o sistema deve* marcar `companies.suspended_until = now() + 72h` (cooldown). NÃO despromove tier.

**Critério de aceite**:
- Campo `suspended_until`.
- UI mostra "Suspensa temporariamente".

**Fonte**: legado [[00-product/portfolio-de-produtos]] Produto 3.

---

## COM-027 — Reputação: visibilidade do breakdown

**Prioridade**: S  
**EARS**: *Quando* usuário clica no tier da empresa, *o sistema deve* retornar breakdown das métricas que o geraram (contratos, avaliações, condomínios distintos, vídeos, tempo de plataforma).

**Critério de aceite**:
- Endpoint `GET /companies/{id}/reputation-breakdown`.
- Zero black box.

**Fonte**: [[00-product/portfolio-de-produtos]] Produto 3 diferencial.

---

## COM-028 — Banco de Talentos: empresa busca currículos

**Prioridade**: S (M3)  
**EARS**: *Quando* empresa Plus/Pro/Marketing/Vizinhança emite `GET /talents?categoria&regiao&palavra_chave`, *o sistema deve* listar vídeo-currículos ativos de moradores pagantes.

**Dependências**: ABAC: Plus/Pro/Marketing/Vizinhança têm acesso (matriz funcional).

**Critério de aceite**:
- Filtros: área de atuação, região (CEP), palavra-chave.
- Player do vídeo-currículo.
- "Demonstrar Interesse" → dispara notificação morador (sujeito a GR-002 regras de notification).

**Fonte**: legado `novo escopo-7.md` §4.7, matriz funcional "Visualizar currículos".

---

## COM-029 — Banco de Talentos: morador publica vídeo-currículo

**Prioridade**: S (M3)  
**EARS**: *Quando* morador pagante publica vídeo-currículo (`POST /videos?type=resume`), *o sistema deve* validar: ≤ 90 segundos, lock 90d, via Mux presigned, fixo no perfil (não galeria).

**Dependências**: BIL-019, GR-029, CNT-001.

**Critério de aceite**:
- Validação duration pós-transcoding Mux.
- Se existe vídeo-currículo < 90d → 409.

**Fonte**: legado `novo escopo-7.md` §1.4, `content.md` Req 29.

---

## COM-030 — Vizinhança (Parceiro): perfil público

**Prioridade**: S  
**EARS**: *Quando* parceiro ativo, *o sistema deve* expor perfil público: logo, nome, endereço, fotos (≤3), promoções ativas.

**Critério de aceite**:
- Visibilidade: moradores em raio (geo).
- Selo "Síndico-aprovado" renderizado quando concedido.

**Fonte**: legado `commercial.md` Req 27.3, `DADOS PARA CADASTRO.md` Parceiros.

---

## COM-031 — Vizinhança: 7 tipos de benefício

**Prioridade**: S  
**EARS**: *Quando* parceiro cria benefício, *o sistema deve* aceitar `benefit_type ∈ {desconto_percentual, desconto_fixo, produto_gratuito, combo_promocional, avaliacao_gratuita, brinde, beneficio_exclusivo}`.

**Critério de aceite**:
- Tabela `neighborhood_benefits`.

**Fonte**: legado `commercial.md` Req 27.3.

---

## COM-032 — Vizinhança: escopo de promoção

**Prioridade**: S  
**EARS**: *Quando* parceiro cria promoção, *o sistema deve* aceitar `scope ∈ {aberta_bairro, exclusiva_condominio}`. Exclusiva exige código de inscrição do condomínio (negociado com síndico).

**Critério de aceite**:
- Validação: exclusiva requer `condominium_id`.
- Busca filtra por CEP do morador.

**Fonte**: legado `commercial.md` Req 27.3, `novo escopo-7.md` §3.1.

---

## COM-033 — Vizinhança: cupom padrão ID_LOJA+SEQ+PALAVRA

**Prioridade**: S  
**EARS**: *Quando* morador clica "Pegar Cupom", *o sistema deve* (1) validar cooldown 4h/parceiro (BIL-020), (2) gerar código `{ID_LOJA(5)}{SEQ(3)}{PALAVRA(5)}`, exibir na tela.

**Exemplo**: `00123055PAO`.

**Critério de aceite**:
- ID_LOJA: 5 dígitos (ID único plataforma).
- SEQ: 3 dígitos (sequencial do cupom).
- PALAVRA: 5 letras maiúsculas (palavra-chave do dia do parceiro).
- Tabela `coupon_generations`.

**Fonte**: legado `novo escopo-7.md` §3.1.

---

## COM-034 — Vizinhança: "Palavra do Dia" (parceiro)

**Prioridade**: S  
**EARS**: *Quando* parceiro define "palavra do dia" (até 5 letras maiúsculas), *o sistema deve* persistir em `partners.palavra_do_dia` + timestamp. Validade diária.

**Critério de aceite**:
- Validação: máx 5 chars, A-Z only.
- Reset 00:00 SP.

**Fonte**: legado `novo escopo-7.md` §3.1.

---

## COM-035 — Vizinhança: seal "Síndico-aprovado" (via votação)

**Prioridade**: C (M3+)  
**EARS**: *Quando* síndicos com `plan_tier ∈ {plus, pro}` (D-067/D-081/D-096) votam em seal pra parceiro → threshold (ex: 3 síndicos de condomínios distintos na vizinhança), *o sistema deve* marcar `neighborhood_partners.seal_syndic_approved = true`.

**Critério de aceite**:
- Votação leve (não é SupplierVoteSession formal).
- Rastreável via audit.

**Fonte**: [[00-product/portfolio-de-produtos]] Produto 7.

---

## COM-036 — Vizinhança: métricas de cupom (views, cliques, resgates)

**Prioridade**: S  
**EARS**: *Quando* parceiro consulta dashboard, *o sistema deve* exibir views, cliques, cupons gerados, resgates (se rastreável via QR).

**Critério de aceite**:
- Métricas privadas ao parceiro + admin (privacidade — `novo escopo-7.md` §4.2).

**Fonte**: legado `commercial.md` Req 27.3, `novo escopo-7.md` §3.1.

---

## COM-037 — Vizinhança: morador — feed por bairro

**Prioridade**: S  
**EARS**: *Quando* morador pagante acessa aba "Clube de Benefícios/Vizinhança", *o sistema deve* listar parceiros e cupons filtrados por CEP.

**Dependências**: matriz funcional — Base não tem acesso.

**Critério de aceite**:
- Filtro por categoria.
- Badge "Indicado pelo Síndico" / "Síndico-aprovado" quando aplicável.

**Fonte**: legado `commercial.md` Req 27.2, `novo escopo-7.md` §3.1.

---

## COM-038 — Síndico: dashboard de vizinhança (VS1–VS10)

**Prioridade**: S  
**EARS**: *Quando* síndico com `plan_tier ∈ {plus, pro}` (D-067/D-081/D-096) acessa Vizinhança, *o sistema deve* expor: dashboard de parcerias ativas, explorar/buscar parceiros, indicar pra comunidade, criar benefício exclusivo, histórico.

**Critério de aceite**:
- Endpoints CRUD + métricas.

**Fonte**: legado `commercial.md` Req 27.

---

## COM-039 — Empresa: dashboard comercial (CTR, avaliações, receita)

**Prioridade**: S  
**EARS**: *Quando* empresa acessa dashboard, *o sistema deve* exibir: condomínios conectados, CTR Connect Me, taxa aprovação execução, avaliação média, receita estimada (M3+).

**Critério de aceite**:
- Dashboard privado (visibilidade só do dono — `novo escopo-7.md` §4.2).
- Segmentação por período.

**Fonte**: legado `cross-domain.md` Req 44.

---

## COM-040 — Agência de Marketing: dashboard delegado

**Prioridade**: C (M3+)  
**EARS**: *Quando* agência de marketing acessa painel, *o sistema deve* exibir métricas agregadas das empresas que gerencia (engajamento cruzado, views médios, retention rate, top performers).

**Dependências**: IDN-025.

**Critério de aceite**:
- Scope limitado (só empresas delegadas).
- Nunca vê dados de negócio.

**Fonte**: legado `cross-domain.md` Req 45, `content.md` Req 30.

---

## COM-041 — Empresa: marketing link (agência publica link sponsored)

**Prioridade**: C  
**EARS**: *Quando* agência emite "Marketing Link" para empresa que gerencia, *o sistema deve* publicar link destacado em feed/search direcionando a perfis gerenciados.

**Critério de aceite**:
- Tabela `marketing_links`.
- Audit trail.

**Fonte**: legado matriz funcional "Marketing Link".

---

## COM-042 — Validação de busca de empresa por categoria + localização

**Prioridade**: M  
**EARS**: *Quando* síndico busca empresa, *o sistema deve* expor `GET /companies/search?category=&cep=&rating_min=` com filtros combinados, ordenação por rating e proximidade.

**Dependências**: OpenSearch (CNT-005).

**Critério de aceite**:
- Geo-proximidade (raio configurável).
- Facetas (categorias, tiers).

**Fonte**: legado `content.md` Req 31, `commercial.md` Req 27.

---

## COM-043 — Adendo: único mecanismo de modificação

**Prioridade**: M  
**EARS**: *Quando* modificação de registro imutável é necessária (proposta, deal, execution_record), *o sistema deve* exigir criação de `amendments {id, record_type, record_id, previous_record_id, reason, signatures[]}`. Ambas partes (empresa + síndico) assinam.

**Dependências**: INS-016.

**Critério de aceite**:
- Chain via `previous_record_id`.
- Assinatura dupla obrigatória.
- Evento NATS `amendment.signed` → update timeline.

**Fonte**: legado `commercial.md` Req 25.

---

## COM-044 — Empresa-síndica (feature flag, M5+)

**Prioridade**: W  
**EARS**: *Quando* empresa se cadastra como empresa-síndica, deverá gerenciar múltiplos condomínios de síndicos profissionais associados. M5+.

**Fonte**: [[00-product/portfolio-de-produtos]] anti-catálogo.

---

## COM-045 — Usuários secundários de empresa (1 representante ativo em M1)

**Prioridade**: C (M3)  
**EARS**: *Quando* representante legal quer adicionar operador, *o sistema deve* gerar invite com role_type=operator (sem gestão, só operacional).

**Dependências**: IDN-015.

**Critério de aceite**:
- Limite: 1 representante + N operadores (configurável, default 3).
- M1 só representante legal.

**Fonte**: legado `institutional.md` Req 10, [[00-product/personas]].

---

## Fase C — Deltas absorvidos 2026-04-25

### COM-046 — Supplier Analyzer (Fornecedor Master vs Externo)

**Prioridade**: S (M2 — pré-requisito de votação fornecedor em assembleia ASM-021)  
**EARS**: *Quando* síndico cria item de pauta `votacao_fornecedor` ou consulta empresa pré-deal, *o sistema deve* expor `GET /companies/{id}/analysis-card`. *Quando* `id` corresponde a empresa cadastrada na plataforma → renderizar **Fornecedor Master**: `selo_verificado, video_institucional, full_profile, evaluations_history (média + breakdown), reputation_tier, compliance_markers, conflict_declarations? (se síndico tem)`. *Quando* `id` é empresa **externa** (CNPJ informado mas não cadastrado) → renderizar **Fornecedor Externo** com badge **"Empresa não verificada pela Master Síndico"** + apenas `dados_basicos` informados manualmente. *Quando* síndico anexa Supplier Analyzer a item de pauta → grava `agenda_item_attachments` + emite `SupplierAnalysisCreated` event.

**Critério de aceite**:
- Endpoint `GET /companies/{id}/analysis-card` ou `GET /external-suppliers/{cnpj}/basic`.
- Badge **"Empresa não verificada pela Master Síndico"** obrigatório no UI quando externa (frontend recebe flag `is_master_verified: bool`).
- Comparação cross-supplier: matriz lado-a-lado quando 2+ propostas (R53 §5).
- Audit: `SupplierAnalysisCreated` event + `lgpd_logs` (acesso a dados pessoais quando fornecedor externo cadastrado manualmente com CPF de representante).
- Vínculo opcional a `assembly_agenda_items.supplier_analysis_id`.

**Dependências**: COM-018 (SupplierVoteSession), ASM-021, COM-025 (reputation).

**Fonte**: **R53 absorção Fase C**.

---

### COM-047 — Vizinhança taxonomia (38 categorias em 12 grupos)

**Prioridade**: S  
**EARS**: *Quando* parceiro Vizinhança completa onboarding, *o sistema deve* aceitar `category` em enum 38 itens organizados em 12 grupos (R-_chaos absorção Fase C). *Quando* morador busca em `/neighborhood?category=...&cep=...` → filtrar por categoria + raio geo (default 5km).

**Taxonomia canônica (38 categorias × 12 grupos)** — ver arquivo dedicado `01-domain/taxonomies/service-categories` *(a criar quando enums consolidados)* (a criar Fase C):

| Grupo | Itens |
|---|---|
| Alimentação | padaria, restaurante, lanchonete, cafeteria, pizzaria, hamburgueria, sorveteria, doceria, acai (9) |
| Mercado | supermercado, hortifruti, adega (3) |
| Saúde | farmacia (1) |
| Pet | pet_shop, clinica_veterinaria, banho_tosa (3) |
| Academia | academia, crossfit, pilates, yoga (4) |
| Estética | salao_beleza, barbearia, spa (3) |
| Serviços Domésticos | lavanderia, costureira, sapateiro (3) |
| Profissionais | eletricista, encanador, chaveiro, marceneiro, pintor (5) |
| Assistência Técnica | assistencia_celular, assistencia_computadores (2) |
| Automotivo | oficina, lava_jato (2) |
| Cursos | cursos_idiomas, reforco_escolar (2) |
| Outros | outros_servicos (1) |

**Total**: 38 categorias.

**Critério de aceite**:
- Enum Postgres `partner_category` (38 itens) — seed migration.
- Tabela `01-domain/taxonomies/service-categories.md` (a criar) documenta grupos + descrições pt-BR.
- Filtro por grupo (UI agrupa categorias).
- `outros_servicos` aceito como fallback.

**Dependências**: COM-030, COM-037 (feed Vizinhança).

**Fonte**: **R27 + R-_chaos Partner Categories absorção Fase C**.

---

### COM-048 — Service Execution Registry (campos canônicos R23)

**Prioridade**: S (M2)  
**EARS**: *Quando* empresa registra execução (`POST /execution-records` — COM-020 ratificado), *o sistema deve* aceitar campos canônicos R23:

```
{
  condominio_atendido (deal_id ref),
  tipo_servico_executado,
  descricao_execucao,
  area_impactada (enum INS-017 — 26 áreas),
  natureza_atividade (enum: preventiva, corretiva, emergencial, estrategica),
  status_servico (enum: concluido, em_andamento, parcialmente_concluido),
  data_execucao,
  garantia_servico (period),
  orientacoes_tecnicas (text),
  materiais_utilizados (optional, JSON),
  classificacao_risco (enum: baixo, moderado, elevado, critico),
  photos[], videos[]
}
```

**Status flow**: `draft → sent → awaiting_validation → validated | rejected → published (auto when validated)`.

**Critério de aceite**:
- Validação síndico obrigatória (COM-021).
- Apenas após `validated` → integra Timeline tipo `registro_de_execucao` (auto INS-013).
- Vinculado a `closed_deal_id` (FK).
- Display em "Validações Pendentes" do síndico.

**Fonte**: legado COM-020 (cobre minimamente); **R23 absorção Fase C** (campos canônicos completos + classificação de risco + natureza atividade).

---

### COM-049 — Curriculum: 18 áreas de atuação + 9 modalidades de contrato

**Prioridade**: S (M3 — vinculado Banco de Talentos COM-029)  
**EARS**: *Quando* morador pagante cadastra vídeo-currículo (COM-029), *o sistema deve* aceitar metadados estruturados:
- `work_areas[]` — array de 1+ itens em enum 18 áreas (R-_chaos Curriculum Work Areas).
- `contract_modalities[]` — array de 1+ itens em enum 9 modalidades.
- `cnh ∈ {nao, A, B, AB}`.

**Áreas de atuação canônicas (18 itens — R-_chaos absorção Fase C)**:

```
operacoes_condominiais, limpeza_conservacao_higienizacao, manutencao_predial,
seguranca_patrimonial, obras_reformas_adequacoes, administracao_apoio_administrativo,
atendimento_relacionamento_suporte, logistica_almoxarifado_suprimentos,
gestao_coordenacao_lideranca, treinamento_qualidade_processos_compliance,
tecnologia_sistemas_suporte_digital, comercial_vendas_parcerias,
comunicacao_marketing_conteudo, financeiro_contabil_controladoria,
juridico_contratos_compliance_legal, recursos_humanos_gestao_pessoas,
apoio_operacional_externo, outras_areas
```

**Modalidades de contrato (9 itens)**:

```
clt, pj, estagio, temporario, folguista, diarista, freelancer, intermitente, outros
```

**Critério de aceite**:
- Enum Postgres `curriculum_work_area` (18) e `contract_modality` (9).
- Busca em Banco de Talentos (COM-028) filtra por `work_areas` (qualquer item match) + `contract_modalities`.
- `cnh` opcional, mas se presente → filtro disponível.

**Dependências**: COM-028, COM-029.

**Fonte**: **R-_chaos Curriculum Work Areas + Contract Modalities absorção Fase C**.

---

### COM-050 — Company Evaluation: 5 critérios canônicos R26

**Prioridade**: S  
**EARS**: cobertura ratificada — COM-024 já cobre 5 critérios escala 1-10. Texto canônico Fase C:

| Critério | Texto pt-BR | Escala |
|---|---|---|
| `qualidade_servico` | "Qualidade técnica do serviço prestado" | 1-10 |
| `cumprimento_prazo` | "Cumprimento do prazo acordado" | 1-10 |
| `atendimento` | "Qualidade do atendimento e comunicação" | 1-10 |
| `custo_beneficio` | "Adequação custo-benefício" | 1-10 |
| `recomendacao` | "Você recomendaria essa empresa a outro síndico?" | 1-10 |

**Adicional R-_chaos**: empresa pode "Manter privada" OU "Resposta registrada e imutável" (E11 fluxo).

**Fonte**: COM-024 + **R26 absorção Fase C** (texto canônico).

---

## Invariantes de commercial

1. Connect Me é unidirecional — nunca thread/reply.
2. Propostas imutáveis após `sent` — único mecanismo de mudança: adendo.
3. Deal imutável após dupla assinatura.
4. Execução de serviço exige validação obrigatória do síndico.
5. Avaliações são anônimas ao avaliado.
6. Reputação é função pura (GR-028).
7. Fração ideal em supplier vote usa NUMERIC.
8. Vote único por (session, voter) — UNIQUE DB (A-025 legado).
9. Agência tem acesso ZERO a dados de negócio (IDN-025).
10. Cupom: cooldown 4h por (morador, parceiro).

---

## Pendências

- ⚠️ **PENDÊNCIA COM-005**: qual adapter Receita Federal (stub M1 + quem provê em M2+) — ADR.
- ⚠️ **PENDÊNCIA COM-025**: thresholds numéricos finais Bronze→Diamante — dual-check stakeholder (D-010).
- ⚠️ **PENDÊNCIA COM-033**: geração de SEQ global vs por loja — ADR.
- ⚠️ **PENDÊNCIA COM-035**: threshold de votos síndicos pra seal — dual-check.
- ⚠️ **PENDÊNCIA COM-017**: provedor de assinatura digital BR — ADR M3+.

---

## Referências

- Legado: `specs/requirements/commercial.md`, `cross-domain.md` Req 43-45, 102-103, `novo escopo-7.md` §1.6, §3, `institutional.md` Req 6.3.
- [[../global]] GR-028 (reputação), GR-016 (pagination), GR-029 (lock 90d).
- [[../../01-domain/bounded-contexts#commercial]]
- [[../../00-product/portfolio-de-produtos]] Produtos 2, 3, 6b, 7.
- [[../registration-data]] — Empresa, Parceiro.
- [[../matrix-functional]] — Connect Me, Comércio Local, Currículos.
- [[_moc]]
