---
title: "Domínio - Assembleia"
version: "3.0"
date: 2026-03-10
status: "em-revisão"
high_domain: "Assembleia"
requisitos: "Req 48–60.12"
total_requisitos: 20
tags:
  - mastersindico
  - dominio
  - assembleia
  - tempo-real
  - votacao
  - compliance
---

# 🏛️ Domínio Principal: Assembleia (Módulo Live)

> **High-Domain:** Assembleia (Módulo Premium/Cross-Domain) | **Requisitos:** 48–60.12
> Deliberação em tempo real, votação, telão, contingência e transparência.
> **Breadcrumbs:** [[00 - Índice Master Síndico|🏠 Início]] > [[Arquitetura do Ecossistema|🏛️ Arquitetura]] > 📄 Domínio - Assembleia

---

## 🎯 Por que este módulo existe? (Problema & Solução)

**Problemas que estamos resolvendo:**
1. **Caos em assembleias híbridas:** O uso de múltiplas ferramentas (Zoom para vídeo, Google Forms para voto, WhatsApp para dúvidas) gera atas frágeis juridicamente.
2. **Cálculo de Quórum e Fração Ideal:** Falhas humanas ao computar votos por fração ideal em tempo real durante votações tensas.
3. **Falta de Transparência:** Moradores questionam a validade dos votos e procurações após a assembleia.
4. **Desorganização da Pauta:** Assembleias que duram horas pois síndicos não conseguem controlar o tempo de fala de moradores específicos.

**Solução (Master Síndico Live Assembly - Opção B):**
Um ecossistema de *Single Source of Truth* (SSOT) em tempo real via WebSockets. Centraliza check-in, fila de oradores (com timer cronometrado), votação travada por procuração/fração ideal, telão em 2 áreas e auditoria imutável (append-only) para cada ação, gerando a ata quase automaticamente.

**Soluções Equivalentes no Mercado:**
- *Superlógica / Área do Condômino:* Focam em assembleias virtuais assíncronas (abertas por dias).
- *Winker / Condomob:* Oferecem assembleia, mas sem o rigor do "Telão 2 Áreas", Fila de Oradores Cronometrada e Modo Contingência Assistido detalhado aqui.

---

## 🏗️ Visão da Arquitetura de Assembleia

![[Arquitetura Assembleia.canvas]]

---

## 📋 Análise Minuciosa Linha a Linha dos Requisitos Funcionais (FR)

### Fase 1: Pré-Assembleia (Configuração e Engajamento)

#### Req 48: Configuração (Assembly Pre-Configuration)
- Criação base com `title`, `type` (ordinária/extraordinária/implantação), `date`, `time`, `location`, `quorum_type` (simples/qualificada/unânime).
- **Tempo de fala:** Configuração prévia de 2 ou 3 minutos (com extensão máxima de até 5 min).
- **Aviso Crítico:** Emissão da lista de inadimplentes pela administradora é *obrigatória* até **1 hora** antes da 1ª convocação. Falha nisso = **Bloqueio total** da publicação da assembleia.
- **Modalidade MVP:** Somente **Presencial** ativa. Híbrida e Online estarão preparadas na infra (campos `link_transmissao`, etc), mas desativadas no MVP.

#### Req 49: Gestão da Pauta (Agenda)
- Criação de tópicos: informativo, votação simples, votação por fração ideal, escolha de fornecedor, eleição.
- **Fração Ideal:** Upload de planilha Excel estruturada (`bloco`, `unidade`, `fracao_percentual`). Sistema faz checagem matemática: Soma = 100%. Falha rejeita o upload.
- **LOCK_Pauta:** Após publicada, a pauta é trancada. Não permite novos itens, nem mudar peso de quórum ou anexos estruturais. Apenas correções de erros de digitação (typos) são permitidas.

#### Req 50 & 50.1: Notificações e Edital
- Réguas de aviso: 3 dias antes, 1 dia antes, 3 horas antes.
- Edital: Pode ser gerado pelo sistema (PDF) com regras, dados e ordem do dia exportada automaticamente da pauta, ou anexado pronto pela administradora.

#### Req 50.2: Registro de Ciência 🛑 (Bloqueio Estrutural)
- O morador **PERDE ACESSO** aos recursos do app até dar "Ciência" no edital/pauta. (Garante segurança jurídica).
- O sistema retém o tracking de ciência por 6 meses.

#### Req 51, 52 & 55: Engajamento, Procurações e Quórum
- **Enquetes Preliminares:** Válidas até 24h antes da assembleia. Valor puramente consultivo (sem valor jurídico - exibido disclaimer).
- **Procurações:** Exige upload de documento validado pela administradora. Prazo limite até clicar em "Iniciar assembleia". Válida por apenas 24h pós-assembleia, limitando fraudes futuras. O proxy replica exato o voto do morador original.
- **Simulador de Quórum:** Soma de intenção de presença + procuracoes em tempo real para o síndico medir a viabilidade.

---

### Fase 2: Execução Ao Vivo ("The Live Engine")

> [!CAUTION]
> Durante a execução ao vivo, a estabilidade e concorrência são cruciais. A infraestrutura necessita de WebSockets. 

#### Req 56: Check-in & Controle de Presença
- Identificação por App, QR Code ou Terminal (inserindo CPF, Bloco e Unidade).
- Tracking de estado: `presente`, `ausente_momentaneo` (saiu do local/desconectou). Se o usuário desconecta, ele não zera sua presença, entra em "ausente momentâneo".

#### Req 57: Votação em Tempo Real
- Opções binárias (`favor`/`contra`/`abstencao`) ou escala (`escala_1_a_5`).
- **Voto Presencial Assistido:** Síndico/Administradora usa um terminal para computar o voto de quem está sem celular. Exige assinatura de `termo_de_voto` declarando responsabilidade civil no app do operador.
- Ponderação automática pela Fração Ideal e importação de votos em procuração.
- **Regra de Ausência:** Morador atestado ausente momentâneo em uma votação tem seu voto computado como `Abstenção`.

#### Req 60, 60.2 & 60.3: Controle do Presidente e Telão (Big Screen)
- **Painel do Presidente (60.2):** Controle absoluto de rotas: Abre assembleia, chama item, dá a palavra, abre votação, encerra discussão, finaliza assembleia.
- **Telão dividido em 2 áreas (60.3):**
  1. *Presentation Area:* Mostra o slide ou documento atual.
  2. *Operational Panel:* Mostra a Pauta com destaque, quorum presente/necessário, fila de oradores, timer, e resultados.
  *(Isso evita que o síndico tenha que trocar janelas do computador).*

#### Req 60.5: Fila de Fala (Speech Queue)
- O morador clica em "Levantar a Mão" e entra numa fila ordenada.
- O Presidente concedendo a fala ativa um **Timer na tela e no telão**.
- Extensões (`prorrogacao`) podem adicionar +2 min, até o limite de 5 min (configurado no Req 48).
- Alerta visual entra nos últimos 30 segundos.

#### Req 60.10: Modo de Contingência 🛡️
- Se falhar internet ou devices, o Síndico ativa emergência registrando a razão (`internet_failure`, etc).
- O sistema muda o fluxo para aceitar votos presenciais assistidos em massa na mesa diretora, marcando os itens de pauta afetados com uma flag de "contingência ativada" (fundamental para auditoria).

---

### Fase 3: Encerramento e Pós-Assembleia

#### Req 60.6 & 58: Homologação e Finalização
- **Homologação Obrigatória (60.6):** Após uma votação, ela fica em estado temporário; se a assembleia cair, ela pode ser zerada. Apenas depois do clique em **"Homologar Votação"** o registro trava (Lock Definitivo).
- O encerramento definitivo (58) compila presenças, atos do presidente, ata base, enviando notificação gerida (60.8) informando o término para a base.

#### Req 60.7 & 60.9: Consolidação de Dados e Transparência
- Exporta dados estruturados: Horários, atas, duração por item de pauta, atas pré-preenchidas.
- **Indicador de Transparência (0-100):** Calcula engajamento, % de ciência prévia, leitura da pauta, regularidade na homologação e completude de auditoria. Exibido no dossiê da assembleia.

---

## 🛠️ Requisitos Não Funcionais Críticos (NFR)

1. **Alta Disponibilidade e Concorrência (Real-time):** Uso extensivo de WebSockets (`socket.io` ou WS nativo do Fastify) para Check-in, Status de Voto, e Timer de Fila de Oradores. Atrasos < 200ms.
2. **Consistência Numérica Transacional (ACID):** Fração Ideal requer precisão decimal (`numeric / decimal` em vez de float/double no PostgreSQL).
3. **Imutabilidade e Trilha de Auditoria (Req 37 linkado):** Todos os eventos de "Voto Computado", "Check-in Registrado", "Homologação" devem ser Append-Only em tabelas de logging para sustentar viabilidade legal em tribunais.
4. **Recuperação de Estado (Auto-Save):** A votação e o estado da fila de oradores devem persistir em Redis/DB continuamente (Req 57.20). Se a máquina do síndico queimar no meio da assembleia, ao logar em outro note, o estado tem estar idêntico.

---

## 🌐 Rotas da API e WebSockets (Arquitetura)

### RESTful Endpoints (Pré e Pós)
```http
POST   /v1/assemblies/                        → Criar/Configurar
POST   /v1/assemblies/:id/agenda              → Adicionar Item na Pauta
POST   /v1/assemblies/:id/agenda/:itemId/ideal-fractions → Upload Planilha Excel
PUT    /v1/assemblies/:id/publish             → Publicar (Tranca Pauta)
POST   /v1/assemblies/:id/ciencia             → Morador dá Ciência (Desbloqueia App)
POST   /v1/assemblies/:id/proxies             → Envio de Procuração
GET    /v1/assemblies/:id/quorum-simulation   → Simulador de Quórum
POST   /v1/assemblies/:id/finalize            → Encerramento, Homologação Geral
```

### Eventos WebSocket Emitidos (Ao Vivo)
```json
// Namespace: /assembly-live/:id

// Eventos de Entidade (Morador)
emit("morador_checkin", { moradorId, unit, block, timestamp })
emit("morador_raise_hand", { moradorId, timestamp })
emit("morador_cast_vote", { moradorId, itemId, payload }) // Envio de voto

// Eventos de Controle (Mesa/President)
broadcast("sync_state", { currentAgendaItem, quorumPresent, connectedUnits })
broadcast("speaker_timer_start", { moradorId, durationSecs })
broadcast("speaker_timer_tick", { remainingSecs })
broadcast("item_voting_opened", { itemId })
broadcast("item_voting_closed", { itemId })
broadcast("item_voting_homologated", { itemId, results })
broadcast("contingency_activated", { reason, time })
```

---

## 🔗 Relações e Gaps Resolvidos

- O **Telão** e o **President Panel** se comunicam indiretamente. O Presidente envia comando para API -> API dispara evento WebSockets -> Telão consome evento.
- **Risco LGPD Minimizado:** Apenas CPFs necessários durante o check-in no terminal. Registros de votos são preservados mas só processam identificação restrita quando acessado por auditoria (A ata mostra Unidade/Bloco e decisão, sem expor intimidade do voto diretamente na tela pública, apenas métricas).

---

**Breadcrumbs:** [[00 - Índice Master Síndico|🏠 Início]] · **Anterior:** [[Domínio - Conteúdo]]
