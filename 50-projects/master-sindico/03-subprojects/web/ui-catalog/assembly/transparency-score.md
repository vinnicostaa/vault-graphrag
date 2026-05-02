---
title: Transparency Score — UI Spec
type: ui-spec
tags: [ui-catalog, assembly, master-sindico, frontend, transparencia, kpi]
bc: assembly
source: _chaos/ARQUITETURA-ASSEMBLEIA.pdf §8 + MÓDULO-TELAS.pdf
absorbed_in: 2026-04-25 — Fase E
status: absorbed
---

# Transparency Score — Indicador de Transparência da Assembleia

> **Origem**: 3 PDFs Master Síndico §8 — destilados em Fase E (2026-04-25).
> **Backend**: ASM-028 (10 componentes), ASM-030 (widget perfil condomínio).

> Sprint 4-5 · App `assembly` · Rotas: `/assembleias/:id/transparencia`, `/condominios/:id/transparencia`
> Perfis: Síndico, Administradora, Morador (read-only), Público (perfil condomínio).

---

## Filosofia

O **Indicador de Transparência da Assembleia** é um score 0-100 calculado automaticamente após a homologação. Mede 10 componentes em 4 subíndices:

- **Convocação** (3 componentes — 30 pts)
- **Participação** (3 componentes — 30 pts)
- **Votação** (2 componentes — 20 pts)
- **Documentação** (2 componentes — 20 pts)

Não é "ranking" mas sim **proxy de boa governança**.

---

## 10 Componentes (ASM-028 — atualizado Fase E)

| # | Componente                                              | Pontos | Subíndice       |
|---|---------------------------------------------------------|--------|-----------------|
| 1 | Pauta publicada ≥ 8 dias (Lei 4.591/64)                | 10     | Convocação      |
| 2 | Edital distribuído                                      | 10     | Convocação      |
| 3 | % Ciência registrada (ponderado)                        | 10     | Convocação      |
| 4 | % Presença sobre convocados                             | 10     | Participação    |
| 5 | % Votantes sobre aptos                                  | 10     | Participação    |
| 6 | % Visualização documentos+fornecedores (engajamento)    | 10     | Participação    |
| 7 | Quórum atingido em todos os itens                       | 10     | Votação         |
| 8 | Regularidade da homologação (todos itens homologados)   | 10     | Votação         |
| 9 | Completude trilha auditoria (20+ eventos)               | 10     | Documentação    |
| 10| Cumprimento integral da pauta + tempo médio por item    | 10     | Documentação    |

> **NOTA Fase E — discrepância vs ASM-028 atual**: o legado canônico tinha lista parcialmente diferente ("Votação online disponível", "Ata publicada Timeline", "Relatórios públicos acessíveis", "Fala equilibrada", "Sem atrasos encerramento"). A lista do `_chaos/ARQUITETURA §8` é **mais granular e canônica em PT-BR**. Decisão Fase E: **substituir** ASM-028 com lista nova (10 componentes acima).

---

## Tela 1 — Detalhe do Score (Síndico/Admin)

**Rota**: `/assembleias/:id/transparencia`

```
┌─────────────────────────────────────────────────────┐
│  📊 SCORE DE TRANSPARÊNCIA                          │
│  Assembleia Ordinária 2026/1                        │
│                                                     │
│  ┌─SCORE GERAL──────────────────────────────────┐  │
│  │                                               │  │
│  │            87/100                            │  │
│  │           ████████░░                          │  │
│  │                                               │  │
│  │       ⓘ Bom — acima da média (75)            │  │
│  └───────────────────────────────────────────────┘  │
│                                                     │
│  ┌─SUBÍNDICES──────────────────────────────────┐   │
│  │                                              │   │
│  │ 📣 CONVOCAÇÃO        25/30 ████████░         │   │
│  │ ✋ PARTICIPAÇÃO       22/30 ███████░░         │   │
│  │ 🗳 VOTAÇÃO           20/20 ██████████        │   │
│  │ 📑 DOCUMENTAÇÃO      20/20 ██████████        │   │
│  │                                              │   │
│  └──────────────────────────────────────────────┘   │
│                                                     │
│  ┌─COMPONENTES DETALHADOS───────────────────────┐  │
│  │ ✅ Pauta publicada ≥ 8 dias       10/10     │  │
│  │ ✅ Edital distribuído              10/10     │  │
│  │ ⚠️ Ciência registrada (67%)         5/10     │  │
│  │ ⚠️ Presença/convocados (60%)        6/10     │  │
│  │ ✅ Votantes/aptos (90%)             9/10     │  │
│  │ ⚠️ Visualização docs (50%)          7/10     │  │
│  │ ✅ Quórum em todos itens           10/10     │  │
│  │ ✅ Regularidade homologação        10/10     │  │
│  │ ✅ Trilha auditoria completa        10/10     │  │
│  │ ✅ Cumprimento da pauta            10/10     │  │
│  └──────────────────────────────────────────────┘   │
│                                                     │
│  💡 OPORTUNIDADES PARA PRÓXIMA ASSEMBLEIA           │
│  • Aumentar ciência: enviar lembrete adicional T-1d │
│  • Promover engajamento doc: mensagem destacando    │
│    importância de leitura prévia                    │
│                                                     │
│  [📥 Baixar relatório PDF]  [📊 Histórico 12 últ.]  │
└─────────────────────────────────────────────────────┘
```

---

## Tela 2 — Widget de Score (Perfil Público do Condomínio)

**Rota**: `/condominios/:id` (público — qualquer um, parte do perfil institucional)

```
┌──────────────────────────────────────────────┐
│  CONDOMÍNIO AURORA                           │
│  Av. Paulista 1000 • SP                      │
│                                              │
│  ┌────────────────────────────────────────┐ │
│  │ 📊 TRANSPARÊNCIA: 87/100               │ │
│  │ ████████░░                             │ │
│  │ "Bom" — acima da média (75)            │ │
│  │                                        │ │
│  │ Última assembleia: 15/03/2026          │ │
│  │ [Ver detalhes →]                       │ │
│  └────────────────────────────────────────┘ │
│                                              │
│  📈 EVOLUÇÃO 12 ÚLTIMAS ASSEMBLEIAS         │
│  ╭────────────────────────────────╮         │
│  │ 100 ┤    ●●                    │         │
│  │  90 ┤  ●   ●●                  │         │
│  │  80 ┤●─       ●●─●             │         │
│  │  70 ┤            ╲●            │         │
│  │     └──────────────────         │         │
│  ╰────────────────────────────────╯         │
└──────────────────────────────────────────────┘
```

### Regras (ASM-030)

- Widget público (acesso anônimo).
- Cache de 1h (não precisa atualizar real-time).
- Histórico das 12 últimas assembleias.
- Comparativo com benchmark do mercado (sugere média BR).

---

## Faixas de Interpretação

| Score   | Faixa            | Cor        | Mensagem                                    |
|---------|------------------|------------|---------------------------------------------|
| 90-100  | Excelência       | 🟢 Verde   | "Excelente — referência de transparência"   |
| 75-89   | Bom              | 🟢 Verde   | "Bom — acima da média"                      |
| 60-74   | Regular          | 🟡 Amarelo | "Regular — espaço para melhorar"            |
| 40-59   | Baixo            | 🟠 Laranja | "Baixo — atenção a participação/registros" |
| 0-39    | Crítico          | 🔴 Vermelho| "Crítico — risco de contestação jurídica"   |

---

## Mecânica de Pontuação

### Componentes binários (10pt ou 0pt)

- Pauta ≥ 8 dias antes
- Edital distribuído
- Quórum em todos itens
- Regularidade homologação
- Trilha auditoria completa
- Cumprimento integral da pauta

### Componentes proporcionais (0-10pt)

- Ciência registrada: `pontos = round((% ciência / 100) * 10)`.
- Presença/convocados: idem.
- Votantes/aptos: idem.
- Visualização docs+fornecedores: idem.

### Subíndice Convocação

```
score_convocacao = (item1 + item2 + item3) / 30 * 100
```

E assim por diante para os demais.

---

## Componentes

`TransparencyScoreCard`, `TransparencyScoreGauge`, `SubindexBarChart`, `ComponentChecklist`, `OpportunitySuggestionCard`, `TransparencyScoreEvolutionChart` (re-uso de [[./post-assembly|post-assembly]]), `PublicTransparencyWidget`.

## Cross-links

- [[./post-assembly|post-assembly]] — onde score é calculado e exibido inicialmente
- [[./pre-assembly|pre-assembly]] — score projetado em monitoramento
- [[../institutional/transparency|institutional/transparency]] — Painel Transparência institucional macro (do condomínio inteiro)
- [[../../../../04-requirements/functional/assembly|functional/assembly]] — ASM-028, ASM-030
- [[../../../../01-domain/aggregates/Assembly|Assembly]] — campo `transparencyScore`

## Pendências

- ⚠️ Fórmula precisa de "regularidade da homologação" (todos homologados em 24h? 7 dias?).
- ⚠️ "Tempo médio por item" — qual benchmark usar (15min/item? configurável?).
- ⚠️ Histórico 12 assembleias: caching strategy + invalidação após nova homologação.
- ⚠️ Comparativo benchmark BR: precisa data set inicial → seed M4+ com dados públicos de assembleias agregados.
