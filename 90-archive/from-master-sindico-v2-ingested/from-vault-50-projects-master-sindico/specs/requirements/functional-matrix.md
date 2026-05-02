---
title: Functional Matrix — Features × Personas × Plans × Actions
version: 1.0
status: stable
type: requirement
domain: cross-domain
tags: [abac, feature-matrix, access-control, plan-tier]
---

# Functional Matrix — Features × Personas × Plans × Actions

Matriz completa de funcionalidades mapeando cada ação para persona × plan_tier. Fonte de verdade para ABAC middleware (`RequirePermission(action, resource)`).

---

## Legenda

- ✅ = Permitido
- ❌ = Bloqueado
- ⏳ = Trial (bloqueado até conversão)
- 🔒 = Requer unlock via plan upgrade
- `n/a` = Não aplicável

---

## Morador Base (Free)

| Funcionalidade | Ação | Base | Pagante | N1 | N2 | N3 | Plus | Pro | Marketing | Vizinhança |
|---|---|---|---|---|---|---|---|---|---|---|
| **Busca geral** | Ver resultados | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Timeline** | Ler | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Vídeos empresa** | Preview (25%) | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **Vídeos empresa** | Assistir integral | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Comunicados** | Ler | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Eventos condomínio** | Confirmar participação | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Enquetes** | Responder | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Assembleia** | Ver pauta | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Assembleia** | Dar ciência | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Assembleia** | Votar | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Assembleia** | Procuração | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Banco Talentos** | Upload vídeo-currículo | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **Banco Talentos** | Ver currículos (lado empresa) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |
| **Connect Me** | Enviar síndico | ❌ | 2/ano | ❌ | 2/ano | 4/ano | ❌ | ✅ | ❌ | ❌ |
| **Fórum** | Ver tópicos | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| **Fórum** | Comentar | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| **Fórum** | Criar tópico | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **Academia LMS** | Ver Lives | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| **Academia LMS** | Fazer cursos | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| **Academia LMS** | Certificado | ❌ | ✅ | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Vizinhança** | Ver parceiros (bairro) | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **Vizinhança** | Benefícios exclusivos | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |

---

## Síndico N1, N2, N3

| Funcionalidade | Ação | Trial | N1 | N2 | N3 |
|---|---|---|---|---|---|
| **Gerenciar condomínio** | Criar | ⏳ | 1 | ✅ | ✅ |
| **Gerenciar condomínio** | Número máximo | ⏳ | 15 | 15 | 15 |
| **Timeline** | Criar atividade | ⏳ | ⏳ | ✅ | ✅ |
| **Comunicados** | Criar | ⏳ | ⏳ | ✅ | ✅ |
| **Plano Diretor** | Visualizar | ✅ | ✅ | ✅ | ✅ |
| **Assembleia** | Criar | ✅ | ✅ | ✅ | ✅ |
| **Assembleia** | Publicar pauta | ✅ | ✅ | ✅ | ✅ |
| **Assembleia** | Painel presidente | ✅ | ✅ | ✅ | ✅ |
| **Execução empresa** | Validar | ✅ | ✅ | ✅ | ✅ |
| **Propostas** | Validar | ✅ | ✅ | ✅ | ✅ |
| **Connect Me empresa** | Enviar | ✅ | 2/ano | 4/ano | ∞ |
| **Vídeos institucionais** | Upload/mês | ⏳ | 4 | 8 | 12 |
| **Vídeos institucionais** | Perfil público | ❌ | Limitado | ✅ | ✅ |
| **Academia LMS** | Certificado (criar) | ❌ | ❌ | ❌ | ❌ |
| **Banco Talentos** | Não tem acesso | — | — | — | — |
| **Perfil público** | Visibilidade | ❌ | Privado | ✅ | ✅ |
| **Governance markers** | Editar | ✅ | ✅ | ✅ | ✅ |
| **Mini bio + vídeo institucional** | Sim | ❌ | ❌ | ✅ | ✅ |
| **Relatório exportação** | PDF/CSV | ⏳ | ⏳ | ✅ | ✅ |

---

## Empresa Plus, Pro

| Funcionalidade | Ação | Trial | Plus | Pro |
|---|---|---|---|---|
| **Vídeos institucionais** | Upload/mês | ⏳ | 8 | 12 |
| **Vídeos institucionais** | Quem vê | Síndico+preview | Síndico+preview | Síndico+preview |
| **Agência Marketing** | Delegar | ❌ | ✅ | ✅ |
| **Connect Me empresa** | Enviar (E→E) | ❌ | ✅ (Plus) | ✅ |
| **Propostas** | Criar | ✅ | ✅ | ✅ |
| **Registro execução** | Criar | ⏳ | ⏳ | ✅ |
| **Gerenciar usuários** | Criar | ⏳ | ⏳ | ✅ |
| **Banco Talentos** | Ver/buscar currículos | ❌ | ✅ | ✅ |
| **Academia LMS** | Certificado (criar cursos) | ❌ | ❌ | ✅ |
| **Perfil público** | Visibilidade | Privado | ✅ | ✅ |
| **Compliance markers** | Editar | ✅ | ✅ | ✅ |
| **Neighborhood partner** | Criar benefício | ❌ | ❌ | ✅ (via síndico) |

---

## Parceiro da Vizinhança (Local Company)

| Funcionalidade | Ação | Trial | Standard |
|---|---|---|---|
| **Perfil público** | Visibilidade | ✅ | ✅ |
| **Benefícios promoção** | Criar | ⏳ | ✅ |
| **Métricas** | Ver clicks/resgates | ⏳ | ✅ |
| **Campanhas** | Criar | ⏳ | ✅ |
| **Academia LMS** | Acesso | ❌ | ❌ |
| **Banco Talentos** | Acesso | ❌ | ❌ |
| **Vídeos** | Upload | ❌ | ❌ |

---

## Agência de Marketing (Delegated Actor)

| Funcionalidade | Ação | Permitted? |
|---|---|---|
| **Vídeos** | Upload em nome da empresa | ✅ |
| **Vídeos** | Editar metadados | ✅ |
| **Vídeos** | Ver analytics | ✅ |
| **Deals** | Visualizar | ❌ |
| **Connect Me** | Enviar/receber | ❌ |
| **Usuários** | Gerenciar | ❌ |
| **Propostas** | Acessar | ❌ |
| **Token** | Revogação automática após TTL | ✅ |

---

## Admin Master Síndico

| Funcionalidade | Ação |
|---|---|
| Dashboard financeiro (MRR, assinaturas) | ✅ |
| Gestão de usuários (listar, bloquear, editar) | ✅ |
| Moderação de conteúdo (vídeos, fórum) | ✅ |
| Configurações da plataforma (categorias, parâmetros) | ✅ |
| Broadcasts segmentados (email/SMS) | ✅ |
| Aprovação manual de CNPJs | ✅ |
| Controle de inadimplentes | ✅ |
| Gestão global de tenants | ✅ |
| Ver audit trail completa | ✅ |

---

## Quotas por Tipo

### Connect Me (Anual)

| Persona/Plano | Cota |
|---|---|
| Morador Base | 0 |
| Morador Pagante | 4/ano |
| Síndico N1 | 2/ano |
| Síndico N2 | 4/ano |
| Síndico N3 | ∞ |
| Empresa Plus | (E→E apenas) |
| Empresa Pro | ∞ |

### Vídeos (Mensal)

| Plano | Uploads/mês |
|---|---|
| Síndico N1 | 4 |
| Síndico N2 | 8 |
| Síndico N3 | 12 |
| Empresa Plus | 8 |
| Empresa Pro | 12 |

### Condomínios (Hard limit)

| Persona | Máximo |
|---|---|
| Síndico | 15 |

---

## Paywalls (Soft Block)

### Trial

- Síndico: 15 dias
- Empresa: 7 dias
- Parceiro: 30 dias
- Morador: não aplica

### Hard Paywalls (403 `PLAN_REQUIRED`)

| Feature | Plano Mínimo |
|---|---|
| Votar em assembleia | Morador Base |
| Criar comunicado | Síndico N2 |
| Validar execução | Empresa Pro |
| Relatório exportação | Síndico N2 |
| Academia LMS (certificado) | Morador Pagante / Síndico N2 / Empresa Plus+ |

---

## Mapping para Middleware

Exemplos de `RequirePermission(action, resource)` para common flows:

```go
// Handler: criar comunicado
requiredAction := "institutional.communication.create"
abac.RequirePermission(ctx, requiredAction, "communication")
// ABAC valida: role=syndic, plan_tier >= plus → 403 se não

// Handler: votar
requiredAction := "assembly.vote.create"
abac.RequirePermission(ctx, requiredAction, "vote")
// ABAC valida: role=resident, plan_tier != base OR role=syndic → ✅

// Handler: validar execução
requiredAction := "commercial.execution.approve"
abac.RequirePermission(ctx, requiredAction, "execution_record")
// ABAC valida: role=enterprise, plan_tier >= pro → 403 se plus
```

---

## Referências cruzadas

- **Identity.md**: Req 2 (ABAC engine)
- **Personas-and-plans.md**: plano × persona mapping
- **Billing.md**: trial + soft block
- **Content.md**: Req 29 quotas de vídeo
- **Commercial.md**: Req 10 quotas Connect Me
- **Product-overview.md**: decisions que governam estas regras
