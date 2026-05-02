---
title: Graph-RAG neste vault
type: concept
tags: [methodology, rag, graph, obsidian, ai]
created: 2026-04-22
aliases: [Graph RAG, GraphRAG, Knowledge Graph RAG]
---

# Graph-RAG neste vault

**Graph-RAG** = *Retrieval Augmented Generation* onde o retrieval não é só embedding-similarity (RAG clássico), mas também **travessia de grafo** (relações explícitas entre entidades). Mais preciso pra domínios estruturados; menos alucinação; respostas reconstroem caminhos reais.

Neste vault, Graph-RAG **não é plugin nem infra nova** — emerge de 3 camadas que se somam.

## Camada 1: Grafo simbólico (Obsidian nativo)

- **Nós** = notas `.md`. Cada nota cobre **um** conceito ou entidade (atomic notes).
- **Arestas explícitas** = `[[wikilinks]]` no corpo das notas. Obsidian indexa isso como grafo navegável (Graph View).
- **Arestas estruturadas** = frontmatter `links: [[x]], [[y]]` quando a relação é canônica/estável (ex: "este pattern resolve esse antipattern").
- **Hubs** = MOCs (`_moc.md`). Uma MOC é um nó de alto grau de saída que serve de entrypoint.
- **Clusters** = pastas + `tags`. Tag `pattern-behavioral` cluster-iza por temática independente da pasta.

Isso sozinho já é um **knowledge graph**. Plugins como `Graph View` e `Dataview` fazem queries simbólicas em cima.

## Camada 2: Grafo semântico (Smart Connections)

Plugin **Smart Connections** (pasta `.smart-env/` indica que o usuário já tem configurado no vault antigo — replicar aqui):

- Gera **embeddings vetoriais** de cada nota (OpenAI, Claude, ou modelo local).
- Calcula similaridade cosseno entre notas.
- **Arestas implícitas** = "esta nota é semanticamente próxima dessas 10 outras" — mesmo sem `[[link]]` explícito.
- Descobre ponte entre conceitos que o humano ainda não linkou.

Emergência útil: você escreve um antipattern novo e o plugin aponta "3 patterns em 10-knowledge/patterns parecem resolver isso". Você valida, adiciona `[[link]]` explícito, e agora a aresta virou simbólica.

## Camada 3: Retrieval com os dois grafos

Quando agente (Planner, Worker, Reviewer) precisa de contexto pra uma task:

1. **Seed**: identifica nós iniciais (por nome/tag/frontmatter — ex: `type: antipattern` + `tags: concurrency`).
2. **Expansão simbólica**: BFS pelos `[[wikilinks]]` a distância 1-2.
3. **Expansão semântica**: adiciona top-K vizinhos por similaridade vetorial.
4. **Ranking**: score combinado (distância no grafo + similaridade + recência).
5. **Context pack**: notas selecionadas viram contexto do prompt do agente.

Isso é Graph-RAG clássico (ex: Microsoft GraphRAG 2024) **adaptado ao Obsidian**: o grafo já existe, você não precisa construir com LLM como o paper faz — você **cura manualmente** os `[[links]]` à medida que escreve.

## Por que este vault foi desenhado assim

### Atomic notes
Cada nó pequeno + focado. Retrieval traz **exatamente** o conceito, sem arrastar parágrafos irrelevantes.

### Frontmatter tipado
`type: antipattern` filtra o grafo em 1 query Dataview. Agente pode pedir "todos os `type: antipattern` com tag `concurrency` linkados direta ou indiretamente ao conceito de cache" — e isso vira retrieval cirúrgico.

### MOCs como hubs
Hub com alto fan-out acelera retrieval. Se você chega num MOC, você chega em tudo da área em 1 salto.

### Prefixos numéricos (00-, 10-, 20-...)
Ordenação estável pro humano. Irrelevante pro grafo — grafo ignora ordem alfabética.

### Pasta `30-agents/_memory/`
Notas que os agentes **escrevem** (patterns que funcionaram, failures encontradas). Grafo se auto-expande: agente hoje aprende de agente ontem.

## Diferença vs RAG clássico

| Aspecto | RAG clássico | Graph-RAG (este vault) |
|---|---|---|
| Unidade | Chunks arbitrários de texto | Nota atômica = 1 conceito |
| Relações | Implícitas (via similaridade) | Explícitas (`[[link]]`) + implícitas (embeddings) |
| Retrieval | Top-K por cosseno | BFS no grafo + cosseno + ranking combinado |
| Evolução | Estática (re-indexa batch) | Incremental (cada edit propaga) |
| Rastreabilidade | "Este chunk veio deste doc" | `"Este fato está em [[X]], linkado a [[Y]] por causa de [[Z]]"` |
| Curadoria | Zero (LLM chunk automático) | Manual + assistida (Smart Connections sugere arestas) |

## Quando Graph-RAG ganha

- Domínio com **entidades nomeadas estáveis** (decisões D-###, antipatterns AP-###, patterns como Strategy/Builder).
- Conhecimento com **relações causais** ("este pattern resolve este antipattern").
- Necessidade de **explicabilidade** ("por que o agente disse isso?" → "porque a nota X linka pra Y que é do tipo Z").
- Multi-sessão com **memória persistente** (o grafo cresce, não se reseta).

## Quando perde pra RAG clássico

- Corpus gigantesco e não-curado (milhões de páginas scrapeadas).
- Conceitos com nomes instáveis (texto corrido sem entidades bem-definidas).
- Zero trabalho humano/agente de curadoria — aí embedding direto é mais rápido de começar.

## Referências

- [Microsoft GraphRAG paper (2024)](https://arxiv.org/abs/2404.16130) — a fundação técnica.
- [Karpathy LLM Wiki pattern](https://github.com/NicholasSpisak/second-brain) — adaptação pra PKM pessoal.
- [Obsidian forum — Graph-RAG discussions](https://forum.obsidian.md/) — padrões comunitários.
- [[vault-guide]] — como operar este vault no dia a dia.
- [[../../00-meta/STATE]] — estado ativo do projeto.
