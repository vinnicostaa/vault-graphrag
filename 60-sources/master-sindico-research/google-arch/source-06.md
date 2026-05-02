---
title: Source 06 — Colossus Google file system successor GFS
type: source
tags: [research, google, colossus, gfs, storage, filesystem]
source: https://cloud.google.com/blog/products/storage-data-transfer/a-peek-behind-colossus-googles-file-system
created: 2026-04-23
updated: 2026-04-23
aliases: [colossus]
---

# Source 06 — A peek behind Colossus, Google's file system

- **URL primária**: https://cloud.google.com/blog/products/storage-data-transfer/a-peek-behind-colossus-googles-file-system
- **Relacionado**: https://cloud.google.com/blog/products/storage-data-transfer/how-colossus-optimizes-data-placement-for-performance
- **Status**: whitepaper/blog oficial Google Cloud (não há paper acadêmico Colossus tão formal quanto GFS 2003).
- **Data fetch**: 2026-04-23

## Trechos rastreáveis (WebFetch 2026-04-23)

Componentes arquiteturais (cloud.google.com blog):

- **Client Library**: "a cliente mais complexa do sistema"; implementa software-RAID; aplicações escolhem encoding.
- **Curators**: metadata service, horizontalmente escalável.
- **Metadata DB**: em **Bigtable** — destrava escala 100x+ vs GFS.
- **D servers**: network-attached disks; data flui direto client ↔ D (mínimo de network hops).
- **Custodians**: background managers pra durability, availability, rebalance, RAID reconstruction.

Claims de escala:
- Exabytes por cluster; dezenas de milhares de máquinas.
- Powers YouTube, Drive, Gmail, Cloud Storage, Firestore.

Eficiência:
- Hot data em flash, cold em disco.
- Rebalance: dados novos distribuídos entre drives; dados velhos migram pra drives de maior capacidade.
- **Resource disaggregation** — storage pool compartilhado multi-workload.

## Aplicabilidade ao Master Síndico

- **Direta**: NENHUMA. Volume de dados do Master Síndico cabe em disco tradicional por décadas.
- **Inspiracional**:
  - **Disaggregation storage/compute** → já aplicamos: Postgres gerenciado (RDS-like) + S3-compat pra blobs (vídeos, PDFs) + workers de vídeo em pods separados.
  - **Hot/Cold tiering** → inspira política de storage de vídeos/gravações assembleia: <30 dias em standard S3, >30 em IA/Glacier.
- **Por quê não literal**: Colossus é infra-interna GCP; o que consumimos dele indiretamente é via produtos managed (Cloud Storage, Firestore) — mas na stack atual do Master preferimos S3-compat open.

## Links relacionados

- [[source-02]] Bigtable (consumidor de Colossus).
- [[source-17]] Dremel/BigQuery (também storage-compute disaggregated).
- Destilado: [[_destilado#2.4 Colossus]].
