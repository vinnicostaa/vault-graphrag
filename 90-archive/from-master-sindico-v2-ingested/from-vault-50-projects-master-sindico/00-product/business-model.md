---
title: Business Model — Master Síndico
type: project
tags: [master-sindico, product, business-model, pricing]
project: master-sindico
created: 2026-04-22
---

# Business Model — Master Síndico

SaaS multi-persona com planos por tipo de usuário + marketplace take-rate futuro.

---

## Fluxos de receita

### 1. Assinaturas de síndico / condomínio
**Recurring MRR**. Plano varia por tamanho do condomínio e features.

| Plano | Preço (proposta) | Unidades | Features chave |
|---|---|---|---|
| Básico | R$ XX/mês | até 30 | Institucional + Comunicados + Timeline |
| Pro | R$ XXX/mês | até 100 | + Connect Me + Content (Mux) |
| Enterprise | R$ XXXX/mês | até 300+ | + Assembleia live + LMS + SLA |

Trial: 15 dias, sem cartão (confirma depois).

### 2. Assinaturas de empresa condominial
**Recurring MRR**. Plano por quantidade de perfis e features.

| Plano | Preço | Features |
|---|---|---|
| Empresa Básico | R$ X/mês | 1 perfil + Connect Me passive |
| Empresa Pro | R$ XX/mês | perfil + Connect Me active + vídeos Mux |
| Empresa Enterprise | R$ XXX/mês | multi-perfil + API + prioridade busca |

Trial: 7 dias.

Add-on **Agência**: permite gerenciar multi-empresa (R$ Y/mês ou fee por conta).

### 3. Assinaturas de parceiro comércio local
**Recurring MRR** baixo ou **take-rate** sobre vendas via Vizinhança.

Trial: 30 dias (mais longo pra demonstrar valor ao pequeno negócio).

### 4. Marketplace / Connect Me (futuro)
**Take-rate** sobre contratos fechados via Connect Me.
Configurável por plano (alguns absorvem, outros repassam).

### 5. Conteúdo educacional (LMS) — futuro
Cursos pra síndicos, certificação (pagos ou bundle no Enterprise).

### 6. Ad inventory (futuro distante)
Banners/destaques em busca (só se não prejudicar reputação-driven).

---

## Unit economics (projeção)

- CAC síndico: R$ XXX via mkt digital pt-BR (Google + Meta)
- CAC empresa: R$ X via parcerias com agências condominiais
- LTV síndico Pro: 24 meses média × R$ XXX
- Payback: < 6 meses (alvo)
- Gross margin: ≥ 75% (SaaS padrão)

---

## Modelo de cobrança (Stripe)

### Recurring billing
- Cobrança mensal ou anual (anual 2 meses grátis)
- Cartão de crédito principal (Brasil)
- Boleto via Stripe (M2+)
- PIX recorrente (quando Stripe BR liberar)

### Gates de feature
- ABAC decisão inclui `plan_tier` → cache Redis 5min invalidado via webhook Stripe `customer.subscription.*`
- Quotas: Connect Me/mês, uploads Mux/mês, storage

### Churn mitigation
- Cancelar via app com fricção simétrica (não é dark pattern)
- Pausa de 30d disponível (dual-check se vai ser usado)
- Re-engajamento automático pós-cancel (notificação de feature nova, win-back)

---

## Modelo Bronze → Diamante (reputação)

**Determinístico, auditável. Não é input humano.**

| Tier | Critério (proposta) |
|---|---|
| **Bronze** | Default; 0 avaliações |
| **Prata** | 3 contratos bem-avaliados + 90% satisfação |
| **Ouro** | 10 contratos, 3 diferentes condomínios, 92% satisfação |
| **Platina** | 30 contratos, 10 condomínios, 95% satisfação, ≥ 1 case vídeo |
| **Diamante** | 100 contratos, 20+ condomínios, 97% satisfação, cases vídeo + rec. síndico |

Revogação automática em caso de:
- Queixa formal confirmada via moderação
- Abuso detectado por ML (futuro)
- Inatividade > 12 meses

---

## Matriz plano × persona × feature

Ver [[../04-requirements/functional/functional-matrix]].

---

## Sensibilidade LGPD (billing)

- PAN de cartão **nunca** armazenado (Stripe tokeniza)
- Histórico de cobrança guardado por 5 anos (requerimento fiscal + LGPD)
- Cancelamento deleta dados opcionais; mantém obrigatórios (fiscal)

---

## Referências

- Stripe pricing page (dev + prod)
- Billing model: subscription-based com quotas
- Competidores: Superlógica (ERP condominial + pricing saas), CondoLivre, TownSq — analisar diferenças em [[competitor-landscape]] *(a criar)*

## Links

- [[_moc]]
- [[vision]]
- [[personas]]
- [[../01-domain/bounded-contexts]]
- [[../04-requirements/functional/billing]]
- [[../04-requirements/functional/functional-matrix]]
