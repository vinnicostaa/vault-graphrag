[FFV Academy](../index.html)/Estruturas de Dados & Algoritmos

🧮

Blog

# Estruturas de Dados & Algoritmos

Para quem já sabe o básico e quer ir fundo. Aqui o assunto é como os modelos funcionam em produção: memória, roteamento, ferramentas, agentes. O lado técnico que pouca gente explica direito.

9artigos

455XP total

📄PDF da trilha

🖥️Apresentar trilha

[](../aprenda/big-o-sem-misticismo/index.html)

01

## 📈 Big-O sem misticismo: pense em custo, não em "notação"

Big-O como linguagem pra falar de custo; O(1)/O(log n)/O(n)/O(n log n)/O(n²) com exemplos reais; quando constantes importam; amortized vs worst-case; por que bubble sort teórico é ruim mas timsort real venceu.

⏱ 10 min·+40 XP

→[](../aprenda/arrays-hashmaps-e-quando-importam/index.html)

02

## 🗂️ Arrays, hashmaps e quando realmente importam

Arrays contíguos (cache locality), hashmaps amortized O(1), collision strategies (open addressing vs chaining), load factor, quando Map beat Object em JS, Set, WeakMap. Por que hash ruim degrada pra O(n).

⏱ 11 min·+45 XP

→[](../aprenda/arvores-que-voce-realmente-usa/index.html)

03

## 🌳 Árvores que você realmente usa (BST, heap, trie)

BST balanceada (como Map de várias libs), heap (priority queue pra scheduling), trie (autocomplete, filesystem path). Pular AVL e red-black como estudo acadêmico — entender o que a lib padrão usa por baixo.

⏱ 12 min·+50 XP

→[](../aprenda/grafos-na-pratica/index.html)

04

## 🕸️ Grafos na prática: BFS, DFS, Dijkstra e quando aparecem

Grafo como abstração comum (deps de pacote, roteamento, social graph). BFS pra menor número de hops, DFS pra ciclos, Dijkstra pra shortest path com pesos. Representação: adjacency list vs matrix. Detectar ciclos em build systems.

⏱ 13 min·+55 XP

→[](../aprenda/recursao-e-dp-para-quem-odeia/index.html)

05

## 🔁 Recursão e DP para quem odeia: pensando em subproblemas

Recursão sem misticismo: caso base + passo. Memoization top-down, tabulation bottom-up. DP clássicos úteis (knapsack pra budget, LCS pra diff, edit distance pra fuzzy search). Quando recursão estoura stack e como reescrever iterativo.

⏱ 12 min·+50 XP

→[](../aprenda/algoritmos-de-string/index.html)

06

## 🔤 Algoritmos de string: substring, regex internals e fuzzy

indexOf moderno (Two-Way/Boyer-Moore por baixo), regex NFA vs DFA (por que regex ruim trava), fuzzy matching (edit distance, Jaro-Winkler), suffix array básico. Quando buscar em memória vs delegar pro Postgres FTS.

⏱ 11 min·+45 XP

→[](../aprenda/sorting-real/index.html)

07

## 🔢 Sorting real: timsort, quickselect e por que Array.sort basta

Timsort (V8, Python, JDK), quicksort pragmático, mergesort quando estabilidade importa, quickselect pra top-k sem ordenar tudo, counting/radix em casos específicos (ints pequenos). 99% do tempo: use lib padrão, não reinvente.

⏱ 10 min·+40 XP

→[](../aprenda/estruturas-probabilisticas/index.html)

08

## 🎲 Bloom, HyperLogLog, Count-Min: estruturas probabilísticas

Bloom filter (membership com falso positivo, zero falso negativo — cache, dedup); HyperLogLog (cardinalidade com 1-2% erro em MB); Count-Min sketch (frequência aproximada); quando aceitar imprecisão em troca de memória constante.

⏱ 13 min·+55 XP

→[](../aprenda/capstone-resolver-5-problemas-reais/index.html)

09

## 🏁 Capstone: resolver 5 problemas reais (não-LeetCode)

Cinco problemas de produção: (1) dedup de 10M eventos usando Bloom; (2) top-N queries lentas do banco com min-heap; (3) detecção de ciclo em deps de pacote; (4) cursor pagination com tiebreaker estável; (5) rate limit com sliding window counter. Cada um com análise de custo real.

⏱ 16 min·+75 XP

→

[← Voltar à home](../index.html)
