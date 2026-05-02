:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
estruturais](structural-patterns.html){.type}
:::

# Bridge {#bridge .title}

::: aka
Também conhecido como: [Ponte]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

O **Bridge** é um padrão de projeto estrutural que permite que você
divida uma classe grande ou um conjunto de classes intimamente ligadas
em duas hierarquias separadas---abstração e implementação---que podem
ser desenvolvidas independentemente umas das outras.

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge3e59.png?id=bd543d4fb32e11647767301581a5ad54"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/bridge/bridge.png?id=bd543d4fb32e11647767301581a5ad54 640w,/images/patterns/content/bridge/bridge-1.5x.png?id=8ef07b78d61cc3830b7e2b783c76c775 960w,/images/patterns/content/bridge/bridge-2x.png?id=1e905ae5742e5cd10a7eb0e3175ef00d 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Padrão de projeto Bridge" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

*Abstração?* *Implementação?* Soam um pouco assustadoras? Fique calmo
que vamos considerar um exemplo simples.

Digamos que você tem uma classe `Forma` geométrica com um par de
subclasses: `Círculo` e `Quadrado`. Você quer estender essa hierarquia
de classe para incorporar cores, então você planeja criar as subclasses
de forma `Vermelho` e `Azul`. Contudo, já que você já tem duas
subclasses, você precisa criar quatro combinações de classe tais como
`CírculoAzul` e `QuadradoVermelho`.

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/problem-pt-br1871.png?id=a4d713653cca80a3234df1cf04a586e5"
style="aspect-ratio: 1.41;"
srcset="/images/patterns/diagrams/bridge/problem-pt-br.png?id=a4d713653cca80a3234df1cf04a586e5 480w,/images/patterns/diagrams/bridge/problem-pt-br-1.5x.png?id=2389ff0ff9f755794030973bb54dbeac 720w,/images/patterns/diagrams/bridge/problem-pt-br-2x.png?id=fb76d8fac2eb06885c8aa795523f5a71 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Problema padrão Bridge" />
<figcaption><p>O número de combinações de classes cresce em
progressão geométrica.</p></figcaption>
</figure>

Adicionar novos tipos de forma e cores à hierarquia irá fazê-la crescer
exponencialmente. Por exemplo, para adicionar uma forma de triângulo
você vai precisar introduzir duas subclasses, uma para cada cor. E
depois disso, adicionando uma nova cor será necessário três subclasses,
uma para cada tipo de forma. Quanto mais longe vamos, pior isso fica.
:::

::: {.section .solution}
##  Solução {#solution}

Esse problema ocorre porque estamos tentando estender as classes de
forma em duas dimensões diferentes: por forma e por cor. Isso é um
problema muito comum com herança de classe.

O padrão Bridge tenta resolver esse problema ao trocar de herança para
composição do objeto. Isso significa que você extrai uma das dimensões
em uma hierarquia de classe separada, para que as classes originais
referenciem um objeto da nova hierarquia, ao invés de ter todos os seus
estados e comportamentos dentro de uma classe.

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/solution-pt-brda5a.png?id=c62660cbad7604f004d3a202a6b3a5dc"
style="aspect-ratio: 2.3;"
srcset="/images/patterns/diagrams/bridge/solution-pt-br.png?id=c62660cbad7604f004d3a202a6b3a5dc 460w,/images/patterns/diagrams/bridge/solution-pt-br-1.5x.png?id=6b5b1db693975b50d007df51ed141507 690w,/images/patterns/diagrams/bridge/solution-pt-br-2x.png?id=b6be8fdc8bf3b08d9a626f92a89c8bc4 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Solução sugerida pelo padrão Bridge" />
<figcaption><p>Você pode prevenir a explosão de uma hierarquia de classe
ao transformá-la em diversas hierarquias relacionadas.</p></figcaption>
</figure>

Seguindo essa abordagem nós podemos extrair o código relacionado à cor
em sua própria classe com duas subclasses: `Vermelho` e `Azul`. A classe
`Forma` então ganha um campo de referência apontando para um dos objetos
de cor. Agora a forma pode delegar qualquer trabalho referente a cor
para o objeto ligado a cor. Aquela referência vai agir como uma ponte
entre as classes `Forma` e `Cor`. De agora em diante, para adicionar
novas cores não será necessário mudar a hierarquia da forma e vice
versa.

#### Abstração e implementação

O livro GoF []{.pop-i}["Gang of Four" (Gangue dos Quatro) é um apelido
dado aos quatro autores do livro original sobre padrões de projeto:
*Padrões de Projeto: Elementos de Software Reutilizáveis Orientados a
Objetos*
[https://refactoring.guru/pt-br/gof-book](https://www.amazon.com/gp/product/8573076100/).]{.pop-content}
introduz os termos *Abstração* e *Implementação* como parte da definição
do Bridge. Na minha opinião, os termos soam muito acadêmicos e fazem o
padrão parecer algo mais complicado do que realmente é. Tendo lido o
exemplo simples com formas e cores, vamos decifrar o significado por
trás das palavras assustadoras do livro GoF.

*Abstração* (também chamado de *interface*) é uma camada de controle de
alto nível para alguma entidade. Essa camada não deve fazer nenhum tipo
de trabalho por conta própria. Ela deve delegar o trabalho para a camada
de *implementação* (também chamada de *plataforma*).

Observe que não estamos falando sobre *interfaces* ou *classes
abstratas* da sua linguagem de programação. São coisas diferentes.

Quando falamos sobre aplicações reais, a abstração pode ser representada
por uma interface gráfica de usuário (GUI), e a implementação pode ser o
código subjacente do sistema operacional (API) a qual a camada GUI chama
em resposta às interações do usuário.

Geralmente falando, você pode estender tal aplicação em duas direções
independentes:

-   Ter diversas GUIs diferentes (por exemplo, feitas para clientes
    regulares ou administradores).
-   Suportar diversas APIs diferente (por exemplo, para ser capaz de
    lançar a aplicação no Windows, Linux e macOS).

No pior dos cenários, essa aplicação pode se parecer como uma enorme
tigela de espaguete onde centenas de condicionais conectam diferentes
tipos de GUI com vários APIs por todo o código.

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge-3-pt-brbdc0.png?id=16ec535f13c8a40f4aa6cb0ea831f6a8"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/bridge/bridge-3-pt-br.png?id=16ec535f13c8a40f4aa6cb0ea831f6a8 600w,/images/patterns/content/bridge/bridge-3-pt-br-1.5x.png?id=2470af38ee3faa47742e2461e33bd743 900w,/images/patterns/content/bridge/bridge-3-pt-br-2x.png?id=b4589067db0aea04ed2cd5b842bddb5b 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Lidar com modificações é muito mais fácil em um código modular" />
<figcaption><p>Fazer até mesmo uma única mudança em uma base de códigos
monolítica é bastante difícil porque você deve entender <em>a coisa
toda</em> muito bem. Fazer mudanças em módulos menores e bem definidos é
muito mais fácil.</p></figcaption>
</figure>

Você pode trazer ordem para esse caos extraindo o código relacionado a
específicas combinações interface-plataforma para classes separadas.
Contudo, logo você vai descobrir que existirão *muitas* dessas classes.
A hierarquia de classes irá crescer exponencialmente porque adicionando
um novo GUI ou suportando um API diferente irá ser necessário criar mais
e mais classes.

Vamos tentar resolver esse problema com o padrão Bridge. Ele sugere que
as classes sejam divididas em duas hierarquias:

-   Abstração: a camada GUI da aplicação.
-   Implementação: As APIs do sistema operacional.

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge-2-pt-br2007.png?id=cc78547801bb0fd3e95accca562dbeb5"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/bridge/bridge-2-pt-br.png?id=cc78547801bb0fd3e95accca562dbeb5 640w,/images/patterns/content/bridge/bridge-2-pt-br-1.5x.png?id=80026993504989ad15b8116675a2e3e2 960w,/images/patterns/content/bridge/bridge-2-pt-br-2x.png?id=b82a140fafd8fb293f4fed60248d2310 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Arquitetura multiplataforma" />
<figcaption><p>Uma das maneiras de se estruturar uma
aplicação multiplataforma.</p></figcaption>
</figure>

O objeto da abstração controla a aparência da aplicação, delegando o
verdadeiro trabalho para o objeto de implementação ligado.
Implementações diferentes são intercambiáveis desde que sigam uma
interface comum, permitindo que a mesma GUI trabalhe no Windows e Linux.

Como resultado você pode mudar as classes GUI sem tocar nas classes
ligadas a API. Além disso, adicionar suporte para outro sistema
operacional só requer a criação de uma subclasse na hierarquia de
implementação.
:::

::::: {.section .structure-container}
##  Estrutura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/structure-pt-br0416.png?id=0c5664f80a5992ffd9c65570e7531d71"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.47;"
srcset="/images/patterns/diagrams/bridge/structure-pt-br.png?id=0c5664f80a5992ffd9c65570e7531d71 560w,/images/patterns/diagrams/bridge/structure-pt-br-1.5x.png?id=c11cc8fec7865613006f0a22c658e6c3 840w,/images/patterns/diagrams/bridge/structure-pt-br-2x.png?id=315b68833ef896bb21044b64561f6ea8 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Padrão de projeto Bridge" /><img
src="../../images/patterns/diagrams/bridge/structure-pt-br-indexed9a31.png?id=f3235537cd9f2312e2ae65698e00feb2"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.44;"
srcset="/images/patterns/diagrams/bridge/structure-pt-br-indexed.png?id=f3235537cd9f2312e2ae65698e00feb2 560w,/images/patterns/diagrams/bridge/structure-pt-br-indexed-1.5x.png?id=c7bc6c1c2a7168a6f8d4031c1b3deea4 840w,/images/patterns/diagrams/bridge/structure-pt-br-indexed-2x.png?id=f300083d84f0d0441373ec3edfd63f0c 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Padrão de projeto Bridge" />
</figure>
:::

1.  A **Abstração** fornece a lógica de controle de alto nível. Ela
    depende do objeto de implementação para fazer o verdadeiro trabalho
    de baixo nível.

2.  A **Implementação** declara a interface que é comum para todas as
    implementações concretas. Um abstração só pode se comunicar com um
    objeto de implementação através de métodos que são declarados aqui.

    A abstração pode listar os mesmos métodos que a implementação, mas
    geralmente a abstração declara alguns comportamentos complexos que
    dependem de uma ampla variedade de operações primitivas declaradas
    pela implementação.

3.  **Implementações Concretas** contém código plataforma-específicos.

4.  **Abstrações Refinadas** fornecem variantes para controle da lógica.
    Como seu superior, trabalham com diferentes implementações através
    da interface geral de implementação.

5.  Geralmente o **Cliente** está apenas interessado em trabalhar com a
    abstração. Contudo, é trabalho do cliente ligar o objeto de
    abstração com um dos objetos de implementação.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Este exemplo ilustra como o padrão **Bridge** pode ajudar a dividir o
código monolítico de uma aplicação que gerencia dispositivos e seus
controles remotos. As classes `Dispositivo` agem como a implementação,
enquanto que as classes `Controle` agem como abstração.

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/example-enab39.png?id=89c406a189c45885004d7fa094f616b1"
style="aspect-ratio: 1.33;"
srcset="/images/patterns/diagrams/bridge/example-en.png?id=89c406a189c45885004d7fa094f616b1 640w,/images/patterns/diagrams/bridge/example-en-1.5x.png?id=bb870a109edc52920a51b3fc9110e7f4 960w,/images/patterns/diagrams/bridge/example-en-2x.png?id=19bb8272b4082c5f47c4d047e6cb9967 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Exemplo da estrutura do padrão Bridge" />
<figcaption><p>A hierarquia de classe original é dividida em duas
partes: dispositivos e controles remotos.</p></figcaption>
</figure>

A classe de controle remoto base declara um campo de referência que liga
ela com um objeto de dispositivo. Todos os controles trabalham com
dispositivos através da interface geral de dispositivo, que permite que
o mesmo controle suporte múltiplos tipos de dispositivo.

Você pode desenvolver as classes de controle remoto independentemente
das classes de dispositivo. Tudo que é necessário é criar uma nova
subclasse de controle. Por exemplo, um controle remoto básico pode ter
apenas dois botões, mas você pode estendê-lo com funcionalidades
adicionais, tais como uma bateria adicional ou touchscreen.

O código cliente liga o tipo de controle remoto desejado com um objeto
dispositivo específico através do construtor do controle.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// A &quot;abstração&quot; define a interface para a parte &quot;controle&quot; das
// duas hierarquias de classe. Ela mantém uma referência a um
// objeto da hierarquia de &quot;implementação&quot; e delega todo o
// trabalho real para esse objeto.
class RemoteControl is
    protected field device: Device
    constructor RemoteControl(device: Device) is
        this.device = device
    method togglePower() is
        if (device.isEnabled()) then
            device.disable()
        else
            device.enable()
    method volumeDown() is
        device.setVolume(device.getVolume() - 10)
    method volumeUp() is
        device.setVolume(device.getVolume() + 10)
    method channelDown() is
        device.setChannel(device.getChannel() - 1)
    method channelUp() is
        device.setChannel(device.getChannel() + 1)


// Você pode estender classes a partir dessa hierarquia de
// abstração independentemente das classes de dispositivo.
class AdvancedRemoteControl extends RemoteControl is
    method mute() is
        device.setVolume(0)


// A interface &quot;implementação&quot; declara métodos comuns a todas as
// classes concretas de implementação. Ela não precisa coincidir
// com a interface de abstração. Na verdade, as duas interfaces
// podem ser inteiramente diferentes. Tipicamente a interface de
// implementação fornece apenas operações primitivas, enquanto
// que a abstração define operações de alto nível baseada
// naquelas primitivas.
interface Device is
    method isEnabled()
    method enable()
    method disable()
    method getVolume()
    method setVolume(percent)
    method getChannel()
    method setChannel(channel)


// Todos os dispositivos seguem a mesma interface.
class Tv implements Device is
    // ...

class Radio implements Device is
    // ...


// Em algum lugar no código cliente.
tv = new Tv()
remote = new RemoteControl(tv)
remote.togglePower()

radio = new Radio()
remote = new AdvancedRemoteControl(radio)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Aplicabilidade {#applicability}

::::::::: applicability
::: applicability-problem
Utilize o padrão Bridge quando você quer dividir e organizar uma classe
monolítica que tem diversas variantes da mesma funcionalidade (por
exemplo, se a classe pode trabalhar com diversos servidores de base de
dados).
:::

::: applicability-solution
Quanto maior a classe se torna, mais difícil é de entender como ela
funciona, e mais tempo se leva para fazer mudanças. As mudanças feitas
para uma das variações de funcionalidade podem precisar de mudanças
feitas em toda a classe, o que quase sempre resulta em erros ou falha em
lidar com efeitos colaterais.

O padrão Bridge permite que você divida uma classe monolítica em
diversas hierarquias de classe. Após isso, você pode modificar as
classes em cada hierarquia independentemente das classes nas outras.
Essa abordagem simplifica a manutenção do código e minimiza o risco de
quebrar o código existente.
:::

::: applicability-problem
Utilize o padrão quando você precisa estender uma classe em diversas
dimensões ortogonais (independentes).
:::

::: applicability-solution
O Bridge sugere que você extraia uma hierarquia de classe separada para
cada uma das dimensões. A classe original delega o trabalho relacionado
para os objetos pertencentes àquelas hierarquias ao invés de fazer tudo
por conta própria.
:::

::: applicability-problem
Utilize o Bridge se você precisar ser capaz de trocar implementações
durante o momento de execução.
:::

::: applicability-solution
Embora seja opcional, o padrão Bridge permite que você substitua o
objeto de implementação dentro da abstração. É tão fácil quanto designar
um novo valor para um campo.

*A propósito, este último item é o maior motivo pelo qual muitas pessoas
confundem o Bridge com o padrão [Strategy](strategy.html). Lembre-se que
um padrão é mais que apenas uma maneira de estruturar suas classes. Ele
também pode comunicar intenções e resolver um problema.*
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Como implementar {#checklist}

1.  Identifique as dimensões ortogonais em suas classes. Esses conceitos
    independentes podem ser: abstração/plataforma,
    domínio/infraestrutura, front-end/back-end, ou
    interface/implementação.

2.  Veja quais operações o cliente precisa e defina-as na classe
    abstração base.

3.  Determine as operações disponíveis em todas as plataformas. Declare
    aquelas que a abstração precisa na interface geral de implementação.

4.  Crie classes concretas de implementação para todas as plataformas de
    seu domínio, mas certifique-se que todas elas sigam a interface de
    implementação.

5.  Dentro da classe de abstração, adicione um campo de referência para
    o tipo de implementação. A abstração delega a maior parte do
    trabalho para o objeto de implementação que foi referenciado neste
    campo.

6.  Se você tem diversas variantes de lógica de alto nível, crie
    abstrações refinadas para cada variante estendendo a classe de
    abstração básica.

7.  O código cliente deve passar um objeto de implementação para o
    construtor da abstração para associar um com o outro. Após isso, o
    cliente pode esquecer sobre a implementação e trabalhar apenas com o
    objeto de abstração.
:::

:::::: {.section .pros-cons}
##  Prós e contras {#pros-cons}

::::: row
::: col-sm-6
-    Você pode criar classes e aplicações independentes de plataforma.
-    O código cliente trabalha com abstrações em alto nível. Ele não
    fica exposto a detalhes de plataforma.
-    *Princípio aberto/fechado*. Você pode introduzir novas abstrações e
    implementações independentemente uma das outras.
-    *Princípio de responsabilidade única*. Você pode focar na lógica de
    alto nível na abstração e em detalhes de plataforma na
    implementação.
:::

::: col-sm-6
-    Você pode tornar o código mais complicado ao aplicar o padrão em
    uma classe altamente coesa.
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

-   O [Bridge](bridge.html), [State](state.html),
    [Strategy](strategy.html) (e de certa forma o
    [Adapter](adapter.html)) têm estruturas muito parecidas. De fato,
    todos esses padrões estão baseados em composição, o que é delegar o
    trabalho para outros objetos. Contudo, eles todos resolvem problemas
    diferentes. Um padrão não é apenas uma receita para estruturar seu
    código de uma maneira específica. Ele também pode comunicar a outros
    desenvolvedores o problema que o padrão resolve.

-   Você pode usar o [Abstract Factory](abstract-factory.html) junto com
    o [Bridge](bridge.html). Esse pareamento é útil quando algumas
    abstrações definidas pelo *Bridge* só podem trabalhar com
    implementações específicas. Neste caso, o *Abstract Factory* pode
    encapsular essas relações e esconder a complexidade do código
    cliente.

-   Você pode combinar o [Builder](builder.html) com o
    [Bridge](bridge.html): a classe *diretor* tem um papel de abstração,
    enquanto que diferentes *construtores* agem como *implementações*.
:::

::: {.section .implementations}
##  Exemplos de código {#implementations}

[![Bridge em
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](bridge/csharp/example.html "Bridge em C#"){.prog-lang-link}
[![Bridge em
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](bridge/cpp/example.html "Bridge em C++"){.prog-lang-link}
[![Bridge em
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](bridge/go/example.html "Bridge em Go"){.prog-lang-link}
[![Bridge em
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](bridge/java/example.html "Bridge em Java"){.prog-lang-link}
[![Bridge em
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](bridge/php/example.html "Bridge em PHP"){.prog-lang-link}
[![Bridge em
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](bridge/python/example.html "Bridge em Python"){.prog-lang-link}
[![Bridge em
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](bridge/ruby/example.html "Bridge em Ruby"){.prog-lang-link}
[![Bridge em
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](bridge/rust/example.html "Bridge em Rust"){.prog-lang-link}
[![Bridge em
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](bridge/swift/example.html "Bridge em Swift"){.prog-lang-link}
[![Bridge em
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](bridge/typescript/example.html "Bridge em TypeScript"){.prog-lang-link}
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

[Composite []{.fa .fa-arrow-right}](composite.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Adapter](adapter.html){.btn .btn-default
rel="prev"}
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
