---
title: >-
  ⚠️ WARNING — sensitive-flow.ts é GENÉRICO; evoluir para token opaco backend
  antes de produção
type: warning
tags:
  - warning
  - lgpd
  - security
  - auth
  - tech-debt
  - master-sindico
  - web
  - evolution
project: master-sindico
subproject: web
severity: high
status: stopgap-applied
evolution-target: D-WEB-2026-04-28-012-EVOLUTION
created: '2026-04-28'
updated: '2026-04-28'
---

# ⚠️ WARNING — sensitive-flow.ts é GENÉRICO; evoluir para token opaco antes de produção

## Status

🟡 **STOPGAP APLICADO** em 2026-04-28 (sessão Agente 1 — A-WEB-2026-04-28-036). Resolve a vulnerabilidade LGPD imediata (email em search params da URL — DT-WEB-2026-04-28-012) mas **NÃO É a solução final** para defesa em profundidade.

## O que foi aplicado (curto prazo)

`/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/web/apps/auth/src/lib/sensitive-flow.ts`

Helper genérico baseado em `sessionStorage` com TTL 5min, espelhando o pattern de `otp-flow.ts`. Substitui email em URL nas rotas:
- `sign-up.tsx` → `verify-email.tsx`
- `forgot-password/index.tsx` → `forgot-password/code.tsx`

Email passa a residir em `sessionStorage["ms.signup-init"]` ou `["ms.recover-init"]` em vez de `?email=...`.

**Vantagens imediatas:**
- ✅ Email não trafega em URL → sem vazamento via `Referer`, histórico browser, prints, logs CDN
- ✅ TTL 5min limita janela de exposição
- ✅ `sessionStorage` é por aba (não vaza entre abas como `localStorage`)
- ✅ Limpeza explícita após uso (`clearSensitiveFlow`)

## ⚠️ POR QUE NÃO É SUFICIENTE

1. **Email em texto claro no DevTools do browser** — quem tem acesso à máquina do usuário lê tudo via Application > Storage > Session Storage no DevTools. Aceitável para a janela curta de 5min, mas defesa-em-profundidade real exige cifragem ou token opaco.

2. **Sem validação Zod por chave** — `meta?: Record<string, string>` é genérico; injeção de payload arbitrário no storage por outro código no mesmo origin pode passar sem detecção. Schema rigoroso por `SensitiveFlowKey` mitigaria.

3. **Sem audit log** — DPO LGPD pode requisitar trilha de auditoria de acesso a dado pessoal mesmo efêmero. Hoje não temos.

4. **Compartilhamento involuntário ainda possível** — se o usuário compartilha o desktop em tempo real, o conteúdo do storage é visível no DevTools.

## Plano de evolução (M1+ ou M2)

**Recomendação primária — Token opaco backend (Opção A):**

```
1. Endpoint POST /api/v1/auth/flow-token
   - Body: { kind: "signup-init" | "recover-init", email, meta? }
   - Backend gera UUIDv7 → `flow_tokens` (Redis com TTL 5min)
   - Response: { flowId: "uuid-v7" }

2. Frontend salva apenas `flowId` em sessionStorage (não-sensível)

3. Etapa 2 (verify-email / forgot-password/code):
   - GET /api/v1/auth/flow-token/:flowId
   - Backend retorna `{ email, meta }` (cookie httpOnly via rate-limit)
   - Frontend nunca salvou email persistido no client
```

**Pré-requisitos backend:**
- Tabela / Redis namespace `flow_tokens:<uuid>` com TTL 5min e UNIQUE constraint
- Rate limit IP por GET (impedir enumeration de flowIds aleatórios)
- ABAC: nenhuma autenticação requerida (é pré-auth flow), mas device fingerprint match opcional

**Esforço estimado:** ~1-2 dias backend + 0.5 dia web (substituir `sensitive-flow.ts` por `flow-token-client.ts`)

**Alternativas:**
- Opção B: Cifragem client-side com Web Crypto API (chave derivada de cookie httpOnly rotativa) — mais complexo, ainda no client
- Opção C: Schema Zod rigoroso por `SensitiveFlowKey` + audit log local (melhoria incremental do stopgap atual)

## Quando reavaliar

- Antes de **produção M1** (firme 2026-05-18) — decidir A vs B vs C
- Após review DPO — pode ditar nível obrigatório de hardening
- Em qualquer auditoria LGPD externa — esse warning aponta o gap honestamente

## Referências

- `apps/auth/src/lib/sensitive-flow.ts` — implementação atual (com warning inline)
- `apps/auth/src/lib/otp-flow.ts` — pattern análogo já em uso para OTP code
- [[../11-audit/audit-log/2026-04-28-agente-1-fixes]] — sessão de aplicação
- [[../03-subprojects/web/tasks#DT-WEB-2026-04-28-012]] — DT original
- [[../08-security/security]] §5 — defesa em profundidade
- [[../08-security/lgpd]] — compliance LGPD
- LGPD Art. 5º (CPF/email/telefone como dado pessoal); Art. 46-49 (proteção de dados)

## Tracker

- **D-WEB-2026-04-28-012** (este DT) — STOPGAP aplicado
- **D-WEB-2026-04-28-012-EVOLUTION** (próximo) — token opaco backend; aguarda decisão João + alocação backend
