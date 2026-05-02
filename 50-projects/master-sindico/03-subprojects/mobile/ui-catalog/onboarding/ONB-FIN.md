---
title: ONB-FIN — Final Genérico (Mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, onboarding]
project: master-sindico
persona: todos
screen_id: ONB-FIN
sub_produto: governanca-institucional
plan_requirement: any
status: specification
stack: mobile
created: 2026-04-24
---

# ONB-FIN — Final Genérico (Mobile)

## Finalidade

Tela terminal genérica pós-persona (síndico/empresa/parceiro) que explica status `pending_review` e próximos passos.

## Persona e ABAC

- Qualquer persona async (síndico, empresa, parceiro).

## Fluxo mobile

1. Ícone clock + mensagem "Seu cadastro está em análise."
2. Estimativa ("2-5 dias úteis").
3. Info "Você pode explorar conteúdo público enquanto aguarda."
4. CTA "Continuar" → home read-only.

## Widget Flutter principal

`OnbFinPage` reutilizável com variant enum.

## Platform-specific notes

Nenhuma específica.

## Estados

- Shown / Navigating.

## Offline behavior

- Conteúdo fixo.

## Push notification trigger

- Recebe `<persona>.approved.{user_id}` quando aprovado → toast + refresh.

## Links

- [[_moc]]
- [[ONB-SFIN]] (síndico variant)
- Onboarding morador = [[ONB-MFIN]] (auto-approve, não passa aqui)
