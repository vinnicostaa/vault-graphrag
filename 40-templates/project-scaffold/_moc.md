---
title: Project Scaffold — Template
type: template
tags: [template, scaffold, project, replicable]
created: 2026-04-22
---

# Project Scaffold — Template canônico

Esqueleto **replicável** para novos projetos em `vault/50-projects/<slug>/`. Instanciado via [[../../30-agents/playbooks/onboarding-novo-projeto]] após o dev responder ao questionário.

---

## Como usar

### Automático (recomendado)
Disparar [[../../30-agents/playbooks/onboarding-novo-projeto]]. Ele:
1. Entrevista o dev (visão, stack, padrões, não-negociáveis)
2. Copia esse scaffold pra `50-projects/<slug>/`
3. Preenche placeholders com as respostas
4. Registra D-000 em STATE
5. Aplica research-loop nas decisões em aberto

### Manual
```bash
cp -r vault/40-templates/project-scaffold vault/50-projects/<slug>
cd vault/50-projects/<slug>
# grep -rn '<TODO\|<PROJECT_NAME\|<SLUG' . → substituir tudo
# preencher CLAUDE.md, STATE.md, ROADMAP.md, STEERING.md, DoR-DoD.md
# completar cada _moc.md com conteúdo do projeto
```

---

## Estrutura gerada

```
50-projects/<slug>/
├── CLAUDE.md                       # contrato do agente (instância de [[../CLAUDE_md_template]])
├── _moc.md                         # MOC do projeto
├── STATE.md                        # D-### + DT-### + B-### (instância de [[../state-template]])
├── SESSION_CHARTER.md              # ordens da sessão (instância de [[../session-charter-template]])
├── STEERING.md                     # não-negociáveis
├── ROADMAP.md                      # marcos zero→prod
├── DoR-DoD.md                      # critérios ready/done
├── overview.md                     # visão resumida
│
├── 00-product/         _moc.md · vision.md · personas.md · business-model.md · glossary.md
├── 01-domain/          _moc.md · bounded-contexts.md · context-map.md · ubiquitous-language.md · business-rules.md · domain-rules.md · implementation-rules.md · invariants.md · domain-events.md
├── 02-architecture/
│   ├── _moc.md · system-overview.md · clean-arch-mapping.md · cqrs-layout.md · modules-layout.md · shared-providers.md · adapter-layer.md · data-flow.md · tech-stack-matrix.md · database-design.md · cache-strategy.md · observability-stack.md · infrastructure.md
│   ├── adr/            _moc.md (ADRs NNNN-slug.md a criar)
│   └── openapi/        _moc.md (*.yaml a criar)
├── 03-subprojects/     _moc.md
│   ├── backend/        README.md · architecture.md · patterns.md · requirements.md · tasks.md · security.md
│   ├── web/            README.md · architecture.md · patterns.md · requirements.md · tasks.md · security.md · design-system.md
│   └── mobile/         README.md · architecture.md · patterns.md · requirements.md · tasks.md · security.md
├── 04-requirements/    _moc.md · global.md · compliance.md
│   ├── functional/     _moc.md · (um .md por módulo)
│   └── non-functional/ performance · security · accessibility · i18n · scalability · reliability · observability · maintainability
├── 05-tasks/           _moc.md · global.md · backlog.md
│   ├── by-sprint/      _moc.md · sprint-N.md
│   └── by-module/      _moc.md · <modulo>.md
├── 06-execution/       _moc.md · STEPS.md · process-overview.md · development-cycle.md · qa-cycle.md · deploy-cycle.md · release-cycle.md
│   └── playbooks/      rollback · incident-response · hotfix · new-feature-kickoff · (custom)
├── 07-quality/         _moc.md · CLEAN_CODE.md · CLEAN_ARCH.md · CODE_CALISTHENICS.md · PATTERNS.md · code-review-checklist.md · naming-guide.md · logging-guide.md · error-handling-guide.md · api-design-guide.md · commit-pr-guide.md · antipatterns-applied.md
├── 08-security/        _moc.md · BEYOND_CORP.md · threat-model.md · attack-surface-map.md · auth-authz-matrix.md · cve-process.md · dependency-audit.md · secret-management.md · incident-response.md · owasp-top10.md · lgpd-compliance.md · data-retention.md
├── 09-testing/         _moc.md · test-strategy.md · coverage-thresholds.md · test-naming.md · unit-tests.md · integration-tests.md · property-based-tests.md · e2e-tests.md · contract-tests.md · load-tests.md · security-tests.md
├── 10-schedule/        _moc.md · cronograma-macro.md · milestones.md · sprint-calendar.md · release-calendar.md · ceremony-schedule.md
├── 11-audit/           _moc.md · AUDIT.md · dual-check-log.md · sprint-auditor-log.md · postmortems.md
│   └── audit-log/      _moc.md · YYYY-MM-DD-*.md
├── 12-docs/            _moc.md · README.md · adr-index.md · changelog.md · glossary.md · api-docs.md · contributing.md · support.md
│   ├── how-to/         setup-dev-env · criar-modulo · criar-use-case · adicionar-webhook · fazer-migration · rodar-testes-local · deploy-staging · rollback-producao
│   └── runbooks/       (por incidente prevalecente)
├── _handoffs/          (handoffs entre sessões)
└── _archive/           (versões antigas)
```

---

## Arquivos que o scaffold fornece (placeholders)

Cada `_moc.md` tem estrutura com TODOs e links pra arquivos a criar. Cada arquivo "opcional" tem um stub com frontmatter + seções.

### Principais templates derivados
- CLAUDE.md → instância de [[../CLAUDE_md_template]]
- STATE.md → instância de [[../state-template]]
- AUDIT.md → instância de [[../audit-template]]
- SESSION_CHARTER.md → instância de [[../session-charter-template]]

---

## O que o scaffold **não** preenche

- Stack específica (requer decisão do dev)
- Personas (requer discovery)
- Business rules (requer domínio)
- Bounded contexts (requer análise DDD)
- Roadmap (requer prioridade do dev)
- SLOs (requer definição de produto)

Tudo que é parametrizável ou opinionated **fica como TODO** marcado `<TODO: ...>`.

---

## Manutenção deste scaffold

- Refletir mudanças estruturais de [[../../50-projects/master-sindico/_moc]] aqui depois de consolidar
- Adicionar stubs novos quando descobrir gap genérico
- Remover MOCs / arquivos se cair em desuso

---

## Links

- [[../_moc]]
- [[../CLAUDE_md_template]]
- [[../../30-agents/playbooks/onboarding-novo-projeto]]
- [[../../50-projects/master-sindico/_moc]] — referência de projeto completo usando o scaffold
