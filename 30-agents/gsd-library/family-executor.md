---
title: Família — Executor (GSD)
type: note
tags:
  - agents
  - gsd
  - executor
  - family
created: '2026-04-24T00:00:00.000Z'
updated: '2026-04-24T00:00:00.000Z'
aliases:
  - Executor agents
  - family-executor
---

# Família GSD — Executor

Agrupa os agents que **tocam código**: aplicam plans, implementam fixes derivados de review, produzem o artefato de review que alimenta o ciclo. Diferente de Planning (decide antes) e Audit (julga depois), Executor opera no meio — transforma contrato escrito em commits reais, arquivo por arquivo, task por task, finding por finding.

A invariante da família é **commit atômico por unidade**: cada task vira um commit; cada finding de review vira um commit; cada wave vira uma fronteira de paralelismo. Isso preserva rollback pontual (`git checkout -- {file}`), reconstitui histórico de decisões e evita o anti-padrão "big-bang commit" que destrói causalidade.

## 4 agents na família

- [[card-gsd-executor]] — executa PLAN.md aplicando tasks atomicamente, regras de desvio hierárquicas e checkpoints.
- [[card-gsd-code-reviewer]] — revisa arquivos alterados com postura adversarial e produz REVIEW.md estruturado (read-only no código).
- [[card-gsd-code-fixer]] — aplica fixes de REVIEW.md um a um com verificação 3-tier e rollback atômico.
- [[card-gsd-integration-checker]] — verifica cross-fase que conexões existem de fato (existência ≠ integração), traçando fluxos E2E.

## Quando usar esta família

1. **Após PLAN.md aprovado pelo plan-checker**: `executor` entra em loop (task → apply → verify → commit) até toda a wave fechar. Aplicar em ordem topológica do grafo de dependências.
2. **REVIEW.md com findings BLOCKER/WARNING pendentes**: `code-fixer` entra em loop (finding → fix → verify 3-tier → commit) até todos os BLOCKERs fecharem. WARNINGs podem ser skipados se justificados em STATE.
3. **Fim de fase / milestone**: `code-reviewer` corre antes do AUDIT para catar regressões óbvias e produzir REVIEW.md que alimenta o próximo fixer loop. Rodar ANTES de abrir PR, não depois.
4. **Fim de épico multi-fase**: `integration-checker` verifica que as conexões cross-fase (ex: fase 2 expôs API que fase 5 consome) estão realmente wired, não só declaradas. Findings viram AUDIT entries.
5. **Hotfix de produção**: `code-fixer` em modo single-finding (um bug, um commit, uma verificação) sem precisar de PLAN prévio — a regra de atomicidade continua valendo.

## Quando NÃO usar

- **Refactor exploratório sem plano**: executor exige PLAN.md como input. Se não há plano, passe antes pela família Planning; pular etapa aqui produz commits não-atômicos.
- **Mudanças em docs apenas**: família Docs cobre; `doc-writer` tem modos `create/update/supplement/fix` que preservam prosa humana. Executor trata código, não narrativa.
- **Debugging ativo (hipóteses abertas)**: família Debug com `debug-session-manager`. O executor não investiga — ele aplica fix conhecido. Misturar investigação + aplicação quebra atomicidade.
- **Review de arquitetura / ADR**: família Audit (security-auditor, verifier). Code-reviewer é file-level, não architecture-level.
- **Integração que exige dados live (staging, prod)**: integration-checker é estático (grep + trace), não roda sistema. Pra integração dinâmica use testes E2E rodados na CI/CD.

## Interação entre agents da família (pipeline interno)

```
  PLAN.md (aprovado) ──► executor ──► commits atômicos (wave por wave)
                                 │
                                 ▼
  (fim de fase) ─────────► code-reviewer ──► REVIEW.md (findings BLOCKER/WARNING/INFO)
                                 │
                                 ▼
                            code-fixer ──► commits de fix (um por finding)
                                 │
                                 ▼
                          code-reviewer (re-run) ──► REVIEW-2.md (deve ter 0 BLOCKERs)

  (fim de épico) ─────────► integration-checker ──► INTEGRATION.md (gaps cross-fase)
                                 │
                                 ▼
                          planner --gaps (família Planning) ──► PLAN-FIX.md
```

Note a reentrância: `code-reviewer` roda antes E depois de `code-fixer`. O antes gera o backlog; o depois garante que fixes não introduziram regressão. Sem esse loop, fixer pode silenciosamente quebrar outra parte do código.

## Padrões reutilizáveis

- **Atomicidade é contrato**: um task = um commit (executor); um finding = um commit (fixer); um REVIEW.md por scope (reviewer). Nunca "git add -A" — cada arquivo é adicionado por nome, evitando commits que vazam binários, segredos ou arquivos de outras tarefas. Alinha com regra do CLAUDE.md do vault ("prefer adding specific files by name").
- **Rollback seguro é `git checkout -- {file}`**: o commit ainda não aconteceu até a verificação passar, então o checkout reverte só a mudança não-commitada. `Write` para desfazer é proibido — escrita parcial em falha corrompe o arquivo.
- **Regras de desvio hierárquicas**: bugs/security/blockers se resolvem sozinhos durante execução (Rules 1-3); mudanças arquiteturais sempre pausam em checkpoint (Rule 4). Estrutura evita tanto o Worker paralisado pedindo permissão para respirar, quanto o Worker que refatora a arquitetura por conta própria.
- **Verificação 3-tier antes de commit**: (1) sintática (compile/parse); (2) comportamental (teste que falhava passa); (3) contextual (nenhum outro teste regrediu). Os 3 niveis têm custo crescente; falhar cedo no mais barato evita rodar o mais caro.
- **Existência ≠ integração**: o integration-checker formaliza que "arquivo existe" e "função está exportada" são afirmações verdadeiras e inúteis — o que importa é a chamada ocorrendo no ponto certo do fluxo. Mesmo princípio do *data-flow analysis* em compiladores.
- **Stance adversarial no review**: reviewer assume que o código contém defeitos até grep/trace provar o contrário; severidade é obrigatória em todo finding (BLOCKER/WARNING/INFO) — sem severidade = output inválido.

## Correspondência com nosso pipeline

Esta família é o **Worker** do `automation-agents` em diferentes modos operacionais. O `gsd-executor` é o contrato de referência para o Worker em modo "aplicar plano" — nosso Worker deve seguir o mesmo padrão de commit atômico por task e as regras de desvio hierárquicas. O `gsd-code-fixer` é o Worker em modo corretivo (aplicar findings), e combina naturalmente com nosso protocolo de AUDIT.md: findings 🔴 viram fixes atômicos com REVIEW-FIX.md equivalente.

O `gsd-code-reviewer` tem papel híbrido: tecnicamente produz artefato de review (cabe em Reviewer), mas como **produz** um arquivo estruturado e seu output é read-only sobre código, funciona como Worker com saída documental — análogo ao nosso Worker rodando um linter/scanner e gerando relatório. O `gsd-integration-checker` é Worker operando em escopo de milestone, não de fase; no `automation-agents` equivale a uma rotina que o Orchestrator dispara ao final de um épico para validar que o conjunto de PRs da milestone forma um sistema funcional, não apenas um conjunto de deliverables isolados.

**Insight operacional**: nosso Worker tende a misturar "aplicar plano" + "aplicar review-fix" no mesmo modo. A separação `executor` ↔ `code-fixer` do GSD sugere que nosso Worker tenha sub-modos explícitos com prompts distintos — o prompt do fixer é mais restrito (não pode desviar da intenção do finding) do que o do executor (pode resolver blockers descobertos durante a aplicação).

## Patterns atemporais extraídos

1. **Atomicidade como unidade de rastreabilidade**: 1 task = 1 commit = 1 mensagem descritiva. Aplicável em qualquer workflow com git — facilita bisect, revert, review. É o oposto do "squash merge de 40 arquivos em 1 commit genérico".
2. **Verificação escalonada (cheap → expensive)**: parse < unit test < integration test < e2e. Falhar na etapa mais barata é o design ideal. Mesma mentalidade do CI pipeline em pirâmide.
3. **Separação reviewer ≠ fixer ≠ re-reviewer com o mesmo role executando**: o mesmo ator humano/LLM pode fazer os 3 papéis desde que a entrada/saída de cada etapa seja um artefato separado (REVIEW.md → fixes → REVIEW-2.md). O artefato vira a memória externa.
4. **Existência ≠ integração**: generalizável pra qualquer auditoria — auditar conexão, não apenas presença. Em segurança, policy existir ≠ policy aplicada; em telemetria, métrica exportada ≠ métrica alertada.

## Links externos

- [Google SRE — Change Management](https://sre.google/sre-book/release-engineering/) — atomicidade + rollback atômico casa com o executor
- [Trunk-based Development](https://trunkbaseddevelopment.com/) — commits pequenos e atômicos são o alicerce do modelo
- [Fowler — Conway's Law and YAGNI](https://martinfowler.com/bliki/Yagni.html) — executor seguindo rules de desvio evita gold-plating
- [Git — Revert strategy](https://git-scm.com/book/en/v2/Git-Basics-Undoing-Things) — `git checkout -- file` é a unwind tool canônica pré-commit
- [Netflix Tech Blog — Safe Refactoring](https://netflixtechblog.com/) — padrão de review → fix → re-review é a base de refactoring em larga escala

## Fonte

- [[../../60-sources/get-shit-done/_toc]]

## Links

- [[_moc]]
- [[pipeline-mapping]]
- [[../../10-knowledge/methodology/gsd-get-shit-done]]
- [[../../10-knowledge/methodology/sdd-workflow]]
