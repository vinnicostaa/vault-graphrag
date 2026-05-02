---
title: MÓDULO VIZINHANÇA - MORADOR (PDF do cliente — texto extraído)
type: note
tags: [archive, master-sindico, client-pdf, extracted-text]
source: Clients/MasterSindico/contents/MÓDULO VIZINHANÇA - MORADOR.pdf
created: 2026-04-24
---

# MÓDULO VIZINHANÇA - MORADOR

Texto extraído do PDF original do cliente João via `pdftotext -layout`. **Fidelidade limitada** — se PDF é majoritariamente imagem/diagrama visual, o texto extraído pode ser incompleto.

- **PDF original**: 3952436 bytes
- **Texto extraído**: 4093 caracteres
- **Origem**: `~/Documentos/Obsidian Vault/Clients/MasterSindico/contents/MÓDULO VIZINHANÇA - MORADOR.pdf`

## Texto extraído

```text
MÓDULO VIZINHANÇA

JORNADA DO MORADOR — NÍVEL MASTER SÍNDICO



1. PROPÓSITO DO MÓDULO

O módulo Vizinhança permite que moradores descubram:

   •   promoções do bairro

   •   parceiros indicados por síndicos

   •   estabelecimentos próximos

   •   benefícios exclusivos para seu condomínio

Mensagem institucional inicial:

"Seu condomínio faz parte de um bairro vivo.
Descubra estabelecimentos da região que valorizam quem mora na comunidade."



2. ENTRADA NA JORNADA

Botão no app

Vizinhança

Caminho

Painel do Morador
→ Vizinhança



TELA VM1

Explorar Vizinhança

Mensagem institucional

"Conheça estabelecimentos do seu bairro que oferecem benefícios para
moradores da comunidade."



Elementos exibidos

Feed de parceiros da vizinhança.
Cada card exibe:

Nome do estabelecimento
Categoria
Bairro
Promoção ativa (se houver)



FILTROS DISPONÍVEIS

Categoria



LISTA MESTRA DE CATEGORIAS

Alimentação
Padaria
Restaurante
Lanchonete
Cafeteria
Pizzaria
Hamburgueria
Sorveteria
Doceria
Açaí

Mercado
Supermercado
Hortifruti
Adega

Farmácia

Pet shop
Clínica veterinária
Banho e tosa

Academia
Crossfit
Pilates
Yoga

Estética
Salão de beleza
Barbearia
Spa

Lavanderia
Costureira
Sapateiro

Eletricista
Encanador
Chaveiro
Marceneiro
Pintor

Assistência técnica
Celular
Computadores

Oficina
Lava jato

Cursos
Idiomas
Reforço escolar

Outros serviços



OBSERVAÇÃO PARA DEVS

Feed prioriza:

   parceiros do mesmo bairro
   promoções ativas
   parceiros indicados por síndicos



TELA VM2

Detalhe do Estabelecimento

Caminho

VM1 → clicar no parceiro



Informações exibidas
Nome do estabelecimento

Categoria

Endereço

Telefone

Descrição

Promoções disponíveis



Seção

Promoções



Botões

Ver promoção

Entrar em contato

Ir até o local



TELA VM3

Detalhe da Promoção

Caminho

VM2 → Ver promoção



Informações exibidas

Título da promoção

Descrição

Tipo de promoção

Validade



Mensagem institucional

"Esta promoção faz parte da rede de vizinhança da Master Síndico."
Botão

Ativar promoção



TELA VM4

Ativar Promoção

Caminho

VM3 → Ativar promoção



Mensagem exibida

"Apresente esta tela no estabelecimento para ativar seu benefício."



Elementos exibidos

Nome do estabelecimento

Nome da promoção

Validade



Botão

Mostrar promoção



OBSERVAÇÃO PARA DEVS

A ativação gera métrica para o parceiro.



TELA VM5

Promoções Exclusivas do Condomínio

Caminho

Vizinhança
→ Benefícios do meu condomínio
Mensagem institucional

"Alguns parceiros oferecem benefícios exclusivos para moradores do seu
condomínio."



Lista exibida

Promoção

Parceiro

Validade



Observação para devs

Aparece apenas se houver promoções exclusivas.



TELA VM6

Parceiros Indicados pelo Síndico

Caminho

Vizinhança
→ Parceiros indicados



Mensagem institucional

"Estabelecimentos indicados pelo seu síndico para a comunidade."



Lista exibida

Parceiro

Categoria

Descrição



Badge exibida

Indicado pelo Síndico
TELA VM7

Histórico de Promoções Utilizadas

Caminho

Vizinhança
→ Histórico



Lista exibida

Promoção

Estabelecimento

Data de ativação



Objetivo

Controle de benefícios utilizados.



INTERAÇÃO COM OUTRAS JORNADAS



Conexão com o Parceiro

Morador ativa promoções.



Conexão com o Síndico

Síndico pode:

• indicar parceiros
• negociar promoções exclusivas



Conexão com Empresas

Empresas também podem acessar promoções da vizinhança.



REGRAS OPERACIONAIS
Morador pode:

✔ visualizar parceiros
✔ visualizar promoções
✔ ativar promoções
✔ ver parceiros indicados



Morador não pode:

✖ publicar promoções
✖ cadastrar parceiros



MÉTRICAS DO SISTEMA

Admin da plataforma acompanha:

• promoções mais ativadas
• parceiros mais acessados
• bairros com maior engajamento



RESULTADO DA JORNADA

O morador passa a enxergar o condomínio como parte de uma rede de bairro
conectada, criando valor para:

• parceiros locais
• síndicos
• comunidade

```

## Links

- [[_moc]]
