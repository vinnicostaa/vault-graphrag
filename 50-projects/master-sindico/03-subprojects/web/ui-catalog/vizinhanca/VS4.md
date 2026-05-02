---
title: "VS4 — Detalhe da Promoção (Síndico)"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, sindico]
project: master-sindico
persona: sindico
category: VS
screen_id: VS4
sub_produto: vizinhanca
plan_requirement: trial
status: specification
stack: web
created: 2026-04-23
---

# VS4 — Detalhe da Promoção (Síndico)

## Finalidade
Tela de detalhe de uma promoção do parceiro. Exibe título, descrição, tipo de benefício, validade e botão para ativar (gera cupom + métrica para o parceiro como `actor_type=sindico`).

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VS4
- `vault/90-archive/master-sindico-pdfs-cliente-originais/MÓDULO VIZINHANÇA - MORADOR.md` §TELA VM3 (paralelo)

## Persona e ABAC
- **Persona**: Síndico (role `syndic`)
- **Plan tier**: `trial`
- **ABAC action**: `commercial.neighborhood_benefit.read`

## Informações exibidas
- Título da promoção
- Descrição
- **Tipo** (1 dos 7 — `desconto_percentual | desconto_fixo | produto_gratuito | combo_promocional | avaliacao_gratuita | brinde | beneficio_exclusivo`)
- Validade (início → fim)
- Termos / condições (texto)
- Estabelecimento (link para [[VS3]])

## Botão
- **Ativar promoção** → [[VS5]] (gera cupom + lock 4h)

## Componentes
- `PromotionHero` (título + tipo badge + validade chip)
- `PromotionDescription` (markdown limitado)
- `TermsBlock` (sanfona)
- `ActivateButton` (CTA principal — desabilita se promoção encerrou)

## Estados
- **Loading**: skeleton.
- **Encerrada**: badge "Encerrada" + botão desabilitado + texto "Esta promoção não está mais disponível."
- **Próxima do fim** (sugerido < 48h): banner amarelo de urgência.
- **Lock 4h ativo** (já ativou recente): aviso "Aguarde 4h para reativar."
- **Error**: toast com retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Detalhe promoção | `/api/v1/neighborhood/promotions/{id}` | GET |
| Iniciar ativação | `/api/v1/neighborhood/promotions/{id}/activate` | POST |

## Regras de negócio críticas
- Síndico vê promoções **abertas** + **exclusivas do próprio condomínio**.
- Tipo `beneficio_exclusivo` aparece com badge especial.
- Ativação respeita **lock de 4 horas** anti-abuso (D-064).
- Cupom gerado segue formato `ID_LOJA(5)+SEQ(3)+PALAVRA(5)`.

## Ligações
- [[VS3]] · [[VS5]] · [[VS10]] · [[_moc]]
- Sistema de Cupons D-064: `master-sindico-v2/STATE.md`

## Gaps/ressalvas
- Endpoint REST inferido.
- Formato literal de PALAVRA(5) (dicionário curado vs aleatória) sem ADR.
- Sem decisão sobre como mostrar termos longos (truncar vs expandir).
