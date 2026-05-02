---
title: "Arquitetura do Ecossistema Master Síndico"
version: "3.0"
date: 2026-03-10
status: "em-revisão"
tags:
  - mastersindico
  - arquitetura
  - ddd
  - dominio
  - ecossistema
---

# 🏗️ Arquitetura do Ecossistema Master Síndico

> Documento de referência arquitetural. Define os domínios de negócio, fronteiras, tenants, actors, serviços externos, fluxos cross-domain e contratos entre domínios.
> **Voltar para:** [[Índice Master Síndico]]

---

## 1. Visão Conceitual

O Master Síndico **não é um app único** — é um **ecossistema de domínios de negócio integrados** que funciona como uma rede. A analogia mais precisa é pensar como DNS Architecture:

- O **ecossistema Master Síndico** é o root
- Abaixo dele existem poucos **high-domains** (grandes capacidades de negócio)
- Dentro de cada high-domain existem **mid-domains** (serviços reais)
- Dentro de cada mid-domain existem **low-domains** (módulos específicos)

### Princípio Fundamental

> **O mesmo CPF pode transitar entre contextos**: ser morador no Condomínio X, síndico no Condomínio Y, e ter uma empresa cadastrada. A identidade é uma só, mas os contextos de operação são múltiplos.

---

## 2. Hierarquia de Domínios

![[Arquitetura de Domínios.canvas]]

---

## 3. Detalhamento por High-Domain

### 3.1 🔐 IDENTIDADE — O IAM do Ecossistema

> **Analogia:** `accounts.google.com` serve tudo no Google. Aqui vive o usuário, auth, planos, billing. Todo mundo resolve identidade passando por aqui primeiro. É o root resolver do ecossistema.
> **Dependências:** Zero — todos dependem dele, ele não depende de ninguém.
> **Se cair:** Tudo cai.

| Mid-Domain | Responsabilidade | Requisitos |
|-----------|-----------------|-----------|
| **IAM** | Autenticação JWT, ABAC, roles, tenant isolation | [[Domínio - Identidade#Req 1]], [[Domínio - Identidade#Req 2]] |
| **Billing** | Planos (trial/base/plus/pro), Mercado Pago, quotas | [[Domínio - Identidade#Req 4]], [[Domínio - Identidade#Req 4.1]] |
| **Onboarding** | Fluxos por persona (4-7 telas), auto-save, validação | [[Domínio - Identidade#Req 5]] |

**Conceito-chave — O Plano como Chave Global:**

O plano é uma chave global que liga/desliga features across domains. Não é billing por produto individual — um único plano destrava funcionalidades em todos os domínios simultaneamente:

| Plano | Síndico | Empresa | Parceiro |
|-------|---------|---------|----------|
| **Trial** | 15 dias, visualiza tudo, bloqueia ações estratégicas | 7 dias, mesma lógica | 30 dias, mesma lógica |
| **Base** | Funcionalidades básicas | — | — |
| **Plus** | Governança expandida | Perfil + Connect Me | — |
| **Pro** | Tudo liberado | Tudo liberado | Tudo liberado |

**Rotas de API (Estimadas):**

```
POST   /v1/auth/register          → Registro de novo usuário
POST   /v1/auth/login             → Login com JWT
POST   /v1/auth/refresh           → Refresh token
POST   /v1/auth/forgot-password   → Reset de senha
GET    /v1/auth/me                → Sessão atual
POST   /v1/auth/oauth/:provider   → OAuth (Google/Apple)
GET    /v1/billing/plans          → Listar planos
POST   /v1/billing/subscribe      → Assinar plano
PUT    /v1/billing/upgrade        → Upgrade de plano
GET    /v1/billing/history        → Histórico de pagamentos
```

> [!WARNING]
> **Gap:** Decision 4 (Paywall) — não está definido se o bloqueio ao expirar é hard block imediato ou soft block com grace period. Impacta a lógica de frontend inteira.

---

### 3.2 🏛️ INSTITUCIONAL — O Coração do Produto

> **Tudo que é tenant-bound** vive aqui. Tudo que existe dentro do contexto de um condomínio.
> **O síndico é o gatekeeper** — nada entra na Timeline sem validação dele.
> **O condomínio é o tenant** — sem discussão.

| Mid-Domain | Responsabilidade | Requisitos |
|-----------|-----------------|-----------|
| **Governança** | Timeline imutável, Plano Diretor reativo, Compliance, Avaliações | [[Domínio - Institucional#Req 11-17]] |
| **Condomínio** | CRUD condo, unidades, vínculos, mandatos, QR Code | [[Domínio - Institucional#Req 7-10]] |
| **Vizinhança** | Parceiros locais, promoções, métricas, 4 jornadas | [[Domínio - Institucional#Req 27-27.3]] |
| **Assembleia** | Ao vivo, votação real-time, telão, check-in, relatórios | [[Domínio - Assembleia]] |

**Fluxo Central — Governança:**

![[Fluxo Governanca.canvas]]

**Tipos de Publicação na Timeline:**

| Tipo | Origem | Requer Validação |
|------|--------|-----------------|
| `atividade_da_gestao` | Síndico | Não |
| `registro_de_execucao` | Empresa → Síndico | **Sim** |
| `comunicado` | Empresa → Síndico | **Sim** |
| `evento` | Síndico | Não |
| `adendo` | Síndico | Não |
| `resultado_consulta_moradores` | Automático | Não |
| `assembly_result` | Automático | Não |

**Rotas de API (Estimadas):**

```
# Condomínio
POST   /v1/condominiums                    → Criar condomínio
GET    /v1/condominiums/:id                → Detalhe
POST   /v1/condominiums/:id/units          → Cadastrar unidades
POST   /v1/condominiums/:id/residents      → Vincular morador

# Timeline
GET    /v1/condominiums/:id/timeline       → Listar publicações
POST   /v1/condominiums/:id/timeline       → Nova publicação
POST   /v1/condominiums/:id/timeline/:id/addendum → Adendo

# Plano Diretor
GET    /v1/condominiums/:id/master-plan    → Listar ações
POST   /v1/condominiums/:id/master-plan    → Nova ação
# (status atualiza automaticamente via evento da Timeline)

# Compliance
GET    /v1/condominiums/:id/compliance     → Status
POST   /v1/condominiums/:id/compliance/declaration → Declaração anual
```

---

### 3.3 💼 COMERCIAL — A Camada Cross-Tenant

> **Tudo que cruza fronteiras** vive aqui. Empresas, prestação de serviço, propostas, contratos.
> **O Connect Me é marketplace unidirecional** — LGPD by design, dados revelados apenas após aceite.

| Mid-Domain | Responsabilidade | Requisitos |
|-----------|-----------------|-----------|
| **Connect Me** | Formulários unidirecionais, cotas por plano, 4 direções | [[Domínio - Comercial#Req 19-19.3]] |
| **Serviços** | Propostas, dupla confirmação, registro execução, comunicados | [[Domínio - Comercial#Req 20-26]] |
| **Perfis** | Visibilidade por plano, busca, status (Bronze/Prata/Ouro/Diamante) | [[Domínio - Comercial#Req 18]] |

**Fluxo do Connect Me:**

![[Fluxo Connect Me.canvas]]

**4 Direções do Connect Me:**

| Direção | Quotas | Requisito |
|---------|--------|-----------|
| Síndico → Empresa | N1: 2/ano, N2: 4/ano, N3: ilimitado | Req 19 |
| Morador → Síndico | Base: 2/ano, Pg: 4/ano | Req 19.1 |
| Empresa → Empresa | Plus/Pro apenas | Req 19.2 |
| Síndico → Parceiro | N1+: sem limite documentado | Req 19.3 |

> [!WARNING]
> **Gap Crítico:** As quotas são por **ano** (PDF Matriz Funcional) ou por **mês** (requirements.md em algumas seções)? Precisa confirmação do cliente.

> [!WARNING]
> **Gap:** Decision 11 — O Connect Me Morador→Síndico é formulário unidirecional (como todos os outros) ou é um ticket system com resposta? O Req 19.1 como está escrito descreve um mini ticket system, mas isso conflita com o princípio "Connect Me NÃO é chat".

---

### 3.4 📚 CONTEÚDO — Distribuição e Aprendizado

> **A camada de distribuição de conteúdo.** Mídia, player, busca, visibilidade. Serve a todos os outros domínios.
> **Analogia Netflix:** Separar catálogo, streaming e encoding.

| Mid-Domain | Responsabilidade | Requisitos |
|-----------|-----------------|-----------|
| **Mídia** | Upload com trava temporal, paywall 25%, player, streaming | [[Domínio - Conteúdo#Req 29]] |
| **Academia LMS** | Cursos, treinamentos, lives, fórum, biblioteca (5 áreas, 15 telas) | [[Domínio - Conteúdo#Req 98.1]] |

**Regra Crucial — Visibilidade de Vídeos:**

> Moradores **NÃO** acessam vídeos institucionais de empresas normalmente. Eles só visualizam quando a empresa participa de um processo de escolha de fornecedor no módulo assembleia, e **após a votação o acesso é bloqueado**. O vídeo institucional **não é conteúdo público** — é conteúdo que serve a um processo decisório com janela de acesso temporária.

![[Visibilidade Videos.canvas]]

**Limites de Upload por Plano:**

| Plano | Vídeos/mês |
|-------|-----------|
| Síndico N1 | 4 |
| Síndico N2 | 8 |
| Síndico N3 | 12 |
| Empresa Plus | 8 |
| Empresa Pro | 12 |

---

## 4. Serviços Externos (Adapter Layer)

> **Princípio:** Nenhum domínio interno fala direto com providers. Existe um **Adapter Layer** (camada anti-corrupção) que abstrai as integrações. Se amanhã troca Mux por Cloudflare, só muda o adapter, nenhum domínio interno sabe que mudou.

![[Adapter Layer.canvas]]

| Serviço | Uso | Adapter |
|---------|-----|---------|
| **Mux / Cloudflare Stream** | Upload, encoding e streaming de vídeos | `IVideoProvider` |
| **Mercado Pago** | Assinaturas, billing, pagamento de cursos | `IPaymentGateway` |
| **AWS SES / SendGrid** | Emails transacionais (verificação, notificação) | `IEmailProvider` |
| **Twilio / Zenvia** | SMS para OTP e verificação de telefone | `ISMSProvider` |
| **S3 / R2** | PDFs, imagens, anexos, documentos de assembleia | `IStorageProvider` |
| **ViaCEP** | Busca de endereço por CEP | `ICEPLookup` |
| **Google / Apple OAuth** | Login social | `IOAuthProvider` |

---

## 5. Fluxos Cross-Domain

### 5.1 Fluxo: Síndico Contrata Empresa

![[Fluxo Contratacao.canvas]]

### 5.2 Fluxo: Assembleia ao Vivo

![[Fluxo Assembleia Ao Vivo.canvas]]

### 5.3 Fluxo: Vizinhança — Parceiro cria Promoção

![[Fluxo Vizinhanca.canvas]]

---

## 6. Dados Confirmados para Implementação

> [!NOTE]
> Os dados abaixo são **confirmados** pelos PDFs analisados (fonte de verdade). Nenhuma suposição.

| Dado | Valor | Fonte |
|------|-------|-------|
| Telas mapeadas | 114 | 70 (jornadas) + 10 (VS) + 15 (LMS) + 6 (VE) + 6 (VZ) + 7 (VM) |
| Requisitos | 104 | 96 originais + 8 adicionados |
| Critérios de aceite | 770+ | Contagem manual |
| Enums normalizados | 36+ | 27 originais + 9 novos |
| Documentos analisados | 23/23 | 7 texto + 16 PDFs |
| Decisões resolvidas | 6 | Decisions 1, 2, 3, 5, 7, 9 |
| Decisões pendentes | 5 + quotas | Decisions 4, 6, 8, 10, 11 + unidade de quota |
| Serviços externos | 7 | Mux, MP, SES, Twilio, S3, ViaCEP, OAuth |
| Onboarding: Morador | 4 telas | PDF confirmado |
| Onboarding: Síndico | 6 telas | PDF confirmado |
| Onboarding: Empresa | 7 telas | PDF confirmado |
| Onboarding: Parceiro | 4 telas | PDF confirmado |

---

## 7. O Que NÃO Está Documentado (Gaps Explícitos)

> [!CAUTION]
> **NENHUMA SUPOSIÇÃO FOI FEITA.** Os itens abaixo requerem documentação do cliente antes de qualquer implementação.

| # | Gap | Por que importa | Quem resolve |
|---|-----|-----------------|-------------|
| 1 | **Painel Admin MasterSíndico** | Opera acima de tudo — moderação, dashboards financeiros, broadcasts | Cliente (documento faltante) |
| 2 | **Paywall: hard vs soft block** | Define toda a lógica de UI quando plano expira | Cliente (Decision 4) |
| 3 | **Banco de Talentos: escopo** | 11 telas prontas nos PDFs, mas sem confirmação de escopo | Cliente (Decision 6) |
| 4 | **Mobile: features** | Push notifications, câmera, QR scanner no mobile? | Cliente (Decision 10) |
| 5 | **Connect Me: unidade de quota** | /ano (PDF Matriz) ou /mês (requirements em partes)? | Cliente |
| 6 | **Limite condomínios** | Regra 5 do PDF diz 12, tela S2 diz 15 | Cliente |
| 7 | **Currículo: vínculos** | Cadastro aceita 5, PDF da empresa mostra 3 | Cliente (Decision 6) |
| 8 | **Plano Diretor: status** | PDF tem 7, .ini tem 11 | Cliente |
| 9 | **Connect Me Morador→Síndico** | Unidirecional ou ticket? | Cliente (Decision 11) |

**Voltar para:** [[Índice Master Síndico]]
