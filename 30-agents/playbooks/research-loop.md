---
title: Playbook — Research-Loop
type: playbook
tags: [playbook, research, agent, dual-check]
created: 2026-04-22
---

# Playbook — Research-Loop

Ciclo **obrigatório** de pesquisa antes de afirmar fato técnico, escolher lib, adotar pattern, desenhar contrato público, decidir segurança. **Nunca trate nada como verdade absoluta antes desse ciclo**.

## Quando disparar

Gatilhos que exigem o ciclo completo:

- Escolha de lib / framework / stack
- Escolha ou aplicação de pattern (DDD, CQRS, SAGA, etc.) em contexto novo
- Decisão de arquitetura ou contrato público (API, evento de domínio)
- Decisão de segurança (auth, cookies, ABAC, CORS)
- Mudança de schema DB ou regra de negócio crítica
- Resposta a CVE / vuln report
- Pedido de "qual a melhor prática pra X"

Gatilhos que **dispensam** (mas ainda exigem validação local):
- Refactor puro sem mudança semântica
- Bug-fix com causa clara e teste reproduzível
- Typo / doc fix

## Etapas

### Etapa 1 — Consultar o vault (memória durável)

```
□ grep/search em 10-knowledge/<subpasta>/ pelo tema
□ grep em 50-projects/<projeto>/STATE.md por D-### anterior sobre o mesmo
□ grep em 50-projects/<projeto>/11-audit/AUDIT.md por A-### correlato
□ listar notas em 60-sources/ que tocam o assunto
```

Se o vault já tem D-### decidindo isso, **ler a decisão, conferir se ainda vale** (ver §validação). Se vale, reaproveita. Se não vale, abre D-### sucessor.

### Etapa 2 — Docs oficiais e atualizadas

Ordem de preferência de fonte:

1. **Context7 MCP** (se disponível) — `resolve-library-id("<lib>")` → `get-library-docs(lib_id, topic)`
2. **Site oficial da lib/tech** — buscar seção docs da versão exata que o projeto usa
3. **Repo oficial** — `README.md`, `CHANGELOG.md`, `docs/`
4. **RFCs, PEPs, specs** quando aplicável (HTTP, OAuth, OIDC, JSON, WebSocket)
5. **Conference talks oficiais** (speakers maintainers da lib) — só como complemento

**Nunca começar por** StackOverflow, Medium, tutoriais aleatórios, respostas de LLMs sem checagem.

Anotar junto com cada fonte:
- URL completa
- Versão da doc consultada (ex: "Go 1.26", "Stripe API 2025-09-30.acacia")
- Data da consulta

### Etapa 3 — Como grandes empresas fazem

Buscar 2+ fontes de engenharia pública:

| Fonte | Uso |
|---|---|
| Netflix Tech Blog | Distribuídos, observability, chaos |
| Stripe Engineering | Idempotência, billing, webhooks, APIs |
| Uber Engineering | Escala, arquitetura, dados |
| Meta / Google / Shopify / GitHub / Airbnb blog | Geral |
| AWS / GCP / Azure architecture guides | Infra, SaaS patterns |
| ADRs publicados (ex: https://adr.github.io/adrs/) | Decisões reais |
| Postmortems (incident.fyi, incidents.sre.page) | O que deu errado e como corrigiram |
| Conference talks (QCon, StrangeLoop, KubeCon, Gophercon) | Deep dives |

**Não** usar 1 blog post como "isso é a prática" — cruzar com pelo menos 1 outra fonte que corrobore. Se discordam, registrar o trade-off.

### Etapa 4 — Mapear pattern recomendado

```
□ O pattern já existe em vault/10-knowledge/patterns/?
   └── SIM → linkar; se a aplicação ao contexto atual difere do canônico, adicionar seção "Aplicado ao projeto <X>"
   └── NÃO → criar nota atômica em vault/10-knowledge/patterns/ com:
            • Definição curta (1 parágrafo)
            • Quando usar / quando não usar
            • Exemplo canônico (linguagem-agnóstico ou na stack do projeto)
            • Trade-offs
            • Fontes consultadas
            • Links: vizinhos no grafo
```

### Etapa 5 — Dual-check (§[[dual-check]])

Agente-A escreveu a proposta. Agente-B (contexto limpo) revisa:
- Re-pesquisa as mesmas fontes
- Avalia: concorda / discorda / ressalva
- Registra divergências

**Saída do dual-check** faz parte do D-### — não é separado.

### Etapa 6 — Registrar decisão

Escrever em `50-projects/<projeto>/STATE.md`:

```markdown
### D-### <título curto da decisão> — <data ISO>
**Status**: ✅ Decidido | ⏳ Em aberto | 🔁 Revisitado | ❌ Rejeitado

**Contexto**: <por quê precisa decidir agora>
**Alternativas consideradas**:
- Opção A: <descrição + pró/contra>
- Opção B: ...

**Decisão**: <qual foi escolhida>
**Rationale**: <por quê>
**Fontes consultadas**:
- <URL> (versão, data)
- <URL> (...)

**Dual-check**: Agente-A propôs X; Agente-B validou / contestou Y; consolidação: Z.
**Data alvo de reavaliação**: <quando checar se ainda vale>
**Impacto**: <quem/o quê é afetado, links pra specs/tasks>
```

Se a decisão é **arquitetural**, criar adicionalmente um ADR em `50-projects/<projeto>/02-architecture/adr/NNNN-<slug>.md` usando o template ADR ([[../../40-templates/_moc|adr-template (a destilar)]] *(a criar)*).

## Validação de decisão anterior (ainda vale?)

Critérios que obrigam revisita:
- Fonte oficial atualizou (nova major release da lib, deprecação)
- Bug/incidente derivou da decisão
- Novo requisito tornou a decisão subótima
- CVE/vuln na abordagem
- > 12 meses sem revisita

## Anti-alucinação

Checklist durante pesquisa:

- [ ] Cada claim técnico tem URL de fonte
- [ ] Versão da lib na fonte **bate** com a versão do projeto
- [ ] Não há "provavelmente", "eu acho", "geralmente" sem citação
- [ ] Se encontrei informação conflitante, registrei o conflito (não escolhi a que "pareceu melhor")
- [ ] Se não encontrei fonte, **marquei como pendência** — não preenchi com treino

## Critério de "pronto"

```
□ D-### escrito em STATE.md
□ Fontes anotadas (mínimo 2 oficiais + 1 empresa de referência)
□ Pattern linkado em 10-knowledge/ (ou criado se faltava)
□ Dual-check documentado
□ ADR criado se arquitetural
□ Data de reavaliação registrada
```

## Links

- [[_moc]]
- [[dual-check]]
- [[dependency-audit]]
- [[../autonomous-execution]]
- [[../mcp-integration]]
- [[../../CLAUDE]]
- [[../../10-knowledge/methodology/sdd-workflow]]
