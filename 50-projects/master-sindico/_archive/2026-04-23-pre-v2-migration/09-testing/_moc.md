---
title: MOC — 09-testing
type: moc
tags: [moc, master-sindico, testing, qa]
project: master-sindico
created: 2026-04-22
---

# 09-testing — Estratégia e práticas de teste

Pirâmide testes + PBT + property-based + integration + E2E + load + security.

## Notas canônicas

### Estratégia
- [[test-strategy]] — pirâmide: 70% unit, 20% integration, 10% E2E; PBT horizontal em regras críticas
- [[coverage-thresholds]] — domain 95%, app 90%, infra 85%, http 75%; tooling `.coverage.yml`
- [[test-naming]] — Given/When/Then em Go via `t.Run`; arquivos `*_test.go` ao lado; `_integration_test.go` com tag `integration`

### Por tipo
- [[unit-tests]] — table-driven, happy + error paths; mocks/stubs manuais (sem mocktail); `pgregory.net/rapid` pra PBT
- [[integration-tests]] — `testcontainers-go` (Postgres real); TruncateAll entre tests; `internal/shared/testutil`
- [[property-based-tests]] — regras: fração ideal, quotas Connect Me, ABAC matrix, money BRL, tenant isolation
- [[e2e-tests]] — Playwright para web; Appium / Flutter integration para mobile; fluxos críticos
- [[contract-tests]] — OpenAPI lint via spectral; contract test via dredd ou schemathesis
- [[load-tests]] — k6; cenários: 10k concurrent users assembleia; 1k req/s auth; 100 req/s Connect Me
- [[security-tests]] — gosec, govulncheck, Semgrep rules customizadas, ZAP baseline, Trivy

### Práticas
- [[test-data]] — factories (não fixtures estáticas); dados aleatórios com seed; dados reais proibidos em testes
- [[test-isolation]] — cada teste isolado; sem side-effect entre testes
- [[flaky-test-policy]] — quarentena + root cause; nunca skip sem ticket
- [[mutation-testing]] — piloto em `domain/` de identity com `go-mutesting`

## Gates CI (obrigatórios)

```bash
go build ./...
go vet ./...
go test -race -count=1 ./...
go test -tags=integration -count=1 ./...
go test -coverprofile=coverage.out ./...
go tool cover -func=coverage.out
gosec -severity=high ./...
govulncheck ./...
```

Falha de qualquer uma = bloqueia merge.

## Fontes

- [[../.kiro/steering/testing]] *(backend)* — testing strategy original
- [[../../../10-knowledge/testing/_moc]]
- [[../../../10-knowledge/testing/testing-strategy]]

## Vizinhos

- [[../_moc]]
- [[../06-execution/_moc]] — QA cycle
- [[../07-quality/_moc]]
- [[../08-security/_moc]] — security testing
- [[../11-audit/_moc]] — findings de teste viram A-###
