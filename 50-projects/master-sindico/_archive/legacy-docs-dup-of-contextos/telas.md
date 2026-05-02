
# Jornada do Morador

**Nome do botão:** Painel do Morador

### TELA 1 – SPLASH

Exibição institucional.

**Texto:** Master Síndico
Governança condominial estruturada.

**Botão:** Entrar

**Rodapé:** Ao acessar, você concorda com os Termos de Uso e Política de Privacidade.

### TELA 2 – SELECIONAR CONDOMÍNIO

**Título:** Selecione o condomínio que deseja acessar
Lista de condomínios vinculados:
Card contendo: Nome do condomínio
- Endereço resumido
- Unidade
- Tipo da unidade (Residencial ou Comercial)
- Vínculo (Proprietário / Inquilino / Morador autorizado)
- Papel (Titular / Dependente)

**Botões:** Acessar
Botões gerais: Adicionar novo condomínio
- Gerenciar meus acessos
Explicação técnica: Permitir múltiplas unidades e múltiplos condomínios por usuário.

### TELA 3 – BUSCAR CONDOMÍNIO POR ID

**Título:** Adicionar acesso a condomínio
Campo obrigatório: ID do condomínio
Após inserção: Sistema deve exibir: Nome do condomínio
- Endereço completo
- CEP

**Mensagem:** Confirme se este é o condomínio correto antes de prosseguir.

**Botões:** Confirmar
- Voltar

**Justificativa:** Confirmação visual reduz risco de erro e fraude.

### TELA 4 – IDENTIFICAÇÃO DA UNIDADE

**Título:** Informe os dados da sua unidade

**Campos obrigatórios:** 
Tipo da unidade: Residencial
- Comercial
- Número da unidade
- Bloco/Torre (opcional)
Seu vínculo: Proprietário
- Inquilino
- Morador autorizado
Seu papel: Titular
- Dependente

**Regras exibidas:** 
1 titular por unidade.
Até 2 dependentes por unidade.
Unidade vazia não permite dependentes.
Se selecionar Titular: Mensagem: Caso exista inquilino ou outro morador, ele deverá ser cadastrado como dependente.

**Botões:** Avançar
- Voltar

### TELA 5 – TERMO DE CIÊNCIA E RESPONSABILIDADE

**Título:** Declaração formal de vínculo

**Texto completo:** 
Declaro, sob minha responsabilidade civil e penal, que possuo vínculo legítimo com a unidade informada neste cadastro, na condição declarada. Confirmo que as informações prestadas são verdadeiras, completas e atualizadas.
Estou ciente de que a prestação de informações falsas poderá resultar na suspensão ou cancelamento do meu acesso à plataforma, bloqueio da conta e adoção das medidas legais cabíveis.
Declaro estar ciente de que o acesso às informações do condomínio é condicionado à manutenção do vínculo legítimo com a unidade e que alterações no status da unidade poderão impactar meu acesso.
Autorizo o tratamento dos meus dados pessoais para fins de controle de acesso, rastreabilidade e segurança, nos termos da Lei Geral de Proteção de Dados.
Checkbox obrigatório: Declaro que li e concordo.

**Botões:** Confirmar e solicitar acesso
- Voltar

### TELA 6 – STATUS DA UNIDADE

**Título:** Situação atual da unidade
Campo obrigatório: Status da unidade
Lista definitiva:
- Ativa – Ocupada pelo proprietário
- Ativa – Ocupada por inquilino
- Unidade vazia
- Alugada
- Vendida
- Encerrado por solicitação
Regras automáticas:
Unidade vazia: Remove dependentes. Bloqueia novos dependentes. Mantém apenas titular.
Alugada: Permite titular proprietário. Permite dependente (inquilino).
Vendida: Remove titular e dependentes.
Encerrado: Remove todos os acessos.

**Botão:** Salvar e continuar

### TELA 7 – AVALIAÇÃO OBRIGATÓRIA

**Título:** Avaliação da gestão

**Mensagem:** Sua avaliação contribui para a melhoria contínua do condomínio.
Escala de 1 a 10:
- Comunicação da gestão
- Organização das ações
- Confiança na gestão

**Botão:** Enviar e continuar

**Regra:** Bloqueia acesso até envio.

### TELA 8 – PAINEL DO MORADOR

Topo fixo: Nome do condomínio
- ID do condomínio
- Ícone QR Code

**Botão:** Trocar condomínio

**Mensagem inicial:** Bem-vindo ao Painel do Morador.
Aqui você acompanha as ações, participa das decisões e acessa as informações oficiais do seu condomínio.
Cards principais:
- Linha do Tempo
- Plano Diretor
- Eventos
- Votação de Fornecedor

### TELA 9 – LINHA DO TEMPO

Estrutura cronológica por cards.
Cada card contém:
- Tipo da atividade
- Título
- Data
- Nível de importância
- Status
- Empresa envolvida

**Botão:** Ver detalhes
Detalhes exibidos:
- Descrição
- Fotos
- Vídeos
- Anexos públicos
- Impacto esperado (se autorizado)
- Próxima ação (se autorizado)
Somente conteúdo liberado pelo síndico.

### TELA 10 – PLANO DIRETOR (VISÃO DO MORADOR)

**Título:** Plano Diretor da Gestão
Lista estruturada:
- Título da ação
- Status
- Prazo de conclusão (se autorizado)
Não exibir: Observações internas
- Responsável
- Notas estratégicas

### TELA 11 – EVENTOS

Lista de eventos:
- Tipo
- Título
- Período
- Status
- Taxa de visualização
Ao abrir evento:
- Descrição
- Data início
- Data fim
- Local
- Anexos

**Botões:** 
- Ciente
- Confirmo participação
Após Ciente: Registro de ciência realizado.
Após Confirmo participação: Participação confirmada. Pode alterar até o prazo final.
Notificação enviada ao síndico.

### TELA 12 – VOTAÇÃO DE FORNECEDOR

**Título:** Escolha da proposta

**Exibe:** 
- Descrição da contratação
- Critérios definidos pelo síndico
- Prazo de votação
Para cada fornecedor:
- Nome
- Status (Verificada / Não verificada)
- Resumo técnico
- Valor total
- Condições de pagamento
- Prazo de início
- Prazo de conclusão
- Garantia

**Botões:** Ver proposta completa
- Ver vídeo
- Votar nesta proposta
Mensagem após voto:
Seu voto foi registrado com sucesso. O resultado será consolidado ao término do período definido pelo síndico.

### TELA 13 – GERENCIAR ACESSOS

Lista de unidades vinculadas.

**Opções:** 
- Atualizar status
- Encerrar acesso
- Editar vínculo

**Mensagem:** Mantenha seus dados atualizados para garantir acesso legítimo.
- NOTIFICAÇÕES AUTOMÁTICAS
- Nova votação aberta
- Prazo de votação encerrando
- Novo evento publicado
- Nova atividade relevante
- Alteração de status
- Encerramento de acesso

# Jornada do Síndico

**Nome do botão:** Governança
Entrada: Botão GOVERNANÇA
Área: Central de Governança

**Objetivo:** Organização profissional + Blindagem institucional
- REGRAS GERAIS (VALEM PARA TODA A JORNADA)
R1. Confirmação de contexto (anti-erro): antes de publicar qualquer conteúdo (linha do tempo, evento, votação, comunicado), o app exige confirmação: “Publicar para Condomínio X?”
R2. Voz em todas descrições: todos os campos de descrição aceitam comando de voz e texto.
R3. Nada é deletado: ações do Plano Diretor e registros formais não são excluídos; apenas editar ou ocultar (com trilha).
R4. Origem externa sempre pendente: tudo que vier de empresa (proposta, registro de serviço, comunicado) entra como Aguardando validação.
R5. Pós-fechamento é imutável: após encerramento de votação / confirmação de negócio fechado / encerramento de evento, dados estruturais ficam bloqueados; alteração só por Adendo Formal.
R6. Continuidade institucional: novo síndico herda histórico; antigo mantém leitura e exportação (provas).
R7. Compliance não trava onboarding: condomínio pode ser criado e operado; compliance pode ficar “Pendente” até gatilhos (ver módulo).
R8. Assinatura obrigatória: compliance só fica “Em dia” quando todos os usuários cadastrados assinarem.
R9. Notificações internas: toda ação importante gera notificação no app (exceto “novo morador solicitando acesso”, que você removeu).
R10. LGPD e rastreabilidade: compartilhamento de dados é registrado com finalidade, timestamp, usuário e consentimento.

**Justificativa para devs:** regras reduzem erro humano, garantem prova documental e habilitam auditoria/BI.

## BLOCO 1 — ENTRADA E SELETOR DE CONDOMÍNIO

### TELA 1 — GOVERNANÇA (Entrada)

Mensagem principal:
Governança é responsabilidade, organização e prova documental. Aqui você registra sua gestão com rastreabilidade formal.

**Botões:** 
- Selecionar condomínio
- Cadastrar novo condomínio
Rodapé jurídico (fixo):
Ao acessar esta área você declara ciência de que os registros realizados possuem caráter formal e poderão compor histórico documental da gestão condominial, assumindo responsabilidade pelas informações inseridas, nos termos dos Termos de Uso e da legislação aplicável.

**Links:** Termos de Uso | Política de Privacidade
- ---

### TELA 2 — SELETOR DE CONDOMÍNIO

Abas: Ativos | Encerrados
Card (Ativo):
- Nome do condomínio
- Endereço completo
- Tipo (Residencial/Comercial/Misto/Shopping/Galeria/Edifício Garagem)
Mandato: Eleito / Contratado / Interino (com datas)
Indicadores rápidos: Pendências de validação + Próximas ações (contador)
Botões do card:
- Acessar
- Editar dados
Card (Encerrado):
- Data de encerramento + motivo
Acesso: Leitura

**Botão:** Acessar histórico
Botões gerais:
- Cadastrar novo condomínio
- Encerrar mandato (para condomínio ativo, com permissão)

**Justificativa devs:** síndico pode gerir múltiplos condomínios sem risco de publicar no local errado.
- ---

## BLOCO 2 — CADASTRO DE CONDOMÍNIO (ROBUSTO)

### TELA 3 — CADASTRAR CONDOMÍNIO (Etapa 1: Endereço)

**Campos:** 
- CEP (busca)
- Logradouro (auto)
- Número
- Complemento (opcional)
- Bairro (auto)
- Cidade (auto)
- UF (auto)
Campos do condomínio:
Tipo de condomínio: Residencial | Comercial | Misto | Shopping Center | Galeria Comercial | Edifício Garagem
- Quantidade total de unidades (obrigatório)
Separação (se misto): unidades residenciais | unidades comerciais (opcional, mas recomendado)
Upload: Foto da fachada (opcional)

**Botões:** Avançar | Voltar
- ---

### TELA 4 — CADASTRAR CONDOMÍNIO (Etapa 2: Mandato)

**Campos:** 
Tipo de mandato: Eleito | Contratado | Interino
- Data início
- Data término (validade do mandato)
- Observações (voz/texto)

**Botões:** Avançar | Voltar
- ---

### TELA 5 — IDENTIDADE DO CONDOMÍNIO (Etapa 3: ID/QR/Código de barras)

Sistema gera automaticamente:
- ID alfanumérico seguro (sequência difícil de decifrar)
- QR Code
- Código de barras
Mensagens:
Este é o identificador oficial do seu condomínio na Master Síndico. Use para convidar moradores com segurança.

**Botões:** 
- Copiar ID
- Compartilhar convite
- Concluir e acessar Central
Mensagem automática para compartilhar (editável, mas com modelo padrão):
Acompanhe tudo o que está sendo realizado no nosso condomínio com transparência e organização. Baixe o app Master Síndico e use o ID: [ID]. Sua participação fortalece a gestão.

**Justificativa devs:** ID/QR/código de barras aumenta adesão e reduz erro de digitação.
- ---

### TELA 6 — CONFIRMAÇÃO DE CONTEXTO (anti-erro global)

Sempre que o síndico for publicar algo, exibir modal:
Você está prestes a publicar para:
Condomínio: [Nome]
Endereço: [Resumo]

**Botões:** Confirmar | Trocar condomínio | Cancelar

**Justificativa devs:** evita erro crítico em síndico multi-carteira.
- ---

## BLOCO 3 — CENTRAL DE GOVERNANÇA (HOME COMPLETA)

### TELA 7 — CENTRAL DE GOVERNANÇA (Home do condomínio)

Topo fixo (sempre visível):
- Nome do condomínio
- ID + botão copiar
- QR Code + botão ampliar
- Código de barras + botão ampliar

**Botão:** Alterar condomínio
Indicadores (painel):
Plano Diretor: total ações | ações vencendo | ações atrasadas
Natureza: % preventiva | % corretiva | % emergencial | % estratégica
Risco: quantidade alto/crítico em aberto
- Validações pendentes (contador)
- Média de avaliação dos moradores (1–10)
- Avaliações institucionais pendentes (contador)
Mandato: data final + alerta (se próximo)
- Score de Governança (0–100, privado)
- Score de Compliance (0–100, privado)
- Taxa de adesão de moradores (titular/dependente por unidade)
Cards (acessos):
- Plano Diretor
- Linha do Tempo
- Eventos
- Votação de Fornecedor
- Serviços e Propostas
- Connect Me
- Validações Pendentes
- Avaliações
- Indicadores
- Usuários
- Compliance
- Relatório de Gestão
- Modo Assembleia
- Cadastrar Parceiro da Vizinhança

**Justificativa devs:** home deve expor riscos, pendências e próximos passos (produto “proteção + organização”).
- ---

## BLOCO 4 — USUÁRIOS E PERMISSÕES (até 3 usuários)

### TELA 8 — USUÁRIOS (Gestão de acesso)

**Mensagem:** 
Não compartilhe sua senha. Cada usuário deve ter seu próprio cadastro para manter rastreabilidade e responsabilidades.
Estrutura:
Usuário 1: Síndico Principal (fixo)
Usuário 2: Convidar (opcional)
Usuário 3: Convidar (opcional)
Perfis possíveis no convite:
- Síndico Preposto
- Administradora
- Conselho (leitura ampliada)

**Botões:** 
- Convidar usuário
- Gerenciar permissões
- Histórico de acessos
- ---

### TELA 9 — CONVIDAR USUÁRIO

**Campos:** Nome | E-mail | Telefone | Perfil | Mensagem (voz/texto, opcional)

**Botões:** Enviar convite | Voltar
- ---

### TELA 10 — ACEITE DO CONVIDADO (co-responsabilidade)

**Texto:** 
Você foi convidado para atuar na governança do condomínio. Suas ações serão registradas e poderão compor histórico documental.
Termo de ciência do usuário (obrigatório):
Declaro ciência de que minha atuação nesta plataforma é rastreável e deve refletir informações verdadeiras e diligentes, sob pena de responsabilização na forma aplicável.

**Checkbox:** Li e concordo

**Botões:** Aceitar | Recusar
- ---

### TELA 11 — MATRIZ DE PERMISSÕES (toggles)

Permissões configuráveis (por usuário convidado):
- Criar/editar Plano Diretor
- Registrar atividade na Linha do Tempo
- Tornar registro público aos moradores
- Criar evento
- Encerrar evento
- Criar votação
- Encerrar votação
- Cadastrar proposta
- Gerar link externo (proposta/serviço/comunicado)
- Validar proposta
- Validar registro de serviço
- Validar comunicado pós-fechamento
- Marcar negócio fechado
- Confirmar negócio fechado
- Criar adendo formal
- Exportar relatório
- Acessar compliance
- Assinar compliance

**Regra:** usuários podem encerrar votação e confirmar negócio fechado se tiverem permissão.

**Botões:** Salvar | Voltar

**Justificativa devs:** governança real não pode depender de uma única pessoa; trilha de auditoria preserva segurança.
- ---

## BLOCO 5 — LISTAS PADRONIZADAS (seleções fechadas)

Estas listas devem ser usadas em: Plano Diretor, Linha do Tempo, Eventos, Votações, Decisões sensíveis, Relatórios.
- 5.1 Tipo de atividade (seleção)
Administrativo; Financeiro; Departamento Pessoal; Jurídico; Manutenção; Obras; Sanitário; Segurança; Sustentabilidade; Tecnologia; Comunicação; Outro
- (Em cada macro, permitir subtipos conforme mapeamento completo que você aprovou.)
- 5.2 Áreas impactadas (múltipla)
Estrutural; Elétrica; Hidráulica; Sanitária; Segurança; Financeira; Jurídica; Trabalhista; Administrativa; Tecnologia; Ambiental; Patrimonial; Convivência; Reputacional; Operacional; Outros
- 5.3 Natureza da ação (obrigatória)
- Preventiva; Corretiva; Emergencial; Estratégica; Obrigação legal; Deliberação assemblear
- 5.4 Classificação de risco
Categoria (múltipla): Operacional; Financeiro; Jurídico; Estrutural; Reputacional; Sanitário; Trabalhista; Tecnologia/Segurança da Informação
Nível (única): Baixo; Médio; Alto; Crítico
- 5.5 Nível de importância
- Baixa; Moderada; Alta; Crítica; Estratégica
- 5.6 Impacto esperado (múltipla)
Redução de risco jurídico; Redução de custo; Melhoria estrutural; Melhoria operacional; Aumento de segurança; Valorização patrimonial; Conformidade legal; Melhoria na convivência; Eficiência administrativa; Transparência institucional; Mitigação de passivo trabalhista
- 5.7 Próxima ação (múltipla)
Aguardar orçamento; Aguardar assembleia; Aguardar parecer técnico; Executar próxima etapa; Solicitar laudo; Negociar contrato; Realizar vistoria; Encerrar ação; Reavaliar estratégia; Formalizar contrato; Comunicar moradores; Arquivar documentação
- 5.8 Status (única)
Planejada; Em andamento; Parcialmente concluída; Concluída; Suspensa; Cancelada; Atrasada; Em análise; Aguardando aprovação; Aguardando proposta; Aguardando validação

**Justificativa devs:** listas fechadas garantem dados estruturados, relatórios consistentes e futura IA/BI.
- ---

## BLOCO 6 — PLANO DIRETOR

### TELA 12 — LISTA DO PLANO DIRETOR

**Filtros:** por status, natureza, risco, data limite, área impactada

**Botão:** Nova ação

### TELA 13 — NOVA AÇÃO (Plano Diretor)

**Campos:** Título
- Descrição (voz/texto)
- Tipo de atividade
- Áreas impactadas
- Natureza (obrigatória)
- Importância
- Risco (categoria + nível)
- Impacto esperado
- Próxima ação
- Data início
- Data limite
- Responsável (opcional)
- Observações internas (opcional)

**Botões:** Salvar | Voltar

**Regras:** não excluir; apenas ocultar/editar; trilha registra versões.
- ---

## BLOCO 7 — LINHA DO TEMPO

### TELA 14 — LINHA DO TEMPO (lista)

**Filtros:** período, status, natureza, risco, público/gestão

**Botão:** Novo registro

### TELA 15 — NOVO REGISTRO (Linha do tempo)

**Campos:** Tipo de atividade
- Título
- Descrição (voz/texto)
- Fotos (opcional)
- Vídeos (opcional)
- Anexos (opcional)
- Áreas impactadas
- Natureza
- Importância
- Risco (categoria + nível)
- Status
- Impacto esperado
- Próxima ação

**Visibilidade:** Gestão | Público aos moradores

**Botões:** Publicar | Voltar
Antes de publicar: modal “Condomínio X?” (Regra R1)
- ---

## BLOCO 8 — SERVIÇOS E REGISTROS POR FORNECEDOR (interno/externo + validação)

### TELA 16 — SERVIÇOS (Hub)

**Cards:** Solicitar registro de serviço
- Registros recebidos (pendentes)
- Registros validados
- Negócios fechados (atalho)

### TELA 17 — SOLICITAR REGISTRO DE SERVIÇO

Opção A: Fornecedor da plataforma
- Buscar empresa / selecionar conectadas

**Botão:** Solicitar registro
Opção B: Fornecedor externo (link)

**Botão:** Gerar link externo de registro

**Mensagem:** 
Se a empresa não estiver cadastrada, ela poderá registrar o serviço como “não verificada” e será convidada a se cadastrar.

**Botões:** Gerar | Voltar

### TELA 18 — LINK EXTERNO (empresa não verificada)

Exibe link + botão copiar/compartilhar.
O fornecedor externo deve preencher (lead obrigatório):
- Razão social (ou nome)
- CNPJ (obrigatório para externo)
- Nome do responsável
- E-mail
- Telefone
- Categoria do serviço
- Descrição do serviço executado (voz/texto)
- Data de execução
- Evidências (fotos/vídeo/anexo)
- Garantia (se aplicável)
- Observações
Status público: Empresa não verificada (badge cinza)

### TELA 19 — VALIDAÇÃO DO SÍNDICO (registro de serviço)

Botões em ordem:
- 1. Avaliar (atalho para avaliação institucional pós-serviço, se aplicável)
- 2. Validar
- 3. Solicitar ajuste
- 4. Rejeitar (justificativa obrigatória)

**Justificativa devs:** validação mantém integridade do histórico e evita poluição/abuso.
- ---

## BLOCO 9 — EVENTOS (mais tipos + métricas)

### TELA 20 — EVENTOS (lista + métricas)

Indicadores por evento: Visualizações
- “Ciente”
- “Confirmo participação”

**Botão:** Criar evento

### TELA 21 — CRIAR EVENTO

Tipos (ampliado): Assembleia Ordinária; Assembleia Extraordinária; Reunião de Conselho; Prestação de Contas; Audiência Jurídica; Enquete Consultiva; Votação Interna; Comunicado Operacional; Comunicado Emergencial; Aviso de Manutenção; Apresentação de Projeto; Reunião com Fornecedor; Treinamento; Simulado de Emergência; Interdição Temporária; Mudança de Procedimento; Campanha Institucional; Atualização Normativa; Outro

**Campos:** Título
- Descrição (voz/texto)
- Data início/fim
- Local (opcional)
- Anexos (opcional)
- Visibilidade (Gestão/Público)

**Botões:** Publicar | Voltar
- Modal “Condomínio X?” antes de publicar
- ---

## BLOCO 10 — VOTAÇÃO DE FORNECEDOR (módulo completo)

### TELA 22 — VOTAÇÕES (hub)

**Cards:** Criar votação
- Propostas recebidas (pendentes)
- Votações em andamento
- Resultados
- Negócios fechados

### TELA 23 — CRIAR VOTAÇÃO

**Campos:** Título do serviço/compra
- Descrição (voz/texto)
- Critérios de escolha (múltipla seleção – lista completa abaixo)
- Prazo para envio de propostas (definido pelo síndico)
- Prazo de votação dos moradores (definido pelo síndico)
- Permitir renovar/editar prazos (sim/não)

**Botões:** Criar | Voltar
- Modal “Condomínio X?” antes de publicar
- Lista completa de critérios do síndico (seleção múltipla)
Menor valor global; Melhor custo-benefício; Melhor condição de pagamento; Menor prazo de execução; Garantia mais extensa; Certificações técnicas; Seguro de responsabilidade civil; Experiência comprovada; Histórico na plataforma; Avaliação institucional; Capacidade operacional; Equipe técnica especializada; Atendimento emergencial; Conformidade legal e fiscal; Sustentabilidade; Inovação técnica; Prazo de mobilização; Clareza do escopo; Transparência documental; Condições de manutenção pós-serviço; Referências anteriores; Outro (campo aberto)

### TELA 24 — PROPOSTAS (hub)

**Opções:** Cadastrar proposta (síndico)
- Solicitar proposta a empresa da plataforma
- Gerar link externo para proposta

### TELA 25 — CADASTRAR PROPOSTA (síndico)

**Campos obrigatórios:** Descrição técnica detalhada
- Valor total
- Condições de pagamento
- Prazo para iniciar
- Prazo para concluir
- Garantia
- Escopo delimitado
- Responsável técnico (quando aplicável)
- Anexos (opcional)
- Vídeo (opcional)

**Status:** Pronta para validação/publicação

**Botões:** Salvar | Voltar

### TELA 26 — PROPOSTA POR FORNECEDOR (interno ou externo)

Se fornecedor preenche: Status: Aguardando validação do síndico
Badge: verificada / não verificada

### TELA 27 — VALIDAÇÃO DA PROPOSTA (síndico)

**Botões:** Validar (entra na votação)
- Solicitar ajuste
- Rejeitar (justificativa obrigatória)

**Regra:** proposta só aparece aos moradores após validação.

### TELA 28 — ACOMPANHAMENTO DA VOTAÇÃO (síndico)

**Exibe:** Propostas validadas
- Prazo restante para propostas
- Prazo restante de votação
- Participação (quantos moradores votaram)

**Botões:** Editar/renovar prazo (propostas)
- Editar/renovar prazo (votação)

### TELA 29 — RESULTADO PARCIAL E FINAL (síndico)

**Exibe:** Parcial durante votação
- Final após encerramento
- Total de votos
- Percentuais
- Log de encerramento

**Regra:** após encerramento, imutável.

### TELA 30 — VISÃO DO RESULTADO PARA EMPRESA

A empresa vê: Seu resultado e status (selecionada ou não)
Não vê concorrência.

**Justificativa devs:** reduz atrito concorrencial e protege marketplace.
- ---

## BLOCO 11 — NEGÓCIO FECHADO (com termos robustos + dupla confirmação + bloqueio)

### TELA 31 — MARCAR NEGÓCIO FECHADO

Pode ser marcado por: Síndico (permitido)
- Empresa (permitido)
- Usuários com permissão (permitido)

**Campos:** Valor final
- Condições de pagamento (estruturadas)
- Prazo para iniciar
- Prazo para concluir
- Garantia confirmada
- Observações (voz/texto)
- Anexos (contrato/OS/ata – opcional, mas recomendado)

**Botões:** Avançar | Voltar

### TELA 32 — TERMO DE EXECUÇÃO DA EMPRESA (robusto)

Texto (empresa confirma): Declaro que executarei o serviço conforme escopo e condições aprovadas, observando normas técnicas e legislação aplicável, mantendo regularidade fiscal e trabalhista durante a execução. Assumo responsabilidade por vícios de execução e danos comprovadamente decorrentes da minha atuação, bem como pelo cumprimento dos prazos e garantias registradas, salvo hipóteses fortuitas justificadas e formalizadas por adendo.

**Checkbox:** Li e confirmo

**Botões:** Avançar | Voltar

### TELA 33 — TERMO DE CIÊNCIA DO SÍNDICO (robusto)

Texto (síndico confirma): Declaro ciência integral das condições técnicas e comerciais registradas, comprometendo-me a cumprir as obrigações financeiras pactuadas e a garantir condições de acesso e execução. Declaro que alterações posteriores somente terão validade mediante adendo formal registrado na plataforma. Declaro que registrei esta contratação para fins de rastreabilidade e transparência da gestão.

**Checkbox:** Li e confirmo

**Botões:** Avançar | Voltar

### TELA 34 — CONFIRMAÇÃO DUPLA (final)

Texto fixo: Confirmo que as condições registradas refletem integralmente o acordo firmado entre as partes.
- Checkbox obrigatório

**Registro:** data/hora/IP/usuário

**Botões:** Confirmar negócio fechado | Cancelar
Regra imutabilidade: após confirmação, bloqueio total; apenas adendo.
- ---

## BLOCO 12 — COMUNICAÇÃO PÓS-FECHAMENTO (empresa cria, síndico valida)

### TELA 35 — COMUNICADOS PÓS-FECHAMENTO (hub)

Tipos sugeridos (seleção): Aviso de início de serviço
- Comunicado de intervenção/obra
- Circular informativa
- Orientação ao morador
- Comunicado de segurança
- Comunicado de interdição temporária
- Comunicado de conclusão
- Outro
Campos padrão: Título
- Descrição (voz/texto)
- Data início
- Data término
- Anexos (opcional)

**Visibilidade:** Público (moradores) ou Gestão

**Regra:** empresa pode criar, mas fica Aguardando validação do síndico.

### TELA 36 — VALIDAÇÃO DO COMUNICADO (síndico)

**Botões:** Validar e publicar
- Solicitar ajuste
- Rejeitar (justificativa obrigatória)
- Modal “Condomínio X?” antes de publicar
- ---

## BLOCO 13 — AVALIAÇÕES (moradores + institucional de fornecedores)

### TELA 37 — AVALIAÇÕES (hub)

**Cards:** Média geral moradores (1–10)
- Avaliações obrigatórias (status agregado)
- Avaliar fornecedor (apenas com negócio fechado)
- Avaliações enviadas aos fornecedores (histórico)

### TELA 38 — AVALIAÇÃO INSTITUCIONAL DO FORNECEDOR (pós-serviço)

Escala 1–10: Qualidade técnica
- Cumprimento de prazo
- Comunicação
- Organização documental
- Confiabilidade
Campo: Observações (voz/texto)

**Botões:** Enviar | Voltar

**Regra:** empresa recebe imediatamente e pode:
- autorizar visibilidade pública ou manter privada
- responder (agradecer/justificar/desculpar)

**Justificativa devs:** feedback fecha ciclo de qualidade sem “expor” automaticamente.
- ---

## BLOCO 14 — VALIDAÇÕES PENDENTES (fila única)

### TELA 39 — VALIDAÇÕES PENDENTES

Fila contendo: Propostas
- Registros de serviço
- Comunicados
Cada item mostra: Origem (empresa verificada / não verificada)
- Data
- Prazo (se houver)
- Ação recomendada

**Botões:** Validar
- Solicitar ajuste
- Rejeitar (justificativa obrigatória)
- ---

## BLOCO 15 — CONNECT ME (síndico → empresa) com LGPD robusta

### TELA 40 — CONNECT ME (hub)

**Cards:** Nova solicitação
- Minhas solicitações enviadas
- Solicitações recebidas (quando aplicável ao síndico)

### TELA 41 — NOVA SOLICITAÇÃO CONNECT ME

**Campos obrigatórios:** Categoria do serviço
- Descrição do serviço (voz/texto)
- Urgência (baixa/média/alta)
- Prazo desejado
- Orçamento estimado (opcional)
- Bairro
- Cidade
- Observações (opcional)
Texto LGPD (robusto): Ao enviar, autorizo o compartilhamento do meu nome, telefone e e-mail com a empresa selecionada, exclusivamente para fins de contato e negociação do serviço descrito, nos termos da Lei nº 13.709/2018 (LGPD). Estou ciente de que a empresa só visualizará meus dados após indicar interesse.

**Checkbox:** Li e autorizo

**Botões:** Enviar | Voltar

### TELA 42 — INTERESSE DA EMPRESA (gatilho)

Quando a empresa marcar “Tenho interesse”, liberar contato do síndico para ela.

**Registro:** empresa + data/hora + finalidade + dados liberados

**Justificativa devs:** mantém valor do Connect Me, cria rastreabilidade LGPD e evita vazamento.
- ---

## BLOCO 16 — PARCEIRO DA VIZINHANÇA (lead estruturado)

### TELA 43 — CADASTRAR PARCEIRO DA VIZINHANÇA (hub)

**Opções:** Cadastrar parceiro (dados básicos pelo síndico)
- Gerar link para parceiro se cadastrar (lead)

**Mensagem:** 
Convide empresas locais para integrarem sua rede. Empresas externas entram como “não verificadas” e podem se cadastrar.

### TELA 44 — FORMULÁRIO DO SÍNDICO (dados mínimos do parceiro)

Campos (para devs): Nome fantasia
- Categoria
- Bairro/cidade
- Contato (nome, telefone, e-mail)
- Observação (voz/texto)

**Botões:** Salvar | Gerar link | Voltar
- ---

## BLOCO 17 — INDICADORES (as telas que você pediu)

### TELA 45 — INDICADORES (hub)

**Cards:** Taxa de visualização dos eventos
- Taxa de participação (ciente/confirmo)
- Taxa de moradores na plataforma
- Titulares vs dependentes por unidade
Votações: participação e evolução
- Pendências por tipo (proposta/serviço/comunicado)

### TELA 46 — EVENTOS: taxa de visualização/abertura

Lista por evento com: Visualizações
- Ciente
- Confirmo participação
- Exportar resumo

### TELA 47 — ADESÃO DA PLATAFORMA (moradores)

**Exibe:** Total de unidades
- Unidades com titular cadastrado
- Dependentes cadastrados
- Distribuição titular/dependente
- Evolução mensal (opcional)

**Justificativa devs:** síndico precisa provar engajamento e ter controle de comunicação/participação.
- ---

## BLOCO 18 — COMPLIANCE (botão separado + assinaturas obrigatórias + auditoria + bloqueios)

### TELA 48 — COMPLIANCE (entrada)

**Mensagem:** Ativar compliance fortalece sua blindagem documental sem travar sua operação. Você pode concluir agora ou depois, mas alguns marcos exigirão compliance em dia.

**Status:** Em dia / Pendente (faltam assinaturas) / Pendente (declaração anual)

**Botões:** Ver pendências
- Assinar/Coletar assinaturas
- Auditoria
- Adendos e imutabilidade
- Mapa de riscos
- Conformidade contratual
- LGPD (registro de compartilhamentos)
- Exportar dossiê
Gatilhos recomendados (para devs):
- Obrigatório antes de “Exportar relatório final”
- Obrigatório antes de “Negócio fechado” (opcional, se você quiser endurecer)
- Ou obrigatório após X dias do cadastro do condomínio

### TELA 49 — DECLARAÇÃO ANUAL (síndico principal)

- Texto robusto (já aprovado na versão anterior)
- Checkbox + Confirmar

**Registro:** IP/timestamp/usuário

### TELA 50 — ASSINATURA OBRIGATÓRIA DE TODOS OS USUÁRIOS

Lista de usuários: Principal + convidados (até 3)
Status por usuário: Assinado / Pendente
Ações: Solicitar assinatura (notificação interna)
- Lembrete

**Regra:** compliance só vira “Em dia” quando TODOS assinarem.

### TELA 51 — PUBLICAÇÃO AUTOMÁTICA NA LINHA DO TEMPO (institucional)

Toda assinatura/declaração gera item automático: Tipo: Administrativo → Conformidade & Compliance

**Visibilidade:** Gestão (padrão)
- Não editável

### TELA 52 — TRILHA DE AUDITORIA VISÍVEL

Mostra: usuário, perfil, ação, campo, versão anterior, versão atual, data/hora
- Imutável

### TELA 53 — IMUTABILIDADE + ADENDO FORMAL

- Lista de registros bloqueados e botão “Criar adendo” Adendo exige justificativa + anexos opcionais
- Gera nova camada sem alterar original

### TELA 54 — MAPA DE RISCO (mandato)

- Exibe distribuição por categoria e nível
- Lista ações críticas (alto/crítico)

### TELA 55 — CONFORMIDADE CONTRATUAL

Lista negócios fechados e status documental: Contrato anexado / OS / Proposta / Justificado

### TELA 56 — LGPD: registro de compartilhamentos

Mostra histórico Connect Me: empresa, data, finalidade, dados liberados

### TELA 57 — SCORE interno (governança e compliance)

- Exibe score e recomendações internas (privado)
- ---

## BLOCO 19 — RELATÓRIO E MODO ASSEMBLEIA

### TELA 58 — RELATÓRIO DE GESTÃO (gerar PDF)

Composição: Mandato
- Resumo executivo
- Plano Diretor completo + status
- Natureza (% prev/corr/emerg/estrat)
- Mapa de risco
- Linha do tempo consolidada
- Eventos + métricas
- Votações + resultados
- Contratações + termos + pagamentos/prazos
- Validações e adendos
- Avaliações (moradores e fornecedores)
- Compliance (declarações e assinaturas)
- Indicadores de adesão

**Botão:** Gerar PDF | Voltar

### TELA 59 — MODO ASSEMBLEIA (apresentação)

Exibe em modo “slides”: Indicadores principais
- Ações concluídas e pendentes
- Riscos críticos
- Eventos e participação
- Votações e decisões
- Contratações e prazos
- Próximas ações

**Botões:** Iniciar apresentação
- Exportar resumo
- ---

## BLOCO 20 — ENCERRAMENTO DE MANDATO (com motivo)

### TELA 60 — ENCERRAR MANDATO

Motivo (obrigatório): Término natural; Não reeleito; Renúncia; Destituição; Rescisão contratual; Transferência; Intervenção judicial; Incapacidade temporária; Outro (descrever)
- Data efetiva
- Observações (voz/texto)

**Botão:** Encerrar mandato | Cancelar
Regra pós-encerramento: antigo fica leitura; novo herda histórico.

## MÓDULO COMPLIANCE

- Botão dedicado na Central de Governança

**Objetivo:** 
Blindagem documental, rastreabilidade e mitigação de risco jurídico.
O módulo NÃO bloqueia o cadastro inicial do condomínio.
Mas determinados marcos exigem compliance ativo.
- ESTRUTURA DO MÓDULO

### TELA C1 — PAINEL DE COMPLIANCE

### TELA C2 — DECLARAÇÃO ANUAL DO SÍNDICO

### TELA C3 — ASSINATURA DOS USUÁRIOS (obrigatória)

### TELA C4 — MATRIZ DE CONFLITO DE INTERESSE

### TELA C5 — TRILHA DE AUDITORIA VISÍVEL

### TELA C6 — IMUTABILIDADE E ADENDOS

### TELA C7 — MAPA DE RISCO CONSOLIDADO

### TELA C8 — CONFORMIDADE CONTRATUAL

### TELA C9 — REGISTRO LGPD

### TELA C10 — SCORE INTERNO DE GOVERNANÇA

### TELA C11 — DOSSIÊ EXPORTÁVEL

### TELA C1 — PAINEL DE COMPLIANCE

Mensagem de abertura:
Compliance é a camada de proteção da sua gestão.
Aqui ficam registradas declarações formais, responsabilidades assumidas, trilhas de auditoria e evidências institucionais.

**Indicadores:** 
Status geral:
- Em dia
- Parcial
- Pendente
Pendências:
- Declaração anual
- Usuários sem assinatura
- Negócios fechados sem termo completo
- Ações críticas sem classificação de risco
- Mandato próximo do vencimento

**Botões:** 
- Declaração anual
- Assinaturas pendentes
- Auditoria
- Imutabilidade
- Mapa de risco
- Conformidade contratual
- LGPD
- Score
- Exportar Dossiê

### TELA C2 — DECLARAÇÃO ANUAL DO SÍNDICO

Obrigatória 1 vez por ano ou a cada novo mandato.
Texto jurídico robusto:
Declaro que exerço a função de síndico com diligência, observando a legislação aplicável, a convenção condominial, o regimento interno e as deliberações assembleares. Declaro que os registros realizados nesta plataforma refletem, dentro do meu conhecimento, informações verdadeiras e que busco atuar com transparência, responsabilidade administrativa e mitigação de riscos institucionais.
Declaro ainda que compreendo que os registros aqui efetuados possuem caráter documental e poderão ser utilizados como histórico formal da gestão.
- Checkbox obrigatório

**Botão:** Confirmar declaração
Registro automático:
- Data
- Hora
- IP
- Usuário
Publicação automática na Linha do Tempo: Tipo: Administrativo → Conformidade e Governança

**Visibilidade:** Gestão
Imutável.
Justificativa dev:
Cria camada formal de responsabilidade anual.

### TELA C3 — ASSINATURA OBRIGATÓRIA DOS USUÁRIOS

Lista de usuários cadastrados (até 3):
- Síndico principal
- Síndico preposto
- Administradora
- Conselho
Status individual:
- Assinado
- Pendente
Texto para usuários:
Declaro ciência de que minha atuação nesta plataforma é rastreável e deve refletir informações verdadeiras e diligentes. Estou ciente de que minhas ações poderão compor histórico documental da gestão condominial.

**Botão:** Assinar
Regra técnica:
Compliance só fica “Em dia” quando TODOS assinarem.
Publicação automática na Linha do Tempo para cada assinatura.

### TELA C4 — MATRIZ DE CONFLITO DE INTERESSE

Síndico deve declarar:
Possuo ou não possuo relação direta ou indireta com fornecedores cadastrados.

**Campos:** 
- Possui vínculo com fornecedor?
- Sim / Não
Se Sim: Descrever empresa
- Natureza da relação
- Condomínio impactado

**Botão:** Confirmar declaração
Registro imutável.
Justificativa dev: Protege contra alegação futura de favorecimento.

### TELA C5 — TRILHA DE AUDITORIA VISÍVEL

Lista estruturada:
- Usuário
- Perfil
- Ação realizada
- Campo alterado
- Versão anterior
- Versão atual
- Data
- Hora
Filtro por:
- Usuário
- Data
- Tipo de ação
Imutável.

**Justificativa:** Base probatória.

### TELA C6 — IMUTABILIDADE E ADENDOS

Exibe registros bloqueados:
- Votações encerradas
- Negócios fechados
- Declarações
- Eventos encerrados

**Botão:** Criar adendo formal
Adendo exige:
- Justificativa
- Descrição (voz/texto)
- Anexo opcional
Registro novo, sem alterar original.

**Justificativa:** Permite correção sem apagar histórico.

### TELA C7 — MAPA DE RISCO CONSOLIDADO

Consolida:
- Ações classificadas por risco
- Distribuição por categoria
- Quantidade alto/crítico
- Ações vencidas
Indicador visual:
- Baixo
- Médio
- Alto
- Crítico

**Justificativa:** Transforma dados em visão estratégica.

### TELA C8 — CONFORMIDADE CONTRATUAL

Lista de todos os negócios fechados.
Exibe por item:
- Valor
- Prazo início
- Prazo conclusão
- Garantia
- Contrato anexado (sim/não)
- Termos assinados (sim/não)
- Comunicação pós-fechamento validada (sim/não)
Alertas:
- Prazo de garantia próximo do fim
- Prazo de execução vencido

**Justificativa:** Evita passivo por esquecimento.

### TELA C9 — REGISTRO LGPD

**Exibe:** 
- Histórico Connect Me
- Empresa
- Data
- Finalidade
- Dados compartilhados
- Consentimento registrado
Texto fixo:
Os dados foram compartilhados mediante autorização expressa para finalidade específica, conforme Lei 13.709/2018.
Imutável.

**Justificativa:** Proteção contra questionamento de vazamento.

### TELA C10 — SCORE INTERNO DE GOVERNANÇA

Privado.
Cálculo baseado em:
- Percentual de ações preventivas
- Cumprimento de prazos
- Classificação de risco adequada
- Participação dos moradores
- Compliance ativo
- Validações em dia
- Trilha consistente

**Exibe:** 
- Score Governança
- Score Compliance
Com recomendações:
- Regularizar assinaturas
- Classificar risco pendente
- Atualizar ação vencida
- Encerrar votação aberta

**Justificativa:** Gamificação profissional e orientação.

### TELA C11 — DOSSIÊ DE GOVERNANÇA EXPORTÁVEL

**Botão:** Gerar Dossiê
Compila:
- Declarações anuais
- Assinaturas dos usuários
- Matriz de conflito
- Mapa de risco
- Trilha de auditoria (resumo)
- Contratações
- Termos assinados
- LGPD (resumo)
- Indicadores
Formato PDF estruturado.

**Uso:** 
- Assembleia
- Processo judicial
- Auditoria
- Seguro
- REGRAS DE GATILHO (IMPORTANTÍSSIMO)
Compliance passa a ser obrigatório para:
- Encerrar mandato
- Gerar relatório final
- Marcar negócio fechado (opcional, se quiser endurecer)
- Exportar dossiê
Sistema exibe:
Para prosseguir, finalize as pendências de compliance.
- CONTINUIDADE ENTRE MANDATOS
Quando novo síndico assume:
Ele deve:
- Assinar nova declaração
- Atualizar conflito de interesse
Mas mantém histórico anterior visível (somente leitura).
Isso cria:
Linha institucional contínua.

# Jornada do Empresa

**Nome do botão:** Painel Empresarial
- E1 — HOME DO PAINEL EMPRESARIAL
- Objetivo
Central estratégica de negócios e gestão documental da empresa.
- Mensagem institucional
Sua central de oportunidades, formalização contratual e proteção institucional no mercado condominial.
- Indicadores superiores (cards dinâmicos)
- Novas oportunidades
- Propostas aguardando validação
- Votações em andamento
- Negócios ativos
- Execuções pendentes de validação
- Garantias próximas do vencimento
- Avaliações recebidas
Cada indicador redireciona para a respectiva tela.

### 🔷 E2 — OPORTUNIDADES (CONNECT ME)

- Lista exibe
- Condomínio
- Bairro
- Cidade
- Tipo de condomínio
- Categoria do serviço
- Urgência
- Prazo desejado
- Data da solicitação
- Status
- Status possíveis
- Nova
- Interesse manifestado
- Encerrada
- Seleção estruturada – Urgência
- Baixa
- Média
- Alta
- Emergencial
- Botões
- Abrir
- Tenho interesse
- Recusar
- Motivos estruturados de recusa
- Fora da área atendida
- Sem disponibilidade operacional
- Escopo incompatível
- Capacidade técnica insuficiente
- Outro (campo aberto obrigatório)
- Regra sistêmica
Contato do síndico (nome, telefone, e-mail) somente exibido após clicar “Tenho interesse”.
Registro automático LGPD: Usuário
- Data
- Hora
- Finalidade (Negociação contratual)
- Versão do termo vigente

### 🔷 E3 — DETALHE DA OPORTUNIDADE

- Exibe
- Descrição detalhada
- Categoria
- Urgência
- Prazo desejado
- Observações adicionais
- Botões
- Tenho interesse
- Recusar
- Voltar
- Enviar proposta
Após manifestação de interesse: Dados do síndico liberados.

### 🔷 E4 — ENVIAR PROPOSTA

- Estados da proposta
- Rascunho
- Enviada
- Aguardando validação
- Validada
- Em votação
- Encerrada
- Selecionada
- Não selecionada
- Campos obrigatórios
- Descrição técnica detalhada
- Escopo delimitado
- Valor total
- Condições de pagamento
- Prazo para iniciar
- Prazo para concluir
- Garantia oferecida
- Responsável técnico
- Declaração de conformidade técnica
- Seleções estruturadas – Condições de pagamento
- À vista
- Entrada + parcelas
- Parcelado fixo
- Por medição
- Após entrega
- Outro
- Seleções estruturadas – Garantia
- Sem garantia
- 90 dias
- 6 meses
- 1 ano
- Superior a 1 ano
- Conforme contrato específico
- Regra de prazo
Prazo para envio definido pelo síndico.
Se expirar → status “Prazo expirado”.
Pode ser renovado pelo síndico → nova contagem.
- Regra de imutabilidade
Após validação do síndico → bloqueio de edição.
Alteração apenas via módulo de Adendo Formal.

### 🔷 E5 — PROPOSTAS

- Lista exibe
- Condomínio
- Serviço
- Valor
- Status
- Prazo de votação
- Data de envio

**Botão:** Abrir

### 🔷 E6 — VOTAÇÃO

- Exibe
- Prazo restante
- Status
- Resultado individual
- Regra
Empresa visualiza apenas: Selecionada
- Não selecionada
Nunca visualiza concorrência ou votos individuais.
- Prazo
Definido pelo síndico.
Pode ser prorrogado.
Encerramento pode ser manual ou automático.

### 🔷 E7 — NEGÓCIOS FECHADOS

- Lista exibe
- Condomínio
- Serviço
- Valor final
- Condições de pagamento
- Prazo início
- Prazo conclusão
- Garantia
- Status execução
- Estados do negócio
- Aguardando confirmação dupla
- Confirmado
- Em execução
- Concluído
- Garantia ativa
- Garantia encerrada

### 🔷 E8 — TERMO DE EXECUÇÃO

- Texto institucional versionado
Declaro executar o serviço conforme escopo validado, observando normas técnicas aplicáveis, legislação vigente e regras de segurança do trabalho, assumindo responsabilidade por vícios de execução, falhas técnicas e danos comprovadamente decorrentes da minha atuação.
Declaro possuir capacidade técnica, operacional e regularidade jurídica compatível com o objeto contratado.
- Registro obrigatório
- Versão do termo
- Usuário
- IP
- Data
- Hora
- Regra
Após 72 horas → bloqueio definitivo.
Alteração apenas por adendo formal com nova confirmação dupla.

### 🔷 E9 — EXECUÇÃO (REGISTRO DE SERVIÇO)

- Campos obrigatórios
- Descrição da atividade
- Data
- Fotos ou vídeo
- Classificação de risco
- Natureza da ação
- Observações
- Seleção estruturada – Classificação de risco
- Baixo
- Moderado
- Elevado
- Crítico
- Seleção estruturada – Natureza
- Preventiva
- Corretiva
- Emergencial
- Estratégica
- Estados
- Rascunho
- Enviado para validação
- Aprovado
- Publicado
- Bloqueado para edição
- Regra
Somente após validação do síndico integra Linha do Tempo.

### 🔷 E10 — COMUNICADOS PÓS-FECHAMENTO

- Tipos estruturados
- Aviso de início
- Intervenção técnica
- Interdição temporária
- Orientação de segurança
- Entrega de obra
- Relatório técnico
- Comunicado de conclusão
- Outro
- Campos
- Título
- Descrição
- Data início
- Data término
- Anexos
- Vídeo
- Estados
- Rascunho
- Enviado
- Aprovado
- Publicado

### 🔷 E11 — AVALIAÇÕES

- Critérios (escala 1 a 10)
- Qualidade técnica
- Cumprimento de prazo
- Comunicação
- Organização documental
- Confiabilidade
- Empresa pode
- Autorizar publicação
- Manter privada
- Responder formalmente
Resposta registrada e imutável.

### 🔷 E12 — INTELIGÊNCIA

- Exibe indicadores agregados
- Total de oportunidades
- Propostas enviadas
- Taxa de conversão
- Tempo médio de resposta
- Tempo médio de execução
- Avaliação média
- Volume por categoria
- Volume por região
Dados anonimizados.

### 🔷 E13 — USUÁRIOS

- Limite
- 3 usuários
- Perfis
- Administrador
- Operacional
- Agência de Marketing
- Permissões
- Administrador – total
- Operacional – propostas, execução, comunicados
- Agência – apenas vídeos institucionais
Todos assinam termo individual versionado.

### 🔷 E14 — COMPLIANCE EMPRESARIAL

- Inclui
- Declaração anual de regularidade fiscal
- Declaração anual de regularidade técnica
- Declaração anual trabalhista
- Histórico LGPD
- Controle de garantias
- Módulo de adendo formal
- Regra
Sem declaração anual atualizada → bloqueio de novas propostas.

### 🔷 E15 — ADENDO FORMAL

- Tipos
- Alteração comercial
- Alteração de prazo
- Alteração técnica
- Alteração de garantia
- Outro
- Regras
- Justificativa obrigatória
- Anexo opcional
- Exige confirmação dupla
- Registro imutável

### 🔷 E16 — AUDITORIA

- Exibe
- Usuário
- Ação
- Condomínio
- Data
- Hora
- IP
Imutável.
Observação:
- Comunicados após fechamentos, registros de serviços devem ser publicados na linha do tempo