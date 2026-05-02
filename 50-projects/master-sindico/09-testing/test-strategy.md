> **Ver knowledge global atemporal** — [[../../../10-knowledge/testing/testing-strategy]] + [[../../../10-knowledge/testing/bdd-behavior-driven-development]].
>
> Este arquivo é **aplicação MS** — thresholds específicos, ferramentas por stack, casos de BDD usados (assembleia, votação, cupom). Conceitos canônicos (TDD, pirâmide, PBT) estão no knowledge global.

---

title: Test Strategy — Master Síndico v2
type: guide
tags: [testing, strategy, tdd, pbt, pyramid, master-sindico, v2]
source: vault/10-knowledge/testing/testing-strategy.md + 90-ingested/from-vault-50-projects-master-sindico/09-testing/test-strategy.md
created: 2026-04-23
updated: 2026-04-23
aliases: [testing-strategy-master-sindico]
---

# Test Strategy — Master Síndico v2

Pirâmide **70-20-10** (unit / integration / E2E) + PBT obrigatório em invariantes críticos + gates duros em CI. Filosofia TDD: **teste vem antes**. Cada BC tem quota de testes estimada e gates não negociáveis.

> Herda [[../CLAUDE]] §8.5 e [[../STEERING]] §25. Universal em [[../../vault/10-knowledge/testing/testing-strategy]].

---

## 1. Pirâmide canônica (alvo)

```
                  10%  E2E (Playwright + Patrol)
                20%  Integration (testcontainers-go + real DB/Redis)
              70%  Unit (go test + pgregory.net/rapid PBT)
```

- **Unit** (70%) — base rápida (< 100ms cada), stubs hand-rolled, cobre lógica domain/use-case/middleware isoladamente.
- **Integration** (20%) — PG 18 real + Redis 7 real via testcontainers-go, cobre repositories, migrations, tx, cross-tenant isolation, idempotency.
- **E2E** (10%) — fluxos críticos end-to-end (web via Playwright, mobile via Patrol/flutter integration_test).

**Inversão (pirâmide invertida)** = red flag: mais E2E que unit → unit não cobre cenários; refatorar.

---

## 2. Quando cada tipo

### Unit
- Regra de domínio (VO validation, state transition, factory precondition).
- Use case com stubs de repo/provider (sem DB/HTTP).
- Middleware com fake `Context`.
- Pattern: **table-driven** + **arrange-act-assert** + nomenclatura `TestX_when_Y_should_Z`.

### Property-Based (PBT)
- Invariantes matemáticos/combinatórios (fração ideal, quotas, ABAC matrix, pagination).
- Mutações que preservam propriedade.
- Quando muitas combinações possíveis de input.
- Obrigatório: INV-016 (quota 300 casos), INV-021 (fração 100), INV-086 (ABAC 400), INV-071 (vote TOCTOU), INV-043 (evaluation TOCTOU). Ver [[pbt]].

### Integration
- Repositories + sqlc + migrations.
- UoW com tx real.
- Cross-tenant isolation (100+ casos sistemáticos).
- Idempotency (webhook redis SetNX + processamento).
- Rate limit (cleanup goroutine + memory).
- Pattern: `//go:build integration` + `TestMain` sobe container reusable + `TruncateAll` no setup.

### E2E
- Signup flow completo (síndico onboarding 6 telas → dashboard).
- Connect Me end-to-end (criar RFP → empresa expressa interesse → LGPD log → proposta).
- Upload vídeo Mux (upload → moderation → approve → publish → lock 90d).
- Assembleia live (criar → publicar edital → iniciar → votar → fechar → homologar).
- Pattern: smoke tests pós-deploy + regression tests semanal.

### Load
- k6 scripts pra SLO validation. Ver [[load]].

### Security
- gosec + govulncheck + OWASP ZAP baseline (stretch M3+). Ver [[security]].

---

## 3. Estimativa de testes por BC (M1-M3)

Ordem de grandeza pro gate ≥ 85% global:

| BC | Unit | Integration | E2E | PBT | Total |
|---|---|---|---|---|---|
| **identity** | ~150 | ~30 | 3 (signup, login, device-swap) | 2 (CPFVO, EmailVO) | ~185 |
| **billing** | ~120 | ~25 | 2 (checkout, upgrade) | 3 (quota, trial transition, Money) | ~150 |
| **institutional** | ~200 | ~40 | 3 (onboard condo, add unit, publish announcement) | 3 (FracaoIdeal, TimelineAppend, 15 condos cap) | ~246 |
| **commercial** | ~250 | ~50 | 4 (Connect Me, proposal, closed deal, reputation) | 5 (ReputationCalc, quota, evaluation TOCTOU, ConnectMe flow, CouponCode) | ~309 |
| **content** | ~180 | ~35 | 3 (upload video, moderation, playback grant) | 3 (video lock 90d, visibility grant revoke, metrics isolation) | ~221 |
| **assembly** | ~220 | ~45 | 3 (create assembly, cast vote, homologate) | 4 (Vote TOCTOU, Quorum by fraction, Proxy rules, 15min absence) | ~272 |
| **cross-domain** | ~100 | ~50 (inclui 100+ cross-tenant) | — | 1 (ABAC engine 400 casos) | ~151 |
| **core/pkg** | ~80 | — | — | 2 (ULID, Clock) | ~82 |
| **Total** | **~1300** | **~275** | **~18** | **~23 properties** | **~1616** |

Proporção: 1300/(1300+275+18) = **81% unit**, 17% integration, 1% E2E puro + 1% PBT — alinhado com pirâmide 70-20-10 considerando que PBT é contado em unit para CI.

---

## 4. Gates CI duros (não negociável)

Toda PR precisa passar **todos**:

```bash
go build ./...                                  # compila
go vet ./...                                    # static analysis
go test -race -count=1 -shuffle=on ./...        # unit + race + shuffle
go test -tags=integration -count=1 ./...        # integration com containers
gosec -severity=high -exit-status ./...         # SAST (0 findings HIGH)
govulncheck ./...                               # CVE (0 em reachable)
go tool cover -func=coverage.out \
  | python3 scripts/check_coverage.py \
    --thresholds=.coverage.yml                  # thresholds por camada
```

Web (SolidJS):

```bash
bun run typecheck
bun run test          # vitest
bun run build
bun run check         # biome
```

Mobile (Flutter):

```bash
flutter analyze
flutter test
dart run build_runner build --delete-conflicting-outputs  # se @injectable mexido
```

### Pipeline CI exemplo

```yaml
name: CI
on: [push, pull_request]
jobs:
  backend:
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with: { go-version: '1.26' }
      - run: go build ./...
      - run: go vet ./...
      - run: go test -race -count=1 -shuffle=on -coverprofile=coverage.out ./...
      - run: python3 scripts/check_coverage.py
      - run: gosec -severity=high -exit-status ./...
      - run: govulncheck ./...
  backend-integration:
    needs: backend
    services: { postgres: { image: postgres:18 }, redis: { image: redis:7 } }
    steps:
      - run: go test -tags=integration -count=1 ./...
```

---

## 5. Filosofia TDD — ciclo Red → Green → Refactor

```
1. RED       — escreve teste que falha (descreve comportamento/invariante desejado)
2. GREEN     — implementa o mínimo pra passar (sem over-engineer)
3. REFACTOR  — melhora código mantendo verde
```

**Exceções permitidas** (raras, documentadas em [[../STATE]] como D-###):
- Spike exploratório pra validar SDK externo — spike **descartado**, impl final nasce do zero TDD.
- Refatoração puramente estética (rename, move) — testes existentes cobrem.
- Migration SQL — o teste é a migration rodar `up`/`down` em integração.

---

## 6. Test isolation

- **Cada teste**: estado isolado — `TruncateAll()` em integration; fresh stubs em unit.
- **Sem shared state** entre testes (sem vars globais carregadas em `TestMain`).
- **Sem ordem dependente**: `go test -shuffle=on` precisa passar.
- **Race detection sempre**: `-race` em todos os runs.

---

## 7. Factories vs fixtures

### Factories (preferido)

```go
func NewUserFactory() UserFactory {
    return UserFactory{id: ULIDNew(), email: faker.Email(), cpf: faker.CPF()}
}
func (f UserFactory) WithRole(r Role) UserFactory { f.role = r; return f }
func (f UserFactory) Build() *User { /* chama NewUser */ }
```

### Fixtures estáticas (proibido)
Quebram ao mínimo refactor; fresh state > reuso.

---

## 8. Flaky test policy

1. Detectado → **quarentena** imediata com `t.Skip("flaky, ticket DT-###")`.
2. Ticket DT-### em [[../STATE]] com dump + repro.
3. SLA 1 sprint pra resolver.
4. **Nunca** `go test -count=100 || true` como "solução".

---

## 9. Mutation testing (piloto M2+)

- `go-mutesting` em `domain/` de identity (piloto M2).
- Meta inicial: kill rate ≥ 60%.
- Expandir se valor comprovado.

---

## 10. Chaos engineering (M3+)

Derivado de [[../13-research/netflix/_destilado]] §3:

- **M1-M2**: não introduzir. Focar em pirâmide + PBT + cross-tenant.
- **M3**: latency injection em staging (Toxiproxy) validando CB + fallback.
- **M4+**: Chaos Monkey blast=1 pod em prod com kill-switch.

Pré-requisitos (DoD pra introduzir):
- [ ] SLO definido ≥ 99.5% por BC.
- [ ] Alerting em burn rate.
- [ ] Runbooks publicados.
- [ ] Rollback automatizado no CD.

---

## 11. Dados em testes (LGPD)

**Proibido**:
- Dados reais de usuários.
- Tokens reais.
- Keys reais.

**Permitido**:
- `github.com/go-faker/faker/v4`.
- CPF válido algorítmico (módulo-11) **gerado**, nunca copiado.
- Seed determinístico pra reproduzir flakes.

---

## 12. Ferramentas

| Tool | Finalidade |
|---|---|
| `go test` | unit + integration |
| `pgregory.net/rapid@v1.2.0` | PBT |
| `testcontainers/testcontainers-go` | PG/Redis/NATS real em tests |
| `stretchr/testify` | asserts (`require`) |
| `go-faker/faker` | dados fake |
| `gosec` (OWASP) | SAST |
| `golang.org/x/vuln/cmd/govulncheck` | CVE reachable |
| `Playwright` | E2E web |
| `patrol` / `integration_test` | E2E Flutter |
| `k6` | load |
| `Toxiproxy` | chaos staging (M3+) |

---

## 13. Links

- [[_moc]]
- [[unit]]
- [[integration]]
- [[e2e]]
- [[load]]
- [[security]]
- [[pbt]]
- [[coverage-thresholds]]
- [[../07-quality/_moc]]
- [[../07-quality/PATTERNS]]
- [[../07-quality/antipatterns]]
- [[../01-domain/invariants]]
- [[../08-security/cve-process]]
- [[../../vault/10-knowledge/testing/testing-strategy]]
