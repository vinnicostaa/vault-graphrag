---
title: Registration Data — Dados de Cadastro por Persona
type: requirement
tags: [requirements, registration, cadastro, onboarding, persona, master-sindico, v2]
source: legado contextos/DADOS PARA CADASTRO + ONBOARDING_CADASTRO_VS_PERFIL + institutional.md
created: 2026-04-23
updated: 2026-04-23
---

# Registration Data — Dados de Cadastro por Persona

Especificação detalhada de dados obrigatórios (CADASTRO — bloqueia ativação) e opcionais (PERFIL — não bloqueia) por persona. Derivado de `DADOS PARA CADASTRO.md` + `ONBOARDING_CADASTRO_VS_PERFIL.md` + `institutional.md` Req 6.2-6.4.

Organização: para cada persona — (1) Cadastro obrigatório, (2) Perfil opcional, (3) Certificações/marcadores, (4) Flags de controle, (5) Mensagens de erro.

Validação canônica:
- **CPF**: algoritmo dígito verificador (11 dígitos).
- **CNPJ**: algoritmo dígito verificador (14 dígitos) + Receita Federal (M2+).
- **CEP**: validação ViaCEP adapter (XD-018).
- **Email**: regex RFC 5322 simplificada + DNS MX (opcional).
- **Telefone**: E.164 (ex: `+5511999999999`).
- **Data**: ISO 8601 (`YYYY-MM-DD`).

> Ver [[functional/identity]] IDN-016, IDN-017 (cadastro mínimo + obrigatório) e IDN-029 (imutabilidade).

---

## 1. Morador (Base / Pagante)

### 1.1 Cadastro (obrigatório, bloqueia ativação)

| Campo | Tipo | Validação | Imutável? | Exemplo | Máscara |
|---|---|---|---|---|---|
| `name` | string ≥2 chars | trim + non-empty | ❌ | `João Silva` | — |
| `email` | string | regex RFC 5322 + Zitadel verified | ❌¹ | `joao@gmail.com` | — |
| `password` | string | Zitadel policy (8+, 1 maiúscula, 1 núm, 1 especial) | — | — | hidden |
| `cpf` | string | dígito verificador | ✓ | `123.456.789-00` | `###.###.###-##` |
| `birth_date` | date | ≥ 18 anos | ❌² | `1990-05-15` | `YYYY-MM-DD` |
| `phone` | string E.164 | regex + país BR | ❌ | `+5511988887777` | `+55 (##) #####-####` |
| `cep` | string 8 dígitos | ViaCEP valida | ❌ | `01310-100` | `#####-###` |
| `condominium_ref` | string (ID ou nome) | resolve via INS-042 | ❌ | `CD2024AB` ou `Edf. Primavera` | — |

**Notas**:
- ¹ Email mudança via Zitadel (re-verify).
- ² Campo protegido por audit (log alteração).
- **Termo LGPD obrigatório** (IDN-024, LGPD-001).

### 1.2 Perfil (opcional, não bloqueia)

| Campo | Tipo | Restrição |
|---|---|---|
| `photo` | upload | 5MB max, JPG/PNG/WebP, ≥200×200 |
| `endereco_completo` | string | até 200 chars |
| `video_curriculum_id` | ref video | morador pagante apenas (COM-029) |
| `professional_bio` | text | até 500 chars (pagante para Banco Talentos) |
| `area_atuacao` | enum | pagante para busca em Banco Talentos |
| `experiencia_profissional` | text | até 2000 chars |
| `notificacoes_preferencias` | JSON | default opt-in email, opt-out SMS |

### 1.3 Flags de controle

- `status`: `pending_verification | incomplete | active | suspended | deleted`
- `visibility_ready`: boolean (true quando CPF + nascimento + condomínio + endereço básico)
- `verified`: boolean (email Zitadel verified)
- `banned`: boolean
- `ban_expires_at`: timestamptz NULL
- `role`: `resident` (imutável)
- `role_type`: `titular | dependent` (derivado do membership)

### 1.4 Mensagens de erro canônicas (pt-BR)

```
"cpf_invalido" → "CPF inválido. Verifique os dígitos e tente novamente."
"cpf_ja_cadastrado" → "Este CPF já está cadastrado. Faça login ou recupere sua senha."
"email_invalido" → "Email em formato inválido."
"email_nao_verificado" → "Confirme seu email antes de prosseguir. Reenviamos o link?"
"cep_nao_encontrado" → "CEP não localizado. Verifique e tente novamente."
"condominio_nao_encontrado" → "Condomínio não localizado. Confirme o código com seu síndico."
"senha_fraca" → "Senha deve ter 8+ caracteres, 1 maiúscula, 1 número, 1 especial."
"idade_minima" → "Cadastro permitido apenas para maiores de 18 anos."
"telefone_invalido" → "Telefone em formato inválido. Use DDD + número."
"termo_lgpd_obrigatorio" → "Aceite os termos de uso e política de privacidade para continuar."
```

---

## 2. Síndico (N1 / N2 / N3 — morador ou profissional)

### 2.1 Cadastro (obrigatório, bloqueia ativação)

| Campo | Tipo | Validação | Imutável? | Exemplo |
|---|---|---|---|---|
| `name` | string ≥2 chars | trim | ❌ | `Maria Oliveira` |
| `professional_email` | string | regex + Zitadel verified | ❌ | `maria@msindico.com.br` |
| `password` | string | policy Zitadel | — | — |
| `cpf` OR `cnpj` | string | dígito verificador | ✓ | `123.456.789-00` |
| `birth_date` | date | ≥ 18 anos | ❌ | `1985-03-22` |
| `professional_address` | string | não-vazio + CEP válido | ❌ | `Rua X, 100, SP/SP` |
| `professional_cep` | string 8d | ViaCEP | ❌ | `04567-890` |
| `phone` | string E.164 | regex | ❌ | `+5511999887766` |
| `city_state_of_operation` | string (cidade/UF) | vocabulary BR | ❌ | `São Paulo/SP` |
| `sindico_type` | enum | `eleito, profissional, interino` | ❌³ | `profissional` |
| `years_of_experience` | int | 0-60 | ❌ | `10` |
| `condominiums_under_management` | int | 0-15 | ❌ | `3` |

**Nota** ³: sindico_type pode mudar ao longo da carreira (via ticket), mas marcado como imutável no primeiro cadastro pra governança.

### 2.2 Perfil (opcional, não bloqueia)

| Campo | Tipo | Restrição |
|---|---|---|
| `professional_photo` | upload | 5MB max, ≥400×400 |
| `mini_bio_profissional` | text | ≤ 500 chars (N2/N3) |
| `video_institucional_id` | ref video | N2/N3 apenas, lock 90d (CNT-004) |
| `formacao_certificacoes` | JSON array | `[{certificacao, data_obtencao, orgao}]` |
| `administradoras_vinculadas` | JSON array | IDs de empresas síndicas |
| `areas_especializacao` | JSON array | enum (administrativa, técnica, jurídica, etc.) |
| `disponibilidade_contato` | enum | horário preferencial |

### 2.3 Governance Markers (15 autodeclarados, checkboxes)

| Marcador | Tipo | Exibição |
|---|---|---|
| `membro_abracs` | boolean | Selo ABRACS |
| `seguro_responsabilidade_civil` | boolean | Ícone seguro |
| `assessoria_juridica` | boolean + text (qual) | Ícone jurídico |
| `assessoria_contabil` | boolean + text | Ícone contábil |
| `assessoria_seguranca_trabalho` | boolean + text | Ícone segurança |
| `conformidade_lgpd` | boolean | Selo LGPD |
| `conformidade_nr1` | boolean | Selo NR-1 |
| `programa_compliance` | boolean | Selo compliance |
| `selo_reclame_aqui` | boolean + link | Widget RA |
| `outra_certificacao` | text livre (max 500 chars) | Lista |
| `premiacoes` | text livre (max 500 chars) | Lista |

**Disclaimer fixo exibido em UI**: _"Marcadores são informativos, baseados em autodeclaração, e não representam certificação oficial da plataforma."_

### 2.4 Flags de controle

- `status`: `pending_verification | incomplete | pending_payment | active | suspended | deleted`
- `visibility_ready`: documento + tipo + cidade/estado + endereço profissional
- `verified`: boolean
- `banned`: boolean
- `mfa_enabled`: boolean (recomendado N3)
- `role`: `syndic` (imutável)
- `role_type`: `principal | auxiliary | assistant` (por condomínio)
- `plan_tier`: `syndic_n1 | syndic_n2 | syndic_n3` (imutável após conversão paid)

### 2.5 Mensagens de erro

```
"cpf_cnpj_invalido" → "Documento inválido. Verifique e tente novamente."
"email_profissional_invalido" → "Email profissional em formato inválido."
"data_nascimento_invalida" → "Data de nascimento inválida ou menor de 18 anos."
"anos_experiencia_invalido" → "Anos de experiência entre 0 e 60."
"condominios_exceded" → "Limite de 15 condomínios por conta. Contate suporte para exceção."
"tipo_sindico_obrigatorio" → "Selecione o tipo de síndico (eleito, profissional ou interino)."
```

---

## 3. Empresa do Mercado Condominial (Plus / Pro)

### 3.1 Cadastro (obrigatório)

| Campo | Tipo | Validação | Imutável? |
|---|---|---|---|
| `razao_social` | string | non-empty | ❌⁴ |
| `representante_legal_email` | string | regex + Zitadel verified | ❌ |
| `password` | string | policy Zitadel | — |
| `cnpj` | string | dígito verificador + Receita M2+ | ✓ |
| `data_aniversario_empresa` | date | data passada | ❌ |
| `representante_legal_nome` | string | non-empty | ❌ |
| `email_comercial` | string | regex | ❌ |
| `telefone_comercial` | string E.164 | regex | ❌ |
| `endereco_comercial_cep` | string 8d | ViaCEP | ❌ |
| `endereco_comercial` | string | resolvido via CEP | ❌ |
| `financeiro_nome` | string | non-empty | ❌ |
| `financeiro_telefone` | string E.164 | regex | ❌ |
| `financeiro_email` | string | regex | ❌ |
| `categoria_principal` | enum | config plataforma | ❌⁵ |
| `subcategorias_atuacao` | array | até 5 items | ❌⁵ |
| `cidades_estados_atuacao` | array | {cidade, UF} | ❌ |

**Notas**:
- ⁴ Razão social normalmente não muda; alteração via suporte (lei).
- ⁵ Categorias definidas pelo Admin (ver CNT-029).

### 3.2 Perfil (opcional)

| Campo | Tipo | Restrição |
|---|---|---|
| `logo` | upload | 5MB, ≥200×200 |
| `nome_fantasia` | string | até 100 chars |
| `descricao_institucional` | text | até 2000 chars (sem contato/preço — COM-003) |
| `videos_institucionais` | ref array | lock 90d cada (CNT-004), quota plano |
| `videos_tecnicos` | ref array | quota plano |
| `portfolio_fotos` | upload array | até 10, 5MB cada |
| `portfolio_cases` | text/JSON | estudos de caso |

### 3.3 Compliance Markers (25 autodeclarados)

**ISOs e certificações**:
- `iso_9001`, `iso_14001`, `iso_45001`, `iso_37001`, `iso_19600_37301`

**Boolean checkboxes**:
- `seguro_responsabilidade_civil`, `assessoria_juridica`, `assessoria_contabil`, `assessoria_seguranca_trabalho`, `responsavel_tecnico`, `regularidade_conselhos_profissionais`, `conformidade_lgpd`, `conformidade_nr1`, `programa_compliance`, `esg`, `cipa`, `selo_verde_sustentavel`, `selo_empresa_amiga_crianca`, `selo_carbon_free`, `selo_eureciclo`, `selo_reclame_aqui`

**Texto livre**:
- `nrs` (lista de NRs aplicáveis)
- `outras_certificacoes` (até 500 chars)
- `certificacao_propria` (até 500 chars)
- `premiacoes` (até 500 chars)

**Disclaimer**: _"Certificações são autodeclaradas, não certificadas por terceiro."_

### 3.4 Flags

- `status`: `pending_verification | incomplete | pending_payment | active | suspended | blocked | deleted`
- `visibility_ready`: CNPJ verificado + endereço comercial + nome fantasia
- `cnpj_verified`: boolean (Receita Federal stub M1, pleno M2+)
- `plan_tier`: `enterprise_plus | enterprise_pro`
- `reputation_tier`: `bronze | prata | ouro | platina | diamante` (derivado, GR-028)
- `suspended_until`: timestamptz NULL (harassment cooldown)
- `role`: `enterprise` (imutável)
- `role_type`: `administrator | operator`

### 3.5 Mensagens de erro

```
"cnpj_invalido" → "CNPJ inválido. Verifique os dígitos."
"cnpj_ja_cadastrado" → "Este CNPJ já está cadastrado na plataforma."
"cnpj_inativo" → "CNPJ consta como inativo na Receita Federal. Regularize antes de prosseguir."
"categorias_excedidas" → "Selecione até 5 subcategorias de atuação."
"email_representante_legal_invalido" → "Email do representante legal inválido."
"endereco_comercial_obrigatorio" → "Endereço comercial com CEP é obrigatório."
```

---

## 4. Parceiro da Vizinhança (Comércio Local)

### 4.1 Cadastro (obrigatório)

| Campo | Tipo | Validação | Imutável? |
|---|---|---|---|
| `razao_social_nome` | string | non-empty | ❌ |
| `representante_legal_email` | string | regex + Zitadel | ❌ |
| `password` | string | policy | — |
| `cnpj_ou_cpf` | string | dígito verificador | ✓ |
| `data_aniversario` | date | data passada | ❌ |
| `representante_legal_nome` | string | non-empty | ❌ |
| `endereco_cep` | string 8d | ViaCEP | ❌ |
| `endereco` | string | via CEP | ❌ |
| `financeiro_nome` | string | non-empty | ❌ |
| `financeiro_telefone` | string E.164 | regex | ❌ |
| `financeiro_email` | string | regex | ❌ |

### 4.2 Perfil (opcional)

| Campo | Tipo | Restrição |
|---|---|---|
| `logo` | upload | 5MB, ≥300×300 |
| `nome_fantasia` | string | até 100 chars |
| `email_comercial` | string | regex |
| `telefone_comercial` | string E.164 | regex |
| `whatsapp` | string E.164 | regex |
| `instagram` | string | handle |
| `fotos` | upload array | até 3 fotos, 5MB cada |
| `descricao` | text | até 1000 chars |
| `horario_funcionamento` | JSON | por dia da semana |

### 4.3 Flags

- `status`: `pending_verification | incomplete | pending_payment | active | suspended | deleted`
- `visibility_ready`: documento + nome fantasia + endereço
- `seal_syndic_approved`: boolean (via votação síndicos N2+)
- `plan_tier`: `local_company_standard`
- `role`: `local_company` (imutável)

### 4.4 Mensagens de erro

```
"documento_invalido" → "CPF ou CNPJ inválido."
"endereco_obrigatorio" → "Endereço com CEP é obrigatório para aparecer no feed de vizinhança."
```

---

## 5. Agência de Marketing

### 5.1 Cadastro (obrigatório)

| Campo | Tipo | Validação | Imutável? |
|---|---|---|---|
| `razao_social` | string | non-empty | ❌ |
| `representante_legal_email` | string | regex + Zitadel | ❌ |
| `password` | string | policy | — |
| `cnpj` | string | dígito + Receita M2+ | ✓ |
| `data_aniversario_empresa` | date | — | ❌ |
| `representante_legal_nome` | string | non-empty | ❌ |
| `email_comercial` | string | regex | ❌ |
| `telefone_comercial` | string E.164 | regex | ❌ |
| `endereco_comercial_cep` | string 8d | ViaCEP | ❌ |
| `financeiro_nome/telefone/email` | — | regex | ❌ |
| `cidade_estado_atuacao` | string | vocabulary BR | ❌ |

**Nota**: Agência NÃO se cadastra via fluxo público. Cadastro via invite token emitido por Empresa Plus/Pro (IDN-025).

### 5.2 Perfil (opcional)

| Campo | Tipo | Restrição |
|---|---|---|
| `logo` | upload | 5MB |
| `nome_fantasia` | string | — |
| `portfolio` | text/refs | clients/cases |
| `especialidades` | array enum | marketing digital, SEO, vídeo, etc. |

### 5.3 Flags

- `role`: `marketing` (imutável)
- `status`: herda via token
- `managed_enterprises`: array de `company_id` que agência gerencia
- `token_expires_at`: timestamptz (TTL 90d default)
- `scopes`: array (`videos:upload, videos:edit, analytics:read`)

### 5.4 Mensagens de erro

```
"convite_expirado" → "Convite expirado. Solicite novo convite à empresa."
"scope_insuficiente" → "Você não tem permissão para essa ação via convite da empresa."
```

---

## 6. Admin Master Síndico (interno)

### 6.1 Dados

| Campo | Tipo | Nota |
|---|---|---|
| `name` | string | funcionário |
| `email` | string (`@mastersindico.com.br`) | corporativo |
| `password` | string | policy + rotação |
| `admin_type` | enum | `super_admin, moderator, support, financeiro` |
| `mfa_enabled` | boolean | **obrigatório M2+** |
| `ip_allowlist` | array (super_admin M3+) | IPs corporativos |

**Não passa por fluxo público**. Provisionado via Zitadel admin.

### 6.2 Flags

- `role`: `admin` (imutável)
- `admin_bypass`: boolean (super_admin)
- `mfa_enabled`: true sempre
- `scopes_admin`: array granular (modular admin powers)

### 6.3 Mensagens de erro

```
"acesso_admin_negado" → "Acesso negado. Autentique-se com MFA."
"ip_nao_autorizado" → "Acesso admin restrito a IPs corporativos."
```

---

## Lógica de front-end (sem código)

### Fluxo canônico por persona

1. **Cadastro mínimo** (IDN-016): `{name, email, password, role}` → Zitadel → verificação email.
2. **Verificação email** (IDN-002): modal bloqueante até confirm.
3. **Onboarding por etapas** (IDN-017, IDN-019):
   - **Etapa A**: Documento (CPF/CNPJ) + campos obrigatórios CADASTRO.
   - **Etapa B**: Pagamento (se plano pago).
   - **Etapa C**: Customização PERFIL (opcional).
4. **Ativação**: `status → active` quando CADASTRO + pagamento ok.
5. **Visibility ready**: quando PERFIL mínimo preenchido → aparece em busca.

### Regras de bloqueio

- Login permitido após email verified.
- Perfil só aparece em busca se `visibility_ready = true`.
- Features premium só liberam se `status = active AND plan_tier OK`.

---

## Sugestão de schema (Drizzle/sqlc alignment)

**Observação**: schema atual do legado (TS) tem várias colunas NOT NULL. Migrar pra Go:

- **Opção 1** (recomendada): criar perfil SOMENTE quando CADASTRO completo. Campos customização remain NULL.
- **Opção 2**: permitir NULL em perfil (migration).

Decisão: **Opção 1** — perfil só existe com CADASTRO completo; customização via subsequent PATCH.

---

## Pendências

- ⚠️ **PENDÊNCIA Pagantes / categorias empresa**: lista exata de `categoria_principal` enum — dual-check com stakeholder.
- ⚠️ **PENDÊNCIA Governance markers**: total de 15 no legado — reconciliar se inclui 2-3 texto livre + 12 boolean ou outra fatiação.
- ⚠️ **PENDÊNCIA Compliance markers empresa**: confirmar lista de 25 (atualmente 23 + outras + premiações + cert_propria = 26 — dual-check).
- ⚠️ **PENDÊNCIA Morador Pagante vs Síndico N2**: sobreposições (ex: vídeo institucional síndico vs vídeo empresa) — verificar validadores.
- ⚠️ **PENDÊNCIA Mensagens erro i18n**: vocabulary final de códigos de erro — ADR i18n.

---

## Referências

- Legado: `DADOS PARA CADASTRO.md`, `ONBOARDING_CADASTRO_VS_PERFIL.md`, `institutional.md` Req 6.2-6.4.
- [[functional/identity]] IDN-016, IDN-017, IDN-019, IDN-029.
- [[functional/institutional]] INS-039, INS-040, INS-041.
- [[functional/commercial]] COM-001, COM-002, COM-030.
- [[matrix-functional]] — features por persona.
- [[../00-product/personas]] — descrição das personas.
- [[global]] GR-023 (termos versionados).
- [[_moc]]
