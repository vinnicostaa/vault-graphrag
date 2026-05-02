 Master Síndico — Explicação Detalhada Ponta a Ponta

  1. O que é — a frase de um parágrafo

  Master Síndico é uma plataforma SaaS brasileira de governança condominial que conecta
   síndicos, moradores, empresas do mercado condominial, agências de marketing e
  comércio local num ecossistema unificado, resolvendo três dores estruturais do
  mercado: (1) falta de critério na contratação de prestadores, via um sistema de
  reputação auditável Bronze→Diamante calculado deterministicamente; (2) fragilidade da
   memória institucional, via timeline imutável que preserva cada decisão e ato da
  gestão; e (3) isolamento das transações, via ecossistema onde perfis, vídeos,
  contratos, cursos e contato direto vivem sob uma mesma camada de governança
  documentada e legalmente válida no Brasil.

  2. O que não é — quatro guard-rails importantes

  Master Síndico não é ERP condominial (não faz boleto, cobrança bancária, emissão
  fiscal, folha de pagamento ou contabilidade — o mercado já tem Superlógica,
  Condomínio21, Apsa pra isso, e o Master Síndico pode até integrar via adapter no M4+
  mas jamais substituir). Não é app de comunicação interna (não é WhatsApp-substituto;
  Connect Me é formulário unidirecional estruturado, nunca thread de mensagens — o que
  TownSq e Condomob fazem). Não é sistema financeiro (não intermedia pagamento entre
  síndico e empresa; só cobra a própria assinatura SaaS via Stripe). Não é plataforma
  genérica de videoconferência (assembleia live via Livekit é vertical dedicada a
  votação fracional + ata imutável, não Google Meet substituto).

  Essas quatro negativas existem porque o produto já teve tentativas internas de
  "adicionar um módulo financeiro" ou "fazer chat morador-síndico" e cada uma foi
  bloqueada em steering como não-negociável — fugiria do eixo "governança aplicada, não
   gerenciamento operacional".

  3. Invariante-chave do produto — a coisa que nunca pode mudar

  ▎ Identidade é única (1 CPF ou 1 CNPJ = 1 User). Contextos de operação são múltiplos.

  O mesmo indivíduo pode ser simultaneamente:
  - Síndico no Condomínio X (role institutional profissional ou eleito)
  - Morador no Condomínio Y (role institutional, com voto por fração ideal)
  - Representante legal da Empresa Z (role commercial, com reputação Bronze→Diamante
  acumulada)
  - Admin interno Master Síndico (role operacional da plataforma)

  Tecnicamente isso é expresso como: User é aggregate raiz em identity; Membership é
  aggregate em institutional que projeta esse User em um tenant específico com
  role+plano+quota independentes. Nenhuma duplicação de cadastro. O CPF/CNPJ é chave
  natural, imutável após primeira definição, validado no construtor da Value Object
  (CPFVO.New(), CNPJVO.New()).

  Consequência prática: quando João cria a empresa dele de portaria, não vira "conta
  nova" — sua identidade existente ganha um membership novo em commercial como
  representante da empresa. O perfil profissional dele como síndico no Condomínio X
  continua intacto. A timeline do condomínio onde ele mora registra o voto dele como
  morador, não como representante de empresa.

  Essa invariante é o que torna possível o ecossistema funcionar — sem ela, o mesmo
  indivíduo teria 3 contas e a plataforma viraria um silo fragmentado como os
  concorrentes.

  ---
  4. As 7 personas que operam na plataforma

  Persona 1 — Síndico

  O gestor eleito ou contratado do condomínio. Decisor principal das contratações. Dois
   sub-tipos conceituais: síndico morador (condômino eleito em assembleia) e síndico
  profissional (prestador externo, às vezes gerenciando múltiplos condomínios via
  memberships múltiplos — a feature "empresa-síndica" com delegação formal fica pra
  backlog M5+).

  Três níveis de plano que determinam acesso funcional e posicionamento:

  - N1 "Aprender" — iniciante em certificação. Consumo ampliado (ebooks ✓, cursos ⚠️
  limitados), Connect Me Síndico→Empresa 2 RFPs por ano, visível em busca. Não publica
  vídeos, não convoca assembleia ainda. É o ponto de entrada: quem acaba de ser eleito,
   quem está estudando pra virar profissional.
  - N2 "Atuar" — síndico operacional. Publica até 4 vídeos/mês (instrucionais +
  institucionais), abre 4 Connect Me/ano, convoca assembleia com limitações, cadastra
  plano diretor parcial, perfil visível ampliado. É o tier onde um síndico ativo
  gerenciando um condomínio vive.
  - N3 "Consolidar" — profissional estabelecido. Vídeos ilimitados, Connect Me
  ilimitado, assembleia full, dashboard de métricas, exportação PDF da timeline da
  gestão, livestream próprio. É o tier comercialmente mais valioso — síndico
  profissional que gerencia 5+ condomínios e precisa de alavancagem de marca.

  Dados obrigatórios de cadastro (bloqueiam ativação até completar): nome completo,
  email profissional, senha forte, CPF válido (algoritmo dígito verificador +
  unicidade), data de nascimento (≥18 anos), endereço profissional com CEP, telefone,
  tipo (morador ou profissional), cidade/estado de atuação, tempo de atuação em anos,
  quantidade de condomínios administrados. Dados de perfil (opcionais): foto
  profissional (5MB, 200×200), mini-bio (máx 500 chars), vídeo institucional via Mux,
  formação/certificações, administradoras vinculadas, checkboxes de credenciais (Membro
   ABRACS, Seguro de Responsabilidade Civil, Assessoria Jurídica, Assessoria Contábil,
  Assessoria em Segurança do Trabalho, Conformidade LGPD, Conformidade NR-1, Programa
  de Compliance, Selo Reclame Aqui, Outra Certificação especificar, Premiações).

  Trial: 15 dias com cartão pré-autorizado. Device policy: 1 dispositivo ativo
  (default; override só via suporte).

  Persona 2 — Morador

  O condômino participante. Vota em assembleia (peso do voto = fração ideal da unidade
  dele), acompanha timeline, consome conteúdo institucional. Dois sub-tipos comerciais:

  - Morador Base — acesso gratuito via vinculação pelo síndico. Vota ✓, vê timeline ✓,
  ata ✓, Connect Me Morador→Síndico 2/ano, vídeos síndico ilimitado, vídeos empresa
  preview 25%.
  - Morador Pagante — assinatura expande quotas: Connect Me 4/ano, vídeos empresa
  integral ✓, cursos ✓, certificados LMS ✓, vídeo-currículo 90s no Banco de Talentos,
  Vizinhança completa.

  Dados de cadastro: nome, email, senha, telefone, CPF, data nascimento (≥18), CEP
  (auto-completa endereço), condomínio vinculado (ID ou nome). Perfil opcional: foto,
  endereço completo, vídeo-currículo (MP4/WebM até 500MB, 90 segundos via Mux, lock de
  90 dias pós-upload).

  Jornada canônica documentada como M1-M13 no legado (arquivo jornada-morador.md):
  login → timeline → assembleia → votação → Connect Me abrir → ver perfil empresa →
  upload vídeo-currículo.

  Trial: não se aplica (acesso via vinculação).

  Persona 3 — Empresa do Mercado Condominial

  PJ com CNPJ ativo (validação Receita Federal M2+; stub em M1) que presta serviço ao
  condomínio: portaria, limpeza, manutenção, segurança, reforma, assessoria
  jurídica/contábil, paisagismo, desinsetização, etc. É o seller do marketplace Connect
   Me.

  Dois tiers comerciais:

  - Empresa Plus — responde Connect Me (não abre), publica 8 vídeos institucionais/mês
  (via Mux, lock 90d), visualiza currículos de moradores, recebe avaliações que
  alimentam reputação Bronze→Diamante, badge "Verificada" quando validada.
  - Empresa Pro — tudo do Plus + 12 vídeos/mês + Connect Me E→E (empresa pra empresa,
  ex: subcontratação, consórcio pra obra grande) + cupons/promoções na Vizinhança +
  dashboard completo + perfil destacado em busca.

  Dados obrigatórios: razão social, email do representante legal (único no sistema),
  senha, CNPJ válido + único, data de aniversário da empresa, nome do representante
  legal, email comercial, telefone comercial, endereço comercial completo com CEP,
  objeto financeiro ({nome, telefone, email}), categorias de serviço (seleção múltipla
  de lista controlada), estados/cidades de atuação. Perfil opcional: logo, nome
  fantasia, descrição institucional (≤1000 chars), certificações (ISO 9001, ISO 14001,
  B Corp, etc.), vídeos institucionais, portfolio (até 10 fotos/case studies).

  Verificação (M2+ pleno): CNPJ ativo via Receita Federal, situação regular PGFN/CADIN.
   Badge "Verificada" visual quando todas validações OK.

  Trial: 7 dias.

  Jornada canônica E1-E16: hub operacional, oportunidades, propostas, negócios,
  avaliações, inteligência de mercado.

  Persona 4 — Agência de Marketing

  Agência que gerencia a presença digital de múltiplas empresas condominiais.
  Tratamento técnico em MVP (M1-M3) é como actor delegado — não tem tenant próprio,
  recebe delegação via scoped token (impersonation) pra operar em nome de cada empresa
  cliente. Tenant próprio multi-empresa com dashboard consolidado fica pra M5+ quando
  demanda validar.

  Telas dedicadas agora expandidas de AM1 a AM9: Login+MFA, Dashboard multi-empresa,
  Seleção de empresa (ativação de impersonation scoped token), Audit log de ações
  delegadas (o que fez em nome de quem, quando — auditável), Revogação de delegation
  (operada pela empresa com confirmação duplo step), Billing & plan da empresa
  (read-only — agência não altera pagamento), Relatórios consolidados (export PDF/CSV),
   Marketing Link publisher (links sponsored que destacam empresas no feed), Perfil
  próprio da agência.

  Pode agir em nome da empresa para: publicar vídeo institucional, abrir Connect Me (se
   Pro), responder Connect Me, editar perfil institucional, visualizar métricas. Não
  pode: alterar plano/assinatura, adicionar/remover representante legal, aceitar
  contratos em nome da empresa (só a empresa diretamente).

  Dados obrigatórios: mesmos da Empresa + lista de empresas gerenciadas (via invite +
  aceite da empresa, registrado em audit log).

  Persona 5 — Parceiro da Vizinhança

  Comércio local (pizzaria, petshop, farmácia, academia, padaria, restaurante,
  papelaria). Diferenciação crítica: não presta serviço ao condomínio — vende ao
  morador como consumidor final. Isso é uma distinção importante de produto: empresa
  condominial entra em Connect Me formal; parceiro vizinhança vive na feature
  Vizinhança com dinâmica de hyperlocal.

  Permissões: perfil público no mapa Vizinhança (visível a moradores em raio geográfico
   configurável, default 2km), cupom/promoção publicada com lock de 4 horas (o morador
  tem janela de 4h pra apresentar/redimir; depois expira — evita promoção perpétua),
  Palavra-chave do Dia no tier Pro (gamification: morador redime cupom especial
  digitando palavra), mini-seal "Síndico-aprovado" concedido via votação informal dos
  síndicos N2+ da vizinhança (agrega curadoria).

  Dados: pode ser CPF (MEI) ou CNPJ (PJ), data de aniversário (pessoa ou empresa),
  representante legal, financeiro, endereço com CEP. Perfil: logo, nome fantasia, até 3
   fotos.

  Trial: 30 dias (o mais longo — porque o modelo hyperlocal precisa de tempo pra
  validar demanda no raio da empresa).

  Persona 6 — Admin Master Síndico

  Time interno da plataforma. Não passa por fluxo de cadastro público. Sub-tipos:
  - Super Admin — permissão total, admin_bypass em ABAC (hierarquia 1 da matriz). MFA
  obrigatório M1, IP allowlist M3+.
  - Moderador — aprova/rejeita vídeos institucionais na janela de moderação manual 48h,
   resolve harassment reports.
  - Suporte — vê dados de tenant para resolver ticket (cada visualização de PII
  registrada em lgpd_logs append-only).
  - Financeiro — reconciliação Stripe, refunds, disputas.

  Permissões cobrem: gerenciar tenants (suspender/reativar), moderar conteúdo
  cross-tenant, broadcast global de comunicado de plataforma, painel de métricas
  agregadas, audit logs leitura, refunds e disputas Stripe.

  Device policy: 1 dispositivo + MFA obrigatório M1 (reclassificado pelo dual-check;
  era M2 no plano original) + step-up MFA em 8 endpoints críticos (DELETE /me, publish
  minutes, approve >R$10k, moderation actions, impersonation start, bulk operations,
  config changes, user role modification).

  Persona 7 — Sub-roles operacionais (derivadas, não personas próprias)

  Conceituais, implementadas como scoped permissions em ABAC, não como cadastro
  separado:

  - Subsíndico — delegado do síndico titular; maioria das permissões do síndico exceto
  encerrar mandato próprio e designar novo síndico. Decisão T8 do legado.
  - Conselho (fiscal/consultivo) — colegiado que aprova contratos acima de threshold
  configurável; participa de SupplierVoteSession quando Connect Me fecha contrato
  relevante (votação colegiada rastreável).
  - Morador dependente — titular de unidade registra dependentes (filhos menores,
  cônjuge sem membership próprio); acesso restrito (visualização, sem votação).
  - Representante secundário de empresa — adicional ao representante legal; feature
  flag M3+ (hoje 1 representante ativo por empresa).

  ---
  5. Os 7 sub-produtos integrados — detalhamento completo

  Sub-produto 1 — Governança Institucional (M1, núcleo)

  Propósito: o núcleo do sistema, sem o qual nada mais faz sentido. Cadastro do
  Condomínio (tenant primário), das Unidades com fração ideal distribuída, dos
  Memberships (User ↔ Condomínio ↔ Unit + Role), da Timeline imutável (INSERT-only, sem
   UPDATE, sem DELETE, sem deleted_at), do Plano Diretor plurianual (3-5 anos, com
  template + acompanhamento de execução em percentual), de Comunicados Oficiais
  (imutáveis pós-publish), Eventos calendarizáveis, Enquetes internas (diferentes de
  assembleia formal — levantamento de opinião leve).

  Bounded context: institutional (principal), depende de identity pra
  User/Session/ABAC.

  Invariantes críticas:
  - INV-021: Σ fração ideal das unidades ≤ 100% (enforçado via CHECK constraint em
  Postgres com NUMERIC(10,4) + validação PBT em condomínios 1000+ unidades; descoberto
  gap no float64 Go, migrou pra decimal.Decimal)
  - INV-022: Timeline INSERT-only (nenhuma query de UPDATE/DELETE permitida; enforçado
  via lint custom em sqlc + grant de role DB sem UPDATE/DELETE em timeline_entries)
  - INV-023: Membership ativa única por (user, condominium) em dado momento (UNIQUE
  constraint parcial WHERE active=true)
  - INV-024: Announcement imutável pós-publish (transição draft → published é terminal;
   correção vira nova entry na timeline)

  Personas-alvo: Síndico (criador/editor/decisor), Subsíndico/Conselho (editores
  delegados), Morador (consumidor: timeline, ata, comunicados, vota enquetes).

  Diferencial competitivo: timeline como prova documental legal. ERPs concorrentes
  (Superlógica, Igora/APSA, Condomínio21) armazenam registros fragmentados em silos
  financeiros; Master Síndico unifica "decisão-acontecimento-documento" numa sequência
  imutável juridicamente auditável. Em eventual contestação judicial (síndico acusado
  de má gestão, morador contestando cobrança extraordinária), a timeline é evidência
  forense. Diferencial reforçado pelo research condominial BR que mapeou fraudes comuns
   (orçamentos falsos, atas manipuladas retroativamente, prestação de contas vazando
  por WhatsApp).

  Marco de entrega: M1 (2026-05-08) — base contratual. Sem Governança funcionando, M1
  não é M1.

  ---
  Sub-produto 2 — Connect Me (Marketplace de Contratação Formal) (M2)

  Propósito: resolver a dor #1 (critério na contratação) operacionalmente. É um
  marketplace unidirecional, não chat. Síndico tem problema → abre RFP (Request for
  Proposal) estruturado → empresas qualificadas na categoria recebem notificação →
  empresas enviam proposta formal → síndico escolhe → contrato com milestones →
  execução → avaliação pós-contrato → reputação recalculada.

  Quatro fluxos:

  1. Síndico → Empresa — o fluxo principal. Síndico abre RFP descrevendo (ex: "portaria
   24h, 3 postos, orçamento teto R$ 30k/mês, prazo contrato 12 meses, início em 30
  dias"). Empresas qualificadas na categoria "portaria" + região que atendem recebem.
  Quota: N1 2/ano, N2 4/ano, N3 ∞.
  2. Morador → Síndico — morador solicita algo formal ao síndico (reforma na própria
  unidade precisando autorização, denúncia estruturada com evidência, manifestação de
  interesse pra virar pauta em assembleia). Quota: Base 2/ano, Pagante 4/ano.
  3. Empresa → Empresa (só Plus/Pro abrem, Plus responde) — networking B2B entre
  empresas condominiais. Subcontratação (empresa grande precisando terceirizar parte do
   serviço), consórcio pra obra grande, referência cruzada. Plus responde apenas; Pro
  abre com quota limitada.
  4. Síndico → Parceiro (Vizinhança) — síndico convida comércio local pra
  parceria/benefício ("queremos parceria com academia local pra dar desconto a
  moradores"). Vira acordo leve, não contrato formal.

  Regras centrais inegociáveis:

  - Formulário estruturado, não texto livre: campos obrigatórios categoria, descrição
  anexos (via Mux/S3 presigned).
  - Cotas anuais controladas — decrementa no ato de "publicar RFP", não de "criar
  rascunho". Permite síndico esboçar sem consumir quota.
  - Sem thread de mensagens. Se empresa tem dúvida: envia proposta com a dúvida
  marcada; síndico recebe; se aceita a linha, avança. Reply unidirecional: proposta da
  empresa é final (revisões explícitas limitadas a 2, não chat).
  - Auditabilidade completa: RFP + todas as propostas recebidas + escolha + contrato +
  execução + avaliação → tudo registrado imutável na timeline institucional do
  condomínio.
  - Proposta única por (ConnectMeRequest, Company): UNIQUE DB (INV-043 fechado pelo
  A-DC-PEN-012 com DTO estrito em INS-007 — o id do request e da empresa vêm de
  path+context, nunca de body).

  Bounded context: commercial (principal), cross-domain com institutional (emite evento
   ContractClosed que gera TimelineEntry) e content (opcionalmente anexa vídeos
  institucionais da empresa à proposta).

  Diferencial: unidirecional evita "ruído de WhatsApp" + decisão com trilha clara +
  auditabilidade total (pentest-evidence em disputa judicial). ERPs registram contrato
  firmado; Master Síndico registra processo completo de escolha — se amanhã surge
  dúvida sobre por que a empresa X foi escolhida e não a Y, a plataforma tem as 5
  propostas que chegaram com os valores de cada uma. Em mercado onde fraudes comuns
  incluem "3 orçamentos forjados com CNPJs ligados", o formalismo cura essa dor.

  ---
  Sub-produto 3 — Reputação Bronze → Diamante (M2, determinística)

  Propósito: qualificar empresas (e em futuro, síndicos profissionais) num tier público
   automatizado baseado em métricas auditáveis. Zero black box, zero ML opaco, zero
  "pagou pra subir". Cinco tiers com thresholds exatos:

  ┌──────────┬──────────────────────────────────────────────────────────────────────┐
  │   Tier   │                     Threshold mínimo cumulativo                      │
  ├──────────┼──────────────────────────────────────────────────────────────────────┤
  │ Bronze   │ Default ao criar empresa (cadastro completo + CNPJ verificado)       │
  ├──────────┼──────────────────────────────────────────────────────────────────────┤
  │ Prata    │ ≥ 3 contratos fechados + avaliação média ≥ 3,0/5                     │
  ├──────────┼──────────────────────────────────────────────────────────────────────┤
  │ Ouro     │ ≥ 10 contratos + avaliação média ≥ 3,5/5 + ≥ 3 condomínios distintos │
  │          │  atendidos                                                           │
  ├──────────┼──────────────────────────────────────────────────────────────────────┤
  │ Platina  │ ≥ 30 contratos + avaliação média ≥ 4,0/5 + ≥ 10 condomínios          │
  │          │ distintos + ≥ 1 vídeo institucional aprovado + ≥ 1 case em vídeo     │
  ├──────────┼──────────────────────────────────────────────────────────────────────┤
  │          │ ≥ 100 contratos + avaliação média ≥ 4,5/5 + ≥ 30 condomínios         │
  │ Diamante │ distintos + ≥ 3 vídeos institucionais + múltiplos cases + tempo de   │
  │          │ plataforma ≥ 12 meses                                                │
  └──────────┴──────────────────────────────────────────────────────────────────────┘

  Fórmula parametrizada (D-051, dual-check): pesos 40/25/15/10/10 em (contratos
  fechados / avaliação média / condomínios distintos / vídeos institucionais / casos
  vídeo).

  Regras invioláveis:

  diretamente. Nenhum ML. Nenhum "boost promocional".
  - Recálculo em eventos: novo ContractClosed → CompanyEvaluationSubmitted →
  VideoApproved → handler subscreve e chama RecalculateReputation(companyID); aggregate
   atualiza tier se cruza threshold (up ou down).
  - Degradação permitida: se avaliação média cai após denúncia confirmada + contratos
  cancelados, tier pode descer. Mesma função, mesmos inputs.
  - Suspensão temporária por harassment confirmado: flag suspended=true sobre o tier
  atual por 72h enquanto moderação manual avalia. NÃO é despromoção (o tier permanece
  no estado); é marcação.
  - Gap-SEC-008 + 010 (identificados em Fase 5): decay temporal (empresa mantém tier
  alto sem fechar contratos novos) e farm detection (anéis de condomínios-síndicos
  fantasmas inflando reputação) são ajustes previstos M3+.

  Bounded context: commercial (contém Company + CompanyEvaluation + reputation_events),
   cross-domain com content (vídeos institucionais aprovados) e institutional
  (condomínios distintos verificados via histórico de memberships).

  Diferencial: zero opacidade. Síndico clica no badge Platina de uma empresa e vê as
  métricas que geram o tier (45 contratos, 4,2 de média em 38 avaliações, 18
  condomínios diferentes, 2 vídeos institucionais, 1 case). Compare com "estrelas"
  genéricas de marketplaces onde você nunca sabe se foi review pago. Anti-fraude
  estrutural: impossível comprar tier porque os inputs são observáveis e auditáveis —
  contratos reais registrados no ClosedDeal aggregate, avaliações de síndicos reais com
   membership válido, vídeos passaram por moderação manual.

  ---
  Sub-produto 4 — Conteúdo & Vídeos (M2, Mux-powered)

  Propósito: infraestrutura de upload, moderação, publicação, busca e distribuição de
  vídeos. Dois fluxos distintos sobrepondo mesma infra:

  a) Vídeo Institucional (empresa Plus/Pro, síndico N2+):
  - Upload direto pro Mux via presigned URL (backend nunca proxia bytes — regra dura
  GR-019 pra performance + segurança).
  - Mux dispara webhook video.asset.ready → VideoModeration aggregate entra em estado
  pending.
  - Admin Moderador aprova/rejeita em janela de 48 horas.
  - Aprovação → VideoPublished event → publicação + lock de 90 dias (o publisher não
  pode apagar/substituir nos 90 dias seguintes).
  - Indexação OpenSearch (categoria, tag, texto descrição) pra busca.
  - Playback via signed URL Mux que expira (anti-hotlinking).

  b) Vídeo-Currículo 90s (morador, Banco de Talentos):
  - Mesmo fluxo técnico, mas limitado a 90 segundos e categorizado como talent_resume.
  - Lock 90 dias pós-upload.
  - Empresa Plus/Pro busca candidatos por skill/categoria/CEP/raio e convida via
  Connect Me Morador→Síndico fluxo especial.

  Regras invioláveis:

  - Lock de 90 dias — motivador: churn prevention + compromisso editorial.
   na busca. Ou se compromete, ou não publica. Bypass só com 4-eyes policy (admin
  solicita + outro admin aprova + motivo obrigatório + audit log — D-054).
  - Moderação antes de visibilidade pública — upload → VideoModeration.pending → admin
  → publish. Se reprovado: motivo obrigatório, publisher notificado, pode corrigir e
  re-submeter.
  - Privacidade de métricas cross-tenant — views/likes de vídeo da empresa X não vazam
  pra empresa Y (ABAC strict; tenant isolation enforçada).
  - Cascade de moderação futuro: M1 manual 48h → M2 Rekognition (AWS) + OpenAI
  Moderation (transcript, grátis) + manual fallback → M3 ML próprio (se volume
  justificar — gatilho ≥ 300 min/mês). Decision ADR-0024.
  - Quotas mensais por tier: N2 síndico 4/mês, N3 ilimitado, Empresa Plus 8/mês, Pro
  12/mês. Reset no dia 1 do mês UTC.

  Bounded context: content (principal). Cross-domain com commercial (vídeo
  institucional empresa alimenta reputação Platina+) e assembly (gravação de assembleia
   via Livekit Egress Room Composite integra com content pra arquivamento com lock
  90d).

  Stack: Mux (upload presigned + HLS transcoding + signed playback + webhooks
  asset.ready/errored — confirmado no consolidado providers 108K linhas), AWS S3
  (thumbnails + storyboard), OpenSearch (busca textual + filtragem), FCM/SES
  (notificação quando moderação conclui).

  Diferencial: lock 90d é único no mercado — nenhum concorrente faz isso. Vem de lição
  aprendida: outros marketplaces (Instagram, LinkedIn) priorizam freshness; condomínio
  prioriza compromisso documentado. Moderação editorial + categorização estruturada
  alinha com Connect Me (vídeos organizados por categoria de serviço = matching com
  RFPs).

  ---
  Sub-produto 5 — Assembleias Live & Votações (M3, Livekit)

  Propósito: o sub-produto mais ambicioso tecnicamente e o de maior valor legal.
  Automatiza o fluxo completo da assembleia condominial juridicamente válida no Brasil.

  Fluxo ponta a ponta:

  1. Convocação com edital PDF — síndico cria Assembly draft, preenche pauta
  (AgendaItem por tópico, com tipo de decisão: deliberativa/consultiva/informativa,
  proxy permitido sim/não), sistema gera edital em PDF padronizado (nome condomínio,
  data/hora/local/link live, pauta detalhada, orientações de quórum e proxy, assinado
  digitalmente — Lei 14.063/2020, nível "avançado" ou "qualificado" via gov.br).
  2. Publicação do edital — AssemblyScheduled event → notificações push + email pros
  moradores com antecedência regulamentar (30 dias para AGO, 10 dias para AGE).
  Assembly não inicia sem edital publicado (INV-061, A-013 legado).
  3. Registro de procuração antes da sessão — morador outorga voto a outro morador via
  Proxy aggregate. Específica (um item da pauta) ou geral (toda a assembleia).
  Rastreável, auditável, revocable até o início da sessão.
  4. Abertura de sessão live — Livekit SFU (WebRTC) — D-009 ADR-0011. Participantes
  entram na sala via token gerado no backend (não self-service Livekit). Validação
  membership ativo no condomínio.
  5. Controles de sessão:
    - Fila de fala cronometrada — moradores levantam mão; síndico concede palavra;
  cronômetro server-side (attributes Livekit + UpdateParticipant com revogação de
  canPublish).
    - Votação em tempo real por fração ideal — morador vota "sim/não/abstenção" em
  AgendaItem; peso do voto = unit.fracao_ideal da unidade dele. Quórum é calculado em
  decimais (ex: maioria simples = 50% + 1 fração das frações presentes via proxy ou
  in-person; alteração de convenção = 2/3 das frações conforme Código Civil;
  destituição de síndico = 1/2 + 1). Vote único por (agenda_item, voter) — UNIQUE DB +
  ErrConflict no handler (A-025 do legado já resolvido).
    - Presença auditada — AttendanceRecord com timestamp de entrada/saída de cada
  participante, identificação via Membership.
  6. Geração automática de ata imutável — pós-sessão, sistema monta Minutes aggregate
  mapeando pauta → resultado (aprovado/rejeitado/adiado) + quórum calculado +
  timestamps + lista de presença + votos agregados. Síndico revisa; se há correção
  necessária, vira nova entry na timeline (não edita a ata). Publicação → Minutes
  imutável pós-publish (INV-041).
  7. Assinatura digital BR — ata assinada via gov.br (nível avançado ou qualificado
  conforme Lei 14.063/2020 — confirmed no research condominial BR). Hash da ata gravado
   em timeline_entries com nonce pra defesa contra tampering.
  8. Modalidade online/híbrida: MVP M3 prioriza presencial + live streaming opcional da
   sessão. Online pleno (voto remoto + presença remota validada) é backlog M4+ conforme
   Decision 1 do legado.

  Bounded context: assembly (principal). Cross-domain: gera TimelineEntry em
  institutional pós-ata; eventos de voto podem afetar commercial (SupplierVoteSession
  sob assembleia pra contratos acima de threshold — discussão pendente P-CM-005 sobre
  BC ownership).

  Stack: Livekit Cloud (SFU WebRTC gerenciado — D-010 ADR-0011), Livekit Egress Room
  Composite pra gravação em MP4 → S3 tenant-isolated (não Mux — não precisa de CDN
  porque a gravação é arquivo legal, não distribuição de conteúdo), PDF generation lib
  (edital + ata), integração gov.br pra assinatura digital.

  Diferencial: ata gerada automaticamente + assinatura gov.br + quórum fracional
  nativo. Outros sistemas (TownSq, uCondo) fazem comunicado de assembleia + enquete;
  nenhum faz o ciclo completo com valor legal. Eliminando a dependência de secretário
  humano digitando ata à mão (fonte comum de disputa retroativa) é valor direto.

  ---
  Sub-produto 6 — LMS (Academia) + Banco de Talentos (M3 thin, M4+ pleno)

  Duas ofertas distintas sobrepostas tecnicamente à mesma infra de content:

  6a) LMS Master Síndico Academia:
  - Trilhas de capacitação em 5 áreas (Gestão, Direito Condominial, Finanças Básicas,
  Comunicação e Conflitos, Aspectos Técnicos de Manutenção).
  - 3 trilhas piloto M3: Síndico Iniciante, Síndico Intermediário, Síndico Profissional
   (mapeadas a N1/N2/N3 do plano).
  - Recursos por trilha: videoaulas via Mux (mesma infra de Conteúdo), ebooks em PDF
  (S3), quizzes com questões múltipla escolha (aprovação ≥ 70%), certificados gerados
  em PDF com assinatura digital verificável via URL pública.
  - M4+ pleno: parceria ABRACS pra homologação oficial das trilhas (feature flag;
  depende de comercial).

  6b) Banco de Talentos:
  - Vídeo-currículo morador 90s (infra de content, lock 90d).
  - Perfil estendido com habilidades (skill taxonomy curada; inspiração LinkedIn
  destilado, mas sem endorsements — anti-fraude), disponibilidade, pretensão salarial
  opcional.
  - Empresa Plus/Pro busca candidatos filtrando por categoria/skill/CEP/raio.
  - Convite via Connect Me (fluxo especial Empresa→Morador "oportunidade de trabalho")
  — não rompe unidirecionalidade, formalizado como "oferta estruturada".

  Bounded context: content (LMS usa infra de courses/enrollments/quizzes/certificates)
  + commercial (Banco de Talentos: discovery + match empresa↔morador).

  Diferencial: capacitação contextualizada ao mercado BR (direito condominial real,
  conflitos típicos, compliance NR-1 aplicada a prestadores) + certificados
  verificáveis linkáveis no Connect Me (síndico que fez trilha "Intermediário" tem
  badge no perfil que aumenta confiança de quem abre RFP pra ele, se ele também for
  prestador via empresa delegada). Banco de Talentos atende segmento sub-atendido —
  morador como candidato (não executivo, não especialista tech; doméstica, porteiro,
  técnico de manutenção procurando trabalho nos condomínios da região).

  ---
  Sub-produto 7 — Vizinhança (M2)

  Propósito: conectar comércio local a moradores dos condomínios da região. Diferente
  de Connect Me (que é formal, empresa condominial ↔ condomínio), Vizinhança é
  hyperlocal consumidor-final.

  Fluxo:
  - Parceiro se cadastra definindo endereço + raio de atendimento (default 2km).
  - Sistema indexa geograficamente (PostGIS opcional, ou lat/long + raio simples em
  PostgreSQL).
  - Morador na app vê mapa da vizinhança com parceiros dentro do raio.
  - Parceiro publica cupom/promoção ("20% off pizza hoje" — lock 4h; após expira).
  - Morador "salva" cupom e apresenta no estabelecimento físico (QR code ou código
  alfanumérico).
  - Palavra-chave do dia (gamification Pro) — morador redime cupom especial digitando
  palavra secreta do dia.
  - Síndico N2+ pode conceder mini-seal "Síndico-aprovado" via votação leve (não
  supplier_vote formal do Connect Me) — agrega curadoria social.

  Regras:
  - Cupom lock 4h rigoroso (anti-spam + urgência).
  - Seal síndico-aprovado é opinião do síndico local, rastreável (quem concedeu,
  quando), revogável.
  - Parceiro NÃO recebe Connect Me aberto (pra não confundir com empresa condominial).

  Bounded context: commercial cross-domain com institutional (seal do síndico afeta
  visibilidade + é registrado na timeline do condomínio como "parceria local").

  Diferencial: hyperlocal condominial com curadoria de síndico. Mercado atual tem
  Groupon/Peixe-Urbano decadente + iFood sem curadoria + nada específico pra comércio
  bairro ↔ condômino vizinho. Master Síndico se posiciona como canal de confiança:
  parceiro chega ao morador via síndico que aprovou.

  ---
  6. Como tudo se conecta — o grafo de dependências

  Olhando o produto inteiro de cima:

  Governança Institucional (M1, obrigatório)
           │
           ├─> Billing (transversal, habilita quotas)
           │
           ├──> Connect Me (M2) ───────┐
           │        │                   │
           │        ├─> Reputação ←─────┤ (feed de avaliações)
           │        │        │          │
           │        │        └─> Conteúdo/Vídeos (M2) ← vídeos institucionais
           │        │                   │
           │        └─> Banco Talentos (M3) ← vídeo-currículo 90s
           │                            │
           ├──> Conteúdo (M2) ──────────┘
           │        │
           │        └─> LMS (M3 thin) ← mesma infra Mux
           │
           ├──> Assembleia Live (M3) ─→ Minutes ─→ Timeline
           │
           └──> Vizinhança (M2) ← seal de síndico institucional

  Regras de acoplamento:

  - Governança Institucional é pré-requisito absoluto. Sem condomínio criado +
  - Billing é habilitador transversal (não é "sub-produto" por si só). Cada sub-produto
   tem feature flags por plano validadas pelo FeatureUsage aggregate em billing.
  - Connect Me + Reputação são acoplados bidirecional: toda avaliação pós-contrato em
  Connect Me alimenta a fórmula de reputação; reputação filtra/ordena empresas em
  Connect Me.
  - Conteúdo é transversal: perfil de empresa precisa de vídeo institucional aprovado
  pra subir a Platina+; Connect Me mostra vídeos anexados; LMS usa mesma infra Mux;
  Banco de Talentos usa mesma infra Mux.
  - Assembleias alimenta Governança (ata vira timeline entry) e pode disparar
  supplier_vote que feed Connect Me (contrato formalizado por assembleia).
  - Vizinhança é semi-isolado; integra via seal do síndico (metadado na timeline
  institucional).

  Comunicação inter-BC: estritamente por domain events (nenhum BC importa domain/ ou
  application/ de outro BC). 82 eventos catalogados (E-001..E-082 em domain-events.md).
   Ex: ContractClosed (publisher commercial) → subscribers: institutional (timeline
  entry), commercial internal (reputação recálculo), content (se contrato referencia
  vídeo).

  ---
  7. Monetização e planos — o modelo de receita

  SaaS recorrente mensal/anual via Stripe. Abstraído atrás de IPaymentGateway pra
  permitir Pagar.me/Mercado Pago/PIX nativo futuro. Cinco fluxos de receita:

  1. Síndico paga N1/N2/N3 (ranges aprox: R$ 30-50 / R$ 80-150 / R$ 200-400 mensais —
  valores comerciais variáveis, não fixados em spec canônica).
  2. Empresa paga Plus/Pro (R$ 150-250 / R$ 400-800).
  3. Parceiro Vizinhança paga único tier (R$ 50-100).
  4. Morador pagante upgrade (R$ 10-25).
  5. Morador base gratuito — acesso via vinculação pelo síndico; upside de conversão
  pra pagante ou pra síndico futuro.
  6. Admin interno — sem cobrança.
  7. Agência — compartilha plano da empresa cliente (M5+ potencial add-on próprio).

  Trial persona-aware:
  - Síndico 15 dias (cartão pré-autorizado obrigatório)
  - Empresa 7 dias
  - Parceiro 30 dias
  - Morador — sem trial

  Quotas com reset periódico:
  - Vídeos: mensal (dia 1 UTC) — com D-052 storage UTC + apresentação America/Sao_Paulo
  - Connect Me: anual (dia 1 do ano UTC)
  - Cupom Vizinhança: não reseta; expira em 4h cada
  - Palavra-chave do dia: diária
  - Ver vídeos empresa integral: sem reset (enquanto subscription ativa)

  Invariante billing:
  - Money em centavos BRL (int64, nunca float — MoneyVO)
  - Webhook Stripe idempotente: UNIQUE (provider, event_id) em WebhookEvent aggregate +
   Redis L1 cache — ADR-0027 pós-fix Fase 5
  - Trial único por (user, plano) — DB enforce
  - Cancelamento sempre com grace period — nunca revoga imediato (exceto fraude
  confirmada)
  - PII em logs billing proibido — Masked() obrigatório

  Mecânica de upsell/cross-sell:
  - Síndico N1 → N2/N3 trigger: "atingiu quota Connect Me" → CTA inline
  - Morador Base → Pagante trigger: tenta ver vídeo empresa integral → paywall
  - Empresa Plus → Pro trigger: "quer abrir E→E" ou "publicar cupom Vizinhança"
  - Síndico → Empresa: caso real — síndico profissional que vira responsável legal de
  administradora
  - Morador → Síndico: morador eleito síndico na próxima assembleia migra conta
  preservando histórico

  ---
  8. Roadmap de entrega por marco

  ┌───────┬────────────────────┬───────────────────────────────────────────────────┐
  │ Marco │     Data firme     │                 Escopo contratual                 │
  ├───────┼────────────────────┼───────────────────────────────────────────────────┤
  │       │ 2026-05-08         │ Governança Institucional base (condomínio +       │
  │ M1    │ (considerar        │ unidades + membership + timeline) + Identity      │
  │       │ 2026-05-18 após    │ (Zitadel OIDC+PKCE + MFA admin) + Billing (Stripe │
  │       │ Fase 5)            │  + trial persona-aware + quotas)                  │
  ├───────┼────────────────────┼───────────────────────────────────────────────────┤
  │       │                    │ Connect Me 4 fluxos + Reputação Bronze→Diamante + │
  │ M2    │ 2026-06-20        │ Conteúdo/Vídeos (Mux + lock 90d + moderação        │
  │       │                   │ manual) + Vizinhança (mapa + cupons + seal)        │
  ├───────┼───────────────────┼────────────────────────────────────────────────────┤
  │       │                   │ Assembleia Live (Livekit + votação quórum          │
  │ M3    │ 2026-07-20        │ fracional + ata imutável + proxy) + LMS thin (5    │
  │       │                   │ áreas, 3 trilhas) + Banco de Talentos + polish     │
  │       │                   │ operacional (a11y WCAG 2.1 AA + SLO 99.5%)         │
  ├───────┼───────────────────┼────────────────────────────────────────────────────┤
  │       │                   │ Validação Receita Federal pleno, NR-1 psicossocial │
  │ M4+   │ Q3-Q4 2026        │  módulo, moderação ML, breach notification 72h     │
  │       │                   │ automática, empresa-síndica, agência tenant        │
  │       │                   │ próprio                                            │
  ├───────┼───────────────────┼────────────────────────────────────────────────────┤
  │       │                   │ Vizinhança geográfica avançada, assembleia híbrida │
  │ M5+   │ 2027              │  online, marketplace financeiro (split pagamento   │
  │       │                   │ se regulatório permitir), integração ERP           │
  │       │                   │ (Superlógica/Igora)                                │
  └───────┴───────────────────┴────────────────────────────────────────────────────┘

  A Fase 5 do dual-check security sugeriu postergar M1 em ~10 dias úteis (2026-05-18)
  pra fechar 14 🔴 técnicos (já fechados agora pelos fixers) + 7 bloqueadores LGPD
  humanos (DPO, DPAs, DPIA, política versionada) — sua decisão pendente na Fase 6.

  ---
  9. O que diferencia Master Síndico no mercado brasileiro

  Research condominial BR mapeou 6 concorrentes principais:

  - Superlógica — ERP líder BR, 5 tiers, API OAuth2, 88,8% Reclame Aqui. Foco
  financeiro/contábil.
  - TownSq — app comunidade (32k condomínios BR). Foco comunicação.
  - Condomínio21 / Group Software — ERP tradicional legacy.
  - APSA — administradora física 89 anos (100k imóveis). Foco operacional humano.
  - uCondo / Condomob — SaaS modernos (47k / 6k lares). Mix ERP+app leve.
  - Zelo / Genesis / Thomaz Multi — administradoras regionais.

  Diferenciais defensáveis identificados pelo research (mapeados como gaps que Master
  Síndico preenche):

  1. Governança que operacionaliza Cap. VII do Código Civil (arts. 1.331-1.358) —
  nenhum concorrente transforma as obrigações legais do Código em fluxo de produto.
  TownSq comunica, Superlógica cobra, ninguém governa.
  2. Connect Me unidirecional — todos os marketplaces BR tendem a virar chat (Meta
  Marketplace, OLX B2B). A unidirecionalidade estrita do Connect Me é anti-padrão
  contra-intuitivo que protege contra fraudes conhecidas documentadas no research (3
  orçamentos obrigatórios forjados com CNPJs ligados).
  3. Reputação determinística — mercado tem "4 estrelas genéricas" que não explicam
  nada. Bronze→Diamante explicável combate a sensação comum "empresa com 5 estrelas no
  app mas que no condomínio do vizinho foi péssima".
  4. Assembleia live gov.br-native — nenhum concorrente tem o stack legal completo (MP
  2.200-2/2001 + Lei 14.063/2020 + Lei 14.309/2022 consolidados). Competitivo tem
  enquete + acta-texto; Master Síndico tem fluxo completo juridicamente válido.
  5. Módulo NR-1 psicossocial (a partir 2026-05-25) — janela de oportunidade
  regulatória imediata. Nenhum concorrente tem módulo SST estruturado pra prestadores
  condominiais (portaria, limpeza) cumprirem PGR.
  6. LMS embutido — uCondo tem conteúdo, ninguém tem trilha estruturada com
  certificação verificável.
  7. Conselho fiscal como persona — gap total do mercado (todos tratam como "morador
  com flag"). Master Síndico modela como sub-role ABAC distinta com SupplierVoteSession
   próprio.

  ---
  Isso é o que é o Master Síndico em detalhe ponta a ponta. A v2 em master-sindico-v2/
  tem 752 arquivos destilados, 3,4 MB de produção documental, que expandem cada
  parágrafo acima em invariantes, state machines, aggregates Go, fluxos de integração
  com providers, ADRs justificando cada decisão e runbooks operacionais. Se quiser eu
  expandir alguma parte específica — um sub-produto, uma persona, um BC, uma integração
   provider, as regras de compliance LGPD — tenho fonte primária pra cada tópico.
