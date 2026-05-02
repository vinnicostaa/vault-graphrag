---
title: Glossário — Master Síndico
type: project
tags: [master-sindico, product, glossary, ubiquitous-language]
project: master-sindico
created: 2026-04-22
---

# Glossário — Master Síndico

Termos-chave. Versão navegável. Ubiquitous language detalhada em [[../01-domain/ubiquitous-language]].

---

## A

- **ABAC** — Attribute-Based Access Control. Decisão de autorização baseada em atributos (user, role, tenant, action, plan), não só role.
- **Administrador do condomínio** — sindônimo de Síndico (uso legal).
- **Agência de marketing** — persona que gerencia múltiplas empresas na plataforma.
- **Anti-Corruption Layer (ACL)** — adapter entre SDK externo e domínio (ex: `IStripeAdapter`).
- **Assembleia** — reunião formal de moradores, com pauta, votação, ata.
- **Ata** — documento formal publicado pós-assembleia; imutável; PDF assinado digitalmente.
- **AUDIT.md** — arquivo vivo de findings descobertos em revisão (A-###).

## B

- **BeyondCorp** — modelo Google de zero-trust; adaptado aqui como stack middleware + ABAC + 1-device.
- **Bounded Context (BC)** — pedaço do domínio com linguagem própria e fronteira explícita. No Master Síndico: identity, billing, institutional, commercial, content, assembly.

## C

- **Condomínio** — tenant primário; contém unidades, memberships, timeline.
- **Connect Me** — funcionalidade que permite síndico abrir "RFP" e receber propostas de empresas aprovadas.
- **Constituição (técnica)** — [[../STEERING]]; conjunto de regras invioláveis do projeto.
- **CNPJ** — Cadastro Nacional de Pessoa Jurídica (empresas).
- **CPF** — Cadastro de Pessoas Físicas.
- **CQRS** — Command Query Responsibility Segregation. Commands mudam estado; queries só leem.

## D

- **D-###** — ID de decisão viva em [[../STATE]].
- **Device Fingerprint** — hash do cliente (UA + hardware hint + etc.) usado na policy 1-device.
- **Diamante** — tier máximo de reputação (100+ contratos, 20+ condomínios, etc.).
- **Domain Event** — evento publicado pelo domínio quando algo significativo ocorreu (ex: `VoteCast`, `VideoApproved`).
- **DoD** — Definition of Done. Critério pra fechar task. Ver [[../DoR-DoD]].
- **DoR** — Definition of Ready. Critério pra começar task.
- **DPO** — Data Protection Officer (LGPD).
- **DT-###** — ID de dívida técnica declarada em STATE.

## E

- **Empresa condominial** — persona que presta serviço ao mercado condominial (portaria, limpeza, manutenção).
- **Escopo público** — superfície atingível sem auth (rotas `/auth/*`, `/webhooks/*`, `/health/*`).
- **Expand-contract** — padrão de migration destrutiva em 2 fases (adiciona novo → migra → remove antigo).

## F

- **Factory method** — `NewX` (cria + valida) e `ReconstructX` (hidrata sem validação).
- **Fração Ideal** — percentual de cada unidade no condomínio; soma = 100%; base pra quórum e rateio.

## G

- **GSD** — **G**et **S**hit **D**one. Metodologia de tasks pequenas, entrega incremental. Ambíguo com "GitHub Spec-Driven" (Spec Kit). Aqui usamos no sentido de ergonomia (dev eficiente).

## H

- **HMAC** — Hash-based Message Authentication Code. Usado em webhooks (Stripe, Mux).

## I

- **IAuditLogger** — interface de audit log cross-módulo (DT-010, Sprint 10).
- **Idempotency** — operação que repetida não muda o resultado (ex: webhook event.id único).
- **Institutional** — bounded context de condomínio/unidade/membership/timeline/plano diretor.
- **Invariante** — propriedade que sempre é verdadeira para o agregado (ex: "soma de fração ideal ≤ 100").

## J

- **Jornada** — sequência típica de ações de uma persona.

## K

- **Kiro** — IDE da AWS, formato de specs (`.kiro/specs/`). Base original das specs do projeto.

## L

- **LGPD** — Lei Geral de Proteção de Dados (Brasil). Análoga ao GDPR.
- **Livekit** — platform open-source de WebRTC; usado em assembleias live.
- **LMS** — Learning Management System. Plataforma de cursos (M3+).

## M

- **M1 / M2 / M3** — Marcos de entrega (ver [[../ROADMAP]]).
- **Membership** — vínculo user↔condomínio↔unidade com role (sindico, morador, …).
- **MOC** — Map of Content. Hub de navegação no Obsidian.
- **Morador** — persona; condômino com unidade.
- **Mux** — platform SaaS de processamento + streaming de vídeo (HLS).

## N

- **NFR** — Non-Functional Requirement (performance, security, a11y, i18n, etc.).
- **North-star metric** — métrica única que resume sucesso do produto.

## O

- **OIDC** — OpenID Connect. Extensão do OAuth 2.0 usada com Zitadel.
- **Ouro** — tier de reputação (10 contratos, 3 condomínios, 92% satisfação).

## P

- **Parceiro (de comércio local)** — persona; estabelecimento comercial próximo.
- **PBT** — Property-Based Testing. Testes de propriedades geradas aleatoriamente (`pgregory.net/rapid`).
- **PKCE** — Proof Key for Code Exchange. Extensão OAuth pra evitar code interception.
- **Plano Diretor** — plano plurianual do condomínio (obras, marcos) — parte do `institutional`.
- **Platina** — tier de reputação (30 contratos, 10 condomínios, 95%, ≥ 1 case vídeo).
- **Prata** — tier de reputação (3 contratos bem-avaliados).

## Q

- **Quórum** — mínimo de presença/voto necessário pra assembleia deliberar. Base em fração ideal.
- **Quota** — limite de uso por plano (Connect Me/mês, uploads/mês, storage).

## R

- **Reputação** — função determinística sobre métricas auditáveis (Bronze→Diamante).
- **Request ID** — identificador único por request, propagado em headers e logs.

## S

- **Saga** — padrão transacional distribuído com compensação explícita (usado em Mux, Livekit, Stripe).
- **SaaS** — Software as a Service. Modelo de entrega.
- **SAST** — Static Application Security Testing (gosec, Semgrep).
- **SCA** — Software Composition Analysis (govulncheck, osv-scanner).
- **SDD** — Spec-Driven Development. Ver [[../../../10-knowledge/methodology/spec-driven-development]].
- **Síndico** — persona principal; decisor do condomínio.
- **sqlc** — gerador de queries Go tipadas a partir de SQL.
- **STATE.md** — arquivo vivo de decisões (D-###), dívidas (DT-###), bloqueadores (B-###).
- **Steering** — regras duras; ver [[../STEERING]].
- **Stripe** — PSP (Payment Service Provider) usado para billing.

## T

- **Tenant** — condomínio. Isolamento estrito entre tenants.
- **Timeline** — sequência imutável de entries institucionais de um condomínio.
- **Token bucket** — algoritmo de rate limiting.
- **TOCTOU** — Time-Of-Check to Time-Of-Use. Classe de race condition (ver AUDIT A-025, A-026, A-029).
- **Trial** — período gratuito por persona (15d síndico, 7d empresa, 30d parceiro).

## U

- **Unidade (imóvel)** — apartamento/casa/sala no condomínio; tem fração ideal e owner.
- **UoW** — Unit of Work. Agrupa operações em uma transação DB.
- **UUIDv7** — UUID com prefixo timestamp; ordenável + sem colisão global.

## V

- **Value Object (VO)** — objeto imutável sem identidade, com regra própria (Email, CPF, Money).
- **Vizinhança** — feature de integração com comércio local (Parceiros).

## W

- **Webhook** — callback HTTP assíncrono (Stripe, Mux, Livekit). HMAC obrigatório.

## Z

- **Zitadel** — Identity Provider open-source (OIDC). Nossa stack de auth.

---

## Links

- [[_moc]]
- [[vision]]
- [[personas]]
- [[business-model]]
- [[../01-domain/ubiquitous-language]]
- [[../CLAUDE]]
