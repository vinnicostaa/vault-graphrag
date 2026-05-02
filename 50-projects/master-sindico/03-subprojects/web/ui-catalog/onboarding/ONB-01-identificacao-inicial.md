---
title: ONB-01 — Identificação Inicial (comum)
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding]
project: master-sindico
persona: all
category: ONB
screen_id: ONB-01
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-01 — Identificação Inicial (comum a todas as personas)

## Finalidade

Tela 1 comum. Captura nome completo (ou nome da empresa) + e-mail. É o insumo para criar conta no Zitadel (provedor OIDC, D-056 agnóstico) e disparar e-mail de validação. Após esta tela, o fluxo diverge por persona.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] Tela 1 (literal)
- [[../../../../../../90-archive/master-sindico-pdfs-cliente-originais/ONBOARDING DOS PERFIS]]
- [[../../../../04-requirements/registration-data]]

## Persona e ABAC

- **Primária**: qualquer role selecionada em [[ONB-00-boas-vindas]].
- ABAC: `identity.draft.write` (pre-auth) — grava em Redis, não no banco definitivo.

## Fluxo da tela

1. Mensagem principal: **"Vamos começar pelo essencial."**
2. Mensagem de apoio (literal): _"Esses dados garantem seu acesso à plataforma, permitem salvar seu progresso e personalizar sua experiência desde o primeiro momento."_
3. Campos obrigatórios:
   - **Nome completo** (se morador/síndico/parceiro PF) **OU Nome da empresa** (se empresa/parceiro PJ).
   - **E-mail**.
4. Mensagem de tranquilização (literal): _"Você pode concluir o cadastro agora ou continuar depois. Suas informações ficam salvas automaticamente."_
5. Botão **[Continuar]** → dispara e-mail de validação + vai para tela 2 específica da persona.

## Componentes

- Form com validação inline (regex email, nome ≥ 2 chars).
- Banner auto-save (Redis).
- CTA persistente.

## Estados (loading/empty/error/success)

- **Error**: e-mail inválido · e-mail já cadastrado.
- **Success**: toast "Verifique seu e-mail para confirmar".

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Salvar draft + e-mail | `/api/v1/onboarding/draft` | POST | `{step:1, name, email}` | `{draft_id, verification_token_sent:true}` |

## Regras de negócio críticas

- Auto-save em Redis chave `onboarding:{user_id}:{step}`, **TTL 30 dias**.
- E-mail precisa ser único no Zitadel; se já existe, sugerir "fazer login ou recuperar senha" (mensagem canônica em [[../../../../04-requirements/registration-data]] §1.4).
- Disparo de validação: `IEmailProvider` (Resend default, D-056 agnóstico).
- Senha **NÃO** pedida aqui — Zitadel define senha em tela pós-validação (fluxo Stripe-like §2.5).

## Ligações

- Origem: [[ONB-00-boas-vindas]].
- Destino por persona:
  - Morador → [[ONB-M2]]
  - Síndico → [[ONB-S2]]
  - Empresa → [[ONB-E2]]
  - Parceiro → [[ONB-P2]]
- Relacionados: [[../../../../client-canvas/Fluxo Registro]].

## Gaps/ressalvas

- Validação de e-mail antes de prosseguir: decisão atual é **permitir prosseguir** e travar ativação final; token validado até aceitar termos na última tela.
- Internacionalização (i18n) pt-BR prioridade M1; en-US M3+.

## Links

- [[_moc]]
- [[ONB-00-boas-vindas]]
- [[../../../../04-requirements/registration-data]]
