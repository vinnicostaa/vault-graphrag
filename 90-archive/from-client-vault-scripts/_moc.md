---
title: Scripts do vault do cliente (gerador de canvas)
type: moc
tags: [archive, master-sindico, scripts, tools, canvas-generator]
source: ~/Documentos/Obsidian Vault/Clients/MasterSindico/{generate.js, generate_seq.js}
created: 2026-04-24
---

# Scripts do vault do cliente

Scripts JavaScript que o cliente João usa no vault `Clients/MasterSindico/` para **gerar canvas do Obsidian programaticamente** a partir de specs JS (nodes + edges).

**Não são conteúdo de produto** — são ferramentas de autoria. Arquivados para:
1. **Referência histórica** — documentar como o cliente produziu os 23 canvas em `vault/50-projects/master-sindico/client-canvas/`
2. **Reutilização eventual** — se precisar gerar canvas novos (ex: fluxos de features novas M2/M3), o pattern está aqui

## Arquivos

- `generate.js` (22kb, 2026-03-10) — gerador base `createCanvas(filename, spec)` lendo `{nodes, edges}` do spec e escrevendo JSON canvas (`nodes[].{id, type, text, label, x, y, width, height, color}` + `edges[].{id, fromNode, fromSide, toNode, toSide}`).
- `generate_seq.js` (8kb, 2026-03-10) — variante focada em diagramas de sequência (altura default 80 px vs 100 px do base).

## Como usar

Rodar via Node dentro de uma pasta com spec JS:

```bash
node generate.js <spec-file.js>
# → gera <spec-file>.canvas
```

Spec format (inferido da leitura do código):
```js
module.exports = {
  filename: 'Fluxo X.canvas',
  nodes: [
    { id: 'n1', text: 'Início', x: 0, y: 0, width: 250, height: 80 },
    { id: 'n2', text: 'Fim', x: 400, y: 0 },
  ],
  edges: [
    { from: 'n1', to: 'n2', fromSide: 'right', toSide: 'left' }
  ]
};
```

## Canvas gerados

Os 23 canvas produzidos com esses scripts estão em [[../../50-projects/master-sindico/client-canvas/_moc]].

## Links

- [[../_moc]]
- [[../../50-projects/master-sindico/client-canvas/_moc]]
- [[../../50-projects/master-sindico/STATE#d-093]]
