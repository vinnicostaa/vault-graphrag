::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
comportamentais](behavioral-patterns.html){.type}
:::

# Mediator {#mediator .title}

::: aka
Também conhecido como:
[Mediador, ]{style="display:inline-block"}[Intermediário, ]{style="display:inline-block"}[Intermediary, ]{style="display:inline-block"}[Controlador, ]{style="display:inline-block"}[Controller]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

O **Mediator** é um padrão de projeto comportamental que permite que
você reduza as dependências caóticas entre objetos. O padrão restringe
comunicações diretas entre objetos e os força a colaborar apenas através
do objeto mediador.

<figure class="image">
<img
src="../../images/patterns/content/mediator/mediator6761.png?id=0264bd857a231b6ea2d0c537c092e698"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/mediator/mediator.png?id=0264bd857a231b6ea2d0c537c092e698 640w,/images/patterns/content/mediator/mediator-1.5x.png?id=b3d5df41892274b5c84282bae8737441 960w,/images/patterns/content/mediator/mediator-2x.png?id=250c2bf72ca1fdee2e6d97ed5a4765f2 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Padrão de projeto Mediator" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Digamos que você tem uma caixa de diálogo para criar e editar perfis de
clientes. Ela consiste em vários controles de formulário tais como
campos de texto, caixas de seleção, botões, etc.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/problem1-pt-brc20a.png?id=1e97dcb10b710c550984a6ae837af943"
style="aspect-ratio: 2.22;"
srcset="/images/patterns/diagrams/mediator/problem1-pt-br.png?id=1e97dcb10b710c550984a6ae837af943 600w,/images/patterns/diagrams/mediator/problem1-pt-br-1.5x.png?id=69b8c2148a1e28a273fddec6da4a6743 900w,/images/patterns/diagrams/mediator/problem1-pt-br-2x.png?id=a9b879712d1efcbd54c119fca169f30c 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Relações caóticas entre elementos de uma interface de usuário" />
<figcaption><p>As relações entre os elementos da interface de usuário
podem se tornar caóticas a medida que a
aplicação evolui.</p></figcaption>
</figure>

Alguns dos elementos do formulário podem interagir com outros. Por
exemplo, selecionando a caixa de "Eu tenho um cão" pode revelar uma
caixa de texto escondida para inserir o nome do cão. Outro exemplo é o
botão enviar que tem que validar todos os campos antes de salvar os
dados.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/problem2a7f7.png?id=072c51eee4dd90c0972866440c87c1c3"
style="aspect-ratio: 3.27;"
srcset="/images/patterns/diagrams/mediator/problem2.png?id=072c51eee4dd90c0972866440c87c1c3 360w,/images/patterns/diagrams/mediator/problem2-1.5x.png?id=b805abc211e60fa567fb114192e24608 540w,/images/patterns/diagrams/mediator/problem2-2x.png?id=d298ec82a47fa2938ed6cf64b7da78c1 720w"
sizes="(max-width: 720px) 100vw, 360px" loading="lazy" width="360"
alt="Elementos da UI são interdependentes" />
<figcaption><p>Os elementos podem ter várias relações com outros
elementos. Portanto, mudanças a alguns elementos podem afetar
os outros.</p></figcaption>
</figure>

Ao ter essa lógica implementada diretamente dentro do código dos
elementos de formulários você torna as classes dos elementos muito
difíceis de se reutilizar em outros formulários da aplicação. Por
exemplo, você não será capaz de usar aquela classe de caixa de seleção
dentro de outro formulário porque ela está acoplado com o campo de texto
do nome do cão. Você pode ter ou todas as classes envolvidas na
renderização do formulário de perfil, ou nenhuma.
:::

::: {.section .solution}
##  Solução {#solution}

O padrão Mediator sugere que você deveria cessar toda comunicação direta
entre componentes que você quer tornar independentes um do outro. Ao
invés disso, esses componentes devem colaborar indiretamente, chamando
um objeto mediador especial que redireciona as chamadas para os
componentes apropriados. Como resultado, os componentes dependem apenas
de uma única classe mediadora ao invés de serem acoplados a dúzias de
outros colegas.

No nosso exemplo com o formulário de edição de perfil, a classe diálogo
em si pode agir como mediadora. O mais provável é que a classe diálogo
já esteja ciente de todos seus sub-elementos, então você não precisa
introduzir novas dependências nessa classe.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/solution1-pt-br0d90.png?id=ba812960a5a90c76b95306ea9a961fb7"
style="aspect-ratio: 2.22;"
srcset="/images/patterns/diagrams/mediator/solution1-pt-br.png?id=ba812960a5a90c76b95306ea9a961fb7 600w,/images/patterns/diagrams/mediator/solution1-pt-br-1.5x.png?id=3b17c48965b196400ab0d02bc454d35b 900w,/images/patterns/diagrams/mediator/solution1-pt-br-2x.png?id=0b0167e23ede0d15bb966b13425b482b 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Os elementos UI devem se comunicar através do mediador." />
<figcaption><p>Os elementos UI devem se comunicar indiretamente, através
do objeto mediador.</p></figcaption>
</figure>

A mudança mais significativa acontece com os próprios elementos do
formulário. Vamos considerar o botão de enviar. Antes, cada vez que um
usuário clicava no botão, ele teria que validar os valores de todos os
elementos de formulário. Agora seu único trabalho é notificar a caixa de
diálogo sobre o clique. Ao receber essa notificação, a própria caixa de
diálogo realiza as validações ou passa a tarefa para os elementos
individuais. Portanto, ao invés de estar amarrado a uma dúzia de
elementos de formulário, o botão está dependente apenas da classe
diálogo.

Você pode ir além e fazer a dependência ainda mais frouxa extraindo a
interface comum de todos os tipos de caixas de diálogo. A interface deve
declarar o método de notificação que todos os elementos do formulário
podem usar para notificar a caixa de diálogo sobre eventos acontecendo a
aqueles elementos. Portanto, nosso botão enviar deve agora ser capaz de
trabalhar com qualquer caixa de diálogo que implemente aquela interface.

Dessa forma, o padrão Mediator permite que você encapsule uma complexa
rede de relações entre vários objetos em apenas um objeto mediador.
Quanto menos dependências uma classe tenha, mais fácil essa classe se
torna para se modificar, estender, ou reutilizar.
:::

::: {.section .analogy}
##  Analogia com o mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/live-example3f2a.png?id=aa1de3cb7b63aa623e63578a1e43399a"
style="aspect-ratio: 1.85;"
srcset="/images/patterns/diagrams/mediator/live-example.png?id=aa1de3cb7b63aa623e63578a1e43399a 370w,/images/patterns/diagrams/mediator/live-example-1.5x.png?id=315109aa1099cc6e7c06057fa139881c 555w,/images/patterns/diagrams/mediator/live-example-2x.png?id=fd55a9f9d8cf49effa223555c7369504 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Torre de controle de tráfego aéreo" />
<figcaption><p>Pilotos de aeronaves não falam entre si diretamente na
hora de decidir quem é o próximo a aterrissar seu avião. Toda a
comunicação passa pela torre de controle.</p></figcaption>
</figure>

Os pilotos de aeronaves que se aproximam ou partem da área de controle
do aeroporto não se comunicam diretamente entre si. Ao invés disso falam
com um controlador de tráfego aéreo, que está sentando em uma torre alta
perto da pista de aterrissagem. Sem o controlador do tráfego aéreo os
pilotos precisariam estar cientes de cada avião nas redondezas do
aeroporto, discutindo as prioridades de aterrissagem com um comitê de
dúzias de outros pilotos. Isso provavelmente aumentaria em muito as
estatísticas de acidentes aéreos.

A torre não precisa fazer o controle de todo o voo. Ela existe apenas
para garantir o condicionamento da área do terminal devido ao número de
pessoas envolvidas ali, o que poderia ser demais para um piloto.
:::

::::: {.section .structure-container}
##  Estrutura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/structure71f0.png?id=1f2accc7820ecfe9665b6d30cbc0bc61"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.21;"
srcset="/images/patterns/diagrams/mediator/structure.png?id=1f2accc7820ecfe9665b6d30cbc0bc61 520w,/images/patterns/diagrams/mediator/structure-1.5x.png?id=958b373815bf6a56cd9b38763ed01dce 780w,/images/patterns/diagrams/mediator/structure-2x.png?id=5191daa1c9d4caa36e38af3c5b7d1522 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Estrutura do padrão de projeto Mediator" /><img
src="../../images/patterns/diagrams/mediator/structure-indexed5302.png?id=a82d4cf1b92a4f72af32f231ffd21131"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.21;"
srcset="/images/patterns/diagrams/mediator/structure-indexed.png?id=a82d4cf1b92a4f72af32f231ffd21131 520w,/images/patterns/diagrams/mediator/structure-indexed-1.5x.png?id=81d4f842ae5157a3cb09f8f3c05159dd 780w,/images/patterns/diagrams/mediator/structure-indexed-2x.png?id=88722da01a5c82b0452078c9339ca497 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Estrutura do padrão de projeto Mediator" />
</figure>
:::

1.  Os **Componentes** são várias classes que contém alguma lógica de
    negócio. Cada componente tem uma referência a um mediador, declarada
    com o tipo de interface do mediador. O componente não está ciente da
    classe atual do mediador, então você pode reutilizar o componente em
    outros programas ao ligá-lo com um mediador diferente.

2.  A interface do **Mediador** declara métodos de comunicação com os
    componentes, os quais geralmente incluem apenas um método de
    notificação. Os componentes podem passar qualquer contexto como
    argumentos desse método, incluindo seus próprios objetos, mas apenas
    de tal forma que nenhum acoplamento ocorra entre um componente
    destinatário e a classe remetente.

3.  Os **Mediadores Concretos** encapsulam as relações entre vários
    componentes. Os mediadores concretos quase sempre mantém referências
    de todos os componentes os quais gerenciam e, algumas vezes, até
    gerenciam o ciclo de vida deles.

4.  Componentes não devem estar cientes de outros componentes. Se algo
    importante acontece dentro ou para um componente, ele deve apenas
    notificar o mediador. Quando o mediador recebe a notificação, ele
    pode facilmente identificar o remetente, o que é suficiente para
    decidir que componente deve ser acionado em retorno.

    Da perspectiva de um componente, tudo parece como uma caixa preta. O
    remetente não sabe quem vai acabar lidando com o seu pedido, e o
    destinatário não sabe quem enviou o pedido em primeiro lugar.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Neste exemplo, o padrão **Mediator** ajuda você a eliminar dependências
mútuas entre várias classes UI: botões, caixas de seleção, e textos de
rótulos.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/example576a.png?id=3151c153533e816e226be0ef977211e8"
style="aspect-ratio: 1.24;"
srcset="/images/patterns/diagrams/mediator/example.png?id=3151c153533e816e226be0ef977211e8 560w,/images/patterns/diagrams/mediator/example-1.5x.png?id=cf441780d96f3306ac54d6809f85b87d 840w,/images/patterns/diagrams/mediator/example-2x.png?id=02064e5a7c4f065f806747e1b04ac1b0 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Exemplo de estrutura do padrão Mediator" />
<figcaption><p>Estrutura das classes UI caixa
de diálogo.</p></figcaption>
</figure>

Um elemento, acionado por um usuário, não se comunica com outros
elementos diretamente, mesmo que pareça que ele deva fazer isso. Ao
invés disso, o elemento apenas precisa fazer o mediador saber do evento,
passando qualquer informação de contexto junto com a notificação.

Neste exemplo, todas caixas de diálogo de autenticação agem como o
mediador. Elas sabem quais os elementos concretos devem colaborar e
facilita sua comunicação indireta. Ao receber a notificação de um
evento, a caixa de diálogo decide que elemento deve lidar com o evento e
redireciona a chamada de acordo.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// A interface mediadora declara um método usado pelos
// componentes para notificar o mediador sobre vários eventos. O
// mediador pode reagir a esses eventos e passar a execução para
// outros componentes.
interface Mediator is
    method notify(sender: Component, event: string)


// A classe mediadora concreta. A rede entrelaçada de conexões
// entre componentes individuais foi desentrelaçada e movida
// para dentro do mediador.
class AuthenticationDialog implements Mediator is
    private field title: string
    private field loginOrRegisterChkBx: Checkbox
    private field loginUsername, loginPassword: Textbox
    private field registrationUsername, registrationPassword,
                  registrationEmail: Textbox
    private field okBtn, cancelBtn: Button

    constructor AuthenticationDialog() is
        // Cria todos os objetos componentes e passa o atual
        // mediador em seus construtores para estabelecer links.

    // Quando algo acontece com um componente, ele notifica o
    // mediador. Ao receber a notificação, o mediador pode fazer
    // alguma coisa por conta própria ou passar o pedido para
    // outro componente.
    method notify(sender, event) is
        if (sender == loginOrRegisterChkBx and event == &quot;check&quot;)
            if (loginOrRegisterChkBx.checked)
                title = &quot;Log in&quot;
                // 1. Mostra componentes de formulário de login.
                // 2. Esconde componentes de formulário de
                // registro.
            else
                title = &quot;Register&quot;
                // 1. Mostra componentes de formulário de
                // registro.
                // 2. Esconde componentes de formulário de
                // login.
        if (sender == okBtn &amp;&amp; event == &quot;click&quot;)
            if (loginOrRegister.checked)
                // Tenta encontrar um usuário usando as
                // credenciais de login.
                if (!found)
                    // Mostra uma mensagem de erro acima do
                    // campo login.
            else
                // 1. Cria uma conta de usuário usando dados dos
                // campos de registro.
                // 2. Loga aquele usuário.
                // ...

// Os componentes se comunicam com o mediador usando a interface
// do mediador. Graças a isso, você pode usar os mesmos
// componentes em outros contextos ao ligá-los com diferentes
// objetos mediadores.
class Component is
    field dialog: Mediator

    constructor Component(dialog) is
        this.dialog = dialog

    method click() is
        dialog.notify(this, &quot;click&quot;)

    method keypress() is
        dialog.notify(this, &quot;keypress&quot;)

// Componentes concretos não falam entre si. Eles têm apenas um
// canal de comunicação, que é enviar notificações para o
// mediador.
class Button extends Component is
    // ...

class Textbox extends Component is
    // ...

class Checkbox extends Component is
    method check() is
        dialog.notify(this, &quot;check&quot;)
    // ...</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Aplicabilidade {#applicability}

::::::::: applicability
::: applicability-problem
Utilize o padrão Mediator quando é difícil mudar algumas das classes
porque elas estão firmemente acopladas a várias outras classes.
:::

::: applicability-solution
O padrão lhe permite extrair todas as relações entre classes para uma
classe separada, isolando quaisquer mudanças para um componente
específico do resto dos componentes.
:::

::: applicability-problem
Utilize o padrão quando você não pode reutilizar um componente em um
programa diferente porque ele é muito dependente de outros componentes.
:::

::: applicability-solution
Após você aplicar o Mediator, componentes individuais se tornam alheios
aos outros componentes. Eles ainda podem se comunicar entre si, mas de
forma indireta, através do objeto mediador. Para reutilizar um
componente em uma aplicação diferente, você precisa fornecer a ele uma
nova classe mediadora.
:::

::: applicability-problem
Utilize o Mediator quando você se encontrar criando um monte de
subclasses para componentes apenas para reutilizar algum comportamento
básico em vários contextos.
:::

::: applicability-solution
Como todas as relações entre componentes estão contidas dentro do
mediador, é fácil definir novas maneiras para esses componentes
colaborarem introduzindo novas classes mediadoras, sem ter que mudar os
próprios componentes.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Como implementar {#checklist}

1.  Identifique um grupo de classes firmemente acopladas que se
    beneficiariam de estar mais independentes (por exemplo, para uma
    manutenção ou reutilização mais fácil dessas classes).

2.  Declare a interface do mediador e descreva o protocolo de
    comunicação desejado entre os mediadores e os diversos componentes.
    Na maioria dos casos, um único método para receber notificações de
    componentes é suficiente.

    Essa interface é crucial quando você quer reutilizar classes
    componente em diferentes contextos. Desde que o componente trabalhe
    com seu mediador através da interface genérica, você pode ligar o
    componente com diferentes implementações do mediador.

3.  Implemente a classe concreta do mediador. Essa classe se beneficia
    por armazenar referências a todos os componentes que gerencia.

4.  Você pode ainda ir além e fazer que o mediador fique responsável
    pela criação e destruição de objetos componente. Após isso, o
    mediador pode montar uma [fábrica](abstract-factory.html) ou uma
    [fachada](facade.html).

5.  Componentes devem armazenar uma referência ao objeto do mediador. A
    conexão é geralmente estabelecida no construtor do componente, onde
    o objeto mediador é passado como um argumento.

6.  Mude o código dos componentes para que eles chamem o método de
    notificação do mediador ao invés de métodos de outros componentes.
    Extraia o código que envolve chamar os outros componentes para a
    classe do mediador. Execute esse código sempre que o mediador receba
    notificações daquele componente.
:::

:::::: {.section .pros-cons}
##  Prós e contras {#pros-cons}

::::: row
::: col-sm-6
-    *Princípio de responsabilidade única*. Você pode extrair as
    comunicações entre vários componentes para um único lugar, tornando
    as de mais fácil entendimento e manutenção.
-    *Princípio aberto/fechado*. Você pode introduzir novos mediadores
    sem ter que mudar os próprios componentes.
-    Você pode reduzir o acoplamento entre os vários componentes de um
    programa.
-    Você pode reutilizar componentes individuais mais facilmente.
:::

::: col-sm-6
-    Com o tempo um mediador pode evoluir para um [Objeto
    Deus](https://pt.wikipedia.org/wiki/Objeto_deus).
:::
:::::
::::::

::: {.section .relations}
##  Relações com outros padrões {#relations}

-   O [Chain of Responsibility](chain-of-responsibility.html),
    [Command](command.html), [Mediator](mediator.html) e
    [Observer](observer.html) abrangem várias maneiras de se conectar
    remetentes e destinatários de pedidos:

    -   O *Chain of Responsibility* passa um pedido sequencialmente ao
        longo de um corrente dinâmica de potenciais destinatários até
        que um deles atua no pedido.
    -   O *Command* estabelece conexões unidirecionais entre remetentes
        e destinatários.
    -   O *Mediator* elimina as conexões diretas entre remetentes e
        destinatários, forçando-os a se comunicar indiretamente através
        de um objeto mediador.
    -   O *Observer* permite que destinatários inscrevam-se ou cancelem
        sua inscrição dinamicamente para receber pedidos.

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

-   A diferença entre o [Mediator](mediator.html) e o
    [Observer](observer.html) é bem obscura. Na maioria dos casos, você
    pode implementar qualquer um desses padrões; mas às vezes você pode
    aplicar ambos simultaneamente. Vamos ver como podemos fazer isso.

    O objetivo primário do *Mediator* é eliminar dependências múltiplas
    entre um conjunto de componentes do sistema. Ao invés disso, esses
    componentes se tornam dependentes de um único objeto mediador. O
    objetivo do *Observer* é estabelecer comunicações de uma via
    dinâmicas entre objetos, onde alguns deles agem como subordinados de
    outros.

    Existe uma implementação popular do padrão Mediator que depende do
    *Observer*. O objeto mediador faz o papel de um publicador, e os
    componentes agem como assinantes que inscrevem-se ou removem a
    inscrição aos eventos do mediador. Quando o *Mediator* é
    implementado dessa forma, ele pode parecer muito similar ao
    *Observer*.

    Quando você está confuso, lembre-se que você pode implementar o
    padrão Mediator de outras maneiras. Por exemplo, você pode ligar
    permanentemente todos os componentes ao mesmo objeto mediador. Essa
    implementação não se parece com o *Observer* mas ainda irá ser uma
    instância do padrão Mediator.

    Agora imagine um programa onde todos os componentes se tornaram
    publicadores permitindo conexões dinâmicas entre si. Não haverá um
    objeto mediador centralizado, somente um conjunto distribuído de
    observadores.
:::

::: {.section .implementations}
##  Exemplos de código {#implementations}

[![Mediator em
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](mediator/csharp/example.html "Mediator em C#"){.prog-lang-link}
[![Mediator em
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](mediator/cpp/example.html "Mediator em C++"){.prog-lang-link}
[![Mediator em
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](mediator/go/example.html "Mediator em Go"){.prog-lang-link}
[![Mediator em
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](mediator/java/example.html "Mediator em Java"){.prog-lang-link}
[![Mediator em
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](mediator/php/example.html "Mediator em PHP"){.prog-lang-link}
[![Mediator em
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](mediator/python/example.html "Mediator em Python"){.prog-lang-link}
[![Mediator em
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](mediator/ruby/example.html "Mediator em Ruby"){.prog-lang-link}
[![Mediator em
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](mediator/rust/example.html "Mediator em Rust"){.prog-lang-link}
[![Mediator em
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](mediator/swift/example.html "Mediator em Swift"){.prog-lang-link}
[![Mediator em
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](mediator/typescript/example.html "Mediator em TypeScript"){.prog-lang-link}
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

[Memento []{.fa .fa-arrow-right}](memento.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Iterator](iterator.html){.btn .btn-default
rel="prev"}
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
