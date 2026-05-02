---
title: Sprint 9 — Mobile (feature assembly full — fora escopo M1)
type: sprint-spec
tags: [master-sindico, sprint-9, mobile, tasks]
project: master-sindico
stack: mobile
sprint: 9
period: "2026-04-22 (sessão autônoma)"
status: completed
milestone_target: m3
created: 2026-04-24
---

# Sprint 9 Mobile — Feature assembly completa (fora escopo M1)

**Período**: 2026-04-22 (parte da sessão autônoma backend)
**Status**: ✅ concluído — porém **fora do escopo M1** (A-FASE10-DEV-004 🟡)
**Objetivo**: Implementar a feature assembly completa no app Flutter (domain + data + presentation + tests) — **adiantando M3**. Esse sprint documenta honestamente o desalinhamento com o roadmap: M1 mobile deveria priorizar timeline read + home morador (escopo vault), mas o que foi codado foi assembly.

## Escopo desta sprint (mobile)

Feature assembly: páginas de lista, detalhe, ciência de pauta, votação básica. Auth também recebe patch (`createdAt` hardcoded → parcial — DT-MB-007/008 continuam). Home segue placeholder. Timeline **ausente** — daí o finding A-FASE10-DEV-004.

## Tasks

| ID | Título | Status | Link detalhe | A-### relacionado | D-### |
|---|---|---|---|---|---|
| mobile-9.1 | `features/assembly/domain/entities/assembly.dart` + `agenda_item.dart` + `vote.dart` | ✅ | `Development/app/lib/features/assembly/domain/` | — | — |
| mobile-9.2 | `features/assembly/data/models/*.dart` + `datasources` remote + local | ✅ | `Development/app/lib/features/assembly/data/` | — | — |
| mobile-9.3 | `features/assembly/presentation/bloc/assembly_bloc.dart` + states/events | ✅ | `Development/app/lib/features/assembly/presentation/bloc/` | — | — |
| mobile-9.4 | Pages: lista (`/condominiums/:id/assemblies`), detalhe, ciência, votação | ✅ | idem presentation/pages | DT-MB-012 | — |
| mobile-9.5 | Tests unit + bloc + widget feature assembly | ✅ | `test/features/assembly/` | — | — |
| mobile-9.6 | `features/auth` patch `createdAt` (ainda hardcoded — DT-MB-006) | 🟡 parcial | `Development/app/lib/features/auth/` | — | — |
| mobile-9.7 | F26 Audit app/ qualidade — Dio CVE patched | ✅ | [[../../../SESSION_CHARTER#F26]] | — | — |
| mobile-9.8 | F29 Setup .env placeholders Flutter (`String.fromEnvironment`) | ✅ | [[../../../SESSION_CHARTER#F29]] | — | — |
| mobile-9.9 | Diagnóstico A-FASE10-DEV-004 🟡 (assembly fora escopo + timeline ausente) | ✅ registrado | [[../../../AUDIT#a-fase10-dev-004]] | A-FASE10-DEV-004 | — |
| mobile-9.10 | Diagnóstico A-FASE10-DEV-003 🟠 (D-048/049/050 não aplicadas) | ✅ registrado | [[../../../AUDIT#a-fase10-dev-003]] | A-FASE10-DEV-003 | D-048, D-049, D-050 |

## Dependências

- Sessão autônoma backend 22/04.
- Backend assembly foundation ([[../../../backend/tasks/by-sprint/sprint-7]]).
- Decisões: D-048/049/050 decididas no vault mas **não aplicadas** em código mobile.

## Entregáveis

- Feature assembly navegável no app Flutter (dev).
- 2 findings 🟠/🟡 registrados no AUDIT.

## Gates (`<verify>`)

- `fvm flutter analyze` ✅ (com avisos não-críticos)
- `fvm flutter test` ✅
- `fvm flutter build apk --debug` ✅
- `fvm flutter build ios --no-codesign` ✅

## Pós-sprint

- **O que deu certo**: feature real acabou entregue; estrutura Clean Arch se mostrou escalável.
- **Bloqueadores**: feature **errada** (assembly é M3; deveria ser timeline M1); stakeholder vê entrega desalinhada.
- **Dívidas criadas**: reescopar Sprint 10 — priorizar timeline read + home morador + auth completo; mover assembly para M3 ou formalizar mudança de escopo com D-### novo.
- **Próxima sprint**: [[sprint-10]] — M1 correto (auth OIDC completo + home shell + timeline read).

## Links

- [[sprint-0]]
- [[sprint-10]]
- [[../../../../_moc]]
- [[../../../../SESSION_CHARTER]]
- [[../../../../AUDIT]]
