
# 📂 Plano de Reorganização do Vault

> **Breadcrumbs:** [[00 - Índice Master Síndico|🏠 Início]] > 📄 Reorganização
> Para resolver o caos visual e sistemático, siga este plano de pastas.

---

## 🏛️ 1. Nova Estrutura de Pastas

Mova os arquivos existentes para esta hierarquia:

### `00 - Central Hub/`
*O cérebro do projeto.*
- `00 - Índice Master Síndico.md`
- `🗺️ Mapa de Artefatos Visuais.md`
- `Roadmap 2026.md`
- `Arquitetura de Domínios.canvas`
- `Modelo de Tenants.canvas`
- `Stack Tecnologica.canvas`

### `01 - Infrastructure/`
*O alicerce técnico (Go Edition).*
- `System Architecture.md`
- `System Architecture - Infrastructure.md`
- `System Architecture - API Design.md`
- `Adapter Layer.canvas`
- `System Architecture.canvas`

### `02 - Business Contexts/`
*Um subfolder para cada Vertical Slice.*
- `01 - Identity/`
    - `Domínio - Identidade.md`
    - `Modelo ABAC.canvas`
    - `Modelo Planos.canvas`
    - `Dominio Identidade.canvas`
- `02 - Governance/`
    - `Domínio - Institucional.md`
    - `Fluxo Governanca.canvas`
    - `Fluxo Registro.canvas`
- `03 - Marketplace/`
    - `Domínio - Comercial.md`
    - `Dominio Comercial.canvas`
    - `Fluxo Connect Me.canvas`
    - `Fluxo Contratacao.canvas`
    - `Fluxo Ciclo Servico.canvas`
- `04 - Content/`
    - `Domínio - Conteúdo.md`
    - `Dominio Conteudo.canvas`
    - `Mapa Academia.canvas`
    - `Visibilidade Videos.canvas`
- `05 - Assembly/`
    - `Domínio - Assembleia.md`
    - `Arquitetura Assembleia.canvas`
    - `Fluxo Assembleia Ao Vivo.canvas`

### `03 - Data Model/`
*Onde vive o schema.*
- `Enums e Listas Mestre.md`
- `System Architecture - Data Model.md`

### `04 - Execution/`
*Gestão do dia a dia.*
- `Definition of Done.md`
- `Sprint 1 Identity.md`
- `05 - Entregas/` (Manter esta pasta aqui)

---

## 🛠️ 2. Próximos Passos para o Arquiteto
1. [ ] Mover os arquivos conforme a lista acima.
2. [ ] Validar se os links `[[link]]` continuam funcionando (Obsidian costuma atualizar automático).
3. [ ] Usar o [[🗺️ Mapa de Artefatos Visuais]] para acessar os Canvas em vez de procurá-los na barra lateral.

**Voltar para:** [[00 - Índice Master Síndico|Índice Principal]]
