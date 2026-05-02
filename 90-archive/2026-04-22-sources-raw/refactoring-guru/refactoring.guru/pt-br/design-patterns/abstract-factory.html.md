::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
criacionais](creational-patterns.html){.type}
:::

# Abstract Factory {#abstract-factory .title}

::: aka
Também conhecido como: [Fábrica abstrata]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

O **Abstract Factory** é um padrão de projeto criacional que permite que
você produza famílias de objetos relacionados sem ter que especificar
suas classes concretas.

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-pt-br0bb7.png?id=9d3c1f21c36d3014a29809e6c36e3861"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-pt-br.png?id=9d3c1f21c36d3014a29809e6c36e3861 640w,/images/patterns/content/abstract-factory/abstract-factory-pt-br-1.5x.png?id=30194a4333267f5d60364f8d607ab59e 960w,/images/patterns/content/abstract-factory/abstract-factory-pt-br-2x.png?id=cf558dbebac15889e5b9da321db58a1c 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Padrão Abstract Factory" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagine que você está criando um simulador de loja de mobílias. Seu
código consiste de classes que representam:

1.  Uma família de produtos relacionados, como: `Cadeira` + `Sofá` +
    `MesaDeCentro`.

2.  Várias variantes dessa família. Por exemplo, produtos `Cadeira` +
    `Sofá` + `MesaDeCentro` estão disponíveis nessas variantes:
    `Moderno`, `Vitoriano`, `ArtDeco`.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/problem-pt-br2c6c.png?id=98a0309f5f32f1636a423aa9c979595d"
style="aspect-ratio: 1.4;"
srcset="/images/patterns/diagrams/abstract-factory/problem-pt-br.png?id=98a0309f5f32f1636a423aa9c979595d 420w,/images/patterns/diagrams/abstract-factory/problem-pt-br-1.5x.png?id=f97579da958e2b68f72c71fe84ecce1b 630w,/images/patterns/diagrams/abstract-factory/problem-pt-br-2x.png?id=28605835eb52c2cae2858caf85d95c0e 840w"
sizes="(max-width: 720px) 100vw, 420px" loading="lazy" width="420"
alt="Famílias de produtos e suas variantes." />
<figcaption><p>Famílias de produtos e suas variantes.</p></figcaption>
</figure>

Você precisa de um jeito de criar objetos de mobília individuais para
que eles combinem com outros objetos da mesma família. Os clientes ficam
muito bravos quando recebem mobília que não combina.

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-comic-1-pt-br653b.png?id=32fc10f3f88357e93580ac001b3b41af"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-comic-1-pt-br.png?id=32fc10f3f88357e93580ac001b3b41af 600w,/images/patterns/content/abstract-factory/abstract-factory-comic-1-pt-br-1.5x.png?id=5addcca62a15f41503c05412a544b33c 900w,/images/patterns/content/abstract-factory/abstract-factory-comic-1-pt-br-2x.png?id=b4fa8df044aac32a458e654e14897d82 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>Um sofá no estilo Moderno não combina com cadeiras de
estilo Vitoriano</p></figcaption>
</figure>

E ainda, você não quer mudar o código existente quando adiciona novos
produtos ou famílias de produtos ao programa. Os vendedores de mobílias
atualizam seus catálogos com frequência e você não vai querer mudar o
código base cada vez que isso acontece.
:::

::: {.section .solution}
##  Solução {#solution}

A primeira coisa que o padrão Abstract Factory sugere é declarar
explicitamente interfaces para cada produto distinto da família de
produtos (ex: cadeira, sofá ou mesa de centro). Então você pode fazer
todas as variantes dos produtos seguirem essas interfaces. Por exemplo,
todas as variantes de cadeira podem implementar a interface `Cadeira`;
todas as variantes de mesa de centro podem implementar a interface
`MesaDeCentro`, e assim por diante.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/solution1a1ac.png?id=71f2018d8bb443b9cce90d57110675e3"
style="aspect-ratio: 1.5;"
srcset="/images/patterns/diagrams/abstract-factory/solution1.png?id=71f2018d8bb443b9cce90d57110675e3 420w,/images/patterns/diagrams/abstract-factory/solution1-1.5x.png?id=4366e971c850514cde5d33cb7956de8b 630w,/images/patterns/diagrams/abstract-factory/solution1-2x.png?id=eacec286153e058db9255d26541c0529 840w"
sizes="(max-width: 720px) 100vw, 420px" loading="lazy" width="420"
alt="A hierarquia da classe Cadeira" />
<figcaption><p>Todas as variantes do mesmo objeto podem ser movidas para
uma mesma hierarquia de classe.</p></figcaption>
</figure>

O próximo passo é declarar a *fábrica abstrata*---uma interface com uma
lista de métodos de criação para todos os produtos que fazem parte da
família de produtos (por exemplo, `criarCadeira`, `criarSofá` e
`criarMesaDeCentro`). Esses métodos devem retornar tipos **abstratos**
de produtos representados pelas interfaces que extraímos previamente:
`Cadeira`, `Sofá`, `MesaDeCentro` e assim por diante.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/solution24d8e.png?id=53975d6e4714c6f942633a879f7ac571"
style="aspect-ratio: 2;"
srcset="/images/patterns/diagrams/abstract-factory/solution2.png?id=53975d6e4714c6f942633a879f7ac571 640w,/images/patterns/diagrams/abstract-factory/solution2-1.5x.png?id=581a6cdc0dcd5511f1c30e509b1d4a7f 960w,/images/patterns/diagrams/abstract-factory/solution2-2x.png?id=b21557081f75ac7b4110427e89a10378 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="A hierarquia das classes _fábricas_" />
<figcaption><p>Cada fábrica concreta corresponde a uma variante de
produto específica.</p></figcaption>
</figure>

Agora, e o que fazer sobre as variantes de produtos? Para cada variante
de uma família de produtos nós criamos uma classe fábrica separada
baseada na interface `FábricaAbstrata`. Uma fábrica é uma classe que
retorna produtos de um tipo em particular. Por exemplo, a classe
`FábricaMobíliaModerna` só pode criar objetos `CadeiraModerna`,
`SofáModerno`, e `MesaDeCentroModerna`.

O código cliente tem que funcionar com ambos as fábricas e produtos via
suas respectivas interfaces abstratas. Isso permite que você mude o tipo
de uma fábrica que passou para o código cliente, bem como a variante do
produto que o código cliente recebeu, sem quebrar o código cliente
atual.

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-comic-2-pt-bre8b0.png?id=7306dbbfe1a4101c3c17a0a16011c069"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-comic-2-pt-br.png?id=7306dbbfe1a4101c3c17a0a16011c069 600w,/images/patterns/content/abstract-factory/abstract-factory-comic-2-pt-br-1.5x.png?id=65549eccc014904a45fc1f88d05b55a4 900w,/images/patterns/content/abstract-factory/abstract-factory-comic-2-pt-br-2x.png?id=2cd9c27f43767ba725f0b133f7974864 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>O cliente não deveria se importar com a classe concreta
da fábrica com a qual está trabalhando.</p></figcaption>
</figure>

Digamos que o cliente quer que uma fábrica produza uma cadeira. O
cliente não precisa estar ciente da classe fábrica, e nem se importa que
tipo de cadeira ele receberá. Seja ela um modelo Moderno ou no estilo
Vitoriano, o cliente precisa tratar todas as cadeiras da mesma maneira,
usando a interface abstrata `Cadeira`. Com essa abordagem, a única coisa
que o cliente sabe sobre a cadeira é que ela implementa o método
`sentar` de alguma maneira. E também, seja qual for a variante da
cadeira retornada, ela sempre irá combinar com o tipo de sofá ou mesa de
centro produzido pelo mesmo objeto fábrica.

Há mais uma coisa a se clarificar: se o cliente está exposto apenas às
interfaces abstratas, o que realmente cria os objetos fábrica então?
Geralmente, o programa cria um objeto fábrica concreto no estágio de
inicialização. Antes disso, o programa deve selecionar o tipo de fábrica
dependendo da configuração ou definições de ambiente.
:::

::::: {.section .structure-container}
##  Estrutura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/structureb809.png?id=a3112cdd98765406af94595a3c5e7762"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/abstract-factory/structure.png?id=a3112cdd98765406af94595a3c5e7762 720w,/images/patterns/diagrams/abstract-factory/structure-1.5x.png?id=d2964e541620d9c087e693bd0e0a0836 1080w,/images/patterns/diagrams/abstract-factory/structure-2x.png?id=c4d3634ec2e74e02a0fe1a83ce9b50f6 1440w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="720"
alt="Padrão de projeto Abstract Factory" /><img
src="../../images/patterns/diagrams/abstract-factory/structure-indexed028d.png?id=6ae1c99cbd90cf58753c633624fb1a04"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.56;"
srcset="/images/patterns/diagrams/abstract-factory/structure-indexed.png?id=6ae1c99cbd90cf58753c633624fb1a04 700w,/images/patterns/diagrams/abstract-factory/structure-indexed-1.5x.png?id=473be2975c5716162e6969e6532210ac 1050w,/images/patterns/diagrams/abstract-factory/structure-indexed-2x.png?id=cb6d4e1e89826c42966dc7097374f889 1400w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="700"
alt="Padrão de projeto Abstract Factory" />
</figure>
:::

1.  **Produtos Abstratos** declaram interfaces para um conjunto de
    produtos distintos mas relacionados que fazem parte de uma família
    de produtos.

2.  **Produtos Concretos** são várias implementações de produtos
    abstratos, agrupados por variantes. Cada produto abstrato
    (cadeira/sofá) deve ser implementado em todas as variantes dadas
    (Vitoriano/Moderno).

3.  A interface **Fábrica Abstrata** declara um conjunto de métodos para
    criação de cada um dos produtos abstratos.

4.  **Fábricas Concretas** implementam métodos de criação fábrica
    abstratos. Cada fábrica concreta corresponde a uma variante
    específica de produtos e cria apenas aquelas variantes de produto.

5.  Embora fábricas concretas instanciam produtos concretos, assinaturas
    dos seus métodos de criação devem retornar produtos *abstratos*
    correspondentes. Dessa forma o código cliente que usa uma fábrica
    não fica ligada a variante específica do produto que ele pegou de
    uma fábrica. O **Cliente** pode trabalhar com qualquer variante de
    produto/fábrica concreto, desde que ele se comunique com seus
    objetos via interfaces abstratas.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Este exemplo ilustra como o padrão **Abstract Factory** pode ser usado
para criar elementos UI multiplataforma sem ter que ligar o código do
cliente às classes UI concretas, enquanto mantém todos os elementos
criados consistentes com um sistema operacional escolhido.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/exampled62f.png?id=5928a61d18bf00b047463471c599100a"
style="aspect-ratio: 1.42;"
srcset="/images/patterns/diagrams/abstract-factory/example.png?id=5928a61d18bf00b047463471c599100a 640w,/images/patterns/diagrams/abstract-factory/example-1.5x.png?id=baee3bac793ec97e0ec91c49d9382e7d 960w,/images/patterns/diagrams/abstract-factory/example-2x.png?id=eb5127b1d6486f6fad73be2d5e444449 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Diagrama de classe para o exemplo de padrão Abstract Factory" />
<figcaption><p>Exemplo das classes UI multiplataforma.</p></figcaption>
</figure>

É esperado que os mesmos elementos UI de um aplicativo multiplataforma
se comportem de forma semelhante, mas que se pareçam um pouco diferentes
nos diferentes sistemas operacionais. Além disso, é seu trabalho
garantir que os elementos UI coincidam com o estilo do sistema
operacional atual. Você não vai querer que seu programa renderize
controles macOS quando é executado no Windows.

A interface da fábrica abstrata declara um conjunto de métodos de
criação que o código cliente pode usar para produzir diferentes tipos de
elementos de UI que coincidam com o SO particular. Fábricas concretas
correspondem a sistemas operacionais específicos e criam os elementos de
UI que corresponde com aquele SO em particular.

Funciona assim: quando a aplicação inicia, ela checa o tipo de sistema
operacional que está sendo utilizado. A aplicação usa essa informação
para criar um objeto fábrica de uma classe que corresponde com o sistema
operacional. O resto do código usa essa fábrica para criar elementos UI.
Isso previne que elementos errados sejam criados.

Com essa abordagem, o código cliente não depende de classes concretas de
fábricas e elementos UI desde que ele trabalhe com esses objetos através
de suas interfaces abstratas. Isso também permite que o código do
cliente suporte outras fábricas ou elementos UI que você possa adicionar
no futuro.

Como resultado, você não precisa modificar o código do cliente cada vez
que adicionar uma variação de elementos de UI em sua aplicação. Você só
precisa criar uma nova classe fábrica que produza esses elementos e
modificar de forma sutil o código de inicialização da aplicação de forma
que ele selecione aquela classe quando apropriado.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// A interface fábrica abstrata declara um conjunto de métodos
// que retorna diferentes produtos abstratos. Estes produtos são
// chamados uma família e estão relacionados por um tema ou
// conceito de alto nível. Produtos de uma família são
// geralmente capazes de colaborar entre si. Uma família de
// produtos pode ter várias variantes, mas os produtos de uma
// variante são incompatíveis com os produtos de outro variante.
interface GUIFactory is
    method createButton():Button
    method createCheckbox():Checkbox


// As fábricas concretas produzem uma família de produtos que
// pertencem a uma única variante. A fábrica garante que os
// produtos resultantes sejam compatíveis. Assinaturas dos
// métodos fabrica retornam um produto abstrato, enquanto que
// dentro do método um produto concreto é instanciado.
class WinFactory implements GUIFactory is
    method createButton():Button is
        return new WinButton()
    method createCheckbox():Checkbox is
        return new WinCheckbox()

// Cada fábrica concreta tem uma variante de produto
// correspondente.
class MacFactory implements GUIFactory is
    method createButton():Button is
        return new MacButton()
    method createCheckbox():Checkbox is
        return new MacCheckbox()


// Cada produto distinto de uma família de produtos deve ter uma
// interface base. Todas as variantes do produto devem
// implementar essa interface.
interface Button is
    method paint()

// Produtos concretos são criados por fábricas concretas
// correspondentes.
class WinButton implements Button is
    method paint() is
        // Renderiza um botão no estilo Windows.

class MacButton implements Button is
    method paint() is
        // Renderiza um botão no estilo macOS.

// Aqui está a interface base de outro produto. Todos os
// produtos podem interagir entre si, mas a interação apropriada
// só é possível entre produtos da mesma variante concreta.
interface Checkbox is
    method paint()

class WinCheckbox implements Checkbox is
    method paint() is
        // Renderiza uma caixa de seleção estilo Windows.

class MacCheckbox implements Checkbox is
    method paint() is
        // Renderiza uma caixa de seleção no estilo macOS.


// O código cliente trabalha com fábricas e produtos apenas
// através de tipos abstratos: GUIFactory, Button e Checkbox.
// Isso permite que você passe qualquer subclasse fábrica ou de
// produto para o código cliente sem quebrá-lo.
class Application is
    private field factory: GUIFactory
    private field button: Button
    constructor Application(factory: GUIFactory) is
        this.factory = factory
    method createUI() is
        this.button = factory.createButton()
    method paint() is
        button.paint()


// A aplicação seleciona o tipo de fábrica dependendo da atual
// configuração do ambiente e cria o widget no tempo de execução
// (geralmente no estágio de inicialização).
class ApplicationConfigurator is
    method main() is
        config = readApplicationConfigFile()

        if (config.OS == &quot;Windows&quot;) then
            factory = new WinFactory()
        else if (config.OS == &quot;Mac&quot;) then
            factory = new MacFactory()
        else
            throw new Exception(&quot;Error! Unknown operating system.&quot;)

        Application app = new Application(factory)</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Aplicabilidade {#applicability}

::::::: applicability
::: applicability-problem
Use o Abstract Factory quando seu código precisa trabalhar com diversas
famílias de produtos relacionados, mas que você não quer depender de
classes concretas daqueles produtos-eles podem ser desconhecidos de
antemão ou você simplesmente quer permitir uma futura escalabilidade.
:::

::: applicability-solution
O Abstract Factory fornece a você uma interface para a criação de
objetos de cada classe das famílias de produtos. Desde que seu código
crie objetos a partir dessa interface, você não precisará se preocupar
em criar uma variante errada de um produto que não coincida com produtos
já criados por sua aplicação.
:::

::: applicability-problem
Considere implementar o Abstract Factory quando você tem uma classe com
um conjunto de [métodos fábrica](factory-method.html) que desfoquem sua
responsabilidade principal.
:::

::: applicability-solution
Em um programa bem desenvolvido *cada classe é responsável por apenas
uma coisa*. Quando uma classe lida com múltiplos tipos de produto, pode
valer a pena extrair seus métodos fábrica em uma classe fábrica
solitária ou uma implementação plena do Abstract Factory.
:::
:::::::
::::::::

::: {.section .checklist}
##  Como implementar {#checklist}

1.  Mapeie uma matriz de tipos de produtos distintos versus as variantes
    desses produtos.

2.  Declare interfaces de produto abstratas para todos os tipos de
    produto. Então, faça todas as classes concretas de produtos
    implementar essas interfaces.

3.  Declare a interface da fábrica abstrata com um conjuntos de métodos
    de criação para todos os produtos abstratos.

4.  Implemente um conjunto de classes fábricas concretas, uma para cada
    variante de produto.

5.  Crie um código de inicialização da fábrica em algum lugar da
    aplicação. Ele deve instanciar uma das classes fábrica concretas,
    dependendo da configuração da aplicação ou do ambiente atual. Passe
    esse objeto fábrica para todas as classes que constroem produtos.

6.  Escaneie o código e encontre todas as chamadas diretas para
    construtores de produtos. Substitua-as por chamadas para o método de
    criação apropriado no objeto fábrica.
:::

:::::: {.section .pros-cons}
##  Prós e contras {#pros-cons}

::::: row
::: col-sm-6
-    Você pode ter certeza que os produtos que você obtém de uma fábrica
    são compatíveis entre si.
-    Você evita um vínculo forte entre produtos concretos e o código
    cliente.
-    *Princípio de responsabilidade única*. Você pode extrair o código
    de criação do produto para um lugar, fazendo o código ser de fácil
    manutenção.
-    *Princípio aberto/fechado*. Você pode introduzir novas variantes de
    produtos sem quebrar o código cliente existente.
:::

::: col-sm-6
-    O código pode tornar-se mais complicado do que deveria ser, uma vez
    que muitas novas interfaces e classes são introduzidas junto com o
    padrão.
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

-   O [Builder](builder.html) foca em construir objetos complexos passo
    a passo. O [Abstract Factory](abstract-factory.html) se especializa
    em criar famílias de objetos relacionados. O *Abstract Factory*
    retorna o produto imediatamente, enquanto que o *Builder* permite
    que você execute algumas etapas de construção antes de buscar o
    produto.

-   Classes [Abstract Factory](abstract-factory.html) são quase sempre
    baseadas em um conjunto de [métodos fábrica](factory-method.html),
    mas você também pode usar o [Prototype](prototype.html) para compor
    métodos dessas classes.

-   O [Abstract Factory](abstract-factory.html) pode servir como uma
    alternativa para o [Facade](facade.html) quando você precisa apenas
    esconder do código cliente a forma com que são criados os objetos do
    subsistema.

-   Você pode usar o [Abstract Factory](abstract-factory.html) junto com
    o [Bridge](bridge.html). Esse pareamento é útil quando algumas
    abstrações definidas pelo *Bridge* só podem trabalhar com
    implementações específicas. Neste caso, o *Abstract Factory* pode
    encapsular essas relações e esconder a complexidade do código
    cliente.

-   As [Fábricas Abstratas](abstract-factory.html),
    [Construtores](builder.html), e [Protótipos](prototype.html) podem
    todos ser implementados como [Singletons](singleton.html).
:::

::: {.section .implementations}
##  Exemplos de código {#implementations}

[![Abstract Factory em
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](abstract-factory/csharp/example.html "Abstract Factory em C#"){.prog-lang-link}
[![Abstract Factory em
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](abstract-factory/cpp/example.html "Abstract Factory em C++"){.prog-lang-link}
[![Abstract Factory em
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](abstract-factory/go/example.html "Abstract Factory em Go"){.prog-lang-link}
[![Abstract Factory em
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](abstract-factory/java/example.html "Abstract Factory em Java"){.prog-lang-link}
[![Abstract Factory em
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](abstract-factory/php/example.html "Abstract Factory em PHP"){.prog-lang-link}
[![Abstract Factory em
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](abstract-factory/python/example.html "Abstract Factory em Python"){.prog-lang-link}
[![Abstract Factory em
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](abstract-factory/ruby/example.html "Abstract Factory em Ruby"){.prog-lang-link}
[![Abstract Factory em
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](abstract-factory/rust/example.html "Abstract Factory em Rust"){.prog-lang-link}
[![Abstract Factory em
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](abstract-factory/swift/example.html "Abstract Factory em Swift"){.prog-lang-link}
[![Abstract Factory em
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](abstract-factory/typescript/example.html "Abstract Factory em TypeScript"){.prog-lang-link}
:::

::: {.section .extras}
##  Conteúdo extra {#extras}

-   Leia nossa [Comparação Factory](factory-comparison.html) para
    aprender mais sobre as diferenças entre os vários padrões e
    conceitos Factory.
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

[Comparação Factory []{.fa
.fa-arrow-right}](factory-comparison.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Factory Method](factory-method.html){.btn
.btn-default rel="prev"}
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
