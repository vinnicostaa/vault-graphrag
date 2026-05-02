:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
criacionais](creational-patterns.html){.type}
:::

# Prototype {#prototype .title}

::: aka
Também conhecido como:
[Protótipo, ]{style="display:inline-block"}[Clone]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

O **Prototype** é um padrão de projeto criacional que permite copiar
objetos existentes sem fazer seu código ficar dependente de
suas classes.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype3cb9.png?id=e912b1ada20bbf7b2ffc09e93b9fab20"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/prototype/prototype.png?id=e912b1ada20bbf7b2ffc09e93b9fab20 640w,/images/patterns/content/prototype/prototype-1.5x.png?id=a0f76894fb657783b65b9ed899857468 960w,/images/patterns/content/prototype/prototype-2x.png?id=670789c80c8a114e25838ede2da4a881 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Padrão de Desing Prototype" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Digamos que você tenha um objeto, e você quer criar uma cópia exata
dele. Como você o faria? Primeiro, você tem que criar um novo objeto da
mesma classe. Então você terá que ir por todos os campos do objeto
original e copiar seus valores para o novo objeto.

Legal! Mas tem uma pegadinha. Nem todos os objetos podem ser copiados
dessa forma porque alguns campos de objeto podem ser privados e não
serão visíveis fora do próprio objeto.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-1-pt-br07a5.png?id=0711b5299e43efa41db833251fbca3a2"
style="aspect-ratio: 2.08;"
srcset="/images/patterns/content/prototype/prototype-comic-1-pt-br.png?id=0711b5299e43efa41db833251fbca3a2 579w,/images/patterns/content/prototype/prototype-comic-1-pt-br-1.5x.png?id=59ed2084974deb7f74311571b632d73f 868w,/images/patterns/content/prototype/prototype-comic-1-pt-br-2x.png?id=606c24687a30820034c4a130e5ed3092 1157w"
sizes="(max-width: 720px) 100vw, 579px" loading="lazy" width="579"
alt="O que pode dar errado quando copiando algo &quot;do lado de fora&quot;?" />
<figcaption><p>Copiando um objeto “do lado de fora” <a
href="https://vimeo.com/120816425">nem sempre</a>
é possível.</p></figcaption>
</figure>

Há ainda mais um problema com a abordagem direta. Uma vez que você
precisa saber a classe do objeto para criar uma cópia, seu código se
torna dependente daquela classe. Se a dependência adicional não te
assusta, tem ainda outra pegadinha. Algumas vezes você só sabe a
interface que o objeto segue, mas não sua classe concreta, quando, por
exemplo, um parâmetro em um método aceita quaisquer objetos que seguem
uma interface.
:::

::: {.section .solution}
##  Solução {#solution}

O padrão Prototype delega o processo de clonagem para o próprio objeto
que está sendo clonado. O padrão declara um interface comum para todos
os objetos que suportam clonagem. Essa interface permite que você clone
um objeto sem acoplar seu código à classe daquele objeto. Geralmente,
tal interface contém apenas um único método `clonar`.

A implementação do método `clonar` é muito parecida em todas as classes.
O método cria um objeto da classe atual e carrega todos os valores de
campo para do antigo objeto para o novo. Você pode até mesmo copiar
campos privados porque a maioria das linguagens de programação permite
objetos acessar campos privados de outros objetos que pertençam a mesma
classe.

Um objeto que suporta clonagem é chamado de um *protótipo*. Quando seus
objetos têm dúzias de campos e centenas de possíveis configurações,
cloná-los pode servir como uma alternativa à subclasses.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-2-pt-br3540.png?id=7219e5245c5bd94d629c213448c6995c"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/content/prototype/prototype-comic-2-pt-br.png?id=7219e5245c5bd94d629c213448c6995c 343w,/images/patterns/content/prototype/prototype-comic-2-pt-br-1.5x.png?id=bbdd46357b19e931bc92b78854c5843d 515w,/images/patterns/content/prototype/prototype-comic-2-pt-br-2x.png?id=048706f8f926adac09825d661adc75fe 687w"
sizes="(max-width: 720px) 100vw, 343px" loading="lazy" width="343"
alt="Protótipos pré-construidos" />
<figcaption><p>Pré construir protótipos pode ser uma alternativa
às subclasses.</p></figcaption>
</figure>

Funciona assim: você cria um conjunto de objetos, configurados de
diversas formas. Quando você precisa um objeto parecido com o que você
configurou, você apenas clona um protótipo ao invés de construir um novo
objeto a partir do nada.
:::

::: {.section .analogy}
##  Analogia com o mundo real {#analogy}

Na vida real, os protótipos são usados para fazer diversos testes antes
de se começar uma produção em massa de um produto. Contudo, nesse caso,
os protótipos não participam de qualquer produção, ao invés disso fazem
um papel passivo.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-3-pt-brbf16.png?id=830f953552ee356f2a88494ecac9d0ea"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/prototype/prototype-comic-3-pt-br.png?id=830f953552ee356f2a88494ecac9d0ea 600w,/images/patterns/content/prototype/prototype-comic-3-pt-br-1.5x.png?id=f524da8d75880d7a28469a6eb7fe4857 900w,/images/patterns/content/prototype/prototype-comic-3-pt-br-2x.png?id=9bed7de74c04ace642c8ab3c97e0c9bd 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="A divisão celular" />
<figcaption><p>A divisão de uma célula.</p></figcaption>
</figure>

Já que protótipos industriais não se copiam por conta própria, uma
analogia ao padrão é o processo de divisão celular chamado mitose
(biologia, lembra?). Após a divisão mitótica, um par de células
idênticas são formadas. A célula original age como um protótipo e tem um
papel ativo na criação da cópia.
:::

:::::::: {.section .structure-container}
##  Estrutura {#structure}

::::::: structure
#### Implementação básica

:::: prototype-normal
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/structure87f6.png?id=088102c5e9785ff45debbbce86f4df81"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.25;"
srcset="/images/patterns/diagrams/prototype/structure.png?id=088102c5e9785ff45debbbce86f4df81 500w,/images/patterns/diagrams/prototype/structure-1.5x.png?id=beec6a1a5242268e10e39f9fdc0b894b 750w,/images/patterns/diagrams/prototype/structure-2x.png?id=ba75079f42f08028ae4cdbda0cfecc26 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="A estrutura de um padrão de projeto Prototype" /><img
src="../../images/patterns/diagrams/prototype/structure-indexed6499.png?id=0e1c809842f5c43aca0541a2eba1f844"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.27;"
srcset="/images/patterns/diagrams/prototype/structure-indexed.png?id=0e1c809842f5c43aca0541a2eba1f844 520w,/images/patterns/diagrams/prototype/structure-indexed-1.5x.png?id=05b072b5b83f5ff1024a7ba448ea9d59 780w,/images/patterns/diagrams/prototype/structure-indexed-2x.png?id=74584ac729c0f6b49d2a97a53cc266ff 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="A estrutura de um padrão de projeto Prototype" />
</figure>
:::
::::

1.  A interface **Protótipo** declara os métodos de clonagem. Na maioria
    dos casos é apenas um método `clonar`.

2.  A classe **Protótipo Concreta** implementa o método de clonagem.
    Além de copiar os dados do objeto original para o clone, esse método
    também pode lidar com alguns casos específicos do processo de
    clonagem relacionados a clonar objetos ligados, desfazendo
    dependências recursivas, etc.

3.  O **Cliente** pode produzir uma cópia de qualquer objeto que segue a
    interface do protótipo.

#### Implementação do registro do protótipo

:::: prototype-pool
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/structure-prototype-cache9d19.png?id=609c2af5d14ed55dcbb218a00f98e7d5"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.15;"
srcset="/images/patterns/diagrams/prototype/structure-prototype-cache.png?id=609c2af5d14ed55dcbb218a00f98e7d5 550w,/images/patterns/diagrams/prototype/structure-prototype-cache-1.5x.png?id=8ca0b829185d49c5acbe19966a0659d4 825w,/images/patterns/diagrams/prototype/structure-prototype-cache-2x.png?id=a1e4514bbcc9b10968b856f19b407105 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="O registro do prototype" /><img
src="../../images/patterns/diagrams/prototype/structure-prototype-cache-indexed1801.png?id=10a4a84a1a318f59dbc2b806fc936d04"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.12;"
srcset="/images/patterns/diagrams/prototype/structure-prototype-cache-indexed.png?id=10a4a84a1a318f59dbc2b806fc936d04 550w,/images/patterns/diagrams/prototype/structure-prototype-cache-indexed-1.5x.png?id=cb56c95533a4020368c48db9f9e2a37d 825w,/images/patterns/diagrams/prototype/structure-prototype-cache-indexed-2x.png?id=47b99eb7ae51158bdbb31deea4f5e98f 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="O registro do prototype" />
</figure>
:::
::::

1.  O **Registro do Protótipo** fornece uma maneira fácil de acessar
    protótipos de uso frequente. Ele salva um conjunto de objetos pré
    construídos que estão prontos para serem copiados. O registro de
    protótipo mais simples é um hashmap `nome → protótipo`. Contudo, se
    você precisa de um melhor critério de busca que apenas um nome
    simples, você pode construir uma versão muito mais robusta do
    registro.
:::::::
::::::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Neste exemplo, o padrão **Prototype** permite que você produza cópias
exatas de objetos geométricos, sem acoplamento com o código das classes
deles.

<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/examplee491.png?id=47bc6c1058cb100b81e675b5ca6bda6c"
style="aspect-ratio: 1.42;"
srcset="/images/patterns/diagrams/prototype/example.png?id=47bc6c1058cb100b81e675b5ca6bda6c 470w,/images/patterns/diagrams/prototype/example-1.5x.png?id=067f3a2415370fadf5f92aadaa12b5e2 705w,/images/patterns/diagrams/prototype/example-2x.png?id=80393e0df17ae8130e5ada832d494949 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Exemplo da estrutura do padrão Prototype" />
<figcaption><p>Clonando um conjunto de objetos que pertencem à uma
hierarquia de classe.</p></figcaption>
</figure>

Todas as classes de formas seguem a mesma interface, que fornece um
método de clonagem. Uma subclasse pode chamar o método de clonagem
superior antes de copiar seus próprios valores de campo para o objeto
resultante.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Protótipo base.
abstract class Shape is
    field X: int
    field Y: int
    field color: string

    // Um construtor normal.
    constructor Shape() is
        // ...

    // O construtor do protótipo. Um objeto novo é inicializado
    // com valores do objeto existente.
    constructor Shape(source: Shape) is
        this()
        this.X = source.X
        this.Y = source.Y
        this.color = source.color

    // A operação de clonagem retorna uma das subclasses Shape.
    abstract method clone():Shape


// Protótipo concreto. O método de clonagem cria um novo objeto
// e passa ele ao construtor. Até o construtor terminar, ele tem
// uma referência ao clone fresco. Portanto, ninguém tem acesso
// ao clone parcialmente construído. Isso faz com que o clone
// resultante seja consistente.
class Rectangle extends Shape is
    field width: int
    field height: int

    constructor Rectangle(source: Rectangle) is
        // Uma chamada para o construtor pai é necessária para
        // copiar campos privados definidos na classe pai.
        super(source)
        this.width = source.width
        this.height = source.height

    method clone():Shape is
        return new Rectangle(this)


class Circle extends Shape is
    field radius: int

    constructor Circle(source: Circle) is
        super(source)
        this.radius = source.radius

    method clone():Shape is
        return new Circle(this)


// Em algum lugar dentro do código cliente.
class Application is
    field shapes: array of Shape

    constructor Application() is
        Circle circle = new Circle()
        circle.X = 10
        circle.Y = 10
        circle.radius = 20
        shapes.add(circle)

        Circle anotherCircle = circle.clone()
        shapes.add(anotherCircle)
        // A variável `anotherCircle` contém uma cópia exata do
        // objeto `circle`.

        Rectangle rectangle = new Rectangle()
        rectangle.width = 10
        rectangle.height = 20
        shapes.add(rectangle)

    method businessLogic() is
        // O protótipo arrasa porque permite que você produza
        // uma cópia de um objeto sem saber coisa alguma sobre
        // seu tipo.
        Array shapesCopy = new Array of Shapes.

        // Por exemplo, nós não sabemos os elementos exatos no
        // vetor shapes. Tudo que sabemos é que eles são todos
        // shapes. Mas graças ao polimorfismo, quando nós
        // chamamos o método `clone` em um shape, o programa
        // checa sua classe real e executa o método de clonagem
        // apropriado definido naquela classe. É por isso que
        // obtemos clones apropriados ao invés de um conjunto de
        // objetos Shape simples.
        foreach (s in shapes) do
            shapesCopy.add(s.clone())

        // O vetor `shapesCopy` contém cópias exatas dos filhos
        // do vetor `shape`.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Aplicabilidade {#applicability}

::::::: applicability
::: applicability-problem
Utilize o padrão Prototype quando seu código não deve depender de
classes concretas de objetos que você precisa copiar.
:::

::: applicability-solution
Isso acontece muito quando seu código funciona com objetos passados para
você de um código de terceiros através de alguma interface. As classes
concretas desses objetos são desconhecidas, e você não pode depender
delas mesmo que quisesse.

O padrão Prototype fornece o código cliente com uma interface geral para
trabalhar com todos os objetos que suportam clonagem. Essa interface faz
o código do cliente ser independente das classes concretas dos objetos
que ele clona.
:::

::: applicability-problem
Utilize o padrão quando você precisa reduzir o número de subclasses que
somente diferem na forma que inicializam seus respectivos objetos.
Alguém pode ter criado essas subclasses para ser capaz de criar objetos
com uma configuração específica.
:::

::: applicability-solution
O padrão Prototype permite que você use um conjunto de objetos pré
construídos, configurados de diversas formas, como protótipos.

Ao invés de instanciar uma subclasse que coincide com alguma
configuração, o cliente pode simplesmente procurar por um protótipo
apropriado e cloná-lo.
:::
:::::::
::::::::

::: {.section .checklist}
##  Como implementar {#checklist}

1.  Crie uma interface protótipo e declare o método `clonar` nela. Ou
    apenas adicione o método para todas as classes de uma hierarquia de
    classes existente, se você tiver uma.

2.  Uma classe protótipo deve definir o construtor alternativo que
    aceita um objeto daquela classe como um argumento. O construtor deve
    copiar os valores de todos os campos definidos na classe do objeto
    passado para a nova instância recém criada. Se você está mudando uma
    subclasse, você deve chamar o construtor da classe pai para permitir
    que a superclasse lide com a clonagem de seus campos privados.

    Se a sua linguagem de programação não suporta sobrecarregamento de
    métodos, você pode definir um método especial para copiar os dados
    do objeto. O construtor é um local mais conveniente para se fazer
    isso porque ele entrega o objeto resultante logo depois que você
    chamar o operador `new`.

3.  O método de clonagem geralmente consiste em apenas uma linha:
    executando um operador `new` com a versão protótipo do construtor.
    Observe que toda classe deve explicitamente sobrescrever o método de
    clonagem and usar sua própria classe junto com o operador `new`. Do
    contrário, o método de clonagem pode produzir um objeto da classe
    superior.

4.  Opcionalmente, crie um registro protótipo centralizado para
    armazenar um catálogo de protótipos usados com frequência.

    Você pode implementar o registro como uma nova classe factory ou
    colocá-lo na classe protótipo base com um método estático para
    recuperar o protótipo. Esse método deve procurar por um protótipo
    baseado em critérios de busca que o código cliente passou para o
    método. O critério pode ser tanto uma string ou um complexo conjunto
    de parâmetros de busca. Após o protótipo apropriado ser encontrado,
    o registro deve cloná-lo e retornar a cópia para o cliente.

    Por fim, substitua as chamadas diretas para os construtores das
    subclasses com chamadas para o método factory do registro do
    protótipo.
:::

:::::: {.section .pros-cons}
##  Prós e contras {#pros-cons}

::::: row
::: col-sm-6
-    Você pode clonar objetos sem acoplá-los a suas classes concretas.
-    Você pode se livrar de códigos de inicialização repetidos em troca
    de clonar protótipos pré-construídos.
-    Você pode produzir objetos complexos mais convenientemente.
-    Você tem uma alternativa para herança quando lidar com
    configurações pré determinadas para objetos complexos.
:::

::: col-sm-6
-    Clonar objetos complexos que têm referências circulares pode ser
    bem complicado.
:::
:::::
::::::

::: {.section .relations}
##  Relações com outros padrões {#relations}

-   Muitos projetos começam usando o [Factory
    Method](factory-method.html) (menos complicado e mais customizável
    através de subclasses) e evoluem para o [Abstract
    Factory](abstract-factory.html), [Prototype](prototype.html), ou
    [Builder](builder.html) (mais flexíveis, mas mais complicados).

-   Classes [Abstract Factory](abstract-factory.html) são quase sempre
    baseadas em um conjunto de [métodos fábrica](factory-method.html),
    mas você também pode usar o [Prototype](prototype.html) para compor
    métodos dessas classes.

-   O [Prototype](prototype.html) pode ajudar quando você precisa salvar
    cópias de [comandos](command.html) no histórico.

-   Projetos que fazem um uso pesado de [Composite](composite.html) e do
    [Decorator](decorator.html) podem se beneficiar com frequência do
    uso do [Prototype](prototype.html). Aplicando o padrão permite que
    você clone estruturas complexas ao invés de reconstruí-las do zero.

-   O [Prototype](prototype.html) não é baseado em heranças, então ele
    não tem os inconvenientes dela. Por outro lado, o *Prototype*
    precisa de uma inicialização complicada do objeto clonado. O
    [Factory Method](factory-method.html) é baseado em herança mas não
    precisa de uma etapa de inicialização.

-   Algumas vezes o [Prototype](prototype.html) pode ser uma alternativa
    mais simples a um [Memento](memento.html). Isso funciona se o
    objeto, o estado no qual você quer armazenar na história, é
    razoavelmente intuitivo e não tem ligações para recursos externos,
    ou as ligações são fáceis de se restabelecer.

-   As [Fábricas Abstratas](abstract-factory.html),
    [Construtores](builder.html), e [Protótipos](prototype.html) podem
    todos ser implementados como [Singletons](singleton.html).
:::

::: {.section .implementations}
##  Exemplos de código {#implementations}

[![Prototype em
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](prototype/csharp/example.html "Prototype em C#"){.prog-lang-link}
[![Prototype em
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](prototype/cpp/example.html "Prototype em C++"){.prog-lang-link}
[![Prototype em
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](prototype/go/example.html "Prototype em Go"){.prog-lang-link}
[![Prototype em
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](prototype/java/example.html "Prototype em Java"){.prog-lang-link}
[![Prototype em
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](prototype/php/example.html "Prototype em PHP"){.prog-lang-link}
[![Prototype em
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](prototype/python/example.html "Prototype em Python"){.prog-lang-link}
[![Prototype em
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](prototype/ruby/example.html "Prototype em Ruby"){.prog-lang-link}
[![Prototype em
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](prototype/rust/example.html "Prototype em Rust"){.prog-lang-link}
[![Prototype em
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](prototype/swift/example.html "Prototype em Swift"){.prog-lang-link}
[![Prototype em
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](prototype/typescript/example.html "Prototype em TypeScript"){.prog-lang-link}
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

[Singleton []{.fa .fa-arrow-right}](singleton.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Builder](builder.html){.btn .btn-default
rel="prev"}
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
