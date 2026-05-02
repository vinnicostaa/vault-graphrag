---
title: Functional Requirements — Assembly BC
type: requirement
tags:
  - requirements
  - functional
  - assembly
  - voting
  - quorum
  - livekit
  - minutes
  - homologation
  - master-sindico
  - v2
  - ears
source: >-
  legacy specs/requirements/assembly + cross-domain + _chaos/requirements(6).md
  (2026-04-13) + _chaos/mastersindico-requirements.pdf (2026-03-09) +
  _chaos/ARQUITETURA FUNCIONAL DO MÓDULO ASSEMBLEIA.pdf
created: 2026-04-23T00:00:00.000Z
updated: '2026-04-25'
fase_c_absorvido: true
---

# Functional Requirements — Assembly BC

> **Banner Fase C (2026-04-25)**: enriquecido com deltas R48/R52/R57 do `_chaos/requirements(6).md` + `mastersindico-requirements.pdf`. **R52 Proxy revisado**: confirmado 100% digital (Lei 14.063 — assinatura digital ICP-Brasil), validade 24h pós-encerramento; **rejeitada absorção do "upload PDF físico"** do _chaos. Telas (AS1-AS8 painel administradora, presidente) → Fase D.
> **Banner Fase E (2026-04-25)**: 11 novos Reqs (ASM-034 a ASM-044) absorvidos de 3 PDFs Master Síndico (`ARQUITETURA FUNCIONAL §1-13`, `DIAGRAMA VISUAL`, `MÓDULO DAS TELAS`). Inclui: painel administradora 4 capabilities canônicas, mesa diretora pré-iniciar, vídeo institucional pre-roll, telão 2 áreas spec técnico, status final do item (4 estados), microfone alternado + transcrição, apresentação dentro da plataforma, audit trail 22+ eventos canônicos, modo contingência ratificado, cláusulas legais Master Síndico, tipos de voto canônicos. **ASM-028 score reescrito** (4 subíndices granulares — substitui lista anterior).

Requisitos funcionais do **bounded context `assembly`**. Responsabilidade: scheduling de assembleia, edital PDF, pauta, ciência obrigatória, check-in, votação real-time com quorum fracional, procurações (proxy), fila de fala cronometrada, ata imutável, homologação, score de transparência.

Formato **EARS**. Prioridade: M (M1), S (M2), C (M3+), W (backlog). M3 é o marco principal deste BC.

> Destilado de `specs/requirements/assembly.md` + `cross-domain.md` Req 60.2-60.9 + `novo escopo-7.md` §2.1-2.2 + [[00-product/portfolio-de-produtos]] Produto 5.

---

## ASM-001 — Criar Assembly

**Prioridade**: C (M3)  
**EARS**: *Quando* síndico emite `POST /assemblies`, *o sistema deve* validar campos obrigatórios `{title, type, modality, scheduled_date, scheduled_time, location, quorum_type, tempo_fala_configuravel}` e criar `assembly {status='em_preparacao'}`. *Quando* `modality ∈ {hibrida, online}` → preparar campos `{link_transmissao, participantes_online, controle_audio_remoto}` mas **manter desabilitados em M3 MVP** (M4+); *Quando* M3 → forçar `modality = presencial`. *Quando* síndico configura `tempo_fala`, *o sistema deve* aceitar `2_minutos` ou `3_minutos` com opção de `prorrogacao=+2 minutes` (max total 5 min). *Quando* síndico vincula administradora via token (R48 §4) → cria `AssemblyAdministratorLink` (D-127). *Quando* `lista_inadimplentes` ainda não foi enviada e falta < 1h pra primeira_convocacao → bloqueia publicação (R48 §5-§6). *Quando* assembleia inicia → exibe vídeo institucional curto Master Síndico explicando regras (R48 §12 — pre-roll obrigatório).

**Assembly Types** (R-_chaos enums absorção Fase C — 3 itens):
```
ordinaria, extraordinaria, implantacao
```

> **Nota Fase C**: lista anterior `{ordinaria, extraordinaria, ratificacao, outra}` substituída pela canônica do `_chaos/requirements(6).md` R-_chaos enum. `implantacao` substitui `ratificacao` (mais alinhado a contexto condominial — primeira assembleia constitutiva).

**Status enum** (R48 §8 absorção Fase C — 5 itens):
```
em_preparacao → publicada → em_andamento → encerrada
                                    └→ cancelada (qualquer estado pré-em_andamento)
```

**Modalities**: **MVP M3 = presencial**; `hibrida`, `online` campos prepared but disabled M4+ (R48 §10).

**Quorum types**: `simple_majority, qualified_majority, unanimous`.

**Critério de aceite**:
- Validação: data futura, **mín 8 dias corridos de antecedência** (Lei 4.591/64 Art. 24 §1). Bloqueio hard no handler; sem override admin (INV-118).
- Só 1 assembleia ativa por condomínio (status ∈ {em_preparacao, publicada, em_andamento}).
- Cancelamento possível até T-3 dias.
- `lista_inadimplentes` upload via Excel/CSV até T-1h (R48 §5).
- Vídeo institucional pre-roll Master Síndico — fixed asset gerenciado pelo admin (CNT-029 broadcast).
- Assembly Status state machine documentada em `01-domain/state-machines.md`.

**Fonte**: legado `assembly.md` Req 48 + **Lei 4.591/64 Art. 24 §1** (Q-027 Fase 9 — bug 5d corrigido para 8d). **R48 absorção Fase C** (status enum 5, modality preparado online/híbrida M4+, tempo_fala 2/3min+prorroga, lista_inadimplentes T-1h, vídeo institucional pre-roll). Assembleia convocada <8d é **anulável na justiça por vício de convocação**; constraint reforçada em INV-118.

---

## ASM-002 — Edital PDF (geração automática ou upload manual)

**Prioridade**: C  
**EARS**: *Quando* síndico emite `POST /assemblies/{id}/edital`, *o sistema deve* gerar PDF a partir de template (data, hora, local, pauta resumida, regras) OU aceitar upload manual.

**Critério de aceite**:
- Template HTML → PDF via wkhtmltopdf ou Puppeteer (async job).
- Storage S3/MinIO.
- Assinatura digital M4+.

**Fonte**: legado `assembly.md` Req 50.1.

---

## ASM-003 — Pauta: 5 tipos de item

**Prioridade**: C  
**EARS**: *Quando* síndico adiciona item à pauta, *o sistema deve* aceitar `item_type ∈ {informativo, consulta_moradores, votacao_simples, votacao_fracao_ideal, votacao_fornecedor}`.

**Critério de aceite**:
- Estrutura: `{order, title, description, item_type, attachment_url, votacao_required}`.
- Mín 1, máx 20 itens por pauta.

**Fonte**: legado `assembly.md` Req 49.

---

## ASM-004 — Pauta: publicação (lock)

**Prioridade**: C  
**EARS**: *Quando* síndico publica pauta (`POST /assemblies/{id}/agenda/publish`), *o sistema deve* setar `published_at=now()`, travar edições. Correção via adendo.

**Critério de aceite**:
- Publish dispara notificações (ASM-005).
- Congela snapshot de fração ideal (ASM-007).

**Fonte**: legado `assembly.md` Req 49, [[STEERING]] §41.

---

## ASM-005 — Notificações: T-3d, T-1d, T-3h

**Prioridade**: C  
**EARS**: *Quando* pauta é publicada, *o sistema deve* agendar notificações automáticas:
- T-3 dias: "Assembleia em 3 dias — leia a pauta".
- T-1 dia: "Assembleia amanhã às Hh — confirme presença".
- T-3 horas: "Assembleia começa em 3 horas".

Canais: Push (FCM), Email (Resend/SES), SMS (Twilio opt-in M3+).

**Critério de aceite**:
- Scheduler cron horário.
- Tracking: quem clicou.

**Fonte**: legado `assembly.md` Req 50.

---

## ASM-006 — Ciência obrigatória de pauta (app bloqueado)

**Prioridade**: C  
**EARS**: *Enquanto* morador não clica "Li e entendi a pauta", *o sistema deve* bloquear app (modal bloqueante em rotas autenticadas) até confirmação. Após assembleia encerrada → auto-unlock.

**Critério de aceite**:
- Tabela `assembly_acknowledgments {user_id, assembly_id, acknowledged_at, ip}`.
- Middleware: se `acknowledged_at IS NULL` pra assembly ativa → 403.
- Tracking 6 meses.

**Fonte**: legado `assembly.md` Req 50.2.

---

## ASM-007 — Fração ideal: snapshot ao publish

**Prioridade**: C  
**EARS**: *Quando* pauta é publicada, *o sistema deve* congelar snapshot de frações ideais em `assembly_fractions {assembly_id, unit_id, fracao_ideal}`. Votação usa este snapshot (proteção contra alteração mid-flight).

**Dependências**: GR-033, INS-005.

**Critério de aceite**:
- Snapshot imutável.
- Upload opcional via Excel: 2 colunas (unidade, fração); soma = 100% (validação).
- Lock após publish (correção via adendo).

**Fonte**: legado `assembly.md` Req 49, [[STEERING]] §44.

---

## ASM-008 — Pauta: anexos (PDF, vídeos)

**Prioridade**: C  
**EARS**: *Quando* síndico anexa documento ou vídeo a item, *o sistema deve* aceitar PDF/Excel (via S3 presigned) e vídeo (via Mux presigned — CNT-001).

**Critério de aceite**:
- Tamanho limite GR-018.
- Vídeo de fornecedor: filtro via acervo cadastrado.

**Fonte**: legado `assembly.md` Req 49, `novo escopo-7.md` §2.1.

---

## ASM-009 — Enquetes preliminares (consultivas)

**Prioridade**: C  
**EARS**: *Quando* síndico cria enquete antes da assembleia, *o sistema deve* aceitar tipos: `sim_nao_nao_sei, multiple_choice, scale_1_to_5`. Resultado **não é vinculante** (informativo).

**Critério de aceite**:
- Publicado na pauta.
- Não confunde com votação oficial.

**Fonte**: legado `assembly.md` Req 51.

---

## ASM-010 — Procurações (Proxy) — 100% digital, validade 24h pós-encerramento

**Prioridade**: C (M3+)  
**EARS**: *Quando* morador emite procuração via app/web, *o sistema deve* (Fase C ratificação): (1) validar `proxy_holder` é user da plataforma (CPF/CNPJ pré-cadastrado — pode ser morador do mesmo condomínio OU terceiro registrado), (2) aceitar `scope ∈ {all_items, specific_items[]}` (R52 §1), (3) **NÃO exigir upload de documento físico PDF** (rejeitado em D-_chaos absorção Fase C — Lei 14.063 dispensa documento físico quando assinatura digital ICP-Brasil válida), (4) gerar assinatura digital pelo morador via fluxo OIDC + step-up MFA (IDN-036), (5) administradora valida via painel (R52 §3 — admin-side validation gera evento `ProxyValidated`), (6) prazo de submissão: **até momento antes de "Iniciar assembleia"** (R52 §4); (7) cancelamento permitido pré-início. *Quando* procuração ativa e titular **também** vota → voto do titular prevalece (proxy NÃO pode divergir — invariante 8). *Quando* assembleia encerra → procuração permanece válida +24h (R52 §8) → expira automaticamente.

**Critério de aceite**:
- Tabela `proxies {id, authorized_by_user_id, proxy_holder_user_id, assembly_id, scope (enum), specific_items_jsonb, signature_hmac, validated_by_user_id, validated_at, valid_until_timestamp, revoked_at, created_at}`.
- `valid_until = assembly.ended_at + 24h` (calculado quando assembly encerra).
- Assinatura digital ICP-Brasil pleno em M4+; M3 stub HMAC + audit trail (Lei 14.063 aceita ambos como válidos no contexto digital nativo da plataforma).
- **Sem upload de documento PDF físico** (rejeitado Fase C — discrepância do `_chaos/requirements(6).md` R52 §2/§9 corrigida por decisão canônica do João).
- Substituição de proxy: revogar atual + emitir nova (não UPDATE).
- LGPD: dados do `proxy_holder` registrados em `lgpd_logs` (purpose: assembly_representation).

> **Decisão Fase C — discrepância resolvida**: o `_chaos/requirements(6).md` R52 §2/§9 menciona "require proxy document upload (physical document for validation)" e "physical presentation". **REJEITADO** com base no contexto canônico Master Síndico v2 (plataforma 100% digital + Lei 14.063 + alinha STEERING UX mobile-first + benchmark Stripe/DocuSign). Documentação inicial absorvida do legado físico não se aplica ao MVP digital.

**Dependências**: IDN-036 (step-up MFA pra assinatura), GR-023 (versionamento termos), ADR-0028 (HMAC keyed), Lei 14.063 (assinatura digital BR), D-129 (membership_id).

**Fonte**: legado `assembly.md` Req 52; **R52 absorção Fase C com rejeição parcial** (rejeita upload PDF físico; mantém scope, validity 24h, validation pela administradora).

---

## ASM-011 — Simulador de quorum

**Prioridade**: C  
**EARS**: *Quando* síndico emite `GET /assemblies/{id}/simulator?present_fraction=0.60`, *o sistema deve* retornar: "Quorum atingido? SIM/NÃO + Votos necessários para X passar" baseado em fração ideal presente.

**Critério de aceite**:
- Função pura (sem persistência).
- Retorna `{quorum_reached, votes_needed_by_type}`.

**Fonte**: legado `assembly.md` Req 55.

---

## ASM-012 — Check-in (3 modalidades)

**Prioridade**: C  
**EARS**: *Quando* morador faz check-in, *o sistema deve* aceitar 3 modalidades:
1. **App (padrão)**: QR code + scan + biometria/PIN (M3+).
2. **Terminal (kiosk)**: CPF + bloco/unidade + confirmação.
3. **Manual**: síndico confirma presença (ex: deficiente visual).

**Critério de aceite**:
- Tabela `assembly_check_ins`.
- Latência WebSocket < 200ms pro Painel (ASM-017).
- Ausente momentâneo ≠ definitivo.

**Fonte**: legado `assembly.md` Req 56.

---

## ASM-013 — Termos legais de assembleia

**Prioridade**: C  
**EARS**: *Quando* morador acessa 1ª assembleia, *o sistema deve* exigir aceite de termos (regras, direitos, penalidades). Modal bloqueante. Versão rastreada.

**Dependências**: GR-023, IDN-024.

**Critério de aceite**:
- Tabela `assembly_legal_acceptances`.

**Fonte**: legado `assembly.md` Req 60.12.

---

## ASM-014 — Votação real-time: 4 tipos

**Prioridade**: C  
**EARS**: *Quando* síndico abre votação, *o sistema deve* suportar:
1. **Votação Simples** — sim/não (maioria simples).
2. **Votação com Fração Ideal** — peso pela fração (> 50% de fração).
3. **Votação de Fornecedor** — entre 2+ propostas com fração (COM-018).
4. **Eleição** — candidatos com fração.

**Critério de aceite**:
- Tabelas `assembly_votings`, `assembly_votes`, `assembly_voting_results`.
- WebSocket `/ws/assemblies/{id}/voting` latência < 200ms.

**Fonte**: legado `assembly.md` Req 57.

---

## ASM-015 — Voto único por (agenda_item, voter)

**Prioridade**: C  
**EARS**: *Quando* voto é submetido, *o sistema deve* persistir com UNIQUE `(agenda_item_id, voter_id)`. Tentativa de re-voto → `409 conflict`.

**Dependências**: [[STEERING]] §42, A-025/A-026 legado.

**Critério de aceite**:
- DB UNIQUE constraint.
- `ErrConflict` tratado no use case.
- PBT: qualquer sequência de POST → 1 voto persiste.

**Fonte**: legado `assembly.md` Req 57 invariante 3, A-025.

---

## ASM-016 — Voto: imutável, presencial assistido, ausência momentânea

**Prioridade**: C  
**EARS**: *Quando* voto é registrado, *o sistema deve* persistir imutável (`{voter_id, agenda_item_id, choice, fracao_ideal_snapshot, ip, user_agent, device_fingerprint, origin ∈ {app, presencial_assistido}, cast_at}`). *Quando* `origin = presencial_assistido` (síndico/administradora vota em nome de morador via terminal — R57 §6) → registrar adicionalmente `{operator_id, vote_type, unit, terminal_session_id}` + termo_de_voto assinado pelo morador. *Quando* morador está ausente momentaneamente (saiu da sessão), *o sistema deve* dar janela de 15 min pra voltar. *Após* 15 min → abstenção automática (registra com `auto_abstention=true`). *Quando* voting encerra com quorum não alcançado, *o sistema deve* marcar voting como `inconclusive` e permitir reabertura (R57 §14).

**Termo de voto canônico** (R57 §8 absorção Fase C — texto fixo, versionado):

> "Declaro que exerço este voto em nome da unidade vinculada ao meu acesso, sob minha responsabilidade."

**Critério de aceite**:
- Voto presencial assistido: termo_de_voto obrigatório assinado pelo morador (HMAC-keyed, audit_trail).
- Rejeita mudança pós-envio (R57 §15 + INV-039/INV-071).
- `origin` enum + log completo (R57 §16).
- Auto-save real-time (R57 §19-§20) — Redis backup + cont-sync.
- Reabertura permitida pré-homologação (R57 §13-§14) — síndico zera + redoes; nova `voting_session_id` distinta (preserva histórico).
- Abstenção automática registrada com flag `auto_abstention=true` (distingue de abstenção ativa).

**Fonte**: legado `assembly.md` Req 57; **R57 §6-§8, §13-§20 absorção Fase C** (presencial_assistido + termo de voto + reabertura + auto-save real-time).

---

## ASM-017 — Painel do Presidente (síndico)

**Prioridade**: C  
**EARS**: *Quando* síndico acessa `/assemblies/{id}/president`, *o sistema deve* renderizar dashboard com: pauta, itens, presentes real-time, votações abertas, fila de fala, alertas (quorum crítico, tempo excedido). Síndico NÃO vota (imparcialidade).

**Critério de aceite**:
- Acesso: `role_type=principal` ou designado.
- WebSocket listener em check-in, vote, speech.
- Telemetria % presença em tempo real.

**Fonte**: legado `assembly.md` Req 60.2.

---

## ASM-018 — Telão 2 áreas (apresentação + operacional)

**Prioridade**: C  
**EARS**: *Quando* telão é ativado, *o sistema deve* renderizar 2 áreas:
- **Presentation**: pauta atual, gráficos resultado, comunicados, vídeos empresa (durante supplier vote).
- **Operational**: quorum %, fila de fala, alertas.

**Critério de aceite**:
- Layout responsivo 2 monitores.
- WebSocket mesma conexão, múltiplos listeners.
- Atualização < 200ms.

**Fonte**: legado `assembly.md` Req 60.3.

---

## ASM-019 — Fila de fala cronometrada

**Prioridade**: C  
**EARS**: *Quando* morador clica "Quero falar", *o sistema deve* adicionar à fila FIFO. Síndico aprova entrada. Timer inicia (2 ou 3 min — config ASM-001). Alerta 30s antes. Síndico pode estender +1 min (1x). Timeout → Livekit corta áudio automaticamente.

**Dependências**: CNT-022, Livekit integration.

**Critério de aceite**:
- Tabelas `assembly_speech_queue`, `assembly_speeches`.
- WebSocket `/ws/assemblies/{id}/speech`.
- Integração Livekit API de mute.

**Fonte**: legado `assembly.md` Req 60.5.

---

## ASM-020 — Live streaming (Livekit, M3+)

**Prioridade**: C  
**EARS**: *Quando* assembleia híbrida/online (M4+), *o sistema deve* criar Livekit room, gerenciar participants (síndico=moderador, moradores=participantes), gravar automaticamente.

**Critério de aceite**:
- TURN/STUN gerenciados Livekit Cloud.
- Recording → S3.
- Chat metadata channel.

**Fonte**: legado `assembly.md` Req 68 (integration), [[00-product/portfolio-de-produtos]] Produto 5.

---

## ASM-021 — Votação de Fornecedor integrada

**Prioridade**: C  
**EARS**: *Quando* pauta tem item `votacao_fornecedor`, *o sistema deve* integrar `SupplierVoteSession` (COM-018): mostrar vídeos empresa durante votação (CNT-009 grants), aplicar quorum fracional.

**Dependências**: COM-018, CNT-009.

**Critério de aceite**:
- Vídeos empresa liberados a moradores durante votação ativa; revogados ao encerrar.

**Fonte**: legado `assembly.md` Req 57 + `commercial.md` Req 21.

---

## ASM-022 — Modo contingência (offline)

**Prioridade**: C  
**EARS**: *Quando* WebSocket cai, *o sistema deve* salvar votos em Redis local (`assembly:{id}:offline_votes:{user_id}`). *Quando* conexão volta → sync automático. Conflito: último voto válido.

**Critério de aceite**:
- Auto-sync periódico.
- Notificação: "Modo offline — votos serão sincronizados".

**Fonte**: legado `assembly.md` Req 60.10.

---

## ASM-023 — Encerramento de assembleia

**Prioridade**: C  
**EARS**: *Quando* síndico clica "Encerrar Assembleia", *o sistema deve* validar todas as votações encerradas, gerar ata PDF automaticamente, publicar na Timeline tipo `assembly_result`, transicionar `status: in_progress → closed`.

**Dependências**: INS-011.

**Critério de aceite**:
- Job async pra ata.
- Evento NATS `assembly.closed` → `timeline.entry.created`.

**Fonte**: legado `assembly.md` Req 58.

---

## ASM-024 — Ata: geração automática

**Prioridade**: C  
**EARS**: *Quando* ata é gerada, *o sistema deve* conter: data, hora, local, presentes (lista anonimizada), pauta + resultado, falas (texto/resumo), procurações usadas, assinatura digital síndico (M4+).

**Critério de aceite**:
- Template HTML → PDF.
- S3 backup.

**Fonte**: legado `assembly.md` Req 58.

---

## ASM-025 — Ata: imutabilidade pós-homologação

**Prioridade**: C  
**EARS**: *Quando* ata é homologada (ASM-027), *o sistema deve* travar edições. Correção via adendo em Timeline.

**Critério de aceite**:
- Campo `homologated_at NOT NULL` → lock.
- Audit trail.

**Fonte**: legado `assembly.md` Req 58, Req 60.6, [[STEERING]] §43.

---

## ASM-026 — Relatórios de assembleia (6 tipos)

**Prioridade**: C  
**EARS**: *Quando* síndico emite `GET /assemblies/{id}/reports/{type}?format=csv|pdf`, *o sistema deve* gerar:
1. **Presença** — moradores presentes/ausentes, fração.
2. **Votação** — resultado detalhado, votos por opção.
3. **Procurações** — lista de proxies usadas.
4. **Fala** — registro, tempo por morador, topics.
5. **Consolidado** — resumo executivo (1-2 pgs).
6. **Transparência** — score de transparência (ASM-028).

**Critério de aceite**:
- Anônimo: lista de presença sem nomes (só unidade/fração).
- Acesso: síndico + admin.

**Fonte**: legado `assembly.md` Req 59.

---

## ASM-027 — Homologação (votação obrigatória pós-ata)

**Prioridade**: C  
**EARS**: *Quando* ata é gerada, *o sistema deve* abrir votação "Aprova a ata?" com maioria simples. *Quando* aprovada → `homologated_at=now()` → ata imutável.

**Critério de aceite**:
- Tabela `assembly_homologations`.
- Contestação via adendo (INS-016).

**Fonte**: legado `assembly.md` Req 60.6.

---

## ASM-028 — Score de transparência (10 componentes — Fase E)

**Prioridade**: C  
**EARS**: *Quando* assembleia é homologada, *o sistema deve* calcular score (0-100) em 4 subíndices canônicos (atualizado Fase E `_chaos/ARQUITETURA-ASSEMBLEIA.pdf §8`):

**Convocação (30pt)**:
1. Pauta publicada ≥ 8 dias (10pt) — conforme ASM-001 / Lei 4.591/64 Art. 24 §1 (binário).
2. Edital distribuído (10pt) — binário.
3. % Ciência registrada (10pt) — proporcional `round((% / 100) * 10)`.

**Participação (30pt)**:
4. % Presença sobre convocados (10pt) — proporcional.
5. % Votantes sobre aptos (10pt) — proporcional.
6. % Visualização documentos+fornecedores (10pt) — proporcional (engajamento prévio).

**Votação (20pt)**:
7. Quórum atingido em todos os itens (10pt) — binário.
8. Regularidade da homologação (10pt) — todos itens homologados em ≤ 7 dias (binário).

**Documentação (20pt)**:
9. Completude trilha auditoria 20+ eventos (10pt) — binário.
10. Cumprimento integral da pauta + tempo médio por item (10pt) — composto (50% + 50%).

> **NOTA Fase E — substituição de lista**: a lista anterior ("Votação online disponível", "Ata publicada Timeline", "Relatórios públicos acessíveis", "Fala equilibrada", "Sem atrasos encerramento") é substituída pela canônica do `_chaos/ARQUITETURA-ASSEMBLEIA.pdf §8` (mais granular e alinhada à arquitetura PT-BR).

**Critério de aceite**:
- Função SQL ou job NATS.
- `assemblies.transparency_score` (0-100) + 4 subíndices em colunas separadas (`score_convocacao`, `score_participacao`, `score_votacao`, `score_documentacao`).
- Histórico ao longo do tempo (12 últimas assembleias do condomínio em ASM-030).
- Faixas: Excelência 90-100, Bom 75-89, Regular 60-74, Baixo 40-59, Crítico 0-39.

**Fonte**: legado `assembly.md` Req 60.7 + **`_chaos/ARQUITETURA-ASSEMBLEIA.pdf §8` absorção Fase E (substitui lista anterior)**.

---

## ASM-029 — Notificações pós-encerramento

**Prioridade**: C  
**EARS**: *Quando* assembleia encerra, *o sistema deve* disparar: "Ata disponível", "Ata homologada", "Score: X/100". Push + email + SMS (opt-in).

**Critério de aceite**:
- Personalizadas (resultado votações relevantes ao morador).

**Fonte**: legado `assembly.md` Req 60.8.

---

## ASM-030 — Widget score de transparência (perfil condomínio)

**Prioridade**: C  
**EARS**: *Quando* score é atualizado, *o sistema deve* renderizar badge "Transparência: X/100" no perfil público do condomínio + histórico 12 assembleias.

**Dependências**: INS-038.

**Critério de aceite**:
- Componente SolidJS.
- Benchmark comparativo.

**Fonte**: legado `assembly.md` Req 60.9.

---

## ASM-031 — Auto-save Redis (recuperação de crash)

**Prioridade**: C  
**EARS**: *Quando* estado de assembly muda (votação aberta, presença), *o sistema deve* auto-save em Redis. *Quando* crash → recuperação < 1 min.

**Critério de aceite**:
- NFR crítico (sprint 7 legado).
- Keys `assembly:{id}:state:*`.

**Fonte**: legado `assembly.md` Req 60.10.

---

## ASM-032 — Votação assíncrona (MVP M3 — simplificação)

**Prioridade**: C (M3 MVP)  
**EARS**: *Enquanto* M3 MVP limita assembleia **presencial + votação assíncrona** (via app), *o sistema deve* implementar votação com janela de tempo (aberto → encerrado), sem WebRTC live. WebRTC Livekit em M4+.

**Critério de aceite**:
- 1 voto por unidade.
- Janela de tempo (só vota se evento "Aberto").
- Feedback visual imediato.

**Fonte**: legado `novo escopo-7.md` §2.1-2.2 (menciona assembleia assíncrona).

---

## ASM-033 — Link temporário de evento (compartilhamento)

**Prioridade**: C  
**EARS**: *Quando* síndico gera link temporário de evento (edital, pautas, votação), *o sistema deve* emitir URL com TTL configurável + janela de acesso.

**Critério de aceite**:
- Signed URL.
- Audit: "quem acessou o link".

**Fonte**: legado `novo escopo-7.md` §2.1.

---

## Invariantes de assembly

1. 1 assembleia ativa por condomínio.
2. Pauta imutável após publicação (versionamento via adendo).
3. Voto único por (agenda_item, voter) — UNIQUE DB.
4. Voto imutável pós-envio.
5. Fração ideal NUMERIC(5,4), snapshot congelado ao publish.
6. Ausência momentânea: 15 min window antes de abstenção.
7. Ata auto-publicada na Timeline (imutável após homologação).
8. Procurador não pode votar diferente do titular.
9. Síndico não vota (imparcialidade).
10. WebSocket latência < 200ms (NFR).

---

## Pendências

- ⚠️ **PENDÊNCIA ASM-020**: Livekit Cloud vs self-hosted — ADR M4+ (custo).
- ⚠️ **PENDÊNCIA ASM-032**: decidir entre M3 assíncrona vs live — reconciliar matriz cliente vs specs legado.
- ⚠️ **PENDÊNCIA ASM-024**: template de ata (jurídico review) — dual-check BR advogado.
- ⚠️ **RESOLVIDO Fase E ASM-028**: lista de componentes substituída por canônica `_chaos/ARQUITETURA §8` (4 subíndices). Pendência "fala equilibrada" eliminada (não está na lista canônica).
- ⚠️ **PENDÊNCIA ASM-010**: assinatura digital procuração — ADR provedor (Bry, ITI, AC SERPRO) M4+; M3 stub HMAC.
- ⚠️ **PENDÊNCIA ASM-039 (Fase E)**: provider de transcrição (Whisper local vs Speechmatics cloud) + algoritmo de "captação ótima" (VAD + RMS).
- ⚠️ **PENDÊNCIA ASM-040 (Fase E)**: conversor PowerPoint → PDF (cliente-side via libreoffice headless OU servidor) — ADR.
- ⚠️ **PENDÊNCIA ASM-022/042 (Fase E)**: detecção automática de problemas (heartbeat WebSocket threshold) + sync conflict resolution policy (LWW vs reject).

---

## Fase E — Reqs novos absorvidos dos 3 PDFs Master Síndico (2026-04-25)

> **Fonte**: `_chaos/ARQUITETURA-ASSEMBLEIA.pdf` (1281 linhas) + `DIAGRAMA-VISUAL.pdf` (289 linhas) + `MÓDULO-DAS-TELAS.pdf` (661 linhas). Absorvidos em Fase E em paralelo aos demais materiais já cobertos pela Fase C (R48/R52/R57).

---

## ASM-034 — Painel da Administradora (4 capabilities canônicas)

**Prioridade**: C (M3)  
**EARS**: *Quando* síndico cede cargo de administradora via convite (D-127), *o sistema deve* atribuir `PermissionGroup pg.assembly.administrator_delegation` (ADR-0039) com 4 capabilities canônicas:

1. `assembly.fracao_ideal.import` — importar fração ideal via planilha (ASM-007).
2. `assembly.inadimplencia.register` — registrar inadimplência até T-1h da 1ª convocação (R48 §5).
3. `assembly.proxy.validate` — validar procurações (ASM-010 / Lei 14.063 — sem PDF físico).
4. `assembly.assembly.homologate` — homologar votação por item (ASM-027).

**Capabilities adicionais**:
- `assembly.observation.add` — observações (com ou sem validação cruzada do síndico — configurável).
- `assembly.edital.generate` — gerar edital automático ou anexar.
- `assembly.assembly.end` — encerrar assembleia (alternativo ao síndico).
- `assembly.vote.assisted_register` — voto presencial assistido (R57 §6 + ASM-016).
- `assembly.contingency.activate` — ativar modo contingência (ASM-022).

**Critério de aceite**:
- Capability check via ABAC + AssemblyAdministratorLink ativo.
- Cada ação dispara evento `AssemblyAdministratorLinkUsed` (audit append-only).
- Cargo auto-invalida em `Assembly.End()` (D-127 cascade).

**Fonte**: `_chaos/ARQUITETURA §3.3-3.4` + `MÓDULO-TELAS TELA 3,7,8` + D-127.

---

## ASM-035 — Mesa Diretora (composição obrigatória pré-iniciar)

**Prioridade**: C  
**EARS**: *Quando* síndico clica "Iniciar assembleia", *o sistema deve* validar cadastro completo da mesa diretora antes de transição `published → in_progress`:

- Presidente da mesa (obrigatório — pode ser o próprio síndico): `{nome, CPF}`.
- Secretários (1 a 3): `{nome, CPF}` cada.
- Administradora presente (obrigatório se houver `AssemblyAdministratorLink` ativo): `{empresa, nome, CPF}`.
- Auditor (opcional, D-132): `{empresa, nome, CPF}` → cria Membership efêmera `role=assembly_auditor` ABAC, válida durante a assembleia.
- Fornecedores presentes (opcional, lista): `{empresa, representante, CPF}`.

**Critério de aceite**:
- Bloqueia `Start()` se composição inválida.
- Persiste em `assembly_directorship` (snapshot estrutural).
- Auditor: cria/destrói Membership automaticamente (D-132).
- Síndico não vota (INV-081 — imparcialidade).

**Fonte**: `_chaos/ARQUITETURA §5.4-5.5` + `MÓDULO-TELAS TELA 9` + D-132.

---

## ASM-036 — Vídeo Institucional Pre-Roll (regra)

**Prioridade**: C  
**EARS**: *Quando* `Assembly.Start()` é executado, *o sistema deve* disparar reprodução obrigatória de vídeo institucional Master Síndico ANTES de liberar painel ao vivo. Vídeo apresenta regras de participação, fala, votação e deliberação. Não é skippable.

**Critério de aceite**:
- Asset gerenciado pela Master Síndico (CNT-029 broadcast asset).
- Player Mux com `autoplay=true`, `skippable=false`.
- Estado bloqueado até término (`videoEndedAt != null`).
- Versionamento: `institutional_video_id` registrado em audit trail.

**Fonte**: `_chaos/ARQUITETURA §5.6` + R48 §12 (Fase C já mencionou em ASM-001; aqui ratifica com spec técnico).

---

## ASM-037 — Telão 2 áreas (especificação técnica)

**Prioridade**: C  
**EARS**: *Quando* síndico ativa telão (`POST /assemblies/:id/telao/start`), *o sistema deve* renderizar layout dual em rota pública `/assembleias/:id/live/telao` com signed token:

**Área 1 — Apresentação (60% da largura)**:
- Renderiza: PDF | imagem | vídeo | resultado de votação tabular/gráfico.
- Quem apresenta: síndico, administradora, auditor (capability `assembly.presentation.show`).
- Materiais aceitos: PDF, PNG/JPG, MP4. PowerPoint **deve ser convertido em PDF** antes do upload.

**Área 2 — Painel operacional (40% da largura)**:
- Pauta completa com item destacado.
- Unidades presentes / online / ausentes.
- Fila de fala (cronômetro).
- Quórum presente / quórum necessário (item atual).
- Relógio da assembleia.

**Critério de aceite**:
- Layout responsivo otimizado projetor 1920x1080+.
- WebSocket `/ws/assemblies/:id/state` mesma conexão, múltiplos listeners.
- Atualização < 200ms.
- Token de acesso público (signed URL — TTL = duração assembleia).
- Durante votação ativa: Área 1 muda automaticamente para tabela de votos (`unidade | voto`).

**Fonte**: `_chaos/ARQUITETURA §5.7-5.8 + §5.12` + `MÓDULO-TELAS TELÃO`.

---

## ASM-038 — Status final do item de pauta (4 estados)

**Prioridade**: C  
**EARS**: *Quando* síndico finaliza um item de pauta, *o sistema deve* permitir 4 status terminais:

- `RESOLVIDO` — votação encerrada com resultado válido (aprovado/rejeitado/escolha).
- `ADIADO` — síndico decide adiar para próxima assembleia.
- `NÃO_RESOLVIDO` — quórum não atingido / votação inconclusiva.
- `PREJUDICADO` — perdeu sentido (ex: já resolvido em outro item ou item dependente foi rejeitado).

Cada item registra:
- `hora_abertura`
- `hora_entrada_votacao`
- `hora_encerramento`
- `duracao_total_discussao`

**Critério de aceite**:
- Enum atualiza `AgendaItem.state` (state machine).
- Imutável após homologação.
- Aparece em ata + registro final + relatórios.

**Fonte**: `_chaos/ARQUITETURA §5.9` + `DIAGRAMA-VISUAL §"FINALIZAÇÃO DO ITEM"`.

---

## ASM-039 — Microfone alternado + transcrição (controle de fala avançado)

**Prioridade**: C (M3+)  
**EARS**: *Quando* fila de fala chama um morador (ASM-019), *o sistema deve* habilitar microfone via Livekit API com **alternância automática** (mute/unmute) para captação ótima e transcrição. *Quando* fala é registrada, *o sistema deve* marcar como microfonada (apenas falas microfonadas vão para a ata).

**Critério de aceite**:
- Livekit API `RoomService.UpdateParticipant` para mute/unmute programático.
- Algoritmo de captação: VAD (Voice Activity Detection) + RMS sliding window — pendência ADR.
- Transcrição assíncrona: Whisper (local) ou Speechmatics (cloud) — pendência ADR M4+.
- Lembrete UX bem visível pré-assembleia: "Apenas falas microfonadas serão registradas em ata".

**Pendências**:
- ⚠️ ADR provider de transcrição (Whisper local vs cloud — custo).
- ⚠️ Algoritmo "captação ótima" precisa spec.

**Fonte**: `_chaos/ARQUITETURA §5.10`.

---

## ASM-040 — Apresentação dentro da plataforma

**Prioridade**: C  
**EARS**: *Quando* síndico/administradora/auditor apresenta material, *o sistema deve* permitir upload e renderização sem sair da plataforma. Materiais aceitos: PDF, imagem (PNG/JPG), vídeo (MP4 — Mux player).

**Critério de aceite**:
- Storage S3/MinIO (presigned URL).
- PowerPoint não suportado nativo — converter em PDF antes (cliente-side ou servidor — definir).
- Durante apresentação: bloqueia outras Áreas-1 do telão.
- Marca em audit trail: `MaterialPresented {assembly_id, item_id, material_type, presenter_id, started_at}`.

**Fonte**: `_chaos/ARQUITETURA §5.7`.

---

## ASM-041 — Trilha de auditoria 20+ eventos canônicos

**Prioridade**: C  
**EARS**: *Quando* qualquer ação relevante de assembleia ocorre, *o sistema deve* gravar evento em audit trail imutável (append-only) com campos `{usuario_responsavel, perfil, data_hora, assembleia_id, condominio_id, unidade_id?, justificativa?}`.

**Eventos canônicos mínimos (20+)**:
1. `AssemblyCreated`
2. `AssemblyAdministratorLinkGranted` (D-127)
3. `EditalGenerated` / `EditalUploaded`
4. `AssemblyPublished`
5. `CienciaRegistered`
6. `AttendanceConfirmed`
7. `FracaoIdealImported`
8. `InadimplenciaRegistered` / `InadimplenciaLiberadaManual`
9. `ProxyCreated`
10. `ProxyValidated` / `ProxyRejected`
11. `CheckIn` (3 origens: app, qr, terminal)
12. `AssemblyStarted`
13. `MesaDiretoraCadastrada` (presidente + secretários)
14. `AgendaItemStateChanged`
15. `SpeechRequested` / `SpeechCalled` / `SpeechEnded`
16. `VoteCast` (com origin: app, presencial_assistido)
17. `HomologationApproved`
18. `ContingencyModeActivated` / `ContingencyModeDeactivated`
19. `MaterialPresented`
20. `AssemblyClosed`
21. `MinutesGenerated` / `MinutesPublished` / `MinutesHomologated`
22. `TransparencyScoreCalculated`

**Critério de aceite**:
- Schema unificado `audit_trail_events` (cross-domain).
- Cada evento exporta-se em relatório CSV + PDF (ASM-026).
- Imutabilidade enforced via append-only constraint.
- Usado para componente 9 do score de transparência (ASM-028).

**Fonte**: `_chaos/ARQUITETURA §9` + `cross-domain` audit-trail.

---

## ASM-042 — Modo de Contingência (ratificação + spec UX)

**Prioridade**: C  
**EARS**: *Quando* síndico ou administradora (capability `assembly.contingency.activate`) ativa modo de contingência, *o sistema deve*:

- Aceitar motivo enum `{queda_internet, falha_dispositivo, impossibilidade_votacao_digital, outro}` + descrição livre.
- Permitir ativação por item específico OU para assembleia inteira.
- Bloquear voto pelo app (banner "Dirija-se ao terminal de apoio").
- Manter voto presencial assistido como único caminho (ASM-016).
- Registrar audit trail: `ContingencyModeActivated {responsible_id, reason, item_id?, activated_at}`.
- Marcar items afetados com flag `contingency_active=true`.
- Permitir desativação manual quando situação normalizada.
- Aparecer em relatório (ASM-026) + ata homologada.

**Critério de aceite (NFR)**:
- Auto-save Redis em `assembly:{id}:state:*` durante contingência (ASM-031).
- Recovery < 1min em crash.
- Conflict resolution: LWW (last-write-wins) timestamp para offline votes.

**Fonte**: `_chaos/ARQUITETURA §11`. **Ratifica e enriquece ASM-022 com spec UX completo**.

---

## ASM-043 — Cláusulas funcionais de proteção Master Síndico (legais)

**Prioridade**: C (M3)  
**EARS**: *O sistema deve* exibir, em momentos adequados, os seguintes termos legais de proteção da plataforma:

**Termo de uso da assembleia** (no aceite inicial):
> "A Master Síndico atua como meio tecnológico de organização e registro, não se responsabilizando pelo mérito das deliberações nem pela verificação material da representação civil além das informações e declarações fornecidas pelos usuários e responsáveis do condomínio."

**Declaração de veracidade** (na ciência):
> "O usuário declara que: participa na condição legítima vinculada à unidade, as informações prestadas são verdadeiras, os documentos enviados correspondem à realidade."

**LGPD** (no primeiro acesso):
> "O usuário toma ciência do tratamento de dados para: convocação, participação, votação, rastreabilidade, histórico da assembleia."

**Autorização de gravação** (se assembleia for gravada):
> Checkbox específico de ciência. Bloqueia entrada se não marcado.

**Critério de aceite**:
- Versionamento de termos (GR-023).
- Aceites registrados em `assembly_legal_acceptances`.
- Modal bloqueante (não pode dispensar).
- Auditoria: `LegalTermsAccepted {user_id, term_version, accepted_at, ip}`.

**Fonte**: `_chaos/ARQUITETURA §12.1-12.4`.

---

## ASM-044 — Tipos de voto canônicos (UI dropdown)

**Prioridade**: C  
**EARS**: *Quando* síndico configura tipo de votação para um item de pauta, *o sistema deve* aceitar 2 modalidades canônicas (`MÓDULO-TELAS` + `DIAGRAMA-VISUAL`):

- `SIM_NAO_ABSTENCAO` — voto binário com abstenção como 3ª opção válida.
- `ESCALA_1_5_ABSTENCAO` — Likert 1-5 + abstenção (qualidade percebida; agregado por média ponderada).

**Adicionalmente** (já em ASM-014, ratificação):
- `escolha_fornecedor` — múltipla escolha entre N fornecedores.
- `eleicao` — múltiplos candidatos com fração ideal.

**Critério de aceite**:
- Enum `voting_type` em DB.
- UI dropdown na criação/edição de item pré-publicação.
- Cada tipo gera tela de votação específica (ASM-014 + ver `live-session.md`).

**Fonte**: `_chaos/MÓDULO-TELAS TELA 4` + `DIAGRAMA-VISUAL §VOTAÇÃO`.

---

## Fase 12 — Req ASM-ELE-AVAL (2026-04-24)

### ASM-ELE-AVAL — Avaliação escondida obrigatória em pauta de eleição

**Fonte**: [[../../STATE#D-104 — Avaliação escondida de gestão em eleição ATIVADA (fecha Q40)|D-104]]; [[../../01-domain/invariants#INV-ASM-023]].
**Escopo**: spec M1 / implementação M3.

**Enunciado EARS**:

- **Quando** uma assembleia estiver ativa e o morador autenticado estiver em pauta tipo `eleicao_sindico`, **o sistema deve** exibir a tela de avaliação escondida de gestão antes de liberar o formulário de voto no candidato.
- **Quando** o morador tentar submeter voto em candidato sem ter submetido avaliação escondida prévia, **o sistema deve** rejeitar o voto com erro `PRECONDITION_FAILED` e mensagem "Avaliação confidencial obrigatória antes do voto".
- **Quando** o morador submeter a avaliação escondida, **o sistema deve** gravar em `hidden_assessments` com `morador_hash = HMAC(mandate_id, user_id)` e liberar o voto.
- **Quando** o síndico atual tentar acessar `/api/v1/assembly/:id/hidden-assessments/individual/:morador_id`, **o sistema deve** retornar `403 FORBIDDEN` independentemente do ABAC (regra absoluta).
- **Quando** o mandato do síndico encerrar, **o sistema deve** calcular agregação (fatia de 25% em INV-GOV-001) sobre o conjunto de avaliações escondidas daquele mandato.

**Dados da avaliação** (canonizado via [[../../STATE#d-110|D-110]], 5 perguntas escala 1-10, campos fixos sem texto livre):

1. Como você avalia a **transparência** da gestão atual?
2. Como você avalia a **qualidade das decisões** tomadas?
3. Como você avalia a **resposta às demandas dos moradores**?
4. Como você avalia o **cuidado com o patrimônio** do condomínio?
5. Como você avalia se o síndico **mantém os documentos em dia** (atas, declarações, recibos)?

**ABAC**: apenas role `resident` do condomínio pós-check-in. Zero acesso individual posterior a qualquer role, inclusive `admin`.

**Audit**: evento `HiddenAssessmentSubmitted(mandate_id, morador_hash, submitted_at)` SEM payload de respostas. Audit log registra fato, não conteúdo.

**UX obrigatória** (ver [[../../03-subprojects/web/ui-catalog/assembleia/ASM-AVAL-ELEICAO]]):
- Aviso "Esta avaliação é confidencial — não vai para o síndico atual"
- NavGuard bloqueia navegação
- Submit enabled só com todas respondidas
- Pós-submit: redirect para voto; **não exibir** score ao morador

**Erros**: duplicata 409 `já avaliou`; assembleia não ativa 409; pauta não é eleição → endpoint não aplica.

---

## Referências

- Legado: `specs/requirements/assembly.md`, `cross-domain.md` Req 60.2-60.9, `novo escopo-7.md` §2.
- [[../global]] GR-033 (fração ideal), GR-035 (event-driven).
- [[../../01-domain/bounded-contexts#assembly]]
- [[../../00-product/portfolio-de-produtos]] Produto 5.
- [[../matrix-functional]] — Gestão Condominial, Votação.
- [[_moc]]
