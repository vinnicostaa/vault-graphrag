[FFV Academy](../index.html)/Engenharia de Software

🏗️

Hub · Engenharia

# DevOps, engenharia moderna, distribuídos e SRE — sair do coder, virar engenheiro de sistemas.

Oito trilhas para virar engenheiro de sistemas de verdade: Docker + Kubernetes + CI/CD profissional, engenharia na era dos agents, API Design & Contratos, Security Engineering (OAuth2/OIDC, OWASP, pentest), Testing Engineering (TDD, property-based, mutation, Pact), Accessibility & Inclusive Engineering (WCAG, ARIA, screen readers), sistemas distribuídos (CAP, Raft, sagas, event sourcing) e observabilidade + SRE (OpenTelemetry, SLOs, incident response).

21

trilhas

159

artigos

9850

XP total

37h

leitura

TRILHAS DESTE HUB

## 21 caminhos curados — leia em ordem ou salte.

[](../devops-containers/index.html)

TRILHA 01

📦

### DevOps & Containers

Docker e Kubernetes do zero ao avançado — os dois pilares de toda infra moderna, explicados para durar.

- Docker Completo: do zero ao production-ready
- Kubernetes Completo: do Pod ao cluster de produção
- GitHub Actions: CI/CD profissional do zero
- Jenkins Pipelines: o CI/CD da era enterprise
- \+ 3 artigos

7 ARTIGOS · 640 XPAbrir→

[](../engenharia-software/index.html)

TRILHA 02

🏗️

### Engenharia de Software Moderna

SDD, gerenciamento e criação de agents, testes profissionais, segurança real e arquitetura — deixar de ser coder e virar engenheiro de software de verdade.

- Engenheiro vs Coder: o que mudou na era dos agents
- Spec-Driven Development (SDD): a nova espinha dorsal
- Gerenciando Agents: orquestração, contexto e custo
- Criando Agents Customizados: do subagent ao MCP
- \+ 4 artigos

8 ARTIGOS · 675 XPAbrir→

[](../api-design/index.html)

TRILHA 03

🔌

### API Design & Contratos

REST maduro (Richardson levels, idempotência), versionamento sem dor, GraphQL quando faz sentido (N+1, dataloader), gRPC + protobuf, OpenAPI como contrato vivo, paginação profissional, idempotency keys e webhooks, rate limiting. O que diferencia uma API amadora de uma que sustenta produto.

- REST maduro: Richardson levels, idempotência e HATEOAS (raramente)
- Versionamento sem dor: URL, header, sunset e estratégia de migração
- GraphQL quando faz sentido: N+1, DataLoader e federation
- gRPC + Protobuf: RPC tipado e streaming bidirecional
- \+ 5 artigos

9 ARTIGOS · 485 XPAbrir→

[](../security-engineering/index.html)

TRILHA 04

🛡️

### Security Engineering

Segurança como disciplina própria: threat modeling STRIDE, authn vs authz, OAuth2/OIDC do zero, JWT/Paseto/sessions, password hashing moderno (argon2), OWASP Top 10 com código, secrets management (Vault/SOPS), supply chain (SBOM, sigstore), Zero Trust + mTLS, capstone pentest em app próprio.

- Threat modeling com STRIDE: de onde vêm os ataques
- Authn vs Authz: a diferença e as armadilhas
- OAuth2 e OIDC do zero: fluxos e PKCE
- JWT, Paseto ou sessions: quando cada um
- \+ 6 artigos

10 ARTIGOS · 595 XPAbrir→

[](../testing-engineering/index.html)

TRILHA 05

🧪

### Testing Engineering

Testes como disciplina: test pyramid/trophy/diamond, TDD e BDD quando funcionam, test doubles rigorosos (mock/stub/fake/spy), property-based com fast-check, mutation testing com Stryker, integration vs contract vs e2e, performance testing com k6. Capstone: harness de testes completo.

- Test pyramid, trophy ou diamond: formato certo pro seu time
- TDD e BDD: quando ajudam e quando viram cerimônia
- Test doubles: mock, stub, fake, spy, dummy (Meszaros)
- Property-based testing com fast-check: achar bugs em edge cases
- \+ 4 artigos

8 ARTIGOS · 455 XPAbrir→

[](../acessibilidade/index.html)

TRILHA 06

♿

### Accessibility & Inclusive Engineering

A11y como disciplina, não afterthought: WCAG 2.2/3 e a realidade legal (EU Accessibility Act 2025, ADA lawsuits), semantic HTML como alicerce, ARIA certo (ou a ausência dele), keyboard + focus management, screen readers na prática (NVDA/VoiceOver), automated testing com axe, capstone remediando app real.

- Accessibility por que agora: WCAG, POUR e legal landscape
- Semantic HTML: o básico que todo mundo ignora
- ARIA: quando usar, quando NÃO usar
- Keyboard navigation e focus management: sem mouse também
- \+ 3 artigos

7 ARTIGOS · 375 XPAbrir→

[](../sistemas-distribuidos/index.html)

TRILHA 07

🧭

### Sistemas Distribuídos

CAP, consensus, idempotência, sagas, event sourcing — a base técnica que separa "funciona no localhost" de "funciona com 3 nós e 1000 req/s".

- CAP e PACELC: o teorema que define toda arquitetura distribuída
- Modelos de Consistência: strong, eventual, causal, read-your-writes
- Consensus e Raft: como nós discordam e chegam a acordo
- Idempotência e Retries: o antídoto pra rede que quebra
- \+ 5 artigos

9 ARTIGOS · 755 XPAbrir→

[](../observabilidade-sre/index.html)

TRILHA 08

🔭

### Observabilidade & SRE

Métricas RED/USE, OpenTelemetry, SLOs, error budgets, incident response — o que separa "fazer deploy" de operar sistema em produção.

- Observability: os 3 pilares (logs, métricas, traces) e por que não basta
- Métricas RED e USE: os frameworks que cobrem 90% dos casos
- OpenTelemetry end-to-end: instrumentação app → backend
- Logs Estruturados: JSON, correlation IDs e levels com propósito
- \+ 4 artigos

8 ARTIGOS · 635 XPAbrir→

[](../tech-leadership/index.html)

TRILHA 09

🎯

### Tech Leadership & Staff Engineering

O topo da carreira técnica: ADRs e decisões reversíveis vs irreversíveis, mentoria e code review como ferramenta pedagógica, estimativas sem mentir, lidar com legacy, carreira técnica vs gestão, comunicação com executivos, capstone de ADR completo real.

- ADRs: decisões reversíveis vs irreversíveis
- Mentoria técnica: multiplicar sem gargalar
- Code review como ferramenta pedagógica
- Estimativas sem mentir: \# Hofstadter + cone of uncertainty
- \+ 3 artigos

7 ARTIGOS · 400 XPAbrir→

[](../dx-productivity/index.html)

TRILHA 10

🛠️

### DX & Developer Productivity

O chão invisível da produtividade: shell (zsh/bash sérios), dotfiles reproduzíveis (chezmoi), devcontainers + codespaces, Makefiles e task runners modernos, editor produtividade (VS Code + Neovim), terminal multiplexers (tmux, zellij). Capstone: setup de dev do zero em 20min.

- Shell zsh/bash sérios
- Dotfiles reproduzíveis: chezmoi + GNU Stow
- Devcontainers + Codespaces: dev env efêmero
- Makefiles e task runners: make, just, task
- \+ 3 artigos

7 ARTIGOS · 380 XPAbrir→

[](../system-design-interview/index.html)

TRILHA 11

🧩

### System Design Interview Prep

System design como disciplina de interview e de carreira: framework estruturado, back-of-envelope, cases canônicos (URL shortener, Twitter feed, rate limiter, chat, search, notification, distributed cache) e templates de whiteboard. Nível sênior/staff.

- Framework de system design interview
- Back-of-envelope: cálculos que convencem
- Case: URL shortener
- Case: Twitter feed / timeline
- \+ 6 artigos

10 ARTIGOS · 580 XPAbrir→

[](../technical-writing/index.html)

TRILHA 12

✍️

### Technical Writing & RFCs

Escrita técnica como superpoder de sênior: design docs, RFCs, ADRs, postmortems blameless, READMEs editoriais, docs de API vivas. Templates reais (Google/Meta/Stripe). Escrever bem = pensar bem e escalar decisões.

- Por que escrita técnica é alavanca sênior
- Design docs: estrutura que funciona
- RFCs: quando escrever e como
- ADRs: decisões arquiteturais registradas
- \+ 3 artigos

7 ARTIGOS · 360 XPAbrir→

[](../graphql/index.html)

TRILHA 13

🔺

### GraphQL completo

GraphQL além do hype: schema design disciplinado, resolvers sem N+1, DataLoader, Apollo server/client em produção, subscriptions real-time, federation para escalar com múltiplos times. Quando GraphQL bate REST e quando não.

- GraphQL mental model: quando vale a pena
- Schema design sério
- Resolvers + DataLoader (evitando N+1)
- Apollo server + client em produção
- \+ 3 artigos

7 ARTIGOS · 400 XPAbrir→

[](../platform-engineering/index.html)

TRILHA 14

🏗️

### Platform Engineering & IDPs

Platform engineering real: Internal Developer Platforms (IDPs), Backstage como catalog, golden paths, self-service infra, platform as product mindset, métricas DORA/SPACE. Distinto de DevOps: time dedicado, stakeholder = dev interno.

- Platform Engineering vs DevOps
- Backstage: developer portal da Spotify
- Golden paths / paved road
- Self-service infra via UI/API
- \+ 3 artigos

7 ARTIGOS · 390 XPAbrir→

[](../performance-engineering/index.html)

TRILHA 15

⚡

### Performance Engineering

Perf engineering como disciplina: flamegraphs, eBPF, profilers por linguagem, cache analysis, lock contention, I/O bottlenecks. Método científico: hipótese → medir → otimizar → repetir. Sem chutes.

- Perf engineering: método
- Flamegraphs em produção
- eBPF / bcc / bpftrace básico
- Profilers por linguagem (comparado)
- \+ 3 artigos

7 ARTIGOS · 405 XPAbrir→

[](../cryptography-applied/index.html)

TRILHA 16

🔐

### Cryptography Applied

Cripto aplicada (não acadêmica): PKI + X.509, TLS 1.3 deep, key management (KMS/Vault/HSM), JWT vs PASETO vs sessions, mTLS e zero-trust, cert rotation. Foco em usar cripto corretamente, não reinventar.

- Cripto aplicada: mental model
- PKI + X.509 certificates
- TLS 1.3 deep: handshake, 0-RTT, cipher suites
- Key management: KMS, Vault, HSM
- \+ 3 artigos

7 ARTIGOS · 410 XPAbrir→

[](../kafka-streaming/index.html)

TRILHA 17

🌊

### Event Streaming / Kafka Depth

Kafka sério em 2026: arquitetura (brokers, partitions, replication), schema registry (Avro/Protobuf), CDC com Debezium, exactly-once semantics, outbox pattern, Kafka Streams + ksqlDB, alternativas (Redpanda, Pulsar). Cost-aware.

- Kafka em produção: arquitetura profunda
- Schema registry: Avro, Protobuf, JSON Schema
- CDC com Debezium avançado
- Exactly-once semantics
- \+ 3 artigos

7 ARTIGOS · 415 XPAbrir→

[](../real-time-systems/index.html)

TRILHA 18

📡

### Real-time Systems

Real-time moderno: WebSockets em produção, SSE, WebRTC (voice/video/data), CRDTs para colaboração (Yjs/Automerge), presence systems, LiveKit/mediasoup para SFU. Quando WebSocket vs SSE vs polling.

- WebSockets em produção
- Server-Sent Events (SSE)
- WebRTC: voice, video, data channels
- CRDTs: Yjs, Automerge
- \+ 3 artigos

7 ARTIGOS · 410 XPAbrir→

[](../product-engineering/index.html)

TRILHA 19

🧪

### Product Engineering & Experimentation

Engineering que pensa como produto: feature flags (GrowthBook, Unleash), A/B testing rigoroso, CUPED para reduzir variance, guardrails, product analytics (PostHog, Mixpanel). Decisão baseada em dado estatístico, não sentimento.

- Product engineering: o mindset
- Feature flags: GrowthBook + Unleash
- A/B testing estatisticamente rigoroso
- CUPED + variance reduction
- \+ 3 artigos

7 ARTIGOS · 390 XPAbrir→

[](../career-engineering/index.html)

TRILHA 20

🚀

### Career Engineering

Carreira tech como sistema, não sorte: resume que converte, LinkedIn sem hype, behavioral interview com frameworks, negotiation (sem deixar dinheiro na mesa), promo docs que avançam, portfolio técnico público. Baseado em playbooks reais (Levels.fyi, Gergely Orosz, haseeb).

- Mental model: junior → sr → staff
- Resume tech que converte
- LinkedIn dev prático (sem hype)
- Behavioral interview: STAR + brag doc
- \+ 3 artigos

7 ARTIGOS · 360 XPAbrir→

[](../chaos-engineering/index.html)

TRILHA 21

🌪️

### Chaos Engineering

Chaos engineering como disciplina (não gambiarra): princípios (steady-state hypothesis, minimize blast radius), Chaos Monkey, Gremlin, LitmusChaos, game days estruturados, fault injection controlada. Para sistemas que realmente resistem.

- Chaos engineering: os princípios
- Chaos Monkey, Simian Army, Gremlin
- LitmusChaos no Kubernetes
- Game days estruturados
- \+ 2 artigos

6 ARTIGOS · 335 XPAbrir→

EXPLORAR OUTROS HUBS

## Siga por outro tema.

[](../ia/index.html)

🧠

Inteligência Artificial

11 trilhas

→

[](../aws/index.html)

☁️

AWS Cloud

5 trilhas

→

[](../claude-anthropic/index.html)

⊕

Claude & Anthropic

3 trilhas

→

[](../fundamentos/index.html)

🧱

Fundamentos Técnicos

4 trilhas

→

[](../programacao/index.html)

💻

Programação & Algoritmos

10 trilhas

→

[](../dados/index.html)

🏭

Dados & Analytics Engineering

3 trilhas

→

[](../construcao/index.html)

🏗️

Construção & Clientes

6 trilhas

→
