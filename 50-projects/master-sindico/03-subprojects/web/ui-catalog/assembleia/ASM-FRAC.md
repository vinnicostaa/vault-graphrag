---
title: ASM-FRAC — Upload Planilha Fração Ideal
type: ui-screen
tags: [master-sindico, ui-catalog, web, assembleia]
project: master-sindico
persona: administradora
secondary_persona: sindico
category: ASM
screen_id: ASM-FRAC
sub_produto: assembleia-live
status: specification
stack: web
created: 2026-04-24
---

# ASM-FRAC — Upload Planilha Fração Ideal

## Finalidade

Uploader da planilha estrutural do condomínio (bloco · unidade · fração · CPF titular), que alimenta o cálculo de peso de voto em deliberações por fração ideal. **Importação única persistente** — nova importação só por correção ou mudança estrutural.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/03-Modulos/Assembleia-Live-Funcional]] §4.14 (Fração ideal)
- [[../../../../04-requirements/functional/assembly]] Req 56
- [[../../../../client-canvas/Arquitetura Assembleia]]

## Persona e ABAC

- **Primária**: `administradora` do condomínio (responsável pela planilha).
- **Secundária**: `syndic` com permissão (casos sem administradora).
- ABAC: `institutional.fraction.upload`.

## Fluxo da tela

1. Upload de arquivo `.xlsx` / `.csv` (template baixável).
2. Parser mostra preview das primeiras 20 linhas.
3. Validador exibe card de integridade:
   - Soma total das frações = 100% (ou parâmetro do condomínio).
   - Ausência de duplicidade (bloco+unidade único).
   - Ausência de campos em branco.
   - CPF com dígito verificador correto.
4. Se erros: exibe lista navegável (linha X · tipo de erro · sugestão).
5. Se ok: botão **[Confirmar importação]**.
6. Substitui planilha anterior (arquivada com timestamp).

## Componentes

- Dropzone multiformat.
- Preview tabular (TanStack Table).
- Validation summary com links "ir para linha".
- Modal de confirmação antes de sobrescrever.

## Estados (loading/empty/error/success)

- **Loading**: progress durante parse + validação (pode demorar 5-30s dependendo tamanho).
- **Error**: lista de erros não-fixáveis automaticamente.
- **Success**: badge verde + resumo (total unidades, total frações) + CTA para simulador [[ASM-SIM]].

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Upload | `/api/v1/condominiums/:id/fractions/upload` | POST | multipart xlsx | `{import_id, status:"validating"}` |
| Status | `/api/v1/condominiums/:id/fractions/imports/:import_id` | GET | — | `{status, errors[], preview}` |
| Confirmar | `/api/v1/condominiums/:id/fractions/imports/:import_id/commit` | POST | — | `{committed_at, units:n, sum:100}` |
| Template | `/api/v1/condominiums/fractions/template` | GET | — | xlsx download |

## Regras de negócio críticas

- **Persistência**: tipo `NUMERIC(5,4)` no PostgreSQL (nunca float — §11 NFR). Exemplo: `0.0125` = 1,25%.
- Soma total = 1.0000 (com tolerância ≤ 0.0001 para arredondamento).
- CPF titular opcional mas recomendado (cruza com tabela de moradores cadastrados).
- Importação substitui a anterior (anterior vai para histórico, não é deletada — INV append-only).
- Bloqueia deliberações por fração ideal até planilha válida.

## Ligações

- Origem: [[ASM-CFG]] (pré-requisito) ou [[ASM-DASH]].
- Destino: [[ASM-SIM]] (simulador), [[ASM-VOTO]] (peso na votação).
- Relacionados: [[../../../../04-requirements/functional/assembly]] Req 56.

## Gaps/ressalvas

- Mesclagem com cadastro de moradores: se CPF do titular na planilha ≠ CPF do morador cadastrado, como resolver? Política atual: warning, não bloqueio.
- Upload grande (> 10.000 linhas): considerar worker assíncrono. Default M1 síncrono para condomínios pequenos/médios.

## Links

- [[_moc]]
- [[ASM-SIM]]
- [[ASM-INAD]]
