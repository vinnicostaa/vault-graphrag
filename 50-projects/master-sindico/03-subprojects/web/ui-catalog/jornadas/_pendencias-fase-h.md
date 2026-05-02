---
title: Pendências detectadas na Fase D — para Fase H avaliar
type: note
tags:
  - master-sindico
  - ui-catalog
  - jornadas
  - pendencias
  - fase-d
  - fase-h
created: 2026-04-25T00:00:00.000Z
status: open
---

# Pendências detectadas na Fase D (jornadas)

> Lista consolidada de **divergências, ambiguidades e itens novos** detectados ao destilar 9 PDFs do cliente em specs de tela. Cada item indica origem (PDF + tela), tipo, gravidade e proposta de resolução. **Fase H** avalia, decide e atualiza canônicos (functional/, invariants, aggregates).

## A. Divergências numéricas (PDF vs catálogo macro / vault)

### A.1 — Limite de condomínios por síndico (S2)

- **PDF Síndico** (linha 56): "Síndico pode consultar até **12** condomínios"
- **Catálogo macro `ui-catalog.md`**: limite **15**
- **Tipo**: divergência numérica
- **Gravidade**: média (impacta plan-tier `pro`)
- **Resolução proposta**: confirmar com cliente. Provável 15 (mais recente). Atualizar PDF/contrato se necessário.

### A.2 — Duração do vídeo Banco de Talentos (M-CV-2 / TEL2)

- **PDF Currículo** (linha 21): "Apresente-se em até **60 segundos**"
- **Catálogo macro `ui-catalog.md`**: TEL2 = **90s**
- **Tipo**: divergência
- **Gravidade**: baixa (UX)
- **Resolução proposta**: PDF é mais recente — adotar **60s**. Atualizar macro + Reqs COM-049.

### A.3 — Vínculos no PDF export (E-CV-3)

- **PDF Empresa-Currículo** (linha 27): "Experiência (últimos **3** vínculos)" no PDF gerado
- **D-099 + PDF Currículo morador**: morador cadastra **5** vínculos
- **Tipo**: divergência intencional? (export pode ser truncado em 3 mesmo guardando 5)
- **Gravidade**: baixa
- **Resolução proposta**: export truncado em 3 ok, mas dado completo permanece com 5. Confirmar.

### A.4 — Indicadores Dashboard Empresa (E10)

- **PDF Empresa**: ~8 indicadores listados
- **Catálogo macro**: 13 indicadores
- **Tipo**: complementação
- **Gravidade**: baixa
- **Resolução proposta**: macro engloba PDF; manter 13 (super-set).

## B. Ambiguidades / valores não especificados

### B.1 — Idade mínima dependente (M14)

- **PDF Morador**: não especifica idade mínima
- **Vault macro**: menciona "≥ 13 anos" como política a confirmar
- **Tipo**: gap
- **Gravidade**: alta (regulatório / LGPD criança)
- **Resolução proposta**: definir política formal (sugestão: ≥ 16 anos para acesso direto; 13-15 com consentimento titular).

### B.2 — Idade do morador no cadastro (M-CV-5)

- **PDF Currículo**: campo "Idade" como integer (não data nascimento)
- **Tipo**: data integrity
- **Gravidade**: média
- **Resolução proposta**: capturar data de nascimento (M2 onboarding já tem) e calcular idade derivada. Não duplicar.

### B.3 — Cooldown solicitação de contato (E-CV-6)

- **PDF Empresa-Currículo**: não especifica janela cooldown entre tentativas de contato com mesmo candidato
- **Tipo**: gap UX/abuse
- **Gravidade**: média (anti-spam)
- **Resolução proposta**: definir cooldown (sugestão: 30 dias por candidato + máx 5 candidatos/dia/empresa).

### B.4 — Subtema do vídeo agência (MK4)

- **PDF Marketing**: "Subtema do vídeo (preenchido automaticamente com base no segmento da empresa)"
- **Tipo**: gap — falta mapeamento canônico segmento → subtemas
- **Gravidade**: média
- **Resolução proposta**: criar tabela de mapeamento (enum `01-domain/enums/segmento-subtemas-video.md`).

### B.5 — Resposta única a perguntas (M7)

- **PDF Morador**: "Enviar resposta" sem especificar se permite mudar depois
- **Tipo**: ambiguidade
- **Gravidade**: média (dados estatísticos)
- **Resolução proposta**: resposta única (final) — alinhado com avaliação de gestão M12 (também única por ciclo).

### B.6 — Termos longos S29/S30

- **PDF Síndico**: textos institucionais completos (Termo de Responsabilidade, Declaração Anual)
- **Tipo**: completeness
- **Gravidade**: alta (jurídico)
- **Resolução proposta**: preservar texto íntegro em `04-requirements/functional/compliance.md` ou `00-product/legal-terms/` para versionamento e display.

## C. Itens novos / aggregates potencialmente faltantes

### C.1 — Aggregate `ContactRequest` (E-CV-6)

- **Origem**: PDF Empresa-Currículo descreve fluxo "empresa preenche form → plataforma notifica morador → morador autoriza/recusa"
- **Tipo**: aggregate novo (verificar se `01-domain/aggregates/` já tem)
- **Gravidade**: alta
- **Resolução proposta**: criar/confirmar aggregate `ContactRequest` com state machine (`pending → notified → accepted | rejected | expired`).

### C.2 — Aggregate `PromotionActivation` (VM4)

- **Origem**: PDF Vizinhança Morador descreve geração de métrica via ativação
- **Tipo**: aggregate
- **Gravidade**: média
- **Resolução proposta**: criar/confirmar aggregate. State machine: `created → activated → confirmed (estabelecimento) | expired`.

### C.3 — Aggregate `ResponsibilityTerm` versionado (S29)

- **Origem**: PDF Síndico — termo precisa de versionamento + assinatura
- **Tipo**: aggregate
- **Gravidade**: alta (compliance)
- **Resolução proposta**: aggregate `ResponsibilityTerm` com `{user_id, condominium_id, version_term, timestamp, signed_at}` (R4 imutável).

### C.4 — Aggregate `ManagementEvaluation` (M12)

- **Origem**: PDF Morador — avaliação bimestral anônima
- **Tipo**: aggregate (verificar)
- **Gravidade**: média
- **Resolução proposta**: aggregate `ManagementEvaluation` apenas com agregação estatística (sem author_id por design — privacy by design).

### C.5 — Aggregate `MoradorQuestion` (S12 + M7)

- **Origem**: PDF Síndico (criação) + PDF Morador (resposta)
- **Tipo**: aggregate (verificar se existe)
- **Gravidade**: média
- **Resolução proposta**: aggregate `MoradorQuestion(question, answer_type, deadline, audience, results)`.

### C.6 — Aggregate `CompanyTag` + `CandidateCompanyTag` (E-CV-2)

- **Origem**: PDF Empresa-Currículo — tags manuais por empresa
- **Tipo**: aggregate novo
- **Gravidade**: baixa
- **Resolução proposta**: criar entity (não aggregate root provavelmente). Tabelas: `company_tags(id, company_id, name)` + `candidate_company_tags(company_id, candidate_id, tag_id)`.

### C.7 — Logs / analytics tables

- **Origem**: PDF Empresa-Currículo (E-CV-7) define `search_logs`, `candidate_views`, `pdf_exports`
- **Tipo**: tabelas analytics
- **Gravidade**: baixa
- **Resolução proposta**: criar como projections/read-models (não aggregates) em CQRS. Documentar em `02-architecture/projections/`.

## D. Cross-checks com Reqs canônicos (Fase F)

### D.1 — Categorias Vizinhança

- **PDF Parceiro Vizinhança + PDF Vizinhança Morador**: lista mestre 38 categorias B2C
- **Catálogo macro**: COM-047 (38 categorias)
- **Verificação Fase F**: confirmar que `01-domain/enums/parceiro-vizinhanca-categorias.md` está completa e alinhada.

### D.2 — Áreas de interesse currículo

- **PDF Currículo**: 18 áreas listadas
- **D-099 / COM-049**: 18 áreas + 9 modalidades
- **Verificação Fase F**: criar `01-domain/enums/curriculum-areas.md` (18) e `01-domain/enums/curriculum-modalidades.md` (9) se não existem.

### D.3 — Tipos de atividade Timeline (S12)

- **PDF Síndico**: 26 tipos de atividade canônicos
- **Verificação Fase F**: criar `01-domain/enums/timeline-activity-types.md` se não existe.

### D.4 — Áreas impactadas (S8/S12)

- **PDF Síndico**: 26 áreas impactadas
- **Verificação Fase F**: criar `01-domain/enums/areas-impactadas.md`.

### D.5 — Riscos associados (S12)

- **PDF Síndico**: 9 níveis de risco
- **Verificação Fase F**: criar `01-domain/enums/risk-levels.md`.

### D.6 — Tipos de evento (S16)

- **PDF Síndico**: 13 tipos de evento
- **Verificação Fase F**: criar `01-domain/enums/event-types.md`.

### D.7 — Tipos de comunicado (S24)

- **PDF Síndico**: 8 tipos de comunicado
- **Verificação Fase F**: criar `01-domain/enums/communicado-types.md`.

### D.8 — Tipos de vídeo (MK4)

- **PDF Marketing**: 12 tipos de vídeo
- **Verificação Fase F**: criar `01-domain/enums/video-types.md`.

### D.9 — Marcadores governança (Onboarding TELA 4 sindico + TELA 5 empresa)

- **PDF Onboarding**: lista de marcadores autodeclarados
- **Verificação Fase F**: criar `01-domain/enums/governance-markers.md`.

## E. Invariantes a confirmar / criar

- `INV-TIMELINE-INSERT-ONLY` (R3)
- `INV-AUDIT-IMMUTABLE` (S31)
- `INV-LGPD-DATA-REVEAL` (S20, E4) — dados só revelados após "Tenho interesse"
- `INV-DEPENDENT-MAX-2` (M14, Regra 3)
- `INV-MEMBERSHIP-HISTORY-PRESERVED` (M15)
- `INV-VIDEO-VISIBILITY-VOTING` (MK4) — vídeo só visível durante votação
- `INV-VIDEO-HIDE-NOT-DELETE` (MK5)
- `INV-PROMOTION-SEGMENTATION` (VZ3, VM5)
- `INV-PROMOTION-HISTORY` (VZ6)
- `INV-MARKETING-LINK-NO-AUTO-VINCULO` (E16, MK8)
- `INV-AVALIACAO-ANONIMA` (M12)
- `INV-CONDO-ID-RATE-LIMIT` (M3) — anti-scraping
- `INV-CURRICULUM-NO-DIRECT-CONTACT` (E-CV-2, E-CV-6)
- `INV-COMPLIANCE-TERM-ACTIVE` (S29) — gate

Verificar existência em `01-domain/invariants.md`. Criar as faltantes na Fase H.

## F. Pendências de UX cross-cutting

- **R2 Voice input** em todos os textareas (R2 catálogo macro) — confirmar implementação Web Speech API + i18n PT-BR.
- **R3 Insert-only / nada deletado** — propagar pattern em todos os edits (S13, MK5, etc.).
- **Trial blockers** — telas trial-only ou com gates (S5 empresa síndica vinculada talvez exija plus+).

## Vizinhos

- [[_moc|jornadas/_moc]]
- [[../../../STATE|STATE]] (D-099, D-126, D-129, D-133, D-134)
- [[../../../02-architecture/adr/_moc|ADRs]]
- [[../../../01-domain/invariants|invariants]]
- [[../../../04-requirements/functional/_moc|functional MOC]]
