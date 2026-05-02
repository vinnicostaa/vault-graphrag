---
title: Structured Logging — logs como dados
type: concept
tags: [observability, logging, structured-logs, audit]
created: 2026-04-24
updated: 2026-04-24
aliases: [Structured logging, JSON logs, Logs estruturados]
---

# Structured Logging — logs como dados

Log de sistema moderno **não é texto livre**. É um stream de eventos estruturados, indexáveis, filtráveis e agregáveis. Texto solto com `printf` é patrimônio perdido: impossível de consultar, caro de processar, inutilizável em incidente.

## Unstructured vs Structured

```
# ❌ Unstructured — só humano lê, busca é grep frágil
2026-04-24 10:32:11 INFO user 42 logged in from IP 10.0.0.5 (session=abc123)

# ✅ Structured JSON — consultável
{"ts":"2026-04-24T10:32:11Z","level":"info","event":"user_login",
 "user_id":"42","ip":"10.0.0.5","session_id":"abc123",
 "trace_id":"0af7651916cd43dd8448eb211c80319c"}

# ✅ logfmt — meio-termo legível + parseável
ts=2026-04-24T10:32:11Z level=info event=user_login user_id=42 ip=10.0.0.5
```

Formato canônico moderno: **JSON lines** (`.jsonl`) — uma linha por evento, um JSON por linha. Facilita ingestão em Loki, Elastic, BigQuery, Datadog.

## Níveis — quando usar cada

| Nível | Uso | Volume esperado |
|---|---|---|
| **trace** | Debug hiper-detalhado (entrada/saída de função) | Off em produção |
| **debug** | Fluxo interessante durante desenvolvimento | Off em produção estável |
| **info** | Evento de negócio normal (login, pedido criado) | Alto |
| **warn** | Algo inesperado mas recuperável (retry, fallback acionado) | Médio |
| **error** | Falha que exige ação ou impacta cliente | Baixo — cada um merece atenção |
| **fatal** | Processo vai terminar/reiniciar | Raríssimo |

Regra dura: **`error` não é rotineiro**. Se todo request "saudável" loga um `error`, o nível está errado ou há bug silencioso. Error rate deve **casar** com a taxa de 5xx/falhas visíveis.

## Context propagation — o que todo log deve carregar

Log sem contexto é inútil em incidente distribuído. Todo evento deve carregar:

```json
{
  "trace_id":       "w3c-traceparent-hex",   // une logs de todos os serviços da request
  "span_id":        "id do span local",
  "request_id":     "id gerado no edge",
  "user_id":        "quem fez (se autenticado)",
  "tenant_id":      "isolamento multi-tenant",
  "session_id":     "sessão opcional",
  "service":        "nome do serviço que emitiu",
  "version":        "build / commit sha"
}
```

Propagação via **context** da linguagem:
- Go — `context.Context` + logger enriquecido no middleware
- Node — AsyncLocalStorage / `pino` child loggers
- Python — `contextvars` + `structlog.contextvars`
- Rust — `tracing` crate com `Span` escopado

Middleware HTTP injeta trace_id/request_id **uma vez**; downstream só propaga.

## PII safety — mascarar antes de escrever

Log é artefato de longa retenção com acesso amplo. PII em log viola LGPD/GDPR:

- **Nunca logar em claro**: CPF, CNPJ, senha, cartão, CVV, token, email, telefone, endereço completo, data de nascimento
- **Mascarar**: `CPF: ***.***.***-12`, `email: j***@***.com`
- **Hash determinístico** quando precisa correlacionar (SHA-256 + salt fixo por ambiente)
- **Custom marshaler** no DTO: `json.Marshaler`, `MarshalLogObject` (zerolog), `repr` (structlog)
- **Allowlist** em vez de denylist — logar campos explicitamente seguros, nunca o objeto inteiro

Scanner de CI (regex de CPF/CNPJ/email) em arquivos de log fixture + revisão de PR vazam frequentemente — automatizar.

Ver [[../security/beyond-corp]] e [[../security/owasp-top-10]] sobre exposure em logs.

## Ferramentas por stack

| Stack | Lib | Formato nativo |
|---|---|---|
| Go stdlib 1.21+ | **log/slog** | JSON / text |
| Go | **zerolog**, **zap** | JSON rápido (alloc-free) |
| Node.js | **pino**, **winston** | JSON |
| Python | **structlog**, **loguru** | JSON / logfmt |
| Rust | **tracing** + `tracing-subscriber` | JSON |
| JVM | **logback** + logstash-encoder, **log4j2** | JSON |
| .NET | **Serilog** | JSON |

`slog` (Go stdlib) e `tracing` (Rust) integram nativamente com OpenTelemetry — preferir quando possível.

## Sampling em alto volume

Quando um serviço emite 100k req/s, logar tudo é caro. Estratégias:

- **Head sampling** — decisão na entrada (1 em cada N requests). Simples, perde contexto de erro raro.
- **Tail sampling** (OTel Collector) — bufferiza e decide ao fim do trace: se houve erro, manter 100%; se foi ok, 1%. Preserva anomalias.
- **Probability** — por rota/tipo. Endpoints críticos 100%, saúde/ping 0.1%.
- **Rate limit por chave** — `max 10 logs/s por request_id` evita loop que despeja milhões de linhas.

Logs de **erro** sempre 100% — nunca samplar.

## Logs, Metrics, Traces — os 3 pilares e quando usar cada

| Pilar | Pergunta que responde | Volume | Custo |
|---|---|---|---|
| **Logs** | "O que exatamente aconteceu neste request?" | Alto | Alto (retenção, indexação) |
| **Metrics** | "Quanto, quantos, quão rápido no agregado?" | Baixo (séries fixas) | Baixo |
| **Traces** | "Por onde passou essa request através dos serviços?" | Médio | Médio |

**Regra prática**:
- Alertar em **metrics** (RED/USE/SLO — ver [[red-use-sli-slo]])
- Diagnosticar em **traces** (onde latência sumiu?)
- Confirmar com **logs** (o que exatamente aconteceu naquele span?)

Ter log para **cada** evento é desnecessário se metrics já respondem. Log para evento **de negócio** (quem fez o quê) e para erro.

## Retenção por tipo

| Tipo | Retenção típica | Racional |
|---|---|---|
| Audit / compliance (LGPD, financial) | **5+ anos** | Exigência legal; custa armazenar, inegociável |
| Erro / exceção | 90-180 dias | Debugging pós-fato, investigação |
| Info geral | 7-30 dias | Debug recente, hot storage |
| Debug / trace | 1-7 dias | Troubleshooting imediato |
| Metrics agregadas | 1-2+ anos (cold) | Análise de tendência, capacity planning |

Arquivar logs frios em object storage barato (S3, GCS) com lifecycle policy. Não manter tudo em índice quente.

## Antipatterns

- **Log com interpolação de string**: `log.Info("user " + id + " did X")` — perde estrutura, erra ao parsear. Sempre `log.Info(msg, "user_id", id)` ou equivalente.
- **Swallow + log**: `catch(e) { log.error(e); }` sem rethrow — esconde bug, cria 5xx silencioso.
- **Logar objeto inteiro** com PII sem custom marshaler.
- **Log no hot loop** sem sample — 10Mlinhas em 5 minutos travam o coletor.
- **Nível global `debug` em prod** — custo de disco + vazamento de PII.
- **Timestamp local** em vez de UTC ISO-8601 — incompatível com agregadores.

## Links

- [[_moc]]
- [[red-use-sli-slo]]
- [[../security/beyond-corp]]
- [[../security/owasp-top-10]]
- [[../_moc]]
