---
title: Coverage thresholds — Master Síndico
type: guide
tags: [testing, coverage, thresholds, ci-gate, master-sindico, v2]
source: vault/10-knowledge/testing/testing-strategy.md + 90-ingested
created: 2026-04-23
updated: 2026-04-23
aliases: [coverage-thresholds-master-sindico]
---

# Coverage thresholds — Master Síndico v2

Coverage thresholds **duros por camada**, validados em CI. Violação bloqueia merge. Arquivo canônico `.coverage.yml` na raiz do `backend/` (pós-migração) define limites e exclusões.

> Herda [[../CLAUDE]] §8 e [[../STEERING]] §25. Pirâmide 70-20-10 em [[test-strategy]].

---

## 1. Thresholds por camada

| Camada | Mínimo | Racional |
|---|---|---|
| `internal/modules/*/domain/` | **95%** | Regras de negócio + invariantes. Pouco código, todos os paths. |
| `internal/modules/*/application/` | **90%** | Use cases; cobre comandos, queries, error paths |
| `internal/modules/*/infrastructure/database/` | **85%** | Repositories + sqlc; happy + erros de driver |
| `internal/modules/*/infrastructure/http/` | **75%** | Handlers — contrato + decode + auth guards |
| `internal/modules/*/infrastructure/providers/` | **85%** | Adapter SDK externo + mapeamento erros |
| `internal/shared/middleware/` | **90%** | Segurança: todas as branches (ABAC, rate limit, authenticate) |
| `internal/shared/providers/` | **90%** | Infra compartilhada (Redis wrapper, pg pool) |
| `core/` | **95%** | Contracts + errors — invariantes do framework |
| `pkg/` | **95%** | Utilidades puras (logger, money, utils) |
| **Global** | **≥ 85%** | Sanity check |

---

## 2. Arquivo canônico `.coverage.yml`

```yaml
# backend/.coverage.yml
version: 1
global:
  threshold: 85

# Camadas específicas (caminho ant-style)
paths:
  - match: "internal/modules/*/domain/**"
    threshold: 95
    reason: "regras de negócio + invariantes"

  - match: "internal/modules/*/application/**"
    threshold: 90
    reason: "use cases (CQRS)"

  - match: "internal/modules/*/infrastructure/database/**"
    threshold: 85

  - match: "internal/modules/*/infrastructure/http/**"
    threshold: 75

  - match: "internal/modules/*/infrastructure/providers/**"
    threshold: 85

  - match: "internal/shared/middleware/**"
    threshold: 90

  - match: "internal/shared/providers/**"
    threshold: 90

  - match: "core/**"
    threshold: 95

  - match: "pkg/**"
    threshold: 95

exclude:
  # Generated ou trivial — não contabiliza no denominador
  - "**/generated/**"
  - "**/sqlc/**"
  - "**/mocks/**"
  - "cmd/**"
  - "**/*_mock.go"
  - "**/*_gen.go"
  - "**/testutil/**"
  - "**/testdata/**"

enforcement:
  fail_under_threshold: true
  fail_on_missing_tests: true  # arquivo sem test file → fail (com exceção em exclude)

reporters:
  - type: text
  - type: html
    output: coverage/index.html
  - type: junit
    output: coverage/junit.xml
```

---

## 3. Script de verificação em CI

```python
# scripts/check_coverage.py
"""
Parse `go tool cover -func=coverage.out` e valida contra .coverage.yml.
Saída: 0 se OK; 1 se qualquer camada abaixo do threshold.
"""
import subprocess, yaml, fnmatch, sys

def parse_cover_func():
    out = subprocess.check_output(['go', 'tool', 'cover', '-func=coverage.out']).decode()
    # linhas: "path/to/file.go:12:\tFuncName\t95.0%"
    results = {}
    for line in out.splitlines():
        if line.startswith('total:') or '\t' not in line: continue
        path, _, pct = line.split('\t')
        path = path.split(':')[0]
        results.setdefault(path, []).append(float(pct.strip().strip('%')))
    return {p: sum(v)/len(v) for p, v in results.items()}

def main():
    cfg = yaml.safe_load(open('.coverage.yml'))
    results = parse_cover_func()
    failures = []
    for path, pct in results.items():
        if any(fnmatch.fnmatch(path, ex) for ex in cfg.get('exclude', [])): continue
        threshold = cfg['global']['threshold']
        for rule in cfg.get('paths', []):
            if fnmatch.fnmatch(path, rule['match']):
                threshold = rule['threshold']
                break
        if pct < threshold:
            failures.append((path, pct, threshold))

    for path, pct, th in failures:
        print(f"FAIL {path}: {pct:.1f}% < {th}%")
    sys.exit(1 if failures else 0)

main()
```

Rodar:

```bash
go test -race -count=1 -coverprofile=coverage.out ./...
python3 scripts/check_coverage.py
```

---

## 4. Gate de CI

```yaml
# .github/workflows/ci.yml
jobs:
  coverage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with: { go-version: '1.26' }
      - run: go build ./...
      - run: go vet ./...
      - run: go test -race -count=1 -coverprofile=coverage.out ./...
      - run: python3 scripts/check_coverage.py
      - uses: actions/upload-artifact@v4
        if: always()
        with: { name: coverage-html, path: coverage/ }
```

---

## 5. Integration coverage

Tests com tag `integration` rodam separados e geram `coverage_integration.out`. Merge via `gocovmerge`:

```bash
go test -tags=integration -count=1 -coverprofile=coverage_integration.out ./...
gocovmerge coverage.out coverage_integration.out > coverage_merged.out
python3 scripts/check_coverage.py --profile=coverage_merged.out
```

Thresholds aplicam sobre **merged** (unit + integration somados).

---

## 6. Exceções permitidas

Exceção a threshold só por **D-###** em [[../STATE]] com prazo:

```yaml
exceptions:
  - path: "internal/modules/commercial/infrastructure/database/migrations/**"
    reason: "migrações testadas via up/down em integration"
    threshold: 0
    owner: dev
    valid_until: 2026-12-31
    decision_ref: D-042
```

---

## 7. Como falhar CI (resumo)

| Condição | Ação |
|---|---|
| `go build` quebra | Falha imediata |
| `go vet` warn | Falha |
| `go test -race` falha ou race detectado | Falha |
| `go test -tags=integration` falha | Falha |
| Qualquer camada abaixo do threshold | Falha |
| Global < 85% | Falha |
| Test file ausente em módulo não-excluído | Falha (fail_on_missing_tests) |
| `gosec -severity=high` > 0 | Falha |
| `govulncheck` reachable CVE | Falha |

---

## 8. Medição adicional (benchmark, não-gate)

- **Mutation kill rate** (`go-mutesting`, M2+ piloto identity) — meta 60% no `domain/`.
- **Coverage branch** (não-gate) — `go test -covermode=count` + `-coverprofile`.
- **Flaky rate** (% de retries em CI) — dashboard Grafana; meta < 1%.

---

## 9. Coverage report HTML

Gerar localmente:

```bash
go tool cover -html=coverage.out -o coverage/index.html
```

Em CI, artifact `coverage-html` anexa ao PR — review pode navegar arquivos específicos.

---

## 10. Anti-patterns coverage

| Sintoma | Fix |
|---|---|
| `exclude: "**/*"` para bypass | Blocker; D-### obrigatório com prazo |
| `t.Skip` sistemático pra inflar coverage | Flaky policy + ticket DT-### |
| Tests que rodam sem asserts | Code review manual |
| Coverage acima do threshold mas 0 asserts críticos | Mutation test piloto M2+ |
| Ignore `cmd/` sem motivo | Bootstrap é integration; aceitar baixo |

---

## 11. Checklist review coverage

- [ ] `.coverage.yml` reflete a realidade de exclusões?
- [ ] Threshold da camada coerente com tabela §1?
- [ ] Exceções documentadas com D-###?
- [ ] Mutation test piloto no domain de identity (M2+)?
- [ ] Merged unit+integration report no PR?

---

## 12. Links

- [[_moc]]
- [[test-strategy]]
- [[unit]]
- [[integration]]
- [[pbt]]
- [[security]]
- [[../07-quality/CLEAN_ARCH]]
- [[../CLAUDE]]
- [[../STEERING]]
- [[../../vault/10-knowledge/testing/testing-strategy]]
