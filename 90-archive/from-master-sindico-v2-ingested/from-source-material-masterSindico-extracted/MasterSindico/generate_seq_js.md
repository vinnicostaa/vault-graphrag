---
title: "generate_seq (js)"
type: source
tags: [code, js, converted]
source: 50-projects/master-sindico/MasterSindico_extracted/MasterSindico/generate_seq.js
converted: 2026-04-22
---

# generate_seq

Arquivo `js` convertido wrap em code block. Conteúdo original preservado.

```javascript
const fs = require('fs');
const path = require('path');

function createCanvas(filename, spec) {
  const nodes = [];
  const edges = [];
  let nodeId = 1;
  let edgeId = 1;

  for (const n of spec.nodes) {
    nodes.push({
      id: n.id || `n${nodeId++}`,
      type: n.type || (n.label ? "group" : "text"),
      text: n.text,
      label: n.label,
      x: n.x,
      y: n.y,
      width: n.width || 250,
      height: n.height || 80,
      color: n.color
    });
  }

  for (const e of spec.edges) {
    edges.push({
      id: e.id || `e${edgeId++}`,
      fromNode: e.from,
      fromSide: e.fromSide || "right",
      toNode: e.to,
      toSide: e.toSide || "left",
      label: e.label,
      style: e.style
    });
  }

  const output = { nodes, edges };
  fs.writeFileSync(filename, JSON.stringify(output, null, 2));
  console.log(`Created ${filename}`);
}

const dir = "/home/vinni/Documentos/Obsidian Vault/Clients/MasterSindico";

// 14. Fluxo Central - Governanca
createCanvas(path.join(dir, "02 - Domínios e Requisitos", "Fluxo Governanca.canvas"), {
  nodes: [
    { id: "S1", text: "👔 Síndico\nCria atividade da gestão", x: 0, y: 0 },
    { id: "T1", text: "📜 Timeline\nRecebe atividade", x: 400, y: 0 },
    { id: "PD1", text: "📋 Plano Diretor\nStatus atualizado auto", x: 800, y: 0 },
    
    { id: "E1", text: "🏢 Empresa\nEnvia registro execução", x: 0, y: 150 },
    { id: "S2", text: "👔 Síndico\nValida registro", x: 400, y: 150 },
    { id: "T2", text: "📜 Timeline\nPublica registro (imutável)", x: 800, y: 150 },
    
    { id: "M1", text: "🏠 Morador\nVisualiza Timeline", x: 400, y: 300 },
    { id: "M2", text: "🏠 Morador\nAvalia gestão", x: 800, y: 300 }
  ],
  edges: [
    { from: "S1", to: "T1" },
    { from: "T1", to: "PD1" },
    
    { from: "E1", to: "S2" },
    { from: "S2", to: "T2" },
    
    { from: "T2", to: "M1", label: "modo leitura", fromSide: "bottom", toSide: "top" },
    { from: "S2", to: "M2", label: "avalia (3 perguntas)", fromSide: "bottom", toSide: "top" }
  ]
});

// 15. Fluxo Connect Me
createCanvas(path.join(dir, "02 - Domínios e Requisitos", "Fluxo Connect Me.canvas"), {
  nodes: [
    { id: "O1", text: "Originador\nCria formulário", x: 0, y: 0 },
    { id: "CM1", text: "Connect Me\nRegistra LGPD", x: 400, y: 0 },
    { id: "D1", text: "Destinatário\nRecebe notificação", x: 800, y: 0 },
    
    { id: "D2", text: "Destinatário\n'Tenho Interesse'", x: 800, y: 150 },
    { id: "CM2", text: "Connect Me\nRevela dados cruzados", x: 400, y: 150 },
    { id: "O2", text: "Originador\nRecebe contato", x: 0, y: 150 },
    
    { id: "D3", text: "Destinatário\n'Não Atendo'", x: 800, y: 300 },
    { id: "CM3", text: "Connect Me\nNotifica recusa", x: 400, y: 300 },
    { id: "O3", text: "Originador\nFim do fluxo", x: 0, y: 300 }
  ],
  edges: [
    { from: "O1", to: "CM1" },
    { from: "CM1", to: "D1" },
    
    { from: "D1", to: "D2", fromSide: "bottom", toSide: "top" },
    { from: "D2", to: "CM2", fromSide: "left", toSide: "right" },
    { from: "CM2", to: "O2", fromSide: "left", toSide: "right" },
    
    { from: "D1", to: "D3", fromSide: "bottom", toSide: "top" },
    { from: "D3", to: "CM3", fromSide: "left", toSide: "right" },
    { from: "CM3", to: "O3", fromSide: "left", toSide: "right" }
  ]
});

// 16. Fluxo Síndico Contrata Empresa
createCanvas(path.join(dir, "02 - Domínios e Requisitos", "Fluxo Contratacao.canvas"), {
  nodes: [
    { id: "S1", text: "👔 Síndico\nCria Connect Me", x: 0, y: 0 },
    { id: "E1", text: "🏢 Empresa\nTenho interesse", x: 400, y: 0 },
    { id: "N1", text: "🤝 Negociação\nFora da plataforma", x: 800, y: 0 },
    
    { id: "S2", text: "✅ Validation Hub\nRegistra 'Negócio Fechado'", x: 800, y: 150 },
    { id: "E2", text: "🏢 Empresa\nEnvia Registro de Execução", x: 400, y: 150 },
    { id: "S3", text: "✅ Validation Hub\nValida Registro", x: 0, y: 150 },
    
    { id: "T1", text: "📜 Timeline\nPublica (Imutável)", x: 0, y: 300 },
    { id: "E3", text: "🏢 Empresa\nEnvia Comunicado Técnico", x: 400, y: 300 },
    { id: "S4", text: "✅ Validation Hub\nValida e Publica", x: 800, y: 300 }
  ],
  edges: [
    { from: "S1", to: "E1" },
    { from: "E1", to: "N1" },
    
    { from: "N1", to: "S2", fromSide: "bottom", toSide: "top" },
    { from: "S2", to: "E2", fromSide: "left", toSide: "right" },
    { from: "E2", to: "S3", fromSide: "left", toSide: "right" },
    
    { from: "S3", to: "T1", fromSide: "bottom", toSide: "top" },
    { from: "E2", to: "E3", fromSide: "bottom", toSide: "top" },
    { from: "E3", to: "S4", fromSide: "right", toSide: "left" }
  ]
});

// 17. Fluxo Assembleia ao Vivo
createCanvas(path.join(dir, "02 - Domínios e Requisitos", "Fluxo Assembleia Ao Vivo.canvas"), {
  nodes: [
    { id: "S1", text: "👔 Síndico\nConfigura assembleia", x: 0, y: 0 },
    { id: "A1", text: "📋 Administradora\nImporta dados/inadimplência", x: 400, y: 0 },
    { id: "M1", text: "🏠 Moradores\nCiência e Confirmação", x: 800, y: 0 },
    
    { id: "M2", text: "🏠 Moradores\nCheck-in Ao Vivo", x: 0, y: 150 },
    { id: "S2", text: "👔 Síndico\nInicia pauta/votação", x: 400, y: 150 },
    { id: "M3", text: "🏠 Moradores\nLevanta mão / Vota", x: 800, y: 150 },
    
    { id: "S3", text: "👔 Síndico\nHomologa/Encerra", x: 800, y: 300 },
    { id: "T1", text: "📜 Timeline\nPublica resultados", x: 400, y: 300 },
    { id: "R1", text: "📊 Sistema\nGera relatórios", x: 0, y: 300 }
  ],
  edges: [
    { from: "S1", to: "A1" },
    { from: "A1", to: "M1" },
    
    { from: "M1", to: "M2", fromSide: "bottom", toSide: "top", style: "dashed" },
    { from: "M2", to: "S2" },
    { from: "S2", to: "M3" },
    
    { from: "M3", to: "S3", fromSide: "bottom", toSide: "top" },
    { from: "S3", to: "T1", fromSide: "left", toSide: "right" },
    { from: "T1", to: "R1", fromSide: "left", toSide: "right" }
  ]
});

// 18. Fluxo Parceiro Vizinhanca
createCanvas(path.join(dir, "02 - Domínios e Requisitos", "Fluxo Vizinhanca.canvas"), {
  nodes: [
    { id: "PV1", text: "🏪 Parceiro\nCria promoção", x: 0, y: 0 },
    { id: "VZ1", text: "🏘️ Vizinhança\nDefine escopo", x: 400, y: 0 },
    
    { id: "M1", text: "🏠 Morador (Aberta)\nVê no feed", x: 800, y: -100 },
    { id: "M2", text: "🏠 Morador (Exclusiva)\nVê se do condomínio", x: 800, y: 100 },
    
    { id: "M3", text: "🏠 Morador\nAtiva promoção", x: 800, y: 250 },
    { id: "PV2", text: "🏪 Parceiro\nAtualiza métricas", x: 400, y: 250 },
    { id: "S1", text: "👔 Síndico\nIndica/Cria benefício", x: 0, y: 250 }
  ],
  edges: [
    { from: "PV1", to: "VZ1" },
    { from: "VZ1", to: "M1", fromSide: "top", toSide: "left" },
    { from: "VZ1", to: "M2", fromSide: "bottom", toSide: "left" },
    
    { from: "M2", to: "M3", fromSide: "bottom", toSide: "top" },
    { from: "M1", to: "M3", fromSide: "bottom", toSide: "top" },
    { from: "M3", to: "PV2", fromSide: "left", toSide: "right" },
    { from: "PV2", to: "S1", fromSide: "left", toSide: "right" }
  ]
});

// 19. Fluxo Registro Identidade
createCanvas(path.join(dir, "02 - Domínios e Requisitos", "Fluxo Registro.canvas"), {
  nodes: [
    { id: "U1", text: "Usuário\nPOST /v1/auth/register", x: 0, y: 0 },
    { id: "R1", text: "User Registry\nValida/Hash/Cria User", x: 400, y: 0 },
    { id: "E1", text: "Email Service\nEnvia verificação", x: 800, y: 0 },
    
    { id: "U2", text: "Usuário\nAcessa link email", x: 800, y: 150 },
    { id: "R2", text: "User Registry\nAtiva conta (Active)", x: 400, y: 150 },
    { id: "O1", text: "Onboarding\nFluxo por persona", x: 0, y: 150 }
  ],
  edges: [
    { from: "U1", to: "R1" },
    { from: "R1", to: "E1" },
    
    { from: "E1", to: "U2", fromSide: "bottom", toSide: "top" },
    { from: "U2", to: "R2", fromSide: "left", toSide: "right" },
    { from: "R2", to: "O1", fromSide: "left", toSide: "right" }
  ]
});

```
