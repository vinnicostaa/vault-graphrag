---
title: "JORNADA DO MORADOR"
type: source
tags: [pdf, converted]
source: 50-projects/master-sindico/content/contents/JORNADA DO MORADOR.pdf
converted: 2026-04-22
---

# JORNADA DO MORADOR

> PDF convertido via pdftotext. Layout original perdido; texto preservado.

JORNADA DO MORADOR

PAINEL DO MORADOR — MASTER SÍNDICO



REGRAS ESTRUTURAIS DO PERFIL MORADOR

Regra 1 — A Linha do Tempo é a principal fonte de informação

O morador visualiza:

   •   atividades da gestão

   •   registros de execução validados

   •   comunicados

   •   eventos

Tudo publicado pelo síndico ou validado pela gestão.



Regra 2 — Morador não publica conteúdos

O morador apenas:

   •   visualiza informações

   •   responde perguntas da gestão

   •   participa de eventos

   •   avalia a gestão



Regra 3 — Vínculo com unidade é obrigatório

O acesso ao conteúdo do condomínio depende do vínculo com uma unidade.

Cada unidade pode ter:

1 titular
até 2 dependentes



Regra 4 — Dependentes vinculados ao titular

Dependentes:

   •   não podem alterar status da unidade
   •   não podem remover titular

   •   não podem incluir novos dependentes



Regra 5 — Unidade vazia não permite dependentes

Se o status da unidade for Unidade vazia, dependentes são automaticamente
removidos.



TELA M1 — PAINEL DO MORADOR

Caminho

App → Botão Painel do Morador



Mensagem institucional

Este espaço conecta você à gestão do seu condomínio.

Aqui você acompanha o que está sendo feito, entende as prioridades da
administração e participa da construção de um ambiente mais organizado, seguro
e transparente.



Botões

   •   Selecionar condomínio

   •   Meu vínculo

   •   Avaliar gestão



Observações para devs

Se o morador tiver vínculo com mais de um condomínio, apresentar lista.



TELA M2 — SELECIONAR CONDOMÍNIO

Caminho

M1 → Selecionar condomínio
Mensagem institucional

Selecione o condomínio ao qual você deseja se conectar para acompanhar as
informações da gestão.



Botões

   •   Buscar condomínio por ID

   •   Ver condomínios vinculados



TELA M3 — BUSCAR CONDOMÍNIO

Caminho

M2 → Buscar condomínio



Campos

ID do condomínio



Validação automática

Sistema exibe:

   •   Nome do condomínio

   •   Endereço completo

   •   Cidade



Mensagem

Confirme se este é o condomínio correto antes de prosseguir com o cadastro da
unidade.



Botões

   •   Confirmar

   •   Buscar novamente
Observações para devs

Esse ID foi gerado na jornada do síndico.



TELA M4 — CADASTRO DA UNIDADE

Caminho

M3 → Confirmar condomínio



Mensagem institucional

Para acessar as informações do condomínio é necessário informar a unidade à
qual você está vinculado.



Campos

Bloco/Torre (se houver)
Número da unidade
Tipo da unidade



Lista mestre — tipo da unidade
Residencial
Comercial
Vaga de garagem


Tipo de vínculo

Proprietário residente
Proprietário não residente
Inquilino



Termo de ciência e responsabilidade

Declaro, sob minha responsabilidade, que possuo vínculo legítimo com a unidade
informada, seja na condição de proprietário, inquilino ou morador autorizado.

Estou ciente de que o acesso às informações do condomínio depende da
veracidade dessas informações e que a inserção de dados falsos poderá resultar
no bloqueio do acesso à plataforma.
Checkbox

Li e concordo com o termo de responsabilidade.



Botões

Cadastrar unidade
Cancelar



TELA M5 — HOME DO CONDOMÍNIO

Caminho

M4 → Unidade cadastrada



Mensagem institucional

Acompanhe a gestão do seu condomínio e participe das decisões que impactam o
ambiente onde você vive ou trabalha.



Cards

Linha do Tempo
Plano Diretor
Eventos
Comunicados
Avaliação da gestão
Meu vínculo



MÓDULO LINHA DO TEMPO



TELA M6 — LINHA DO TEMPO

Caminho

M5 → Linha do Tempo
Mensagem institucional

A Linha do Tempo apresenta as atividades registradas pela gestão do condomínio,
permitindo acompanhar o que foi realizado e o que está em andamento.



Tipos de publicações visíveis

Atividade da gestão
Registro de execução
Comunicado
Evento
Adendo
Resultado de pergunta aos moradores



Card da publicação

Data
Tipo
Descrição
Área impactada
Empresa envolvida (quando houver)



Botão

Ver detalhes



TELA M7 — DETALHE DA PUBLICAÇÃO

Caminho

M6 → Ver detalhes



Informações exibidas

Tipo de atividade
Descrição completa
Área impactada
Natureza da atividade
Nível de importância
Impacto esperado
Empresa responsável (se houver)
Período de execução
Anexos



Se existir pergunta aos moradores

Pergunta exibida

Opções de resposta

Sim
Não
Não sei



Botão

Enviar resposta



MÓDULO PLANO DIRETOR



TELA M8 — PLANO DIRETOR

Caminho

M5 → Plano Diretor



Mensagem institucional

O Plano Diretor apresenta as ações planejadas pela gestão para melhorar e
preservar o condomínio.



Card da ação

Nome da ação
Área impactada
Prazo previsto
Status



Status possíveis
Planejada
Em contratação
Em execução
Concluída
Suspensa
Reprogramada
Atrasada



Botão

Ver detalhes



TELA M9 — DETALHE DA AÇÃO

Caminho

M8 → Ver ação



Informações exibidas

Descrição da ação
Área impactada
Natureza
Impacto esperado
Prazo
Status atual



Seção

Histórico de atividades vinculadas à ação

Exibe atividades da Linha do Tempo relacionadas.



MÓDULO EVENTOS



TELA M10 — EVENTOS

Caminho
M5 → Eventos



Mensagem institucional

Eventos organizam atividades importantes do condomínio e permitem a
participação da comunidade.



Card do evento

Título
Tipo de evento
Data
Local



Botões

Confirmar participação
Marcar como ciente



MÓDULO COMUNICADOS



TELA M11 — COMUNICADOS

Caminho

M5 → Comunicados



Mensagem institucional

Os comunicados mantêm os moradores informados sobre atividades relevantes
da gestão.



Card

Título
Tipo
Data de início
Data de expiração
Botão

Marcar como ciente



MÓDULO AVALIAÇÃO DA GESTÃO



TELA M12 — AVALIAR GESTÃO

Caminho

M5 → Avaliar gestão



Mensagem institucional

Sua avaliação ajuda a administração a compreender a percepção da comunidade
e aprimorar a gestão do condomínio.



Perguntas

Qual seu nível de satisfação com a gestão do síndico?

Escala de 1 a 10

Você considera que a gestão tem sido transparente na comunicação com os
moradores?

Escala de 1 a 10

Você percebe evolução na organização do condomínio durante esta gestão?

Escala de 1 a 10



Botão

Enviar avaliação



Regras

Avaliação obrigatória bimestralmente.
Observações para devs

   •    não mostrar autor da avaliação

   •    registrar apenas estatística



MÓDULO MEU VÍNCULO



TELA M13 — MEU VÍNCULO

Caminho

M5 → Meu vínculo



Informações exibidas

Bloco
Unidade
Tipo de vínculo
Status da unidade



Lista mestre — status da unidade

Ocupada pelo proprietário
Ocupada por inquilino
Unidade vazia



Botão

Editar vínculo



TELA M14 — GERENCIAR ACESSOS

Caminho

M13 → Editar vínculo



Ações
Incluir dependente
Remover dependente
Atualizar status da unidade (titular)
Alterar vínculo (titular)



Mensagem de segurança

Não compartilhe sua senha.
Cada morador deve possuir seu próprio acesso.



Observações para devs

Dependentes aparecem vinculados ao titular.



TELA M15 — ENCERRAR VÍNCULO COM UNIDADE

Caminho

M13 → Encerrar vínculo



Mensagem

Se você não possui mais vínculo com esta unidade, é possível encerrar seu acesso
às informações do condomínio.



Motivos disponíveis

Mudança definitiva
Venda da unidade
Encerramento de contrato de locação
Erro no cadastro



Botão

Encerrar vínculo



Observações para devs

Manter histórico da participação do usuário.
