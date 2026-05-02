---
title: "BeyondCorp: A New Approach to Enterprise Security (Ward & Beyer, 2014)"
type: source
tags: [research, beyond-corp, zero-trust, google, canonical]
source: https://research.google/pubs/beyondcorp-a-new-approach-to-enterprise-security/
created: 2026-04-23
updated: 2026-04-23
---

# Fonte 01 — BeyondCorp: A New Approach to Enterprise Security

## Metadata

- **Autores**: Rory Ward, Betsy Beyer
- **Instituição**: Google Inc.
- **Publicação**: `;login:` The USENIX Magazine, Vol. 39, No. 6, Dezembro 2014, pp. 6–11
- **URL oficial (página Google Research)**: https://research.google/pubs/beyondcorp-a-new-approach-to-enterprise-security/
- **URL USENIX**: https://www.usenix.org/publications/login/dec14/ward
- **URL PDF**: https://www.usenix.org/system/files/login/articles/login_dec14_02_ward.pdf
- **Data de acesso**: 2026-04-23
- **Tipo**: whitepaper / paper de revista técnica

## Resumo

Paper inaugural da série BeyondCorp. Documenta a mudança de paradigma da segurança empresarial da Google após o ataque **Operation Aurora** (2009): sair do modelo perímetro-central (firewall delimita "intranet confiável") para um modelo onde **nenhum dispositivo ou rede é confiável por padrão**.

Os autores argumentam que virtualmente toda empresa usa firewalls pra impor segurança de perímetro, mas esse modelo falha porque, uma vez rompido o perímetro, o atacante tem acesso relativamente livre à intranet privilegiada. Com mobile e cloud, o perímetro torna-se impraticável.

A proposta é "remover a exigência de uma intranet privilegiada e mover as aplicações corporativas pra internet", confiando em autenticação forte de usuário + verificação de dispositivo + políticas de acesso dinâmicas no lugar de "estar na rede certa".

Componentes introduzidos:
- **Meta-inventory de dispositivos** com certificados digitais únicos.
- **Trust Inferrer** que avalia postura do device (patches, config, software instalado) e emite nível de confiança.
- **Access Proxy (AP)** como ponto único de entrada HTTPS aplicando políticas.
- **Access Control Engine (ACE)** como motor de decisão cruzando user × device × resource × context.
- **Tiers de acesso**: recursos exigem tier mínimo de confiança.

Foi seguido por uma série de papers (2016–2018) detalhando arquitetura, deploy, access proxy e UX.

## Trechos-chave

> "Virtually every company today uses firewalls to enforce perimeter security. However, this security model is problematic because, when that perimeter is breached, an attacker has relatively easy access to a company's privileged intranet."
> — Ward & Beyer, 2014, p. 6

> "As companies adopt mobile and cloud technologies, the perimeter is becoming increasingly difficult to enforce. Google is taking a different approach to network security. We are removing the requirement for a privileged intranet and moving our corporate applications to the Internet."
> — Ward & Beyer, 2014, p. 6

> "Access depends on the device and the user rather than the network from which the device connects."
> — síntese do modelo, reiterada em toda a série BeyondCorp

## Aplicabilidade ao Master Síndico

**Sim, canônico.**

Justificativa: o master-sindico é multi-tenant SaaS com 7 sub-produtos acessíveis de internet pública (síndicos em mobile, moradores em web/mobile, fornecedores Connect Me em web). **Não existe rede "interna" confiável** — toda requisição vem da internet. Os princípios do paper são 1:1 aplicáveis:

- O `condo_id` (tenant) **não é perímetro de confiança**; é chave de autorização.
- Identidade (JWT curto assinado pelo IdP) + device_id + ABAC são o perímetro real.
- A decisão "acesso permitido?" deve sair do PDP central, não de `if role == 'sindico'` espalhado.

Ver `_destilado.md` §2 pra mapeamento detalhado por BC.

## Tags/tópicos

- zero-trust / beyond-corp
- identity-as-perimeter
- device-trust
- access-proxy
- policy-engine
- enterprise-security
