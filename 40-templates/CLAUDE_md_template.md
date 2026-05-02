---
title: CLAUDE.md — <PROJECT_NAME>
type: agent-contract
tags: [claude, agent, project-contract]
project: <PROJECT_SLUG>
created: <YYYY-MM-DD>
template_version: 1.0
---

<!--
  TEMPLATE genérico de CLAUDE.md para novos projetos em vault/50-projects/<slug>/.
  Instanciado pelo playbook [[30-agents/playbooks/onboarding-novo-projeto]].
  Campos entre <CHEVRONS> e blocos `<TODO: ...>` devem ser substituídos.
  Campos opcionais: deixar o bloco inteiro ou remover.
-->

# CLAUDE.md — <PROJECT_NAME>

Contrato do agente para o projeto **<PROJECT_NAME>**. Complementa `vault/CLAUDE.md` (raiz do vault) com regras específicas deste projeto.

> Ordem de leitura toda sessão: `vault/CLAUDE.md` → este arquivo → `STATE.md` → `11-audit/AUDIT.md` → `SESSION_CHARTER.md` → `05-tasks/by-sprint/sprint-<N>.md`.

---

## 1. Visão do produto

- **O que é**: <1-2 frases>
- **Para quem**: <personas principais>
- **Problema que resolve**: <1 frase>
- **Por quê agora**: <timing / mercado>
- **North-star metrics (6 meses)**: <1-3 métricas>
- **Fora de escopo** (guard contra scope-creep): <lista curta>

Detalhamento em [[00-product/vision]], [[00-product/personas]], [[00-product/business-model]].

---

## 2. Sub-projetos

<TODO: listar cada sub-projeto com responsabilidade, stack e owner>

| Sub-projeto | Responsabilidade | Stack | Owner | Pasta |
|---|---|---|---|---|
| `backend` | <...> | <...> | <...> | [[03-subprojects/backend/README]] |
| `web` | <...> | <...> | <...> | [[03-subprojects/web/README]] |
| `mobile` | <...> | <...> | <...> | [[03-subprojects/mobile/README]] |

---

## 3. Stack confirmada (e decisões pendentes)

<TODO: preencher com o que já foi decidido; marcar pendências como D-### no STATE>

| Camada | Tech | Status |
|---|---|---|
| Backend linguagem | <Go 1.26 / Node 22 / Rust 1.86 / ...> | ✅ D-00X |
| Framework API | <Gin / Fastify / Axum / ...> | ✅ |
| ORM / Query | <sqlc / Prisma / SeaORM / ...> | ✅ |
| Auth | <Zitadel / Auth0 / Clerk / ...> | ✅ D-00X |
| DB primária | <Postgres 18 / ...> | ✅ |
| Cache | <Redis 7 / ...> | ✅ |
| Fila / eventos | <NATS JetStream / SQS / ...> | ✅ |
| Frontend | <SolidJS / React / Vue / ...> | ⏳ D-00X |
| Mobile | <Flutter / React Native / Nativo / ...> | ⏳ D-00X |
| Hospedagem | <Railway / AWS ECS / Vercel / ...> | ✅ |
| Observability | <OpenTelemetry + Grafana / Sentry / ...> | ✅ |

---

## 4. Arquitetura e padrões (constituição do projeto)

Herda a constituição técnica de [[../../CLAUDE#8-padrões-de-engenharia-inegociáveis-constituição-técnica]]. Especializações:

### Padrões obrigatórios
- **<padrão 1>** — porquê aqui (link pra `10-knowledge/patterns/...`)
- **<padrão 2>** — ...

### Padrões proibidos neste projeto
- **<padrão proibido 1>** — porquê (link pra STATE D-### ou antipattern)

### Camadas Clean Architecture aplicadas
Ver [[02-architecture/clean-arch-mapping]].

```
<diagrama / estrutura de pastas real do repo>
```

---

## 5. Bounded contexts (DDD)

<TODO: listar contextos e status>

| Contexto | Responsabilidade | Status |
|---|---|---|
| `<ctx-1>` | <...> | Implementado / Sprint N |
| `<ctx-2>` | <...> | ... |

Detalhes em [[01-domain/bounded-contexts]] e [[01-domain/context-map]].

---

## 6. Fluxo canônico de execução (SDD 5-fase)

Ver [[../../10-knowledge/methodology/sdd-workflow]] e [[../../30-agents/playbooks/plan-and-execute]].

- **Discuss**: ler `05-tasks/by-sprint/sprint-<N>.md`, `04-requirements/...`, `02-architecture/...` antes de editar
- **Plan**: escrever `<plan>` com `<verify>` (contrato, não sugestão)
- **Execute**: ordem canônica (migration → query → entity → repo → use case → handler → wire-up → testes)
- **Verify**: build + lint + test unit + test integration + coverage + gosec/vuln
- **Ship**: commit imperativo, PR com Release Notes, review

Gate 0 (toda sessão): abrir `11-audit/AUDIT.md`. Itens 🔴🟡 **bloqueiam** task nova.

---

## 7. Coverage thresholds (`.coverage.yml`)

<TODO: ajustar se divergir do default; default é o recomendado>

| Camada | Mínimo |
|---|---|
| `domain/` | 95% |
| `application/` | 90% |
| `infrastructure/database/` | 85% |
| `infrastructure/http/` | 75% |
| `shared/middleware/` | 90% |
| `core/`, `pkg/` | 95% |
| **Global** | ≥ 85% |

---

## 8. Security posture (BeyondCorp adaptado)

- **Zero trust**: toda request autenticada e autorizada; sem "rede interna confiável"
- **ABAC**: admin_bypass → role → roleType → planTier → tenant_match
- **Tenant isolation**: `WHERE tenant_id = $ctx.tenant_id` em toda query
- **Device fingerprint**: 1 dispositivo ativo por conta (configurável)
- **PII nunca em logs**: mascarar CPF/CNPJ/email/tokens
- **Secrets**: nunca em código/commit/log/erro; só em env vars gitignored

Detalhamento em [[08-security/BEYOND_CORP]], [[08-security/threat-model]].

---

## 9. MCPs e plugins relevantes

<TODO: marcar os que este projeto usa>

| Recurso | Uso neste projeto |
|---|---|
| `obsidian` (MCP) | Ler/escrever notas do vault |
| `context7` (plugin) | Docs oficiais de libs (obrigatório antes de usar SDK) |
| `postgres` (MCP) | Inspeção read-only de schema |
| `github` (MCP) | Issues, PRs |
| `stripe` (plugin) | <se billing> |
| `figma` (plugin) | <se UI-heavy> |
| `railway` (plugin) | <se Railway> |

---

## 10. Modo autônomo — limites

Herda [[../../30-agents/autonomous-execution]]. Categorias **destrutivas** que **pausam** o modo autônomo e exigem confirmação humana:

1. `DROP TABLE / DROP COLUMN` ou migration destrutiva
2. Deploy em produção
3. `git push --force` em branch compartilhada
4. Remover dependência crítica
5. Operação em dado pessoal de usuário (LGPD)

Em dúvida técnica não-destrutiva (escolha de lib, pattern): **NÃO** pausar — rodar [[../../30-agents/playbooks/research-loop]] + [[../../30-agents/playbooks/dual-check]] e registrar D-###.

---

## 11. Fontes de verdade (hierarquia de consulta)

1. `04-requirements/**` + `05-tasks/**` — canônico para o quê implementar
2. `02-architecture/**` — canônico para como
3. `01-domain/**` — regras de negócio e invariantes
4. `STATE.md` + `11-audit/AUDIT.md` — decisões vivas e findings
5. Código real — quando divergir da doc, código é o estado atual; atualizar doc antes de implementar mais
6. `legacy-reference` (se há legado) — **referência de domínio**, não template de código

---

## 12. Idioma

- Corpo de notas: **pt-BR**
- Frontmatter, identifiers: **inglês**
- APIs públicas com docstring bilíngue (EN em cima, pt-BR embaixo) — templates de código

---

## 13. Checklist de fim de sessão

- [ ] `STATE.md` atualizado com D-### novos
- [ ] `11-audit/AUDIT.md` com findings novos/fechados
- [ ] `SESSION_CHARTER.md` registra o que foi feito
- [ ] Notas criadas com frontmatter + ≥ 2 `[[links]]`
- [ ] MOCs das pastas impactadas atualizados
- [ ] Gates verdes (build, lint, test, coverage, security scan)
- [ ] Docs sincronizadas com código (README, OpenAPI, CHANGELOG)

---

## 14. Links essenciais do projeto

- [[_moc]] — mapa do projeto
- [[STATE]] — decisões vivas
- [[11-audit/AUDIT]] — findings abertos
- [[SESSION_CHARTER]] — sessão atual
- [[STEERING]] — não-negociáveis
- [[ROADMAP]] — cronograma macro
- [[DoR-DoD]] — definition of ready/done
- [[00-product/vision]] — visão do produto
- [[01-domain/bounded-contexts]]
- [[02-architecture/system-overview]]
- [[06-execution/STEPS]] — zero → prod
- [[07-quality/PATTERNS]]
- [[08-security/BEYOND_CORP]]
- [[09-testing/test-strategy]]

## 15. Links externos (vault)

- [[../../CLAUDE]] — contrato raiz do vault
- [[../../30-agents/playbooks/_moc]] — playbooks obrigatórios
- [[../../10-knowledge/_moc]] — conhecimento técnico
- [[../../20-stacks/_moc]] — templates por stack
- [[../../40-templates/_moc]] — templates replicáveis
