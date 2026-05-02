---
title: Source — Gödel Scheduler (ByteDance, open-sourced CNCF 2024)
type: source
tags: [research, bytedance, kubernetes, scheduler, godel, cncf, task-scheduling]
source: https://www.cncf.io/blog/2024/04/02/godel-scheduler-open-sourced-a-unified-scheduler-for-online-and-offline-workloads/
created: 2026-04-23
updated: 2026-04-23
aliases: [godel-scheduler, bytedance-scheduler]
---

# Source — Gödel Scheduler: unified scheduler ByteDance

## Metadados

- **CNCF blog (oficial)**: https://www.cncf.io/blog/2024/04/02/godel-scheduler-open-sourced-a-unified-scheduler-for-online-and-offline-workloads/
- **Repo**: https://github.com/kubewharf/godel-scheduler (Apache 2.0, kubewharf = ByteDance open-source org)
- **Autoridade**: altíssima — CNCF blog + repo oficial.

## Contexto

A busca por "ByteTask" **não retornou sistema com esse nome específico**. O scheduler em produção de larga escala da ByteDance que é publicamente documentado é o **Gödel Scheduler** (e o Plan Scheduler do ByConity para data warehouse). "ByteTask" pode ser um nome interno antigo ou confusão com outro sistema; usei Gödel como equivalente funcional documentado.

## O que é

**Extensão do Kubernetes Scheduler** desenhada pra unificar workloads **online** (microservices, serving) e **offline** (batch, ML training, big data) na mesma infraestrutura. Alternativa drop-in ao kube-scheduler nativo com mesmo plugin model.

## Arquitetura em 3 componentes

1. **Dispatcher** — queueing, distribution pra instâncias de scheduler, particionamento de nós.
2. **Scheduler** — decisões de placement + preemption. **Two-tier framework**: Unit-level + Pod-level.
3. **Binder** — executa o binding final, detecta conflitos, executa preemption.

## Inovações

- **Optimistic concurrency** — etapa mais cara (filter + score de nós) move pro scheduler component, permitindo execução concorrente. Aumenta throughput.
- **Two-layer abstraction** — "Unit" = conjunto de pods correlatos que precisa schedular atomicamente (gang scheduling). Útil pra distributed training (todos os workers precisam subir juntos ou nenhum sobe).
- **Unified resource pool** — não segrega clusters online/offline. Resource pooling entre microservices, recsys, ML/big data, storage. Melhora utilização.

## Performance / números

- **Single shard**: 2.000+ pods/s.
- **Multi-shard**: 5.000+ pods/s.
- **Maior cluster**: 20.000+ nós, 1.000.000+ pods.
- **30x** throughput vs. kube-scheduler nativo.

## Open-source status

- Open-sourced 2023 (ByteDance) / contribuído pra CNCF 2024.
- Repo: https://github.com/kubewharf/godel-scheduler
- Licença: Apache 2.0.

## Sistemas correlatos ByteDance

- **ByConity Plan Scheduler** — scheduler de SQL queries em data warehouse cloud-native da ByteDance. Acessa Resource Manager, escolhe nós pra execução. Referência: https://byconity.github.io/blog/2023-05-24-byconity-announcement-opensources-its-cloudnative-data-warehouse
- **Krypton** — engine real-time de serving SQL analítico (VLDB 2023). https://www.vldb.org/pvldb/vol16/p3528-chen.pdf

## Aplicabilidade ao master-sindico

### M1-M2: overkill

- Volume de jobs do master-sindico:
  - Cleanup LGPD (retention policy): 1x/dia.
  - Re-cálculo de reputação: 1x/hora.
  - Envio de emails/push (convocação, notificação, lembrete): on-demand.
  - Cobrança/boleto: diário em horários fixos.
  - Backup por tenant: noturno.
- **K8s CronJob** (ou Cloud Run Jobs + Cloud Scheduler se GCP) resolve tudo.

### Quando reconsiderar

- Centenas de tenants com jobs customizados por tenant → escalonamento por prioridade + isolamento → **Temporal.io** ou **river-queue** são next steps antes de precisar de algo Gödel-like.
- Temporal.io é state-of-the-art pra workflows duráveis com retry + saga compensation — **provavelmente adotar em M2** quando crescer complexidade de sagas (ex: "criar assembleia" = convocar + enviar material + abrir votação + fechar + gerar ata).

### Lições conceituais reaproveitáveis

- **Gang scheduling / "unit"** — agrupar tasks correlatas que precisam rodar juntas ou nenhuma. Padrão aplicável a **sagas** do master-sindico: envio de convocação (email + push + WhatsApp) é uma unit; ou todos enviam ou compensa tudo.
- **Optimistic concurrency** — não travar antecipadamente; detectar conflito e retry. Aplicável a updates de reputação/score concorrentes.

## Fontes complementares

- [VLDB 2024 — Practical insights into large-scale Spark workloads at ByteDance](https://www.vldb.org/pvldb/vol17/p3759-shi.pdf).
- [ByteDance microservices workflow characterization (Wiley)](https://onlinelibrary.wiley.com/doi/abs/10.1002/smr.2467).
- [Anyscale — ByteDance offline inference scaling](https://www.anyscale.com/blog/how-bytedance-scales-offline-inference-with-multi-modal-llms-to-200TB-data).

## Caveat sobre "ByteTask"

Nenhuma fonte primária documentou sistema chamado "ByteTask" da ByteDance. **Hipóteses**:

- Nome interno/antigo não exportado publicamente.
- Pode ser uma denominação de domínio chinês (字节任务) que não tem análogo inglês em literatura acessível.
- Possível confusão do usuário com "ByteGraph" ou "Gödel scheduler".

**Recomendação**: tratar "ByteTask" como equivalente funcional a Gödel Scheduler até prova em contrário. Revisar se aparecer referência oficial.

## Links

- [[_destilado#6. Task scheduling em escala (Gödel / ByteTask)]]
- [[_moc]]
