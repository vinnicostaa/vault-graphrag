---
title: Pre-Assembly — UI Spec
type: ui-spec
tags: [ui-catalog, assembly, master-sindico, frontend, pre-assembly]
bc: assembly
source: _chaos/ARQUITETURA-ASSEMBLEIA.pdf §3-4 + DIAGRAMA-VISUAL.pdf §1-2 + MÓDULO-TELAS.pdf TELA 1-8
absorbed_in: 2026-04-25 — Fase E
status: absorbed
---

# Pre-Assembly — Configuração e Preparação

> **Origem**: 3 PDFs Master Síndico (`ARQUITETURA FUNCIONAL`, `DIAGRAMA VISUAL`, `MÓDULO DAS TELAS`) — destilados em Fase E (2026-04-25).
> **Decisões aplicáveis**: D-127 (AssemblyAdministratorLink entity), D-130 (Vote entity), D-132 (Auditor role efêmera), D-133 (Assembly = conferência Livekit), ASM-010 (procuração 100% digital).

> Sprint 2-3 · App `assembly` (porta 3004) · Rotas: `/assembleias/nova`, `/assembleias/:id/configurar`, `/assembleias/:id/monitoramento`
> Perfis: Síndico (criação), Administradora (validações + uploads operacionais), Morador (ciência + procurações + enquetes).

---

## Fluxo macro pré-assembleia

```
PUBLICAÇÃO ───┬─→ NOTIFICAÇÃO MORADORES ─→ CIÊNCIA ─→ CONFIRMAÇÃO PRESENÇA
              ├─→ VISUALIZAÇÃO PAUTA
              ├─→ ENQUETES PRELIMINARES (24h antes)
              ├─→ CADASTRO PROCURAÇÃO (100% digital — ASM-010)
              └─→ ANÁLISE FORNECEDORES
              
PAINEL ADMINISTRADORA ──┬─→ IMPORTAR FRAÇÃO IDEAL (planilha)
                        ├─→ REGISTRAR INADIMPLÊNCIA (T-1h cutoff)
                        └─→ VALIDAR PROCURAÇÕES (Lei 14.063)
                        
SIMULADOR DE QUÓRUM (síndico/administradora — projeção em tempo real)
```

---

## Tela 1 — Wizard de criação (Step 1 Dados Gerais)

**Rota**: `/assembleias/nova` (Síndico)

```
┌──────────────────────────────────────────────────────┐
│  CRIAR ASSEMBLEIA — Step 1/4 Dados Gerais           │
│                                                      │
│  Tipo *               [Ordinária        ▼]          │
│                       (ordinaria/extraordinaria/     │
│                        implantacao — D-_chaos)       │
│                                                      │
│  Modalidade *         [Presencial       ▼]          │
│                       ⚠️ Híbrida/Online: M4+ futuro  │
│                                                      │
│  Data *               [__/__/____ 📅]               │
│  Hora 1ª convocação * [__:__ 🕐]                   │
│  Hora 2ª convocação * [__:__ 🕐]                   │
│                                                      │
│  Local físico         [_______________]             │
│  Link transmissão     [_______________] (M4+)       │
│                                                      │
│  Tempo máximo de fala *  ( ) 2 min  (●) 3 min      │
│  Permite prorrogação  [✓] +2 min (1 vez)           │
│                                                      │
│  Descrição geral                                     │
│  [_______________________________________]          │
│  [_______________________________________]          │
│                                                      │
│  ⚠️ Mín 8 dias corridos de antecedência (Lei         │
│  4.591/64 Art. 24 §1) — INV-118                     │
│                                                      │
│  [← Cancelar]            [Salvar]  [Próximo →]     │
└──────────────────────────────────────────────────────┘
```

### Regras

- Tipo enum: `ordinaria | extraordinaria | implantacao` (R-_chaos canônico — D-_chaos absorção Fase C).
- Modalidade: M3 forçada em `presencial`; campos `link_transmissao`, `participantes_online`, `controle_audio_remoto` preparados mas desabilitados.
- 2ª convocação obrigatoriamente posterior à 1ª.
- Data ≥ NOW() + 8 dias corridos (hard, sem override admin).
- Tempo de fala 2 ou 3 min; prorroga +2 min (max total 5 min).
- ASM-001 backend.

### Mensagem institucional

> "Uma assembleia bem organizada começa com informações claras. Preencha os dados com atenção para garantir uma convocação transparente e respeitosa com todos os moradores."

---

## Tela 2 — Wizard Step 2 (Habilitar Administradora — D-127)

**Rota**: `/assembleias/:id/configurar/administradora` (Síndico)

```
┌──────────────────────────────────────────────────┐
│  HABILITAR ADMINISTRADORA — Step 2/4             │
│                                                  │
│  ( ) Empresa cadastrada Master Síndico          │
│      [🔍 Buscar por CNPJ ou nome]               │
│                                                  │
│  (●) Administradora externa                      │
│      Razão social * [____________________]      │
│      CNPJ *         [__.___.___/____-__]        │
│      Responsável *  [____________________]      │
│      CPF *          [___.___.___-__]            │
│      Email *        [____________________]      │
│      Telefone *     [(__) _____-____]           │
│                                                  │
│  [✓] Confirmo que esta empresa foi contratada   │
│      para administrar este condomínio.           │
│                                                  │
│  [← Voltar]                  [Enviar convite →] │
└──────────────────────────────────────────────────┘
```

### Fluxo backend (ASM-001 + D-127)

1. Síndico submete → cria `AssemblyAdministratorLink {status: pending_invite}`.
2. Sistema gera token HMAC keyed (ADR-0028) com `{assembly_id, company_id|user_id, expires_at=Assembly.scheduled_date+24h}`.
3. Email enviado com link `/admin-invite/:token`.
4. Administradora aceita → `accepted` → `active` quando `Assembly.Start()`.
5. Auto-invalida em `Assembly.End()` (cascade).
6. PermissionGroup `pg.assembly.administrator_delegation` (4 capabilities — ver `aggregates/AssemblyAdministratorLink.md`).

### Mensagem institucional

> "A administradora atua como parceira na organização e validação da assembleia, contribuindo para a segurança e legitimidade das decisões."

---

## Tela 3 — Wizard Step 3 (Cadastro completo da pauta)

**Rota**: `/assembleias/:id/configurar/pauta` (Síndico + Administradora pode observar)

Reaproveita `PautaAccordion` de [[./management|management.md]]. **Adições Fase E**:

- Campo **`impacto_financeiro` (boolean)** — quando `true` → libera campo `valor_estimado` (currency BRL).
- Campo **`quorum_exigido` (percentual)** — por item (sobrescreve `quorum_type` da assembleia).
- Tipo de votação por item: `SIM/NÃO/ABSTENÇÃO` ou `ESCALA 1-5 + ABSTENÇÃO` (novo Fase E — derivado do diagrama visual).
- Anexar vídeo explicativo (opcional, S3 presigned).
- Selecionar fornecedor participante quando `tipo_item = ESCOLHA_FORNECEDOR` (filtra acervo CNT-009 — ver [[../commercial/connect-me|commercial/connect-me]]).

### Lock pós-publicação

```
LOCK_Pauta = TRUE → não pode:
- alterar título
- alterar descrição
- alterar quórum
- alterar tipo de votação
- remover item
- adicionar item
```

Correção via adendo (timeline event não destrutivo — INS-016).

---

## Tela 4 — Wizard Step 4 (Anexar Edital)

**Rota**: `/assembleias/:id/configurar/edital`

```
┌──────────────────────────────────────────────────┐
│  EDITAL DE CONVOCAÇÃO — Step 4/4                │
│                                                  │
│  ( ) Anexar PDF pronto                           │
│      [📎 Selecionar arquivo]                    │
│      └ edital_convocacao_2026_03.pdf (1.2 MB)   │
│                                                  │
│  (●) Gerar automaticamente                       │
│      ✓ Identificação condomínio                  │
│      ✓ Tipo + data + horários convocação        │
│      ✓ Modalidade + local                        │
│      ✓ Ordem do dia (importada da pauta)        │
│      ✓ Regras participação / procuração / voto  │
│      ✓ Logo administradora                       │
│      ✓ Campo de assinatura                       │
│                                                  │
│  Quem pode gerar/anexar:                         │
│  • Síndico                                       │
│  • Administradora com permissão (D-127)          │
│                                                  │
│  [← Voltar]               [Publicar Assembleia]│
└──────────────────────────────────────────────────┘
```

### Regras

- Edital obrigatório antes de publicar (ASM-002).
- Geração assíncrona via job (wkhtmltopdf ou Puppeteer); status: `pending → ready → failed`.
- ao publicar: dispara notificações (push + email + SMS opt-in) + gera QR code da assembleia.

### Mensagem institucional

> "O edital é o documento que convoca oficialmente os moradores para a assembleia. Ele garante que todos tenham acesso às informações necessárias para participar das decisões do condomínio."

---

## Tela 5 — Painel de Monitoramento Pré-Assembleia

**Rota**: `/assembleias/:id/monitoramento` (Síndico + Administradora)

```
┌──────────────────────────────────────────────────────────┐
│  MONITORAMENTO PRÉ-ASSEMBLEIA                            │
│  Assembleia Ordinária 2026/1 • 15/03/2026 às 19h        │
│  T-3 dias                                                 │
│                                                           │
│  ┌─CIÊNCIA──────────┐  ┌─CONFIRMAÇÃO PRESENÇA────────┐ │
│  │ 67%  ████████░░░ │  │ Presencial:  42% ███████░░  │ │
│  │ 80/120 unidades  │  │ Online (M4): —              │ │
│  │ [ver pendentes]  │  │ Não defini:  18%            │ │
│  │                  │  │ Não vou:     8%             │ │
│  └──────────────────┘  └──────────────────────────────┘ │
│                                                           │
│  ┌─PROCURAÇÕES─────────────────────────────────────────┐│
│  │ Cadastradas: 12  Aprovadas: 9  Pendentes: 3 ⚠️    ││
│  │ [Validar agora →]                                   ││
│  └──────────────────────────────────────────────────────┘│
│                                                           │
│  ┌─QUÓRUM ESTIMADO─────────────────────────────────────┐│
│  │ Por item:                                            ││
│  │ • Pauta 1 (50%): 67% ✅ atingirá                    ││
│  │ • Pauta 2 (66%): 58% ⚠️ insuficiente                ││
│  │ • Pauta 3 (75%): 58% 🔴 longe do mínimo             ││
│  │ [Abrir simulador →]                                  ││
│  └──────────────────────────────────────────────────────┘│
│                                                           │
│  📊 Score de Transparência projetado: 78/100            │
│  [ver detalhe →]                                         │
└──────────────────────────────────────────────────────────┘
```

### Componentes

`PreAssemblyMonitorPanel`, `CienciaProgressCard`, `AttendanceConfirmationCard`, `ProxyValidationCard`, `QuorumProjectionCard`, `TransparencyScoreProjectionWidget`.

### Mensagem institucional

> "Quanto maior a participação dos moradores, mais representativas serão as decisões tomadas na assembleia."

---

## Tela 6 — Cadastro de Procuração (Morador) — 100% Digital (ASM-010)

**Rota**: `/assembleias/:id/procuracao/nova` (Morador autenticado)

> ⚠️ **D-_chaos / ASM-010 — REJEIÇÃO de upload físico**: NÃO há upload de PDF físico de procuração. Procuração 100% digital via assinatura digital (Lei 14.063 — ICP-Brasil).

```
┌──────────────────────────────────────────────────────┐
│  CADASTRAR PROCURAÇÃO                                │
│  Para: Assembleia Ordinária 2026/1                   │
│                                                      │
│  REPRESENTADO (titular)                              │
│  Bloco / Unidade *  [A] / [302]                     │
│  Nome              João da Silva (auto)             │
│  CPF               123.456.789-00 (auto)            │
│                                                      │
│  PROCURADOR (quem recebe)                            │
│  Buscar por CPF *   [___.___.___-__] [🔍]          │
│  └ ✓ Encontrado: Maria Souza (Bloco B Apt 105)      │
│  └ ✓ Mesma condomínio (validado)                    │
│                                                      │
│  ESCOPO DA PROCURAÇÃO *                              │
│  ( ) Para todos os itens da pauta                    │
│  (●) Para itens específicos                          │
│      [✓] Pauta 1: Contratação fachada                │
│      [✓] Pauta 3: Escolha fornecedor                 │
│      [ ] Pauta 2: Paisagismo                         │
│                                                      │
│  DECLARAÇÕES                                         │
│  [✓] Declaro estar ciente que esta procuração       │
│      é digital, com validade jurídica conforme       │
│      Lei 14.063/2020 (assinatura digital ICP-Brasil)│
│  [✓] Concedo poderes a Maria Souza para votar em    │
│      meu nome conforme escopo selecionado           │
│  [✓] Estou ciente que se eu votar pessoalmente, meu │
│      voto prevalece sobre o do procurador           │
│  [✓] Procuração permanece válida até 24h após o     │
│      encerramento da assembleia                      │
│                                                      │
│  [Cancelar]    [Confirmar com biometria/PIN] →     │
└──────────────────────────────────────────────────────┘
```

### Fluxo

1. Morador busca procurador por CPF (deve ser user da plataforma).
2. Sistema valida `proxy_holder_user_id` é morador do mesmo condomínio OU terceiro registrado.
3. Step-up MFA (IDN-036) — biometria/PIN/TOTP.
4. HMAC keyed (ADR-0028) gera `signature_hmac` sobre `{authorized_by, proxy_holder, scope, items, assembly_id, timestamp}`.
5. Estado: `pending_validation` → administradora valida via Tela 8.
6. Prazo: até momento antes de "Iniciar assembleia" (ASM-010).
7. `valid_until = Assembly.ended_at + 24h` (calculado quando assembly encerra).

### Estados

- `pending_validation` (aguardando administradora)
- `active` (validada, pode ser usada)
- `revoked` (cancelada pelo titular pré-início)
- `used` (procurador já votou)
- `expired` (24h pós encerramento)

### Mensagem informativa pós-criação

> "Procuração registrada. A administradora analisará e validará antes do início da assembleia. Você receberá notificação quando aprovada."

---

## Tela 7 — Análise de Fornecedores (Morador)

**Rota**: `/assembleias/:id/fornecedores` (Morador — disponível só durante período da assembleia)

```
┌──────────────────────────────────────────────────────┐
│  FORNECEDORES PARTICIPANTES                          │
│  Pauta 3: Escolha do Fornecedor para Fachada        │
│                                                      │
│  ┌────────────────────────────────────────────────┐ │
│  │  🏆 EMPRESA ALPHA FACHADAS                     │ │
│  │  ⭐ Fornecedor Master ⓘ Verificada Master Sínd. │ │
│  │  📹 [Vídeo institucional 2:30]                  │ │
│  │  Proposta: R$45.000 • 60 dias                   │ │
│  │  Escopo: pintura + impermeabilização            │ │
│  │  Diferenciais: garantia 5 anos                  │ │
│  │  [Ver detalhe completo →]                       │ │
│  └────────────────────────────────────────────────┘ │
│                                                      │
│  ┌────────────────────────────────────────────────┐ │
│  │  BETA RECUPERAÇÃO PREDIAL                       │ │
│  │  ⓘ Empresa NÃO verificada pela Master Síndico  │ │
│  │  Proposta: R$52.000 • 45 dias                   │ │
│  │  Escopo: pintura + recuperação estrutural       │ │
│  │  [Ver detalhe →]                                 │ │
│  └────────────────────────────────────────────────┘ │
│                                                      │
│  ⚠️ Vídeos das empresas estarão disponíveis também  │
│  durante a votação ao vivo (libera CNT-009 grant)   │
└──────────────────────────────────────────────────────┘
```

### Regras

- Acesso apenas durante a janela da assembleia (publicada → encerrada+24h).
- Fornecedor Master: badge selo + vídeo institucional + verificação verificada.
- Fornecedor externo: dados básicos + indicação "não verificada pela Master Síndico" (proteção legal — `12-CLAUSULAS-FUNCIONAIS`).
- ASM-008 (anexos pauta) + ASM-021 (votação fornecedor integrada com COM-018).

---

## Tela 8 — Validar Procurações (Administradora)

**Rota**: `/admin/assembleias/:id/procuracoes` (Administradora com `pg.assembly.administrator_delegation` D-127)

```
┌──────────────────────────────────────────────────────┐
│  VALIDAR PROCURAÇÕES                                 │
│  Assembleia Ordinária 2026/1 • 12 cadastradas       │
│                                                      │
│  Filtros: [Pendentes ▼] [Aprovadas] [Rejeitadas]   │
│                                                      │
│  ┌────────────────────────────────────────────────┐ │
│  │ ⏳ PENDENTE                                     │ │
│  │ Representado: João Silva (A-302) CPF *.789-00  │ │
│  │ Procurador:   Maria Souza (B-105) CPF *.123-44 │ │
│  │ Escopo: Itens 1, 3 (específicos)               │ │
│  │ Cadastrada: 12/03 às 14h32                     │ │
│  │ Assinatura HMAC: ✓ válida (ADR-0028)           │ │
│  │ Step-up MFA: ✓ verificada (IDN-036)            │ │
│  │                                                 │ │
│  │ [✗ Rejeitar com justificativa]  [✓ Aprovar →] │ │
│  └────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────┘
```

### Regras

- Administradora aprova OU rejeita (com justificativa obrigatória).
- Aprovação dispara `ProxyValidated` (event audit-trail).
- Voto do procurador será replicado para a unidade representada respeitando peso fração ideal.
- ASM-010 + R52 §3.

---

## Tela 9 — Importar Fração Ideal (Administradora)

**Rota**: `/admin/assembleias/:id/fracao-ideal` (Administradora com `assembly.fracao_ideal.import`)

```
┌──────────────────────────────────────────────────────┐
│  IMPORTAR FRAÇÃO IDEAL                               │
│                                                      │
│  Modelo da planilha:                                 │
│  | bloco | unidade | fração ideal | CPF titular |   │
│                                                      │
│  [📎 Upload planilha .xlsx ou .csv]                 │
│                                                      │
│  Validações automáticas:                             │
│  • Soma total = 100% ± 0.0001 (tolerância)          │
│  • Sem duplicidade (bloco + unidade)                │
│  • Sem campos obrigatórios vazios                   │
│  • Fração NUMERIC(5,4) por unidade                  │
│                                                      │
│  [📥 Baixar template]   [Validar e importar →]      │
└──────────────────────────────────────────────────────┘
```

### Regras

- Persistência única (importação 1x por condomínio).
- Reimportação só por correção/mudança estrutural.
- Snapshot congelado em `assembly_fractions` ao publicar pauta (ASM-007 / INV-083).
- ASM-007 backend.

---

## Tela 10 — Registrar Inadimplência (Administradora)

**Rota**: `/admin/assembleias/:id/inadimplencia` (Administradora com `assembly.inadimplencia.register`)

```
┌──────────────────────────────────────────────────────┐
│  CADASTRO DE INADIMPLÊNCIA                           │
│  ⏰ Cutoff: até 1h antes da 1ª convocação            │
│                                                      │
│  ( ) Upload planilha (CSV/Excel)                     │
│  (●) Seleção manual                                  │
│                                                      │
│  Lista de unidades:                                  │
│  ┌────────────────────────────────────────────────┐ │
│  │ Bloco A Apt 101  ✅ APTO       [bloquear]     │ │
│  │ Bloco A Apt 102  🚫 BLOQUEADO  [liberar voto] │ │
│  │ Bloco A Apt 103  ✅ APTO       [bloquear]     │ │
│  │ Bloco B Apt 201  🚫 BLOQUEADO  [liberar voto] │ │
│  │ ...                                            │ │
│  └────────────────────────────────────────────────┘ │
│                                                      │
│  ───────────────────────────────────────────────    │
│                                                      │
│  LIBERAR VOTO (permissão especial pós-cutoff)        │
│  Quando morador apresenta comprovante de pagamento   │
│  de última hora:                                     │
│                                                      │
│  Unidade: [B-201]  Justificativa: [_____________]   │
│  Responsável: Maria (admin)  Horário: auto          │
│  [✓] Confirmo que recebi comprovante de pagamento   │
│                                                      │
│  [Liberar voto →]                                    │
└──────────────────────────────────────────────────────┘
```

### Regras (NOVO ASM-### — sec 4.15)

- Upload até T-1h da 1ª convocação (R48 §5).
- Pós-cutoff: só botão "Liberar voto" individual com `{responsável, horário, justificativa}` em audit_trail.
- Quem libera: síndico OU administradora.
- Estados: `apto_a_votar | inapto_a_votar | liberado_manual`.

---

## Tela 11 — Simulador de Quórum

**Rota**: `/assembleias/:id/simulador` (Síndico + Administradora)

```
┌──────────────────────────────────────────────────────┐
│  SIMULADOR DE QUÓRUM                                 │
│                                                      │
│  Cenário hipotético — fração presente: [60%] ━●━━ │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │ Pauta 1 — Maioria simples (50%)              │   │
│  │ ✅ Atinge — sobra 10%                        │   │
│  │ Votos necessários: 51 (de 120 unidades)      │   │
│  ├──────────────────────────────────────────────┤   │
│  │ Pauta 2 — Maioria qualificada (66%)         │   │
│  │ ⚠️ Marginal — falta 6%                       │   │
│  │ Votos necessários: 80                        │   │
│  ├──────────────────────────────────────────────┤   │
│  │ Pauta 3 — Unanimidade (100%)                 │   │
│  │ 🔴 Inviável com presença atual               │   │
│  └──────────────────────────────────────────────┘   │
│                                                      │
│  Dados ao vivo:                                      │
│  • Total unidades: 120                               │
│  • Total frações: 100% (1.0000)                      │
│  • Ciência: 67% (80 unidades)                        │
│  • Confirmação presença: 42% (50 unidades)           │
│  • Procurações registradas: 12                       │
│  • Procurações aprovadas: 9                          │
└──────────────────────────────────────────────────────┘
```

### Regras

- Função pura (sem persistência) — ASM-011.
- Slider permite simular cenários de presença.
- Mostra projeção por item (cada item tem `quorum_exigido` próprio — Fase E nova regra).

---

## Componentes

`AssemblyWizard`, `WizardSteps`, `AdminInviteForm`, `EditalUploader`, `EditalGenerator`, `PreAssemblyMonitorPanel`, `ProxyForm`, `ProxyValidationList`, `SupplierAnalysisCard`, `FracaoIdealUploader`, `InadimplenciaTable`, `LiberarVotoButton`, `QuorumSimulator`, `QuorumSimulatorSlider`.

## Cross-links

- [[./management|management]] — wizard genérico + lista de assembleias
- [[./voting|voting]] — sistema de votação (durante assembleia ao vivo)
- [[./live-session|live-session]] — sessão ao vivo
- [[./check-in|check-in]] — modalidades de check-in
- [[./administrator-panel|administrator-panel]] — painel admin completo
- [[../../../../04-requirements/functional/assembly|functional/assembly]] — ASM-001/002/004/005/007/010/011
- [[../../../../01-domain/aggregates/Assembly|Assembly]]
- [[../../../../01-domain/aggregates/Proxy|Proxy]]
- [[../../../../01-domain/aggregates/AssemblyAdministratorLink|AssemblyAdministratorLink]] (D-127)
- [[../../../../STATE]] — D-127, D-130, D-132, D-133

## Pendências UX

- ⚠️ "Hating imediato" mencionado em PDF §4.4 Eleição não está claro se é avaliação escondida (já existe ASM-ELE-AVAL) ou se é mecanismo distinto. **Assume-se** alinhamento com ASM-ELE-AVAL pré-existente.
- ⚠️ Provider de assinatura digital ICP-Brasil (Bry, ITI) — ADR pendente para M4+ (M3 stub HMAC).
