# ✅ Definition of Done (DoD)

> **Breadcrumbs:** [[00 - Índice Master Síndico|🏠 Início]] > 📄 DoD
> Checklist sistemático para garantir a qualidade de cada Vertical Slice em Go.

---

## 🛠️ Checklist Técnico (Backend Go)
- [ ] **Interfaces:** Ports definidas em `/internal/domain`.
- [ ] **Repository:** Implementação Ent/GORM validada.
- [ ] **UseCase:** Regra de negócio isolada e testada.
- [ ] **Transport:** Handler Gin documentado via Swagger.
- [ ] **Security:** Permissão Casbin ABAC validada no middleware.
- [ ] **Audit:** Log de auditoria gerado para ações críticas.

---

## 💻 Checklist de Interface (SolidJS & Flutter)
- [ ] **Design Tokens:** Uso correto de cores (UnoCSS) e componentes (Kobalte).
- [ ] **Responsividade:** Web testada em Desktop e Mobile.
- [ ] **Mobile Parity:** Tela em Flutter espelhando 1:1 a funcionalidade Web.
- [ ] **Validation:** Validação de entrada via Struct Tags (Back) e Forms (Front).

---

## 🔄 Checklist de Integração & Infra
- [ ] **Events:** Evento publicado no NATS JetStream (se aplicável).
- [ ] **Search:** Ingestão de dados no OpenSearch configurada.
- [ ] **Storage:** Upload de mídia via S3/R2 Adapter testado.
- [ ] **Monitoring:** Métricas base coletadas.

---

## 👥 Validação Humana
- [ ] **Stakeholder:** João (Master Síndico) aprovou o fluxo da funcionalidade.
- [ ] **QA:** Caminhos de erro testados (Ex: tentar burlar quota do Connect Me).

**Voltar para:** [[00 - Índice Master Síndico|Portal do Arquiteto]]
