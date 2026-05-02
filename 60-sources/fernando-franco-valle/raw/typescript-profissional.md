[FFV Academy](../index.html)/TypeScript Profissional

🔷

Blog

# TypeScript Profissional

Para quem já sabe o básico e quer ir fundo. Aqui o assunto é como os modelos funcionam em produção: memória, roteamento, ferramentas, agentes. O lado técnico que pouca gente explica direito.

10artigos

565XP total

📄PDF da trilha

🖥️Apresentar trilha

[](../aprenda/typescript-como-mental-model/index.html)

01

## 🧠 TypeScript como mental model: tipos são prova, não anotação

Por que TS não é "JS com comentários" e sim um sistema de tipos estrutural com inference poderosa. Tipos são asserções sobre o runtime; o compilador prova que o fluxo respeita essas asserções. O clique mental que muda tudo.

⏱ 11 min·+45 XP

→[](../aprenda/narrowing-discriminated-unions/index.html)

02

## 🎯 Narrowing e discriminated unions: o coração real do TypeScript

Narrowing via typeof, instanceof, \`in\`, truthiness, equality. Discriminated unions com tag field (\`kind: "ok" \| "err"\`). Exhaustiveness check com \`never\`. Por que \`as\` quase sempre significa que você abandonou o sistema.

⏱ 13 min·+55 XP

→[](../aprenda/generics-de-verdade/index.html)

03

## ⚙️ Generics de verdade: variance, constraints e conditional types

Generics além de \`Array\<T\>\`: constraints com \`extends\`, infer para extrair tipo de função/array, conditional types, variance (covariant, contravariant, invariant). Quando reescrever libs com generics robustos em vez de copiar-colar.

⏱ 14 min·+60 XP

→[](../aprenda/tipos-utilitarios-e-quando-nao-usar/index.html)

04

## 🧰 Tipos utilitários (Partial, Pick, Omit...) e quando NÃO usar

Partial, Required, Readonly, Pick, Omit, Record, Exclude, Extract, NonNullable, ReturnType, Awaited. O que cada um faz, quando é gambiarra (ex: Partial\<T\> escondendo modelagem fraca) e como prefers tipos nomeados a combinações monstruosas.

⏱ 10 min·+45 XP

→[](../aprenda/type-safety-em-boundaries/index.html)

05

## 🛡️ Type safety em boundaries: Zod, io-ts e validação runtime

Tipo compilado não protege no runtime. Todo boundary (API, form, localStorage, query param) precisa parse com schema Zod/Valibot/io-ts. Padrão \`safeParse\` retornando Result. Como inferir tipo TS do schema com z.infer, fechando o loop.

⏱ 12 min·+55 XP

→[](../aprenda/async-await-sem-pegadinha/index.html)

06

## ⏳ Async/await sem pegadinha: promises, AbortController e cancelamento

Promise é um valor, não uma função. \`async\` wrappa função em Promise. \`await\` é a única forma sã de consumir. AbortController para cancelar fetch, setTimeout, streams. \`Promise.all\` vs \`Promise.allSettled\` vs \`Promise.race\`. Erros silenciosos em \`.then\` sem catch.

⏱ 12 min·+50 XP

→[](../aprenda/erros-como-valores/index.html)

07

## 🚨 Erros como valores: Result, neverthrow e por que \`throw\` quebra

\`throw\` é goto tipado fraco — quebra inference, não aparece na assinatura, impede exhaustiveness. Result\<T, E\> (estilo Rust) transforma erros em retorno explícito. Biblioteca neverthrow, padrões de railway-oriented programming, quando \`throw\` ainda faz sentido (unrecoverable).

⏱ 13 min·+55 XP

→[](../aprenda/performance-em-node/index.html)

08

## 🚀 Performance em Node: event loop, streams e backpressure

Event loop fases (timers, poll, check, close), libuv thread pool, sync vs async FS, Readable/Writable/Transform streams, backpressure via \`pipeline()\`, worker_threads quando CPU-bound. Como achar o gargalo com clinic.js, --prof e heap snapshot.

⏱ 15 min·+65 XP

→[](../aprenda/monorepo-pnpm-turbo/index.html)

09

## 📦 Monorepo profissional: pnpm workspaces + Turbo + shared configs

Por que times sérios usam monorepo: shared packages, atomic commits cruzando apps, CI unificada. pnpm workspaces (packages/\*), Turbo pipeline.json com cache, shared tsconfig base, shared ESLint/Prettier. Anti-padrões (circular deps, shared mutáveis).

⏱ 13 min·+55 XP

→[](../aprenda/capstone-cli-tool-ts/index.html)

10

## 🏁 Capstone: construir um CLI tool TypeScript end-to-end

Projeto consolidando a trilha: CLI com commander/citty, flags tipadas com Zod, async com AbortController, erros via Result, publicação em npm com bin/, tests com vitest, release automático com changesets. Saída: binário usável de verdade.

⏱ 18 min·+80 XP

→

[← Voltar à home](../index.html)
