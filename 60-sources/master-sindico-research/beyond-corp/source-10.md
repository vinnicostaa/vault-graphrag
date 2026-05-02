---
title: "Cloudflare One / Cloudflare Zero Trust"
type: source
tags: [research, zero-trust, cloudflare, ztna, sase, access-proxy]
source: https://developers.cloudflare.com/cloudflare-one/
created: 2026-04-23
updated: 2026-04-23
---

# Fonte 10 — Cloudflare One / Zero Trust

## Metadata

- **Instituição**: Cloudflare, Inc.
- **URL principal**: https://developers.cloudflare.com/cloudflare-one/
- **URL reference arch**: https://developers.cloudflare.com/reference-architecture/implementation-guides/zero-trust/
- **URL ZT for SaaS**: https://developers.cloudflare.com/reference-architecture/design-guides/zero-trust-for-saas/
- **URL ZT for startups**: https://developers.cloudflare.com/reference-architecture/design-guides/zero-trust-for-startups/
- **URL ZTNA access policies**: https://developers.cloudflare.com/reference-architecture/design-guides/designing-ztna-access-policies/
- **Data de acesso**: 2026-04-23
- **Tipo**: documentação de produto + reference architecture

## Resumo

Cloudflare One é a plataforma SASE (Secure Access Service Edge) da Cloudflare que unifica segurança e rede em plano de controle único. É a **alternativa cloud-agnóstica** a Google IAP / AWS Verified Access — funciona em qualquer cloud ou on-prem.

**Componentes principais**:

- **Cloudflare Access** — ZTNA. Autentica usuários via IdP (OAuth/SAML) antes de liberar HTTP(S) para aplicação. Substitui VPN.
- **Cloudflare Tunnel (cloudflared)** — cria conexão outbound-only da infra pro edge Cloudflare; elimina necessidade de IP público ou porta aberta.
- **Secure Web Gateway (SWG) / Gateway** — inspeciona DNS/HTTP/egress, aplica filtros.
- **DLP (Data Loss Prevention)** — inspeção de conteúdo pra evitar exfiltração.
- **Remote Browser Isolation (RBI)** — navegador executa em sandbox remoto.
- **CASB (Cloud Access Security Broker)** — governança de SaaS.
- **Email Security** — Area 1 / filtering.
- **WARP client** — agent de device que roteia tráfego corp via Cloudflare; enables device posture.

**Fluxo ZTNA Cloudflare Access**:
1. Dev/admin cria aplicação em Access, define policy (ex.: "email @master-sindico.com.br + device WARP + país BR").
2. Usuário acessa `admin.master-sindico.com.br`; Cloudflare intercepta; se não logado, redireciona pro IdP.
3. Após authN, Cloudflare emite cookie assinado + JWT header ao backend.
4. Cloudflare Tunnel entrega request ao origin privado (sem IP público).

## Trechos-chave

> "Zero Trust (...) operates on the principle of least privilege. Rather than trusting users once connected, the system assumes that threats can exist both inside and outside the network. Therefore, every request is authenticated and authorized based on identity and context before granting access."
> — Cloudflare One docs

> "Access — Authenticate users accessing your applications, seamlessly onboard third-party users, and log every event."
> — Cloudflare One docs

> "Cloudflare Tunnel — outbound-only connections from your infrastructure to Cloudflare's global network."
> — Cloudflare One docs

> "Secure Web Gateway (SWG) — Inspect and filter DNS, network, HTTP, and egress traffic to enforce your company's Acceptable Use Policy."
> — Cloudflare One docs

> "The Cloudflare One Client enables organizations to build device posture rules, and enforce security policies anywhere by privately routing corporate device traffic through Cloudflare's network."
> — Cloudflare One docs

## Aplicabilidade ao Master Síndico

**Sim, alto — especialmente pra edge + ZTNA do admin console + fornecedores externos.**

Justificativa e casos de uso:

1. **Edge protection (produto público)**: Cloudflare CDN + WAF + Bot Management + DDoS protection cobrem o lado público dos BCs (web do morador, portal fornecedor, PWA institucional). Economiza custo vs CloudFront+AWS WAF e tem melhor UX.

2. **ZTNA admin console (equipe interna)**: Cloudflare Access protege `admin.master-sindico.com.br` sem VPN. Policy: "Google Workspace master-sindico + WARP device + país BR". Substitui VPN; onboarding de dev/suporte em minutos.

3. **ZTNA parceiros externos (B2B)**: administradoras grandes que precisam acessar dashboards consolidados podem ser onboard via Cloudflare Access com identidade própria (SAML federation) — sem criar contas no master-sindico e sem compartilhar credentials.

4. **Cloudflare Tunnel pra ambientes não-AWS**: se algum serviço legado rodar em VPS/on-prem (migração gradual), Tunnel expõe via Cloudflare sem abrir porta.

5. **Reference architecture "Zero Trust for SaaS"** da Cloudflare é literalmente um guia pra montar SaaS multi-tenant com ZT — **leitura obrigatória pra arquiteto de master-sindico**.

**Trade-off**: Cloudflare cria uma dependência crítica de edge. Em caso de incidente Cloudflare, o produto cai. Mitigação: multi-CDN (Cloudflare + fallback DNS), mas custo+complexidade. Pra M1, aceitar a dependência em troca de velocidade.

⚠️ PENDÊNCIA: ADR "Edge: Cloudflare vs CloudFront+WAF" — impacto financeiro e de UX.

## Tags/tópicos

- cloudflare
- zero-trust
- sase
- ztna
- cloudflare-access
- cloudflare-tunnel
- edge
- waf
- cloud-agnostic
