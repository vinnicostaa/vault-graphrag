---
title: ONB-S4 — Síndico Governança e Estrutura
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding, sindico]
project: master-sindico
persona: sindico
category: ONB
screen_id: ONB-S4
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-S4 — Síndico · Governança e Estrutura (Tela 4, 15 marcadores)

## Finalidade

Autodeclaração de **15 marcadores** de governança que o síndico possui (ABRACS, RC, jurídico, contábil, segurança trabalho, LGPD, NR-1, compliance, Reclame Aqui, outras certificações, premiações). Exibidos em selos no perfil público (sujeitos a disclaimer — §3 functional).

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] — TELA 4 Síndico
- [[../../../../../../90-archive/master-sindico-pdfs-cliente-originais/ONBOARDING DOS PERFIS]]
- [[../../../../04-requirements/registration-data]] §2.3 (Governance Markers)

## Persona e ABAC

- `syndic` pré-ativação.
- ABAC: `identity.draft.write` + `institutional.syndic.markers`.

## Fluxo da tela

1. Mensagem principal (literal): **"Gestão também é estrutura."**
2. Mensagem de apoio (literal): _"Esses marcadores ajudam a demonstrar cuidado com riscos, organização e boas práticas administrativas."_
3. Lista de 15 marcadores (checkboxes + campo "qual" quando aplicável):
   1. Membro ABRACS
   2. Seguro de Responsabilidade Civil
   3. Assessoria Jurídica (+ qual)
   4. Assessoria Contábil (+ qual)
   5. Assessoria em Segurança do Trabalho (+ qual)
   6. Conformidade com LGPD
   7. Conformidade com NR-1
   8. Programa de Compliance
   9. Selo Reclame Aqui (+ link)
   10. Outras certificações (campo aberto, 500 chars)
   11. Premiações (campo aberto, 500 chars)
   12-15. Derivados (ex: certificação própria, ISO aplicável, outras) — subcategorias autodeclaradas.
4. **Disclaimer fixo** (literal): _"Marcadores são informativos, baseados em autodeclaração, e não representam certificação oficial da plataforma."_
5. Botão **[Continuar]** → [[ONB-S5]].

## Componentes

- Lista checkbox com sub-form condicional.
- Disclaimer fixo no rodapé.
- Contador "X/15 marcadores habilitados" (opcional, informativo).

## Estados (loading/empty/error/success)

- **Success**: auto-save.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Salvar marcadores | `/api/v1/onboarding/draft` | PATCH | `{step:4, markers:{...}}` | 204 |

## Regras de negócio críticas

- **15 marcadores canônicos** (Req 6.2 `institutional.md`).
- Disclaimer obrigatório em UI (cliente exige).
- Nenhum marcador é validado pela plataforma M1.
- Evidências (upload) **backlog M3+** — anti-fraude.

## Ligações

- Origem: [[ONB-S3]].
- Destino: [[ONB-S5]].
- Relacionados: [[../../../../04-requirements/registration-data]].

## Gaps/ressalvas

- Validação pela Master Síndico de cada marcador: fora de M1.
- Badge público: só aparece para N2/N3 (N1 sem perfil público — ver [[ONB-S5]]).

## Links

- [[_moc]]
- [[ONB-S3]]
- [[ONB-S5]]
