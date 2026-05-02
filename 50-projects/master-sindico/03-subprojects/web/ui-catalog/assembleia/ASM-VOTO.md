---
title: ASM-VOTO — Votação
type: ui-screen
tags: [master-sindico, ui-catalog, web, assembleia]
project: master-sindico
persona: morador
secondary_persona: sindico
category: ASM
screen_id: ASM-VOTO
sub_produto: assembleia-live
status: specification
stack: web
created: 2026-04-24
---

# ASM-VOTO — Votação Oficial

## Finalidade

Tela de votação oficial pelo morador presente (app / browser) ou lançamento pelo operador (terminal — presencial assistido §5.2). Suporta **Sim / Não / Abstenção** (votação simples) ou **Escala 1-5 + Abstenção** (votação escalada). Aplica peso fracionado conforme [[ASM-FRAC]] quando o item for de tipo `votacao_fracao_ideal`.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/03-Modulos/Assembleia-Live-Funcional]] §6 (Votação)
- [[../../../../04-requirements/functional/assembly]] Req 57

## Persona e ABAC

- **Primária**: `resident` com unidade apta (não-inadimplente, ou liberada via override — [[ASM-INAD]]).
- **Secundária**: `syndic`/`administradora` lançam voto presencial assistido pelo operador.
- ABAC: `assembly.vote.cast` ou `assembly.vote.cast_assisted`.

## Fluxo da tela

**Morador**:
1. Notificação "Votação aberta: Item X".
2. Tela mostra título, descrição, quórum exigido, opções:
   - Simples: **[Sim] [Não] [Abstenção]**
   - Escala: **[1] [2] [3] [4] [5] [Abstenção]**
3. Confirmação dupla ("Você está votando Sim. Confirmar?").
4. Registrado → exibe badge "Seu voto: Sim".
5. Não pode alterar após envio (salvo reabertura do item pelo síndico).

**Operador (presencial assistido)**:
1. Lista unidades check-in terminal sem voto.
2. Seleciona unidade + escolhe opção + confirma.
3. Registro com `vote_mode=presencial_assistido` + `operator_id`.

**Procurador**:
1. Vota em seu próprio nome + uma linha por unidade representada.

## Componentes

- Botões XL acessíveis (thumb-friendly mobile, alto contraste).
- Confirmação dupla com contador 2s.
- Badge de voto registrado.
- Lista de unidades (operador) com busca.

## Estados (loading/empty/error/success)

- **Error**: voto já registrado · item não aberto · unidade inapta · conexão perdida (auto-retry).
- **Success**: toast + atualização telão [[ASM-TELAO]].

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Votar | `/api/v1/assemblies/:id/agenda/:item/vote` | POST | `{value:"yes"|"no"|"abstain"|1..5, assisted?:bool, unit?}` | `{vote_id, at}` |
| Listar status item | `/api/v1/assemblies/:id/agenda/:item/votes` | GET | — | `{open:bool, counts, weighted_counts}` |

## Regras de negócio críticas

- **Tipos de voto** (§6): simples Sim/Não/Abstenção; escalado 1-5+Abstenção.
- **Procuração**: procurador vota → voto replicado para unidade representada, respeitando fração ideal (§6 regra).
- **Peso**: itens `votacao_fracao_ideal` aplicam `NUMERIC(5,4)` da [[ASM-FRAC]]; itens simples contam 1 voto/unidade.
- **Append-only** (INV): voto gravado não é reescrito; revisão só via "voto substituto" em reabertura — gera novo evento + audit.
- **ADR-0033 ata imutável**: após homologação ([[ASM-HOML]]), votos entram na ata e qualquer ajuste exige adendo formal.
- Latência < 200ms P95 WebSocket (§11 NFR).

## Ligações

- Origem: [[ASM-PAINEL-SIND]] (síndico abre votação).
- Destino: [[ASM-HOML]], [[ASM-TELAO]] (atualiza ao vivo).
- Relacionados: [[ASM-ENQP]], [[ASM-FRAC]], [[ASM-PROC-VAL]].

## Gaps/ressalvas

- Secret ballot (voto secreto): **não implementado** — voto é identificado por unidade (requisito legal de rastreabilidade em condomínios). Confirmar com jurídico cliente.
- Alteração de voto antes de encerrar votação: política default **não permitido** — ajustar se cliente pedir.

## Links

- [[_moc]]
- [[ASM-HOML]]
- [[ASM-PAINEL-SIND]]
