---
title: "Domínio - Cross-Domain & Integrações"
version: "3.0"
date: 2026-03-10
status: "em-revisão"
high_domain: "Cross-Domain"
requisitos: "Req 33–47, 61–67, 102–103"
total_requisitos: 24
tags:
  - mastersindico
  - dominio
  - cross-domain
  - integracoes
  - compliance
  - seguranca
---

# 🛡️ Domínio Principal: Cross-Domain & Integrações

> **High-Domain:** Cross-Domain / Infraestrutura | **Requisitos:** 33–47, 61–67, 102–103
> Este domínio engloba serviços horizontais que atendem a toda a aplicação: Compliance Governamental, Trilha de Auditoria, Analytics, Segurança e Integrações de Terceiros.
> **Breadcrumbs:** [[00 - Índice Master Síndico|🏠 Início]] > [[Arquitetura do Ecossistema|🏛️ Arquitetura]] > 📄 Domínio - Cross-Domain

---

## Visão da Arquitetura Horizontal

![[Arquitetura Horizontal.canvas]]

---

## 🏛️ Governança e Compliance (Req 33–42)

**Problema Resolvido:** Síndicos perdem controle legal e contratual sobre o mandato. Condomínios sofrem com multas por LGPD ou falta de assinaturas.
**Solução:** Dashboards em tempo real de compliance, bloqueios automáticos de operação (ex: bloqueio de aplicativo se morador não der ciência).

### Principais Componentes:
- **Req 33 (Compliance Dashboard):** Agrega `declaracao_anual`, `assinaturas`, `conflito_interesse` e gera a métrica fundamental `governance_score` (0-100). Status: `em_dia`, `pendente`, `atrasado`.
- **Req 34 (Declaração Anual):** Submetida por mandato/anualmente. Se atrasar, o sistema **bloqueia** exportação de dossiê e finalização de mandato.
- **Req 35 (Assinaturas Obrigatórias):** Gere termos de aceite. Bloqueia ações vitais até morador assinar LGPD/termos.
- **Req 36 (Matriz de Conflitos):** Declarações obrigatórias de relação (familiar, societário) entre Síndico e Empresas. O sistema _flag_ negociações com conflitos.
- **Req 37 (Trilha de Auditoria - CRÍTICO):** Log *append-only* de ações essenciais (Timeline, Negócios, Validações). Armazenamento exigido por 5 anos (LGPD compliance total). Retém IPs, `user_agent`, `before/after_state`.
- **Req 38 (Mapa de Risco):** Extrai ações do Plano Diretor e gera matriz Probabilidade vs Impacto.
- **Req 39 (Compliance Contratual):** Tracking de entregáveis em negócios fechados.
- **Req 40 (Registro LGPD):** Tracker específico append-only para revelação de dados no _Connect Me_ (quem viu o que, a que horas, com qual IP). Funções de "Esquecer Usuário" (Anonimização) embutidas.
- **Req 41 & 42 (Score e Dossiê):** Governance Score diário comparado à média da plataforma. Permite exportar dossiê PDF assinado digitalmente no Fim do Mandato (Req 46).

---

## 📊 Analytics e Indicadores (Req 43–45)

**Problema Resolvido:** Ausência de dados cruzados entre o engajamento da comunidade e a prestação de serviços das empresas.

- **Req 43 (Dashboard do Síndico):**
  - Métricas: total de moradores, `governance_score`, taxa de conclusão do Plano Diretor, `morador_satisfaction_avg`.
  - Exportação em PDF, charts evolutivos.
- **Req 44 (Dashboard da Empresa):**
  - Métricas: condomínios conectados, CTR e Aceite do *Connect Me*, registros aprovados/rejeitados, avaliação institucional média.
- **Req 45 (Dashboard da Agência):**
  - Métricas: engajamento cruzado das empresas vinculadas (views, watch time, retenção por tipo de vídeo).

---

## 🔄 Gestão de Mandato (Req 46–47)

- **Req 46 (Encerrar Mandato):** O síndico exige status `compliance = em_dia`. Se finalizado, todos os moradores associados àquela gestão podem ter view reset ou bloqueio até novo síndico. Gera dossiê automaticamente e revoga acessos.
- **Req 47 (Transferência):** Passa o bastão digitalmente convidando novo síndico. Transfere histórico do condomínio, linha do tempo e negócios; **NÃO transfere** avaliações pessoais do síndico.

---

## 🔒 Segurança Adicional (Req 102–103)

- **Req 102 (Restrições de IP/Device):**
  - Módulo antifraude: Limita a **1 device ativo** (device fingerprint) ou registro por IP por plano tier.
  - CAPTCHA forçado para tentativas suspeitas.
- **Req 103 (Visibilidade Extrita de Conteúdo):**
  - Controle severo entre Tenants. Vídeos de empresas são blindados a moradores, liberados momentaneamente **exclusivamente** durante Assembleias (Escolha de Fornecedor) e revogados após fim da votação.

---

## 🔌 Integrações de Terceiros (Req 61–67)

Arquitetura exige o uso de um padrão **Adapter Layer** via Hexagonal/Clean Architecture, evitando vínculo duro do Core Domain (`@myfm/core`) com as APIs externas.

| Requisito | Propósito | Tecnologia Selecionada / Solução | Falhas / Webhooks |
|-----------|-----------|----------------------------------|-------------------|
| **Req 61** | Billing & Assinaturas | **Mercado Pago API** | Retries. Webhooks de atualizações (Pix, Boleto, CC). Transações logadas no Audit. |
| **Req 62** | Comunicação (Email) | **AWS SES ou SendGrid** | Templates institucionais, batch sending. Tracker de bounces, delivered e opened. |
| **Req 63** | Comunicação (SMS) | **Twilio ou Zenvia** | Fallback de notificações críticas (Edital de Assembleia). Rate limiting per user. |
| **Req 64** | Video Streaming | **Mux ou Cloudflare Stream** | Geração de URLs HLS adaptativas, Thumbnails e eventos de playback completos. URLs devem ser assinadas. |
| **Req 65** | Armazenamento de Arquivos | **AWS S3 (ou similar R2)** | Upload documentais. Organização isolada via Prefix `tenant_id/`. ACL baseada em assinaturas curtas (Pre-Signed URLs). |
| **Req 66** | Geocoding de Endereço | **ViaCEP** | Caching embutido para poupar API Rate Limits. |
| **Req 67** | Notificações Push Mobile | **Firebase Cloud Messaging** | Alertas urgentes (Início da Assembleia, Resposta Connect Me). |

---

**Breadcrumbs:** [[00 - Índice Master Síndico|🏠 Início]] · **Anterior:** [[Domínio - Assembleia]]