---
title: ONB-M4 — Morador Participação e Convivência
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding, morador]
project: master-sindico
persona: morador
category: ONB
screen_id: ONB-M4
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-M4 — Morador · Participação e Convivência (Tela 4)

## Finalidade

Aceites obrigatórios (3 termos) antes de o morador ser ativado. Consentimentos LGPD, termos gerais e termo específico de avaliações.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] — TELA 4 Morador (literal)
- [[../../../../../../90-archive/master-sindico-pdfs-cliente-originais/ONBOARDING DOS PERFIS]]
- [[../../../../04-requirements/compliance-lgpd]]

## Persona e ABAC

- **Primária**: `resident` pré-ativação.
- ABAC: `identity.terms.accept`.

## Fluxo da tela

1. Mensagem principal (literal): **"Participar também é responsabilidade."**
2. Mensagem de apoio (literal): _"Avaliações, interações e participações em eventos devem refletir sua percepção real, sempre com respeito e boa-fé."_
3. **3 aceites obrigatórios** (checkboxes + link para texto completo):
   - **Termo Geral de Uso da Plataforma** (texto literal do cliente pendente; stub com template).
   - **Política de Privacidade e LGPD** (LGPD Art. 8º — consentimento).
   - **Termo de Avaliações e Participação** (boa-fé + respeito).
4. Botão **[Finalizar cadastro]** — habilita quando todos marcados → [[ONB-MFIN]].

## Componentes

- Accordion expansível com texto completo.
- Checkboxes com estado de versão de termo.
- Botão primário desabilitado até todos aceites.

## Estados (loading/empty/error/success)

- **Error**: alguma caixa não marcada → tooltip.
- **Success**: vai para confirmação.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Aceitar termos | `/api/v1/onboarding/terms/accept` | POST | `{versions:{general, privacy, participation}}` | `{accepted_at}` |

## Regras de negócio críticas

- Versões de termo versionadas por hash + data; mudança dispara re-aceite (LGPD-### consent versioning).
- `lgpd_logs` com `legal_basis = consent` para Privacidade e `legitimate_interest` para termo geral.
- Retenção 5 anos + purga pós-vencimento (INV-091).
- Texto oficial precisa de revisão jurídica Master Síndico — M1 stub.

## Ligações

- Origem: [[ONB-M3]].
- Destino: [[ONB-MFIN]].
- Relacionados: [[../../../../04-requirements/compliance-lgpd]].

## Gaps/ressalvas

- Texto **literal** dos 3 termos pendente de confirmação jurídica do cliente. Stubs a partir do template das outras personas.
- Re-aceite em mudança: fluxo implementado mas UX do banner de "termo atualizado" separado em M2.

## Links

- [[_moc]]
- [[ONB-M3]]
- [[ONB-MFIN]]
