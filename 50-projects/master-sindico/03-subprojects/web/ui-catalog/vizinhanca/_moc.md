---
title: "Vizinhança — MOC (29 telas, 4 jornadas)"
type: moc
tags: [master-sindico, ui-catalog, web, vizinhanca, sindico, empresa, morador, parceiro]
project: master-sindico
persona: multi
sub_produto: vizinhanca
status: specification
stack: web
created: 2026-04-23
updated: 2026-04-23
---

# Vizinhança — MOC

Map of Content do catálogo UI do módulo **Vizinhança**. **29 telas** distribuídas em **4 jornadas integradas**: Síndico (VS), Empresa (VE), Morador (VM) e Parceiro (VZ).

> "O condomínio faz parte do bairro. A comunidade começa no condomínio e se estende pelo bairro. A Master Síndico conecta você diretamente com moradores, síndicos e empresas da região."

## Decision 12 — persona Parceiro
**Parceiro da Vizinhança é persona independente com login próprio** (role `local_company`). Aceita CPF (CNPJ não obrigatório). NÃO acessa LMS Academia (INV-067) e NÃO acessa Banco de Talentos.

## Telas por jornada

### VS — Síndico (10 telas)
- [[VS1]] Painel · [[VS2]] Explorar Parceiros · [[VS3]] Detalhe Parceiro · [[VS4]] Detalhe Promoção · [[VS5]] Ativar Promoção
- [[VS6]] Indicar Parceiro Local · [[VS7]] Criar Benefício Exclusivo · [[VS8]] Benefícios do Condomínio
- [[VS9]] Parceiros Indicados · [[VS10]] Histórico de Ativações

### VE — Empresa (6 telas)
- [[VE1]] Painel · [[VE2]] Explorar Parceiros · [[VE3]] Detalhe Parceiro
- [[VE4]] Detalhe Promoção (apenas abertas) · [[VE5]] Ativar · [[VE6]] Histórico

### VM — Morador (7 telas)
- [[VM1]] Feed do Bairro · [[VM2]] Detalhe Parceiro · [[VM3]] Detalhe Promoção · [[VM4]] Ativar Promoção
- [[VM5]] Benefícios do Condomínio · [[VM6]] Parceiros Indicados pelo Síndico · [[VM7]] Histórico de Ativações

### VZ — Parceiro (6 telas)
- [[VZ1]] Cadastro · [[VZ2]] Perfil · [[VZ3]] Criar Promoção · [[VZ4]] Promoções Ativas · [[VZ5]] Métricas · [[VZ6]] Histórico

## Regras críticas (canônicas)
1. **Promoções exclusivas** (`exclusiva_condominio`) só são visíveis a **moradores do condomínio selecionado**. Síndico vê (se criou); Empresa **NÃO** vê; outros moradores **NÃO** veem.
2. **Promoções abertas** (`aberta_bairro`) visíveis a moradores + síndicos + empresas da região.
3. **Sistema de Cupons (D-064 ativo)**: ID composto `ID_LOJA(5) + SEQ(3) + PALAVRA(5)` + **lock 4 horas** anti-replay/anti-fraude.
4. **Empresa NÃO cria promoções** nem cadastra parceiros. Apenas explora e ativa abertas.
5. **Parceiro NÃO publica em condomínios**, NÃO registra serviços, NÃO acessa painel empresarial.
6. **Síndico NÃO edita** promoções criadas pelo parceiro; pode criar exclusiva (VS7) em parceria.
7. **Métricas separadas por actor_type** (`morador | sindico | empresa`) — 3 contadores distintos.

## 7 Tipos de benefício
`desconto_percentual | desconto_fixo | produto_gratuito | combo_promocional | avaliacao_gratuita | brinde | beneficio_exclusivo`

## Connect Me Síndico → Parceiro (Req 19.3)
Síndico pode iniciar contato direto via Connect Me propondo promoção exclusiva (alternativo a VS6 que só envia link de cadastro).

## Fontes canônicas
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md`
- `vault/90-archive/master-sindico-pdfs-cliente-originais/MÓDULO VIZINHANÇA - MORADOR.md`
- `vault/90-archive/master-sindico-pdfs-cliente-originais/Jornada do Parceiro da Vizinhança.md`
- `vault/50-projects/master-sindico/client-canvas/Fluxo Vizinhanca.md`
- `vault/50-projects/master-sindico/client-canvas/Jornadas Vizinhanca.md`
- `vault/50-projects/master-sindico/00-product/sub-produtos/08-vizinhanca.md`
- `backend/.kiro/specs/master-sindico/requirements/commercial.md` Req 27-27.3

## Ligações
- [[../../../../00-product/sub-produtos/08-vizinhanca]]
- [[../../../../00-product/sub-produtos/02-connect-me]]
- [[../sindico/_moc]] · [[../empresa/_moc]] · [[../morador/_moc]]

## Gaps consolidados
- Categorias canônicas (38+) sem catálogo final unificado (cliente reorganizou).
- Sistema de Cupons D-064 — formato literal de PALAVRA(5) (dicionário curado vs aleatória).
- Endpoints REST de ativação/lock 4h sem ADR.
