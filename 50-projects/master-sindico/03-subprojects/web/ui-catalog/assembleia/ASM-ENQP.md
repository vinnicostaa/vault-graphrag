---
title: ASM-ENQP — Enquete Preliminar
type: ui-screen
tags: [master-sindico, ui-catalog, web, assembleia]
project: master-sindico
persona: morador
secondary_persona: sindico
category: ASM
screen_id: ASM-ENQP
sub_produto: assembleia-live
status: specification
stack: web
created: 2026-04-24
---

# ASM-ENQP — Enquete Preliminar

## Finalidade

Tela para moradores responderem enquete **sem valor jurídico** (sim/não) sobre cada item deliberativo da pauta antes da assembleia. Objetivos: engajamento, preparação, medir sentimento da base. **Não substitui** a deliberação oficial.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/03-Modulos/Assembleia-Live-Funcional]] §4.11 (Enquetes preliminares)
- [[../../../../04-requirements/functional/assembly]] Req 54

## Persona e ABAC

- **Primária**: `resident` — responde enquete.
- **Secundária**: `syndic`/`administradora` — monitoram resultados agregados em dashboard.
- ABAC: `assembly.poll.answer` (morador) / `assembly.poll.read` (síndico).

## Fluxo da tela

1. Lista de itens deliberativos com enquete habilitada (tipos `votacao_simples` e `votacao_fracao_ideal`).
2. Cada item mostra: título · descrição resumida · link para detalhes · 3 opções: **[Sim] [Não] [Ainda não decidi]**.
3. Morador seleciona resposta → envio imediato.
4. Ao concluir, agradecimento + CTA "Ver outros itens".
5. Visão síndico: gráfico agregado (% sim, % não, % indeciso, % não respondeu).

## Componentes

- Cards por item com radio buttons grandes (thumb-friendly em mobile).
- Gráfico de barras (síndico).
- Sem contador público (privacidade §2.3 excopo).

## Estados (loading/empty/error/success)

- **Empty síndico**: "Aguardando respostas".
- **Success morador**: "Resposta registrada — não substitui a votação oficial".

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Responder | `/api/v1/assemblies/:id/poll/:item_id/answer` | POST | `{answer:"yes"|"no"|"undecided"}` | `{at}` |
| Dashboard | `/api/v1/assemblies/:id/poll/stats` | GET | — | `{items[{id, yes, no, undecided, pending}]}` |

## Regras de negócio críticas

- **Opções apenas sim / não / indeciso** (§4.11 — "apenas sim ou não", estendemos com "indeciso" para granularidade de UX).
- Apenas tipos `votacao_simples` e `votacao_fracao_ideal` têm enquete automática.
- **Sem valor jurídico** — disclaimer visível em toda tela.
- Resposta pode ser alterada até abertura da votação oficial na assembleia ao vivo.
- Não computa no quórum real — totalmente separado do voto oficial.

## Ligações

- Origem: [[ASM-CIEN]] (após ciência, morador pode participar).
- Destino: [[ASM-VOTO]] (votação oficial).
- Relacionados: [[ASM-PAUTA]].

## Gaps/ressalvas

- "Enquete preliminar" vs "simulação de sentimento" — nome final em UX pendente, manter "Enquete" por aderência ao cliente.
- Exibir ao morador, na hora da votação oficial, como ele respondeu na enquete preliminar: UX pendente (ajuda vs influencia?).

## Links

- [[_moc]]
- [[ASM-VOTO]]
- [[ASM-PAUTA]]
