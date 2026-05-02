---
title: "LMS5 — Conclusão do Curso + Certificado"
type: ui-screen
tags: [master-sindico, ui-catalog, web, lms, academia, certificado]
project: master-sindico
persona: any
category: LMS
screen_id: LMS5
sub_produto: lms-academia
plan_requirement: plus
status: specification
stack: web
created: 2026-04-23
---

# LMS5 — Conclusão do Curso + Certificado

## Finalidade
Tela celebrativa após conclusão de 100% do curso. Mostra mensagem institucional + botões para baixar e compartilhar o certificado em PDF. Geração assíncrona com `serialNumber` único e QR de verificação pública.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/contents/MÓDULO ACADEMIA MASTER SÍNDICO.md` §LMS5

## Mensagem canônica (literal)
> "Parabéns por concluir este curso da Academia Master Síndico."

## Persona e ABAC
- **Personas elegíveis**: Síndico (`plus`+; **trial NÃO recebe — Q5 aberta**), Empresa (`plus`+), Morador (pagante)
- **NUNCA recebem**: Parceiro Vizinhança (INV-067), Marketing, Síndico tier `trial` (Q5)
- **Plan tier**: `plus`+
- **ABAC action**: `content.certificate.earn`

## Botões canônicos
- **Baixar certificado** (PDF)
- **Compartilhar certificado** (LinkedIn / link público)
- **Ver outros cursos** (cross-sell para [[LMS2]])

## Geração (canônica)
- `progressPercent == 100` → `CourseCompleted` (E-060)
- Handler `GenerateCertificate` cria `Certificate` com `serialNumber` único
- PDF gerado async (job) com QR code apontando para `/certificates/verify/{serial}` (público, sem auth)
- Emite `CertificateGenerated` (E-061)
- Email ao aluno com link

## Componentes
- `CelebrationHero` (animação + mensagem)
- `CertificatePreview` (mini PDF embutido se possível)
- `DownloadButton`
- `ShareButton` (Web Share API + LinkedIn deep-link)
- `RecommendationCarousel` (próximos cursos)

## Estados
- **Generating**: spinner "Gerando seu certificado (10-30s)..."
- **Ready**: download + share visíveis.
- **Failed**: toast "Falha ao gerar — tentar novamente"; botão Retry.
- **Não elegível** (Q5 trial): mensagem alternativa "Curso concluído! Faça upgrade para Plus para receber o certificado."
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Status do certificado | `/api/v1/academy/certificates/by-enrollment/{eid}` | GET |
| Verificar publicamente | `/certificates/verify/{serial}` | GET (público, sem auth) |

## Regras de negócio críticas
- **INV-068**: `Enrollment.progressPercent == 100` dispara `Certificate` (idempotente).
- `serialNumber` UNIQUE imutável.
- PDF inclui: nome, curso, carga horária, data, serialNumber, QR para verificação.
- Verificação pública sem auth.
- Certificado imutável pós-issue.
- Assinatura digital BR (MP 2.200-2 ICP-Brasil): Sprint 3+/M4+ — pendente CNT-019.

## Ligações
- [[LMS3]] · [[LMS4]] · [[LMS2]] · [[_moc]]

## Gaps/ressalvas
- **Q5 aberta**: Síndico tier `trial` recebe certificado? (padrão atual: não)
- Endpoint REST inferido.
- Provedor assinatura digital ICP-Brasil sem ADR.
- Certificado de **trilha** (não só de curso) backlog M4+ (G-09).
- Compartilhamento social: Web Share API tem suporte limitado em desktop.
