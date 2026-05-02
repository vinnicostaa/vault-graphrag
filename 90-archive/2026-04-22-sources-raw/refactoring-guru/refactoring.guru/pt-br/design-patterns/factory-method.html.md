::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
criacionais](creational-patterns.html){.type}
:::

# Factory Method {#factory-method .title}

::: aka
Também conhecido como: [Método
fábrica, ]{style="display:inline-block"}[Construtor
virtual]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

O **Factory Method** é um padrão criacional de projeto que fornece uma
interface para criar objetos em uma superclasse, mas permite que as
subclasses alterem o tipo de objetos que serão criados.

<figure class="image">
<img
src="../../images/patterns/content/factory-method/factory-method-pt-brad74.png?id=07b6103a0a765193618361dd598343ac"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/factory-method/factory-method-pt-br.png?id=07b6103a0a765193618361dd598343ac 640w,/images/patterns/content/factory-method/factory-method-pt-br-1.5x.png?id=7699509d91fc38629abb527a699b748e 960w,/images/patterns/content/factory-method/factory-method-pt-br-2x.png?id=ee375cc4c219f28c05f01faa8664310c 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Padrão do Factory Method" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagine que você está criando uma aplicação de gerenciamento de
logística. A primeira versão da sua aplicação pode lidar apenas com o
transporte de caminhões, portanto a maior parte do seu código fica
dentro da classe `Caminhão`.

Depois de um tempo, sua aplicação se torna bastante popular. Todos os
dias você recebe dezenas de solicitações de empresas de transporte
marítimo para incorporar a logística marítima na aplicação.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/problem1-pt-br10ba.png?id=9202c40a1010d1635451286afc9a0600"
style="aspect-ratio: 2.4;"
srcset="/images/patterns/diagrams/factory-method/problem1-pt-br.png?id=9202c40a1010d1635451286afc9a0600 600w,/images/patterns/diagrams/factory-method/problem1-pt-br-1.5x.png?id=ac378e38d8a859f4d6794af224c3ea39 900w,/images/patterns/diagrams/factory-method/problem1-pt-br-2x.png?id=313bf4a10ac4f75391947e1ad6fa788a 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="A adição de uma nova classe de transporte ao programa causa um problema" />
<figcaption><p>Adicionar uma nova classe ao programa não é tão simples
se o restante do código já estiver acoplado às
classes existentes.</p></figcaption>
</figure>

Boa notícia, certo? Mas e o código? Atualmente, a maior parte do seu
código é acoplada à classe `Caminhão`. Adicionar `Navio` à aplicação
exigiria alterações em toda a base de código. Além disso, se mais tarde
você decidir adicionar outro tipo de transporte à aplicação,
provavelmente precisará fazer todas essas alterações novamente.

Como resultado, você terá um código bastante sujo, repleto de
condicionais que alteram o comportamento da aplicação, dependendo da
classe de objetos de transporte.
:::

::: {.section .solution}
##  Solução {#solution}

O padrão Factory Method sugere que você substitua chamadas diretas de
construção de objetos (usando o operador `new`) por chamadas para um
método *fábrica* especial. Não se preocupe: os objetos ainda são criados
através do operador `new`, mas esse está sendo chamado de dentro do
método fábrica. Objetos retornados por um método fábrica geralmente são
chamados de *produtos*.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution16ec2.png?id=fc756d2af296b5b4d482e548214d08ef"
style="aspect-ratio: 2.3;"
srcset="/images/patterns/diagrams/factory-method/solution1.png?id=fc756d2af296b5b4d482e548214d08ef 620w,/images/patterns/diagrams/factory-method/solution1-1.5x.png?id=22d3b6bb74e966d1cb3a4d8f380cefa3 930w,/images/patterns/diagrams/factory-method/solution1-2x.png?id=c482b3cd7a4d8dd73b4c8c12dfcaa03c 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="A estrutura das classes creators" />
<figcaption><p>As subclasses podem alterar a classe de objetos
retornados pelo método fábrica.</p></figcaption>
</figure>

À primeira vista, essa mudança pode parecer sem sentido: apenas mudamos
a chamada do construtor de uma parte do programa para outra. No entanto,
considere o seguinte: agora você pode sobrescrever o método fábrica em
uma subclasse e alterar a classe de produtos que estão sendo criados
pelo método.

Porém, há uma pequena limitação: as subclasses só podem retornar tipos
diferentes de produtos se esses produtos tiverem uma classe ou interface
base em comum. Além disso, o método fábrica na classe base deve ter seu
tipo de retorno declarado como essa interface.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution2-pt-brf5e7.png?id=eab22776c8ad946edac7fc203a336aa3"
style="aspect-ratio: 1.96;"
srcset="/images/patterns/diagrams/factory-method/solution2-pt-br.png?id=eab22776c8ad946edac7fc203a336aa3 490w,/images/patterns/diagrams/factory-method/solution2-pt-br-1.5x.png?id=40593219dbe559998b76f7d1565096d8 735w,/images/patterns/diagrams/factory-method/solution2-pt-br-2x.png?id=eb443323f441c1af29e37c160803c380 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="A estrutura da hierarquia de produtos" />
<figcaption><p>Todos os produtos devem seguir a
mesma interface.</p></figcaption>
</figure>

Por exemplo, ambas as classes `Caminhão` e `Navio` devem implementar a
interface `Transporte`, que declara um método chamado `entregar`. Cada
classe implementa esse método de maneira diferente: caminhões entregam
carga por terra, navios entregam carga por mar. O método fábrica na
classe `LogísticaViária` retorna objetos de caminhão, enquanto o método
fábrica na classe `LogísticaMarítima` retorna navios.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution3-pt-brdc66.png?id=23e681e656e960fc99b10d1ba2923d2b"
style="aspect-ratio: 1.83;"
srcset="/images/patterns/diagrams/factory-method/solution3-pt-br.png?id=23e681e656e960fc99b10d1ba2923d2b 640w,/images/patterns/diagrams/factory-method/solution3-pt-br-1.5x.png?id=1a4725320877f79d8e53778fcf1affbc 960w,/images/patterns/diagrams/factory-method/solution3-pt-br-2x.png?id=603fd35046f4aa62103c4e7925b5d112 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="A estrutura do código após a aplicação do padrão Factory Method" />
<figcaption><p>Desde que todas as classes de produtos implementem uma
interface comum, você pode passar seus objetos para o código cliente
sem quebrá-lo.</p></figcaption>
</figure>

O código que usa o método fábrica (geralmente chamado de código
*cliente*) não vê diferença entre os produtos reais retornados por
várias subclasses. O cliente trata todos os produtos como um
`Transporte` abstrato. O cliente sabe que todos os objetos de transporte
devem ter o método `entregar`, mas como exatamente ele funciona não é
importante para o cliente.
:::

::::: {.section .structure-container}
##  Estrutura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/structure78c3.png?id=4cba0803f42517cfe8548c9bc7dc4c9b"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/factory-method/structure.png?id=4cba0803f42517cfe8548c9bc7dc4c9b 660w,/images/patterns/diagrams/factory-method/structure-1.5x.png?id=ece8300890afc14494770a6b6d370428 990w,/images/patterns/diagrams/factory-method/structure-2x.png?id=9ea3aa8a47f8be22e12e523c15b448fd 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="A estrutura do código após a aplicação do padrão Factory Method" /><img
src="../../images/patterns/diagrams/factory-method/structure-indexed6136.png?id=4c603207859ca1f939b17b60a3a2e9e0"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/factory-method/structure-indexed.png?id=4c603207859ca1f939b17b60a3a2e9e0 660w,/images/patterns/diagrams/factory-method/structure-indexed-1.5x.png?id=626b6d7f577e1c265cce244678b2cf7f 990w,/images/patterns/diagrams/factory-method/structure-indexed-2x.png?id=c794e4f2d05013fb176464a1d1a5d7ab 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="A estrutura do código após a aplicação do padrão Factory Method" />
</figure>
:::

1.  O **Produto** declara a interface, que é comum a todos os objetos
    que podem ser produzidos pelo criador e suas subclasses.

2.  **Produtos Concretos** são implementações diferentes da interface do
    produto.

3.  A classe **Criador** declara o método fábrica que retorna novos
    objetos produto. É importante que o tipo de retorno desse método
    corresponda à interface do produto.

    Você pode declarar o método fábrica como abstrato para forçar todas
    as subclasses a implementar suas próprias versões do método. Como
    alternativa, o método fábrica base pode retornar algum tipo de
    produto padrão.

    Observe que, apesar do nome, a criação de produtos **não** é a
    principal responsabilidade do criador. Normalmente, a classe
    criadora já possui alguma lógica de negócio relacionada aos
    produtos. O método fábrica ajuda a dissociar essa lógica das classes
    concretas de produtos. Aqui está uma analogia: uma grande empresa de
    desenvolvimento de software pode ter um departamento de treinamento
    para programadores. No entanto, a principal função da empresa como
    um todo ainda é escrever código, não produzir programadores.

4.  **Criadores Concretos** sobrescrevem o método fábrica base para
    retornar um tipo diferente de produto.

    Observe que o método fábrica não precisa **criar** novas instâncias
    o tempo todo. Ele também pode retornar objetos existentes de um
    cache, um conjunto de objetos, ou outra fonte.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Este exemplo ilustra como o **Factory Method** pode ser usado para criar
elementos de interface do usuário multiplataforma sem acoplar o código
do cliente às classes de UI concretas.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/examplec59f.png?id=67db9a5cb817913444efcb1c067c9835"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/factory-method/example.png?id=67db9a5cb817913444efcb1c067c9835 640w,/images/patterns/diagrams/factory-method/example-1.5x.png?id=921d97276e5e2fd8e64609c9cfa021e7 960w,/images/patterns/diagrams/factory-method/example-2x.png?id=a2470830778e318263155000dbdc5870 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="A estrutura do código após a aplicação do padrão Factory Method exemplo" />
<figcaption><p>Exemplo de diálogo de
plataforma cruzada.</p></figcaption>
</figure>

A classe base diálogo usa diferentes elementos da UI do usuário para
renderizar sua janela. Em diferentes sistemas operacionais, esses
elementos podem parecer um pouco diferentes, mas ainda devem se
comportar de forma consistente. Um botão no Windows ainda é um botão no
Linux.

Quando o método fábrica entra em ação, você não precisa reescrever a
lógica da caixa de diálogo para cada sistema operacional. Se declararmos
um método fábrica que produz botões dentro da classe base da caixa de
diálogo, mais tarde podemos criar uma subclasse de caixa de diálogo que
retorna botões no estilo Windows do método fábrica. A subclasse herda a
maior parte do código da caixa de diálogo da classe base, mas, graças ao
método fábrica, pode renderizar botões estilo Windows na tela.

Para que esse padrão funcione, a classe base da caixa de diálogo deve
funcionar com botões abstratos: uma classe base ou uma interface que
todos os botões concretos seguem. Dessa forma, o código da caixa de
diálogo permanece funcional, independentemente do tipo de botão com o
qual ela trabalha.

Obviamente, você também pode aplicar essa abordagem a outros elementos
da UI. No entanto, com cada novo método fábrica adicionado à caixa de
diálogo, você se aproxima do padrão [Abstract
Factory](abstract-factory.html). Não se preocupe, falaremos sobre esse
padrão mais tarde.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// A classe criadora declara o método fábrica que deve retornar
// um objeto de uma classe produto. As subclasses da criadora
// geralmente fornecem a implementação desse método.
class Dialog is
    // A criadora também pode fornecer alguma implementação
    // padrão do Factory Method.
    abstract method createButton():Button

    // Observe que, apesar do seu nome, a principal
    // responsabilidade da criadora não é criar produtos. Ela
    // geralmente contém alguma lógica de negócio central que
    // depende dos objetos produto retornados pelo método
    // fábrica. As subclasses pode mudar indiretamente essa
    // lógica de negócio ao sobrescreverem o método fábrica e
    // retornarem um tipo diferente de produto dele.
    method render() is
        // Chame o método fábrica para criar um objeto produto.
        Button okButton = createButton()
        // Agora use o produto.
        okButton.onClick(closeDialog)
        okButton.render()


// Criadores concretos sobrescrevem o método fábrica para mudar
// o tipo de produto resultante.
class WindowsDialog extends Dialog is
    method createButton():Button is
        return new WindowsButton()

class WebDialog extends Dialog is
    method createButton():Button is
        return new HTMLButton()


// A interface do produto declara as operações que todos os
// produtos concretos devem implementar.
interface Button is
    method render()
    method onClick(f)

// Produtos concretos fornecem várias implementações da
// interface do produto.
class WindowsButton implements Button is
    method render(a, b) is
        // Renderiza um botão no estilo Windows.
    method onClick(f) is
        // Vincula um evento de clique do SO nativo.

class HTMLButton implements Button is
    method render(a, b) is
        // Retorna uma representação HTML de um botão.
    method onClick(f) is
        // Vincula um evento de clique no navegador web.


class Application is
    field dialog: Dialog

    // A aplicação seleciona um tipo de criador dependendo da
    // configuração atual ou definições de ambiente.
    method initialize() is
        config = readApplicationConfigFile()

        if (config.OS == &quot;Windows&quot;) then
            dialog = new WindowsDialog()
        else if (config.OS == &quot;Web&quot;) then
            dialog = new WebDialog()
        else
            throw new Exception(&quot;Error! Unknown operating system.&quot;)

    // O código cliente trabalha com uma instância de um criador
    // concreto, ainda que com sua interface base. Desde que o
    // cliente continue trabalhando com a criadora através da
    // interface base, você pode passar qualquer subclasse da
    // criadora.
    method main() is
        this.initialize()
        dialog.render()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Aplicabilidade {#applicability}

::::::::: applicability
::: applicability-problem
Use o Factory Method quando não souber de antemão os tipos e
dependências exatas dos objetos com os quais seu código deve funcionar.
:::

::: applicability-solution
O Factory Method separa o código de construção do produto do código que
realmente usa o produto. Portanto, é mais fácil estender o código de
construção do produto independentemente do restante do código.

Por exemplo, para adicionar um novo tipo de produto à aplicação, só será
necessário criar uma nova subclasse criadora e substituir o método
fábrica nela.
:::

::: applicability-problem
Use o Factory Method quando desejar fornecer aos usuários da sua
biblioteca ou framework uma maneira de estender seus componentes
internos.
:::

::: applicability-solution
Herança é provavelmente a maneira mais fácil de estender o comportamento
padrão de uma biblioteca ou framework. Mas como o framework reconheceria
que sua subclasse deve ser usada em vez de um componente padrão?

A solução é reduzir o código que constrói componentes no framework em um
único método fábrica e permitir que qualquer pessoa sobrescreva esse
método, além de estender o próprio componente.

Vamos ver como isso funcionaria. Imagine que você escreva uma aplicação
usando um framework de UI de código aberto. Sua aplicação deve ter
botões redondos, mas o framework fornece apenas botões quadrados. Você
estende a classe padrão `Botão` com uma gloriosa subclasse
`BotãoRedondo`. Mas agora você precisa informar à classe principal
`UIFramework` para usar a nova subclasse no lugar do botão padrão.

Para conseguir isso, você cria uma subclasse `UIComBotõesRedondos` a
partir de uma classe base do framework e sobrescreve seu método
`criarBotão`. Enquanto este método retorna objetos `Botão` na classe
base, você faz sua subclasse retornar objetos `BotãoRedondo`. Agora use
a classe `UIComBotõesRedondos` no lugar de `UIFramework`. E é isso!
:::

::: applicability-problem
Use o Factory Method quando deseja economizar recursos do sistema
reutilizando objetos existentes em vez de recriá-los sempre.
:::

::: applicability-solution
Você irá enfrentar essa necessidade ao lidar com objetos grandes e
pesados, como conexões com bancos de dados, sistemas de arquivos e
recursos de rede.

Vamos pensar no que deve ser feito para reutilizar um objeto existente:

1.  Primeiro, você precisa criar algum armazenamento para manter o
    controle de todos os objetos criados.
2.  Quando alguém solicita um objeto, o programa deve procurar um objeto
    livre dentro desse conjunto.
3.  \...e retorná-lo ao código cliente.
4.  Se não houver objetos livres, o programa deve criar um novo (e
    adicioná-lo ao conjunto de objetos).

Isso é muito código! E tudo deve ser colocado em um único local para que
você não polua o programa com código duplicado.

Provavelmente, o lugar mais óbvio e conveniente onde esse código deve
ficar é no construtor da classe cujos objetos estamos tentando
reutilizar. No entanto, um construtor deve sempre retornar **novos
objetos** por definição. Não pode retornar instâncias existentes.

Portanto, você precisa ter um método regular capaz de criar novos
objetos e reutilizar os existentes. Isso parece muito com um método
fábrica.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Como implementar {#checklist}

1.  Faça todos os produtos implementarem a mesma interface. Essa
    interface deve declarar métodos que fazem sentido em todos os
    produtos.

2.  Adicione um método fábrica vazio dentro da classe criadora. O tipo
    de retorno do método deve corresponder à interface comum do produto.

3.  No código da classe criadora, encontre todas as referências aos
    construtores de produtos. Um por um, substitua-os por chamadas ao
    método fábrica, enquanto extrai o código de criação do produto para
    o método fábrica.

    Pode ser necessário adicionar um parâmetro temporário ao método
    fábrica para controlar o tipo de produto retornado.

    Neste ponto, o código do método fábrica pode parecer bastante feio.
    Pode ter um grande operador `switch` que escolhe qual classe de
    produto instanciar. Mas não se preocupe, resolveremos isso em breve.

4.  Agora, crie um conjunto de subclasses criadoras para cada tipo de
    produto listado no método fábrica. Sobrescreva o método fábrica nas
    subclasses e extraia os pedaços apropriados do código de construção
    do método base.

5.  Se houver muitos tipos de produtos e não fizer sentido criar
    subclasses para todos eles, você poderá reutilizar o parâmetro de
    controle da classe base nas subclasses.

    Por exemplo, imagine que você tenha a seguinte hierarquia de
    classes: a classe base `Correio` com algumas subclasses:
    `CorreioAéreo` e `CorreioTerrestre`; as classes `Transporte` são
    `Avião`, `Caminhão` e `Trem`. Enquanto a classe `CorreioAéreo` usa
    apenas objetos `Avião`, o `CorreioTerrestre` pode funcionar com os
    objetos `Caminhão` e `Trem`. Você pode criar uma nova subclasse (por
    exemplo, `CorreioFerroviário`) para lidar com os dois casos, mas há
    outra opção. O código do cliente pode passar um argumento para o
    método fábrica da classe `CorreioTerrestre` para controlar qual
    produto ele deseja receber.

6.  Se, após todas as extrações, o método fábrica base ficar vazio, você
    poderá torná-lo abstrato. Se sobrar algo, você pode tornar isso em
    um comportamento padrão do método.
:::

:::::: {.section .pros-cons}
##  Prós e contras {#pros-cons}

::::: row
::: col-sm-6
-    Você evita acoplamentos firmes entre o criador e os produtos
    concretos.
-    *Princípio de responsabilidade única*. Você pode mover o código de
    criação do produto para um único local do programa, facilitando a
    manutenção do código.
-    *Princípio aberto/fechado*. Você pode introduzir novos tipos de
    produtos no programa sem quebrar o código cliente existente.
:::

::: col-sm-6
-    O código pode se tornar mais complicado, pois você precisa
    introduzir muitas subclasses novas para implementar o padrão. O
    melhor cenário é quando você está introduzindo o padrão em uma
    hierarquia existente de classes criadoras.
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

-   Você pode usar o [Factory Method](factory-method.html) junto com o
    [Iterator](iterator.html) para permitir que uma coleção de
    subclasses retornem diferentes tipos de iteradores que são
    compatíveis com as coleções.

-   O [Prototype](prototype.html) não é baseado em heranças, então ele
    não tem os inconvenientes dela. Por outro lado, o *Prototype*
    precisa de uma inicialização complicada do objeto clonado. O
    [Factory Method](factory-method.html) é baseado em herança mas não
    precisa de uma etapa de inicialização.

-   O [Factory Method](factory-method.html) é uma especialização do
    [Template Method](template-method.html). Ao mesmo tempo, o *Factory
    Method* pode servir como uma etapa em um *Template Method* grande.
:::

::: {.section .implementations}
##  Exemplos de código {#implementations}

[![Factory Method em
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](factory-method/csharp/example.html "Factory Method em C#"){.prog-lang-link}
[![Factory Method em
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](factory-method/cpp/example.html "Factory Method em C++"){.prog-lang-link}
[![Factory Method em
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](factory-method/go/example.html "Factory Method em Go"){.prog-lang-link}
[![Factory Method em
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](factory-method/java/example.html "Factory Method em Java"){.prog-lang-link}
[![Factory Method em
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](factory-method/php/example.html "Factory Method em PHP"){.prog-lang-link}
[![Factory Method em
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](factory-method/python/example.html "Factory Method em Python"){.prog-lang-link}
[![Factory Method em
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](factory-method/ruby/example.html "Factory Method em Ruby"){.prog-lang-link}
[![Factory Method em
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](factory-method/rust/example.html "Factory Method em Rust"){.prog-lang-link}
[![Factory Method em
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](factory-method/swift/example.html "Factory Method em Swift"){.prog-lang-link}
[![Factory Method em
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](factory-method/typescript/example.html "Factory Method em TypeScript"){.prog-lang-link}
:::

::: {.section .extras}
##  Conteúdo extra {#extras}

-   Leia nossa [Comparação Factory](factory-comparison.html) se você não
    conseguiu entender a diferença entre os vários padrões e conceitos
    Factory.
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

[Abstract Factory []{.fa .fa-arrow-right}](abstract-factory.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Padrões
criacionais](creational-patterns.html){.btn .btn-default rel="prev"}
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
