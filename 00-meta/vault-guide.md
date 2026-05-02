---
title: Vault Guide — como operar este vault
type: guide
tags: [meta, governance]
created: 2026-04-22
updated: 2026-04-23
---

# Vault Guide

Guia operacional do vault `automation-agents`. Lê isto se você (humano ou agente) vai criar/editar notas aqui dentro.

## Princípios

1. **O vault é o cérebro dos agentes**. Qualquer conhecimento que precisar ser recuperado em futuras sessões mora aqui.
2. **Não é banco de arquivos**. Nota que não será lida e linkada mais de uma vez pode virar nota só no arquivo de origem.
3. **Atomic notes**. 1 nota = 1 conceito. Se cabe em 40-80 linhas, ótimo. Mais que 200 linhas sem split, suspeite.
4. **Link first, folder second**. Pasta é organização secundária. Navegação real acontece por `[[backlinks]]` + MOCs.
5. **Global sobre projeto**. Conhecimento atemporal vai no global (`10-knowledge/`, `20-stacks/`, `30-agents/`, `40-templates/`). Projeto só tem o que é regra de negócio ou ubíquo ao produto. O resto é **referência** ao global.

## Estrutura

Prefixos numéricos (Johnny.Decimal) pra ordenação estável na sidebar do Obsidian:

```
00-meta/         governança do vault em si (STATE, AUDIT, CHARTER, guide)
10-knowledge/    conhecimento técnico atemporal (Clean Arch, patterns, providers, etc.)
20-stacks/       templates por stack (Go, TS, Rust, Flutter, Postgres)
30-agents/       operação do sistema multi-agente
40-templates/    arquivos replicáveis (SPEC, STATE template, etc.)
50-projects/     projetos concretos (Master Síndico como exemplo/cliente)
60-sources/      matéria-prima externa (sites baixados, repos, PDFs)
90-archive/      versões antigas, rascunhos consolidados
```

**Regra**: 3 níveis máximo. `10-knowledge/patterns/strategy-pattern.md` OK. `10-knowledge/patterns/design/behavioral/strategy.md` **não** — ou acha outro agrupamento ou vira nota plana com tags.

## Frontmatter canônico

Toda nota começa com frontmatter YAML:

```yaml
---
title: <título legível pra humano>
type: concept | pattern | antipattern | template | moc | note | guide | project | source | agent-contract | playbook | state | audit | summary
tags: [topic-1, topic-2]
source: <origem se destilado de fora>   # opcional
created: 2026-04-22
updated: 2026-04-23                      # opcional, atualizar em edits relevantes
aliases: [Outro Nome, Sinônimo]          # opcional, Obsidian usa em [[Outro Nome]]
---
```

`type` é o discriminador mais útil — permite dataview queries como "todos os antipatterns", "todos os templates", etc.

## MOCs (Maps of Content)

Cada pasta principal tem `_moc.md`. Função: servir de hub pro Obsidian Graph View e pra navegação humana.

Estrutura padrão de um MOC:

```markdown
---
title: MOC — <área>
type: moc
tags: [moc]
---

# <Área>

1 parágrafo descrevendo o que vive aqui.

## Notas

- [[nota-1]] — 1 linha de hook
- [[nota-2]] — 1 linha de hook

## Vizinhos no grafo

- [[MOC da pasta irmã]] — relação
- [[MOC raiz]] (_index)
```

**Não** precisa listar tudo — só o que importa ser achado sem scroll. Use Dataview query se a lista é dinâmica.

## Como criar uma nota nova

1. **Decidir o tipo**: concept / pattern / antipattern / template / moc / note / guide / project / source / agent-contract / playbook / state / audit / summary. Ver lista canônica em frontmatter acima. `state`/`audit`/`summary` adicionados em D-vault-013 (2026-04-24).
2. **Escolher a pasta**: segue o prefix + área temática. Na dúvida, `10-knowledge/` + tag.
3. **Nome do arquivo**: `kebab-case.md`. Sem data, sem números de prefixo (exceto em pastas numeradas).
4. **Frontmatter** obrigatório (template acima).
5. **Linkar** pelo menos 2 outras notas no corpo via `[[...]]`. Se a nota nasce isolada, é sinal de que ainda não pertence.
6. **Atualizar o MOC** da pasta se for nota de destaque.

## Como ingerir material externo

Quando destilar conteúdo de fonte externa ou de `60-sources/`:

1. Criar nota atômica em `10-knowledge/...` com o conceito extraído.
2. Frontmatter inclui `source: <origem>` pra rastreabilidade.
3. Linkar de volta pro material original se ele ainda existir no vault.
4. **Nunca** copiar parágrafos inteiros — destilar em linguagem própria. Citação de código é OK.
5. Marcar no material original (ou em `60-sources/_ingested/`) que já foi processado.

## Regra de separação projeto ↔ global (importante)

Ao criar conteúdo num projeto (`50-projects/<slug>/`), **pergunte**:

- É regra de negócio do produto? → fica no projeto
- É linguagem ubíqua do domínio? → fica no projeto
- É fluxo específico desse produto? → fica no projeto
- É conhecimento técnico público/open-source (pattern, método, provider) → vai pro global
- É aplicação concreta de conhecimento global? → **fica no projeto MAS linka pro global** ("ver padrão em `[[../../../10-knowledge/...]]`; aqui aplicado a X")

Se duplicar pattern/provider/princípio no projeto: errou. Refatora pra linkar o global.

## Governança

- `50-projects/<slug>/STATE.md` — decisões ativas (D-###) e bloqueadores
- `50-projects/<slug>/11-audit/AUDIT.md` — findings abertos (A-###)
- `50-projects/<slug>/SESSION_CHARTER.md` — briefing da sessão corrente (se houver)

Atualizar estes 3 toda sessão. Especialmente STATE quando tomar decisão nova.

## Idioma

- Corpo das notas: **pt-BR**
- Frontmatter, títulos de código, identifiers técnicos: **inglês**
- Quando a nota é API pública bilingual (extraído do padrão `CODING_MANUAL` do vault antigo), começa com docstring EN acima e pt-BR abaixo — mas isso é **só** em templates de código, não em conteúdo.

## Mutação via MCP Obsidian (regra dura)

Este vault tem um clone filesystem (`automation-agents/vault/` no disco) que NÃO sincroniza automaticamente. Toda criação/edição/movimentação de nota passa por `mcp__obsidian__*`:

- Leitura: `mcp__obsidian__read_note`, `list_directory`, `get_notes_info`, `search_notes`
- Escrita: `mcp__obsidian__write_note`, `update_frontmatter`, `patch_note`
- Movimentação: `mcp__obsidian__move_note`
- Exclusão: `mcp__obsidian__delete_note` (com confirmPath)

Editar o clone filesystem direto é proibido — descompassa o vault real.

## Ferramentas de suporte

- **Smart Connections** — embeddings vetoriais locais (sem API key, sem cloud); ativar pra ter busca semântica complementando o grafo simbólico.
- **Dataview** — queries estruturadas via frontmatter (`type`, `tags`, `source`).
- **Graph View** — visualiza o grafo; MOCs viram hubs.
- **Claude Code + MCP Obsidian** — como este vault é mantido (padrão Karpathy LLM Wiki: LLM é o mantenedor, Obsidian é o visualizador).
- Ver [[../10-knowledge/methodology/graph-rag]] pra entender a arquitetura completa.

## Checklist antes de fechar sessão

- [ ] STATE.md tem alguma decisão nova da sessão registrada?
- [ ] AUDIT.md tem algum finding novo aberto ou fechado?
- [ ] Notas criadas têm frontmatter + pelo menos 2 `[[links]]`?
- [ ] MOCs impactados foram atualizados?
- [ ] Nada de conteúdo específico de cliente em `10-knowledge/` (conteúdo específico vai em `50-projects/`)?
- [ ] Nada foi duplicado entre global e projeto (projeto só referencia)?
