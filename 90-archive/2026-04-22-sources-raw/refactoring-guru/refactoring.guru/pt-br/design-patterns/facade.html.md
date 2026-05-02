::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
estruturais](structural-patterns.html){.type}
:::

# Facade {#facade .title}

::: aka
Também conhecido como: [Fachada]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

O **Facade** é um padrão de projeto estrutural que fornece uma interface
simplificada para uma biblioteca, um framework, ou qualquer conjunto
complexo de classes.

<figure class="image">
<img
src="../../images/patterns/content/facade/facade721d.png?id=1f4be17305b6316fbd548edf1937ac3b"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/facade/facade.png?id=1f4be17305b6316fbd548edf1937ac3b 640w,/images/patterns/content/facade/facade-1.5x.png?id=a6af5003b243b59990842d24bdaae2df 960w,/images/patterns/content/facade/facade-2x.png?id=b69fce5943703f5f07c0ba38e3baaed0 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Padrão de projeto Facade" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagine que você precisa fazer seu código funcionar com um amplo
conjunto de objetos que pertencem a uma sofisticada biblioteca ou
framework. Normalmente, você precisaria inicializar todos aqueles
objetos, rastrear as dependências, executar métodos na ordem correta, e
assim por diante.

Como resultado, a lógica de negócio de suas classes vai ficar firmemente
acoplada aos detalhes de implementação das classes de terceiros,
tornando difícil compreendê-lo e mantê-lo.
:::

::: {.section .solution}
##  Solução {#solution}

Uma fachada é uma classe que fornece uma interface simples para um
subsistema complexo que contém muitas partes que se movem. Uma fachada
pode fornecer funcionalidades limitadas em comparação com trabalhar com
os subsistemas diretamente. Contudo, ela inclui apenas aquelas
funcionalidades que o cliente se importa.

Ter uma fachada é útil quando você precisa integrar sua aplicação com
uma biblioteca sofisticada que tem dúzias de funcionalidades, mas você
precisa de apenas um pouquinho delas.

Por exemplo, uma aplicação que carrega vídeos curtos engraçados com
gatos para redes sociais poderia potencialmente usar uma biblioteca de
conversão de vídeo profissional. Contudo, tudo que ela realmente precisa
é uma classe com um único método `codificar(nomeDoArquivo, formato)`.
Após criar tal classe e conectá-la com a biblioteca de conversão de
vídeo, você terá sua primeira fachada.
:::

::: {.section .analogy}
##  Analogia com o mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/facade/live-example-pt-brb11b.png?id=8a15add170dece5ecfbbe4b9ece8ed32"
style="aspect-ratio: 2.58;"
srcset="/images/patterns/diagrams/facade/live-example-pt-br.png?id=8a15add170dece5ecfbbe4b9ece8ed32 490w,/images/patterns/diagrams/facade/live-example-pt-br-1.5x.png?id=4de57299c90bc5f7e2efa47dde518404 735w,/images/patterns/diagrams/facade/live-example-pt-br-2x.png?id=49db75cdf0dc26c757ab7f48f6298afd 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Um exemplo de fazer um pedido por telefone" />
<figcaption><p>Fazer pedidos por telefone.</p></figcaption>
</figure>

Quando você liga para uma loja para fazer um pedido, um operador é sua
fachada para todos os serviços e departamentos da loja. O operador
fornece a você uma simples interface de voz para o sistema de pedido,
pagamentos, e vários sistemas de entrega.
:::

::::: {.section .structure-container}
##  Estrutura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/facade/structure2883.png?id=258401362234ac77a2aaf1cde62339e7"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.47;"
srcset="/images/patterns/diagrams/facade/structure.png?id=258401362234ac77a2aaf1cde62339e7 560w,/images/patterns/diagrams/facade/structure-1.5x.png?id=98d134de0c368d382909ba162ec21767 840w,/images/patterns/diagrams/facade/structure-2x.png?id=528ca429555bce293b7c3bd90954e097 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Estrutura do padrão Facade" /><img
src="../../images/patterns/diagrams/facade/structure-indexeddb1c.png?id=2da06d6b850701ea15cf72f9d2642fb8"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.58;"
srcset="/images/patterns/diagrams/facade/structure-indexed.png?id=2da06d6b850701ea15cf72f9d2642fb8 600w,/images/patterns/diagrams/facade/structure-indexed-1.5x.png?id=fad7d667f4d8d9a7d522748bcd37dfde 900w,/images/patterns/diagrams/facade/structure-indexed-2x.png?id=4d181bcf1df5d58c533e6c92461e9571 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Estrutura do padrão Facade" />
</figure>
:::

1.  A **Fachada** fornece um acesso conveniente para uma parte
    particular da funcionalidade do subsistema. Ela sabe onde direcionar
    o pedido do cliente e como operar todas as partes móveis.

2.  Uma classe **Fachada Adicional** pode ser criada para prevenir a
    poluição de uma única fachada com funcionalidades não relevantes que
    podem torná-lo mais uma estrutura complexa. Fachadas adicionais
    podem ser usadas tanto por clientes como por outras fachadas.

3.  O **Subsistema Complexo** consiste em dúzias de objetos variados.
    Para tornar todos eles em algo que signifique alguma coisa, você tem
    que mergulhar fundo nos detalhes de implementação do subsistema,
    tais como objetos de inicialização na ordem correta e suprí-los com
    dados no formato correto.

    As classes do subsistema não estão cientes da existência da fachada.
    Elas operam dentro do sistema e trabalham entre si diretamente.

4.  O **Cliente** usa a fachada ao invés de chamar os objetos do
    subsistema diretamente.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Neste exemplo, o padrão **Facade** simplifica a interação com um
framework complexo de conversão de vídeo.

<figure class="image">
<img
src="../../images/patterns/diagrams/facade/example16bc.png?id=2249d134e3ff83819dfc19032f02eced"
style="aspect-ratio: 1.5;"
srcset="/images/patterns/diagrams/facade/example.png?id=2249d134e3ff83819dfc19032f02eced 570w,/images/patterns/diagrams/facade/example-1.5x.png?id=034ecbe0f3cc108f4ae49d05d1c77dbe 855w,/images/patterns/diagrams/facade/example-2x.png?id=f2c846d74041626c923ff3e8919b68a9 1140w"
sizes="(max-width: 720px) 100vw, 570px" loading="lazy" width="570"
alt="Exemplo da estrutura do padrão Facade" />
<figcaption><p>Um exemplo de isolamento de múltiplas dependências dentro
de uma única classe fachada.</p></figcaption>
</figure>

Ao invés de fazer seu código funcionar com dúzias de classes framework
diretamente, você cria a classe fachada que encapsula aquela
funcionalidade e a esconde do resto do código. Essa estrutura também
ajuda você a minimizar o esforço usando para atualizar para futuras
versões do framework ou substituí-lo por outro. A única coisa que você
precisaria mudar em sua aplicação seria a implementação dos métodos da
fachada.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Essas são algumas das classes de um framework complexo de um
// conversor de vídeo de terceiros. Nós não controlamos aquele
// código, portanto não podemos simplificá-lo.

class VideoFile
// ...

class OggCompressionCodec
// ...

class MPEG4CompressionCodec
// ...

class CodecFactory
// ...

class BitrateReader
// ...

class AudioMixer
// ...


// Nós criamos uma classe fachada para esconder a complexidade
// do framework atrás de uma interface simples. É uma troca
// entre funcionalidade e simplicidade.
class VideoConverter is
    method convert(filename, format):File is
        file = new VideoFile(filename)
        sourceCodec = (new CodecFactory).extract(file)
        if (format == &quot;mp4&quot;)
            destinationCodec = new MPEG4CompressionCodec()
        else
            destinationCodec = new OggCompressionCodec()
        buffer = BitrateReader.read(filename, sourceCodec)
        result = BitrateReader.convert(buffer, destinationCodec)
        result = (new AudioMixer()).fix(result)
        return new File(result)

// As classes da aplicação não dependem de um bilhão de classes
// fornecidas por um framework complexo. Também, se você decidir
// trocar de frameworks, você só precisa reescrever a classe
// fachada.
class Application is
    method main() is
        convertor = new VideoConverter()
        mp4 = convertor.convert(&quot;funny-cats-video.ogg&quot;, &quot;mp4&quot;)
        mp4.save()</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Aplicabilidade {#applicability}

::::::: applicability
::: applicability-problem
Utilize o padrão Facade quando você precisa ter uma interface limitada
mas simples para um subsistema complexo.
:::

::: applicability-solution
Com o passar do tempo, subsistemas ficam mais complexos. Até mesmo
aplicar padrões de projeto tipicamente leva a criação de mais classes.
Um subsistema pode tornar-se mais flexível e mais fácil de se reutilizar
em vários contextos, mas a quantidade de códigos padrão e de
configuração que ele necessita de um cliente cresce cada vez mais. O
Facade tenta consertar esse problema fornecendo um atalho para as
funcionalidades mais usadas do subsistema que corresponde aos
requerimentos do cliente.
:::

::: applicability-problem
Utilize o Facade quando você quer estruturar um subsistema em camadas.
:::

::: applicability-solution
Crie fachadas para definir pontos de entrada para cada nível de um
subsistema. Você pode reduzir o acoplamento entre múltiplos subsistemas
fazendo com que eles se comuniquem apenas através de fachadas.

Por exemplo, vamos retornar ao nosso framework de conversão de vídeo.
Ele pode ser quebrado em duas camadas: relacionados a vídeo e áudio.
Para cada camada, você cria uma fachada e então faz as classes de cada
camada se comunicarem entre si através daquelas fachadas. Essa abordagem
se parece muito com o padrão [Mediator](mediator.html).
:::
:::::::
::::::::

::: {.section .checklist}
##  Como implementar {#checklist}

1.  Verifique se é possível providenciar uma interface mais simples que
    a que o subsistema já fornece. Você está no caminho certo se essa
    interface faz o código cliente independente de muitas classes do
    subsistema.

2.  Declare e implemente essa interface em uma nova classe fachada. A
    fachada deve redirecionar as chamadas do código cliente para os
    objetos apropriados do subsistema. A fachada deve ser responsável
    por inicializar o subsistema e gerenciar seu ciclo de vida a menos
    que o código cliente já faça isso.

3.  Para obter o benefício pleno do padrão, faça todo o código cliente
    se comunicar com o subsistema apenas através da fachada. Agora o
    código cliente fica protegido de qualquer mudança no código do
    subsistema. Por exemplo, quando um subsistema recebe um upgrade para
    uma nova versão, você só precisa modificar o código na fachada.

4.  Se a fachada ficar [grande demais](../smells/large-class.html),
    considere extrair parte de seu comportamento para uma nova e
    refinada classe fachada.
:::

:::::: {.section .pros-cons}
##  Prós e contras {#pros-cons}

::::: row
::: col-sm-6
-    Você pode isolar seu código da complexidade de um subsistema.
:::

::: col-sm-6
-    Uma fachada pode se tornar [um objeto
    deus](https://pt.wikipedia.org/wiki/Objeto_deus) acoplado a todas as
    classes de uma aplicação.
:::
:::::
::::::

::: {.section .relations}
##  Relações com outros padrões {#relations}

-   O [Facade](facade.html) define uma nova interface para objetos
    existentes, enquanto que o [Adapter](adapter.html) tenta fazer uma
    interface existente ser utilizável. O *Adapter* geralmente envolve
    apenas um objeto, enquanto que o *Facade* trabalha com um inteiro
    subsistema de objetos.

-   O [Abstract Factory](abstract-factory.html) pode servir como uma
    alternativa para o [Facade](facade.html) quando você precisa apenas
    esconder do código cliente a forma com que são criados os objetos do
    subsistema.

-   O [Flyweight](flyweight.html) mostra como fazer vários pequenos
    objetos, enquanto o [Facade](facade.html) mostra como fazer um único
    objeto que represente um subsistema inteiro.

-   O [Facade](facade.html) e o [Mediator](mediator.html) têm trabalhos
    parecidos: eles tentam organizar uma colaboração entre classes
    firmemente acopladas.

    -   O *Facade* define uma interface simplificada para um subsistema
        de objetos, mas ele não introduz qualquer nova funcionalidade. O
        próprio subsistema não está ciente da fachada. Objetos dentro do
        subsistema podem se comunicar diretamente.
    -   O *Mediator* centraliza a comunicação entre componentes do
        sistema. Os componentes só sabem do objeto mediador e não se
        comunicam diretamente.

-   Uma classe [fachada](facade.html) pode frequentemente ser
    transformada em uma [singleton](singleton.html) já que um único
    objeto fachada é suficiente na maioria dos casos.

-   O [Facade](facade.html) é parecido como o [Proxy](proxy.html) no
    quesito que ambos colocam em buffer uma entidade complexa e
    inicializam ela sozinhos. Ao contrário do *Facade*, o *Proxy* tem a
    mesma interface que seu objeto de serviço, o que os torna
    intermutáveis.
:::

::: {.section .implementations}
##  Exemplos de código {#implementations}

[![Facade em
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](facade/csharp/example.html "Facade em C#"){.prog-lang-link}
[![Facade em
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](facade/cpp/example.html "Facade em C++"){.prog-lang-link}
[![Facade em
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](facade/go/example.html "Facade em Go"){.prog-lang-link}
[![Facade em
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](facade/java/example.html "Facade em Java"){.prog-lang-link}
[![Facade em
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](facade/php/example.html "Facade em PHP"){.prog-lang-link}
[![Facade em
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](facade/python/example.html "Facade em Python"){.prog-lang-link}
[![Facade em
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](facade/ruby/example.html "Facade em Ruby"){.prog-lang-link}
[![Facade em
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](facade/rust/example.html "Facade em Rust"){.prog-lang-link}
[![Facade em
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](facade/swift/example.html "Facade em Swift"){.prog-lang-link}
[![Facade em
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](facade/typescript/example.html "Facade em TypeScript"){.prog-lang-link}
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

[Flyweight []{.fa .fa-arrow-right}](flyweight.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Decorator](decorator.html){.btn .btn-default
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
