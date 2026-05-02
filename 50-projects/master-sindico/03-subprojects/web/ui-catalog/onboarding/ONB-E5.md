---
title: ONB-E5 — Empresa Conformidade (25 marcadores)
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding, empresa]
project: master-sindico
persona: empresa
category: ONB
screen_id: ONB-E5
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-E5 — Empresa · Conformidade, Governança e Boas Práticas (25 marcadores)

## Finalidade

Autodeclaração dos **25 marcadores** de conformidade que a empresa possui (ISOs, ESG, selos, NRs, LGPD, compliance, certificações próprias). Base dos selos exibidos no perfil institucional.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] — TELA 5 Empresa (lista literal dos 25)
- [[../../../../04-requirements/registration-data]] §3.3

## Persona e ABAC

- `enterprise` pré-ativação.
- ABAC: `institutional.company.markers`.

## Fluxo da tela

1. Lista de **25 marcadores** (checkboxes + sub-campos):
   - Membro ABRACS · Programa de Compliance · Assessoria Jurídica · Assessoria Contábil · Assessoria em Segurança do Trabalho
   - Seguro de Responsabilidade Civil · Responsável Técnico · CIPA · NRs aplicáveis (campo aberto)
   - Regularidade em Conselhos Profissionais · Conformidade com LGPD · Conformidade com NR-1
   - ISO 9001 · ISO 14001 · ISO 45001 · ISO 37001 · ISO 19600 / ISO 37301
   - ESG · Selo Verde ou Sustentável · Selo Carbon Free · Selo EuReciclo · Selo Empresa Amiga da Criança
   - Selo Reclame Aqui · Premiações (campo aberto) · Outra certificação (aberto) · Certificação própria (aberto)
2. **Disclaimer literal** (obrigatório em UI): _"Esses marcadores são informativos, baseados em autodeclaração, e não representam certificação oficial da plataforma."_
3. Botão **[Continuar]** → [[ONB-E6]].

## Componentes

- Lista em grid (agrupada por categoria: compliance, gestão, ISO, ESG, selos).
- Tooltips explicativos.
- Disclaimer fixo.

## Estados (loading/empty/error/success)

- **Success**: auto-save.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Salvar marcadores | `/api/v1/onboarding/draft` | PATCH | `{step:5, markers:{25 keys}}` | 204 |

## Regras de negócio críticas

- Nenhum marcador validado pela plataforma (M1).
- Disclaimer obrigatório renderizado.
- Campo "Certificação própria": até 500 chars + sem contato/preço (regra similar COM-003).

## Ligações

- Origem: [[ONB-E4]].
- Destino: [[ONB-E6]].

## Gaps/ressalvas

- Evidências (upload certificado ISO): backlog M3+ anti-fraude.

## Links

- [[_moc]]
- [[ONB-E4]]
- [[ONB-E6]]
