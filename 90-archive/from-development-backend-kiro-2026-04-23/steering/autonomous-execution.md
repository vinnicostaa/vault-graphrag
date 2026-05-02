---
inclusion: always
---

# Autonomous Execution — Modo de Operação Não-Supervisionado

Quando o João (ou outro dev senior do projeto) autoriza explicitamente operação prolongada sem supervisão — ex: "vou dormir, volto em X horas, só pare quando acabar" / "autonomia total" / "você decide" / "total permissão" — o agente **muda de modo**: em vez de pausar e perguntar em cada decisão técnica, **decide e executa** seguindo o protocolo rigoroso deste documento.

**Regra fundamental**: o agente **não pode parar por dúvida técnica**. Dúvida técnica é resolvida por pesquisa sistemática + decisão documentada, não por interrupção do trabalho.

---

## Quando ativar modo autônomo

Palavras-chave que o usuário diz e que acionam o modo:

- "vou dormir / saio agora / volto em X horas"
- "não pare até acabar" / "só pare quando acabar"
- "autonomia total" / "total permissão" / "você tem total permissão"
- "você decide" / "deixo em suas mãos"
- "trabalhe sozinho" / "siga sem me perguntar"
- "execute tudo" / "resolve tudo"
- Auto mode flag ativo (system reminder)

Default é **não-autônomo** — agente pergunta antes de decisões não-triviais. Modo autônomo é **opt-in explícito** pelo usuário, não inferência.

---

## Regra de ouro do modo autônomo

> **Parar por dúvida técnica é violação do protocolo.** Se há ambiguidade, ela é resolvida por pesquisa, não por interrupção.

A única exceção são as 5 categorias de pausa obrigatória listadas abaixo — todas envolvem **ação destrutiva ou com blast radius** que exige confirmação humana. Nenhuma envolve "qual padrão usar" ou "qual abordagem é melhor".

---

## Protocolo de decisão autônoma

Toda decisão técnica não-trivial (escolha de padrão, estratégia de integração, trade-off arquitetural, severidade de finding, correção de bug ambíguo) segue este fluxo **obrigatório** — sem atalhos:

### Fase 1 — Levantamento de fontes (5 obrigatórias)

Consulta **paralela** (tudo de uma vez, não sequencial):

1. **Docs oficiais versionadas** — via Context7 MCP (`resolve-library-id` → `get-library-docs`). Treino do modelo pode estar desatualizado; Context7 entrega a versão do `go.mod`.
2. **Source code do package** — ler arquivos reais em `$GOMODCACHE` (`go env GOMODCACHE` → `$GOPATH/pkg/mod/<lib>@<versão>/`). Descobre struct shapes, getters disponíveis, quirks de versão que a doc pode omitir.
3. **Práticas de grandes empresas** — que padrão Google/Uber/Stripe/Netflix/GitHub/Shopify usa para este problema em produção. WebSearch / WebFetch para engineering blogs públicos + papers. Se a prática é controversa, **documentar as duas escolas** (ex: "Uber + Netflix usam X, Google usa Y; neste projeto Y cabe melhor porque...").
4. **Boas práticas genéricas** — SOLID, DDD, Clean Architecture, OWASP, 12-Factor, SRE playbooks. Essas regras não mudam com hype e são tiebreaker quando empresas grandes divergem.
5. **Contexto do projeto** — `specs/requirements/` + `specs/design/` + `steering/` + `STATE.md` + `AUDIT.md` + código existente. **O padrão que o projeto já adota vence** qualquer tendência externa moderna — consistência > modernidade.

Nenhuma das 5 sozinha é verdade absoluta. Cada uma contribui um ângulo:

| Fonte | O que valida | Limite |
|---|---|---|
| Docs oficiais | API correta na versão atual | Pode omitir edge cases do source |
| Source packages | Shape real + quirks | Pode confundir implementation detail com contrato |
| Grandes empresas | Escalabilidade comprovada | Pode ser over-engineering para o escopo do projeto |
| Boas práticas | Princípios estáveis | Podem conflitar entre si (DRY vs YAGNI) |
| Contexto do projeto | Coerência interna | Pode ter herdado má prática do legacy |

### Fase 2 — Dual-agent validation (para decisões de alto impacto)

Decisões de **alto impacto** (afetam arquitetura, segurança, schema de banco, contrato de API público) **exigem** o pattern dual-agent — ver `.claude/skills/dual-validate.md`. Resumo:

1. Orquestrador (main agent) spawna **Analista** — subagente em **Opus** — tarefa: pesquisar as 5 fontes e produzir uma recomendação fundamentada.
2. Orquestrador spawna **Revisor** — subagente independente em **Opus** — tarefa: **não** confiar na recomendação do Analista; pesquisar as mesmas 5 fontes **de forma independente** e julgar se a recomendação bate.
3. Convergência total → orquestrador executa.
4. Divergência → orquestrador lê ambos os relatórios, identifica **onde** divergem, pesquisa 1 fonte adicional para desempatar, decide com justificativa registrada em `STATE.md`.

**Ambos os subagentes são obrigatoriamente Opus** — Sonnet/Haiku não têm fôlego para a tarefa de validação cruzada.

Para decisões de **baixo/médio impacto** (escolha local, refactor interno, ordem de campos), o orquestrador decide sozinho seguindo a Fase 1. Dual-agent é custoso — usar quando justifica.

### Fase 3 — Registro de decisão

Toda decisão autônoma não-trivial é registrada em `.kiro/STATE.md` no formato:

```
### D-0XX — {título curto}
- **Contexto**: {problema}
- **Decisão tomada autonomamente em {data ISO}**: {escolha}
- **Fontes consultadas**:
  - Context7: {lib}@{versão} topic "{topic}"
  - Source package: `$GOMODCACHE/{lib}@{v}/{path}`
  - Grandes empresas: {Uber/Netflix/Stripe — link/referência}
  - Boas práticas: {DDD/SOLID/OWASP — qual regra}
  - Contexto do projeto: {arquivo/padrão existente}
- **Alternativas rejeitadas**: {A, B, C} — por quê
- **Dual-agent usado**: {sim/não} — {se sim, resumo da convergência/divergência}
- **Reversibilidade**: {baixa/média/alta} — se alta, sem burocracia; se baixa, nota explícita
```

Isso deixa trilha para o João revisar de manhã e saber **exatamente** o que foi decidido e por quê. Evita surpresa.

### Fase 4 — Execução

Executa normalmente seguindo ciclo SDD 5-fase (`sdd-workflow.md`). A decisão já registrada vira o plano (Fase 2 do SDD).

---

## O que NUNCA decidir sozinho (pausa obrigatória mesmo em modo autônomo)

Estas são as únicas categorias de pausa válidas em modo autônomo. Cada uma envolve **blast radius irreversível** ou **violação de contrato de segurança**:

1. **Ações destrutivas em dado real**: `DROP TABLE`, `TRUNCATE`, `DELETE` sem `WHERE`, `rm -rf` fora de `/tmp`, `git push --force` em `main`, revogar/rotacionar credencial em produção.
2. **Credenciais / secrets**: gerar, rotacionar, inserir em `.env`, expor em log, compartilhar em commit ou PR body.
3. **Exfiltração de dados**: posting em canal externo (Slack, GitHub comment, webhook público), upload para serviço de terceiro, impressão em arquivo não-gitignored.
4. **Quebra de compatibilidade** com contrato público já deployado: mudar schema JSON de resposta de endpoint live, alterar assinatura de webhook consumido por terceiro, renomear env var de produção.
5. **Scope creep** radical: task pede X, agente identifica Y fundamental — Y deve ser **registrado** em AUDIT.md ou STATE.md, **não** implementado no escopo desta task sem autorização. Dual-track: task atual prossegue, Y vira item separado.

Fora destas 5 categorias, **o agente decide**. "Qual padrão de retry usar" **não** é pausa — é pesquisa + decisão autônoma. "Qual tipo de cache para session" **não** é pausa. "Renomear variável ambígua" **não** é pausa. "Promover VO a tipo separado" **não** é pausa.

---

## Anti-padrões que quebram o modo autônomo

Comportamentos que o agente **não** pode exibir em modo autônomo:

❌ "Vou aguardar sua confirmação para prosseguir" — pausa inválida (salvo categorias acima)
❌ "Prefere X ou Y?" — escolha é do agente após pesquisa
❌ "Tenho algumas opções para avaliar" — avalie e escolha
❌ "Posso prosseguir com X?" — autorização já foi dada; prossiga
❌ "Antes de continuar, gostaria de confirmar" — sem confirmar, executar
❌ Decidir sem registro em STATE.md — falta de rastro
❌ Pular Context7 por "já conhecer a lib" — treino pode estar desatualizado
❌ Usar dual-agent para decisão de baixo impacto — custo sem retorno
❌ Spawnar subagentes em Sonnet/Haiku para dual-validate — Opus é obrigatório

---

## Comunicação durante execução autônoma

O agente **continua comunicando progresso** (uma linha por fase, momentos-chave), mas não pergunta. Formato esperado:

```
> Discutindo Fase 1.7 (Quotas + Feature Usage). Lendo requirements + design + antipatterns.
> Plano escrito com <verify>: 6 arquivos novos, 2 editados, coverage target 90%.
> Decisão autônoma registrada como D-025: SELECT FOR UPDATE vs advisory lock — escolhi FOR UPDATE (Uber usa em inventory; pgx suporta nativo; testes PBT cobrem).
> Executando: migration + sqlc + entity + repo + use case + handler + testes.
> Verificação: build limpo, vet limpo, 2000 iterações PBT passando, coverage 94.6%.
> Commit `billing: Implement feature usage with SELECT FOR UPDATE (task 1.6)`.
> README atualizado (backend). AUDIT.md checado — zero findings.
> Próxima task: 2.1 (Institutional — Condominium Registry).
```

Tom é **report**, não **consulta**. Frases terminam em ponto, não em ponto de interrogação. Verbo no pretérito (ação concluída) ou gerúndio (ação em curso), não condicional.

---

## Ao acordar / voltar, o que o João vê

Espera-se que o usuário, ao voltar, encontre:

1. **Progresso mensurável** — múltiplas tasks concluídas, commits criados, PRs abertos se infra permite, testes passando.
2. **STATE.md atualizado** — com todas as decisões autônomas tomadas (`D-0XX` entries) + dívidas descobertas durante execução.
3. **AUDIT.md em dia** — se ciclo de sprint virou noite, auditor rodou e findings estão registrados.
4. **Diff explicável** — cada arquivo modificado tem commit que descreve o **porquê**, e o porquê bate com as decisões em STATE.md.
5. **Zero bloqueios silenciosos** — se houve algo que o agente não podia decidir (uma das 5 categorias de pausa), há um bloco no início do relatório final dizendo exatamente qual decisão ficou pendente e por quê.

O teste de qualidade da sessão autônoma: o João consegue reconstruir o raciocínio sem precisar perguntar ao agente. Tudo está no trail de commits + STATE.md + AUDIT.md + release notes.

---

## Relacionamento com outros steering/skills

- `sdd-workflow.md` — o ciclo 5-fase continua inteiro. Modo autônomo não pula fases; ele **substitui** "pausar para confirmar" por "decidir e registrar" dentro de cada fase.
- `mcp-integration.md` — Context7 é obrigatório na Fase 1 do protocolo de decisão. Postgres MCP pode ser usado para `explain_plan` em decisões de query.
- `.claude/skills/dual-validate.md` — skill concreta do pattern dual-agent.
- `.claude/skills/task-executor.md` — usa este steering quando em modo autônomo; Gate 0 (AUDIT.md) continua valendo.
- `.claude/skills/sprint-auditor.md` — mesmo protocolo: decisão sobre severidade de finding usa as 5 fontes.
- `STATE.md` — canal oficial de registro de decisão autônoma.
