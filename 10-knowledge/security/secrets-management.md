---
title: Secrets Management — infra-side (Vault, SOPS, KMS, PGP/age)
type: concept
tags:
  - security
  - secrets
  - secrets-management
  - vault
  - kms
  - pgp
  - sops
  - infra
scope: infra
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Secrets Management
  - Vault
  - SOPS
---

# Secrets Management

**O que é**: gestão do ciclo de vida de **credentials de infra/aplicação** — API keys, DB passwords, TLS private keys, webhook secrets, service account tokens. Diferente de [[credential-storage]] (client-side, user-facing).

> **Scope**: infra / dev / CI-CD. **Não** é client-side (cookie/localStorage/keychain de user). Ver [[credential-storage]] para isso.

## Princípios

1. **Zero secret em código** — nunca `.env` commitado, nunca hardcoded.
2. **Rotação periódica** — 30-90 dias padrão; imediato em breach.
3. **Least privilege** — escopo mínimo por credential.
4. **Audit trail** — quem acessou qual secret quando.
5. **Ephemeral > static** — preferir workload identity ([[spiffe-spire]]) a secret estático.

## Categorias de solução

### 1. **Secret manager SaaS**

| Produto | Scope | Diferencial |
|---|---|---|
| **HashiCorp Vault** | Multi-cloud + on-prem | Dynamic secrets (DB, cloud creds geradas on-demand) |
| **AWS Secrets Manager** | AWS | Integração IAM nativa; rotation Lambda-based |
| **AWS Parameter Store** | AWS | Mais barato para secrets simples |
| **Azure Key Vault** | Azure | HSM opcional; managed identity integration |
| **GCP Secret Manager** | GCP | Versioning + IAM binding |
| **Doppler** | Multi-cloud | UX developer-friendly; sync N targets |
| **1Password Secrets Automation** | Multi-cloud | Brand trust; ótima UX |
| **Infisical** | Open-source | Alternativa open a Doppler |

### 2. **Encrypted-at-rest em repo**

Commit criptografado no git; decrypt em deploy.

| Tool | Cifra | Uso |
|---|---|---|
| **[age](https://age-encryption.org/)** | X25519 + ChaCha20-Poly1305 | Modern, simple; substitui GPG |
| **PGP / GPG** | RSA/ECC + AES | Clássico; mais complexo |
| **SOPS** (Mozilla) | AES via KMS/PGP/age backend | Per-field encryption em YAML/JSON |
| **git-crypt** | AES-256 | Transparent encryption de arquivos |
| **Ansible Vault** | AES-256 | Bundle com Ansible |
| **sealed-secrets** (Bitnami) | RSA-OAEP | K8s-specific; one-way encrypt for cluster |

**SOPS** é o padrão-ouro atual — multi-backend (KMS, age, PGP), per-field, bom git-diff.

### 3. **KMS (Key Management Service)**

Não guarda o secret; guarda a **chave** que cripta/decripta secrets.

- **AWS KMS**: envelope encryption; FIPS 140-2 Level 3 opcional.
- **GCP Cloud KMS**: symmetric + asymmetric; HSM-backed opcional.
- **Azure Key Vault Keys**: HSM opcional.
- **HashiCorp Vault Transit** (on-prem).

**Envelope encryption**: KMS criptografa uma DEK (data encryption key); DEK criptografa o secret. Rotação de KEK não re-cripta todos os secrets.

### 4. **Workload identity (substitui secrets)**

Em vez de secret estático, workload tem identidade que permite **derivar credenciais temporárias**.

- AWS IAM Role for Service Accounts (IRSA) — pod K8s assume role via OIDC.
- GCP Workload Identity Federation — GitHub Actions → GCP sem chave JSON.
- [[spiffe-spire|SPIFFE/SPIRE]] — multi-cloud.
- Azure Managed Identity.

**Regra 2026**: onde workload identity existe, prefira a secret estático. Eliminar é melhor que rotacionar.

## PGP / age — casos de uso

### PGP / GPG (histórico)

- **Pros**: padrão maduro; web of trust.
- **Cons**: UX complexa; keys longas; muitas opções ruins (RSA-1024, fingerprint attacks).
- **Usar ainda para**: repos antigos, compliance que exige PGP específico, email encryption (que ninguém faz mais).

### age (moderno, recomendado)

- Pequeno (~1000 linhas Go vs milhões GPG).
- Chaves curtas (X25519).
- Simples: `age -r <recipient> -o secret.age < secret.txt`.
- Integrado em SOPS.
- **SSH keys como recipients** (reuso de infra existente).

```bash
# Criar par de chaves
age-keygen -o ~/.age/key.txt

# Criptografar para recipient
age -r age1q... -o secret.age < secret.txt

# Descriptografar
age -d -i ~/.age/key.txt secret.age > secret.txt
```

### PGP em git (signing)

Separado do secrets — `git commit -S` assina commit com GPG. Hoje **SSH signing** (`gpg.format = ssh`) ou **Sigstore** mais recomendado.

## Padrão canônico — app consome secret

### Pattern A: Secret manager SDK

```ts
import { SecretsManagerClient, GetSecretValueCommand } from '@aws-sdk/client-secrets-manager'

const client = new SecretsManagerClient({ region: 'us-east-1' })
const resp = await client.send(new GetSecretValueCommand({ SecretId: 'prod/db/password' }))
const password = resp.SecretString!

// Cache in memory; refresh on error
```

### Pattern B: Injection via ambiente (Doppler, K8s Secret)

```bash
# Doppler CLI — wraps process com env
doppler run -- node app.js
```

No Kubernetes:
```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: app-secrets
        key: db-password
```

### Pattern C: File mount (preferido em K8s para secrets grandes)

```yaml
volumes:
  - name: tls-cert
    secret: { secretName: app-tls }
volumeMounts:
  - name: tls-cert
    mountPath: /etc/tls
    readOnly: true
```

### Pattern D: Dynamic secrets (Vault)

Vault gera DB password on-demand com TTL 1h → app reconecta ao expirar.

```bash
vault read database/creds/app-role
# → { username: "v-app-xyz...", password: "...", lease: "1h" }
```

## Rotação — como fazer certo

### Strategy: 2-version overlap

```
t0: app usa secret v1
t1: gerar v2; app aceita v1 OU v2
t2: update app config para usar v2
t3: remover v1
```

Zero downtime. Aplicável a DB passwords, API keys, JWT signing keys.

### Automated rotation

- AWS Secrets Manager: Lambda customizável rota.
- Vault: dynamic secrets = rotação intrínseca.
- Doppler: rotation schedules.
- Manual: pior opção; operational burden.

## Anti-patterns

1. **`.env` commitado** — mesmo que gitignored depois, histórico git tem. Use BFG ou git-filter-repo para remover.
2. **Secret em CI variável plain** — GitHub Secrets OK (encrypted); outros CIs variam. Verifique.
3. **Secret em Dockerfile** (`ENV DB_PASSWORD=...`) — vaza em `docker history` + imagem. Use build secret (BuildKit) ou mount em runtime.
4. **Secret em log** — [[../runtime-antipatterns/op-022-log-com-pii|OP-022]]. Sanitize.
5. **Mesmo secret em dev + staging + prod** — breach em dev explode prod.
6. **Rotação manual esquecida** — automatize ou agende no calendário com owner.
7. **Sem audit de acesso** — não sabe se atacante já leu. KMS audit logs são obrigatórios.
8. **Secret em variável de ambiente ambient** (visível via `/proc/<pid>/environ` se atacante tem read) — preferir file mount ou runtime fetch.
9. **PGP/age com chave privada commitada** — autodestruição. Sempre gitignore private keys.
10. **Vault/Secrets Manager como "store de config"** — secrets são sensíveis; config comum vai em env vars ou ConfigMap.

## LGPD/GDPR

Secret que dá acesso a dados pessoais é vetor. Controles:
- Audit trail (quem acessou secret quando).
- Breach notification (72h GDPR, LGPD).
- Criptografia at-rest + in-transit.
- Retention policy: secrets antigos não precisam ficar.

## Como grandes empresas fazem

- **Netflix**: Metatron (interno) + AWS Secrets Manager + Vault. Dynamic credentials DB.
- **Airbnb**: Chef Vault (legado) → migrando para Vault + IAM.
- **GitHub**: Vault interno + repository secrets + Environment secrets.
- **Stripe**: internal secret manager; rotação stripe-created-keys frequente.
- **Shopify**: Doppler + 1Password Business.
- **Spotify**: Vault + service mesh (mTLS via SPIFFE).

## Relações

- **Client-side storage**: [[credential-storage]] (complementa — lado user)
- **Workload identity** (elimina secrets): [[spiffe-spire]], AWS IAM Roles, GCP WIF
- **Princípio**: [[least-privilege]]
- **Antipatterns runtime**: [[../runtime-antipatterns/op-022-log-com-pii|OP-022]], [[../runtime-antipatterns/op-023-webhook-sem-hmac|OP-023]]
- **Regras operacionais**: [[security-principles#Secrets-nunca-em-repo]]

## Fontes

- [HashiCorp Vault docs](https://developer.hashicorp.com/vault/docs) (consultada 2026-04-24)
- [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/) (consultada 2026-04-24)
- [GCP Secret Manager](https://cloud.google.com/secret-manager/docs) (consultada 2026-04-24)
- [SOPS — getsops](https://github.com/getsops/sops) (consultada 2026-04-24)
- [age encryption](https://age-encryption.org/) (consultada 2026-04-24)
- [Bitnami sealed-secrets](https://github.com/bitnami-labs/sealed-secrets) (consultada 2026-04-24)
- [NIST SP 800-57 — Key Management](https://csrc.nist.gov/publications/detail/sp/800-57-part-1/rev-5/final) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[credential-storage]]
- [[spiffe-spire]]
- [[least-privilege]]
- [[security-principles]]
