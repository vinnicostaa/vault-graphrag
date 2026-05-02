---
title: MOC — Client Canvas (diagramas originais do João)
type: moc
tags: [moc, master-sindico, client-vault, canvas, diagramas]
project: master-sindico
source: ~/Documentos/Obsidian Vault/Clients/MasterSindico (2026-03-10 a 2026-03-24)
created: 2026-04-24
---

# Client Canvas — diagramas originais do João

23 diagramas `.canvas` (formato nativo Obsidian) criados pelo cliente no vault `Clients/MasterSindico/` entre 2026-03-10 e 2026-03-24. Preservados aqui para **abertura nativa no Obsidian** — clique em cada link para abrir o diagrama interativo.

**Origem**: `~/Documentos/Obsidian Vault/Clients/MasterSindico/` — vault ativo do cliente. Os MDs dessa mesma pasta estão 100% idênticos ao que já foi ingerido em `vault/90-archive/from-master-sindico-v2-ingested/from-vault-50-projects-master-sindico/client-vault/` — **só os canvas + scripts JS são novos**.

> Conteúdo MD do client-vault **já está destilado** em `04-requirements/functional/*`, `01-domain/aggregates/`, `00-product/sub-produtos/`, `02-architecture/`. Estes canvas complementam com **visão visual** dos fluxos — valor pedagógico e de onboarding.

---

## Arquitetura de alto nível (4 canvas)

- [[Arquitetura de Domínios]] — **16 nodes**, 13 edges. Mapa HIGH-DOMAIN: Identidade 🔐 (IAM, Perfis), Institucional 🏛️ (Condomínio, Assembleia, Governança, Notificações), Comercial 💼 (Connect Me, Vizinhança), Conteúdo 📚 (Mídia, Academia LMS) + CROSS-DOMAIN ⚙️. **Começar por aqui** em onboarding de agente.
- [[Arquitetura Horizontal]] — **12 nodes**, 4 edges. Serviços cross-domain (Governança & Compliance, Trilha de Auditoria append-only, Indicators, Segurança) + integrações Adapter Layer (Mercado Pago, SES/SendGrid, Mux/Cloudflare, AWS S3, Firebase, ViaCEP). Atenção: **lista providers desatualizados** — stack canônica atualizada em [[../CLAUDE#4-stack-confirmada]].
- [[Stack Tecnologica]] — stack declarada (Go 1.22, Gin, Ent, NATS, OpenSearch, Zitadel, PostgreSQL, Redis) — **versão de março/2026, pré-reconciliação D-067**. Ent → sqlc; Casbin → ABAC próprio; OpenSearch → PG tsvector; AWS ECS → Railway. Ver [[../CLAUDE#4-stack-confirmada]] para canônico atual.
- [[System Architecture]] — mapa de sistema com nodes cross-BC.

## Fluxos de negócio (7 canvas)

- [[Fluxo Connect Me]] — **9 nodes**, 8 edges. Originador → cria formulário → Connect Me registra LGPD → Destinatário recebe notificação → aceita ("Tenho Interesse") ou recusa ("Não Atendo") → Connect Me revela dados cruzados ou notifica recusa → Originador recebe contato ou fim.
- [[Fluxo Ciclo Servico]] — **10 nodes**, 11 edges. Ciclo completo referenciando Reqs: Connect Me (Req 20) → Proposta (21) → Votação Fornecedor → Negócio Fechado (22) → Registro Execução (23) → Atividade Técnica (23.1) → Comunicado Pós-Deal (24) → Avaliação (26) → Adendo (25) → Timeline.
- [[Fluxo Contratacao]] — **9 nodes**, 8 edges. Jornada formal: Síndico cria Connect Me → Empresa tenho interesse → Negociação fora da plataforma → Validation Hub registra Negócio Fechado → Empresa envia Registro de Execução → Validation Hub valida → Timeline publica imutável → Empresa Comunicado Técnico → Validation Hub valida e publica.
- [[Fluxo Governanca]] — **8 nodes**, 6 edges. Síndico cria atividade → Timeline recebe → Plano Diretor status atualizado auto → Empresa envia registro execução → Síndico valida → Timeline publica imutável → Morador avalia gestão → Morador visualiza Timeline.
- [[Fluxo Assembleia Ao Vivo]] — **9 nodes**, 8 edges. Síndico configura → Administradora importa dados → Moradores ciência e confirmação → Moradores check-in ao vivo → Síndico inicia pauta/votação → Moradores levanta mão / vota → Síndico homologa/encerra → Timeline publica resultados → Sistema gera relatórios.
- [[Fluxo Vizinhanca]] — **7 nodes**, 7 edges. Parceiro cria promoção → Vizinhança define escopo (aberta vs exclusiva) → Morador Aberta vê no feed / Morador Exclusiva vê se do condomínio → Morador ativa promoção → Parceiro atualiza métricas / Síndico indica/cria benefício.
- [[Fluxo Registro]] — **6 nodes**, 5 edges. Signup: POST /v1/auth/register → User Registry valida/hash/cria → Email Service envia verificação → Usuário acessa link → User Registry ativa conta → Onboarding por persona.

## Domínios individuais (4 canvas)

- [[Dominio Identidade]] — **8 nodes**, 5 edges. Autenticação (JWT · OAuth) · Autorização (ABAC · Roles) · Registro (4 tipos de usuário) · Onboarding (4-7 telas por persona) · Billing (Trial · Base · Plus · Pro) · Feature Access (Matriz) · Perfis (Síndico · Empresa).
- [[Dominio Comercial]] — **9 nodes**, 6 edges. Registro Empresa (CNPJ · Perfil) · Connect Me (4 direções) · Propostas (Envio · Validação) · Negócio Fechado (Dupla confirmação) · Execução de Serviço (Registro · Validação) · Avaliação (5 critérios) · Vizinhança (Parceiros · Promoções) → Timeline.
- [[Dominio Conteudo]] — **6 nodes**, 5 edges. Vídeos Institucionais (Upload · Player) · Agência Marketing (Gestão delegada) · Busca & Discovery (Full-text) · Academia LMS (5 áreas) → Governança.
- [[Arquitetura Assembleia]] — **15 nodes**, 11 edges. Assembleia em 3 fases: ⏳ Pré (Config/Pauta, Edital PDF, Registro de Ciência, Procurações & Simulador Quórum) · 🔴 Ao Vivo (Check-in App/Terminal/QR, Painel do Presidente, Telão, Fila de Fala timer 2/3 min, Votação Tempo Real com Peso Fracionado) · 📜 Pós (Homologação Lock Definitivo, Consolidação dados da Ata, Trilha de Auditoria).

## Integrações e modelos específicos (6 canvas)

- [[Adapter Layer]] — **23 nodes**, 14 edges. Provider interfaces e implementações: `IVideoProvider` (VideoAdapter), `IPaymentGateway`, `IEmailProvider`, `ISMSProvider`, `IStorageProvider`, `ICEPLookup` — mapeando cada domínio interno para o adapter e serviço externo correspondente.
- [[Modelo ABAC]] — estrutura da matriz ABAC (role × plan_tier × action).
- [[Modelo Planos]] — hierarquia de planos (trial, base, plus, pro) + features por tier.
- [[Modelo de Tenants]] — multi-tenancy model.
- [[Visibilidade Videos]] — regras de quem vê quais vídeos (lock 90d, paywall, privacidade de métricas).
- [[Mapa Academia]] — áreas da Academia LMS.
- [[Jornadas Vizinhanca]] — jornadas VS (síndico) + VM (morador) + VE (empresa) + VZ (parceiro).

## Super-mapa

- [[Master Canvas]] — canvas-mãe que referencia os demais.

---

## Observação crítica — **canvas podem estar desatualizados**

Diagramas datados de 2026-03-10 a 2026-03-24 refletem o **entendimento do cliente antes da reconciliação de stack (D-067, 2026-04-23)**. Elementos que podem divergir da canônica atual:

- **Mercado Pago** → Stripe (D-004)
- **Ent ORM** → pgx + sqlc (ADR-0001)
- **Casbin** → ABAC próprio em middleware (ADR-0021)
- **OpenSearch** → PostgreSQL tsvector (D-067)
- **AWS SES/ECS** → Resend + Railway (D-067)

Os **fluxos de negócio** (Connect Me, Contratação, Governança, Vizinhança) são canônicos — refletem requisitos que foram absorvidos em `04-requirements/functional/*` e que continuam vigentes.

---

## Scripts do vault do cliente

Os scripts `generate.js` (22kb) e `generate_seq.js` (8kb) que o cliente usa para **gerar canvas automaticamente** (a partir de spec JS com nodes + edges) estão arquivados em `[[../../../90-archive/from-client-vault-scripts/]]`. São ferramentas operacionais do cliente, não conteúdo de produto.

---

## Links

- [[../CLAUDE]] — contrato do projeto
- [[../_moc]] — MOC raiz
- [[../02-architecture/_moc]] — arquitetura destilada
- [[../04-requirements/_moc]] — requisitos destilados
- [[../01-domain/_moc]] — domínio + ubiquitous language
- `~/Documentos/Obsidian Vault/Clients/MasterSindico/` — origem (mantido pelo cliente)
