---
title: Assembly Management — UI Spec
type: ui-spec
tags: [ui-catalog, master-sindico, assembly, livekit]
bc: assembly
source: _chaos/09-ASSEMBLY-MANAGEMENT.md (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# 09 — Gestão de Assembleias

> **Origem**: `_chaos/09-ASSEMBLY-MANAGEMENT.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Tradução aplicada**: `N1/N2/N3` → `trial/base/plus/pro` (ADR-0032 / D-081 / D-096); "My Síndico" → "Master Síndico".
> **⚠️ Reconciliação D-133 (2026-04-25)**: Assembleia é **conferência tipo Meet** (presencial/híbrida/online) com votação ao vivo via **Livekit** (canônico ADR-0019/CLAUDE §4). NÃO é "live broadcast institucional" (esse é [[../content/live-streaming|content/live-streaming]] separado).

> Sprint 2 · App `assembly` (porta 3004) · Rotas: `/assembleias`, `/assembleias/:id`, `/assembleias/nova`, `/assembleias/:id/editar`
> Exclusivo Síndico `plus`/`pro` (criação) e Moradores (visualização/votação).

---

## Regras de negócio

- **CRUD de eventos** com: Título, Data, Edital (PDF / Texto), Status (Agendado, Aberto, Encerrado).
- **Pautas são sub-itens**: Título, Descrição, Vídeo / Áudio do Síndico, Anexos de fornecedores.
- **Link temporário compartilhável** com janela de acesso definida.
- **Modalidades suportadas** (M3+):
  - **Presencial** (MVP M3): check-in via app / QR Code / terminal CPF + bloco + unidade
  - **Híbrida** e **online**: Livekit (telão 2 áreas + fila de fala + voto fracional)
- **Votação vinculada a pautas** (ver [[./voting|voting]]).
- **Mobile**: Morador visualiza / vota; Síndico pode criar / editar simples pelo celular.

> Para Lives institucionais broadcast (vídeos ao vivo de empresas), ver [[../content/live-streaming|content/live-streaming]] (provider Mux Live ou Cloudflare Stream — D-133).

## Layout — Lista de assembleias

```
┌──────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                │
│          │  ASSEMBLEIAS E EVENTOS        [+ Nova →] btn  │
│          │  h2 Michroma                                   │
│          │                                                │
│          │  [Todas] [Agendadas] [Abertas] [Encerradas]   │
│          │                                                │
│          │  ┌────────────────────────────────────────────┐│
│          │  │ 📋 Assembleia Ordinária 2026/1             ││
│          │  │ 🟢 ABERTA • 15/03/2026 • 5 pautas         ││
│          │  │ Votação até: 20/03/2026              [→]   ││
│          │  └────────────────────────────────────────────┘│
│          │  ┌────────────────────────────────────────────┐│
│          │  │ 📋 Manutenção Elevadores                   ││
│          │  │ ⏳ AGENDADA • 01/04/2026 • 3 pautas        ││
│          │  │                                      [→]   ││
│          │  └────────────────────────────────────────────┘│
│          │  ┌────────────────────────────────────────────┐│
│          │  │ 📋 Escolha Prestador Fachada               ││
│          │  │ 🔴 ENCERRADA • 10/01/2026 • Resultado: ✅  ││
│          │  │                                      [→]   ││
│          │  └────────────────────────────────────────────┘│
└──────────────────────────────────────────────────────────┘
```

```css
.assembly-card {
  padding: 16px; background: var(--card); border: 1px solid var(--border);
  border-radius: var(--radius-xl); cursor: pointer;
  transition: all var(--duration-moderate) var(--ease-out);
}
.assembly-card:hover {
  border-color: var(--primary); box-shadow: var(--shadow-sm);
}
.assembly-status {
  display: inline-flex; align-items: center; gap: 4px;
  padding: 2px 8px; border-radius: var(--radius-full);
  font-size: 11px; font-weight: 600;
}
.assembly-status.aberta    { background: oklch(0.627 0.170 149.2 / 0.1); color: var(--success); }
.assembly-status.agendada  { background: oklch(0.769 0.165 70.1 / 0.1); color: var(--warning); }
.assembly-status.encerrada { background: var(--muted); color: var(--muted-foreground); }
```

## Wizard de criação (4 steps)

```
[1. Dados Gerais] ─── [2. Pautas] ─── [3. Votação] ─── [4. Revisão]
     ●═══════════════○═══════════════○═══════════════○
```

### Step 1 — Dados Gerais
Título, Data, Edital (upload PDF ou campo texto rico), Janela de acesso (data início / fim).

### Step 2 — Pautas (a área mais complexa)

```
┌────────────────────────────────────────────────────┐
│  Pautas da Assembleia             [+ Nova Pauta]   │
│                                                    │
│  ▼ PAUTA 1: Contratação para fachada              │
│  │  Descrição: Lorem ipsum...                      │
│  │                                                 │
│  │  📹 Vídeo do Síndico: [Upload] [Gravar]        │
│  │  🎵 Áudio: [Upload] [Gravar]                   │
│  │                                                 │
│  │  📎 Vídeos de Fornecedores:                     │
│  │  [🔍 Buscar no acervo de empresas]             │
│  │  [thumb Emp.A] [thumb Emp.B] [thumb Emp.C]      │
│  │                                                 │
│  │  [⬆] [⬇] [🗑]                                 │
│  │                                                 │
│  ▶ PAUTA 2: Paisagismo... (collapsed)             │
│  ▶ PAUTA 3: Orçamento... (collapsed)              │
│                                                    │
│  [← Voltar]                    [Próximo Passo →]  │
└────────────────────────────────────────────────────┘
```

Pautas usam Accordion (Kobalte). Cada pauta expandida mostra campos.
Buscar vídeos de fornecedores: modal com search no acervo das empresas cadastradas. Habilita exibição integral durante votação para morador `base` (ver [[../content/video-player#paywall-25]]).

### Step 3 — Configuração de Votação
Para cada pauta, definir: tipo (Aprovar / Reprovar, Múltipla Escolha, Fração ideal DECIMAL, Eleição), candidatos, regras. Detalhes em [[./voting|voting]].

### Step 4 — Revisão e Publicação
Resumo de tudo, preview do link, [Publicar Assembleia →].

```css
.wizard-steps {
  display: flex; align-items: center; justify-content: center;
  gap: 0; margin-bottom: 32px;
}
.wizard-step { display: flex; align-items: center; gap: 8px; }
.wizard-step-circle {
  width: 32px; height: 32px; border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  font-family: 'Manrope'; font-size: 14px; font-weight: 600;
}
.wizard-step-circle.active     { background: var(--primary); color: var(--primary-foreground); }
.wizard-step-circle.completed  { background: var(--primary); color: var(--primary-foreground); }
.wizard-step-circle.upcoming   { background: var(--muted); color: var(--muted-foreground); border: 2px solid var(--border); }
.wizard-step-line { width: 60px; height: 2px; background: var(--border); }
.wizard-step-line.completed { background: var(--primary); }
.wizard-step-label {
  font-family: 'Manrope'; font-size: 13px; color: var(--muted-foreground);
}
.wizard-step.active .wizard-step-label { color: var(--primary); font-weight: 600; }
```

## Detalhe da assembleia (Morador)

```
┌──────────────────────────────────────────────┐
│  Assembleia Ordinária 2026/1                  │
│  🟢 ABERTA • Votação até 20/03/2026          │
│                                               │
│  📄 Edital [Baixar PDF]                       │
│                                               │
│  PAUTAS:                                      │
│                                               │
│  ▼ Pauta 1: Contratação Fachada              │
│  │  Descrição completa...                     │
│  │  📹 [Player: Vídeo do Síndico]            │
│  │  📎 Fornecedores:                          │
│  │  [Player Emp.A] [Player Emp.B] [Player C] │
│  │  [VOTAR NESTA PAUTA →]                    │
│  │                                            │
│  ▼ Pauta 2: Paisagismo                       │
│  │  ...                                       │
│  │  [✅ Você já votou] (se já votou)          │
└──────────────────────────────────────────────┘
```

## Link temporário

Síndico `pro` pode gerar link de compartilhamento com janela de acesso.
Link leva para versão read-only da assembleia, acessível sem login durante a janela (ABAC trata token assinado).

```css
.temp-link-card {
  padding: 16px; background: var(--card);
  border: 1px solid oklch(0.715 0.120 84.2 / 0.3);
  border-radius: var(--radius-md);
  display: flex; align-items: center; gap: 12px;
}
.temp-link-url {
  flex: 1; font-family: var(--font-mono); font-size: 13px;
  color: var(--muted-foreground); overflow: hidden; text-overflow: ellipsis;
}
.temp-link-copy { cursor: pointer; color: var(--primary); }
.temp-link-expiry { font-size: 12px; color: var(--warning); margin-top: 8px; }
```

## Componentes

`AssemblyListPage`, `AssemblyCard`, `AssemblyStatusBadge`, `AssemblyWizard`, `WizardSteps`, `WizardStep`, `PautaAccordion`, `PautaForm`, `VendorVideoSearch`, `AssemblyDetail`, `TempLinkCard`, `EditalViewer`.

## Links

### Telas detalhadas (Fase E — 2026-04-25)

- [[./pre-assembly|pre-assembly]] — wizard 4 steps + procuração 100% digital + análise fornecedores + simulador quórum
- [[./live-session|live-session]] — painel ao vivo síndico + mesa diretora + telão 2 áreas + fila fala + voto presencial assistido
- [[./check-in|check-in]] — 3 modalidades (app, QR code, terminal)
- [[./post-assembly|post-assembly]] — encerramento + ata + relatórios + histórico
- [[./administrator-panel|administrator-panel]] — painel admin (D-127 — 4 capabilities + audit trail)
- [[./transparency-score|transparency-score]] — indicador 10 componentes (4 subíndices)
- [[./contingency-mode|contingency-mode]] — modo offline + auto-save Redis

### Outros

- [[./voting|voting]] — sistema de votação (10)
- [[../../../web/ui-catalog#e-app-assembly-as1-as8]] — catálogo macro AS1-AS8
- [[../content/live-streaming|content/live-streaming]] — Lives institucionais (D-133, NÃO é assembleia)
- [[../../../../STATE]] — D-127, D-130, D-132, D-133
- [[../../patterns/ui-states|patterns/ui-states]]
- [[../../../../02-architecture/design-tokens|design-tokens]]
