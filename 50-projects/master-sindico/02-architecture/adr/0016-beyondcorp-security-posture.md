---
title: ADR-0016 — BeyondCorp como security posture fundadora
type: adr
tags: [adr, security, beyondcorp, zero-trust, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0016 — BeyondCorp como security posture

## Status

accepted — 2026-04-23

## Contexto

Master Síndico hospeda dados sensíveis (CPFs de moradores, atas com decisões financeiras, vídeos-currículo) para milhares de condomínios. Modelo de segurança baseado em rede "confiável" interna é falho — herança direta de NIST SP 800-207 e BeyondCorp (Google, 2014).

## Decisão

**Zero-trust adaptado BeyondCorp** aplicado aos 7 tenets do NIST SP 800-207 ([[../../13-research/beyond-corp/_destilado]] §1.2):

1. **Todos os recursos têm controle próprio** — cada endpoint passa por ABAC.
2. **Comunicação segura sempre** — TLS 1.3 leste-oeste (mTLS M2+).
3. **Acesso per-session / per-request** — JWT curto ≤ 15 min + refresh bound a device.
4. **Política dinâmica** — ABAC matrix ordenada com scope + context.
5. **Monitora postura** — device fingerprint + trust tier.
6. **AuthN + AuthZ dinâmicos** — step-up MFA em ação crítica.
7. **Telemetria alimenta policy** — log → Grafana → alert → (M3+) risk-based auth.

### Implementação concreta M1

| Item | M1 |
|---|---|
| IdP centralizado | Zitadel OIDC + PKCE ([[0003-zitadel-oidc-provider]]) |
| ABAC engine | Matrix ordenada + Redis cache 5min ([[0012-abac-redis-cache]]) |
| mTLS leste-oeste | N/A (monolito M1); M2+ quando extrair serviço |
| Device fingerprint + trust tier | ✅ ([[0014-one-device-policy]]) |
| Step-up MFA ações críticas | ✅ |
| Audit trail append-only | ✅ timeline + `lgpd_logs` ([[0013-timeline-immutable]]) |
| WAF + rate limit edge | Cloudflare + rate limit app |
| PKCE obrigatório | ✅ |
| PII masked em logs | ✅ linter CI |

### Ações críticas exigem step-up

Endpoints marcados `auth_level: step_up` forçam reautenticação MFA mesmo com sessão válida:

- Aprovar pagamento (billing).
- Encerrar votação (assembly).
- Publicar minutes (assembly).
- Mudar síndico / transferir condomínio.
- Apagar ata — na prática nunca, imutável.
- Admin cross-tenant read/write.

### Tiers de acesso

- **Tier 1 (público)**: landing, cadastro, login.
- **Tier 2 (autenticado)**: leitura de timeline, votar, consumir vídeos.
- **Tier 3 (síndico/admin)**: criar anúncio, aprovar RFP, encerrar votação — **exige MFA ativa**.
- **Tier 4 (platform admin)**: cross-tenant — step-up MFA + session curta 1h.

## Consequências

### Positivas

- Defesa em profundidade: ABAC + RLS + tenant_id em query + audit trail.
- "Assume breach" — comprometimento de um nível (ex: session leak) não dá acesso full; ABAC, RLS, step-up seguram.
- Preparado para certificação SOC 2 / ISO 27001 M4+.

### Negativas

- Custo cognitivo: ABAC matrix exige disciplina em cada novo endpoint.
- Step-up MFA adiciona friction em ações críticas — UX cuidadosa mitiga (biometria, não "login de novo").
- Device fingerprint tem falsos positivos — tier confirmed + "lembrar dispositivo" mitigam.

## M2+ upgrades

- mTLS leste-oeste via service mesh (Linkerd preferido sobre Istio — overhead menor).
- SPIFFE/SPIRE para identidade de workload.
- Binary Authorization no pipeline CI.
- Atestação de plataforma mobile (Play Integrity / DeviceCheck).
- Audit WORM (S3 Object Lock ou QLDB).
- Continuous risk scoring (behavior anomaly → step-up dinâmico).

## Alternativas consideradas

1. **VPN + rede "interna"** — modelo tradicional; falha no assume breach.
2. **BeyondCorp puro (Access Proxy literal)** — GCP-only; adaptamos conceitos sem IAP físico.
3. **Sem zero-trust (perímetro clássico)** — descartado.

## Referências

- [[../../13-research/beyond-corp/_destilado]] (inteiro)
- [[../../13-research/google-arch/_destilado]] §3.2 (IAP)
- [[0012-abac-redis-cache]]
- [[0014-one-device-policy]]
- [[0013-timeline-immutable]]
- [[../observability]] §2.1.2 (PII masking)
- NIST SP 800-207
