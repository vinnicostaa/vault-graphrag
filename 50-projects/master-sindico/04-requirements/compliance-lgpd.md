---
title: Compliance LGPD — Master Síndico v2
type: requirement
tags: [requirements, compliance, lgpd, privacy, audit, master-sindico, v2]
source: legado cross-domain Req 39 + DADOS PARA CADASTRO termos + Lei 13.709/2018
created: 2026-04-23
updated: 2026-04-23
---

# Compliance LGPD — Master Síndico v2

Controles LGPD concretos da plataforma. Derivado de Lei 13.709/2018, ANPD guidance, termos legados ([[../90-ingested/from-vault-50-projects-master-sindico/contextos/DADOS PARA CADASTRO.md]]) e GRs transversais.

Formato: `LGPD-###` + artigo LGPD mapeado + controle + responsabilidade + evidência + status.

**Controlador**: Master Síndico (plataforma).  
**Operadores**: fornecedores com DPA (Stripe, Mux, Zitadel, Livekit, AWS, Twilio/Zenvia, Resend/SES, FCM).  
**Titulares**: usuários finais (síndicos, moradores, representantes empresa, parceiros).

> Legislação-base: Lei 13.709/2018 (LGPD), Decreto 11.058/2022, Resoluções ANPD.

---

## LGPD-001 — Consentimento explícito e versionado (Art. 8º)

**Controle**: Consent capture obrigatório em onboarding e re-captura em mudança de versão de termos.

**EARS**: *Quando* usuário aceita termos, *o sistema deve* persistir `user_term_acceptances {user_id, term_type, terms_version, accepted_at, ip, user_agent}` append-only. *Quando* nova versão é publicada, *o sistema deve* exigir re-aceite modal bloqueante.

**Termos obrigatórios** (derivados de `DADOS PARA CADASTRO.md` termos 1-14):
- Termo de Aceite Geral de Uso.
- Política de Privacidade e LGPD (tratamento de dados).
- Termo de Registro de Gestão e Linha do Tempo (síndico).
- Termo de Indicadores, Selos e Certificação Própria.
- Termo de Avaliações e Participação em Eventos (morador).
- Termo de Diagnósticos, Autoavaliações, Questionários.
- Termo de Links Temporários.
- Termo de Conteúdo Técnico (empresa).
- Termo de Uso do Connect Me.
- Termo de Promoções — Parceiro da Vizinhança.
- Código de Conduta e Uso Indevido.
- Termos de Assembleia Legal (Req 60.12 legado).

**Dependências**: IDN-024, GR-023.

**Evidência**: tabela `terms_versions` + `user_term_acceptances`, ambas append-only.

**Status**: planejado M1.

**Fonte**: Lei 13.709/2018 Art. 8º, `DADOS PARA CADASTRO.md`.

---

## LGPD-002 — Finalidade específica (Art. 6º I)

**Controle**: Propósito explícito para cada coleta de dado.

**EARS**: *Quando* sistema coleta dado pessoal, *o sistema deve* documentar finalidade na política (ex: "email pra autenticação + comunicados oficiais + Connect Me interesse"). *Quando* ANPD inspeciona → docs demonstram adequação.

**Evidência**: Política de Privacidade versionada + mapa de tratamento (`12-docs/data-treatment-map.md` a criar).

**Status**: planejado M1.

**Fonte**: Lei Art. 6º I "finalidade".

---

## LGPD-003 — Base legal para tratamento (Art. 7º, 11)

**Controle**: Toda operação de dado tem base legal registrada.

**Bases mapeadas**:
- `consent` — consentimento (primário pra dados sensíveis; e.g., saúde em banco de talentos).
- `contract` — execução de contrato (cadastro pra usar plataforma).
- `legitimate_interest` — interesse legítimo (governance score, antifraude).
- `legal_obligation` — obrigação legal (retenção audit trail 5 anos).
- `exercise_rights` — exercício regular de direitos (defesa judicial).

**EARS**: *Quando* operação de tratamento é executada, *o sistema deve* incluir `legal_basis` em `lgpd_logs`.

**Dependências**: XD-010.

**Evidência**: `lgpd_logs.legal_basis`.

**Status**: planejado M1.

**Fonte**: Lei 13.709/2018 Art. 7º.

---

## LGPD-004 — Direito de acesso (Art. 18 I-II)

**Controle**: Usuário solicita cópia dos dados.

**EARS**: *Quando* usuário emite `GET /me/data`, *o sistema deve* retornar JSON com dados pessoais tratados + finalidade + base legal + compartilhamentos (com quem).

**Dependências**: IDN-023.

**Evidência**: endpoint documentado em OpenAPI.

**Status**: planejado M2.

**Fonte**: Lei Art. 18 II-IV.

---

## LGPD-005 — Direito de portabilidade (Art. 18 V)

**Controle**: Export em formato interoperável.

**EARS**: *Quando* usuário emite `GET /me/export`, *o sistema deve* gerar ZIP com JSON estruturado (schema publicado em `/docs/portability-schema.json`). *Quando* tamanho > 50 MB → async + link presigned TTL 48h.

**Dependências**: IDN-023, GR-025.

**Evidência**: schema JSON versionado; teste end-to-end.

**Status**: planejado M2.

**Fonte**: Lei Art. 18 V.

---

## LGPD-006 — Direito de correção (Art. 18 III)

**Controle**: Usuário corrige dados inexatos.

**EARS**: *Quando* usuário emite `PATCH /me/profile`, *o sistema deve* permitir correção de campos NÃO imutáveis (exclui `role`, `cpf`/`cnpj` — IDN-029). Para imutáveis → ticket manual com documentação.

**Evidência**: audit trail `identity.profile.updated`.

**Status**: M1 (partial — campos imutáveis via suporte).

**Fonte**: Lei Art. 18 III.

---

## LGPD-007 — Direito de exclusão (Art. 18 VI) — SLA 15 dias (atualizado D-101 Fase 11)

**Controle**: Exclusão de dados pessoais em prazo definido — implementado via **soft_delete universal + pseudonimização HMAC-keyed + retenção 5y** ([[../STATE#d-101-—-lgpd-soft-delete-universal-pseudonimização-hmac-keyed-fecha-a-dc-sec-004-lgpd-m1-007|D-101]] + [[../02-architecture/adr/0037-soft-delete-universal-pseudonymize|ADR-0037]]).

**EARS (atualizado 2026-04-24)**: *Quando* usuário emite `POST /me/deletion-request`, *o sistema deve* (1) criar ticket `account_deletion_request`, (2) notificar DPO, (3) em ≤ 15 dias úteis: **soft_delete** (`UPDATE users SET deleted_at = NOW()`) + **pseudonimizar** PII (email, telefone, foto, CPF de profile) via HMAC-keyed com key rotating anual ([[../02-architecture/adr/0028-lgpd-keyed-hmac|ADR-0028]]) + **soft_delete** histórico institucional (timeline/ata mantém row com `deleted_at IS NOT NULL`; actor_id pseudonimizado). **Hard_delete físico apenas via cron `lgpd_retention_cleanup` após 5 anos** (obrigação Art. 15 + Art. 16 + normativa fiscal).

**Princípio D-101**: NUNCA hard_delete durante runtime; SEMPRE soft_delete universal. Race condition com webhooks Stripe resolvida por construção. Ver [[../08-security/lgpd#4.4.1 política-de-deleção-soft-delete-universal-d-101-fase-11-2026-04-24|lgpd.md §4.4.1]].

**Exceções retention**: dados em `audit_trail` e `lgpd_logs` são preservados (obrigação legal 5 anos LGPD Art. 16 II); `audit_log_entries` é a única tabela sem soft_delete intermediário — hard_delete pós-5y direto.

**Dependências**: IDN-022, GR-024.

**Evidência**: `lgpd_logs` action=`deletion_requested` → `deletion_completed`.

**Status**: planejado M1 (manual fulfillment); M2 (auto).

**Fonte**: Lei Art. 18 VI, prazo ANPD Resolução CD/ANPD 4/2023.

---

## LGPD-008 — Direito de oposição (Art. 18 IX)

**Controle**: Usuário se opõe a tratamento com base em interesse legítimo.

**EARS**: *Quando* usuário emite `POST /me/opposition-request?purpose=<X>`, *o sistema deve* registrar oposição; DPO review; se procedente → interrompe tratamento daquele propósito.

**Status**: planejado M3.

**Fonte**: Lei Art. 18 IX.

---

## LGPD-009 — Revogação de consentimento (Art. 8º §5º)

**Controle**: Usuário revoga consentimento previamente dado.

**EARS**: *Quando* usuário revoga consent pra tratamento específico (ex: "não quero receber SMS"), *o sistema deve* interromper o tratamento, registrar em `lgpd_logs`.

**Status**: planejado M2 (SMS opt-out); M3 (granular).

**Fonte**: Lei Art. 8º §5º.

---

## LGPD-010 — Data minimization (Art. 6º III)

**Controle**: Coletar apenas dados necessários.

**EARS**: *Quando* formulário é desenhado, *o sistema deve* justificar cada campo (imprescindível pra finalidade X). *Quando* novo campo é adicionado → review DPO.

**Evidência**: review periódico (semestral) de formulários vs finalidade.

**Status**: M1 (design review).

**Fonte**: Lei Art. 6º III.

---

## LGPD-011 — Audit trail LGPD (Art. 37)

**Controle**: Operador mantém registro de operações de tratamento.

**EARS**: *Quando* evento de tratamento ocorre (acesso, compartilhamento, exclusão, consent, revogação), *o sistema deve* registrar em `lgpd_logs` append-only com retenção 5 anos.

**Dependências**: XD-010, GR-026.

**Evidência**: tabela + retenção configurada.

**Status**: planejado M1.

**Fonte**: Lei Art. 37.

---

## LGPD-012 — Breach notification 72h (Art. 48)

**Controle**: Notificar ANPD em até 72h se incidente afetar direitos/liberdades.

**EARS**: *Quando* incidente de segurança é detectado afetando dados pessoais, *o sistema deve* (1) DPO classificar severidade em ≤ 24h, (2) se afeta direitos → notificar ANPD via sistema online em ≤ 72h, (3) notificar titulares impactados com orientações.

**Dependências**: Incident response plan em `08-security/`.

**Evidência**: playbook + runbook + ANPD notification template.

**Status**: planejado M1 (runbook) / M2 (automação detection).

**Fonte**: Lei Art. 48, Res. ANPD 15/2024.

---

## LGPD-013 — DPO registrado

**Controle**: Encarregado de proteção de dados nomeado e divulgado.

**EARS**: *Quando* usuário acessa footer, *o sistema deve* exibir: "DPO: <nome>, <email>, <telefone>".

**Evidência**: footer site + Política de Privacidade + entry em `12-docs/dpo-contact.md`.

**Status**: planejado M1.

**Fonte**: Lei Art. 41.

---

## LGPD-014 — DPA com operadores (Art. 39)

**Controle**: Data Processing Agreement assinado com cada operador.

**EARS**: *Quando* plataforma usa fornecedor que processa dados pessoais, *o sistema deve* ter DPA assinado antes de produção.

**Fornecedores críticos**:
| Fornecedor | Dados tratados | DPA status | País |
|---|---|---|---|
| Stripe | email, billing, device fingerprint | ✓ Stripe DPA template (adesão) | US/EU |
| Mux | vídeos (conteúdo user) | ⚠️ a verificar | US |
| Zitadel Cloud (OIDC) | email, password hash, MFA | ⚠️ a verificar | DE/CH |
| Livekit Cloud | audio/video streams assembleias | ⚠️ a verificar | US |
| AWS (S3/SES/ECS) | arquivos, email | ✓ AWS DPA | US/BR |
| Twilio/Zenvia | telefone, SMS | ⚠️ a verificar (Zenvia é BR!) | US / BR |
| Resend | email | ⚠️ a verificar | US |
| FCM (Google) | device tokens | ✓ Google DPA | US |
| Railway | infraestrutura + logs | ⚠️ a verificar | US |
| Cloudflare R2 (alt storage) | arquivos | ✓ Cloudflare DPA | US |
| ViaCEP | CEP (dado público) | N/A | BR |

**Evidência**: contratos arquivados + `12-docs/dpa-register.md` (a criar).

**Status**: M1 (audit + assinar pendentes).

**Fonte**: Lei Art. 39, guidance ANPD.

---

## LGPD-015 — Transferência internacional (Art. 33-36)

**Controle**: Dados transferidos ao exterior só com base legal.

**Cenário**: Stripe (US), Mux (US), Zitadel Cloud (DE/CH), Livekit (US).

**EARS**: *Quando* DPA com operador estrangeiro → prever salvaguarda (cláusulas contratuais padrão, adequação equivalente, ou consent).

**Evidência**: cláusula ICP + aderência ANPD Res. 13/2024.

**Status**: M1 review.

**Fonte**: Lei Art. 33.

---

## LGPD-016 — Dados sensíveis (Art. 5º II, 11)

**Controle**: Dados sensíveis requerem tratamento especial.

**Cenário no Master Síndico**:
- Biometria (Req 56 check-in via biometria app M3+) → consent específico + destacado.
- Saúde (potencial banco de talentos M3) → avoid, ou consent destacado.

**EARS**: *Quando* dado sensível é coletado, *o sistema deve* (1) consent destacado (não genérico), (2) finalidade específica, (3) base legal adequada.

**Status**: M3 (quando biometria ativa).

**Fonte**: Lei Art. 11.

---

## LGPD-017 — Data classification (CRITICAL / HIGH / NORMAL)

**Controle**: Classificar dados por criticidade com controles diferenciados.

**Classificação**:

| Classe | Exemplos | Controles |
|---|---|---|
| **CRITICAL** | CPF, CNPJ, password hash (Zitadel), Stripe data, votos, fração ideal | Encryption at rest (DB column), encryption in transit (TLS 1.3), logging ≤ 0 PII, acesso logado, retenção controlada |
| **HIGH** | email, telefone, endereço, device fingerprint, face (foto), vídeos privados, comentários | Encryption at rest (storage level), TLS, audit trail, opt-out permitido |
| **NORMAL** | timeline pública, vídeos institucionais aprovados, perfil público empresa, governance score | TLS, versioning, immutability via audit |

**Dependências**: GR-007 (PII masking), GR-026 (audit trail).

**Evidência**: `12-docs/data-classification.md` (a criar).

**Status**: M1 planejamento; M2 encryption at rest column-level (CPF).

**Fonte**: best practice + LGPD adequação.

---

## LGPD-018 — Retention policies

**Controle**: Dados retidos pelo prazo necessário + obrigação legal.

**Políticas**:
| Categoria | Retenção | Base |
|---|---|---|
| User ativo | Enquanto conta ativa | Contract |
| User deletado | **soft_delete em 15d + pseudonimização** (HMAC-keyed); hard_delete físico apenas após 5 anos via cron (D-101 Fase 11 + ADR-0037) | Art. 15, Art. 16, Art. 18 VI |
| `audit_trail` | 5 anos | Obrigação legal |
| `lgpd_logs` | 5 anos | Art. 37 |
| Stripe transactions | 7 anos | Obrigação fiscal |
| Vídeos privados | Enquanto user ativo; 90d pós-deletion | Contract |
| Vídeos institucionais arquivados | 7 anos | Obrigação contratual |
| Backups DB | 1 ano | Operacional |
| Logs aplicação | 30 dias | Operacional |

**Status**: M1 runbook / M2 auto-purge jobs.

**Fonte**: Lei + CTN (fiscal).

---

## LGPD-019 — Privacy by design

**Controle**: Design considera privacidade desde concepção.

**Princípios aplicados**:
- **Rede social cega** (CNT-012): métricas só visíveis ao dono — evita "palco de vaidade", reduz vigilância.
- **Connect Me unidirecional** ([[STEERING]] §38): sem histórico, sem thread; minimiza tracking.
- **Visibilidade controlada** (matriz funcional): perfil empresa só visível a quem precisa.
- **Anonimização em enquetes** (INS-030): síndico vê contagem, não nomes.
- **Avaliação anônima** (COM-024): síndico anônimo pra empresa.

**Fonte**: [[STEERING]] §24, LGPD Art. 46.

---

## LGPD-020 — Training + awareness

**Controle**: Time técnico treinado em LGPD.

**Threshold**: 100% devs + ops treinados antes de manusear dados production.

**Status**: M1 runbook + primeiro training.

**Fonte**: ANPD guidance.

---

## LGPD-021 — Vendor security questionnaire

**Controle**: Due diligence antes de contratar operador.

**EARS**: *Quando* novo fornecedor é avaliado, *o sistema deve* questionário de segurança (criptografia, certificações ISO 27001, SOC2, DPA).

**Status**: M1 template.

**Fonte**: best practice.

---

## LGPD-022 — Cookie consent + banner

**Controle**: Usuário consente cookies não-essenciais.

**EARS**: *Quando* usuário 1ª visita ao site web, *o sistema deve* exibir banner consent (essenciais pre-aprovados; analytics/marketing opt-in).

**Status**: M1.

**Fonte**: LGPD + diretiva-e EU (aderência voluntária).

---

## LGPD-023 — Criptografia em trânsito (TLS 1.3)

**Controle**: TLS mínimo 1.2, preferência 1.3.

**Threshold**: 0 conexões HTTP (redirect 301 → HTTPS); TLS 1.3 suportado.

**Fonte**: NFR-021 security practice.

---

## LGPD-024 — Criptografia em repouso

**Controle**: Dados em repouso criptografados.

**Threshold**:
- Railway/ECS: encryption at rest nativo (AES-256).
- Colunas CRITICAL (CPF, CNPJ): column-level encryption (pgcrypto) M2+.
- Backups: encrypted.
- S3/R2: SSE-S3 ou SSE-KMS.

**Status**: M1 (storage nativo) / M2 (column encryption).

**Fonte**: LGPD Art. 46 II.

---

## LGPD-025 — Incident response plan

**Controle**: Plano escrito de resposta a incidentes.

**EARS**: *Quando* alert de possível incidente é recebido, *o sistema deve* seguir runbook: (1) triagem em < 1h, (2) contenção, (3) DPO classifica, (4) notificação ANPD + titulares em 72h se aplicável, (5) pós-mortem.

**Evidência**: `08-security/incident-response-plan.md` (a criar).

**Status**: M1 runbook.

**Fonte**: Lei Art. 48.

---

## LGPD-026 — Access control principle of least privilege

**Controle**: Acesso a dados pessoais por princípio do menor privilégio.

**EARS**: *Quando* dev precisa acessar dados prod, *o sistema deve* exigir (1) ticket aprovado, (2) sessão temporária com MFA, (3) audit log completo.

**Status**: M2.

**Fonte**: LGPD Art. 46 II.

---

## LGPD-027 — Anonymization/pseudonymization

**Controle**: Dados anonimizados quando análise agregada possível.

**EARS**: *Quando* dashboard agrega métricas, *o sistema deve* usar IDs pseudonimizados ou aggregation only (nunca row-level).

**Exemplo**: governance score agrega dados sem exposição individual.

**Status**: M1 design.

**Fonte**: LGPD Art. 12.

---

## LGPD-028 — Portabilidade pra Connect Me dados revelados

**Controle**: Quando síndico revela dados a empresa via "Tenho Interesse", registra em `lgpd_logs` com versão de termos + propósito.

**EARS**: Ver COM-007.

**Fonte**: `cross-domain.md` Req 39, LGPD Art. 18 IX.

---

## LGPD-029 — Cancelamento do Connect Me pós-revelação

**Controle**: Se síndico solicita remoção de contato que foi revelado, empresa é notificada mas dados já revelados ficam no histórico (registro de tratamento).

**Status**: M2.

**Fonte**: Lei Art. 16 (conservação enquanto obrigação legal).

---

## LGPD-030 — DPO ombudsman canal

**Controle**: Canal exclusivo DPO `dpo@mastersindico.com.br` monitorado.

**EARS**: *Quando* titular envia solicitação DPO, *o sistema deve* responder em ≤ 15 dias.

**Status**: M1 setup canal.

**Fonte**: Lei Art. 41.

---

## Mapa de direitos × endpoints

| Direito LGPD | Endpoint / Mecanismo | SLA | Status |
|---|---|---|---|
| Acesso (Art. 18 II-IV) | `GET /me/data` | 15d | M2 |
| Correção (III) | `PATCH /me/profile` ou DPO | Imediato / 15d manual | M1 parcial |
| Anonimização/bloqueio (IV) | DPO ticket | 15d | M2 |
| Portabilidade (V) | `GET /me/export` | 15d (async) | M2 |
| Eliminação (VI) | `POST /me/deletion-request` | 15d | M1 |
| Informação compart. (VII) | Política + `GET /me/sharings` | — | M2 |
| Revogação consent (§5º Art. 8º) | `POST /me/consent/revoke` | Imediato | M2 |
| Oposição (IX) | `POST /me/opposition-request` | 15d DPO | M3 |

---

## Checklist pré-M1

- [ ] Termos 1-14 versionados e publicados (`terms_versions`).
- [ ] Política de Privacidade publicada.
- [ ] DPO nomeado + canal `dpo@mastersindico.com.br`.
- [ ] DPAs: Stripe ✓, AWS ✓, Google FCM ✓; Mux, Zitadel, Livekit, Twilio/Zenvia, Resend → assinar/verificar.
- [ ] Data treatment map (`12-docs/data-treatment-map.md`).
- [ ] Data classification doc (`12-docs/data-classification.md`).
- [ ] Incident response playbook (`08-security/incident-response-plan.md`).
- [ ] Cookie banner live.
- [ ] Deletion endpoint (IDN-022) operacional.
- [ ] Consent versioning operacional.
- [ ] Audit trail + LGPD logs append-only + retenção.

---

## Pendências (dual-check Fase 3)

- ⚠️ **PENDÊNCIA LGPD-014**: validar DPAs pendentes (Mux, Zitadel, Livekit, Twilio, Resend) — legal review.
- ⚠️ **PENDÊNCIA LGPD-015**: cláusulas contratuais padrão ANPD Res. 13/2024 — aplicar aos DPAs.
- ⚠️ **PENDÊNCIA LGPD-017**: encryption column-level CPF/CNPJ — ADR M2 (pgcrypto vs app-level AES-GCM).
- ⚠️ **PENDÊNCIA LGPD-007** (pós-D-101): blobs Mux recebem **hard_delete imediato** via API (blobs não entram no soft_delete universal — são binary assets, não dados relacionais). Confirmar Mux API suporta purga síncrona com prazo ≤ 15d.
- ⚠️ **PENDÊNCIA LGPD-025**: incident response plan — elaborar pré-M1.
- ⚠️ **PENDÊNCIA LGPD-020**: curso de LGPD pra devs — agendar.

---

## Referências

- Lei 13.709/2018.
- Decreto 11.058/2022.
- ANPD Res. 4/2023 (procedimentos), Res. 13/2024 (transferência internacional), Res. 15/2024 (incidentes).
- Legado: `cross-domain.md` Req 39, `DADOS PARA CADASTRO.md` termos 1-14.
- [[global]] GR-012, GR-023, GR-024, GR-025, GR-026.
- [[non-functional]] NFR-031, NFR-032, NFR-033, NFR-036.
- [[functional/identity]] IDN-022, IDN-023, IDN-024.
- [[functional/cross-domain]] XD-008, XD-010.
- [[_moc]]
