---
title: ASM-PAINEL-SIND — Painel do Síndico Ao Vivo
type: ui-screen
tags: [master-sindico, ui-catalog, web, assembleia]
project: master-sindico
persona: sindico
secondary_persona: administradora
category: ASM
screen_id: ASM-PAINEL-SIND
sub_produto: assembleia-live
status: specification
stack: web
created: 2026-04-24
---

# ASM-PAINEL-SIND — Painel do Síndico Ao Vivo

## Finalidade

Cockpit operacional do síndico/administradora no dia da assembleia. Antes de iniciar: cadastra mesa (presidente, secretários, administradora presente, auditor, fornecedores) e aciona **[Iniciar assembleia]**. Durante: controla ordem do dia, fluxo de fala, abertura e encerramento de votação, homologações.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/03-Modulos/Assembleia-Live-Funcional]] §5.4 (Painel inicial), §5.5 (Papéis da mesa), §5.6 (Início)
- [[../../../../04-requirements/functional/assembly]] Req 58
- [[../../../../client-canvas/Fluxo Assembleia Ao Vivo]]

## Persona e ABAC

- **Primária**: `syndic` operando ao vivo.
- **Secundária**: `administradora` (co-operação).
- ABAC: `assembly.live.operate` + step-up MFA em ações críticas (iniciar/encerrar).

## Fluxo da tela

### Pré-início — Cadastro da mesa
1. Form com:
   - **Presidente da mesa** (nome + CPF + função).
   - **Até 3 secretários**.
   - **Administradora presente** (nome empresa, representante, CPF).
   - **Auditor** (opcional — nome, função, CPF).
   - **Fornecedores presentes** (opcional — empresa + representante).
2. Checklist pré-início:
   - [x] Fração ideal importada ([[ASM-FRAC]])
   - [x] Inadimplência aplicada (cutoff ok — [[ASM-INAD]])
   - [x] Procurações validadas ([[ASM-PROC-VAL]])
   - [x] Mesa cadastrada
3. Botão **[Iniciar assembleia]** (grande, vermelho, com confirmação dupla + step-up).

### Durante
- Pauta lateral com item **em destaque**.
- Área central: controle do item atual (abrir discussão, abrir votação, encerrar votação, homologar, declarar resolvido/adiado/prejudicado).
- Fila de fala ([[ASM-FILA-FALA]]) embutida/expandível.
- KPIs live: unidades presentes · ausentes · online ativos · quórum atual · relógio assembleia.
- Botão **[Encerrar assembleia]** sempre visível.

## Componentes

- Form mesa com multi-ator.
- Checklist semafórica.
- Painel de controle de item (state machine visível).
- Modal step-up MFA em iniciar/encerrar.

## Estados (loading/empty/error/success)

- **Error iniciar**: algum checklist vermelho → bloqueia.
- **Success iniciar**: transição animada para modo "ao vivo" + roda vídeo curto institucional Master Síndico (§5.6) antes de liberar painel.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Cadastrar mesa | `/api/v1/assemblies/:id/board` | POST | `{president, secretaries[], administradora, auditor, suppliers[]}` | 204 |
| Iniciar | `/api/v1/assemblies/:id/start` | POST | X-Step-Up-Token | `{status:"live", started_at}` |
| Abrir item | `/api/v1/assemblies/:id/agenda/:item/open` | POST | — | `{item_status:"in_discussion"}` |
| Abrir votação | `/api/v1/assemblies/:id/agenda/:item/open-vote` | POST | — | `{item_status:"in_vote"}` |
| Encerrar votação | `/api/v1/assemblies/:id/agenda/:item/close-vote` | POST | — | `{votes_summary}` |
| Homologar item | `/api/v1/assemblies/:id/agenda/:item/homologate` | POST | `{final_status}` | `{homologated_at}` |

## Regras de negócio críticas

- **Iniciar** não pode ser revertido — append-only.
- Vídeo institucional pré-início (§5.6) — obrigatório M1 (≤ 90s).
- **Latência WebSocket < 200ms P95** — todo evento sincroniza com [[ASM-TELAO]] e apps de moradores em tempo real (§11 NFR).
- Append-only em todos os eventos (check-in, voto, homologação — INV-ASM-events).
- Step-up MFA obrigatório em iniciar/encerrar (ADR-0026).

## Ligações

- Origem: [[ASM-SIM]], [[ASM-CHKIN]].
- Destino: [[ASM-TELAO]], [[ASM-VOTO]] (controls), [[ASM-HOML]], [[ASM-ENCER]].
- Relacionados: [[../../../../client-canvas/Fluxo Assembleia Ao Vivo]].

## Gaps/ressalvas

- Papéis da mesa (§5.5 — abrir formalmente, autorizar falas, declarar resultado): UX com múltiplos operadores da mesa pendente (quem opera o painel?). Default M1: síndico opera; presidente declara verbalmente.
- Modo contingência (§6 — Req 60.10): botão dedicado para ativação em falhas — detalhar em M2.

## Links

- [[_moc]]
- [[ASM-TELAO]]
- [[ASM-VOTO]]
