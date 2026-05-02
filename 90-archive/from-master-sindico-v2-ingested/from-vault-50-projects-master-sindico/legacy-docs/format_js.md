---
title: "format (js)"
type: source
tags: [code, js, converted]
source: 50-projects/master-sindico/legacy-docs/format.js
converted: 2026-04-22
---

# format

Arquivo `js` convertido wrap em code block. Conteúdo original preservado.

```javascript
const fs = require('fs');
const path = require('path');

const inputPath = path.join(__dirname, 'telas.txt');
const outputPath = path.join(__dirname, 'telas.md');

const content = fs.readFileSync(inputPath, 'utf8');
const lines = content.split('\n');

let md = [];
for (let i = 0; i < lines.length; i++) {
  let line = lines[i].trim();
  
  if (!line) {
    continue;
  }
  
  if (line.match(/^Rafael Mimosa/)) {
    continue;
  }

  // Headers
  if (line.match(/^\*Jornada/)) {
    md.push(`\n# ${line.replace(/\*/g, '')}\n`);
    continue;
  }
  if (line.match(/^BLOCO \d+/)) {
    md.push(`\n## ${line}\n`);
    continue;
  }
  if (line.match(/^TELA \w+/)) {
    md.push(`\n### ${line}\n`);
    continue;
  }
  if (line.match(/^🔷 E\d+/)) {
    md.push(`\n### ${line}\n`);
    continue;
  }
  if (line.match(/^MÓDULO COMPLIANCE/)) {
    md.push(`\n## ${line}\n`);
    continue;
  }
  if (line.match(/^TELA C\d+/)) {
    md.push(`\n### ${line}\n`);
    continue;
  }

  // Common Key-Value prefixes to bold
  const boldPrefixes = [
    'Nome do botão', 'Título', 'Objetivo', 'Mensagem', 'Mensagem inicial', 'Rodapé',
    'Campos obrigatórios', 'Campos', 'Botões', 'Botão', 'Regras exibidas',
    'Regras', 'Regra', 'Regra sistêmica', 'Regra de prazo', 'Regra de imutabilidade',
    'Justificativa', 'Justificativa para devs', 'Justificativa devs', 'Link', 'Links',
    'Texto completo', 'Texto', 'Checkbox', 'Registro', 'Status', 'Estados', 'Opção', 'Opções',
    'Exibe', 'Cards', 'Indicadores', 'Filtros', 'Tipos', 'Visibilidade', 'Uso'
  ];

  let matchedPrefix = false;
  for (const prefix of boldPrefixes) {
    if (line.toLowerCase().startsWith(prefix.toLowerCase() + ':')) {
       md.push(`\n**${line.substring(0, prefix.length + 1)}** ${line.substring(prefix.length + 1).trim()}`);
       matchedPrefix = true;
       break;
    }
  }

  if (matchedPrefix) continue;

  // Simple lines -> list items, but avoid if it's a long paragraph
  if (line.length < 120 && !line.endsWith('.') && !line.includes(':')) {
    md.push(`- ${line}`);
  } else {
    // Normal paragraph
    md.push(line);
  }
}

// Clean up weird double bullet points or excessive newlines
const outputText = md.join('\n').replace(/\n{3,}/g, '\n\n');

fs.writeFileSync(outputPath, outputText);
console.log('telas.md foi gerado com sucesso!');

```
