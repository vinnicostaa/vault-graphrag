---
title: Threat Model — Master Síndico
type: project
tags: [master-sindico, security, threat-model, stride]
project: master-sindico
created: 2026-04-22
---

# Threat Model — Master Síndico

Modelo de ameaças usando **STRIDE** (segurança técnica) + **LINDDUN** (privacidade). Revisado trimestralmente e quando feature nova atravessa superfície pública.

---

## Assets

| Asset | Sensibilidade | Impacto de compromisso |
|---|---|---|
| **Credenciais** (Zitadel tokens, API keys) | Crítico | Comprometimento total do tenant |
| **Dados pessoais** (CPF, CNPJ, endereço, email, telefone) | Crítico (LGPD) | Multa LGPD + reputação |
| **Atas de assembleia** | Alto | Fraude eleitoral condominial |
| **Fundos via Stripe** (dados de cartão NÃO armazenados) | Alto | Fraude financeira + PCI |
| **Vídeos Mux** (privados + institucionais) | Médio | Exposição indevida |
| **Reputação de empresas** | Médio | Manipulação de ranking |
| **Timeline institucional** | Médio (legal) | Manipulação de memória auditável |
| **Código-fonte do backend** | Médio | Exposição de lógica + facilita ataque |

---

## Actors

| Actor | Motivação | Nível de acesso base |
|---|---|---|
| **Morador legítimo** | Usar app | Autenticado, role=morador, tenant=seu condomínio |
| **Síndico legítimo** | Gerir condomínio | Autenticado, role=sindico, tenant=seu condomínio |
| **Empresa contratada** | Vender serviços, construir reputação | Autenticado, role=empresa |
| **Atacante externo** | Financeiro / fraude | Zero acesso |
| **Insider malicioso** | Vingança / sabotagem | Pode ser morador/síndico legítimo abusando |
| **Atacante automatizado (bots)** | Credential stuffing, scraping | Zero acesso |
| **Supply chain** | Injeção via dependência comprometida | Depende da dep |

---

## Trust Boundaries

```
[Cliente (browser / app)]
          ↓ HTTPS (TLS 1.3)
[Railway edge / CDN]
          ↓
[API Go (Gin abstraído)]
          ↓ internal (pool)                    ↘ HTTPS (TLS 1.2+)
    [PostgreSQL 18]  [Redis 7]    [Stripe API]  [Mux API]  [Livekit self-hosted]  [Zitadel]
          ↓
    [Backups S3/cloud]
```

Boundaries:
- Cliente ↔ API: autenticação + ABAC
- API ↔ DB/Redis: pool com credenciais; rede privada Railway
- API ↔ Provider externo: HMAC webhook + API key
- Dev ↔ Railway: 2FA + audit log

---

## STRIDE por superfície

### Superfície: Endpoint público `/auth/*`

| Ameaça | Exemplo | Mitigação |
|---|---|---|
| **S**poofing | Request fingindo ser Zitadel callback | PKCE `code_verifier` em cookie encrypted; state validation |
| **T**ampering | Modificar `code` na URL | Code só vale 1x; TTL 10min |
| **R**epudiation | "Eu não fiz login ontem" | Audit log com IP + device fingerprint + timestamp |
| **I**nfo Disclosure | Token em logs | Proibido em log; verificação manual em code review |
| **D**oS | Brute force login | Rate limit `/auth/login` 20rpm/IP burst 5 |
| **E**oP | Privilege escalation via token forjado | Zitadel introspection em toda request auth (não confia em claims client-side) |

### Superfície: API `/api/v1/*`

| Ameaça | Exemplo | Mitigação |
|---|---|---|
| **S**poofing | Session hijack | httpOnly cookie Secure SameSite=Lax; 1-device policy |
| **T**ampering | Modificar body em transit | TLS 1.3; body size limit |
| **R**epudiation | Ação não rastreável | IAuditLogger (DT-010) |
| **I**nfo Disclosure | Cross-tenant read | ABAC + tenant_isolation; 100+ testes |
| **D**oS | Query expensiva (N+1) | Pagination cursor obrigatória; A-020 fix N+1 UPDATE |
| **E**oP | Morador vê dados de síndico | ABAC matrix; testes PBT |

### Superfície: Webhooks `/webhooks/stripe`, `/webhooks/mux`

| Ameaça | Exemplo | Mitigação |
|---|---|---|
| **S**poofing | Request forjando Stripe | HMAC verify **antes** de parse; timestamp < 5min |
| **T**ampering | Alterar evento | HMAC em body completo; idempotency via event.id |
| **R**epudiation | Evento duplicado processado 2x | `stripe_webhook_events` table com unique event.id |
| **I**nfo Disclosure | Payload vazado em erro | Log só metadata; nunca body cru |
| **D**oS | Replay massivo | Rate limit; idempotency descarta replay |
| **E**oP | Webhook "criando" subscription privilegiada | Webhook só **atualiza** estado; nunca cria novo user |

### Superfície: WebSocket `/api/v1/assemblies/:id/live`

| Ameaça | Exemplo | Mitigação |
|---|---|---|
| **S**poofing | Usuário não-elegível entrando | Token JWT Livekit gerado no backend após ABAC |
| **T**ampering | Votar com payload manipulado | Voto validado server-side com UoW + UNIQUE(agenda_item_id, voter_id) |
| **R**epudiation | "Não votei assim" | Audit log do voto com hash do payload + session |
| **I**nfo Disclosure | Outros moradores vendo fala privada | ACL do Livekit + room por condominium + papel |
| **D**oS | Flood de WebSocket | Rate limit conn + heartbeat timeout |
| **E**oP | Morador moderando assembleia | Role check explícito (sindico only) |

---

## LINDDUN (privacidade)

| Ameaça | Aplicação | Mitigação |
|---|---|---|
| **L**inkability | Correlacionar CPF entre tenants | CPF é único mas `user_id` UUID é usado externamente; audit opcional vincula |
| **I**dentifiability | Identificar user pelo padrão de uso | Métricas agregadas; sem tracking indivíduo |
| **N**on-repudiation (falha) | Não poder provar que X fez ação | Audit trail + signature (digital sign em atas) |
| **D**etectability | Saber que X existe no sistema | Busca só em `status=published`; perfis privados |
| **D**isclosure of information | Vazamento PII | Encryption at rest + in transit; no log |
| **U**nawareness | User não sabe que dados foram coletados | Termos + consent explícito no cadastro |
| **N**on-compliance | Violar LGPD | Consent, exclusão, portabilidade implementados |

---

## Superfícies específicas a vigiar

### Upload de vídeo (Mux)
- Saga A-027: órfão evitado via compensação
- Validação MIME/size no Mux direct upload
- Moderação manual antes de `published`
- Trava 90d por política de churn

### Assembleia Live (Livekit)
- Saga A-028: room órfão evitado via compensação
- JWT de participante gerado com ACL mínimo (seu microfone, seu chat)
- Retry + reconciliação em `ListParticipants` (A-033)
- `EndRoom` idempotente (A-034)

### Stripe Webhook
- Idempotência por `event.id`
- Processamento defensivo: se evento desconhecido, log warning mas 200 OK (evita retry infinito)

### Busca OpenSearch
- Resultados filtrados por tenant antes de retornar
- Não indexar dados sensíveis (email, telefone completos)

---

## Regras PBT (property-based) obrigatórias

1. **Fração ideal**: soma de units ≤ 100.00% em qualquer condomínio (agenda_item_pbt_test.go)
2. **Quotas billing**: `Consume` nunca deixa saldo < 0; `Unlimited` nunca decrementa
3. **ABAC matrix**: admin_bypass vence; tenant_mismatch sempre 403; role_not_allowed sempre 403
4. **Money BRL**: op sempre em int64 centavos; no floats
5. **Tenant isolation**: cross-tenant read/write sempre 403 em qualquer combinação

---

## Atualização

Este doc é revisado:
- Quando feature nova atravessa nova superfície
- Trimestralmente
- Em incident retrospective (se expôs ameaça nova)

## Links

- [[_moc]]
- [[BEYOND_CORP]]
- [[cve-process]]
- [[owasp-top10]]
- [[../11-audit/AUDIT]]
- [[../../../10-knowledge/security/_moc]]
