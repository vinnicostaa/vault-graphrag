---
title: Design Assets — Figma, tokens, identidade visual
version: 1.0
status: stable
type: design
domain: cross-domain
tags: [frontend, design, figma, tokens]
---

# Design Assets

Referência central para assets visuais do Master Síndico. Fonte autoritativa para o frontend SolidJS e mobile Flutter.

## Figma — Arquivo oficial

**URL**: https://www.figma.com/design/pxJkmn8rzezKnCYXcA7lyL/Master-sindico-APP---UX?t=WWlSkgKIWW4HJf8S-0

**Nome do arquivo**: Master-sindico-APP---UX

**Quando usar**:
- Implementação de nova tela do frontend (ver frames do design)
- Extrair design tokens (cores, tipografia, spacing) — se não estiverem na seção "Design System" abaixo
- Validar design system do SolidJS vs o que está no Figma
- Exportar assets (ícones SVG, imagens)

**Como acessar via agente**:

1. **Via plugin Figma** (quando instalado no Sprint 1.7+):
   ```
   /plugin install figma@claude-plugins-official
   # Plugin pede FIGMA_ACCESS_TOKEN na primeira execução
   # Após: /implement-design, /create-design-system-rules, /code-connect-components
   ```

2. **Via MCP server do Figma**: se preferir MCP direto, config em `.mcp.json`:
   ```json
   "figma": {
     "command": "npx",
     "args": ["-y", "@figma/mcp-server@latest"],
     "env": { "FIGMA_ACCESS_TOKEN": "${FIGMA_ACCESS_TOKEN}" }
   }
   ```

**Credenciais**:
- `FIGMA_ACCESS_TOKEN` em `backend/.env` (no `.gitignore`). Gerar em [Figma Settings → Personal access tokens](https://www.figma.com/settings).

**Permissões do token**:
- File Content: read (para ler frames e componentes)
- Current User: read
- **Não** dar permissões de write — evitar que agente modifique arquivo Figma sem aprovação humana

## Design System (canônico — definido no `design.md`)

Paleta:
```css
--primary:    oklch(0.715 0.120 84.2)   /* Gold CTA */
--secondary:  oklch(0.218 0.055 256.1)  /* Navy */
--accent:     oklch(0.871 0.129 82.0)   /* Gold Light */
--background: oklch(0.976 0.003 264.5)  /* Light mode */
--background: oklch(0.183 0.031 263.4)  /* Dark mode */
```

Tipografia:
- **Manrope** (body) — regular, medium, bold
- **Michroma** (headings) — `letter-spacing: 0.05em`

Componentes base: **Kobalte** (headless, acessíveis) + **UnoCSS** (utilitários + tokens).

**Regra dura**: paleta não muda sem aprovação explícita do João. Se Figma tiver cor diferente, o spec aqui vence até o João confirmar mudança.

## Padrões UX (extraídos do plugin `ui-ux-pro-max` + Figma)

Quando `ui-ux-pro-max` skill estiver instalado, usar para:
- Validar contraste 4.5:1 em todo texto
- Validar responsividade (mobile-first)
- Validar focus states em todos os interactivos
- Validar `cursor-pointer` em botões
- Validar `prefers-reduced-motion` respeitado

## Workflow de tradução design → código

1. Abrir frame no Figma (URL acima)
2. Invocar plugin Figma ou MCP: `/implement-design <frame_id>`
3. Gerar componente SolidJS base com UnoCSS
4. **Substituir** quaisquer tokens hex/rgb por variáveis do design system (cores acima)
5. **Substituir** bibliotecas de componentes geradas por Kobalte (padrão do projeto)
6. Validar acessibilidade (4.5:1, focus, ARIA) — via `security-guidance` plugin + ui-ux-pro-max
7. Testar em mobile viewport (Rsbuild dev server)
8. PR com screenshots antes/depois

## Assets estáticos

- Logos e identidade: exportar de Figma, armazenar em `web/packages/ui/src/assets/brand/`
- Ícones: preferir [Tabler Icons](https://tabler-icons.io/) via `@unocss/preset-icons` (já configurado em `uno.config.ts` de cada app)
- Ilustrações custom: exportar SVG do Figma, otimizar via SVGO, armazenar em `web/packages/ui/src/assets/illustrations/`

## Manutenção deste arquivo

- URL do Figma muda: atualizar aqui + notificar João
- Paleta/tipografia mudam: atualizar aqui + `design.md` (monolítico) + todos os `uno.config.ts` dos apps
- Novos componentes Kobalte: atualizar `web/packages/ui/` + referência aqui

## Referências

- `web/AGENTS.md` — padrões de frontend (fora do escopo D1 desta reengenharia; atualizará quando frontend entrar)
- `.kiro/steering/plugins.md` seção Figma
- `.kiro/steering/mcp-integration.md` para Figma MCP (se adotar ao invés do plugin)
