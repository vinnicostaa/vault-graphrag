---
title: MOC — Playbooks de Agente
type: moc
tags: [moc, agents, playbooks]
created: 2026-04-22
---

# Playbooks de Agente

Procedimentos repetíveis que o agente **deve** seguir em situações canônicas. Cada playbook é um contrato: se o gatilho acontece, o playbook roda. Não é sugestão.

## Playbooks

- [[research-loop]] — ciclo obrigatório de pesquisa antes de afirmar ou decidir
- [[dual-check]] — validação por segundo agente com contexto limpo
- [[dependency-audit]] — auditoria periódica de dependências (atualização + CVE)
- [[cve-scan]] — scan de vulnerabilidades e resposta a CVE
- [[onboarding-novo-projeto]] — handshake inicial agente↔dev pra novo projeto
- [[plan-and-execute]] — como planejar antes de executar código em tasks não-triviais
- [[rollback]] — quando e como reverter mudança
- [[community-summary]] — Graph-RAG Global Search: sumários narrativos bottom-up por community, dentro do `_moc.md` existente (ver [[../../00-meta/STATE|D-vault-003]])
- [[gsd-resync]] — re-sincronização da `gsd-library/` quando upstream `gsd-build/get-shit-done` evolui (ver [[../../00-meta/STATE|D-vault-004]])
- [[direct-write-for-vault-mutation]] — quando operador principal escreve direto no vault (via MCP) em vez de delegar a subagent; formaliza padrão pós-incidentes A-vault-010/042/043 (ver [[../../00-meta/STATE|D-vault-018]])

## Relação com o resto do vault

- [[../autonomous-execution]] — define o modo autônomo; playbooks são executados dentro dele
- [[../mcp-integration]] — MCPs usados pelos playbooks (Context7, Obsidian, etc.)
- [[../../CLAUDE]] — contrato raiz, referencia os playbooks como obrigatórios
- [[../../10-knowledge/methodology/sdd-workflow]] — ciclo 5-fase que consome esses playbooks
