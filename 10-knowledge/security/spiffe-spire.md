---
title: SPIFFE & SPIRE — Workload identity via mTLS attestation
type: concept
tags:
  - security
  - spiffe
  - spire
  - workload-identity
  - mtls
  - zero-trust
  - cncf
doc-oficial: https://spiffe.io
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - SPIFFE
  - SPIRE
  - Workload Identity
---

# SPIFFE & SPIRE — Workload Identity

**O que é**: padrão aberto (CNCF graduated 2022) para dar identidade criptográfica a workloads (pods, containers, VMs, processes) sem senhas hardcoded. **SPIFFE** (Secure Production Identity Framework For Everyone) define formato; **SPIRE** é a implementação de referência (servidor + agentes).

> Resolve o "secret zero problem": como workload A sabe que é "A" e prova para workload B — sem hardcoded secret.

## Conceitos core

### SPIFFE ID

URI único por workload:
```
spiffe://trust-domain.example/ns/prod/sa/billing-service
```

- `trust-domain.example` — domínio de confiança (similar a "DNS zone" para identities).
- Path específico (namespace, service account) define workload.

### SVID — SPIFFE Verifiable Identity Document

Assinatura criptográfica que prova identidade:
- **X.509-SVID**: certificado X.509 com SPIFFE ID como SAN URI. Usado para **mTLS**.
- **JWT-SVID**: JWT com `sub = spiffe://...`. Usado para HTTP APIs stateless.

Short-lived (minutos a horas). Rotados automaticamente pelo SPIRE.

### Trust bundle

Conjunto de public keys/CAs que um trust domain confia. Distribuído aos workloads.

## Arquitetura SPIRE

```
┌──────────────────────────────────────────────────────┐
│  SPIRE Server                                        │
│  (HA cluster; backing by Postgres/MySQL/SQLite)      │
│  ┌──────────────────────────────────────────────┐    │
│  │ CA (intermediate) — signs SVIDs              │    │
│  │ Registration API — admin configures trust    │    │
│  │ Node attestation validator                   │    │
│  │ Workload attestation validator               │    │
│  └──────────────────────────────────────────────┘    │
└──────────────────────┬───────────────────────────────┘
                       │ mTLS
                       ▼
┌──────────────────────────────────────────────────────┐
│  SPIRE Agent (DaemonSet em k8s; por node)            │
│  ┌──────────────────────────────────────────────┐    │
│  │ Node attestation (k8s, AWS IID, GCP, EC2)     │    │
│  │ Workload API (Unix socket / gRPC)             │    │
│  │ SVID rotation                                  │    │
│  └──────────────────────────────────────────────┘    │
└──────────────────────┬───────────────────────────────┘
                       │ Unix socket
                       ▼
┌──────────────────────────────────────────────────────┐
│  Workload (pod/container)                            │
│  - Fetch SVID via SPIFFE Workload API                 │
│  - Use SVID para mTLS a outros workloads              │
└──────────────────────────────────────────────────────┘
```

**Attestation chain**:
1. SPIRE Server confia platform (K8s, AWS, GCP) para provar que node é legítimo (node attestation).
2. SPIRE Agent, rodando no node, chama Workload API pelo pod (Unix socket).
3. Agent valida workload via **workload attestation** (PID + namespace + labels + K8s SA + image hash).
4. Se matches selector em rule de registration, emite SVID.
5. Workload usa SVID para comunicar com outros workloads via mTLS.

## Node + Workload attestation — "como sei que é você?"

### Node attestation (plugins)

- `k8s_psat`: K8s projected service account token.
- `aws_iid`: AWS instance identity document.
- `gcp_iit`: GCP instance identity token.
- `join_token`: token manual (menos seguro).

### Workload attestation

- `k8s`: namespace + service account + pod label + image hash.
- `unix`: UID/GID + process binary path + hash.
- `docker`: container label + image.

## Padrão canônico — integração

### No código

Usando Go SPIFFE library:
```go
import (
  "github.com/spiffe/go-spiffe/v2/workloadapi"
  "github.com/spiffe/go-spiffe/v2/svid/x509svid"
)

// Fetch SVID
source, err := workloadapi.NewX509Source(context.Background())
if err != nil { log.Fatal(err) }
defer source.Close()

svid, err := source.GetX509SVID()
// svid.Certificates, svid.PrivateKey
```

### Servidor mTLS

```go
import "github.com/spiffe/go-spiffe/v2/spiffetls/tlsconfig"

// Accept qualquer SVID do trust domain
tlsConfig := tlsconfig.MTLSServerConfig(source, source, tlsconfig.AuthorizeAny())

// OU: accept só IDs específicos
authorizer := tlsconfig.AuthorizeID(spiffeid.Must("example.org", "billing"))
tlsConfig := tlsconfig.MTLSServerConfig(source, source, authorizer)

server := &http.Server{ TLSConfig: tlsConfig, Handler: mux }
```

### Cliente mTLS

```go
tlsConfig := tlsconfig.MTLSClientConfig(source, source, authorizer)
client := &http.Client{ Transport: &http.Transport{ TLSClientConfig: tlsConfig } }
```

Zero senha, zero API key, zero secret hardcoded.

## Quando usar

- **Service mesh** (Istio usa SPIFFE internamente para mTLS).
- **Microservices internos** com mTLS.
- **Cross-cloud identity** (workload em AWS precisa autenticar em GCP service).
- **Zero Trust networks** — identidade por workload, não por IP.
- **Substituir API keys estáticas** entre services.

## Quando **não**

- App pequeno, monolito — overhead alto.
- Sem orchestrator (bare metal sem k8s/nomad) — attestation difícil.
- Latência ultra-baixa — rotação + mTLS handshake adiciona ~ms.

## Integrações

| Produto | Uso de SPIFFE/SPIRE |
|---|---|
| **Istio** | Interno (mTLS entre proxies Envoy) — pode usar SPIRE como CA |
| **Linkerd** | Identidade própria inspirada em SPIFFE |
| **HashiCorp Consul** | Connect usa SPIFFE identities |
| **AWS App Mesh** | Não nativo; combinado via sidecar |
| **Kuma** | SPIFFE-compatible |
| **Cilium (service mesh)** | Suporta SPIFFE |
| **Tornjak** | UI management para SPIRE |

## Anti-patterns

1. **Hardcoded API key M2M** — SPIFFE existe para eliminar. Migre.
2. **Certificado longo-prazo** — SVIDs são curtos; não bypass com cert de 1 ano.
3. **Workload attestation laxa** (só pod name) — atacante com exec no pod cria workload malicioso. Valide image hash + labels.
4. **SPIRE Server sem HA** — single point of failure para toda comunicação interna. Sempre HA.
5. **Misturar authz no mTLS** — mTLS prova identidade; authorization vai em [[opa-rego|OPA]]/[[zanzibar-openfga]].
6. **Trust domain único gigante** — separe por ambiente (prod/staging) ou região.
7. **Join token manual em prod** — use node attestation real (k8s_psat, aws_iid).

## Alternativas / complementares

- **AWS IAM Roles for Service Accounts (IRSA)** em EKS — workload identity AWS-nativo.
- **GCP Workload Identity Federation** — conta externa → GCP sem keys estáticas.
- **HashiCorp Vault** — authentication methods + secret rotation (complementar).
- **Cert-manager + Venafi** — tradicionais para certs mas sem workload attestation automática.
- **Azure Managed Identity** — Azure-nativo.

SPIFFE ganha em multi-cloud e K8s-first.

## Como grandes empresas usam

- **Bloomberg**: SPIRE em produção para workload identity interna.
- **ByteDance**: adotou SPIRE.
- **Uber**: SPIFFE para service mesh interno.
- **Square (Block)**: SPIFFE para mTLS microservices.
- **Intuit**: SPIRE como parte de Zero Trust.

## Relações

- **Princípio**: [[least-privilege]] (workload sem senha hardcoded), [[beyond-corp]] (Zero Trust para workloads)
- **Complementa**: [[iam]] (identity lifecycle), [[secrets-management]] (substitui parcialmente static secrets)
- **Integra com**: [[opa-rego]] (authz sobre identidades SPIFFE), service mesh (Istio)
- **Alternativa**: AWS IAM, GCP Workload Identity Federation (cloud-specific)

## Fontes

- [SPIFFE docs](https://spiffe.io/docs/latest/) (consultada 2026-04-24)
- [SPIRE docs](https://spiffe.io/docs/latest/spire-about/) (consultada 2026-04-24)
- [CNCF SPIFFE](https://www.cncf.io/projects/spiffe/) (consultada 2026-04-24)
- [Solving the Bottom Turtle (book — Rifflock, Evenson, Feldman)](https://spiffe.io/book/) (consultada 2026-04-24)
- [go-spiffe lib](https://github.com/spiffe/go-spiffe) (consultada 2026-04-24)
- [Istio — SPIFFE Identity](https://istio.io/latest/docs/concepts/security/#istio-identity) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[iam]]
- [[least-privilege]]
- [[beyond-corp]]
- [[secrets-management]]
- [[opa-rego]]
