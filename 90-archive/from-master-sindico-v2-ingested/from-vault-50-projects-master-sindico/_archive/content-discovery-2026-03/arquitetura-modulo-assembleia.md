---
title: "ARQUITETURA FUNCIONAL DO MÓDULO ASSEMBLEIA"
type: source
tags: [pdf, converted]
source: 50-projects/master-sindico/content/contents/ARQUITETURA FUNCIONAL DO MÓDULO ASSEMBLEIA.pdf
converted: 2026-04-22
---

# ARQUITETURA FUNCIONAL DO MÓDULO ASSEMBLEIA

> PDF convertido via pdftotext. Layout original perdido; texto preservado.

ARQUITETURA FUNCIONAL DO MÓDULO ASSEMBLEIA — MASTER SÍNDICO

1. Objetivo do módulo

O módulo Assembleia da Master Síndico é a estrutura digital de:

   •   organização prévia da assembleia

   •   convocação e rastreabilidade de ciência

   •   preparação de quórum, procurações, fração ideal e elegibilidade de voto

   •   condução da assembleia presencial, híbrida ou online

   •   apoio à fala, votação e deliberação

   •   transparência operacional em tempo real

   •   geração de histórico, trilha de auditoria e base documental pós-assembleia

A plataforma não substitui:

   •   a convenção do condomínio

   •   a administradora

   •   o presidente da mesa

   •   a assessoria jurídica

   •   a formalização externa de assinatura da ata

A plataforma atua como:

   •   meio tecnológico de organização e registro

   •   ambiente de transparência e controle

   •   apoio operacional à assembleia



2. Estrutura macro do módulo

O módulo será dividido em 5 blocos:

   1. Configuração estrutural do condomínio

   2. Pré-assembleia

   3. Assembleia ao vivo

   4. Encerramento

   5. Pós-assembleia e histórico
3. CONFIGURAÇÃO ESTRUTURAL DO CONDOMÍNIO

3.1 Cadastro-base do condomínio

Campos:

   •     nome do condomínio (confirmar)

   •     endereço (confirmar)

   •     blocos (confirmar)

   •     unidades (confirmar)

   •     síndico principal (confirmar)

   •     subsíndico, (preencher)

   •     conselho, se houver (preencher)

   •     administradora, se houver (preencher)

   •     auditor, se houver (preencher)

3.3 Habilitação da administradora

Antes de estruturar a assembleia, o síndico precisa habilitar a administradora.

Caso seja empresa Master Síndico

Vinculação por:

   •     busca por CNPJ

   •     seleção no diretório Master

Caso não seja empresa Master Síndico

Criação de acesso externo controlado.

Fluxo:

   1. síndico informa empresa, CNPJ, responsável, CPF, e-mail e telefone

   2. sistema envia convite

   3. administradora valida o acesso por token

   4. confirma CPF do responsável

   5. aceita termo de responsabilidade

   6. passa a ter acesso apenas ao módulo assembleia daquele condomínio
3.4 Permissões da administradora

A administradora poderá:

   •   visualizar a assembleia em construção

   •   adicionar observações

   •   gerar edital automático ou

   •   anexar edital pronto

   •   subir planilha de fração ideal

   •   subir planilha de inadimplência

   •   validar procurações

   •   liberar voto de unidade regularizada

   •   registrar voto presencial assistido

   •   homologar votação

   •   encerrar assembleia

Observações da administradora poderão exigir validação do síndico,
conforme configuração.



4. PRÉ-ASSEMBLEIA

4.1 Criação da assembleia

O síndico cria a assembleia.

Campos:

   •   tipo: ordinária, extraordinária ou implantação

   •   data

   •   horário da 1ª convocação

   •   horário da 2ª convocação

   •   modalidade: presencial, híbrida ou online

   •   local físico, se houver

   •   link digital, se houver

   •   descrição geral
   •    tempo máximo de fala por morador: 2 ou 3 minutos com botão de
        prorrogação de mais 2 min.

4.2 Cadastro dos itens da pauta (quando o item estiver sendo apresentado na
assembleia o conteúdo abaixo deverá estar sendo apresentado na tela)

Os itens são ilimitados.

Cada item deve conter:

   •    título

   •    descrição

   •    item informativo ou deliberativo

   •    tipo do item

   •    impacto financeiro: sim ou não

   •    valor estimado, se houver

   •    quórum exigido para aprovação daquele item

   •    anexo de documento

   •    anexo de vídeo explicativo (caso ele queira fazer)

4.3 Tipos de item

Tipos possíveis:

   •    informativo

   •    votação simples

   •    votação por fração ideal

   •    escolha de fornecedor

   •    eleição

4.4 Regras por tipo de item

Informativo

Sem votação oficial.

Votação simples

Gera:

   •    enquete preliminar sem valor jurídico

   •    votação oficial na assembleia
Votação por fração ideal

Gera:

   •    enquete preliminar sem valor jurídico

   •    votação oficial com validação por peso da fração ideal

Escolha de fornecedor

Gera:

   •    enquete preliminar sem valor jurídico

   •    comparação de propostas

   •    votação oficial na assembleia

Eleição

Gera:

   •    avaliação/percepção da gestão, sem caráter jurídico (não será exibido aos
        moradores – hating imediato assim que é cadastrado o item assembleia – a
        avaliação deverá aparecer após o registro de ciência do morador)

   •    votação oficial de eleição na assembleia



As enquetes estarão disponíveis até 24h antes da assembleia.



4.5 Travamento jurídico da pauta

Depois que o síndico Publicar assembleia:

   •    não pode adicionar item

   •    não pode alterar título

   •    não pode alterar descrição

   •    não pode alterar quórum

   •    não pode alterar anexos estruturantes



4.6 Edital de convocação

Forma 1 — anexo livre

Administradora ou síndico anexa o edital pronto.
Forma 2 — gerador automático da plataforma tirar

Campos:

   •   identificação do condomínio

   •   tipo da assembleia

   •   data

   •   horários de 1ª e 2ª convocação

   •   modalidade

   •   local ou link

   •   ordem do dia importada da pauta

   •   regras de participação

   •   regras de procuração

   •   regras de votação

   •   logo da administradora

   •   campo de assinatura

Quem pode gerar:

   •   síndico

   •   administradora com permissão

4.7 Publicação da assembleia

Ao publicar, o sistema dispara:

   •   e-mail

   •   notificação no app

   •   SMS

Cada assembleia gera um QR code próprio.

4.8 Ciência obrigatória

A ciência é obrigatória.

Até registrar ciência:

   •   o app fica bloqueado para outras funções

O sistema registra e armazena a informação por 6 meses:
   •   quem deu ciência

   •   quando deu ciência

   •   quem ainda não deu ciência

   •   quem não abriu a comunicação

Síndico e administradora visualizam:

   •   percentual de ciência

   •   lista nominal de pendentes

   •   identificação literal de quem não abriu

4.9 Confirmação de participação

É opcional.

O morador pode marcar:

   •   participarei presencialmente

   •   participarei online

   •   ainda não defini

   •   Não participarei

Painel mostra:

   •   percentual de confirmação

   •   quem confirmou

   •   quem não confirmou

   •   quem marcou que não participaria

4.10 Lembretes automáticos da assembléia

Disparos:

   •   3 dias antes

   •   1 dia antes

   •   3 horas antes

4.11 Enquetes preliminares (opções de resposta apenas sim ou não)

Permitidas para:

   •   votação simples
   •   votação por fração ideal

Regras das enquetes:

   •   sem valor jurídico

   •   servem para engajamento e preparação

   •   não substituem deliberação

4.12 Fornecedores

Moradores podem acessar o perfil previamente (apenas durante o período da
assembleia)

   •   proposta

   •   escopo

   •   diferenciais

Fornecedor Master

Mostra:

   •   selo

   •   vídeo institucional

Fornecedor externo

Mostra:

   •   dados básicos

   •   proposta

   •   escopo

   •   indicação de empresa não verificada pela Master Síndico

4.13 Procurações

A procuração pode ser apresentada até o momento anterior ao síndico clicar
Iniciar assembleia.

Fluxo do morador

Após registrar ciência, ele pode:

   •   confirmar presença

   •   cadastrar procurações

Campos e anexos:
   •      bloco e unidade do representado (X)

   •      nome e CPF do representado

   •      bloco e unidade do procurador (X)

   •      nome e CPF do procurador

   •      checkbox de ciência e responsabilidade

   •      confirmação final com mensagem informando que ele precisa
          apresentar a procuração fisicamente para validação.

Fluxo da administradora

Visualiza:

   •      dados do procurador

   •      dados do representado

Ações:

   •      aprovar (ao aprovar, o número da unidade representante receberá o mesmo
          voto que a unidade representada) (X) – respeitando os pesos individuais

   •      rejeitar com justificativa

Efeito:

   •      voto adicional é computado automaticamente

   •      funciona para voto simples e fração ideal



Obs: vínculo da procuração após a votação é de apenas 24h, depois do
encerramento da assembleia perde o efeito.



4.14 Fração ideal

Para o MVP, a melhor arquitetura é upload por planilha.

Modelo da planilha

   •      bloco

   •      unidade

   •      fração ideal

   •      CPF titular
Regras de validação

    •   soma total das frações = 100% ou parâmetro equivalente

    •   ausência de duplicidade de bloco + unidade

    •   ausência de campos obrigatórios

Persistência

    •   importação única

    •   permanece salva no condomínio

    •   nova importação apenas por correção ou mudança estrutural



4.15 Inadimplência

Upload até 1 hora antes da 1ª convocação.

Pode ser feito por:

    •   administradora

    •   síndico

Sistema classifica:

    •   apto a votar

    •   inapto a votar

Após esse prazo, caso um morador apresente um comprovante de pagamento
de ultima hora.

Permissão especial e manual, deverá clicar no botão:
Liberar voto

Usuários com permissão para liberar voto:

    •   síndico

    •   administradora

Campos obrigatórios que a plataforma tem que registrar:

Checkbox de aprovação para votação (criar texto)

•   responsável

•   horário
4.16 Simulador de quórum

Síndico e administradora visualizam:

   •   total de unidades

   •   total de frações

   •   percentual de ciência

   •   percentual de confirmação de presença

   •   número de procurações registradas a serem aprovadas

   •   número de procurações aprovadas

   •   quórum projetado por item



4.17 Termos obrigatórios na pré-assembleia

Antes de concluir ciência ou participação, o morador deve aceitar:

   •   termo de ciência da convocação

   •   termo LGPD

   •   declaração de veracidade das informações prestadas

   •   declaração de identidade e legitimidade de uso do acesso da unidade

   •   checkbox de autorização de gravação, caso a assembleia seja gravada por
       terceiros

Declaração de identidade e veracidade

Texto funcional:
“Declaro, sob minha responsabilidade, que participo desta assembleia na
condição legítima vinculada à unidade informada e que as informações prestadas
são verdadeiras.”

Isso ajuda a isentar a Master Síndico quanto à verificação material da identidade
do votante.



5. ASSEMBLEIA AO VIVO

5.1 Formas de check-in

Cada assembleia possui QR code próprio.

O check-in pode ser feito de 3 formas:
1. Pelo app

Morador faz check-in pelo próprio celular.

2. Por QR code

Escaneia o QR da assembleia e entra.

3. Por terminal de apoio na entrada

Tablet ou notebook de apoio.

Morador informa:

   •   CPF

   •   bloco

   •   unidade

O sistema registra a presença digitalmente.

5.2 Como vota quem entrou pelo terminal

Esse morador vota por voto presencial assistido.

O voto será lançado no painel por:

   •   síndico

   •   administradora

O sistema registra:

   •   unidade

   •   horário

   •   operador responsável

   •   tipo de voto: presencial assistido

5.3 Tratamento do abandono da assembleia

Melhor prática funcional:

Regra de presença

Se o morador fizer check-in e sair, o sistema marca o status como:

   •   presente

   •   ausente momentâneo

   •   desconectado/saiu
Regra de voto

   •    votos já computados continuam válidos

   •    itens ainda não votados ficam indisponíveis enquanto ele estiver ausente,
        entra como abstenção.

   •    se ele retornar, volta a poder votar nos itens ainda abertos

Transparência

No telão e no painel:

   •    presentes

   •    online ativos

   •    ausentes/desconectados

Isso evita fraude e mostra claramente a base votante atual.

5.4 Painel inicial do síndico

Antes do início, o sistema exige cadastro de:

   •    presidente da mesa

   •    até 3 secretários

   •    administradora presente: nome da administradora, nome, função e CPF

   •    auditor (opcional): nome, função e CPF

   •    fornecedores presentes (opcional): nome da empresa e representante

5.5 Papéis do presidente da mesa

O presidente da mesa terá papel funcional claro:

Pode:

   •    abrir formalmente a assembleia

   •    conduzir a ordem dos trabalhos

   •    chamar itens da pauta

   •    autorizar e ordenar falas

   •    abrir e encerrar votação

   •    declarar item resolvido, adiado ou prejudicado

   •    encerrar discussão de item
   •   autorizar reabertura de item, quando necessário

   •   declarar resultado

   •   autorizar encerramento formal da assembleia

5.6 Início da assembleia

O síndico/administradora clica em Iniciar assembleia.

Antes da abertura:

   •   roda vídeo curto da Master Síndico explicando regras de participação, fala,
       votação e deliberação

5.7 Apresentação dentro da plataforma

O síndico, administradora ou auditor podem apresentar material sem sair da
plataforma. Disponibilizar opção do marco que o material devera ser inserido.

Formatos aceitos:

   •   PDF

   •   imagem

   •   vídeo

PowerPoint deve ser convertido em PDF antes do upload.

5.8 Estrutura do telão

O telão será dividido em duas áreas.

Área 1 — apresentação

Mostra:

   •   apresentação de documento de apoio (pptx, word, pdf, documentos e/ou
       vídeos (síndico/condomínio/administradora/auditor)).

Área 2 — painel operacional

Mostra:

   •   pauta toda (com marcação do item em destaque)

   •   unidades presentes

   •   unidades online (habilitar depois)

   •   unidades ausentes

   •   fila de mãos levantadas
   •     cronômetro da fala

   •     status dos itens

   •     quórum presente

   •     quórum necessário

   •     relógio da assembleia

5.9 Status dos itens

Status possíveis:

   •     resolvido

   •     adiado

   •     Não resolvido

   •     prejudicado

Cada item terá:

   •     hora de abertura

   •     hora de entrada em votação

   •     hora de encerramento

   •     duração total discussão do tema

5.10 Controle de fala

O tempo máximo de fala foi definido na criação da assembleia.

Fluxo:

   1. morador aperta “levantar mão”

   2. entra na fila

   3. pode abaixar a mão

   4. síndico ou presidente chama a unidade pelo microfone

         O microfone deverá ser habilitado de forma alternada para melhor
         captação do áudio e transcrição da gravação.

   5. cronômetro inicia

   6. fala é registrada (apenas as falas microfonadas serão registradas em ata).
      Orientar o sindico a falar isso antes da assembleia.

Painel do síndico mostra:
   •   fila completa de fala

   •   ordem

   •   unidade

   •   possibilidade de remover



   5.11 Votação oficial

Quando um item entra em votação:

   •   o sistema libera apenas unidades aptas

   •   aplica quórum e regra do item

   •   aplica peso da fração ideal, se houver

   •   considera procurações aprovadas

   •   aceita voto pelo app

   •   aceita voto presencial assistido

   •   tem que ter a opção de zerar e refazer a votação

Regra de segurança do voto

   •   um voto por unidade/posição legítima

   •   voto não pode ser alterado após enviado

   •   voto fica logado com data, hora e origem

Termo antes do voto

Antes de votar, o morador deve confirmar:
“Declaro que exerço este voto em nome da unidade vinculada ao meu acesso, sob
minha responsabilidade.”

5.12 Transparência no telão

O telão deve mostrar o conteúdo do voto.

Mostra:

   •   unidade que já votou

   •   unidade pendente

   •   total de votos realizados

   •   total restante
   •   quórum presente

   •   quórum necessário (se não atingir o quórum mínimo para o item, a
       votação será encerrada como inconclusiva e abrirá para uma nova
       votação).

   •   % de votos nas opções candidatas

5.13 Homologação da votação

Após o encerramento da votação de um item, poderá homologar:

   •   Síndico ou

   •   administradora

   botão: homologar votação (e encerrar para votos e aparece o resultado final).



6. ENCERRAMENTO DA ASSEMBLEIA

6.1 Condições de encerramento

Síndico ou administradora podem clicar em Encerrar assembleia.

O sistema verifica:

   •   se todos os itens têm status final

   •   se não há votação aberta

   •   se presidente e secretários estão cadastrados

6.2 Dados consolidados do encerramento

O sistema registra:

   •   data

   •   hora de início

   •   hora de término

   •   duração total

   •   presidente

   •   secretários

   •   administradora presente

   •   auditor presente

   •   fornecedores presentes
   •   total de presentes

   •   total de presentes online (quando habilitado)

   •   total de presentes presenciais

   •   total de ausências/desconexões durante a reunião

   •   lista de presença com CPF, bloco e unidade

   •   total de votos válidos

   •   votos por item

   •   votos por procuração

   •   votos presenciais assistidos

   •   unidades bloqueadas por inadimplência

   •   liberações manuais de voto

   •   homologações realizadas

   •   homologações pendentes

   •   duração de cada item apresentado no edital

6.3 Mensagem automática aos moradores

Ao encerrar, todos recebem mensagem:

“A assembleia foi encerrada em [data], das [hora início] às [hora término]. Os
resultados serão disponibilizados após conferência. Obrigado pela participação.”



7. PÓS-ASSEMBLEIA E HISTÓRICO

7.1 Relatórios exportáveis

O sistema deve permitir exportação de:

   •   CSV de presença

   •   CSV de votos

   •   CSV de registro de apresentação de procurações

   •   relatório por item

   •   relatório de inadimplência e liberações

   •   relatório de trilha de auditoria
7.2 Base para ata

O sistema gera base estruturada com:

   •   dados da assembleia

   •   participantes institucionais

   •   presença

   •   pauta

   •   status dos itens

   •   resultados das votações

   •   tempos registrados

   •   observações operacionais

A assinatura permanece externa.

7.3 Histórico permanente de assembleias

Cada condomínio terá uma linha do tempo com:

   •   data da assembleia

   •   tipo

   •   edital

   •   pauta

   •   relatórios

   •   exportações

   •   status final

   •   histórico de presença

   •   histórico de deliberações



8. INDICADOR DE TRANSPARÊNCIA DA ASSEMBLEIA

A plataforma calculará um indicador próprio.

Componentes do indicador

   •   percentual de ciência

   •   percentual de presença sobre convocados
   •   percentual de votantes sobre aptos

   •   percentual de leitura da pauta

   •   percentual de visualização de documentos e fornecedores

   •   completude da trilha de auditoria

   •   regularidade da homologação

   •   cumprimento integral da pauta

   •   tempo médio por item

   •   percentual de itens com documentação prévia

Exibição

Painel do síndico:

   •   índice geral

   •   subíndices de convocação, participação, votação e documentação



9. TRILHA DE AUDITORIA

Todos os eventos relevantes serão logados.

Eventos mínimos

   •   criação da assembleia

   •   habilitação da administradora

   •   geração/anexo de edital

   •   publicação

   •   ciência

   •   confirmação de presença

   •   upload de fração ideal

   •   upload de inadimplência

   •   cadastro de procuração

   •   aprovação/rejeição de procuração

   •   liberação manual de voto

   •   check-in
   •   início da assembleia

   •   cadastro do presidente e secretários

   •   alteração de status dos itens

   •   pedidos de fala

   •   votos

   •   votos presenciais assistidos

   •   homologações

   •   encerramento

Campos do log

   •   usuário responsável

   •   perfil

   •   data e hora

   •   assembleia

   •   condomínio

   •   unidade, quando aplicável

   •   justificativa, quando aplicável



10. RELÓGIO DA ASSEMBLEIA

Relógio geral

   •   início

   •   término

   •   duração total

Relógio por item

   •   abertura

   •   entrada em votação

   •   encerramento

   •   duração do item
11. MODO DE CONTINGÊNCIA

Esse ponto precisa existir para reduzir risco operacional.

Situações previstas

   •   queda de internet

   •   falha de dispositivo

   •   impossibilidade temporária de votação digital

Regras do modo contingência

   •   ativação manual pelo síndico ou administradora

   •   registro obrigatório do motivo

   •   continuação da assembleia com voto presencial assistido/manual

   •   marcação de que o item entrou em contingência

   •   exportação posterior da ocorrência no relatório

Transparência

No histórico do item deve constar:

   •   que houve contingência

   •   horário de início

   •   horário de normalização, se houver

   •   quem ativou



12. CLÁUSULAS FUNCIONAIS DE PROTEÇÃO DA MASTER SÍNDICO

A plataforma deve exibir, em momentos adequados, os seguintes termos:

12.1 Termo de uso da assembleia

A Master Síndico atua como meio tecnológico de organização e registro, não se
responsabilizando pelo mérito das deliberações nem pela verificação material da
representação civil além das informações e declarações fornecidas pelos
usuários e responsáveis do condomínio.

12.2 Declaração de veracidade

O usuário declara que:

   •   participa na condição legítima vinculada à unidade
   •   as informações prestadas são verdadeiras

   •   os documentos enviados correspondem à realidade

12.3 LGPD

O usuário toma ciência do tratamento de dados para:

   •   convocação

   •   participação

   •   votação

   •   rastreabilidade

   •   histórico da assembleia

12.4 Autorização de gravação

Se a assembleia for gravada, o usuário deve marcar checkbox específico de
ciência.



13. O QUE ESTA ARQUITETURA ENTREGA

Essa arquitetura resolve:

   •   convocação rastreável

   •   ciência obrigatória

   •   identificação de quem não abriu

   •   pauta estruturada e travada

   •   edital manual ou automático

   •   participação formal da administradora

   •   quórum por item

   •   impacto financeiro por item

   •   fração ideal única

   •   inadimplência com bloqueio e liberação

   •   procuração física digitalizada com validação

   •   check-in por app, QR e terminal

   •   voto presencial assistido
  •   assembleia presencial, online e híbrida

  •   ordem de fala com cronômetro

  •   apresentação do síndico dentro da plataforma

  •   telão com transparência operacional

  •   homologação da votação

  •   tratamento do abandono da assembleia

  •   relatórios exportáveis

  •   histórico permanente

  •   indicador de transparência

  •   trilha de auditoria

  •   relógio da assembleia

  •   modo de contingência

  •   proteção funcional da Master Síndico




salvamento automático de tudo que já deliberado na assembleia.
