---
title: Source 15 — Large-scale cluster management at Google with Borg
type: source
tags: [research, google, borg, cluster-management, kubernetes, scheduling]
source: https://research.google/pubs/large-scale-cluster-management-at-google-with-borg/
created: 2026-04-23
updated: 2026-04-23
aliases: [borg-paper]
---

# Source 15 — Large-scale cluster management at Google with Borg

- **URL primária**: https://research.google/pubs/large-scale-cluster-management-at-google-with-borg/
- **PDF**: https://research.google.com/pubs/archive/43438.pdf
- **Conferência**: EuroSys 2015
- **Autores**: Verma, Pedrosa, Korupolu, Oppenheimer, Tune, Wilkes
- **Data fetch**: 2026-04-23

## Trechos rastreáveis (WebFetch 2026-04-23)

### Escala
- "Hundreds of thousands of jobs, from many thousands of different applications, across a number of clusters each with up to tens of thousands of machines".

### Mecanismos
- "High utilization by combining admission control, efficient task-packing, over-commitment, and machine sharing with process-level performance isolation".
- Declarative job specification language, name service integration, real-time job monitoring.

### Conceitos hierárquicos
- **Jobs** → **Tasks** → **Allocs** (reserva de recursos compartilhada por tasks relacionadas).
- **Borgmaster** centralizado + **Borglets** em cada node.

### Kubernetes mapping (kubernetes.io/blog, também referenciado)
- "Borglet became Kubelet, alloc became pod, and Borg containers became docker".
- K8s "pod is a resource envelope for one or more containers that are always scheduled onto the same machine and can share resources".

## Aplicabilidade ao Master Síndico

### M1
- NÃO precisamos de Kubernetes ainda. Docker Compose ou **k3s single-node** é suficiente (Go binários pequenos, 5-10 réplicas).

### M2 (Q3 2026)
- **Kubernetes gerenciado** (EKS/GKE/DOKS). Razões herdadas de Borg:
  - **Declarative**: YAML manifests == BorgCfg conceitualmente.
  - **Bin-packing + over-commitment**: `requests < limits`; nodepools dedicados pra workers Livekit (NIC-heavy/GPU).
  - **Process-level isolation**: cgroups v2 + seccomp + não confiar em container como security boundary — combinar com NetworkPolicy + namespaces por ambiente.

### Padrão anti-copy-paste
- "IP address per pod, aligning network identity with application identity" — **não compartilhar pods entre bounded contexts**. Regra: 1 BC = 1 Deployment.
- Over-commitment só em best-effort workloads (batch jobs de relatório), nunca em API síncrona com SLO de latência.

## Links relacionados

- [[source-16]] Borg, Omega, Kubernetes (evolução).
- Destilado: [[_destilado#6 Cluster management]].
