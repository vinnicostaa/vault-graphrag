---
title: SAML 2.0 — Security Assertion Markup Language
type: concept
tags:
  - security
  - saml
  - sso
  - federation
  - enterprise
  - xml
doc-oficial: https://docs.oasis-open.org/security/saml/v2.0/
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - SAML 2
  - SAML
---

# SAML 2.0

**O que é**: standard OASIS (2005) para **federação de identidade** baseado em XML. SSO corporativo clássico; competidor direto de OIDC em contexto enterprise. Dominante em B2B/enterprise legacy; OIDC ganhou em B2C e novos sistemas.

> Em projetos **novos**, prefira [[oidc]]. SAML 2 é necessário quando integra com IdP enterprise legado (ADFS, Shibboleth, PingFederate) que não oferece OIDC.

## Papéis (análogos a OAuth)

- **IdP (Identity Provider)** — quem autentica.
- **SP (Service Provider)** — aplicação que consome a assertion.
- **Subject (Principal)** — usuário.

Não existe separação OAuth-like entre AS e RS; SAML é monolítico.

## Fluxos

### SP-Initiated SSO (mais comum)

```
1. User → SP: GET /resource
2. SP detecta sem sessão → redirect com AuthnRequest (XML assinado, codificado em URL)
3. User → IdP: /sso/saml?SAMLRequest=<base64-xml-deflated>
4. IdP autentica user
5. IdP → User: HTML form POST automático para SP com SAMLResponse (assinado)
6. User → SP: POST /acs (Assertion Consumer Service) com SAMLResponse
7. SP valida assinatura + audience + conditions → cria sessão
```

### IdP-Initiated SSO

User loga no IdP primeiro; clica app → IdP envia SAMLResponse direto ao SP (sem request). **Vulnerável por design** a CSRF — OAuth/OIDC evitaram por isso.

## Estrutura do SAMLResponse

XML com (simplificado):
```xml
<samlp:Response>
  <saml:Issuer>https://idp.example.com</saml:Issuer>
  <ds:Signature>...</ds:Signature>          <!-- opcional; pode assinar só Assertion -->
  <samlp:Status>
    <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success"/>
  </samlp:Status>
  <saml:Assertion>
    <saml:Issuer>https://idp.example.com</saml:Issuer>
    <ds:Signature>...</ds:Signature>
    <saml:Subject>
      <saml:NameID Format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress">
        user@example.com
      </saml:NameID>
      <saml:SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
        <saml:SubjectConfirmationData Recipient="https://sp.example.com/acs"
                                       NotOnOrAfter="2026-04-24T10:15:00Z"/>
      </saml:SubjectConfirmation>
    </saml:Subject>
    <saml:Conditions NotBefore="..." NotOnOrAfter="...">
      <saml:AudienceRestriction>
        <saml:Audience>https://sp.example.com</saml:Audience>
      </saml:AudienceRestriction>
    </saml:Conditions>
    <saml:AuthnStatement AuthnInstant="2026-04-24T10:00:00Z">
      <saml:AuthnContext>
        <saml:AuthnContextClassRef>
          urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport
        </saml:AuthnContextClassRef>
      </saml:AuthnContext>
    </saml:AuthnStatement>
    <saml:AttributeStatement>
      <saml:Attribute Name="email">...</saml:Attribute>
      <saml:Attribute Name="groups">...</saml:Attribute>
    </saml:AttributeStatement>
  </saml:Assertion>
</samlp:Response>
```

## Ataques clássicos (aprender para NÃO replicar)

### 1. **XML Signature Wrapping (XSW)**

Atacante insere assertion maliciosa em local diferente; lib valida **a primeira** mas SP processa a **maliciosa**. Históricamente afetou SAP, Office 365, vários IdPs/SPs.

**Mitigação**: usar libs atualizadas que fazem **XML canonicalization + verify reference URI + XPath seguro**. **Nunca** escrever parser SAML do zero.

### 2. **XML External Entity (XXE)**

Se parser aceita DTD/external entity: `<!ENTITY xxe SYSTEM "file:///etc/passwd">` vaza arquivos do servidor.

**Mitigação**: desabilitar DTD em parser (`setFeature("disallow-doctype-decl", true)`).

### 3. **Signature stripping / Comment injection**

Atacante usa comentários XML ou namespace redefinitions para bypass de validação.

**Mitigação**: canonicalization strict (C14N 1.1 ou XML-C14N).

### 4. **Replay attacks**

`NotOnOrAfter` muito longo + sem ID único invalidado = replay do SAMLResponse.

**Mitigação**: validar `ID` único + TTL curto (5min) + deny list de IDs usados.

### 5. **Audience confusion**

Response emitido para SP-A usado em SP-B.

**Mitigação**: validar `<AudienceRestriction>` **sempre**.

### 6. **Binding downgrade (HTTP-Redirect vs HTTP-POST)**

HTTP-Redirect binding passa SAMLRequest na URL (pode vazar). HTTP-POST é preferido; alguns SPs aceitam ambos → atacante downgrade.

**Mitigação**: SP declara `HTTP-POST-only` no metadata.

## Bindings (como SAML trafega)

- **HTTP-Redirect** — URL query string (param `SAMLRequest`/`SAMLResponse` deflate+base64). Para AuthnRequest (tamanho pequeno).
- **HTTP-POST** — form automático. Para SAMLResponse (tamanho grande com assinatura XML).
- **HTTP-Artifact** — SP recebe artifact curto; resolve via back-channel call ao IdP. Mais seguro; menos usado.
- **SOAP** — back-channel (attribute query, logout notification).

## Metadata XML

Cada parte publica metadata (XML) com endpoints, certificates, bindings suportados:
```xml
<EntityDescriptor entityID="https://sp.example.com">
  <SPSSODescriptor>
    <KeyDescriptor use="signing">...</KeyDescriptor>
    <AssertionConsumerService
      Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST"
      Location="https://sp.example.com/acs" index="0"/>
  </SPSSODescriptor>
</EntityDescriptor>
```

IdP importa metadata do SP (ou vice-versa) → auto-config. Rotação de cert por novo metadata.

## SAML 2 vs OIDC

| Dimensão | SAML 2 | OIDC |
|---|---|---|
| Formato | XML (verboso) | JSON + JWT (compacto) |
| Assinatura | XMLDSig (complexa) | JWS (simples) |
| Mobile-friendly | Ruim (XML grande + redirect) | Bom (JWT em Bearer header) |
| Discovery | Metadata XML estático | `/.well-known/openid-configuration` dinâmico |
| Tooling moderno | Poucas libs ativas | Abundante |
| Enterprise legacy | Dominante | Crescendo |
| API authorization | Não (é apenas AuthN) | Sim (via OAuth base) |
| Historicamente seguro | Vulnerabilidades XSW/XXE comuns | Menos superfície |

**Regra**: se projeto precisa suportar cliente corporativo com ADFS/Entra ID SAML, integre. Caso contrário, OIDC.

## Providers / IdPs

- **Entra ID (Microsoft)** — SAML + OIDC; enterprise dominante.
- **Okta / Auth0 / Ping** — SAML + OIDC; bridges perfeitas.
- **ADFS (Active Directory Federation Services)** — Microsoft on-prem; só SAML (AD FS 2012+ tem OIDC básico).
- **Shibboleth** — academic/research federations.
- **Keycloak / Zitadel** — open-source; suportam SAML + OIDC (SP ou IdP).

## SDKs (use libs ativas!)

| Linguagem | Lib |
|---|---|
| Node | `samlify`, `passport-saml` (ambas ativas) |
| Go | `crewjam/saml` |
| Python | `python3-saml` (OneLogin) |
| Java | OpenSAML, Spring Security SAML |
| Ruby | `ruby-saml` |

**Regra**: nunca implementar parser XML/assinatura do zero; lista acima é auditada.

## Grandes empresas — padrões

- **Salesforce, Workday, ServiceNow, SAP**: SAML-first como SSO B2B; OIDC crescendo em novos produtos.
- **Google Workspace / Microsoft 365**: SAML para integração com apps enterprise; OIDC para apps modernos.
- **AWS IAM Identity Center (antes SSO)**: SAML para federação de contas AWS; OIDC para apps.

## Anti-patterns

- **Implementar SAML do zero** — vetor garantido de XSW/XXE.
- **Aceitar HTTP-Redirect binding para SAMLResponse** — use POST apenas.
- **`NotOnOrAfter` em horas/dias** — TTL 5min.
- **Sem deny list de `ID` da assertion** — replay possível.
- **Não validar `AudienceRestriction`** — cross-SP.
- **Lib SAML desatualizada** — CVEs comuns (ex.: `python3-saml` teve CVE-2023).
- **IdP-Initiated SSO em produção nova** — prefira SP-Initiated por CSRF-safer.

## Relações

- **Alternativa moderna**: [[oidc]] (preferir em projeto novo)
- **Componente de SSO**: [[sso]]
- **Formato de prova**: diferente de [[jwt]] (SAML é XML)
- **Contexto**: [[iam]], [[identity-provider]], [[sso]] (conceito; ver `sso.md`)

## Fontes

- [SAML 2.0 Core — OASIS](https://docs.oasis-open.org/security/saml/v2.0/saml-core-2.0-os.pdf) (consultada 2026-04-24)
- [SAML 2.0 Bindings — OASIS](https://docs.oasis-open.org/security/saml/v2.0/saml-bindings-2.0-os.pdf) (consultada 2026-04-24)
- [SAML 2.0 Profiles — OASIS](https://docs.oasis-open.org/security/saml/v2.0/saml-profiles-2.0-os.pdf) (consultada 2026-04-24)
- [Awesome SAML — ataques documentados](https://github.com/mhaskar/Awesome-SAML) (consultada 2026-04-24)
- [SAML Raider — Burp plugin para pentest](https://github.com/CompassSecurity/SAMLRaider) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[oidc]] — alternativa moderna
- [[sso]]
- [[iam]]
- [[../providers/zitadel/_moc]]
