---
title: "Decisões e Pendências"
version: "3.0"
date: 2026-03-10
status: "concluido"
tags:
  - mastersindico
  - referencia
  - decisoes
  - arquitetura
---

# ⚖️ Decisões Arquiteturais e Pendências

> Histórico das abordagens decididas com o cliente e pendências em aberto baseadas na análise de 23 documentos (Sprints 1+2 e adicionais).
> **Breadcrumbs:** [[00 - Índice Master Síndico|🏠 Início]] > [[Arquitetura do Ecossistema|🏛️ Arquitetura]] > 📄 Decisões e Pendências

---

## 🟢 Decisões Resolvidas (Documentadas nos Requisitos)

| # | Tópico | Contexto / Dúvida | Resolução (Status) |
|---|--------|-------------------|--------------------|
| **1** | **Escopo da Assembleia** | PDF citava "Assembleia Ao Vivo na TV" mas MVP parecia ser só Votação Digital simples. | **Resolvido (Opção B):** Assembleia ao vivo presencial, motor Real-Time/WebSockets com Telão Dividido (Presidente vs Apresentação). Opção online/híbrida inativa no MVP. (Req 48-60) |
| **2** | **Níveis de Plano** | Como as contas seriam cobradas? PDF mencionava planos, mas não os nomes. | **Resolvido:** Matriz definida como Trial, Base, Plus, Pro para todas as personas comerciais. Moradores possuem Base e Pagante (Pg). (Req 4) |
| **3** | **Fluxos de Onboarding** | Precisamos de telas de onboarding separadas por persona? | **Resolvido:** Sim. Cadastros densos e específicos para: Morador, Síndico, Empresa e Parceiro da Vizinhança (incluindo declarações e marcadores diferentes). (Req 5, 27) |
| **5** | **Escopo do Connect Me** | Apenas Síndico contactava Empresa? O PDF *Master M. 2025* sugeria outras setas. | **Resolvido:** Múltiplos fluxos unidirecionais aprovados: Morador→Síndico, Empresa→Empresa (Plus/Pro) e Síndico→Parceiro da Vizinhança. (Req 19, 19.1, 19.2, 19.3) |
| **7** | **Escopo do LMS (Academia)** | LMS é um sistema à parte ou do M.S.? Quem aprova? | **Resolvido:** Academia In-App completa. 5 áreas (Cursos, Treinamentos, Lives, Fórum, Biblioteca), 3 níveis e bloqueado estritamente para "Parceiros da Vizinhança". (Req 98.1) |
| **9** | **Parceiro da Vizinhança** | É usuário logado ou só card promocional inserido por síndicos? | **Resolvido:** Persona com login. Não exige CNPJ (aceita CPF). Pode criar promoções (6 telas próprias), acessar dashboard simples de clicks, mas sem acesso à Timeline ou LMS. (Req 27) |

---

## 🟡 Conflitos e INCONSISTÊNCIAS Resolvidas

1. **Tempo de Duração do Vídeo-Currículo (Req Banco de Talentos):**
   - PDF `ESTRUTURA DE CADASTRO.pdf` referia a 60s. Documento de Arquitetura dizia 90s.
   - **Resolução:** Prevalece o mais recente: **90 segundos**.
   
2. **Escopo de Telas do Morador (Connect Me vs Jornada):**
   - O arquivo `TELA DO MORADOR.ini` ocultava validações cruciais e campos exigidos pelo `JORNADA DO SÍNDICO E MORADOR MS .pdf`.
   - **Resolução:** O arquivo .ini foi descartado como *Source of Truth*. Telas M1-M15 expandidas para comportar RG/Selfie/Termo de Ciência de LGPD.

3. **Natureza do Connect Me Morador → Síndico:**
   - Princípio diz "Isso não é um Chat, não tem histórico visível" mas Req 19.1 descrevia um sistema de tickets com histórico (`allow sindico to respond with message, history maintained`).
   - **Resolução Exigida pelo Cliente (Decision 11):** Precisamos que o cliente decida entre a Opção A (Formulário Unidirecional Frio - sem chat de volta) vs Opção B (Sistema de Tickets bidirecional isolado).

---

## 🔴 Pendências Críticas (Necessitam Input do Cliente)

### 1. Painel Admin Master Síndico (Decision 8)
- A plataforma não tem mapeamento de como a própria startup Master Síndico fará aprovação manual dos CNPJs, controle dos assinaturas inadimplentes do Mercado Pago, gerência global de Tenants e relatórios B2B.

### 2. Paywall e Bloqueios Financeiros (Decision 4)
- **Questão:** Se uma assinatura acabar, o bloqueio é imediato (Hard block)? O condomínio inteiro é bloqueado ou apenas o síndico?
- **Impacto:** O que acontece quando o Plano Base vence: o Morador perde a timeline ou só o síndico para de postar?

### 3. Banco de Talentos / Currículo (Decision 6)
- **Questão:** Estava previsto no `ESTRUTURA DE CADASTRO.pdf` com 11 Telas densas. Ele entra na Sprint 1/2? Empresas pagam à parte para recrutar moradores?
- **Gap Arquitetural:** Há um GAP entre 3 últimos empregos vs 5 últimos empregos.

### 4. Modelo de Interação Morador→Síndico (Connect Me) (Decision 11)
- **Escolha:** "Sistema Frio" (síndico só aperta Status = Resolvido) ou "Mini-Ticket" (síndico pode mandar mensagens de volta)?

---

## 🔧 Decisões Técnicas Internas (Engenharia)

- **Arquitetura (Monolith vs. Microservice):** Empregar Módulo Monolítico Inicial (Domain-Driven, Erlang OTP Pattern no Node), estruturado fisicamente via `Adapter_Layer` para futuras separações (Billing, Video, Core, Realtime).
- **Tempo Real & WebSocket:** Indispensável para o Motor de Assembleias Ao Vivo e para o Painel do Presidente (Check-ins e Timers rodando a <200ms de latência).
- **Consistência de Dados (ACID):** Decidido uso de `DECIMAL/NUMERIC` puro no PostgreSQL para a "Fração Ideal" da Assembleia, barrando pontos flutuantes em votações de orçamento multimilionárias do condomínio.

---

**Breadcrumbs:** [[00 - Índice Master Síndico|🏠 Início]] · **Anterior:** [[Enums e Listas Mestre]]
