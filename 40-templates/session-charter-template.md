---
title: SESSION_CHARTER.md — template
type: template
tags: [template, session, governance, autonomous]
source: .kiro/SESSION_CHARTER.md (padrão extraído)
created: 2026-04-22
aliases: [SESSION_CHARTER template, Session Charter]
---

# SESSION_CHARTER — `<título da sessão>` (`<YYYY-MM-DD>`)

**Escopo da sessão**: `<descrição de 1-2 linhas do objetivo da sessão>`

**Duração esperada**: `<N horas>` — `<modo: ponto a ponto | longa autônoma | multi-sessão>`

**Data de início**: `<YYYY-MM-DD>`. **Última atualização**: `<YYYY-MM-DD>`.

---

## 1. Ordens do usuário (cronológico, preservando palavras-chave)

Reproduzir as solicitações do usuário **literalmente**, pra manter rastreabilidade e evitar "eu achei que era isso" depois. Use blockquote com aspas.

### 1.1 `<contexto da ordem 1>`
> "`<citação textual da ordem do usuário>`"

### 1.2 `<contexto da ordem 2>`
> "`<citação>`"

### 1.N `<...>`

---

## 2. Princípios inegociáveis (consolidados de memórias + steering + ordens)

### Engenharia

- **`<lista-de-paradigmas-e-patterns-obrigatorios>`** — origem: `<memória|steering|pedido>`
- Exemplos típicos: DDD, Clean Arch, CQRS, SOLID, TDD, SDD, GSD, Saga, ACID, Code Calisthenics

### Qualidade sobre velocidade

- **`<prioridade-1>` > `<prioridade-2>` > ...** — ex: segurança > escalabilidade > robustez > velocidade
- **Errar é inaceitável**: não deixar passar nada; revisar ao terminar cada task.

### Modo de operação

- **Default**: `<autônomo|supervisionado>` — `<regra aplicável>`
- **Modo autônomo** (se ativo nesta sessão): não pausa por dúvida técnica. Pesquisa 5 fontes + registra em STATE.md como D-0XX. Pausa só em 5 categorias destrutivas — ver [[../30-agents/autonomous-execution]].

### Gates duros (cada task, antes de marcar concluída)

1. `<build-cmd>` — limpo
2. `<lint/vet-cmd>` — limpo
3. `<test-cmd>` `-race -count=1` — todos verdes
4. `<integration-test-cmd>` — todos verdes
5. Coverage por camada: `<thresholds-específicos-do-projeto>`
6. `<security-scanner>` `-severity=high` — zero findings
7. `<vuln-check>` — zero CVEs

---

## 3. Fontes de verdade (prioridade de consulta)

1. `<path-canônico-specs>` — **canônico** pra implementação
2. `<content-ou-context-dir>` — documentos-fonte originais (requirements.md, design.md, tasks.md mestres)
3. Obsidian vault — decisões históricas, arquitetura documentada (pode estar parcialmente desatualizado — marcar)
4. **Código real** — quando divergir da doc, **código é o estado atual**
5. `<steering-dir>/*.md` — regras duras por tópico
6. `STATE.md` — decisões vivas (D-0XX) e dívidas (DT-0XX)
7. `AUDIT.md` — bugs descobertos em revisão (A-0XX)

### Paths importantes fora do projeto

- `<path-workspace>/CLAUDE.md` — workspace multi-projeto (se aplicável)
- `<path-content>/` — PDFs e docs originais
- `<settings-local>` — permissões workspace-level
- `<settings-user>` — user-level (modelo, language, plugins globais)
- `<memory-dir>` — memórias persistentes entre sessões

---

## 4. Lista de features/tasks desta sessão (F1-FN)

Estado: `<data>` **status final** — todas concluídas / em andamento / bloqueado.

| ID | Título | Status | Evidência |
|---|---|---|---|
| F1 | `<descrição curta>` | ✅ / 🔨 / 📋 | `<A-### / commit / arquivo>` |
| F2 | `...` | ... | ... |
| FN | `...` | ... | ... |

---

## 5. Decisões tomadas nesta sessão (D-0XX ativas)

Lista resumida — detalhe completo em `STATE.md#decisões-arquiteturais`.

- **D-0XX** — `<título>` — registrada em `<data>`
- **D-0XX** — `<título>` — registrada em `<data>`

---

## 6. Findings descobertos nesta sessão (A-0XX ativos)

Lista resumida — detalhe completo em `AUDIT.md`.

- **A-0XX** — `<título>` — `<severidade>` — `<arquivo:linha>`
- **A-0XX** — `<título>` — `<severidade>` — `<arquivo:linha>`

---

## 7. Bloqueadores encontrados e como foram resolvidos

### `<bloqueador 1>`

- **Sintoma**: `<o que apareceu>`
- **Causa raiz**: `<o que estava errado>`
- **Resolução**: `<o que foi feito>`
- **Tempo perdido**: `<estimativa>`

---

## 8. Ao terminar a sessão — checklist de fechamento

- [ ] **STATE.md atualizado** com todas as decisões autônomas (`D-0XX`) + dívidas descobertas
- [ ] **AUDIT.md em dia** — findings registrados, resolvidos movidos pro histórico
- [ ] **Diff explicável** — cada arquivo modificado tem commit que descreve o **porquê**
- [ ] **Zero bloqueios silenciosos** — se houve pausa obrigatória (ver autonomous-execution), bloco no início do relatório final
- [ ] **README do projeto atualizado** se mudança feature-level
- [ ] **Release notes** — seção no PR body
- [ ] **Próxima sessão** — briefing inicial escrito: o que começar, o que cuidar

---

## 9. Teste de qualidade da sessão

Usuário consegue **reconstruir o raciocínio** sem precisar perguntar ao agente? Tudo está em:

- Commits + mensagens
- STATE.md decisões
- AUDIT.md findings
- PR release notes
- Este SESSION_CHARTER

Se precisa perguntar → documentação incompleta → ajustar.

---

## Links

- [[_moc]]
- [[state-template]]
- [[audit-template]]
- [[../10-knowledge/methodology/sdd-workflow]]
- [[../30-agents/autonomous-execution]]
- [[../30-agents/mcp-integration]]
