---
title: Playbook — Rollback
type: playbook
tags: [playbook, execution, incident, rollback]
created: 2026-04-22
---

# Playbook — Rollback

Reverter mudança que já foi aplicada mas causou regressão, incident ou não cumpriu critério. Cobre: commit local, merged PR, deploy em produção, migration de DB.

## Princípios

1. **Reverter é decisão valorizada**. Hesitar amplia o estrago.
2. **Roll forward > rollback** quando fix é simples e rápido (< 15min). Caso contrário, rollback e depois fix com calma.
3. **Nunca rollback destrutivo** sem confirmar preservação de dados novos criados após a mudança.
4. **Sempre registrar** rollback em AUDIT.md com root cause + lições.

## Níveis

### Nível 1 — Código local (não commitado)
- `git restore <file>` ou `git checkout <file>`
- Se arquivos novos: `git clean -n` (dry-run) → `git clean -f`
- **Não** usa `git reset --hard` sem confirmar com o dev

### Nível 2 — Commit local (não pushado)
- `git reset --soft HEAD~1` pra manter mudanças staged
- `git reset --mixed HEAD~1` pra manter mudanças unstaged (default)
- `git reset --hard HEAD~1` **só com confirmação do dev** (destrutivo)

### Nível 3 — PR mergeado
- **Preferido**: `git revert <sha>` → novo commit de reverso → PR reverso
- Vantagem: preserva histórico, permite forward roll depois
- Evita `git reset --hard origin/main` + force-push (destrutivo e visível a todos)

### Nível 4 — Deploy em produção
- Se plataforma tem versionamento (Railway, ECS, Vercel):
  - Promover release anterior
- Se é container: retag anterior como `:latest` → rollout
- Se k8s: `kubectl rollout undo deployment/<name>`
- Validar: smoke test em health-check + endpoint crítico + métricas
- Se falhou rollback: escalar

### Nível 5 — Migration de DB
**Mais delicado**. Regras:

1. **Migrations são forward-only por default** (goose, migrate, Prisma, Alembic). Down-migrations são frágeis.
2. Antes de aplicar migration em prod, ter **backup** pronto (snapshot / pg_dump).
3. Migration destrutiva (DROP COLUMN, DROP TABLE) **nunca** em single-step — usar padrão **expand + contract**:
   - Expand: adicionar coluna nova (nullable), dual-write no código
   - Migrate data
   - Contract (sprint depois): remover coluna velha

Se migration foi aplicada e causou incidente:
- Se é pura ALTER (adição): normalmente seguro; fix forward
- Se é DROP: considerar restore de backup parcial; NUNCA rollback cego (perde dados criados após o DROP)
- Decisão sempre envolve o dev humano

## Fluxo operacional

### 1. Detectar
- Alerta (Sentry, Grafana)
- Smoke test falhou
- Dev reportou
- Métricas degradadas pós-deploy

### 2. Decidir (dentro de 5min)
| Cenário | Ação |
|---|---|
| Fix forward < 15min viável | Roll forward: hot-fix PR urgente |
| Fix forward >= 15min ou não claro | Rollback |
| Bug causa perda de dados / corrupção | Rollback **agora**, analise depois |
| Bug causa vazamento de dado | Rollback + incident response |

### 3. Executar
- Anunciar no canal do time (ou em SESSION_CHARTER se solo): "ROLLBACK em andamento — <motivo>"
- Executar no nível apropriado (1-5)
- Validar saúde: liveness + readiness + endpoint crítico
- Desativar feature flag se o culpado é feature-flagged

### 4. Estabilizar
- Monitorar 15min pós-rollback
- Se métricas não voltaram: o culpado era outra mudança; re-investigar

### 5. Registrar em AUDIT.md

```markdown
### A-### — Rollback de <mudança> — <data ISO> ✅ Resolvido
- **Descoberta**: <quando>
- **Severidade**: critical | high
- **Origem**: commit <sha> / PR #<num>
- **Sintoma**: <o que falhou>
- **Raiz**: <causa real — não sintoma>
- **Ação**: rollback nível <N> via <comando>
- **Tempo até detect**: <m>
- **Tempo até mitigar**: <m>
- **Tempo de impacto total**: <m>
- **Lições**: <o que evitaria — gate, teste, canary, etc.>
- **Action items**:
  - [ ] Criar teste que reproduz o bug
  - [ ] Adicionar gate/monitor pra detectar antes
  - [ ] (opcional) Re-deploy forward com fix após correção
```

## Quando **NÃO** fazer rollback

- Mudança criou estrutura nova que usuários já usaram (dados novos no banco)
  → fix forward, não pode rollback sem perder dados
- Mudança passou por dual-check, gates, staging, e foi merged há > 48h
  → problema provavelmente não é **a** mudança; investigar antes
- Dev humano ainda **não** autorizou rollback destrutivo (nível 5 com DROP)

## Comunicação

Durante rollback em prod:
- Anúncio inicial: "ROLLBACK em andamento — <motivo em 1 linha>"
- Confirmação: "ROLLBACK concluído — sistema estável"
- Post-rollback: "Post-mortem em AUDIT A-###"

Se escopo é incident-level (muitos usuários afetados):
- Status page atualizada
- Notificação de clientes se SLA tocado
- Incident report dentro de 72h

## Links

- [[_moc]]
- [[plan-and-execute]]
- [[cve-scan]]
- [[../../10-knowledge/observability/_moc]]
- [[../../CLAUDE]]
