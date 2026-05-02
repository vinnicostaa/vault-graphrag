---
title: Família — Audit (GSD)
type: note
tags:
  - agents
  - gsd
  - audit
  - family
created: '2026-04-24T00:00:00.000Z'
updated: '2026-04-24T00:00:00.000Z'
aliases:
  - Audit agents
  - family-audit
---

# Família GSD — Audit

Agrupa os agents que **julgam depois que o código foi escrito**: verificam se o que foi implementado satisfaz o que foi prometido, dimensão por dimensão. Planning define o contrato; Executor produz commits; Audit é onde a narrativa bate (ou não) com a realidade. É a etapa que separa "task marcada como done" de "goal realmente alcançado".

A invariante da família é **falsificar o SUMMARY**: documentos produzidos pelo Executor são hipótese, não evidência. O verifier traça goal-backward até arquivos reais; o eval-auditor grep por chamadas das libs, não por dependências declaradas; o security-auditor exige evidência `file:line` para cada threat mitigate; o nyquist-auditor só aceita teste *rodando e passando* como gap fechado. "Arquivo existe" é afirmação fraca; "arquivo existe, é substancial, está wired, e o dado flui" é o bar.

## 8 agents na família

- [[card-gsd-verifier]] — verifica (pós-execução) que o goal da fase foi alcançado com 4 níveis de artifact + data-flow trace.
- [[card-gsd-eval-planner]] — desenha estratégia de eval AI (Seções 5-7 do AI-SPEC.md) com rubrics PASS/FAIL específicos.
- [[card-gsd-eval-auditor]] — verifica retroativamente se a eval strategy foi realmente implementada; scores COVERED/PARTIAL/MISSING.
- [[card-gsd-nyquist-auditor]] — preenche gaps de validação criando testes comportamentais reais que podem falhar.
- [[card-gsd-security-auditor]] — verifica mitigações declaradas em `<threat_model>` via grep; produz SECURITY.md.
- [[card-gsd-ui-researcher]] — produz UI-SPEC.md (contrato visual) consumido pelo ui-checker e ui-auditor.
- [[card-gsd-ui-checker]] — valida UI-SPEC.md antes do planning (BLOCK/FLAG/PASS por dimensão).
- [[card-gsd-ui-auditor]] — audita retroativamente o frontend implementado contra UI-SPEC (6 pillars, 1-4 cada).

## Quando usar esta família

1. **Fim de fase, pré-PR**: `verifier` roda contra o SUMMARY.md do Executor para falsificar cada claim. Output VERIFICATION.md vira gate — se tem gap, não abre PR.
2. **Fim de fase envolvendo LLM/AI**: `eval-auditor` verifica que a estratégia de eval desenhada pelo `eval-planner` foi implementada (libs chamadas, não só instaladas; dimensões realmente cobertas).
3. **Gaps de teste após verifier flag "insufficient coverage"**: `nyquist-auditor` entra e escreve testes comportamentais reais — é a única família que **pode** gerar código (de teste), mas nunca código de produção.
4. **Milestone envolvendo API pública, auth, PII ou pagamento**: `security-auditor` exige evidência grepável para cada mitigação declarada no threat model. Findings viram AUDIT entries com severidade.
5. **Feature com componente UI**: loop `ui-researcher` (pré-plan) → `ui-checker` (pré-impl) → Executor → `ui-auditor` (pós-impl). Contrato visual existe antes do primeiro commit de CSS.

## Quando NÃO usar

- **Code review superficial (typos, estilo, formatação)**: família Executor com `code-reviewer`. Audit é goal-level, não linha-por-linha.
- **Debug ativo de bug**: família Debug. Auditor não investiga causa; ele verifica que *o que estava prometido* foi entregue.
- **Verificação runtime (perf, load, latência)**: auditores GSD são estáticos (grep, read, parse). Perf exige benchmarking real e testes de carga.
- **Docs vs código drift**: família Docs tem `doc-verifier` específico; redundante chamar security-auditor pra isso.
- **Sem contrato a priori (SPEC, threat model, UI-SPEC)**: auditor precisa de ground truth pra comparar. Se não tem contrato, o output é opinião — use Executor/reviewer instead.

## Interação entre agents da família (pipeline interno)

```
  (pre-planning)
  ui-researcher ──► UI-SPEC.md ──► ui-checker ──► VERDICT (BLOCK/FLAG/PASS)
                                           │
                                           ▼ (PASS)
  eval-planner ──► AI-SPEC §5-7       [Planner + Executor tomam conta]
                                           │
                                           ▼
  (post-execution)                     SUMMARY.md
                                           │
            ┌──────────────┬──────────────┼──────────────┬──────────────┐
            ▼              ▼              ▼              ▼              ▼
       verifier      eval-auditor   security-auditor   ui-auditor   nyquist-auditor
    (goal-backward)  (dim rubrics)   (threat evidence) (6 pillars)  (gap → test)
            │              │              │              │              │
            └──────────────┴──────┬───────┴──────────────┴──────────────┘
                                  ▼
                         AUDIT.md consolidado ──► planner --gaps (replan)
```

Ordem importa: `verifier` roda primeiro (gate mais amplo); os auditores dimensionais rodam em paralelo depois. `nyquist-auditor` só entra se algum auditor anterior flagou gap de teste — não é etapa default, é resposta a falha.

## Padrões reutilizáveis

- **Postura adversarial como default**: todo auditor parte de "a coisa falhou até eu provar o contrário" — verifier falsifica SUMMARY, eval-auditor assume que dimensões estão MISSING, ui-auditor assume que cada pilar tem falhas. Isso evita o modo soft em que PARTIAL/WARNING são usados para não parecer duro.
- **Evidência concreta vence reivindicação**: claim sem `file:line` é claim inválido. Mitigação precisa de grep no arquivo certo; dimensão de eval precisa de lib *chamada*, não instalada; pillar UI precisa de contagem de classes Tailwind, não "acho que ficou bom". Análogo direto a *audit trail* contábil (SOX) e *evidence of compliance* em PCI-DSS.
- **Severidade obrigatória e honesta**: todo finding carrega BLOCKER/WARNING classificação. Downgradar BLOCKER para WARNING pra "não conflitar com o implementador" é falha crítica documentada nos raws — é literalmente um failure mode nomeado em cada agent.
- **Contratos antes de implementação (ui-researcher, eval-planner)**: duas famílias têm um planner embutido que cria o *contrato* (UI-SPEC, AI-SPEC Seções 5-7) que depois será auditado. Isso fecha o loop: contrato escrito → código implementado → auditor compara implementação contra contrato — sem ambiguidade sobre "o que era pra ser".
- **Read-only sobre código, write-only sobre artefatos**: todo auditor é proibido de modificar implementação. Fix é responsabilidade de outro agent (code-fixer, planner com `--gaps`, etc.). Isso mantém a separação auditor ≠ corretor — o mesmo padrão que segregação de função em auditoria contábil (*principle of separation of duties*).
- **Falsificar, não verificar**: verifier e auditores seguem Popper — tentar provar que a claim é falsa é mais produtivo que tentar confirmar. Confirmation bias em auditor = auditor inútil.

## Correspondência com nosso pipeline

Esta família é o **Reviewer** do `automation-agents` no estado puro — 6 dos 8 agents são literalmente Reviewers em dimensões diferentes (correctness, security, UI, eval, teste-coverage). O `gsd-verifier` é o contrato de referência para o Reviewer padrão em `automation-agents` — nosso Reviewer deve seguir o mesmo padrão de verificação em 4 níveis + data-flow trace, e o mesmo output estruturado (VERIFICATION.md com gaps YAML para replanning).

Dois agents têm papel híbrido: `gsd-eval-planner` tem natureza de **Planner** (cria contrato pré-implementação), e `gsd-ui-researcher` é **Support** (produz UI-SPEC como contrato consumido por múltiplos agents — cabe em `30-agents/` como agent de suporte à fase UI). A divisão entre `ui-researcher` (cria o contrato) → `ui-checker` (valida o contrato antes do planning) → `ui-auditor` (audita o implementado contra o contrato) é um pattern que o `automation-agents` pode replicar em qualquer domínio de alta ambiguidade: pré-contrato explícito impede discussão estética virar debate inútil durante PR review, porque o contrato já foi aprovado upstream.

A família também sugere que nosso Reviewer padrão deveria ter modos de operação (security / UI / eval / integration), cada um com playbook próprio em `30-agents/playbooks/` — em vez de um Reviewer monolítico tentando cobrir tudo.

**Insight operacional**: o Dual-check do CLAUDE.md §5 é conceitualmente equivalente ao verifier adversarial desta família. A diferença é que o GSD estrutura dual-check por dimensão (security, eval, UI) em vez de agente genérico — escalar isso no `automation-agents` significaria ter playbooks de Dual Reviewer por dimensão, não um Reviewer único invocado com contexto limpo.

## Patterns atemporais extraídos

1. **Stance adversarial como default do auditor**: aplicável em qualquer revisão — segurança, compliance, code review, peer review acadêmico. Revisor que começa presumindo mérito vira rubber-stamp; revisor que começa presumindo falha encontra problemas reais.
2. **Evidência grepável > reivindicação**: pattern replicável em qualquer domínio de auditoria. Compliance que não aponta `file:line` (ou seu equivalente — contrato, log, config) é compliance-teatro.
3. **Contrato explícito pré-execução**: UI-SPEC, AI-SPEC, threat model — todos são contratos travados antes do código. O pattern evita que "subjetividade" vire desculpa; o que está fora do contrato é bug, não gosto.
4. **Segregation of duties auditor ↔ corretor**: auditor que conserta = conflito de interesse. O pattern é canônico em auditoria financeira (ISAE 3000) e transfere direto para LLM agents — o mesmo agent fixar um finding que ele mesmo flagou cria viés.

## Links externos

- [OWASP ASVS (Application Security Verification Standard)](https://owasp.org/www-project-application-security-verification-standard/) — rubrics de security-auditor espelham ASVS levels
- [IEEE 1012 — Software Verification and Validation](https://standards.ieee.org/ieee/1012/6427/) — 4 níveis de artifact do verifier segue o padrão
- [Karl Popper — Conjectures and Refutations](https://en.wikipedia.org/wiki/Conjectures_and_Refutations) — base filosófica do "falsificar, não verificar"
- [ISAE 3000 — Assurance Engagements Other Than Audits of Historical Financial Information](https://www.iaasb.org/publications/isae-3000-revised-assurance-engagements-other-audits-or-reviews-historical-financial) — segregation of duties referência canônica
- [Google — Eval-Driven Development for LLM Apps](https://research.google/pubs/) — eval-planner + eval-auditor espelham a metodologia

## Fonte

- [[../../60-sources/get-shit-done/_toc]]

## Links

- [[_moc]]
- [[pipeline-mapping]]
- [[../../10-knowledge/methodology/gsd-get-shit-done]]
- [[../../10-knowledge/methodology/sdd-workflow]]
