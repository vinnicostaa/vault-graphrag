---
title: "LMS Academia — MOC (LMS1-LMS15)"
type: moc
tags: [master-sindico, ui-catalog, web, lms, academia, sindico, empresa, morador]
project: master-sindico
persona: any
sub_produto: lms-academia
status: specification
stack: web
created: 2026-04-23
updated: 2026-04-23
---

# LMS — Academia Master Síndico — MOC

Map of Content do catálogo UI do módulo **LMS / Academia Master Síndico**. **15 telas** (LMS1-LMS15) cobrindo as 5 áreas: Cursos, Treinamentos, Lives, Fórum, Biblioteca + Trilhas + Certificado + Meu Progresso.

> "O conhecimento transforma gestão. A Academia Master Síndico reúne cursos, treinamentos e experiências para fortalecer o ecossistema condominial."

## Acesso (matriz)
| Persona | Acesso |
|---|---|
| Síndico | Todas as 5 áreas (certificado tier `plus`+; tier `trial` Q5 aberta sem certificado) |
| Morador Pagante | Todas as 5 áreas, certificado habilitado (tier `plus`/`pro`) |
| Morador Base | Parcial: Lives (replay público), Fórum, Biblioteca; sem cursos completos, sem certificado |
| Empresa Plus | Consumir + certificado |
| Empresa Pro | Consumir + criar cursos certificados (workflow moderação admin) |
| Agência Marketing | Consumir, sem certificado |
| Admin Master Síndico | Total + curador (cria, modera, agenda Lives) |
| **Parceiro Vizinhança** | ❌ **BLOQUEADO** (INV-067; ABAC DENY + UI oculta o botão) |

## Telas
- [[LMS1]] Home Academia · [[LMS2]] Explorar Cursos · [[LMS3]] Página do Curso · [[LMS4]] Aula
- [[LMS5]] Conclusão + Certificado · [[LMS6]] Treinamentos · [[LMS7]] Página do Treinamento
- [[LMS8]] Agenda de Lives · [[LMS9]] Sala da Live · [[LMS10]] Replay
- [[LMS11]] Fórum · [[LMS12]] Criar Tópico · [[LMS13]] Discussão
- [[LMS14]] Biblioteca · [[LMS15]] Página do Material

> Nota de divergência: O briefing usa esta numeração (LMS1 entrada → LMS5 certificado → LMS6/7 treinamentos → LMS8/9/10 lives → LMS11-13 fórum → LMS14/15 biblioteca). O sub-produto destilado v2 usa outra ordem lógica (LMS5 player, LMS7 certificado etc.). **Adoto a numeração do briefing**; pendência de uniformização (G-05).

## Regras críticas
1. **Parceiro Vizinhança NÃO acessa Academia** — INV-067 (hard ABAC + UI oculta o botão "Academia").
2. **Lives = admin + Empresa Pro + agência delegada** — D-097 Fase 11 (revoga D-062/D-076 admin-only). Primeira live = apresentação oficial da plataforma. ABAC: `live.create` allow iff `role=admin` OR (`role=enterprise` AND `tier=pro`) OR (`role=marketing` AND active delegation).
3. **Síndico tier N1 (trial) sem certificado** — Q5 aberta. Tier `plus`+ requerido.
4. **Fórum**: moradores apenas comentam; só `syndic | enterprise | admin` criam tópicos (regra de qualidade técnica).
5. **Quiz aprovação ≥ 70%** (default `passThreshold`); reprovação permite retry ilimitado.
6. **Certificado verificável publicamente** em `/certificates/verify/{serial}` sem auth; `serialNumber` UNIQUE imutável.
7. **Cursos pagos** integram Stripe via `IPaymentGateway`; trial mostra preview da 1ª aula.
8. **Empresa Pro** cria cursos certificados via workflow `draft → submitted → published|rejected` com moderação admin.
9. **Lives replay disponível por 30 dias** pós-evento.
10. **Lock 90d em curso publicado** (RC-7, análogo a GR-029 dos vídeos institucionais).

## 4 tipos de Live (`LiveType`)
`debates_condominiais | lancamento_solucoes | palestras_tecnicas | entrevistas_especialistas`

## Fontes canônicas
- `vault/90-archive/from-development-content-2026-04-23/contents/ARQUITETURA DO MÓDULO LMS.md`
- `vault/90-archive/from-development-content-2026-04-23/contents/MÓDULO ACADEMIA MASTER SÍNDICO.md`
- `vault/50-projects/master-sindico/00-product/sub-produtos/06-lms-academia.md`
- `vault/50-projects/master-sindico/03-subprojects/backend/.kiro/specs/master-sindico/requirements/content.md` Req 98.1
- `master-sindico-v2/STATE.md` D-062 (Lives admin-only MVP), D-058 (admin role transversal)

## Ligações
- [[../../../../00-product/sub-produtos/06-lms-academia]]
- [[../../../../00-product/sub-produtos/04-conteudo-videos]]
- [[../vizinhanca/_moc]] (parceiro bloqueado nessa intersecção)

## Gaps consolidados
- Q5: Síndico tier `trial` recebe certificado? (Padrão atual: não)
- Q28: Lives empresa Pro pós-MVP — escopo?
- Aggregate formal `Live` (LMS) ausente — distinto de `LiveSession` (Assembly).
- ADR provedor de Live LMS (Mux Live vs Cloudflare Stream) pendente — CNT-022.
- Numeração LMS vs briefing (G-05).
