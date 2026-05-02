---
title: SDD — Gaps típicos na adoção (catálogo G-###)
type: antipattern
tags: [methodology, sdd, gsd, gaps, adoption, antipattern]
created: 2026-04-22
updated: 2026-04-24
aliases: [sdd-gaps, SDD adoption gaps, gaps SDD]
---

# SDD — Gaps típicos na adoção

Catálogo atemporal de **gaps comuns** que qualquer projeto adotando Spec-Driven Development (SDD) tende a acumular, independente da escola ([[kiro-vs-spec-kit|Kiro, GitHub Spec Kit ou Tessl]]). Use como checklist de antipattern em auditoria ou dual-check de adoção metodológica.

> **Contrato do catálogo**: cada gap tem identificador estável `G-###`, sintoma observável, impacto, fix de baixo/médio custo. Gaps **não** bloqueiam operação — são polish. Mas todos se pagam em clareza cognitiva.

---

## G-001 — Constitution implícita em vez de explícita

**Sintoma**
- Princípios invioláveis do projeto vivem **distribuídos** em N arquivos de steering (ou em `CLAUDE.md`, ou na cabeça do tech lead)
- Dev novo precisa ler 10+ arquivos pra saber "o que **não** posso mudar sem autorização"
- Revisor pergunta "isso aqui é regra dura ou preferência?" em PR

**Impacto**
- Alto custo cognitivo de onboarding
- Risco de violar invariante sem perceber (dev novo)
- Revisor improvisa critério em review, inconsistência entre revisores

**Fix (baixo custo)**
- Criar `CONSTITUTION.md` (Spec Kit style) ou `do-dont.md` explícito com:
  - Padrões de engenharia inegociáveis (DDD, TDD, Clean Arch, CQRS, SOLID, stack-specific calisthenics)
  - Categorias destrutivas que **pausam** modo autônomo (ex: mudança de schema em prod, exposição de PII)
  - Hierarquia de fontes de verdade (ex: spec > código > discussão Slack)
  - 10 Regras de Ouro do produto (se existirem)
- Pode ser renomeação de steering existente, não precisa criar do zero

**Quando ignorar**
- Projeto < 3 pessoas, todos no mesmo contexto há > 6 meses

---

## G-002 — Workflow manual em vez de slash/skill automatizado

**Sintoma**
- As 4–5 fases do ciclo (Discuss/Plan/Execute/Verify/Ship) são operadas **manualmente** ou guiadas por prompt a cada nova task
- Não existem comandos repetíveis `/specify`, `/plan`, `/tasks` (Spec Kit style) ou skill `/task-executor` (Kiro style)
- Cada agente/dev improvisa a ergonomia do ciclo

**Impacto**
- Modo autônomo de longa duração (10h+) acumula desvio de processo
- Dual-check inconsistente (cada revisor pede coisa diferente)
- Onboarding de agente novo demora — não tem playbook executável

**Fix (médio custo)**
- **Spec Kit**: 9 slash commands (6 core + 3 optional) já vêm prontos — ver [[kiro-vs-spec-kit#9-slash-commands-spec-kit-2026-6-core-3-optional]]
- **Kiro-style**: criar skills `/task-executor`, `/sprint-auditor`, `/dual-validate`, `/go-review` (ou equivalentes do stack) com checkpoints explícitos
- Skill deve ter **gate de aprovação** configurável — automação não é anti-human-review

**Quando ignorar**
- Modo autônomo < 2h tipicamente; cada sessão é supervisionada por humano

---

## G-003 — Terminologia ambígua ("GSD", "spec", "plan")

**Sintoma**
- Time cita "GSD" internamente significando **uma** coisa (ex: "Get Shit Done", methodology de velocidade) enquanto a indústria externa usa "GSD" pra **outra** (ex: "GitHub Spec-Driven Development" do Spec Kit)
- Mesma palavra aparece em steering files e em literatura externa com sentidos diferentes
- Dev novo pesquisa "GSD" no Google e encontra literatura que não bate com o projeto

**Impacto**
- Ruído cognitivo quando time lê blog posts / papers / conference talks
- Risco de importar prática errada ("o GSD deles faz X, vamos adotar") quando é sigla diferente
- Dual-check externo (convidado outside) confunde termos

**Fix (2 min)**
- Adicionar **uma linha** no arquivo que introduz a sigla, explicitando:
  > _"Não confundir com **GitHub Spec-Driven Development** do [GitHub Spec Kit](https://github.com/github/spec-kit) — projeto adota a mesma filosofia ('spec é fonte de verdade') mas com taxonomia Kiro / XYZ."_
- Se possível, usar sigla nova não-colidente ("Ship-GSD", "Project-SDD", etc)

**Quando ignorar**
- Nunca. É fix barato com alto retorno cognitivo.

---

## G-004 — `<verify>` / acceptance não-parseáveis por máquina

**Sintoma**
- Critério de pronto é **prosa em Markdown** dentro de bloco `<verify>` XML-like ou lista de checkboxes livres
- Humano e agente LLM leem OK, mas CI não consegue consumir automaticamente
- Não existe gate "PR merge bloqueado se task.md não tem `<verify>` válido"

**Impacto**
- Tasks merge-áveis com acceptance ambíguo
- Automação de coverage/gate de qualidade fica parcial
- Retrabalho de "eu achei que tinha entregue, mas sem verify o revisor diz que não"

**Fix (médio custo, só se vale a pena)**
- **Opção A**: migrar `<verify>` pra YAML ou checkbox com anchors estáveis (machine-parseable)
- **Opção B**: manter Markdown mas adicionar script validador em CI (`make verify-tasks`) que parse blocos `<verify>` e falha se malformado
- **Opção C (mais usada)**: deixar como está — LLM consome Markdown tão bem quanto YAML; automação CI-total raramente paga o esforço

**Quando priorizar**
- Só se automação de CI-gate virou requisito de compliance ou escala (> 20 tasks paralelas)

---

## G-005 — Reference layer ausente (só spec + código)

**Sintoma**
- Projeto tem specs formais e código, mas **nada** destila lições do legado
- Antipatterns observados em produção não estão catalogados
- Reimplementação repete erros que o legacy já resolveu (mas ninguém lembra que resolveu)

**Impacto**
- Regras de negócio validadas em prod se perdem na reescrita
- Bugs antigos ressurgem no sistema novo
- Onboarding de dev novo não aprende "o que **não** fazer aqui"

**Fix (médio custo, alto ROI)**
- Criar camada `reference/` ou `10-knowledge/antipatterns/` com:
  - Catálogo AP-### ou equivalente (ver [[../antipatterns/_moc]])
  - Sumário do legado: regras validadas + decisões arquiteturais ruins a **não** replicar
  - Mapa "onde o legacy documenta regra X" (pra consultar sem ler tudo)
- Ver padrão em [[legacy-as-reference]]

**Quando ignorar**
- Projeto greenfield sem legado nenhum

---

## G-006 — STATE/AUDIT que só o tech lead mantém

**Sintoma**
- Arquivos STATE.md/AUDIT.md existem mas são atualizados só em onda (fim de sprint, retrospectiva)
- D-### e A-### ficam abertos indefinidamente sem SLA
- Dev júnior não sabe onde registrar decisão pequena — "isso é D-### ou commit message?"

**Impacto**
- Decisão importante vive só em PR description (perde-se)
- Dívida técnica acumula invisível
- Gate 0 de sessão (bloquear task se AUDIT 🔴) não funciona porque ninguém sabe ler AUDIT

**Fix (baixo custo, alto ROI)**
- Template explícito de D-### e A-### em steering (formato, quando criar, quando fechar)
- Skill `/sprint-auditor` roda no fim de sprint, força atualização
- SLA: A-023 🔴 aberto > 24h bloqueia merge em main (gate automatizado)

**Quando ignorar**
- Projeto < 5 pessoas, cadência diária, tudo conversado em standup

---

## G-007 — Workflow mismatch: spec com granularidade errada pro tamanho do problema

**Fonte**: [Martin Fowler — *Understanding SDD* (15 out 2025)](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) — Fowler usa a analogia *"sledgehammer to crack a nut"* para o caso de um bug fix expandido em múltiplas user stories + dezenas de acceptance criteria.

**Sintoma**
- Bug pequeno (3 linhas) entra no mesmo ciclo SDD de feature grande; acaba com spec-of-spec de 200 linhas pra um fix trivial.
- Feature média gera doc tão extenso que ninguém lê inteiro.
- Ciclo single-size; não há "spec leve vs spec pesado" pra magnitudes diferentes.

**Impacto**
- Peso cerimonial mata velocidade em micro-tasks.
- Developers passam a **pular SDD** em bugs pequenos, cavando dívida de spec.
- Feature grande continua sendo tratada como se fosse atômica, falhando em quebrar adequadamente.

**Fix proposto pelo vault** (não prescrito por Fowler — ele identifica o problema sem prescrever solução; esta é uma proposta atemporal derivada)
- Definir **3 templates** por ordem de grandeza: `bugfix.md` (< 50 linhas), `feature.md` (1-2 sprints), `epic.md` (M1/M2/M3).
- Explicitar quais seções são **obrigatórias** por template (ex: bug não precisa de *constitution check*; epic precisa).
- Gate opcional: task entre 50-500 linhas pode escolher; < 50 usa `bugfix`; > 500 obriga `epic`.

**Quando ignorar**
- Time de 1 pessoa, magnitudes homogêneas.

---

## G-008 — Review burden: verbosidade de artefatos MD excede o esforço de revisar código direto

**Fonte**: [Martin Fowler — *Understanding SDD* (15 out 2025)](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) — Fowler relata explicitamente *"Spec-kit created a LOT of markdown files for me to review. They were repetitive... tedious to review"* e *"I'd rather review code than all these markdown files"*.

**Sintoma**
- PR chega com pouco código e muita spec/plan/tasks/research (proporção invertida, onde o doc excede o código).
- Reviewer desiste de ler tudo; aprova baseado em heurística superficial ("LGTM, spec cobre").
- Agente LLM gera spec verbose demais por padrão (completude > clareza).

> Exemplo ilustrativo (não é número citado pela fonte): *"200 linhas de código acompanhadas de 800 linhas de artefatos MD".*

**Impacto**
- Review vira rubber-stamp.
- Custo real de feature pequena dispara por causa de burocracia de spec.
- Team performance cai no quadrante "muita cerimônia, pouco sinal".

**Fix proposto pelo vault** (não prescrito por Fowler; proposta derivada)
- **Budget de palavras** por artefato: spec.md ≤ 300 linhas; plan.md ≤ 500 linhas; tasks.md ≤ 150 linhas. Over-budget requer justificativa.
- **Sumários executivos** no topo de cada artefato (≤ 10 linhas).
- Skill `/sdd-lint` que avisa "você está escrevendo spec pra bug trivial — use template `bugfix`" (ver G-007).
- Review deve começar por **spec resumida** + **diff do código**, não spec completa.

**Quando ignorar**
- Projeto onde compliance exige doc extensa (regulado — saúde, finanças, aviônica). Nesse caso, verbosity é feature, não bug.

---

## Exemplos de aplicação

- **Projeto regulado multi-tenant com legado descontinuado**: G-001 e G-003 aparecem frequentemente juntos (steering distribuído em 10+ arquivos + sigla "GSD" ambígua com GitHub Spec-Driven). G-007 e G-008 emergem quando o time escala o uso de SDD sem calibrar tamanho de spec. Aplicações concretas vivem em `50-projects/<slug>/06-execution/sdd-gsd-aplicado.md`.

---

## Links no grafo

- [[_moc]]
- [[kiro-vs-spec-kit]] — as 3 escolas
- [[spec-driven-development]] — filosofia
- [[sdd-workflow]] — ciclo 5-fase
- [[sdd-maturity-checklist]] — cobertura SDD (positivo do catálogo)
- [[legacy-as-reference]] — camada Reference que evita G-005
- [[../antipatterns/_moc]] — catálogo AP-### de antipatterns de código
- [[../../00-meta/STATE]] — D-vault-002 registra esta adição de G-007/G-008
