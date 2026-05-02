---
title: ONB-00 — Boas-vindas (Seleção de Perfil)
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding]
project: master-sindico
persona: all
category: ONB
screen_id: ONB-00
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-00 — Boas-vindas e Seleção de Perfil

## Finalidade

Tela de entrada única da plataforma Master Síndico. Apresenta proposta de valor e pede ao usuário a **seleção do papel** que define qual fluxo de onboarding ele vai seguir: Morador, Síndico, Empresa ou Parceiro da Vizinhança. Base do onboarding parametrizado (§2.5 excopo).

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] Tela 0 (literal)
- [[../../../../../../90-archive/master-sindico-pdfs-cliente-originais/ONBOARDING DOS PERFIS]]
- [[../../../../00-product/personas]]

## Persona e ABAC

- **Primária**: nenhuma ainda — usuário anônimo.
- **Secundária**: Zitadel emite `role` apenas após [[ONB-01-identificacao-inicial]].

## Fluxo da tela

1. Mensagem principal: **"Bem-vindo à Master Síndico."**
2. Mensagem de apoio (literal): _"A Master Síndico é um ambiente digital onde a gestão condominial deixa de ser invisível. Aqui, informações são organizadas, decisões ganham histórico e relações se tornam mais claras."_
3. Mensagem de orientação (literal): _"Para começar, precisamos entender qual é o seu papel no universo condominial."_
4. Seleção única obrigatória (cards/radio):
   - Morador
   - Síndico
   - Empresa do Mercado Condominial
   - Comércio Local — Parceiro da Vizinhança
5. Botão **[Continuar]** → navega para [[ONB-01-identificacao-inicial]].
6. Se URL parametrizada (`?role=sindico&plan=pro` — §2.5 excopo) → pula esta tela e vai direto para Tela 1 com role já fixada.

## Componentes

- 4 cards grandes com ícone + descrição (shadcn/solid-ui).
- Stepper discreto no topo (passo 1 de N, N varia por persona).
- Progress indicator persistente (Redis auto-save).

## Estados (loading/empty/error/success)

- **Empty**: CTA claro "Escolha seu papel para começar".
- **Error**: nenhum card selecionado ao clicar continuar → alert inline.
- **Success**: transição suave.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Salvar draft | `/api/v1/onboarding/draft` | POST | `{step:0, role}` | `{draft_id, ttl_seconds}` |

## Regras de negócio críticas

- Auto-save em Redis com **TTL 30 dias** (chave `onboarding:{draft_id}:{step}`).
- Role selecionada aqui vira `claim` em Zitadel após conclusão (imutável — IDN-029).
- URL parametrizada: respeita `role` e `plan` via searchParams (Stripe-like — §2.5 excopo).
- Admin **não** auto-registra (IDN-001) — sem card admin aqui.

## Ligações

- Destino: [[ONB-01-identificacao-inicial]].
- Relacionados: [[../../../../00-product/personas]], [[../../../../client-canvas/Fluxo Registro]].

## Gaps/ressalvas

- "Marketing" (agência) não aparece como persona auto-registrável — é empresa que emite token de agência (XD-). Fica fora deste seletor.
- Decision 12 confirma Parceiro aceita CPF ou CNPJ — tratado em [[ONB-P2]].

## Links

- [[_moc]]
- [[ONB-01-identificacao-inicial]]
- [[../../../../00-product/personas]]
