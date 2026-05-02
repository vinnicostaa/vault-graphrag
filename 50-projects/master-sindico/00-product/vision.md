---
title: Vision — Master Síndico
type: note
tags: [product, vision, master-sindico, v2]
source: vault/50-projects/master-sindico/00-product/vision.md
created: 2026-04-23
updated: 2026-04-23
---

# Visão do Produto — Master Síndico

## O que é

**Master Síndico** é uma plataforma SaaS de **governança condominial** (Brasil). Conecta **síndicos, moradores, empresas do mercado condominial, agências de marketing e comércio local** num ecossistema unificado com foco em três dores centrais:

1. **Critério na contratação** — via sistema de reputação auditável (Bronze → Prata → Ouro → Platina → Diamante) calculado deterministicamente a partir de métricas (contratos fechados, avaliações recebidas, condomínios atendidos, casos em vídeo).
2. **Memória institucional auditável** — timeline imutável append-only que registra toda decisão/evento relevante da gestão condominial; governança não sofre revisão retroativa.
3. **Ecossistema integrado** — perfis públicos, vídeos institucionais, cursos (LMS), contato direto estruturado (Connect Me), banco de talentos (morador vídeo-currículo), descoberta local (Vizinhança).

## O que NÃO é

- ❌ **ERP condominial** — não faz cobrança boleto, emissão fiscal, folha de pagamento ou controle financeiro contábil. (Concorrentes ERP: Superlógica, Igora, Cond21.)
- ❌ **App de comunicação interna** — Connect Me é formulário unidirecional RFP, não chat/WhatsApp substituto. (Concorrente comunicação: TownSq.)
- ❌ **Sistema financeiro** — não intermedia pagamento entre síndico e empresa (só sua assinatura via Stripe).
- ❌ **Plataforma genérica de videoconferência** — assembleia live (Livekit) é vertical dedicada a votação/ata, não Meet-substituto.
- ❌ **Rede social pública** — conteúdo escopado por condomínio (timeline, comunicados) ou por perfil profissional (empresa).

## Invariante-chave do produto

> **Identidade é única (1 CPF ou 1 CNPJ = 1 User), contextos de operação são múltiplos.**

O mesmo indivíduo pode ser:
- Síndico no Condomínio X (role institutional)
- Morador no Condomínio Y (role institutional)
- Representante legal da Empresa Z (role commercial)
- Admin da Master Síndico (role operational — interno da plataforma)

Sem duplicação de cadastro. A identidade é o núcleo; os **memberships** são projeções sobre tenants diferentes. Cada membership carrega role + plano + quotas independentes.

## Pilares de governança aplicada

### 1. Critério

- Reputação **determinística**: fórmula pública auditável, não ML proprietário. Evita fraude, evita opacidade.
- Escala: Bronze (default), Prata (≥3 contratos), Ouro (≥10), Platina (≥30), Diamante (≥100). Parâmetros exatos destilados em [[../01-domain/business-rules]].
- Visível em perfil, search, Connect Me. Afeta ranking em marketplace.

### 2. Registro

- Timeline **imutável** por condomínio: INSERT-only, sem UPDATE, sem DELETE, sem `deleted_at`. Correção vira nova entry.
- Events registrados: decisões de síndico, assembleias, contratos fechados, avaliações emitidas, comunicados publicados.
- Base da "prova documental" que diferencia o produto de ERPs.

### 3. Reputação

- Função pura de inputs auditáveis: contratos, avaliações, condomínios atendidos, vídeos institucionais publicados.
- **Nenhum input humano direto** altera tier (proíbe "pagou X, virou Diamante" e "síndico rival baixou reputação por birra"). Só os inputs alimentam a fórmula.
- Denúncia de harassment (Sprint 4 legado) pode suspender temporariamente enquanto moderação manual 48h resolve.

### 4. Integração

- Connect Me conecta síndico ↔ empresa em 4 fluxos formais unidirecionais. Detalhes: [[../04-requirements/functional/commercial]] (quando destilado) ou legado `client-vault/02 - Domínios e Requisitos/Domínio - Commercial.md`.
- Banco de Talentos conecta morador ↔ empresa via vídeo-currículo 90s (lock 90d).
- Vizinhança conecta comércio local ↔ morador do condomínio (geográfico, raio).

## Fora de escopo (guardrails)

Entradas que periodicamente aparecem como pedido mas **não entram no MVP (M1/M2/M3)**:

- **Assembleia online/híbrida completa** — MVP assembleia presencial com live streaming opcional (backend Livekit existe; UI mobile/web só M3+). Online pleno backlog M4+.
- **Empresa-síndica** (empresa que gerencia múltiplos condomínios como síndica profissional delegada) — backlog M5+.
- **Agência de Marketing como tenant multi-empresa próprio** — tratada como actor delegado (impersonation / scoped tokens) no MVP. Tenant próprio só com demanda validada (M5+).
- **LMS pleno** (pós-graduação síndico com certificação ABRACS) — M3 thin (infraestrutura + 3 trilhas piloto); maturidade M4+.
- **Moderação ML** — manual 48h em M1/M2/M3; ML pipeline (OpenAI Moderation / AWS Rekognition / Hive) a partir de M4 quando volume justifica.
- **Multi-região / multi-país** — BR only em M1-M6. Sem i18n pleno no MVP (só pt-BR).
- **Marketplace financeiro** (split de pagamento entre síndico-empresa via plataforma) — backlog M5+ se demanda regulatória BR permitir.
- **Integração com ERP** (Superlógica, Igora) — backlog M4+ se parceria comercial acontecer.

## Métricas de sucesso (a definir quantitativos na Fase 4)

Qualitativos estabilizados:

- Síndico consegue contratar empresa verificada em < 5 min (do RFP Connect Me ao contrato fechado).
- Morador consegue acompanhar timeline da gestão sem pedir explicação ao síndico.
- Empresa recebe feedback estruturado pós-contrato que alimenta reputação.
- Assembleia registrada gera ata imutável automática, reduzindo disputas.
- Síndico novato tem onboarding orientado (dados obrigatórios bloqueiam acesso até completar).

## Herança + fontes

- Legado vivo: `../vault/50-projects/master-sindico/00-product/vision.md` + `overview.md` (integrados aqui).
- Cliente original: `../vault/50-projects/master-sindico/client-vault/00 - Visão Geral/` (linguagem original do cliente).
- Destilado Fase 1 desta sessão (relatório "domínio/requisitos/client-vault" inline).

## Links

- [[personas]] — personas operantes
- [[portfolio-de-produtos]] — 7 sub-produtos
- [[glossary]] — ubiquitous language
- [[business-model]] — planos e quotas
- [[../01-domain/bounded-contexts]] — BCs que implementam a visão
- [[../STEERING]] — não-negociáveis derivados desta visão
- [[../ROADMAP]] — quando cada pilar entrega
