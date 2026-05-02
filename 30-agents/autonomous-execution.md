---
title: Autonomous Execution — modo não-supervisionado
type: guide
tags: [agents, autonomous, methodology, sdd]
source: .kiro/steering/autonomous-execution.md
created: 2026-04-22
aliases: [Autonomous Mode, Auto Mode, Modo Autônomo]
---

# Autonomous Execution — operação não-supervisionada

Modo de operação quando o usuário autoriza **explicitamente** trabalho prolongado sem supervisão. O agente **decide e executa** em vez de pausar a cada decisão técnica, seguindo protocolo rigoroso pra manter rastreabilidade.

**Regra fundamental**: agente **não pode parar por dúvida técnica**. Dúvida técnica se resolve por pesquisa sistemática + decisão documentada, não por interrupção.

## Quando ativar

Palavras-chave explícitas do usuário:
- "vou dormir / saio agora / volto em X horas"
- "não pare até acabar" / "só pare quando acabar"
- "autonomia total" / "total permissão" / "você tem total permissão"
- "você decide" / "deixo em suas mãos"
- "trabalhe sozinho" / "siga sem me perguntar"
- "execute tudo" / "resolve tudo"
- Auto mode flag ativa (system reminder)

Default é **não-autônomo** — agente pergunta antes de decisões não-triviais. Modo autônomo é **opt-in explícito**, não inferência.

## Regra de ouro

> **Parar por dúvida técnica é violação do protocolo.** Se há ambiguidade, ela é resolvida por pesquisa, não por interrupção.

A única exceção são as 5 categorias de [[#Pausa obrigatória]] listadas abaixo — todas envolvem **ação destrutiva ou com blast radius**. Nenhuma envolve "qual padrão usar" ou "qual abordagem é melhor".

## Protocolo de decisão autônoma

Toda decisão técnica não-trivial (escolha de padrão, estratégia de integração, trade-off arquitetural, severidade de finding) segue este fluxo **sem atalhos**:

### Fase 1 — Levantamento de 5 fontes (paralelo)

Consulta **paralela**, nunca sequencial:

1. **Docs oficiais versionadas** — via Context7 MCP (`resolve-library-id` → `get-library-docs`). Treino do modelo pode estar desatualizado; Context7 entrega a versão do lockfile.
2. **Source code do package** — ler arquivos reais no cache local (`$GOMODCACHE`, `node_modules`, `~/.cargo/registry`, etc.). Descobre shapes reais, quirks de versão.
3. **Práticas de grandes empresas** — padrão Google/Uber/Stripe/Netflix/GitHub/Shopify pra este problema em produção. WebSearch/WebFetch em engineering blogs + papers. Se prática é controversa, **documentar as duas escolas** (ex: "Uber + Netflix usam X, Google usa Y; neste projeto Y cabe melhor porque...").
4. **Boas práticas genéricas** — SOLID, DDD, Clean Architecture, OWASP, 12-Factor, SRE playbooks. São tiebreaker quando empresas grandes divergem.
5. **Contexto do projeto** — specs, steering, STATE, AUDIT, código existente. **Padrão que o projeto já adota vence** tendência externa moderna — consistência > modernidade.

Nenhuma fonte sozinha é verdade absoluta:

| Fonte | O que valida | Limite |
|---|---|---|
| Docs oficiais | API correta na versão atual | Pode omitir edge cases |
| Source packages | Shape real + quirks | Pode confundir detail com contrato |
| Grandes empresas | Escalabilidade comprovada | Pode ser over-engineering |
| Boas práticas | Princípios estáveis | Podem conflitar entre si (DRY vs YAGNI) |
| Contexto do projeto | Coerência interna | Pode ter herdado má prática do legacy |

### Fase 2 — Dual-agent validation (decisões de alto impacto)

Decisões que afetam **arquitetura, segurança, schema de banco, contrato de API público** exigem dual-agent:

1. **Orquestrador** (main agent) spawna **Analista** — subagente em **Opus**. Tarefa: pesquisar as 5 fontes e produzir recomendação fundamentada.
2. Orquestrador spawna **Revisor** — subagente independente em **Opus**. Tarefa: **não** confiar na recomendação do Analista; pesquisar as mesmas 5 fontes **de forma independente** e julgar se a recomendação bate.
3. **Convergência total** → orquestrador executa.
4. **Divergência** → orquestrador lê ambos os relatórios, identifica **onde** divergem, pesquisa 1 fonte adicional pra desempatar, decide com justificativa registrada em STATE.

**Ambos em Opus obrigatoriamente.** Sonnet/Haiku não têm fôlego pra validação cruzada.

Decisões de **baixo/médio impacto** (escolha local, refactor interno, ordem de campos) — orquestrador decide sozinho seguindo a Fase 1. Dual-agent é caro; usa só quando justifica.

### Fase 3 — Registro de decisão em STATE

Toda decisão autônoma não-trivial vai em `STATE.md`:

```
### D-0XX — {título curto}
- **Contexto**: {problema}
- **Decisão tomada autonomamente em {data ISO}**: {escolha}
- **Fontes consultadas**:
  - Context7: {lib}@{versão} topic "{topic}"
  - Source package: `$CACHE/{lib}@{v}/{path}`
  - Grandes empresas: {referência}
  - Boas práticas: {princípio aplicado}
  - Contexto do projeto: {arquivo/padrão existente}
- **Alternativas rejeitadas**: {A, B, C} — por quê
- **Dual-agent usado**: {sim/não} — {resumo da convergência/divergência}
- **Reversibilidade**: {baixa/média/alta} — se baixa, nota explícita
```

Isso dá trail pro humano revisar de manhã e saber **exatamente** o que foi decidido e por quê.

### Fase 4 — Execução

Executa seguindo ciclo SDD 5-fase (ver [[../10-knowledge/methodology/sdd-workflow]] — a criar). Decisão registrada vira o plano (Fase 2 do SDD).

## Pausa obrigatória (5 categorias)

Estas são as **únicas** pausas válidas em modo autônomo. Cada uma envolve **blast radius irreversível** ou **violação de contrato de segurança**:

1. **Ações destrutivas em dado real** — `DROP TABLE`, `TRUNCATE`, `DELETE` sem `WHERE`, `rm -rf` fora de `/tmp`, `git push --force` em `main`, revogar/rotacionar credencial em produção.
2. **Credenciais / secrets** — gerar, rotacionar, inserir em `.env`, expor em log, compartilhar em commit ou PR body.
3. **Exfiltração de dados** — posting em canal externo (Slack, GitHub comment, webhook público), upload pra serviço de terceiro, print em arquivo não-gitignored.
4. **Quebra de compatibilidade** com contrato público já deployado — mudar schema JSON de resposta de endpoint live, alterar assinatura de webhook consumido por terceiro, renomear env var de produção.
5. **Scope creep radical** — task pede X, agente identifica Y fundamental. Y vai pra AUDIT.md ou STATE.md, **não** se implementa no escopo atual sem autorização. Dual-track: task prossegue, Y vira item separado.

Fora destas 5, **agente decide**. "Qual padrão de retry usar" **não** é pausa — é pesquisa + decisão autônoma. "Qual cache pra session" **não** é pausa. "Renomear variável ambígua" **não** é pausa. "Promover VO a tipo separado" **não** é pausa.

## Antipatterns que quebram autonomous mode

❌ "Vou aguardar sua confirmação para prosseguir" — pausa inválida (salvo 5 categorias)
❌ "Prefere X ou Y?" — escolha é do agente após pesquisa
❌ "Tenho algumas opções para avaliar" — avalie e escolha
❌ "Posso prosseguir com X?" — autorização já foi dada; prossiga
❌ "Antes de continuar, gostaria de confirmar" — execute sem confirmar
❌ Decidir sem registro em STATE.md — falta rastro
❌ Pular Context7 por "já conhecer a lib" — treino pode estar stale
❌ Usar dual-agent pra decisão de baixo impacto — custo sem retorno
❌ Spawnar dual-validate em Sonnet/Haiku — Opus é obrigatório

## Comunicação durante execução autônoma

Agente **continua comunicando progresso** (uma linha por fase, momentos-chave), **sem** perguntar. Tom é **report**, não **consulta**:

```
> Discutindo task X. Lendo requirements + design + antipatterns.
> Plano escrito com <verify>: 6 arquivos novos, 2 editados, coverage target 90%.
> Decisão autônoma D-025: SELECT FOR UPDATE vs advisory lock — escolhi FOR UPDATE (Uber usa em inventory; pgx suporta nativo; PBT cobre).
> Executando: migration + sqlc + entity + repo + use case + handler + testes.
> Verificação: build limpo, vet limpo, 2000 iterações PBT passando, coverage 94.6%.
> Commit `<módulo>: <descrição> (task X)`.
> README atualizado. AUDIT checado — zero findings.
> Próxima task: Y.
```

Frases terminam em ponto, não interrogação. Verbos no pretérito (ação concluída) ou gerúndio (ação em curso), não condicional.

## O que o usuário vê ao voltar

1. **Progresso mensurável** — múltiplas tasks concluídas, commits criados, PRs abertos, testes passando.
2. **STATE.md atualizado** — todas as decisões autônomas (`D-0XX`) + dívidas descobertas.
3. **AUDIT.md em dia** — se sprint virou noite, auditor rodou e findings registrados.
4. **Diff explicável** — cada arquivo tem commit que descreve o **porquê**, bate com STATE.
5. **Zero bloqueios silenciosos** — se houve pausa obrigatória, bloco no início do relatório final explicando o quê e por quê.

Teste de qualidade: usuário consegue reconstruir o raciocínio **sem perguntar ao agente**. Tudo está em commits + STATE.md + AUDIT.md + release notes.

## Relacionamento com outros steerings/skills

- **[[../10-knowledge/methodology/sdd-workflow]]** *(a criar)* — ciclo 5-fase continua inteiro. Modo autônomo substitui "pausar pra confirmar" por "decidir e registrar" dentro de cada fase.
- **mcp-integration** *(a criar)* — Context7 obrigatório na Fase 1. Postgres MCP usado pra `EXPLAIN ANALYZE` em decisões de query.
- **dual-validate skill** — implementação concreta do dual-agent pattern.
- **task-executor skill** — usa este steering em modo autônomo; Gate 0 (AUDIT) continua válido.
- **STATE.md do projeto** — canal oficial de registro de decisão autônoma.

## Links

- [[_moc]] — MOC de agentes
- [[claude-code-plugins]] — plugins que entram nas fases SDD
- [[../10-knowledge/principles/do-dont-checklist]] — checklist consolidado
