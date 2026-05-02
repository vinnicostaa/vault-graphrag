---
title: Clean Code — princípios canônicos
type: concept
tags: [principles, clean-code, readability, naming, refactoring]
source: Robert C. Martin, "Clean Code" (2008)
created: 2026-04-23
aliases: [Clean Code Uncle Bob, Código Limpo]
---

# Clean Code — princípios canônicos

Destilação dos princípios de *Clean Code* (Robert C. Martin, 2008) aplicáveis a qualquer stack. Não é estilo — é **disciplina para reduzir o custo de manutenção**. Código é lido 10x mais do que escrito.

> **Teste empírico do livro**: qualquer pessoa que entre na base pela primeira vez deve conseguir entender a função em **~1 minuto**. Se não consegue: o código não é limpo.

## 1. Nomes significativos

```go
// ❌
func d(x int) int { return x * 86400 }
d := u.Sub  // "d" de quê?
arr := []int{...}

// ✅
func DaysToSeconds(days int) int { return days * 86400 }
activeSubscription := user.Subscription
userIDs := []string{...}
```

Regras:
- Nome deve **revelar intenção** — se precisa de comentário explicando, escolhe errado
- Evitar **disinformação**: `accountList` não sendo uma `List<Account>` engana
- **Distinção significativa**: `customerInfo` vs `customerData` não distinguem nada
- **Pronunciável**: `genymdhms` → `generationTimestamp`
- **Pesquisável**: `MAX_CLASSES_PER_STUDENT` > `7` (número mágico espalhado)
- **Evitar prefixos tipo**: `m_dsc`, `IShapeFactory` (se-tá-na-linguagem-use-a-linguagem)

## 2. Funções pequenas, uma coisa

> "Funções devem ser pequenas. **Muito** pequenas. Depois devem ser ainda menores."

- **Tamanho alvo**: 5-20 linhas. Acima de 50: decompor.
- **Um nível de abstração**: dentro de uma função, comandos devem estar no mesmo nível conceitual. Misturar `db.Query(...)` com `businessRule.Apply(...)` viola.
- **Fazer uma coisa**: se a função faz A e B, é duas funções.

```go
// ❌ Mistura abstrações + faz múltiplas coisas
func HandleLogin(w http.ResponseWriter, r *http.Request) {
    body, _ := io.ReadAll(r.Body)
    var req LoginRequest
    json.Unmarshal(body, &req)

    row := db.QueryRow("SELECT * FROM users WHERE email = $1", req.Email)
    var user User
    row.Scan(&user.ID, &user.Email, &user.PasswordHash)

    if !bcrypt.CompareHashAndPassword(user.PasswordHash, req.Password) {
        w.WriteHeader(401)
        return
    }

    token := jwt.New(...)
    // ...mais 40 linhas...
}

// ✅ Cada função uma coisa, um nível de abstração
func HandleLogin(w http.ResponseWriter, r *http.Request) {
    req, err := parseLoginRequest(r)
    if err != nil { writeError(w, err); return }

    user, err := authenticate(r.Context(), req)
    if err != nil { writeError(w, err); return }

    token := issueToken(user)
    writeSuccess(w, token)
}
```

## 3. Comentários — quase sempre um cheiro

Código ruim pede comentário; **código bom expressa**. Exceções legítimas:
- **Licenças/copyright** no topo
- **Aviso de não-óbvio**: "this runs in nanosecond-critical path, don't alloc"
- **Explicação de porquê** (nunca de quê/como): "acts as a retry barrier because the provider is flaky — see #1234"
- **TODO/FIXME** com ticket de rastreio

```go
// ❌
// Incrementa i por 1
i++

// ❌ comentário mentindo (código mudou, comentário não)
// Retorna user.Age em anos
func GetAge() int { return user.birthYear }

// ✅ porquê + link
// Mux cobra se não cancelarmos o upload antes do timeout de 10min.
// Ver ADR-0004 e caso A-027.
defer mux.CancelUpload(ctx, assetID)
```

**Regra**: antes de escrever comentário, tentar renomear função/variável pra dizer o que o comentário diria.

## 4. Formatação consistente

- **Linhas curtas**: ~80-120 caracteres. Linha de 200 força scroll horizontal, perde-se.
- **Abstração vertical**: conceitos relacionados próximos; conceitos distantes separados por espaço em branco.
- **Ordem top-down**: função que usa outras vem antes; detalhes descem.
- **Ferramenta decide estilo**: `gofmt`, `prettier`, `rustfmt`, `black`, `biome`. Debate sobre tabs vs espaços é perda de tempo.

## 5. Objetos e estruturas de dados são diferentes

- **Objeto**: expõe comportamento, esconde dados. Adicionar comportamento novo é fácil; adicionar tipo novo é difícil.
- **Estrutura de dados**: expõe dados, poucos comportamentos. Adicionar função nova é fácil; adicionar campo novo é difícil.

```go
// Estrutura de dados — DTO
type UserResponse struct {
    ID    string `json:"id"`
    Email string `json:"email"`
}

// Objeto — Entity
type User struct {
    id    UserID
    email Email
    role  Role
}
func (u *User) Ban(reason string) error { ... }
func (u *User) CanAccess(resource Resource) bool { ... }
```

Misturar = "data class com métodos mágicos escondidos". Regra: **DTOs sem lógica, entidades sem exposição**.

## 6. Error handling não é feature secundária

- Erros são **parte do contrato** — documentar tipos de erro que a função retorna
- Nunca **engolir** erros silenciosamente: `_ = fn()` é bug (ver [[../runtime-antipatterns/_moc|OP-020]] — erro engolido)
- **Não usar exceptions** para controle de fluxo — exceção para exceção
- **Early return em erro**, caminho feliz no nível 0

```go
// ✅
user, err := repo.FindByID(ctx, id)
if err != nil {
    return fmt.Errorf("FindByID: %w", err)  // wrap para stack trace
}
if err := user.Deactivate(); err != nil {
    return fmt.Errorf("Deactivate: %w", err)
}
return repo.Save(ctx, user)
```

## 7. Boundaries — isolamento de terceiros

Todo código de terceiro (SDK, biblioteca externa) é **fronteira instável**. Wrappar.

- SDK Stripe/Mux/Twilio vive em `infrastructure/providers/<provider>.go`
- Código de domínio **nunca** importa tipos do SDK
- Mapper converte `stripe.Subscription` → `domain.Subscription`
- Se o SDK muda API: **um arquivo** altera, não a codebase inteira

Ver [[../architecture/clean-architecture-layers]] e [[solid#5-dip]].

## 8. Testes são código de produção

- Nomes descritivos: `TestChargeCard_FailsWhenCardExpired`, não `TestCase17`
- **Um conceito por teste** — múltiplos asserts tudo bem; múltiplos comportamentos: refatorar
- **AAA** (Arrange-Act-Assert) visível
- Testes **rápidos** (unitário milissegundos), **determinísticos** (sem rand não-seedado, sem `sleep`), **isolados** (sem ordem implícita)

Ver [[../testing/testing-strategy]] e [[../testing/bdd-behavior-driven-development]].

## 9. Regra do escoteiro

> "Deixe o acampamento mais limpo do que o encontrou."

Toque de meia dúzia de linhas numa função chata? Aproveita e renomeia a variável confusa ao lado. Multiplicado por 100 pessoas × 100 commits: codebase se **autolimpa**.

**Regra pragmática no contexto SDD**: refactor **dentro do escopo da task**. Não abrir refactor paralelo que explode o diff e impede review.

## 10. Duplicação é raiz do mal

DRY — mas com critério. **3 tipos** de duplicação:

1. **Duplicação literal** (copy-paste): remover sempre
2. **Duplicação estrutural** (código parecido, valores diferentes): candidato a generic/template/strategy
3. **Duplicação de conhecimento** (regra de negócio em 2+ lugares): **crítico** — divergência silenciosa no futuro

Regra **do 3**: 2 cópias pode esperar; 3ª cópia = refactor obrigatório.

Abstrair cedo demais é outro tipo de duplicação disfarçada — ver [[solid]] seção "Quando NÃO aplicar" e [[../methodology/gsd-get-shit-done]] sobre YAGNI.

## 11. Função tem poucos parâmetros

- 0 é ideal
- 1-2 ok
- 3 questionável
- 4+ quase sempre sinal para agrupar em struct ou dividir função

```go
// ❌
func CreateUser(name, email, cpf, phone string, role Role, plan Plan, referralCode string) (*User, error)

// ✅
func CreateUser(ctx context.Context, input CreateUserInput) (*User, error)
type CreateUserInput struct {
    Name, Email, CPF, Phone string
    Role                     Role
    Plan                     Plan
    ReferralCode             string
}
```

## 12. Evitar flags booleanas

Parâmetro `bool` quase sempre sinaliza que a função **faz 2 coisas** — uma quando `true`, outra quando `false`.

```go
// ❌
func RenderPage(user User, isAdmin bool) { ... }

// ✅ Duas funções
func RenderPage(user User) { ... }
func RenderAdminPage(user User) { ... }
```

Exceção legítima: flags que representam **opções de configuração**, não comportamento.

## 13. Command-Query Separation

- **Command**: faz algo, não retorna (ou retorna só erro/status)
- **Query**: retorna dado, **sem efeito colateral**

Misturar confunde: `if (setUser(id)) { ... }` — setou? leu? mudou quê?

Ver [[../architecture/cqrs]] para aplicação a nível arquitetural.

## 14. Princípio da menor surpresa

Método chamado `GetUser` **nunca** pode mutar estado. Construtor **nunca** deve fazer I/O pesado. `equals` **nunca** deve retornar `null`. Convenções da linguagem/domínio **governam**.

## 15. Structured programming

Uma entrada, uma saída por função (Dijkstra). **Early returns** são exceção aceita — reduzem indent, deixam caminho feliz plano.

`goto`, `break` com label, `continue` aninhado: evitar.

## 16. Small classes (SRP, repetido por importância)

- **Raciocínio por peso emocional**: classe que dá "preguiça abrir" geralmente viola SRP
- **Nomes revelam violação**: `UserManager`, `AccountHelper`, `GenericProcessor` = God Object em potencial
- **Contagem**: ≤ 10 campos, ≤ 15 métodos públicos

## 17. Tratamento de concorrência como design

Threading/goroutines não são "adicionadas depois" — mudam a estrutura.
- **Imutabilidade** reduz surface
- **Canais** em vez de locks onde couber (Go): "share memory by communicating"
- **Identificar** regiões críticas **antes** de paralelizar

## Checklist rápido de review

- [ ] Todo nome revela intenção?
- [ ] Nenhuma função passa de 50 linhas?
- [ ] Nenhum comentário descreve **o quê** (só **por quê**)?
- [ ] Error handling: nada de `_ = err`?
- [ ] Boundaries: nada de SDK externo em `domain/`?
- [ ] Testes nomeados por comportamento?
- [ ] Código duplicado 3x? → refactor
- [ ] Funções com ≤ 3 parâmetros?
- [ ] Boolean flag parâmetro? → dividir função

## Por que importa

Martin argumenta (e estudos empíricos como *A Philosophy of Software Design* de John Ousterhout corroboram): **custo total de um sistema é dominado pela manutenção**, não pela escrita original. Código limpo é a única alavanca real sobre esse custo dentro do poder do desenvolvedor.

Em contextos com LLM-as-collaborator (agentes), clean code é ainda mais crítico: **o agente lê pra entender o contexto**. Código sujo → agente usa mais contexto pra entender → menos espaço para gerar → pior resultado.

## Referências

- Robert C. Martin. *Clean Code* (2008) — texto canônico
- Martin Fowler. *Refactoring* (2nd ed, 2018) — catálogo de refactors complementar
- John Ousterhout. *A Philosophy of Software Design* (2018) — contraponto elegante, foco em "deep modules"
- Kent Beck. *Tidy First?* (2023) — sobre quando/como aplicar limpeza incremental

## Links

- [[_moc]]
- [[solid]]
- [[code-calisthenics]]
- [[do-dont-checklist]]
- [[../architecture/clean-architecture-layers]]
- [[../architecture/ddd-strategic-tactical]]
- [[../antipatterns/_moc]]
- [[../testing/testing-strategy]]
