---
title: Cross-Domain — Compliance, Analytics, Integrações e Antifraude
version: 1.0
status: stable
type: requirement
domain: cross-domain
tags: [compliance, audit-trail, lgpd, analytics, integration, antifraude]
---

# Cross-Domain — Compliance, Analytics, Integrações e Antifraude

Requisitos transversais que atravessam múltiplos bounded contexts: compliance e governança, trilha de auditoria, LGPD, dashboards de analytics e integrações com provedores externos.

---

## Req 33–42 — Compliance e Governança

Matriz de conformidade com scoring automático, declarações e auditoria append-only.

### Req 33 — Governance Score

Score de conformidade 0-100 calculado diariamente.

### 10 Componentes (propostos)

1. **LGPD Compliance**: dados pessoais protegidos (sem vazamentos) — 10pt
2. **Declarações em dia**: anual completada — 10pt
3. **Assinaturas obrigatórias**: todos os usuários assinaram termos — 10pt
4. **Trilha de auditoria**: logs completos disponíveis — 10pt
5. **Nenhuma reclamação formal**: 0 reclamações ativas — 10pt
6. **Fundo de Reserva em dia**: contribuições registradas — 10pt
7. **Assembleias regularizadas**: homolagadas — 10pt
8. **Sem atrasos em rotinas**: Timeline atualizada, comunicados regulares — 10pt
9. **Avaliação de gestão**: avaliações completadas (Req 15) — 10pt
10. **Política interna**: regimento documentado, público — 10pt

### Requisitos MUST

- Cálculo automático diário (job noturno)
- Histórico: acompanhar evolução (12 meses)
- Visibilidade: público (transparência condominial)
- Alertas: se score cai < 60, síndico recebe notificação

### Notas de implementação

- Tabelas: `governance_scores`, `governance_score_history`
- Job: cálculo via aggregações SQL + lógica de negócio

---

### Req 34 — Declaração Anual Obrigatória

Síndico declara conformidade com regras.

### Conteúdo

- Checklist: "Confirmou LGPD", "Realizou auditoria interna", "Comunicou mudanças", etc.
- Assinatura digital
- Data de envio

### Requisitos MUST

- Uma declaração por ano (12 meses)
- Atrasada: bloqueia exportação de dossiê (Req 46)
- Histórico: 5 anos de retenção (LGPD)

### Notas de implementação

- Tabelas: `compliance_declarations`

---

### Req 35 — Assinaturas Obrigatórias

Todos os usuários cadastrados devem assinar termos.

### Termos

1. Termos de Serviço (genérico)
2. Política de Privacidade (LGPD)
3. Código de Conduta (regras de comunidade)

### Requisitos MUST

- Novo usuário: aceitar antes de usar app
- Versionamento: se termos mudam, usuários devem aceitar nova versão
- Tracking: data, IP, versão assinada

### Notas de implementação

- Tabelas: `user_term_acceptances`

---

### Req 36 — Matriz de Conflitos de Interesse

Síndico declara conflitos (ex: sócio de empresa que apresenta proposta).

### Requisitos MUST

- Declaração obrigatória no onboarding (Req 5)
- Update periódico (anual)
- Impacto: votação em pauta onde síndico tem conflito é marcada como "abstém obrigatória"

### Notas de implementação

- Tabelas: `conflict_of_interest_declarations`

---

### Req 37 — Trilha de Auditoria Append-only

Registro imutável de todas as ações críticas.

### Eventos rastreados

- Login/logout
- Criação/modificação de pauta
- Votação
- Publicação de comunicado
- Validação de execução
- Mudança de permissões de usuário
- Exclusão de dados (soft delete)

### Dados rastreados por evento

- `user_id`, `tenant_id`, `action`, `resource_type`, `resource_id`
- `timestamp`, `ip_address`, `user_agent`
- `changes_before`, `changes_after` (JSON)

### Requisitos MUST

- **Append-only**: nunca UPDATE, nunca DELETE de audit logs
- Retenção: 5 anos (LGPD)
- Índices: para busca rápida por usuário/data/ação
- Sigilo: apenas admin + síndico principal podem acessar

### Notas de implementação

- Tabelas: `audit_trail`
- Integração: evento NATS centralizado `audit.log.created`
- Compressão: após 1 ano, archive em MinIO (S3 Glacier)

---

### Req 38 — Mapa de Riscos

Identificação de riscos potenciais do condomínio.

### Tipos de risco

- Financeiro (falta de fundo de reserva)
- Compliance (atrasos em conformidade)
- Estrutural (problemas prediais)
- Social (conflitos entre moradores)
- Segurança (vulnerabilidades)

### Requisitos MUST

- Síndico cria/atualiza mapa (anual)
- Integração com Master Plan (Req 12) — identificar onde há risco

### Notas de implementação

- Tabelas: `risk_maps`, `risk_items`

---

### Req 39-42 — LGPD Registry, Dossiê e Certificações

#### Req 39 — LGPD Registry

Tracker de revelação de dados pessoais (especialmente em Connect Me).

### Rastreamento

- Quem recebeu dados de quem
- Data, IP, termos aceitos
- Propósito (ex: "Orçamento de serviço")
- Versão de termos

### Requisitos MUST

- Log automático quando síndico revela dados em Connect Me (Req 19)
- Retenção: 5 anos
- Direito ao esquecimento: sob demanda, purga logs após 5 anos

### Notas de implementação

- Tabelas: `lgpd_logs` (central, integrado com todos os módulos)

---

#### Req 40 — Dossiê PDF Assinado Digitalmente

Documento consolidado gerado no fim do mandato.

### Conteúdo

- Histórico do mandato (datas, decisões)
- Timeline consolidada
- Atas de assembleias
- Relatórios de compliance
- Signatures digitais (síndico + admin)

### Requisitos MUST

- Geração automática no encerramento do mandato (Req 46)
- Armazenamento: MinIO (permanente)
- Acesso: síndico pode baixar (histórico preservado)

### Notas de implementação

- Job async: gera PDF ao `mandates.end_date` ser atingida

---

#### Req 41-42 — Certificações e Compliance Markers

Autodeclarações (Req 6.2, 6.3) com disclaim.

### Requisitos MUST

- Disclaimer obrigatório: "Certificações são autodeclaradas, não certificadas por terceiro"
- Sem validação (confiança no síndico/empresa)
- Visibilidade: pública (diferencial de reputação)

---

## Req 43–45 — Analytics

Dashboards customizados por role com métricas-chave.

### Req 43 — Dashboard Síndico

| Métrica | Descrição |
|---------|---|
| Total de moradores | Contagem de titulares + dependentes |
| Governance Score | 0-100 (Req 33) |
| Taxa conclusão Plano Diretor | % de atividades concluídas |
| Satisfação média | Média de avaliação de gestão (Req 15) |
| Próximos marcos | Datas importantes (assembleia, reavaliação, etc) |

### Requisitos MUST

- Atualização em tempo real (ou 1x/hora)
- Histórico: gráficos de 12 meses
- Export: CSV/PDF para relatório

---

### Req 44 — Dashboard Empresa

| Métrica | Descrição |
|---------|---|
| Condomínios conectados | # de condos com deals ativos |
| CTR Connect Me | Click-through rate de propostas |
| Taxa aprovação | % de registros de execução aprovados |
| Avaliação média | Score de avaliação por síndicos |
| Receita (estimada) | Valor de deals (Sprint 3+) |

### Requisitos MUST

- Segmentado por período (última semana, mês, trimestre, ano)

---

### Req 45 — Dashboard Agência de Marketing

| Métrica | Descrição |
|---------|---|
| Engajamento cruzado | Quantas empresas elas gerenciam ativos |
| Views médios | Média de views dos vídeos |
| Retention rate | Taxa de conclusão de vídeos |
| Top performers | Empresas com melhor performance |

---

## Req 46–47 — Gestão de Mandato

### Req 46 — Encerramento de Mandato

Transição segura de síndicos com preservação de histórico.

### Fluxo

1. Síndico principal inicia encerramento
2. Sistema valida: `compliance = em_dia` (Req 34 completa)
3. Dossiê PDF é gerado (Req 40)
4. Condomínio fica "sem síndico" até novo síndico assume
5. Histórico: preservado para auditoria

### Requisitos MUST

- Bloqueio: não consegue encerrar se compliance atrasada
- Transição: novo síndico herda histórico (read-only)

### Notas de implementação

- Tabelas: `mandates` (campo `ended_at`)

---

### Req 47 — Transferência de Mandato

Novo síndico assume, herda histórico, mas não herda avaliações pessoais.

### Requisitos MUST

- Histórico de Timeline: visível para novo síndico
- Avaliações de gestão (Req 15): **NÃO** são herdadas (reset)
- Governance Score: reseta (novo síndico, novo ciclo)

---

## Req 61–68 — Integrações (Adapter Layer)

Provedores externos abstraídos via interfaces, permitindo troca sem impacto no código de negócio.

| Req | Serviço | Adapter | Status |
|-----|---------|---------|--------|
| 61 | Stripe | `IPaymentGateway` | Sprint 1 (implementado) |
| 62 | Resend | `IEmailProvider` | Sprint 1 (implementado) |
| 63 | Twilio/Zenvia | `ISMSProvider` | Sprint 2 (blueprint) |
| 64 | Mux | `IVideoProvider` | Sprint 1 (implementado) |
| 65 | MinIO/Cloudflare R2 | `IStorageProvider` | Sprint 1 (implementado) |
| 66 | ViaCEP | `ICEPLookup` | Sprint 1 (adapter) |
| 67 | Firebase Cloud Messaging | `IPushProvider` | Sprint 2 (blueprint) |
| 68 | Livekit | `ILiveProvider` | Sprint 2 (blueprint) |

### Req 61 — Stripe (Billing)

Ver `billing.md` Req 4.

---

### Req 62 — Resend (Email)

Envio de email transacional com templates pt-BR.

### Requisitos MUST

- Templates: pré-built para notificações chave (boas-vindas, assembleia, etc)
- Personalização: variáveis de template por evento
- Domínio: noreply@mastersindico.com.br
- Bounce handling: inválida email se bounce persistente

### Notas de implementação

- Interface: `IEmailProvider` (`internal/shared/providers/email/`)
- Implementação: `resend.EmailProvider`

---

### Req 63 — Twilio/Zenvia (SMS)

Envio de SMS para OTP, notificações críticas.

### Requisitos MUST

- Fallback: se Twilio falha, tenta Zenvia
- Consentimento: morador opt-in para SMS
- Rate limiting: máx 1 SMS por usuário por minuto

### Notas de implementação

- Interface: `ISMSProvider`

---

### Req 64 — Mux (Vídeo)

Ver `content.md` Req 29.

---

### Req 65 — MinIO/Cloudflare R2 (Storage)

Armazenamento de objetos S3-compatível.

### Requisitos MUST

- Endpoints: MinIO (dev/local), Cloudflare R2 (prod via AWS SDK)
- Buckets: `videos`, `documents`, `media`, `public`
- Ciclo de vida: archive após 1 ano, delete após 7 anos (LGPD)
- Assinatura pré-preenchida: URLs com expiração 24h (default)

### Notas de implementação

- Interface: `IStorageProvider`
- Implementação: wrapper em torno de `aws-sdk-go-v2/s3`

---

### Req 66 — ViaCEP (Endereço)

Lookup de endereço via CEP.

### Requisitos MUST

- Cache: Redis `cep:{cep}` com TTL 30 dias
- Fallback: se ViaCEP indisponível, permite input manual
- Validação: recusa CEP inválido

### Notas de implementação

- Interface: `ICEPLookup`

---

### Req 67 — Firebase Cloud Messaging (Push)

Notificações push para mobile.

### Requisitos MUST

- Registro de device token ao login (mobile)
- Tópicos: `assembly_{condominium_id}`, `user_{user_id}`
- Consentimento: opt-in obrigatório

### Notas de implementação

- Interface: `IPushProvider`

---

### Req 68 — Livekit (Conferência ao Vivo)

Conferência WebRTC para assembleias online/híbrida (futuro).

### Requisitos MUST

- Rooms: 1 por assembleia
- Recording: automático (armazena em MinIO)
- Permissões: síndico = moderador, moradores = participantes
- API para corte de áudio (timeout fila de fala — Req 60.5)

### Notas de implementação

- Interface: `ILiveProvider`

---

## Req 102 — Antifraude

Proteção contra roubo de conta e cadastros fraudulentos.

### Requisitos MUST

- **1 dispositivo ativo por conta** (Rule 8) — login em 2º device invalida 1º (implementado — Req 1)
- **CAPTCHA**: acionado após 3 tentativas de login falhadas (Sprint 2)
- **Bloqueio por IP**: múltiplos cadastros do mesmo IP em < 1h → suspende (manual review)
- **Detecção de anomalia**: login de IP novo → SMS de confirmação (Sprint 3)

### Notas de implementação

- Integração reCAPTCHA v3 (frontend) ou Cloudflare Turnstile
- Job: analisa `audit_trail` para detect padrões suspeitos

---

## Req 103 — Visibilidade Estrita

Aplicação da Rule 4 — vídeos de empresa blindados a moradores.

### Requisitos MUST

- Vídeos de empresa: **bloqueados** por default para moradores
- Unlock momentâneo: apenas durante votação de fornecedor (Req 57)
- Acesso revelado: via `video_visibility_grants` vinculado a votação
- Auto-revogação: quando votação encerra, revogar acesso (NATS event)

### Notas de implementação

- Middleware ABAC: valida `video_visibility_grants` antes de servir stream
- Evento NATS: `assembly.voting.closed` → `video.visibility.revoked`

---

## Invariantes cross-domain

1. Audit trail é append-only — nunca DELETE, nunca UPDATE
2. LGPD logs: retenção 5 anos, direito ao esquecimento após vencimento
3. Governance Score: calculado diariamente, histórico de 12 meses
4. Compliance Declarations: obrigatórias 1x/ano, bloqueia certas ações se atrasadas
5. Todas as assinaturas: rastreadas com timestamp + IP (versionadas)
6. Integrações: agnósticas (IPaymentGateway, IEmailProvider, etc) — swap sem código de negócio

---

## Referências cruzadas

- **Req 1** (Identity): audit trail de login
- **Req 4** (Billing): Stripe integration (Req 61)
- **Req 5** (Billing): onboarding signature acceptance
- **Req 15** (Institutional): avaliação de gestão → impacto governance score
- **Req 19** (Commercial): LGPD logs de Connect Me (Req 39)
- **Req 29** (Content): visibilidade estrita de vídeo (Req 103)
- **Req 33** (Cross-Domain): governance score (Req 43-45 dashboards)
- **Req 57** (Assembly): votação desbloqueia vídeos (Req 103)
