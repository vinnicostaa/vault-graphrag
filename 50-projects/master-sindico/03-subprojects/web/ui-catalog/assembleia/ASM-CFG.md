---
title: ASM-CFG — Configurar Assembleia
type: ui-screen
tags: [master-sindico, ui-catalog, web, assembleia]
project: master-sindico
persona: sindico
category: ASM
screen_id: ASM-CFG
sub_produto: assembleia-live
status: specification
stack: web
created: 2026-04-24
---

# ASM-CFG — Configurar Assembleia

## Finalidade

Tela de criação/edição dos dados estruturais da assembleia: tipo, datas, horários de convocação, modalidade, local/link, tempo de fala. É a fundação editável antes de LOCK_Pauta (§4.5 funcional) disparar após "Publicar".

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/03-Modulos/Assembleia-Live-Funcional]] §4.1 (Criação da assembleia)
- [[../../../../../../90-archive/master-sindico-pdfs-cliente-originais/ARQUITETURA FUNCIONAL DO MÓDULO ASSEMBLEIA]] bloco 2
- [[../../../../04-requirements/functional/assembly]] Req 48-49
- [[../../../../client-canvas/Arquitetura Assembleia]]

## Persona e ABAC

- **Primária**: `syndic` com `assembly.create/update` no tenant do condomínio.
- **Secundária**: `administradora` com permissão `assembly.edit_pre_publish` (caso esteja vinculada e autorizada).
- Auditoria: toda edição em estado `draft` grava `audit_trail` com action `assembly.updated`.

## Fluxo da tela

1. Campo **Tipo**: `ordinária | extraordinária | implantação` (enum).
2. **Data** (date picker, futuro).
3. **Horário 1ª convocação** + **Horário 2ª convocação** (HH:MM, BRT — GR-033 timezone).
4. **Modalidade**: `presencial (MVP)` ativo; `híbrida` e `online` visíveis mas desabilitados (badge "em breve").
5. **Local físico** (texto livre + autocomplete CEP via ViaCEP).
6. **Link digital** (opcional, futuro — inativo no MVP).
7. **Descrição geral** (rich text, até 2000 chars).
8. **Tempo máximo de fala por morador**: radio `2 min | 3 min`; checkbox "permitir prorrogação de 2 min".
9. Botões: **Salvar rascunho** · **Avançar para Pauta** ([[ASM-PAUTA]]).

## Componentes

- Form validado com Zod/Valibot (front) + validator Go (back).
- Date picker com bloqueio de datas < hoje+1d.
- Inputs `aria-required` e `aria-describedby` para mensagens de erro.
- Auto-save a cada 20s em Redis (chave `assembly:draft:{id}`).

## Estados (loading/empty/error/success)

- **Loading**: spinner ao buscar rascunho existente.
- **Error**: banner de campo inválido (1ª convocação >= 2ª, data passada).
- **Success**: toast "Rascunho salvo" + CTA avançar habilita.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Criar draft | `/api/v1/assemblies` | POST | `{condominium_id, type, date, call_1, call_2, modality, venue, speech_limit}` | `{assembly_id, status:"draft"}` |
| Atualizar | `/api/v1/assemblies/:id` | PATCH | campos delta | 204 |
| Buscar draft | `/api/v1/assemblies/:id` | GET | — | Assembly |

## Regras de negócio críticas

- **1ª convocação < 2ª convocação** no mesmo dia (ou 2ª em dia seguinte — Lei 4.591/64 permite); front valida + back invariante.
- Modalidade `híbrida` e `online` persistidas se preenchidas mas **não ativam** fluxo ao vivo no M1 (travadas no backend — INV-ASM-modalidade-mvp).
- Edição livre **apenas** enquanto `status = draft`. Após publicar ([[ASM-PUB]]), vira LOCK_Pauta + alterações exigem assembleia nova.
- Timezone SEMPRE America/Sao_Paulo no render, UTC no banco (GR-033).

## Ligações

- Origem: [[ASM-DASH]].
- Destino: [[ASM-PAUTA]], [[ASM-EDITAL]], [[ASM-PUB]].
- Relacionados: [[../../../../client-canvas/Arquitetura Assembleia]].

## Gaps/ressalvas

- Modalidade híbrida/online: UI mostra placeholder mas não validada end-to-end no M1 (ver Q27 quebras-detectadas).
- Campo "auditor" (§3.1 cadastro-base) fica em tela separada de configuração do condomínio, não aqui.

## Links

- [[_moc]]
- [[ASM-PAUTA]]
- [[../../../../STATE]]
