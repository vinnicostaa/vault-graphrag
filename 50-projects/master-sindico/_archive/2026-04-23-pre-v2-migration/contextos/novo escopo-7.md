---
title: "novo escopo-7"
type: source
tags: [pdf, converted]
source: 50-projects/master-sindico/contextos/novo escopo-7.pdf
converted: 2026-04-22
---

# novo escopo-7

> PDF convertido via pdftotext. Layout original perdido; texto preservado.

ANEXO I: ESCOPO TÉCNICO E
CRONOGRAMA DE ENTREGAS (SOW)
PROJETO: PLATAFORMA MASTER SÍNDICO (WEB & MOBILE)

1. DEFINIÇÃO DO OBJETO
Desenvolvimento de plataforma digital do tipo CMS Headless (Content Management
System) voltada à gestão condominial e integração com comércio local. A solução é
composta por três artefatos de software distintos e integrados:
   1. Backend (API/Core): Responsável por regras de negócio, banco de dados e
      integrações.
   2. Frontend Web: Aplicação responsiva para navegadores.
   3. Aplicativo Mobile: Aplicação nativa (iOS/Android) para consumo de conteúdo e
      interação.




2. PREMISSAS TÉCNICAS E LIMITADORES GERAIS
Para fins de aceite e validação das entregas, ficam estabelecidas as seguintes regras:

2.1. Paridade de Funcionalidades (Web/Mobile)
O Aplicativo Mobile espelhará estritamente as funcionalidades de consumo e interação
disponíveis na versão Web para os perfis finais (Síndico, Morador, Empresa).
Funcionalidades de administração global da plataforma ("MasterSíndico") são
exclusivas da Web.

2.2. Limitação de Notificações (Email/SMS)
O sistema NÃO enviará notificações de interações sociais entre usuários (ex: "fulano
curtiu seu vídeo", "novo comentário", "empresa postou vídeo"). O disparo de E-mails e
SMS limita-se estritamente a:
   •   (a) Validação de cadastro/segurança (confirmação de email, redefinição de
     senha);
   • (b) Formulário de contato "Connect Me" (disparo único de formulário
     preenchido);
   • (c) Comunicados oficiais do Administrador MasterSíndico (broadcasts
     institucionais).
Importante: O custo de disparos via gateways de terceiros (ex: AWS SES, SendGrid,
Twilio, Zenvia) é de responsabilidade do CONTRATANTE e não está incluso no
desenvolvimento.

2.3. Privacidade de Métricas e Feedback
A plataforma não exibe contadores públicos de engajamento. Métricas de "Likes" e
"Comentários" são visíveis exclusivamente para o autor do conteúdo (dono do perfil).
Para terceiros, a interface é "cega" (sem contadores, sem lista de comentários).
Regra de Visualização:
   • Se Usuario_Logado == Dono_do_Conteudo: Exibe contador de likes e lista de
       comentários.
   •   Se Usuario_Logado != Dono_do_Conteudo: Oculta contadores e comentários.
       Usuário vê apenas o conteúdo.

2.4. Regra de Upload Trimestral
Vídeos classificados como "Institucionais" (Empresas) e "Vídeo-Currículo" (Moradores)
possuem trava sistêmica (hard-coded) permitindo apenas 1 (uma) atualização a cada
90 dias.
Lógica de Negócio (Backend):
   • Antes de autorizar upload, o sistema verifica: Existe vídeo deste tipo postado nos
     últimos 90 dias?
         • Se SIM → Bloqueia Upload.
         • Se NÃO → Permite Upload e substitui/arquiva o anterior.

2.5. Onboarding Parametrizado (Cadastro Inteligente)
O cadastro de usuários ocorre mediante interpretação de parâmetros de URL ou
metadata (provenientes de Landing Pages externas), definindo automaticamente o
perfil e plano do usuário no ato do registro.
Fluxo de Cadastro (Stripe-like):
   1. Usuário acessa URL parametrizada: mastersindico.com/cadastro?
      role=sindico&plan=pro
   2. Sistema identifica a Role e Plano via SearchParams/Metadata
   3. Cadastro inicial → Disparo de Email de Validação
   4. Usuário confirma email → Define senha e ativa conta
   5. Backend cria registro no DB com a Role correta já travada
2.6. Vídeo-Currículo como Vídeo Fixo no Perfil
O vídeo-currículo do Morador Pagante é um vídeo fixo exibido no perfil, sujeito à trava
trimestral de 90 dias. Não é uma galeria ou lista de vídeos, mas um único vídeo
institucional de apresentação profissional (até 90 segundos).




3. DETALHAMENTO DAS SPRINTS (CRONOGRAMA DE
EXECUÇÃO)
SPRINT 1: FUNDAÇÃO, IDENTIDADE E ACESSO
Foco: Garantia de acesso seguro, gestão de perfis, motor de visibilidade e onboarding
inteligente.

1.1. Arquitetura de Permissões (Core ABAC)
Backend:
    • Implementação do sistema de roles (Base, Morador Pg, Síndico N1/N2/N3, Emp.
      Plus/Pro, Marketing, Vizinhança).
    • Lógica estritamente baseada na Matriz Funcional.
    • Trava de Escopo: Não haverá criação de "permissões personalizadas" dinâmicas
      pelo admin nesta fase.
Web & Mobile:
    • Middleware de proteção de rotas.
    • Se o usuário não tem o plano, a tela nem carrega (retorna 403).

1.2. Onboarding e Autenticação (Fluxo Stripe-like)
Web & Mobile:
    • Página de Cadastro Inteligente: O sistema identifica o tipo de usuário via
      SearchParams/Metadata da URL.
    • Exemplo: mastersindico.com/cadastro?role=sindico&plan=pro
    • Fluxo de Ativação: Cadastro inicial → Disparo de Email de Validação → Definição
      de Senha e Ativação.
Backend:
    • Validação de unicidade de e-mail.
    • Criação do registro no DB com a Role correta travada.
    • Sistema de autenticação JWT + Passport.
Limitação (Anti-Scope Creep): Não há validação de documentos por OCR ou biometria
facial. Validação é via e-mail/token simples.

1.3. Gestão de Perfis e Visibilidade
Backend:
   • CRUD de usuários com campos específicos por persona:
        • Morador: Dados básicos + vínculo com unidade.
        • Síndico: Dados profissionais + vínculo com múltiplos condomínios (para
          cálculo de Status Bronze/Prata/Ouro).
        • Empresa: Dados PJ + Seleção de até 5 categorias técnicas (conforme
          Manual Cap. 6).
Web (Frontend):
   • Tela de Cadastro/Login.
   • Edição de Perfil: Upload de foto, bio, e dados cadastrais.
   • Configurações de Usuário: Alterar senha, gerenciar notificações (on/off).
Mobile:
   • Espelhamento exato das telas de Login, Cadastro e Edição de Perfil da Web.
Regra de Visibilidade:
   • Perfil do Morador Pagante: Agora visível e indexável na busca para empresas
     (conforme alteração da Matriz Funcional). Inclui área para Vídeo-Currículo (ver
     item 1.4).
   • Perfil da Empresa: Exibição de dados e categorias.
   • Perfil do Síndico: Dados profissionais e Status (Bronze/Prata/Ouro/Diamante).
Backend (Trava de Segurança): O perfil só aparece na busca se o plano permitir
(middleware de visibilidade).

1.4. Motor de Upload com Trava Temporal (Vídeo Institucional/Currículo)
Lógica de Negócio (Backend):
    • Implementação da regra "Trimestral": Antes de autorizar o upload de um vídeo
      do tipo "Institucional" (Empresa) ou "Currículo" (Morador), o sistema verifica:
          • Existe vídeo deste tipo postado nos últimos 90 dias?
                 • Se SIM → Bloqueia Upload.
                 • Se NÃO → Permite Upload e substitui/arquiva o anterior.
Frontend (Web & Mobile):
   • Exibição do vídeo fixo no perfil.
   • Mensagem de bloqueio quando usuário tenta fazer upload antes dos 90 dias.
Vídeo-Currículo (Morador Pagante):
   •   Vídeo de até 90 segundos.
   •   Fixo no perfil (não é galeria).
   •   Sujeito à trava trimestral.
   •   Visível para Empresas que tenham acesso à funcionalidade "Visualização de
       Currículos".

1.5. Motor de Busca e Catálogo (Search Engine)
Backend:
   • Endpoints de busca com filtros:
        • Busca Geral: Retorna mix de resultados (Síndicos, Empresas, Comércio
          Local).
        • Filtro por Categoria de Serviço (Manual Cap. 6).
        • Filtro por CEP (Exclusivo para Comércio Local/Vizinhança).
Web (Frontend):
   • Página de Busca com filtros laterais.
   • Listagem de Cards (Empresas, Síndicos, Moradores Pagantes).
   • Regra de Visibilidade: Perfil só aparece se a Matriz Funcional permitir.
Mobile:
   • Tela de Busca nativa com os mesmos filtros da Web.

1.6. Funcionalidade "Connect Me" (Formulário de Contato Unidirecional)
Definição: Formulário de contato unidirecional. Não constitui sistema de chat. Não
salva histórico de conversa na plataforma, não tem reply, não tem chat.
Fluxo:
   1. Usuário preenche formulário no perfil do destinatário (Síndico ou Empresa).
   2. Backend processa.
   3. Sistema dispara um formulário para o destinatário com os dados do interessado.
Regras de Direção (conforme Matriz Funcional):
   • Morador → Síndico (com cota anual: 2/ano para Base, 4/ano para Pagante).
   • Síndico → Empresa.
   • Empresa Plus/Pro → Empresa (entre empresas).
Limitação - Anti-Prospecção:
   • Empresas de Marketing NÃO têm botão Connect Me para abordar usuários
       proativamente (Anti-spam/Prospecção).
   • O contato parte sempre do interesse do contratante (Morador/Síndico) para o
     contratado, via busca.
Backend:
   • Validação de cota de uso.
   • Log de disparos.
Importante: Não é SISTEMA DE ENVIO de EMAIL/SMS.
1.7. Player de Vídeo e Consumo (Mux/Cloudflare Integration)
Backend:
   • Integração API Mux ou Cloudflare Stream para upload e streaming.
   • Lógica de Paywall: Se usuário for "Base" e vídeo for de "Empresa", cortar stream
     aos 25% (Matriz Funcional).
Web (Frontend):
   • Player de vídeo customizado.
   • Contador de visualização (regra: contar apenas se >70% assistido para
     pontuação do síndico).
Mobile:
   • Player nativo otimizado para mobile.
Limitação: Não há editor de vídeo dentro da plataforma. O upload é do arquivo pronto.


SPRINT 2: MOTOR DE GESTÃO CONDOMINIAL (CMS DE EVENTOS)
Foco: Ferramentas operacionais do Síndico, Transparência e Sistema de Avaliação.
2.1. Gestão de Assembleias e Eventos (O Coração do CMS)
Backend:
   • CRUD de Eventos.
   • Estrutura: Título, Data, Edital (PDF/Texto), Status (Agendado, Aberto, Encerrado).
   • Pautas: Sub-itens do evento contendo:
         • Título
         • Descrição
         • Vídeo/Áudio explicativo do Síndico
         • Anexos (Vídeos de fornecedores)
Web (Área do Síndico):
   • Wizard de criação de Assembleia.
   • Upload de vídeos/áudios por pauta.
   • Seleção de vídeos de fornecedores (buscando do acervo das empresas
     cadastradas).
   • Geração de Link Temporário: Criar link de compartilhamento de eventos (edital,
     pautas, votação) com janela de acesso definida.
Mobile:
  • Morador: Visualiza a lista de assembleias. Entra na "Sala da Assembleia"
     (interface passiva de consumo das pautas).
   • Síndico: Pode criar/editar assembleias simples pelo celular (upload de vídeos da
     galeria).
Limitação: Não é votação em blockchain. Não é assembleia virtual ao vivo
(Zoom/Meet). É assíncrono.

2.2. Sistema de Votação Assíncrona
Backend:
   • Lógica de votação vinculada a uma Pauta.
   • Tipos: Aprovar/Reprovar ou Múltipla Escolha (Fornecedor A, B, C).
   • Segurança:
        • 1 voto por unidade (exige regra de negócio para validar unidade).
        • Janela de tempo (só vota se o evento estiver "Aberto").
Web & Mobile:
   • Interface de votação.
   • Feedback visual imediato ("Seu voto foi registrado").

2.3. Painel de Transparência e Gestão
Backend:
   • Endpoints para:
         • Plano de Reeleição
         • Percentual de Execução do Plano Diretor
         • Timeline da Gestão
   • Gerador de PDF: Compilar a "Linha do Tempo" da gestão em um arquivo
     estático.
Web (Área do Síndico):
   •   Dashboard para inputar dados: "O que prometi" vs "O que fiz".
   •   Campo para Atividades Planejadas e Percentual de Execução.
   •   Campo para Observações.
   •   Botão "Exportar Gestão em PDF".
Mobile (Morador):
   • Visualização da Timeline da gestão (apenas leitura).

2.4. Avaliação Objetiva da Gestão (Rating)
Backend:
   • Formulário de avaliação fixo (perguntas definidas no sistema).
   • Perguntas são objetivas e geradas pela plataforma (o síndico não cria
     perguntas personalizadas - Manual Cap 5, item 3.3).
   • Média ponderada.
   • Sistema de Janela de Acesso: Avaliação disponível antes de atividades ou
     eventos específicos definidos pelo MasterSíndico.
Web & Mobile:
    • Morador: Tela de avaliação (estrelas + tags + perguntas objetivas).
    • Síndico: Visualiza nota consolidada e resultado da avaliação.
Privacidade: O resultado consolidado é visível para o Síndico. Moradores veem apenas
que "avaliaram", mas não veem avaliações de outros moradores.
Limitação: Perguntas são fixas do sistema, o síndico não cria perguntas
personalizadas.



SPRINT 3: VIZINHANÇA E PROMOÇÕES (MOTOR DE CUPONS)
Foco: Integração com comércio local. Sistema de cupons de desconto (substitui o
conceito anterior de "Tickets de Suporte").

3.1. Engine de Cupons ("Promoção do Dia")
Definição: Mecanismo de geração de códigos promocionais para uso no comércio
físico. NÃO É SISTEMA DE HELPDESK/CHAMADO DE SUPORTE.
Backend (Lógica Crítica):
     • Gerador de Código Promocional: Formato ID_LOJA(5) + SEQUENCIAL(3) +
      PALAVRA(5).
        • Exemplo: Loja ID 00123 + Seq 055 + Palavra PAO = 00123055PAO
        • ID_LOJA: 5 dígitos (ID único do comércio na plataforma)
        • SEQUENCIAL: 3 dígitos (número sequencial do cupom gerado)
        • PALAVRA: 5 letras (palavra-chave do dia definida pelo lojista)
   • Trava de Tempo: O usuário final (Morador) só pode gerar 1 cupom a cada 4
     horas para o mesmo estabelecimento.
   •   Lógica de Validação:
          • Sistema valida se já existe cupom ativo gerado pelo usuário nas últimas 4h.
          • Se SIM → Bloqueia geração.
          • Se NÃO → Gera novo cupom e exibe na tela.
Web (Painel da Vizinhança/Lojista):
   • Campo para definir a "Palavra do Dia" (máximo 5 letras, apenas letras
     maiúsculas).
   • Visualização simples de quantos cupons foram gerados no dia/semana/mês.
   • Interface para cadastro de "Promoção do Dia" (Título, Regra, Validade).
Mobile & Web (Morador/Síndico):
  • Aba "Clube de Benefícios" ou "Vizinhança".
   • Busca de benefícios filtrada por CEP.
   • Listagem de ofertas baseadas no CEP do usuário.
   • Botão "Pegar Cupom/Promoção":
         • Sistema valida tempo (4h).
         • Se permitido: Gera código no padrão definido e exibe na tela.
         • Se bloqueado: Exibe mensagem "Você já possui um cupom ativo. Aguarde
           X horas para gerar novo cupom."
Funcionalidade Adicional para Síndico:
   • Síndico pode ativar promoções exclusivas para os moradores de seus
     condomínios, a partir do número de inscrição.
   • Estabelecimento (cadastrado na plataforma) pode optar por:
         • Promoção exclusiva a um condomínio (via número de inscrição negociado
           com síndico).
         • Promoção regional (aberta para todos os usuários da região/CEP).
Limitação: O termo "Ticket" no código e na interface refere-se exclusivamente ao
Cupom de Desconto gerado no padrão numérico definido. Não existe sistema de
"Helpdesk/Chamado de Suporte" entre moradores e empresas.



SPRINT 4: CONTEÚDO EDUCACIONAL, FEEDBACK PRIVADO E
ADMINISTRAÇÃO MASTER
Foco: LMS (Learning Management System), Engajamento Privado, Fórum, Lives e
Controle Global da Plataforma.
4.1. Sistema de Cursos e Treinamentos (LMS)
Backend:
     • Estrutura hierárquica: Curso → Módulo → Aula (Vídeo + Descrição + Anexo).
   • Controle de progresso do aluno (% concluído por curso/módulo).
   • Geração de Certificado: PDF dinâmico com nome, data e QR Code de validação
     ao completar 100% do curso.
Web (Área da Empresa/Marketing):
  • Construtor de Curso (Create Course Wizard):
         •   Criar curso.
         •   Adicionar módulos.
         •   Upload de aulas (vídeos).
         •   Upload de Ebooks (PDF) como material complementar.
Mobile & Web (Aluno - Síndico/Morador):
  • Área "Meus Cursos":
         •   Listagem de cursos inscritos.
         •   Player de aula (semelhante ao Youtube/Udemy).
         •   Barra de progresso do curso.
         •   Download de Certificado (quando 100% concluído).
Acesso a Cursos (conforme Matriz Funcional):
   • Base: Sem acesso a módulos/cursos.
   • Morador Pg: Sem acesso (salvo se curso for público/gratuito).
   • Síndico N1: Sem acesso a cursos certificados.
   • Síndico N2/N3: Acesso a módulos e cursos com certificado.
   • Emp. Plus: Sem permissão para criar cursos.
   • Emp. Pro: Pode criar módulos e cursos certificados.


4.2. Sistema de Feedback Privado ("Rede Social Cega")
Conceito: A plataforma não é uma rede social. Engajamento não é palco, é sinal.
Lógica de Negócio (Backend):
    • Sistema registra Likes e Comentários no banco de dados.
    • Regra de Visualização (Frontend):
          • Se Usuario_Logado == Dono_do_Conteudo: Exibe contador de likes e lista
             de comentários.
         •   Se Usuario_Logado != Dono_do_Conteudo: Oculta contadores e
             comentários. Usuário vê apenas o conteúdo (vídeo, post).
Objetivo:
   • Evitar "rede social" e competição pública de vaidade.
   • Proteger privacidade das interações.
   • Gerar métricas internas para análise da plataforma sem expor números
     publicamente.
Web & Mobile:
  • Interação: Usuários podem curtir e comentar conteúdos (vídeos, posts do
       fórum).
   •   Exibição:
           • Dono do Conteúdo: Painel privado mostrando total de likes, lista de
              comentários, métricas de visualização.
            • Público Geral: Interface "cega" – sem contadores, sem lista de
              comentários visível. Apenas consome o conteúdo.
Backend:
   • APIs de registro de likes/comentários.
   • APIs de consulta de métricas (acessível apenas ao dono do conteúdo e ao
     MasterSíndico).

4.3. Fórum e Comunidade
Backend:
   • Categorias de tópicos (definidas conforme categorias técnicas do Manual Cap.
     6).
   • Posts e Respostas.
   • Permissões: Quem pode postar vs. Quem pode ler (conforme Matriz Funcional).
   • Sistema de Likes em posts/respostas (com visualização privada conforme item
     4.2).
Web & Mobile:
   • Interface de Fórum:
         • Listagem de Tópicos.
         • Tela de Detalhe do Post (com respostas).
         • Funcionalidade de "Curtir" (Like).
         • Sistema de busca por palavra-chave.
Moderação:
  • Reativa (botão de denúncia).
   • Não há IA de moderação automática.
   • MasterSíndico pode aprovar/reprovar/remover posts denunciados.
Acesso ao Fórum (conforme Matriz Funcional):
   • Base: Sem acesso.
   • Morador Pg: Sem acesso.
   • Síndico N1/N2/N3: Participação completa.
   • Emp. Plus/Pro: Participação completa.
   • Marketing: Participação completa.

4.4. Lives Institucionais (Streaming ao Vivo)
Restrição Crítica: Apenas o Administrador MasterSíndico (Dono da Plataforma) pode
iniciar transmissões ao vivo. Síndicos e Empresas NÃO fazem live (Matriz Funcional).
Backend:
   • Integração Mux Live Streaming ou Cloudflare Stream.
   • Ingest RTMP: Geração de chave RTMP para OBS/software de transmissão.
   • Gravação automática da live (disponibilizar após término).
   • Controle de acesso (apenas MasterSíndico pode criar/gerenciar lives).
Web (Painel MasterSíndico):
   • Tela para iniciar transmissão (gerar chave RTMP).
   • Painel de controle (iniciar/encerrar live).
   • Configurações de chat (ativar/desativar).
Mobile & Web (Usuários Finais):
  • Notificação Push/Email: "Live ao vivo agora!"
  • Player de Live na home ou aba dedicada.
   • Chat opcional (se habilitado pelo MasterSíndico).
Acesso a Lives (conforme Matriz Funcional):
   • Base: Acesso às lives gravadas (após término).
   • Morador Pg: Acesso às lives ao vivo e gravadas.
   • Síndico N1/N2/N3: Acesso às lives ao vivo e gravadas.
   • Emp. Plus/Pro: Acesso às lives ao vivo e gravadas.
   • Marketing: Acesso às lives ao vivo e gravadas.

4.5. Aba de Podcast (Integração YouTube)
Funcionalidade Transversal: Disponível para usuários conforme plano.
Backend:
   • APIs de integração com YouTube (via YouTube Data API).
   • Cache de metadados (título, descrição, thumbnail).
   • Controle de acesso conforme plano.
Web & Mobile:
  • Player YouTube embarcado (iframe ou player nativo).
   • Listagem de episódios/vídeos do canal oficial do MasterSíndico.
   • Sincronização automática de novos episódios.
Acesso (conforme definido na proposta MVP):
   • Disponível para usuários pagantes e planos superiores.

4.6. Painel Administrativo Global (MasterSíndico - Web Only)
Exclusivo Web: Funcionalidades administrativas não estão disponíveis no Mobile.
Funcionalidades:
4.6.1. Dashboard Financeiro:
   • Visão geral de assinaturas ativas/canceladas.
   • Integração com Mercado Pago (ou gateway de pagamento escolhido).
   • Relatórios de receita recorrente (MRR).
4.6.2. Gestão de Usuários:
   •   Listar todos os usuários (com filtros por perfil/plano).
   •   Bloquear/Desbloquear usuário.
   •   Editar dados de qualquer usuário (em casos excepcionais).
   •   Visualizar histórico de atividade.
4.6.3. Moderação de Conteúdo:
   •   Aprovar/Reprovar vídeos denunciados.
   •   Remover vídeos que violem diretrizes.
   •   Suspender funcionalidades de empresas reincidentes.
   •   Moderar posts do Fórum (aprovar/reprovar/remover).
4.6.4. Configurações da Plataforma:
   •   Edição de textos de termos de uso, política de privacidade.
   •   Upload de banners da home.
   •   Configuração de categorias técnicas (adicionar/remover/editar).
   •   Definição de parâmetros globais (ex: tempo de janela de votação padrão, limite
       de vídeos por mês, etc.).
4.6.5. Envio de Comunicados Oficiais (Broadcast Email/SMS):
   • Sistema de envio de comunicados institucionais para todos os usuários ou
     segmentado por perfil/plano.
   • Editor de mensagem (texto + CTA).
   • Agendamento de envio.
   • Log de disparos.
Importante: Este é o único canal oficial de comunicação em massa da plataforma.
4.7. Banco de Talentos (Currículos em Vídeo)
Backend:
   • Upload de Vídeo-Currículo (até 90 segundos).
   • Armazenamento de dados profissionais (nome, área, experiência, contato).
   • Trava Trimestral: Morador só pode atualizar vídeo-currículo a cada 90 dias.
Web & Mobile (Morador Pagante):
  • Aba "Oportunidade":
         • Gravar/Enviar vídeo-currículo (diretamente da câmera ou upload).
         • Preencher ficha cadastral profissional.
         • Vídeo fica fixo no perfil (não é galeria).
Web (Empresas/Síndicos):
  • Visualização da lista de currículos (apenas se o plano permitir - Matriz
      Funcional):
          • Emp. Plus: Pode visualizar currículos.
          • Emp. Pro: Pode visualizar currículos.
          • Marketing: Pode visualizar currículos.
          • Vizinhança: Pode visualizar currículos.
   • Filtros: Área de atuação, região, palavra-chave.
   • Player do vídeo-currículo.
   • Botão "Demonstrar Interesse" (dispara notificação ou email para o morador -
     sujeito às regras de notificação).
Importante: A visualização de currículos é uma funcionalidade de valor para empresas,
estimulando planos superiores.




4. LIMITAÇÕES TÉCNICAS E EXCLUSÕES
Não estão inclusos no escopo deste contrato, sendo considerados itens adicionais
sujeitos a novo orçamento:
4.1. Chat em Tempo Real
O sistema não contempla chat, mensageria instantânea tipo WhatsApp, status
"online/digitando" ou histórico de conversação. A comunicação resume-se a:
   •   Formulários de disparo único (Connect Me).
   •   Sistema de Cupons (Promoção do Dia).
4.2. Videoconferência
O módulo de Assembleias é assíncrono (baseado em vídeos gravados e janelas de
votação). Não inclui salas de reunião ao vivo (tipo Zoom/Google Meet) integradas.

4.3. Prospecção Ativa
Não há ferramentas para empresas extraírem listas de contatos de moradores ou
realizarem disparos em massa (SPAM). O contato parte sempre do interesse do
contratante.

4.4. Edição de Mídia A plataforma realiza o armazenamento e
reprodução de arquivos. Não inclui ferramentas de edição, corte, filtros
ou tratamento de vídeo/imagem dentro do aplicativo.
4.5. Moderação Automática (IA)
A moderação de conteúdo é reativa (baseada em denúncias de usuários) e manual
(executada pelo Administrador MasterSíndico). Não há algoritmos de IA para detecção
automática de conteúdo impróprio.

4.6. Relatórios Personalizados de BI
Os relatórios inclusos são os explicitamente citados (Timeline da Gestão, Certificados,
Dashboard Financeiro básico). Relatórios personalizados de Business Intelligence são
escopo extra.


5. REGRAS DE OURO (LIMITES DO CONTRATO)
5.1. Backend como Verdade Única
Toda regra de negócio reside na API. O Frontend Web e o Mobile são apenas interfaces
de consumo. Se a API diz que o usuário não pode ver X, o Mobile não mostrará X.

5.2. Multimídia
O escopo inclui o armazenamento e reprodução de vídeos e PDFs. Não inclui edição,
corte, filtros ou tratamento de mídia dentro do app.
5.3. Connect Me ≠ Chat
A funcionalidade "Connect Me" é um formulário de contato (Input → Disparo de
Email/SMS). Não salva histórico de conversa na plataforma, não tem reply, não tem
chat.

5.4. Sistema de Cupons ≠ Helpdesk
O termo "Cupom" (ou "Ticket" no código legado) refere-se exclusivamente ao Cupom
de Desconto gerado no padrão numérico definido (ID_LOJA + SEQ + PALAVRA). Não
existe sistema de "Helpdesk/Chamado de Suporte" entre moradores e empresas.

5.5. Trava de Prospecção
Empresas não conseguem listar moradores para enviar mensagens em massa. O
contato parte sempre do interesse do contratante (Morador/Síndico) para o contratado,
via busca.

5.6. Uploads Trimestrais
A regra de 90 dias para vídeos institucionais (Empresas) e vídeo-currículo (Moradores)
é hard-coded (fixa no código). Exceções exigem intervenção manual no banco de
dados ou desenvolvimento extra.

5.7. Lives Restritas ao MasterSíndico
Apenas o Administrador MasterSíndico pode iniciar transmissões. O sistema não é
uma plataforma de videoconferência para assembleias.

5.8. Feedback Privado
Likes e comentários são visíveis apenas ao dono do conteúdo. Não há rankings
públicos, não há competição por engajamento.

5.9. Emails e SMS
O custo de disparo de Email (ex: AWS SES/SendGrid) e SMS (ex: Twilio/Zenvia) é custo
de infraestrutura do CONTRATANTE, não incluso no desenvolvimento. O sistema
apenas integra com a API.

5.10. Notificações Limitadas
O sistema NÃO envia notificações sobre interações sociais. Email/SMS são exclusivos
para: validação de conta, Connect Me e comunicados oficiais do MasterSíndico.
6. MATRIZ FUNCIONAL DE REFERÊNCIA
A Matriz Funcional anexa a este documento define com precisão quais funcionalidades
estão disponíveis para cada perfil de usuário (Base, Morador Pg, Síndico N1/N2/N3,
Emp. Plus/Pro, Marketing, Vizinhança).
Regra Geral: O que não está explicitamente liberado na Matriz Funcional para
determinado perfil, não está disponível para esse perfil.
Exemplos de Aplicação da Matriz:
   • Morador Base: Acesso gratuito a eventos do condomínio (edital, votação,
      avaliação), busca geral, preview de 25% dos vídeos de empresas. Não tem
       acesso a fórum, lives, cursos, currículos, Connect Me.
   •   Morador Pagante: Tudo do Base + Connect Me com Síndicos (4/ano), cadastro
       de vídeo-currículo, perfil visível para empresas.
   • Síndico N1: Capacitação (vídeos, fórum, lives). Sem perfil público, sem
     publicação de vídeos, sem cursos certificados.
   • Síndico N2: Tudo do N1 + perfil público institucional, publicação de vídeos
       (4/mês), Connect Me ativo, cursos certificados.
   •   Síndico N3: Tudo do N2 + ferramentas de gestão completas (link de evento,
       avaliação objetiva, plano de reeleição, exportação PDF).
   •   Empresa Plus: 8 vídeos/mês, participação no fórum/lives, até 100 vídeos
       armazenados, visualização de currículos. Sem Connect Me, sem cursos
       certificados.
   •   Empresa Pro: 12 vídeos/mês, criação de cursos certificados, vídeos
       institucionais, Connect Me ativo, até 150 vídeos armazenados.
   •   Empresa de Marketing: Criação de cursos, vídeos institucionais, Marketing Link
       (tag por vídeo para gerar leads), participação fórum/lives.
   •   Vizinhança (Comércio Local): Cadastro de promoções (palavra do dia),
       visualização de currículos, perfil básico.


7. CRONOGRAMA RESUMIDO
          Duração
 Sprint Estimada                                Entregas Principais
Sprint 3 semanas         Fundação, Identidade, Acesso, Onboarding, Connect Me,
1                        Busca, Player
Sprint 3 semanas         Motor de Gestão Condominial (Assembleias, Votação,
          Duração
 Sprint Estimada                           Entregas Principais
2                     Transparência, Avaliação)
Sprint 3 semanas      Vizinhança e Promoções (Motor de Cupons), Clube de
3                     Benefícios
Sprint 3 semanas      Conteúdo Educacional (LMS), Feedback Privado, Fórum, Lives,
4                     Painel Admin, Currículos
Duração Total Estimada: 12 semanas
Modelo de Pagamento: 20% - 20% - 20% - 40%

8. CONSIDERAÇÕES FINAIS
Este documento constitui o Anexo I do contrato de desenvolvimento da Plataforma
MASTER SÍNDICO e estabelece com clareza e precisão:
   •   O escopo técnico completo.
   •   As funcionalidades detalhadas por sprint.
   •   As limitações e exclusões.
   •   As regras de governança de conteúdo, feedback e notificações.
   •   A matriz funcional de referência.
Regra fundamental: O que não está escrito neste documento não existe no contrato e
será considerado escopo adicional sujeito a novo orçamento.
A arquitetura conceitual e as decisões de produto aqui definidas protegem:
   •   A integridade da plataforma.
   •   A experiência dos usuários.
   •   A reputação de todos os atores do ecossistema condominial.
   •   A viabilidade técnica e comercial do projeto.


Data de Elaboração: Janeiro de 2026
Versão: 3.0 (Revisada e Consolidada)
