# Skill: Go Code Review

**Referências obrigatórias**: `.kiro/steering/do-dont.md` (checklist consolidado), `.kiro/steering/go-patterns.md` (padrões), `.kiro/steering/security.md` (regras de segurança), `.kiro/specs/master-sindico/reference/antipatterns-to-avoid.md` (armadilhas do legacy).

Quando solicitado a revisar código Go neste projeto, avalie os seguintes pontos em ordem de prioridade:

## 1. Dependency Rule (bloqueador)
- `domain/` importa algo de `infrastructure/` ou `application/`? → ERRO CRÍTICO
- `shared/middleware/` importa de `modules/`? → ERRO CRÍTICO
- Handler importa `gin` diretamente? → ERRO CRÍTICO
- `core/errors/` importa Gin? → ERRO CRÍTICO

## 2. Error Handling (bloqueador)
- `_ = fn()` em operação falível? → PROIBIDO
- `err.Error() == "string"` para comparar erros? → PROIBIDO, use `errors.Is/As`
- Handler escrevendo resposta de erro diretamente em vez de `ctx.SetError(err)`? → PROIBIDO
- Erro engolido sem log? → PROIBIDO

## 3. Padrões Go
- if/else aninhado em vez de early return? → Refatorar
- `float64` para valores monetários? → PROIBIDO, use `int64` centavos
- Variáveis abreviadas (`uow`, `condo`, `q`)? → Renomear para nome completo
- Falta de `context.Context` como primeiro parâmetro em função com I/O? → Adicionar

## 4. DDD / Clean Architecture
- Entidade com campos públicos? → Tornar privado + getter
- Value Object mutável? → Tornar imutável
- Use case fazendo mais de uma coisa? → Separar (CQRS)
- Repository interface no lugar errado (não em `domain/repositories/`)? → Mover
- Falta de compile-time interface check? → Adicionar `var _ Interface = (*Impl)(nil)`

## 5. Segurança (detalhes em `steering/security.md`)
- Dados sensíveis em log (CPF/CNPJ, tokens, senhas)? → Remover/mascarar
- SQL construído por concatenação de string? → Usar sqlc/parâmetros posicionais
- Cookie sem `HttpOnly` ou `Secure` (em prod)? → Corrigir
- Segredo hardcoded? → Mover para config + Secrets Manager
- Webhook sem validação de assinatura **antes** do parsing? → Corrigir
- Input aceito sem validação server-side? → **Never trust the frontend**
- JWT Profile ausente na introspection Zitadel? → Usar service account key

## 6. Antipatterns do legacy (`reference/antipatterns-to-avoid.md`)
- Vazamento de domínio entre bounded contexts? → Isolar via `shared/types/` ou eventos
- If/else hell em Update use cases? → Strategy pattern
- Duplicação de `authorize()` em múltiplos métodos? → Middleware `RequirePermission`
- Race condition em quotas (incrementUsage sem lock)? → `SELECT ... FOR UPDATE`
- Validação só na aplicação (DB aceita lixo)? → CHECK constraints

## Formato do relatório
Para cada problema encontrado:
1. **Arquivo e linha** onde está o problema
2. **Por que** é um problema (não apenas o que está errado)
3. **Como corrigir** com exemplo de código

Não avance se houver problemas críticos (1 ou 2). Corrija primeiro.
