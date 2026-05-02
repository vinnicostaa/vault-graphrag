:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
estruturais](structural-patterns.html){.type}
:::

# Adapter {#adapter .title}

::: aka
Também conhecido como:
[Adaptador, ]{style="display:inline-block"}[Wrapper]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

O **Adapter** é um padrão de projeto estrutural que permite objetos com
interfaces incompatíveis colaborarem entre si.

<figure class="image">
<img
src="../../images/patterns/content/adapter/adapter-pt-br93f8.png?id=05f144d30c63000fbe59e09f29bb488d"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/adapter/adapter-pt-br.png?id=05f144d30c63000fbe59e09f29bb488d 640w,/images/patterns/content/adapter/adapter-pt-br-1.5x.png?id=ffb0c6e4e02fa6d362246a61ba901ce8 960w,/images/patterns/content/adapter/adapter-pt-br-2x.png?id=5ff00eda9fde2ce573a0bc017c5231e7 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Padrão de projeto Adapter" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagine que você está criando uma aplicação de monitoramento do mercado
de ações da bolsa. A aplicação baixa os dados as ações de múltiplas
fontes em formato XML e então mostra gráficos e diagramas maneiros para
o usuário.

Em algum ponto, você decide melhorar a aplicação ao integrar uma
biblioteca de análise de terceiros. Mas aqui está a pegadinha: a
biblioteca só trabalha com dados em formato JSON.

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/problem-pt-br9d52.png?id=5429f5de17156d304a588b7cbaa7ed10"
style="aspect-ratio: 2.41;"
srcset="/images/patterns/diagrams/adapter/problem-pt-br.png?id=5429f5de17156d304a588b7cbaa7ed10 530w,/images/patterns/diagrams/adapter/problem-pt-br-1.5x.png?id=aeaf6eab098717aba046957c25ce8ff2 795w,/images/patterns/diagrams/adapter/problem-pt-br-2x.png?id=17a2cc65a2b1252513f7fc913c1e18cd 1060w"
sizes="(max-width: 720px) 100vw, 530px" loading="lazy" width="530"
alt="A estrutura da aplicação antes da integração com a biblioteca de análises" />
<figcaption><p>Você não pode usar a biblioteca “como ela está” porque
ela espera os dados em um formato que é incompatível com
sua aplicação.</p></figcaption>
</figure>

Você poderia mudar a biblioteca para que ela funcione com XML. Contudo,
isso pode quebrar algum código existente que depende da biblioteca. E
pior, você pode não ter acesso ao código fonte da biblioteca para começo
de conversa, fazendo dessa abordagem uma tarefa impossível.
:::

::: {.section .solution}
##  Solução {#solution}

Você pode criar um *adaptador*. Ele é um objeto especial que converte a
interface de um objeto para que outro objeto possa entendê-lo.

Um adaptador encobre um dos objetos para esconder a complexidade da
conversão acontecendo nos bastidores. O objeto encobrido nem fica ciente
do adaptador. Por exemplo, você pode encobrir um objeto que opera em
metros e quilômetros com um adaptador que converte todos os dados para
unidades imperiais tais como pés e milhas.

Adaptadores podem não só converter dados em vários formatos, mas também
podem ajudar objetos com diferentes interfaces a colaborar. Veja aqui
como funciona:

1.  O adaptador obtém uma interface, compatível com um dos objetos
    existentes.
2.  Usando essa interface, o objeto existente pode chamar os métodos do
    adaptador com segurança.
3.  Ao receber a chamada, o adaptador passa o pedido para o segundo
    objeto, mas em um formato e ordem que o segundo objeto espera.

Algumas vezes é possível criar um adaptador de duas vias que pode
converter as chamadas em ambas as direções.

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/solution-pt-bre9e1.png?id=ffe986cb8e979f54610072f35928d04e"
style="aspect-ratio: 1.51;"
srcset="/images/patterns/diagrams/adapter/solution-pt-br.png?id=ffe986cb8e979f54610072f35928d04e 530w,/images/patterns/diagrams/adapter/solution-pt-br-1.5x.png?id=69db8e22a6e1007ed23e22b2f71f11cb 795w,/images/patterns/diagrams/adapter/solution-pt-br-2x.png?id=90d8e38b7e2dfdd94708a9dc857bcaba 1060w"
sizes="(max-width: 720px) 100vw, 530px" loading="lazy" width="530"
alt="Solução do Adapter" />
</figure>

Vamos voltar à nossa aplicação da bolsa de valores. Para resolver o
dilema dos formatos incompatíveis, você pode criar adaptadores
XML-para-JSON para cada classe da biblioteca de análise que seu código
trabalha diretamente. Então você ajusta seu código para comunicar-se com
a biblioteca através desses adaptadores. Quando um adaptador recebe uma
chamada, ele traduz os dados entrantes XML em uma estrutura JSON e passa
a chamada para os métodos apropriados de um objeto de análise encoberto.
:::

::: {.section .analogy}
##  Analogia com o mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/adapter/adapter-comic-1-pt-br8656.png?id=a33f9306db5a3932525827fe93a9676a"
style="aspect-ratio: 1.78;"
srcset="/images/patterns/content/adapter/adapter-comic-1-pt-br.png?id=a33f9306db5a3932525827fe93a9676a 533w,/images/patterns/content/adapter/adapter-comic-1-pt-br-1.5x.png?id=e89383ed01bd5f711ed23455c5702dfc 800w,/images/patterns/content/adapter/adapter-comic-1-pt-br-2x.png?id=11d68b9a648ab4bb7d5605141c0b85e7 1067w"
sizes="(max-width: 720px) 100vw, 533px" loading="lazy" width="533"
alt="Exemplo do padrão Adapter" />
<figcaption><p>Uma mala antes e depois de uma viagem
ao exterior.</p></figcaption>
</figure>

Quando você viaja do Brasil para a Europa pela primeira vez, você pode
ter uma pequena surpresa quando tenta carregar seu laptop. O plugue e os
padrões de tomadas são diferentes em diferentes países. É por isso que
seu plugue do Brasil não vai caber em uma tomada da Alemanha. O problema
pode ser resolvido usando um adaptador de tomada que tenha o estilo de
tomada Brasileira e o plugue no estilo Europeu.
:::

:::::::: {.section .structure-container}
##  Estrutura {#structure}

::::::: structure
#### Adaptador de objeto

Essa implementação usa o princípio de composição do objeto: o adaptador
implementa a interface de um objeto e encobre o outro. Ele pode ser
implementado em todas as linguagens de programação populares.

:::: adapter-object
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/structure-object-adaptera83b.png?id=33dffbe3aece294162440c7ddd3d5d4f"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.81;"
srcset="/images/patterns/diagrams/adapter/structure-object-adapter.png?id=33dffbe3aece294162440c7ddd3d5d4f 580w,/images/patterns/diagrams/adapter/structure-object-adapter-1.5x.png?id=c1b8a87b74fc4ce5639212fe19ee06fe 870w,/images/patterns/diagrams/adapter/structure-object-adapter-2x.png?id=03e8052e168c962d6bc369bbb13b0945 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="Estrutura de um padrão de projeto Adapter (o objeto adaptador)" /><img
src="../../images/patterns/diagrams/adapter/structure-object-adapter-indexedd6ac.png?id=a20b311948b361a058097e5bcdbf067a"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.76;"
srcset="/images/patterns/diagrams/adapter/structure-object-adapter-indexed.png?id=a20b311948b361a058097e5bcdbf067a 600w,/images/patterns/diagrams/adapter/structure-object-adapter-indexed-1.5x.png?id=72a1c8fde064c4915379fb34a1a66881 900w,/images/patterns/diagrams/adapter/structure-object-adapter-indexed-2x.png?id=759771515f08d74d53cf4fe500f814a3 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Estrutura de um padrão de projeto Adapter (o objeto adaptador)" />
</figure>
:::
::::

1.  O **Cliente** é uma classe que contém a lógica de negócio do
    programa existente.

2.  A **Interface do Cliente** descreve um protocolo que outras classes
    devem seguir para ser capaz de colaborar com o código cliente.

3.  O **Serviço** é alguma classe útil (geralmente de terceiros ou
    código legado). O cliente não pode usar essa classe diretamente
    porque ela tem uma interface incompatível.

4.  O **Adaptador** é uma classe que é capaz de trabalhar tanto com o
    cliente quanto o serviço: ela implementa a interface do cliente
    enquanto encobre o objeto do serviço. O adaptador recebe chamadas do
    cliente através da interface do cliente e as traduz em chamadas para
    o objeto encobrido do serviço em um formato que ele possa entender.

5.  O código cliente não é acoplado à classe concreta do adaptador desde
    que ele trabalhe com o adaptador através da interface do cliente.
    Graças a isso, você pode introduzir novos tipos de adaptadores no
    programa sem quebrar o código cliente existente. Isso pode ser útil
    quando a interface de uma classe de serviço é mudada ou substituída:
    você pode apenas criar uma nova classe adaptador sem mudar o código
    cliente.

#### Adaptador de classe

Essa implementação utiliza herança: o adaptador herda interfaces de
ambos os objetos ao mesmo tempo. Observe que essa abordagem só pode ser
implementada em linguagens de programação que suportam herança múltipla,
tais como C++.

:::: adapter-class
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/structure-class-adapterf043.png?id=e1c60240508146ed3b98ac562cc8e510"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.72;"
srcset="/images/patterns/diagrams/adapter/structure-class-adapter.png?id=e1c60240508146ed3b98ac562cc8e510 550w,/images/patterns/diagrams/adapter/structure-class-adapter-1.5x.png?id=299a79bdfa10ac53213ba02408255430 825w,/images/patterns/diagrams/adapter/structure-class-adapter-2x.png?id=ddca3e3e4d972b7c58207daba8d24866 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Padrão de projeto Adapter (classe adaptadora)" /><img
src="../../images/patterns/diagrams/adapter/structure-class-adapter-indexedd30e.png?id=250b5c485a7dfba7c16b89a9201538fb"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.72;"
srcset="/images/patterns/diagrams/adapter/structure-class-adapter-indexed.png?id=250b5c485a7dfba7c16b89a9201538fb 550w,/images/patterns/diagrams/adapter/structure-class-adapter-indexed-1.5x.png?id=b56d736f8076d34b1896de0a2b22a219 825w,/images/patterns/diagrams/adapter/structure-class-adapter-indexed-2x.png?id=9ae1182ef2a34d2ea65f4b4f94a4019e 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Padrão de projeto Adapter (classe adaptadora)" />
</figure>
:::
::::

1.  A **Classe Adaptador** não precisa encobrir quaisquer objetos porque
    ela herda os comportamentos tanto do cliente como do serviço. A
    adaptação acontece dentro dos métodos sobrescritos. O adaptador
    resultante pode ser usado em lugar de uma classe cliente existente.
:::::::
::::::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Esse exemplo do padrão **Adapter** é baseado no conflito clássico entre
pinos quadrados e buracos redondos.

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/examplef4c9.png?id=9d2b6857ce256f2c669383ce4df3d0aa"
style="aspect-ratio: 1.68;"
srcset="/images/patterns/diagrams/adapter/example.png?id=9d2b6857ce256f2c669383ce4df3d0aa 640w,/images/patterns/diagrams/adapter/example-1.5x.png?id=76e68b9cea3b371e465e81587e57e5d9 960w,/images/patterns/diagrams/adapter/example-2x.png?id=0ac62d1bc151e8ce6abad8e8502756cf 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Exemplo da estrutura do padrão Adapter" />
<figcaption><p>Adaptando pinos quadrados para
buracos redondos.</p></figcaption>
</figure>

O adaptador finge ser um pino redondo, com um raio igual a metade do
diâmetro do quadrado (em outras palavras, o raio do menor círculo que
pode acomodar o pino quadrado).

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Digamos que você tenha duas classes com interfaces
// compatíveis: RoundHole (Buraco Redondo) e RoundPeg (Pino
// Redondo).
class RoundHole is
    constructor RoundHole(radius) { ... }

    method getRadius() is
        // Retorna o raio do buraco.

    method fits(peg: RoundPeg) is
        return this.getRadius() &gt;= peg.getRadius()

class RoundPeg is
    constructor RoundPeg(radius) { ... }

    method getRadius() is
        // Retorna o raio do pino.


// Mas tem uma classe incompatível: SquarePeg (Pino Quadrado).
class SquarePeg is
    constructor SquarePeg(width) { ... }

    method getWidth() is
        // Retorna a largura do pino quadrado.


// Uma classe adaptadora permite que você encaixe pinos
// quadrados em buracos redondos. Ela estende a classe RoundPeg
// para permitir que objetos do adaptador ajam como pinos
// redondos.
class SquarePegAdapter extends RoundPeg is
    // Na verdade, o adaptador contém uma instância da classe
    // SquarePeg.
    private field peg: SquarePeg

    constructor SquarePegAdapter(peg: SquarePeg) is
        this.peg = peg

    method getRadius() is
        // O adaptador finge que é um pino redondo com um raio
        // que encaixaria o pino quadrado que o adaptador está
        // envolvendo.
        return peg.getWidth() * Math.sqrt(2) / 2


// Em algum lugar no código cliente.
hole = new RoundHole(5)
rpeg = new RoundPeg(5)
hole.fits(rpeg) // true

small_sqpeg = new SquarePeg(5)
large_sqpeg = new SquarePeg(10)
// Isso não vai compilar (tipos incompatíveis).
hole.fits(small_sqpeg)

small_sqpeg_adapter = new SquarePegAdapter(small_sqpeg)
large_sqpeg_adapter = new SquarePegAdapter(large_sqpeg)
hole.fits(small_sqpeg_adapter) // true
hole.fits(large_sqpeg_adapter) // false</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Aplicabilidade {#applicability}

::::::: applicability
::: applicability-problem
Utilize a classe Adaptador quando você quer usar uma classe existente,
mas sua interface não for compatível com o resto do seu código.
:::

::: applicability-solution
O padrão Adapter permite que você crie uma classe de meio termo que
serve como um tradutor entre seu código e a classe antiga, uma classe de
terceiros, ou qualquer outra classe com uma interface estranha.
:::

::: applicability-problem
Utilize o padrão quando você quer reutilizar diversas subclasses
existentes que não possuam alguma funcionalidade comum que não pode ser
adicionada a superclasse.
:::

::: applicability-solution
Você pode estender cada subclasse e colocar a funcionalidade faltante
nas novas classes filhas. Contudo, você terá que duplicar o código em
todas as novas classes, o que [cheira muito
mal](../smells/duplicate-code.html).

Uma solução muito mais elegante seria colocar a funcionalidade faltante
dentro da classe adaptadora. Então você encobriria os objetos com as
funcionalidades faltantes dentro do adaptador, ganhando tais
funcionalidades de forma dinâmica. Para isso funcionar, as classes alvo
devem ter uma interface em comum, e o campo do adaptador deve seguir
aquela interface. Essa abordagem se parece muito com o padrão
[Decorator](decorator.html).
:::
:::::::
::::::::

::: {.section .checklist}
##  Como implementar {#checklist}

1.  Certifique-se que você tem ao menos duas classes com interfaces
    incompatíveis:

    -   Uma classe *serviço* útil, que você não pode modificar (quase
        sempre de terceiros, antiga, ou com muitas dependências
        existentes).
    -   Uma ou mais classes *cliente* que seriam beneficiadas com o uso
        da classe serviço.

2.  Declare a interface cliente e descreva como o cliente se comunica
    com o serviço.

3.  Cria a classe adaptadora e faça-a seguir a interface cliente. Deixe
    todos os métodos vazios por enquanto.

4.  Adicione um campo para a classe do adaptador armazenar uma
    referência ao objeto do serviço. A prática comum é inicializar esse
    campo via o construtor, mas algumas vezes é mais conveniente
    passá-lo para o adaptador ao chamar seus métodos.

5.  Um por um, implemente todos os métodos da interface cliente na
    classe adaptadora. O adaptador deve delegar a maioria do trabalho
    real para o objeto serviço, lidando apenas com a conversão da
    interface ou formato dos dados.

6.  Os Clientes devem usar o adaptador através da interface cliente.
    Isso irá permitir que você mude ou estenda o adaptador sem afetar o
    código cliente.
:::

:::::: {.section .pros-cons}
##  Prós e contras {#pros-cons}

::::: row
::: col-sm-6
-    *Princípio de responsabilidade única*. Você pode separar a
    conversão de interface ou de dados da lógica primária do negócio do
    programa.
-    *Princípio aberto/fechado*. Você pode introduzir novos tipos de
    adaptadores no programa sem quebrar o código cliente existente,
    desde que eles trabalhem com os adaptadores através da interface
    cliente.
:::

::: col-sm-6
-    A complexidade geral do código aumenta porque você precisa
    introduzir um conjunto de novas interfaces e classes. Algumas vezes
    é mais simples mudar a classe serviço para que ela se adeque com o
    resto do seu código.
:::
:::::
::::::

::: {.section .relations}
##  Relações com outros padrões {#relations}

-   O [Bridge](bridge.html) é geralmente definido com antecedência,
    permitindo que você desenvolva partes de uma aplicação
    independentemente umas das outras. Por outro lado, o
    [Adapter](adapter.html) é comumente usado em aplicações existentes
    para fazer com que classes incompatíveis trabalhem bem juntas.

-   O [Adapter](adapter.html) fornece uma interface completamente
    diferente para acessar um objeto existente. Por outro lado, com o
    padrão [Decorator](decorator.html), a interface permanece a mesma ou
    é estendida. Além disso, o *Decorator* oferece suporte à composição
    recursiva, o que não é possível quando você usa o *Adapter*.

-   Com [Adapter](adapter.html), você acessa um objeto existente por
    meio de uma interface diferente. Com [Proxy](proxy.html), a
    interface permanece a mesma. Com [Decorator](decorator.html), você
    acessa o objeto por meio de uma interface aprimorada.

-   O [Facade](facade.html) define uma nova interface para objetos
    existentes, enquanto que o [Adapter](adapter.html) tenta fazer uma
    interface existente ser utilizável. O *Adapter* geralmente envolve
    apenas um objeto, enquanto que o *Facade* trabalha com um inteiro
    subsistema de objetos.

-   O [Bridge](bridge.html), [State](state.html),
    [Strategy](strategy.html) (e de certa forma o
    [Adapter](adapter.html)) têm estruturas muito parecidas. De fato,
    todos esses padrões estão baseados em composição, o que é delegar o
    trabalho para outros objetos. Contudo, eles todos resolvem problemas
    diferentes. Um padrão não é apenas uma receita para estruturar seu
    código de uma maneira específica. Ele também pode comunicar a outros
    desenvolvedores o problema que o padrão resolve.
:::

::: {.section .implementations}
##  Exemplos de código {#implementations}

[![Adapter em
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](adapter/csharp/example.html "Adapter em C#"){.prog-lang-link}
[![Adapter em
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](adapter/cpp/example.html "Adapter em C++"){.prog-lang-link}
[![Adapter em
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](adapter/go/example.html "Adapter em Go"){.prog-lang-link}
[![Adapter em
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](adapter/java/example.html "Adapter em Java"){.prog-lang-link}
[![Adapter em
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](adapter/php/example.html "Adapter em PHP"){.prog-lang-link}
[![Adapter em
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](adapter/python/example.html "Adapter em Python"){.prog-lang-link}
[![Adapter em
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](adapter/ruby/example.html "Adapter em Ruby"){.prog-lang-link}
[![Adapter em
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](adapter/rust/example.html "Adapter em Rust"){.prog-lang-link}
[![Adapter em
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](adapter/swift/example.html "Adapter em Swift"){.prog-lang-link}
[![Adapter em
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](adapter/typescript/example.html "Adapter em TypeScript"){.prog-lang-link}
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

[Bridge []{.fa .fa-arrow-right}](bridge.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Padrões
estruturais](structural-patterns.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::
