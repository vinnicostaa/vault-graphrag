---
title: Playbook — Community Summary (Graph-RAG Global Search)
type: playbook
tags: [playbook, agent, graph-rag, summary, global-search]
created: 2026-04-24
updated: 2026-04-24
aliases: [community-summary, graph-rag global search, bottom-up summary, agent summary]
---

# Playbook — Community Summary

Gera **sumários narrativos agregados** por pasta do vault, adaptando o conceito de *community summaries* do [Microsoft GraphRAG](https://microsoft.github.io/graphrag/) para a topologia Johnny.Decimal + `[[wikilinks]]` deste vault. Fecha a lacuna de **Global Search** identificada em [[../../10-knowledge/methodology/graph-rag]] (vault implementa Local Search; faltava equivalente holístico).

> **Decisão arquitetural**: sumário vive **dentro do `_moc.md` existente** como seção delimitada por marcadores HTML — não é arquivo separado. Ver [[../../00-meta/STATE|D-vault-003]] para alternativas consideradas e rationale.

## Gatilho

Roda em uma das 3 condições:

- **Manual**: operador pede `community-summary <path>` ou skill `/graph-rag-refresh`.
- **Periódico**: cron semanal (ex.: domingo 22h), **com gate de recência** (§ passo 0.5).
- **Incremental**: community teve nota nova/modificada → summary marcado stale → próximo ciclo recompõe.

Playbook **nunca** roda em community `_archive/`, `_session/`, `_inbox/` — são buffers operacionais, não conhecimento estabilizado.

## Conceito de "community" neste vault

- **Leaf community** = subpasta terminal (ex.: `10-knowledge/patterns/`, `10-knowledge/principles/`, `30-agents/playbooks/`).
- **Intermediate community** = pasta-raiz (ex.: `10-knowledge/`, `30-agents/`, `20-stacks/`).
- **Root** = `/` do vault (não geramos summary na raiz — `_index.md` já cumpre esse papel).

Simplificação vs Microsoft GraphRAG: comunidades **não** são detectadas via Leiden clustering sobre o grafo; usamos as pastas Johnny.Decimal como proxy. Ganho: zero overhead de descoberta; zero depedência de LLM para cluster-izar. Perda: communities cross-folder (notas linkadas em pastas diferentes) ficam fora. Compensação parcial: **passo 4 "Bridges"** usa embeddings (Smart Connections) para sugerir links inter-pasta.

## Artefato de saída

Seção dentro do `_moc.md` existente, idempotente via marcadores HTML:

```markdown
<!-- agent-summary:begin v=1 generated=2026-04-24 note_count=22 coverage_version=<hash> reviewed=true -->
## Agent Summary

<1 parágrafo de visão geral pt-BR>

### Conceitos centrais
- [[nota-a]] — razão da centralidade
- [[nota-b]] — razão
- ...

### Relações estruturais
- <nota-x cita/é citada por nota-y porque...>

### Bridges (cross-folder)
- [[../outra-pasta/nota-z]] — próxima semanticamente; tag <X> em comum

### Gaps observados
- conceito <C> aparece em N notas sem nota dedicada — candidato a criar

<!-- agent-summary:end -->
```

Humano continua dono do restante do `_moc.md`. Agente **só** escreve entre os marcadores (idempotência garantida por regex). Se `_moc.md` não tem o bloco, agente insere antes de `## Vizinhos` ou no final.

### Campos do delimitador HTML

| Campo | Função |
|---|---|
| `v=1` | Versão do gerador; quando playbook sobe pra v2, recompor forçado |
| `generated=<ISO-date>` | Data da última geração |
| `note_count=<N>` | Total de notas consideradas |
| `coverage_version=<hash>` | Hash curto (6 char) do concat dos `mtime` das notas; invalidação precisa |
| `reviewed=true\|false` | Dual-check amostral do passo 5.5 passou? Se `false`, retrieval global **exclui** esta community |

## Protocolo — 10 passos (0 a 9)

### 0 — Bootstrap: ler `_moc.md` como prior

Ler o `_moc.md` da community **antes** de ler as notas. A curadoria humana ali é **peso prior** para centralidade: notas listadas em "Destaques" ou "Leitura obrigatória" ganham +2 no score. Não descarta o conteúdo humano — integra.

### 0.5 — Gate de recência + lock

- **Gate de recência**: listar notas da community; se `max(mtime) > now - 30min`, abortar ciclo (humano pode estar editando). Retorna "skipped: recent-edit".
- **Lock advisory**: criar `<community>/.summary.lock` via `write_note` com `agent_id + epoch`. TTL = 10min (via `created` no lock). Se lock existe e `now - created < 10min`, abortar ("skipped: locked-by=X"). Se expirou, tomar o lock.

### 1 — Gate de coerência (entrada)

- Community tem ≥ 1 relação inter-nota (pelo menos 1 `[[wikilink]]` interno)? Se não, community é conjunto de ilhas — não vale summary. Abortar ("skipped: no-inter-links").
- Community tem ≤ 20 notas? Se N > 20, **quebrar por tag**: agrupar por tag dominante, gerar 1 summary por sub-cluster, depois agregar. (Em pastas grandes evita saturar contexto do agente.)

### 2 — Read pass

Ler todas as notas da community **incluindo** `_moc.md` (já lido no passo 0 mas mantido no set). **Excluir**:
- arquivos que começam com `_` **exceto** `_moc.md`
- `.summary.lock` (próprio lock)
- arquivos > 300 linhas (nota "hiperpopulada" — flaggear no passo 9 como gap estrutural, não consumir token)

Para cada nota, capturar: `title`, `type`, `tags`, primeiros 200 caracteres do corpo, lista de `[[wikilinks]]` emitidos.

### 3 — Extract pass

Computar 3 métricas por nota:

- **Centralidade (humana)**: +2 se listada em seção "Destaques"/"Leitura obrigatória" do `_moc.md`; +1 se listada em qualquer seção do `_moc.md`; 0 caso contrário.
- **Centralidade (estrutural)**: número de outras notas da community que citam essa nota via `[[wikilink]]`. **K=2 mínimo** para virar "central".
- **Coesão temática**: tags em comum com a tag dominante da community.

Nota é **central** se `centralidade_humana ≥ 2` OU `centralidade_estrutural ≥ K=2`.

Gaps detectados: termos mencionados em ≥ 3 notas sem nota dedicada no vault (verificar via `mcp__obsidian__list_directory`). São candidatos a criar.

### 4 — Bridges (cross-folder, via semântica)

Opcional mas recomendado — recupera parte do ganho Leiden que a simplificação por pastas perde.

Via Smart Connections (se disponível) ou via grep por tag/aliases em `10-knowledge/`, `30-agents/`, `20-stacks/`:

- Para cada **nota central** da community, buscar N=3 notas **semanticamente próximas** (embedding cosine ≥ 0.85) **em outra pasta**.
- Listar como "Bridges" — sugestão de `[[link]]` cross-folder que curador humano pode ratificar.

Se Smart Connections indisponível, substituir por "tag overlap": notas em outra pasta com ≥ 2 tags em comum com as notas centrais.

### 5 — Synthesize pass

Escrever a seção "Agent Summary" em pt-BR:

- **1 parágrafo de visão geral** — "Esta community reúne N notas sobre <tema>. Foca em <3 sub-áreas>."
- **Conceitos centrais** — 3-5 bullets, cada um com `[[link]]` para a nota e 1 linha justificando a centralidade. **Toda afirmação tem link**.
- **Relações estruturais** — 2-3 bullets do tipo "`[[nota-a]] cita [[nota-b]]` porque ambas tratam de X".
- **Bridges** — N=3 sugestões cross-folder (do passo 4).
- **Gaps observados** — do passo 3.

Regra dura: **nenhuma afirmação sem `[[link]]` para nota-fonte**. Se precisa dizer algo que nenhuma nota sustenta, **não diz**. Segue o §6 Anti-alucinação de [[research-loop]].

### 5.5 — Dual-check amostral (antes de flag `reviewed=true`)

Playbook **não** escreve `reviewed=true` de cara. Protocolo:

1. Summary é escrito com `reviewed=false` no delimitador.
2. Dispara [[dual-check]] amostral: agente-B em contexto limpo lê summary + **30% das notas-fonte escolhidas aleatoriamente**. Tarefa: validar que cada claim do summary rastreia para uma nota-fonte sem alucinação.
3. Se agente-B aprova → update do delimitador para `reviewed=true` via `patch_note`. Só agora o summary entra no retrieval global.
4. Se agente-B reprova → summary permanece `reviewed=false` (visível mas sem força de retrieval) + listar correções em STATE como `DT-summary-<community>-<date>`.

### 6 — Write idempotente

Via `mcp__obsidian__patch_note`:

- Se bloco `<!-- agent-summary:begin ... -->` existe → `patch` substitui inteiro.
- Se não existe → `patch` insere **antes** da seção `## Vizinhos` (se existir) ou no final do arquivo.

### 7 — Cascata

Após escrever summary de leaf community, **marcar community-mãe como stale**: atualizar `coverage_version` no delimitador do `_moc.md` da pasta-mãe para string fixa `STALE`. Próximo ciclo recompõe a mãe lendo os summaries das filhas.

Intermediate community, no synthesize pass, lê **apenas** as seções "Agent Summary" dos filhos (não as notas individuais) — é o equivalente ao map-reduce do Microsoft GraphRAG.

### 8 — Garbage collection

- Se community foi **removida** (pasta não existe): nada a fazer — `_moc.md` sumiu junto.
- Se community foi **esvaziada** (N=0 notas válidas): manter o bloco mas com note_count=0 + parágrafo "Community atualmente vazia; pode ser candidata a deletar via `mcp__obsidian__delete_note` do MOC se não houver plano de popular."

### 9 — Release do lock + report

- Deletar `.summary.lock`.
- Reportar 1 linha: `community-summary: <path> — note_count=<N> reviewed=<bool> bridges=<B> gaps=<G>`.

## Retrieval — como usar os summaries

Não fazer parte do playbook em si (é responsabilidade do agente que consulta), mas documentado aqui para referência:

- **Local query** (ex.: "como implementar strategy?"): BFS por `[[links]]` a partir do seed — **sem** consultar summary.
- **Global query** (ex.: "qual o estado geral dos patterns?"): ler `_moc.md#agent-summary` da community + **amostrar N=5 notas-fonte citadas no summary**. Summary é sinalizador, não autoridade final.
- **DRIFT query** (ex.: "adapter resolve qual antipattern, de modo geral?"): combinar local BFS + summary da community do nó seed.

Se `reviewed=false` no delimitador, retrieval **exclui** aquela community do modo global (cai pra local search apenas).

## Piloto obrigatório antes do rollout 100%

Seguindo precedente do [[../../00-meta/STATE|D-vault-001]] (piloto de 3 patterns antes de batch de 19):

1. Escolher 1 leaf community pequena como piloto — recomendado: `10-knowledge/principles/` (pequena, focada, conceitos estáveis).
2. Rodar playbook completo.
3. Operador humano **revisa 100%** do output + do dual-check.
4. Se aprovado, generalizar. Registrar lições em `STATE D-vault-003` sub-item `piloto-lessons`.
5. **Não expandir para `10-knowledge/patterns/` ou outras antes de aprovação do piloto.**

## Anti-padrão

- ❌ Escrever summary que contém afirmação sem `[[link]]` para nota-fonte.
- ❌ Parafrasear fonte externa no summary — só notas internas entram.
- ❌ Rodar em community com alta atividade recente (gate do passo 0.5).
- ❌ Flag `reviewed=true` sem passar por dual-check amostral.
- ❌ Rodar em paralelo na mesma community (lock do passo 0.5).
- ❌ Modificar qualquer coisa no `_moc.md` **fora** dos delimitadores HTML.
- ❌ Criar arquivo `_summary.md` ou `_digest.md` separado (rejeitado em D-vault-003).

## Requisitos de execução

- Ferramentas mínimas: `mcp__obsidian__read_note`, `mcp__obsidian__list_directory`, `mcp__obsidian__write_note`, `mcp__obsidian__patch_note`, `Agent` (para dual-check amostral do passo 5.5).
- Não consumir embedding API se não disponível — passo 4 fallback para "tag overlap".
- Dual-check amostral em Opus (ou Sonnet se Opus indisponível); nunca Haiku.

## Links

- [[_moc]]
- [[research-loop]] — etapas de leitura sistemática; summary reutiliza
- [[dual-check]] — amostral no passo 5.5
- [[../autonomous-execution]] — playbook roda em modo autônomo
- [[../../10-knowledge/methodology/graph-rag]] — conceito fundamentador
- [[../../00-meta/STATE]] — D-vault-003 registra a decisão
