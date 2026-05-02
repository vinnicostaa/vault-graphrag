---
title: Playbook — Dependency Audit
type: playbook
tags:
  - playbook
  - security
  - dependencies
  - cve
created: 2026-04-22T00:00:00.000Z
---

# Playbook — Dependency Audit

Auditoria periódica de dependências: estão atualizadas? têm CVE? licenças compatíveis? versão lockada bate com pinada? breaking changes na próxima major?

Roda ao **início de todo sprint**, em **PR que toca dependências**, e **manual** em incident response.

## Objetivos

1. Nenhuma dependência com CVE high/critical não mitigado
2. Toda dependência em major version **suportada** (não fim-de-vida)
3. Lockfile consistente com manifesto
4. Licença compatível com o projeto (sem GPL em código proprietário)
5. Trilha: cada bump de major registrada em STATE D-###

## Ferramentas por stack

### Go
```bash
# Updates disponíveis
go list -u -m all

# Vulnerabilidades
govulncheck ./...
# (Instala: go install golang.org/x/vuln/cmd/govulncheck@latest)

# Módulo outdated + breaking
go mod tidy
go mod verify

# Licença audit
go-licenses check ./...
# (Instala: go install github.com/google/go-licenses@latest)
```

### TypeScript / Node
```bash
npm outdated
npm audit
# ou
pnpm outdated
pnpm audit
# ou com Bun
bun outdated
```

Fontes externas:
- https://github.com/advisories — GitHub Advisory DB
- https://osv.dev — OSV Database
- https://snyk.io/vuln — Snyk DB (também em CI)

### Rust
```bash
cargo outdated
cargo audit
# (Instala: cargo install cargo-audit cargo-outdated)
cargo deny check
```

### Flutter / Dart
```bash
flutter pub outdated
dart pub deps --style=tree
# CVE check manual via osv-scanner contra pubspec.lock
osv-scanner -L pubspec.lock
```

### Container / SBOM
```bash
# Gerar SBOM
syft . -o spdx-json > sbom.json

# Scan vulnerabilidades
trivy fs .
trivy sbom sbom.json

# grype também funciona
grype dir:.
```

## Fluxo operacional

### 1. Coleta
Rodar todos os comandos aplicáveis e salvar output em `50-projects/<p>/11-audit/audit-log/YYYY-MM-DD-dep-audit.md`:

```markdown
---
title: Dep Audit — YYYY-MM-DD
type: audit-log
tags: [audit, dependency, <sprint-N>]
date: 2026-04-22
---

## Go — govulncheck
<output>

## Go — outdated
<output>

## TS — audit
<output>

...
```

### 2. Triagem
Classificar cada finding:

| Severidade | SLA | Ação |
|---|---|---|
| Critical | 24h | Hotfix imediato; bloqueia deploy |
| High | 7d | Plano de mitigação no mesmo sprint |
| Medium | 30d | Task no backlog priorizado |
| Low | 90d | Task no backlog |

### 3. Registro
Para cada finding não-trivial:
- Abrir `A-###` em AUDIT.md com link pro log
- Se mitigação requer decisão de versão: abrir `D-###` em STATE.md

### 4. Mitigação
- **Patch disponível**: atualizar (patch bump), rodar testes, PR
- **Minor bump** com breaking: revisar changelog, ajustar código, PR
- **Major bump**: playbook [[research-loop]] + [[dual-check]] antes
- **Sem patch**: aplicar workaround (feature flag, input validation, revokar feature), abrir issue upstream se aplicável

### 5. Regressão
Após mitigação, re-rodar o scan específico pra confirmar zero finding.

## Integração com CI

O workflow de CI deve falhar se:
- `govulncheck` retorna CVE com severity >= high
- `npm audit` retorna vuln com severity >= high
- `cargo audit` retorna finding com severity >= high
- Lockfile drift (manifesto pede X, lock tem Y)

Exceção (permitir merge) só via:
- Label `risk-accepted` + link pro D-### em STATE com sprint alvo de resolução
- Approver humano com motivo documentado

## Frequência

| Evento | Ação |
|---|---|
| Início de cada sprint | Audit completo (todos os stacks) |
| PR que toca `go.mod` / `package.json` / `Cargo.toml` / `pubspec.yaml` | Audit incremental só do stack afetado |
| Alerta de CVE externa (GitHub Advisory email) | Trigger imediato |
| Quinzenal (cron automatizado, se configurado) | Audit completo |
| Incident response | Audit completo |

## O que **não** fazer

- ❌ Suprimir finding sem registrar motivo e owner
- ❌ Fazer major bump em sexta-feira à tarde
- ❌ Bumpar só "porque tem update" sem necessidade — custo de regressão existe
- ❌ Ignorar patches de segurança por "causa barulho no lockfile"
- ❌ Mexer em vendored deps sem registrar em STATE

## Licenças: matriz de compatibilidade

| Licença origem | Produto proprietário SaaS | Biblioteca open | Cliente distribuído |
|---|---|---|---|
| MIT, BSD-2/3, ISC, Apache-2.0 | ✅ | ✅ | ✅ |
| LGPL-2.1/3 | ✅ dynamic link | ✅ | ⚠️ cuidado static link |
| GPL-2.0/3 | ❌ | ⚠️ força GPL | ❌ |
| AGPL-3.0 | ❌ (SaaS conta como distribuição) | ⚠️ | ❌ |
| Proprietária / sem licença | ❌ | ❌ | ❌ |

Usar `go-licenses` / `license-checker` (npm) / `cargo-deny` pra enforçar no CI.

## Links

- [[_moc]]
- [[cve-scan]]
- [[research-loop]]
- [[../../10-knowledge/security/_moc]]
- [[../../CLAUDE]]
