---
title: Source 16 — Borg Omega and Kubernetes lessons learned
type: source
tags: [research, google, borg, omega, kubernetes, lessons-learned, scheduler]
source: https://research.google/pubs/borg-omega-and-kubernetes/
created: 2026-04-23
updated: 2026-04-23
aliases: [borg-omega-k8s]
---

# Source 16 — Borg, Omega, and Kubernetes

- **URL primária**: https://research.google/pubs/borg-omega-and-kubernetes/
- **PDF**: https://research.google.com/pubs/archive/44843.pdf
- **Publicação**: ACM Queue vol.14 n.1 (2016), também Communications of the ACM.
- **Autores**: Burns, Grant, Oppenheimer, Brewer, Wilkes
- **Data fetch**: 2026-04-23

## Trechos rastreáveis

### Três gerações
- **Borg (1ª)**: monolítico Borgmaster, admission control + task packing + over-commitment.
- **Omega (2ª)**: "stored cluster state in a centralized Paxos-based transaction-oriented store accessed by different parts of the cluster control plane using optimistic concurrency control" → desacoplou Borgmaster em componentes peers.
- **Kubernetes (3ª)**: concebido num mundo de containers Linux + public cloud. "Open source—a contrast to Borg and Omega, which were developed as purely Google-internal systems".

### Lessons learned chave
- Kubernetes "is built from a set of composable building blocks that can readily be extended by its users" — extensibilidade via CRDs.
- "Kubernetes allocates an IP address per pod, aligning network identity with application identity" — lição do Borg onde identidade de rede era bagunçada.
- Otimistic concurrency + event watch = padrão K8s herdado de Omega.

## Aplicabilidade ao Master Síndico

### Lições operacionais
1. **Extensibility via composable building blocks** — favorecer CRDs/operators em vez de soluções custom fora do K8s. Ex: `VerticalPodAutoscaler`, `ExternalSecrets`, `cert-manager`.
2. **Optimistic concurrency** — padrão aplicável em domínio: `OCC` via column `version INT` em agregados DDD; update `WHERE id = ? AND version = ?` (já aplicado em pgx).
3. **Network identity = application identity** — em Master, cada BC tem Service K8s próprio + mTLS (M2+); sem "shared pods".
4. **Evolução incremental** — não pular fases: M1 Compose → M2 K8s single cluster → M3 multi-cluster (se/quando).

### Não-aplicável diretamente
- Omega (Paxos + OCC planet-scale) não precisa ser replicado — Postgres + advisory locks + version-based OCC cobre.

## Links relacionados

- [[source-15]] Borg paper original.
- [[source-19]] Chubby (Paxos que inspirou Omega store).
- Destilado: [[_destilado#6 Cluster management]].
