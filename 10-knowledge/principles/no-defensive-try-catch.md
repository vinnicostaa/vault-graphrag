---
title: No try/catch defensivo — capturar só com ação útil
type: concept
tags: [principle, error-handling, clean-code, defensive-programming]
created: 2026-04-24
updated: 2026-04-24
aliases: [No defensive try/catch, Error handling principle]
---

# No try/catch defensivo — capturar só com ação útil

Regra: **capturar exceção/erro apenas quando há ação útil a tomar**. Caso padrão é **propagar**. Try/catch genérico "por segurança" esconde bugs, encobre root cause e gera código defensivo inútil.

## Por que try/catch defensivo é antipattern

```ts
// ❌ Defensivo: captura porque "pode dar errado"
try {
  const user = await repo.findById(id);
  return user;
} catch (e) {
  console.log(e);
  return null;
}
```

Problemas:

1. **Engole identidade do erro** — chamador recebe `null` sem saber se é `not_found`, `timeout`, `db_down`, `permission_denied`
2. **Esconde bug** — `console.log` não alerta, build passa, incidente vira fantasma em produção
3. **Quebra controle de fluxo** — camadas acima tratam `null` como "não existe" mesmo quando era falha de infra
4. **Remove stack trace útil** — captura arrasa quem investiga depois
5. **Falsa sensação de robustez** — código "sobrevive" mas em estado inválido

## Quando capturar é legítimo

Existe ação útil concreta. Cinco casos:

### 1. Log com contexto e **rethrow**

```go
tx, err := db.Begin(ctx)
if err != nil {
    return fmt.Errorf("begin tx for subscription %s: %w", subID, err)
}
```

Enriquece com contexto (`subID`) e propaga. `%w` preserva a cadeia.

### 2. Compensação / rollback

```python
async with transaction() as tx:
    try:
        await create_order(tx)
        await charge_card(tx)
    except PaymentDeclined:
        await notify_user("payment failed")
        raise  # continua propagando
```

Capturou para disparar efeito colateral legítimo (notificar), rethrow para que a tx caia.

### 3. Transformação de erro (camada de abstração)

Mapear erro de infra → erro de domínio:

```rust
repo.find_user(id)
    .map_err(|e| match e {
        sqlx::Error::RowNotFound => DomainError::UserNotFound(id),
        sqlx::Error::Database(db_err) if db_err.constraint() == Some("uq_email") =>
            DomainError::EmailAlreadyTaken,
        other => DomainError::Infrastructure(other.into()),
    })
```

Camada de domínio **não** precisa saber `sqlx::Error`. Mapper converte.

### 4. Retry com backoff

```go
err := retry.Do(func() error {
    return httpClient.Do(req)
}, retry.Attempts(3), retry.Delay(100*time.Millisecond))
```

Só aplicável a erro **transiente** (5xx, timeout, connection refused). Nunca retry em `400`, `422`, `UniqueViolation`.

### 5. Boundary do programa (root do handler)

Ponto mais alto tem que transformar exceção em resposta:

```ts
// HTTP handler — último nível útil
try {
  const result = await useCase.execute(cmd);
  res.status(200).json(result);
} catch (e) {
  logger.error({ err: e, route: req.path, trace_id: req.id }, "handler error");
  if (e instanceof DomainError) return res.status(422).json(e.toJSON());
  return res.status(500).json({ code: "INTERNAL" });
}
```

**Uma única** captura de topo por request, não espalhada.

## Caso padrão: propagar

```go
// ✅ Go idiomático — propaga até camada que sabe lidar
user, err := repo.FindByID(ctx, id)
if err != nil {
    return nil, fmt.Errorf("FindByID %s: %w", id, err)
}
if err := user.Deactivate(); err != nil {
    return nil, fmt.Errorf("Deactivate: %w", err)
}
return user, repo.Save(ctx, user)
```

```rust
// ✅ Rust — ? propaga automaticamente
fn deactivate(id: UserId) -> Result<User, DomainError> {
    let mut user = repo.find_by_id(id)?;
    user.deactivate()?;
    repo.save(&user)?;
    Ok(user)
}
```

```ts
// ✅ TS — só throw, sem catch defensivo
async function deactivate(id: UserId): Promise<User> {
  const user = await repo.findById(id);
  user.deactivate();
  await repo.save(user);
  return user;
}
```

## Por stack

| Stack | Idioma |
|---|---|
| **Go** | `if err != nil { return fmt.Errorf("op: %w", err) }` — nunca `panic` exceto em bug realmente irrecuperável (init) |
| **Rust** | `?` propaga; `Result<T, E>` no tipo; `panic!` só em invariante de programa |
| **TS / JS** | `throw` ou padrão `Result<T, E>` explícito; `async/await` propaga automaticamente |
| **Python** | `raise` ou wrap com `raise X from e` para preservar cadeia |
| **Java / Kotlin** | checked exception no tipo ou sealed class `Result`; sem `catch(Exception)` genérico |
| **Elixir** | `{:error, reason}` idiomático; `with` encadeia early-return |

## Antipatterns comuns

### Catch-all silencioso

```python
# ❌ Proibido
try:
    do_work()
except Exception:
    pass
```

Bug disfarçado de "funcionou". Deve logar + rethrow ou não ser capturado.

### Pokemon exception handling ("gotta catch 'em all")

`catch(Exception)` ou `catch(Throwable)` em ponto qualquer. Captura até `OutOfMemoryError`, bug de programação e erro de domínio no mesmo balde. Programa segue em estado corrompido.

### Log e return null

```java
try { return fetchThing(); }
catch (Exception e) { log.warn("fail", e); return null; }
```

Chamador não distingue `null = não existe` de `null = falhou`. Use `Optional`, `Result`, ou deixe propagar.

### Catch para "silenciar warning" da IDE

Nunca. Se a IDE avisa de exceção checked, é a linguagem lembrando que essa condição precisa de decisão explícita.

### Retry cego

Retry sem classificar erro retriáveis vs não-retriáveis aplica retry em `UniqueViolation` e duplica registros. Sempre decidir: "este erro é transiente?"

## Heurística de review

Para cada `try`/`catch` num PR, perguntar:

- [ ] Há **ação concreta** no catch (log enriquecido + rethrow / compensação / transformação / retry / boundary)?
- [ ] Se sim, a ação está **correta** (tipo certo do erro, contexto adicionado)?
- [ ] Se não, **remover o try/catch**.

Regra mais simples: catch sem rethrow e sem efeito colateral observável = **delete**.

## Links

- [[_moc]]
- [[clean-code]]
- [[code-calisthenics]]
- [[do-dont-checklist]]
- [[../antipatterns/_moc]]
- [[../observability/structured-logging]]
