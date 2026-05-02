# 09 — GESTÃO DE ASSEMBLEIAS

> Sprint 2 · Rotas: /assembleias, /assembleias/:id, /assembleias/nova, /assembleias/:id/editar
> O coração do CMS. Exclusivo Síndico N2/N3 (criação) e Moradores (visualização/votação).

---

## REGRAS DE NEGÓCIO

- CRUD de eventos com: Título, Data, Edital (PDF/Texto), Status (Agendado, Aberto, Encerrado)
- Pautas são sub-itens: Título, Descrição, Vídeo/Áudio do Síndico, Anexos de fornecedores
- Link temporário compartilhável com janela de acesso definida
- Assembleia é ASSÍNCRONA — não é videoconferência (Zoom/Meet)
- Votação vinculada a pautas (ver doc 10)
- Mobile: Morador visualiza/vota. Síndico pode criar/editar simples pelo celular

## LAYOUT — LISTA DE ASSEMBLEIAS

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
  border-radius: 12px; cursor: pointer; transition: all 200ms;
}
.assembly-card:hover { border-color: var(--primary); box-shadow: 0 2px 8px rgb(0 0 0 / 0.05); }
.assembly-status { display: inline-flex; align-items: center; gap: 4px; padding: 2px 8px; border-radius: 9999px; font-size: 11px; font-weight: 600; }
.assembly-status.aberta { background: oklch(0.627 0.170 149.2 / 0.1); color: var(--success); }
.assembly-status.agendada { background: oklch(0.769 0.165 70.1 / 0.1); color: var(--warning); }
.assembly-status.encerrada { background: var(--muted); color: var(--muted-foreground); }
```

## WIZARD DE CRIAÇÃO (4 steps)

```
[1. Dados Gerais] ─── [2. Pautas] ─── [3. Votação] ─── [4. Revisão]
     ●═══════════════○═══════════════○═══════════════○
```

### Step 1 — Dados Gerais
Título, Data, Edital (upload PDF ou campo texto rico), Janela de acesso (data início/fim)

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
Buscar vídeos de fornecedores: modal com search no acervo das empresas cadastradas.

### Step 3 — Configuração de Votação
Para cada pauta, definir: tipo (Aprovar/Reprovar ou Múltipla Escolha), candidatos, regras

### Step 4 — Revisão e Publicação
Resumo de tudo, preview do link, [Publicar Assembleia →]

```css
.wizard-steps { display: flex; align-items: center; justify-content: center; gap: 0; margin-bottom: 32px; }
.wizard-step { display: flex; align-items: center; gap: 8px; }
.wizard-step-circle {
  width: 32px; height: 32px; border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  font-family: 'Manrope'; font-size: 14px; font-weight: 600;
}
.wizard-step-circle.active { background: var(--primary); color: var(--primary-foreground); }
.wizard-step-circle.completed { background: var(--primary); color: var(--primary-foreground); }
.wizard-step-circle.upcoming { background: var(--muted); color: var(--muted-foreground); border: 2px solid var(--border); }
.wizard-step-line { width: 60px; height: 2px; background: var(--border); }
.wizard-step-line.completed { background: var(--primary); }
.wizard-step-label { font-family: 'Manrope'; font-size: 13px; color: var(--muted-foreground); }
.wizard-step.active .wizard-step-label { color: var(--primary); font-weight: 600; }
```

## DETALHE DA ASSEMBLEIA (Morador)

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

## LINK TEMPORÁRIO

Síndico N3 pode gerar link de compartilhamento com janela de acesso.
Link leva para versão read-only da assembleia, acessível sem login durante a janela.

```css
.temp-link-card {
  padding: 16px; background: var(--card); border: 1px solid var(--primary) / 0.3;
  border-radius: var(--radius); display: flex; align-items: center; gap: 12px;
}
.temp-link-url { flex: 1; font-family: monospace; font-size: 13px; color: var(--muted-foreground); overflow: hidden; text-overflow: ellipsis; }
.temp-link-copy { cursor: pointer; color: var(--primary); }
.temp-link-expiry { font-size: 12px; color: var(--warning); margin-top: 8px; }
```

## COMPONENTES

AssemblyListPage, AssemblyCard, AssemblyStatusBadge, AssemblyWizard, WizardSteps, WizardStep, PautaAccordion, PautaForm, VendorVideoSearch, AssemblyDetail, TempLinkCard, EditalViewer
