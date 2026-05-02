[FFV Academy](../index.html)/Claude Code: do zero ao poder total

⊕

Blog

# Claude Code: do zero ao poder total

Para quem já sabe o básico e quer ir fundo. Aqui o assunto é como os modelos funcionam em produção: memória, roteamento, ferramentas, agentes. O lado técnico que pouca gente explica direito.

18artigos

1255XP total

📄PDF da trilha

🖥️Apresentar trilha

[](../aprenda/anthropic-ecossistema/index.html)

01

## ⊕ O ecossistema Anthropic: Claude, modelos, produtos e roadmap

Claude 3, 3.5, 4 — o que diferencia Haiku, Sonnet e Opus. API, Claude.ai, Claude Code, MCP: o mapa completo do que a Anthropic oferece e quando usar cada produto.

⏱ 8 min·+40 XP

→[](../aprenda/claude-code-primeiros-passos/index.html)

02

## 🖥️ Claude Code: instalação, autenticação e primeiro uso real

Como instalar o Claude Code, autenticar com sua conta Anthropic, os primeiros comandos no terminal e como Claude Code difere de outros assistentes de IA.

⏱ 10 min·+50 XP

→[](../aprenda/claude-code-modos-de-uso/index.html)

03

## 🔀 Modos de uso: interativo, não-interativo, pipe e headless

Claude Code tem 4 modos principais. Interativo (chat), não-interativo (scripts), pipe (stdin/stdout) e headless (automação CI/CD). Quando e como usar cada um.

⏱ 12 min·+60 XP

→[](../aprenda/claude-code-claude-md/index.html)

04

## 📋 CLAUDE.md: como dar memória, contexto e personalidade ao agente

O CLAUDE.md é o arquivo que define como Claude age no seu projeto — comandos, contexto, regras e preferências. Como criar um CLAUDE.md eficaz e o que evitar.

⏱ 13 min·+65 XP

→[](../aprenda/claude-code-permissoes/index.html)

05

## 🔐 Permissões e segurança: o que Claude pode e não pode fazer

Sistema de permissões do Claude Code: trust levels, o que é permitido por padrão, como configurar limites, sandbox e boas práticas de segurança ao usar IA no terminal.

⏱ 11 min·+55 XP

→[](../aprenda/claude-code-workflow-diario/index.html)

06

## 🔄 Workflow diário: explore → plan → code → commit

O loop de produtividade do Claude Code: explorar o codebase, planejar com o agente, implementar com controle e commitar com confiança. Como encaixar Claude Code no seu dia a dia real.

⏱ 14 min·+70 XP

→[](../aprenda/claude-code-context-management/index.html)

07

## 🧠 Gerenciamento de contexto: como Claude Code pensa e decide

A janela de contexto é o limite do agente — como ela funciona, o que Claude Code vê, como compactar contexto, quando reiniciar e como usar subagents para preservar contexto limpo.

⏱ 13 min·+65 XP

→[](../aprenda/claude-code-skills-commands/index.html)

08

## ⚡ Skills e slash commands: criar seus próprios workflows

Skills são workflows customizados invocados via /nome-do-comando. Como criar skills para tarefas repetitivas: deploy, revisão de código, geração de documentação.

⏱ 13 min·+65 XP

→[](../aprenda/claude-code-subagents/index.html)

09

## 🤖 Subagents: delegar tarefas a agentes isolados

Subagents são instâncias isoladas do Claude que rodam em janelas de contexto separadas. Como criar, configurar e usar subagents para tarefas paralelas, pesadas ou especializadas — sem poluir o contexto principal.

⏱ 15 min·+75 XP

→[](../aprenda/claude-code-hooks/index.html)

10

## 🪝 Hooks: automatizar revisões, validações e ações customizadas

Hooks são scripts que rodam automaticamente em eventos do Claude Code (antes/depois de editar, submeter, etc.). Como criar hooks para lint, testes, validação e notificações.

⏱ 14 min·+70 XP

→[](../aprenda/claude-code-mcp-na-pratica/index.html)

11

## 🔌 MCP na prática: conectar Drive, GitHub, Slack e bancos de dados

Model Context Protocol (MCP) permite que Claude acesse ferramentas externas. Como configurar servidores MCP prontos para Google Drive, GitHub, Postgres, Slack e mais.

⏱ 15 min·+75 XP

→[](../aprenda/claude-code-github-integration/index.html)

12

## 🐙 Claude Code + GitHub: PRs, issues e code review automatizados

Como integrar Claude Code com GitHub para automatizar revisão de PRs, triagem de issues, geração de changelogs e análise de diff. OAuth, MCP do GitHub e workflows com Actions.

⏱ 14 min·+70 XP

→[](../aprenda/claude-code-sdk/index.html)

13

## 📦 Claude Agent SDK: usar Claude Code como biblioteca

O Claude Agent SDK transforma Claude Code numa biblioteca Python/TypeScript. query(), built-in tools, hooks programáticos, subagents, MCP, sessions — tudo via código, sem terminal.

⏱ 16 min·+80 XP

→[](../aprenda/claude-cowork/index.html)

14

## 🏢 Claude Cowork: plugins, tarefas agendadas e pesquisa em escala

Claude Cowork vai além do chat: plugins especializados, tarefas agendadas, pesquisa profunda em arquivos e na web, tudo integrado ao seu fluxo de trabalho diário.

⏱ 14 min·+70 XP

→[](../aprenda/claude-code-cheatsheet-pratico/index.html)

15

## 📋 Cheatsheet prático: 50+ comandos, 30 atalhos, 20 flags — a referência executiva

O canivete suíço do Claude Code em 2026: 70+ slash commands built-in organizados por categoria, atalhos de teclado (Ctrl+O transcript, Esc+Esc rewind, Shift+Tab modes), flags da CLI mais usadas, 15 padrões do dia a dia e 10 variáveis de ambiente essenciais. A página que você deixa aberta em um monitor.

⏱ 18 min·+90 XP

→[](../aprenda/claude-code-paralelismo-na-pratica/index.html)

16

## ⚡ Paralelismo na prática: worktrees, fan-out, background e múltiplas sessões

Rode N subagents em paralelo sem conflito de arquivos: git worktrees automáticos, tasks em background (Ctrl+B), múltiplas sessões simultâneas, /batch para migrações massivas, /loop para polling, /schedule para cron remoto. O módulo que transforma Claude Code de assistente em frota.

⏱ 17 min·+85 XP

→[](../aprenda/claude-code-multi-projeto-multi-contexto/index.html)

17

## 🗂️ Multi-projeto e contextos persistentes: sessions, settings hierarchy e team onboarding

Trabalhe em múltiplos repos ao mesmo tempo com contextos isolados: sessions nomeadas (--name, --resume, --from-pr), --add-dir para cross-project, settings hierarchy (enterprise/user/project/local), /team-onboarding para gerar guia do time e /branch/--fork-session para experimentar sem destruir.

⏱ 16 min·+80 XP

→[](../aprenda/capstone-claude-code-team-playbook/index.html)

18

## 🏁 Capstone: playbook de Claude Code pro seu time

Projeto: documentar CLAUDE.md profissional pra seu repo + catálogo de skills + hooks de CI (code review auto, spec review) + settings hierarchy org-wide + onboarding guide. Entregável: PR no repo do time com Claude Code setup maduro.

⏱ 20 min·+90 XP

→

[← Voltar à home](../index.html)
