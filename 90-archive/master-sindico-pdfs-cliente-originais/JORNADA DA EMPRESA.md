---
title: JORNADA DA EMPRESA (PDF do cliente — texto extraído)
type: note
tags: [archive, master-sindico, client-pdf, extracted-text]
source: Clients/MasterSindico/contents/JORNADA DA EMPRESA.pdf
created: 2026-04-24
---

# JORNADA DA EMPRESA

Texto extraído do PDF original do cliente João via `pdftotext -layout`. **Fidelidade limitada** — se PDF é majoritariamente imagem/diagrama visual, o texto extraído pode ser incompleto.

- **PDF original**: 62183 bytes
- **Texto extraído**: 6190 caracteres
- **Origem**: `~/Documentos/Obsidian Vault/Clients/MasterSindico/contents/JORNADA DA EMPRESA.pdf`

## Texto extraído

```text
JORNADA DA EMPRESA — PAINEL EMPRESARIAL

Plataforma Master Síndico

A jornada da empresa é responsável por três pilares:

• relacionamento com síndicos (Connect Me)
• registro técnico das atividades executadas
• construção da reputação institucional

Todas as interações da empresa com o condomínio passam pela validação do
síndico, garantindo governança e rastreabilidade.



TELA E1

PAINEL EMPRESARIAL

Caminho
App → Painel Empresarial

Mensagem institucional

Este é o espaço onde sua empresa se conecta ao ecossistema condominial.
Aqui você acompanha oportunidades, registra atividades técnicas e fortalece a
transparência dos serviços prestados aos condomínios.

Cards disponíveis

Oportunidades (Connect Me)
Registrar execução
Atividades técnicas
Comunicados técnicos
Status de envios
Dashboard empresarial
Perfil institucional
Usuários da empresa
Gestão de agência de marketing
Marketing Link

Botões

Acessar módulo

Regras

Todos os envios da empresa passam por validação do síndico antes de aparecer
para moradores.
Observação para devs

Painel deve apresentar também notificações de pendências, como:

registros aguardando validação
comunicados rejeitados
oportunidades novas.



TELA E2

OPORTUNIDADES (CONNECT ME)

Caminho
Painel Empresarial → Oportunidades

Mensagem institucional

Síndicos podem registrar demandas de serviço para empresas da plataforma.
Aqui você acompanha as oportunidades e decide se deseja participar da
conversa.

Cards exibidos

Condomínio
Cidade
Bairro
Categoria do serviço
Subcategoria
Descrição da necessidade
Data da solicitação

Botões

Tenho interesse
Recusar



TELA E3

RECUSAR OPORTUNIDADE

Caminho
E2 → Recusar

Mensagem institucional
Ao recusar uma solicitação, informe o motivo para manter a transparência da
relação com o síndico.

Campos

Motivo da recusa

Lista de opções

Fora da área de atendimento
Serviço fora da especialidade da empresa
Agenda indisponível no período solicitado
Conflito com serviços já agendados
Orçamento incompatível com o escopo
Outro motivo

Campo adicional

Justificativa detalhada

Botão

Enviar recusa

Regras

Sistema envia automaticamente mensagem ao síndico informando a recusa.

Observação para devs

Registrar histórico da recusa no banco de dados da oportunidade.



TELA E4

DETALHE DA OPORTUNIDADE

Caminho
E2 → Tenho interesse

Mensagem institucional

Ao demonstrar interesse, sua empresa passa a ter acesso aos contatos do síndico
para continuidade da conversa.

Informações exibidas

Nome do condomínio
Nome do síndico
Telefone
Email
Descrição completa da demanda

Botões

Entrar em contato
Marcar como em negociação
Encerrar oportunidade

Regras

Empresa pode atualizar status da oportunidade para organização interna.



TELA E5

REGISTRAR EXECUÇÃO

Caminho
Painel Empresarial → Registrar execução

Mensagem institucional

Registrar a execução de um serviço fortalece a transparência da gestão e contribui
para a memória técnica do condomínio.

Campos

Condomínio atendido
Tipo de serviço executado
Descrição da execução
Área impactada
Natureza da atividade
Status do serviço
Data da execução
Garantia do serviço
Orientações técnicas ao condomínio
Materiais utilizados (opcional)
Fotos ou vídeos

Botões

Salvar rascunho
Enviar para validação do síndico

Regras
Após validação do síndico, o registro será publicado na Linha do Tempo do
condomínio.

Observação para devs

Registro deve aparecer no módulo Validações Pendentes do Síndico.



TELA E6

ATIVIDADE TÉCNICA

Caminho
Painel Empresarial → Atividade técnica

Mensagem institucional

Empresas podem contribuir com informações técnicas relevantes para a gestão
do condomínio, fortalecendo a prevenção de problemas.

Campos

Tipo de atividade técnica
Descrição técnica
Área impactada
Nível de importância
Risco identificado
Impacto esperado
Recomendação técnica
Anexos

Botões

Salvar rascunho
Enviar para validação do síndico

Regras

Após validação, a atividade aparece na Linha do Tempo do condomínio.



TELA E7

COMUNICADOS TÉCNICOS

Caminho
Painel Empresarial → Comunicados técnicos

Mensagem institucional
Comunicados técnicos permitem orientar moradores e gestores sobre cuidados,
recomendações e informações relevantes após a execução de serviços.

Campos

Tipo de comunicado
Título
Descrição

Encerrar acesso



MARKETING LINK

O Marketing Link é um mecanismo de conexão entre empresa e agência de
marketing.

Fluxo:

Empresa acessa perfil da agência
→ clica em Marketing Link
→ preenche formulário
→ agência recebe solicitação



TELA E16

FORMULÁRIO MARKETING LINK

Caminho
Perfil da agência → Botão Marketing Link

Mensagem institucional

Se sua empresa deseja fortalecer sua presença institucional na plataforma Master
Síndico, registre aqui sua solicitação de contato com esta agência de marketing.

A agência receberá suas informações e poderá entrar em contato para entender as
necessidades da sua empresa.



CAMPOS DO FORMULÁRIO

Empresa

Nome da empresa
Nome do responsável
Telefone
Email



NECESSIDADE DE MARKETING

Tipo de apoio desejado

Produção de vídeos institucionais
Gestão de conteúdos técnicos
Estratégia de posicionamento no mercado condominial
Gestão de redes sociais
Consultoria de marketing para empresas de serviço
Outro



PRIORIDADE

Imediato
Nos próximos 30 dias
Sem prazo definido



DESCRIÇÃO DA NECESSIDADE

Campo aberto

Descreva brevemente o tipo de apoio que sua empresa procura.



BOTÕES

Enviar solicitação
Cancelar



REGRAS OPERACIONAIS

Ao enviar o formulário:

A agência recebe:

• nome da empresa
• telefone
• email
• descrição da necessidade
A solicitação aparece no painel da agência em:

Solicitações de Marketing Link



OBSERVAÇÃO PARA DESENVOLVEDORES

O Marketing Link:

• não gera vínculo automático entre empresa e agência
• apenas gera uma solicitação de contato

A conexão entre empresa e agência só acontece quando:

empresa decide convidar a agência para gerenciar seu perfil institucional.

```

## Links

- [[_moc]]
