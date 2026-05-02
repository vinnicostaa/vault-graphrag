---
title: Haystack — Facebook's photo storage (needle in a haystack)
type: source
tags: [research, meta, haystack, photo-storage, blob-storage, xfs, object-store, source]
source: https://engineering.fb.com/2009/04/30/core-infra/needle-in-a-haystack-efficient-storage-of-billions-of-photos/
created: 2026-04-23
updated: 2026-04-23
aliases: [source-11-haystack]
---

# source-11 — Haystack: Facebook's photo storage

- **URL (blog)**: https://engineering.fb.com/2009/04/30/core-infra/needle-in-a-haystack-efficient-storage-of-billions-of-photos/
- **URL (paper OSDI 2010)**: https://www.usenix.org/legacy/event/osdi10/tech/full_papers/Beaver.pdf
- **Publicado**: 2009-04-30 (blog) / 2010 (paper OSDI)
- **Autor/Canal**: Beaver et al. (Facebook) / Engineering at Meta
- **Tipo**: paper + post de engenharia

## Sinopse

Haystack substituiu NFS pra armazenar bilhões de fotos no Facebook. Reduz drasticamente I/O de disco consolidando milhares de imagens ("needles") em poucos arquivos grandes ("haystack store files"), com índice in-memory.

## Pontos-chave

1. **Problema NFS**: metadata overhead excede cache. 1 foto = múltiplos I/O por read/upload. Insustentável na escala Facebook.
2. **Solução Haystack**: aggregation layer — milhares de imagens empacotadas em 1 arquivo. Metadata in-memory (Google sparse hash), dados no disco.
3. **3 componentes**:
   - **Store** — storage físico + filesystem-level metadata.
   - **Directory** — mapeamento volume lógico → físico.
   - **Cache** — CDN interno, evita calls desnecessárias ao Store.
4. **Needle**: tupla `(offset, key, alternate_key, cookie) → header+data+footer`. Log-structured append-only.
5. **Filesystem**: XFS (extent-based, preallocation, menos fragmentação).
6. **Escala em 2009**: 260B+ imagens, 20+ PB.

## Aplicabilidade ao Master Síndico

**Direta: nenhuma.** Master Síndico não precisa construir storage de blobs.

**Indireta — decisão de storage**:

- **Usar S3/MinIO/equivalente** (commodity cloud) pra vídeos e imagens. O problema que Haystack resolveu (metadata overhead em NFS) já está resolvido pelos object stores modernos.
- **S3 já é Haystack-like internamente** — agrega blobs, metadata indexado, CDN na frente.
- **Não implementar storage próprio** — §14 do CLAUDE.md (no premature abstraction).

**Lição conceitual que vale herdar**:

1. **In-memory index para lookup rápido** — aplicável ao cache ABAC, cache de ranking Connect Me. Lookup O(1) em RAM beats lookup em DB sempre.
2. **Append-only log-structured** pra escrita de auditoria — timeline institucional imutável se encaixa nesse pattern (writes baratos, compactação/expurgo raro, imutabilidade como invariante).
3. **Cache como CDN interno** — pra feed de vídeos, CDN edge (CloudFront/Cloudflare) é padrão; análogo ao Haystack Cache.

## Confiança / datas

- Datas confirmadas via fetch (blog 2009-04-30) e consenso OSDI 2010 — **alta confiança**.
- Fonte primária Meta + paper acadêmico.
- Números (260B, 20PB) são de 2009; escala atual é muito maior mas irrelevante pro nosso contexto.
