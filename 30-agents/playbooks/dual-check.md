---
title: Playbook — Dual-Check
type: playbook
tags: [playbook, agent, review, dual-check]
created: 2026-04-22
---

# Playbook — Dual-Check (validação por segundo agente)

Processo onde **um segundo agente com contexto limpo** revisa uma decisão ou implementação do primeiro agente. Objetivo: pegar alucinação, bias de raciocínio, premissas não declaradas, leitura apressada de fonte.

**Regra dura**: em modo autônomo sem supervisão humana, dual-check é obrigatório para qualquer decisão de alto impacto.

## Quando disparar

| Nível | Gatilho | Dual-check? |
|---|---|---|
| 🔴 Alto | Arquitetura, contrato público, schema DB, regra segurança, pattern novo | **Sim** sempre |
| 🟡 Médio | Refactor cross-módulo, integração com SDK externo, deploy config | **Sim** em autônomo |
| 🟢 Baixo | Bug-fix isolado, typo, doc, testes locais | Não obrigatório |

Ver também gatilhos em [[../../CLAUDE#5-dual-check-2o-par-de-olhos-obrigatório]].

## Fluxo

### Preparação (agente-A, o propositor)

Escrever **pacote de revisão** auto-contido:

```markdown
## Dual-Check Request — <data ISO> — <título>

### Contexto
<1 parágrafo: o que se quer decidir/fazer e por quê>

### Artefatos a revisar
- Arquivo(s) alterado(s): <paths + ranges de linha>
- Spec relacionada: <link>
- Task: <link>

### Proposta do agente-A
<proposta completa>

### Fontes consultadas
- <URL> (versão, data)
- ...

### Alternativas consideradas e descartadas
- <alt 1> — motivo do descarte
- ...

### Pontos onde o A está incerto
<áreas de baixa confiança, premissas, áreas não pesquisadas>

### Critérios de aceite
- <comportamento X>
- <gate: build/lint/test passa>
- <coverage threshold>
- <compat backward, se aplicável>
```

### Execução (agente-B, revisor)

Instanciar via `Agent` tool, `subagent_type=general-purpose` ou `Plan`, com **contexto limpo** (não reusar histórico do A).

```
Prompt do agente-B:
  Você está fazendo dual-check do agente-A.
  Leia o pacote em <path>.
  NÃO assuma que A está certo. Re-pesquise as fontes.
  Responda nos blocos abaixo. Sem narrar raciocínio interno.

  ## Avaliação
  - Concordo integralmente / Concordo com ressalvas / Discordo

  ## Se ressalvas ou discordo
  - Ponto específico: <quote de A>
  - Problema: <o que está errado ou faltando>
  - Fonte que contradiz: <URL>
  - Proposta alternativa: <...>

  ## Riscos detectados que A não listou
  - <lista>

  ## Correção factual
  - A disse: <claim>
  - Fonte oficial diz: <outra coisa>

  ## Premissas não declaradas de A
  - <lista>

  ## Veredicto: aprovado / aprovado-com-ajustes / reprovado
```

### Consolidação

- **A e B concordam integralmente** → decisão segue, registra em STATE D-### com "dual-check: aprovado por B em <data>"
- **A e B concordam com ressalvas** → A incorpora ressalvas, re-submete ao B se mudança relevante; caso contrário, prossegue citando as ressalvas
- **A e B divergem** → terceiro ciclo:
  - Ambos re-pesquisam com foco no ponto de divergência
  - Se ainda divergem → escalar pro usuário humano
  - Nunca o A "ganha" por default

### Registro

Em STATE.md, campo `dual-check` do D-###:

```yaml
dual-check:
  agent_a: primary_session
  agent_b: review_session_<timestamp>
  verdict: aprovado | aprovado-com-ajustes | escalado
  divergence_summary: <se houve>
  resolution: <como resolveu>
  date: 2026-04-22
```

## Requisitos de execução

- Agente-B **NÃO** recebe o histórico da conversa do A — apenas o pacote auto-contido.
- Agente-B deve usar as **mesmas** ferramentas disponíveis (Context7, WebFetch, Obsidian MCP).
- Agente-B não pode editar código nesta fase — só revisão.
- Tempo mínimo do B: o suficiente pra re-abrir pelo menos 1 fonte que A citou + 1 fonte nova.

## Modo paralelo (acelerado)

Em decisão urgente, 2 agentes-B podem rodar em paralelo (dois Opus independentes). Se ambos aprovam, reforça. Se divergem entre si, A precisa reconciliar.

## Anti-padrão a evitar

- ❌ Agente-B que só lê a proposta do A e diz "ok parece bom" — isso é rubber-stamp, não dual-check
- ❌ Agente-B que recebe o contexto inteiro da conversa do A (vai enviesar)
- ❌ Pular dual-check porque "é óbvio" — o óbvio é exatamente onde alucinação se esconde
- ❌ Deixar A reescrever a avaliação do B antes de registrar

## Links

- [[_moc]]
- [[research-loop]]
- [[../autonomous-execution]]
- [[../../CLAUDE]]
