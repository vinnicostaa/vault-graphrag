---
title: "Onboarding por Persona — extraído de ONBOARDING DOS PERFIS.pdf"
updated: 2026-04-22
maintainer: agent-3 (analista)
type: catalog
status: stable
source: Content/contents/ONBOARDING DOS PERFIS.pdf, requirements.md Decision 3 RESOLVED
---

# Onboarding por Persona

Fluxo de cadastro inicial. **Decision 3 (resolvida)**: 4 fluxos separados — Morador (4 telas), Síndico (6), Empresa (7), Parceiro (4).

---

## Tela 0 — Boas-vindas (comum)

**Mensagem principal**: "Bem-vindo à Master Síndico."

**Mensagem de apoio**: "A Master Síndico é um ambiente digital onde a gestão condominial deixa de ser invisível. Aqui, informações são organizadas, decisões ganham histórico e relações se tornam mais claras."

**Mensagem de orientação**: "Para começar, precisamos entender qual é o seu papel no universo condominial."

**Seleção de perfil** (1 escolha):
- Morador
- Síndico
- Empresa do Mercado Condominial
- Comércio Local — Parceiro da Vizinhança

**Ação**: Continuar

---

## Tela 1 — Identificação Inicial (comum a todos)

**Mensagem principal**: "Vamos começar pelo essencial."

**Mensagem de apoio**: "Esses dados garantem seu acesso à plataforma, permitem salvar seu progresso e personalizar sua experiência desde o primeiro momento."

**Campos obrigatórios**:
- Nome completo OU nome da empresa
- E-mail

**Mensagem de tranquilização**: "Você pode concluir o cadastro agora ou continuar depois. Suas informações ficam salvas automaticamente."

**Auto-save**: Redis com TTL 30 dias (`onboarding:{user_id}:{step}`).

---

## Perfil MORADOR — 4 telas (TELA 2 a TELA 5)

### TELA 2 — Sua Identidade
**Mensagem**: "Você faz parte da vida do condomínio."

**Apoio**: "Na Master Síndico, o morador acompanha a gestão, entende o que está sendo feito e participa de forma mais consciente."

**Campos**:
- Foto de perfil
- Nome completo
- Data de nascimento
- Telefone
- CPF

### TELA 3 — Vínculo com o Condomínio
**Mensagem**: "Agora vamos conectar você ao seu condomínio."

**Apoio**: "Isso permite acesso a registros, comunicações e conteúdos relacionados ao local onde você vive."

**Campos**:
- CEP
- Endereço
- Condomínio (nome ou código de 8 caracteres alfanuméricos)

### TELA 4 — Participação e Convivência
**Mensagem**: "Participar também é responsabilidade."

**Apoio**: "Avaliações, interações e participações em eventos devem refletir sua percepção real, sempre com respeito e boa-fé."

**Aceites obrigatórios**:
- Termo Geral de Uso da Plataforma
- Política de Privacidade e LGPD
- Termo de Avaliações e Participação

**Ação**: Finalizar cadastro

---

## Perfil SÍNDICO — 6 telas (TELA 2 a TELA 6 + Final)

### TELA 2 — Sua Identidade de Gestão
**Mensagem**: "Ser síndico é assumir responsabilidade."

**Apoio**: "Aqui, sua atuação não fica restrita ao momento. Ela é registrada, organizada e se transforma em histórico de gestão."

**Campos**:
- Foto profissional
- Nome completo
- Data de nascimento
- E-mail profissional
- Telefone
- CPF

### TELA 3 — Sua Atuação
**Mensagem**: "Conte como você atua hoje."

**Apoio**: "Essas informações ajudam a contextualizar sua experiência e o alcance da sua gestão."

**Campos**:
- Endereço profissional
- Cidade e Estado de atuação
- Tipo de síndico (Morador OU Profissional)
- Tempo de atuação
- Quantidade de condomínios sob gestão

### TELA 4 — Governança e Estrutura (15 marcadores autodeclarados)
**Mensagem**: "Gestão também é estrutura."

**Apoio**: "Esses marcadores ajudam a demonstrar cuidado com riscos, organização e boas práticas administrativas."

**15 Marcadores**:
1. Membro ABRACS
2. Seguro de Responsabilidade Civil
3. Assessoria Jurídica
4. Assessoria Contábil
5. Assessoria em Segurança do Trabalho
6. Conformidade com LGPD
7. Conformidade com NR-1
8. Programa de Compliance
9. Selo Reclame Aqui
10. Outras certificações (campo aberto)
11. Premiações (campo aberto)
+ 4 derivados de 1-11

### TELA 5 — Sua Presença Profissional (N2/N3 apenas)
**Mensagem**: "Mostre quem está por trás da gestão."

**Campos** (níveis 2 e 3):
- Mini bio profissional
- Vídeo institucional (sujeito à trava trimestral 90 dias)
- Formação e certificações
- Administradoras vinculadas

**Nota**: N1 não tem perfil público — telas 5 são opcionais para N1.

### TELA 6 — Registros e Responsabilidade
**Mensagem**: "A plataforma registra. A responsabilidade é sua."

**Aceites obrigatórios**:
- Termo de Registro de Gestão e Linha do Tempo
- Termo de Indicadores, Selos e Certificação Própria
- Termo de Links Temporários e Compartilhamento

---

## Perfil EMPRESA — 7 telas (TELA 2 a TELA 8)

### TELA 2 — Identidade Institucional
**Campos**: Logo, Razão social, Nome fantasia, CNPJ, Data de aniversário da empresa, Nome do representante legal, E-mail do representante legal

### TELA 3 — Contatos e Organização
**Campos**: E-mail comercial, Telefone comercial, Endereço comercial com CEP, Dados do financeiro (nome, telefone, e-mail)

### TELA 4 — Atuação no Mercado
**Campos**: Cidades e Estados de atuação, Categoria principal, Subcategorias de atuação (até 5)

### TELA 5 — Conformidade, Governança e Boas Práticas (25 marcadores)
**Marcadores**:
- Membro ABRACS, Programa de Compliance, Assessoria Jurídica, Assessoria Contábil, Assessoria em Segurança do Trabalho
- Seguro de Responsabilidade Civil, Responsável Técnico, CIPA, NRs aplicáveis (campo aberto)
- Regularidade em Conselhos Profissionais, Conformidade com LGPD, Conformidade com NR-1
- ISO 9001, ISO 14001, ISO 45001, ISO 37001, ISO 19600 / ISO 37301
- ESG, Selo Verde ou Sustentável, Selo Carbon Free, Selo EuReciclo, Selo Empresa Amiga da Criança
- Selo Reclame Aqui, Premiações (campo aberto), Outra certificação (campo aberto), Certificação própria (campo aberto)

**Disclaimer**: "Esses marcadores são informativos, baseados em autodeclaração, e não representam certificação oficial da plataforma."

### TELA 6 — Conteúdo Institucional
**Campos**: Descrição institucional, Vídeo institucional, Vídeos técnicos, Portfólio

**Regra dura**: Sem chamadas comerciais, sem preços, sem dados de contato no conteúdo público — direcionar via Connect Me.

### TELA 7 — Uso da Plataforma
**Mensagem**: "A plataforma facilita conexões, não garante contratos."

**Aceites**:
- Termo de Conteúdo Técnico
- Termo de Uso do Connect Me

---

## Perfil PARCEIRO DA VIZINHANÇA — 4 telas (TELA 2 a TELA 4 + Final)

### TELA 2 — Presença Local
**Mensagem**: "Seu negócio faz parte do cotidiano do condomínio."

**Campos**:
- Logo
- Nome do estabelecimento
- CNPJ ou CPF (CPF aceito — Decision 12)
- Data de aniversário
- Nome do representante legal
- E-mail do representante legal

### TELA 3 — Contato e Visual
**Campos**: E-mail comercial, Telefone comercial, Endereço com CEP, Dados financeiros, Até 3 fotos (galeria)

### TELA 4 — Aceites
**Aceites obrigatórios**:
- Termo de Promoções
- Código de Conduta de Uso Indevido

---

## Tela Final (comum a todas as personas)

**Mensagem**: "Cadastro concluído com sucesso!"

**Nota**: "Algumas informações podem requerer revisão antes de se tornarem visíveis."

**Ação**: Acessar plataforma

**Backend**: após conclusão, migrar dados de Redis para tabelas de profile (`institutional.SindicoProfile`, `commercial.EnterpriseProfile`, etc) por bounded context.

---

## Mapa: Onboarding → Spec viva

| Persona | Telas (PDF) | Spec |
|---|---|---|
| Morador | 4 | `requirements/billing.md` Req 5 + `requirements/institutional.md` Req 6.X morador |
| Síndico | 6 | `requirements/institutional.md` Req 6.2 (15 governance markers) |
| Empresa | 7 | `requirements/institutional.md` Req 6.3 (25 compliance markers) |
| Parceiro | 4 | `requirements/institutional.md` Req 6.4 |

---

## Referências

- [[Inventario-Telas]] — todas as telas do produto
- [[../regras-de-negocio/regras-canonicas]] — Decision 3 + 12
- `backend/.kiro/specs/master-sindico/requirements/billing.md` Req 5 — onboarding canônico
- `backend/.kiro/specs/master-sindico/requirements/institutional.md` Req 6.2/6.3/6.4 — perfis profissionais
