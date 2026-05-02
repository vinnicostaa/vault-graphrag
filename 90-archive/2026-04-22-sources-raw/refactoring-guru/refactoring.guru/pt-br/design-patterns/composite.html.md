::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
estruturais](structural-patterns.html){.type}
:::

# Composite {#composite .title}

::: aka
Também conhecido como: [Árvore de
objetos, ]{style="display:inline-block"}[Object
tree]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

O **Composite** é um padrão de projeto estrutural que permite que você
componha objetos em estruturas de árvores e então trabalhe com essas
estruturas como se elas fossem objetos individuais.

<figure class="image">
<img
src="../../images/patterns/content/composite/compositec299.png?id=73bcf0d94db360b636cd745f710d19db"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/composite/composite.png?id=73bcf0d94db360b636cd745f710d19db 640w,/images/patterns/content/composite/composite-1.5x.png?id=098acddafae70b1ec1cb1b05e833c3ae 960w,/images/patterns/content/composite/composite-2x.png?id=8847e6f8e2cb892ed2229faba83bd1b7 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Padrão de projeto Composite" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Usar o padrão Composite faz sentido apenas quando o modelo central de
sua aplicação pode ser representada como uma árvore.

Por exemplo, imagine que você tem dois tipos de objetos: `Produtos` e
`Caixas`. Uma `Caixa` pode conter diversos `Produtos` bem como um número
de `Caixas` menores. Essas `Caixas` menores também podem ter alguns
`Produtos` ou até mesmo `Caixas` menores que elas, e assim em diante.

Digamos que você decida criar um sistema de pedidos que usa essas
classes. Os pedidos podem conter produtos simples sem qualquer
compartimento, bem como caixas recheadas com produtos\... e outras
caixas. Como você faria para determinar o preço total desse pedido?

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/problem-pt-br1bbe.png?id=76ac3c711bfdff9dcaebc2f31e3b4359"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/composite/problem-pt-br.png?id=76ac3c711bfdff9dcaebc2f31e3b4359 370w,/images/patterns/diagrams/composite/problem-pt-br-1.5x.png?id=4a644423549997201ca25f6eef1dbe0d 555w,/images/patterns/diagrams/composite/problem-pt-br-2x.png?id=f05a72073b0a7f7ca2cd48d301e78f32 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Estrutura de um pedido complexo" />
<figcaption><p>Um pedido pode envolver vários produtos, embalados em
caixas, que são embalados em caixas maiores e assim em diante. Toda a
estrutura se parece com uma árvore de cabeça
para baixo.</p></figcaption>
</figure>

Você pode tentar uma solução direta: desempacotar todas as caixas,
verificar cada produto e então calcular o total. Isso pode ser viável no
mundo real; mas em um programa, não é tão simples como executar uma
iteração. Você tem que conhecer as classes dos `Produtos` e `Caixas` que
você está examinando, o nível de aninhamento das caixas e outros
detalhes cabeludos de antemão. Tudo isso torna uma solução direta muito
confusa ou até impossível.
:::

::: {.section .solution}
##  Solução {#solution}

O padrão Composite sugere que você trabalhe com `Produtos` e `Caixas`
através de uma interface comum que declara um método para a contagem do
preço total.

Como esse método funcionaria? Para um produto, ele simplesmente
retornaria o preço dele. Para uma caixa, ele teria que ver cada item que
ela contém, perguntar seu preço e então retornar o total para essa
caixa. Se um desses itens for uma caixa menor, aquela caixa também deve
verificar seu conteúdo e assim em diante, até que o preço de todos os
componentes internos sejam calculados. Uma caixa pode até adicionar um
custo extra para o preço final, como um preço de embalagem.

<figure class="image">
<img
src="../../images/patterns/content/composite/composite-comic-1-pt-brc0a6.png?id=f1e77dc84bf43a9b5b12a835969619b1"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/composite/composite-comic-1-pt-br.png?id=f1e77dc84bf43a9b5b12a835969619b1 600w,/images/patterns/content/composite/composite-comic-1-pt-br-1.5x.png?id=ceabe6409b846bf8b1b395404033dffc 900w,/images/patterns/content/composite/composite-comic-1-pt-br-2x.png?id=04fc5e12140e571385a62757db164ad0 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Solução sugerida pelo padrão Composite" />
<figcaption><p>O padrão Composite permite que você rode um comportamento
recursivamente sobre todos os componentes de uma árvore
de objetos.</p></figcaption>
</figure>

O maior benefício dessa abordagem é que você não precisa se preocupar
sobre as classes concretas dos objetos que compõem essa árvore. Você não
precisa saber se um objeto é um produto simples ou uma caixa
sofisticada. Você pode tratar todos eles com a mesma interface. Quando
você chama um método os próprios objetos passam o pedido pela árvore.
:::

::: {.section .analogy}
##  Analogia com o mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/live-examplee724.png?id=548a7cec45b493af66e8bfe524a137d3"
style="aspect-ratio: 1.22;"
srcset="/images/patterns/diagrams/composite/live-example.png?id=548a7cec45b493af66e8bfe524a137d3 280w,/images/patterns/diagrams/composite/live-example-1.5x.png?id=2129195f4103a5a4d3440a086ce3dc87 420w,/images/patterns/diagrams/composite/live-example-2x.png?id=b555458f20fc30425ae6dada5da492af 560w"
sizes="(max-width: 720px) 100vw, 280px" loading="lazy" width="280"
alt="Um exemplo de uma estrutura militar" />
<figcaption><p>Um exemplo de uma estrutura militar.</p></figcaption>
</figure>

Exércitos da maioria dos países estão estruturados como hierarquias. Um
exército consiste de diversas divisões; uma divisão é um conjunto de
brigadas, e uma brigada consiste de pelotões, que podem ser divididos em
esquadrões. Finalmente, um esquadrão é um pequeno grupo de soldados
reais. Ordens são dadas do topo da hierarquia e são passadas abaixo para
cada nível até cada soldado saber o que precisa ser feito.
:::

::::: {.section .structure-container}
##  Estrutura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/composite/structure-pt-bre2bd.png?id=d8a3392bb00602989fb6c4d7ae404f95"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.82;"
srcset="/images/patterns/diagrams/composite/structure-pt-br.png?id=d8a3392bb00602989fb6c4d7ae404f95 360w,/images/patterns/diagrams/composite/structure-pt-br-1.5x.png?id=8a071601235ad828c20489b9095efd57 540w,/images/patterns/diagrams/composite/structure-pt-br-2x.png?id=b107593ce17057df4b163d994b4c91d1 720w"
sizes="(max-width: 720px) 100vw, 360px" loading="lazy" width="360"
alt="Estrutura do padrão de projeto Composite" /><img
src="../../images/patterns/diagrams/composite/structure-pt-br-indexedebbf.png?id=658ceff7fe7f42bdf8b9a801b806350f"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/composite/structure-pt-br-indexed.png?id=658ceff7fe7f42bdf8b9a801b806350f 400w,/images/patterns/diagrams/composite/structure-pt-br-indexed-1.5x.png?id=def030bcc634b72bd7c8fdbad395dd39 600w,/images/patterns/diagrams/composite/structure-pt-br-indexed-2x.png?id=c321366870c69da43c508c71b296ce2a 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="Estrutura do padrão de projeto Composite" />
</figure>
:::

1.  A interface **Componente** descreve operações que são comuns tanto
    para elementos simples como para elementos complexos da árvore.

2.  A **Folha** é um elemento básico de uma árvore que não tem
    sub-elementos.

    Geralmente, componentes folha acabam fazendo boa parte do verdadeiro
    trabalho, uma vez que não tem mais ninguém para delegá-lo.

3.  O **Contêiner** (ou *composite*) é o elemento que tem sub-elementos:
    folhas ou outros contêineres. Um contêiner não sabe a classe
    concreta de seus filhos. Ele trabalha com todos os sub-elementos
    apenas através da interface componente.

    Ao receber um pedido, um contêiner delega o trabalho para seus
    sub-elementos, processa os resultados intermediários, e então
    retorna o resultado final para o cliente.

4.  O **Cliente** trabalha com todos os elementos através da interface
    componente. Como resultado, o cliente pode trabalhar da mesma forma
    tanto com elementos simples como elementos complexos da árvore.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Nesse exemplo, o padrão **Composite** deixa que você implemente pilhas
de formas geométricas em um editor gráfico.

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/example7ea7.png?id=98ba81d07c979038dd2ebf3c83a2e19f"
style="aspect-ratio: 0.84;"
srcset="/images/patterns/diagrams/composite/example.png?id=98ba81d07c979038dd2ebf3c83a2e19f 370w,/images/patterns/diagrams/composite/example-1.5x.png?id=ae008efd4146628cd7303925d4ce90be 555w,/images/patterns/diagrams/composite/example-2x.png?id=d21edef39d3792e8a4c6736727ac7305 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Exemplo de estrutura do Composite" />
<figcaption><p>Exemplo do editor de formas geométricas.</p></figcaption>
</figure>

A classe `GráficoComposto` é um contêiner que pode ter qualquer número
de sub-formas, incluindo outras formas compostas. Uma forma composta tem
os mesmos métodos que uma forma simples. Contudo, ao invés de fazer algo
próprio, uma forma composta passa o pedido recursivamente para todas as
suas filhas e "soma" o resultado.

O código cliente trabalha com todas as formas através da interface única
comum à todas as classes de forma. Portanto, o cliente não sabe se está
trabalhando com uma forma simples ou composta. O cliente pode trabalhar
com estruturas de objeto muito complexas sem ficar acoplado à classe
concreta que formou aquela estrutura.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// A interface componente declara operações comuns para ambos os
// objetos simples e complexos de uma composição.
interface Graphic is
    method move(x, y)
    method draw()

// A classe folha representa objetos finais de uma composição.
// Um objeto folha não pode ter quaisquer sub-objetos.
// Geralmente, são os objetos folha que fazem o verdadeiro
// trabalho, enquanto que os objetos composite somente delegam
// para seus sub componentes.
class Dot implements Graphic is
    field x, y

    constructor Dot(x, y) { ... }

    method move(x, y) is
        this.x += x, this.y += y

    method draw() is
        // Desenhar um ponto em X e Y.

// Todas as classes componente estendem outros componentes.
class Circle extends Dot is
    field radius

    constructor Circle(x, y, radius) { ... }

    method draw() is
        // Desenhar um círculo em X e Y com raio R.

// A classe composite representa componentes complexos que podem
// ter filhos. Objetos composite geralmente delegam o verdadeiro
// trabalho para seus filhos e então &quot;somam&quot; o resultado.
class CompoundGraphic implements Graphic is
    field children: array of Graphic

    // Um objeto composite pode adicionar ou remover outros
    // componentes (tanto simples como complexos) para ou de sua
    // lista de filhos.
    method add(child: Graphic) is
        // Adiciona um filho para o vetor de filhos.

    method remove(child: Graphic) is
        // Remove um filho do vetor de filhos.

    method move(x, y) is
        foreach (child in children) do
            child.move(x, y)

    // Um composite executa sua lógica primária em uma forma
    // particular. Ele percorre recursivamente através de todos
    // seus filhos, coletando e somando seus resultados. Já que
    // os filhos do composite passam essas chamadas para seus
    // próprios filhos e assim em diante, toda a árvore de
    // objetos é percorrida como resultado.
    method draw() is
        // 1. Para cada componente filho:
        //     - Desenhe o componente.
        //     - Atualize o retângulo limitante.
        // 2. Desenhe um retângulo tracejado usando as
        // limitantes.

// O código cliente trabalha com todos os componentes através de
// suas interfaces base. Dessa forma o código cliente pode
// suportar componentes folha simples e composites complexos.
class ImageEditor is
    field all: CompoundGraphic

    method load() is
        all = new CompoundGraphic()
        all.add(new Dot(1, 2))
        all.add(new Circle(5, 3, 10))
        // ...

    // Combina componentes selecionados em um componente
    // composite complexo.
    method groupSelected(components: array of Graphic) is
        group = new CompoundGraphic()
        foreach (component in components) do
            group.add(component)
            all.remove(component)
        all.add(group)
        // Todos os componentes serão desenhados.
        all.draw()</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Aplicabilidade {#applicability}

::::::: applicability
::: applicability-problem
Utilize o padrão Composite quando você tem que implementar uma estrutura
de objetos tipo árvore.
:::

::: applicability-solution
O padrão Composite fornece a você com dois tipos básicos de elementos
que compartilham uma interface comum: folhas simples e contêineres
complexos. Um contêiner pode ser composto tanto de folhas como por
outros contêineres. Isso permite a você construir uma estrutura de
objetos recursiva aninhada que se assemelha a uma árvore.
:::

::: applicability-problem
Utilize o padrão quando você quer que o código cliente trate tanto os
objetos simples como os compostos de forma uniforme.
:::

::: applicability-solution
Todos os elementos definidos pelo padrão Composite compartilham uma
interface comum. Usando essa interface o cliente não precisa se
preocupar com a classe concreta dos objetos com os quais está
trabalhando.
:::
:::::::
::::::::

::: {.section .checklist}
##  Como implementar {#checklist}

1.  Certifique-se que o modelo central de sua aplicação possa ser
    representada como uma estrutura de árvore. Tente quebrá-lo em
    elementos simples e contêineres. Lembre-se que contêineres devem ser
    capazes de conter tanto elementos simples como outros contêineres.

2.  Declare a interface componente com uma lista de métodos que façam
    sentido para componentes complexos e simples.

3.  Crie uma classe folha que represente elementos simples. Um programa
    pode ter múltiplas classes folha diferentes.

4.  Crie uma classe contêiner para representar elementos complexos.
    Nessa classe crie um vetor para armazenar referências aos
    sub-elementos. O vetor deve ser capaz de armazenar tanto folhas como
    contêineres, então certifique-se que ele foi declarado com um tipo
    de interface componente.

    Quando implementar os métodos para a interface componente, lembre-se
    que um contêiner deve ser capaz de delegar a maior parte do trabalho
    para os sub-elementos.

5.  Por fim, defina os métodos para adicionar e remover os elementos
    filhos no contêiner.

    Tenha em mente que essas operações podem ser declaradas dentro da
    interface componente. Isso violaria o *princípio de segregação de
    interface* porque os métodos estarão vazios na classe folha.
    Contudo, o cliente será capaz de tratar de todos os elementos de
    forma igual, mesmo ao montar a árvore.
:::

:::::: {.section .pros-cons}
##  Prós e contras {#pros-cons}

::::: row
::: col-sm-6
-    Você pode trabalhar com estruturas de árvore complexas mais
    convenientemente: utilize o polimorfismo e a recursão a seu favor.
-    *Princípio aberto/fechado*. Você pode introduzir novos tipos de
    elemento na aplicação sem quebrar o código existente, o que agora
    funciona com a árvore de objetos.
:::

::: col-sm-6
-    Pode ser difícil providenciar uma interface comum para classes cuja
    funcionalidade difere muito. Em certos cenários, você precisaria
    generalizar muito a interface componente, fazendo dela uma interface
    de difícil compreensão.
:::
:::::
::::::

::: {.section .relations}
##  Relações com outros padrões {#relations}

-   Você pode usar o [Builder](builder.html) quando criar árvores
    [Composite](composite.html) complexas porque você pode programar
    suas etapas de construção para trabalhar recursivamente.

-   O [Chain of Responsibility](chain-of-responsibility.html) é
    frequentemente usado em conjunto com o [Composite](composite.html).
    Neste caso, quando um componente folha recebe um pedido, ele pode
    passá-lo através de uma corrente de todos os componentes pai até a
    raiz do objeto árvore.

-   Você pode usar [Iteradores](iterator.html) para percorrer árvores
    [Composite](composite.html).

-   Você pode usar o [Visitor](visitor.html) para executar uma operação
    sobre uma árvore [Composite](composite.html) inteira.

-   Você pode implementar nós folha compartilhados da árvore
    [Composite](composite.html) como [Flyweights](flyweight.html) para
    salvar RAM.

-   O [Composite](composite.html) e o [Decorator](decorator.html) tem
    diagramas estruturais parecidos já que ambos dependem de composição
    recursiva para organizar um número indefinido de objetos.

    Um *Decorador* é como um *Composite* mas tem apenas um componente
    filho. Há outra diferença significativa: o *Decorador* adiciona
    responsabilidades adicionais ao objeto envolvido, enquanto que o
    *Composite* apenas "soma" o resultado de seus filhos.

    Contudo, os padrões também podem cooperar: você pode usar o
    *Decorador* para estender o comportamento de um objeto específico na
    árvore [Composite](composite.html)

-   Projetos que fazem um uso pesado de [Composite](composite.html) e do
    [Decorator](decorator.html) podem se beneficiar com frequência do
    uso do [Prototype](prototype.html). Aplicando o padrão permite que
    você clone estruturas complexas ao invés de reconstruí-las do zero.
:::

::: {.section .implementations}
##  Exemplos de código {#implementations}

[![Composite em
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](composite/csharp/example.html "Composite em C#"){.prog-lang-link}
[![Composite em
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](composite/cpp/example.html "Composite em C++"){.prog-lang-link}
[![Composite em
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](composite/go/example.html "Composite em Go"){.prog-lang-link}
[![Composite em
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](composite/java/example.html "Composite em Java"){.prog-lang-link}
[![Composite em
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](composite/php/example.html "Composite em PHP"){.prog-lang-link}
[![Composite em
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](composite/python/example.html "Composite em Python"){.prog-lang-link}
[![Composite em
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](composite/ruby/example.html "Composite em Ruby"){.prog-lang-link}
[![Composite em
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](composite/rust/example.html "Composite em Rust"){.prog-lang-link}
[![Composite em
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](composite/swift/example.html "Composite em Swift"){.prog-lang-link}
[![Composite em
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](composite/typescript/example.html "Composite em TypeScript"){.prog-lang-link}
:::

:::::: {#book-promo .banner-set2}
::::: {.prom .banner-content .banner-bg .banner-striped .banner-with-image data-id="DP: 1: Support our free website and own the eBook!" creative-id="standard-pt-br" position="content_bottom"}
::: banner-image
[![](../../images/patterns/banners/patterns-book-banner-3a29e.png?id=7d445df13c80287beaab234b4f3b698c){width="200"
height="200" loading="lazy"
srcset="/images/patterns/banners/patterns-book-banner-3-2x.png?id=0cc3f77ab421d1a5c02ee46488231c3a 2x"}](book.html)
:::

::: banner-text
### Apoie nosso website gratuito e ganhe o eBook! {#apoie-nosso-website-gratuito-e-ganhe-o-ebook .title}

-   22 padrões de projeto e 8 princípios explicados a fundo.
-   439 páginas bem estruturadas, fáceis de se ler e livres de jargões.
-   225 diagramas e ilustrações claras e úteis.
-   Um arquivo com exemplos de código em 11 línguas.
-   Suportado por todos os dispositivos: formatos PDF/EPUB/MOBI/KFX.

[ Saiba mais...](book.html){.btn .btn-secondary}
:::
:::::
::::::

::: next
#### Leia a seguir

[Decorator []{.fa .fa-arrow-right}](decorator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Bridge](bridge.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner-removable .banner-removable-patterns data-id="DP: Sidebar" creative-id="standard-sidebar-pt-br" position="sidebar"}
:::::: banner-inner
:::: image3d-book-right
::: {.image3d-cover style="background: #0b3752;"}
[![](../../images/patterns/book/web-cover-pt-brb3eb.png?id=cc7d821610ba9186e987ffc0f83476a8){width="250"
height="375" loading="lazy"
srcset="/images/patterns/book/web-cover-pt-br-2x.png?id=95380743a2833ed832101aa568c16373 2x"}](book.html)
:::
::::

::: {style="margin-top: 1rem"}
Este artigo é parte de nosso eBook\
**Mergulho nos Padrões de Projeto**.

[ Saiba mais...](book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
