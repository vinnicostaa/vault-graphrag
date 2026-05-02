---
title: MOC — Assembleia (ui-catalog web)
type: moc
tags: [master-sindico, ui-catalog, web, assembleia, moc]
project: master-sindico
sub_produto: assembleia-live
status: specification
stack: web
created: 2026-04-24
---

# MOC — Telas Web da Assembleia Master Síndico

21 telas cobrindo os 5 blocos macro do módulo Assembleia Live: (1) Configuração estrutural, (2) Pré-assembleia, (3) Assembleia ao vivo, (4) Encerramento, (5) Pós-assembleia e histórico.

Fonte canônica única: [[../../../../../../90-archive/from-development-content-2026-04-23/03-Modulos/Assembleia-Live-Funcional]] + PDFs do cliente em `90-archive/master-sindico-pdfs-cliente-originais/`.

---

## Bloco 1 — Configuração & Criação

- [[ASM-DASH]] — Dashboard do Módulo Assembleia (síndico)
- [[ASM-CFG]] — Configurar Assembleia (tipo, data, convocações, modalidade)
- [[ASM-PAUTA]] — Cadastrar Itens da Pauta (5 tipos, ilimitados)
- [[ASM-EDITAL]] — Anexar / Gerar Edital (≥8 dias Lei 4.591/64)
- [[ASM-PUB]] — Publicar Edital (LOCK_Pauta + QR code)

## Bloco 2 — Pré-assembleia

- [[ASM-CIEN]] — Ciência Obrigatória (app travado morador)
- [[ASM-TERM]] — 5 Termos Obrigatórios (LGPD + identidade)
- [[ASM-PROC-CAD]] — Cadastro Procuração (morador)
- [[ASM-PROC-VAL]] — Validação Procuração (administradora)
- [[ASM-ENQP]] — Enquete Preliminar (sim/não, sem valor jurídico)
- [[ASM-FRAC]] — Upload Fração Ideal (NUMERIC(5,4), soma=100%)
- [[ASM-INAD]] — Upload Inadimplência (cutoff 1h antes)
- [[ASM-SIM]] — Simulador de Quórum

## Bloco 3 — Assembleia ao Vivo

- [[ASM-CHKIN]] — Check-in (App / QR / Terminal)
- [[ASM-PAINEL-SIND]] — Painel do Síndico (mesa + iniciar)
- [[ASM-TELAO]] — Telão 2 áreas
- [[ASM-VOTO]] — Votação (Sim/Não/Abstenção ou 1-5)
- [[ASM-FILA-FALA]] — Fila de Fala (2/3 min + prorrogação)
- [[ASM-HOML]] — Homologação (resolvido/adiado/prejudicado)

## Bloco 4 — Encerramento

- [[ASM-ENCER]] — Encerrar + Registro Final (ata imutável ADR-0033)

## Bloco 5 — Pós-assembleia

- [[ASM-REL]] — 6 Relatórios + Score Transparência (10 componentes)

---

## Matriz de personas (primária)

| Persona | Telas |
|---|---|
| síndico | DASH · CFG · PAUTA · EDITAL · PUB · PAINEL-SIND · HOML · ENCER · REL · SIM |
| administradora | FRAC · INAD · PROC-VAL · (co-operação em quase tudo) |
| morador | CIEN · TERM · PROC-CAD · CHKIN · VOTO · FILA-FALA · ENQP |

## Invariantes & ADRs relevantes

- **INV-ASM-live-<200ms**: WebSocket P95 em check-in, votação, fila de fala (§11 NFR).
- **INV-ASM-events-append-only**: voto, presença, homologação são eventos imutáveis.
- **ADR-0033**: ata imutável após [[ASM-ENCER]]; ajustes só via adendo administrativo.
- **ADR-0026**: step-up MFA em publicar, iniciar, encerrar, homologar.
- **Lei 4.591/64**: prazo ≥8 dias de convocação; template da ata conforme lei.
- **LGPD (§4.17 + Art. 8º)**: 5 termos obrigatórios com hash versionado.

## Fluxo macro (resumo diagrama §12)

`DASH → CFG → PAUTA → EDITAL → PUB → CIEN → [TERM · PROC-CAD · ENQP · FRAC · INAD · PROC-VAL · SIM] → CHKIN → PAINEL-SIND → TELAO → VOTO → FILA-FALA → HOML → ENCER → REL`

## Ligações externas

- [[../../../05-assembleia-live]]
- [[../../../../client-canvas/Arquitetura Assembleia]]
- [[../../../../client-canvas/Fluxo Assembleia Ao Vivo]]
- [[../../../../04-requirements/functional/assembly]]
- [[../../../admin-privilegios]]
- [[../../../../_moc]] (MOC web geral)

## Gaps & pendências

- Q27 — Mudança de escopo (assíncrono → live) documentada em `quebras-detectadas`.
- Q40 — Eleição gera avaliação escondida: regra não migrada para requirements canônico.
- Híbrida/online: modalidade preparada mas inativa em M1.
- Transcrição automática de fala microfonada: backlog M2/M3.
- Assinatura digital ICP-Brasil da ata: fora de M1.
