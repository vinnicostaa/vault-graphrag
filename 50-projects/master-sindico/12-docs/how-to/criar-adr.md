---
title: How-to — Criar ADR (Architecture Decision Record)
type: guide
tags: [how-to, adr, decision, architecture, master-sindico, v2]
source: padrão MADR + adr.github.io + STATE/STEERING + adr-index
created: 2026-04-23
updated: 2026-04-23
---

# How-to — Criar ADR

**ADR** (Architecture Decision Record) formaliza decisão arquitetural. Deve acompanhar D-### em [[../../STATE]] sempre que impacto for arquitetural.

Padrão: **MADR v4** simplificado + adaptações Master Síndico v2.

---

## Quando criar ADR

- ✅ Decisão de topologia (multi-tenant, single-region vs multi, stack infra)
- ✅ Escolha de provider externo primário (Stripe, Mux, Livekit, Zitadel)
- ✅ Pattern arquitetural (event-driven, CQRS, saga)
- ✅ Padrão de dado (money int64 cents, fração ideal NUMERIC)
- ✅ Security policy (ABAC matrix estrutura, PKCE mandatory)
- ✅ Infra crítica (Railway vs ECS, NATS vs Kafka)
- ❌ Bug fix — não precisa
- ❌ Feature nova dentro de BC existente — registrar D-### se decisão não-trivial, mas ADR só se mexe em arquitetura
- ❌ Refactor interno sem mudar contratos

Regra prática: se **alguém que entra no projeto em 6 meses** precisar entender "por que fizeram assim", merece ADR.

---

## Processo

### 1. Research-loop + dual-check

Obrigatório pra decisão não-trivial:
1. [[../../06-execution/playbooks/research-loop]] — 5-stage (existente / doc oficial / big tech / patterns / dual-check)
2. [[../../06-execution/playbooks/dual-check]] — revisor contexto limpo
3. Log em [[../../11-audit/dual-check-log/_moc]]

### 2. Allocate ADR number

Próximo número sequencial da tabela em [[../adr-index]] (atualmente planejadas ADR-0001..ADR-0020; próxima livre é ADR-0021+).

Formato: `ADR-XXXX` com zero-padding 4 dígitos.

### 3. Criar arquivo

Path: `02-architecture/adr/ADR-XXXX-<slug>.md`

Exemplo: `ADR-0021-feed-connect-me-deterministico-vs-ml.md`.

### 4. Preencher template

Usar estrutura abaixo:

---

```markdown
---
title: ADR-XXXX — <título curto>
type: adr
tags: [adr, architecture, <tópico>, master-sindico, v2]
status: proposed | accepted | deprecated | superseded-by-ADR-YYYY
date: YYYY-MM-DD
deciders: [<lista>]
consulted: [<lista>]
informed: [<lista>]
d_ref: D-### (STATE)
dual_check_ref: dual-check-log/YYYY-MM-DD-<slug>
---

# ADR-XXXX — <título>

## Status

📝 **proposed** | ✅ **accepted** | ⚠️ **deprecated** | 🔁 **superseded-by-ADR-YYYY**

**Data**: YYYY-MM-DD
**Última atualização**: YYYY-MM-DD

## Contexto e problema

<1-3 parágrafos descrevendo:>
- Qual é o problema sendo resolvido
- Por que decidir agora
- Restrições relevantes (custos, prazos, stack existente)
- Forças em jogo (trade-offs latentes)

## Drivers da decisão

- Requirement relacionado: GR-### / IDN-### / BIL-### / ...
- Invariante afetado: I-###
- Marco: M1/M2/M3/pós
- Objetivo: <explícito>

## Opções consideradas

### Opção A: <nome>

**Descrição**: <o que é>

**Prós**:
- <item>
- <item>

**Contras**:
- <item>
- <item>

**Custo estimado**: <dev-weeks / $ mensal>

### Opção B: <nome>

(idem)

### Opção C: <nome>

(idem)

## Decisão

**Escolhida**: Opção X — `<nome>`.

**Rationale**:
<por que essa, não as outras — explícito>

## Consequências

### Positivas

- <item>
- <item>

### Negativas / Trade-offs aceitos

- <item>
- <item>

### Neutras

- <item>

### Impacto em outras partes

- Afeta invariante X: <sim/não — como>
- Afeta BC Y: <sim/não — como>
- Afeta SLO: <sim/não — como>
- Afeta LGPD: <sim/não — como>

## Alternativas rejeitadas — por quê

- **Opção B**: rejeitada porque <motivo explícito>
- **Opção C**: rejeitada porque <motivo>

## Migration / rollout plan

<se aplicável: como migrar do estado atual pra novo>

1. <step>
2. <step>

**Rollback plan**: <como reverter se der errado>

## Validação

- [ ] Como saberemos que foi decisão boa? (critério mensurável)
- [ ] Data de revisão: YYYY-MM-DD

## Fontes

- <URL + data + trecho relevante>
- [[../../13-research/<vertical>/source-NN]]
- Doc oficial: <link>
- Engineering blog: <link> (big tech X, YYYY)

## Links

- [[../../STATE]] D-###
- [[../../STEERING]]
- [[../../11-audit/dual-check-log/YYYY-MM-DD-<slug>]]
- [[../adr-index]]
- ADR superseded by / supersedes (se aplicável)
```

---

### 5. Update adr-index

Editar [[../adr-index]]:

- Adicionar entrada na tabela principal (status + data)
- Atualizar tabela "ADRs novas" se aplicável

### 6. Update STATE

Editar [[../../STATE]]:

- Se D-### existia: atualizar status → ✅ Fechado + linkar ADR
- Se não existia: criar novo D-### (com link ADR)

### 7. Announce

- Postar em `#dev` ou `#product`: "ADR-XXXX <título> foi aceita"
- Mencionar D-### fechado
- Sprint seguinte: time revisa

---

## Status lifecycle

```
proposed → accepted → (eventualmente)
                        ↓
                   deprecated (parou de ser relevante)
                        ou
                   superseded (substituída por ADR nova)
```

**Nunca deletar ADR**. Mesmo deprecated, permanece pro histórico.

Se ADR é substituída: criar ADR nova com `supersedes: ADR-XXXX`; antiga recebe `superseded-by: ADR-YYYY` mas fica.

---

## Exemplos canônicos (do legado + futuros)

- **ADR-0005** — Zitadel como IdP (decisão forte; stack-defining)
- **ADR-0011** — sqlc v1.30 (substitui Drizzle beta — AP-011 resolvido)
- **ADR-0015** — Domain Events Outbox pattern
- **ADR-0019** — Railway infra (🟡 aberta; pode supersedar em M4+ se migrar ECS)

---

## Anti-padrões

- ❌ ADR sem rationale ("decidimos porque fizemos")
- ❌ ADR sem alternativas consideradas
- ❌ ADR depois do fato (escrita pra justificar algo já feito sem research)
- ❌ ADR que cita "melhor prática" sem fonte
- ❌ ADR sem critério de validação
- ❌ Deletar ADR deprecated (perde histórico)

---

## Links

- [[_moc]]
- [[../adr-index]]
- [[criar-novo-bc]]
- [[adicionar-novo-provider]]
- [[../../02-architecture/adr/_moc]]
- [[../../STATE]]
- [[../../06-execution/playbooks/research-loop]]
- [[../../06-execution/playbooks/dual-check]]
