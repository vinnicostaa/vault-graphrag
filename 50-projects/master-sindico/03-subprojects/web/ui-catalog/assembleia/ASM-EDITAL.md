---
title: ASM-EDITAL — Anexar / Gerar Edital Automático
type: ui-screen
tags: [master-sindico, ui-catalog, web, assembleia]
project: master-sindico
persona: sindico
secondary_persona: administradora
category: ASM
screen_id: ASM-EDITAL
sub_produto: assembleia-live
status: specification
stack: web
created: 2026-04-24
---

# ASM-EDITAL — Anexar ou Gerar Edital de Convocação

## Finalidade

Permite ao síndico (ou administradora autorizada) optar entre anexar um edital pronto (PDF) ou gerar o edital automaticamente via template da plataforma, importando dados de [[ASM-CFG]] e [[ASM-PAUTA]]. Gate jurídico antes da publicação ([[ASM-PUB]]).

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/03-Modulos/Assembleia-Live-Funcional]] §4.6 (Edital de convocação)
- [[../../../../../../90-archive/master-sindico-pdfs-cliente-originais/MÓDULO DAS TELAS ASSEMBLEIA MASTER SÍNDICO]] bloco edital
- [[../../../../04-requirements/functional/assembly]] Req 51
- Lei 4.591/64 (prazo mínimo convocação) + CC Art. 1.354

## Persona e ABAC

- **Primária**: `syndic` (`assembly.edital.write`).
- **Secundária**: `administradora` com permissão — §4.6 "Quem pode gerar: síndico OU administradora com permissão".

## Fluxo da tela

1. Seletor principal **[Anexar edital pronto] | [Gerar edital automático]**.
2. Se **Anexar**: uploader PDF (max 10MB); preview inline; botão remover.
3. Se **Gerar**: wizard mostra preview dos campos compilados:
   - Identificação do condomínio
   - Tipo da assembleia
   - Data + horário 1ª e 2ª convocação
   - Modalidade, local/link
   - Ordem do dia (importada de [[ASM-PAUTA]])
   - Regras de participação
   - Regras de procuração
   - Regras de votação
   - Logo da administradora (se vinculada)
   - Campo de assinatura
4. Botão **Gerar PDF** → backend renderiza via template (Typst/LaTeX/chromium headless).
5. Campo de revisão manual antes de aceitar.
6. CTA **Confirmar edital** → habilita [[ASM-PUB]].

## Componentes

- Toggle dual (anexar/gerar).
- Preview PDF embutido (pdfjs).
- Wizard step-by-step com checkboxes de revisão.

## Estados (loading/empty/error/success)

- **Loading**: barra de progresso ao gerar PDF (pode levar 5-15s).
- **Empty**: CTA grande "Escolha como formalizar a convocação".
- **Error**: falha na geração ⇒ fallback para anexo manual.
- **Success**: badge verde "Edital pronto".

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Upload anexo | `/api/v1/assemblies/:id/edital/upload` | POST | multipart PDF | `{edital_id, url}` |
| Gerar auto | `/api/v1/assemblies/:id/edital/generate` | POST | — | `{edital_id, url, pdf_sha256}` |
| Substituir | `/api/v1/assemblies/:id/edital` | PUT | novo PDF | 204 |

## Regras de negócio críticas

- **Prazo mínimo ≥ 8 dias corridos** entre publicação do edital e 1ª convocação (Lei 4.591/64 Art. 25 — convenção pode exigir mais). Front + back validam; se abaixo, bloqueia [[ASM-PUB]] com aviso jurídico.
- SHA-256 do PDF gerado é gravado para trilha de imutabilidade (ADR-0033 ata imutável se aplica à ata, mas edital também tem hash para integridade).
- Edital anexado vira artefato imutável após publicação (lock).
- Geração automática precisa dos campos de [[ASM-CFG]] completos + ao menos 1 item em [[ASM-PAUTA]].

## Ligações

- Origem: [[ASM-PAUTA]].
- Destino: [[ASM-PUB]].
- Relacionados: [[../../../../client-canvas/Arquitetura Assembleia]].

## Gaps/ressalvas

- Template de edital: versão inicial Jinja-like; estilização responsiva para impressão a validar.
- Assinatura digital ICP-Brasil do edital: **fora do escopo M1** — anexo manual ou geração sem assinatura digital oficial.

## Links

- [[_moc]]
- [[ASM-PUB]]
- [[ASM-PAUTA]]
