---
title: "Sub-produto 9 — Compliance"
type: spec
tags: [sub-produto, compliance, lgpd, audit, c1-c11, master-sindico, v2, fase-8]
source: Development/Content/03-Modulos/Compliance-Detalhado.md + cross-domain.md Req 33-42 + institutional.md Req 11/34-37 + backend/internal/modules/institutional/ + regras-canonicas.md RC-10
created: 2026-04-23
updated: 2026-04-23
status: destilado
dual_check: pending
---

# Sub-produto 9 — Compliance

Camada transversal de blindagem documental, rastreabilidade, LGPD e dossiê auditável. **NÃO bloqueia onboarding**; bloqueia 4 gatilhos críticos (encerrar mandato, relatório final, negócio fechado, exportar dossiê). 11 telas (C1-C11) na Central de Governança do síndico, 1 Score Governança único (D-061), `lgpd_logs` append-only com retenção LGPD de 5 anos, adendos como único mecanismo de correção (Req 25).

---

## § 1. Fontes consultadas

| # | Fonte | Trecho/Linhas | Data |
|---|---|---|---|
| F1 | `Development/Content/03-Modulos/Compliance-Detalhado.md` | 1-285 (canônico principal — C1-C11) | 2026-04-22 |
| F2 | `Development/backend/.kiro/specs/master-sindico/requirements/cross-domain.md` | 16-227 (Req 33-42 Compliance) | stable 1.0 |
| F3 | `Development/backend/.kiro/specs/master-sindico/requirements/cross-domain.md` | 279-314 (Req 46-47 Gestão Mandato) | stable 1.0 |
| F4 | `Development/backend/.kiro/specs/master-sindico/requirements/institutional.md` | 242-294 (Req 11 Timeline + Req 12 Plano Diretor) | stable 1.0 |
| F5 | `Development/backend/.kiro/specs/master-sindico/requirements/institutional.md` | 16-114 (Req 6.2/6.3 Governance/Compliance markers) | stable 1.0 |
| F6 | `Development/backend/internal/modules/institutional/domain/entities/compliance.go` | 1-201 (aggregates reais) | código vivo |
| F7 | `Development/backend/internal/modules/institutional/domain/repositories/i_compliance_repository.go` | 1-23 (porta única ICompliance) | código vivo |
| F8 | `Development/backend/internal/modules/institutional/application/use_cases/compliance_use_cases.go` | 1-223 (4 UCs: SignTerm, SubmitDecl, CheckDeclCurrentYear, ListAudit) | código vivo |
| F9 | `Development/backend/internal/modules/institutional/application/use_cases/end_mandate_use_case.go` | 45-62 (gatilho bloqueia se declaração anual ausente) | código vivo |
| F10 | `Development/backend/internal/modules/institutional/infrastructure/database/queries/compliance.sql` | 1-33 (3 tabelas SQL: responsibility_term_signatures, annual_declarations, audit_log_entries) | código vivo |
| F11 | `Development/backend/internal/modules/institutional/infrastructure/http/routes/compliance_handler.go` | 1-124 (3 endpoints) | código vivo |
| F12 | `Development/backend/internal/modules/institutional/domain/enums/enums.go` | 1-327 (enums institucionais) | código vivo |
| F13 | `Development/Content/02-Jornadas/Inventario-Telas.md` | 269-285 (C1-C11 mapa) + 56-81 (S6 home, S28-S31 entrada) | 2026-04-22 |
| F14 | `Development/Content/regras-de-negocio/regras-canonicas.md` | 44 (Regra 10 LGPD by design) + 180 (AuditLogger pendente DT-010) | 2026-04-22 |
| F15 | `Development/backend/.kiro/specs/master-sindico/requirements/functional-matrix.md` | 95 (Compliance markers empresa toggleável) | stable |
| F16 | Inconsistência detectada: `Compliance-Detalhado.md:204` | "Score Governança 1-10 + Score Compliance 0-100" vs Req 33 "único 0-100" | Q25 quebras-detectadas |

---

## § 2. Propósito

Compliance é a **camada transversal de proteção jurídica + LGPD + memória probatória** que operacionaliza:

- **Cap. VII do Código Civil (arts. 1.347-1.356)** — deveres do síndico (prestar contas, convocar assembleia, defender interesses comuns)
- **Lei 13.709/2018 (LGPD)** — rastreabilidade de revelação de dados pessoais em Connect Me (Req 19)
- **Deveres fiduciários** — declarações, conflitos de interesse, trilha imutável de decisões
- **Transparência condominial** — dossiê exportável para assembleia, seguro, processo judicial

**Princípio operacional** (F1:15): *"o módulo NÃO bloqueia o cadastro inicial do condomínio. Mas determinados marcos exigem compliance ativo (4 gatilhos)."*

**Princípio documental** (F1:70): *"Compliance só fica 'Em dia' quando TODOS [os usuários] assinarem."*

---

## § 3. Diferencial

Compliance do Master Síndico **não é checklist estático** — é arquitetura viva que:

1. **Operacionaliza Cap. VII CC** — declarações, conflitos, mandato = obrigações legais digitalizadas com IP/timestamp/hash.
2. **Adendos vs edit** (Rule 2, F14:36) — correção sem apagar histórico; nova entry com link para original (`previous_entry_id`).
3. **Append-only enforcement** — DB-level via query pattern (nunca UPDATE/DELETE em `audit_log_entries`, `lgpd_logs`, `annual_declarations`, timeline).
4. **Hash SHA-256 do termo assinado** (F6:11-14) — integridade jurídica; nova versão de termo exige nova assinatura (F6:22-23).
5. **Score único transparente** (D-061, F2:20-48) — 0-100, 10 componentes, job noturno, histórico 12 meses, alerta < 60.
6. **LGPD by design** (F14:44) — `lgpd_logs` não é afterthought; é tabela central integrada a todos os módulos; direito ao esquecimento após 5 anos.

---

## § 4. 4 Gatilhos obrigatórios detalhados

Gatilhos que **bloqueiam** se compliance não estiver "em dia" (F1:234-243).

| Gatilho | Trigger técnico | Verifica | Código real | Mensagem ao usuário |
|---|---|---|---|---|
| **G1: encerrar_mandato** | `EndMandateUseCase.Execute` | Declaração anual do ano corrente submetida (por UserID + CondoID) | F9:55-61 `ErrBadRequest "declaração anual do ano corrente não submetida"` | "Para encerrar o mandato, submeta a declaração anual de {YEAR}." |
| **G2: gerar_relatorio_final** | `GenerateFinalReport` (pendente) | G1 + assinaturas de todos os usuários em dia + dossiê gerado | spec Req 40 F2:194-215 | "Para gerar relatório, complete gatilhos G1 + G3." |
| **G3: marcar_negocio_fechado** | `CloseDeal` (commercial) | Termos assinados + contrato anexado + comunicação pós-fechamento validada | F1:151-152, spec Req 25 adendo | "Para fechar negócio, complete termos pendentes (C8)." (opcional — F1:239) |
| **G4: exportar_dossie** | `ExportDossier` (pendente — C11) | Declaração anual + avaliação de gestão em dia + todos compliance markers | F1:209-230, Req 46 F2:293 "Bloqueio: não consegue encerrar se compliance atrasada" | "Para exportar dossiê, finalize pendências de compliance." |

**Mensagem padrão de bloqueio** (F1:242-243): *"Para prosseguir, finalize as pendências de compliance."*

**Regra de continuidade entre mandatos** (F1:247-252):
- Novo síndico assume → deve assinar nova declaração + atualizar conflito de interesse.
- Histórico anterior mantém-se visível (read-only).
- Governance Score reseta (novo ciclo — F2:311-312).

---

## § 5. 11 Telas C1-C11

Fonte primária: F1 (`Compliance-Detalhado.md`). Rotas derivadas de F11 (handler real) + convenção `/app/compliance/...`.

| ID | Nome | Rota proposta | Persona | Ações-chave | Fonte |
|---|---|---|---|---|---|
| **C1** | Painel de Compliance | `/app/compliance` | Síndico principal | Ver status geral (Em dia / Parcial / Pendente); listar pendências; 9 botões (Declaração, Assinaturas, Auditoria, Imutabilidade, Risco, Contratual, LGPD, Score, Dossiê) | F1:24-37 |
| **C2** | Declaração Anual do Síndico | `/app/compliance/annual-declaration` | Síndico principal | Ler texto jurídico versionado; checkbox + "Confirmar"; sistema registra `data, hora, IP, user, version`; publica imutável na Timeline tipo `administrativo → conformidade_e_governanca` | F1:40-57; spec F2:51-70 Req 34; código F11:74-102 POST |
| **C3** | Assinatura Obrigatória dos Usuários | `/app/compliance/signatures` | Síndico principal + Auxiliar + Administradora + Conselho | Listar usuários (até 3); ver status por usuário (Assinado/Pendente); enviar convite; Compliance só "Em dia" quando TODOS assinaram | F1:60-71; spec F2:73-91 Req 35; código F11:37-66 POST |
| **C4** | Matriz de Conflito de Interesse | `/app/compliance/conflicts` | Síndico principal | Declarar "Possuo ou não possuo relação direta ou indireta"; se Sim: empresa, natureza, condomínio impactado; registro imutável | F1:74-84; spec F2:95-107 Req 36 |
| **C5** | Trilha de Auditoria Visível | `/app/compliance/audit` | Síndico principal + Admin MS (only) | Listar eventos (user, perfil, ação, campo, ver_ant, ver_atual, data, hora); filtros; append-only; retenção 5 anos | F1:88-110; spec F2:111-142 Req 37; código F11:105-123 GET |
| **C6** | Imutabilidade e Adendos | `/app/compliance/amendments` | Síndico principal (cria) + todos (leitura) | Ver registros bloqueados (votações, negócios, declarações, eventos); botão "Criar adendo formal" com justificativa + descrição + anexo opcional; cria nova Timeline entry tipo `adendo` com link ao original | F1:114-128; spec Req 25 commercial + Req 11 Timeline F4:252 |
| **C7** | Mapa de Risco Consolidado | `/app/compliance/risk-map` | Síndico principal | Visualizar 8 tipos × 4 níveis; contagem alto/crítico; ações vencidas; atualização anual | F1:132-143; spec F2:146-166 Req 38 |
| **C8** | Conformidade Contratual | `/app/compliance/contracts` | Síndico principal + Auxiliar | Listar negócios fechados (valor, prazo início/fim, garantia, contrato anexado, termos assinados, comunicação validada); alertas garantia/prazo | F1:147-158 |
| **C9** | Registro LGPD | `/app/compliance/lgpd` | Síndico principal | Listar histórico Connect Me (empresa, data, finalidade, dados compartilhados, consentimento); imutável; texto fixo Lei 13.709/2018 | F1:162-175; spec F2:170-190 Req 39 |
| **C10** | Score Interno de Governança | `/app/compliance/score` | Síndico principal (privado) | Ver Score 0-100 (D-061); decomposição por 10 componentes; recomendações automáticas (regularizar assinaturas, classificar risco, atualizar ação, encerrar votação) | F1:179-204; spec F2:20-48 Req 33 |
| **C11** | Dossiê de Governança Exportável | `/app/compliance/dossier` | Síndico principal (gera) + Admin MS (valida assinatura) | Botão "Gerar Dossiê"; compila declarações + assinaturas + conflito + risco + trilha + contratações + termos + LGPD + indicadores; PDF assinado digitalmente; armazena MinIO permanente | F1:208-230; spec F2:194-214 Req 40 |

**Entrada na Central de Governança** (F13:56): S6 home card #9 "Compliance"; S28-S31 = aliases antigos para C1, C2, C3, C5.

---

## § 6. Personas × role

| Persona | Role Zitadel | Capacidade em Compliance | Restrições |
|---|---|---|---|
| **Síndico principal** | `syndic` + role_type `principal` | Emissor: assina termos (C3), submete declaração anual (C2), declara conflitos (C4), cria adendos (C6); leitor+ator em todas as 11 telas | 1 principal/condomínio (institutional Req 10) |
| **Síndico auxiliar** | `syndic` + role_type `auxiliary` | Assina termo (C3); leitura C1/C5/C8; não submete declaração anual (exclusiva do principal) | Até 2 auxiliares/condomínio |
| **Assistente** | `syndic` + role_type `assistant` | Assina termo (C3); leitura C1 painel geral | Sem poder de voto (Req 10); não cria adendos |
| **Administradora vinculada** | `syndic` via empresa síndica | Assina termo (C3); audit log das ações; token por email | Escopo conforme permissões configuráveis (institutional Req 7) |
| **Conselho** | `resident` convidado | Assina termo (C3) — "Declaro ciência que minha atuação é rastreável" (F1:66-67) | Leitura C5 limitada; sem C2/C4/C10/C11 |
| **Morador** | `resident` | Leitura C9 (seus próprios dados LGPD); sem acesso a C1-C8/C10-C11 | Direito ao esquecimento (LGPD — Req 39) |
| **Admin MS** | `admin` | Moderador global; acessa C5 (trilha) + C11 (valida assinatura digital do dossiê); audit log de cross-tenant access | Admin via `apps/admin` dedicada (D-134 revoga D-058/D-072) |
| **Empresa / Parceiro** | `enterprise` / `local_company` | **N/A em Compliance condominial**; próprios Compliance markers (25, autodeclarados, F5:73) em perfil (Req 6.3) | Autodeclarado com disclaimer obrigatório |

---

## § 7. Tiers × role (matriz)

Planos globais Stripe-style: `trial | base | plus | pro` (D-032). Compliance é **transversal a todos os tiers** — não é feature paga.

| Tela | Síndico `trial` (15d) | Síndico `base` | Síndico `plus` | Síndico `pro` | Admin MS |
|---|---|---|---|---|---|
| C1 Painel | ✅ | ✅ | ✅ | ✅ | ✅ |
| C2 Declaração anual | ✅ | ✅ | ✅ | ✅ | ❌ (read) |
| C3 Assinaturas | ✅ | ✅ | ✅ | ✅ | ❌ |
| C4 Conflitos | ✅ | ✅ | ✅ | ✅ | ❌ |
| C5 Auditoria | ✅ (próprio condo) | ✅ (próprio) | ✅ (próprio) | ✅ (próprio) | ✅ (global) |
| C6 Adendos | ✅ | ✅ | ✅ | ✅ | ❌ |
| C7 Mapa risco | ✅ (básico) | ✅ (básico) | ✅ (completo) | ✅ (completo + export) | ❌ |
| C8 Contratual | ⚠️ (limitado) | ✅ | ✅ | ✅ | ❌ |
| C9 LGPD | ✅ | ✅ | ✅ | ✅ | ✅ (global read) |
| C10 Score | ✅ (read) | ✅ | ✅ (+ recomendações) | ✅ (+ recomendações + histórico 12m) | ✅ (read global) |
| C11 Dossiê | ❌ (bloqueado) | ✅ (1 dossiê/mandato) | ✅ (ilimitado) | ✅ (ilimitado + assinatura digital) | ✅ (valida) |

**Justificativa `plan_tier=pro` para Síndico**: histórico 12 meses de Score (F2:40) + dossiê ilimitado + assinatura digital avançada são diferenciais do tier mais alto. (Fonte F2:40 usa label legado "N3 Consolidar" banido por D-067; RC-4 citada lá refere-se a tier pro do Síndico, não deve ser confundida com a RC-4 canônica em `regras-canonicas.md` que trata de visibilidade de vídeo.)

---

## § 8. Aggregates

### § 8.1 ResponsibilityTermSignature (F6:24-77)

Assinatura imutável de um termo pelo usuário.

| Campo | Tipo | Regra | Fonte |
|---|---|---|---|
| `id` | string (ULID/UUID) | PK | F6:25 |
| `condominium_id` | string | FK tenant | F6:26 |
| `user_id` | string | FK identity | F6:27 |
| `term_version` | string | 1-32 chars; `ErrInvalidTermVersion` | F6:11, F6:38-40 |
| `term_hash` | string | SHA-256 hex (64 chars); `ErrInvalidTermHash`; lowercase normalizado | F6:12, F6:41-43, F6:79-85 |
| `signed_ip` | string | 3-45 chars (IPv4/IPv6); `ErrInvalidSignedIP` | F6:13, F6:44-46 |
| `signed_user_agent` | string | header HTTP | F6:31 |
| `signed_at` | time.Time | `time.Now()` no New | F6:55, F6:32 |

**Invariantes** (F7:13-14):
- Factory `NewResponsibilityTermSignature` valida e cria.
- `ReconstructResponsibilityTermSignature` para hidratação do repo (sem validação).
- Getters somente (private fields). Sem setters.
- Nova versão de termo → nova assinatura (não atualizar). Integridade jurídica.

### § 8.2 AnnualDeclaration (F6:90-140)

Declaração anual do síndico — bloqueia encerramento de mandato se ausente do ano corrente.

| Campo | Tipo | Regra | Fonte |
|---|---|---|---|
| `id` | string | PK | F6:91 |
| `condominium_id` | string | FK | F6:92 |
| `user_id` | string | FK (síndico principal) | F6:93 |
| `declaration_year` | int | 2020-2100; `ErrInvalidDeclYear` | F6:14, F6:105-107 |
| `declaration_text` | string | 10-20000 chars; `ErrInvalidDeclText` | F6:15, F6:108-110 |
| `signed_ip` | string | 3-45 chars | F6:111-113 |
| `submitted_at` | time.Time | `time.Now()` | F6:117 |

**Invariantes**:
- 1 declaração por `(condominium_id, user_id, declaration_year)` — UNIQUE composto.
- Bloqueia `EndMandateUseCase` se ano corrente não existe (F9:55-62).
- Retenção 5 anos (F2:65).

### § 8.3 AuditLogEntry (F6:146-200)

Entry imutável do audit trail — retenção 5 anos (LGPD) via job, aplicação nunca DELETE/UPDATE.

| Campo | Tipo | Regra | Fonte |
|---|---|---|---|
| `id` | string | PK | F6:147 |
| `actor_id` | string | "" se sistêmico | F6:148 |
| `actor_ip` | string | opcional | F6:149 |
| `action` | string | 2-100 chars; `ErrInvalidAuditAction` | F6:16, F6:162-164 |
| `target_type` | string | 2-100 chars; `ErrInvalidAuditTarget` | F6:17, F6:165-167 |
| `target_id` | string | resource FK | F6:152 |
| `condominium_id` | string | tenant scope | F6:153 |
| `payload` | []byte | JSONB; default `{}` se vazio | F6:154, F6:168-170 |
| `created_at` | time.Time | `time.Now()` | F6:155 |

**Invariantes** (F2:131-136):
- Append-only: nunca UPDATE, nunca DELETE.
- Retenção 5 anos via compactação MinIO S3 Glacier após 1 ano (F2:141).
- Índices: por usuário, data, ação (F2:135).
- Sigilo: apenas `admin` + `syndic principal` podem acessar (F1:110, F2:136).

### § 8.4 Compliance (composite VO — não implementado ainda)

Estado agregado para UI C1 (Painel).

| Campo | Tipo | Derivação |
|---|---|---|
| `overall_status` | enum `em_dia / parcial / pendente` | (F1:27) função das sub-verificações |
| `annual_declaration_pending` | bool | Não existe registro para `year = current` |
| `users_signatures_pending` | int | Count de usuários sem `ResponsibilityTermSignature` |
| `unclosed_deals_without_terms` | int | Count de deals `closed` sem `terms_signed=true` |
| `actions_without_risk_classification` | int | Count de Plano Diretor activities sem `risk_level` |
| `mandate_near_expiry` | bool | `mandate.end_date - today < 90d` |

### § 8.5 ConflictOfInterestDeclaration (spec F2:95-108 — não implementado)

| Campo | Tipo | Regra |
|---|---|---|
| `id` | string | PK |
| `user_id` | string | FK síndico |
| `condominium_id` | string | FK |
| `has_conflict` | bool | Sim/Não |
| `vendor_company_id` | string | FK se `has_conflict=true` |
| `nature_of_relationship` | string | descrição livre |
| `declared_at` | time.Time | timestamp |
| `is_immutable` | bool | `true` após submit |

**Invariante** (F2:102-103): votação em pauta com conflito do síndico → marcada como "abstém obrigatória".

### § 8.6 RiskAssessment (spec F2:146-165 — não implementado)

Mapa de riscos anual, integrado com Master Plan (Req 12).

| Campo | Tipo | Regra |
|---|---|---|
| `id` | string | PK |
| `condominium_id` | string | FK |
| `risk_type` | enum | 8 tipos (§ 9) |
| `risk_level` | enum | baixo / médio / alto / crítico (§ 9) |
| `description` | string | contexto |
| `mitigation_plan` | string | ação proposta |
| `due_date` | time.Time | prazo |
| `status` | enum | aberto / em_andamento / mitigado / aceito |

### § 8.7 LGPDLog (spec F2:170-190 — `lgpd_logs` central)

| Campo | Tipo | Regra |
|---|---|---|
| `id` | string | PK |
| `event_type` | enum | `data_revealed / cross_tenant_access / export / delete` |
| `source_id` | string | quem revelou (user_id ou syndic_id) |
| `target_id` | string | quem recebeu (company_id ou resident_id) |
| `condominium_id` | string | tenant scope |
| `ip_address` | string | IPv4/IPv6 |
| `purpose` | string | "Orçamento de serviço", "Assembleia ata", etc. |
| `terms_version` | string | versão aceita pelo target |
| `data_shared` | []byte | JSONB; campos compartilhados (hashed/pseudonimizado) |
| `created_at` | time.Time | UTC |

**Invariante**: append-only, 5 anos, purga após vencimento via direito ao esquecimento (F2:186-187).

### § 8.8 ScoreGovernanca (D-061 — único)

Ver § 10.

---

## § 9. Enums

Dedicados ao Compliance (proposta — nem todos implementados em F12 ainda).

### § 9.1 ComplianceStatus (3)

```
em_dia | parcial | pendente
```

Fonte: F1:27. **Invariante**: `em_dia` requer TODOS: declaração anual ok + assinaturas todas + conflitos atualizados + ações críticas classificadas + mandato > 90d do fim.

### § 9.2 ComplianceRiskType (8)

Consolidação F2:150-156 (5 spec) + complemento F1:146-158 (3 operacionais).

| # | Enum | Exemplo |
|---|---|---|
| 1 | `financeiro` | Falta de fundo de reserva |
| 2 | `compliance` | Atrasos em declarações anuais |
| 3 | `estrutural` | Problemas prediais (laudo vencido) |
| 4 | `social` | Conflitos entre moradores |
| 5 | `seguranca` | Vulnerabilidades físicas/digitais |
| 6 | `contratual` | Prazo de garantia/execução vencido (F1:154-155) |
| 7 | `legal` | Processo judicial, notificação |
| 8 | `operacional` | Manutenção programada atrasada |

### § 9.3 ComplianceRiskLevel (4)

```
baixo | medio | alto | critico
```

Fonte: F1:141. UI renderiza cores distintas. `critico` dispara alerta + highlight.

### § 9.4 ComplianceCheckStatus (3)

```
pending | passed | failed
```

Fonte: proposta — usado por job noturno de Score.

### § 9.5 SignatureStatus (2)

```
pendente | assinado
```

Fonte: F1:64. Binário — ou assinou, ou não.

### § 9.6 AuditActionType (proposto — taxonomia de F6:162 "action")

Taxonomia operacional (strings livres hoje, candidatos a enum):

```
sign_responsibility_term | submit_annual_declaration | create_adendo
| close_deal | publish_communication | validate_execution
| login | logout | grant_permission | revoke_permission
| soft_delete | cross_tenant_reveal | export_dossier
| update_master_plan | vote_cast | homologate_assembly
```

(Ver F6:73 uso real de `"sign_responsibility_term"`, F6:141 `"submit_annual_declaration"`.)

### § 9.7 LGPDEventType (4)

```
data_revealed | cross_tenant_access | export | delete
```

Fonte: proposta consolidada Req 39. `data_revealed` = Connect Me form completed; `cross_tenant_access` = admin MS vendo outro tenant; `export` = dossiê/CSV; `delete` = direito ao esquecimento.

---

## § 10. Score Governança único (D-061)

**Decisão arquitetural**: 1 Score único Governança 0-100 (resolve Q25 quebras-detectadas). Fonte de verdade: Req 33 (F2:20-48). Supera texto legado de F1:204 "Score Governança 1-10 + Score Compliance 0-100 (dois scores)".

### § 10.1 Fórmula

```
Score = Σ (componente_i × peso_i) / Σ pesos
      = média ponderada de 10 componentes × 10pt = 0-100
```

Cálculo **automático diário** via job noturno (F2:39); aplicação **nunca** faz UPDATE manual.

### § 10.2 Inputs e pesos (F2:26-35)

| # | Componente | Peso | Input técnico | Cálculo |
|---|---|---|---|---|
| 1 | LGPD Compliance | 10 | `lgpd_logs` sem vazamentos 12m | 0 logs de vazamento = 10pt; 1+ = proporcional |
| 2 | Declarações em dia | 10 | `annual_declarations` ano corrente existe | Existe = 10pt; ausente = 0pt |
| 3 | Assinaturas obrigatórias | 10 | Count `responsibility_term_signatures` / total users | % × 10 |
| 4 | Trilha de auditoria | 10 | `audit_log_entries` ≥ N eventos/mês | Threshold-based |
| 5 | Sem reclamações formais | 10 | Count reclamações ativas (commercial harassment) | 0 = 10pt; escala descendente |
| 6 | Fundo de Reserva em dia | 10 | Billing: contribuições registradas > threshold | % do mínimo legal |
| 7 | Assembleias regularizadas | 10 | Assembly: homologadas / programadas | Taxa × 10 |
| 8 | Sem atrasos em rotinas | 10 | Timeline: frequência + comunicados regulares | Heurística |
| 9 | Avaliação de gestão | 10 | Req 15: score médio escalar 1-10 | Normalizar para 0-10 |
| 10 | Política interna | 10 | Regimento documentado + público | Boolean × 10 |

**Total**: 10 × 10 = 100.

### § 10.3 Visibilidade e alertas

- **Visibilidade**: público (F2:41 "transparência condominial") — **mudança vs F1:181** "Privado". Decisão: spec viva vence.
- **Histórico**: 12 meses (F2:40) — tabela `governance_score_history`.
- **Alerta**: se Score < 60 → notificação ao síndico principal (F2:42) + recomendações automáticas em C10 (F1:196-201).

### § 10.4 Recomendações automáticas (F1:196-201)

Gatilho de recomendação por componente falho:
- Componente 2 falho → "Regularizar declaração anual"
- Componente 3 falho → "Enviar lembrete de assinatura aos usuários pendentes"
- Componente 4/8 falho → "Classificar risco pendente em Plano Diretor"
- Componente 7 falho → "Homologar assembleia {X}" ou "Encerrar votação aberta"

### § 10.5 Tabelas

Proposta (não implementadas ainda):

```sql
governance_scores (
    condominium_id, computed_at, overall_score,
    component_1 ... component_10, payload JSONB
)
governance_score_history (
    condominium_id, month, score, delta
)
```

---

## § 11. Risk Map 8×4 (matriz)

Matriz consolidada proposta para C7 (F1:132-143 + F2:146-166).

| Tipo \ Nível | Baixo (1-3pt) | Médio (4-6pt) | Alto (7-8pt) | Crítico (9-10pt) |
|---|---|---|---|---|
| **Financeiro** | Fundo > 20% | Fundo 10-20% | Fundo 5-10% | Fundo < 5% / inadimplência > 30% |
| **Compliance** | Declaração em dia | 1 assinatura pendente | Declaração vencida < 30d | Declaração vencida > 30d + score < 40 |
| **Estrutural** | Laudos em dia | 1 laudo a vencer 90d | Laudo vencido < 30d | Laudo vencido > 30d / falha estrutural relatada |
| **Social** | Sem reclamações | 1-2 reclamações abertas | 3+ reclamações / conflito assembleia | Processo judicial ativo |
| **Segurança** | Sistemas ok | Câmera/alarme falho | Arrombamento recente | Incidente grave (roubo, agressão) |
| **Contratual** | Garantias vigentes | 1 garantia vence 30d | Garantia vencida | Contratação sem contrato formal |
| **Legal** | Sem notificações | Notificação administrativa | Processo cível | Processo criminal contra gestão |
| **Operacional** | Manutenção ok | 1 manutenção atrasada | Múltiplas atrasadas | Sistema crítico parado (elevador, bomba) |

**Consolidação C7** (F1:137-141): Distribuição por categoria, quantidade alto/crítico, ações vencidas. **Indicador visual** de cor (verde/amarelo/laranja/vermelho).

---

## § 12. Adendos (C6)

### § 12.1 Princípio

**Rule 2** (F14:36): *"Imutabilidade — Timeline, Negócio Fechado, votos homologados nunca editados. Único mecanismo de mudança: adendo (Req 25)."*

### § 12.2 Registros bloqueados (F1:116-120)

- Votações encerradas (Assembly)
- Negócios fechados (Commercial)
- Declarações anuais (Compliance)
- Eventos encerrados (Institutional)

### § 12.3 Mecanismo técnico

Adendo = **nova** TimelineEntry tipo `adendo` (F12:155) com `previous_entry_id` apontando ao original (F4:267 "Campo `previous_entry_id` para adendos (chain de modificações)").

| Campo | Tipo | Regra |
|---|---|---|
| `id` | string | PK nova entry |
| `entry_type` | enum | fixo `"adendo"` |
| `previous_entry_id` | string | FK à entry original (chain) |
| `justification` | string | obrigatório, 10-2000 chars |
| `description` | string | voz ou texto, 10-5000 chars |
| `attachment_url` | string | opcional (MinIO) |
| `author_id` | string | FK identity |
| `created_at` | time.Time | imutável |

**Invariante**: adendo **nunca altera** a entry original. Cria registro novo, preserva histórico. Chain formada por `previous_entry_id` permite rastrear toda cadeia de correções.

### § 12.4 Req 25 (Commercial)

F1:264 confirma: *"C6 (Adendos) ← requirements/commercial.md Req 25 + requirements/institutional.md Req 11."*

Adendo em Commercial: correção de negócio fechado (ex: prazo ajustado, garantia estendida). Mesma mecânica — nova entry, nunca edit.

---

## § 13. LGPD audit (C9)

### § 13.1 Princípios (F14:44 RC-10)

> "LGPD by design — log obrigatório de revelação de dados pessoais. `lgpd_logs` (timestamp, IP, finalidade, versão dos termos). Append-only, retenção 5 anos."

### § 13.2 Eventos rastreados (F2:170-190)

| Evento | Trigger | Exemplo |
|---|---|---|
| `data_revealed` | Síndico revela CPF/telefone a empresa em Connect Me (Req 19) | `{source: syndic_123, target: enterprise_456, purpose: "orçamento", data_shared: ["cpf", "phone"]}` |
| `cross_tenant_access` | Admin MS acessa dados de outro tenant | `{source: admin_ms, target_tenant: condo_789, purpose: "moderation"}` |
| `export` | Síndico exporta dossiê (C11) ou CSV | `{source: syndic_123, target: file_abc, purpose: "assembleia_ordinaria"}` |
| `delete` | Direito ao esquecimento executado | `{source: system, target: resident_999, purpose: "lgpd_right_to_be_forgotten"}` |

### § 13.3 Schema (F2:126-130 audit_trail + extensão LGPD)

```sql
CREATE TABLE lgpd_logs (
    id UUID PRIMARY KEY,
    event_type lgpd_event_type NOT NULL,
    source_id UUID NOT NULL,
    target_id UUID,
    condominium_id UUID,
    ip_address VARCHAR(45) NOT NULL,
    purpose VARCHAR(500) NOT NULL,
    terms_version VARCHAR(32) NOT NULL,
    data_shared JSONB DEFAULT '{}',
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
    -- Sem UPDATE/DELETE — append-only DB-enforced
);
CREATE INDEX idx_lgpd_source ON lgpd_logs (source_id, created_at DESC);
CREATE INDEX idx_lgpd_target ON lgpd_logs (target_id, created_at DESC);
CREATE INDEX idx_lgpd_event ON lgpd_logs (event_type, created_at DESC);
```

### § 13.4 Retenção

- **5 anos ativo**: busca imediata no PG.
- **Após 1 ano**: archive MinIO (S3 Glacier) — F2:142.
- **Após 5 anos**: purga física via job `lgpd_forget` (direito ao esquecimento — F2:186-187).
- **Direito do titular**: solicita purga antecipada → justificativa + aprovação admin MS → entry `delete` em `lgpd_logs` antes de purgar.

### § 13.5 Texto fixo (F1:168)

> *"Os dados foram compartilhados mediante autorização expressa para finalidade específica, conforme Lei 13.709/2018."*

---

## § 14. Dossiê Exportável (C11)

### § 14.1 Conteúdo compilado (F1:212-221)

| Seção | Fonte aggregada |
|---|---|
| 1. Declarações anuais | `annual_declarations` todas do mandato |
| 2. Assinaturas dos usuários | `responsibility_term_signatures` + timestamp + IP |
| 3. Matriz de conflito | `conflict_of_interest_declarations` |
| 4. Mapa de risco | `risk_assessments` com tipo/nível/mitigação |
| 5. Trilha de auditoria (resumo) | `audit_log_entries` sumarizadas por ação |
| 6. Contratações | Commercial deals fechados |
| 7. Termos assinados | Contratos anexados + validação pós-fechamento |
| 8. LGPD (resumo) | `lgpd_logs` contagem por evento |
| 9. Indicadores | Score Governança + histórico 12m |

### § 14.2 Formato e assinatura digital

- **PDF estruturado** (F1:224) com índice clicável.
- **Assinatura digital**: síndico + admin MS (F2:203). ICP-Brasil compatível (Sprint futura).
- **Armazenamento**: MinIO permanente (F2:209).
- **Geração**: job async disparado por `mandates.end_date` atingida (F2:213).

### § 14.3 Uso (F1:227-230)

- Assembleia (prestação de contas)
- Processo judicial (defesa)
- Auditoria (externa ou contratual)
- Seguro (sinistro de RC)

### § 14.4 Requisitos MUST (F2:206-210)

- Geração automática no encerramento do mandato (G1 passou).
- Armazenamento MinIO permanente.
- Acesso: síndico pode baixar (histórico preservado).

---

## § 15. Regras + invariantes

### § 15.1 Regras canônicas (F14 + F1)

| # | Regra | Fonte |
|---|---|---|
| R-COMP-1 | Compliance NÃO bloqueia onboarding inicial | F1:15 |
| R-COMP-2 | Compliance "Em dia" requer TODOS usuários assinados | F1:70 |
| R-COMP-3 | Declaração anual: 1x/ano OU a cada novo mandato | F1:42, F2:63 |
| R-COMP-4 | Adendo é o único mecanismo de correção pós-publicação | F14:36, F4:252-253 |
| R-COMP-5 | `lgpd_logs` append-only; retenção 5 anos; archive 1 ano | F14:44, F2:141-142 |
| R-COMP-6 | Audit trail: apenas admin MS + síndico principal acessam | F1:110, F2:136 |
| R-COMP-7 | Score Governança único 0-100 (D-061) supera F1:204 Q25 | Req 33 F2:20 |
| R-COMP-8 | Nova versão de termo exige nova assinatura (integridade hash) | F6:22-23 |
| R-COMP-9 | Conflito declarado impede voto do síndico (abstém obrigatória) | F2:102-103 |
| R-COMP-10 | Dossiê gerado só após encerramento bem-sucedido do mandato | F2:213 |
| R-COMP-11 | Compliance markers empresa (25) e governance markers síndico (15) são autodeclarados + disclaimer | F2:221, F5:38 |
| R-COMP-12 | Direito ao esquecimento: purga só após 5 anos do log | F2:186-187 |

### § 15.2 Invariantes técnicas

| ID | Invariante | Enforcement |
|---|---|---|
| INV-COMP-1 | `audit_log_entries`: nunca UPDATE, nunca DELETE | Aplicação (repo sem método); DB-level policy futura |
| INV-COMP-2 | `annual_declarations`: UNIQUE (condo, user, year) | Constraint DB |
| INV-COMP-3 | `responsibility_term_signatures.term_hash` = SHA-256 (64 hex) | Validação entity (F6:79-85) |
| INV-COMP-4 | `signed_ip`: 3-45 chars (IPv4/IPv6) | Validação entity (F6:44-46) |
| INV-COMP-5 | `declaration_year`: 2020-2100 | Validação entity (F6:105-107) |
| INV-COMP-6 | `lgpd_logs` append-only | Aplicação + DB revoke UPDATE/DELETE na role |
| INV-COMP-7 | Score Governança: cálculo batch noturno; UI lê `governance_scores` | Job scheduler |
| INV-COMP-8 | Timeline `adendo` exige `previous_entry_id != null` | Validação entity Timeline |
| INV-COMP-9 | Encerramento mandato bloqueia se declaração ano corrente ausente | F9:55-62 use case policy |
| INV-COMP-10 | Admin MS cross-tenant access → auto-gera entry em `lgpd_logs` tipo `cross_tenant_access` | Middleware |

---

## § 16. State machines

### § 16.1 ComplianceStatus (condomínio)

```
pendente ──┬─> parcial ──> em_dia
           │                  │
           └<──────────────────┘ (qualquer falha volta a pendente/parcial)

Transições:
- pendente → parcial: declaração anual submetida mas assinaturas incompletas
- parcial → em_dia: TODOS requisitos satisfeitos
- em_dia → parcial: 1 usuário revoga ou novo usuário não assinou
- parcial → pendente: declaração anual vencida
```

Fonte: F1:27 "Em dia / Parcial / Pendente" + F1:70 "Compliance só 'Em dia' quando TODOS assinarem".

### § 16.2 SignatureStatus (por usuário × termo)

```
pendente ──> assinado (irreversível para mesma versão)

Nova versão de termo:
assinado (v1) + [new_version (v2)] ──> pendente (v2) ──> assinado (v2)
```

Fonte: F1:64, F6:22-23.

### § 16.3 AnnualDeclaration (por síndico × ano)

```
ausente ──> submitted (via SubmitAnnualDeclarationUseCase)

Nunca volta. Cada ano é independente:
year=2026: ausente → submitted
year=2027: ausente → submitted
```

Fonte: F6:100-119, F8:117-162.

### § 16.4 ConflictDeclaration (por síndico × mandato)

```
não_declarado ──> declarado_sem_conflito ──[anual update]──> declarado_sem_conflito (v2)
                \                             
                 declarado_com_conflito ──[gatilho votação]──> votação abstém obrigatória
```

Fonte: F1:74-84, F2:102-103.

### § 16.5 RiskAssessment (por item de risco)

```
aberto ──> em_andamento ──┬─> mitigado (terminal)
                          └─> aceito (terminal, com justificativa)

aberto ──[vencido prazo]──> crítico (alerta)
```

### § 16.6 DossierExport (C11)

```
pendente ──[síndico clica Gerar]──> em_geração ──[job async]──┬─> pronto (PDF MinIO + assinatura)
                                                               └─> falha (alerta admin)
                                                               
gatilhos bloqueiam se G4 não satisfeito → pendente congelado
```

---

## § 17. Domain events

Eventos publicados via `IEventPublisher` (F15:44-47) + NATS JetStream para cross-module.

| Evento | Quando | Payload | Consumidores |
|---|---|---|---|
| `compliance.term.signed` | `SignResponsibilityTermUseCase.Execute` ok | `{signature_id, condo_id, user_id, term_version, term_hash, signed_at}` | Score recompute, timeline auto-publish |
| `compliance.annual_declaration.submitted` | `SubmitAnnualDeclarationUseCase.Execute` ok | `{decl_id, condo_id, user_id, year, submitted_at}` | Score recompute, timeline publish, mandate unlock |
| `compliance.audit.appended` | `AppendAudit` qualquer origem | `{entry_id, actor_id, action, target_type, target_id, condo_id}` | Search indexer, LGPD correlation |
| `compliance.lgpd.data_revealed` | Connect Me form completed | `{source_id, target_id, condo_id, purpose, data_shared, terms_version}` | Score recompute (componente 1), resident notification |
| `compliance.conflict.declared` | `DeclareConflictUseCase` ok | `{decl_id, syndic_id, has_conflict, vendor_id}` | Assembly abstain-required flag |
| `compliance.risk.updated` | Risk map atualizado | `{risk_id, type, level_before, level_after}` | Alert service, score recompute |
| `compliance.dossier.generated` | Job async completa | `{dossier_id, mandate_id, signed_url, expires_at}` | Admin MS notification |
| `compliance.mandate.blocked` | G1-G4 falha | `{gatilho, reason, pending_items[]}` | UI toast + admin audit |
| `compliance.score.computed` | Job noturno | `{condo_id, score, delta_vs_yesterday, components}` | Alert se < 60, dashboard sync |

**Padrão de nomeação**: `<bc>.<aggregate>.<verb-past-tense>` consistente com F15:11-13.

---

## § 18. Dependências

| Consumidor → Produtor | Relação | Artefato |
|---|---|---|
| Compliance → Institutional Timeline | Publica entries imutáveis (declaração, assinatura) e recebe adendos | Req 11 timeline_entries + `previous_entry_id` |
| Compliance → Content moderação | Comunicados que viram audit event (publish_communication) | Req 14 communications + audit logger |
| Compliance → Commercial HarassmentReport | Reclamações formais são input para Score componente 5 | commercial.harassment_reports |
| Compliance → Commercial Deals | Negócios fechados sem termos bloqueiam G3 | commercial.deals (C8 alimenta) |
| Compliance → Assembly Ata | Homologações alimentam Score componente 7; atas vão para Dossiê | assembly.assembly_results + Req 58 |
| Compliance ← Identity | user_id + role_type para validar quem pode assinar/declarar | identity.users + role_types |
| Compliance ← Billing | Fundo de reserva para Score componente 6 | billing.payments + reserve_fund |
| Compliance ← Connect Me | Disparo de `lgpd_logs` tipo `data_revealed` | commercial.connect_me_forms Req 19 |
| Compliance ← Master Plan | Risk level integrado (ações classificadas por risco) | Req 12 master_plan_risks |
| Compliance → LMS/Academy | Certificados de compliance (futuro — Sprint) | content.courses + certifications |
| Compliance ↔ AuditLogger (DT-010) | Pendência: adapter cross-module centralizado | F14:180, F1:274 Sprint 10 |

---

## § 19. Markers

### § 19.1 Governance markers síndico (15 autodeclarados — F5:34-38)

Checkbox list em perfil do síndico (Req 6.2). Disclaimer obrigatório.

| # | Marker | Descrição |
|---|---|---|
| 1 | `membro_abracs` | Associação Brasileira dos Condomínios, Administradoras e Síndicos |
| 2 | `seguro_responsabilidade_civil` | Apólice RC síndico vigente |
| 3 | `assessoria_juridica` | Contrato com escritório ou advogado interno |
| 4 | `assessoria_contabil` | Contabilidade terceirizada ou interna |
| 5 | `assessoria_seguranca_trabalho` | Engenharia de segurança contratada |
| 6 | `conformidade_lgpd` | Processos internos LGPD documentados |
| 7 | `conformidade_nr1` | NR-1 (gerenciamento de riscos ocupacionais) |
| 8 | `programa_compliance` | Programa formal de integridade |
| 9 | `selo_reclame_aqui` | Selo RA1000 ou equivalente |
| 10 | `outras_certificacoes` | Catch-all |
| 11 | `premiacoes` | Prêmios setoriais |
| 12-15 | (extensão prevista — F5:36 "etc." implicito) | Espaço para 4 markers disponíveis em `plan_tier >= plus` |

**Disclaimer** (F5:38): *"Marcadores são informativos, baseados em autodeclaração, e não representam certificação oficial da plataforma."*

### § 19.2 Compliance markers empresa (25 autodeclarados — F5:73-75)

Perfil da empresa (Req 6.3).

| Categoria | Markers |
|---|---|
| **ISOs** (5) | `iso_9001` (qualidade), `iso_14001` (ambiental), `iso_45001` (saúde/segurança ocupacional), `iso_37001` (antissuborno), `iso_37301` (compliance) |
| **ESG** (3) | `esg_environmental`, `esg_social`, `esg_governance` |
| **Selos** (4) | `selo_empresa_cidada`, `selo_grande_lugar_para_trabalhar` (GPTW), `selo_pro_etica` (CGU), `selo_b_corp` |
| **Certificações setoriais** (5) | `pbqp_h` (construção), `abnt_certified`, `pmo_certified`, `itil_certified`, `six_sigma` |
| **NRs** (5) | `nr_1` (GRO), `nr_5` (CIPA), `nr_6` (EPI), `nr_17` (ergonomia), `nr_35` (altura) |
| **Outros** (3) | `ca_epis_validos` (CA vigente), `crea_cau_ativo`, `outras_certificacoes` |

**Visibilidade** (F5:88): Base visível só para Síndico; Plus/Pro visível para todos. **Toggleável** (F15:95).

---

## § 19.5 Telas & Endpoints

Rastreabilidade consolidada em [[../../03-subprojects/traceability]] §2.6 (Compliance) + §3.6 (matriz inversa).

**Escopo UI**:
- Compliance síndico: C1-C11 (11 telas dedicadas) + S28-S31 (4 telas overlay na jornada síndico)
- Empresa: E8 (termo execução) · E15 (compliance anual)
- Total: **15 telas web**, 0 mobile

**Endpoints-chave**:
- Painel: `/api/v1/compliance/dashboard` · `/pendencies`
- Declaração anual: `/compliance/declaration/template` · `/status` · POST `/compliance/declaration` · `/condominiums/{id}/compliance/annual-declaration`
- Termo responsabilidade: `/compliance/responsibility-term/current` · `/condominiums/{id}/compliance/responsibility-term/sign`
- Assinaturas: `/compliance/signatures/pending` · `/:id/sign`
- Conflito de interesse: `/compliance/conflicts` · `/declare`
- Audit trail: `/compliance/audit-trail` · `/filters` · `/export` · `/condominiums/{id}/compliance/audit`
- Adendos: `/compliance/amendments` POST/GET
- Mapa de risco: `/compliance/risk-map`
- Contratos: `/compliance/contracts`
- LGPD logs: `/compliance/lgpd-logs` · `/{id}` · `/export`
- Dossiê: `/compliance/dossier`
- Score interno: `/condominiums/{id}/governance-score/internal` — 🔴 **gap: endpoint ausente** (C10)

**Invariantes observáveis em UI**: INV-033 (audit append-only) · INV-034 (declaração anual UNIQUE ano) · INV-035 (conflito obrigatório onboarding) · INV-090 (audit DB level) · INV-091 (retenção 5a) · INV-109 (LGPD HMAC keyed rotation) · INV-120 (audit absolutamente imutável) · INV-119 (timeline/ata DB-level).

**4 gatilhos bloqueantes** (mensagem padrão: "Para prosseguir, finalize as pendências de compliance."):
1. Encerrar mandato (S2 `end-mandate` chama compliance summary)
2. Gerar relatório final
3. Marcar negócio fechado (opcional)
4. Exportar dossiê (C11)

**Gaps críticos**: C10 endpoint ausente (governance score 1-10 vs 0-100 pendente D-061) · ICP-Brasil signature provider sem ADR (M3+) · Glacier archive política sem ADR.

**Sprints**: Sprint 3 ✅ backend (audit trail + termo anual + LGPD logs + amendments) · UI Sprint 10 🟡 (C1-C5 bloqueadores M1 parcial).

---

## § 20. Estado no código + gaps + referências

### § 20.1 Estado implementado (verificado em F6-F11)

| Artefato | Status | Sprint |
|---|---|---|
| `ResponsibilityTermSignature` entity + validações | ✅ implementado | Sprint 2-3 institutional |
| `AnnualDeclaration` entity + validações | ✅ implementado | Sprint 2-3 |
| `AuditLogEntry` entity + append-only pattern | ✅ implementado | Sprint 2-3 |
| `ICompliance` repository interface (3 flows agrupados) | ✅ implementado | F7 |
| `SignResponsibilityTermUseCase` + audit best-effort | ✅ implementado | F8:22-96 |
| `SubmitAnnualDeclarationUseCase` + audit | ✅ implementado | F8:98-163 |
| `CheckDeclarationCurrentYearUseCase` (policy G1) | ✅ implementado | F8:166-192 |
| `ListAuditUseCase` | ✅ implementado | F8:194-223 |
| `EndMandateUseCase` com bloqueio G1 | ✅ implementado | F9 |
| HTTP routes: POST sign, POST declaration, GET audit | ✅ implementado | F11 |
| SQL queries + 3 tabelas | ✅ implementado | F10 |

### § 20.2 Gaps (G-COMP-##)

| ID | Gap | Telas impactadas | Esforço | Sprint sugerida |
|---|---|---|---|---|
| **G-COMP-01** | Score Governança: aggregate + job noturno + tabelas `governance_scores` + `governance_score_history` | C10 | M | 10 |
| **G-COMP-02** | Conflict of Interest aggregate + use cases + SQL | C4 | M | 10 |
| **G-COMP-03** | Risk Assessment aggregate + matriz 8×4 + integração Master Plan | C7 | L | 10-11 |
| **G-COMP-04** | `lgpd_logs` entity separada (hoje misturada com audit_log_entries) + consumer Connect Me | C9 | M | 10 |
| **G-COMP-05** | Dossier Export job async + PDF generator + assinatura digital ICP-Brasil | C11 | L | 11-12 |
| **G-COMP-06** | Adendo entity tipo `adendo` + validation `previous_entry_id != null` + UC CreateAdendo | C6 | S | 9 |
| **G-COMP-07** | `IAuditLogger` adapter cross-module centralizado (DT-010) + DI wiring em 3 módulos | todas | M | 10 (F1:274) |
| **G-COMP-08** | Retention job noturno (archive > 1 ano → MinIO Glacier; purge > 5 anos) | C5, C9 | M | 11 |
| **G-COMP-09** | Compliance Status derivado (aggregate VO) + query C1 painel | C1 | S | 9 |
| **G-COMP-10** | Contractual Compliance (C8): alertas garantia + prazo vencido | C8 | M | 10 |
| **G-COMP-11** | UI web SolidJS + mobile Flutter para C1-C11 | todas | XL | 10-12 |
| **G-COMP-12** | Event publisher NATS para `compliance.*` events | todas | S | 9-10 |
| **G-COMP-13** | Alerta Score < 60 + notification service integration | C10 | S | 11 |
| **G-COMP-14** | Direito ao esquecimento: endpoint + workflow aprovação admin | C9 | M | 11 |
| **G-COMP-15** | Idempotency: `SignTerm` e `SubmitDeclaration` precisam de chave (evitar duplo submit) | C2, C3 | S | 9 |

### § 20.3 Inconsistências conhecidas

| ID | Descrição | Resolução |
|---|---|---|
| **Q25** (quebras-detectadas) | F1:204 "Score Gov 1-10 + Score Compliance 0-100" vs F2:20 "Governance Score único 0-100" | **D-061 resolve**: único 0-100; Q25 fechada |
| Q-COMP-A | F1:181 "Score Privado" vs F2:41 "Score público" | Spec viva vence → público |
| Q-COMP-B | F1:264 Adendo = "commercial Req 25 + institutional Req 11" — Req 25 na prática vive em commercial.md | Manter ambos como co-owners da regra |
| Q-COMP-C | `audit_log_entries` hoje serve tanto audit geral quanto candidato a LGPD logs — deveriam ser tabelas separadas? | G-COMP-04 propõe separação; decisão pendente |
| Q-COMP-D | `ListAuditByCondominium` hoje sem filtros (user/date/action — F1:93) | Ampliar assinatura do repo em Sprint 10 |

### § 20.4 Referências cruzadas

- [[01-governanca-institucional]] — Central de Governança + S6 home card 9
- [[02-connect-me]] — gera `lgpd_logs` tipo `data_revealed`
- [[05-assembleia-live]] — homologação alimenta Score componente 7; conflito do síndico = abstém
- `vault/50-projects/master-sindico/backend/.kiro/specs/master-sindico/requirements/cross-domain.md` Req 33-42 (fonte viva)
- `vault/50-projects/master-sindico/backend/.kiro/specs/master-sindico/requirements/institutional.md` Req 11, 34-37, 46-47
- `Development/Content/03-Modulos/Compliance-Detalhado.md` (canônico principal)
- `Development/Content/02-Jornadas/Inventario-Telas.md` §10 C1-C11
- `Development/Content/regras-de-negocio/regras-canonicas.md` Regra 10 LGPD + DT-010
- `Development/backend/internal/modules/institutional/domain/entities/compliance.go` (aggregates reais)
- `Development/backend/internal/modules/institutional/application/use_cases/compliance_use_cases.go` (4 UCs)
- `Development/backend/internal/modules/institutional/application/use_cases/end_mandate_use_case.go` (gatilho G1)
- STATE.md v2 — registrar D-061 (Score único), D-COMP-## novos desta destilação

---

## Apêndice A — Endpoints REST atuais (F11)

| Verbo | Path | Handler | Body | Resposta |
|---|---|---|---|---|
| POST | `/api/v1/condominiums/:condominium_id/compliance/terms/sign` | `ComplianceHandler.SignTerm` | `{term_version, term_hash}` | `{status: "signed"}` |
| POST | `/api/v1/condominiums/:condominium_id/compliance/annual-declaration` | `ComplianceHandler.SubmitDeclaration` | `{declaration_year, declaration_text}` | `{status: "submitted"}` |
| GET | `/api/v1/condominiums/:condominium_id/compliance/audit?limit=&offset=` | `ComplianceHandler.ListAudit` | — | `{entries: [AuditLogDTO...]}` |

**ClientIP + UserAgent** extraídos do request, nunca do body (F11:37-38).

## Apêndice B — Tabelas SQL existentes (F10)

```sql
-- responsibility_term_signatures
id, condominium_id, user_id, term_version, term_hash, signed_ip, signed_user_agent, signed_at

-- annual_declarations
id, condominium_id, user_id, declaration_year, declaration_text, signed_ip, submitted_at

-- audit_log_entries
id, actor_id, actor_ip, action, target_type, target_id, condominium_id, payload, created_at
```

Queries nomeadas sqlc: `CreateTermSignature`, `LatestTermSignature`, `SubmitAnnualDeclaration`, `GetAnnualDeclaration`, `AppendAuditLog`, `ListAuditByCondominium`.

## Apêndice C — Mapa tela × spec (F1:256-268)

| Tela | Spec viva |
|---|---|
| C2 Declaração | cross-domain.md Req 34 |
| C3 Assinaturas | cross-domain.md Req 35 |
| C4 Conflitos | cross-domain.md Req 36 |
| C5 Auditoria | cross-domain.md Req 37 (append-only) |
| C6 Adendos | commercial.md Req 25 + institutional.md Req 11 |
| C7 Risco | cross-domain.md Req 38 |
| C9 LGPD | cross-domain.md Req 39 (lgpd_logs) |
| C10 Score | cross-domain.md Req 33 (único) + Req 41 |
| C11 Dossiê | cross-domain.md Req 40 + Req 42 |

## Apêndice D — Checklist dual-check

- [ ] Conferir D-061 (Score único) foi registrado em `STATE.md`
- [ ] Validar que 15 gaps G-COMP-## entraram em `AUDIT.md` ou backlog
- [ ] Confirmar que F1:204 Q25 está marcada como resolvida
- [ ] Checar consistência com [[01-governanca-institucional]] quanto ao card 9 "Compliance" em S6
- [ ] Threat-model STRIDE (Spoofing de assinatura, Tampering de audit, Repudiation de declaração, Info disclosure LGPD, DoS do job score, Elevation via admin) — não coberto aqui, abrir em `08-security/`
