---
title: Runbook — DNS + Domain Setup (Cloudflare zone + Registro.br)
type: guide
tags:
  - runbook
  - dns
  - cloudflare
  - registrobr
  - master-sindico
  - v2
source: Vault deploy-strategy + Cloudflare DNS docs + Registro.br guide
created: '2026-04-27'
updated: '2026-04-27'
---

# Runbook — DNS + Domain Setup

Procedimento end-to-end para configurar `mastersindico.com.br` na Cloudflare como zone canônica + delegar NS no Registro.br.

**Pré-requisito**: domínio `mastersindico.com.br` registrado em Registro.br pelo cliente.

**Bloqueador**: TODA a arquitetura D-138 depende disto. Faça PRIMEIRO.

## 1. Adicionar zone na Cloudflare

```bash
# Via dashboard (preferido):
# 1. https://dash.cloudflare.com → "Add a Site"
# 2. Digite: mastersindico.com.br
# 3. Plano: Free (suficiente; Pro só se Page Rules avançadas)
# 4. CF escaneia DNS existentes (importa A/CNAME/MX/TXT)

# Via API (alternativa CI):
curl -X POST https://api.cloudflare.com/client/v4/zones \
  -H "Authorization: Bearer $CF_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"name":"mastersindico.com.br","jump_start":true}'
```

CF retorna 2 nameservers (varia por conta), ex:
- `aria.ns.cloudflare.com`
- `walt.ns.cloudflare.com`

**Anote estes NS — próximo passo precisa.**

## 2. Delegar NS no Registro.br

**Quem**: João (titular do domínio).

```
1. Login https://registro.br
2. Painel → Domínios → mastersindico.com.br
3. "Alterar servidores DNS"
4. Substituir NS atuais pelos 2 NS Cloudflare anotados:
   ns1: aria.ns.cloudflare.com
   ns2: walt.ns.cloudflare.com
5. Salvar
6. Aguardar propagação 1-24h (.com.br via Registro.br pode levar 6-12h típico)
```

**Validar propagação**:
```bash
dig NS mastersindico.com.br +short
# Deve retornar os 2 NS Cloudflare; se ainda mostra Registro.br, aguardar
dig +trace mastersindico.com.br
```

## 3. Universal SSL (cert wildcard automático)

Após NS delegados:
- CF emite cert `mastersindico.com.br` + `*.mastersindico.com.br` automaticamente
- Validação via DNS-01 challenge (Let's Encrypt) ou DCV TXT (CF-managed)
- Tempo: 5-15min após NS ativos

**Validar**:
```bash
echo | openssl s_client -connect mastersindico.com.br:443 -servername mastersindico.com.br 2>/dev/null | openssl x509 -noout -subject -issuer -dates
# Issuer: C=US, O=Google Trust Services LLC ou Let's Encrypt (ambos OK via CF)
```

**SSL Mode**:
- Dashboard → SSL/TLS → Overview
- **Full (strict)** quando origin tiver cert válido (cloudflared + CF Origin CA cert recomendado)
- **Full** transitório M1 OK; **Strict** obrigatório M2 (vault [[../../08-security/lgpd]])

## 4. DNS records canônicos

Criar via dashboard DNS ou API. Todos com **proxy ativado** (orange cloud) exceto MX:

```
Type    Name              Value/Target                                    Proxy
─────   ─────             ─────                                           ─────
A       @                 (apex Pages — automático via Pages custom dom) Sim
CNAME   www               mastersindico-lp.pages.dev                      Sim
CNAME   auth              mastersindico-auth.pages.dev                    Sim
CNAME   app               mastersindico-cms.pages.dev                     Sim
CNAME   admin             mastersindico-admin.pages.dev                   Sim
CNAME   academia          mastersindico-lms.pages.dev                     Sim
CNAME   vizinhanca        mastersindico-forum.pages.dev                   Sim
CNAME   assembleia        mastersindico-assembly.pages.dev                Sim
CNAME   api               mastersindico-bff.workers.dev (route attach)    Sim
CNAME   webhooks          mastersindico-webhooks.workers.dev              Sim
CNAME   cdn               (R2 custom domain — configurado em R2 dashboard) Sim
MX      mail              route1.mx.cloudflare.net (priority 1)           Não (DNS only)
MX      mail              route2.mx.cloudflare.net (priority 2)           Não
MX      mail              route3.mx.cloudflare.net (priority 3)           Não
TXT     mail              "v=spf1 include:_spf.mx.cloudflare.net ~all"    Não
TXT     resend._domainkey "<DKIM key from Resend dashboard>"              Não
TXT     @                 "v=spf1 include:_spf.resend.com ~all"           Não
CAA     @                 0 issue "letsencrypt.org"                       Não
CAA     @                 0 issue "google.com"                            Não
CAA     @                 0 iodef "mailto:lgpd@mastersindico.com.br"      Não
```

## 5. Subdomínios via Pages Custom Domain (alternativa CNAME)

Para os 7 apps Pages, em vez de CNAME manual, usar **Pages → Custom Domain** (recomendado — CF gerencia cert per-domain):

```
Dashboard → Workers & Pages → mastersindico-<APP> → Custom Domains → Add
  Domain: <subdomain>.mastersindico.com.br
  CF cria CNAME automático apontando para o projeto Pages
```

## 6. DNSSEC (opcional M2)

Habilitar pré-prod aumenta segurança contra DNS spoofing:
- Dashboard → DNS → Settings → DNSSEC → Enable
- Copia DS record gerado
- Adicionar DS no Registro.br (Painel → Domínios → DNSSEC)
- Aguardar propagação 24h

**Validar**:
```bash
dig DS mastersindico.com.br +short
dig +dnssec mastersindico.com.br
```

## 7. Email Routing inbound

**Configuração** (dashboard → Email → Routing):
1. Enable Email Routing
2. CF adiciona MX records automáticos (linha 4 da tabela §4)
3. Custom addresses:
   - `lgpd@mastersindico.com.br` → forward DPO email
   - `suporte@mastersindico.com.br` → forward suporte email
   - `billing@mastersindico.com.br` → forward financeiro
   - `bounces+resend@mastersindico.com.br` → Worker handler `webhooks` (mark email inválido)
   - `bounces+twilio@mastersindico.com.br` → idem para SMS
4. Verify: enviar email teste para `lgpd@`; deve cair na caixa do DPO

## 8. SPF + DKIM + DMARC (outbound Resend)

**SPF** (linha 8 §4): autoriza Resend enviar em nome de `@mastersindico.com.br`.

**DKIM**: pegar valor no dashboard Resend → Domains → `mastersindico.com.br` → DKIM. Adicionar TXT record (linha 6).

**DMARC** (recomendado M2):
```
TXT  _dmarc  "v=DMARC1; p=quarantine; rua=mailto:lgpd@mastersindico.com.br; pct=100; sp=reject"
```

**Validar DKIM**:
```bash
dig TXT resend._domainkey.mastersindico.com.br +short
# Deve retornar a chave DKIM
```

## 9. Workers + Pages routes attach

Workers BFF (`api.`) e Webhooks (`webhooks.`) precisam route attached:

```jsonc
// wrangler.jsonc do BFF
{
  "routes": [
    { "pattern": "api.mastersindico.com.br/*", "custom_domain": true }
  ]
}
```

Após `wrangler deploy`, CF cria CNAME automático (sobrescreve manual se houver).

## 10. R2 custom domain

```
Dashboard → R2 → mastersindico-uploads → Settings → Custom Domains → Add
  Domain: cdn.mastersindico.com.br
  CF gera CNAME automático
```

## 11. Validação completa pós-setup

```bash
# Todos os subdomínios respondem com cert válido
for sub in www auth app admin academia vizinhanca assembleia api webhooks cdn; do
  echo "=== $sub ==="
  curl -I https://$sub.mastersindico.com.br -o /dev/null -s -w "Status: %{http_code} | Cert valid: %{ssl_verify_result}\n"
done

# DNSSEC (se habilitado)
dig +dnssec mastersindico.com.br | grep -E "AD flag|RRSIG"

# Email Routing
echo "Test inbound" | mail -s "Test LGPD" lgpd@mastersindico.com.br
# Verifica caixa do DPO em 1-2min
```

## 12. Anti-padrões

- ❌ Manter NS Registro.br + tentar DNS records via Cloudflare (não funciona)
- ❌ Proxy desligado (orange cloud cinza) em CNAMEs de Pages — perde WAF/DDoS/cache
- ❌ MX com proxy ativado (mail não funciona — proxy CF não roteia SMTP)
- ❌ Esquecer CAA — provider futuro pode não emitir cert
- ❌ Hardcode IP em DNS records (CF Pages/Workers usa anycast — IP muda)

## Links

- [[_moc]]
- [[../../06-execution/deploy-strategy]] §1, §2.1
- [[tunnel-cloudflared-aws]]
- Cloudflare DNS docs: https://developers.cloudflare.com/dns/
- Registro.br DNS: https://registro.br/dominio/regras/
- Universal SSL: https://developers.cloudflare.com/ssl/edge-certificates/universal-ssl/
- DNSSEC: https://developers.cloudflare.com/dns/dnssec/
- Email Routing: https://developers.cloudflare.com/email-routing/
- Resend DKIM setup: https://resend.com/docs/dashboard/domains/introduction
