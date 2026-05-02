---
title: MOC — Sub-produtos Master Síndico
type: moc
tags: [moc, sub-produtos, master-sindico, v2, fase-8]
created: 2026-04-23
updated: 2026-04-23
---

# Sub-produtos do Master Síndico

11 sub-produtos integrados. Cada arquivo abaixo é destilação **exaustiva** (método Fase 8 — leitura arquivo-por-arquivo, catálogo fiel com fonte+data). Planos globais (trial/base/plus/pro — ADR-0032); personas carregam matriz de permissão (D-066). Agnosticismo de provedor (D-056).

## Catálogo

- [[01-governanca-institucional]] — Condomínio, unidades, timeline, plano diretor, comunicados, eventos, enquetes, validações
- [[02-connect-me]] — Marketplace formal unidirecional (4 direções) + Marketing Link
- [[03-reputacao]] — Bronze→Diamante determinística
- [[04-conteudo-videos]] — Vídeos institucionais + Shorts + Lock 90d + Moderação
- [[05-assembleia-live]] — Live WebSocket + telão + votação fracional + ata imutável (5 blocos)
- [[06-lms-academia]] — Cursos, trilhas, quizzes, certificados, biblioteca, fórum, lives (15 telas)
- [[07-banco-de-talentos]] — Vídeo-currículo 90s + busca empresa (11 telas)
- [[08-vizinhanca]] — 4 jornadas (VZ/VS/VE/VM, 29 telas) + Sistema de Cupons ativo
- [[09-compliance]] — C1-C11 + 4 gatilhos + auditoria + trilha LGPD
- [[10-marketing-agencias]] — Sub-produto próprio para agências (login Zitadel + delegação ABAC)
- [[11-administradoras-condominiais]] — Empresa que administra condomínios (entidade + vínculo)

## Método Fase 8

Cada sub-produto segue:
1. **Leitura exaustiva** arquivo-por-arquivo (não grep)
2. **Catálogo fiel** com trechos citados: `arquivo:linha (YYYY-MM-DD)`
3. **Fontes canônicas**:
   - `Development/backend/.kiro/specs/master-sindico/requirements/*.md` (Abril/2026)
   - `Development/backend/internal/modules/*/` (código Go verdade última)
   - `Development/Content/03-Modulos/*.md` + `02-Jornadas/*.md`
   - `~/Documentos/Obsidian Vault/Clients/MasterSindico/02 - Domínios e Requisitos/*.md`
   - Matriz funcional (sem N1/N2/N3 — reescrita por ADR-0032/D-067)
4. **Estrutura padrão**: descrição, personas-alvo, tiers de plano aplicáveis, features, quotas, telas (tela-por-tela com fluxo), regras, invariantes, dependências inter-sub-produto, estado no código
5. **Dual-check obrigatório** (agent B em contexto limpo revisa)

## Telas & Endpoints (traceability)

A rastreabilidade ponta-a-ponta (tela × sub-produto × requirement × endpoint × invariante × sprint) está consolidada em [[../../03-subprojects/traceability]].

**Highlights**:
- 161 telas web + 59 mobile catalogadas
- ~240 endpoints REST/WS distintos identificados
- 116 invariantes canônicos cruzados
- 7 fluxos cross-stack documentados (onboarding síndico/morador, Connect Me, assembleia, vídeo delegado, encerrar mandato, avaliação bimestral)
- Cobertura por sprint (0-10) + matriz ABAC (actions × persona × plan × tela) + matriz de hardening de segurança (INV-101..INV-123)

**Sub-produtos com seção "Telas & Endpoints" expandida** (referenciando a matriz):
- [[01-governanca-institucional#§ 15.5 Telas & Endpoints]] — núcleo MVP
- [[02-connect-me#18.5 Telas & Endpoints]] — marketplace formal
- [[04-conteudo-videos#19.5 Telas & Endpoints]] — Mux + LMS
- [[05-assembleia-live#§ 19.5 Telas & Endpoints]] — 21 telas + 5 mobile
- [[09-compliance#§ 19.5 Telas & Endpoints]] — C1-C11 + 4 gatilhos
- [[10-marketing-agencias#18.5 Telas & Endpoints]] — delegação ABAC

## Links

- [[../../03-subprojects/traceability]] — matriz ponta-a-ponta (~940 linhas)
- [[../portfolio-de-produtos]] (legado Fase 3 — ser reescrito)
- [[../personas]] (legado Fase 3 — ser reescrito)
- [[../business-model]] (legado Fase 3 — ser reescrito)
- [[../../02-architecture/adr/0032-global-plans-stripe-style]]
- [[../../STATE]] §D-056..D-068
