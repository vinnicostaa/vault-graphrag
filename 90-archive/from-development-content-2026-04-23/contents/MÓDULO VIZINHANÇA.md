---
title: MÓDULO VIZINHANÇA (ini do cliente)
type: note
tags: [archive, master-sindico, client-ini]
created: 2026-04-24
---

# MÓDULO VIZINHANÇA

Conteúdo original do arquivo `.ini` do cliente (arquivo de texto com especificações de módulo/tela).

## Conteúdo

```ini
MÓDULO VIZINHANÇA
JORNADA DA EMPRESA — NÍVEL MASTER SÍNDICO
1. PROPÓSITO DO MÓDULO PARA A EMPRESA
O módulo Vizinhança permite que empresas que atuam em condomínios também:
• descubram parceiros locais
• utilizem promoções da comunidade
• fortaleçam relacionamento com o bairro
• participem da economia local
Mensagem institucional inicial:
"Empresas que atuam em condomínios também fazem parte da vida do bairro.
Descubra parceiros locais que valorizam quem constrói a comunidade."
2. ENTRADA NA JORNADA
Botão no Painel Empresarial
Vizinhança
Caminho
Painel Empresarial
→ Vizinhança
TELA VE1
Painel da Vizinhança para Empresas
Mensagem institucional
"A comunidade também é formada por empresas que prestam serviços nos condomínios.
Explore parceiros locais e benefícios disponíveis na região."
Seções exibidas
Explorar parceiros do bairro
Promoções disponíveis
Histórico de promoções ativadas
Botões principais
Explorar parceiros
Ver promoções
TELA VE2
Explorar Parceiros da Vizinhança
Caminho
VE1 → Explorar parceiros
Feed exibido
Lista de parceiros cadastrados.
Cada card mostra:
Nome do estabelecimento
Categoria
Bairro
Promoção ativa
Filtros disponíveis
Categoria
Bairro
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
Botões disponíveis
Ver parceiro
TELA VE3
Detalhe do Parceiro
Caminho
VE2 → Ver parceiro
Informações exibidas
Nome do estabelecimento
Categoria
Endereço
Telefone
Descrição
Promoções disponíveis
Botões
Ver promoção
Entrar em contato
TELA VE4
Detalhe da Promoção
Caminho
VE3 → Ver promoção
Informações exibidas
Título da promoção
Descrição
Tipo de promoção
Validade
Mensagem institucional
"Esta promoção faz parte da rede de vizinhança da Master Síndico."
Botão
Ativar promoção
TELA VE5
Ativar Promoção
Caminho
VE4 → Ativar promoção
Mensagem exibida
"Apresente esta tela no estabelecimento para ativar seu benefício."
Elementos exibidos
Nome do estabelecimento
Nome da promoção
Validade
Botão
Mostrar promoção
Observação para devs
Ativação gera métrica para o parceiro.
TELA VE6
Histórico de Promoções Utilizadas
Caminho
VE1 → Histórico
Lista exibida
Promoção
Estabelecimento
Data de ativação
Objetivo
Controle de promoções utilizadas pela empresa.
REGRAS OPERACIONAIS
Empresa pode
✔ explorar parceiros da vizinhança
✔ visualizar promoções
✔ ativar promoções
✔ entrar em contato com parceiros
Empresa não pode
✖ criar promoções
✖ editar promoções
✖ cadastrar parceiros
VISIBILIDADE
Promoções abertas aparecem para
• moradores
• síndicos
• empresas
Promoções exclusivas aparecem apenas para
• moradores do condomínio
Empresas não visualizam promoções exclusivas.
MÉTRICAS DO SISTEMA
Para parceiros da vizinhança
• número de ativações por empresas
• número de ativações por moradores
• número de ativações por síndicos
RESULTADO DA JORNADA
A empresa passa a participar da economia do bairro, fortalecendo relações com parceiros locais e integrando-se à comunidade condominial.
Agora o Módulo Vizinhança está completo, com quatro jornadas integradas:
1️⃣ Parceiro da vizinhança
2️⃣ Morador
3️⃣ Síndico
4️⃣ Empresa```
