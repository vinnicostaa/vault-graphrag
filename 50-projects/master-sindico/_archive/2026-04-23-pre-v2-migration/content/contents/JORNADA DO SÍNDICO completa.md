---
title: "JORNADA DO SÍNDICO completa"
type: source
tags: [pdf, converted]
source: 50-projects/master-sindico/content/contents/JORNADA DO SÍNDICO completa.pdf
converted: 2026-04-22
---

# JORNADA DO SÍNDICO completa

> PDF convertido via pdftotext. Layout original perdido; texto preservado.

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

   •   Alta

   •   Crítica

   •   Emergencial

Lista mestre — Impacto esperado

   •   Redução de risco

   •   Adequação legal

   •   Melhoria da infraestrutura

   •   Preservação patrimonial

   •   Aumento da segurança

   •   Melhoria operacional

   •   Economia financeira

   •   Melhoria da qualidade de vida

Status inicial automático

   •   Planejada

Botões

   •   Salvar ação
   •   Cancelar

Observações para devs

   •   Não permitir edição manual do status

   •   Aplicar cores por status



TELA S9 — LISTA DE AÇÕES DO PLANO DIRETOR

Caminho
S7 → Ver ações cadastradas → S9

Objetivo
Listar todas as ações do Plano Diretor com situação atual.

Card da ação

   •   Nome

   •   Área impactada

   •   Prazo

   •   Status

   •   Última atualização

Status possíveis

   •   Planejada

   •   Em contratação

   •   Em execução

   •   Concluída

   •   Suspensa

   •   Reprogramada

   •   Atrasada

Botões

   •   Ver ação

   •   Editar ação

   •   Ocultar ação

Observações para devs
   •    “Editar ação” não altera status

   •    Status vem da Linha do Tempo

   •    “Atrasada” pode ser calculado automaticamente



TELA S10 — DETALHE DA AÇÃO DO PLANO DIRETOR

Caminho
S9 → Ver ação → S10

Objetivo
Exibir os detalhes da ação e o histórico relacionado.

Exibe

   •    Nome

   •    Descrição

   •    Área impactada

   •    Natureza

   •    Nível de importância

   •    Impacto esperado

   •    Prazo

   •    Status

Seção: Registros vinculados

   •    Lista de publicações da Linha do Tempo ligadas à ação

Botões

   •    Criar atividade vinculada

   •    Editar ação

   •    Ocultar ação

Observações para devs

   •    Relação por master_plan_action_id

   •    Mostrar linha evolutiva da ação



MÓDULO LINHA DO TEMPO
TELA S11 — LINHA DO TEMPO

Caminho
S6 → Linha do Tempo → S11

Objetivo
Centralizar todos os registros institucionais do condomínio.

Mensagem institucional
A Linha do Tempo é a memória institucional do condomínio. Ela registra as ações
da gestão, as execuções de empresas, os comunicados e os eventos relevantes.

Botões

   •   Criar atividade

   •   Ver histórico

   •   Filtrar publicações

Tipos de publicação

   •   Atividade da gestão

   •   Registro de execução

   •   Comunicado

   •   Evento

   •   Adendo

   •   Resultado de consulta aos moradores

Observações para devs

   •   Tudo que for público aqui pode aparecer ao morador

   •   Filtros por período, tipo, empresa, área impactada, status, vínculo ao PD



TELA S12 — CRIAR ATIVIDADE

Caminho
S11 → Criar atividade → S12

Objetivo
Publicar uma atividade administrativa, técnica ou operacional da gestão.

Mensagem institucional
Registrar uma atividade é transformar uma ação da gestão em histórico
institucional, fortalecendo a transparência e a previsibilidade para moradores e
empresas.

Campos

   •   Data da atividade

   •   Tipo de atividade

   •   Descrição detalhada

   •   Área impactada

   •   Natureza

   •   Nível de importância

   •   Risco associado

   •   Impacto esperado

   •   Próxima ação

   •   Empresa envolvida (opcional)

   •   Período da atividade:

          o   Data início

          o   Data término

   •   Garantia (quando aplicável)

   •   Orientação técnica / administrativa

   •   Anexos

   •   Vincular ao Plano Diretor? Sim/Não

   •   Se Sim: selecionar ação do Plano Diretor

   •   Adicionar pergunta aos moradores? Sim/Não

Lista mestre — Tipos de atividade

   •   Manutenção preventiva

   •   Manutenção corretiva

   •   Inspeção técnica

   •   Vistoria técnica

   •   Contratação de serviço
   •   Execução de serviço

   •   Reparo emergencial

   •   Obra de melhoria

   •   Adequação normativa

   •   Atualização de infraestrutura

   •   Monitoramento ambiental

   •   Revisão técnica

   •   Auditoria técnica

   •   Limpeza técnica

   •   Controle de pragas

   •   Limpeza de reservatórios

   •   Treinamento técnico

   •   Adequação legal

   •   Revisão de equipamentos

   •   Atualização administrativa

   •   Contratação de fornecedor

   •   Encerramento de contrato

   •   Avaliação de fornecedor

   •   Implantação de sistema

   •   Atualização de normas

   •   Ação emergencial

Lista mestre — Risco associado

   •   Sem risco identificado

   •   Risco operacional

   •   Risco estrutural

   •   Risco sanitário

   •   Risco elétrico

   •   Risco hidráulico
   •   Risco de incêndio

   •   Risco jurídico

   •   Risco ambiental

Lista mestre — Próxima ação

   •   Monitorar evolução

   •   Nova inspeção

   •   Contratar fornecedor

   •   Executar manutenção

   •   Atualizar plano diretor

   •   Registrar conclusão

   •   Acompanhar garantia

   •   Reavaliar necessidade

Bloco — Pergunta aos moradores
Se ativado:

   •   Pergunta

   •   Tipo de resposta:

          o   Sim / Não / Não sei

          o   Escala 1 a 5

          o   Campo aberto

          o   Múltipla escolha

   •   Prazo da pergunta

   •   Público:

          o   Todos os moradores

          o   Apenas titulares

Botões

   •   Salvar rascunho

   •   Publicar atividade

   •   Cancelar
Observações para devs

   •      Se vinculada ao PD e publicada, atualizar automaticamente o status da
          ação

   •      Pergunta não pode ser editada após publicação

   •      Atividade editada deve manter histórico anterior e criar nova versão com
          justificativa



TELA S13 — EDITAR ATIVIDADE

Caminho
S11 → Ver publicação → Editar → S13

Objetivo
Ajustar atividade já publicada com controle de versão.

Campos

   •      Todos os campos da atividade

   •      Campo obrigatório: justificativa da edição

Botões

   •      Salvar nova versão

   •      Cancelar

Observações para devs

   •      Não sobrescrever publicação anterior

   •      Criar novo registro de versão

   •      Gerar auditoria



TELA S14 — HISTÓRICO DA LINHA DO TEMPO

Caminho
S11 → Ver histórico → S14

Objetivo
Listar todas as publicações com filtros avançados.

Filtros

   •      Período
   •   Tipo de publicação

   •   Empresa

   •   Área impactada

   •   Status

   •   Vínculo ao Plano Diretor

Card da publicação

   •   Tipo

   •   Data

   •   Título

   •   Empresa, se houver

   •   Vínculo ao Plano Diretor, se houver

   •   Status

   •   Botão: Ver detalhes



MÓDULO EVENTOS

TELA S15 — EVENTOS

Caminho
S6 → Eventos → S15

Objetivo
Gerenciar eventos do condomínio.

Mensagem institucional
Eventos fortalecem organização, previsibilidade e participação da comunidade
condominial.

Botões

   •   Criar evento

   •   Ver eventos



TELA S16 — CRIAR EVENTO

Caminho
S15 → Criar evento → S16
Campos

   •   Tipo de evento

   •   Título

   •   Descrição

   •   Data

   •   Hora

   •   Local

   •   Anexos

   •   Exigir confirmação de presença? Sim/Não

   •   Exigir “Ciente”? Sim/Não

Lista mestre — Tipos de evento

   •   Assembleia ordinária

   •   Assembleia extraordinária

   •   Manutenção programada

   •   Inspeção técnica

   •   Obra programada

   •   Interrupção programada de serviços

   •   Treinamento de equipe

   •   Campanha institucional

   •   Evento comunitário

   •   Reunião administrativa

   •   Atualização normativa

   •   Simulado de emergência

   •   Outros

Botões

   •   Publicar evento

   •   Salvar rascunho

   •   Cancelar
Observações para devs

   •    Moradores podem marcar Ciente e Confirmar participação

   •    Alimenta dashboard do síndico



TELA S17 — LISTA DE EVENTOS

Caminho
S15 → Ver eventos → S17

Card

   •    Título

   •    Tipo

   •    Data

   •    Local

   •    Status

   •    Taxa de confirmação

Botões

   •    Editar evento

   •    Cancelar evento

   •    Ver respostas



MÓDULO CONNECT ME

TELA S18 — CONNECT ME

Caminho
S6 → Connect Me → S18

Objetivo
Gerenciar conexões entre síndico, empresas e moradores.

Cards

   •    Connect Me criado

   •    Connect Me recebido
Mensagem institucional
Conexões qualificadas fortalecem a gestão e aproximam síndicos, moradores e
empresas das necessidades reais do condomínio.



TELA S19 — CRIAR CONNECT ME

Caminho
S18 → Criar solicitação → S19

Campos

   •    Categoria do serviço

   •    Subcategoria

   •    Descrição da necessidade

   •    Cidade

   •    Bairro

   •    Compartilhar dados de contato mediante interesse? Sim/Não

Botão

   •    Enviar solicitação

Observações para devs

   •    Empresas visualizam oportunidade em seu painel

   •    Dados do síndico só aparecem após “Tenho interesse”



TELA S20 — CONNECT ME RECEBIDO

Caminho
S18 → Connect Me recebido → S20

Exibe

   •    Nome do morador

   •    Nome do condomínio

   •    Telefone

   •    E-mail

   •    Observação
Botões

   •     Tenho interesse

   •     Recusar

Regras

   •     Responder em até 48h

   •     Se não responder, mover para recusadas

Observações para devs

   •     Dados completos do morador só aparecem após “Tenho interesse”,
         conforme LGPD



MÓDULO REGISTROS DE EXECUÇÃO

TELA S21 — REGISTROS DE EXECUÇÃO

Caminho
S6 → Registros de execução → S21

Objetivo
Gerenciar registros enviados pelas empresas e publicados no condomínio.

Mensagem institucional
Os registros de execução transformam serviços prestados em prova institucional
da gestão e da entrega realizada.

Card

   •     Empresa

   •     Tipo de serviço

   •     Data

   •     Status

   •     Ação vinculada ao PD, se houver

Status

   •     Aguardando validação

   •     Solicitado ajuste

   •     Rejeitado

   •     Publicado
Botões

   •      Ver registro

   •      Filtrar registros

Filtros

   •      Empresa

   •      Tipo de atividade

   •      Status

   •      Período



TELA S22 — DETALHE DO REGISTRO DE EXECUÇÃO

Caminho
S21 → Ver registro → S22

Exibe

   •      Empresa responsável

   •      Descrição do serviço

   •      Área impactada

   •      Natureza

   •      Status do serviço

   •      Período de execução

   •      Garantia

   •      Orientações técnicas

   •      Materiais utilizados

   •      Anexos

Botões

   •      Validar registro

   •      Solicitar ajuste

   •      Rejeitar registro

Observações para devs
   •   Após validação:

          o     vira publicação da Linha do Tempo

          o     se estiver vinculado a uma atividade do PD, atualiza status da ação



MÓDULO COMUNICADOS

TELA S23 — COMUNICADOS

Caminho
S6 → Comunicados → S23

Objetivo
Gerenciar comunicados institucionais e técnicos.

Mensagem institucional
Comunicados bem estruturados reduzem ruído, fortalecem pertencimento e
organizam a comunicação entre gestão e comunidade.

Botões

   •   Criar comunicado

   •   Ver comunicados



TELA S24 — CRIAR COMUNICADO

Caminho
S23 → Criar comunicado → S24

Campos

   •   Tipo de comunicado

   •   Título

   •   Descrição

   •   Data de início

   •   Data de expiração

   •   Anexos

   •   Exibir “Ciente” ao morador? Sim/Não

Lista mestre — Tipos de comunicado

   •   Aviso operacional
   •   Interrupção programada

   •   Orientação aos moradores

   •   Aviso de segurança

   •   Comunicado institucional

   •   Conclusão de serviço

   •   Recomendação de manutenção

   •   Alerta preventivo

Botões

   •   Publicar comunicado

   •   Salvar rascunho

   •   Cancelar

Observações para devs

   •   Comunicado da empresa entra como pendente de validação nesse módulo

   •   Guardar histórico de versões



TELA S25 — LISTA DE COMUNICADOS

Caminho
S23 → Ver comunicados → S25

Card

   •   Tipo

   •   Título

   •   Data de início

   •   Data de expiração

   •   Status

   •   Taxa de leitura

Botões

   •   Editar

   •   Ocultar
   •   Ver métricas



MÓDULO VALIDAÇÕES PENDENTES

TELA S26 — VALIDAÇÕES PENDENTES

Caminho
S6 → Validações Pendentes → S26

Objetivo
Centralizar tudo que precisa de validação da gestão antes de se tornar registro
institucional.

Mensagem institucional
Toda informação enviada por empresas precisa ser validada antes de entrar no
histórico oficial do condomínio.

Seções

   •   Registros de execução

   •   Atividades criadas pela empresa

   •   Comunicados enviados pela empresa

Card padrão

   •   Empresa

   •   Tipo de envio

   •   Data

   •   Condomínio

   •   Status

Botões

   •   Validar

   •   Solicitar ajuste

   •   Rejeitar

   •   Ver detalhes

Observações para devs

   •   Toda decisão aqui gera auditoria

   •   Publicação validada entra na Linha do Tempo
MÓDULO DASHBOARD

TELA S27 — DASHBOARD DO SÍNDICO

Caminho
S6 → Dashboard → S27

Objetivo
Reunir os principais indicadores da gestão.

Mensagem institucional
Os indicadores da gestão ajudam a transformar rotina em estratégia, percepção
em dados e ações em evolução concreta do condomínio.

Indicadores

   •   Moradores cadastrados

   •   Percentual de unidades com acesso

   •   Avaliação média da gestão

   •   Ações do Plano Diretor:

          o   Planejadas

          o   Em contratação

          o   Em execução

          o   Concluídas

          o   Atrasadas

   •   Atividades publicadas

   •   Eventos realizados

   •   Comunicados enviados

   •   Registros de execução publicados

   •   Taxa de resposta às perguntas aos moradores

   •   Tendência de opinião dos moradores

   •   Taxa de leitura de comunicados

   •   Taxa de confirmação em eventos

Observações para devs
   •    Filtros por período

   •    Exportação futura para relatório



MÓDULO COMPLIANCE

TELA S28 — COMPLIANCE

Caminho
S6 → Compliance → S28

Objetivo
Reunir responsabilidade formal da gestão e trilha de auditoria.

Mensagem institucional
A governança condominial exige responsabilidade, transparência e registro
adequado das ações da gestão.

Cards

   •    Termo de Responsabilidade da Gestão

   •    Declaração Anual de Responsabilidade

   •    Auditoria da Gestão



TELA S29 — TERMO DE RESPONSABILIDADE DA GESTÃO

Caminho
S28 → Termo de Responsabilidade → S29

Texto
Declaro, na condição de responsável pela administração do condomínio
registrado nesta plataforma, que as informações inseridas e publicadas no
ambiente de governança refletem, de forma fidedigna, as ações realizadas no
âmbito da gestão condominial.

Comprometo-me a utilizar a plataforma Master Síndico de forma responsável,
registrando atividades, comunicações e decisões administrativas com veracidade
e boa-fé.

Estou ciente de que:

   •    a plataforma atua como ferramenta de registro e organização da gestão
        condominial;
   •   as decisões administrativas relacionadas ao condomínio são de
       responsabilidade exclusiva da administração e das instâncias deliberativas
       do próprio condomínio;

   •   os registros realizados poderão compor a memória institucional da gestão,
       sendo preservados para fins de histórico administrativo e rastreabilidade
       das ações.

Declaro, ainda, que utilizarei o sistema respeitando os princípios de
transparência, responsabilidade e boa gestão condominial.

Checkbox

   •   Li e concordo com o Termo de Responsabilidade da Gestão

Botões

   •   Assinar termo

   •   Voltar

Observações para devs
Registrar:

   •   user_id

   •   condominium_id

   •   version_term

   •   timestamp



TELA S30 — DECLARAÇÃO ANUAL DE RESPONSABILIDADE

Caminho
S28 → Declaração anual → S30

Mensagem institucional
A declaração anual reafirma o compromisso da gestão com a veracidade das
informações e com a condução responsável da administração do condomínio.

Texto
Declaro que, durante o período de gestão registrado nesta plataforma, as
informações publicadas pela administração refletem as ações realizadas no
condomínio e foram inseridas de boa-fé, com o objetivo de garantir transparência
e organização administrativa.

Reconheço que:
   •   os registros realizados na plataforma constituem histórico institucional da
       gestão;

   •   a plataforma Master Síndico atua como ferramenta de apoio à governança
       condominial, não sendo responsável pelas decisões administrativas do
       condomínio;

   •   eventuais deliberações e contratações realizadas no âmbito da gestão são
       de responsabilidade da administração condominial e das assembleias
       regularmente constituídas.

Checkbox

   •   Declaro que as informações registradas refletem, de boa-fé, as ações da
       gestão no período

Botões

   •   Registrar declaração anual

   •   Voltar

Observações para devs

   •   Exigir a cada 12 meses

   •   Alertar no dashboard se pendente



TELA S31 — AUDITORIA DA GESTÃO

Caminho
S28 → Auditoria → S31

Objetivo
Permitir rastreabilidade de tudo que foi feito na governança.

Mensagem institucional
A auditoria registra as atividades realizadas pela administração na plataforma,
garantindo transparência, continuidade e memória institucional.

Registros monitorados

   •   Criação de ação do Plano Diretor

   •   Alteração de ação do Plano Diretor

   •   Publicação de atividade

   •   Vinculação de atividade ao Plano Diretor
   •   Validação de registro de execução

   •   Solicitação de ajuste

   •   Rejeição de registro

   •   Publicação de comunicado

   •   Alteração de dados do condomínio

   •   Cadastro de evento

   •   Alteração de evento

   •   Atualização de mandato

   •   Registro de avaliação da empresa

   •   Criação de pergunta aos moradores

   •   Encerramento de vínculo administrativo relevante

Card da auditoria

   •   Data

   •   Hora

   •   Usuário responsável

   •   Tipo de ação

   •   Descrição da ação

Observações para devs

   •   Auditoria é imutável

   •   Não pode ser apagada nem editada
