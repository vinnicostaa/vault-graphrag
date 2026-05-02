---
title: ARQUITETURA FUNCIONAL DO MÓDULO ASSEMBLEIA (PDF do cliente — texto extraído)
type: note
tags: [archive, master-sindico, client-pdf, extracted-text]
source: Clients/MasterSindico/contents/ARQUITETURA FUNCIONAL DO MÓDULO ASSEMBLEIA.pdf
created: 2026-04-24
---

# ARQUITETURA FUNCIONAL DO MÓDULO ASSEMBLEIA

Texto extraído do PDF original do cliente João via `pdftotext -layout`. **Fidelidade limitada** — se PDF é majoritariamente imagem/diagrama visual, o texto extraído pode ser incompleto.

- **PDF original**: 206719 bytes
- **Texto extraído**: 9353 caracteres
- **Origem**: `~/Documentos/Obsidian Vault/Clients/MasterSindico/contents/ARQUITETURA FUNCIONAL DO MÓDULO ASSEMBLEIA.pdf`

## Texto extraído

```text
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
```

## Links

- [[_moc]]
