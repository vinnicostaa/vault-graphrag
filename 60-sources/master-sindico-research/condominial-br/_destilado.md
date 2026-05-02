---
title: Destilado — Mercado condominial BR para Master Síndico v2
type: note
tags: [master-sindico, concorrentes, regulacao, br, destilado, research]
source: 13-research/condominial-br/source-01 a source-16
created: 2026-04-23
updated: 2026-04-23
aliases: [destilado-condominial-br, mapa-competitivo-condominial]
projeto: master-sindico-v2
sprint: Sprint 10 / M1 2026-05-08
fontes_total: 16
status: inicial-revisar
---

# Destilado — Mercado condominial BR para Master Síndico v2

> Fontes pesquisadas (16): `source-01` a `source-16` neste diretório. Cada uma tem frontmatter, URLs e ponto-de-valor para o produto.

---

## 1. Mapa competitivo

### 1.1 Tabela comparativa principal

| Dimensão                     | Superlógica                        | TownSq                              | Igora*                   | Cond21 (Group)                 | uCondo / Condomob          | **Master Síndico (nós)**                          |
|------------------------------|------------------------------------|-------------------------------------|--------------------------|--------------------------------|----------------------------|---------------------------------------------------|
| **Posicionamento**           | ERP para administradoras (B2B)     | App de comunidade (B2B + síndico)  | *não verificado*         | ERP tradicional admin.         | SaaS moderno híbrido       | SaaS de **governança** + comunidade + reputação  |
| **Foco primário**            | Financeiro, cobrança               | Comunicação, reservas, portaria    | —                        | Financeiro + contabilidade     | Financeiro + comunicação   | Governança institucional + 7 sub-produtos         |
| **Público**                  | Administradoras médio/grande       | Síndicos + administradoras          | —                        | Administradoras legado         | Síndicos + administradoras | Síndico (profissional e morador) + condomínio    |
| **Mobile app morador**       | Sim (Gruvi) — crítica adoção       | Sim (core product)                  | —                        | Parcial (Medições)             | Sim                        | Sim (core)                                        |
| **Assembleia virtual**       | Enterprise (via Zoom)              | Não nativo                          | —                        | Suporte híbrido                | Parcial                    | **Nativo full-stack + assinatura gov.br**         |
| **LMS (treinamento síndico)**| Não                                | Não                                 | —                        | Não                            | Não                        | **Sim (sub-produto)**                             |
| **Reputação de prestador**   | Não                                | Não                                 | —                        | Não                            | Parcial (avaliação livre)  | **Bronze→Diamante determinístico**                |
| **Marketplace prestadores**  | Não                                | Básico (contatos)                   | —                        | Não                            | Básico                     | **Connect Me — unidirecional validado**           |
| **API pública**              | **Sim, OAuth2, RESTful, webhook**  | Não documentada publicamente        | —                        | Limitada                       | Limitada                   | **Sim desde dia 1 (API-first)**                   |
| **LGPD by design**           | Parcial (tem incidentes)           | Parcial                             | —                        | Parcial                        | Blog forte                 | **Sim (nativo)**                                  |
| **Compliance trabalhista**   | Folha via ERP                      | Folha no plano Admin Digital        | —                        | Parcial                        | Parcial                    | **Módulo PGR/NR-1 + SST (diferencial)**           |
| **Assinatura gov.br**        | Não documentado                    | Não                                 | —                        | Não                            | Não                        | **Sim, nativo (avançada)**                        |
| **Pricing transparente**     | Sob consulta                       | Sob consulta                        | —                        | Sob consulta                   | Sob consulta               | (a definir — transparência = ativo)              |
| **Range plano BR**           | Econômico → Enterprise III         | Essencial → Admin Digital           | —                        | Cond21 / Cond21 One            | Variável                   | Bronze → Diamante (tier-based)                   |
| **Presença**                 | ~líder de mercado BR               | 32k condomínios BR (+ US/CA/MX)     | —                        | Incumbente BR                  | 47k lares (uCondo)         | Sprint 10 / M1 — 1º cliente master-sindico       |

\* **Igora** — não foi possível localizar empresa/produto com esse nome específico no mercado condominial BR ativo. Pode ser:
- Marca regional muito pequena sem presença digital.
- Nome alternativo de outro produto (ex.: "Igora" pode ser typo).
- Produto descontinuado.

Sugestão: confirmar com o stakeholder se existe fonte direta ou se é outro nome. Ver também `source-04-apsa.md` sobre "Apusia".

### 1.2 Segmentos atendidos pelos concorrentes

```
  Síndico morador ────► TownSq, uCondo, Condomob
                         │
                         ├──► (gap: governança profissional formal)
                         │     └──► MASTER SÍNDICO
                         │
  Síndico profissional ─► TownSq, uCondo
                         │
  Administradora       ─► Superlógica, Cond21, APSA (self), Group
                         │
  Conselho fiscal      ─► NENHUM (gap explícito)
                         │     └──► oportunidade p/ MASTER SÍNDICO
                         │
  Morador              ─► Todos (via app)
  Prestador            ─► NENHUM (gap explícito)
                               └──► MASTER SÍNDICO Connect Me
```

---

## 2. Posição estratégica do Master Síndico

### 2.1 Diferenciais estruturais

#### A — Governança (não ERP)
- Master Síndico **não compete com Superlógica/Cond21** no financeiro operacional.
- Operacionaliza **Cap. VII do Código Civil (arts. 1.331-1.358)**: mandato do síndico, convocação AGO, quórum por tipo de deliberação, destituição, conselho fiscal, prestação de contas.
- Pode integrar com ERPs via API (Superlógica primeiro — `source-15`).
- **Tese**: o síndico tem problemas que o ERP não resolve.

#### B — Connect Me unidirecional
- Prestador propõe; síndico convoca; **morador nunca é exposto ao prestador diretamente**.
- Protege privacidade (LGPD) e combate fornecedor cativo / kickback (`source-14`).
- Avaliação pós-serviço alimenta reputação determinística.
- Todos os concorrentes têm marketplace aberto ou nenhum — Master Síndico ocupa o meio.

#### C — Reputação Bronze→Diamante determinística
- Não é 5 estrelas subjetivas. É agregado objetivo:
  - CNPJ ativo + tempo de mercado.
  - PGR/NR-1 em dia (`source-12`).
  - Histórico de serviços em condomínios distintos.
  - SLA médio cumprido.
  - Taxa de conclusão sem chargeback/disputa.
  - Avaliações explícitas (calibradas).
- **Vantagem**: não é manipulável por review-farm; síndico confia.

#### D — Assembleia live
- Full-stack próprio (não é só "link do Zoom").
- Integração **gov.br** para assinatura avançada (`source-11`, `source-16`).
- Voto eletrônico com identificação inequívoca (Lei 14.309/2022 — `source-10`).
- Ata PDF/A + hash + carimbo do tempo + trilha auditável.
- Exportação ICP-Brasil opcional p/ registro cartorial.
- **Nenhum concorrente tem isso end-to-end em BR**.

#### E — LMS embutido
- Conteúdo curado + certificação → alimenta reputação do síndico.
- Parcerias potenciais com ABRASCOND/ABRASSP (`source-07`).
- Complementa (não substitui) cursos presenciais das associações.

#### F — Vizinhança (sub-produto)
- Comunidade intra e inter-condomínios.
- Diferente do chat do TownSq — tem camada editorial + moderação institucional.

#### G — Conteúdo/vídeos
- Marketing de conteúdo jurídico (NR-1, LGPD, CCT) — modelo validado por uCondo (`source-05`).
- Combinado com LMS vira **universidade do síndico**.

### 2.2 Gaps explícitos no mercado

| Gap                                         | Quem endereça hoje | Oportunidade Master Síndico          |
|---------------------------------------------|--------------------|--------------------------------------|
| Governança formal do mandato do síndico     | Ninguém            | **Core do produto**                  |
| Reputação objetiva de prestador             | Ninguém            | **Connect Me + Bronze→Diamante**     |
| Compliance SST integrado (PGR/NR-1)         | Ninguém            | **Módulo SST**                       |
| LMS do síndico                              | Associações (off)  | **LMS nativo**                       |
| Integração gov.br nativa                    | Ninguém            | **Assembleia live**                  |
| Transparência pública em prestação contas   | Parcial            | **Contas públicas por padrão**       |
| Conselho fiscal como persona                | Ninguém            | **Views específicas**                |
| Detecção de fraude assistida                | Ninguém            | **Comparação extrato ↔ balancete**   |

---

## 3. Obrigações regulatórias concretas

### 3.1 LGPD (Lei 13.709/2018) — `source-09`

| Item                       | Status no condomínio                                                            | Impacto no Master Síndico                                           |
|----------------------------|----------------------------------------------------------------------------------|---------------------------------------------------------------------|
| Classificação de dados     | Nome, CPF, RG, telefone = dado pessoal; biometria, placas, CFTV = dado sensível | RIPD automático; classificação no schema                           |
| Base legal                 | Execução de contrato (convenção) + legítimo interesse (segurança)               | Registrar base legal por finalidade                                 |
| Direito à exclusão         | Limitado por obrigação legal (atas, histórico financeiro)                       | Workflow de exclusão com triagem de exceção                         |
| Retenção de atas           | **10 anos** (prescrição civil, art. 205 CC)                                     | Política de retenção por tipo de documento                          |
| Retenção de CFTV           | Convenção do condomínio; padrão 30 dias (MJ recomenda)                          | Feature: rotação automática                                         |
| Biometria de ex-morador    | Exclusão ativa obrigatória                                                      | Trigger na baixa do morador                                         |
| Vazamento (caso real)      | Jundiaí/Campinas 2024 — facial de terceirizado vazou                            | Criptografia at-rest + at-transit + rotação de chave                |
| Multa                      | Até 2% faturamento, R$50M/infração; ANPD em advertência em 2024-2025            | Preparar p/ futura escalada sancionatória                          |
| Responsável                | Condomínio = controlador; síndico = responde civil/judicialmente                | DPO opcional como feature Diamante                                  |

### 3.2 Código Civil + Lei 4.591/64 — `source-10`

- **Cap. VII CC (arts. 1.331-1.358)** operacionalizável no produto:
  - Art. 1.347 — síndico eleito, mandato máx. 2 anos.
  - Art. 1.348 — atribuições (representação, convocação, prestação de contas).
  - Art. 1.350 — AGO anual obrigatória.
  - Art. 1.341 — quórum por tipo de obra (necessária = maioria; útil = maioria; voluptuária = 2/3).
  - Art. 1.357 — destituição 2/3.
  - Art. 1.356 — conselho fiscal.
- **Lei 4.591/64** ainda vigente em **incorporação imobiliária** (art. 28+). Master Síndico pode oferecer módulo "recebimento do condomínio" (transição incorporadora → condomínio constituído).

### 3.3 Assinaturas digitais de ata — `source-11`, `source-16`

**Stack legal**:
```
MP 2.200-2/2001 (ICP-Brasil)
  + Lei 14.063/2020 (níveis: simples/avançada/qualificada)
  + Lei 14.309/2022 (assembleia virtual autorizada)
  = ata eletrônica válida
```

**Regra prática**:
- Ata comum da AGO/AGE → **avançada** (gov.br serve).
- Ata p/ averbação em cartório → **qualificada** (ICP-Brasil exigida pelo cartório).
- Convenção → **qualificada**.
- Voto em assembleia → **avançada** (identificação inequívoca).

**STJ dez/2024** — assinatura sem ICP-Brasil NÃO é automaticamente inválida se autoria + integridade são comprovadas. Reforça viabilidade de gov.br.

### 3.4 NR-1 e PGR — `source-12`

- NR-1 atualizada 2020 instituiu **PGR** (substitui PPRA).
- **NR-1 atualizada 2026 (vigor 25/05/2026)** inclui **fatores psicossociais**.
- Condomínio com CLT (porteiro, zelador, faxineira) → PGR obrigatório.
- Elaborado por eng./téc. de Segurança do Trabalho.
- Prestador terceirizado tem seu PGR próprio — síndico é **corresponsável**.

**Oportunidade**: Master Síndico pode oferecer "Selo Compliance SST" visível no perfil — prestadores sem PGR não passam na Connect Me.

### 3.5 Trabalhista / CCT — `source-08`

- **SECOVI-SP** negocia CCT anual com sindicato de empregados em SP.
- CCT 2024/2025 define reajustes, pisos, benefícios.
- Módulo "alerta CCT" no Master Síndico → notifica síndico quando nova CCT publicada.

---

## 4. Cases reais que informam decisões

### 4.1 Fraudes — `source-14`
- **Seleta (SP)** desviou R$ 900k de 3 edifícios em 2024-2025.
- Múltiplos condomínios SP com dívidas R$ 1M+ após administradoras sumirem.
- **Padrões**: fornecedor cativo (5-10% kickback), superfaturamento, extratos adulterados, orçamentos fantasmas, desvio direto.

**Contra-medidas de produto**:
- 3 orçamentos obrigatórios com CNPJs independentes.
- Comparação extrato banco ↔ balancete (detecção automática).
- Prestação de contas pública por padrão (radical).
- Trilha append-only de atas e lançamentos.
- Connect Me só com prestadores verificados (CNPJ + PGR + histórico).

### 4.2 Vazamento LGPD — `source-09`
- Jundiaí/Campinas 2024: reconhecimento facial de empresa terceirizada vazou → dados de centenas na dark web.
- **Lição**: Master Síndico nunca deve armazenar biometria no servidor próprio sem criptografia forte; ou melhor, delegar autenticação sem biometria armazenada no backend.

### 4.3 Dores do síndico — `source-13`
- Baixa participação em assembleia → cumplicidade velada.
- Síndico estressado, hipervigilante, em burnout.
- Medo de processo por negligência.
- Prestação de contas abre flanco p/ contestação quando mal feita.

**Lição de produto**:
- Defaults seguros. Checklist obrigatório antes de enviar convocação, aprovar despesa, fechar mês.
- Bem-estar do síndico: modo "não perturbe", escalonamento de urgência, limitar notificações.

### 4.4 App Gruvi chicken-and-egg — `source-01`, `source-13`
- App do morador só funciona se % alta da população instalou.
- **Lição**: Master Síndico precisa de fallback sem app (web + e-mail + PDF) → UX progressiva, não impositiva.

---

## 5. Expectativas do usuário síndico/morador BR

### Baseado em reviews, fóruns, SindicoLab, SindicoNet

**Síndico quer**:
1. **Respostas rápidas** (SLA 4 dias p/ reclamação — fonte: SindicoNet).
2. **Documentação fácil** — chat + foto vira registro.
3. **Transparência sem ficar vulnerável** — confidencialidade do reclamante.
4. **Prestação de contas automatizada** — sem precisar montar manualmente.
5. **Integração com financeiro** (não quer outro silo).
6. **Mobile-first** — vida de síndico é no trânsito, no elevador, no domingo.
7. **Baixar regulação** — resumos de CCT, NR-1, LGPD aplicados ao contexto.

**Morador quer**:
1. **Boleto fácil** — Pix, app, sem fricção.
2. **Saber pra onde vai o dinheiro** — prestação de contas visível.
3. **Reclamar sem virar vizinho-inimigo** — anonimato na reclamação.
4. **Reservar área comum rápido** — sem ligação, sem fila.
5. **Participar de assembleia sem sair de casa** (Lei 14.309/2022 resolveu juridicamente).
6. **Saber as regras** — convenção, regimento interno acessíveis.

**Conselho fiscal quer** (persona **esquecida** pelo mercado):
1. **Acesso total read-only** ao financeiro.
2. **Download de extratos, balancetes, notas fiscais**.
3. **Tickets abertos de dúvida** que o síndico precisa responder.
4. **Trilha auditável** de decisões.

---

## 6. Gaps de mercado que Master Síndico pode explorar

1. **Governança institucional** — ninguém operacionaliza Cap. VII CC. Core differential.
2. **Compliance SST no produto** — PGR/NR-1 integrado; atestado visível.
3. **Connect Me unidirecional** — ninguém tem marketplace validado + unidirecional.
4. **Reputação determinística** — todos têm 5 estrelas subjetivas; ninguém tem cálculo objetivo auditável.
5. **LMS do síndico** — associações têm cursos off, nenhum concorrente embute no produto.
6. **Conselho fiscal como persona** — todos os concorrentes esquecem.
7. **Assembleia live end-to-end** — ninguém tem full-stack com gov.br + ICP-Brasil opcional + carimbo do tempo + arquivo PDF/A.
8. **Transparência por design** — contas públicas, trilha append-only, auditoria assistida.
9. **Integração gov.br** — ecossistema público BR é subexplorado.
10. **Prestação de contas automatizada** com detecção de fraude assistida.

---

## 7. Riscos competitivos

| Risco                                                      | Probabilidade | Mitigação                                                        |
|------------------------------------------------------------|---------------|------------------------------------------------------------------|
| Superlógica copia Connect Me / LMS                         | Média         | Velocidade + DNA de produto social + parcerias institucionais   |
| TownSq adiciona governança formal                          | Média         | Foco profundo em 7 sub-produtos integrados vs TownSq genérico    |
| Administradoras tradicionais (APSA, Zelo) fazem white-label| Baixa         | Elas não têm tech capacity; serão clientes ou resellers         |
| ANPD escala multas → concorrentes correm atrás             | Alta (2026+)  | LGPD-by-design desde dia 1 vira moat                            |
| NR-1 psicossocial vira obrigação                           | Confirmada    | Módulo SST preparado; primeiro a chegar no mercado              |

---

## 8. Recomendações acionáveis para Sprint 10 / M1 (2026-05-08)

### MVP crítico
1. **Governança formal** — módulo mandato síndico + convocação AGO + quórum por deliberação.
2. **Assembleia live** — com gov.br integrado (diferencial imediato).
3. **Connect Me unidirecional** — básico (3 orçamentos obrigatórios).
4. **Reputação Bronze→Diamante** — cálculo determinístico mesmo que simplificado.
5. **API-first** — preparar adapter p/ Superlógica ser plugável desde dia 1.

### Quick wins regulatórios
1. Templates PGR/NR-1 (uCondo faz em blog; Master Síndico faz no produto).
2. Checklist LGPD aplicado (RIPD básico, consentimento do morador).
3. Alertas CCT (via scraping SECOVI-SP ou feed).

### Parcerias institucionais
1. ABRASSP / ABRASCOND — validação do LMS.
2. SECOVI-SP — endosso institucional (crítico em SP, região-foco).
3. Provedor gov.br — integração técnica.
4. ICP-Brasil certificadoras (Certisign, Soluti) — opcional qualificada.

### Comunicação / marketing
1. Blog jurídico modelo uCondo (foco: NR-1 psicossocial 2026 é notícia quente).
2. Canal YouTube com conteúdo síndico profissional (preencher lacuna SindicoLab).
3. Presença em eventos SECOVI-SP e ABRASCOND.

---

## 9. Próximos passos de pesquisa

- [ ] Confirmar "Igora" e "Apusia" com stakeholder (fontes diretas?). Ver `source-04-apsa.md`.
- [ ] Mapear integração técnica profunda da API Superlógica (schema completo, rate limits, pricing de integração).
- [ ] Pesquisa focada em **Cibus** — não encontrado em pesquisa aberta.
- [ ] Benchmark de preço de mercado (pesquisa com clientes / sondagem em grupos de síndicos).
- [ ] Analisar Kiper com mais profundidade (citado em query, poucos dados).
- [ ] Condomob — análise técnica de API (parece ter).
- [ ] ANOREG BR — módulo cartório (ata notariada) — não coberto nesta rodada; recomendado spike dedicado.
- [ ] Mapa CCTs de outras capitais (RJ, BH, Curitiba, POA) para expansão pós-M1.

---

## 10. Arquivo de fontes (links rápidos)

| # | Arquivo                                    | Tópico                                  |
|---|--------------------------------------------|-----------------------------------------|
| 01 | `source-01-superlogica.md`                | Superlógica ERP                         |
| 02 | `source-02-townsq.md`                     | TownSq app                              |
| 03 | `source-03-group-software-cond21.md`      | Condomínio21 / Group                    |
| 04 | `source-04-apsa.md`                       | APSA administradora (nota sobre Apusia) |
| 05 | `source-05-ucondo-condomob.md`            | uCondo + Condomob                       |
| 06 | `source-06-administradoras-regionais.md`  | Zelo / Genesis / Thomaz                 |
| 07 | `source-07-associacoes-sindicos.md`       | ABRASCOND / ABRASSP / AABIC / ANSG      |
| 08 | `source-08-secovi-sp.md`                  | SECOVI-SP + CCT                         |
| 09 | `source-09-lgpd-condominio.md`            | LGPD aplicada a condomínio              |
| 10 | `source-10-codigo-civil-lei-condominio.md`| Código Civil arts. 1331-1358 + 4591/64 + 14.309/2022 |
| 11 | `source-11-assinatura-eletronica.md`      | MP 2.200-2 + ICP-Brasil + visão geral   |
| 12 | `source-12-nr1-pgr.md`                    | NR-1 + PGR + SST                        |
| 13 | `source-13-reclame-aqui-dores.md`         | Dores (Reclame Aqui + blogs síndico)    |
| 14 | `source-14-fraudes-casos.md`              | Fraudes em condomínio BR                |
| 15 | `source-15-api-superlogica-tecnico.md`    | API técnica Superlógica                 |
| 16 | `source-16-lei-14063.md`                  | Lei 14.063/2020 detalhada + gov.br      |
