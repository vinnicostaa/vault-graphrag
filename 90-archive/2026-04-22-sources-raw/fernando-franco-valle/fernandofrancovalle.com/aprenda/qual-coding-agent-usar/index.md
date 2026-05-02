FFVAcademy

Material oficial de estudo

INTELIGÊNCIA ARTIFICIAL · FERRAMENTAS DE IA PARA CÓDIGO

⚖️

# Qual Ferramenta Usar e Quando

Tempo de leitura

12 min

Experiência

+80 XP

Nível

Intermediário

Como usar este material

1.  Leia o conteúdo ativamente — sublinhe, anote, questione
2.  No final, responda o quiz sem consultar o texto
3.  Confira o gabarito comentado e revise os pontos que errou
4.  Volte ao FFV Academy para registrar progresso e ganhar XP

fernandofrancovalle.com/aprenda/qual-coding-agent-usar

21 de abril de 2026

[FFV Academy](../../index.html)/[IA](../../ia/index.html)/Ferramentas de IA para Código/Qual Ferramenta Usar e Quando

⚖️

# Qual Ferramenta Usar e Quando

⏱ 12 min de leitura·+80 XP

Leve este módulo pra qualquer lugar

📄PDF

🖥️Apresentar

Pré-requisitos (0/4)0%

- ⬜[🤖 Claude Code: Filosofia e Arquitetura](../claude-code-arquitetura/index.html)(Ferramentas de IA para Código)
- ⬜[☁️ OpenAI Codex: o Agente na Nuvem](../openai-codex-cloud/index.html)(Ferramentas de IA para Código)
- ⬜[🖥️ Cursor, Copilot e os IDEs Aumentados](../cursor-copilot-ides/index.html)(Ferramentas de IA para Código)
- ⬜[☁️ Amazon Q e Kiro: a Aposta da AWS](../amazon-q-kiro/index.html)(Ferramentas de IA para Código)

Recomendamos completar os pré-requisitos antes de seguir, mas nada te impede de continuar.

Depois de entender como cada ferramenta funciona por dentro, a pergunta prática: qual usar? A resposta honesta é "depende" — mas depende de coisas específicas e mensuráveis. Sem achismo.

## O erro mais comum: escolher pela hype

A maioria das comparações online é baseada em *qual ferramenta completou este benchmark específico mais rápido*. Isso é quase inútil para decidir o que usar no seu trabalho. O que importa é diferente:

→Onde fica o código quando a ferramenta está "pensando"? (implicações de privacidade)

→O agente tem acesso ao ambiente de execução real? (testes, compilação, lint)

→O fluxo de revisão combina com o nível de confiança que você tem na saída?

→O time já usa a infraestrutura onde a ferramenta se integra melhor?

## Matriz de decisão por contexto

📋 Tarefa longa e complexa (feature completa, refatoração grande)

✓ Claude Code

Loop agêntico com acesso real ao ambiente. Pode rodar testes, verificar se o build passou, iterar com base nos resultados. Contexto longo (claude-sonnet suporta 200k tokens) permite manter o estado de uma tarefa multi-hora.

Alt: Cursor Agent — Boa opção se você prefere feedback visual durante a execução

Alt: Codex — Viável se a tarefa é bem definida e você quer execução assíncrona

📋 Múltiplas tarefas independentes em paralelo

✓ OpenAI Codex

Modelo assíncrono permite submeter N tarefas e receber N PRs. Ideal para sprints onde o time quer acelerar tarefas bem definidas (bug fixes, testes, documentação) sem bloquear o trabalho em curso.

Alt: Claude Code — Possível com múltiplos terminais, mas menos elegante

Alt: Copilot Workspace — Similar para tasks vinculadas a issues GitHub

📋 Desenvolvedor novo aprendendo a codebase

✓ Cursor / GitHub Copilot

Feedback visual inline reduz a fricção. O dev vê as sugestões no contexto do código, aceita linha por linha, entende o que está sendo mudado. Chat no Cursor permite fazer perguntas sobre o código sem sair do editor.

Alt: Claude Code — Funciona, mas a alternância terminal↔editor aumenta a carga cognitiva

📋 Projeto AWS-heavy (Lambda, CDK, DynamoDB, API Gateway)

✓ Amazon Q Developer

Treinado especificamente com documentação AWS. Entende quotas, limites, IAM policies, melhores práticas de arquitetura serverless. Menos hallucinations em recursos AWS que modelos genéricos.

Alt: Claude Code — Bom com documentação AWS incluída no contexto via WebFetch

Alt: Cursor + Copilot — Funcional mas sem a profundidade AWS do Q

📋 Código legado Java (8/11) precisando migrar para versão moderna

✓ Amazon Q Developer

O recurso de transformação de código do Q foi construído especificamente para isso. Ele tem um pipeline dedicado de análise, planejamento e execução de migrações Java que nenhuma outra ferramenta tem de forma nativa.

Alt: Claude Code — Pode fazer, mas sem o pipeline especializado — mais trabalhoso

📋 Time com requisitos rigorosos de compliance (HIPAA, PCI-DSS, SOC2)

✓ Claude Code

Seus arquivos permanecem na sua máquina. Só os prompts (texto) trafegam pela API. Isso é mais fácil de auditar e justificar em processos de compliance do que soluções que clonam seu repositório em infraestrutura de terceiros.

Alt: Cursor — Depende de onde o modelo está hospedado — pode ser configurado com modelo self-hosted

Alt: Copilot Enterprise — Microsoft tem certificações de compliance relevantes

📋 Feature nova com requisitos complexos e multi-time

✓ Kiro

O spec-driven development força clareza antes de execução. A spec serve de contrato entre PM, designer e dev. O rastreamento tasks → código → spec reduz ambiguidade e facilita revisão.

Alt: Claude Code + CLAUDE.md detalhado — Pode simular parte dos benefícios com um plano bem estruturado

## Os benchmarks: o que os números mostram em 2026

Em vez de opinião, dados públicos. Abril/2026:

SWE-BENCH VERIFIED — FRONTIER CLUSTER

Claude Opus 4.6 77,2%

Claude Sonnet 4.6 77,1%

GPT-5.1-Codex-Max 76,8%

Gemini 3 Pro 76,8%

codex-max (base) 76,6%

Claude Haiku 4.5 76,4%

Spread total: ~0,8 pontos

O que essa lista te diz: em 2026, escolher modelo frontier é *praticamente um coin flip*. O ganho médio esperado trocando de um para outro está dentro do ruído estatístico do benchmark. O que NÃO é ruído é o scaffold:

SWE-BENCH PRO (nov/2025) — MESMO MODELO, HARNESS DIFERENTE

Claude Opus 4.5 em SEAL Harness 45,9%

Claude Opus 4.5 em scaffold X 50,1%

Claude Opus 4.5 em Claude Code 55,4%

Spread: 9,5 pontos trocando só o harness

MODELO "MENOR" + SCAFFOLD BOM BATE MODELO "MAIOR"

Confucius Code Agent + Claude Sonnet 4.5 52,7%

Claude Opus 4.5 nativo (sem agent scaffold) 52,0%

Sonnet com scaffold vence Opus sem

EFEITO DO TURN BUDGET (SWE-agent paper)

Mesmo modelo, 50 turnos max → ~23%

Mesmo modelo, 250 turnos max → ~45%+

Dar 5x mais turnos quase dobra o resultado

💡

Regra útil: se o benchmark de uma ferramenta não informa *qual modelo, qual versão, quantos turnos*, descarte o número. Comparar "ferramenta A: 60%" vs "ferramenta B: 55%" sem essas variáveis é comparar rankings de futebol de anos diferentes.

## Mitos comuns (e a realidade dos dados)

✗ Mito: Modelo maior = melhor output.

→ Em abril/2026, spread entre frontier no SWE-bench Verified é 0,8pt. Tamanho já não diferencia ferramenta.

✗ Mito: O modelo mais recente é sempre o melhor.

→ Regressões acontecem em domínios específicos. GPT-5 melhorou front-end mas o SWE-bench Pro mostrou quedas em algumas categorias vs GPT-4.1. Teste em SEU workload.

✗ Mito: Harness não importa, só o modelo.

→ Mesmo modelo, mesmo benchmark: Claude Code 55,4% vs SEAL 45,9%. 9,5 pontos de diferença. Falso.

✗ Mito: Se o harness parseia muitos formatos de tool call, ele fica lento.

→ Parsing custa microssegundos. O que muda performance é edit format + turn budget + context management, não CPU de parsing.

✗ Mito: Ferramenta AWS precisa do Amazon Q.

→ Para código que usa AWS, Claude Code com bom contexto empata. O moat do Q é IAM nativo + Code Transformation com build farm — integração, não inteligência.

## Custo real: além do preço por token

O custo de uma ferramenta de coding agent vai além do preço da API. A conta completa:

``` p-4
Custo total = (tokens × preço/token)
            + tempo do dev revisando output
            + custo de bugs introduzidos
            + overhead de aprender a ferramenta
            + custo de integração ao workflow existente
            - tempo economizado em tarefas manuais

// Uma ferramenta barata que gera muito output ruim
// custa mais que uma cara que acerta na primeira.
```

O verdadeiro KPI é **throughput de código correto por hora de trabalho** — não tokens por dólar.

## Recomendação prática: não escolha um

A conclusão contraintuitiva depois de entender todas as ferramentas: as melhores equipes de engenharia não escolhem *uma* ferramenta — elas usam ferramentas diferentes para contextos diferentes.

STACK PRAGMÁTICO (2025)

Cursor ou CopilotNo IDE — autocomplete e edições rápidas durante o desenvolvimento normal

Claude CodePara tarefas longas, refatorações complexas, debug difícil — quando você precisa do agente com acesso real ao ambiente

Codex (cloud)Para tasks bem definidas em paralelo — bug fixes, testes, docs — enquanto você trabalha em outra coisa

Q DeveloperSe você trabalha com AWS — não faz sentido usar ferramenta genérica quando existe uma especializada

💡

O desenvolvedor que mais se beneficia de IA não é o que encontrou a ferramenta certa — é o que entende o que cada ferramenta faz bem e mal o suficiente para escolher a certa para cada situação.

📣 Curtiu? Ajuda a espalhar

𝕏X / Twitter

inLinkedIn

WWhatsApp

🔗Copiar link

🧩

## Quiz rápido

3 perguntas · Acerte tudo e ganhe o badge 🎯 Gabarito

⚡ **Time Attack** — 30s por pergunta · +30 XP se 100% no tempo

Começar quiz

### Próximos passos sugeridos

[📜](../spec-driven-development/index.html)

Spec-Driven Development (SDD): a nova espinha dorsal

Engenharia de Software Moderna · 18 min · +85 XP

→[🤖](../agentes-padroes/index.html)

Agent Patterns: ReAct, Reflexion e Tree of Thoughts

Engenharia AI-Native · 18 min · +90 XP

→

Continue lendo

[](../o-que-e-ia/index.html)

🤖CONTINUE NO HUB

### O que é Inteligência Artificial?

Fundamentos da IA · 6 min →

[](../o-que-e-cloud/index.html)

☁️EXPLORAR · AWS CLOUD

### O que é Cloud Computing?

AWS Cloud Practitioner · 8 min →

MATERIAL DE REVISÃO

## Gabarito comentado — Qual Ferramenta Usar e Quando

Responda primeiro sem consultar o texto. O gabarito traz a resposta correta com explicação densa — use como tutor offline.

1.  Questão 1

    Você tem 2 horas para implementar uma feature de autenticação OAuth2 que exige mexer em 8 arquivos diferentes (middleware, rotas, models, testes). Você prefere trabalhar no terminal. Qual ferramenta maximiza o throughput nesse cenário?

    1.  A.Cursor — o diff visual facilita revisar cada arquivo mudado e você não precisa sair do editor
    2.  B.Claude Code — o loop agêntico mantém estado entre arquivos, roda os testes no final de cada iteração, e o contexto longo (200k) aguenta o histórico de toda a sessão sem truncar
    3.  C.GitHub Copilot — completions inline em cada arquivo são mais rápidas que agentes para tarefas multi-arquivo
    4.  D.OpenAI Codex — submeter a tarefa assincronamente libera você para trabalhar em outra coisa enquanto ele gera o PR

    Resposta: B · Explicação

    Para tarefas multi-arquivo com dependências cruzadas, Claude Code no terminal é superior: ele lê arquivos explicitamente (não via embeddings), executa os testes após cada mudança e vê o resultado, mantém o plano completo no contexto. Cursor é excelente para feedback visual, mas o fluxo de terminal agêntico ganha em tarefas onde o modelo precisa iterar sobre erros de compilação/teste.

2.  Questão 2

    No SWE-bench Pro (novembro 2025), o mesmo modelo base (Claude Opus) testado em três harnesses diferentes obteve 45.9%, 50.1% e 55.4%. O que isso prova sobre a escolha de ferramenta?

    1.  A.Que o SWE-bench não é confiável e resultados variam aleatoriamente entre execuções
    2.  B.Que modelos mais novos sempre superam modelos mais antigos — o resultado de 55.4% é do modelo mais recente
    3.  C.Que o harness (scaffold de agente) impacta o resultado em quase 10 pontos percentuais com o MESMO modelo — a infraestrutura importa tanto quanto o modelo
    4.  D.Que Claude Code trapaceia nos benchmarks por ter acesso a dados de treino do SWE-bench

    Resposta: C · Explicação

    Mesmo modelo, mesmo benchmark, três harnesses: spread de 9.5 pontos. Isso destrói o argumento de que "escolher o modelo certo é tudo". O loop de agente, o edit format, o turn budget e o context management do scaffold explicam mais do resultado do que a diferença entre modelos frontier — que em abril/2026 têm apenas 0.8 pontos de spread no SWE-bench Verified.

3.  Questão 3

    Sua equipe usa GitHub Copilot para refactoring de uma codebase de 200k linhas em Python. Os devs reclamam que Copilot "esquece" o contexto entre arquivos e sugere o mesmo padrão problemático que vocês estão tentando remover. Qual é a limitação técnica raiz?

    1.  A.Copilot tem bugs na versão atual que causam sugestões desatualizadas — atualizar resolve
    2.  B.Copilot é um IDE completion tool: o contexto é janela local (arquivo atual + alguns adjacentes via embeddings). Ele não mantém estado de uma "sessão de refactoring" — cada completão é independente
    3.  C.Python não é bem suportado pelo Copilot — migrar para TypeScript resolve o problema
    4.  D.É necessário upgradar para o Copilot Enterprise que tem context window maior

    Resposta: B · Explicação

    Copilot (e a maioria dos IDE completions) opera com contexto local: arquivo aberto + vizinhos via embeddings semânticos. Para um refactoring que exige consciência de padrão em todo o codebase, você precisa de um agente que navegue explicitamente os arquivos relevantes (grep para ocorrências, leia cada um, entenda o padrão). Essa é exatamente a diferença entre IDE completion e agente com loop.

FFV Academy

Escola de engenharia para a era da IA

Qual Ferramenta Usar e Quando

Ferramentas de IA para Código

fernandofrancovalle.com/aprenda/qual-coding-agent-usar

Gerado em 21 de abril de 2026

Conteúdo editorial gratuito. Permitido estudo pessoal e uso em times. Não republicar em outros canais sem autorização escrita.
