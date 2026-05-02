---
title: Família — Docs (GSD)
type: note
tags:
  - agents
  - gsd
  - docs
  - family
created: '2026-04-24T00:00:00.000Z'
updated: '2026-04-24T00:00:00.000Z'
aliases:
  - Docs agents
  - family-docs
---

# Família GSD — Docs

Os 4 agents desta família formam um **pipeline de documentação com separação estrita de concerns**: classificar material existente, sintetizar/detectar conflitos, escrever novo conteúdo ancorado no código real, e verificar adversarialmente o que foi escrito. A família cobre tanto a **ingestão** de docs legadas (ADR/PRD/SPEC/DOC heterogêneos que o usuário já tinha) quanto a **geração** de docs canônicas (README, ARCHITECTURE, API etc.) contra codebase vivo.

A filosofia comum: docs são **artefatos factuais que driftam**. A defesa em profundidade é classificar com precedência explícita (ADR > SPEC > PRD > DOC), preservar conflitos em vez de fundi-los silenciosamente, escrever só o que se pode provar via Read/Grep/Glob, e depois **verificar com stance adversarial** que o que foi escrito realmente casa com o código. Nenhum dos 4 agents prompta o usuário; todos retornam saídas estruturadas que o orchestrator encadeia.

## 4 agents na família

- [[card-gsd-doc-classifier]] — classifica **1 doc** por spawn (paralelo), escreve JSON de classificação com tipo + confidence + scope + cross_refs + locked status.
- [[card-gsd-doc-synthesizer]] — consome N JSONs do classifier, aplica precedência, detecta ciclos, emite `INGEST-CONFLICTS.md` em 3 buckets e `SYNTHESIS.md` como entry point downstream.
- [[card-gsd-doc-verifier]] — verifica **factualidade** de claims (paths, comandos, endpoints, functions, deps) contra filesystem com stance adversarial e skip rules pré-extração.
- [[card-gsd-doc-writer]] — escreve/atualiza docs em 4 modos (`create | update | supplement | fix`) explorando codebase via Read/Grep/Glob, com templates discovery-first e markers `<!-- VERIFY: ... -->`.

## Quando usar esta família

1. **Ingestão de projeto legado**: chegou repo com 50 docs heterogêneos (ADR, PRD, SPEC, notas de reunião, README desatualizado). `doc-classifier` em paralelo (1 por arquivo), depois `doc-synthesizer` consolida — saída vira entry point para Planner derivar requirements.
2. **Geração de README/ARCHITECTURE canônico**: `doc-writer` em modo `create` com discovery-first. O writer grep pelo código real pra popular seções; markers `<!-- VERIFY: ... -->` onde não há evidência direta.
3. **Docs driftou após refactor grande**: `doc-verifier` em modo adversarial contra paths, endpoints, comandos. Output DRIFT.md alimenta `doc-writer` em modo `fix` para atualização cirúrgica.
4. **Atualização incremental preservando prosa humana**: `doc-writer` em modo `supplement` — acrescenta seções novas sem reescrever prosa que o dev cuidou à mão. Evita o anti-padrão "LLM reescreveu meu README inteiro".
5. **Conflito entre ADR e PRD**: `doc-synthesizer` detecta e *preserva* ambos no bucket "Competing Sources" em vez de escolher — humano decide no MR. Nunca merge silencioso entre sources travados.

## Quando NÃO usar

- **Doc que é na verdade log/diário de decisão** (STATE.md, D-###): não é doc factual pra verificar — é registro temporal. Use padrão append do CLAUDE.md do vault.
- **Narrativa de engineering blog / post mortem**: prosa humana com voz autoral. `doc-writer` mode `create` com temperature baixa gera texto chato; escrever manualmente.
- **API spec OpenAPI canônica**: geração via codegen (swag, tsoa, similar) — `doc-writer` é sobre docs narrativas que explicam, não sobre contratos gerados.
- **Classificação de 1 doc isolado**: overhead de spawn + JSON não vale — leia direto. Classifier só ganha em paralelo N ≥ 3.
- **Doc cuja fonte de verdade não é código** (política interna, playbook humano): `doc-verifier` não tem como grep `file:line` — vira fricção sem valor.

## Interação entre agents da família (pipeline interno)

```
  (ingestão de legado)
  [doc1.md, doc2.md, ... docN.md]
         │
         │ (fan-out: 1 classifier por doc, paralelo)
         ▼
  classifier-<hash1>.json  classifier-<hash2>.json  ...  classifier-<hashN>.json
         │
         │ (fan-in: 1 synthesizer consome todos)
         ▼
  SYNTHESIS.md (entry point canônico)
  INGEST-CONFLICTS.md (3 buckets: unresolved | competing | stale)

  (manutenção contínua)
  código muda ──► doc-verifier ──► DRIFT.md ──► doc-writer mode=fix ──► doc atualizado
                                                       │
                                                       │ (re-verify)
                                                       ▼
                                                 doc-verifier (re-run) ──► VERIFIED

  (criação nova)
  requisito documental ──► doc-writer mode=create ──► novo doc com VERIFY markers
                                                       │
                                                       ▼
                                                 doc-verifier ──► markers resolvidos ou flagados
```

Regra chave: `doc-writer` e `doc-verifier` nunca rodam no mesmo spawn — o escritor e o auditor devem ser atores distintos (segregação de função). Mesma lógica de auditoria da família Audit.

## Padrões reutilizáveis

1. **Por que 4 agents e não 1 só?** Separação de **responsabilidade e de nível de trust**: classifier lê um doc; synthesizer cruza N classificações; writer produz novo conteúdo; verifier ataca o que foi produzido. Misturar papéis vaza confiança indevida — um writer que também "valida" tende a confiar no que acabou de escrever. Segregação força stance adversarial explícita no verifier.
2. **Paralelismo por fan-out/fan-in**: classifier spawna em **paralelo** (1 por arquivo), synthesizer **agrega** serialmente. Contrato entre etapas é JSON com schema fixo; hash de path no filename previne colisão entre `README.md` múltiplos no mesmo repo. Mesmo padrão de MapReduce, aplicado a docs.
3. **Precedência explícita > merge implícito**: ADR > SPEC > PRD > DOC com override opcional; LOCKED vs LOCKED nunca auto-resolve, competing PRDs preservam todas as variantes. "Surface conflict rather than pick" é a regra de ouro. Análogo ao *law of the hierarchy of norms* em direito (constituição > lei > decreto).
4. **Discovery-first templating**: cada template declara "Required Sections" + "Content Discovery" (quais arquivos grep/read para descobrir os fatos) + "Format Notes". Estrutura é fixa; descoberta adapta ao projeto. Zero fabricação permitida — VERIFY markers para o que não se pode provar do repo.
5. **VERIFY markers como TODO legível por máquina**: `<!-- VERIFY: reason -->` é simultaneamente placeholder pra humano E entrada pra verifier grep-ar. O pattern replica `FIXME`/`TODO` mas com semântica específica (verificação factual pendente).

## Correspondência com nosso pipeline

| GSD agent | Nosso role | Uso |
|---|---|---|
| `gsd-doc-classifier` | **Planner** (sub-etapa: triagem) | Classificação inicial de material existente antes de sintetizar decisões/requisitos |
| `gsd-doc-synthesizer` | **Dual Reviewer** + **Planner** | Aplicação de precedência entre fontes, surfacing de conflitos em vez de merge silencioso |
| `gsd-doc-verifier` | **Dual Reviewer** (modo adversarial de doc) | Dual-check específico para drift doc ↔ código; invocado após qualquer mudança significativa de doc |
| `gsd-doc-writer` | **Worker** (especializado em docs) | Modo `fix` encaixa pós-verifier, modo `supplement` preserva prosa humana, modo `create/update` gera canônicos |

Note que a pipeline classifier → synthesizer mapeia bem para nossa ingestão de material legado dentro de `50-projects/<p>/` (D-### antigos, ADRs informais, PRDs de reunião), e writer + verifier mapeiam para manutenção contínua de `04-requirements/`, `02-architecture/` e READMEs.

**Insight operacional**: o vault `automation-agents` hoje assume que docs são escritos manualmente ou pelo Worker ad-hoc. Adotar o pipeline de 4 agents significaria ter um Worker-de-docs explícito com modos e um Dual Reviewer que roda `doc-verifier` automaticamente em toda mudança de ARCHITECTURE.md, README.md ou `04-requirements/`. O ganho é garantir que docs drifted sejam detectadas cedo, não em review de PR 3 semanas depois.

## Patterns atemporais extraídos

1. **Pipeline de docs como fan-out/fan-in paralelo**: classificar N docs em paralelo + sintetizar um só documento canônico. Aplicável além de docs — qualquer ingestão de N fontes heterogêneas (logs de N serviços, issues de N repos, ADRs de N teams).
2. **Surface conflict, don't resolve**: auto-merge entre fontes LOCKED é anti-padrão — escolher por heurística apaga contexto que o humano precisa pra decidir. Pattern transferível pra qualquer workflow de consolidação (feature flags conflitantes, policies sobrepostas).
3. **Discovery-first templating**: template declara onde buscar a verdade + estrutura final; gerador adapta a estrutura aos findings do codebase. Pattern vale pra qualquer code-gen com anchor na base de código real — não só docs.
4. **Segregação escritor ≠ verificador**: o mesmo LLM num spawn separado com prompt adversarial produz verificação mais sólida que "self-review". Custo baixo (segundo spawn) e ganho alto (detecta confabulação do primeiro).

## Links externos

- [Diátaxis framework](https://diataxis.fr/) — separação tutorial/how-to/reference/explanation ajuda a definir quais tipos cada `doc-writer` mode gera
- [Google — Documentation that drives engineering](https://research.google/pubs/pub43146/) — caso de doc-as-code canônico
- [GitLab Handbook — Docs as Code](https://handbook.gitlab.com/handbook/engineering/ux/technical-writing/) — mesmo princípio de verificação automática de drift
- [Martin Fowler — Living Documentation](https://martinfowler.com/bliki/LivingDocumentation.html) — conceito base que o `doc-verifier` operacionaliza
- [Stripe Press — Scaling Developer Documentation](https://stripe.com/blog) — case study sobre manter docs sincronizadas com API em produção

## Fonte
- [[../../60-sources/get-shit-done/_toc]]

## Links
- [[_moc]]
- [[pipeline-mapping]]
- [[../../10-knowledge/methodology/gsd-get-shit-done]]
