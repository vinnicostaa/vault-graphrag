---
title: "LMS3 — Página do Curso"
type: ui-screen
tags: [master-sindico, ui-catalog, web, lms, academia, curso]
project: master-sindico
persona: any
category: LMS
screen_id: LMS3
sub_produto: lms-academia
plan_requirement: plus
status: specification
stack: web
created: 2026-04-23
---

# LMS3 — Página do Curso

## Finalidade
Detalhe completo do curso: título, professor, descrição, carga horária, número de aulas, conteúdo programático. Botão "Inscrever-se" (gratuito) ou "Comprar e inscrever-se" (pago → Stripe).

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/contents/MÓDULO ACADEMIA MASTER SÍNDICO.md` §LMS3

## Persona e ABAC
- **Personas**: Síndico, Empresa, Morador (pagante), Admin
- **❌ BLOQUEADO**: Parceiro Vizinhança
- **Plan tier mínimo**: `plus` para matrícula completa (trial mostra preview)
- **ABAC action**: `content.course.read`

## Informações exibidas
- Título do curso
- Professor (perfil linkável)
- Descrição
- Carga horária
- Número de aulas
- Conteúdo programático (módulos + aulas)
- Pré-requisitos
- Avaliações de alunos (M4+)

## Botões
- **Inscrever-se** (gratuito) → [[LMS4]] primeira aula
- **Comprar e inscrever-se** (pago) → fluxo Stripe → [[LMS4]]

## Componentes
- `CourseHero` (título + professor + carga)
- `CourseDescription` (markdown)
- `SyllabusAccordion` (módulos / aulas)
- `EnrollButton` (estado depende: matriculado / comprar / 403)
- `LockedBanner` (se tier insuficiente: "Plano requerido: Plus")

## Estados
- **Loading**: skeleton.
- **Não matriculado**: CTA primário visível.
- **Matriculado (active)**: botão "Continuar curso" → [[LMS4]] na aula atual.
- **Concluído**: badge + link [[LMS5]] certificado.
- **403 Parceiro Vizinhança**: redirect com mensagem.
- **403 tier insuficiente**: banner "Plano requerido: Plus" + CTA upgrade.
- **Cursos pagos sem pagamento**: "Comprar" abre fluxo Stripe.
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Detalhe curso | `/api/v1/academy/courses/{id}` | GET |
| Matricular | `/api/v1/academy/courses/{id}/enroll` | POST |

## Regras de negócio críticas
- ABAC: se `role=local_company` → 403; se `tier` insuficiente → banner.
- Matrícula cria `Enrollment` (UNIQUE user_id+course_id).
- Curso pago dispara fluxo Stripe (`IPaymentGateway`); enrollment criado pós-pagamento confirmado.
- Trial mostra **preview da 1ª aula** (sem certificado).
- Lock 90d em curso publicado (RC-7 — protege integridade do certificado emitido).

## Ligações
- [[LMS2]] · [[LMS4]] · [[LMS5]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Trial preview: "1ª aula livre" precisa configuração por curso.
- Avaliações de alunos backlog M4+.
- Recommendation cross-domain (Req 23) backlog.
