:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
criacionais](creational-patterns.html){.type}
:::

# Builder {#builder .title}

::: aka
Também conhecido como: [Construtor]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

O **Builder** é um padrão de projeto criacional que permite a você
construir objetos complexos passo a passo. O padrão permite que você
produza diferentes tipos e representações de um objeto usando o mesmo
código de construção.

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-pt-brb5d8.png?id=d58b5e4de4608667aaea7ad68b0d46eb"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/builder/builder-pt-br.png?id=d58b5e4de4608667aaea7ad68b0d46eb 640w,/images/patterns/content/builder/builder-pt-br-1.5x.png?id=bcea791ae2efef219d8163e32f42be40 960w,/images/patterns/content/builder/builder-pt-br-2x.png?id=00aa53ab243764701365816278ca425c 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Padrão de projeto Builder" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagine um objeto complexo que necessite de uma inicialização passo a
passo trabalhosa de muitos campos e objetos agrupados. Tal código de
inicialização fica geralmente enterrado dentro de um construtor
monstruoso com vários parâmetros. Ou pior: espalhado por todo o código
cliente.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/problem16274.png?id=11e715c5c97811f848c48e0f399bb05e"
style="aspect-ratio: 1.71;"
srcset="/images/patterns/diagrams/builder/problem1.png?id=11e715c5c97811f848c48e0f399bb05e 600w,/images/patterns/diagrams/builder/problem1-1.5x.png?id=a46778018c4f0f4bbf2357940c1f5f40 900w,/images/patterns/diagrams/builder/problem1-2x.png?id=02ffbd7ad294600e42aa78989d441b4d 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Muitas subclasses criam outro problema" />
<figcaption><p>Você pode tornar o programa muito complexo ao criar
subclasses para cada possível configuração de
um objeto.</p></figcaption>
</figure>

Por exemplo, vamos pensar sobre como criar um objeto `Casa`. Para
construir uma casa simples, você precisa construir quatro paredes e um
piso, instalar uma porta, encaixar um par de janelas, e construir um
teto. Mas e se você quiser uma casa maior e mais iluminada, com um
jardim e outras miudezas (como um sistema de aquecimento, encanamento, e
fiação elétrica)?

A solução mais simples é estender a classe base `Casa` e criar um
conjunto de subclasses para cobrir todas as combinações de parâmetros.
Mas eventualmente você acabará com um número considerável de subclasses.
Qualquer novo parâmetro, tal como o estilo do pórtico, irá forçá-lo a
aumentar essa hierarquia cada vez mais.

Há outra abordagem que não envolve a propagação de subclasses. Você pode
criar um construtor gigante diretamente na classe `Casa` base com todos
os possíveis parâmetros que controlam o objeto casa. Embora essa
abordagem realmente elimine a necessidade de subclasses, ela cria outro
problema.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/problem27ab5.png?id=2e91039b6c7d2d2df6ee519983a3b036"
style="aspect-ratio: 1.71;"
srcset="/images/patterns/diagrams/builder/problem2.png?id=2e91039b6c7d2d2df6ee519983a3b036 600w,/images/patterns/diagrams/builder/problem2-1.5x.png?id=3d302cf2762fd04d70cbb91cb54e923c 900w,/images/patterns/diagrams/builder/problem2-2x.png?id=5e7975a91c0e4f4ba960f908cc9c2ea2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="O Construtor telescópico" />
<figcaption><p>O construtor com vários parâmetros tem um lado ruim: nem
todos os parâmetros são necessários todas as vezes.</p></figcaption>
</figure>

Na maioria dos casos a maioria dos parâmetros não será usada, tornando
[as chamadas do construtor em algo feio de se
ver](../smells/long-parameter-list.html). Por exemplo, apenas algumas
casas têm piscinas, então os parâmetros relacionados a piscinas serão
inúteis nove em cada dez vezes.
:::

::: {.section .solution}
##  Solução {#solution}

O padrão Builder sugere que você extraia o código de construção do
objeto para fora de sua própria classe e mova ele para objetos separados
chamados *builders*. "Builder" significa "construtor", mas não usaremos
essa palavra para evitar confusão com os construtores de classe.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/solution13e5f.png?id=8ce82137f8935998de802cae59e00e11"
style="aspect-ratio: 1.46;"
srcset="/images/patterns/diagrams/builder/solution1.png?id=8ce82137f8935998de802cae59e00e11 410w,/images/patterns/diagrams/builder/solution1-1.5x.png?id=8ab77eb73760a75c35940bae243d43f2 615w,/images/patterns/diagrams/builder/solution1-2x.png?id=a9c2ab02f0b2aca1a7512022194dd113 820w"
sizes="(max-width: 720px) 100vw, 410px" loading="lazy" width="410"
alt="Aplicando o padrão Builder" />
<figcaption><p>O padrão Builder permite que você construa objetos
complexos passo a passo. O Builder não permite que outros objetos
acessem o produto enquanto ele está sendo construído.</p></figcaption>
</figure>

O padrão organiza a construção de objetos em uma série de etapas
(`construirParedes`, `construirPorta`, etc.). Para criar um objeto você
executa uma série de etapas em um objeto builder. A parte importante é
que você não precisa chamar todas as etapas. Você chama apenas aquelas
etapas que são necessárias para a produção de uma configuração
específica de um objeto.

Algumas das etapas de construção podem necessitar de implementações
diferentes quando você precisa construir várias representações do
produto. Por exemplo, paredes de uma cabana podem ser construídas com
madeira, mas paredes de um castelo devem ser construídas com pedra.

Nesse caso, você pode criar diferentes classes construturas que
implementam as mesmas etapas de construção, mas de maneira diferente.
Então você pode usar esses builders no processo de construção (i.e, um
pedido ordenado de chamadas para as etapas de construção) para produzir
diferentes tipos de objetos.

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-comic-1-pt-br0f96.png?id=d6acdaf7126338c276bd9605fe34c00f"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/builder/builder-comic-1-pt-br.png?id=d6acdaf7126338c276bd9605fe34c00f 600w,/images/patterns/content/builder/builder-comic-1-pt-br-1.5x.png?id=f2d0aec40700bf841807d636bbbd1473 900w,/images/patterns/content/builder/builder-comic-1-pt-br-2x.png?id=4c66b74b29397acffd5fbf2f9b5f1bee 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>Builders diferentes executam a mesma tarefa de
várias maneiras.</p></figcaption>
</figure>

Por exemplo, imagine um builder que constrói tudo de madeira e vidro, um
segundo builder que constrói tudo com pedra e ferro, e um terceiro que
usa ouro e diamantes. Ao chamar o mesmo conjunto de etapas, você obtém
uma casa normal do primeiro builder, um pequeno castelo do segundo, e um
palácio do terceiro. Contudo, isso só vai funcionar se o código cliente
que chama as etapas de construção é capaz de interagir com os builders
usando uma interface comum.

#### Diretor

Você pode ir além e extrair uma série de chamadas para as etapas do
builder que você usa para construir um produto em uma classe separada
chamada *diretor*. A classe diretor define a ordem na qual executar as
etapas de construção, enquanto que o builder provê a implementação
dessas etapas.

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-comic-2-pt-br52e7.png?id=9101f30e9784ecd4c8a6367bfac9cc7f"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/content/builder/builder-comic-2-pt-br.png?id=9101f30e9784ecd4c8a6367bfac9cc7f 343w,/images/patterns/content/builder/builder-comic-2-pt-br-1.5x.png?id=8bf3b48b51ced4c1a925b00b02425210 515w,/images/patterns/content/builder/builder-comic-2-pt-br-2x.png?id=56812a286b082e2fe72d06b7b6e58d25 687w"
sizes="(max-width: 720px) 100vw, 343px" loading="lazy" width="343" />
<figcaption><p>O diretor sabe quais etapas de construção executar para
obter um produto que funciona.</p></figcaption>
</figure>

Ter uma classe diretor em seu programa não é estritamente necessário.
Você sempre pode chamar as etapas de construção em uma ordem específica
diretamente do código cliente. Contudo, a classe diretor pode ser um bom
lugar para colocar várias rotinas de construção para que você possa
reutilizá-las em qualquer lugar do seu programa.

Além disso, a classe diretor esconde completamente os detalhes da
construção do produto do código cliente. O cliente só precisa associar
um builder com um diretor, inicializar a construção com o diretor, e
então obter o resultado do builder.
:::

::::: {.section .structure-container}
##  Estrutura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/builder/structure05e4.png?id=fe9e23559923ea0657aa5fe75efef333"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.79;"
srcset="/images/patterns/diagrams/builder/structure.png?id=fe9e23559923ea0657aa5fe75efef333 460w,/images/patterns/diagrams/builder/structure-1.5x.png?id=91ea8cd3137b403542c23abf63938f00 690w,/images/patterns/diagrams/builder/structure-2x.png?id=dca1b1508e23c266cbedc80ffb84311a 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Estrutura do padrão de projeto Builder" /><img
src="../../images/patterns/diagrams/builder/structure-indexedadac.png?id=44b3d763ce91dbada5d8394ef777437f"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.81;"
srcset="/images/patterns/diagrams/builder/structure-indexed.png?id=44b3d763ce91dbada5d8394ef777437f 470w,/images/patterns/diagrams/builder/structure-indexed-1.5x.png?id=d57a6ff9342ea31736ea98e5066e0748 705w,/images/patterns/diagrams/builder/structure-indexed-2x.png?id=153eed9a51784cbe00d0ca8b3d6b268d 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Estrutura do padrão de projeto Builder" />
</figure>
:::

1.  A interface **Builder** declara etapas de construção do produto que
    são comuns a todos os tipos de builders.

2.  **Builders Concretos** provém diferentes implementações das etapas
    de construção. Builders concretos podem produzir produtos que não
    seguem a interface comum.

3.  **Produtos** são os objetos resultantes. Produtos construídos por
    diferentes builders não precisam pertencer a mesma interface ou
    hierarquia da classe.

4.  A classe **Diretor** define a ordem na qual as etapas de construção
    são chamadas, então você pode criar e reutilizar configurações
    específicas de produtos.

5.  O **Cliente** deve associar um dos objetos builders com o diretor.
    Usualmente isso é feito apenas uma vez, através de parâmetros do
    construtor do diretor. O diretor então usa aquele objeto builder
    para todas as futuras construções. Contudo, há uma abordagem
    alternativa para quando o cliente passa o objeto builder ao método
    de produção do diretor. Nesse caso, você pode usar um builder
    diferente a cada vez que você produzir alguma coisa com o diretor.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Este exemplo do padrão **Builder** ilustra como você pode reutilizar o
mesmo código de construção de objeto quando construindo diferentes tipos
de produtos, tais como carros, e como criar manuais correspondentes para
eles.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/example-pt-br6c67.png?id=b1085f7f0c5ce6cc8d3a7ea23cca3d4e"
style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/builder/example-pt-br.png?id=b1085f7f0c5ce6cc8d3a7ea23cca3d4e 500w,/images/patterns/diagrams/builder/example-pt-br-1.5x.png?id=50a3da5273069a734473b4b7b317b7f5 750w,/images/patterns/diagrams/builder/example-pt-br-2x.png?id=4231aacbae6b56d54aa09e993f04b63f 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="Exemplo da estrutura do padrão Builder" />
<figcaption><p>O exemplo do passo a passo da construção de carros e do
manual do usuário que se adapta a aqueles modelos
de carros.</p></figcaption>
</figure>

Um carro é um objeto complexo que pode ser construído em centenas de
maneiras diferentes. Ao invés de inchar a classe `Carro` com um
construtor enorme, nós extraímos o código de montagem do carro em uma
classe de construção de carro separada. Essa classe tem um conjunto de
métodos para configurar as várias partes de um carro.

Se o código cliente precisa montar um modelo de carro especial e
ajustado, ele pode trabalhar com o builder diretamente. Por outro lado,
o cliente pode delegar a montagem à classe diretor, que sabe como usar
um builder para construir diversos modelos de carros populares.

Você pode ficar chocado, mas cada carro precisa de um manual (sério,
quem lê aquilo?). O manual descreve cada funcionalidade do carro, então
os detalhes dos manuais variam de acordo com os diferentes modelos. É
por isso que faz sentido reutilizar um processo de construção existente
para ambos os carros e seus respectivos manuais. É claro, construir um
manual não é o mesmo que construir um carro, e é por isso que devemos
providenciar outra classe builder que se especializa em compor manuais.
Essa classe implementa os mesmos métodos de construção que seus parentes
builder de carros, mas ao invés de construir partes de carros, ela as
descreve. Ao passar esses builders ao mesmo objeto diretor, podemos
tanto construir um carro como um manual.

A parte final é obter o objeto resultante. Um carro de metal e um manual
de papel, embora relacionados, são coisas muito diferentes. Não podemos
colocar um método para obter resultados no diretor sem ligar o diretor
às classes de produto concretas. Portanto, nós obtemos o resultado da
construção a partir do builder que fez o trabalho.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Usar o padrão Builder só faz sentido quando seus produtos são
// bem complexos e requerem configuração extensiva. Os dois
// produtos a seguir são relacionados, embora eles não tenham
// uma interface em comum.
class Car is
    // Um carro pode ter um GPS, computador de bordo, e alguns
    // assentos. Diferentes modelos de carros (esportivo, SUV,
    // conversível) podem ter diferentes funcionalidades
    // instaladas ou equipadas.

class Manual is
    // Cada carro deve ter um manual do usuário que corresponda
    // a configuração do carro e descreva todas suas
    // funcionalidades.


// A interface builder especifica métodos para criar as
// diferentes partes de objetos produto.
interface Builder is
    method reset()
    method setSeats(...)
    method setEngine(...)
    method setTripComputer(...)
    method setGPS(...)

// As classes builder concretas seguem a interface do
// builder e fornecem implementações específicas das etapas
// de construção. Seu programa pode ter algumas variações de
// builders, cada uma implementada de forma diferente.
class CarBuilder implements Builder is
    private field car:Car

    // Uma instância fresca do builder deve conter um objeto
    // produto em branco na qual ela usa para montagem futura.
    constructor CarBuilder() is
        this.reset()

    // O método reset limpa o objeto sendo construído.
    method reset() is
        this.car = new Car()

    // Todas as etapas de produção trabalham com a mesma
    // instância de produto.
    method setSeats(...) is
        // Define o número de assentos no carro.

    method setEngine(...) is
        // Instala um tipo de motor.

    method setTripComputer(...) is
        // Instala um computador de bordo.

    method setGPS(...) is
        // Instala um sistema de posicionamento global.

    // Builders concretos devem fornecer seus próprios
    // métodos para recuperar os resultados. Isso é porque
    // vários tipos de builders podem criar produtos
    // inteiramente diferentes que nem sempre seguem a mesma
    // interface. Portanto, tais métodos não podem ser
    // declarados na interface do builder (ao menos não em
    // uma linguagem de programação de tipo estático).
    //
    // Geralmente, após retornar o resultado final para o
    // cliente, espera-se que uma instância de builder comece
    // a produzir outro produto. É por isso que é uma prática
    // comum chamar o método reset no final do corpo do método
    // `getProduct`. Contudo este comportamento não é
    // obrigatório, e você pode fazer seu builder esperar por
    // uma chamada explícita do reset a partir do código cliente
    // antes de se livrar de seu resultado anterior.
    method getProduct():Car is
        product = this.car
        this.reset()
        return product

// Ao contrário dos outros padrões criacionais, o Builder
// permite que você construa produtos que não seguem uma
// interface comum.
class CarManualBuilder implements Builder is
    private field manual:Manual

    constructor CarManualBuilder() is
        this.reset()

    method reset() is
        this.manual = new Manual()

    method setSeats(...) is
        // Documenta as funcionalidades do assento do carro.

    method setEngine(...) is
        // Adiciona instruções do motor.

    method setTripComputer(...) is
        // Adiciona instruções do computador de bordo.

    method setGPS(...) is
        // Adiciona instruções do GPS.

    method getProduct():Manual is
        // Retorna o manual e reseta o builder.


// O diretor é apenas responsável por executar as etapas de
// construção em uma sequência em particular. Isso ajuda quando
// produzindo produtos de acordo com uma ordem específica ou
// configuração. A rigor, a classe diretor é opcional, já que o
// cliente pode controlar os builders diretamente.
class Director is
    // O diretor trabalha com qualquer instância builder que
    // o código cliente passar a ele. Dessa forma, o código
    // cliente pode alterar o tipo final do produto recém
    // montado. O diretor pode construir diversas variações
    // do produto usando as mesmas etapas de construção.
    method constructSportsCar(builder: Builder) is
        builder.reset()
        builder.setSeats(2)
        builder.setEngine(new SportEngine())
        builder.setTripComputer(true)
        builder.setGPS(true)

    method constructSUV(builder: Builder) is
        // ...


// O código cliente cria um objeto builder, passa ele para o
// diretor e então inicia o processo de construção. O resultado
// final é recuperado do objeto builder.
class Application is

    method makeCar() is
        director = new Director()

        CarBuilder builder = new CarBuilder()
        director.constructSportsCar(builder)
        Car car = builder.getProduct()

        CarManualBuilder builder = new CarManualBuilder()
        director.constructSportsCar(builder)

        // O produto final é frequentemente retornado de um
        // objeto builder uma vez que o diretor não está
        // ciente e não é dependente de builders e produtos
        // concretos.
        Manual manual = builder.getProduct()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Aplicabilidade {#applicability}

::::::::: applicability
::: applicability-problem
Use o padrão Builder para se livrar de um "construtor telescópico".
:::

::: applicability-solution
Digamos que você tenha um construtor com dez parâmetros opcionais.
Chamar um monstro desses é muito inconveniente; portanto, você
sobrecarrega o construtor e cria diversas versões curtas com menos
parâmetros. Esses construtores ainda se referem ao principal, passando
alguns valores padrão para qualquer parâmetro omitido.

<figure class="code">
<pre class="code" lang="java"><code>class Pizza {
    Pizza(int size) { ... }
    Pizza(int size, boolean cheese) { ... }
    Pizza(int size, boolean cheese, boolean pepperoni) { ... }
    // ...</code></pre>
<figcaption><p>Criar tal monstro só é possível em linguagens que
suportam sobrecarregamento de método, tais como C#
ou Java.</p></figcaption>
</figure>

O padrão Builder permite que você construa objetos passo a passo, usando
apenas aquelas etapas que você realmente precisa. Após implementar o
padrão, você não vai mais precisar amontoar dúzias de parâmetros em seus
construtores.
:::

::: applicability-problem
Use o padrão Builder quando você quer que seu código seja capaz de criar
diferentes representações do mesmo produto (por exemplo, casas de pedra
e madeira).
:::

::: applicability-solution
O padrão Builder pode ser aplicado quando a construção de várias
representações do produto envolvem etapas similares que diferem apenas
nos detalhes.

A interface base do builder define todas as etapas de construção
possíveis, e os buildrs concretos implementam essas etapas para
construir representações particulares do produto. Enquanto isso, a
classe diretor guia a ordem de construção.
:::

::: applicability-problem
Use o Builder para construir árvores [Composite](composite.html) ou
outros objetos complexos.
:::

::: applicability-solution
O padrão Builder permite que você construa produtos passo a passo. Você
pode adiar a execução de algumas etapas sem quebrar o produto final.
Você pode até chamar etapas recursivamente, o que é bem útil quando você
precisa construir uma árvore de objetos.

Um builder não expõe o produto não finalizado enquanto o processo de
construção estiver executando etapas. Isso previne o código cliente de
obter um resultado incompleto.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Como implementar {#checklist}

1.  Certifique-se que você pode definir claramente as etapas comuns de
    construção para construir todas as representações do produto
    disponíveis. Do contrário, você não será capaz de implementar o
    padrão.

2.  Declare essas etapas na interface builder base.

3.  Crie uma classe builder concreta para cada representação do produto
    e implemente suas etapas de construção.

    Não se esqueça de implementar um método para recuperar os resultados
    da construção. O motivo pelo qual esse método não pode ser declarado
    dentro da interface do builder é porque vários builders podem
    construir produtos que não tem uma interface comum. Portanto, você
    não sabe qual será o tipo de retorno para tal método. Contudo, se
    você está lidando com produtos de uma única hierarquia, o método de
    obtenção pode ser adicionado com segurança para a interface base.

4.  Pense em criar uma classe diretor. Ela pode encapsular várias
    maneiras de construir um produto usando o mesmo objeto builder.

5.  O código cliente cria tanto os objetos do builder como do diretor.
    Antes da construção começar, o cliente deve passar um objeto builder
    para o diretor. Geralmente o cliente faz isso apenas uma vez,
    através de parâmetros do construtor do diretor. O diretor usa o
    objeto builder em todas as construções futuras. Existe uma
    alternativa onde o builder é passado diretamente ao método de
    construção do diretor.

6.  O resultado da construção pode ser obtido diretamente do diretor
    apenas se todos os produtos seguirem a mesma interface. Do contrário
    o cliente deve obter o resultado do builder.
:::

:::::: {.section .pros-cons}
##  Prós e contras {#pros-cons}

::::: row
::: col-sm-6
-    Você pode construir objetos passo a passo, adiar as etapas de
    construção ou rodar etapas recursivamente.
-    Você pode reutilizar o mesmo código de construção quando
    construindo várias representações de produtos.
-    *Princípio de responsabilidade única*. Você pode isolar um código
    de construção complexo da lógica de negócio do produto.
:::

::: col-sm-6
-    A complexidade geral do código aumenta uma vez que o padrão exige
    criar múltiplas classes novas.
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

-   Você pode usar o [Builder](builder.html) quando criar árvores
    [Composite](composite.html) complexas porque você pode programar
    suas etapas de construção para trabalhar recursivamente.

-   Você pode combinar o [Builder](builder.html) com o
    [Bridge](bridge.html): a classe *diretor* tem um papel de abstração,
    enquanto que diferentes *construtores* agem como *implementações*.

-   As [Fábricas Abstratas](abstract-factory.html),
    [Construtores](builder.html), e [Protótipos](prototype.html) podem
    todos ser implementados como [Singletons](singleton.html).
:::

::: {.section .implementations}
##  Exemplos de código {#implementations}

[![Builder em
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](builder/csharp/example.html "Builder em C#"){.prog-lang-link}
[![Builder em
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](builder/cpp/example.html "Builder em C++"){.prog-lang-link}
[![Builder em
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](builder/go/example.html "Builder em Go"){.prog-lang-link}
[![Builder em
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](builder/java/example.html "Builder em Java"){.prog-lang-link}
[![Builder em
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](builder/php/example.html "Builder em PHP"){.prog-lang-link}
[![Builder em
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](builder/python/example.html "Builder em Python"){.prog-lang-link}
[![Builder em
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](builder/ruby/example.html "Builder em Ruby"){.prog-lang-link}
[![Builder em
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](builder/rust/example.html "Builder em Rust"){.prog-lang-link}
[![Builder em
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](builder/swift/example.html "Builder em Swift"){.prog-lang-link}
[![Builder em
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](builder/typescript/example.html "Builder em TypeScript"){.prog-lang-link}
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

[Prototype []{.fa .fa-arrow-right}](prototype.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Comparação
Factory](factory-comparison.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::
