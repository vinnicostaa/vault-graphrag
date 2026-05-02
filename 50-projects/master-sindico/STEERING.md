---
title: STEERING — Master Síndico v2
type: state
tags:
  - steering
  - non-negotiables
  - master-sindico
  - v2
created: 2026-04-23T00:00:00.000Z
updated: 2026-04-23T00:00:00.000Z
---

# STEERING — Master Síndico v2

Não-negociáveis do projeto. Mudança aqui requer decisão explícita em [[STATE]] como D-### com dual-check.

---

## Princípios do produto

1. **Governança aplicada, não ERP**. Master Síndico conecta síndicos, moradores, empresas do mercado condominial, agências de marketing e comércio local via **critério na contratação (reputação auditável), memória institucional imutável (timeline), ecossistema unificado**. Não é ERP, chat, sistema financeiro ou app de comunicação interna.

2. **Identidade única, contextos múltiplos**. 1 CPF ou 1 CNPJ = 1 User. O mesmo User pode ser síndico no Condomínio X, morador no Y, representante da Empresa Z.

3. **Brasileiro primeiro**. Termos jurídicos BR preservados (condomínio, fração ideal, edital, ata, plano diretor, assembleia ordinária/extraordinária, quórum fracional). Interface pt-BR. EN só em código/frontmatter/identifiers.

4. **7 sub-produtos integrados** (ver [[00-product/portfolio-de-produtos]]):
   1. Governança Institucional
   2. Connect Me (marketplace de contratos unidirecional)
   3. Reputação Bronze→Diamante (determinística)
   4. Conteúdo & Vídeos (Mux, trava 90d)
   5. Assembleias Live & Votações (Livekit, quórum fracional)
   6. LMS & Banco de Talentos
   7. Vizinhança (comércio local)

5. **Auditabilidade imutável**. Timeline é append-only, sem UPDATE/DELETE. Ata de assembleia é imutável pós-publish. Governança não sofre revisão retroativa.

---

## Princípios de arquitetura

6. **Clean Architecture estrita** — dependências apontam pra dentro (`infrastructure → application → domain`). Domínio zero import de framework/SDK.

7. **DDD tático** — aggregates com estado encapsulado (entidades privadas + getters), factories `NewX` (com validação) e `ReconstructX` (sem validação, vindo do repo), repositório interface em domain e impl em infrastructure.

8. **CQRS** — command ≠ query, arquivos separados.

9. **UnitOfWork intra-módulo, Saga inter-módulo** — quando cruza provider externo (Mux/Livekit/Stripe), saga com compensação explícita.

10. **Provider abstraction** — `IPaymentGateway`, `IVideoProvider`, `IEmailProvider`, etc. SDK externo isolado em `infrastructure/providers/<provider>/`.

11. **Multi-tenant estrito** — toda query escopada por condomínio tem `WHERE condominium_id = $tenant_id`. Cross-tenant testado sistematicamente.

12. **Imutabilidade por design** — value objects imutáveis, timeline append-only, money em `int64` centavos, UUIDv7 ordenável.

13. **OpenAPI 3.1 contrato-first** — escrever spec antes do handler; versioning `/api/v1` com deprecation 6m em breaking changes.

14. **Event-driven entre BCs** — nenhum BC importa `domain/` ou `application/` de outro BC. Comunicação via domain events.

---

## Princípios de segurança (BeyondCorp adaptado)

15. **Zero-trust sem rede confiável** — toda request passa por auth + autz.

16. **ABAC matrix ordenada** — admin_bypass → role → action → tenant → scope. Cache Redis 5min com invalidação via webhook Stripe/Zitadel.

17. **1 device per account** — `device_fingerprint` (IP + UA + canvas hash). Login novo invalida sessão anterior. Audit log da troca de device (A-023).

18. **PKCE obrigatório** em OIDC. `code_verifier` em cookie `gorilla/securecookie`.

19. **Rate limiting estruturado** — `/auth/*` 20rpm burst 5; `/api/v1/*` 60rpm burst 10. Token bucket com cleanup (A-032).

20. **PII nunca em logs** — CPF/CNPJ/email/token sempre mascarados via `Masked()`. CI bloqueia via grep regex.

21. **Webhook HMAC verificado antes de parse** — Stripe, Mux, Livekit. Rejeita body inválido com 400.

22. **CORS allowlist estrita** — wildcard `*` bloqueado em staging/prod via `Config.Validate()` (A-005).

23. **Secrets gitignored** — `.env`, `zitadel-key*.json`. Nunca em código ou repo público (A-018).

24. **LGPD by design** — consentimento versionado, direito de exclusão SLA 15d, portabilidade via `GET /me/export`, audit trail `lgpd_logs` append-only 5 anos, breach notification 72h.

---

## Princípios de qualidade

25. **Coverage thresholds duros** — domain 95%, application 90%, infrastructure 85%, http 75%, global ≥ 85%. Gate de CI.

26. **Gates obrigatórios pra merge** — `go build`, `go vet`, `go test -race`, `go test -tags=integration`, `gosec -severity=high` 0 findings, `govulncheck` 0 CVEs reachable.

27. **Code Calisthenics Go-flavor** — sem bloco `else`, early return / guard clauses, sem `any`/`interface{}` onde tipo concreto serve, máximo um nível de indent por método quando razoável.

28. **TDD/BDD + PBT em invariantes críticas** — fração ideal, quotas, ABAC matrix, vote único, pagination. Property-based tests obrigatórios.

29. **SDD 5-fase** — Discuss → Plan → Execute → Verify → Ship. Spec vive em `04-requirements/`.

30. **GSD** — tasks pequenas, shipping incremental, nada de big-bang.

---

## Princípios de processo

31. **Research-loop antes de decidir** — analisar vault + doc oficial + engineering blogs big tech + patterns aplicáveis + dual-check. Registrar em D-### com fontes linkadas.

32. **Dual-check obrigatório** em: arquitetura, contratos públicos, schema DB, security policy, regra de negócio em prod, conflito de fontes, modo autônomo.

33. **Nada de alucinação** — se não validou, escreve `⚠️ PENDÊNCIA: não validado em doc oficial`. Bloqueia até pesquisa.

34. **D-### + ADR + STATE** — toda decisão arquitetural tem registro.

35. **A-### + AUDIT** — toda descoberta tem registro; 🔴 bloqueiam task nova.

36. **Legado intocado até Fase 6** — nenhum `rm` em `vault/50-projects/master-sindico/`, `_archive/`, `_source-material/` antes da confirmação explícita do usuário.

---

## Não-negociáveis multi-produto (novos)

37. **Sub-produto = bounded context(s) mapeado(s)** — nenhum sub-produto flutuante sem BC atribuído.

38. **Connect Me é unidirecional** — formulário RFP, não chat. Cotas anuais. Nunca virar thread de mensagens.

39. **Reputação é determinística** — função de métricas (contratos fechados, avaliações, condomínios atendidos, casos vídeo). Nenhum input humano direto altera tier; só os inputs alimentam a fórmula.

40. **Vídeos travam 90 dias pós-publish** — institucional (empresa/síndico) e vídeo-currículo (morador). Churn prevention.

41. **Timeline imutável** — sem UPDATE, sem DELETE, sem `deleted_at`. Correção vira nova entry.

42. **Vote único por (agenda_item, voter)** — UNIQUE DB constraint + tratamento `ErrConflict` no use case (A-025 resolvido no legado).

43. **Ata imutável pós-publish** — Minutes não sofre edição. Correção vira nova entry na timeline.

44. **Quórum fracional** — voto por fração ideal, não por cabeça. Σ fração ideal ≤ 100%.

45. **Trial persona-aware** — síndico 15d, empresa 7d, parceiro 30d, morador — (sem trial).

---

## Referências

- Herança direta: [[../vault/50-projects/master-sindico/STEERING]] (se existir) + [[../vault/CLAUDE]] §8.
- Plano: `/home/vinni/.claude/plans/squishy-drifting-avalanche.md`.
