[FFV Academy](../index.html)/Engenharia AI-Native

🧬

Blog

# Engenharia AI-Native

Para quem já sabe o básico e quer ir fundo. Aqui o assunto é como os modelos funcionam em produção: memória, roteamento, ferramentas, agentes. O lado técnico que pouca gente explica direito.

11artigos

1000XP total

📄PDF da trilha

🖥️Apresentar trilha

[](../aprenda/rag-fundamentos/index.html)

01

## 🧩 RAG: por que "só jogar tudo no LLM" não funciona

Retrieval-Augmented Generation na prática: limite de contexto, alucinação, arquitetura em dois estágios (retrieve → generate), quando RAG vence fine-tuning.

⏱ 16 min·+80 XP

→[](../aprenda/chunking-embeddings/index.html)

02

## 🔪 Chunking e Embeddings: as decisões que fazem ou quebram seu RAG

Fixed vs semantic vs recursive chunking, overlap, contextual retrieval, escolha de embedding (OpenAI, Voyage, BGE), redução de dimensão, cosine vs dot product.

⏱ 17 min·+85 XP

→[](../aprenda/hybrid-search-reranking/index.html)

03

## 🎯 Hybrid Search + Reranking: do BM25 ao cross-encoder

BM25 + vector, reciprocal rank fusion, cross-encoder reranking (Cohere, Jina, Voyage), HyDE, query expansion — o pipeline de retrieval de produção.

⏱ 18 min·+90 XP

→[](../aprenda/rag-evaluation/index.html)

04

## 📊 Avaliando RAG: recall@k, nDCG e LLM-as-judge

Golden dataset, métricas de retrieval (recall@k, MRR, nDCG), métricas de generation (faithfulness, context relevance, answer relevance), RAGAS, LLM-as-judge sem vazamento.

⏱ 16 min·+80 XP

→[](../aprenda/agentes-padroes/index.html)

05

## 🤖 Agent Patterns: ReAct, Reflexion e Tree of Thoughts

ReAct (think-act-observe), Reflexion (self-critique), Tree of Thoughts, Plan-and-Execute, Router — padrões empíricos com quando cada um funciona e quando quebra.

⏱ 18 min·+90 XP

→[](../aprenda/multi-agent-systems/index.html)

06

## 🕸️ Multi-Agent Systems: orchestrator-worker, swarms e handoffs

Orchestrator-worker, swarm com handoffs (OpenAI Swarm), CrewAI, hierarquias, quando multi-agent vale (e quando só aumenta custo).

⏱ 17 min·+85 XP

→[](../aprenda/context-engineering/index.html)

07

## 🧠 Context Engineering: prompt caching, subagents e skills

Anthropic prompt caching, janela de contexto, compaction, subagent delegation, skills (Agent Skills), CLAUDE.md/AGENTS.md, context window budget.

⏱ 16 min·+80 XP

→[](../aprenda/mcp-servers/index.html)

08

## 🔌 MCP Deep Dive: construindo um servidor profissional

Model Context Protocol em profundidade: stdio vs HTTP, tools/resources/prompts, autenticação, rate limit, logging, exemplo real em TypeScript e Python.

⏱ 19 min·+90 XP

→[](../aprenda/llm-apis-producao/index.html)

09

## 🚀 LLM APIs em Produção: streaming, structured output, batch e cache

Streaming SSE, tool use, structured output com JSON schema/Zod, batch API (50% desconto), prompt caching, retry com jitter, rate limit handling.

⏱ 16 min·+80 XP

→[](../aprenda/llmops-drift-canary/index.html)

10

## 📈 LLMOps: eval harness, drift detection e canary de prompts

Eval harness (promptfoo, LangSmith, custom), regressão de prompt, canary/A-B de prompts, drift detection, cost attribution, SLO de qualidade.

⏱ 18 min·+90 XP

→[](../aprenda/capstone-ai-native-rag-producao/index.html)

11

## 🏁 Capstone: RAG production-grade — de ponta a ponta

Projeto: RAG completo — ingestão (chunking + embed), store (Postgres pgvector ou Pinecone), retrieval (hybrid BM25 + vector + reranker), eval harness (golden set + LLM judge), observability (Langfuse), cost attribution. Deploy com feature flag pra canary.

⏱ 45 min·+150 XP

→

[← Voltar à home](../index.html)
