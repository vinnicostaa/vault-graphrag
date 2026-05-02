---
title: JORNADA DO SÍNDICO completa (PDF do cliente — texto extraído)
type: note
tags: [archive, master-sindico, client-pdf, extracted-text]
source: Clients/MasterSindico/contents/JORNADA DO SÍNDICO completa.pdf
created: 2026-04-24
---

# JORNADA DO SÍNDICO completa

Texto extraído do PDF original do cliente João via `pdftotext -layout`. **Fidelidade limitada** — se PDF é majoritariamente imagem/diagrama visual, o texto extraído pode ser incompleto.

- **PDF original**: 244619 bytes
- **Texto extraído**: 8831 caracteres
- **Origem**: `~/Documentos/Obsidian Vault/Clients/MasterSindico/contents/JORNADA DO SÍNDICO completa.pdf`

## Texto extraído

```text
JORNADA DO SÍNDICO — GOVERNANÇA MASTER SÍNDICO

Botão no aplicativo

Governança

Descrição no app
Espaço de gestão institucional do condomínio, com planejamento, registro de
atividades, validações, comunicação com moradores e integração com empresas.



REGRAS ESTRUTURAIS DA JORNADA DO SÍNDICO

Regra 1 — A Linha do Tempo é a memória oficial do condomínio

Tudo que for considerado relevante para a gestão precisa gerar registro na Linha
do Tempo.

Tipos de publicação:

   •   Atividade da gestão

   •   Registro de execução validado

   •   Comunicado publicado

   •   Evento publicado

   •   Adendo

   •   Resultado consolidado de pergunta aos moradores

Regra 2 — O Plano Diretor não é atualizado manualmente

O status de cada ação do Plano Diretor só pode mudar quando houver publicação
vinculada na Linha do Tempo.

Regra 3 — Empresa não publica direto para moradores

Empresa pode enviar:

   •   registro de execução

   •   atividade técnica

   •   comunicado técnico

Mas tudo passa por validação do síndico ou da empresa síndica vinculada,
quando essa permissão estiver ativa.

Regra 4 — Edição com preservação de histórico

Toda edição relevante deve manter versão anterior e gerar trilha de auditoria.
Regra 5 — Síndico pode consultar até 12 condomínios

Para manter coerência com suas decisões anteriores, vou fixar a base em 12
condomínios.

Regra 6 — Empresa síndica vinculada

Pode:

   •    visualizar publicações

   •    editar publicações do síndico

   •    ocultar publicações

   •    validar registros de empresas

   •    validar comunicados de empresas



TELA S1 — ENTRADA DA GOVERNANÇA

Caminho
App → Governança → S1

Objetivo
Apresentar a área de governança e formalizar o uso responsável da plataforma.

Mensagem institucional
Bem-vindo à Governança Master Síndico.
Aqui a gestão do condomínio deixa de ser apenas rotina administrativa e passa a
ser memória institucional, com transparência, rastreabilidade e organização.

Rodapé institucional
Ao acessar esta área, você declara que utilizará a plataforma de forma
responsável, registrando informações verdadeiras e relacionadas à administração
do condomínio.

Botão

   •    Entrar na Governança

Observações para devs

   •    Exibir apenas no primeiro acesso



TELA S2 — MEUS CONDOMÍNIOS
Caminho
S1 → S2

Objetivo
Permitir seleção de condomínio e acesso ao histórico de governança de cada um.

Mensagem institucional
Cada condomínio possui sua própria estrutura de gestão, histórico institucional e
comunidade. Selecione o condomínio que deseja administrar.

Card do condomínio

   •   Nome do condomínio

   •   Cidade / Estado

   •   Quantidade de unidades

   •   Status do mandato

   •   Período do mandato

   •   Avaliação institucional média

   •   Empresa síndica vinculada, se houver

Botões

   •   Entrar

   •   Cadastrar novo condomínio

Observações para devs

   •   Limite de até 15 condomínios por síndico

   •   Ordenar por ativos primeiro

   •   Se condomínio estiver encerrado, mostrar selo “Histórico / leitura”



TELA S3 — CADASTRAR CONDOMÍNIO

Caminho
S2 → Cadastrar novo condomínio → S3

Objetivo
Criar a identidade institucional do condomínio dentro da plataforma.

Mensagem institucional
Um cadastro bem estruturado evita erros, facilita o acesso dos moradores e
fortalece a organização da gestão desde o início.
Campos

   •   Nome do condomínio

   •   CEP

   •   Endereço

   •   Número

   •   Complemento

   •   Bairro

   •   Cidade

   •   Estado

   •   Tipo de condomínio

   •   Quantidade total de unidades

   •   Possui blocos/torres? Sim/Não

   •   Cadastro de blocos/torres

   •   Foto da fachada (opcional)

Lista mestre — Tipo de condomínio

   •   Residencial

   •   Comercial

   •   Misto

   •   Shopping center

   •   Galeria comercial

   •   Edifício garagem

Se possuir blocos/torres
Campos:

   •   Nome do bloco/torre

   •   Quantidade de unidades por bloco

Botões

   •   Continuar

   •   Cancelar
Observações para devs

   •   Gerar automaticamente:

          o     ID único do condomínio

          o     QR Code

          o     link do app

   •   Botão “Copiar ID com mensagem”

   •   Essa estrutura alimenta o cadastro do morador



TELA S4 — DADOS DA GESTÃO

Caminho
S3 → S4

Objetivo
Registrar o tipo de mandato e o período de responsabilidade da gestão.

Mensagem institucional
Toda gestão precisa de um marco temporal claro. Este vínculo organiza a
responsabilidade administrativa do mandato.

Campos

   •   Tipo de mandato

   •   Data de início

   •   Data de término

   •   Vincular empresa síndica? Sim/Não

Lista mestre — Tipo de mandato

   •   Síndico eleito

   •   Síndico profissional

   •   Síndico interino

Botões

   •   Confirmar

   •   Voltar

Observações para devs
   •   Se “Vincular empresa síndica = Sim”, seguir para S5

   •   Se não, concluir cadastro e ir para Home da Governança



TELA S5 — CADASTRO / VÍNCULO DA EMPRESA SÍNDICA

Caminho
S4 → Vincular empresa síndica → S5

Objetivo
Cadastrar ou vincular empresa síndica com poderes operacionais na governança.

Mensagem institucional
A empresa síndica vinculada amplia a continuidade da gestão e a rastreabilidade
das ações, preservando o histórico institucional do condomínio.

Campos

   •   Razão social

   •   Nome fantasia

   •   CNPJ

   •   Responsável legal

   •   Telefone

   •   E-mail corporativo

   •   Site

   •   Cidade / Estado

   •   Logo da empresa

Validação

   •   Envio de token por e-mail

Permissões

   •   Visualizar publicações

   •   Editar publicações do síndico

   •   Ocultar publicações

   •   Validar registros de execução enviados por empresas

   •   Validar comunicados técnicos enviados por empresas
Botões

   •    Enviar token

   •    Validar vínculo

   •    Voltar

Observações para devs

   •    Permissões parametrizáveis

   •    Toda ação da empresa síndica também entra na auditoria



TELA S6 — HOME DA GOVERNANÇA

Caminho
S2 → Entrar → S6

Objetivo
Centralizar todos os módulos da gestão.

Mensagem institucional
Este é o centro de comando da gestão do condomínio. Aqui você planeja, registra,
valida, comunica e acompanha a evolução da administração.

Cards

   •    Plano Diretor

   •    Linha do Tempo

   •    Eventos

   •    Connect Me

   •    Registros de execução

   •    Comunicados

   •    Validações Pendentes

   •    Dashboard

   •    Compliance

Observações para devs

   •    Mostrar badges de pendência:

           o     validações pendentes
          o   avaliações bimestrais recebidas

          o   ações atrasadas do PD

          o   eventos próximos



MÓDULO PLANO DIRETOR

TELA S7 — PLANO DIRETOR

Caminho
S6 → Plano Diretor → S7

Objetivo
Visualizar e administrar o planejamento estruturado da gestão.

Mensagem institucional
O Plano Diretor organiza as ações estruturais e estratégicas do condomínio. Seu
avanço é registrado conforme a gestão publica atividades vinculadas na Linha do
Tempo.

Botões

   •   Cadastrar nova ação

   •   Ver ações cadastradas



TELA S8 — CADASTRAR AÇÃO DO PLANO DIRETOR

Caminho
S7 → Cadastrar nova ação → S8

Objetivo
Criar uma ação estratégica do plano de gestão.

Campos

   •   Nome da ação

   •   Descrição

   •   Área impactada

   •   Natureza

   •   Nível de importância

   •   Impacto esperado
   •   Prazo limite

Lista mestre — Área impactada

   •   Estrutura predial

   •   Sistema hidráulico

   •   Sistema elétrico

   •   Sistema de incêndio

   •   Reservatórios

   •   Elevadores

   •   Garagem

   •   Fachada

   •   Cobertura

   •   Segurança patrimonial

   •   Administração

   •   Portaria

   •   Playground

   •   Academia

   •   Salão de festas

   •   Corredores

   •   Jardins

   •   Casa de máquinas

   •   Rede de esgoto

   •   Rede de drenagem

   •   Sistema de gás

   •   Iluminação

   •   Sistema de câmeras

   •   Controle de acesso

   •   Áreas comuns

   •   Outros
Lista mestre — Natureza

   •   Preventiva

   •   Corretiva

   •   Adequação normativa

   •   Melhoria estrutural

   •   Monitoramento

   •   Modernização

   •   Investigação técnica

   •   Regularização legal

Lista mestre — Nível de importância

   •   Baixa

   •   Moderada
```

## Links

- [[_moc]]
