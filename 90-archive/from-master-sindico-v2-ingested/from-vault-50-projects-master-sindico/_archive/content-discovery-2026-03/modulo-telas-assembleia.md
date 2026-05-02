---
title: "MÓDULO DAS TELAS ASSEMBLEIA MASTER SÍNDICO"
type: source
tags: [pdf, converted]
source: 50-projects/master-sindico/content/contents/MÓDULO DAS TELAS ASSEMBLEIA MASTER SÍNDICO.pdf
converted: 2026-04-22
---

# MÓDULO DAS TELAS ASSEMBLEIA MASTER SÍNDICO

> PDF convertido via pdftotext. Layout original perdido; texto preservado.

MÓDULO ASSEMBLEIA MASTER SÍNDICO

MAPA DE TELAS DETALHADO (MVP PRESENCIAL)



PERFIS DO SISTEMA

Perfis com acesso ao módulo:

Síndico

Administradora

Morador

Telão público da assembleia



ESTRUTURA DE NAVEGAÇÃO

Menu principal

→ Condomínio

→ Módulo Assembleia



TELA 1 — DASHBOARD DO MÓDULO ASSEMBLEIA

Perfil

Síndico



Caminho

Menu principal

→ Condomínio

→ Assembleia

→ Dashboard



Objetivo da tela

Centralizar todas as assembleias do condomínio.
Campos exibidos

Campo              Tipo   Descrição

nome_assembleia texto     identificação

data               data   data da assembleia

status             enum   situação

% ciência          número moradores que registraram ciência

% confirmação      número moradores que confirmaram presença

quórum estimado número baseado nas confirmações



Status possíveis

EM_PREPARACAO

PUBLICADA

EM_ANDAMENTO

ENCERRADA



Ações

Botões:

Criar nova assembleia

Abrir assembleia

Ver relatório



Mensagem institucional

A assembleia é o momento em que moradores e gestão constroem juntos o futuro
do condomínio.
Decisões bem organizadas fortalecem a transparência, a confiança e a
convivência entre todos.
Orientação para dev

   •    tela inicial do módulo

   •    deve carregar rapidamente

   •    dados resumidos

   •    acesso às assembleias existentes



TELA 2 — CRIAR ASSEMBLEIA

Caminho

Dashboard

→ Criar assembleia



Objetivo

Registrar os dados principais da assembleia.



Campos

Campo                        Tipo

tipo_assembleia              select

data_assembleia              date

hora_primeira_convocacao time

hora_segunda_convocacao time

modalidade                   select

local                        texto

tempo_fala_padrao            select

descricao                    texto



Modalidade

PRESENCIAL (ativo)
HIBRIDA (desabilitado)

ONLINE (desabilitado)

Campos preparados para futuro:

link_transmissao

participantes_online

controle_audio_remoto



Tempo de fala

2 minutos

3 minutos

Com prorrogação de até mais 2 min.



Regras

   •   data deve ser futura

   •   segunda convocação deve ser posterior à primeira

   •   tempo de fala máximo 5 min



Mensagem institucional

Uma assembleia bem organizada começa com informações claras.
Preencha os dados com atenção para garantir uma convocação transparente e
respeitosa com todos os moradores.



Botões

Salvar
Salvar e continuar
Cancelar



TELA 3 — HABILITAR ADMINISTRADORA

Caminho
Criar assembleia

→ Vincular administradora



Campos

Campo       Tipo

empresa     texto

CNPJ        texto

responsável texto

CPF         texto

email       email

telefone    telefone



Sistema gera

token_acesso_administradora



Mensagem institucional

A administradora atua como parceira na organização e validação da assembleia,
contribuindo para a segurança e legitimidade das decisões.



Checkbox obrigatório

☑ Confirmo que esta empresa foi contratada para administrar este condomínio.



TELA 4 — CADASTRO COMPLETO DA PAUTA

Caminho

Criar assembleia

→ Configurar pauta
Objetivo

Definir todos os temas que serão discutidos e deliberados.



Campos do item

Campo               Tipo

ordem               número

titulo_item         texto

descricao_item      texto longo

tipo_item           select

impacto_financeiro boolean

valor_estimado      moeda

quorum_exigido      percentual

tipo_votacao        select



Tipos de item

INFORMATIVO

VOTACAO

ESCOLHA_FORNECEDOR

ELEICAO



Tipo de votação

SIM / NÃO / ABSTENÇÃO

ESCALA 1–5 + ABSTENÇÃO



Campos adicionais

Anexar documento
Anexar vídeo explicativo
Fornecedor participante (se houver)



Regras importantes

Após publicar assembleia

LOCK_Pauta = TRUE

Não pode:

   •    alterar texto

   •    alterar quórum

   •    alterar tipo de votação

   •    remover item



Mensagem institucional

A pauta é o guia das decisões da assembleia.
Informações claras ajudam os moradores a compreender os temas e votar com
responsabilidade.



TELA 5 — ANEXAR EDITAL

Caminho

Configurar assembleia

→ Anexar edital



Campo

Upload de arquivo

edital_convocacao.pdf



Regra

Obrigatório antes da publicação.



Mensagem institucional
O edital é o documento que convoca oficialmente os moradores para a
assembleia.
Ele garante que todos tenham acesso às informações necessárias para participar
das decisões do condomínio.



TELA 6 — MONITORAMENTO DA PRÉ-ASSEMBLEIA

Caminho

Assembleia publicada

→ Monitoramento



Painel mostra

Ciência registrada
Moradores que não abriram
Confirmação de presença
Procurações registradas
Procurações aprovadas
Quórum estimado



Mensagem institucional

Quanto maior a participação dos moradores, mais representativas serão as
decisões tomadas na assembleia.



TELA 7 — CADASTRO DE INADIMPLÊNCIA

Caminho

Painel assembleia

→ Inadimplência



Opções

Upload de planilha

ou
seleção manual (ao lado de cada unidade, a administradora deverá bloquer ao
voto os moradores inadimplentes).



Status

APTO A VOTAR

VOTO BLOQUEADO



Botão especial

Liberar voto

Campos registrados:

   •     responsável

   •     horário

   •     justificativa



Mensagem institucional

O controle de inadimplência garante que o direito de voto seja exercido conforme
as regras do condomínio.



TELA 8 — VALIDAÇÃO DE PROCURAÇÕES

Caminho

Painel assembleia

→ Procurações



Campos

Campo                            Tipo

Nome completo procurador         texto

CPF do procurador                texto

unidade_procurador               texto
Campo                          Tipo

Nome completo do representado texto

CPF do representado            texto

unidade_representada           texto



Ações

Aprovar
Rejeitar



Regra

Se aprovado:

voto_representado = voto_procurador

Considerando:

peso_fracao_ideal



TELA 9 — PAINEL DA ASSEMBLEIA

Caminho

Assembleia publicada

→ Iniciar assembleia



Campos obrigatórios

Presidente da mesa

nome
CPF

Secretários (até 3)

nome
CPF

Administradora
empresa
nome
CPF

Auditor (opcional)

empresa
nome
CPF

Fornecedores presentes

empresa
nome do representante
CPF


Mensagem institucional

A identificação dos responsáveis pela condução da assembleia garante
transparência e legitimidade ao processo deliberativo.



TELA 10 — INICIAR ASSEMBLEIA

Sistema exibe vídeo institucional.



Mensagem

A assembleia é o espaço de diálogo e decisão coletiva do condomínio.
Respeito, escuta e responsabilidade ajudam a construir soluções que beneficiam
toda a comunidade.



TELAS DO MORADOR



TELA — REGISTRO DE CIÊNCIA

Checkboxes obrigatórios

☑ Li o edital de convocação
☑ Autorizo o tratamento dos meus dados conforme LGPD
☑ Declaro participar na condição vinculada à minha unidade
☑ Estou ciente de que a assembleia poderá ser gravada
Mensagem institucional

Ao registrar sua ciência, você confirma que tomou conhecimento da assembleia e
poderá acompanhar decisões que impactam diretamente o seu condomínio.



TELA — VISUALIZAÇÃO DA PAUTA

Mostra:

   •     pauta

   •     documentos

   •     vídeos

   •     fornecedores



Mensagem institucional

Conhecer os temas da assembleia ajuda a tomar decisões mais conscientes e
responsáveis para o condomínio.



TELA — CHECK-IN

Opções

QR code
Check-in pelo app



Mensagem institucional

Seu check-in registra oficialmente sua presença e garante transparência na
participação da assembleia.



TELÃO DA ASSEMBLEIA



Área 1

apresentação
Área 2

   •     pauta completa

   •     item atual destacado

   •     unidades presentes

   •     unidades ausentes

   •     fila de fala

   •     cronômetro

   •     relógio assembleia

   •     quórum presente

   •     quórum necessário



Durante votação

Mostra:

| Unidade | Voto |



RESULTADO

Agora o módulo está:

✔ com caminhos claros
✔ com regras
✔ com orientações para dev
✔ com mensagens institucionais fortes
✔ com termos e checkboxes
✔ com tela de pauta completa
✔ com registro manual de inadimplência
