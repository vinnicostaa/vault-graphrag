# Skill: Dual-Agent Validation (Opus + Opus)

Skill para decisões técnicas de **alto impacto** em modo autônomo. Implementa o pattern que o João definiu: um subagente **Analista** pesquisa e propõe; um subagente **Revisor** pesquisa **independentemente** e julga se a proposta bate — sem tratar a análise do primeiro como verdade absoluta.

**Ambos os subagentes rodam em Opus.** Sonnet e Haiku não têm o fôlego para a tarefa de validação cruzada com 5 fontes.

Ver `.kiro/steering/autonomous-execution.md` para o contexto completo de quando ativar modo autônomo e o protocolo de decisão em 4 fases.

---

## Quando invocar

### Usar dual-agent (alto impacto):

- Escolha arquitetural que afeta >1 bounded context
- Escolha de padrão de segurança (auth flow, crypto, webhook validation, rate limit strategy)
- Schema de banco novo (nova tabela com relacionamentos, enum novo, migration irreversível)
- Contrato público novo ou alteração de contrato existente (endpoint, webhook payload, event schema)
- Integração com SDK externo (Stripe, Zitadel, Mux, Twilio) — decisão de como modelar a interface
- Interpretação ambígua de requisito — duas leituras possíveis do spec

### NÃO usar dual-agent (baixo/médio impacto — agente decide sozinho):

- Nome de variável / renomear campo
- Ordem de argumentos em função interna
- Escolha entre duas implementações equivalentes do mesmo padrão já usado no projeto
- Refactor dentro de um arquivo
- Severidade low/medium de finding em `AUDIT.md`
- Correção óbvia de bug com fix único evidente

Dual-agent custa tempo de pesquisa e contexto. Usar onde o custo de errar uma decisão é maior que o custo de validá-la.

---

## Protocolo de execução

### Passo 1 — Orquestrador (main agent) prepara o briefing

Antes de spawnar os subagentes, o orquestrador escreve **um único briefing compartilhado** com:

1. **Problema**: o que precisa ser decidido, em uma frase sem jargão do projeto.
2. **Contexto**: trechos específicos de `requirements/` + `design/` + código existente. Arquivo:linha quando relevante.
3. **Restrições duras do projeto**: regras que não podem ser violadas (dependency rule, pt-BR em comentários, `int64` para dinheiro, tenant isolation, etc).
4. **Perguntas explícitas**: lista numerada — o que cada subagente deve responder. Tipicamente 2-4 perguntas. Ex:
   - "P1: Qual padrão de retry é apropriado aqui?"
   - "P2: Qual lib Go é estado-da-arte e compatível com pgx v5?"
   - "P3: Como Uber/Stripe/Netflix resolvem isso em produção?"
   - "P4: Qual dos padrões consultados melhor se encaixa no ciclo SDD 5-fase deste projeto?"
5. **O que NÃO é escopo**: evita que subagente gaste esforço em decisões vizinhas.

O briefing é **idêntico** para os dois. Qualquer diferença entre briefings vicia o experimento.

### Passo 2 — Spawnar Analista (Opus)

```
Agent({
  description: "Analista dual-validate — {tópico}",
  subagent_type: "general-purpose",
  model: "opus",
  prompt: `
    Você é o ANALISTA de uma validação dual-agente. Seu par é um REVISOR independente
    que receberá o mesmo briefing e não verá sua resposta.

    Missão: produzir uma RECOMENDAÇÃO fundamentada para as perguntas abaixo.

    Fontes OBRIGATÓRIAS (todas, nenhuma opcional):
    1. Docs oficiais via Context7 MCP: resolve-library-id → get-library-docs(topic=...)
    2. Source do package em $GOMODCACHE (go env GOMODCACHE) — ler arquivos relevantes
    3. Práticas de grandes empresas (Google, Uber, Stripe, Netflix, Shopify, GitHub) via
       WebSearch/WebFetch em engineering blogs e papers
    4. Boas práticas genéricas: SOLID, DDD, Clean Architecture, OWASP, 12-Factor, SRE
    5. Contexto do projeto: ler .kiro/specs/, .kiro/steering/, código existente

    BRIEFING:
    {briefing escrito no Passo 1}

    FORMATO DE SAÍDA (obrigatório):
    ## Recomendação
    Resposta direta para cada pergunta em 1-2 parágrafos.

    ## Fundamento por fonte
    - Docs oficiais: {versão consultada + citação específica}
    - Source package: {arquivo + linha + observação}
    - Grandes empresas: {quem faz o quê + link se disponível}
    - Boas práticas: {qual princípio + por que aplica aqui}
    - Contexto do projeto: {qual padrão existente + por que bate}

    ## Alternativas consideradas e rejeitadas
    {A, B, C — por que cada uma foi descartada}

    ## Riscos da recomendação
    {o que pode dar errado + como mitigar}

    ## Confiança: {baixa / média / alta}
    {uma frase justificando o nível}

    Regras duras:
    - NUNCA apresente recomendação sem ter consultado as 5 fontes.
    - Se uma fonte não é aplicável, diga explicitamente por quê (ex: "source package
      não se aplica — decisão é puramente arquitetural sem lib externa envolvida").
    - Se há conflito entre fontes, EXPLICITE o conflito em vez de esconder.
    - NÃO responda em menos de 800 palavras. Decisão de alto impacto exige fundamento.
    - NÃO cite memória de treino sem verificar — "Eu sei que Go usa X" não é fonte válida.
  `
})
```

### Passo 3 — Spawnar Revisor (Opus) em **paralelo** com o Analista

Mesmo briefing, prompt diferente:

```
Agent({
  description: "Revisor dual-validate — {tópico}",
  subagent_type: "general-purpose",
  model: "opus",
  prompt: `
    Você é o REVISOR de uma validação dual-agente. Há um ANALISTA paralelo que recebeu
    o mesmo briefing e produzirá uma recomendação separadamente. VOCÊ NÃO VÊ A RESPOSTA
    DELE. Isso é deliberado: sua análise deve ser INDEPENDENTE.

    Missão: responder ao briefing do zero, como se fosse o único agente na missão.
    NÃO tente adivinhar o que o Analista vai dizer e concordar antecipadamente; NÃO
    tente ser contrário por esporte. Pesquise honestamente e chegue à SUA conclusão.

    [mesmo conjunto de 5 fontes obrigatórias do Analista]

    BRIEFING:
    {mesmo briefing do Analista}

    FORMATO DE SAÍDA (obrigatório, idêntico ao do Analista):
    [mesmo formato]

    Regra dura extra do Revisor:
    - Além das 5 fontes, você DEVE listar as hipóteses que descartou e POR QUÊ.
    - Quando divergir de uma prática amplamente adotada (Google/Uber/Stripe), explicite.
    - Sua confiança declarada final importa mais que a do Analista no tiebreak — o
      Orquestrador pondera por confiança quando houver divergência.
  `
})
```

**Paralelo, não sequencial.** Ambos spawns em um único `message` do orquestrador. Se spawnados em sequência, o segundo contamina o contexto do primeiro.

### Passo 4 — Orquestrador lê ambos os relatórios

Leitura lado a lado, comparando:

1. **Recomendação convergente** → executar imediatamente. Registrar em `STATE.md`:
   ```
   ### D-0XX — {tópico}
   - Decisão autônoma dual-validated em {data}
   - Ambos subagentes (Analista + Revisor, Opus) convergiram em: {escolha}
   - Confiança Analista: {nível} | Confiança Revisor: {nível}
   - Fontes consultadas: {resumo da união}
   ```

2. **Recomendação divergente** → orquestrador **não escolhe o maior nível de confiança às cegas**. Em vez disso:
   - Identifica o **ponto específico** da divergência (tipicamente é 1 sub-pergunta de 4, não tudo).
   - Para esse ponto, faz 1 pesquisa adicional focada (pode ser 3ª chamada Context7 ou 1 leitura de source).
   - Decide com base nessa pesquisa de desempate + contexto do projeto como tiebreak final (consistência > modernidade).
   - Registra em `STATE.md` **ambos os pontos de vista** + o desempate:
   ```
   ### D-0XX — {tópico}
   - Divergência dual-validate em {data}
   - Analista recomendou: {X} (confiança {n})
   - Revisor recomendou: {Y} (confiança {m})
   - Ponto específico da divergência: {descrição}
   - Pesquisa de desempate: {fonte + achado}
   - Decisão final: {X ou Y ou Z híbrido} — por quê
   ```

3. **Ambos com baixa confiança** → pausar e registrar em AUDIT.md como `spec-gap` severidade high. Não inventar decisão sob baixa confiança; isso viola o protocolo autônomo.

### Passo 5 — Executar

A decisão é a nova fonte para a Fase 2 (Plan) do ciclo SDD. Plano cita `D-0XX` em STATE.md como justificativa. Prossegue com Execute → Verify → Ship normalmente.

---

## Budget e quando abortar

- Cada subagente Opus consome contexto e tempo. Orçamento típico: **10-20 min de wall clock** por decisão dual-validate.
- Se o orquestrador identifica que a decisão é menor do que pensava (ex: Analista volta dizendo "isso é trivialmente X, padrão estabelecido"): cancelar o Revisor se ainda não retornou e tomar a decisão com base só no Analista + registro simplificado em STATE.md. Poupar Opus não é preguiça — é eficiência.
- Se o orquestrador spawna 3+ dual-validates no mesmo sprint, há cheiro de **spec insuficiente**. Registrar em AUDIT.md e incluir melhoria de spec no escopo.

---

## Regras duras

- ✅ **Ambos subagentes em Opus.** Explícito no parâmetro `model: "opus"`. Sonnet/Haiku não têm fôlego para esta tarefa.
- ✅ **Ambos em paralelo** — um único message do orquestrador com dois Agent calls.
- ✅ **Briefing idêntico** — qualquer assimetria vicia o experimento.
- ✅ **Registro em STATE.md** — sempre, mesmo em convergência total.
- ❌ **Não** perguntar ao usuário no meio de um dual-validate — modo autônomo manda seguir.
- ❌ **Não** tratar Analista como "primário" e Revisor como "rubber stamp". Revisor tem autoridade igual.
- ❌ **Não** esconder divergências — elas são o sinal mais valioso do protocolo.
- ❌ **Não** usar dual-validate para decisão trivial. É anti-desperdício.

---

## Exemplo concreto (hipotético)

**Decisão**: como modelar invalidação de cache Redis do ABAC quando o plano do user muda via webhook Stripe.

**Briefing compartilhado**:
```
Problema: Subscription.activated event deve invalidar cache de AuthUser no Redis
para que ABAC enxergue o novo tier imediatamente.

Contexto:
- Redis key: ms:auth:{zitadelUserID}, TTL 5min
- Webhook Stripe é assinado e idempotente
- Cache é populado no middleware Authenticate após introspection

Perguntas:
P1: Invalidar no handler do webhook diretamente, ou via domain event?
P2: Delete simples vs SET com TTL=0 vs SCAN+DEL com padrão?
P3: Como Stripe/Uber/Netflix modelam cache invalidation cross-service?
P4: Qual opção se alinha melhor com Sprint 6 (NATS JetStream)?

Restrições: dependency rule, pt-BR em log, idempotência, nunca log sem user_id.
Não escopo: mudar TTL, trocar Redis.
```

**Analista volta**: "Domain event (`SubscriptionActivated`) com handler em identity que faz DEL. Alinha com Sprint 6 porque quando migrar para NATS, só mudar o publisher. Confiança alta."

**Revisor volta**: "Domain event com DEL. Uber tem artigo sobre `cache invalidation via events`. Confiança alta."

**Convergência total** → executar. Registrar D-0XX citando ambos + source do artigo Uber + código já existente do InMemoryPublisher.

---

## Quando o protocolo falha

O pattern falha quando:

- Ambos subagentes dão **a mesma resposta errada** porque consultaram as mesmas fontes enviesadas (ex: blog post antigo propagado). Mitigação: o orquestrador **sempre** faz sanity check final lendo source do package se a decisão envolve SDK — source é a verdade última.
- **Questão filosófica** sem resposta técnica ("DDD vale a pena aqui?"). Dual-validate resolve problemas técnicos, não religiosos. Para filosóficos: registrar em STATE.md como decisão de projeto do João, não tomar sozinho.
- **Timing** — dual-validate leva minutos. Se a task tem deadline em segundos (hotfix prod), pular dual e decidir sozinho registrando risco.
