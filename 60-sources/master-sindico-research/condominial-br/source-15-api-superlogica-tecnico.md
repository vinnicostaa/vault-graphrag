---
title: Superlógica API v2 — análise técnica para integração
type: source
tags: [api, superlogica, integracao, oauth2, webhook, tecnico, br]
source: https://superlogica.com/api-superlogica-v2/
created: 2026-04-23
updated: 2026-04-23
aliases: [source-15-api-superlogica-tecnico, api-superlogica, superlogica-api]
fontes_consultadas:
  - https://superlogica.com/api-superlogica-v2/
  - https://apicondominios.superlogica.com/
  - https://assinaturas.superlogica.com/hc/pt-br/articles/360008275514-Integra%C3%A7%C3%B5es-via-API
  - https://imobiliarias.superlogica.com/hc/pt-br/articles/12249428568855-API-s-Integra%C3%A7%C3%B5es
  - https://blog.superlogica.com/assinaturas/webhooks-mais-integracao-superlogica-com-outros-sistemas/
  - https://www.organizemeucondominio.com.br/site/integracao-com-a-api-superlogica/
relevancia: alta
categoria: tecnico-integracao
---

# Superlógica API v2 — análise técnica para integração

## Arquitetura geral
- Construída com **Sensedia** (empresa brasileira especialista em APIs).
- Padrões: **RESTful, JSON, OAuth2, HTTPS**.
- 3 verticais de API:
  - `apicondominios.superlogica.com` (Condomínios)
  - `apiassinaturas.superlogica.com` (Assinaturas / SaaS)
  - `superlogicaimobiliarias.docs.apiary.io` (Imobiliárias)

## Autenticação
- **App Token** gerado no ERP:
  - Caminho: Todos os usuários (canto superior direito) → API → Aplicações → Novo App Token.
- **OAuth2** como padrão.
- Headers típicos: `app_token` + `access_token` (por cliente final).

## Recursos
- **Sandbox** público p/ desenvolvedores (evita contaminar dados reais em integração).
- **API Browser** embutido na documentação — simula requisições no browser.
- **Webhooks**: evento-driven (pagamento confirmado, novo morador, etc.) — disponível via "App Webhooks".
- Suporte técnico: `api@superlogica.com`.
- Documentação: `superlogica.com/developers`.

## Entidades esperadas (inferidas por integrações públicas)
- Condomínio
- Unidade (apartamento/loja)
- Morador/proprietário/inquilino
- Boleto / cobrança
- Movimentação financeira
- Prestador / fornecedor
- Assembleia + ata (provável)

## Integradores terceiros conhecidos
- **Organize Meu Condomínio** integra via API Superlógica.
- Ecosystem de parceiros é crescente.

## Recomendações para Master Síndico

### Estratégia de integração
1. **Master Síndico como camada de governança SOBRE o ERP** (não substituir):
   - Puxar dados financeiros read-only do Superlógica (boletos, inadimplência, fluxo).
   - Oferecer governança + comunicação + LMS + reputação sem duplicar o ERP.
   - Cliente: administradoras que já rodam Superlógica e querem UX moderno + compliance.

2. **Adapter/connector pattern** na arquitetura:
   - `erp.superlogica.Adapter` implementa interface comum `ERPAdapter`.
   - Similar adapter p/ Cond21, uCondo, Condomob (prioridade menor).
   - Evita lock-in de integração.

3. **Webhooks bidirecionais**:
   - Master Síndico publica eventos de governança (ata aprovada, mandato expirando).
   - Consome eventos financeiros do ERP (cobrança, pagamento).

4. **Consentimento do cliente** (administradora gera o App Token e dá pro Master Síndico).

### Risco competitivo
- Superlógica já vende Gruvi (app morador) + assembleia virtual (Zoom) → pode tentar competir com Master Síndico direto.
- **Diferencial sustentável**: governança + reputação + LMS + vizinhança são multi-layer; Superlógica não tem DNA de produto social/comunidade.
