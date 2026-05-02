---
title: Zitadel — Antipatterns (o que não fazer)
type: note
tags: [provider, zitadel, identity, oidc, antipatterns]
source: https://zitadel.com/docs
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [Zitadel Antipatterns]
---

# Zitadel — Antipatterns

Erros recorrentes em integrações com Zitadel (ou qualquer IdP OIDC). Cada item tem o sintoma, por que é ruim e a correção canônica. Linkar para [[patterns]] e [[integration-guide]] quando couber.

## 1. Armazenar `access_token` em DB

**Sintoma**: tabela `sessions(access_token TEXT, ...)`.

**Por quê é ruim**:
- `access_token` é **curto** (minutos). Persistir gera lixo + ilusão de "ainda válido".
- Se for opaco, precisa introspection a cada uso — persistir não elimina round-trip.
- Se for JWT, leak do DB = tokens usáveis até `exp`. Sem `jti` revogação, não dá para invalidar.

**Correção**: persistir apenas `refresh_token`, **encriptado em repouso** (KMS/libsodium), indexado por `user_sub`. `access_token` vive em memória/cookie da request e some após uso.

## 2. `email` ou `preferred_username` como chave local

**Sintoma**: `users.email UNIQUE` como FK para permissões/relacionamentos.

**Por quê é ruim**: ambos são **mutáveis** no Zitadel (user troca email, admin renomeia username). Mudança quebra referências, histórico e audit trail.

**Correção**: `external_sub TEXT NOT NULL UNIQUE` como chave (do claim `sub`); `email` vira atributo de display, sincronizado. Ver [[patterns]] §4.

## 3. Validar JWT só localmente, sem introspection

**Sintoma**: middleware valida assinatura + `exp` e pronto.

**Por quê é ruim**: **não detecta revogação**. Logout, sessão expulsa, senha trocada — token continua válido até `exp`. Em incidente de segurança, a janela de exposição é o TTL do JWT.

**Correção**:
- Rotas sensíveis → introspection obrigatória (com cache 5 min).
- Rotas de alta frequência → assinatura local + cache + Action `post-revocation` invalidando cache do app.
- Nunca aceitar JWT "confiando no `exp`" em rota que escreve ou lê PII.

Ver [[patterns]] §1.

## 4. Cache de introspection com TTL longo

**Sintoma**: `cache.Set(token, claims, 1*time.Hour)`.

**Por quê é ruim**: revogação efetiva demora até o TTL expirar. Para produto com requisito BeyondCorp, é inaceitável.

**Correção**: TTL ≤ 5 min; invalidação ativa via `Action` webhook; rotas críticas bypassam cache.

## 5. ABAC no cliente

**Sintoma**: SPA decide "se role é X, mostra botão; se chamar endpoint sem role, 403 no front".

**Por quê é ruim**: **cliente é território inimigo**. Qualquer user abre DevTools, muda JS, chama endpoint. Regra canônica **precisa** estar no servidor.

**Correção**: matriz ABAC no middleware do backend; cliente só faz **ocultação de UI** (UX), nunca enforcement. Ver [[../../security/beyond-corp]].

## 6. `client_secret` em repositório ou log

**Sintoma**: `.env` commitado; log de debug com `Authorization: Basic ...`; secret em helm values.yaml no git.

**Por quê é ruim**: leak permanente. Rotacionar invalida tudo que usa; leak silencioso dura até auditoria.

**Correção**:
- Vault externo (1Password, Doppler, AWS Secrets, Railway secrets, K8s sealed secrets).
- `.env.example` commitado com placeholders; `.env` gitignored.
- Logger com redação de headers sensíveis (`Authorization`, `Cookie`, `X-Api-Key`).
- **Preferir `private_key_jwt`** sobre `client_secret` — chave pública no IdP, privada no cofre, sem senha compartilhada. Ver [[patterns]] §8.

## 7. Skip PKCE em SPA/mobile

**Sintoma**: `Application` do tipo Native com `authMethodType=basic` e `client_secret` embutido no app.

**Por quê é ruim**: binário do app / bundle JS é **público**. Secret embutido vaza por decompilação em minutos. Sem PKCE, o `code` interceptado no redirect basta pra roubar token.

**Correção**: `Application` tipo Native/SPA com `authMethodType=none`; PKCE `S256` obrigatório; redirect URI registrado de forma exata (custom scheme em mobile com verificação de assinatura).

## 8. Ignorar `state` e `nonce` no callback

**Sintoma**: callback só pega `code` e troca por token.

**Por quê é ruim**:
- Sem `state`, CSRF: atacante inicia fluxo, manda URL de callback para vítima logada, vítima autoriza code do atacante na conta dela.
- Sem `nonce`, replay: token capturado pode ser re-injetado em outro browser.

**Correção**: gerar `state` e `nonce` aleatórios por fluxo, guardar em cookie httpOnly assinado, validar no callback, rejeitar em mismatch.

## 9. Refresh sem rotation

**Sintoma**: app guarda refresh_token uma vez, usa para sempre; ignora o novo token que vem no response do refresh.

**Por quê é ruim**: Zitadel **invalida** o refresh antigo após uso. Na próxima chamada, `invalid_grant` — sessão morre. Pior: perde a proteção contra roubo (uso duplicado do mesmo refresh).

**Correção**: após cada refresh, sobrescrever o store com `next.RefreshToken` atomicamente. Tratar `invalid_grant` como sinal de roubo → revogar sessão inteira. Ver [[patterns]] §2.

## 10. Multi-tenant falso (um tenant único na Organization)

**Sintoma**: todos os users do produto numa única `Organization`; `tenant_id` é coluna no DB do app.

**Por quê é ruim**: perde isolamento de audit, branding, admin delegation e domain verification. Mistura audit trails entre tenants — leak de logs cruza fronteiras.

**Correção**: uma `Organization` por tenant do produto; `Project Grant` cede acesso. `tenant_id` local deriva de claim `urn:zitadel:iam:org:id`. Ver [[patterns]] §6.

## 11. Confiar em `email_verified` sem checar o claim

**Sintoma**: middleware pega `email` do token e confia que existe/é legítimo.

**Por quê é ruim**: `email` pode vir sem verificação. Atacante cria conta com email alheio, provider externo federado (social) não exige verificação em todos os casos.

**Correção**: sempre validar `email_verified == true` antes de usar email para qualquer decisão. Se `false`, bloquear ou pedir verify-email flow.

## 12. Audience (`aud`) não validado

**Sintoma**: middleware valida assinatura mas aceita qualquer `aud`.

**Por quê é ruim**: token emitido para `Application X` é usado em `Application Y` do mesmo tenant. Escalonamento lateral silencioso.

**Correção**: verifier configurado com `expectedAudience = <client_id>` (ou `projectId` no scope `urn:zitadel:iam:org:project:id:<id>:aud`). Rejeitar se `aud` não contiver o esperado.

## 13. Chave de machine user no filesystem sem rotação

**Sintoma**: `zitadel-key.json` gerado na setup, mountado no container, nunca rotacionado.

**Por quê é ruim**: chave eterna = leak eterno. Um dump de volume persiste acesso indefinido.

**Correção**: rotação trimestral (ou por política); gerar nova key no Zitadel, deploy com overlap (aceita as duas), revogar antiga. Alertar em métrica `zitadel.key.age_days > 90`.

## Vizinhos no grafo

- [[_moc]]
- [[overview]]
- [[integration-guide]]
- [[patterns]]
- [[../_moc]]
- [[../../security/beyond-corp]]
- [[../../security/_moc]]
