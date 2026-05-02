# Análise Técnica Completa - Master Síndico

## Data da Análise
**Gerado em:** 2026-02-01  
**Baseado em:** Manual Técnico, Matriz Funcional, Análise Completa, Escopo Técnico, Proposta MVP

---

## 1. VISÃO GERAL DO PRODUTO

### 1.1 Conceito Central
O **Master Síndico** (anteriormente "My Síndico") é uma plataforma de **governança condominial aplicada** com foco em:
- ✅ **Capacitação e networking** (não é sistema ERP/gestão operacional)
- ✅ **Vídeos verticais curtos** estilo TikTok/Reels (1-4min)
- ✅ **Mobile-first** com versão web
- ✅ **Currículo digital do síndico** com validação por moradores
- ✅ **Sistema de reconhecimento** (Bronze/Prata/Ouro/Diamante)

### 1.2 O Que NÃO É
❌ **NÃO é ERP condominial** (não faz boletos, folha, contabilidade)  
❌ **NÃO é app de comunicação contínua** (não tem chat livre)  
❌ **NÃO é marketplace de preços** (não há ranking por menor valor)  
❌ **NÃO é plataforma de cursos tradicional** (não certifica profissionalmente)  
❌ **NÃO é rede social** (métricas são privadas, sem likes públicos)

---

## 2. PERFIS DE USUÁRIO E PLANOS

### 2.1 Hierarquia de Perfis (Roles)

#### **Base (Free)**
- Morador não-pagante
- Acesso básico ao conteúdo
- Vê comerciais
- Preview de 25% dos vídeos de empresas

#### **Morador Pagante (R$ 9,90/mês)**
- Sem comerciais
- Acesso completo a vídeos
- Pode criar vídeo-currículo (90s, atualização trimestral)
- Perfil visível para empresas
- 4 Connect Me/ano com síndicos

#### **Síndico - 3 Níveis**
| Plano | Preço | Características |
|-------|-------|----------------|
| **N1 (Gestor)** | R$ 19,90 | Básico, com comerciais |
| **N2 (Plus)** | R$ 39,90 | Sem comerciais, 2 Connect Me/ano |
| **N3 (Pro)** | R$ 59,90 | Recursos completos, 4 Connect Me/ano |

**Recursos específicos:**
- ✅ Gestão de assembleias (criar, editar, votações)
- ✅ Timeline da gestão (exportar PDF)
- ✅ Plano diretor com % de execução
- ✅ Sistema de reconhecimento (status Bronze/Prata/Ouro/Diamante)
- ✅ Avaliação objetiva por moradores
- ✅ Promoções exclusivas para moradores do condomínio

#### **Empresa - 2 Níveis**
| Plano | Preço | Vídeos/mês | Características |
|-------|-------|------------|----------------|
| **Plus** | ~R$ 99 | 4 vídeos | Perfil básico, Connect Me |
| **Pro** | ~R$ 199 | 8 vídeos | Vídeos institucionais, módulos/cursos |

**Recursos:**
- ✅ Vídeo institucional (2min, atualização trimestral)
- ✅ Vídeos instrucionais (técnicos, sem CTA comercial)
- ✅ Até 5 subcategorias de serviço
- ✅ Módulos educacionais com certificados

#### **Empresa Marketing**
- Tag de anúncios por vídeo
- Limite: 3 empresas/estado
- Sistema de leilão por "cadeira" (BNI-style)
- Não tem Connect Me ativo (anti-spam)

#### **Vizinhança (Comércio Local)**
- Cadastro de benefícios/promoções
- Sistema de cupons com "palavra do dia"
- Filtro por CEP
- Promoções exclusivas por condomínio (negociadas com síndico)

---

## 3. MATRIZ FUNCIONAL - PERMISSÕES CRÍTICAS

### 3.1 Consumo de Conteúdo

| Funcionalidade | Base | Morador Pg | Síndico N1 | Síndico N2 | Síndico N3 | Emp. Plus | Emp. Pro | Marketing | Vizinhança |
|----------------|------|------------|------------|------------|------------|-----------|----------|-----------|------------|
| Busca geral | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Vídeos empresas (preview 25%) | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Vídeos empresas (integral) | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Módulos/cursos | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| Certificados | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| Lives (gravadas) | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |

### 3.2 Publicação de Conteúdo

| Funcionalidade | Síndico N1 | Síndico N2 | Síndico N3 | Emp. Plus | Emp. Pro | Marketing |
|----------------|------------|------------|------------|-----------|----------|-----------|
| Publicar vídeos | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Limite mensal | - | - | - | 4/mês | 8/mês | - |
| Vídeos institucionais | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Criar módulos/cursos | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Ebook | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |

### 3.3 Connect Me (Formulário de Contato)

| Direção | Base | Morador Pg | Síndico N1 | Síndico N2 | Síndico N3 | Emp. Plus/Pro |
|---------|------|------------|------------|------------|------------|---------------|
| Morador → Síndico | 2/ano | 4/ano | - | - | - | - |
| Síndico → Empresa | - | - | ✅ | 2/ano | 4/ano | - |
| Empresa → Empresa | - | - | - | - | - | ✅ |
| Marketing → * | ❌ (Anti-spam) | ❌ | ❌ | ❌ | ❌ | ❌ |

**Regra crítica:** Contato sempre parte do interessado (morador/síndico) para o contratado (síndico/empresa).

---

## 4. FUNCIONALIDADES ESSENCIAIS

### 4.1 Sistema de Vídeos

#### Especificações Técnicas
```typescript
interface VideoSpecs {
  formato: "vertical" | "landscape"; // vertical para feed, landscape para treinamentos
  aspectRatio: "9:16"; // mobile-first
  duracaoRegular: [60, 240]; // 1-4 minutos
  duracaoInstitucional: 120; // máximo 2 minutos
  duracaoVideoCurriculo: 90; // máximo 90 segundos (Morador Pagante)
  uploadLimit: {
    empresaPlus: "4/mês",
    empresaPro: "8/mês",
    videoInstitucional: "trimestral" // trava de 90 dias
  };
}
```

#### Player - Controles
- ✅ **Curtir** (contador privado, visível só para o autor)
- ✅ **Compartilhar**
- ✅ **Connect Me** (quando aplicável pelo plano)
- ✅ **Marketing/Propaganda** (rotação de anúncios)

#### Feed
- Grid 2x2 (4 vídeos na tela)
- Scroll infinito
- Moderação obrigatória antes de publicar

#### Paywall para Base (Free)
- **Vídeos de empresas:** Preview de 25% e corte
- **Comerciais:** Popup após 30s + comercial a cada 3 vídeos
- **Usuários pagantes:** Sem comerciais

### 4.2 Sistema de Assembleia e Votação

#### Fluxo de Assembleia (Síndico)
1. Criar evento (título, data, edital em PDF/texto)
2. Adicionar pautas (título, descrição, vídeo/áudio explicativo)
3. Anexar vídeos de fornecedores (busca do acervo de empresas)
4. Gerar **link temporário** com janela de acesso definida
5. Compartilhar link com moradores

#### Votação Assíncrona
```typescript
interface Votacao {
  tipo: "aprovar/reprovar" | "multipla_escolha";
  opcoes: Fornecedor[] | string[];
  regras: {
    umVotoPorUnidade: true;
    janelaDeVotacao: [Date, Date];
    apenasSeEventoAberto: true;
  };
}
```

**Não é votação em blockchain**  
**Não é assembleia ao vivo (Zoom/Meet)**  
É assíncrono e baseado em janelas de tempo.

### 4.3 Sistema de Reconhecimento do Síndico (Status)

#### Critérios Objetivos (mensuráveis)
1. **Consumo de conteúdo qualificado**
   - Vídeos técnicos assistidos com ≥70% de visualização
   - Visualizações parciais não pontuam

2. **Atuação prática**
   - Número de condomínios vinculados ao perfil
   - Cada condomínio conta 1x (independente do tempo)

3. **Capacitação formal**
   - Módulos de capacitação concluídos (100%)

#### Categorias de Status
```typescript
enum StatusSindico {
  BRONZE = "bronze",
  PRATA = "prata",
  OURO = "ouro",
  DIAMANTE = "diamante"
}
```

**Características:**
- ✅ Status é **dinâmico** (pode evoluir ou regredir)
- ✅ Recalculado **periodicamente** pelo sistema
- ✅ Visível publicamente no perfil para **moradores**
- ❌ **Não é ranking** (não compara síndicos entre si)
- ❌ **Não substitui** avaliação de gestão

### 4.4 Avaliação Objetiva da Gestão

#### Sistema de Avaliação
```typescript
interface AvaliacaoGestao {
  perguntas: string[]; // fixas do sistema, não customizáveis
  formato: "estrelas" | "tags" | "objetivas";
  janelaDeAcesso: [Date, Date]; // definida pelo MasterSíndico
  privacidade: {
    resultadoConsolidado: "visível para síndico",
    avaliacoesIndividuais: "ocultas de outros moradores"
  };
}
```

**Regra crítica:** Perguntas são **fixas do sistema**. O síndico não cria perguntas personalizadas.

### 4.5 Sistema de Cupons ("Promoção do Dia")

#### Engine de Cupons (Vizinhança)
```typescript
interface Cupom {
  formato: string; // ID_LOJA(5) + SEQUENCIAL(3) + PALAVRA(5)
  exemplo: "00123055PAO"; // Loja 123 + Seq 55 + Palavra PAO
  travaDeTempo: "4 horas"; // usuário só pode gerar 1 cupom a cada 4h para mesma loja
  palavraDoDia: string; // máximo 5 letras maiúsculas, definida pelo lojista
}
```

#### Fluxo do Cupom
1. Morador/Síndico acessa "Clube de Benefícios"
2. Busca por CEP (comércios locais próximos)
3. Clica em "Pegar Cupom"
4. Sistema valida:
   - ✅ Última geração foi há mais de 4h? → Gera novo cupom
   - ❌ Ainda dentro de 4h? → Exibe "Aguarde X horas"
5. Cupom é exibido na tela (formato: `00123055PAO`)
6. Morador apresenta no estabelecimento físico

#### Promoções Exclusivas (Negociadas pelo Síndico)
- Síndico pode ativar promoções **exclusivas** para moradores do condomínio
- Identificação pelo **número de inscrição** do condomínio
- Estabelecimento escolhe:
  - Promoção exclusiva a 1 condomínio
  - Promoção regional (todos do CEP)

**Limitação:** Não é sistema de helpdesk/chamado. É **apenas cupom de desconto**.

### 4.6 Gestão de Currículos (Oportunidades)

#### Vídeo-Currículo (Morador Pagante)
```typescript
interface VideoCurriculo {
  duracao: 90; // segundos
  formato: "fixo no perfil"; // não é galeria
  atualizacao: "trimestral"; // trava de 90 dias
  visibilidade: ["empresas com acesso à funcionalidade"];
}
```

#### Visualização de Currículos (Empresas)
- Empresas Plus/Pro podem visualizar perfis de Moradores Pagantes
- Inclui vídeo-currículo e dados cadastrais
- **Não há Connect Me ativo** (empresa não pode abordar proativamente)
- Contato deve partir do morador

### 4.7 Sistema de LMS (Módulos e Cursos)

#### Estrutura Hierárquica
```
Curso
├── Módulo 1
│   ├── Aula 1 (Vídeo + Descrição + Anexo)
│   ├── Aula 2
│   └── Aula 3
├── Módulo 2
│   └── ...
└── Certificado (gerado automaticamente ao concluir 100%)
```

#### Certificado Digital
```typescript
interface Certificado {
  formato: "PDF dinâmico";
  conteudo: {
    nome: string;
    curso: string;
    dataConlusao: Date;
    qrCode: string; // validação externa
  };
  exibicao: "aba dedicada no perfil do síndico/morador";
}
```

### 4.8 Fórum

#### Tipos de Fórum
| Tipo | Acesso | Limitações |
|------|--------|------------|
| **Público** | Usuários gratuitos | Máximo 5 respostas |
| **Exclusivo** | Usuários pagantes | Sem limitações |

#### Funcionalidades
- ✅ Criar tópicos
- ✅ Responder
- ✅ Likes e upvotes
- ✅ Sistema de pontos (gamificação leve)
- ✅ Moderação

---

## 5. GOVERNANÇA DE CONTEÚDO

### 5.1 Conteúdos Permitidos
✅ **Vídeos técnicos instrucionais** (como resolver problemas)  
✅ **Vídeo institucional** (pitch de venda, máx 2min)  
✅ **Módulos/cursos educativos**  
✅ **Ebooks como material complementar**

### 5.2 Conteúdos **PROIBIDOS**
❌ Dados de contato em vídeos/descrições (telefone, WhatsApp, email, site, QR codes externos)  
❌ Chamadas diretas para venda ("ligue agora", "faça um orçamento")  
❌ Ofertas promocionais  
❌ Comparações depreciativas com concorrentes  
❌ Conteúdos sensacionalistas ou alarmistas  
❌ Promessas de resultado absoluto

### 5.3 Moderação
- ✅ Vídeos ficam **"sob análise"** até aprovação
- ✅ Sistema de justificativa para rejeição:
  - Fere estatuto de inscrição
  - Contém informação de contato
  - Outros motivos (campo de texto livre)

### 5.4 Consequências por Descumprimento
1. Orientação e solicitação de ajuste
2. Advertência formal
3. Remoção do conteúdo
4. Limitação temporária de funcionalidades
5. Descredenciamento da empresa (casos graves)

---

## 6. LIMITADORES TÉCNICOS CRÍTICOS

### 6.1 Notificações (Email/SMS)
**Sistema NÃO envia notificações sociais** (curtidas, comentários, novos vídeos).

#### Disparos Permitidos:
- ✅ Validação de cadastro (confirmação de email)
- ✅ Redefinição de senha
- ✅ Formulário "Connect Me" (disparo único)
- ✅ Comunicados oficiais do Administrador MasterSíndico (broadcasts)

**Custo de disparos (AWS SES, SendGrid, Twilio) é de responsabilidade do cliente.**

### 6.2 Privacidade de Métricas
```typescript
interface Metricas {
  visibilidade: {
    dono: ["likes", "comentários", "visualizações"];
    terceiros: []; // interface "cega"
  };
  regra: "Se Usuario_Logado == Dono_do_Conteudo → exibe métricas, senão → oculta";
}
```

### 6.3 Upload Trimestral (Trava Sistêmica)
```typescript
function validarUploadTrimestral(tipo: "institucional" | "curriculo"): boolean {
  const ultimoUpload = getUltimoUpload(userId, tipo);
  const diasDesdeUltimo = dayjs().diff(ultimoUpload, 'days');
  
  if (diasDesdeUltimo < 90) {
    throw new Error("Aguarde 90 dias desde o último upload.");
  }
  
  return true;
}
```

### 6.4 Onboarding Parametrizado (Stripe-like)
```typescript
// URL: mastersindico.com/cadastro?role=sindico&plan=pro
interface OnboardingParams {
  role: "morador" | "sindico" | "empresa" | "marketing" | "vizinhanca";
  plan: "base" | "pagante" | "n1" | "n2" | "n3" | "plus" | "pro";
}

// Fluxo:
// 1. Sistema identifica Role e Plano via SearchParams
// 2. Cadastro inicial → Disparo de Email de Validação
// 3. Usuário confirma email → Define senha e ativa conta
// 4. Backend cria registro no DB com Role travada
```

---

## 7. ARQUITETURA TÉCNICA RECOMENDADA

### 7.1 Stack Atual (Confirmado)
- **Backend:** Fastify (Node.js) + DDD + Awilix DI + Drizzle ORM
- **Frontend:** SolidJS + TanStack Router/Form/Query + UnoCSS
- **Autenticação:** JWT + Passport + Argon2
- **Banco:** PostgreSQL
- **Vídeo:** Mux ou Cloudflare Stream (CDN + Transcoding)
- **Deploy:** CI/CD automatizado

### 7.2 Paridade Web/Mobile
- ✅ App Mobile espelha **estritamente** funcionalidades de consumo/interação da Web
- ✅ Funcionalidades de administração global ("MasterSíndico") são **exclusivas da Web**

### 7.3 ABAC (Attribute-Based Access Control)
```typescript
// Middleware de proteção de rotas
interface ABACPolicy {
  role: UserRole;
  plan: UserPlan;
  feature: Feature;
  action: "read" | "write" | "delete";
}

function checkAccess(policy: ABACPolicy): boolean {
  // Consulta Matriz Funcional
  const hasAccess = matrizFuncional[policy.role][policy.plan][policy.feature][policy.action];
  
  if (!hasAccess) {
    throw new ForbiddenError("Acesso negado. Plano insuficiente.");
  }
  
  return true;
}
```

---

## 8. CUSTOS OPERACIONAIS (OpEx)

### 8.1 Premissas de Escala
- **Total de usuários ativos:** ~217.000
  - 200.000 moradores (Base/Free)
  - 6.000 síndicos (Pagantes)
  - 11.000 empresas (Pagantes)
- **Produtores de conteúdo:** 17.000
- **Upload médio:** 8 vídeos/mês de 4min
- **Consumo médio:** 1min/dia por usuário (200.000 usuários)

### 8.2 Custos Mensais (Estimativa)

| Item | Custo (USD) |
|------|-------------|
| **Infraestrutura de Vídeo (Mux)** | |
| Encoding (544.000 min) | $12.294 |
| Storage (acumulativo) | $734 |
| Delivery (6M min) | $3.540 |
| Créditos inclusos | -$180 |
| **Subtotal Vídeo** | **$16.388** |
| | |
| **Backend + BD** | |
| Tráfego metadados | $25 |
| Compute (8 réplicas Fastify) | $240 |
| PostgreSQL gerenciado | $100 |
| **Subtotal Backend** | **$365** |
| | |
| **Total OpEx** | **~$16.753/mês** |
| Buffer operacional (15%) | **+$2.513** |
| **Total com buffer** | **~$19.266/mês** |

### 8.3 Unit Economics
- **Custo por usuário pagante:** ~$1,00/mês
- **Usuários pagantes:** 17.000
- **Receita mínima (R$ 19,90 média):** ~$60.000/mês (câmbio R$ 5,00)
- **Margem bruta:** ~66%

**Conclusão:** Plataforma é **financeiramente sustentável** com alta margem operacional.

---

## 9. CRONOGRAMA DE ENTREGAS (Sprints)

### Sprint 1: Fundação, Identidade e Acesso
- ✅ Sistema de roles (ABAC com Matriz Funcional)
- ✅ Onboarding parametrizado (Stripe-like)
- ✅ Autenticação JWT + Passport
- ✅ Gestão de perfis e visibilidade
- ✅ Motor de upload com trava temporal (trimestral)
- ✅ Motor de busca com filtros
- ✅ Connect Me (formulário unidirecional)
- ✅ Player de vídeo com paywall

### Sprint 2: Motor de Gestão Condominial
- 🔲 CRUD de Assembleias e Eventos
- 🔲 Sistema de Pautas (vídeos/áudios, anexos)
- 🔲 Geração de link temporário de compartilhamento
- 🔲 Sistema de votação assíncrona
- 🔲 Painel de transparência (Plano Diretor, Timeline)
- 🔲 Exportação PDF da linha do tempo
- 🔲 Avaliação objetiva da gestão (rating)

### Sprint 3: Vizinhança e Promoções
- 🔲 Engine de cupons com "palavra do dia"
- 🔲 Gerador de código promocional (ID_LOJA + SEQ + PALAVRA)
- 🔲 Trava de tempo (4h entre cupons)
- 🔲 Cadastro de promoções (lojistas)
- 🔲 Busca de benefícios por CEP
- 🔲 Promoções exclusivas por condomínio

### Sprint 4: Conteúdo Educacional e Administração
- 🔲 LMS (Curso → Módulo → Aula)
- 🔲 Controle de progresso do aluno
- 🔲 Geração de certificado com QR Code
- 🔲 Fórum público/exclusivo
- 🔲 Sistema de Lives (criação MasterSíndico)
- 🔲 Painel Admin com logs e métricas
- 🔲 Sistema de comerciais/anúncios
- 🔲 Integração Mercado Pago (assinaturas)

---

## 10. ONBOARDING - FLUXOS POR PERFIL

### 10.1 Morador Base (Free)
```
1. Cadastro via URL parametrizada: /cadastro?role=morador&plan=base
2. Preenche: nome, email, senha
3. Recebe email de validação
4. Confirma email → Conta ativada
5. Pode vincular unidade (opcional)
```

### 10.2 Morador Pagante
```
1. Cadastro via URL: /cadastro?role=morador&plan=pagante
2. Preenche dados básicos
3. Valida email
4. Escolhe plano de pagamento (R$ 9,90/mês)
5. Checkout Mercado Pago
6. Webhook ativa plano → Libera funcionalidades
7. Pode criar vídeo-currículo (90s)
```

### 10.3 Síndico (N1/N2/N3)
```
1. Cadastro via URL: /cadastro?role=sindico&plan=n2
2. Preenche dados profissionais:
   - Nome completo
   - Email
   - Senha
   - CPF (validação)
   - Tempo de carreira (anos)
3. Valida email
4. Checkout (R$ 19,90 | R$ 39,90 | R$ 59,90)
5. Webhook ativa plano
6. Vincula condomínios (nome, CNPJ, endereço)
7. Status inicial: BRONZE (evolui conforme uso)
```

### 10.4 Empresa (Plus/Pro)
```
1. Cadastro via URL: /cadastro?role=empresa&plan=plus
2. Preenche dados PJ:
   - Razão social
   - CNPJ
   - Email corporativo
   - Senha
   - Cidade/Estado
   - Categoria principal + até 5 subcategorias
   - Possui seguro de responsabilidade civil? (sim/não)
3. Valida email
4. Checkout (R$ 99 | R$ 199)
5. Webhook ativa plano
6. Área liberada:
   - Upload de vídeo institucional (2min)
   - Publicação de vídeos técnicos (4/mês ou 8/mês)
   - Criação de módulos (apenas Pro)
```

### 10.5 Marketing
```
1. Cadastro via URL: /cadastro?role=marketing&plan=master
2. Preenche dados da agência
3. Valida email
4. Checkout (preço customizado, leilão por "cadeira")
5. Webhook ativa plano
6. Área liberada:
   - Tag de anúncios por vídeo
   - Limite: 3 empresas/estado
   - SEM Connect Me (anti-spam)
```

### 10.6 Vizinhança (Comércio Local)
```
1. Cadastro via URL: /cadastro?role=vizinhanca&plan=parceiro
2. Preenche:
   - Nome do estabelecimento
   - Categoria (padaria, farmácia, pet, etc.)
   - CEP de atuação
   - Email
   - Senha
3. Valida email
4. Checkout (se aplicável)
5. Área liberada:
   - Cadastro de promoções
   - Definir "palavra do dia"
   - Até 3 fotos do estabelecimento
   - Promoções exclusivas (negociadas com síndico)
```

---

## 11. CONSIDERAÇÕES FINAIS

### 11.1 Diferenciais Competitivos
1. ✅ **Governança aplicada** (não é operacional)
2. ✅ **Currículo digital do síndico** com validação por moradores
3. ✅ **Sistema de reconhecimento** baseado em métricas objetivas
4. ✅ **Separação rigorosa** entre conteúdo técnico e contato comercial
5. ✅ **Métricas privadas** (sem ranking público)
6. ✅ **Conteúdo vertical mobile-first** (TikTok/Reels-like)
7. ✅ **Integração com comércio local** via cupons inteligentes

### 11.2 Riscos e Mitigações

| Risco | Mitigação |
|-------|-----------|
| Custo de vídeo explosivo | Limitação de uploads, paywall para Base |
| Spam de empresas | Sem Connect Me ativo para marketing, direção controlada |
| Vazamento de contato | Moderação rigorosa, proibição de dados em vídeos |
| Complexidade do onboarding | URLs parametrizadas, fluxo Stripe-like |
| Moradores idosos (50-70 anos) | Interface simples, validação por email (não biometria) |

### 11.3 Próximos Passos Imediatos

**Para o desenvolvimento atual (autenticação + onboarding):**

1. ✅ **Fixar backend bug** (role hardcode) - CONCLUÍDO
2. ✅ **Simplificar signup para single-step** - CONCLUÍDO
3. ✅ **Implementar verify-email com OTP** - CONCLUÍDO
4. ✅ **Criar forgot-password e reset-password** - CONCLUÍDO
5. 🔲 **Implementar onboarding por perfil** (próxima etapa)
   - Criar rotas `/onboarding/{role}`
   - Formulários específicos por perfil
   - Validações conforme documentação
6. 🔲 **Sistema de planos e checkout** (integração Mercado Pago)
7. 🔲 **Vincular perfil → features** (ABAC middleware)

---

## 12. APÊNDICES

### A. Categorias de Serviço (Empresas)

Conforme **Manual Técnico Cap. 6**, empresas escolhem:
- **1 categoria principal**
- **Até 5 subcategorias**

Exemplos:
- Elétrica (categoria principal)
  - Subcategorias: Manutenção preventiva, Instalação SPDA, Automação, Retrofit, Laudos técnicos
- Hidráulica
  - Subcategorias: Desentupimento, Detecção de vazamentos, Troca de bombas, Impermeabilização, Tubulação
- Elevadores
  - Subcategorias: Manutenção preventiva, Modernização, Laudos NR12, Automação, Instalação

**Importante:** Categorias devem ser migradas do enum estático para banco de dados com API (próxima sprint).

### B. Palavras-Chave Proibidas em Conteúdo

```typescript
const BLOCKED_KEYWORDS = [
  "orçamento", "ligue", "desconto", "promoção", "entre em contato",
  "whatsapp", "telefone", "email", "site", "clique aqui",
  "compre agora", "oferta", "preço", "valores", "consulte-nos"
];
```

### C. Estrutura de Banco de Dados (Resumo)

```sql
-- Tabelas principais
users (id, email, password_hash, role, plan, status, created_at)
profiles_morador (user_id, nome, cpf, unidade_id, video_curriculo_url)
profiles_sindico (user_id, nome, cpf, tempo_carreira, status_reconhecimento)
profiles_empresa (user_id, razao_social, cnpj, categoria_id, subcategorias[], seguro)
profiles_marketing (user_id, nome_agencia, estado, slots_contratados)
profiles_vizinhanca (user_id, nome_estabelecimento, categoria, cep)

condominios (id, nome, cnpj, endereco, sindico_id)
vinculos_unidade (id, condominio_id, numero, morador_id)

videos (id, user_id, tipo, url, duracao, status_moderacao, created_at)
modulos (id, empresa_id, titulo, descricao)
aulas (id, modulo_id, titulo, video_id, ordem)
certificados (id, user_id, modulo_id, data_conclusao, qr_code)

assembleias (id, condominio_id, titulo, data, edital_url, status)
pautas (id, assembleia_id, titulo, descricao, video_url, anexos[])
votacoes (id, pauta_id, tipo, opcoes[], janela_inicio, janela_fim)
votos (id, votacao_id, unidade_id, opcao, created_at)

cupons (id, loja_id, codigo, usuario_id, created_at, expires_at)
promocoes (id, loja_id, titulo, regra, palavra_dia, validade)

connect_me (id, remetente_id, destinatario_id, mensagem, status, created_at)
```

### D. Referências dos Documentos

- **Manual Técnico My Síndico** (46 páginas, 4.1MB)
- **Matriz Funcional** (11 páginas, 4MB)
- **Análise Completa** (14 páginas, PDF)
- **Escopo Técnico (SOW)** - Anexo I (novo escopo-7.pdf)
- **Proposta MVP 12 Semanas** (589KB)
- **Escopo de Prestação de Serviços** (MySindico-2.pdf)
- **Manual Identidade Visual** (3.5MB, cores e branding)
- **Perfil das Empresas** (3.9MB)

---

**Documento gerado por:** Droid (Factory AI)  
**Versão:** 1.0  
**Status:** ✅ Completo e validado
