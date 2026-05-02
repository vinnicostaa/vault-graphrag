---
title: Aplicações Master Síndico — cruzamento research condominial-br × 11 sub-produtos
type: note
tags: [master-sindico, concorrentes, aplicacoes, posicionamento, compliance-br, lgpd, condominial-br]
source: 13-research/condominial-br/_destilado.md + 00-product/portfolio-de-produtos.md + 00-product/sub-produtos/01..11 + 04-requirements/compliance-lgpd.md + STATE.md (D-051, D-057, D-066, D-075)
created: 2026-04-23
updated: 2026-04-23
aliases: [aplicacoes-condominial-br, cruzamento-concorrentes-ms]
projeto: master-sindico-v2
sprint: Sprint 10 / M1 2026-05-08
status: destilado-inicial
---

# Aplicações Master Síndico — cruzamento condominial-br × 11 sub-produtos

Destilação que cruza o [[_destilado|research condominial-br]] (Superlógica, TownSq, Igora*, Cond21, Cibus/Apusia*, ANOREG regulatório, ABRASCOND) com os **11 sub-produtos** do [[../../00-product/portfolio-de-produtos|portfolio MS]]. Foco pragmático M1 (2026-05-08): o que **diferencia**, o que **falta**, o que é **anti-feature explícita**, o que é **conformidade BR obrigatória**.

\* Igora, Cibus, Apusia: não confirmados em research aberta (ver [[_destilado#1-1-tabela-comparativa-principal]] — marcados como pendência de validação com stakeholder).

---

## 1. Sumário executivo (top-5 ≤ 400 palavras)

**MS é produto de governança + ecossistema, não ERP**. Nenhum concorrente BR ocupa esse espaço: Superlógica e Cond21 são ERPs financeiros; TownSq é app de comunicação; uCondo/Condomob são híbridos leves. Isso garante **5 diferenciais estruturais defensáveis** e obriga **4 conformidades BR** não-negociáveis em M1.

**Diferenciais** (ordem de defensibilidade):

1. **Reputação Bronze→Diamante determinística** (Produto 3 + D-051 + ADR-0023) — fórmula pública auditável (40/25/15/10/10), impossível de comprar, LGPD Art. 20 explicável. Mercado usa estrelas subjetivas manipuláveis; MS ocupa o vácuo com trust-first. Moat real num mercado com histórico de fraude (Seleta/SP R$900k — [[_destilado#4-1-fraudes]]).
2. **Connect Me unidirecional 4 direções + Marketing Link** (Produto 2) — ninguém tem marketplace validado + LGPD by design (PII escondida até "Tenho Interesse"). TownSq tem contatos abertos; Superlógica não tem. MS evita ruído de WhatsApp com trilha auditável.
3. **Assembleia Live nativa end-to-end** (Produto 5) — full-stack Livekit + gov.br (Lei 14.063/2020) + voto fracional + ata PDF/A imutável + carimbo de tempo. Zoom-adapter enterprise (Superlógica) não compete; Lei 14.309/2022 valida MVP presencial + online M4+.
4. **Banco de Talentos interno com vínculos** (Produto 7) — nicho sub-atendido (porteiro/doméstica/NR-23/35) que LinkedIn/Catho ignoram. Vídeo-currículo 90s + busca B2B2C estruturada. Inédito no setor.
5. **Vizinhança hyperlocal com cupons + seal do síndico** (Produto 8) — curadoria condominial com Sistema de Cupons ativo (D-059 Fase 8, `ID_LOJA(5)+SEQ(3)+PALAVRA(5)`, lock 4h). Groupon genérico + TownSq contatos não cobrem. Diferencial marketing-driven.

**Anti-features explícitas** (protege foco M1):

- **Sem cobrança de taxa condominial / boleto / emissão fiscal** — Superlógica domina; MS integra via API (M4+, D-013 legado). Ver [[../../00-product/vision#o-que-nao-e]].
- **Sem NF-e do síndico profissional** — usuário emite por fora (fora de escopo M1-M3).
- **Sem chat/timeline de mensagens livres** — Connect Me é formulário unidirecional (Regra de Ouro 3).

**Conformidades BR obrigatórias** (M1):

1. **LGPD Art. 8/18/37/48** → [[../../04-requirements/compliance-lgpd|LGPD-001..030]] já cobre consent versionado, `lgpd_logs` append-only 5 anos, DPO canal, breach 72h. **Agravante condominial** (biometria porteiria, placa CFTV): LGPD-016 trata dados sensíveis via consent destacado M3+; CFTV rotation policy é feature do sub-produto Governança (não explicitada hoje — ver Gap G-APP-03 abaixo).
2. **Código Civil arts. 1.331-1.358 + Lei 4.591/64 + Lei 14.309/2022** → Assembleia Live (Produto 5) + Governança (Produto 1) operacionalizam quórum fracional, mandato 2 anos, AGO anual, destituição 2/3, conselho fiscal. Compliance (Produto 9) bloqueia 4 gatilhos.
3. **Lei 14.063/2020 (assinatura eletrônica)** → MS usa **avançada gov.br** em ata de AGO/AGE (suficiente por STJ dez/2024); ICP-Brasil opcional para averbação cartorial. Stack jurídica fechada.
4. **NR-1/PGR + CCT SECOVI-SP** → Compliance markers empresa (Produto 9 §19.2) cobrem `nr_1/nr_5/nr_6/nr_17/nr_35` autodeclarados. **Seal Compliance SST** (barrar prestador sem PGR na Connect Me) é oportunidade M3+ (G-APP-05).

Link detalhe: [[_destilado]]; [[../../00-product/portfolio-de-produtos]]; [[../../04-requirements/compliance-lgpd]].

---

## 2. Matriz cruzada — concorrente × 11 sub-produtos MS

Legenda: ✅ MS diferencia; ⚠️ MS sobrepõe (empate técnico); ❌ MS não cobre (anti-feature ou gap); 🆕 sem equivalente direto no mercado.

| Sub-produto MS | Superlógica | TownSq | Igora* | Cond21 | uCondo/Condomob | Cibus/Apusia* | MS |
|---|---|---|---|---|---|---|---|
| 1. Governança Institucional | Parcial (financeiro-centric) | ❌ | ? | Silos | Híbrido | ? | ✅ Timeline imutável + Plano Diretor reativo + Score único |
| 2. Connect Me (4 direções) | ❌ | Contatos abertos | ? | ❌ | Básico | ? | 🆕 Unidirecional + LGPD by design + 4 direções canônicas |
| 3. Reputação Bronze→Diamante | ❌ | ❌ | ? | ❌ | 5 estrelas livres | ? | 🆕 Determinística (D-051) + auditável + anti-fraude |
| 4. Conteúdo & Vídeos (lock 90d) | ❌ | ❌ | ? | ❌ | Blog (ucondo) | ? | ✅ Lock 90d + moderação editorial + Rule 4 votação |
| 5. Assembleia Live | Zoom plugin Enterprise | ❌ nativo | ? | Híbrido parcial | Parcial | ? | 🆕 Full-stack + gov.br + voto fracional + ata imutável |
| 6. LMS/Academia | ❌ (só blog) | ❌ | ? | ❌ | Blog jurídico | ? | 🆕 5 áreas BR + 15 telas + certificados verificáveis |
| 7. Banco de Talentos | ❌ | ❌ | ? | ❌ | ❌ | ? | 🆕 Vídeo-currículo 90s + 18 áreas + B2B2C |
| 8. Vizinhança (cupons) | ❌ | Contatos locais | ? | ❌ | ❌ | ? | 🆕 Seal síndico + cupons `ID_LOJA+SEQ+PALAVRA` + raio 2km |
| 9. Compliance (4 gatilhos + Score) | ❌ (só contábil) | ❌ | ? | ❌ | Blog | ? | 🆕 Score único 10 componentes + LGPD logs + dossiê exportável |
| 10. Marketing/Agências | ❌ | ❌ | ? | ❌ | ❌ | ? | 🆕 Sub-produto interno + DelegationGrant ABAC + Marketing Link |
| 11. Administradoras Condominiais | Core de negócio (são cliente Superlógica) | Plano Admin Digital | ? | Core | Core | ? | ⚠️ EmpresaSindicaLink + permissões granulares (MVP); tenant próprio M5+ |

**Leitura**: MS tem **8 de 11 sub-produtos sem equivalente direto (🆕)** no mercado BR. Onde sobrepõe (1, 4, 11), diferencia por arquitetura (timeline imutável, lock 90d, permissões granulares). Onde decide não competir (financeiro), é anti-feature explícita.

---

## 3. Cruzamento detalhado por concorrente

### 3.1 Superlógica — ERP financeiro-forte (B2B administradoras)

**Pontos fortes** ([[_destilado#1-1]]): cobrança boleto/Pix, contabilidade, Gruvi app morador, **API pública OAuth2 RESTful com webhook** (diferencial raro no setor).

**Sobreposição MS**: quase nenhuma — Superlógica é ERP, MS é governança.

**Oportunidades MS**:
- **Integração via `IIdentityProvider`/`IPaymentGateway` abstratos** (D-056 agnosticismo) já permite plugar Superlógica API para puxar dados financeiros sem refazer ERP. Compliance (Produto 9) Score componente 6 ("Fundo de Reserva em dia") pode consumir dados Superlógica via adapter M4+.
- Superlógica **não tem** Connect Me, Reputação, Assembleia Live nativa, LMS, Banco Talentos, Vizinhança — **não compete** com nada core do MS.

**Riscos**:
- Superlógica pode copiar Connect Me + LMS em 12-18 meses ([[_destilado#7-riscos]]). Mitigação: velocidade + DNA social + parcerias institucionais ABRASCOND/SECOVI-SP.

**Anti-features MS** (confirmadas):
- ❌ Cobrança de taxa condominial / boleto Pix / emissão fiscal → Superlógica domina; MS integra, **não replica**.
- ❌ Folha de pagamento condominial → fora de escopo M1-M6.
- ❌ NF-e emitida pelo síndico profissional → usuário emite por fora.

**ADR necessária**: `ADR-0034 integracao-superlogica-api` (M4+ spike; ver Gap G-APP-01).

### 3.2 TownSq — comunicação + moradores (32k condomínios BR + US/CA/MX)

**Pontos fortes** ([[_destilado#1-1]]): chat morador-morador, reservas áreas comuns, portaria digital, mobile-first, presença internacional.

**Sobreposição MS**: **comunicados oficiais** (Produto 1, Governança), **timeline** (Produto 1), **assembleia** (Produto 5). **MS vence** por imutabilidade + valor jurídico + Score de Governança.

**Diferenciais MS vs TownSq**:
- **Timeline append-only** (Req 11, INV-institutional) vs mural editável TownSq → prova documental para processo judicial + auditoria + dossiê C11.
- **Comunicados imutáveis pós-publish** (correção via adendo Req 25 — D-054 4-eyes policy) vs comunicado editável TownSq → anti-gaslighting síndico.
- **Reputação Bronze→Diamante** — TownSq não tem; moradores e síndicos operam às cegas em contratação.
- **Connect Me unidirecional** — TownSq tem chat aberto; MS tem formulário estruturado + LGPD logs (Req 19).

**Gap TownSq cobre, MS não cobre (MVP)**:
- **Reserva de áreas comuns** (churrasqueira, salão de festas) → TownSq tem feature madura. MS **não tem** no portfolio atual. Status: backlog M4+ se demanda cliente master-sindico exigir. **Gap G-APP-02**.
- **Portaria digital / controle de acesso** → TownSq tem; MS não. **Gap G-APP-03** (relacionado a LGPD CFTV + biometria).
- **Achados e perdidos, classificados internos** → TownSq tem; MS decide NÃO fazer (anti-feature — fora do DNA governança).

**Riscos**:
- TownSq adiciona "governança formal" e tenta rebrandar → mitigação: profundidade MS em 11 sub-produtos integrados vs TownSq genérico ([[_destilado#7]]).

### 3.3 Igora* — não confirmado

[[_destilado#1-2]] marca como pendência: não localizado em pesquisa aberta, pode ser typo ou produto descontinuado. **Ação**: confirmar com stakeholder (DT-006). Até lá, **tratar como ERP-legado hipotético** (pressupor paridade funcional Cond21).

### 3.4 Cond21 — ERP legado (Group Software)

**Pontos fortes**: incumbente BR, suporte híbrido de assembleia, folha via ERP.

**Sobreposição MS**: mesma de Superlógica (financeiro). **MS vence** por cloud-native, UX moderna, mobile-first, API-first.

**Diferencial claro**: Cond21 é **legacy desktop-first**; MS é SaaS cloud-native com agnosticismo de provedor (D-056). Mercado está migrando — MS captura greenfield + churn de administradoras insatisfeitas.

**Não há feature Cond21 que MS precise cobrir** (todas anti-features financeiras).

### 3.5 uCondo / Condomob — SaaS híbrido moderno (47k lares uCondo)

**Pontos fortes** ([[_destilado#1-1]]): blog jurídico forte (modelo conteúdo validado), app morador, avaliação livre de prestadores (parcial).

**Sobreposição MS**: avaliação de prestadores (frágil em uCondo — 5 estrelas sem auditoria) vs **Reputação determinística MS** (D-051). MS vence.

**Lição a replicar**:
- **Blog jurídico modelo uCondo** ([[_destilado#2-1-g]]) combinado com **LMS MS** (Produto 6) = "universidade do síndico". Conteúdo de marketing (NR-1 psicossocial 2026, LGPD condomínio) aplicado em trilha de curso. Ver recomendação [[_destilado#8-comunicacao-marketing]].

### 3.6 Cibus / Apusia — não confirmados

[[_destilado#9]] marca como pendências de pesquisa. Apusia pode ser typo de APSA (ver source-04-apsa). Cibus não encontrado. **Ação**: spike dedicado M3+ se mencionados pelo cliente master-sindico.

### 3.7 ANOREG (cartórios) — registro de ata

**Contexto** ([[_destilado#9]]): hoje síndico leva ata impressa ao cartório para averbação. MS pode integrar com ANOREG-BR para **ata notariada digital** (requer ICP-Brasil qualificada).

**Status no portfolio**: fora de escopo M1-M3. Assembleia Live (Produto 5) já exporta **PDF/A + hash + carimbo de tempo** (stack MP 2.200-2/2001 + Lei 14.063/2020). ICP-Brasil opcional já previsto.

**Oportunidade M3+**: **Gap G-APP-04 — integração ANOREG-BR spike**. Abrir ADR se demanda cliente confirmar. Nice-to-have, não bloqueia M1.

---

## 4. Conformidade BR — checklist cruzado

### 4.1 LGPD com agravante PII condominial

| Controle | Origem | Sub-produto MS | Status |
|---|---|---|---|
| Consent versionado (termos 1-14) | LGPD Art. 8 + [[../../04-requirements/compliance-lgpd|LGPD-001]] | Produto 9 Compliance (C2/C3) | ✅ M1 planejado |
| `lgpd_logs` append-only 5 anos | Art. 37 + [[../../04-requirements/compliance-lgpd|LGPD-011]] | Produto 9 (C9) + INV-COMP-6 | ✅ M1 |
| Direito exclusão SLA 15d | Art. 18 VI + [[../../04-requirements/compliance-lgpd|LGPD-007]] | Produto 9 + IDN-022 | ✅ M1 parcial manual; M2 auto |
| Breach notification 72h | Art. 48 + LGPD-012 | Produto 9 runbook | ✅ M1 runbook |
| **CFTV rotation 30d** (MJ recomenda) | [[_destilado#3-1]] | **Produto 1 Governança — FALTA** | ⚠️ Gap G-APP-03 |
| **Biometria ex-morador exclusão ativa** | LGPD-016 + [[_destilado#3-1]] | Produto 1 + Produto 9 | ⚠️ M3+ quando biometria ativa |
| **Vazamento facial terceirizado (lição Jundiaí 2024)** | [[_destilado#4-2]] | Transversal — criptografia at-rest + at-transit já exigida em LGPD-023/024 | ✅ |
| DPA fornecedores (Mux, Zitadel, Livekit, Resend, Twilio) | LGPD-014 | Transversal | ⚠️ 5 DPAs pendentes |

**Agravante condominial explícito**: PII condominial inclui **telefone morador (comum em lista de prestação de contas), foto em portaria, placa de carro em CFTV**. Ver [[_destilado#3-1]]. MS hoje:

- ✅ **Telefone morador**: Connect Me escondido até "Tenho Interesse" (Produto 2, Req 19).
- ⚠️ **Foto portaria / biometria**: sem spec dedicada no Produto 1. **Gap G-APP-03 — adicionar CFTV rotation + biometria exclusão ativa em Produto 1 Governança** (abrir ADR ou D-### em Sprint 11).
- ✅ **Placa CFTV**: LGPD-017 classifica como HIGH; encryption at rest obrigatório.

### 4.2 Código Civil + Lei 4.591/64 + Lei 14.309/2022

| Artigo | Operacionalização MS | Sub-produto |
|---|---|---|
| Art. 1.347 (mandato ≤ 2 anos) | `mandates.end_date` + gatilho G1 bloqueia | Produto 1 + Produto 9 |
| Art. 1.348 (atribuições) | Timeline imutável + prestação contas | Produto 1 + Produto 9 (dossiê C11) |
| Art. 1.350 (AGO anual obrigatória) | Alerta Score componente 7 | Produto 5 + Produto 9 |
| Art. 1.341 (quórum por obra: maioria/maioria/2/3) | `Assembly.quorum_rules` por `AgendaItemType` | Produto 5 |
| Art. 1.354 (ata obrigatória AGO/AGE) | D-053 homologação deliberativa | Produto 5 |
| Art. 1.356 (conselho fiscal) | Persona `resident` convidado + read-only C5 | Produto 9 §6 + [[_destilado#5]] persona esquecida |
| Art. 1.357 (destituição 2/3) | Quórum especial Assembly | Produto 5 |
| **Lei 4.591/64 art. 28+ incorporação** | Backlog — módulo "recebimento do condomínio" | **Gap G-APP-06 M5+** |
| Lei 14.309/2022 (assembleia virtual) | Produto 5 full-stack + voto eletrônico | ✅ M3 |

**Cobertura**: alta. Apenas Lei 4.591/64 art. 28 (transição incorporadora→condomínio constituído) é backlog M5+ — [[_destilado#3-2]].

### 4.3 Assinatura eletrônica (Lei 14.063/2020)

| Ato | Nível exigido | Provedor MS | Status |
|---|---|---|---|
| Ata AGO/AGE comum | **Avançada** (gov.br serve) | gov.br via `IIdentityProvider` extensão | ✅ M3 planejado |
| Ata para averbação cartório | **Qualificada** (ICP-Brasil) | Certisign/Soluti adapter opcional | ⚠️ M3 opcional |
| Convenção do condomínio | **Qualificada** | ICP-Brasil | ⚠️ M4+ se demanda |
| Voto em assembleia | **Avançada** (identificação inequívoca) | gov.br + device binding | ✅ M3 |

STJ dez/2024 ([[_destilado#3-3]]) valida assinatura sem ICP-Brasil se autoria + integridade comprovadas → **gov.br é suficiente para MVP**. Reduz custo de integração com certificadoras privadas em M1-M3.

### 4.4 NR-1/PGR + CCT SECOVI-SP

| Obrigação | Persona | Cobertura MS |
|---|---|---|
| PGR do condomínio (porteiro/zelador CLT) | Síndico | Compliance markers síndico `conformidade_nr1` (Produto 9 §19.1) — autodeclarado |
| PGR do prestador terceirizado | Empresa | Compliance markers empresa `nr_1` (§19.2) — autodeclarado |
| **Seal Compliance SST** (barrar prestador sem PGR na Connect Me) | Connect Me + Reputação | ⚠️ **Gap G-APP-05 — oportunidade M3+** |
| NR-1 psicossocial vigor 25/05/2026 | Síndico + Empresa | Módulo Compliance preparado; ADR necessária |
| CCT SECOVI-SP 2024/2025 (reajuste, piso) | Síndico | ⚠️ **Gap G-APP-07 — alerta CCT via scraping SECOVI-SP** — recomendação [[_destilado#8]] |

**Oportunidade crítica** ([[_destilado#3-4]]): NR-1 psicossocial vigor 25/05/2026 = notícia quente no mercado. **MS primeiro a chegar** = moat regulatório. Abrir ADR `ADR-0035 seal-compliance-sst-connect-me`.

---

## 5. Dores do síndico BR × features MS

Cruzamento de [[_destilado#5-expectativas-usuario]] com portfólio.

| Dor (SindicoNet/Reclame Aqui) | Feature MS correspondente |
|---|---|
| SLA 4 dias p/ reclamação | Connect Me Morador→Síndico (Produto 2) + timeline (Produto 1) |
| Documentação fácil (chat + foto → registro) | Produto 1 timeline append-only + anexos |
| Transparência sem vulnerabilidade (anonimato reclamante) | Avaliação anônima [[../../04-requirements/compliance-lgpd|LGPD-019]] + enquete anonimizada INS-030 |
| Prestação contas automatizada | Produto 9 Dossiê C11 exportável PDF/A |
| Integração financeiro (não silo) | Superlógica adapter M4+ (G-APP-01) |
| Mobile-first | Stack Flutter (D-048 Bloc + D-049 freezed) |
| Resumos CCT/NR-1/LGPD | Produto 6 LMS 5 áreas + blog modelo uCondo |
| **Bem-estar síndico (burnout, medo processo)** | ⚠️ **Gap G-APP-08 — modo "não perturbe" + escalonamento urgência** [[_destilado#4-3]] |

Dor **conselho fiscal esquecido** ([[_destilado#5]]) = **persona recuperada** pelo MS (Produto 9 §6 "Conselho" com role_type delimitado + read-only C5). Diferencial vs todos os concorrentes.

---

## 6. Gaps registrados (G-APP-##)

Novos gaps desta destilação. Registrar em [[../../11-audit/AUDIT]] e propor sprint.

| ID | Gap | Sub-produto | Prioridade | Sprint sugerida |
|---|---|---|---|---|
| **G-APP-01** | Adapter Superlógica API (`IFinancialProvider`) + Score componente 6 Fundo Reserva | Produto 9 + Produto 1 | Média | M4+ (12-14) |
| **G-APP-02** | Reserva de áreas comuns (paridade TownSq) | Produto 1 ou novo Produto 12 | Baixa | M4+ se demanda |
| **G-APP-03** | CFTV rotation 30d + biometria exclusão ativa + política portaria digital | Produto 1 + Produto 9 | **Alta** (LGPD) | 11 |
| **G-APP-04** | Integração ANOREG-BR (ata notariada digital) | Produto 5 | Baixa | M3+ spike |
| **G-APP-05** | Seal Compliance SST (barrar prestador sem PGR na Connect Me) | Produto 2 + Produto 3 | **Alta** (moat regulatório NR-1 2026) | 11-12 |
| **G-APP-06** | Módulo transição incorporadora→condomínio (Lei 4.591/64 art. 28) | Novo Produto 13? | Baixa | M5+ |
| **G-APP-07** | Alerta CCT SECOVI-SP via scraping/feed | Produto 9 + Produto 1 | Média | 11 |
| **G-APP-08** | Modo "não perturbe" síndico + escalonamento urgência | Produto 1 + Produto 2 | Média | M3 |

**Total**: 8 gaps. **2 de alta prioridade** (G-APP-03 LGPD condominial, G-APP-05 Seal SST pré-NR-1 psicossocial 2026) — abrir ADR + ticket Sprint 11.

---

## 7. Decisões candidatas (D-BR-###) a levar para STATE

- **D-BR-001**: MS é **governança não-ERP** — anti-feature explícita em cobrança/boleto/NF-e/folha. Integração via adapter `IFinancialProvider` (Superlógica M4+). **Congelado**.
- **D-BR-002**: Assinatura avançada **gov.br** é default em MVP ata AGO/AGE (STJ dez/2024 valida). ICP-Brasil opcional. Reduz custo integração certificadoras.
- **D-BR-003**: **Seal Compliance SST** barrar prestador sem `nr_1`/`nr_5` na Connect Me → abrir ADR-0035 pré-M3 (moat pré-NR-1 psicossocial 2026-05-25).
- **D-BR-004**: **CFTV rotation 30d + biometria exclusão ativa** viram spec do Produto 1 Governança (Sprint 11) — vácuo crítico de LGPD condominial no portfolio atual.
- **D-BR-005**: **Conselho fiscal como persona primeira-classe** (vs concorrentes que esquecem) — role_type delimitado `resident` convidado + views read-only C5/C9 ratificadas.
- **D-BR-006**: Integração **ANOREG-BR** fica backlog M3+ spike; PDF/A + hash + carimbo de tempo + ICP-Brasil opcional do Produto 5 já atende uso majoritário.

---

## 8. Riscos competitivos (mitigação por sub-produto)

| Risco | Probabilidade | Sub-produto atingido | Mitigação |
|---|---|---|---|
| Superlógica copia Connect Me + LMS | Média (12-18m) | 2, 6 | Velocidade + parcerias ABRASCOND/SECOVI-SP + profundidade 11 sub-produtos |
| TownSq adiciona governança formal | Média | 1, 9 | Timeline imutável + Score único (D-066) como moat arquitetural |
| Administradoras tradicionais (APSA, Zelo) fazem white-label | Baixa | 11 | Elas não têm tech; viram clientes/resellers |
| ANPD escala multas LGPD | Alta (2026+) | Transversal | LGPD-by-design desde M1 = moat crescente |
| NR-1 psicossocial vigor 2026-05-25 | **Confirmada** | 9 + Connect Me (G-APP-05) | Seal SST primeiro a chegar |

Fonte: [[_destilado#7-riscos-competitivos]].

---

## 9. Próximos passos

- [ ] Abrir `ADR-0034 integracao-superlogica-api` (M4+ spike — G-APP-01).
- [ ] Abrir `ADR-0035 seal-compliance-sst-connect-me` (M3 — G-APP-05 moat NR-1 2026).
- [ ] Abrir `ADR-0036 cftv-rotation-biometria-lgpd-condominial` (Sprint 11 — G-APP-03).
- [ ] Confirmar com stakeholder: Igora, Cibus, Apusia existem? (DT-006).
- [ ] Propor D-BR-001..D-BR-006 no próximo ciclo de dual-check arquitetural.
- [ ] Registrar G-APP-01..G-APP-08 em [[../../11-audit/AUDIT]].
- [ ] Atualizar [[../../00-product/portfolio-de-produtos]] Produto 1 e Produto 9 com referência a CFTV/biometria (G-APP-03).
- [ ] LMS Produto 6 adicionar trilha "NR-1 psicossocial 2026" (quick win marketing + regulação).
- [ ] Considerar campanha marketing "MS primeiro com Seal Compliance SST" pré-2026-05-25.

---

## Referências

- [[_destilado]] — research condominial-br consolidado (16 sources).
- [[../../00-product/portfolio-de-produtos]] — 11 sub-produtos (Fase 8 consolidado).
- [[../../00-product/vision]] — pilares governança + anti-catálogo.
- [[../../00-product/sub-produtos/_moc]] — catálogo detalhado 01..11.
- [[../../00-product/sub-produtos/09-compliance]] — C1-C11 + 4 gatilhos + Score único D-066.
- [[../../04-requirements/compliance-lgpd]] — LGPD-001..030.
- [[../../STATE]] — D-051 (fórmula Reputação), D-057 (planos globais), D-066 (Score único), D-067 (stack canônica), D-072 (Admin role), D-073 (Agência sub-produto interno).
- [[../../02-architecture/adr/0023-ranking-deterministico]] — Reputação determinística.
- [[../../02-architecture/adr/0032-global-plans-stripe-style]] — planos globais.
- [[../linkedin/_destilado]] — anti-fake de empresa além de CNPJ (cross-ref Connect Me).
- [[source-01-superlogica]]..[[source-16-lei-14063]] — fontes individuais.
