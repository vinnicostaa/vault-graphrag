::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
comportamentais](behavioral-patterns.html){.type}
:::

# State {#state .title}

::: aka
Também conhecido como: [Estado]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

O **State** é um padrão de projeto comportamental que permite que um
objeto altere seu comportamento quando seu estado interno muda. Parece
como se o objeto mudasse de classe.

<figure class="image">
<img
src="../../images/patterns/content/state/state-pt-bre805.png?id=942966fd4dc52793ca2f19f9da6fff3c"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/state/state-pt-br.png?id=942966fd4dc52793ca2f19f9da6fff3c 640w,/images/patterns/content/state/state-pt-br-1.5x.png?id=aa3bcc3878ecee2c20d9137ecf91c01a 960w,/images/patterns/content/state/state-pt-br-2x.png?id=f415eba40971b9d3585055dc3d49ad23 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Padrão de Projeto State" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

O padrão State é intimamente relacionado com o conceito de uma *Máquina
de Estado Finito* []{.pop-i}[Máquina de Estado Finito:
[https://refactoring.guru/pt-br/fsm](https://pt.wikipedia.org/wiki/M%C3%A1quina_de_estados_finita)]{.pop-content}.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/problem10244.png?id=503968745461a0970d1fbb4362dc186f"
style="aspect-ratio: 1.45;"
srcset="/images/patterns/diagrams/state/problem1.png?id=503968745461a0970d1fbb4362dc186f 320w,/images/patterns/diagrams/state/problem1-1.5x.png?id=47c7ca068eceaa9896d320690e6f3368 480w,/images/patterns/diagrams/state/problem1-2x.png?id=ae03c2233939eace11d44925ddeb912d 640w"
sizes="(max-width: 720px) 100vw, 320px" loading="lazy" width="320"
alt="Máquina de Estado Finito" />
<figcaption><p>Máquina de Estado Finito.</p></figcaption>
</figure>

A ideia principal é que, em qualquer dado momento, há um número *finito*
de *estados* que um programa possa estar. Dentro de qualquer estado
único, o programa se comporta de forma diferente, e o programa pode ser
trocado de um estado para outro instantaneamente. Contudo, dependendo do
estado atual, o programa pode ou não trocar para outros estados. Essas
regras de troca, chamadas *transições*, também são finitas e pré
determinadas.

Você também pode aplicar essa abordagem para objetos. Imagine que nós
temos uma classe `Documento`. Um documento pode estar em um de três
estados: `Rascunho`, `Moderação` e `Publicado`. O método `publicar` do
documento funciona um pouco diferente em cada estado:

-   No `Rascunho`, ele move o documento para a moderação.
-   Na `Moderação` ele torna o documento público, mas apenas se o
    usuário atual é um administrador.
-   No `Publicado` ele não faz nada.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/problem2-pt-brc783.png?id=0c8759a32d5589d94830f0d707e037ec"
style="aspect-ratio: 1.27;"
srcset="/images/patterns/diagrams/state/problem2-pt-br.png?id=0c8759a32d5589d94830f0d707e037ec 560w,/images/patterns/diagrams/state/problem2-pt-br-1.5x.png?id=6d10db62e92faa4001ebcbbc5167b79d 840w,/images/patterns/diagrams/state/problem2-pt-br-2x.png?id=070a55f4b7e31f61a4b9512e93f0b8a8 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Possíveis estados de um objeto documento" />
<figcaption><p>Possíveis estados e transições de um
objeto documento.</p></figcaption>
</figure>

Máquinas de estado são geralmente implementadas com muitos operadores de
condicionais (`if` ou `switch`) que selecionam o comportamento
apropriado dependendo do estado atual do objeto. Geralmente esse
"estado" é apenas um conjunto de valores dos campos do objeto. Mesmo se
você nunca ouviu falar sobre máquinas de estado finito antes, você
provavelmente já implementou um estado ao menos uma vez. A seguinte
estrutura de código lembra alguma coisa para você?

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Document is
    field state: string
    // ...
    method publish() is
        switch (state)
            &quot;draft&quot;:
                state = &quot;moderation&quot;
                break
            &quot;moderation&quot;:
                if (currentUser.role == &quot;admin&quot;)
                    state = &quot;published&quot;
                break
            &quot;published&quot;:
                // Não fazer nada.
                break
    // ...</code></pre>
</figure>

A maior fraqueza de uma máquina de estados baseada em condicionais se
revela quando começamos a adicionar mais e mais estados e comportamentos
baseados em estados para a classe `Documento`. A maioria dos métodos irá
conter condicionais monstruosas que selecionam o comportamento
apropriado de um método de acordo com o estado atual. Um código como
esse é muito difícil de se fazer manutenção porque qualquer mudança na
lógica de transição pode necessitar de mudanças de condicionais de
estado em todos os métodos.

O problema tende a ficar maior a medida que o projeto evolui. É muito
difícil prever todos os possíveis estados e transições no estágio
inicial de projeto. Portanto, uma máquina de estados enxuta, construída
com um número limitado de condicionais pode se tornar uma massa inchada
e disforme com o tempo.
:::

::: {.section .solution}
##  Solução {#solution}

O padrão State sugere que você crie novas classes para todos os estados
possíveis de um objeto e extraia todos os comportamentos específicos de
estados para dentro dessas classes.

Ao invés de implementar todos os comportamentos por conta própria, o
objeto original, chamado *contexto*, armazena uma referência para um dos
objetos de estado que representa seu estado atual, e delega todo o
trabalho relacionado aos estados para aquele objeto.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/solution-pt-bref75.png?id=0946e1897194cdfd595d5b7dd581bc5e"
style="aspect-ratio: 1.53;"
srcset="/images/patterns/diagrams/state/solution-pt-br.png?id=0946e1897194cdfd595d5b7dd581bc5e 490w,/images/patterns/diagrams/state/solution-pt-br-1.5x.png?id=13d8de7983d941d693b53bc6b547ae2b 735w,/images/patterns/diagrams/state/solution-pt-br-2x.png?id=ef1c1626cb538bda28b4d16129ae0b3d 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="O documento delega o trabalho para um objeto de estado" />
<figcaption><p>O documento delega o trabalho para um objeto
de estado.</p></figcaption>
</figure>

Para fazer a transição do contexto para outro estado, substitua o objeto
do estado ativo por outro objeto que represente o novo estado. Isso é
possível somente se todas as classes de estado seguirem a mesma
interface e o próprio contexto funcione com esses objetos através
daquela interface.

Essa estrutura pode ser parecida com o padrão [Strategy](strategy.html),
mas há uma diferença chave. No padrão State, os estados em particular
podem estar cientes de cada um e iniciar transições de um estado para
outro, enquanto que estratégias quase nunca sabem sobre as outras
estratégias.
:::

::: {.section .analogy}
##  Analogia com o mundo real {#analogy}

Os botões e interruptores de seu smartphone comportam-se de forma
diferente dependendo do estado atual do dispositivo:

-   Quando o telefone está desbloqueado, apertar os botões leva eles a
    executar várias funções.
-   Quando o telefone está bloqueado, apertar qualquer botão leva a
    desbloquear a tela.
-   Quando a carga da bateria está baixa, apertar qualquer botão mostra
    a tela de carregamento.
:::

::::: {.section .structure-container}
##  Estrutura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/state/structure-pt-brc41d.png?id=50efedb80eab6994524bb46a10669134"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.32;"
srcset="/images/patterns/diagrams/state/structure-pt-br.png?id=50efedb80eab6994524bb46a10669134 540w,/images/patterns/diagrams/state/structure-pt-br-1.5x.png?id=6f61edc1727994cf2101ca50ebc2a0f6 810w,/images/patterns/diagrams/state/structure-pt-br-2x.png?id=758acf6e428ac1e9703cca2f9d92a5ac 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Estrutura do padrão de projeto State" /><img
src="../../images/patterns/diagrams/state/structure-pt-br-indexed72ce.png?id=afdc003c9d3ffb2f21f5c2d4346afb1f"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.26;"
srcset="/images/patterns/diagrams/state/structure-pt-br-indexed.png?id=afdc003c9d3ffb2f21f5c2d4346afb1f 540w,/images/patterns/diagrams/state/structure-pt-br-indexed-1.5x.png?id=c6cf75844e21c319c46dd6792a3fad06 810w,/images/patterns/diagrams/state/structure-pt-br-indexed-2x.png?id=ec7ad2f5908157032b0a651311b30837 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Estrutura do padrão de projeto State" />
</figure>
:::

1.  O **Contexto** armazena uma referência a um dos objetos concretos de
    estado e delega a eles todos os trabalhos específicos de estado. O
    contexto se comunica com o objeto estado através da interface do
    estado. O contexto expõe um setter para passar a ele um novo objeto
    de estado.

2.  A interface do **Estado** declara métodos específicos a estados.
    Esses métodos devem fazer sentido para todos os estados concretos
    porque você não quer alguns dos seus estados tendo métodos inúteis
    que nunca irão ser chamados.

3.  Os **Estados Concretos** fornecem suas próprias implementações para
    os métodos específicos de estados. Para evitar duplicação ou código
    parecido em múltiplos estados, você pode fornecer classes abstratas
    intermediárias que encapsulam alguns dos comportamentos comuns.

    Objetos de estado podem armazenar referências retroativas para o
    objeto de contexto. Através dessa referência o estado pode buscar
    qualquer informação desejada do objeto contexto, assim como iniciar
    transições de estado.

4.  Ambos os estados de contexto e concretos podem configurar o próximo
    estado do contexto e realizar a atual transição de estado ao
    substituir o objeto estado ligado ao contexto.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Neste exemplo, o padrão **State** permite que os mesmos controles de
tocador de mídia se comportem diferentemente, dependendo do atual estado
do tocador.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/example1dc9.png?id=1ffdb109b3ebb85d223b9d1651d2034c"
style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/state/example.png?id=1ffdb109b3ebb85d223b9d1651d2034c 590w,/images/patterns/diagrams/state/example-1.5x.png?id=a9ff56d0a139530fa103d496513dfa06 885w,/images/patterns/diagrams/state/example-2x.png?id=cd81e3ffb8aba5883983a59c111b805f 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Exemplo de estrutura do padrão State" />
<figcaption><p>Exemplo da troca do comportamento de um objeto com
objetos de estado.</p></figcaption>
</figure>

O objeto principal do tocador está sempre ligado ao objeto estado que
realiza a maior parte do trabalho para o tocador. Algumas ações
substituem o objeto do estado atual do tocador por outro, que muda a
maneira do tocador reagir às interações do usuário.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// A classe AudioPlayer age como um contexto. Ela também mantém
// uma referência para uma instância de uma das classes de
// estado que representa o atual estado do tocador de áudio.
class AudioPlayer is
    field state: State
    field UI, volume, playlist, currentSong

    constructor AudioPlayer() is
        this.state = new ReadyState(this)

        // O contexto delega o manuseio das entradas do usuário
        // para um objeto de estado. Naturalmente, o resultado
        // depende de qual estado está ativo, uma vez que cada
        // estado pode lidar com as entradas de forma diferente.
        UI = new UserInterface()
        UI.lockButton.onClick(this.clickLock)
        UI.playButton.onClick(this.clickPlay)
        UI.nextButton.onClick(this.clickNext)
        UI.prevButton.onClick(this.clickPrevious)

    // Outros objetos devem ser capazes de trocar o estado ativo
    // do tocador.
    method changeState(state: State) is
        this.state = state

    // Métodos de UI delegam a execução para o estado ativo.
    method clickLock() is
        state.clickLock()
    method clickPlay() is
        state.clickPlay()
    method clickNext() is
        state.clickNext()
    method clickPrevious() is
        state.clickPrevious()

    // Um estado pode chamar alguns métodos de serviço no
    // contexto.
    method startPlayback() is
        // ...
    method stopPlayback() is
        // ...
    method nextSong() is
        // ...
    method previousSong() is
        // ...
    method fastForward(time) is
        // ...
    method rewind(time) is
        // ...

// A classe de estado base declara métodos que todos os estados
// concretos devem implementar e também fornece uma referência
// anterior ao objeto de contexto associado com o estado.
// Estados podem usar a referência anterior para realizar a
// transição contexto para outro estado.
abstract class State is
    protected field player: AudioPlayer

    // O contexto passa a si mesmo através do construtor do
    // estado. Isso pode ajudar o estado a recuperar alguns
    // dados de contexto úteis se for necessário.
    constructor State(player) is
        this.player = player

    abstract method clickLock()
    abstract method clickPlay()
    abstract method clickNext()
    abstract method clickPrevious()


// Estados concretos implementam vários comportamentos
// associados com um estado do contexto.
class LockedState extends State is

    // Quando você desbloqueia um tocador bloqueado, ele vai
    // assumir um dos dois estados.
    method clickLock() is
        if (player.playing)
            player.changeState(new PlayingState(player))
        else
            player.changeState(new ReadyState(player))

    method clickPlay() is
        // Bloqueado, então não faz nada.

    method clickNext() is
        // Bloqueado, então não faz nada.

    method clickPrevious() is
        // Bloqueado, então não faz nada.


// Eles também podem ativar transições de estado no contexto.
class ReadyState extends State is
    method clickLock() is
        player.changeState(new LockedState(player))

    method clickPlay() is
        player.startPlayback()
        player.changeState(new PlayingState(player))

    method clickNext() is
        player.nextSong()

    method clickPrevious() is
        player.previousSong()


class PlayingState extends State is
    method clickLock() is
        player.changeState(new LockedState(player))

    method clickPlay() is
        player.stopPlayback()
        player.changeState(new ReadyState(player))

    method clickNext() is
        if (event.doubleclick)
            player.nextSong()
        else
            player.fastForward(5)

    method clickPrevious() is
        if (event.doubleclick)
            player.previous()
        else
            player.rewind(5)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Aplicabilidade {#applicability}

::::::::: applicability
::: applicability-problem
Utilize o padrão State quando você tem um objeto que se comporta de
maneira diferente dependendo do seu estado atual, quando o número de
estados é enorme, e quando o código estado específico muda com
frequência.
:::

::: applicability-solution
O padrão sugere que você extraia todo o código estado específico para um
conjunto de classes distintas. Como resultado, você pode adicionar novos
estados ou mudar os existentes independentemente uns dos outros,
reduzindo o custo da manutenção.
:::

::: applicability-problem
Utilize o padrão quando você tem uma classe populada com condicionais
gigantes que alteram como a classe se comporta de acordo com os valores
atuais dos campos da classe.
:::

::: applicability-solution
O padrão State permite que você extraia ramificações dessas condicionais
para dentro de métodos de classes correspondentes. Ao fazer isso, você
também limpa para fora da classe principal os campos temporários e os
métodos auxiliares envolvidos no código estado específico.
:::

::: applicability-problem
Utilize o State quando você tem muito código duplicado em muitos estados
parecidos e transições de uma máquina de estado baseada em condições.
:::

::: applicability-solution
O padrão State permite que você componha hierarquias de classes estado e
reduza a duplicação ao extrair código comum para dentro de classes
abstratas base.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Como implementar {#checklist}

1.  Decida qual classe vai agir como contexto. Poderia ser uma classe
    existente que já tenha código dependente do estado; ou uma nova
    classe, se o código específico ao estado estiver distribuído em
    múltiplas classes.

2.  Declare a interface do estado. Embora ela vai espelhar todos os
    métodos declarados no contexto, mire apenas para aqueles que possam
    conter comportamento específico ao estado.

3.  Para cada estado real, crie uma classe que deriva da interface do
    estado. Então vá para os métodos do contexto e extraia todo o código
    relacionado a aquele estado para dentro de sua nova classe.

    Ao mover o código para a classe estado, você pode descobrir que ela
    depende de membros privados do contexto. Há várias maneiras de
    contornar isso:

    -   Torne esses campos ou métodos públicos.
    -   Transforme o comportamento que você está extraindo para um
        método público dentro do contexto e chame-o na classe estado.
        Essa maneira é feia mas rápida, e você pode sempre consertá-la
        mais tarde.
    -   Aninhe as classes estado dentro da classe contexto, mas apenas
        se sua linguagem de programação suporta classes aninhadas.

4.  Na classe contexto, adicione um campo de referência do tipo de
    interface do estado e um setter público que permite sobrescrever o
    valor daquele campo.

5.  Vá até o método do contexto novamente e substitua as condicionais de
    estado vazias por chamadas aos métodos correspondentes do objeto
    estado.

6.  Para trocar o estado do contexto, crie uma instância de uma das
    classes estado e a passe para o contexto. Você pode fazer isso
    dentro do próprio contexto, ou em vários estados, ou no cliente.
    Aonde quer que isso seja feito, a classe se torna dependente da
    classe estado concreta que ela instanciou.
:::

:::::: {.section .pros-cons}
##  Prós e contras {#pros-cons}

::::: row
::: col-sm-6
-    *Princípio de responsabilidade única*. Organiza o código
    relacionado a estados particulares em classes separadas.
-    *Princípio aberto/fechado*. Introduz novos estados sem mudar
    classes de estado ou contexto existentes.
-    Simplifica o código de contexto ao eliminar condicionais de
    máquinas de estado pesadas.
:::

::: col-sm-6
-    Aplicar o padrão pode ser um exagero se a máquina de estado só tem
    alguns estados ou raramente muda eles.
:::
:::::
::::::

::: {.section .relations}
##  Relações com outros padrões {#relations}

-   O [Bridge](bridge.html), [State](state.html),
    [Strategy](strategy.html) (e de certa forma o
    [Adapter](adapter.html)) têm estruturas muito parecidas. De fato,
    todos esses padrões estão baseados em composição, o que é delegar o
    trabalho para outros objetos. Contudo, eles todos resolvem problemas
    diferentes. Um padrão não é apenas uma receita para estruturar seu
    código de uma maneira específica. Ele também pode comunicar a outros
    desenvolvedores o problema que o padrão resolve.

-   O [State](state.html) pode ser considerado como uma extensão do
    [Strategy](strategy.html). Ambos padrões são baseados em composição:
    eles mudam o comportamento do contexto ao delegar algum trabalho
    para objetos auxiliares. O *Strategy* faz esses objetos serem
    completamente independentes e alheios entre si. Contudo, o *State*
    não restringe dependências entre estados concretos, permitindo que
    eles alterem o estado do contexto à vontade.
:::

::: {.section .implementations}
##  Exemplos de código {#implementations}

[![State em
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](state/csharp/example.html "State em C#"){.prog-lang-link}
[![State em
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](state/cpp/example.html "State em C++"){.prog-lang-link}
[![State em
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](state/go/example.html "State em Go"){.prog-lang-link}
[![State em
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](state/java/example.html "State em Java"){.prog-lang-link}
[![State em
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](state/php/example.html "State em PHP"){.prog-lang-link}
[![State em
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](state/python/example.html "State em Python"){.prog-lang-link}
[![State em
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](state/ruby/example.html "State em Ruby"){.prog-lang-link}
[![State em
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](state/rust/example.html "State em Rust"){.prog-lang-link}
[![State em
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](state/swift/example.html "State em Swift"){.prog-lang-link}
[![State em
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](state/typescript/example.html "State em TypeScript"){.prog-lang-link}
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

[Strategy []{.fa .fa-arrow-right}](strategy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Observer](observer.html){.btn .btn-default
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
