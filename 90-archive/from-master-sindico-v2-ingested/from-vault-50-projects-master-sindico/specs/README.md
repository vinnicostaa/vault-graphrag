# DEPRECATED — web/.kiro/specs/master-sindico/ foi removido em 2026-04-22

As specs do Master Síndico são **canônicas** em `backend/.kiro/specs/master-sindico/`.
O `web/` não mantém specs próprios — consome as mesmas fontes do backend porque
são vertical slices do mesmo produto (ver `web/CLAUDE.md` seção "Fontes de verdade").

Os arquivos antes neste diretório (`design.md`, `requirements.md`, `tasks.md`) eram
**cópias stale de 2026-04-13** com divergência crítica:

- Stack obsoleta: mencionavam Fastify + SES + ECS Fargate + OpenSearch gerenciado
  (todos superados pela stack atual: Gin abstraído + Resend + Railway + PostgreSQL tsvector).
- 0 tasks marcadas `[x]` apesar de Sprints 0-8 completos no backend.
- Arquitetura descrita como "hexagonal" em vez de "Clean Architecture + DDD + CQRS" real.

## Onde estão as specs ativas

Caminho absoluto: `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/backend/.kiro/specs/master-sindico/`

Relativo a este arquivo: `../../../../backend/.kiro/specs/master-sindico/`

## Rastreabilidade

- AUDIT: `backend/.kiro/AUDIT.md` — item A-017 (reconciliação de specs)
- STATE: `backend/.kiro/STATE.md` — D-030 (registrou decisão de não criar specs em web/app)
- Commit/Data: 2026-04-22 (sessão autônoma de auditoria total)
