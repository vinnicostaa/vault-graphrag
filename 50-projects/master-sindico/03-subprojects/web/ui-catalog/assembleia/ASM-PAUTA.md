> ✅ **Q40 RESOLVIDA 2026-04-24** via [[../../../../STATE#d-104-—-avaliação-escondida-de-gestão-em-eleição-ativada-fecha-q40|D-104]]. Ver [[ASM-AVAL-ELEICAO]] e [[../../../../01-domain/invariants#inv-asm-023|INV-ASM-023]]. Referências a "Q40 pendente" neste arquivo são histórico; a regra está canonizada.

---
title: ASM-PAUTA — Cadastrar Itens da Pauta
type: ui-screen
tags: [master-sindico, ui-catalog, web, assembleia]
project: master-sindico
persona: sindico
secondary_persona: administradora
category: ASM
screen_id: ASM-PAUTA
sub_produto: assembleia-live
status: specification
stack: web
created: 2026-04-24
---

# ASM-PAUTA — Cadastrar Itens da Pauta

## Finalidade

Cadastro de **itens ilimitados** da pauta da assembleia, contemplando os 5 tipos canônicos (informativo, votação simples, votação por fração ideal, escolha de fornecedor, eleição). É o insumo obrigatório para gerar edital automático ([[ASM-EDITAL]]) e base jurídica da deliberação.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/03-Modulos/Assembleia-Live-Funcional]] §4.2, §4.3, §4.4 (tipos e regras por tipo)
- [[../../../../../../90-archive/master-sindico-pdfs-cliente-originais/MÓDULO DAS TELAS ASSEMBLEIA MASTER SÍNDICO]] — pauta
- [[../../../../04-requirements/functional/assembly]] Req 50

## Persona e ABAC

- **Primária**: `syndic` (`assembly.agenda.write`).
- **Secundária**: `administradora` pode adicionar **observações** (§3.4), não itens, salvo permissão explícita.

## Fluxo da tela

1. Lista de itens (drag-to-reorder) com badge do tipo.
2. Botão **[+ Novo item]** abre modal:
   - Título (obrigatório).
   - Descrição (rich text).
   - Seletor **Informativo | Deliberativo**; se deliberativo, mostra sub-seletor de tipo (5 opções).
   - **Impacto financeiro**: toggle sim/não; se sim, campo valor estimado (BRL, cents).
   - **Quórum exigido para aprovação** (%). Placeholder conforme tipo (ex: simples 50%, fração ideal por peso).
   - **Anexo de documento** (upload PDF/imagem, até 10MB).
   - **Anexo de vídeo explicativo** (opcional, link YouTube/Mux — lock 90d não se aplica aqui).
3. Botão **Salvar item** → adiciona à lista.

## Componentes

- Modal com sub-forms condicionais por tipo.
- Uploader com progress bar (S3 presigned URL — [[../../backend/README]] infra).
- Chips coloridos por tipo (info cinza, simples azul, fração dourado, fornecedor verde, eleição roxo).

## Estados (loading/empty/error/success)

- **Empty**: CTA grande "Adicione o primeiro item da ordem do dia".
- **Error**: validação inline (título obrigatório, quórum 0-100).
- **Success**: item aparece na lista + toast.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Listar itens | `/api/v1/assemblies/:id/agenda` | GET | — | array AgendaItem |
| Criar item | `/api/v1/assemblies/:id/agenda` | POST | `{title, description, kind, informative, financial_impact, amount_cents, quorum_pct, docs[], video_url}` | `{item_id, order}` |
| Reordenar | `/api/v1/assemblies/:id/agenda/reorder` | PATCH | `{order:[ids]}` | 204 |
| Remover | `/api/v1/assemblies/:id/agenda/:item_id` | DELETE | — | 204 |

## Regras de negócio críticas

- **Itens ilimitados** (§4.2) — sem cap de quantidade.
- 5 tipos canônicos (§4.3): `informativo`, `votacao_simples`, `votacao_fracao_ideal`, `escolha_fornecedor`, `eleicao`.
- Por tipo (§4.4): informativo não gera voto; simples/fração geram enquete preliminar + voto oficial; fornecedor adiciona comparação de propostas; eleição gera avaliação de gestão escondida (Q40 pendente — ver [[../../../../../../90-archive/from-development-content-2026-04-23/regras-de-negocio/quebras-detectadas|Q40]]).
- **LOCK_Pauta (§4.5)**: após publicar ([[ASM-PUB]]), **não** pode adicionar/alterar título/descrição/quórum/anexos estruturantes.
- Valor em cents (GR-010 money-handling).

## Ligações

- Origem: [[ASM-CFG]].
- Destino: [[ASM-EDITAL]], [[ASM-ENQP]] (enquetes preliminares geradas automaticamente).
- Relacionados: [[../../../../client-canvas/Arquitetura Assembleia]].

## Gaps/ressalvas

- Q40: "eleição gera avaliação escondida" não mapeada em `assembly.md` canônico — pendência de requirement.
- Upload de vídeo explicativo: host (Mux vs YouTube) não decidido — abrir pendência.

## Links

- [[_moc]]
- [[ASM-EDITAL]]
- [[ASM-ENQP]]
