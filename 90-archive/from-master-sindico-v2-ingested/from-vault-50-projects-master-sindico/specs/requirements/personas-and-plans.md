---
title: Personas, Planos, Trials e Quotas
version: 1.0
status: stable
type: requirement
domain: cross-domain
tags: [persona, plan, trial, quota, pricing]
---

# Personas, Planos, Trials e Quotas

Referência consolidada. Um mesmo usuário (1 CPF/CNPJ) pode ter **múltiplos vínculos** com personas diferentes em tenants diferentes (ex: síndico no Condomínio X, morador no Y, sócio de empresa Z).

## Personas

| Persona | Descrição | Tenant | Perfil |
|---|---|---|---|
| **Síndico** | Gestor de um ou mais condomínios | Operador do Condomínio | `institutional.SyndicProfile` |
| **Morador** | Residente de uma unidade em um condomínio | Pertencente ao Condomínio | `institutional.ResidentProfile` |
| **Empresa** | Prestador de serviço do mercado condominial | Próprio Tenant | `commercial.EnterpriseProfile` |
| **Agência de Marketing** | Gerencia conteúdo em nome de empresa | Actor delegado (não é tenant) | Sem perfil próprio, vinculado a Empresa |
| **Parceiro da Vizinhança** | Comércio local | Próprio Tenant | `commercial.LocalCompanyProfile` |
| **Admin Master Síndico** | Equipe interna da plataforma | Global | Sem perfil (admin) |

## Planos por Persona

### Morador

| Slug | Nome | Preço | Trial |
|---|---|---|---|
| `resident_base` | Base (gratuito) | R$ 0,00 | — |
| `resident_paid` | Pagante | R$ 9,90/mês | — |

### Síndico

| Slug | Nome | Preço mensal | Preço anual | Trial |
|---|---|---|---|---|
| `syndic_n1` | N1 Aprender | — | — | 15 dias |
| `syndic_n2` | N2 Atuar | R$ XX | R$ XX | 15 dias |
| `syndic_n3` | N3 Consolidar | R$ XX | R$ XX | 15 dias |

### Empresa

| Slug | Nome | Trial |
|---|---|---|
| `enterprise_plus` | Plus | 7 dias |
| `enterprise_pro` | Pro | 7 dias |

### Agência / Marketing

| Slug | Nome |
|---|---|
| `marketing_standard` | Standard |

### Parceiro da Vizinhança

| Slug | Nome | Trial |
|---|---|---|
| `local_company_standard` | Standard | 30 dias |

**Preços exatos**: definidos em `plans` table no banco. Admin pode alterar via painel. Specs mantém apenas estrutura.

## Trial por Persona (Req 4.1)

| Persona | Duração | Comportamento durante trial |
|---|---|---|
| **Síndico** | 15 dias | Tudo visível; bloqueia: criar atividade na Timeline, criar comunicados, exportar relatórios, gerenciar múltiplos condomínios |
| **Empresa** | 7 dias | Tudo visível; bloqueia: publicar vídeos, registrar execuções, gerenciar usuários |
| **Parceiro** | 30 dias | Tudo visível; bloqueia: criar promoções, ver métricas, criar campanhas |
| **Morador Pagante** | — | Sem trial — pagamento imediato ou permanece Base |

Mensagem padrão ao bloquear: _"Essa funcionalidade está disponível nos planos ativos da plataforma Master Síndico."_

Expirou o trial? Dados permanecem (**soft block**, sem grace period — Decision 4). Usuário perde acesso a features premium, mantém leitura e dados cadastrados.

## Quotas

### Connect Me (por **ano** — Decision 10)

| Persona/Plano | Cota anual |
|---|---|
| Morador Base | 2/ano |
| Morador Pagante | 4/ano |
| Síndico N1 | 2/ano |
| Síndico N2 | 4/ano |
| Síndico N3 | ilimitado |
| Empresa Plus | (não aplicável — E→E só Plus/Pro) |
| Empresa Pro | conforme contrato |
| Parceiro Standard | N/A (recebe, não envia) |

Reset: 1 de janeiro de cada ano. Tracking: `feature_usages` table + Redis `ms:quota:{userID}:connect_me:{year}`.

### Vídeos Institucionais (mensal)

| Plano | Uploads/mês |
|---|---|
| Síndico N1 | 4 |
| Síndico N2 | 8 |
| Síndico N3 | 12 |
| Empresa Plus | 8 |
| Empresa Pro | 12 |

Trava trimestral adicional: cada vídeo específico só pode ser **substituído** após 90 dias.

### Condomínios por Síndico (limite duro)

**15 condomínios** por conta de síndico. Acima: precisa de nova conta ou solicitação especial ao admin.

## Matriz Funcional Resumida

Ver `functional-matrix.md` para tabela completa. Extrato:

| Funcionalidade | Base | Pg | N1 | N2 | N3 | Plus | Pro | Marketing | Vizinhança |
|---|---|---|---|---|---|---|---|---|---|
| Busca geral | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Vídeos empresa (preview 25%) | ✅ | ✅ | ✅ | ✅ | — | — | — | — | — |
| Vídeos empresa (integral) | — | — | — | — | ✅ | — | — | — | — |
| Publicar vídeos | — | — | — | 4/mês | 8/mês | 8/mês | 12/mês | — | — |
| Connect Me | — | 2/ano | — | 2/ano | 4/ano | — | ✅ | — | — |
| Cursos Certificados (consumir) | — | — | — | ✅ | ✅ | — | — | — | — |
| Cursos Certificados (criar) | — | — | — | — | — | — | ✅ | — | — |
| Perfil público | — | — | limitado | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Criar promoções | — | — | — | — | — | — | — | — | ✅ |
| Vídeo-currículo | — | ✅ | — | — | — | — | — | — | — |
| Visualizar currículos | — | — | — | — | — | ✅ | ✅ | ✅ | ✅ |

## Enum — `user_role` (Zitadel + PG)

```
syndic | resident | enterprise | marketing | local_company | admin
```

## Enum — `user_role_type` (sub-contexto operacional, só local — Decision T8)

```
principal        — síndico principal ou titular de unidade
auxiliary        — síndico auxiliar
assistant        — assistente de síndico (sem voto, acesso operacional)
titular          — titular de unidade (morador principal)
dependent        — dependente de unidade (familiar/filho)
administrator    — administrador de empresa
operator         — operador técnico de empresa (sem gestão)
```

Zitadel **não** conhece `role_type`. É lógica interna.

## Enum — `plan_tier`

```
trial | base | plus | pro
```

## Paywall — Soft Block (Decision 4)

**Sem grace period.** Quando subscription expira:

1. Middleware ABAC identifica `plan_tier = expired/canceled`
2. Features premium → 403 `PLAN_REQUIRED`
3. Dados permanecem intocados (não deleta, não oculta leitura)
4. Usuário vê banner "Reativar assinatura" + link para Stripe Customer Portal
5. Pagamento → webhook Stripe `subscription.updated` → atualiza `users.plan_id` → ABAC libera

## Fluxo de upgrade/downgrade

1. Usuário escolhe novo plano em Stripe Customer Portal
2. Stripe emite evento `subscription.updated`
3. Backend `IPaymentGateway.ParseWebhookEvent` → `UpsertSubscription`
4. Atualiza `users.plan_id`
5. Cache Redis de auth invalidado
6. Próximo request: novo `plan_tier` na ABACContext

**Billing proporcional**: Stripe faz automaticamente. Backend apenas reflete estado.
