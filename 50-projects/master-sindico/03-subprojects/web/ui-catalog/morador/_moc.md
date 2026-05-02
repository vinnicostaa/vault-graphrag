---
title: MOC — UI Catalog Morador (Web)
type: moc
tags: [master-sindico, ui-catalog, web, morador, moc]
project: master-sindico
persona: morador
sub_produto: governanca-institucional
status: specification
stack: web
created: 2026-04-24
updated: 2026-04-24
---

# MOC — Catálogo UI Morador (Web)

Mapa de conteúdo das **15 telas (M1–M15)** da jornada do morador na app web. IDs estáveis derivados do `Inventario-Telas.md` canônico. Spec individual por tela em arquivos próprios.

## Fontes primárias

- `vault/90-archive/from-development-content-2026-04-23/contents/####################TELA DO MORADOR#####(1).md` — texto literal do cliente (ini)
- `vault/90-archive/master-sindico-pdfs-cliente-originais/JORNADA DO MORADOR.md` — PDF original com regras estruturais (R1-R5)
- `vault/90-archive/from-development-content-2026-04-23/02-Jornadas/Inventario-Telas.md` — IDs M1-M15
- `vault/90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona.md` — onboarding morador (4 telas)
- `vault/50-projects/master-sindico/00-product/sub-produtos/01-governanca-institucional.md` — escopo institucional

## Regras estruturais do morador (R1-R5)

- **R1**: Linha do Tempo é a principal fonte de informação
- **R2**: Morador NÃO publica conteúdo (só visualiza, responde, participa, avalia)
- **R3**: Vínculo com unidade é obrigatório (sem vínculo → sem acesso)
- **R4**: Dependentes não podem alterar status, remover titular ou incluir novos dependentes
- **R5**: Unidade vazia → dependentes removidos automaticamente

## Catálogo

| ID | Título | Status | Persona | Sub-produto | Tier mínimo |
|---|---|---|---|---|---|
| [[M1]] | Painel do Morador | specification | Morador | Governança Institucional | any |
| [[M2]] | Selecionar Condomínio | specification | Morador | Governança Institucional | any |
| [[M3]] | Buscar Condomínio por ID | specification | Morador | Governança Institucional | any |
| [[M4]] | Identificação da Unidade | specification | Morador | Governança Institucional | any |
| [[M5]] | Termo de Ciência de Vínculo | specification | Morador | Governança Institucional + Compliance | any |
| [[M6]] | Linha do Tempo | specification | Morador | Governança Institucional | any |
| [[M7]] | Detalhe da Publicação LT | specification | Morador | Governança Institucional | any |
| [[M8]] | Plano Diretor (visão morador) | specification | Morador | Governança Institucional | any |
| [[M9]] | Detalhe Ação PD (morador) | specification | Morador | Governança Institucional | any |
| [[M10]] | Eventos | specification | Morador | Governança Institucional | any |
| [[M11]] | Comunicados | specification | Morador | Governança Institucional | any |
| [[M12]] | Avaliar Gestão (bimestral obrigatória) | specification | Morador | Governança + Reputação | any |
| [[M13]] | Meu Vínculo | specification | Morador | Governança Institucional | any |
| [[M14]] | Gerenciar Dependentes | specification | Morador (Titular) | Governança Institucional | any |
| [[M15]] | Encerrar Vínculo | specification | Morador | Governança + Compliance | any |

## Fluxos principais

### Onboarding novo vínculo
M3 (busca por ID) → M4 (unidade) → M5 (termo) → M1 (painel)

### Uso diário
M1 (home) → M6/M8/M10/M11 (consumo de informação) → M7/M9 (detalhes)

### Avaliação obrigatória
Gatilho bimestral → M12 (bloqueante) → M1 (desbloqueio)

### Gestão do vínculo
M1 → M13 → M14 (titular gerencia dependentes) ou M15 (encerrar)

## Ligações cross-vault

- [[../../../../CLAUDE]] — contrato do subproject web
- [[../../design-system]] — tokens, componentes
- [[../../ui-catalog]] — catálogo geral (todas as personas)
- [[../../requirements]] — requisitos web
- [[../../../mobile/ui-catalog/morador/_moc]] — equivalente mobile (a criar)
- [[../../../backend/requirements]] — endpoints
- [[../../../../00-product/sub-produtos/01-governanca-institucional]] — sub-produto base
- [[../../../../client-canvas/_moc]] — canvas do cliente

## Gaps agregados (aparecem em várias telas)

- **Inconsistência cards M1**: cliente lista 4 cards (LT/PD/Eventos/Votação) mas PDF JORNADA M5 tem 6 (inclui Comunicados, Avaliação, Meu Vínculo). Decidir layout final com cliente.
- **Estado empty pouco documentado**: várias telas (M2, M6, M8, M10, M11) não têm texto canônico para empty — usaram-se inferências.
- **Endpoints inferidos**: cliente não documenta API; todos os endpoints listados são deduzidos da estrutura de domínios `institutional/*` e marcados como tal.
- **Periodicidade da avaliação (M12)**: cliente diz "obrigatória" mas não define janela de tolerância nem comportamento exato do bloqueio.
- **Foto de identificação (M4)**: capturada no momento, mas sem definição de OCR/biometria — tratar como evidência humana no MVP.

## Status

15/15 telas especificadas. Próximo: revisão dual-check com agentes de produto + sincronização com mobile/ui-catalog/morador.
