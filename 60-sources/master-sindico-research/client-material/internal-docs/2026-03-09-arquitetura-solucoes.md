# Master Síndico — Arquitetura de Soluções

> **Versão:** 1.0 — Março 2026
> **Tipo:** Documento de Trabalho — Arquiteto de Soluções
> **Destino:** Obsidian → referência para Claude Code / dev manual
> **Status:** Em construção — pendências mapeadas explicitamente com 🔴

---

## 1. VISÃO DO ECOSSISTEMA

A Master Síndico **não é um aplicativo**. É um **ecossistema de produtos** integrados por uma camada de identidade comum, operando sobre múltiplos tipos de tenant.

### 1.1 Definição Oficial

Plataforma digital de **governança condominial aplicada**, composta por:

- **Backend (API/Core)** — regras de negócio, banco de dados, integrações
- **Frontend Web** — SPA responsiva (SolidJS)
- **Aplicativo Mobile** — iOS/Android (espelhamento do Web para perfis finais)

### 1.2 O Que a Plataforma NÃO É (Limites Contratuais)

| NÃO É | Justificativa |
|-------|---------------|
| Rede social | Sem rankings públicos, sem contadores visíveis, sem competição de vaidade |
| Chat / Mensageria | Connect Me = formulário unidirecional, sem histórico, sem reply |
| Videoconferência (tipo Zoom) | 🔴 **CONFLITO**: escopo original dizia assembleias assíncronas, documento de Assembleia descreve assembleia ao vivo — pendente resolução com cliente |
| Ferramenta de prospecção | Contato parte sempre do interesse do contratante via busca |
| Editor de mídia | Armazenamento e reprodução apenas; upload de arquivo pronto |
| Moderação por IA | Moderação reativa (denúncias) + manual (admin MasterSíndico) |
| Helpdesk | "Ticket" = Cupom de Desconto, não chamado de suporte |

### 1.3 Regra de Ouro

> **Backend como Verdade Única.** Toda regra de negócio reside na API. O frontend é apenas interface de consumo. Se a API diz que o usuário não pode ver X, o frontend não mostra X.

---

## 2. TIPOS DE TENANT

O ecossistema opera com **dois tipos de tenant confirmados** e **um actor delegado**:

### 2.1 Tenant: Condomínio

O condomínio é a unidade central de isolamento de dados na camada de governança.

**Quem opera dentro:**
- Síndico (proprietário do tenant, até 15 condomínios por síndico)
- Moradores (vinculados a unidades: 1 titular + até 2 dependentes por unidade)
- Empresa Síndica Vinculada (actor delegado com permissões parametrizáveis)
- Administradora (actor operacional no módulo Assembleia)

**O que existe dentro do tenant:**
- Linha do Tempo (memória institucional, append-only, imutável após publicação)
- Plano Diretor (reativo — status muda apenas via publicação na Linha do Tempo)
- Eventos
- Comunicados
- Votações / Assembleias
- Compliance (Termo, Declaração Anual, Auditoria)
- Avaliação da Gestão
- Dashboard do Síndico

**Regras estruturais do tenant:**
- Toda ação do síndico exige confirmação de contexto ("Publicar para Condomínio X?")
- Empresa não publica direto — tudo passa por validação do síndico
- Registros publicados são imutáveis — apenas adendo
- Plano Diretor não é atualizado manualmente — status vem da Linha do Tempo
- Edição mantém versão anterior + trilha de auditoria
- Novo síndico herda histórico; antigo mantém leitura

### 2.2 Tenant: Empresa

A empresa é uma entidade independente com estrutura organizacional própria que **cruza fronteiras de condomínios**.

**Quem opera dentro:**
- Administrador (gestão completa da empresa)
- Operador Técnico (registrar execução, criar atividade técnica, enviar comunicados, publicar vídeos)
- Agência de Marketing (actor delegado — acesso restrito a conteúdo institucional)

**O que existe dentro do tenant:**
- Oportunidades (Connect Me recebido)
- Registros de Execução (enviados para validação do síndico)
- Atividades Técnicas
- Comunicados Técnicos
- Status de Envios
- Dashboard Empresarial
- Perfil Institucional
- Gestão de Agência de Marketing
- Marketing Link

**Regras estruturais do tenant:**
- Todo envio ao condomínio passa por validação do síndico
- Dados do síndico só revelados após "Tenho Interesse"
- Recusa de oportunidade exige motivo estruturado
- Agência vinculada via token, empresa pode encerrar acesso a qualquer momento

### 2.3 Actor Delegado: Agência de Marketing

A agência **NÃO é tenant**. É um actor com escopo extremamente restrito.

**O que pode fazer:**
- Publicar vídeos institucionais para empresas que a contrataram
- Ver biblioteca de vídeos
- Acompanhar métricas de desempenho (views, retenção)
- Receber Marketing Links (solicitações de contato)

**O que NÃO pode acessar:**
- Connect Me
- Registros de execução
- Comunicados técnicos
- Linha do Tempo de qualquer condomínio
- Dados de síndicos ou moradores
- Dashboard de governança

**Regra crítica de vídeos:** Moradores NÃO acessam vídeos institucionais normalmente. Visualizam apenas quando a empresa participa de processo de escolha de fornecedor no módulo assembleia. Após votação, acesso é bloqueado.

### 2.4 Actor Delegado: Empresa Síndica Vinculada

Cadastrada pelo síndico (S5), vinculada por token de e-mail.

**Permissões parametrizáveis:**
- Visualizar publicações
- Editar publicações do síndico
- Ocultar publicações
- Validar registros de execução de empresas
- Validar comunicados técnicos de empresas

Toda ação entra na trilha de auditoria.

### 2.5 Actor Delegado: Administradora (Assembleia)

Opera exclusivamente no módulo de Assembleia.

**O que pode fazer:**
- Importar fração ideal
- Registrar inadimplência (upload planilha ou manual)
- Validar procurações
- Homologar votação

---

## 3. HIGH-DOMAINS

Pensando no ecossistema como rede (DNS Architecture), os nós principais são:

```
                    ┌──────────┐
                    │ IDENTITY │
                    └────┬─────┘
                         │
          ┌──────────────┼──────────────┐
          │              │              │
    ┌─────▼─────┐  ┌─────▼─────┐  ┌────▼──────┐
    │INSTITUTION│  │COMMERCIAL │  │  CONTENT  │
    │    AL     │  │           │  │           │
    └───────────┘  └───────────┘  └───────────┘
```

### 3.1 HIGH-DOMAIN: Identity

**Função:** resolver identidade para todo o ecossistema. Como `accounts.google.com` serve tudo no Google.

**Responsabilidades:**
- Autenticação (JWT + Passport)
- Registro de usuário
- Gestão de planos e billing
- Trial (7 dias, plano Pro)
- Role assignment (ABAC)
- Onboarding inteligente (identificação de tipo via SearchParams)

**Regras de billing:**
- Trial: 7 dias com acesso Pro completo
- Cobrança: mensal ou anual a partir da data de assinatura
- Expiração: recursos bloqueados retroativamente — o que fez com recurso pago fica inacessível (não deleta, trava)
- O plano é uma chave global que liga/desliga features across domains

**Não depende de ninguém. Todos dependem dele.**

🔴 **PENDENTE:** Matriz funcional nova (define quem pode o quê por plano)
🔴 **PENDENTE:** Detalhamento dos planos por persona (quais planos existem para morador, síndico, empresa)
🔴 **PENDENTE:** Fluxo de onboarding completo por tipo de usuário (campos de cadastro por persona)

### 3.2 HIGH-DOMAIN: Institutional

**Função:** tudo que é tenant-bound ao condomínio. O coração operacional da plataforma.

**Mid-domains identificados:**

#### 3.2.1 MID: Condominium Registry

Registro e identidade do condomínio.

- CRUD de condomínio (S3: nome, CEP, endereço, tipo, unidades, blocos/torres, foto fachada)
- Geração automática: ID único, QR Code, link do app
- Dados da gestão (S4: tipo mandato, datas, vínculo empresa síndica)
- Cadastro/vínculo empresa síndica (S5: razão social, CNPJ, token, permissões)
- Seletor de condomínio (S2: ativos vs encerrados, até 15 por síndico)
- Encerramento de mandato (com motivo obrigatório)

**Tipos de condomínio:** Residencial, Comercial, Misto, Shopping Center, Galeria Comercial, Edifício Garagem

**Tipos de mandato:** Síndico eleito, Síndico profissional, Síndico interino

#### 3.2.2 MID: Tenant Membership (Morador)

Vínculo entre usuário e unidade do condomínio.

- Buscar condomínio por ID (M3)
- Cadastro de unidade (M4: bloco/torre, número, tipo, vínculo)
- Termo de ciência e responsabilidade (obrigatório)
- Gestão de acessos (M14: incluir/remover dependentes, atualizar status)
- Encerrar vínculo (M15: com motivo estruturado, mantém histórico)

**Regras:**
- 1 titular por unidade, até 2 dependentes
- Dependentes não alteram status da unidade, não removem titular, não incluem novos dependentes
- Unidade vazia = dependentes removidos automaticamente
- Acesso ao conteúdo depende do vínculo com unidade

**Tipos de unidade:** Residencial, Comercial, Vaga de garagem
**Tipos de vínculo:** Proprietário residente, Proprietário não residente, Inquilino
**Status da unidade:** Ocupada pelo proprietário, Ocupada por inquilino, Unidade vazia

#### 3.2.3 MID: Governance Engine

O coração pesado da plataforma. Opera dentro do tenant do condomínio.

**Low-domains:**

**LOW: Linha do Tempo**
- Memória institucional do condomínio (S11)
- Append-only, imutável após publicação — apenas adendo
- Tipos de publicação: atividade da gestão, registro de execução, comunicado, evento, adendo, resultado de consulta aos moradores
- Criar atividade (S12): 26 tipos de atividade, 9 tipos de risco, 8 próximas ações
- Editar atividade (S13): gera nova versão com justificativa, nunca sobrescreve
- Filtros: período, tipo, empresa, área impactada, status, vínculo ao PD
- Tudo que for público pode aparecer ao morador

**LOW: Plano Diretor**
- Ações estratégicas da gestão (S7-S10)
- Status **NÃO é atualizado manualmente** — muda via publicação vinculada na Linha do Tempo
- Status automático inicial: Planejada
- Status possíveis: Planejada, Em contratação, Em execução, Concluída, Suspensa, Reprogramada, Atrasada
- "Atrasada" pode ser calculado automaticamente
- Cada ação tem: área impactada (26 opções), natureza (8 opções), nível de importância (5 opções), impacto esperado (10 opções)
- Seção de registros vinculados: lista de publicações da Linha do Tempo ligadas por `master_plan_action_id`

**LOW: Eventos**
- Gerenciamento de eventos (S15-S17)
- 13 tipos de evento (assembleia ordinária, extraordinária, manutenção programada, etc.)
- Moradores podem: confirmar participação, marcar como ciente
- Métricas: taxa de confirmação, taxa de ciência
- Alimenta dashboard do síndico

**LOW: Comunicados**
- Comunicados institucionais e técnicos (S23-S25)
- 8 tipos de comunicado
- Comunicado da empresa entra como pendente de validação
- Controle de expiração (data início/fim)
- Métricas: taxa de leitura
- Histórico de versões

**LOW: Validações Pendentes**
- Hub centralizado (S26)
- Três seções: registros de execução, atividades de empresas, comunicados de empresas
- Toda decisão gera auditoria
- Publicação validada entra na Linha do Tempo

**LOW: Pergunta aos Moradores**
- Embutida dentro de uma atividade da Linha do Tempo
- Tipos de resposta: Sim/Não/Não sei, Escala 1-5, Campo aberto, Múltipla escolha
- Público: todos moradores ou apenas titulares
- Prazo definido
- Resultado consolidado vira publicação na Linha do Tempo
- Pergunta não pode ser editada após publicação

**LOW: Dashboard do Síndico**
- Indicadores (S27): moradores cadastrados, % unidades com acesso, avaliação média, ações PD por status, atividades publicadas, eventos realizados, comunicados enviados, registros de execução publicados, taxa de resposta a perguntas, tendência de opinião, taxa de leitura, taxa de confirmação eventos
- Filtros por período

**LOW: Compliance**
- Termo de Responsabilidade da Gestão (S29): texto jurídico, checkbox, registro IP/timestamp/user
- Declaração Anual (S30): obrigatória a cada 12 meses ou novo mandato, alerta no dashboard
- Auditoria (S31): imutável, 15+ tipos de ação monitorada, card com data/hora/usuário/tipo/descrição

**LOW: Avaliação da Gestão**
- Obrigatória bimestralmente (M12)
- 3 perguntas fixas, escala 1-10
- Não mostra autor, registra apenas estatística
- Síndico vê resultado consolidado, moradores não veem avaliações de outros

#### 3.2.4 MID: Assembly Engine

🔴 **CONFLITO DE ESCOPO — PENDENTE RESOLUÇÃO COM CLIENTE**

O módulo de Assembleia descrito nos documentos novos é significativamente mais complexo que o escopo original. Inclui:

**Pré-Assembleia:**
- Criar/configurar assembleia (tipo, data, convocação, modalidade)
- Vincular administradora (token de acesso)
- Cadastrar pauta (itens, quórum, votação, anexos)
- Anexar edital
- Notificação moradores → ciência → confirmação participação
- Enquetes preliminares
- Cadastro de procuração
- Análise de fornecedores
- Painel administradora: importar fração ideal, registrar inadimplência, validar procurações
- Simulador de quórum

**Dia da Assembleia (ao vivo):**
- Check-in: app, QR Code, terminal de apoio
- Mesa diretora: presidente, secretários, administradora, auditor, fornecedores
- Vídeo institucional de abertura
- Condução de pauta com item atual
- Controle de fala: levantar mão → fila → autorização → cronômetro
- Votação: sim/não/abstenção OU escala 1-5 + abstenção
- Procuração: voto replicado considerando fração ideal
- Telão: pauta, item, unidades presentes/ausentes, fila de fala, cronômetro, quórum, votos
- Homologação (síndico ou administradora)
- Finalização item: resolvido, adiado, não resolvido, prejudicado

**Pós-Assembleia:**
- Registro final (data, mesa, presentes, votos, procurações, inadimplentes)
- Relatórios: presença, votos, procurações, auditoria, decisões
- Alimenta Linha do Tempo

🔴 **PENDENTE:** Definir se assembleia ao vivo entra no escopo contratual atual ou é escopo adicional
🔴 **PENDENTE:** Se entrar, requer WebSocket/real-time, o que impacta toda a infraestrutura

### 3.3 HIGH-DOMAIN: Commercial

**Função:** tudo que cruza fronteiras entre tenants. Relações empresa↔condomínio, empresa↔empresa, empresa↔agência.

**Mid-domains identificados:**

#### 3.3.1 MID: Connect Me

Marketplace de contato unidirecional com LGPD by design.

**Fluxo do Síndico (S18-S19):**
- Criar solicitação: categoria, subcategoria, descrição da necessidade, cidade, bairro
- Opção de compartilhar dados de contato mediante interesse

**Fluxo da Empresa (E2-E4):**
- Recebe como "Oportunidade"
- Tenho Interesse → revela dados do síndico (nome, telefone, email, descrição)
- Recusar → motivo estruturado obrigatório (6 opções + justificativa)
- Status interno: em negociação, encerrada

**Fluxo do Morador → Síndico:**
🔴 **PENDENTE:** Detalhamento no documento novo (existia no escopo antigo com cotas 2/ano base, 4/ano pagante)

**Regras:**
- Dados do síndico só revelados após "Tenho Interesse"
- Recusa envia mensagem automática ao síndico
- Responder em até 48h ou move para recusadas
- Marketing Link: formulário empresa→agência (6 tipos de apoio, 3 níveis de prioridade)

**Direções confirmadas:** Síndico→Empresa, Empresa→Síndico (recebe), Empresa→Agência (Marketing Link)
🔴 **PENDENTE:** Confirmar se Morador→Síndico e Empresa→Empresa continuam na nova versão

#### 3.3.2 MID: Service & Contract

Fluxo de prestação de serviço entre empresa e condomínio.

**Registrar Execução (E5):**
- Campos: condomínio, tipo serviço, descrição, área impactada, natureza, status, data, garantia, orientações técnicas, materiais, fotos/vídeos
- Status: rascunho → enviado para validação → validado OU ajuste OU rejeitado → publicado na LT

**Atividade Técnica (E6):**
- Empresa contribui com informações técnicas relevantes
- Campos: tipo, descrição, área impactada, importância, risco, impacto, recomendação, anexos
- Também passa por validação do síndico

**Comunicados Técnicos (E7):**
- Orientações pós-serviço para moradores/gestores
- Passa por validação do síndico

**Status de Envios (E8-E9):**
- Hub: registros de execução, atividades técnicas, comunicados
- Status: rascunho, enviado, aguardando validação, solicitado ajuste, aprovado, publicado, rejeitado
- Empresa pode editar e reenviar

**Validação do lado do Síndico (S21-S22, S26):**
- Validar, solicitar ajuste ou rejeitar (justificativa obrigatória)
- Após validação: vira publicação na Linha do Tempo
- Se vinculado ao PD: atualiza status da ação automaticamente

#### 3.3.3 MID: Institutional Profile

Perfil público da empresa dentro da plataforma.

**Campos (E11):** razão social, nome fantasia, CNPJ, cidade/estado, telefone, email, site, logo, descrição, segmento, subcategoria
- Visível por síndicos

**Usuários da Empresa (E12):**
- Administrador: gestão completa
- Operador Técnico: registrar execução, atividade técnica, comunicados, publicar vídeos

**Gestão de Agência (E13-E15):**
- Convidar agência via token
- Gerenciar acessos
- Encerrar acesso a qualquer momento

#### 3.3.4 MID: Marketing Agency

Operação da agência de marketing dentro do ecossistema.

**Painel da Agência (MK1-MK8):**
- Empresas atendidas (aparecem após aceitar convite)
- Gerenciar empresa: publicar vídeo, biblioteca, dashboard por empresa
- Publicar vídeo: 12 tipos de vídeo institucional, subtema automático por segmento
- Biblioteca: título, tipo, data, visualizações; ocultar não remove histórico
- Dashboard por empresa: total vídeos, views, mais visto, tempo médio, taxa retenção
- Dashboard geral da agência: empresas atendidas, total vídeos, total views, mais acessado, empresa com maior engajamento
- Marketing Link recebido: lista de solicitações de contato

**Regra de visibilidade de vídeos:** moradores NÃO acessam vídeos institucionais normalmente. Apenas em processo de escolha de fornecedor no módulo assembleia. Após votação, acesso bloqueado.

### 3.4 HIGH-DOMAIN: Content

**Função:** camada de distribuição de conteúdo que serve a todos os outros domínios. Mídia, player, busca, visibilidade.

🔴 **NOTA:** Este high-domain utiliza definições do escopo original e da Matriz Funcional anterior. Documentos novos ainda não detalharam esta camada.

#### 3.4.1 MID: Media

- Upload com trava temporal (90 dias para vídeos institucionais e currículos)
- Integração com Mux ou Cloudflare Stream
- Lógica de paywall: usuário Base vê 25% dos vídeos de empresa
- Contagem de visualização: >70% assistido para pontuar
- Limites de armazenamento por plano (Plus: 100, Pro: 150)
- Limites de upload mensal por plano (N2: 4/mês, Plus: 8/mês, Pro: 12/mês)

🔴 **PENDENTE:** Confirmar se regras de paywall e limites mudaram na nova Matriz Funcional

#### 3.4.2 MID: Search

- Motor de busca com filtros: categoria de serviço, CEP, tipo de ator
- Middleware de visibilidade: perfil só aparece se plano permitir
- Busca geral retorna mix (síndicos, empresas, comércio local)
- Perfil do Morador Pagante indexável na busca para empresas

🔴 **PENDENTE:** Confirmar se regras de busca mudaram na nova versão

#### 3.4.3 MID: Video Curriculum (Banco de Talentos)

- Upload de vídeo-currículo (até 90 segundos)
- Trava trimestral: atualização a cada 90 dias
- Fixo no perfil (não é galeria)
- Ficha cadastral profissional
- Visível para empresas (Plus, Pro, Marketing, Vizinhança)
- Botão "Demonstrar Interesse"

🔴 **PENDENTE:** Confirmar se continua no escopo novo

#### 3.4.4 MID: Player

- Player de vídeo customizado
- Player nativo otimizado para mobile
- Paywall: corte de stream aos 25% para usuário Base em vídeos de empresa
- Vídeos de síndicos: ilimitado para todos

🔴 **PENDENTE:** Confirmar se arquitetura do player mudou

---

## 4. CONEXÕES ENTRE HIGH-DOMAINS (A REDE)

```
Identity ──→ Institutional : "esse usuário tem contexto nesse condomínio?"
Identity ──→ Commercial    : "esse plano permite essa ação comercial?"
Identity ──→ Content       : "esse plano permite ver esse conteúdo?"

Institutional ──→ Commercial : "esse condomínio precisa de serviço, essa empresa oferece"
Commercial ──→ Institutional : "essa empresa enviou registro, síndico valida"

Content ──→ Identity       : "quem pode ver resolve com o plano do usuário"
Content ──→ Commercial     : "vídeo institucional da empresa no processo de fornecedor"
Content ──→ Institutional  : "vídeo/áudio por pauta na assembleia"
```

### 4.1 Fluxos Cross-Domain Confirmados

**Fluxo de Execução de Serviço:**
```
Síndico cria atividade [Institutional]
  → Empresa executa serviço [externo]
    → Empresa registra execução [Commercial]
      → Síndico valida [Institutional]
        → Registro publicado na Linha do Tempo [Institutional]
          → Se vinculado ao PD, status da ação atualiza [Institutional]
```

**Fluxo de Atividade Técnica da Empresa:**
```
Empresa identifica necessidade [Commercial]
  → Empresa cria atividade técnica [Commercial]
    → Síndico valida [Institutional]
      → Publicação na Linha do Tempo [Institutional]
```

**Fluxo Connect Me (Síndico → Empresa):**
```
Síndico cria solicitação [Commercial/Connect Me]
  → Empresa recebe como Oportunidade [Commercial]
    → "Tenho Interesse" → dados revelados [Identity + LGPD]
    → OU "Recusar" → motivo + notificação [Commercial]
```

**Fluxo Marketing Link:**
```
Empresa acessa perfil da agência [Commercial/Institutional Profile]
  → Preenche formulário Marketing Link [Commercial]
    → Agência recebe solicitação [Commercial/Marketing Agency]
      → Contato externo
        → Empresa convida agência [Commercial/Institutional Profile]
          → Agência publica vídeos [Content/Media]
```

---

## 5. EXTERNAL SERVICES

Serviços que os domínios internos **consomem** mas nunca **são**:

| Serviço | Provider Indicado | Quem Consome | Função |
|---------|-------------------|-------------|--------|
| Video Streaming | Mux ou Cloudflare Stream | Content/Media | Upload, encoding, streaming |
| Payment Gateway | Mercado Pago | Identity/Billing | Assinaturas, cobrança mensal/anual |
| Email Transacional | AWS SES ou SendGrid | Identity, Commercial | Validação conta, Connect Me, broadcasts |
| SMS | Twilio ou Zenvia | Identity | OTP, verificação |
| Object Storage | S3 ou equivalente | Content, Institutional | PDFs, imagens, anexos, documentos |
| CEP Lookup | ViaCEP ou similar | Institutional/Registry | Auto-fill endereço no cadastro de condomínio |

> **Decisão arquitetural:** cada domínio interno não deveria falar direto com providers. Adapter layer que abstrai integrações — troca de provider não impacta domínios.

🔴 **PENDENTE:** Se assembleia ao vivo entrar no escopo, será necessário WebSocket service (Socket.io, Ably, Pusher ou similar) para real-time

---

## 6. STACK TECNOLÓGICA DO FRONTEND

| Camada | Tecnologia | Versão |
|--------|-----------|--------|
| Framework UI | SolidJS | ^1.9.10 |
| Build Tool | Rsbuild | ^1.7.1 |
| Roteamento | @tanstack/solid-router | ^1.157.16 |
| Data Fetching | @tanstack/solid-query | ^5.90.23 |
| Formulários | @tanstack/solid-form + @tanstack/zod-form-adapter | ^1.28.0 / ^0.42.1 |
| Componentes Base | @kobalte/core | ^0.13.11 |
| OTP | @corvu/otp-field | ^0.1.4 |
| CSS Engine | UnoCSS (postcss + attributify + icons + typography + web-fonts) | ^66.6.0 |
| Validação | Zod | ^4.3.6 |
| HTTP Client | Axios | ^1.13.4 |
| Variants | class-variance-authority | ^0.7.1 |
| Toasts | solid-sonner | ^0.2.8 |
| Ícones | @iconify/json + logos + vscode-icons | — |
| Linter/Formatter | Biome | 2.3.8 |
| TypeScript | typescript | ^5.9.3 |
| Schemas Compartilhados | mastersindico-schemas (workspace:*) | monorepo |

### 6.1 Autenticação (do index.tsx existente)

- AuthProvider como contexto global
- useAuthGuard hook para proteção de rotas
- Router recebe `context.auth` para guards em `beforeLoad`
- ColorModeProvider do Kobalte para dark/light mode
- QueryClient global via QueryContext

---

## 7. DESIGN SYSTEM — TOKENS CSS (IMUTÁVEIS)

> ⚠️ **Paleta de cores, identidade visual e consistência de marca NÃO podem ser alterados.**

### 7.1 Cores (OKLCH)

| Token | Light | Dark | Uso |
|-------|-------|------|-----|
| `--primary` | oklch(0.715 0.120 84.2) | mesmo | Gold CTA |
| `--secondary` | oklch(0.218 0.055 256.1) | oklch(0.274 0.077 256.3) | Navy |
| `--accent` | oklch(0.871 0.129 82.0) | mesmo | Gold Light |
| `--surface` | oklch(0.218 0.055 256.1) | mesmo | Navy (sidebar, topbar, footer) |
| `--destructive` | oklch(0.568 0.200 26.4) | oklch(0.505 0.190 27.5) | Erros |
| `--success` | oklch(0.627 0.170 149.2) | mesmo | Confirmações |
| `--warning` | oklch(0.769 0.165 70.1) | mesmo | Alertas |

### 7.2 Paleta HEX (Identidade Visual)

- Gold escuro: #C69C3F
- Gold claro: #FFCC6A
- Navy escuro: #071A33
- Navy médio: #0A274C

### 7.3 Tipografia

- **Body:** Manrope (sans-serif)
- **Headings:** Michroma (sans-serif) — letter-spacing: 0.05em
- **Border-radius:** 0.5rem (`--radius`)

---

## 8. INVENTÁRIO DE TELAS POR JORNADA

### 8.1 Jornada do Morador (15 telas)

| Tela | Nome | Módulo |
|------|------|--------|
| M1 | Painel do Morador | Home |
| M2 | Selecionar Condomínio | Navegação |
| M3 | Buscar Condomínio | Onboarding |
| M4 | Cadastro da Unidade | Onboarding |
| M5 | Home do Condomínio | Home |
| M6 | Linha do Tempo | Governança (leitura) |
| M7 | Detalhe da Publicação | Governança (leitura) |
| M8 | Plano Diretor | Governança (leitura) |
| M9 | Detalhe da Ação | Governança (leitura) |
| M10 | Eventos | Eventos |
| M11 | Comunicados | Comunicados |
| M12 | Avaliar Gestão | Avaliação |
| M13 | Meu Vínculo | Membership |
| M14 | Gerenciar Acessos | Membership |
| M15 | Encerrar Vínculo | Membership |

### 8.2 Jornada do Síndico (31 telas)

| Tela | Nome | Módulo |
|------|------|--------|
| S1 | Entrada da Governança | Onboarding |
| S2 | Meus Condomínios | Navegação |
| S3 | Cadastrar Condomínio | Registry |
| S4 | Dados da Gestão | Registry |
| S5 | Cadastro Empresa Síndica | Registry |
| S6 | Home da Governança | Home |
| S7 | Plano Diretor | PD |
| S8 | Cadastrar Ação PD | PD |
| S9 | Lista Ações PD | PD |
| S10 | Detalhe Ação PD | PD |
| S11 | Linha do Tempo | LT |
| S12 | Criar Atividade | LT |
| S13 | Editar Atividade | LT |
| S14 | Histórico LT | LT |
| S15 | Eventos | Eventos |
| S16 | Criar Evento | Eventos |
| S17 | Lista Eventos | Eventos |
| S18 | Connect Me | Connect Me |
| S19 | Criar Connect Me | Connect Me |
| S20 | Connect Me Recebido | Connect Me |
| S21 | Registros de Execução | Validações |
| S22 | Detalhe Registro Execução | Validações |
| S23 | Comunicados | Comunicados |
| S24 | Criar Comunicado | Comunicados |
| S25 | Lista Comunicados | Comunicados |
| S26 | Validações Pendentes | Validações |
| S27 | Dashboard | Dashboard |
| S28 | Compliance | Compliance |
| S29 | Termo de Responsabilidade | Compliance |
| S30 | Declaração Anual | Compliance |
| S31 | Auditoria | Compliance |

### 8.3 Jornada da Empresa (16 telas)

| Tela | Nome | Módulo |
|------|------|--------|
| E1 | Painel Empresarial | Home |
| E2 | Oportunidades (Connect Me) | Connect Me |
| E3 | Recusar Oportunidade | Connect Me |
| E4 | Detalhe da Oportunidade | Connect Me |
| E5 | Registrar Execução | Service |
| E6 | Atividade Técnica | Service |
| E7 | Comunicados Técnicos | Service |
| E8 | Status de Envios | Service |
| E9 | Detalhe do Registro | Service |
| E10 | Dashboard Empresarial | Dashboard |
| E11 | Perfil Institucional | Profile |
| E12 | Usuários da Empresa | Profile |
| E13 | Gestão de Agência | Marketing |
| E14 | Convidar Agência | Marketing |
| E15 | Gerenciar Acessos Agência | Marketing |
| E16 | Formulário Marketing Link | Marketing |

### 8.4 Jornada da Agência de Marketing (8 telas)

| Tela | Nome | Módulo |
|------|------|--------|
| MK1 | Painel da Agência | Home |
| MK2 | Empresas Atendidas | Gestão |
| MK3 | Gerenciar Empresa | Gestão |
| MK4 | Publicar Vídeo | Content |
| MK5 | Biblioteca de Vídeos | Content |
| MK6 | Dashboard da Empresa | Dashboard |
| MK7 | Dashboard Geral | Dashboard |
| MK8 | Marketing Link Recebido | Marketing |

### 8.5 Módulo Assembleia

🔴 **PENDENTE RESOLUÇÃO DE ESCOPO** — Telas não numeradas, diagrama de fluxo apenas

---

## 9. LISTAS MESTRE (ENUMS DO SISTEMA)

Dados estruturados que alimentam selects/dropdowns em todo o sistema:

### 9.1 Área Impactada (26 itens)
Estrutura predial, Sistema hidráulico, Sistema elétrico, Sistema de incêndio, Reservatórios, Elevadores, Garagem, Fachada, Cobertura, Segurança patrimonial, Administração, Portaria, Playground, Academia, Salão de festas, Corredores, Jardins, Casa de máquinas, Rede de esgoto, Rede de drenagem, Sistema de gás, Iluminação, Sistema de câmeras, Controle de acesso, Áreas comuns, Outros

### 9.2 Natureza da Ação (8 itens)
Preventiva, Corretiva, Adequação normativa, Melhoria estrutural, Monitoramento, Modernização, Investigação técnica, Regularização legal

### 9.3 Nível de Importância (5 itens)
Baixa, Moderada, Alta, Crítica, Emergencial

### 9.4 Impacto Esperado (10 itens)
Redução de risco, Adequação legal, Melhoria da infraestrutura, Preservação patrimonial, Aumento da segurança, Melhoria operacional, Economia financeira, Melhoria da qualidade de vida

### 9.5 Tipos de Atividade (26 itens)
Manutenção preventiva, Manutenção corretiva, Inspeção técnica, Vistoria técnica, Contratação de serviço, Execução de serviço, Reparo emergencial, Obra de melhoria, Adequação normativa, Atualização de infraestrutura, Monitoramento ambiental, Revisão técnica, Auditoria técnica, Limpeza técnica, Controle de pragas, Limpeza de reservatórios, Treinamento técnico, Adequação legal, Revisão de equipamentos, Atualização administrativa, Contratação de fornecedor, Encerramento de contrato, Avaliação de fornecedor, Implantação de sistema, Atualização de normas, Ação emergencial

### 9.6 Risco Associado (9 itens)
Sem risco identificado, Risco operacional, Risco estrutural, Risco sanitário, Risco elétrico, Risco hidráulico, Risco de incêndio, Risco jurídico, Risco ambiental

### 9.7 Próxima Ação (8 itens)
Monitorar evolução, Nova inspeção, Contratar fornecedor, Executar manutenção, Atualizar plano diretor, Registrar conclusão, Acompanhar garantia, Reavaliar necessidade

### 9.8 Status do Plano Diretor (7 itens)
Planejada, Em contratação, Em execução, Concluída, Suspensa, Reprogramada, Atrasada

### 9.9 Tipos de Evento (13 itens)
Assembleia ordinária, Assembleia extraordinária, Manutenção programada, Inspeção técnica, Obra programada, Interrupção programada de serviços, Treinamento de equipe, Campanha institucional, Evento comunitário, Reunião administrativa, Atualização normativa, Simulado de emergência, Outros

### 9.10 Tipos de Comunicado (8 itens)
Aviso operacional, Interrupção programada, Orientação aos moradores, Aviso de segurança, Comunicado institucional, Conclusão de serviço, Recomendação de manutenção, Alerta preventivo

### 9.11 Tipos de Condomínio (6 itens)
Residencial, Comercial, Misto, Shopping center, Galeria comercial, Edifício garagem

### 9.12 Tipos de Mandato (3 itens)
Síndico eleito, Síndico profissional, Síndico interino

### 9.13 Tipos de Unidade (3 itens)
Residencial, Comercial, Vaga de garagem

### 9.14 Tipos de Vínculo do Morador (3 itens)
Proprietário residente, Proprietário não residente, Inquilino

### 9.15 Status da Unidade (3 itens)
Ocupada pelo proprietário, Ocupada por inquilino, Unidade vazia

### 9.16 Motivos de Encerramento de Vínculo (4 itens)
Mudança definitiva, Venda da unidade, Encerramento de contrato de locação, Erro no cadastro

### 9.17 Motivos de Recusa Connect Me — Empresa (6 itens)
Fora da área de atendimento, Serviço fora da especialidade, Agenda indisponível, Conflito com serviços agendados, Orçamento incompatível, Outro motivo

### 9.18 Tipos de Vídeo Institucional (12 itens)
Apresentação institucional, Apresentação da equipe técnica, Demonstração de serviço, Explicação técnica, Boas práticas condominiais, Orientação preventiva, Caso real de atendimento, Treinamento técnico, Explicação de legislação/normas, Dicas de manutenção, Conteúdo educativo para síndicos, Conteúdo educativo para moradores

### 9.19 Tipos de Apoio Marketing (6 itens)
Produção de vídeos institucionais, Gestão de conteúdos técnicos, Estratégia de posicionamento, Gestão de redes sociais, Consultoria de marketing, Outro

---

## 10. PENDÊNCIAS CONSOLIDADAS

### 🔴 Documentos que faltam

| Item | Impacto | Bloqueia |
|------|---------|----------|
| Matriz Funcional nova | Define quem pode o quê por plano — impacta TODOS os domínios | Middleware de rotas, visibilidade, paywall |
| Painel Admin MasterSíndico | Define o domínio que opera por cima de tudo | Hierarquia dos high-domains |
| Jornada da Vizinhança / Comércio Local | Define se é high-domain ou mid-domain | Classificação de domínios |
| Detalhamento do LMS | Define se Content é high-domain ou precisa de split | Classificação de domínios |

### 🔴 Decisões pendentes com cliente

| Decisão | Contexto |
|---------|----------|
| Assembleia ao vivo vs assíncrona | Escopo original dizia assíncrona; documento novo descreve ao vivo com real-time. Muda infra, prazo e custo. |
| Escopo do módulo Assembleia em qual sprint | Não estava nas Sprints 1-2 originais; é complexidade de produto inteiro |
| Detalhamento dos planos por persona | Quais planos existem para morador, síndico, empresa — preço, features por plano |
| Fluxo completo de onboarding por tipo | Campos de cadastro específicos por persona |
| Regras de paywall e limites na nova versão | 25% preview, limites de upload, armazenamento — confirmam ou mudaram? |
| Morador→Síndico Connect Me | Continua existindo na nova versão? Com quais cotas? |
| Empresa→Empresa Connect Me | Continua existindo na nova versão? |
| Banco de Talentos / Vídeo-currículo | Continua no escopo? |

### 🔴 Decisões arquiteturais internas

| Decisão | Opções | Depende de |
|---------|--------|-----------|
| Modular Monolith vs Microservices | OTP Pattern com bounded contexts claros vs deploy monolítico inicial | Tamanho da equipe, orçamento, prazo |
| WebSocket service | Necessário se assembleia ao vivo entrar no escopo | Decisão de escopo da assembleia |
| Estratégia de real-time | Polling vs WebSocket vs SSE para validações pendentes, dashboard | Requisitos de latência |

---

## 11. PRÓXIMOS PASSOS

1. Receber documentos pendentes (Matriz Funcional, Admin, Vizinhança, LMS)
2. Resolver conflito de escopo da Assembleia com cliente
3. Finalizar hierarquia de high/mid/low domains
4. Mapear contratos entre domínios (API contracts)
5. Definir estratégia de deployment (monolith first vs microservices)
6. Definir cronograma de entregas por domínio (substituir sprints lineares)
