---
title: Spec-Driven Development (SDD) — conceito e filosofia
type: concept
tags: [methodology, sdd, spec-kit, specifications]
source: 60-sources/spec-kit/spec-driven.md
created: 2026-04-22
aliases: [SDD, Spec-Driven Development, Specification-Driven Development]
---

# Spec-Driven Development (SDD) — conceito e filosofia

Destilado do [spec-driven.md do GitHub spec-kit](https://github.com/github/spec-kit/blob/main/spec-driven.md). Complementa [[sdd-workflow]] (que é operacional — fases) com a **filosofia e princípios** que motivam o método.

Nosso ciclo de 5 fases de [[sdd-workflow]] é a prática; este doc é o **porquê**.

## A inversão de poder

Por décadas, **código foi rei**. Specs serviam código — eram andaime descartado quando "o trabalho de verdade" (codar) começava. PRD, design doc, diagramas: todos subordinados ao código. Código era verdade. Documentação era "boa intenção".

**SDD inverte essa hierarquia**: specs não servem código — código serve specs.

- PRD **não é guia** pra implementação → é **fonte** que gera implementação
- Technical plans **não são docs** que informam coding → são **definições precisas** que produzem código

> Isso não é melhoria incremental. É repensar fundamentalmente o que impulsiona desenvolvimento.

## O gap eliminado

O gap entre spec e implementação é a doença original do software. Tentamos preencher com mais doc, mais detalhe, mais processo — falhamos porque aceitamos o gap como inevitável.

**SDD elimina o gap** ao fazer specs + implementation plans **executáveis** — quando spec gera código, não há gap, só **transformação**.

Isso é possível agora porque IA entende specs complexas e produz implementação. Mas IA bruta sem estrutura → caos. SDD provê a estrutura.

## Princípios centrais

1. **Specifications as Lingua Franca** — spec é artefato primário. Código é sua expressão em uma linguagem/framework específicos. Manter software = evoluir specs.

2. **Executable Specifications** — specs devem ser precisas, completas, inambíguas o suficiente pra gerar sistemas funcionais. **Isso elimina o gap** entre intenção e implementação.

3. **Continuous Refinement** — validação de consistência é **contínua**, não um gate único. IA analisa specs por ambiguidade, contradição, lacuna — processo ongoing.

4. **Research-Driven Context** — research agents coletam contexto crítico ao longo do processo de spec: compatibilidade de lib, benchmarks, implicações de segurança, restrições organizacionais. Ver [[../../30-agents/mcp-integration]] (Context7) e [[../../30-agents/autonomous-execution]] §protocolo 5 fontes.

5. **Bidirectional Feedback** — realidade de produção informa evolução da spec. Métricas, incidentes, aprendizados operacionais viram inputs pra refinar spec.

6. **Branching for Exploration** — gerar múltiplas abordagens de implementação a partir da mesma spec, pra explorar trade-offs (performance × maintainability × UX × custo).

## Por que agora

Três tendências tornam SDD **possível e necessário**:

1. **IA** alcançou threshold onde natural language specs geram código confiável. Não é substituir devs — é amplificar efetividade automatizando tradução mecânica spec → impl. Amplifica exploração + criatividade + "começar de novo" fácil.

2. **Complexidade exponencial** do software. Sistemas integram dezenas de serviços, frameworks, deps. Alinhar tudo via processo manual vira impossível. SDD provê alinhamento sistemático via geração.

3. **Pace de mudança acelerado**. Pivotar não é exceção — é regra. Req mudam rapidamente. Dev tradicional trata mudança como disrupção (propagar mudança manualmente = lento ou arriscado). SDD transforma mudança em **regeneração sistemática**.

## O workflow (alto nível)

```
ideia vaga
  ↓ diálogo iterativo com IA
PRD compreensivo (AI faz clarifying questions, identifica edge cases)
  ↓ research agents coletam contexto
implementation plan (mapeia req → decisão técnica; toda escolha tem rationale)
  ↓ continuous consistency validation
código gerado (conforme spec + plan; testes também gerados)
  ↓ produção + feedback
spec atualizada (métricas, incidentes, pivô → spec evolui → re-geração)
```

Comparação com o ciclo operacional em [[sdd-workflow]]: aquele é **por task** (Discuss → Plan → Execute → Verify → Ship). Este aqui é o **super-ciclo** em que o projeto inteiro vive.

## Comandos `/speckit.*` (workflow automatizado)

Spec-kit formaliza 3 comandos que operacionalizam o método:

### `/speckit.specify <descrição>`

Transforma descrição em **spec estruturada** com gerenciamento automático de repo:
- Auto-numeração de features (001, 002, ...)
- Criação de branch `NNN-nome-semantico`
- Template-based generation de `specs/NNN-nome/spec.md`
- Estrutura de diretório `specs/NNN-nome/` pra todos os artefatos

### `/speckit.plan <detalhes técnicos>`

Transforma spec em **implementation plan**:
- Análise da spec (user stories, acceptance criteria)
- Conformidade com "constitution" do projeto (architectural principles)
- Tradução técnica (req → arquitetura)
- Geração de data models, API contracts, test scenarios
- Quickstart guide com cenários de validação

### `/speckit.tasks`

Plan → **task list executável**:
- Lê `plan.md` + `data-model.md` + `contracts/` + `research.md`
- Deriva tasks a partir de contratos, entidades, cenários
- Marca tasks paralelizáveis com `[P]`
- Escreve `tasks.md` pronto pra execução por agente

## Exemplo: chat feature em 15min

```bash
/speckit.specify Real-time chat system with message history and user presence
# → cria branch 003-chat-system, gera specs/003-chat-system/spec.md estruturado

/speckit.plan WebSocket for real-time, PostgreSQL for history, Redis for presence
# → gera plan.md, research.md (WebSocket lib comparisons), data-model.md (Message + User schemas),
#   contracts/ (WebSocket events, REST endpoints), quickstart.md, tasks.md
```

**15 minutos** = spec completa + plan detalhado + API contracts + data models. Tradicional: ~12 horas de documentação.

## Como SDD se relaciona com o que temos

Nosso setup:

| Conceito SDD | Nossa implementação |
|---|---|
| Specs como artefato primário | `specs/{requirements,design,tasks}/` versionado |
| Executable specs | Ainda não totalmente; tasks.md + `<verify>` blocos no plan são passos |
| Continuous refinement | STATE.md vivo + `sprint-auditor` ao fim de cada sprint |
| Research-driven context | Context7 MCP + 5 fontes do [[../../30-agents/autonomous-execution]] |
| Bidirectional feedback | AUDIT.md (findings pós-deploy + review) retro-alimenta spec |
| Branching for exploration | Ainda manual; spec-kit automatiza |
| Commands `/speckit.*` | **A adotar** — criar skills equivalentes no Claude Code |

**Próximo passo**: avaliar adoção dos comandos `/speckit.*` como skills do Claude Code no automation-agents.

## Comparação rápida

| Dimensão | Traditional | TDD | SDD |
|---|---|---|---|
| Fonte de verdade | Código | Testes | **Spec** |
| Maintenance | Editar código, atualizar doc à mão | Atualizar teste + código | Evoluir spec → regenerar código + teste |
| Mudança de req | Retrabalho manual | Re-teste + re-impl | Re-geração sistemática |
| Dívida técnica | Acumula silenciosa | Visível em testes red | Spec contradiz código → alerta |
| Exploração | Custa tempo real | Mais barato (refactor) | Regenerar do spec com variantes |

## Links

- [[_moc]] — methodology
- [[sdd-workflow]] — implementação operacional do ciclo 5-fase
- [[legacy-as-reference]] — como lidar com código pré-SDD
- [[../../30-agents/autonomous-execution]] — protocolo 5 fontes do research-driven context
- [[../../30-agents/mcp-integration]] — Context7 como research agent
- [[../../40-templates/AGENTS_SPEC]] — §2 e §10 sobre adoção
- [[../../60-sources/spec-kit/_index]] — fonte completa
- [[../../60-sources/get-shit-done/_index]] — fonte complementar (execução)
