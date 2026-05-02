---
title: Assembly — Assembleias, Votações, Atas e Homologação
version: 1.0
status: stable
type: requirement
domain: assembly
tags: [assembly, voting, quorum, agenda, transparency, homologation, legal]
---

# Assembly — Assembleias, Votações, Atas e Homologação

Domínio de assembleias condominiais: configuração, pauta, votação real-time com WebSocket, atas, relatórios e homologação de resultados.

---

## Fase 1: Pré-Assembleia

### Req 48 — Configuração

Síndico configura assembly antes de publicar pauta.

### Campos obrigatórios

- `type` (enum: `ordinaria`, `extraordinaria`, `ratificacao`, `outra`)
- `modality` (enum: MVP **presencial** apenas; online/híbrida futuro)
- `scheduled_date`, `scheduled_time` (data e hora de início)
- `location` (endereço completo)
- `speech_time_minutes` (2 ou 3 minutos padrão)
- `quorum_type` (enum: `simple_majority`, `qualified_majority`, `unanimous`)

### Requisitos MUST

- Validação de data: futura, com mínimo 5 dias de antecedência para notificação
- Uma assembleia por vez (no máximo 1 ativa por condomínio)
- Cancelamento possível até 3 dias antes (com notificação de cancelamento)

### Notas de implementação

- Tabelas: `assemblies`
- Enum: `assembly_status` (scheduled, published, in_progress, closed, homologated)

---

### Req 49 — Pauta

Estrutura de agenda com itens, fração ideal para votação, lock após publicação.

### 5 Tipos de item

1. `informativo` — informação do síndico (sem votação)
2. `consulta_moradores` — enquete consultiva (sem valor jurídico)
3. `votacao_simples` — sim/não
4. `votacao_fracao_ideal` — pesado pela fração ideal
5. `votacao_fornecedor` — entre 2+ propostas validadas

### Estrutura de item

- `order` (sequência na pauta)
- `title` (ex: "Aprovação de reforma do elevador")
- `description` (contexto)
- `item_type` (um dos 5 acima)
- `attachment_url` (PDF, Excel, etc)
- `votacao_required` (boolean — se item requer votação)

### Fração Ideal

- Upload via Excel: 2 colunas (unidade, fração)
- Soma deve ser = 100% (validação automática)
- Exemplo: "Apt 101: 0,5% | Apt 102: 0,5% | ... | Total: 100%"
- Storage: NUMERIC(5,4) no PostgreSQL (nunca float — T10)
- Lock após publicação: não pode editar fração (único jeito: adendo — Req 25)

### Requisitos MUST

- Pauta criada em draft (edição livre)
- Publicação bloqueia edições (lock)
- Publicação dispara notificações (Req 50)
- Validação: mín. 1 item, máx. 20 itens
- Fração ideal obrigatória se houver votação com peso

### Notas de implementação

- Tabelas: `assembly_agendas`, `assembly_agenda_items`, `assembly_fractions`
- Campo `published_at` → lock (se not null, pauta é read-only)

---

### Req 50 — Notificações

Notificações automáticas em 3 momentos.

### Timing

- **T-3 dias**: "Assembleia em 3 dias — clique para ler pauta"
- **T-1 dia**: "Assembleia amanhã às 10h — confirme presença"
- **T-3 horas**: "Assembleia começa em 3 horas"

### Canais

- Push (via FCM)
- Email (via Resend — template pt-BR)
- SMS (opcional, via Twilio — Sprint 3+)

### Requisitos MUST

- Envio automático quando pauta é publicada (dispara T-3 dias)
- Personalização: horário configurável de envio
- Tracking: registrar quem clicou na notificação

### Notas de implementação

- Tabelas: `assembly_notifications`, `assembly_notification_logs`
- Job agendado: scheduler NATS ou cron (runs a cada hora)

---

### Req 50.1 — Edital

Notificação formal de assembleia com regras.

### Requisitos MUST

- Geração automática em PDF (template preenchido)
- Ou upload manual (síndico redige PDF próprio)
- Contém: data, hora, local, pauta resumida, regras de participação
- Distribuição obrigatória (email + app notification)
- Assinatura digital (Sprint 3+)

### Notas de implementação

- Geração: template HTML → PDF via wkhtmltopdf ou Puppeteer (async job)
- Storage: MinIO

---

### Req 50.2 — Ciência Obrigatória

Morador perde acesso ao app até confirmar ciência de pauta.

### Requisitos MUST

- Modal/banner imperativos no app: "Você precisa dar ciência da pauta para continuar"
- Clique: "Li e entendi a pauta" → registra `acknowledged_at`
- Sem ciência: app fica travado (nenhuma ação possível)
- Tracking: 6 meses (log de quem deu ciência, quando)
- Auto-unlock: após assembleia encerrada, ciência não mais obrigatória

### Notas de implementação

- Tabelas: `assembly_acknowledgments`
- Middleware: antes de qualquer request autenticado, valida se `acknowledged_at is not null`

---

### Req 51 — Enquetes Preliminares

Consultas consultivas, sem valor jurídico.

### Requisitos MUST

- 3 tipos: `sim_nao_nao_sei`, `multiple_choice`, `scale_1_to_5` (igual Req 14.1)
- Criação pelo síndico antes da assembleia
- Resultado não é vinculante (meramente informativo)
- Resultado publicado na pauta

### Notas de implementação

- Tabelas: `assembly_preliminary_surveys`

---

### Req 52 — Procurações

Morador autoriza terceiro a votar em seu lugar.

### Requisitos MUST

- Validação: procurador deve ser morador do mesmo condomínio
- Prazo: válida até 24h **após** assembleia encerrada (permite voto atrasado)
- Proxy replica voto exato: se titular vota, procurador não pode votar diferente (voto é do titular)
- Assinatura digital (Sprint 3+)
- Cancelamento: titular pode revogar até 1h antes da assembleia

### Notas de implementação

- Tabelas: `proxies`
- Campo: `authorized_by_user_id`, `proxy_user_id`, `valid_until`, `revoked_at`

---

### Req 55 — Simulador de Quorum

Ferramenta para síndico estimar quorum necessário.

### Requisitos MUST

- Input: "Quantos moradores comparecem?" (número ou %)
- Output: "Quorum atingido? SIM/NÃO" + "Votos necessários para X passar"
- Cálculo usa fração ideal (não é 1 morador = 1 voto)
- Exemplo: "Com 60% de presença (fração), você atinge 60% do quorum qualificado (2/3)"

### Notas de implementação

- Endpoint: `GET /assemblies/{id}/simulator?present_fraction=0.60`
- Lógica: puro cálculo, sem persistência

---

## Fase 2: Live-Day

### Req 56 — Check-in

Registro de presença com 3 modalidades.

### Modalidades

1. **App (padrão):** QR Code + scan automático
2. **Terminal (kiosk):** CPF + bloco/unidade + confirma identidade
3. **Manual:** síndico confirma presença de morador (ex: deficiente visual)

### Requisitos MUST

- Check-in via app exige biometria ou PIN (Sprint 3+)
- Terminal: validação de CPF contra base de unidades
- Ausente momentâneo ≠ ausente definitivo (morador pode sair e voltar)
- Tracking: hora de check-in, quem confirma (para manual)
- WebSocket: atualiza painel do presidente em tempo real (Req 60.2)

### Notas de implementação

- Tabelas: `assembly_check_ins`
- WebSocket: emitir evento `assembly.attendance.updated` a cada check-in
- Latência-alvo: < 200ms WebSocket (NFR crítico)

---

### Req 57 — Votação Real-time

Votação com 4 tipos, resultado em tempo real via WebSocket.

### 4 Tipos

1. **Votação Simples** — sim/não (maioria simples: > 50%)
2. **Votação com Fração Ideal** — peso pelo % de fração (maioria: > 50% de fração)
3. **Votação de Fornecedor** — escolha entre 2+ propostas com fração ideal (Req 21)
4. **Eleição** — escolha de síndico entre candidatos com fração ideal

### Fluxo

1. Síndico abre votação (via Painel do Presidente — Req 60.2)
2. Morador recebe notificação "Votação aberta: Item 3"
3. Morador clica e vota (opção única, não permite mudança pós-envio)
4. Resultado: contagem em tempo real no painel, app mostra "Você votou"
5. Síndico encerra votação (ou timeout automático)
6. Resultado final: imutável (Rule 2)

### Voto Presencial Assistido

- Terminal no local permite síndico ajudar morador a votar
- Termo de voto obrigatório (`termo_de_voto`): morador confirma que autorizou voto
- Assinatura digital (Sprint 3+)

### Ausência Momentânea

- Se morador fez check-in mas saiu, e votação abre:
  - Morador tem 15 minutos para voltar e votar
  - Após 15 min: abstenção automática (voto é registrado como "absent")
  - Morador pode reabrir e votar dentro do prazo

### Requisitos MUST

- Fração ideal: NUMERIC(5,4), nunca float
- Resultado é imutável → único mecanismo de contestação: adendo (Req 25)
- Timestamp de voto, IP, device fingerprint (audit trail)
- WebSocket: latência < 200ms (NFR crítico)
- Votação não permite mudança: voto é final

### Notas de implementação

- Tabelas: `assembly_votings`, `assembly_votes`, `assembly_voting_results`
- Enum: `voting_status` (open, closed), `vote_option`
- WebSocket: `/ws/assemblies/{id}/voting`
- Redis: auto-save de estado (recuperação de crash — Req 60.10)

---

### Req 60.2 — Painel do Presidente

Dashboard para síndico controlar assembleia em tempo real.

### Funcionalidades

- **Visualização:** pauta, itens do dia, lista de presentes, votações abertas
- **Controle:** abrir/fechar votação, administrar fila de fala (Req 60.5)
- **Alertas:** quorum crítico, tempo de fala excedido, votação atrasada
- **Comunicados:** avisos em tempo real para moradores

### Requisitos MUST

- Acesso: síndico principal (ou designado) apenas
- Telemetria: mostra % presença em tempo real (atualiza a cada check-in)
- Bloqueio: síndico não consegue votar (imparcialidade)

### Notas de implementação

- Página separada: `/assemblies/{id}/president`
- WebSocket conectado: escuta eventos de check-in, votação, fila

---

### Req 60.3 — Telão 2 Áreas

Tela de apresentação e painel operacional.

### Área 1: Presentation

- Pauta actual
- Resultado de votação em tempo real (gráficos)
- Comunicados do síndico
- Vídeos de empresa (durante votação de fornecedor — Req 29, Rule 4)

### Área 2: Operational Panel

- Quorum atual (%)
- Fila de fala (quem fala, tempo restante)
- Alertas (ex: "Votação encerrada")
- Status: "Aguardando item 3" vs "Votação aberta"

### Requisitos MUST

- Telas sincronizadas via WebSocket
- Atualização em < 200ms (NFR crítico)
- Layout responsivo (pode rodar em 2 monitores)

### Notas de implementação

- Componentes SolidJS: `PresidentScreen`, `OperationalPanel`
- WebSocket: mesma conexão, múltiplos listeners

---

### Req 60.5 — Fila de Fala

Timer cronometrado para cada morador falar na assembleia.

### Estrutura

- Morador entra na fila (botão "Quero falar")
- Síndico aprova entrada (via Painel do Presidente)
- Timer inicia (2 ou 3 min conforme Req 48)
- Alerta em 30s para encerrar
- Síndico pode estender +1 min (1x)
- Após timeout: áudio/microfone corta automaticamente (Livekit — Req 68)

### Requisitos MUST

- Ordem FIFO (quem pediu primeiro fala primeiro)
- Personalização de tempo por item (ex: item 3 = 5 min de discussão coletiva)
- Histórico: registra quem falou, por quanto tempo
- Timeout: automático (não requer ação do síndico)

### Notas de implementação

- Tabelas: `assembly_speech_queue`, `assembly_speeches`
- WebSocket: `/ws/assemblies/{id}/speech`
- Integração Livekit: API de corte de microfone

---

### Req 60.10 — Modo Contingência

Fallback para votação offline (sem internet).

### Requisitos MUST

- Se WebSocket cai: app local salva votos em Redis
- Auto-sync: quando conexão volta, envia votos acumulados
- Conflitos: último voto válido (no caso de re-envio)
- Notificação: "Modo offline — seus votos serão sincronizados"

### Notas de implementação

- Redis key: `assembly:{id}:offline_votes:{user_id}`
- Job: sincroniza periodicamente quando conexão restabelecida

---

### Req 60.12 — Termos Legais

Aceite obrigatório de regras de assembleia antes do check-in.

### Conteúdo

- Regras de participação
- Direitos e deveres do morador
- Penalidades por abuso

### Requisitos MUST

- Modal obrigatório antes do primeiro check-in
- Checkbox: "Li e concordo com os termos"
- Versão dos termos: rastreada (future-proof para mudanças)
- Aceite: registrado com timestamp, IP

### Notas de implementação

- Tabelas: `assembly_legal_acceptances`

---

## Fase 3: Pós-Assembleia

### Req 58 — Encerramento

Síndico encerra assembleia e ata é gerada automaticamente.

### Fluxo

1. Síndico clica "Encerrar Assembleia"
2. Sistema valida: todas as votações foram encerradas?
3. Ata é gerada automaticamente (em PDF)
4. Ata é **publicada automaticamente na Timeline** tipo `assembly_result` (Req 11)
5. Status: `in_progress` → `closed`

### Conteúdo da ata

- Data, hora, local, presentes (lista anonimizada)
- Pauta e resultado de cada votação
- Fala de cada morador (texto ou resumo)
- Procurações usadas
- Assinatura digital do síndico (Sprint 3+)

### Requisitos MUST

- Geração automática (template HTML → PDF)
- Armazenamento em MinIO (backup)
- Auto-publicação na Timeline (Rule 2 — imutável)

### Notas de implementação

- Job: async, dispara após `assemblies.status = closed`
- Evento NATS: `assembly.closed` → `timeline.entry.created`

---

### Req 59 — Relatórios

6 tipos de relatório exportáveis em CSV e PDF.

### 6 Tipos

1. **Presença:** lista de moradores presentes/ausentes, fração
2. **Votação:** resultado detalhado de cada votação, votos por opção
3. **Procurações:** lista de procurações usadas
4. **Fala:** registro de falas, tempo por morador, topísticos (palavras-chave)
5. **Consolidado:** resumo executivo (1-2 páginas)
6. **Transparência:** score de transparência (Req 60.7)

### Requisitos MUST

- Geração sob demanda (síndico pode exportar)
- Anônimo: lista de presença NÃO mostra nomes (apenas unidade/fração)
- Confidencialidade: síndico + admin podem acessar

### Notas de implementação

- Templates: HTML/CSV para cada tipo
- Endpoint: `GET /assemblies/{id}/reports/{type}?format=csv|pdf`

---

### Req 60.6 — Homologação

Votação obrigatória de aprovação da ata (torna resultado final imutável).

### Fluxo

1. Ata gerada (Req 58)
2. Nova votação aberta: "Aprova a ata da assembleia?"
3. Quorum: maioria simples (> 50%)
4. Resultado: sim/não
5. Se aprovada: `homologated_at` é preenchido → **imutável daí em diante**

### Requisitos MUST

- Homologação é votação separada (não é automática)
- Após homologação: nenhuma edição é possível (Rule 2)
- Único jeito de contestar: adendo (Req 25)

### Notas de implementação

- Tabelas: `assembly_homologations`

---

### Req 60.7 — Score de Transparência

Pontuação 0-100 calculada baseada em 10 componentes.

### 10 Componentes

1. Pauta publicada com antecedência (≥5 dias: 10pt)
2. Edital distribuído (10pt)
3. Ciência de pauta registrada (10pt)
4. Quorum atingido (10pt)
5. Votação online disponível (10pt)
6. Ata publicada na Timeline (10pt)
7. Relatórios públicos acessíveis (10pt)
8. Score de fala equilibrada (10pt — analisa se uns poucos dominaram)
9. Homologação registrada (10pt)
10. Sem atrasos no encerramento (10pt)

### Cálculo

- Cada componente: 0-10 pontos (binário: tem ou não tem)
- Total: soma / 100 (0-100)
- Exibição: público (transparência)

### Requisitos MUST

- Cálculo automático após homologação
- Histórico: acompanhar score ao longo do tempo (tendência)
- Benchmark: comparar com condomínios similares (futuro)

### Notas de implementação

- Cálculo: função SQL pura ou job NATS
- Armazenamento: `assemblies.transparency_score`

---

### Req 60.8 — Notificações Automáticas Pós-Encerramento

Comunicados automáticos após assembleia.

### Notificações

- "Assembleia encerrada, ata disponível"
- "Ata foi homologada"
- "Score de transparência: 85/100"

### Canais

- Push, email, SMS (opt-in)

### Requisitos MUST

- Disparadas automaticamente (job)
- Personalizadas por morador (mostra resultado de votações relevantes)

---

### Req 60.9 — Indicador de Transparência

Widget visual no perfil do condomínio.

### Requisitos MUST

- Badge: "Transparência: 85/100"
- Histórico gráfico (últimas 12 assembleias)
- Benchmark: "Acima da média" / "Média" / "Abaixo"

### Notas de implementação

- Componente SolidJS: `TransparencyBadge`

---

## NFR Críticos

- **WebSocket latência**: < 200ms (P95) — obrigatório para check-in e votação real-time
- **Fração ideal**: NUMERIC(5,4) no PostgreSQL, nunca float
- **Append-only**: todos os eventos (voto, check-in, homologação) — histórico completo
- **Auto-save Redis**: estado de assembly (votações abertas, presença) — recuperação de crash < 1min

---

## Invariantes de assembly

1. Uma assembleia por vez (no máximo 1 ativa por condomínio)
2. Pauta é imutável após publicação (versionamento via adendo)
3. Voto é imutável (no máximo dentro de timeout aberto)
4. Fração ideal é NUMERIC, nunca float
5. Ausência momentânea ≠ abstenção automática (15 min para voltar)
6. Ata é auto-publicada na Timeline (imutável após homologação)
7. Procuração: morador não consegue votar diferente do procurador

---

## Referências cruzadas

- **Req 11** (Institutional): Timeline auto-publica ata (tipo `assembly_result`)
- **Req 21** (Commercial): votação de fornecedor com propostas validadas
- **Req 29** (Content): exibição de vídeos de empresa durante votação de fornecedor
- **Req 52** (Assembly): procurações — validação e voto de proxy
- **Req 68** (Integration): Livekit para corte de áudio via API
