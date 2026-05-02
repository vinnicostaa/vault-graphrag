---
title: Legacy como referência (não como template)
type: concept
tags: [methodology, refactor, migration]
created: 2026-04-22
aliases: [Legacy Reference Pattern, Como usar código legado]
---

# Legacy como referência, não como template

Padrão de uso de um projeto anterior ao fazer reimplementação (migração de stack, reescrita, refactor grande). Resumo: **legacy preserva regras de negócio validadas em produção; não preserva decisões arquiteturais ruins**.

## O princípio

Um projeto legado é **duas coisas ao mesmo tempo**:

1. **Conhecimento validado** — regras de negócio, casos de borda, cotas, policies que só aparecem quando código bate no mundo real.
2. **Dívida técnica cristalizada** — antipatterns, decisões apressadas, drift entre módulos, God Objects, acoplamentos.

Quem reescreve **sem** o legado perde (1) e vai retrabalhar meses de regra de negócio. Quem reescreve **copiando** o legado carrega (2) e volta ao problema original disfarçado de "reescrita".

A regra é: **extrair (1), rejeitar (2)**.

## O que extrair do legacy

- **Regras de negócio validadas** — quotas, limites, estados válidos, transições permitidas, fluxos de happy-path e edge cases.
- **Value objects com validação** — algoritmos de validação (CPF, CNPJ, Email, Password) testados contra entrada real.
- **Contratos externos** — integrações com SDKs (payment, email, push) que já "conversam direito".
- **Migrações de dados** — schema validado, relacionamentos, invariantes no banco.
- **Telas e jornadas** — UX validada por usuário real.

## O que rejeitar do legacy

- **Arquitetura** — vem refeita desde o início (Clean Arch, DDD, CQRS, ou o que for).
- **Antipatterns identificados** em audit técnico — God Objects, vazamento de domínio, N+1, logs com PII, etc. Catálogo em [[../antipatterns/_moc]].
- **Nomenclatura ruim** — `sub`, `condo`, `usr` viram `subscription`, `condominium`, `user`.
- **Testes (ou falta deles)** — começar do zero com TDD.
- **Dependências obsoletas** — lib deprecated, versão antiga, CVE aberta.

## O processo recomendado

1. **Audit técnico do legacy**. Alguém (ou um agente) lê o código e produz documento listando:
   - Bugs conhecidos
   - Antipatterns identificados
   - Regras de negócio inferidas do código
   - Schema de dados atual
   - Telas/fluxos mapeados
2. **Lista "replicar vs não replicar"** — tabela explícita por módulo/feature.
3. **Plano de reimplementação** — ordem das fases, o que entra em MVP, o que adia.
4. **Código legado vira read-only** durante migração. Nunca editar mais nada lá que não seja hotfix crítico.
5. **Artefatos do legacy viram docs** no novo projeto — referências explícitas (`ver full-stack/contextos/X.md`).

## Sinais de que está usando o legacy errado

- **Copy-paste direto** de arquivos — sinal de "legacy como template" em vez de referência.
- **Estrutura de pastas espelhando o legacy** — novo projeto deve ter estrutura própria, não mimetizar o antigo.
- **Nomes de módulos iguais sem justificativa** — se o legacy tinha `auth/` e a arquitetura nova pede `identity/`, renomear.
- **Puxa dependência por "compatibilidade"** — dependências são escolhidas do zero no novo projeto.

## Exemplo concreto

Ver aplicação em [[../../50-projects/master-sindico/legacy-reference]] — TypeScript (Fastify + Drizzle) sendo referência pra reimplementação em Go (Gin abstraído + pgx + sqlc).

## Vizinhos no grafo

- [[sdd-workflow]] *(a criar)*
- [[../antipatterns/_moc]] — o que **não** replicar
- [[_moc]] — MOC de metodologia
