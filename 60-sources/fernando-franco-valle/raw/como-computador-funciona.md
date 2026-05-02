[FFV Academy](../index.html)/Como o Computador Funciona

🔩

Blog

# Como o Computador Funciona

Para quem já sabe o básico e quer ir fundo. Aqui o assunto é como os modelos funcionam em produção: memória, roteamento, ferramentas, agentes. O lado técnico que pouca gente explica direito.

9artigos

655XP total

📄PDF da trilha

🖥️Apresentar trilha

[](../aprenda/cpu-pipeline-cache/index.html)

01

## ⚙️ CPU: pipeline, cache L1/L2/L3, branch prediction

Fetch-Decode-Execute em detalhe, por que cache miss mata performance, como branch prediction funciona — e o que isso significa para o código que você escreve.

⏱ 17 min·+85 XP

→[](../aprenda/memoria-stack-heap-virtual/index.html)

02

## 🧠 Memória: stack, heap, virtual memory, page fault

Stack vs heap, por que stack overflow acontece, virtual memory como abstração, page fault e swapping — como o SO gerencia memória de todos os processos.

⏱ 16 min·+80 XP

→[](../aprenda/syscalls-user-kernel/index.html)

03

## 🚧 Syscalls: a fronteira entre user-space e kernel

O que é uma syscall, por que trocar entre user mode e kernel mode tem custo, como strace revela o que seu programa faz de verdade.

⏱ 14 min·+70 XP

→[](../aprenda/file-descriptors-io/index.html)

04

## 📁 File descriptors e I/O: o que todo processo compartilha

stdin/stdout/stderr como file descriptors, por que tudo é arquivo no Linux, pipes, redirecionamento — e como isso afeta performance de I/O.

⏱ 13 min·+65 XP

→[](../aprenda/io-bloqueante-nao-bloqueante/index.html)

05

## ⚡ I/O bloqueante, não-bloqueante, async: select/poll/epoll

Por que servidores web usam epoll em vez de um thread por conexão. Blocking vs non-blocking vs async I/O — o fundamento de event loops como asyncio e Node.js.

⏱ 17 min·+85 XP

→[](../aprenda/threads-vs-processos/index.html)

06

## 🔀 Threads vs processos vs fibras: modelo de concorrência

O que diferencia um thread de um processo no SO, context switch, por que green threads existem, e como Go goroutines e Python asyncio resolvem diferente.

⏱ 15 min·+75 XP

→[](../aprenda/containers-namespaces-cgroups/index.html)

07

## 📦 Containers por baixo: namespaces e cgroups no Linux

Docker não é mágica — é Linux namespaces + cgroups + union filesystem. Entender isso explica por que containers são mais leves que VMs e como security funciona.

⏱ 16 min·+80 XP

→[](../aprenda/serializacao-endianness/index.html)

08

## 📡 Serialização, endianness, UTF-8: os bytes que viajam

Por que "olá" tem bytes diferentes em UTF-8 e Latin-1. Big-endian vs little-endian. JSON vs Protobuf vs MessagePack — trade-offs de serialização.

⏱ 11 min·+55 XP

→[](../aprenda/tempo-clocks-ntp/index.html)

09

## 🕐 Tempo distribuído: NTP, clock skew, monotonic vs wall

Por que relógios em servidores diferentes mentem. NTP e PTP, clock drift, por que nunca usar wall clock para medir duração, e como sistemas distribuídos lidam com tempo.

⏱ 12 min·+60 XP

→

[← Voltar à home](../index.html)
