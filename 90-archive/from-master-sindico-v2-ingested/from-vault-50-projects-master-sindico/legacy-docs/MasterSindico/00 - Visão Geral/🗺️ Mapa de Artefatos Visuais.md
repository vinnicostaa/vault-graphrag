
# 🗺️ Mapa de Artefatos Visuais (Canvas)

> **Breadcrumbs:** [[00 - Índice Master Síndico|🏠 Início]] > 📄 Mapa de Artefatos Visuais
> Este documento organiza sistematicamente todos os Canvas do projeto para facilitar a navegação.

---

## 🏛️ Macro-Arquitetura (Visão Geral)
*Estes artefatos definem a estrutura fundamental do ecossistema.*

- [[Arquitetura de Domínios.canvas|🏰 Arquitetura de Domínios]]: Visão High-Level dos 5 domínios principais.
- [[Modelo de Tenants.canvas|👥 Modelo de Tenants]]: Isolamento entre Condomínios e Empresas.
- [[Stack Tecnologica.canvas|🛠️ Stack Tecnológica]]: Definição de Go, SolidJS, Flutter e Infra.
- [[Master Canvas.canvas|🎯 Master Canvas]]: Visão geral do negócio Master Síndico.

---

## ⚙️ Arquitetura Técnica & Infra
*Detalhes de implementação e contratos de sistema.*

- [[System Architecture.canvas|🧱 System Architecture]]: Desenho técnico do Monolito Modular.
- [[Adapter Layer.canvas|🔌 Adapter Layer]]: Abstração de serviços externos (Mux, Stripe, SES).
- [[Arquitetura Horizontal.canvas|↔️ Arquitetura Horizontal]]: Comunicação entre contextos.

---

## 🔐 Domínios & Regras de Acesso
*Lógica de negócio e autorização.*

- [[Dominio Identidade.canvas|🆔 Domínio Identidade]]: Fluxos de IAM e Sessão.
- [[Modelo ABAC.canvas|🛡️ Modelo ABAC]]: Matriz de permissões por Atributo.
- [[Modelo Planos.canvas|💳 Modelo de Planos]]: Chave global de acesso (Trial/Base/Plus/Pro).

---

## 🔄 Fluxos de Processo (User Journeys)
*Diagramas de estado e sequências de ações.*

### Governança & Institucional
- [[Fluxo Governanca.canvas|📜 Fluxo de Governança]]: Ciclo da Timeline e Plano Diretor.
- [[Fluxo Registro.canvas|📝 Fluxo de Registro]]: Onboarding de unidades e moradores.

### Comercial & Marketplace
- [[Dominio Comercial.canvas|💼 Domínio Comercial]]: Visão geral das relações B2B.
- [[Fluxo Connect Me.canvas|🤝 Fluxo Connect Me]]: Marketplace unidirecional e LGPD.
- [[Fluxo Contratacao.canvas|📝 Fluxo de Contratação]]: Do interesse ao Negócio Fechado.
- [[Fluxo Ciclo Servico.canvas|🛠️ Ciclo de Serviço]]: Execução e validação técnica.

### Assembleia & Real-time
- [[Arquitetura Assembleia.canvas|🏟️ Arquitetura de Assembleia]]: Estrutura de pautas e quórum.
- [[Fluxo Assembleia Ao Vivo.canvas|📺 Assembleia Ao Vivo]]: Eventos WebSocket e Telão.

### Conteúdo & Vizinhança
- [[Dominio Conteudo.canvas|📚 Domínio Conteúdo]]: LMS e Biblioteca.
- [[Mapa Academia.canvas|🎓 Mapa da Academia]]: Trilhas de aprendizado.
- [[Visibilidade Videos.canvas|👁️ Visibilidade de Vídeos]]: Regras de paywall e janelas de acesso.
- [[Fluxo Vizinhanca.canvas|🏠 Fluxo Vizinhança]]: Parceiros locais e promoções.
- [[Jornadas Vizinhanca.canvas|🗺️ Jornadas da Vizinhança]]: Visão das 4 personas no bairro.

---

## 📂 Sugestão de Organização de Pastas
Para reduzir o ruído visual no explorador de arquivos, mova os arquivos para:

- `00 - Visão Geral/Canvas/` (Macro-arquitetura)
- `01 - Arquitetura de Sistema/Canvas/` (Técnico & Infra)
- `02 - Domínios e Requisitos/<Nome do Domínio>/Canvas/` (Fluxos específicos)

**Voltar para:** [[00 - Índice Master Síndico|Índice Principal]]
