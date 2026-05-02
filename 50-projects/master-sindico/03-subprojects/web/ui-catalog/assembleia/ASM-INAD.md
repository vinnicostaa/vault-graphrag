---
title: ASM-INAD — Upload Planilha Inadimplência
type: ui-screen
tags: [master-sindico, ui-catalog, web, assembleia]
project: master-sindico
persona: administradora
secondary_persona: sindico
category: ASM
screen_id: ASM-INAD
sub_produto: assembleia-live
status: specification
stack: web
created: 2026-04-24
---

# ASM-INAD — Upload Planilha Inadimplência

## Finalidade

Upload da relação de unidades inadimplentes até **1h antes da 1ª convocação** (§4.15). Sistema classifica moradores como **apto** ou **inapto** a votar. Permite override manual via botão **Liberar voto** (caso morador apresente comprovante de pagamento de última hora).

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/03-Modulos/Assembleia-Live-Funcional]] §4.15 (Inadimplência)
- [[../../../../04-requirements/functional/assembly]] Req 56.2

## Persona e ABAC

- **Primária**: `administradora` (upload).
- **Secundária**: `syndic` (upload e override).
- ABAC: `assembly.delinquency.upload`, `assembly.delinquency.override`.

## Fluxo da tela

1. Upload `.xlsx`/`.csv` com colunas: bloco · unidade · CPF (opcional) · valor inadimplente · dias em atraso.
2. Validação: estrutura + unidades existentes na planilha de fração ([[ASM-FRAC]]).
3. Commit → marca `can_vote=false` nessas unidades para esta assembleia.
4. Lista unidades inaptas com **CTA "Liberar voto"** por linha.
5. Modal **Liberar voto**: campos obrigatórios `responsável`, `horário`, `checkbox aprovação`, `reason`.

## Componentes

- Dropzone + validador (similar [[ASM-FRAC]]).
- Tabela inaptos com botão ação por linha.
- Modal override com campos obrigatórios.

## Estados (loading/empty/error/success)

- **Empty**: "Nenhuma inadimplência registrada — todas as unidades aptas".
- **Error**: cutoff expirou (> 1h antes de 1ª convocação).
- **Success**: contagem aptos/inaptos.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Upload | `/api/v1/assemblies/:id/delinquency/upload` | POST | multipart | `{import_id}` |
| Commit | `/api/v1/assemblies/:id/delinquency/imports/:id/commit` | POST | — | `{apt:n, unapt:n}` |
| Liberar voto | `/api/v1/assemblies/:id/delinquency/:unit_id/override` | POST | `{responsible, reason, approved:true}` | `{released_at}` |

## Regras de negócio críticas

- **Cutoff**: upload aceito até **1h antes da 1ª convocação** (§4.15). Após: bloqueado.
- Override (Liberar voto) permitido **durante** a assembleia pelo síndico ou administradora — registra `responsible`, `horário`, `reason` em audit.
- Audit trail obrigatório — ação sensível (impacta direito a voto).
- Morador inadimplente visualiza status no app com tom neutro (sem exposição pública — LGPD).

## Ligações

- Origem: [[ASM-FRAC]] (cruza dados).
- Destino: [[ASM-SIM]], [[ASM-VOTO]].
- Relacionados: [[../../../../client-canvas/Arquitetura Assembleia]].

## Gaps/ressalvas

- Integração automática com ERP da administradora (boleto/sistema financeiro): **fora do escopo M1** — upload manual de planilha.
- Dependente de titular: se titular inapto, dependente também? Política atual: sim, por unidade.

## Links

- [[_moc]]
- [[ASM-FRAC]]
- [[ASM-SIM]]
