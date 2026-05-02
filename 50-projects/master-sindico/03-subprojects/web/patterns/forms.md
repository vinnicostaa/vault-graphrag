---
title: Pattern — Forms (convenções)
type: pattern
tags:
  - pattern
  - ui
  - web
  - master-sindico
  - forms
created: '2026-04-27'
updated: '2026-04-27'
---

# Pattern — Forms (convenções)

Convenções para formulários no Master Síndico web (SolidJS + Kobalte + Zod).

## Estrutura canônica

- **Validação**: Zod schema co-localizada com o form. Erros aparecem **on blur** (não on change) para reduzir ruído.
- **Submit**: button único principal (`Salvar`/`Confirmar`); secundário (`Cancelar`) ao lado.
- **Loading state**: button com spinner + `aria-busy`; campos disabled durante submit.
- **Error state**: banner vermelho no topo se erro server-side; campos com erro inline destacam.
- **Success**: toast + redirect (ou inline confirmation se sem redirect).
- **Acessibilidade**: `<label>` associado, `aria-invalid`, `aria-describedby` apontando para erro, focus management automático para 1º erro on submit.

## Padrões específicos

- **Auto-save** (rascunho): forms longos (Plano Diretor, Connect Me) salvam draft a cada 30s ou on blur do campo.
- **Multi-step**: barra de progresso + back/next; estado em URL (`?step=2`) para deep-link.
- **Confirmação destrutiva**: modal `AlertDialog` Kobalte para delete/cancelamento.

## Quando evitar

- Modal nested em modal — quebra navegação por teclado e UX mobile. Em vez disso: full-screen sheet em mobile.

## Stack

SolidJS + Kobalte (Form, Field, Label, ErrorMessage, Dialog) + Zod + UnoCSS.

## Cross-refs

- [[legal-terms-checkbox]] — quando form tem aceite legal
- [[ui-states]] — loading/error/empty states
- [[recurring-required-action]] — quando form é obrigatório recorrente
- [[../../07-quality/PATTERNS]]

## Links

- [[_moc]]
