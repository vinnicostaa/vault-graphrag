---
title: STEERING — Master Síndico
type: project
tags: [master-sindico, steering, governance]
project: master-sindico
created: 2026-04-22
---

# STEERING — Master Síndico

Não-negociáveis do projeto. Regras que **não são revisitadas** sem aprovação humana explícita. Complementa [[CLAUDE]] (contrato operacional) com a **constituição do produto**.

Origem: destilado dos `backend/.kiro/steering/*.md` (13 arquivos) + feedback consolidado do dev nas sessões de 2026-04 + [[../../CLAUDE]].

---

## 1. Inegociáveis de produto

1. **Governança, não ERP** — Master Síndico ajuda a **decidir** e **auditar**, não a emitir NF ou processar cobrança operacional fina.
2. **Reputação como núcleo** — sistema Bronze → Prata → Ouro → Platina → Diamante é produto, não *nice-to-have*. Qualquer feature que subverta reputação (ex: permitir reviews anônimas sem moderação) viola a constituição.
3. **Identidade única, contextos múltiplos** — CPF/CNPJ é 1 identidade; pode ser síndico no Condomínio X e morador no Y. Feature que duplique identidade = violação.
4. **Memória institucional imutável** — timeline do condomínio nunca é editada, só corrigida por entry nova (link à original + motivo). `timeline_entries` sem `deleted_at`, sem UPDATE.
5. **Privacidade de métricas** — métricas de vídeo/engajamento não vazam entre tenants; views anônimas por default.
6. **Pt-BR first** — mercado brasileiro; toda UI, doc do cliente e templates em pt-BR antes de qualquer outra localidade.

## 2. Inegociáveis de engenharia

1. **DDD + Clean Arch + CQRS + SOLID + TDD + SDD + Saga + ACID + Code Calisthenics** — bloco canônico; sem dispensa sem D-### fundamentado
2. **Sem bloco `else`** — early return sempre; decisão registrada em memória `feedback_no_else`
3. **`_ = fn()` em I/O crítica = bug** — log de erro obrigatório; histórico em [[11-audit/AUDIT]] A-001..A-008
4. **No `interface{}` / `any`** onde tipo concreto serve
5. **Domain **ZERO** imports de framework/SDK** — Gin, pgx, Stripe SDK, Mux SDK, Livekit SDK ficam em `infrastructure/`
6. **sqlc com colunas explícitas** — sem `SELECT *` (A-021)
7. **Testes antes do merge** — coverage thresholds (ver `CLAUDE.md §8`); `<verify>` é contrato
8. **Dual-check obrigatório** em arquitetura / segurança / contrato público (ver [[../../30-agents/playbooks/dual-check]])
9. **Research-loop obrigatório** antes de adotar pattern ou lib (ver [[../../30-agents/playbooks/research-loop]])

## 3. Inegociáveis de segurança

1. **Zero-trust**. Sem rede interna confiável. Toda request autenticada e autorizada.
2. **BeyondCorp adaptado** é produto, não extra. ABAC matrix, tenant isolation, 1-device.
3. **PII nunca em logs**. CPF, CNPJ, email, token, senha — proibido. Usar `Masked()` ou `user_id`.
4. **Secrets nunca em código**. `.env` gitignored; `zitadel-key*.json` gitignored (A-018).
5. **Webhook signature validada antes de parse**. Stripe, Mux: verificar HMAC, rejeitar body se inválido, **depois** parsear.
6. **CORS wildcard bloqueado em staging/prod** via `Config.Validate()`.
7. **Rate limit separado** por superfície: /auth (mais restrito) e /api/v1.
8. **Multi-tenant** — toda query escopada; cross-tenant tests em CI.

## 4. Inegociáveis de ops

1. **Migrations forward-only por default** — `goose`; destrutivo = expand + contract
2. **Webhooks idempotentes** — Stripe/Mux podem retry; `event_id` único como idempotency key
3. **Gates bloqueiam merge** — build + vet + test + coverage + gosec + govulncheck
4. **Rollback via revert**, não force-push (ver [[../../30-agents/playbooks/rollback]])
5. **Deploy em horário útil** — não sexta tarde; a não ser crítico de segurança

## 5. Inegociáveis de qualidade

1. **Feature sem teste = feature não entregue**
2. **Docs e código sincronizados** — PR que muda rota/schema atualiza OpenAPI no mesmo commit
3. **Legibilidade > esperteza** — código é lido 10x mais do que escrito
4. **Nomes no ubiquitous language** — sem abreviações; `Assembleia`, não `Asm`
5. **Mensagens de erro úteis** — `ErrValidation.WithMessage("starts_at obrigatório")`, não `ErrInternal`
6. **Commit mensagem imperativa**, escopo claro, sem `fix:`/`feat:`; corpo explica **porquê**
7. **Release Notes** em toda PR body

## 6. Alçadas (quem decide o quê)

| Decisão | Quem decide |
|---|---|
| Rebate da ordem canônica (pular migration, por ex) | Dev humano |
| Mudança de stack (Go→Rust, etc.) | Dev humano via D-### + research + dual-check |
| Mudar padrão da constituição (permitir `else`, etc.) | Dev humano via D-### |
| Adicionar dependência crítica | Agente via dep-audit + research-loop + dual-check |
| Bump major | Agente via research-loop + dual-check |
| Bump minor/patch | Agente direto, rodar dep-audit |
| Fix de bug interno | Agente |
| Deploy em produção | Dev humano sempre |
| Rollback nível 5 (DROP) | Dev humano |
| Priorização dentro do sprint | Agente (segue marco M1→M2→M3) |
| Priorização entre marcos | Dev humano |

## 7. O que **não** fazer nunca

1. Criar feature não pedida "enquanto estou aqui"
2. Commitar `.env`, `zitadel-key*.json`, qualquer credencial
3. Mexer em `migrations/` já executadas em ambiente compartilhado — só nova migration
4. `any`/`interface{}` onde tipo concreto serve
5. Wrap de erro silenciando (`_ = fn()`, `return err` sem contexto)
6. Esconder erro em retorno (`return nil` quando houve falha)
7. Usar `else` ou aninhar 3+ ifs
8. SDK externo no domínio
9. Lógica de negócio em handler ou middleware
10. Mock de DB em teste de integração (testcontainers sempre)
11. Mudar schema de resposta pública sem version bump + deprecation

## 8. Lições consolidadas de sessões anteriores

Mantidas aqui pra agente futuro não repetir.

- **A-012 / A-017**: `tasks.md` ficou desatualizado pós-sprint em 2 ocasiões. Task-executor atualiza `tasks.md` **dentro** da fase Ship, antes do commit.
- **A-018**: `.gitignore` deve proteger também `zitadel-key*.json`; não confiar em pattern `[0-9]*[0-9].json`
- **A-022**: webhook externo sem token Zitadel deve ficar **fora** de `/api/v1` (middleware Authenticate bloqueia Mux/Stripe)
- **A-027 / A-028**: chamada a provider externo sem atualização local = órfão. Saga com compensação explícita.
- **D-029**: sincronizar configs globais `~/.claude/settings.json` → `backend/.claude/settings.json`

## Links

- [[CLAUDE]] — contrato operacional do agente
- [[STATE]] — decisões vivas e dívidas
- [[11-audit/AUDIT]] — findings
- [[ROADMAP]]
- [[DoR-DoD]]
- [[../../CLAUDE]] — constituição raiz do vault
