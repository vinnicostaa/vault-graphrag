---
title: "Quebras de Regra Detectadas — Cross-Doc Audit"
updated: 2026-04-22
maintainer: agent-3 (analista)
type: audit
status: open
---

# Quebras de Regra Detectadas — Cross-Doc Audit (2026-04-22)

Auditoria cruzada entre `Content/`, `Content/contents/`, `backend/.kiro/specs/master-sindico/requirements/`, `backend/.kiro/STATE.md`, `backend/.kiro/AUDIT.md`, `backend/CLAUDE.md`, `web/CLAUDE.md`, `app/CLAUDE.md` e memórias persistentes.

**Severidade**: 🔴 critical (regra de negócio rompida ou ambiguidade pode gerar bug em prod) · 🟡 medium (drift documental sem impacto imediato em código) · 🟢 low (cosmético / desatualização sem risco).

**Como ler**: cada item lista *fontes em conflito*, *raiz*, *fix proposto*, *quem deve resolver*. Status `🔴 Aberto` permanece até alguém aplicar o fix.

> **Regra de prioridade**: quando spec viva (`backend/.kiro/specs/master-sindico/`) divergir de qualquer outra fonte, a spec viva vence — atualizar o conflitante. Quando o **código** divergir da spec viva, ler o código e atualizar a spec.

---

## 🔴 CRITICAL

### Q1 — Stack desatualizada em `product-overview.md` (3 menções stale)

- **Status**: 🔴 Aberto
- **Fontes em conflito**:
  - `backend/.kiro/specs/master-sindico/requirements/product-overview.md:36-43` — menciona "AWS SES (`IEmailProvider`)", "OpenSearch (Sprint 5)", "Infra: AWS (ECS Fargate, RDS, ElastiCache, S3, SES, CloudFront, Route53, CloudWatch) + Railway (alternativa/complementar)"
  - `web/CLAUDE.md`, `backend/CLAUDE.md`, `backend/.kiro/SESSION_CHARTER.md` — confirmam **Resend** (não SES), **PostgreSQL tsvector** (não OpenSearch), **Railway primário** (não AWS ECS)
  - `backend/.kiro/AUDIT.md` F30 — entregou 7 templates pt-BR via Resend
  - Memória `project_railway_url.md` — Railway é stack canônica (project ID efe664b7-…)
- **Raiz**: `product-overview.md` foi escrito antes da decisão de migração de cloud (anterior a 2026-04-21). Sprints 0-9 corrigiram `web/AGENTS.md`, `web/CLAUDE.md`, `backend/CLAUDE.md`, mas o `product-overview.md` recortado não foi atualizado.
- **Impacto**: agentes futuros que lerem **só** o product-overview vão considerar provedores errados. Pode gerar PRs com adaptador de SES ou queries de OpenSearch.
- **Fix proposto**: substituir as 3 menções:
  - Linha 36 → "Email: **Resend** (`IEmailProvider`)"
  - Linha 38 → "Busca: **PostgreSQL `tsvector`** (Sprint 5 — descartado OpenSearch por simplicidade operacional)"
  - Linha 43 → "Infra: **Railway** (primário) — plano AWS futuro registrado em `design/deploy-config.md`"
- **Quem resolve**: agente 1 (executor) ou principal — mudança documental, sem código. Bumpar `version: 1.1`.

---

### Q2 — Connect Me Morador→Síndico: quota canônica vs functional-matrix divergem para Base

- **Status**: 🔴 Aberto
- **Fontes em conflito**:
  - `requirements/personas-and-plans.md:80` → "Morador Base | 2/ano"
  - `requirements/commercial.md` Req 19.1 → "Quotas: Base 2/ano, Pagante 4/ano"
  - `requirements/cross-domain.md` Decision 10 → "N1: 2/ano · N2: 4/ano · N3: ilimitado · Base: 2/ano · Pagante: 4/ano"
  - `requirements/functional-matrix.md:43` → "**Connect Me Enviar síndico** | Base ❌ | Pagante 2/ano | N1 ❌ | N2 2/ano | N3 4/ano | Plus ❌ | Pro ✅"
  - `requirements/functional-matrix.md:152` (Quotas Connect Me Anual) → "Morador Base | 0 | Morador Pagante | 4/ano | Sind. N1 | 2/ano | N2 | 4/ano | N3 | ∞"
- **Raiz**: dois documentos canônicos discordam sobre quota do Morador Base. `personas-and-plans` + `commercial` + Decision 10 dizem **2/ano**; `functional-matrix` diz **0** (Base sem Connect Me) e **2/ano para Pagante** (que conflita com Pagante = 4/ano nas outras três fontes).
- **Impacto**: quando ABAC for codificar a quota, vai escolher uma fonte e furar outra. Síntoma em prod: morador Base recebe 403 "QUOTA_EXCEEDED" no 1º envio (se ABAC seguir matrix), ou ultrapassa o cap pretendido (se seguir commercial/personas).
- **Fix proposto**: tomar Decision 10 como autoridade (alinha com `personas-and-plans` e `commercial`):
  - **Morador Base = 2/ano** (corrigir matrix linha 43 + linha 152)
  - **Morador Pagante = 4/ano** (corrigir matrix linha 43)
  - **Sind. N2 = 4/ano** (corrigir matrix linha 43 — está como 2/ano)
  - Confirmar com João se prefere mudar a Decision para "Base = 0, Pagante = 4" (mais restritivo) — abrir D-031 em STATE.md.
- **Quem resolve**: agente principal alinha com João (decisão de produto) ou aplica Decision 10. Atualizar `functional-matrix.md` + `version`.

---

### Q3 — Empresa "Base" mencionada onde só existem `enterprise_plus` e `enterprise_pro`

- **Status**: 🔴 Aberto (semântico)
- **Fontes em conflito**:
  - `requirements/commercial.md` Req 18 → "Visibilidade por plano: **Base** visível apenas para Síndico Gestor/Plus/Pro; Plus/Pro visível para todos"
  - `requirements/personas-and-plans.md` (Empresa) → só lista `enterprise_plus` e `enterprise_pro`. Não há plano "Base" para Empresa.
- **Raiz**: texto herdado do legacy (Drizzle/TS) onde existia "Empresa Base". Stack atual decidiu apenas Plus/Pro.
- **Impacto**: leitor não sabe se "Base" se refere a Empresa em trial, em paywall expirado, ou a uma persona descartada.
- **Fix proposto**: trocar Req 18 para algo como _"Visibilidade por plano: **trial / soft-block** visível apenas para Síndico Gestor / Plus / Pro; Plus/Pro visível para todos."_
- **Quem resolve**: agente principal ou executor — mudança de redação.

---

### Q4 — Agência de Marketing: "não tem login" vs role `marketing` no Zitadel

- **Status**: 🔴 Aberto (conflito real entre dois requisitos canônicos)
- **Fontes em conflito**:
  - `requirements/content.md` Req 30 → "Agência de Marketing — **não é tenant** — **não tem login próprio, não é persona com role**"
  - `requirements/identity.md` Req 2 + decisão T7 → roles primárias incluem `marketing`
  - `requirements/personas-and-plans.md` linha 21 → "Agência de Marketing | Actor delegado (não é tenant) | Sem perfil próprio, vinculado a Empresa"
  - `requirements/personas-and-plans.md` linha 50-53 → "Agência / Marketing — Plano: `marketing_standard`"
- **Raiz**: o Req 30 é absoluto ("não tem login"), mas T7 e o plano `marketing_standard` implicam que existe sim um usuário com role `marketing` que faz login para operar (em nome de empresas que a contrataram).
- **Impacto**: confusão arquitetural — implementador não sabe se cria login + role `marketing` (e usa token só para escopo de empresa) ou se elimina role e usa só token-only delegation.
- **Fix proposto**: clarificar — **agência tem login com role `marketing`**, mas opera **sempre delegada a uma empresa via token**. Trocar Req 30 para: _"Agência de Marketing — **não é tenant**; tem usuário com role `marketing` no Zitadel mas opera apenas delegada a empresas via token de escopo restrito (`videos:upload`, `videos:edit`, `analytics:read`)."_
- **Quem resolve**: agente principal alinha com João, registra D-031/D-032 em STATE.md.

---

### Q5 — Síndico N1 e Academia LMS: `content.md` e `functional-matrix` divergem sobre certificado

- **Status**: 🔴 Aberto
- **Fontes em conflito**:
  - `requirements/content.md` Req 98.1 (Acesso por persona) → "Síndico N1+ | Cursos, Treinamentos, Lives, Fórum, Biblioteca | Fazer | **Certificado**"
  - `requirements/functional-matrix.md` Síndico N1 → "Academia LMS | Certificado | ❌ (N1) | ✅ (N2, N3)"
- **Raiz**: `content.md` lista "N1+" como tendo certificado; `functional-matrix` exclui N1.
- **Impacto**: implementador pode habilitar geração de certificado para N1 (segundo content.md) e gerar surpresa para a área comercial que vendeu N1 sem certificado (segundo matrix).
- **Fix proposto**: alinhar com matrix (mais restrito = consistente com N1 = "Aprender", tier de descoberta sem entrega de credencial). Atualizar `content.md` Req 98.1 para "Síndico **N2+**".
- **Quem resolve**: agente principal alinha com João.

---

## 🟡 MEDIUM

### Q6 — `Content/contents/master-sindico-arquitetura-solucoes.md`: assembly marcada como CONFLITO PENDENTE quando já há Decision 1

- **Status**: 🟡 Aberto (drift documental)
- **Fontes em conflito**:
  - `Content/contents/master-sindico-arquitetura-solucoes.md` (Mar/2026) §3.2.4 → "🔴 PENDENTE: Definir se assembleia ao vivo entra no escopo contratual atual"
  - `requirements/product-overview.md` Decision 1 (estável) → "Live Assembly com WebSocket real-time; modalidade online/híbrida inativa no MVP"
- **Raiz**: documento de Content é mais antigo que a Decision 1. Nunca foi marcado como stale.
- **Fix proposto**: adicionar nota no topo do arquivo: _"⚠️ Documento histórico — Decisões de escopo de Assembly resolvidas em product-overview.md Decision 1 (Mar/2026)."_ E linkar para spec viva.
- **Quem resolve**: este analista (agente 3) — já encaminhado neste vault via [[_MOC]] e este documento.

---

### Q7 — `Content/contents/master-sindico-domain-mapping.md`: usa pt-BR roles, TypeScript, Mercado Pago, Awilix DI

- **Status**: 🟡 Aberto
- **Fontes em conflito**:
  - `Content/contents/master-sindico-domain-mapping.md:150` → roles em pt-BR: `"sindico" | "morador_titular" | "morador_dependente" | "empresa_sindica"`
  - `Content/contents/master-sindico-domain-mapping.md:313` → "Estrutura de Pastas (Backend)" com path `backend/src/` e exemplos TypeScript
  - `Content/contents/master-sindico-domain-mapping.md:503` → "Payment Gateway | Mercado Pago / Stripe — recomendação Mercado Pago (Brasil)"
  - Realidade atual (`requirements/identity.md` T7, `requirements/billing.md`, `STATE.md`) → roles em **inglês**, **Go**, **Stripe**.
- **Raiz**: documento herdado da fase TypeScript/legacy.
- **Fix proposto**: marcar arquivo inteiro como `status: deprecated` na frontmatter; nota no topo apontando para spec viva.
- **Quem resolve**: agente 3 (já fará no commit deste vault).

---

### Q8 — Connect Me E→E: 3 redações diferentes para a mesma quota

- **Status**: 🟡 Aberto
- **Fontes em conflito**:
  - `requirements/personas-and-plans.md:87` → "Empresa Plus | (não aplicável — E→E só Plus/Pro)"
  - `requirements/functional-matrix.md:156` → "Empresa Plus | (E→E apenas)"
  - `requirements/commercial.md` Req 19.2 → "Quota: conforme contrato (ilimitado para Pro, configurável para Plus)"
- **Raiz**: cada arquivo descreve a mesma regra com palavras diferentes; o `personas-and-plans` chega a sugerir "não aplicável" o que pode confundir.
- **Fix proposto**: padronizar em **"Empresa Plus: E→E ilimitado dentro do Connect Me; Empresa Pro: ilimitado e disponibilidade ampliada (D2D, parceria técnica)."** Replicar em todas as três fontes. Eliminar "não aplicável" de Plus.
- **Quem resolve**: agente principal alinha com João sobre se Plus tem cap ou não, depois replica.

---

### Q9 — Vídeos institucionais: matriz mensal vs trava trimestral coexistem mas referências não são cruzadas

- **Status**: 🟡 Aberto (cosmético, mas confunde)
- **Fontes**:
  - `requirements/content.md` Req 29 → quotas mensais por plano + "Trava trimestral: 90 dias entre atualizações do mesmo vídeo"
  - `requirements/billing.md` Req 6 (Quotas — Vídeos Institucionais) → mesmas quotas mensais, mas não menciona explicitamente que a trava trimestral é **por vídeo**, não global
- **Raiz**: a regra é "quota mensal de upload" (cap de N novos vídeos/mês) **separada** de "trava trimestral por vídeo" (substituição do mesmo arquivo). Lendo só billing dá pra interpretar como cap global.
- **Fix proposto**: adicionar nota explícita em `billing.md` Req 6 logo após a tabela de vídeos: _"A quota mensal cobre **novos uploads**. Substituir um vídeo já existente respeita a trava trimestral (Rule 7) — 90 dias mínimo entre atualizações do mesmo `video_id`."_
- **Quem resolve**: agente principal ou executor.

---

### Q10 — Visibilidade de empresa "Base" (Q3) também aparece em `personas-and-plans` matriz

- **Status**: 🟡 Aberto (decorre de Q3)
- **Fonte**: `requirements/personas-and-plans.md:117` → tabela "Vídeos empresa (preview 25%)" inclui colunas Base/Pagante para Empresa, mas Empresa não tem Base.
- **Fix proposto**: junto com Q3 — remover "Base" das colunas da Empresa nas matrizes.

---

### Q11 — `deploy-config.md` declara Railway primário, mas `product-overview.md` ainda lista AWS

- **Status**: 🟡 Aberto (subconjunto de Q1)
- **Fontes**:
  - `requirements/design/deploy-config.md` (criado em F19) → Railway primário (URL conhecida na memória `project_railway_url.md`)
  - `requirements/product-overview.md:43` → AWS ECS Fargate como infra
- **Fix**: incluir junto com Q1.

---

### Q12 — `obsidian-mapping.md` lista vault externo do João como autoridade — mas Obsidian externo está stale

- **Status**: 🟡 Documentado (não é defeito; é nota)
- **Fonte**: `backend/.kiro/specs/master-sindico/reference/obsidian-mapping.md` declara que o vault externo cita Ent ORM, Casbin, Uber-FX (descartados); MS atual é fonte de verdade.
- **Ação**: este vault `Content/` agora consolida tudo. Recomendação: deprecar parcialmente o vault externo (notas pessoais do João continuam, mas notas técnicas devem migrar/linkar para `backend/.kiro/`).
- **Quem resolve**: João (vault é dele). Este analista apenas documenta.

---

### Q13 — `excopo-tecnico-antigo.txt` e `main.go.md` em `Content/` raiz — sem classificação

- **Status**: 🟡 Aberto (organizacional)
- **Fonte**: `Content/excopo-tecnico-antigo.txt`, `Content/main.go.md`
- **Raiz**: arquivos legados sem indicação de status.
- **Fix proposto**: mover para `Content/_inbox/` com nota "histórico — não modificar".
- **Quem resolve**: agente 3 (este). Vou aplicar abaixo.

---

## 🟢 LOW (cosmético / drift sem risco)

### Q14 — `requirements/content.md` Req 29 — 12 tipos de vídeo difere de `Content/contents` (Mar/2026)

- **Detalhe**: `content.md` lista "Apresentação da empresa, Portfólio de serviços, Depoimento de cliente, Tutorial..." (12 itens). `Content/contents/master-sindico-arquitetura-solucoes.md` lista 12 ligeiramente diferentes ("Apresentação institucional, Apresentação da equipe técnica..."). Mesmo cardinal, rótulos divergentes.
- **Fix**: adotar a lista de `content.md` como canônica. Sem ação imediata.

### Q15 — Recortes (`requirements/*.md`) não trazem "Bronze → Diamante" do tier de reputação mencionado em `product-overview.md` "Visão"

- **Detalhe**: `product-overview.md:21` cita "Reputação → sistema de status baseado em evidência (Bronze → Diamante)" mas nenhum recorte detalha critérios, fórmulas, transições.
- **Fix**: criar `requirements/reputation.md` quando o tema entrar em sprint (post-launch — Sprint 10+).

### Q16 — Numeração de migrations: limite por módulo é 100 (identity 001-099, billing 100-199 etc) — risco de overflow se módulo crescer

- **Detalhe**: identity já em ~14 migrations (não preocupante); billing 100-199 dá folga. Sem ação até crescer.

### Q17 — `requirements/cross-domain.md` Req 65 ainda menciona "MinIO/Cloudflare R2" como Storage — D-029 (decisão entre R2 vs S3) está aberta

- **Detalhe**: documento lista as opções mas não fixa. Aceitável porque D-029 está aberta.
- **Fix**: quando D-029 fechar, atualizar Req 65.

### Q18 — Confusão terminológica: "GSD" do projeto = "Get Shit Done" (gsd-build/get-shit-done) ≠ "GSD" da indústria = "GitHub Spec-Driven" (Spec Kit)

- **Detalhe**: `backend/.kiro/steering/sdd-workflow.md:7` cita "Inspirado em parte pelo gsd-build/get-shit-done — adotamos em 3 conceitos específicos". Em 2026, indústria usa "GSD" mais comumente para GitHub Spec-Driven (do GitHub Spec Kit).
- **Impacto**: em conversas externas e leitura de literatura SDD, leitor pode confundir.
- **Fix**: ver [[../sdd-gsd-aplicado]] — recomendação de adicionar nota explicativa em `sdd-workflow.md` (1 linha) e mencionar Spec Kit como referência cognata.
- **Quem resolve**: principal (mudança de doc steering).

---

---

## 🔴 CRITICAL — adicionados na 2ª rodada (leitura de PDFs + monolíticos completos)

### Q19 — Roles em pt-BR (requirements.md original) vs inglês (T7 atual)

- **Status**: 🔴 Aberto (drift profundo, mas confinado a docs históricos)
- **Fontes em conflito**:
  - `Content/requirements.md:72-74` (Glossary) → `sindico, morador_titular, morador_dependente, empresa_administrador, empresa_operador, agencia` — 6 roles em pt-BR
  - `Content/contents/master-sindico-domain-mapping.md:150` → `"sindico" | "morador_titular" | "morador_dependente" | "empresa_sindica"` (pt-BR + tem "empresa_sindica" que nem existe em outras fontes)
  - `requirements/identity.md` Req 2 + T7 → roles **inglês**: `syndic, resident, enterprise, marketing, local_company, admin`
- **Raiz**: documento original (Mar/2026) propôs taxonomia em pt-BR; T7 corrigiu para inglês (compatibilidade Zitadel + i18n).
- **Impacto**: agentes futuros que lerem `Content/requirements.md` ou `domain-mapping.md` podem implementar enum em pt-BR — código vai falhar quando bater no Zitadel.
- **Fix proposto**: adicionar nota no topo de `Content/requirements.md` e `domain-mapping.md` apontando para `requirements/identity.md` como canônico. Não modificar os arquivos originais (são do João).
- **Quem resolve**: agente 3 (este — já refletido em [[regras-canonicas]] §4 e [[../04-Listas-Mestre/Enums-Consolidados]] §1).

### Q20 — Roles "agencia" vs "marketing" / sem "local_company"

- **Status**: 🔴 Aberto (subconjunto de Q19)
- **Fontes**:
  - `Content/requirements.md` lista 6 roles incluindo `agencia` mas SEM `local_company` ou `admin`
  - T7 atual: `syndic, resident, enterprise, **marketing**, **local_company**, **admin**`
- **Raiz**: original tinha "agencia" como rótulo + faltava persona "local_company" (Parceiro Vizinhança) — Decision 12 adicionou Parceiro como persona em Mar/2026
- **Fix**: Decision 12 + T7 vencem. Atualizar referências em arqueologia.

### Q21 — Mercado Pago (requirements.md original) vs Stripe (atual)

- **Status**: 🔴 Documentado (não bloqueante porque billing já implementado com Stripe)
- **Fontes em conflito**:
  - `Content/requirements.md` Req 4.6 → "integrate with **Mercado Pago** via Adapter_Layer"
  - `Content/requirements.md` Req 61 → "Payment Integration (**Mercado Pago**)"
  - `Content/design.md` linha equivalente → "Payment: Mercado Pago"
  - `Content/contents/master-sindico-arquitetura-solucoes.md:533` → "Payment Gateway | Mercado Pago"
  - **Atual**: `requirements/billing.md` Req 4 → Stripe (`IPaymentGateway`); webhook `POST /webhooks/stripe`; SDK `stripe/stripe-go`
- **Raiz**: cliente preferiu Stripe (suporte BR completo + Customer Portal). Mercado Pago descartado em fase de discovery.
- **Fix**: como Q1, é mudança documental sem urgência de código. **Notar com bandeira "stack stale"** no topo dos `Content/*.md` antigos.

### Q22 — Hash de senha bcrypt/Argon2 (original) vs Zitadel-only (atual)

- **Status**: 🔴 Documentado
- **Fontes**:
  - `Content/requirements.md` Req 3.5 → "hash passwords using **bcrypt or argon2**"
  - **Atual**: identity.md Req 3 → "Hash de senha: gerenciado **exclusivamente** pelo Zitadel (Argon2 internamente)"
- **Raiz**: original presumia auth próprio com hash interno; D-XXX mudou para Zitadel cloud (não reinventar auth — antipattern A11 do legacy).
- **Fix**: como Q21, registrar como arqueologia. Código já correto (sem bcrypt).

### Q23 — Sistema de Cupons / "Promoção do Dia" — DESAPARECEU

- **Status**: 🔴 ALERTA CRÍTICO (falta de spec ou removido?)
- **Fonte**:
  - `Content/excopo-tecnico-antigo.txt` Sprint 3 (v3.0 Janeiro 2026) detalha **completo**:
    - "Engine de Cupons (Promoção do Dia)"
    - Formato `ID_LOJA(5) + SEQUENCIAL(3) + PALAVRA(5)` ex: `00123055PAO`
    - Trava de tempo: 1 cupom a cada 4 horas para o mesmo estabelecimento
    - "NÃO É SISTEMA DE HELPDESK/CHAMADO DE SUPORTE" — termo "Ticket" no código refere a Cupom de Desconto
    - Promoções exclusivas de condomínio via número de inscrição
  - **Atual**: `requirements/commercial.md` Req 27.3 (Vizinhança Parceiro) menciona 7 tipos de benefício (`desconto_percentual, desconto_fixo, etc`) MAS **NÃO menciona o formato ID_LOJA+SEQ+PALAVRA, nem trava de 4 horas, nem código numérico gerado**.
- **Impacto**: feature contratual (escopo Janeiro 2026, v3.0) **possivelmente não migrada**. Se for para implementar, é trabalho adicional. Se for para descartar, contratual issue (era obrigação contratada).
- **Fix proposto**: **CONFIRMAR COM JOÃO** se o Sistema de Cupons foi:
  - (a) descartado — abrir D-XXX em STATE.md justificando
  - (b) substituído pelo "Ativar Promoção" da Vizinhança (VS5/VE5/VM4) — gravar mapping
  - (c) ainda pendente de implementação — adicionar a Sprint 10+
- **Severidade**: 🔴 critical (contractual gap potencial).

---

## 🟡 MEDIUM — adicionados na 2ª rodada

### Q24 — Cardinalidade de Tipos de Comunicado divergente

- **Status**: 🟡 Aberto
- **Fontes**:
  - `Content/requirements.md` Req 14 → 8 tipos: `aviso_operacional, interrupcao_programada, orientacao_moradores, aviso_seguranca, comunicado_institucional, conclusao_servico, recomendacao_manutencao, alerta_preventivo`
  - `Content/contents/####################TELA DO MORADOR#####(1).ini` (Bloco S23-S25 + Bloco 12) lista TIPOS ADICIONAIS: `Aviso de início de serviço, Comunicado de intervenção/obra, Circular informativa, Orientação ao morador, Comunicado de segurança, Comunicado de interdição temporária, Comunicado de conclusão`
- **Raiz**: ####TELA####.ini é mais granular (tipos específicos pós-deal); requirements.md tem categorias generalistas.
- **Fix proposto**: adotar 8 tipos do requirements.md como canônico. Tipos do ####TELA####.ini se encaixam em categorias mais granulares dentro de "comunicado_institucional" via subcategoria — adicionar coluna `subtipo: text NULL` na tabela `communications`.
- **Quem resolve**: agente principal define se vale criar subtipos.

### Q25 — Score Governança 0-100 (1) vs Score Governança 1-10 + Score Compliance 0-100 (2)

- **Status**: 🟡 Aberto
- **Fontes**:
  - `Content/####################TELA####(1).ini` Bloco 3 (Central de Governança) → "Score de Governança (1-10) privado" + "Score de Compliance (0-100, privado)" — **DOIS SCORES DIFERENTES**
  - `Content/requirements.md` Req 41 + `requirements/cross-domain.md` Req 33 → **um único** "Governance Score 0-100" (público)
- **Raiz**: ####TELA####.ini propõe 2 scores; requirements consolida em 1.
- **Fix proposto**: confirmar com João. **Hipótese A**: 2 scores diferentes têm sentido (governance = qualidade de gestão visível ao mercado; compliance = saúde documental interna). **Hipótese B**: redundante, manter 1.
- **Quem resolve**: agente principal alinha com João.

### Q26 — 4 Sprints contratuais (Janeiro 2026 v3.0) vs 9 Sprints atuais

- **Status**: 🟡 Aberto (informacional, mas contratual)
- **Fonte**:
  - `Content/excopo-tecnico-antigo.txt` v3.0 Janeiro 2026 § 7 — **4 sprints de 3 semanas cada** (Sprint 1-4) = 12 semanas. Modelo de pagamento: **20-20-20-40**.
  - **Atual**: `tasks/README.md` define 9 sprints (Sprint 0 + 1-8 + post-launch) cobrindo 23/03/2026 → 08/08/2026. Marcos 08/05, 20/06, 20/07.
- **Raiz**: escopo do contrato original era menor (provavelmente sem Live Assembly, sem LMS completo, sem Banco de Talentos, sem Compliance C1-C11). Conforme escopo cresceu (Decisions 1, 6, 7, 8 + 12), sprints foram repaginadas.
- **Fix**: **garantir que escopo expandido foi formalizado contratualmente** (aditivos). Se não, há risco de o cliente alegar entrega excessiva fora de escopo.
- **Quem resolve**: agente principal alinha com João (questão contratual, não técnica).

### Q27 — Assembleia ASSÍNCRONA (contrato original) vs LIVE com WebSocket (atual)

- **Status**: 🟡 Aberto (já mapeado, mas vale formalizar)
- **Fontes**:
  - `Content/excopo-tecnico-antigo.txt` Sprint 2.1 (v3.0 Janeiro 2026) → "É **assíncrono**. Não é votação em blockchain. Não é assembleia virtual ao vivo (Zoom/Meet)." Modelo: edital + pautas com vídeos do síndico + janela de votação.
  - `Content/contents/master-sindico-arquitetura-solucoes.md:295` → "🔴 PENDENTE: Definir se assembleia ao vivo entra no escopo" (Mar/2026)
  - **Decision 1 atual** (resolvida) → "Live Assembly com WebSocket real-time; Telão 2 áreas; fila de fala cronometrada. Modalidade online/híbrida inativa no MVP (só presencial)."
- **Raiz**: escopo cresceu MUITO. Original era assíncrono (simples — vídeos + janela de votação); atual é Live (complexo — WebSocket + telão + fila de fala + procurações com fração ideal + voto presencial assistido + modo contingência + score de transparência).
- **Impacto**: Q26 + Q27 juntos sugerem que houve aditivo contratual (ou está pendente). Confirmar.
- **Fix**: contratual + documental. Atualizar `excopo-tecnico-antigo.txt` com nota "deprecated v3.0 → escopo expandido em vN".

### Q28 — Lives RESTRITAS ao Admin MS (contrato) vs Lives no LMS (atual)

- **Status**: 🟡 Aberto
- **Fontes**:
  - `Content/excopo-tecnico-antigo.txt` Sprint 4.4 → "**Apenas o Administrador MasterSíndico** pode iniciar transmissões. Síndicos e Empresas **NÃO** fazem live."
  - `Content/contents/MÓDULO ACADEMIA MASTER SÍNDICO.ini` LMS8-LMS9 + `Content/contents/ARQUITETURA DO MÓDULO LMS.ini` → Lives são uma das 5 áreas do LMS, com agenda de eventos, participação ao vivo, chat, perguntas. **Não restringe quem cria.**
  - `Content/requirements.md` Req 93 (Lives genérico) + Req 98.1 (Academia LMS) → Lives são feature do LMS, mas Req 98.1 não restringe explicitamente quem cria.
- **Raiz**: contrato original RESTRINGIA lives ao admin MS; specs atuais não reforçam essa regra.
- **Fix proposto**: adicionar regra explícita em `requirements/content.md` Req 98.1 — "Lives na Academia LMS: criação restrita a Admin Master Síndico (Decision do contrato original)" OU confirmar com João se essa regra mudou.

### Q29 — Limite condomínios: 12 (PDF Síndico Mar/2026) vs 15 (Decision 11)

- **Status**: 🟡 Aberto (já corrigido em spec, mas PDF stale)
- **Fontes**:
  - PDF "JORNADA DO SÍNDICO completa.pdf" Regra 5 → "Síndico pode consultar até **12 condomínios**"
  - **Decision 11** (atual) → "**15** condomínios por síndico"
  - `requirements/personas-and-plans.md` linha 105 → "**15 condomínios** por conta de síndico"
- **Raiz**: PDF é Mar/2026, Decision é mais recente.
- **Fix**: PDF é histórico do João, não modificar. Documentação interna OK (15).

### Q30 — Comunicados pós-deal: 5 tipos (Req 24) vs 7+ tipos (####TELA####.ini Bloco 12)

- **Status**: 🟡 Aberto (relacionado a Q24)
- **Fontes**:
  - `requirements/commercial.md` Req 24 → 5 tipos: `execucao_iniciada, execucao_pausada, execucao_retomada, execucao_concluida, atraso_informado`
  - `Content/requirements.md` Req 24 (linhas 711) lista 5 tipos: `orientacao_tecnica, recomendacao_manutencao, alerta_preventivo, conclusao_servico, atualizacao_garantia`
  - `Content/####################TELA####(1).ini` Bloco 12 lista 8: `Aviso de início de serviço, Comunicado de intervenção/obra, Circular informativa, Orientação ao morador, Comunicado de segurança, Comunicado de interdição temporária, Comunicado de conclusão, Outro`
- **Raiz**: 3 listas diferentes! Cada documento tem nomes próprios.
- **Fix proposto**: padronizar para 8 tipos (combinação dos 3) via PRD update. Adicionar `tipo` enum + `subtipo` text NULL na tabela.

---

## 🟢 LOW — adicionados na 2ª rodada (cosméticos/confirmações)

### Q31 — Avaliação bimestral (PDF Morador M12) vs "every 2 months" (Req 15) ✅ CONFIRMADO

- **Fontes coincidem**: PDF M12 "obrigatória bimestralmente" / Req 15 "every 2 months". ✅ Sem ação.

### Q32 — Onboarding 4-6-7-4 telas ✅ CONFIRMADO

- PDF "ONBOARDING DOS PERFIS.pdf" lista exatamente 4 (Morador), 6 (Síndico), 7 (Empresa), 4 (Parceiro). Bate com Decision 3. ✅ Sem ação.

### Q33 — Tempo de fala 2 ou 3 minutos + prorrogação 2 min ✅ CONFIRMADO

- PDF Assembleia Funcional 4.1 + Req 60.5: ambos confirmam. ✅

### Q34 — 5 tipos de item da pauta (informativo, simples, fração ideal, escolha fornecedor, eleição) ✅ CONFIRMADO

- PDF 4.3 + Req 49: ambos confirmam. ✅

### Q35 — Inadimplência cutoff 1h antes da 1ª convocação ✅ CONFIRMADO

- PDF 4.15 + Req 48: ambos confirmam. ✅

### Q36 — Procuração validade 24h pós-encerramento ✅ CONFIRMADO

- PDF 4.13 + Req 52: ambos confirmam. ✅

### Q37 — Auto-close Connect Me em 48h ✅ CONFIRMADO

- ####TELA####.ini Bloco 15 (TELA 41) + Req 19.11: ambos confirmam. ✅

### Q38 — Trava trimestral 90 dias para vídeo institucional E vídeo-currículo ✅ CONFIRMADO

- excopo Regras 2.4 + 2.6 + Req 29 (rule 7) + Decision 6 (90s, 90 dias): todos confirmam. ✅

### Q39 — Banco de Talentos: 3 vs 5 vínculos profissionais

- **Status**: 🟢 Decision pendente (já registrada em requirements.md Decision 6)
- **Fontes**:
  - PDF "ESTRUTURA DE CADASTRO DE CURRÍCULO" TELA 08 → "Últimos **5** vínculos"
  - `Content/requirements.md` Decision 6 alternativa → "last **3** positions"
- **Raiz**: PDF mais recente (Mar/2026) tem 5; documento mais antigo tinha 3.
- **Fix**: adotar 5 (PDF). Confirmar com João via D-XXX.

### Q40 — Eleição na Assembleia gera "avaliação/percepção da gestão" escondida

- **Status**: 🟢 Aberto (regra interessante não migrada)
- **Fonte**: PDF Assembleia 4.4 → "**Eleição** Gera: avaliação/percepção da gestão, sem caráter jurídico (não será exibido aos moradores - hating imediato assim que é cadastrado o item assembleia - a avaliação deverá aparecer após o registro de ciência do morador)"
- **Atual**: `requirements/assembly.md` Req 49 lista `eleicao` como item type, mas **não menciona** essa regra de avaliação escondida.
- **Fix proposto**: adicionar regra em Req 49 ou criar Req 49.1 — "Quando item da pauta é `eleicao`, sistema gera enquete de avaliação/percepção da gestão automaticamente, escondida até morador registrar ciência da pauta."
- **Quem resolve**: agente principal valida com João + atualiza Req.

---

## Resumo executivo (atualizado)

| Severidade | 1ª rodada | 2ª rodada | Total |
|---|---|---|---|
| 🔴 Critical | 5 (Q1-Q5) | 5 (Q19-Q23) | **10** |
| 🟡 Medium | 8 (Q6-Q13) | 7 (Q24-Q30) | **15** |
| 🟢 Low | 5 (Q14-Q18) | 10 (Q31-Q40) | **15** |
| **Total** | **18** | **22** | **40** |

### Prioridade Marco 1 (08/05)
**Resolver**: Q1, Q2, Q3 — afetam código MVP. **Confirmar com João**: Q4, Q5, Q23 (Cupons), Q26 (escopo contratual), Q27 (escopo Assembleia).

### Prioridade documental
Q6-Q11, Q19-Q22, Q24-Q25, Q28, Q30. Sessão de `claude-md-management` ou pré-Marco 1.

### Sem ação necessária (confirmações ✅)
Q31-Q38 (8 itens). Spec viva e PDFs concordam.

### Decisões pendentes com João (consolidadas)
1. **Q2** — Quota Morador Base Connect Me: 2/ano (Decision 10) ou 0 (functional-matrix)?
2. **Q4** — Agência tem login (role marketing) ou opera só via token?
3. **Q5** — Síndico N1 entrega certificado LMS?
4. **Q23** — Sistema de Cupons (ID_LOJA+SEQ+PALAVRA): in-scope, descartado ou substituído por Vizinhança?
5. **Q25** — Score Governança 1-10 + Score Compliance 0-100 (2 scores) ou um único 0-100?
6. **Q26** — Aditivo contratual formalizado para escopo expandido (9 sprints vs 4)?
7. **Q27** — Aditivo contratual para Live Assembly (era assíncrono)?
8. **Q28** — Lives no LMS: restritas a Admin MS ou abertas a síndicos/empresas?
9. **Q39** — Vínculos profissionais Banco Talentos: 3 ou 5?
10. **Q40** — Eleição gera avaliação escondida automaticamente?

---

## Acompanhamento

Quando uma quebra for resolvida:
1. Mover entrada para seção `## ✅ Resolvidos` com data + commit/PR.
2. Atualizar fontes envolvidas (bumpar `version:` no frontmatter).
3. Se a resolução abriu nova decisão → registrar em `backend/.kiro/STATE.md` como `D-0XX`.

## ✅ Resolvidos

*(vazio — esta é a primeira rodada de auditoria)*
