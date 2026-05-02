---
title: MK4 — Publicar Vídeo
type: ui-screen
tags: [master-sindico, ui-catalog, web, marketing, marketing-agencias, conteudo-videos, assembleia]
project: master-sindico
persona: marketing
category: MK
screen_id: MK4
sub_produto: marketing-agencias
plan_requirement: any
status: specification
stack: web
created: 2026-04-23
---

# MK4 — Publicar Vídeo

## Finalidade
Agência publica vídeo institucional em nome da empresa (no contexto carregado em [[MK3]]). 12 tipos de vídeo definidos. Checkbox crítico "Autorizar exibição em processos de escolha de fornecedor" controla se o vídeo aparece em votações de assembleia (módulo `assembly`).

## Fonte canônica
- `vault/90-archive/master-sindico-pdfs-cliente-originais/JORNADA DA EMPRESA DE MARKETING.md` (TELA MK4 — Publicar Vídeo + 12 tipos + checkbox + regras moradores)

## Persona e ABAC
- **Persona primária**: Marketing
- **Plan tier mínimo**: any (no contexto de empresa Plus/Pro)
- **ABAC action**: `content.video.create` (scoped por `agency_id` + `empresa_id`)
- **Restrições**: agência precisa ter contexto de empresa ativo via [[MK3]].

## Fluxo da tela
1. Agência em [[MK3]] → clica card "Publicar vídeo".
2. Sistema carrega formulário com empresa pré-selecionada (do contexto).
3. Agência preenche: título, tipo de vídeo, subtema (auto), descrição, link/upload do vídeo.
4. Agência marca/desmarca checkbox "Autorizar exibição em processos de escolha de fornecedor".
5. Agência clica [Salvar rascunho] (estado=draft) ou [Publicar vídeo] (estado=published).
6. Sistema persiste vídeo + metadados; integração com Mux para encoding/streaming.
7. Vídeo aparece na biblioteca [[MK5]] da empresa.

## Componentes
- `VideoUploadForm`
- `EmpresaContextBadge` (read-only — empresa do contexto)
- `VideoTypeSelect` (12 opções)
- `SubtemaAutoFill` (preenche com base no segmento da empresa — fonte F8)
- `VideoUrlInput` ou `VideoFileUpload` (Mux)
- `CheckboxAuthorization` (autorizar exibição em assembleia)
- `ButtonGroup` (rascunho / publicar)

## Campos
- Empresa vinculada (read-only, do contexto)
- Título do vídeo
- Tipo de vídeo (1 das 12 — abaixo)
- Subtema do vídeo (preenchido automaticamente com base no segmento — fonte F8)
- Descrição do vídeo
- Link do vídeo (Mux URL ou upload direto)
- Checkbox: "Autorizar exibição em processos de escolha de fornecedor"

## 12 tipos de vídeo (fonte PDF MK4)
1. Apresentação institucional da empresa
2. Apresentação da equipe técnica
3. Demonstração de serviço
4. Explicação técnica
5. Boas práticas condominiais
6. Orientação preventiva
7. Caso real de atendimento
8. Treinamento técnico
9. Explicação de legislação ou normas
10. Dicas de manutenção para condomínios
11. Conteúdo educativo para síndicos
12. Conteúdo educativo para moradores

## Estados
- **Loading**: skeleton form
- **Draft saved**: badge "Rascunho salvo" + autosave 30s
- **Encoding**: spinner "Processando vídeo no Mux"
- **Published**: redireciona para [[MK5]]
- **Error**: toast retry; rascunho preservado

## Integração com backend
| Ação UI | Endpoint | Método | Payload |
|---|---|---|---|
| Criar vídeo | `/api/v1/content/videos` | POST | { empresa_id, type, title, description, mux_url, allow_in_assembly } |
| Salvar rascunho | `/api/v1/content/videos/draft` | PUT | partial |
| Upload Mux | Mux API direto | POST | multipart/streaming |

Use case `content.PublishVideo` + integração Mux. ABAC valida `agency.video.publish_for_empresa(empresa_id)`.

## Regras de negócio críticas
- **Visibilidade restrita aos moradores**: moradores NÃO acessam vídeos institucionais normalmente — só durante votação de fornecedor em assembleia (D-064)
- **Após votação encerrada**: acesso bloqueado novamente (R5 — pós-fechamento imutável + visibilidade)
- **Checkbox autorização**: se desmarcado, vídeo fica restrito ao perfil da empresa ([[../empresa/E12]]) — nunca aparece em votação
- **Lock 90 dias** em vídeos publicados (D-066 — confirmar)
- **Atribuição**: vídeo é sempre atribuído à empresa, não à agência (mesmo que agência publique)
- **Auditoria**: registro inclui agency_user_id + empresa_id + timestamp

## Ligações
- **Tela origem**: [[MK3]]
- **Tela destino**: [[MK5]] (biblioteca, após publicar), [[MK3]] (cancelar)
- **Sub-produto**: [[../../../../00-product/sub-produtos/04-conteudo-videos]] + [[../../../../00-product/sub-produtos/10-marketing-agencias]]
- **Backend**: [[../../requirements]]
- **Inventário**: [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Inventario-Telas]]

## Gaps/ressalvas
- Subtema "preenchido automaticamente com base no segmento da empresa": lista derivada das 31 categorias canônicas (F8) — falta mapear para 12 tipos
- Tamanho máximo de vídeo / duração: não especificado — sugerir 10min / 500MB
- "Link do vídeo" (URL externa) vs upload direto: PDF aceita ambos — definir política

## Links
- [[../../../../_moc]]
- [[../../../../CLAUDE]]
- [[../../../../client-canvas/Visibilidade Videos]]
