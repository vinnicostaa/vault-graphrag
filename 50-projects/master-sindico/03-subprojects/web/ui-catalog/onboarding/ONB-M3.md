---
title: ONB-M3 — Morador Vínculo com o Condomínio
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding, morador]
project: master-sindico
persona: morador
category: ONB
screen_id: ONB-M3
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-M3 — Morador · Vínculo com o Condomínio (Tela 3)

## Finalidade

Vincula o morador ao condomínio via código de 8 caracteres alfanuméricos ou busca por CEP + endereço. Este vínculo é pré-requisito para acessar timeline, receber convocações de assembleia e participar da governança.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] — TELA 3 Morador
- [[../../../../../../90-archive/master-sindico-pdfs-cliente-originais/ONBOARDING DOS PERFIS]]
- [[../../../../04-requirements/registration-data]] §1.1

## Persona e ABAC

- **Primária**: `resident` pré-ativação.
- ABAC: `institutional.membership.request` (pendente de confirmação).

## Fluxo da tela

1. Mensagem principal (literal): **"Agora vamos conectar você ao seu condomínio."**
2. Mensagem de apoio (literal): _"Isso permite acesso a registros, comunicações e conteúdos relacionados ao local onde você vive."_
3. Campos:
   - **CEP** (8 dígitos, autocompleta via ViaCEP adapter — XD-018).
   - **Endereço** (rua/nº/complemento — preenchidos automaticamente; editáveis).
   - **Condomínio**: busca por **nome** OU **código de 8 caracteres alfanuméricos** (fornecido pelo síndico).
4. Resultado da busca: lista de condomínios matching (até 10).
5. Ao selecionar condomínio: **[Solicitar vínculo]** → cria membership pendente.
6. Botão **[Continuar]** → [[ONB-M4]].

## Componentes

- Input CEP com mask + lookup automático.
- Busca de condomínio com autocomplete.
- Card do condomínio selecionado com síndico/administradora visíveis.

## Estados (loading/empty/error/success)

- **Empty**: "Não achou? Peça o código ao seu síndico" + CTA para continuar sem vínculo (status `incomplete`).
- **Error**: CEP inexistente · condomínio não encontrado.
- **Success**: card confirmando + mensagem "Aguarde validação do síndico".

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Resolver CEP | `/api/v1/cep/resolve` | GET | `?cep=...` | `{street, city, state}` |
| Buscar condomínio | `/api/v1/condominiums/search` | GET | `?q=...&cep=...` | array Condominium |
| Solicitar membership | `/api/v1/condominiums/:id/memberships/request` | POST | `{unit, block}` | `{membership_id, status:"pending"}` |
| Salvar draft | `/api/v1/onboarding/draft` | PATCH | `{step:3, ...}` | 204 |

## Regras de negócio críticas

- **Código de 8 chars alfanuméricos**: formato `[A-Z0-9]{8}` (ex: `CD2024AB`); gerado pela plataforma ao criar condomínio.
- Membership requer aprovação do síndico/administradora (INS-042) — status inicial `pending_syndic_approval`.
- Morador pode concluir onboarding sem vínculo aprovado (status final = `incomplete` até aprovação).
- ViaCEP via interface (D-056 agnóstico).

## Ligações

- Origem: [[ONB-M2]].
- Destino: [[ONB-M4]].
- Relacionados: [[../../../../04-requirements/registration-data]].

## Gaps/ressalvas

- Geolocalização automática (GPS) opcional para sugerir condomínios próximos: backlog M2.
- Morador em múltiplos condomínios: UX de adicionar segundo vínculo pós-onboarding — telas separadas em conta do usuário.

## Links

- [[_moc]]
- [[ONB-M2]]
- [[ONB-M4]]
