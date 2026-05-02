---
title: Load tests — Master Síndico
type: guide
tags: [testing, load, k6, slo, sla, performance, master-sindico, v2]
source: 13-research/google-arch/_destilado.md + 13-research/netflix/_destilado.md
created: 2026-04-23
updated: 2026-04-23
aliases: [load-tests-master-sindico]
---

# Load tests — Master Síndico v2

Load tests com **k6** validando SLA/SLO derivados de [[../13-research/google-arch/_destilado]] §1. Cenários críticos: auth storm, assembleia 10k concurrent viewers (M3), upload vídeo burst, Connect Me burst. **Gating pré-GA**: não entra em prod sem validar SLO por BC.

> Herda [[test-strategy]] §2 e [[../STEERING]] §27. Referência de availability 99.5% / p95 < 500ms.

---

## 1. Filosofia load testing

- **Validar SLO**, não quebrar sistema.
- **Cenário realista** de tráfego (padrões de uso por persona).
- **Ambiente staging idêntico a prod** (mesma infra, escala reduzida).
- **Injetar progressivamente** (ramp-up) — não spike instantâneo (irreal).
- **Medir burn rate** do error budget (SRE).

---

## 2. SLO targets (derivados de SRE Workbook)

| BC / endpoint | SLI latência (p95) | SLI disponibilidade | Error budget 30d |
|---|---|---|---|
| Governança Institucional `POST /assembleias` | < 300ms | 99.5% | 3.6h |
| Connect Me `POST /connect-me` | < 500ms | 99.5% | 3.6h |
| Connect Me entrega msg | < 1s | 99.5% | — |
| Billing Stripe webhook | < 2s (inbound) | 99.9% | 43min |
| Assembly votar WebSocket | < 200ms | 99.5% | 3.6h |
| Content playback signed URL | < 100ms | 99.9% | 43min |
| Auth `POST /auth/callback` | < 800ms | 99.9% | 43min |
| Healthcheck `/healthz` | < 50ms | 99.99% | 4.3min |

**Upgrade pra 99.9%** após 2 ciclos estáveis em 99.5%.

---

## 3. Cenários canônicos k6

### 3.1 Auth storm (rate limit + protection)

Objetivo: validar `RateLimiter` `/auth/*` 20rpm burst 5 (INV-097, BR-062) protege contra DDoS low-rate.

```js
// tests/load/auth-storm.js
import http from 'k6/http';
import { check } from 'k6';

export const options = {
  scenarios: {
    storm: {
      executor: 'constant-arrival-rate',
      rate: 1000,
      timeUnit: '1s',
      duration: '2m',
      preAllocatedVUs: 500,
    },
  },
  thresholds: {
    'http_req_duration{endpoint:auth}': ['p(95)<800'],
    'checks{rate_limited}': ['rate>0.9'], // >90% rejeitadas (expected)
  },
};

export default function () {
  const res = http.post(`${__ENV.BASE_URL}/auth/callback`, JSON.stringify({ code: 'xxx' }), {
    tags: { endpoint: 'auth' },
    headers: { 'Content-Type': 'application/json' },
  });
  check(res, {
    'rate limited': (r) => r.status === 429,
  }, { rate_limited: true });
}
```

**Gate**: ≥ 90% de requests recebem 429 após burst; p95 latência < 800ms; memória estável (AP-006 fix).

---

### 3.2 Assembleia 10k concurrent viewers (M3)

Objetivo: validar WebSocket + LiveKit suportam 10k users ao vivo numa assembleia, com latência de voto < 200ms (BR-050).

```js
// tests/load/assembly-live.js
import { WebSocket } from 'k6/experimental/websockets';
import { check } from 'k6';

export const options = {
  scenarios: {
    attend: {
      executor: 'ramping-vus',
      startVUs: 0,
      stages: [
        { target: 1000, duration: '1m' },
        { target: 5000, duration: '2m' },
        { target: 10000, duration: '3m' },
        { target: 10000, duration: '5m' },   // sustain
      ],
    },
  },
  thresholds: {
    'ws_session_duration': ['p(95)<11000'],
    'vote_round_trip_ms': ['p(95)<200'],
  },
};

export default function () {
  const ws = new WebSocket(`${__ENV.WS_URL}/assemblies/${__ENV.ASSEMBLY_ID}/live`);
  ws.addEventListener('open', () => {
    // envia check-in
    ws.send(JSON.stringify({ type: 'checkin', user_id: `vu-${__VU}` }));
  });

  ws.addEventListener('message', (msg) => {
    const evt = JSON.parse(msg.data);
    if (evt.type === 'vote_opened') {
      const t0 = Date.now();
      ws.send(JSON.stringify({ type: 'vote', item_id: evt.item_id, option: 'yes' }));
      ws.addEventListener('message', (ack) => {
        if (JSON.parse(ack.data).type === 'vote_recorded') {
          const rtt = Date.now() - t0;
          check(rtt, { 'vote < 200ms': (v) => v < 200 }, { vote_round_trip_ms: true });
        }
      });
    }
  });
}
```

**Gate M3**: 10k users conectados sustém; p95 voto RTT < 200ms; 0 perda de voto; LiveKit mantém SFU estável.

---

### 3.3 Upload vídeo burst (Mux saga)

Objetivo: validar saga upload + Mux aguenta 100 uploads simultâneos (empresas publicando em semana de marketing).

```js
// tests/load/video-upload-burst.js
export const options = {
  scenarios: {
    burst: {
      executor: 'per-vu-iterations',
      vus: 100,
      iterations: 1,
      maxDuration: '5m',
    },
  },
  thresholds: {
    'http_req_duration{stage:create_upload}': ['p(95)<3000'],
    'http_req_failed': ['rate<0.01'],
  },
};

export default function () {
  const token = getUserToken(__VU);
  const create = http.post(`${__ENV.BASE_URL}/api/v1/videos/upload`, '{}', {
    headers: { Authorization: `Bearer ${token}` },
    tags: { stage: 'create_upload' },
  });
  check(create, { 'created 201': (r) => r.status === 201 });

  const uploadURL = create.json('upload_url');
  const file = open('fixtures/video-30mb.mp4', 'b');
  const put = http.put(uploadURL, file, { tags: { stage: 'mux_put' } });
  check(put, { 'mux 200': (r) => r.status === 200 });
}
```

**Gate**: create_upload p95 < 3s; error rate < 1%; saga compensation em falha (simular via fault injection).

---

### 3.4 Connect Me burst

```js
export const options = {
  scenarios: {
    burst: {
      executor: 'constant-arrival-rate',
      rate: 100,
      timeUnit: '1s',
      duration: '5m',
      preAllocatedVUs: 200,
    },
  },
  thresholds: {
    'http_req_duration': ['p(95)<500'],
    'http_req_failed': ['rate<0.01'],
  },
};

export default function () {
  const token = randomUserToken();
  http.post(`${__ENV.BASE_URL}/api/v1/connect-me`, JSON.stringify({ categoria: 'hidraulica', descricao: '...' }), {
    headers: { Authorization: `Bearer ${token}`, 'Content-Type': 'application/json' },
  });
}
```

**Gate**: p95 < 500ms em 500 writes/s; quota decrementa corretamente (INV-045); reputação não é impactada.

---

### 3.5 Webhook Stripe burst (idempotency)

```js
// Simula Stripe retry com mesmo event.id
export default function () {
  const body = genStripeEventPayload();
  const sig = signHMAC(body, __ENV.STRIPE_SECRET);
  for (let i = 0; i < 3; i++) {
    http.post(`${__ENV.BASE_URL}/webhooks/stripe`, body, {
      headers: { 'Stripe-Signature': sig, 'Content-Type': 'application/json' },
    });
  }
}
```

**Gate**: 200 em todas; DB registra **1** processamento do event.id (INV-015, idempotency ledger Redis).

---

## 4. Ambiente

- **Staging** com escala ~30% de prod — extrapolar metrics com headroom.
- **Ferramenta**: k6 OSS (CLI) + k6 Cloud para runs > 10k VUs.
- **Métricas**: k6 publica em InfluxDB → Grafana dashboard `load-<cenario>`.
- **Observability**: durante load, Grafana mostra p95 latência, error rate, queue lag NATS, CPU/mem pods.

---

## 5. Frequência

- **CI gate**: smoke load (100 VUs, 1min) em cada merge a `main`.
- **Pré-release**: full suite (todos cenários) em staging antes de deploy prod.
- **Mensal**: load reality check — replay de padrões reais.
- **M3+**: inclusão em chaos engineering (latency injection + load).

---

## 6. Análise de resultados

- **p50 / p95 / p99** latência por endpoint.
- **Error rate** por código (4xx cliente-side OK, 5xx bloqueia).
- **Throughput** (req/s) vs. VUs — identifica ponto de saturação.
- **Burn rate** do error budget — se 1h de load consome > 10% do budget 30d → não é realista ou sistema não suporta.
- **Saga compensation rate** — falhas na saga precisam compensar corretamente (telemetria).

---

## 7. Anti-patterns load

| Sintoma | Fix |
|---|---|
| Spike instantâneo 0 → 10k VUs | Ramp-up realista (stages) |
| Medir só p50 | p95/p99 são os que importam (user-facing) |
| Load contra prod | Staging identico ou canário |
| Tokens reais em scripts | Fixture de tokens gerados em setup |
| Sem thresholds `thresholds:` | k6 não falha automaticamente |
| Cenário sem warm-up | Cache frio distorce resultado |

---

## 8. Checklist load

- [ ] SLO target definido pro endpoint?
- [ ] Ramp-up realista?
- [ ] Thresholds k6 configurados?
- [ ] Error rate < 1%?
- [ ] p95 dentro do SLO?
- [ ] Burn rate error budget aceitável?
- [ ] Grafana dashboard nomeado `load-<cenario>`?
- [ ] Run em staging isolado?

---

## 9. Links

- [[_moc]]
- [[test-strategy]]
- [[security]]
- [[../02-architecture/observability]]
- [[../13-research/google-arch/_destilado]]
- [[../13-research/netflix/_destilado]]
- [[../STEERING]]
