---
title: Tasks — BC content
type: spec
tags: [tasks, by-module, content, master-sindico, v2]
source: 04-requirements/functional/content.md + backlog.md
created: 2026-04-23
updated: 2026-04-23
---

# Tasks — BC `content`

Tasks do bounded context **content** (vídeos Mux, moderação manual 48h, lock 90d, LMS, busca OpenSearch, ebooks).

Alinhamento: [[../../04-requirements/functional/content]] (CTT-001..CTT-011).

---

## M2

### Vídeos + Mux

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-200 | Player HLS + upload presigned UI (empresa) | 11 | ⏳ |
| T-200a | Upload vídeo-currículo Banco Talentos 90s | 11 | ⏳ |
| T-201 | Moderação manual admin UI (aprovar/rejeitar 48h) | 12 | ⏳ |
| T-202 | Lock 90d enforcement + removal guard (CTT-003) | 11 | ⏳ |
| T-203 | Privacidade métricas cross-tenant + integration tests 10+ | 12 | ⏳ |
| T-204 | `VideoVisibilityGrant` durante votação fornecedor | 12 | ⏳ |

### Busca

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-210 | OpenSearch prod deploy + reindex job | 12 | ⏳ |
| T-210a | Fallback PG tsvector em dev | 12 | ⏳ |

### Research moderação ML (ADR futuro)

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-220 | Research OpenAI Moderation vs AWS Rekognition (ADR M4+) | 13 | ⏳ |

## M3 — LMS + Ebooks

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-250 | LMS estrutura — Course/Module/Lesson + Enrollment | 14 | ⏳ |
| T-250a | Trilha "Síndico Iniciante" seed | 14 | ⏳ |
| T-251 | Ebooks biblioteca base (PDF S3 signed URL) | 15 | ⏳ |
| T-260 | Trilhas Intermediário + Profissional seed | 15 | ⏳ |
| T-261 | Certificados auto 100% + URL pública verificável | 15 | ⏳ |

---

## Invariantes cobertos

I-031..I-038 de [[../../01-domain/bounded-contexts#content]]:
- I-031 Vídeo published → locked 90d
- I-032 Moderação antes de published
- I-033 Métricas não vazam cross-tenant
- I-034 Vídeos empresa bloqueados pra morador default; unlock só durante votação fornecedor
- I-035 Trava trimestral — substituir mesmo vídeo só após 90d
- I-036 Agência Marketing zero acesso a dados de negócio
- I-037 Certificado automático ao 100% Enrollment
- I-038 Parceiro Vizinhança zero acesso LMS

---

## Links

- [[_moc]]
- [[../../04-requirements/functional/content]]
- [[../../01-domain/bounded-contexts#content]]
- [[../backlog]]
