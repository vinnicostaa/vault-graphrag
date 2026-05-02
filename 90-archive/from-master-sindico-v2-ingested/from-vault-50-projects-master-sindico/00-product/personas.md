---
title: Personas — Master Síndico
type: project
tags: [master-sindico, product, personas]
project: master-sindico
created: 2026-04-22
---

# Personas — Master Síndico

5 personas principais. Cada uma tem jornada mapeada em `_archive/content-discovery-2026-03/jornada-*.md` (arquivadas 2026-04-22 durante sanitização — conteúdo intacto, paths atualizados).

---

## 1. Síndico (principal decisor)

**Quem**: pessoa física eleita pelo condomínio (ou profissional contratado) que toma decisões de gestão.

**Dores**:
- Dificuldade em escolher empresas prestadoras sem critério
- Perda de histórico entre mandatos (falta memória institucional)
- Assembleias caóticas (fila de fala, voto manual, ata confusa)
- Comunicação com moradores (pulverizada em vários canais)
- Pressão de moradores em grupos de WhatsApp

**Ganhos que busca**:
- Critério auditável na contratação (reputação)
- Histórico automático do condomínio
- Assembleia digital rastreável
- Canal oficial com moradores
- Proteção contra questionamentos jurídicos (LGPD, prestação de contas)

**Plano típico**: Pro ou Enterprise (paga via condomínio ou reembolso)

**Trial**: 15 dias

**Device primário**: desktop (gestão), mobile secundário (emergências)

**Jornada**: ver [[../_archive/content-discovery-2026-03/JORNADA DO SÍNDICO completa]]

---

## 2. Morador

**Quem**: condômino (CPF + unidade).

**Dores**:
- Não sabe o que está acontecendo no condomínio (decisões, obras, assembleias)
- Dificuldade em contratar serviço para unidade com confiança
- Falta de voz nas decisões (assembleias presenciais limitantes)
- Reclamações pulverizadas em múltiplos grupos

**Ganhos que busca**:
- Transparência (timeline + decisões)
- Acesso fácil a empresas aprovadas (Connect Me)
- Voto remoto em assembleias
- Canal oficial de interação com síndico

**Plano**: gratuito (consumo do plano do condomínio)

**Trial**: não aplica (acesso via condomínio)

**Device primário**: mobile (app) — 90% das interações
**Device secundário**: web (acesso a documentos longos)

**Jornada**: ver [[../_archive/content-discovery-2026-03/JORNADA DO MORADOR]]

---

## 3. Empresa condominial

**Quem**: PJ que presta serviços ao mercado condominial (portaria, limpeza, manutenção, segurança).

**Dores**:
- Dificuldade em prospectar condomínios
- Falta canal direto com síndico (sem passar por portaria/WhatsApp)
- Competição por preço, sem diferenciação por qualidade
- Reputação não é transportada entre condomínios

**Ganhos que busca**:
- Vitrine visual (perfil + vídeos institucionais via Mux)
- Reputação consolidada (Bronze → Diamante)
- Connect Me ativo com notificação
- Histórico de contratos

**Plano**: Empresa (com tiers: Básico, Pro, Enterprise)

**Trial**: 7 dias

**Device**: desktop (cadastro, resposta Connect Me), mobile (notificações)

**Jornada**: ver [[../_archive/content-discovery-2026-03/JORNADA DA EMPRESA]]

---

## 4. Agência de marketing

**Quem**: agência contratada por empresa pra gerenciar sua presença no Master Síndico.

**Dores**:
- Dificuldade em gerenciar múltiplas empresas em múltiplas plataformas
- Falta de métricas agregadas
- Configurar perfil de empresa leva muito tempo

**Ganhos que busca**:
- Dashboard multi-empresa
- API / integração (futuro)
- Templates + assets reutilizáveis
- Relatório consolidado pro cliente

**Plano**: Empresa com add-on "Agência" (permite multi-conta)

**Trial**: compartilha trial da empresa

**Device**: desktop

**Jornada**: ver [[../_archive/content-discovery-2026-03/JORNADA DA EMPRESA DE MARKETING]]

---

## 5. Parceiro de comércio local

**Quem**: estabelecimento comercial local (pizzaria, pet shop, barbearia, academia, loja de bairro) que quer vínculo com condomínios próximos.

**Dores**:
- Prospecção cara (panfleto, Google Ads)
- Sem canal direto com moradores
- Sem selo de confiança

**Ganhos que busca**:
- Descoberta por moradores da região (Vizinhança)
- Ofertas / promoções ao condomínio
- Status aprovado pelo síndico = mini-selo

**Plano**: Parceiro (mensalidade baixa ou taxa sobre conversão)

**Trial**: 30 dias (mais longo para demonstrar valor)

**Device**: mobile (maior parte são pequenos negócios)

**Jornada**: ver [[../_archive/content-discovery-2026-03/Jornada do Parceiro da Vizinhança]]

---

## Mapa persona × plano × trial × ações

| Persona | Plano | Trial | Feature unlock |
|---|---|---|---|
| Síndico | Pro / Enterprise | 15d | criar condomínio, contratar via Connect Me, abrir assembleia |
| Morador | gratuito (via condo) | - | votar, ver timeline, consumir Connect Me |
| Empresa | Empresa Básico/Pro/Enterprise | 7d | perfil vitrine, responder Connect Me, upload vídeo |
| Agência | Empresa + Agência | 7d | multi-conta, dashboard agregado |
| Parceiro | Parceiro | 30d | Vizinhança, ofertas, selo síndico-aprovado |

---

## Jornadas cruzadas (exemplos)

### Síndico contrata via Connect Me
1. Síndico abre Connect Me: "Preciso pintura fachada"
2. Sistema notifica empresas com `reputation ≥ Prata` no raio
3. 3 empresas respondem com proposta
4. Síndico avalia (perfil, vídeos, reputação)
5. Síndico fecha contrato (trilha de auditoria + timeline)
6. Obra executada + avaliação pós (alimenta reputação)

### Morador vota em assembleia
1. Morador recebe push de assembleia agendada
2. Abre app, vê edital (PDF)
3. Assembleia começa: entra live (Livekit)
4. Vota: 1 voto por fração ideal (não por pessoa)
5. Ata publicada imutável na timeline

### Empresa constrói reputação
1. Ganha primeiro contrato via Connect Me (Bronze)
2. 3 contratos entregues bem-avaliados → Prata
3. 10 contratos bem-avaliados em diferentes condomínios → Ouro
4. Case de vídeo institucional publicado → ranqueamento de busca sobe

---

## Links

- [[_moc]]
- [[vision]]
- [[business-model]]
- [[value-proposition]]
- [[../01-domain/bounded-contexts]]
- [[../04-requirements/functional/_moc]]
