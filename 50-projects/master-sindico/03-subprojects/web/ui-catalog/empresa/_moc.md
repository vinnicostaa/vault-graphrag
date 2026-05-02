---
title: MOC — UI Catalog Empresa (E1-E16)
type: moc
tags: [master-sindico, ui-catalog, web, empresa, moc]
project: master-sindico
persona: empresa
created: 2026-04-23
---

# MOC — UI Catalog Empresa

Mapa de conteúdo para as **16 telas da Jornada da Empresa** (E1-E16). Persona: `enterprise` (Administrador + Operador Técnico). Plan tier mínimo: `plus` (Trial bloqueia Connect Me, Marketing Link e gestão de agência).

## Telas (16)

| ID | Nome | Sub-produto | Plan |
|---|---|---|---|
| [[E1]] | Painel Empresarial (home 10 cards + 7 indicadores) | connect-me | plus+ |
| [[E2]] | Oportunidades (Connect Me recebido, contato oculto) | connect-me | plus+ |
| [[E3]] | Recusar Oportunidade (6 motivos) | connect-me | plus+ |
| [[E4]] | Enviar Proposta (reveal LGPD-logado) | connect-me | plus+ |
| [[E5]] | Minhas Propostas (pipeline) | connect-me | plus+ |
| [[E6]] | Atividade Técnica (risco + natureza) | reputacao | plus+ |
| [[E7]] | Negócios Fechados (estados ciclo deal) | connect-me | plus+ |
| [[E8]] | Termo de Execução (LGPD log versionado, imutável dupla) | compliance | plus+ |
| [[E9]] | Registrar Execução (rascunho→enviado→aprovado→publicado) | reputacao | plus+ |
| [[E10]] | Dashboard Empresarial | reputacao | plus+ |
| [[E11]] | Avaliações Recebidas (5 critérios 1-10, privada opcional) | reputacao | plus+ |
| [[E12]] | Perfil Institucional (logo, portfolio, vídeos) | conteudo-videos | plus+ |
| [[E13]] | Usuários da Empresa (Administrador / Operador Técnico) | governanca-institucional | plus+ |
| [[E14]] | Gestão de Agência (delegação) | marketing-agencias | plus+ |
| [[E15]] | Compliance Empresarial (declaração anual fiscal/técnica/trabalhista) | compliance | plus+ |
| [[E16]] | Formulário Marketing Link (intenção, NÃO vínculo) | marketing-agencias | plus+ |

## Regras transversais (aplicam às telas E1-E16)

- **3 pilares Empresa**: relacionamento (Connect Me) + registro técnico + reputação
- **R4 — origem externa pendente**: tudo que vem de empresa entra como "Aguardando validação" do síndico
- **R3 — nada deletado**: editar/ocultar com trilha; INSERT-only
- **R5 — pós-fechamento imutável**: alteração só via Adendo Formal
- **R10 — LGPD**: revelação de dados pessoais é logada (timestamp + IP + finalidade + versão termos)
- **Operador Técnico** vê subset: registrar execução, atividade técnica, comunicados, vídeos institucionais. NÃO envia proposta nem aceita Connect Me

## Regras críticas específicas (do briefing)

- **Agência NÃO acessa Connect Me / Timeline / dados síndico** — apenas vídeos delegados
- **Marketing Link ([[E16]]) ≠ Connect Me**: intenção, não cria vínculo automático
- **Vínculo formal empresa↔agência**: só via [[E14]] (convite + token + checkbox autorização)
- **[[E8]] imutável** após assinatura dupla (empresa + síndico)
- **[[E11]] avaliação pode ser privada** (escolha da empresa) — afeta visibilidade pública mas não a média

## Sub-produtos relacionados

- [[../../../../00-product/sub-produtos/02-connect-me]] — base do funil comercial
- [[../../../../00-product/sub-produtos/03-reputacao]] — execuções + avaliações + dashboard
- [[../../../../00-product/sub-produtos/04-conteudo-videos]] — vídeos institucionais + perfil
- [[../../../../00-product/sub-produtos/09-compliance]] — declarações anuais + LGPD
- [[../../../../00-product/sub-produtos/10-marketing-agencias]] — delegação + Marketing Link

## Fontes canônicas (leitura obrigatória)

- `vault/90-archive/master-sindico-pdfs-cliente-originais/JORNADA DA EMPRESA.md` (E1-E7 + E16)
- `vault/90-archive/master-sindico-pdfs-cliente-originais/JORNADA DA EMPRESA-1.md` (E1-E15 ampliado)
- `vault/90-archive/from-development-content-2026-04-23/contents/####################TELA DO MORADOR#####(1).md` (Painel Empresarial — E1, E2, E4, E5, E7, E8, E9, E11, E12, E14)
- `vault/90-archive/from-development-content-2026-04-23/02-Jornadas/Inventario-Telas.md` (linhas 110-145 — inventário canônico)
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Connect-Me-4-direcoes.md` (4 direções + Marketing Link)
- `vault/90-archive/master-sindico-pdfs-cliente-originais/COMO A EMPRESA VÊ O CURRÍCULO DO MORADOR (1).md` (relacionado a banco-talentos — fora desta entrega)

## Catálogo paralelo

- [[../marketing/_moc]] — 8 telas Agência (MK1-MK8)
- [[../sindico/_moc]] — telas síndico (S1-S31, C1-C11) (a fazer)
- [[../morador/_moc]] — telas morador (M1-M15) (a fazer)

## Links

- [[../../../../CLAUDE]]
- [[../../ui-catalog]]
- [[../../../../client-canvas/Master Canvas]]
- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Inventario-Telas]]

## Gaps consolidados (para futura iteração)

1. Tela de **adendo formal** (após [[E8]] confirmado) não foi especificada nesta entrega
2. **Perfil público de agência** (vitrine que origina [[E16]]) fora do escopo — abrir spec
3. **Rate-limit Marketing Link** por (empresa, agência) sem regra concreta
4. **Limite usuários por empresa** (Plus vs Pro) sem definição comercial
5. **Wébsocket vs polling** para indicadores em tempo real (E1, E10): pendente arquitetura
