---
title: "VZ1 — Cadastro do Parceiro da Vizinhança"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, parceiro, cadastro]
project: master-sindico
persona: parceiro
category: VZ
screen_id: VZ1
sub_produto: vizinhanca
plan_requirement: trial
status: specification
stack: web
created: 2026-04-23
---

# VZ1 — Cadastro do Parceiro da Vizinhança

## Finalidade
Cadastro inicial do estabelecimento que se torna Parceiro da Vizinhança. Decision 12: persona independente com login próprio (role `local_company`); aceita CPF (CNPJ não obrigatório).

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VZ1
- `vault/90-archive/master-sindico-pdfs-cliente-originais/Jornada do Parceiro da Vizinhança.md` §TELA VZ1

## Mensagem institucional canônica
> "Cadastre seu estabelecimento e ofereça benefícios para quem vive e trabalha no bairro."

## Persona e ABAC
- **Persona**: Parceiro (role `local_company`) — após criar a conta
- **Plan tier**: `trial` (cadastro)
- **Caminho canônico**: App → Vizinhança → "Quero ser parceiro da vizinhança"
- **Entrada alternativa**: link de convite enviado por síndico em [[VS6]] (com pré-preenchimento)

## Campos obrigatórios (literais — PDF)
- Nome do estabelecimento
- Categoria principal
- Subcategoria
- Telefone
- WhatsApp
- Email
- CEP
- Endereço
- Número
- Bairro
- Cidade
- Descrição do estabelecimento

## Campos opcionais
- Instagram
- Logo

## Componentes
- `MultiStepForm` (sugerido — UX) ou `SingleForm`
- `CategoryAutocomplete` (lista mestra 38+)
- `CEPLookup` (preenche bairro/cidade automaticamente)
- `LogoUpload` (signed URL)
- `PrimaryButton` ("Cadastrar estabelecimento")

## Estados
- **Inicial**: form vazio (ou pré-preenchido se vindo de convite).
- **Validating**: inline.
- **Submitting**: spinner.
- **Success**: redireciona para [[VZ2]] (perfil) com toast "Bem-vindo à rede de vizinhança!".
- **Error**: toast retry; preserva campos.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Lookup CEP | `/api/v1/utils/cep/{cep}` | GET |
| Cadastrar parceiro | `/api/v1/neighborhood/partners` | POST |
| Upload logo | `/api/v1/storage/sign?type=logo` | POST |

## Regras de negócio críticas
- **Não exige CNPJ** (apenas CPF do dono é aceito — Decision 12).
- Cria entidade `companies` com `is_neighborhood=true`.
- Provisiona role `local_company` no Zitadel.
- **Restrição cruzada Decision 7**: Parceiro NÃO acessa LMS Academia (botão oculto).
- **NÃO acessa Banco de Talentos** (matriz funcional).
- **NÃO publica comunicados em condomínios**, NÃO registra serviços, NÃO acessa painel empresarial.
- Se vindo de convite [[VS6]], pré-preenche campos enviados pelo síndico + atualiza status `convite_enviado → parceiro_cadastrado` em [[VS9]].

## Ligações
- [[VZ2]] · [[VS6]] (entrada alternativa) · [[VS9]] (status muda) · [[_moc]]
- LMS bloqueado: [[../lms/_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Sem ADR sobre validação de CPF do dono (verificar Receita Federal? só formato?).
- Lista mestra 38+ categorias sem catálogo final unificado.
- Provedor CEP (ViaCEP, BrasilAPI?) sem ADR.
- Tamanho máximo logo sem definição.
