---
created: '2026-04-24'
tags:
  - requirements
  - fase1
  - websocket-chat
title: Requisitos — Fase 1
type: note
updated: '2026-04-24'
---

# Requisitos — Fase 1

## Introdução

Fase 1 do sistema de chat via WebSocket em Go com interface CLI. Clientes se conectam ao servidor, recebem um UUIDv7 único, e trocam mensagens diretas entre si por esse identificador. O servidor mantém clientes conectados em memória e persiste histórico de mensagens no SQLite, garantindo entrega offline na reconexão.

Arquitetura inspirada em WhatsApp/Signal/Telegram — ver [[../02-architecture/system-design-referencias]].

Fases futuras:
- **Fase 2**: autenticação com username + Argon2id
- **Fase 3**: E2E com X25519 + ChaCha20-Poly1305 (Double Ratchet simplificado)

---

## Glossário

| Termo | Definição |
|---|---|
| **Servidor** | Processo Go que aceita conexões WebSocket, gerencia clientes e roteia mensagens |
| **Cliente** | Processo Go CLI que se conecta ao Servidor via WebSocket |
| **Sessão** | Conexão WebSocket ativa entre um Cliente e o Servidor |
| **ID de Cliente** | UUIDv7 gerado pelo Servidor e atribuído ao Cliente na conexão |
| **Mensagem Direta** | Mensagem de um remetente para um destinatário identificado pelo ID de Cliente |
| **Histórico** | Mensagens Diretas persistidas no SQLite associadas a um ID de Cliente |
| **Registro de Clientes** | Mapa em memória no Servidor: ID de Cliente ativo → Sessão WebSocket |
| **Payload** | Envelope JSON trafegado sobre o WebSocket |
| **UUIDv7** | UUID versão 7, baseado em timestamp — ordenável cronologicamente |
| **ACK** | Confirmação de recebimento enviada pelo servidor ao remetente após persistir |
| **Heartbeat** | Mecanismo ping/pong para verificar se a conexão está ativa |
| **Status de mensagem** | `pending` (aguardando entrega) ou `delivered` (entregue ao destinatário) |

---

## Requisitos

---

### R-01 — Atribuição de Identidade na Conexão

**User Story:** Como cliente, quero receber um identificador único ao me conectar ao servidor, para que eu possa ser identificado e endereçado por outros clientes.

#### Critérios de Aceitação

1. WHEN um Cliente estabelece uma conexão WebSocket com o Servidor, THE Servidor SHALL gerar um UUIDv7 único para aquela Sessão.
2. WHEN o UUIDv7 é gerado, THE Servidor SHALL enviar ao Cliente um Payload `welcome` contendo o ID de Cliente atribuído.
3. THE Servidor SHALL garantir que nenhum dois IDs de Cliente ativos no Registro de Clientes sejam iguais.
4. WHEN a Sessão é encerrada, THE Servidor SHALL remover o ID de Cliente correspondente do Registro de Clientes.

---

### R-02 — Registro de Clientes Conectados

**User Story:** Como servidor, quero manter um registro dos clientes atualmente conectados, para que eu possa rotear mensagens para os destinatários corretos.

#### Critérios de Aceitação

1. THE Servidor SHALL manter um Registro de Clientes em memória que associa cada ID de Cliente ativo à sua Sessão WebSocket.
2. WHEN um Cliente se conecta, THE Servidor SHALL inserir o ID de Cliente e a referência à Sessão no Registro de Clientes.
3. WHEN uma Sessão é encerrada por qualquer motivo (desconexão normal, erro de rede ou timeout), THE Servidor SHALL remover a entrada correspondente do Registro de Clientes.
4. WHILE múltiplos Clientes estão conectados simultaneamente, THE Servidor SHALL manter o Registro de Clientes consistente sob acesso concorrente.

---

### R-03 — Persistência de Clientes no SQLite

**User Story:** Como operador do sistema, quero que os clientes conhecidos sejam persistidos no banco de dados, para que o histórico de mensagens possa ser associado a identidades duráveis entre reconexões.

#### Critérios de Aceitação

1. WHEN um Cliente se conecta pela primeira vez com um ID de Cliente novo, THE Servidor SHALL inserir um registro do Cliente no SQLite.
2. THE Servidor SHALL armazenar no SQLite, para cada Cliente, o ID de Cliente e o timestamp da primeira conexão.
3. WHEN o banco de dados SQLite não existir no caminho configurado, THE Servidor SHALL criar o arquivo e aplicar o schema inicial automaticamente na inicialização.
4. IF a operação de escrita no SQLite falhar, THEN THE Servidor SHALL registrar o erro em log e encerrar a Sessão com código de fechamento indicando erro interno.

---

### R-04 — Envio de Mensagem Direta

**User Story:** Como cliente, quero enviar uma mensagem para outro cliente pelo ID dele, para que eu possa me comunicar diretamente com um destinatário específico.

#### Critérios de Aceitação

1. WHEN um Cliente envia um Payload `send_message` contendo ID destinatário e conteúdo de texto, THE Servidor SHALL validar que o ID destinatário está no formato UUIDv7.
2. IF o Payload não contiver os campos obrigatórios (ID destinatário e conteúdo), THEN THE Servidor SHALL responder ao remetente com Payload `error` descritivo e descartar a mensagem.
3. WHEN o Payload é válido, THE Servidor SHALL persistir a Mensagem Direta no SQLite **antes** de tentar a entrega (write-ahead — padrão WhatsApp).
4. THE Servidor SHALL armazenar no SQLite: ID da mensagem (UUIDv7), ID remetente, ID destinatário, conteúdo e timestamp de recebimento.
5. WHEN a mensagem é persistida com sucesso, THE Servidor SHALL enviar ao remetente um ACK contendo o ID da mensagem gerado pelo servidor.

---

### R-05 — Entrega de Mensagem para Cliente Online

**User Story:** Como cliente destinatário, quero receber mensagens em tempo real quando estou conectado, para que a comunicação seja imediata.

#### Critérios de Aceitação

1. WHEN o Servidor recebe uma Mensagem Direta válida e o ID destinatário está presente no Registro de Clientes, THE Servidor SHALL encaminhar o Payload `receive_message` para a Sessão do destinatário.
2. WHEN a entrega ao destinatário é bem-sucedida, THE Servidor SHALL marcar a mensagem no SQLite como `delivered`, registrando o timestamp de entrega.
3. IF a escrita na Sessão do destinatário falhar, THEN THE Servidor SHALL registrar o erro em log e manter a mensagem como `pending` para reentrega futura.
4. WHEN a mensagem é entregue ao destinatário, THE Servidor SHALL enviar ao remetente um Payload `delivery_confirmation` contendo o ID da mensagem.

---

### R-06 — Armazenamento de Mensagens para Cliente Offline

**User Story:** Como cliente destinatário, quero receber as mensagens que me foram enviadas enquanto eu estava desconectado, para que nenhuma mensagem seja perdida.

#### Critérios de Aceitação

1. WHEN o Servidor recebe uma Mensagem Direta válida e o ID destinatário **não** está no Registro de Clientes, THE Servidor SHALL persistir a mensagem com status `pending` e enviar ao remetente Payload informando que o destinatário está offline.
2. WHEN um Cliente se reconecta com o mesmo ID de Cliente, THE Servidor SHALL consultar o SQLite por mensagens `pending` associadas àquele ID.
3. WHEN mensagens pendentes são encontradas após reconexão, THE Servidor SHALL encaminhar cada mensagem ao Cliente na ordem cronológica de recebimento.
4. WHEN todas as mensagens pendentes são entregues com sucesso, THE Servidor SHALL atualizar o status de cada mensagem para `delivered`.

---

### R-07 — Protocolo de Mensagens (Payload JSON)

**User Story:** Como desenvolvedor, quero um protocolo de mensagens bem definido entre cliente e servidor, para que a comunicação seja previsível e extensível para as fases futuras.

#### Critérios de Aceitação

1. THE Servidor SHALL aceitar e emitir Payloads no formato JSON sobre o WebSocket.
2. THE Servidor SHALL exigir que todo Payload contenha um campo `type` do tipo string.
3. WHEN o Servidor recebe um Payload com `type` desconhecido, THE Servidor SHALL responder com Payload `error` indicando o tipo inválido e descartar a mensagem.
4. THE Servidor SHALL suportar os seguintes tipos na Fase 1: `welcome`, `send_message`, `receive_message`, `delivery_confirmation`, `error`, `pending_messages`.
5. WHEN o Servidor recebe um Payload que não é JSON válido, THE Servidor SHALL encerrar a Sessão com código WebSocket `1003` (dados não suportados).

---

### R-08 — Operação Concorrente do Servidor

**User Story:** Como operador do sistema, quero que o servidor suporte múltiplos clientes simultâneos sem degradação de consistência.

#### Critérios de Aceitação

1. THE Servidor SHALL tratar cada Sessão em uma goroutine dedicada.
2. WHILE múltiplas goroutines acessam o Registro de Clientes simultaneamente, THE Servidor SHALL proteger o Registro com `sync.RWMutex`.
3. WHILE múltiplas goroutines realizam operações no SQLite simultaneamente, THE Servidor SHALL garantir serialização correta para evitar corrupção de dados.
4. IF uma goroutine de Sessão encerrar com panic, THEN THE Servidor SHALL recuperar o panic, registrar o erro em log e continuar aceitando novas conexões.

---

### R-09 — Interface de Linha de Comando do Cliente

**User Story:** Como usuário, quero interagir com o sistema de chat por meio de uma interface de linha de comando.

#### Critérios de Aceitação

1. WHEN o Cliente é iniciado, THE Cliente SHALL conectar-se ao endereço do Servidor especificado como argumento de linha de comando ou variável de ambiente.
2. WHEN o Cliente recebe o Payload `welcome`, THE Cliente SHALL exibir o ID de Cliente atribuído no terminal.
3. WHEN o usuário digita uma mensagem no formato `<ID_DESTINATÁRIO> <CONTEÚDO>` e pressiona Enter, THE Cliente SHALL construir e enviar o Payload `send_message` correspondente.
4. WHEN o Cliente recebe um Payload `receive_message`, THE Cliente SHALL exibir no terminal o ID do remetente e o conteúdo da mensagem.
5. WHEN o Cliente recebe um Payload `error`, THE Cliente SHALL exibir a mensagem de erro no terminal sem encerrar a conexão.
6. IF a conexão com o Servidor for perdida, THEN THE Cliente SHALL exibir mensagem de desconexão e encerrar com código de saída não-zero.

---

### R-10 — Heartbeat e Detecção de Conexão Morta

**User Story:** Como servidor, quero detectar clientes que perderam a conexão sem fechar explicitamente, para que o Registro de Clientes não acumule entradas fantasma.

#### Critérios de Aceitação

1. THE Servidor SHALL enviar um ping WebSocket a cada 30 segundos para cada Sessão ativa.
2. IF o Cliente não responder com pong dentro de 10 segundos após o ping, THEN THE Servidor SHALL encerrar a Sessão e remover o Cliente do Registro de Clientes.
3. WHEN o Cliente recebe um ping do Servidor, THE Cliente SHALL responder com pong automaticamente (comportamento padrão da lib WebSocket).
4. WHEN uma Sessão é encerrada por timeout de heartbeat, THE Servidor SHALL registrar o evento em log com o ID de Cliente correspondente.

---

### R-11 — Identificação Única de Mensagens

**User Story:** Como desenvolvedor, quero que cada mensagem tenha um ID único gerado pelo servidor, para rastrear o ciclo de vida de cada mensagem e preparar deduplicação nas fases futuras.

#### Critérios de Aceitação

1. THE Servidor SHALL gerar um UUIDv7 único para cada Mensagem Direta recebida e persistida.
2. THE Servidor SHALL incluir o ID da mensagem em todos os Payloads relacionados: ACK ao remetente, entrega ao destinatário e confirmação de entrega.
3. THE Cliente SHALL exibir o ID da mensagem no terminal ao receber confirmação de entrega.
4. THE Servidor SHALL rejeitar com erro qualquer tentativa de reprocessar uma mensagem com ID já existente no SQLite.

---

## Links

- [[../_moc]]
- [[../STATE]]
- [[../02-architecture/system-design-referencias]]
- [[../02-architecture/system-overview]]
- [[../05-tasks/fase1-tasks]]
