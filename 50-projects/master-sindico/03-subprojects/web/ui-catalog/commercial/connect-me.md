---
title: Connect Me — Formulário de Contato (UI Spec)
type: ui-spec
tags: [ui-catalog, master-sindico, commercial, connect-me, lgpd]
bc: commercial
source: _chaos/08-CONNECT-ME.md (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# 08 — Connect Me (Formulário de Contato)

> **Origem**: `_chaos/08-CONNECT-ME.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Tradução aplicada**: `N1/N2/N3` → `trial/base/plus/pro` (ADR-0032 / D-081 / D-096); "Morador Pagante" removido (D-126).

> Sprint 1 · App `cms` · Rota: `/connect-me/:userId` (modal ou page)
> **NÃO é chat. NÃO salva histórico. NÃO tem reply.**

---

## Regras de negócio

- **Formulário de contato unidirecional** — disparo único de email / SMS.
- **Direções permitidas** (matriz canônica — [[../../../../00-product/business-model#41-matriz-consolidada]]):

  | Direção | Origem | Destino | Cota |
  |---|---|---|---|
  | M → S | Morador `base` | Síndico | **0/ano** (D-126 hard paywall) |
  | M → S | Morador `plus`+ (futuro Banco de Talentos addon, M3) | Síndico | 4/ano |
  | S → E | Síndico `base` | Empresa | 2/ano |
  | S → E | Síndico `plus` | Empresa | 4/ano |
  | S → E | Síndico `pro` | Empresa | ilimitado |
  | E → E | Empresa `plus`/`pro` | Empresa | ✅ entre empresas |
  | S → Parceiro | Síndico `plus`+ | Parceiro Vizinhança | ✅ |

- **Empresas de Marketing** (`role=marketing`) NÃO têm Connect Me ativo (anti-prospecção). Operam sob `DelegationGrant` da empresa cliente.
- **O contato parte SEMPRE do interesse do contratante** via busca → perfil → Connect Me (pull-only).
- **Cota validada pelo backend** (`feature_usages`). Reset anual em 1º de janeiro 00:00 UTC.

## Layout

```
┌──────────────────────────────────────────────┐
│  CONNECT ME                                   │
│  Entrar em contato com João Silva (Síndico)   │
│                                               │
│  ⚠️ 3 de 4 contatos disponíveis este ano     │
│  ████████████████████░░░░░ 75%                │
│                                               │
│  [Seu nome _________________________]        │
│  [Seu email ________________________]        │
│  [Telefone (opcional) ______________]        │
│                                               │
│  Assunto: [▼ Selecione categoria]            │
│                                               │
│  Mensagem:                                    │
│  ┌──────────────────────────────────────────┐│
│  │                                          ││
│  │                                   450/500││
│  └──────────────────────────────────────────┘│
│                                               │
│  ☑ Concordo que este formulário será enviado │
│    por email. Não há chat ou reply.           │
│                                               │
│  [Enviar Formulário →] primary               │
│                                               │
│  ℹ️ O contato será feito fora da plataforma  │
│  pelo destinatário.                           │
└──────────────────────────────────────────────┘
```

## CSS

```css
.connect-me-quota {
  display: flex; align-items: center; gap: 12px;
  padding: 12px 16px;
  background: oklch(0.769 0.165 70.1 / 0.08);
  border: 1px solid oklch(0.769 0.165 70.1 / 0.2);
  border-radius: var(--radius-md);
  font-family: 'Manrope'; font-size: 13px; color: var(--foreground);
  margin-bottom: 24px;
}
.connect-me-quota-bar {
  flex: 1; height: 4px; background: var(--muted); border-radius: 2px;
}
.connect-me-quota-fill {
  height: 100%; background: var(--warning); border-radius: 2px;
}
.connect-me-disclaimer {
  font-size: 12px; color: var(--muted-foreground); display: flex; gap: 8px;
  padding: 12px; background: var(--muted); border-radius: var(--radius-md);
  margin-top: 16px;
}
```

## Estados

| Estado | UI |
|---|---|
| Cota disponível | Formulário ativo, barra de cota visível |
| Cota esgotada | Formulário disabled, mensagem "Limite anual atingido. Reseta em 01/01/AAAA." |
| Enviando | Botão spinner + "Enviando..." |
| Enviado | Toast `success` + redirect para perfil do destinatário |
| Erro | Toast `destructive` + form permanece (preserva rascunho) |
| Permission-denied (role bloqueado) | Modal "Connect Me não disponível para sua persona/plano" + CTA upgrade |

> Estados padrão (loading / empty / error / success / permission-denied) seguem [[../../patterns/ui-states|patterns/ui-states]].

## Componentes

`ConnectMeForm`, `QuotaBar`, `ConnectMeDisclaimer`.

## Links

- [[../../../../00-product/sub-produtos/02-connect-me]] — sub-produto canônico (4 fluxos)
- [[../../../../00-product/business-model#41-matriz-consolidada]] — quotas anuais
- [[../../../../STATE]] — D-126 (Morador Pagante removido)
- [[../../../../08-security/lgpd|08-security/lgpd]] — consent + audit Connect Me
- [[../../patterns/ui-states|patterns/ui-states]] — estados obrigatórios
