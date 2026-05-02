---
title: DoR / DoD — Master Síndico
type: project
tags: [master-sindico, definition-of-ready, definition-of-done, process]
project: master-sindico
created: 2026-04-22
---

# Definition of Ready / Definition of Done — Master Síndico

Critérios para considerar uma **task** pronta para começar (DoR) e pronta para fechar (DoD). Instancia o SDD 5-fase para este projeto.

---

## Definition of Ready (entrar em Execute)

Uma task **não começa** (fase Execute do SDD) sem:

### Spec
- [ ] Task está em `05-tasks/by-sprint/sprint-<N>.md` (ou by-module) com ID estável
- [ ] Requirement referenciado em `04-requirements/functional/<módulo>.md` com Req #
- [ ] Design referenciado em `02-architecture/**` ou ADR se decisão nova
- [ ] Critérios de aceite **testáveis** escritos (não "funcione bem")
- [ ] Modelo/diagrama se a task toca arquitetura ou contrato público

### Bloqueadores
- [ ] Sem item 🔴 em [[11-audit/AUDIT]] que conflite com a task
- [ ] Sem D-### em aberto no STATE que bloqueie a decisão principal da task
- [ ] Dependências externas resolvidas (ex: credenciais Stripe staging disponíveis)

### Conhecimento
- [ ] SDKs / libs a usar tiveram doc oficial consultada via Context7 ou WebFetch
- [ ] Patterns aplicáveis mapeados em `07-quality/PATTERNS`
- [ ] Se pattern é novo pro projeto: [[../../30-agents/playbooks/research-loop]] executado

### Ambiente
- [ ] Branch criada a partir de `main` atualizada
- [ ] Docker-compose sobe PG18 + Redis 7 + Zitadel (se auth)
- [ ] `.env` com variáveis necessárias (Stripe CLI rodando se webhook em teste)

---

## Definition of Done (fechar task)

Uma task **não é merged** sem:

### Código
- [ ] Ordem canônica seguida (migration → query → entity → repo → use case → handler → wire-up → testes) ou exceção documentada no plano
- [ ] Sem bloco `else` (early return / guard clause)
- [ ] Sem `_ = fn()` em I/O crítica
- [ ] Sem `SELECT *` em queries novas
- [ ] Domain sem imports de framework/SDK externo
- [ ] Mapper explícito entre DTO/ORM e entidade de domínio
- [ ] Mensagens de erro tipadas (`ErrValidation.WithMessage(...)`)

### Testes
- [ ] Teste unitário do(s) happy path(s)
- [ ] Teste unitário do(s) error path(s) relevantes
- [ ] Teste de integração com testcontainers quando toca DB
- [ ] PBT (property-based) se invariante crítico (ver [[08-security/threat-model#regras-PBT]])
- [ ] Coverage do módulo tocado dentro dos thresholds (domain 95% / app 90% / infra 85%)
- [ ] `go test -race -count=1 ./...` ✅
- [ ] `go test -tags=integration -count=1 ./...` ✅

### Segurança
- [ ] `gosec -severity=high` → 0 findings
- [ ] `govulncheck ./...` → 0 CVEs reachable
- [ ] Secret checklist: nenhum hardcoded, `.env` não commitado
- [ ] PII não aparece em log novo
- [ ] Webhook novo: signature verify antes do parse
- [ ] Nova rota protegida: ABAC aplicado; tenant isolation testado
- [ ] Mudança de auth/cookies: [[../../30-agents/playbooks/dual-check]] aplicado

### Docs
- [ ] OpenAPI atualizado (`02-architecture/openapi/*.yaml`)
- [ ] README do sub-projeto atualizado se feature-level (endpoint novo, env var nova)
- [ ] CHANGELOG do sub-projeto recebe entrada
- [ ] ADR criado se decisão arquitetural nova (`02-architecture/adr/NNNN-<slug>.md`)
- [ ] Comentário no código só onde o **porquê** é não-óbvio (nunca o quê)

### Governança
- [ ] `tasks.md` marca a task com `[x]` **antes** do commit de Ship (preventivo A-012/A-017)
- [ ] STATE.md recebe D-### se houve decisão nova
- [ ] AUDIT.md recebe A-### se descobriu bug em trabalho vizinho
- [ ] Se houve rollback ou incidente: registrado com lição
- [ ] Release Notes escritas na PR body

### Review
- [ ] Dual-check executado se gatilho de [[../../30-agents/playbooks/dual-check]] disparado
- [ ] Self-review feita com olhos frescos (≥ 30min ou manhã seguinte em autônomo)
- [ ] Sprint-auditor agenda registrada ao fim do sprint

---

## Exceções documentadas

Qualquer desvio da DoD **precisa**:
1. `<verify-exception>` no plano da task explicando
2. Entrada em STATE.md como DT-### com sprint alvo
3. Concordância humana se for security/privacy

Exemplo:
```xml
<verify-exception>
  Handler <X> coverage 74% (vs 75% threshold) porque error path "DB down in 100ms window"
  é impossível de testar sem harness de caos; mitigado por teste de integração <Y>.
</verify-exception>
```

---

## DoR/DoD por tipo de task

### Task de bug fix (A-###)
- DoR simplificado: A-### aberto + reprodução em teste
- DoD: teste que reproduz o bug passa; regressão test adicionado

### Task de refactor
- DoR: impacto mapeado (cross-module linkado); antes/depois comparado
- DoD: semântica inalterada (testes existentes passam sem mudança de expectation); performance igual ou melhor

### Task de spike/research
- DoR: pergunta clara a responder; time-box declarado
- DoD: D-### criado com aprendizado; não gera código de produção

### Task de infra
- DoR: staging acessível; rollback plan
- DoD: staging validado; runbook em `12-docs/runbooks/`

---

## Links

- [[CLAUDE]]
- [[STEERING]]
- [[STATE]]
- [[11-audit/AUDIT]]
- [[../../10-knowledge/methodology/sdd-workflow]]
- [[../../30-agents/playbooks/plan-and-execute]]
- [[06-execution/STEPS]]
