---
title: ASM-PROC-VAL — Validação de Procuração (Administradora)
type: ui-screen
tags: [master-sindico, ui-catalog, web, assembleia]
project: master-sindico
persona: administradora
secondary_persona: sindico
category: ASM
screen_id: ASM-PROC-VAL
sub_produto: assembleia-live
status: specification
stack: web
created: 2026-04-24
---

# ASM-PROC-VAL — Validação de Procurações pela Administradora

## Finalidade

Fila de procurações cadastradas por moradores, com ações **Aprovar** ou **Rejeitar com justificativa**. A administradora (ou síndico com permissão) inspeciona dados do representado e do procurador, cruza com apresentação física na entrada, e formaliza digitalmente a validação.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/03-Modulos/Assembleia-Live-Funcional]] §4.13 (Procurações — fluxo administradora)
- [[../../../../04-requirements/functional/assembly]] Req 55.2

## Persona e ABAC

- **Primária**: `administradora` (role `enterprise` com sub-role `administradora-condomínio`).
- **Secundária**: `syndic` caso condomínio não tenha administradora vinculada (fallback).
- ABAC: `assembly.proxy.validate` no tenant do condomínio.

## Fluxo da tela

1. Tabela com filas **Pendentes (n)** · **Aprovadas (n)** · **Rejeitadas (n)**.
2. Cada linha: nome representado · unidade representado · nome procurador · CPF procurador · data cadastro · CTA.
3. Ação **[Aprovar]**: modal de confirmação. Ao confirmar, registra `action=proxy.approved`.
4. Ação **[Rejeitar]**: abre campo **Justificativa obrigatória** (≥20 chars); grava `action=proxy.rejected` + motivo.
5. Indicador "Presença física conferida" (checkbox auxiliar no aprovar — registrada em metadata).

## Componentes

- DataGrid com paginação server-side.
- Modal confirmação aprovar/rejeitar com contador anti-clique-duplo.
- Filtro por unidade e por nome.

## Estados (loading/empty/error/success)

- **Empty**: "Nenhuma procuração cadastrada — aguardando moradores".
- **Error**: validação falha (ex: procuração já validada por outro operador — 409 conflict).
- **Success**: linha muda de status + toast.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Listar | `/api/v1/assemblies/:id/proxies` | GET | `?status=pending` | array Proxy |
| Aprovar | `/api/v1/assemblies/:id/proxies/:proxy_id/approve` | POST | `{physical_verified:true}` | `{status:"approved"}` |
| Rejeitar | `/api/v1/assemblies/:id/proxies/:proxy_id/reject` | POST | `{reason:"..."}` | `{status:"rejected"}` |

## Regras de negócio críticas

- Janela de validação: **até o clique "Iniciar assembleia"** em [[ASM-PAINEL-SIND]].
- Ao aprovar, unidade representante **recebe o mesmo voto** da unidade representada — respeitando pesos individuais de fração ideal (§4.13 efeito).
- Reason obrigatório em rejeição (≥20 chars, categoria opcional — "dados incompletos" / "documento inválido" / "duplicidade").
- Audit trail obrigatório (INV-088 + audit append-only).
- Segregação de função: operador que aprovou não pode ser o mesmo morador representado.

## Ligações

- Origem: [[ASM-PROC-CAD]].
- Destino: [[ASM-SIM]] (afeta simulador de quórum), [[ASM-VOTO]].
- Relacionados: [[../../../../client-canvas/Arquitetura Assembleia]], [[../../../admin-privilegios]] §5 (4-eyes só não se aplica aqui — é operação delegada normal, não admin bypass).

## Gaps/ressalvas

- Upload de evidência (foto procuração) opcional — ver gap em [[ASM-PROC-CAD]].
- Sincronização com check-in físico da unidade representada: se representado também comparece, sistema deve permitir revogar a procuração na hora (pendência UX).

## Links

- [[_moc]]
- [[ASM-PROC-CAD]]
- [[ASM-SIM]]
