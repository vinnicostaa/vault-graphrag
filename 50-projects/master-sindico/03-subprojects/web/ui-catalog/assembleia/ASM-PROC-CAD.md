---
title: ASM-PROC-CAD — Cadastro de Procuração (Morador)
type: ui-screen
tags: [master-sindico, ui-catalog, web, assembleia]
project: master-sindico
persona: morador
category: ASM
screen_id: ASM-PROC-CAD
sub_produto: assembleia-live
status: specification
stack: web
created: 2026-04-24
---

# ASM-PROC-CAD — Cadastro de Procuração pelo Morador

## Finalidade

Permite ao morador (titular) cadastrar procuração indicando que outra pessoa (procurador) votará em seu nome na assembleia. Dados digitais são apenas **declaração** — a apresentação física da procuração é obrigatória na entrada, validada em [[ASM-PROC-VAL]] pela administradora.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/03-Modulos/Assembleia-Live-Funcional]] §4.13 (Procurações)
- [[../../../../04-requirements/functional/assembly]] Req 55
- [[../../../../client-canvas/Fluxo Assembleia Ao Vivo]]

## Persona e ABAC

- **Primária**: `resident` (apenas titular cadastra procuração para sua unidade — dependente não).
- ABAC: `assembly.proxy.create` com `tenant_id = condominium_id` + `unit_id = user.unit_id`.

## Fluxo da tela

1. Header "Procuração para assembleia X".
2. **Representado** (pré-preenchido com dados do morador logado):
   - Bloco e unidade
   - Nome e CPF
3. **Procurador** (campos manuais):
   - Bloco e unidade (do procurador, se morador do mesmo condomínio)
   - Nome completo
   - CPF
4. **Checkbox** "Declaro ciência e responsabilidade: a procuração deve ser apresentada fisicamente no dia da assembleia para validação".
5. Botão **[Cadastrar procuração]**.
6. Confirmação com mensagem destacada: **"Você precisa apresentar a procuração física para validação no dia."**

## Componentes

- Form com CPF masked + validação dígito verificador.
- Autocomplete de morador do mesmo condomínio (opcional, respeitando LGPD — exibe só após confirmação de CPF).
- Alert box amarelo com lembrete físico.

## Estados (loading/empty/error/success)

- **Error**: CPF inválido · procurador é o próprio morador · procuração duplicada na mesma assembleia.
- **Success**: toast + listagem de procurações cadastradas + status "aguardando validação".

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Criar procuração | `/api/v1/assemblies/:id/proxies` | POST | `{representado:{unit,block,name,cpf}, procurador:{unit?,block?,name,cpf}, acknowledged:true}` | `{proxy_id, status:"pending_validation"}` |
| Listar minhas | `/api/v1/assemblies/:id/proxies?mine=true` | GET | — | array Proxy |
| Cancelar | `/api/v1/assemblies/:id/proxies/:proxy_id` | DELETE | — | 204 |

## Regras de negócio críticas

- Apresentação permitida **até antes do clique "Iniciar assembleia"** ([[ASM-PAINEL-SIND]]).
- **Validade do vínculo**: **apenas 24h** após o encerramento da assembleia (§4.13 — depois perde efeito, dados arquivados).
- Procurador pode representar múltiplos titulares (sem limite técnico, convenção define).
- Efeito financeiro: voto adicional computado automaticamente respeitando pesos individuais de fração ideal.
- Requer ciência registrada antes ([[ASM-CIEN]]).

## Ligações

- Origem: [[ASM-CIEN]].
- Destino: [[ASM-PROC-VAL]] (administradora valida).
- Relacionados: [[ASM-VOTO]] (efeito na votação), [[../../../../client-canvas/Fluxo Assembleia Ao Vivo]].

## Gaps/ressalvas

- Upload opcional de foto/scan da procuração: discutir com cliente (reduz necessidade de validação física?). Decisão atual: só físico, M1.
- Unidade do procurador no mesmo condomínio é **opcional** (procurador pode ser externo — caso morador delegue a advogado/familiar não-morador).

## Links

- [[_moc]]
- [[ASM-PROC-VAL]]
- [[ASM-VOTO]]
