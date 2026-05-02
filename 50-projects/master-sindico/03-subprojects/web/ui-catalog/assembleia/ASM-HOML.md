---
title: ASM-HOML — Homologação de Item
type: ui-screen
tags: [master-sindico, ui-catalog, web, assembleia]
project: master-sindico
persona: sindico
secondary_persona: administradora
category: ASM
screen_id: ASM-HOML
sub_produto: assembleia-live
status: specification
stack: web
created: 2026-04-24
---

# ASM-HOML — Homologação de Item da Pauta

## Finalidade

Após encerrar a votação oficial de um item, síndico **ou** administradora homologa o resultado, definindo o status final: **resolvido / adiado / não-resolvido / prejudicado** (§5.9 e §6). A homologação é o gatilho para o item entrar na ata ([[ASM-ENCER]]) e torna-se imutável (ADR-0033).

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/03-Modulos/Assembleia-Live-Funcional]] §5.9 (Status dos itens) e §6 (Homologação)
- [[../../../../04-requirements/functional/assembly]] Req 57.3

## Persona e ABAC

- **Primária**: `syndic` OU `administradora` (qualquer um com permissão).
- ABAC: `assembly.item.homologate` + step-up MFA.

## Fluxo da tela

1. Modal pós-encerramento de votação.
2. Resumo:
   - Total de votos · % aprovação · quórum exigido vs aferido.
   - Se peso fracionado: soma ponderada.
3. Seleção de status final:
   - **Resolvido** (quórum atingido, decisão tomada)
   - **Adiado** (fica para próxima assembleia)
   - **Não-resolvido** (quórum insuficiente)
   - **Prejudicado** (perdeu objeto)
4. Campo **Observação** (opcional, mas recomendado) + campo `reason` obrigatório se status é Adiado ou Prejudicado.
5. Botão **[Homologar]** com confirmação step-up MFA.

## Componentes

- Modal com resumo cards.
- Radio de status com explicação inline.
- Step-up MFA embed.

## Estados (loading/empty/error/success)

- **Error**: step-up expirado · voto ainda aberto · reason faltando.
- **Success**: item move para "finalizado" + trilha registrada.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Homologar | `/api/v1/assemblies/:id/agenda/:item/homologate` | POST | `{status, observation?, reason?}` + X-Step-Up-Token | `{homologated_at, by_user_id}` |

## Regras de negócio críticas

- Item homologado é **imutável** (ADR-0033 ata imutável). Ajuste pós-homologação exige adendo formal ([[../../../admin-privilegios]] §4.2 — admin com adendo administrativo em `audit_trail`).
- Quórum aferido deve respeitar o quórum exigido definido em [[ASM-PAUTA]].
- Reason obrigatório (≥ 20 chars) para status **Adiado** e **Prejudicado**.
- Step-up MFA (ADR-0026) para rota de publicação derivada.
- Cada homologação gera evento append-only + atualiza hash de ata incremental.

## Ligações

- Origem: [[ASM-VOTO]] (após encerrar votação).
- Destino: próximo item ou [[ASM-ENCER]].
- Relacionados: [[../../../admin-privilegios]] (admin pode adendo, não edição).

## Gaps/ressalvas

- Homologação por **voto da mesa** (presidente homologa — §5.5): spec assume que síndico/administradora clica; detalhar UX da mesa em M2.
- Reversão extraordinária (erro grosseiro descoberto pós-homologação): só via adendo administrativo com 4-eyes (D-054).

## Links

- [[_moc]]
- [[ASM-VOTO]]
- [[ASM-ENCER]]
