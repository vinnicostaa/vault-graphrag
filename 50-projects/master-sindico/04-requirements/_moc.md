---
title: MOC — 04-requirements
type: moc
tags: [moc, master-sindico, v2, requirements]
created: 2026-04-23
updated: 2026-04-23
---

# MOC — 04-requirements

Requirements globais (GR-###), funcionais por bounded context, não-funcionais, compliance LGPD, matriz funcional, dados de cadastro. Fase 3 (destilação de requirements) em progresso.

> Derivado do legado vivo (`../90-ingested/from-vault-50-projects-master-sindico/04-requirements/` + `specs/requirements/` + `contextos/`) + [[../STEERING]] + [[../00-product/portfolio-de-produtos]].

## Arquivos

| Arquivo | Descrição | Contagem | Prioridade |
|---|---|---|---|
| [[global]] | Requirements globais GR-001..GR-035 | 35 GRs | M1-M3 |
| [[functional/_moc]] | MOC dos requirements funcionais por BC | — | — |
| [[functional/identity]] | Identity BC — auth, session, multi-context, LGPD | IDN-001..IDN-035 | M1-M3 |
| [[functional/billing]] | Billing BC — planos, Stripe, trial, quotas | BIL-001..BIL-034 | M1-M3 |
| [[functional/institutional]] | Institutional BC — condo, unidade, timeline, master plan | INS-001..INS-043 | M1-M3 |
| [[functional/commercial]] | Commercial BC — Connect Me, reputação, Vizinhança, Banco Talentos | COM-001..COM-045 | M2-M3 |
| [[functional/content]] | Content BC — vídeos, moderação, busca, LMS, fórum | CNT-001..CNT-035 | M1-M3 |
| [[functional/assembly]] | Assembly BC — votação, quórum fracional, ata | ASM-001..ASM-033 | M3 |
| [[functional/cross-domain]] | Cross-domain — ABAC, audit, events, providers | XD-001..XD-035 | M1-M3 |
| [[non-functional]] | NFRs — performance, SLO, escalabilidade, seg, LGPD, WCAG | NFR-001..NFR-038 | M1-M4 |
| [[compliance-lgpd]] | LGPD — controles, consent, DPA, breach | LGPD-001..LGPD-030 | M1-M2 |
| [[matrix-functional]] | Matriz feature × persona × plano × quota — 8 módulos | 80+ features | M1 |
| [[registration-data]] | Dados cadastro por persona (6) — obrigatório × perfil | 6 personas | M1 |

## Totais (aproximados)

- **Requirements globais**: 35 GRs.
- **Requirements funcionais**: ~250 (35 identity + 34 billing + 43 institutional + 45 commercial + 35 content + 33 assembly + 35 cross-domain).
- **NFRs**: 38.
- **Controles LGPD**: 30.
- **Módulos da matriz funcional**: 8 (Consumo Conteúdo, Comunidade, Publicação, Perfil/Visibilidade, Connect Me, Gestão Condominial, Currículos, Comércio Local).

## Vizinhos

- [[../_moc]] — MOC raiz da v2
- [[../CLAUDE]] — contrato do agente
- [[../STATE]] — decisões vivas
- [[../STEERING]] — não-negociáveis
- [[../00-product/portfolio-de-produtos]] — sub-produtos
- [[../00-product/personas]] — personas operantes
- [[../01-domain/bounded-contexts]] — mapeamento BC (a destilar)

## Pendências cruzadas

Resumo das pendências para dual-check Fase 3/4:

- **Preços** BIL-001 (stakeholder).
- **Parâmetros reputação** COM-025 / D-010 (stakeholder).
- **Lista 26 áreas master plan** INS-017 (stakeholder).
- **Provedores M2+**: Receita Federal (XD-020), Email Resend vs SES (XD-015), Storage R2 vs S3 (XD-017), Live Mux vs Cloudflare (CNT-022), assinatura digital BR (ASM-010, CNT-019, NFR-038).
- **Livekit Cloud vs self-hosted** ASM-020.
- **DPAs** LGPD-014 com Mux, Zitadel, Livekit, Twilio/Zenvia, Resend (legal).
- **Column-level encryption** LGPD-017 (ADR pgcrypto).
- **Reconciliar M3** assíncrona vs live assembleia (ASM-032).
