---
title: Building end-to-end security for Messenger (E2EE default rollout)
type: source
tags: [research, meta, messenger, instagram-direct, e2ee, signal-protocol, labyrinth, dm, source]
source: https://engineering.fb.com/2023/12/06/security/building-end-to-end-security-for-messenger/
created: 2026-04-23
updated: 2026-04-23
aliases: [source-09-messenger-e2ee]
---

# source-09 — Messenger E2EE default (Dec 2023)

- **URL**: https://engineering.fb.com/2023/12/06/security/building-end-to-end-security-for-messenger/
- **Publicado**: 2023-12-06
- **Autor/Canal**: Engineering at Meta
- **Tipo**: post técnico
- **Contexto**: rollout de E2EE default pra Messenger pessoal 1:1 (dez/2023). Instagram Direct está em phase-in. Em 2026-03 (9to5Mac) reportagem indica Meta removendo algumas funcionalidades E2EE em Instagram DMs — estado atual instável.

## Sinopse

Meta rolou E2EE default no Messenger pessoal 1:1 em dezembro de 2023. Usa Signal Protocol pra transmissão e **Labyrinth Protocol próprio** pra armazenamento server-side cifrado (diferente do WhatsApp que é device-centric).

## Pontos-chave

1. **Protocolo de transmissão**: **Signal Protocol** (mesmo padrão WhatsApp/Signal).
2. **Storage**: **Labyrinth Protocol** — próprio da Meta. Mensagens ficam cifradas nos servidores Meta; keys sob controle do usuário. Permite histórico multi-device sem depender de device "primário" como WhatsApp.
3. **Desafios listados**:
   - Multi-device via username/password (não é device-bound).
   - Reconstruir features (ex: reactions, replies) compatíveis com E2EE usando tecnologias tipo **OHAI** e **Anonymous Credentials**.
   - Web client (diferente do nativo).
   - Key rotation quando device é removido.
4. **Custo**: Meta reescreveu "quase todo" o codebase de messaging/calling do zero.
5. **Timeline**: rollout começou ago/2023, completo pra Messenger ao fim de 2023.

## Aplicabilidade ao Master Síndico

Detalhado em `_destilado.md` §7.

**Utilidade como contra-exemplo**: **Connect Me NÃO é chat, e isso é decisão firme.**

Motivos reforçados por essa fonte:

1. **E2EE chat é muito caro de construir**. Meta gastou anos, reescreveu quase tudo. Não existe drop-in correto.
2. **Armazenar histórico cifrado** (Labyrinth) é protocolo próprio, fora do escopo de um SaaS pequeno.
3. **Multi-device + E2EE + histórico** é combinação raríssima em produtos, exige engenharia séria.
4. **Moderação E2EE é quase impossível** — se não podemos ler o conteúdo, não podemos moderar. Master Síndico **exige** moderação.
5. **LGPD + E2EE** complicam: direito de esquecimento, export de dados pessoais, subpoena — tudo mais difícil com E2EE.

**Connect Me deve ser**:
- Formulário estruturado tipado (serviço, data, descrição, orçamento).
- Notificações push/email estruturadas.
- State machine (contato → proposta → aceite → contrato → avaliação).
- **Não** free-text chat.

**Se no futuro surgir demanda de "chat" de verdade**: avaliar delegação a SaaS pronto (Intercom, Twilio Conversations, Stream Chat) antes de construir. ADR obrigatória com trade-offs.

## Citações

> "We use the Signal protocol in our products (such as WhatsApp, Messenger, and Instagram Direct)."

> "Meta's Labyrinth encrypted storage protocol describes their protocol for end-to-end encrypting stored message history between devices on a user's account."

## Confiança / datas

- Data confirmada via fetch (2023-12-06) — **alta confiança**.
- Fonte primária Meta Engineering.
- Estado atual Instagram Direct E2EE (abril 2026) — **média confiança** (reportagem 9to5Mac indica reversão parcial; não reconfirmado em Meta oficial nesta sessão).
