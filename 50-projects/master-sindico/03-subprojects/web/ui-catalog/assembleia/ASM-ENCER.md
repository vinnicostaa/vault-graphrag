---
title: ASM-ENCER — Encerrar Assembleia + Registro Final
type: ui-screen
tags: [master-sindico, ui-catalog, web, assembleia]
project: master-sindico
persona: sindico
secondary_persona: administradora
category: ASM
screen_id: ASM-ENCER
sub_produto: assembleia-live
status: specification
stack: web
created: 2026-04-24
---

# ASM-ENCER — Encerrar Assembleia e Gerar Registro Final

## Finalidade

Checkpoint final da assembleia ao vivo: síndico/administradora clica **[Encerrar assembleia]**, sistema compila o **Registro Final** com todos os campos exigidos (§7 funcional) e gera a ata imutável (ADR-0033) pronta para assinatura externa.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/03-Modulos/Assembleia-Live-Funcional]] §7 (Encerramento)
- [[../../../../04-requirements/functional/assembly]] Req 57.4 / Req 60

## Persona e ABAC

- **Primária**: `syndic` OU `administradora`.
- ABAC: `assembly.close` + step-up MFA.

## Fluxo da tela

1. Checklist dos itens da pauta — todos devem estar homologados ([[ASM-HOML]]):
   - Resolvido · Adiado · Não-resolvido · Prejudicado.
2. Se algum item pendente: CTA **"Marcar como prejudicado"** para cada pendente ou impossibilita encerramento.
3. Preview do **Registro Final**:
   - Data de início
   - Data de término
   - Presidente
   - Secretários
   - Administradora
   - Auditor
   - Fornecedores presentes
   - Todos os votos (consolidado)
   - Procurações aprovadas
   - Inadimplentes
   - Liberações de voto
4. Botão **[Encerrar assembleia]** — confirmação dupla + step-up MFA.
5. Pós-encerramento: ata gerada (PDF imutável, SHA-256), link para [[ASM-REL]].

## Componentes

- Checklist semafórica por item.
- Preview de ata (template).
- Step-up modal.

## Estados (loading/empty/error/success)

- **Error**: item ainda em votação · mesa incompleta.
- **Success**: modal de celebração + CTA relatórios.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Encerrar | `/api/v1/assemblies/:id/close` | POST | `{final_note?}` + X-Step-Up-Token | `{closed_at, minutes_url, minutes_sha256}` |
| Pré-encerramento (validação) | `/api/v1/assemblies/:id/close/preflight` | GET | — | `{can_close:bool, pending:[item_ids]}` |

## Regras de negócio críticas

- **Ata imutável** (ADR-0033): após encerrar, PDF da ata fica append-only; SHA-256 registrado em `audit_trail` + `minutes_hash` na tabela de assembleia.
- Lei 4.591/64 compliance: dados mínimos da ata (participantes, deliberações, resultados) em template canônico.
- **Step-up MFA** (ADR-0026 endpoint crítico: "POST /api/v1/assemblies/:id/minutes/publish").
- Pós-encerramento: nenhum voto, presença, procuração pode ser alterado. Apenas adendo administrativo via admin com 4-eyes.
- Dispara domain event `assembly.closed` via NATS para BCs: content (sumário publicável), billing (sem efeito), institutional (atualiza governance score).

## Ligações

- Origem: [[ASM-HOML]] (último item homologado).
- Destino: [[ASM-REL]] (6 relatórios).
- Relacionados: [[../../../admin-privilegios]] §4.2 (adendo se necessário).

## Gaps/ressalvas

- Assinatura digital ICP-Brasil da ata: **fora do escopo M1** — ata gerada com hash, assinatura externa manual.
- Envio automático da ata para cartório: backlog M3+.

## Links

- [[_moc]]
- [[ASM-REL]]
- [[ASM-HOML]]
