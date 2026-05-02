::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
comportamentais](behavioral-patterns.html){.type}
:::

# Visitor {#visitor .title}

::: aka
Também conhecido como: [Visitante]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

O **Visitor** é um padrão de projeto comportamental que permite que você
separe algoritmos dos objetos nos quais eles operam.

<figure class="image">
<img
src="../../images/patterns/content/visitor/visitor4a4c.png?id=f36d100188340db7a18854ef7916f972"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/visitor/visitor.png?id=f36d100188340db7a18854ef7916f972 640w,/images/patterns/content/visitor/visitor-1.5x.png?id=751e251363b632b20df856d72d02ee72 960w,/images/patterns/content/visitor/visitor-2x.png?id=2c5d9ab3046d782c19809d3b80650d65 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Padrão de Projeto Visitor" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagine que sua equipe desenvolve uma aplicação que funciona com
informações geográficas estruturadas em um grafo colossal. Cada vértice
do gráfico pode representar uma entidade complexa como uma cidade, mas
também coisas mais granulares como indústrias, lugares turísticos, etc.
Os vértices estão conectados entre si se há uma estrada entre os objetos
reais que eles representam. Por debaixo dos panos, cada tipo de vértice
é representado por sua própria classe, enquanto que cada vértice
específico é um objeto.

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/problem14338.png?id=e7076532da1e936f3519c63270da8454"
style="aspect-ratio: 2.43;"
srcset="/images/patterns/diagrams/visitor/problem1.png?id=e7076532da1e936f3519c63270da8454 560w,/images/patterns/diagrams/visitor/problem1-1.5x.png?id=250216d3458c38e9855eda96bf71251b 840w,/images/patterns/diagrams/visitor/problem1-2x.png?id=2e5d5143ac55af218754f28761bec17e 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Exportando o grafo para XML" />
<figcaption><p>Exportando o grafo para XML.</p></figcaption>
</figure>

Em algum momento você tem uma tarefa de implementar a exportação do
grafo para o formato XML. No começo, o trabalho parecia muito simples.
Você planejou adicionar um método de exportação para cada classe nó e
então uma alavancagem recursiva para ir a cada nó do grafo, executando o
método de exportação. A solução foi simples e elegante: graças ao
polimorfismo, você não estava acoplando o código que chamava o método de
exportação com as classes concretas dos nós.

Infelizmente, o arquiteto do sistema se recusou a permitir que você
alterasse as classes nó existentes. Ele disse que o código já estava em
produção e ele não queria arriscar quebrá-lo por causa de um possível
bug devido às suas mudanças.

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/problem2-pt-brd88d.png?id=d9e4c49f761f851a6139a4b65df1a217"
style="aspect-ratio: 1.92;"
srcset="/images/patterns/diagrams/visitor/problem2-pt-br.png?id=d9e4c49f761f851a6139a4b65df1a217 500w,/images/patterns/diagrams/visitor/problem2-pt-br-1.5x.png?id=71e179aefa9997a58f054fd53eee097b 750w,/images/patterns/diagrams/visitor/problem2-pt-br-2x.png?id=a093d7fa33bd8ae48b1d7b5172cb50af 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="O método de exportação XML teve que ser adicionado a todas as classes nodo" />
<figcaption><p>O método de exportação XML teve que ser adicionado a
todas as classes nodo, o que trouxe o risco de quebrar toda a aplicação
se quaisquer bugs passarem junto com a mudança.</p></figcaption>
</figure>

Além disso, ele questionou se faria sentido ter um código de exportação
XML dentro das classes nó. O trabalho primário dessas classes era
trabalhar com dados geográficos. O comportamento de exportação XML
ficaria estranho ali.

Houve outra razão para a recusa. Era bem provável que após essa
funcionalidade ser implementada, alguém do departamento de marketing
pediria que você fornecesse a habilidade para exportar para um formato
diferente, ou pediria alguma outra coisa estranha. Isso forçaria você a
mudar aquelas frágeis e preciosas classes novamente.
:::

::: {.section .solution}
##  Solução {#solution}

O padrão Visitor sugere que você coloque o novo comportamento em uma
classe separada chamada *visitante*, ao invés de tentar integrá-lo em
classes já existentes. O objeto original que teve que fazer o
comportamento é agora passado para um dos métodos da visitante como um
argumento, desde que o método acesse todos os dados necessários contidos
dentro do objeto.

Agora, e se o comportamento puder ser executado sobre objetos de classes
diferentes? Por exemplo, em nosso caso com a exportação XML, a
verdadeira implementação vai provavelmente ser um pouco diferente nas
variadas classes nó. Portanto, a classe visitante deve definir não um,
mas um conjunto de métodos, cada um capaz de receber argumentos de
diferentes tipos, como este:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class ExportVisitor implements Visitor is
    method doForCity(City c) { ... }
    method doForIndustry(Industry f) { ... }
    method doForSightSeeing(SightSeeing ss) { ... }
    // ...</code></pre>
</figure>

Mas como exatamente nós chamaríamos esses métodos, especialmente quando
lidando com o grafo inteiro? Esses métodos têm diferentes assinaturas,
então não podemos usar o polimorfismo. Para escolher um método visitante
apropriado que seja capaz de processar um dado objeto, precisaríamos
checar a classe dele. Isso não parece um pesadelo?

<figure class="code">
<pre class="code" lang="pseudocode"><code>foreach (Node node in graph)
    if (node instanceof City)
        exportVisitor.doForCity((City) node)
    if (node instanceof Industry)
        exportVisitor.doForIndustry((Industry) node)
    // ...
}</code></pre>
</figure>

Você pode perguntar, por que não usamos o sobrecarregamento de método?
Isso é quando você dá a todos os métodos o mesmo nome, mesmo se eles
suportam diferentes conjuntos de parâmetros. Infelizmente, mesmo
assumindo que nossa linguagem de programação suporta o sobrecarregamento
(como Java e C#), isso não nos ajudaria. Já que a classe exata de um
objeto nó é desconhecida de antemão, o mecanismo de sobrecarregamento
não será capaz de determinar o método correto para executar. Ele irá
usar como padrão o método que usa um objeto da classe `Nó` base.

Contudo, o padrão Visitor resolve esse problema. Ele usa uma técnica
chamada [Double Dispatch](visitor-double-dispatch.html), que ajuda a
executar o método apropriado de um objeto sem precisarmos de
condicionais pesadas. Ao invés de deixar o cliente escolher uma versão
adequada do método para chamar, que tal delegarmos essa escolha para os
objetos que estamos passando para a visitante como argumentos? Já que os
objetos sabem suas próprias classes, eles serão capazes de escolher um
método adequado na visitante de forma simples. Eles "aceitam" uma
visitante e dizem a ela qual método visitante deve ser executado.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Código cliente
foreach (Node node in graph)
    node.accept(exportVisitor)

// Cidade
class City is
    method accept(Visitor v) is
        v.doForCity(this)
    // ...

// Indústria
class Industry is
    method accept(Visitor v) is
        v.doForIndustry(this)
    // ...</code></pre>
</figure>

Eu confesso. Tivemos que mudar as classes nó de qualquer jeito. Mas ao
menos a mudança foi trivial e ela permite que nós adicionemos novos
comportamentos sem alterar o código novamente.

Agora, se extrairmos uma interface comum para todas as visitantes, todos
os nós existentes podem trabalhar com uma visitante que você introduzir
na aplicação. Se você se deparar mais tarde adicionando um novo
comportamento relacionado aos nós, tudo que você precisa fazer é
implementar uma nova classe visitante.
:::

::: {.section .analogy}
##  Analogia com o mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/visitor/visitor-comic-17f02.png?id=7ee4fa8800f7c4df4e1aa3b1aca2b7f1"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/visitor/visitor-comic-1.png?id=7ee4fa8800f7c4df4e1aa3b1aca2b7f1 600w,/images/patterns/content/visitor/visitor-comic-1-1.5x.png?id=c9cadd73d25cc63fce94312f3c82bfee 900w,/images/patterns/content/visitor/visitor-comic-1-2x.png?id=439032451eb49ebbcb5257f25ecee790 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Agente de seguros" />
<figcaption><p>Um bom agente de seguros está sempre pronto para oferecer
diferentes apólices para vários tipos de organizações.</p></figcaption>
</figure>

Imagine um agente de seguros experiente que está ansioso para obter
novos clientes. Ele pode visitar cada prédio de uma vizinhança, tentando
vender apólices para todos que encontra. Dependendo do tipo de
organização que ocupa o prédio, ele pode oferecer apólices de seguro
especializadas:

-   Se for um prédio residencial, ele vende seguros médicos.
-   Se for um banco, ele vende seguro contra roubo.
-   Se for uma cafeteria, ele vende seguro contra incêndios e enchentes.
:::

::::: {.section .structure-container}
##  Estrutura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/structure-pt-bref7f.png?id=f25c7535e901981fe19994718252991d"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.96;"
srcset="/images/patterns/diagrams/visitor/structure-pt-br.png?id=f25c7535e901981fe19994718252991d 520w,/images/patterns/diagrams/visitor/structure-pt-br-1.5x.png?id=64e7bc30a08108d12994b7f7f6062084 780w,/images/patterns/diagrams/visitor/structure-pt-br-2x.png?id=1b9752d0cbcf22bd81954ae7f436ef82 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Estrutura do padrão de projeto Visitor" /><img
src="../../images/patterns/diagrams/visitor/structure-pt-br-indexed4c48.png?id=19b2cb06ff30c10f816933bd43111367"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.93;"
srcset="/images/patterns/diagrams/visitor/structure-pt-br-indexed.png?id=19b2cb06ff30c10f816933bd43111367 520w,/images/patterns/diagrams/visitor/structure-pt-br-indexed-1.5x.png?id=7490be57d42f3f5d5f1a04976dc76726 780w,/images/patterns/diagrams/visitor/structure-pt-br-indexed-2x.png?id=5bab58108966ea227c498f35539ffe6b 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Estrutura do padrão de projeto Visitor" />
</figure>
:::

1.  A interface **Visitante** declara um conjunto de métodos visitantes
    que podem receber elementos concretos de uma estrutura de objetos
    como argumentos. Esses métodos podem ter os mesmos nomes se o
    programa é escrito em uma linguagem que suporta sobrecarregamento,
    mas o tipo dos parâmetros devem ser diferentes.

2.  Cada **Visitante Concreto** implementa diversas versões do mesmo
    comportamento, feitos sob medida para diferentes elementos concretos
    de classes.

3.  A interface **Elemento** declara um método para "aceitar"
    visitantes. Esse método deve ter um parâmetro declarado com o tipo
    da interface do visitante.

4.  Cada **Elemento Concreto** deve implementar o método de aceitação. O
    propósito desse método é redirecionar a chamada para o método
    visitante apropriado que corresponde com a atual classe elemento.
    Esteja atento que mesmo se uma classe elemento base implemente esse
    método, todas as subclasses deve ainda sobrescrever esse método em
    suas próprias classes e chamar o método apropriado no objeto
    visitante.

5.  O **Cliente** geralmente representa uma coleção de outros objetos
    complexos (por exemplo, uma árvore [Composite](composite.html)).
    Geralmente, os clientes não estão cientes de todas as classes
    elemento concretas porque eles trabalham com objetos daquela coleção
    através de uma interface abstrata.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Neste exemplo, o padrão **Visitor** adiciona suporte a exportação XML
para a hierarquia de classe de formas geométricas.

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/example9405.png?id=d66acd1b9096c47db17ab3bb82b54a59"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/diagrams/visitor/example.png?id=d66acd1b9096c47db17ab3bb82b54a59 560w,/images/patterns/diagrams/visitor/example-1.5x.png?id=534425a20f1128cb81f418cbee30cba8 840w,/images/patterns/diagrams/visitor/example-2x.png?id=f44438b98f13fcb50898baefad67ffff 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Exemplo de estrutura do padrão Visitor" />
<figcaption><p>Exportando vários tipos de objetos para o formato XML
atráves do objeto visitante.</p></figcaption>
</figure>

<figure class="code">
<pre class="code" lang="pseudocode"><code>// O elemento interface declara um método `accept` que toma a
// interface do visitante base como um argumento.
interface Shape is
    method move(x, y)
    method draw()
    method accept(v: Visitor)

// Cada classe concreta de elemento deve implementar o método
// `accept` de tal maneira que ele chama o método visitante que
// corresponde com a classe do elemento.
class Dot implements Shape is
    // ...

    // Observe que nós estamos chamando `visitDot`, que coincide
    // com o nome da classe atual. Dessa forma nós permitimos
    // que o visitante saiba a classe do elemento com o qual ele
    // trabalha.
    method accept(v: Visitor) is
        v.visitDot(this)

class Circle implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitCircle(this)

class Rectangle implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitRectangle(this)

class CompoundShape implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitCompoundShape(this)


// A interface visitante declara um conjunto de métodos
// visitantes que correspondem com as classes elemento. A
// assinatura de um método visitante permite que o visitante
// identifique a classe exata do elemento com o qual ele está
// lidando.
interface Visitor is
    method visitDot(d: Dot)
    method visitCircle(c: Circle)
    method visitRectangle(r: Rectangle)
    method visitCompoundShape(cs: CompoundShape)

// Visitantes concretos implementam várias versões do mesmo
// algoritmo, que pode trabalhar com todas as classes elemento
// concretas.
//
// Você pode usufruir do maior benefício do padrão Visitor
// quando estiver usando ele com uma estrutura de objeto
// complexa, tal como uma árvore composite. Neste caso, pode ser
// útil armazenar algum estado intermediário do algoritmo
// enquanto executa os métodos visitantes sobre vários objetos
// da estrutura.
class XMLExportVisitor implements Visitor is
    method visitDot(d: Dot) is
        // Exporta a ID do dot (ponto) e suas coordenadas de
        // centro.

    method visitCircle(c: Circle) is
        // Exporta a ID do circle (círculo), coordenadas do
        // centro, e raio.


    method visitRectangle(r: Rectangle) is
        // Exporta a ID do retângulo, coordenadas do topo à
        // esquerda, largura e altura.

    method visitCompoundShape(cs: CompoundShape) is
        // Exporta a ID da forma bem como a lista de ID dos seus
        // filhos.


// O código cliente pode executar operações visitantes sobre
// quaisquer conjuntos de elementos sem saber suas classes
// concretas. A operação accept (aceitar) direciona a chamada
// para a operação apropriada no objeto visitante.
class Application is
    field allShapes: array of Shapes

    method export() is
        exportVisitor = new XMLExportVisitor()

        foreach (shape in allShapes) do
            shape.accept(exportVisitor)</code></pre>
</figure>

Se você está se perguntando por que precisamos do método `aceitar` neste
exemplo, meu artigo [Visitor e Double
Dispatch](visitor-double-dispatch.html) responde essa dúvida em
detalhes.
:::

:::::::::: {.section .applicability-container}
##  Aplicabilidade {#applicability}

::::::::: applicability
::: applicability-problem
Utilize o Visitor quando você precisa fazer uma operação em todos os
elementos de uma estrutura de objetos complexa (por exemplo, uma árvore
de objetos).
:::

::: applicability-solution
O padrão Visitor permite que você execute uma operação sobre um conjunto
de objetos com diferentes classes ao ter o objeto visitante
implementando diversas variantes da mesma operação, que correspondem a
todas as classes alvo.
:::

::: applicability-problem
Utilize o Visitor para limpar a lógica de negócio de comportamentos
auxiliares.
:::

::: applicability-solution
O padrão permite que você torne classes primárias de sua aplicação mais
focadas em seu trabalho principal ao extrair todos os comportamentos em
um conjunto de classes visitantes.
:::

::: applicability-problem
Utilize o padrão quando um comportamento faz sentido apenas dentro de
algumas classes de uma uma hierarquia de classe, mas não em outras.
:::

::: applicability-solution
Você pode extrair esse comportamento para uma classe visitante separada
e implementar somente aqueles métodos visitantes que aceitam objetos de
classes relevantes, deixando o resto vazio.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Como implementar {#checklist}

1.  Declare a interface da visitante com um conjunto de métodos
    "visitando", um para cada classe elemento concreta que existe no
    programa.

2.  Declare a interface elemento. Se você está trabalhando com uma
    hierarquia de classes elemento existente, adicione o método de
    "aceitação" para a classe base da hierarquia. Esse método deve
    aceitar um objeto visitante como um argumento.

3.  Implemente os métodos de aceitação em todas as classes elemento
    concretas. Esses métodos devem simplesmente redirecionar a chamada
    para um método visitante no objeto visitante que está vindo e que
    coincide com a classe do elemento atual.

4.  As classes elemento devem trabalhar apenas com visitantes através da
    interface do visitante. Os visitantes, contudo, devem estar cientes
    de todas as classes elemento concretas referenciadas como tipos de
    parâmetros dos métodos visitantes.

5.  Para cada comportamento que não possa ser implementado dentro da
    hierarquia do elemento, crie uma nova classe visitante concreta e
    implemente todos os métodos visitantes.

    Você pode encontrar uma situação onde o visitante irá necessitar
    acesso para alguns membros privados da classe elemento. Neste caso,
    você pode ou fazer desses campos ou métodos públicos, violando o
    encapsulamento do elemento, ou aninhando a classe visitante na
    classe elemento. Está última só é possível se você tiver sorte e
    estiver trabalhando com uma linguagem de programação que suporta
    classes aninhadas.

6.  O cliente deve criar objetos visitantes e passá-los para os
    elementos através dos métodos de "aceitação".
:::

:::::: {.section .pros-cons}
##  Prós e contras {#pros-cons}

::::: row
::: col-sm-6
-    *Princípio aberto/fechado*. Você pode introduzir um novo
    comportamento que pode funcionar com objetos de diferentes classes
    sem mudar essas classes.
-    *Princípio de responsabilidade única*. Você pode mover múltiplas
    versões do mesmo comportamento para dentro da mesma classe.
-    Um objeto visitante pode acumular algumas informações úteis
    enquanto trabalha com vários objetos. Isso pode ser interessante
    quando você quer percorrer algum objeto de estrutura complexa, tais
    como um objeto árvore, e aplicar o visitante para cada objeto da
    estrutura.
:::

::: col-sm-6
-    Você precisa atualizar todos os visitantes a cada vez que a classe
    é adicionada ou removida da hierarquia de elementos.
-    Visitantes podem não ter seu acesso permitido para campos e métodos
    privados dos elementos que eles deveriam estar trabalhando.
:::
:::::
::::::

::: {.section .relations}
##  Relações com outros padrões {#relations}

-   Você pode tratar um [Visitor](visitor.html) como uma poderosa versão
    do padrão [Command](command.html). Seus objetos podem executar
    operações sobre vários objetos de diferentes classes.

-   Você pode usar o [Visitor](visitor.html) para executar uma operação
    sobre uma árvore [Composite](composite.html) inteira.

-   Você pode usar o [Visitor](visitor.html) junto com o
    [Iterator](iterator.html) para percorrer uma estrutura de dados
    complexas e executar alguma operação sobre seus elementos, mesmo se
    eles todos tenham classes diferentes.
:::

::: {.section .implementations}
##  Exemplos de código {#implementations}

[![Visitor em
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](visitor/csharp/example.html "Visitor em C#"){.prog-lang-link}
[![Visitor em
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](visitor/cpp/example.html "Visitor em C++"){.prog-lang-link}
[![Visitor em
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](visitor/go/example.html "Visitor em Go"){.prog-lang-link}
[![Visitor em
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](visitor/java/example.html "Visitor em Java"){.prog-lang-link}
[![Visitor em
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](visitor/php/example.html "Visitor em PHP"){.prog-lang-link}
[![Visitor em
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](visitor/python/example.html "Visitor em Python"){.prog-lang-link}
[![Visitor em
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](visitor/ruby/example.html "Visitor em Ruby"){.prog-lang-link}
[![Visitor em
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](visitor/rust/example.html "Visitor em Rust"){.prog-lang-link}
[![Visitor em
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](visitor/swift/example.html "Visitor em Swift"){.prog-lang-link}
[![Visitor em
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](visitor/typescript/example.html "Visitor em TypeScript"){.prog-lang-link}
:::

## Conteúdo Adicional

-   Confuso com o por que de não podermos simplesmente substituir o
    padrão Visitor com o sobrecarregamento de método? Leia meu artigo
    [Visitor e Double Dispatch](visitor-double-dispatch.html) para
    aprender mais sobre os detalhes sórdidos a respeito.

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

[Visitor e Double Dispatch []{.fa
.fa-arrow-right}](visitor-double-dispatch.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Template Method](template-method.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::
