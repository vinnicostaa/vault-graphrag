---
title: "generate (js)"
type: source
tags: [code, js, converted]
source: 50-projects/master-sindico/client-vault/generate.js
converted: 2026-04-22
---

# generate

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
      height: n.height || 100,
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
      label: e.label
    });
  }

  const output = { nodes, edges };
  fs.writeFileSync(filename, JSON.stringify(output, null, 2));
  console.log(`Created ${filename}`);
}

const dir = "/home/vinni/Documentos/Obsidian Vault/Clients/MasterSindico";

// 1. Modelo de Tenants
createCanvas(path.join(dir, "00 - Visão Geral", "Modelo de Tenants.canvas"), {
  nodes: [
    { id: "g_tc", type: "group", label: "Tenant: Condomínio", x: 0, y: 0, width: 300, height: 350, color: "1" },
    { id: "S", text: "👔 Síndico\naté 15 condos", x: 20, y: 50, width: 260, height: 80 },
    { id: "M", text: "🏠 Morador\ntitular + 2 dep.", x: 20, y: 150, width: 260, height: 80 },
    { id: "ES", text: "🏢 Empresa Síndica\ndelegação parametrizável", x: 20, y: 250, width: 260, height: 80 },
    { id: "g_te", type: "group", label: "Tenant: Empresa", x: 500, y: 0, width: 300, height: 250, color: "2" },
    { id: "EA", text: "👤 Administrador", x: 520, y: 50, width: 260, height: 80 },
    { id: "EO", text: "🔧 Operador Técnico", x: 520, y: 150, width: 260, height: 80 },
    { id: "g_ad", type: "group", label: "Actor Delegado", x: 1000, y: 0, width: 300, height: 150, color: "3" },
    { id: "AG", text: "📸 Agência Marketing\nnão é tenant", x: 1020, y: 50, width: 260, height: 80 },
    { id: "g_ei", type: "group", label: "Entidade Independente", x: 500, y: 300, width: 300, height: 150, color: "4" },
    { id: "PV", text: "🏪 Parceiro Vizinhança\nlogin próprio · CPF", x: 520, y: 350, width: 260, height: 80 }
  ],
  edges: [
    { from: "S", to: "M", label: "gere", fromSide: "bottom", toSide: "top" },
    { from: "ES", to: "S", label: "delegação", fromSide: "top", toSide: "bottom" },
    { from: "EA", to: "AG", label: "convida", fromSide: "right", toSide: "left" },
    { from: "S", to: "EA", label: "Connect Me", fromSide: "right", toSide: "left" },
    { from: "M", to: "S", label: "Connect Me", fromSide: "top", toSide: "bottom" },
    { from: "PV", to: "M", label: "promoções", fromSide: "left", toSide: "right" }
  ]
});

// 2. Stack Tecnológica
createCanvas(path.join(dir, "00 - Visão Geral", "Stack Tecnologica.canvas"), {
  nodes: [
    { id: "g_front", type: "group", label: "Frontend", x: 0, y: 0, width: 300, height: 450, color: "1" },
    { id: "SOLID", text: "SolidJS", x: 20, y: 50, width: 260, height: 80 },
    { id: "TANSTACK", text: "TanStack Router + Query + Form", x: 20, y: 150, width: 260, height: 80 },
    { id: "UNO", text: "UnoCSS", x: 20, y: 250, width: 260, height: 80 },
    { id: "AXIOS", text: "Axios", x: 20, y: 350, width: 260, height: 80 },
    { id: "g_back", type: "group", label: "Backend", x: 400, y: 0, width: 300, height: 550, color: "2" },
    { id: "FASTIFY", text: "Fastify", x: 420, y: 50, width: 260, height: 80 },
    { id: "DRIZZLE", text: "Drizzle ORM", x: 420, y: 150, width: 260, height: 80 },
    { id: "OSO", text: "Oso + jose (JWT)", x: 420, y: 250, width: 260, height: 80 },
    { id: "AWILIX", text: "Awilix (DI)", x: 420, y: 350, width: 260, height: 80 },
    { id: "ZOD", text: "Zod + drizzle-zod", x: 420, y: 450, width: 260, height: 80 },
    { id: "g_infra", type: "group", label: "Infraestrutura", x: 800, y: 0, width: 300, height: 350, color: "3" },
    { id: "PG", text: "PostgreSQL", x: 820, y: 50, width: 260, height: 80 },
    { id: "WS", text: "WebSocket (Assembleia)", x: 820, y: 150, width: 260, height: 80 },
    { id: "S3", text: "Object Storage", x: 820, y: 250, width: 260, height: 80 },
    { id: "g_ext", type: "group", label: "Serviços Externos", x: 1200, y: 0, width: 300, height: 650, color: "4" },
    { id: "MUX", text: "Mux / Cloudflare Stream", x: 1220, y: 50, width: 260, height: 80 },
    { id: "MP", text: "Mercado Pago", x: 1220, y: 150, width: 260, height: 80 },
    { id: "SES", text: "AWS SES / SendGrid", x: 1220, y: 250, width: 260, height: 80 },
    { id: "SMS", text: "Twilio / Zenvia", x: 1220, y: 350, width: 260, height: 80 },
    { id: "CEP", text: "ViaCEP", x: 1220, y: 450, width: 260, height: 80 },
    { id: "OAUTH", text: "Google / Apple OAuth", x: 1220, y: 550, width: 260, height: 80 }
  ],
  edges: [
    { from: "SOLID", to: "FASTIFY", fromSide: "right", toSide: "left" },
    { from: "FASTIFY", to: "PG", fromSide: "right", toSide: "left" },
    { from: "FASTIFY", to: "WS", fromSide: "right", toSide: "left" },
    { from: "FASTIFY", to: "S3", fromSide: "right", toSide: "left" },
    { from: "FASTIFY", to: "MUX", label: "Adapter Layer", fromSide: "right", toSide: "left" },
    { from: "FASTIFY", to: "MP", label: "Adapter Layer", fromSide: "right", toSide: "left" },
    { from: "FASTIFY", to: "SES", label: "Adapter Layer", fromSide: "right", toSide: "left" },
    { from: "FASTIFY", to: "SMS", label: "Adapter Layer", fromSide: "right", toSide: "left" },
    { from: "FASTIFY", to: "CEP", label: "Adapter Layer", fromSide: "right", toSide: "left" },
    { from: "FASTIFY", to: "OAUTH", label: "Adapter Layer", fromSide: "right", toSide: "left" }
  ]
});

// 3. Arquitetura Horizontal
createCanvas(path.join(dir, "02 - Domínios e Requisitos", "Arquitetura Horizontal.canvas"), {
  nodes: [
    { id: "g_cross", type: "group", label: "🛡️ SERVIÇOS CROSS-DOMAIN", x: 0, y: 0, width: 400, height: 500, color: "1" },
    { id: "COMP", text: "Governança & Compliance\n(Dashboard, Matriz, LGPD)", x: 20, y: 50, width: 360, height: 80 },
    { id: "AUDIT", text: "Trilha de Auditoria\n(Append-only)", x: 20, y: 150, width: 360, height: 80 },
    { id: "DASH", text: "Indicators & Analytics\n(Síndico, Empresa, Agência)", x: 20, y: 250, width: 360, height: 80 },
    { id: "SEC", text: "Segurança\n(Fingerprint, Anti-Fraud)", x: 20, y: 350, width: 360, height: 80 },
    { id: "g_ext", type: "group", label: "🔌 INTEGRAÇÕES (Adapter Layer)", x: 600, y: 0, width: 300, height: 650, color: "2" },
    { id: "MP", text: "Mercado Pago\nBilling", x: 620, y: 50, width: 260, height: 80 },
    { id: "SES", text: "AWS SES / SendGrid\nEmails", x: 620, y: 150, width: 260, height: 80 },
    { id: "MUX", text: "Mux / Cloudflare\nVideo Streaming", x: 620, y: 250, width: 260, height: 80 },
    { id: "S3", text: "AWS S3\nStorage", x: 620, y: 350, width: 260, height: 80 },
    { id: "FIRE", text: "Firebase\nPush Notifications", x: 620, y: 450, width: 260, height: 80 },
    { id: "CEP", text: "ViaCEP\nGeocoding", x: 620, y: 550, width: 260, height: 80 }
  ],
  edges: [
    { from: "COMP", to: "AUDIT", fromSide: "bottom", toSide: "top" },
    { from: "DASH", to: "AUDIT", label: "consome dados", fromSide: "top", toSide: "bottom" },
    { from: "SEC", to: "COMP", fromSide: "top", toSide: "bottom" },
    { from: "g_cross", to: "g_ext", fromSide: "right", toSide: "left" }
  ]
});

// 4. Arquitetura de Assembleia
createCanvas(path.join(dir, "02 - Domínios e Requisitos", "Arquitetura Assembleia.canvas"), {
  nodes: [
    { id: "g_pre", type: "group", label: "⏳ Pré-Assembleia", x: 0, y: 0, width: 300, height: 450, color: "1" },
    { id: "CONF", text: "Configuração & Pauta\nLock de Edição", x: 20, y: 50, width: 260, height: 80 },
    { id: "EDITAL", text: "Edital Automático\nPDF", x: 20, y: 150, width: 260, height: 80 },
    { id: "CIENCIA", text: "Registro de Ciência\n(Bloqueia App)", x: 20, y: 250, width: 260, height: 80 },
    { id: "PROX", text: "Procurações &\nSimulador Quórum", x: 20, y: 350, width: 260, height: 80 },
    { id: "g_live", type: "group", label: "🔴 Ao Vivo", x: 400, y: 0, width: 300, height: 550, color: "2" },
    { id: "CHECK", text: "Check-in\nApp/Terminal/QR", x: 420, y: 50, width: 260, height: 80 },
    { id: "PRES", text: "Painel do Presidente\nControle de Fluxo", x: 420, y: 150, width: 260, height: 80 },
    { id: "TELAO", text: "Telão (Big Screen)\nApresentação + Operação", x: 420, y: 450, width: 260, height: 80 },
    { id: "VOZ", text: "Fila de Fala\nTimer: 2/3 min", x: 420, y: 350, width: 260, height: 80 },
    { id: "VOTO", text: "Votação Tempo Real\nPeso Fracionado", x: 420, y: 250, width: 260, height: 80 },
    { id: "g_post", type: "group", label: "📜 Pós-Assembleia", x: 800, y: 0, width: 300, height: 350, color: "3" },
    { id: "HOMOLOG", text: "Homologação\nLock Definitivo", x: 820, y: 50, width: 260, height: 80 },
    { id: "ATA", text: "Consolidação\nDados p/ Ata", x: 820, y: 150, width: 260, height: 80 },
    { id: "AUDIT", text: "Trilha de Auditoria\nTransparência", x: 820, y: 250, width: 260, height: 80 }
  ],
  edges: [
    { from: "CONF", to: "EDITAL", fromSide: "bottom", toSide: "top" },
    { from: "EDITAL", to: "CIENCIA", fromSide: "bottom", toSide: "top" },
    { from: "PROX", to: "g_live", fromSide: "right", toSide: "left" },
    { from: "CIENCIA", to: "g_live", fromSide: "right", toSide: "left" },
    { from: "CHECK", to: "PRES", fromSide: "bottom", toSide: "top" },
    { from: "PRES", to: "VOTO", fromSide: "bottom", toSide: "top" },
    { from: "VOTO", to: "TELAO", fromSide: "bottom", toSide: "top" },
    { from: "VOZ", to: "TELAO", fromSide: "bottom", toSide: "top" },
    { from: "PRES", to: "HOMOLOG", fromSide: "right", toSide: "left" },
    { from: "HOMOLOG", to: "ATA", fromSide: "bottom", toSide: "top" },
    { from: "ATA", to: "AUDIT", fromSide: "bottom", toSide: "top" }
  ]
});

// 5. Visibilidade de Videos
createCanvas(path.join(dir, "02 - Domínios e Requisitos", "Visibilidade Videos.canvas"), {
  nodes: [
    { id: "E", text: "🏢 Empresa\nfaz upload", x: 0, y: 100, width: 200, height: 80 },
    { id: "V", text: "📹 Vídeo Institucional", x: 300, y: 100, width: 200, height: 80 },
    { id: "S", text: "👔 Síndico", x: 700, y: 0, width: 200, height: 80 },
    { id: "M", text: "🏠 Morador", x: 700, y: 200, width: 200, height: 80 }
  ],
  edges: [
    { from: "E", to: "V", label: "upload", fromSide: "right", toSide: "left" },
    { from: "V", to: "S", label: "N1/N2: 25% preview", fromSide: "right", toSide: "left" },
    { from: "V", to: "S", label: "N3/Plus/Pro: completo", fromSide: "right", toSide: "left" },
    { from: "V", to: "M", label: "autorizar_exibicao = true\n+ votação ativa", fromSide: "right", toSide: "left" },
    { from: "M", to: "V", label: "BLOQUEADO\napós votação", fromSide: "left", toSide: "right", style: "dashed" }
  ]
});

// 6. Mapa da Academia
createCanvas(path.join(dir, "02 - Domínios e Requisitos", "Mapa Academia.canvas"), {
  nodes: [
    { id: "g_lms", type: "group", label: "🎓 ACADEMIA MASTER SÍNDICO", x: 400, y: 0, width: 300, height: 550, color: "1" },
    { id: "CURSOS", text: "📖 Cursos\nMódulos · Aulas", x: 420, y: 50, width: 260, height: 80 },
    { id: "TREINA", text: "🏋️ Treinamentos\nCurtos · Práticos", x: 420, y: 150, width: 260, height: 80 },
    { id: "LIVES", text: "📺 Lives\nAgenda · Chat", x: 420, y: 250, width: 260, height: 80 },
    { id: "FORUM", text: "💬 Fórum\n7 categorias", x: 420, y: 350, width: 260, height: 80 },
    { id: "BIBLIO", text: "📚 Biblioteca\n6 tipos", x: 420, y: 450, width: 260, height: 80 },
    { id: "TRILHA_S", text: "Trilha Síndico", x: 0, y: 100, width: 260, height: 80 },
    { id: "TRILHA_E", text: "Trilha Empresa", x: 0, y: 200, width: 260, height: 80 },
    { id: "TRILHA_M", text: "Trilha Morador", x: 0, y: 300, width: 260, height: 80 },
    { id: "CERT", text: "📜 Certificados", x: 800, y: 100, width: 260, height: 80 },
    { id: "REPLAY", text: "Replay", x: 800, y: 250, width: 260, height: 80 }
  ],
  edges: [
    { from: "TRILHA_S", to: "CURSOS", fromSide: "right", toSide: "left" },
    { from: "TRILHA_S", to: "TREINA", fromSide: "right", toSide: "left" },
    { from: "TRILHA_E", to: "CURSOS", fromSide: "right", toSide: "left" },
    { from: "TRILHA_E", to: "TREINA", fromSide: "right", toSide: "left" },
    { from: "TRILHA_M", to: "CURSOS", fromSide: "right", toSide: "left" },
    { from: "TRILHA_M", to: "LIVES", fromSide: "right", toSide: "left" },
    { from: "CURSOS", to: "CERT", fromSide: "right", toSide: "left" },
    { from: "TREINA", to: "CERT", fromSide: "right", toSide: "left" },
    { from: "LIVES", to: "REPLAY", fromSide: "right", toSide: "left" }
  ]
});

// 7. Visão do Domínio (Conteúdo)
createCanvas(path.join(dir, "02 - Domínios e Requisitos", "Dominio Conteudo.canvas"), {
  nodes: [
    { id: "g_cont", type: "group", label: "📚 CONTEÚDO", x: 0, y: 0, width: 300, height: 450, color: "1" },
    { id: "VIDEO", text: "Vídeos Institucionais\nUpload · Player", x: 20, y: 50, width: 260, height: 80 },
    { id: "AGENCY", text: "Agência Marketing\nGestão delegada", x: 20, y: 150, width: 260, height: 80 },
    { id: "SEARCH", text: "Busca & Discovery\nFull-text", x: 20, y: 250, width: 260, height: 80 },
    { id: "LMS", text: "Academia LMS\n5 áreas", x: 20, y: 350, width: 260, height: 80 },
    { id: "GOV", text: "📜 Governança", x: 400, y: 350, width: 260, height: 80 }
  ],
  edges: [
    { from: "VIDEO", to: "SEARCH", label: "conteúdo para", fromSide: "bottom", toSide: "top" },
    { from: "AGENCY", to: "VIDEO", label: "gerencia", fromSide: "top", toSide: "bottom" },
    { from: "SEARCH", to: "VIDEO", label: "descobre", fromSide: "top", toSide: "bottom" },
    { from: "SEARCH", to: "LMS", label: "descobre", fromSide: "bottom", toSide: "top" },
    { from: "LMS", to: "GOV", label: "integra", fromSide: "right", toSide: "left" }
  ]
});

// 8. Visão do Domínio (Comercial)
createCanvas(path.join(dir, "02 - Domínios e Requisitos", "Dominio Comercial.canvas"), {
  nodes: [
    { id: "g_com", type: "group", label: "💼 COMERCIAL", x: 0, y: 0, width: 300, height: 750, color: "1" },
    { id: "REG", text: "Registro Empresa\nCNPJ · Perfil", x: 20, y: 50, width: 260, height: 80 },
    { id: "CM", text: "Connect Me\n4 direções", x: 20, y: 150, width: 260, height: 80 },
    { id: "PROP", text: "Propostas\nEnvio · Validação", x: 20, y: 250, width: 260, height: 80 },
    { id: "DEAL", text: "Negócio Fechado\nDupla confirmação", x: 20, y: 350, width: 260, height: 80 },
    { id: "EXEC", text: "Execução de Serviço\nRegistro · Validação", x: 20, y: 450, width: 260, height: 80 },
    { id: "EVAL", text: "Avaliação\n5 critérios", x: 20, y: 550, width: 260, height: 80 },
    { id: "VIZ", text: "Vizinhança\nParceiros · Promoções", x: 20, y: 650, width: 260, height: 80 },
    { id: "TIMELINE", text: "📜 Timeline", x: 400, y: 450, width: 260, height: 80 }
  ],
  edges: [
    { from: "CM", to: "PROP", label: "oportunidade aceita", fromSide: "bottom", toSide: "top" },
    { from: "PROP", to: "DEAL", label: "proposta aceita", fromSide: "bottom", toSide: "top" },
    { from: "DEAL", to: "EXEC", label: "serviço prestado", fromSide: "bottom", toSide: "top" },
    { from: "EXEC", to: "TIMELINE", label: "validado pelo síndico", fromSide: "right", toSide: "left" },
    { from: "DEAL", to: "EVAL", label: "concluído", fromSide: "bottom", toSide: "top" },
    { from: "REG", to: "CM", label: "perfil visível", fromSide: "bottom", toSide: "top" }
  ]
});

// 9. Fluxo Completo (Comercial)
createCanvas(path.join(dir, "02 - Domínios e Requisitos", "Fluxo Ciclo Servico.canvas"), {
  nodes: [
    { id: "CM", text: "Connect Me\nRequisição", x: 0, y: 150, width: 200, height: 80 },
    { id: "PROP", text: "Proposta\nReq 20", x: 300, y: 150, width: 200, height: 80 },
    { id: "VOTE", text: "Votação Fornecedor\nReq 21", x: 600, y: 150, width: 200, height: 80 },
    { id: "DEAL", text: "Negócio Fechado\nReq 22", x: 900, y: 150, width: 200, height: 80 },
    { id: "EXEC", text: "Registro Execução\nReq 23", x: 1200, y: 0, width: 200, height: 80 },
    { id: "TECH", text: "Atividade Técnica\nReq 23.1", x: 1200, y: 100, width: 200, height: 80 },
    { id: "COM", text: "Comunicado Pós-Deal\nReq 24", x: 1200, y: 200, width: 200, height: 80 },
    { id: "TL", text: "Timeline", x: 1500, y: 100, width: 200, height: 80 },
    { id: "ADENDO", text: "Adendo\nReq 25", x: 1800, y: 100, width: 200, height: 80 },
    { id: "EVAL", text: "Avaliação\nReq 26", x: 1200, y: 300, width: 200, height: 80 }
  ],
  edges: [
    { from: "CM", to: "PROP", fromSide: "right", toSide: "left" },
    { from: "PROP", to: "VOTE", fromSide: "right", toSide: "left" },
    { from: "VOTE", to: "DEAL", fromSide: "right", toSide: "left" },
    { from: "DEAL", to: "EXEC", fromSide: "right", toSide: "left" },
    { from: "DEAL", to: "TECH", fromSide: "right", toSide: "left" },
    { from: "DEAL", to: "COM", fromSide: "right", toSide: "left" },
    { from: "EXEC", to: "TL", label: "validação síndico", fromSide: "right", toSide: "left" },
    { from: "TECH", to: "TL", label: "validação síndico", fromSide: "right", toSide: "left" },
    { from: "COM", to: "TL", label: "validação síndico", fromSide: "right", toSide: "left" },
    { from: "TL", to: "ADENDO", label: "imutável", fromSide: "right", toSide: "left" },
    { from: "DEAL", to: "EVAL", label: "concluído", fromSide: "right", toSide: "left" }
  ]
});

// 10. Mapa de Jornadas (Vizinhança)
createCanvas(path.join(dir, "02 - Domínios e Requisitos", "Jornadas Vizinhanca.canvas"), {
  nodes: [
    { id: "g_viz", type: "group", label: "🏘️ VIZINHANÇA", x: 400, y: 0, width: 300, height: 450, color: "1" },
    { id: "VS", text: "Jornada Síndico\nVS1-VS10", x: 420, y: 50, width: 260, height: 80 },
    { id: "VE", text: "Jornada Empresa\nVE1-VE6", x: 420, y: 150, width: 260, height: 80 },
    { id: "VZ", text: "Jornada Parceiro\nVZ1-VZ6", x: 420, y: 250, width: 260, height: 80 },
    { id: "VM", text: "Jornada Morador\nVM1-VM7", x: 420, y: 350, width: 260, height: 80 }
  ],
  edges: [
    { from: "VZ", to: "VS", label: "cria promoção", fromSide: "left", toSide: "left" },
    { from: "VZ", to: "VM", label: "cria promoção", fromSide: "left", toSide: "left" },
    { from: "VS", to: "VZ", label: "indica parceiro", fromSide: "right", toSide: "right" },
    { from: "VS", to: "VM", label: "cria benefício exclusivo", fromSide: "right", toSide: "right" },
    { from: "VE", to: "VZ", label: "ativa promoção", fromSide: "left", toSide: "left" },
    { from: "VM", to: "VZ", label: "ativa promoção", fromSide: "right", toSide: "right" }
  ]
});

// 11. Visão do Domínio (Identidade)
createCanvas(path.join(dir, "02 - Domínios e Requisitos", "Dominio Identidade.canvas"), {
  nodes: [
    { id: "g_id", type: "group", label: "🔐 IDENTIDADE", x: 0, y: 0, width: 300, height: 750, color: "1" },
    { id: "AUTH", text: "Autenticação\nJWT · OAuth", x: 20, y: 50, width: 260, height: 80 },
    { id: "ABAC", text: "Autorização\nABAC · Roles", x: 20, y: 150, width: 260, height: 80 },
    { id: "REG", text: "Registro\n4 tipos de usuário", x: 20, y: 250, width: 260, height: 80 },
    { id: "BILLING", text: "Billing\nTrial · Base · Plus · Pro", x: 20, y: 350, width: 260, height: 80 },
    { id: "ONBOARD", text: "Onboarding\n4-7 telas por persona", x: 20, y: 450, width: 260, height: 80 },
    { id: "FEATURE", text: "Feature Access\nMatriz de features", x: 20, y: 550, width: 260, height: 80 },
    { id: "PROFILE", text: "Perfis\nSíndico · Empresa", x: 20, y: 650, width: 260, height: 80 }
  ],
  edges: [
    { from: "AUTH", to: "ABAC", fromSide: "bottom", toSide: "top" },
    { from: "ABAC", to: "FEATURE", fromSide: "bottom", toSide: "top" },
    { from: "REG", to: "ONBOARD", fromSide: "bottom", toSide: "top" },
    { from: "ONBOARD", to: "PROFILE", fromSide: "bottom", toSide: "top" },
    { from: "BILLING", to: "FEATURE", fromSide: "bottom", toSide: "top" }
  ]
});

// 12. Modelo ABAC
createCanvas(path.join(dir, "02 - Domínios e Requisitos", "Modelo ABAC.canvas"), {
  nodes: [
    { id: "U", text: "Usuário\nuserId · role · plan", x: 0, y: 0, width: 200, height: 80 },
    { id: "T", text: "Tenant\ntenantId · type", x: 0, y: 100, width: 200, height: 80 },
    { id: "A", text: "Ação\naction · resource", x: 0, y: 200, width: 200, height: 80 },
    { id: "C", text: "Contexto\nquota · time", x: 0, y: 300, width: 200, height: 80 },
    { id: "P", text: "Policy Engine\nOso", x: 400, y: 150, width: 200, height: 80 },
    { id: "R", text: "Resultado\nallowed · features", x: 800, y: 150, width: 200, height: 80 }
  ],
  edges: [
    { from: "U", to: "P", fromSide: "right", toSide: "left" },
    { from: "T", to: "P", fromSide: "right", toSide: "left" },
    { from: "A", to: "P", fromSide: "right", toSide: "left" },
    { from: "C", to: "P", fromSide: "right", toSide: "left" },
    { from: "P", to: "R", fromSide: "right", toSide: "left" }
  ]
});

// 13. Modelo de Planos
createCanvas(path.join(dir, "02 - Domínios e Requisitos", "Modelo Planos.canvas"), {
  nodes: [
    { id: "T", text: "Trial\n15d/7d/30d", x: 0, y: 150, width: 200, height: 80 },
    { id: "B", text: "Base\nGratuito", x: 400, y: 0, width: 200, height: 80 },
    { id: "PLUS", text: "Plus\nMensal/Anual", x: 400, y: 150, width: 200, height: 80 },
    { id: "PRO", text: "Pro\nMensal/Anual", x: 400, y: 300, width: 200, height: 80 }
  ],
  edges: [
    { from: "T", to: "B", label: "expira sem pagamento", fromSide: "right", toSide: "left" },
    { from: "T", to: "PLUS", label: "paga", fromSide: "right", toSide: "left" },
    { from: "T", to: "PRO", label: "paga", fromSide: "right", toSide: "left" },
    { from: "B", to: "PLUS", label: "upgrade", fromSide: "bottom", toSide: "top" },
    { from: "B", to: "PRO", label: "upgrade", fromSide: "bottom", toSide: "top" },
    { from: "PLUS", to: "PRO", label: "upgrade", fromSide: "bottom", toSide: "top" },
    { from: "PRO", to: "PLUS", label: "downgrade", fromSide: "top", toSide: "bottom" },
    { from: "PLUS", to: "B", label: "downgrade", fromSide: "top", toSide: "bottom" },
    { from: "PRO", to: "B", label: "expira", fromSide: "left", toSide: "bottom" },
    { from: "PLUS", to: "B", label: "expira", fromSide: "left", toSide: "bottom" }
  ]
});

console.log("All canvas files created!");

```
