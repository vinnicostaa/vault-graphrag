---
title: Business Rules — Master Síndico
type: project
tags: [master-sindico, domain, business-rules]
project: master-sindico
created: 2026-04-22
---

# Business Rules

Regras de **negócio** — o que o negócio exige. Oposto a regras de domínio (invariantes) e de implementação (como).

Fonte: [[../content/regras-de-negocio/regras-canonicas]], `backend/.kiro/specs/master-sindico/requirements/*.md`, client-vault.

---

## BR-001 — Trial por persona

- Síndico: 15 dias
- Empresa: 7 dias
- Parceiro: 30 dias
- Morador: não tem trial (acesso via condomínio)
- **Não é global** — dep. persona detectada no onboarding

## BR-002 — Identidade única por CPF/CNPJ

- 1 User por CPF ou CNPJ (checado via Zitadel sync)
- Contextos múltiplos: mesmo User pode ser síndico em X, morador em Y

## BR-003 — Reputação Bronze → Diamante é determinística

Ver [[../00-product/business-model#modelo-bronze--diamante-reputação]].

- Calculada por fórmula auditável; **nunca** input humano direto
- Recalculada a cada `CompanyEvaluation` nova
- Revogação automática em `HarassmentReported` confirmado

## BR-004 — 1 dispositivo ativo por conta

- Default: 1
- Admin pode multi-device
- Configurável via `MAX_ACTIVE_SESSIONS` env var
- Troca de device **invalida** a anterior (InvalidateOtherSessions)

## BR-005 — Timeline imutável

- `TimelineEntry` nunca é editado ou deletado
- Correção = novo entry linkando ao original + motivo
- Sem `deleted_at` na tabela

## BR-006 — Connect Me exige empresa aprovada

- Empresa só pode responder ConnectMe se `state=approved`
- Aprovação via moderação (manual no início; ML futuro)
- Empresa sob `HarassmentReported` confirmado é suspensa

## BR-007 — Quórum calculado por fração ideal, não por pessoa

- Assembleia delibera quando ≥ 50% (ou threshold configurado) de `FracaoIdeal` presente
- `Vote` pondera pela fração da unidade do votante

## BR-008 — Vídeo publicado tem trava de 90 dias

- Após `VideoPublished`, o vídeo fica `locked` por 90 dias
- Motivo: churn prevention + evita spam de upload/remove
- Override possível via role `admin` (só em caso legal)

## BR-009 — Privacidade de métricas entre tenants

- Views, cliques, buscas de um tenant **não** vazam pro outro
- Empresas veem agregados globais (sem identificação de condomínio específico)

## BR-010 — LGPD: direito de exclusão

- User pode solicitar exclusão
- Hard delete: `User`, `Sessions`, `DeviceFingerprints`, dados pessoais em `Company`
- Soft archive com pseudonimização: votes, avaliações, minutes (imutáveis por outros requisitos)
- SLA: 15 dias da solicitação

## BR-011 — Webhook idempotency

- Stripe: `event.id` único; processamento defensivo em reentregas
- Mux: `request_id + object.id`
- Livekit: event signing + dedup local

## BR-012 — ABAC default-deny

- Ação não configurada no catálogo → negar
- Admin bypass é explícito; exige `user.is_admin = true` + audit log do bypass

## BR-013 — Assembleia não pode começar sem edital publicado

- `PublishEdital` obrigatório **antes** de `Start` (A-013 fix)
- Edital = PDF com pauta + convocação + data

## BR-014 — Contrato Connect Me gera ExecutionRecord

- Cada entrega marca milestone em `ExecutionRecord`
- Avaliação só disponível após `contract.state=completed`

## BR-015 — Reputação de empresa é global, mas visibilidade é por tenant

- Reputação calculada de todos contratos
- Mas moradores/síndicos só veem avaliações **do próprio tenant** (em detalhe)
- Agregado global (reputação geral) visível a todos

## BR-016 — Moderação de conteúdo

- Vídeos: moderação manual antes de `published`
- Harassment reports: análise em 48h
- Palavras-chave bloqueadas em descriptions (lista a manter)

## BR-017 — Onboarding por persona

- Síndico: CPF + validar email + vincular a 1 condomínio (novo ou existente)
- Morador: convite via síndico ou CPF em `Unit.morador_cpf`
- Empresa: CNPJ + validar CNPJ Receita Federal (stub; impl pós-M2)
- Parceiro: CNPJ ou MEI

## BR-018 — Planos têm quotas; exceder bloqueia ação

- Ex: ConnectMe Básico: 5 requests / mês
- Ao exceder, ação retorna `quota_exhausted`; UI sugere upgrade
- Reset anual (ConnectMe) ou mensal (uploads)

## BR-019 — Subsíndico e Conselho têm permissões específicas

- Subsíndico: pode maioria das ações do síndico (exceto encerrar mandato)
- Conselho: aprovar contratos acima de X (SupplierVote), vigiar atos do síndico

## BR-020 — Transferência de mandato de síndico

- Processo: registrar em timeline + encerrar mandato anterior + iniciar novo
- Síndico anterior perde acesso gradualmente (7 dias de transição)
- Histórico mantido; nunca apagado

---

## Relação com outras fontes

- `backend/.kiro/specs/master-sindico/requirements/*.md` — lista Req 1-103 baseada nestas regras
- [[domain-rules]] — invariantes que garantem estas regras em runtime
- [[implementation-rules]] — como implementamos (CQRS, UoW, Saga) pra suportar estas regras
- [[invariants]] — propriedades que **nunca** violam

## Links

- [[_moc]]
- [[bounded-contexts]]
- [[domain-rules]]
- [[implementation-rules]]
- [[invariants]]
- [[../04-requirements/_moc]]
- [[../00-product/business-model]]
- [[../content/regras-de-negocio/regras-canonicas]]
