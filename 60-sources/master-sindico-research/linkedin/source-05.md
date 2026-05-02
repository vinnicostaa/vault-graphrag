---
title: source-05 — Defending Against Abuse at LinkedIn's Scale
type: source
tags: [research, linkedin, trust-safety, abuse, anti-spam, rate-limit]
source: https://engineering.linkedin.com/blog/2018/12/defending-against-abuse-at-linkedins-scale
created: 2026-04-23
updated: 2026-04-23
aliases: [linkedin-anti-abuse, casal-framework]
---

# source-05 — Defending Against Abuse at LinkedIn's Scale

**URL (atual, pós-redirect)**: https://www.linkedin.com/blog/engineering/trust-and-safety/defending-against-abuse-at-linkedins-scale
**Data**: 2018-12
**Fonte primária**: sim (LinkedIn Engineering — Trust & Safety)

## Por que essa fonte importa

Base de referência pra arquitetura anti-abuse. Contém o pattern **"unified content filtering library"** diretamente aplicável ao harassment report (Sprint 4) e anti-fake de Connect Me.

## Pontos extraídos

### Escala
- 4M transações/s passam por abuse detection.
- 5B counters transientes.
- 200K QPS writes / 3M QPS reads só em counters de velocity.
- 590M+ members em 200+ países.
- 2 novos members/segundo → barra de fake account screening contínua.

### Quatro obstáculos principais
1. **Scoring latency** — abuse detection tem que ser invisível ao produto; síncrono quando necessário, async quando possível.
2. **Downstream latency/errors** — se dependência falha: (i) falha transação, (ii) re-scoring async, (iii) default graceful. Decidido **por endpoint**, não global.
3. **Velocity/frequency** — hundreds de counters por action/entity em massive scale.
4. **Protection by default** — defesas em todos pontos de entrada, thousands de developer teams reusam lib central.

### Camadas de defesa
- **Edge layer** — proteções básicas usando body data do request.
- **Unified content filtering** — biblioteca central que serviços integram pra proteger contra spam, malware, inappropriate content, malicious URLs.
- **Custom models pra endpoints de alto risco** — registro, login, payment, content creation. Modelos dedicados pra vetores conhecidos.
- **Self-serve throttling** — velocity counters que serviços consomem pra rate limit próprio.

### Técnicas de detecção
- ML learned models + regras estatísticas.
- **Adversarial modeling**: "pensar como atacante", monitorar patterns que desviam significativamente do normal.
- Graceful degradation explícita por endpoint.

## Aplicabilidade master-sindico

Detalhada em [[_destilado#4-trust--safety--master-sindico-tem-harassment-report-sprint-4-lições-anti-fake-parciais]].

- **Unified content filtering library** (`internal/safety/content`) — decisão D-LKD-003. Reusável por HarassmentReport, AssemblyComment, ConnectMeMessage, PostBody.
- **Counter-based velocity** — Redis + Postgres (outbox pra persistência auditável) cobre escala master-sindico com sobras. Detectar "morador posta 20 reclamações em 1h".
- **High-risk endpoints**: lista pra master-sindico = registro de empresa (Connect Me), criação de conta síndico, aprovação de contrato, submissão de denúncia. Cada um com rate limit próprio + regra custom.
- **Graceful degradation por endpoint** — adotar como padrão explícito nos ADRs de handlers críticos. Se checagem anti-abuse falha, cada endpoint decide fallback (falha rígida em pagamento, silencioso em post).

## Lacunas que essa fonte expõe no master-sindico

Registrar como A-### em AUDIT se não estiverem no escopo M1:

- **A-LKD-TS-001**: Device fingerprint + IP velocity no cadastro de empresa Connect Me (CNPJ Receita sozinho é insuficiente).
- **A-LKD-TS-002**: Behavioral signals pós-cadastro (empresa nova manda 50 propostas/dia → shadow ban).
- **A-LKD-TS-003**: Graceful degradation policy documentada por endpoint crítico.
- **A-LKD-TS-004**: Auditoria LGPD de toda ação automática (log imutável + recurso humano).

## Quando releer

- Sprint 4 (harassment report) antes de implementar.
- Quando Connect Me abrir cadastro público de empresa.
- Quando surgir evento de fraude real (post-mortem).

## Vizinhos

- [[_destilado]]
- [[source-06]] — Keeping LinkedIn Professional (complementar: perfis inapropriados)
- [[../../08-security/beyond-corp-references]]
