::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
comportamentais](behavioral-patterns.html){.type}
:::

# Chain of Responsibility {#chain-of-responsibility .title}

::: aka
Também conhecido como: [CoR, ]{style="display:inline-block"}[Corrente de
responsabilidade, ]{style="display:inline-block"}[Corrente de
comando, ]{style="display:inline-block"}[Chain of
command]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

O **Chain of Responsibility** é um padrão de projeto comportamental que
permite que você passe pedidos por uma corrente de handlers. Ao receber
um pedido, cada handler decide se processa o pedido ou o passa adiante
para o próximo handler na corrente.

<figure class="image">
<img
src="../../images/patterns/content/chain-of-responsibility/chain-of-responsibilityf2d1.png?id=56c10d0dc712546cc283cfb3fb463458"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/chain-of-responsibility/chain-of-responsibility.png?id=56c10d0dc712546cc283cfb3fb463458 640w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-1.5x.png?id=770c03ad168fa797ec8ed4556ddf0b5c 960w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-2x.png?id=cc104b0a00a410f37fb39da80f392b88 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Padrão de projeto Chain of Responsibility" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagine que você está trabalhando em um sistema de encomendas online.
Você quer restringir o acesso ao sistema para que apenas usuários
autenticados possam criar pedidos. E também somente usuários que tem
permissões administrativas devem ter acesso total a todos os pedidos.

Após um pouco de planejamento, você se dá conta que essas checagens
devem ser feitas sequencialmente. A aplicação pode tentar autenticar um
usuário ao sistema sempre que receber um pedido que contém as
credenciais do usuário. Contudo, se essas credenciais não estão corretas
e a autenticação falha, não há razão para continuar com outras
checagens.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/problem1-pt-br5746.png?id=9ca2089dd17dfdff3a87d1d72bad48b6"
style="aspect-ratio: 2.5;"
srcset="/images/patterns/diagrams/chain-of-responsibility/problem1-pt-br.png?id=9ca2089dd17dfdff3a87d1d72bad48b6 600w,/images/patterns/diagrams/chain-of-responsibility/problem1-pt-br-1.5x.png?id=46da7f8019a2c276729c279dbd4d5c80 900w,/images/patterns/diagrams/chain-of-responsibility/problem1-pt-br-2x.png?id=8e0e104de313620e344aecfff8b2690c 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Problema, resolvido pelo Chain of Responsibility" />
<figcaption><p>O pedido deve passar por uma série de checagens antes do
sistema de encomendas possa lidar ele mesmo com
o pedido.</p></figcaption>
</figure>

Durante os próximos meses você implementou diversas mais daquelas
checagens sequenciais.

-   Um de seus colegas sugeriu que não é seguro passar dados brutos
    diretamente para o sistema de encomendas. Então você adicionou uma
    etapa adicional de validação para limpar os dados no pedido.

-   Mais tarde, alguém notou que o sistema é vulnerável à ataques de
    força bruta. Para evitar isso, você prontamente adicionou uma
    checagem que filtra repetidas falhas vindas do mesmo endereço de IP.

-   Outra pessoa sugeriu que você poderia agilizar o sistema se
    retornasse resultados de cache em pedidos repetidos contendo os
    mesmos dados. Portanto, você adicionou outra checagem que permite
    que o pedido passe através do sistema apenas se não há uma resposta
    adequada armazenada em cache.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/problem2-pt-brd265.png?id=8c6646322723a29f47c32bfda923ee57"
style="aspect-ratio: 1.65;"
srcset="/images/patterns/diagrams/chain-of-responsibility/problem2-pt-br.png?id=8c6646322723a29f47c32bfda923ee57 610w,/images/patterns/diagrams/chain-of-responsibility/problem2-pt-br-1.5x.png?id=412a6b717bacc22da5e44fd4ce209511 915w,/images/patterns/diagrams/chain-of-responsibility/problem2-pt-br-2x.png?id=3ac8a2a373d61ab264db27b8631e5bf1 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Com cada nova checagem o código fica maior, mais bagunçado, e mais feio" />
<figcaption><p>Quanto mais o código cresce, mais bagunçado
ele fica.</p></figcaption>
</figure>

O código das checagens, que já parecia uma bagunça, ficou mais e mais
inchado a medida que você foi adicionando novas funcionalidades. Mudar
uma checagem às vezes afetava outras. E o pior de tudo, quando você
tentou reutilizar as checagens para proteger os componentes do sistema,
você teve que duplicar parte do código uma vez que esses componentes
precisavam de algumas dessas checagens, mas nem todos eles.

O sistema ficou muito difícil de compreender e caro de se manter. Você
lutou com o código por um tempo, até que um dia você decidiu refatorar a
coisa toda.
:::

::: {.section .solution}
##  Solução {#solution}

Como muitos outros padrões de projeto comportamental, o **Chain of
Responsibility** se baseia em transformar certos comportamentos em
objetos solitários chamados *handlers*. No nosso caso, cada checagem
devem ser extraída para sua própria classe com um único método que faz a
checagem. O pedido, junto com seus dados, é passado para esse método
como um argumento.

O padrão sugere que você ligue esses handlers em uma corrente. Cada
handler ligado tem um campo para armazenar uma referência ao próximo
handler da corrente. Além de processar o pedido, handlers o passam
adiante na corrente. O pedido viaja através da corrente até que todos os
handlers tiveram uma chance de processá-lo.

E aqui está a melhor parte: um handler pode decidir não passar o pedido
adiante na corrente e efetivamente parar qualquer futuro processamento.

Em nosso exemplo com sistema de encomendas, um handler realiza o
processamento e então decide se passa o pedido adiante na corrente ou
não. Assumindo que o pedido contenha os dados adequados, todos os
handlers podem executar seu comportamento principal, seja ele uma
checagem de autenticação ou armazenamento em cache.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/solution1-pt-brb9f8.png?id=105269b800a40fdeb22fafd57f089bf6"
style="aspect-ratio: 4;"
srcset="/images/patterns/diagrams/chain-of-responsibility/solution1-pt-br.png?id=105269b800a40fdeb22fafd57f089bf6 640w,/images/patterns/diagrams/chain-of-responsibility/solution1-pt-br-1.5x.png?id=81ebb7a96774daca333f5f948f19c44e 960w,/images/patterns/diagrams/chain-of-responsibility/solution1-pt-br-2x.png?id=d9741e51476672bc9b7219911d25d072 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Handlers estão alinhados um a um, formando uma corrente" />
<figcaption><p>Handlers estão alinhados um a um, formando
uma corrente.</p></figcaption>
</figure>

Contudo, há uma abordagem ligeiramente diferente (e um tanto quanto
canônica) na qual, ao receber o pedido, um handler decide se ele pode
processá-lo ou não. Se ele pode, ele não passa o pedido adiante. Então é
um handler que processa o pedido ou mais ninguém. Essa abordagem é muito
comum quando lidando com eventos em pilha de elementos dentro de uma
interface gráfica de usuário.

Por exemplo, quando um usuário clica um botão, o evento se propaga
através da corrente de elementos GUI que começam com aquele botão,
prossegue para seus contêineres (como planilhas ou painéis), e termina
com a janela principal da aplicação. O evento é processado pelo primeiro
elemento na corrente que é capaz de lidar com ele. Esse exemplo também é
notável porque ele mostra que uma corrente pode sempre ser extraída de
um objeto árvore.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/solution2-pt-br41fa.png?id=0fd34a2b54ad9d0ecad0da41073f0070"
style="aspect-ratio: 1.73;"
srcset="/images/patterns/diagrams/chain-of-responsibility/solution2-pt-br.png?id=0fd34a2b54ad9d0ecad0da41073f0070 520w,/images/patterns/diagrams/chain-of-responsibility/solution2-pt-br-1.5x.png?id=485e264f422ed29fb341c1c618edcdc4 780w,/images/patterns/diagrams/chain-of-responsibility/solution2-pt-br-2x.png?id=62f622a24ffffa2995f159198f237abc 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Uma corrente pode ser formada por uma secção de um objeto árvore" />
<figcaption><p>Uma corrente pode ser formada por uma secção de
um objeto.</p></figcaption>
</figure>

É crucial que todas as classes handler implementem a mesma interface.
Cada handler concreto deve se importar apenas se o seguinte tem o método
`executar`. Dessa maneira você pode compor correntes durante a execução,
usando vários handlers sem acoplar seu código com suas classes
concretas.
:::

::: {.section .analogy}
##  Analogia com o mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-pt-brc45b.png?id=f0a5a7cdb160ee7245bbc71834d13965"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-pt-br.png?id=f0a5a7cdb160ee7245bbc71834d13965 600w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-pt-br-1.5x.png?id=d90b180b18bb94ab8fdb689bc8b147e5 900w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-pt-br-2x.png?id=50106afd2679093970bac8ba219a98c2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Falar com o suporte técnico pode ser difícil" />
<figcaption><p>Uma chamada para o suporte técnico pode atravessar
diversos operadores.</p></figcaption>
</figure>

Você acabou de comprar e instalar um novo hardware em seu computador.
Como você é um geek, o computador tem diversos sistemas operacionais
instalados. Você tenta ligar todos eles para ver se o hardware é
suportado. O Windows detecta e ativa o hardware automaticamente.
Contudo, seu amado Linux se recusa a trabalhar com o novo hardware. Com
uma pequena ponta de esperança, você decide ligar para o número do
suporte técnico escrito na caixa.

A primeira coisa que você ouve é uma voz robótica do outro lado. Ela
sugere nove soluções populares para vários problemas, nenhum dos quais é
relevante para seu caso. Após um tempo, a voz robótica conecta você com
um operador de carne e osso.

Infelizmente, o operador não foi capaz de sugerir algo específico
também. Ele continuava recitando longos protocolos do manual, se
recusando a escutar seus comentários. Após escutar a frase "você tentou
desligar e ligar o computador" pela décima vez, você exige ser conectado
a um engenheiro.

Eventualmente o operador passa sua chamada para um dos engenheiros, que
estava ansioso por contato humano já que estava sentado por horas em sua
escura sala do servidor no subsolo de algum prédio. O engenheiro lhe diz
onde baixar os drivers apropriados para seu novo hardware e como
instalá-los no Linux. Finalmente, a solução! Você termina sua chamada,
transbordando de alegria.
:::

::::: {.section .structure-container}
##  Estrutura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/structure03c9.png?id=848f0fc8dca57a44974d63f8181f5406"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.93;"
srcset="/images/patterns/diagrams/chain-of-responsibility/structure.png?id=848f0fc8dca57a44974d63f8181f5406 380w,/images/patterns/diagrams/chain-of-responsibility/structure-1.5x.png?id=49dfbae38f146af7319f80dce9ea7e2a 570w,/images/patterns/diagrams/chain-of-responsibility/structure-2x.png?id=bb837faaac88e7f2a16f751d0beaa201 760w"
sizes="(max-width: 720px) 100vw, 380px" loading="lazy" width="380"
alt="Estrutura do padrão de projeto da Chain of Responsibility" /><img
src="../../images/patterns/diagrams/chain-of-responsibility/structure-indexedfcb6.png?id=e13a5bf44f9ca47299223116af77cbef"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.88;"
srcset="/images/patterns/diagrams/chain-of-responsibility/structure-indexed.png?id=e13a5bf44f9ca47299223116af77cbef 380w,/images/patterns/diagrams/chain-of-responsibility/structure-indexed-1.5x.png?id=ca660e2cb697b512aadc07fdd3e109cd 570w,/images/patterns/diagrams/chain-of-responsibility/structure-indexed-2x.png?id=4f27e2c48e635f45a78472d707a8df3c 760w"
sizes="(max-width: 720px) 100vw, 380px" loading="lazy" width="380"
alt="Estrutura do padrão de projeto da Chain of Responsibility" />
</figure>
:::

1.  O **Handler** declara a interface, comum a todos os handlers
    concretos. Ele geralmente contém apenas um único método para lidar
    com pedidos, mas algumas vezes ele pode conter outro método para
    configurar o próximo handler da corrente.

2.  O **Handler Base** é uma classe opcional onde você pode colocar o
    código padrão que é comum a todas as classes handler.

    Geralmente, essa classe define um campo para armazenar uma
    referência para o próximo handler. Os clientes podem construir uma
    corrente passando um handler para o construtor ou setter do handler
    anterior. A classe pode também implementar o comportamento padrão do
    handler: pode passar a execução para o próximo handler após checar
    por sua existência.

3.  **Handlers Concretos** contém o código real para processar pedidos.
    Ao receber um pedido, cada handler deve decidir se processa ele e,
    adicionalmente, se passa ele adiante na corrente.

    Os handlers são geralmente auto contidos e imutáveis, aceitando
    todos os dados necessários apenas uma vez através do construtor.

4.  O **Cliente** pode compor correntes apenas uma vez ou compô-las
    dinamicamente, dependendo da lógica da aplicação. Note que um pedido
    pode ser enviado para qualquer handler na corrente---não precisa ser
    ao primeiro.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Neste exemplo, o padrão **Chain of Responsibility** é responsável por
mostrar informação de ajuda contextual para elementos de GUI ativos.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/example-pt-brf38c.png?id=ffc72247c173e7147ad525abc9b94621"
style="aspect-ratio: 1.09;"
srcset="/images/patterns/diagrams/chain-of-responsibility/example-pt-br.png?id=ffc72247c173e7147ad525abc9b94621 610w,/images/patterns/diagrams/chain-of-responsibility/example-pt-br-1.5x.png?id=1eca71d172ed0f185eafed38a253e02c 915w,/images/patterns/diagrams/chain-of-responsibility/example-pt-br-2x.png?id=dd147dabd4a531a6cab61f6bf0f92068 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Exemplo da estrutura do Chain of Responsibility" />
<figcaption><p>As classes de interface do usuário são construídas com o
padrão Composite. Cada elemento é ligado com seu elemento contêiner. A
qualquer momento, você pode construir uma corrente de elementos que
começa com o elemento em si e vai através de todos os elementos do
seu contêiner.</p></figcaption>
</figure>

O GUI da aplicação é geralmente estruturado como uma árvore de objetos.
Por exemplo, a classe `Dialog`, que renderiza a janela principal da
aplicação, seria a raíz do objeto árvore. O dialog contém `Painéis`, que
podem conter outros painéis ou simplesmente elementos de baixo nível
como `Botões` e `CamposDeTexto`.

Um componente simples pode mostrar descrições contextuais breves, desde
que o componente tenha um texto de ajuda assinalado. Mas componentes
mais complexos definem seu próprio modo de mostrar ajuda contextual,
tais como mostrar um pedaço do manual ou abrir uma página de navegador.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/example2-pt-brd5e9.png?id=f031893a4d70e0ba1b75d4203382b8b2"
style="aspect-ratio: 0.81;"
srcset="/images/patterns/diagrams/chain-of-responsibility/example2-pt-br.png?id=f031893a4d70e0ba1b75d4203382b8b2 250w,/images/patterns/diagrams/chain-of-responsibility/example2-pt-br-1.5x.png?id=8535833c74db8fcf0ba07a34aa8b7d94 375w,/images/patterns/diagrams/chain-of-responsibility/example2-pt-br-2x.png?id=5b15416a21448b3dd724190b08e32f62 500w"
sizes="(max-width: 720px) 100vw, 250px" loading="lazy" width="250"
alt="Exemplo da estrutura da Chain of Responsability" />
<figcaption><p>Assim é como um pedido de ajuda atravessa os
objetos GUI.</p></figcaption>
</figure>

Quando um usuário aponta o cursor do mouse para um elemento e aperta a
tecla `F1`, a aplicação detecta o componente abaixo do cursor e manda um
pedido de ajuda. O pedido atravessa todos os elementos do contêiner até
chegar no elemento capaz de mostrar a informação de ajuda.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// A interface do handler declara um método para executar um
// pedido.
interface ComponentWithContextualHelp is
    method showHelp()


// A classe base para componentes simples.
abstract class Component implements ComponentWithContextualHelp is
    field tooltipText: string

    // O contêiner do componente age como o próximo elo na
    // corrente de handlers.
    protected field container: Container

    // O componente mostra um tooltip (dica de contexto) se há
    // algum texto de ajuda assinalado a ele. Do contrário ele
    // passa a chamada adiante ao contêiner, se ele existir.
    method showHelp() is
        if (tooltipText != null)
            // Mostrar dica de contexto.
        else
            container.showHelp()


// Contêineres podem conter tanto componentes simples como
// outros contêineres como filhos. As relações da corrente são
// definidas aqui. A classe herda o comportamento showHelp de
// sua mãe.
abstract class Container extends Component is
    protected field children: array of Component

    method add(child) is
        children.add(child)
        child.container = this


// Componentes primitivos estão de bom tamanho com a
// implementação de ajuda padrão.
class Button extends Component is
    // ...

// Mas componentes complexos podem sobrescrever a implementação
// padrão. Se o texto de ajuda não pode ser fornecido de uma
// nova maneira, o componente pode sempre chamar a implementação
// base (veja a classe Component).
class Panel extends Container is
    field modalHelpText: string

    method showHelp() is
        if (modalHelpText != null)
            // Mostra uma janela modal com texto de ajuda.
        else
            super.showHelp()

// ...o mesmo que acima...
class Dialog extends Container is
    field wikiPageURL: string

    method showHelp() is
        if (wikiPageURL != null)
            // Abre a página de ajuda do wiki.
        else
            super.showHelp()


// Código cliente.
class Application is
    // Cada aplicação configura a corrente de forma diferente.
    method createUI() is
        dialog = new Dialog(&quot;Budget Reports&quot;)
        dialog.wikiPageURL = &quot;http://...&quot;
        panel = new Panel(0, 0, 400, 800)
        panel.modalHelpText = &quot;This panel does...&quot;
        ok = new Button(250, 760, 50, 20, &quot;OK&quot;)
        ok.tooltipText = &quot;This is an OK button that...&quot;
        cancel = new Button(320, 760, 50, 20, &quot;Cancel&quot;)
        // ...
        panel.add(ok)
        panel.add(cancel)
        dialog.add(panel)

    // Imagine o que acontece aqui.
    method onF1KeyPress() is
        component = this.getComponentAtMouseCoords()
        component.showHelp()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Aplicabilidade {#applicability}

::::::::: applicability
::: applicability-problem
Utilize o padrão Chain of Responsibility quando é esperado que seu
programa processe diferentes tipos de pedidos em várias maneiras, mas os
exatos tipos de pedidos e suas sequências são desconhecidos de antemão.
:::

::: applicability-solution
O padrão permite que você ligue vários handlers em uma corrente e, ao
receber um pedido, perguntar para cada handler se ele pode ou não
processá-lo. Dessa forma todos os handlers tem a chance de processar o
pedido.
:::

::: applicability-problem
Utilize o padrão quando é essencial executar diversos handlers em uma
ordem específica.
:::

::: applicability-solution
Já que você pode ligar os handlers em uma corrente em qualquer ordem,
todos os pedidos irão atravessar a corrente exatamente como você
planejou.
:::

::: applicability-problem
Utilize o padrão CoR quando o conjunto de handlers e suas encomendas
devem mudar no momento de execução.
:::

::: applicability-solution
Se você providenciar setters para um campo de referência dentro das
classes handler, você será capaz de inserir, remover, ou reordenar os
handlers de forma dinâmica.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Como implementar {#checklist}

1.  Declare a interface do handler e descreva a assinatura de um método
    para lidar com pedidos.

    Decida como o cliente irá passar os dados do pedido para o método. A
    maneira mais flexível é converter o pedido em um objeto e passá-lo
    para o método handler como um argumento.

2.  Para eliminar código padrão duplicado nos handlers concretos, pode
    valer a pena criar uma classe handler base abstrata, derivada da
    interface do handler.

    Essa classe deve ter um campo para armazenar uma referência ao
    próximo handler na corrente. Considere tornar a classe imutável.
    Contudo, se você planeja modificar correntes no tempo de execução,
    você precisa definir um setter para alterar o valor do campo de
    referência.

    Você também pode implementar o comportamento padrão conveniente para
    o método handler, que vai passar adiante o pedido para o próximo
    objeto a não ser que não haja mais objetos. Handlers concretos irão
    ser capazes de usar esse comportamento ao chamar o método pai.

3.  Um por um crie subclasses handler concretas e implemente seus
    métodos handler. Cada handler deve fazer duas decisões ao receber um
    pedido:

    -   Se ele vai processar o pedido.
    -   Se ele vai passar o pedido adiante na corrente.

4.  O cliente pode tanto montar correntes sozinho ou receber correntes
    pré construídas de outros objetos. Neste último caso, você deve
    implementar algumas classes fábrica para construir correntes de
    acordo com a configuração ou definições de ambiente.

5.  O cliente pode ativar qualquer handler da corrente, não apenas o
    primeiro. O pedido será passado ao longo da corrente até que algum
    handler se recuse a passá-lo adiante ou até ele chegar ao fim da
    corrente.

6.  Devido a natureza dinâmica da corrente, o cliente deve estar pronto
    para lidar com os seguintes cenários:

    -   A corrente pode consistir de um único elo.
    -   Alguns pedidos podem não chegar ao fim da corrente.
    -   Outros podem chegar ao fim da corrente sem terem sido tratados.
:::

:::::: {.section .pros-cons}
##  Prós e contras {#pros-cons}

::::: row
::: col-sm-6
-    Você pode controlar a ordem de tratamento dos pedidos.
-    *Princípio de responsabilidade única*. Você pode desacoplar classes
    que invocam operações de classes que realizam operações.
-    *Princípio aberto/fechado*. Você pode introduzir novos handlers na
    aplicação sem quebrar o código cliente existente.
:::

::: col-sm-6
-    Alguns pedidos podem acabar sem tratamento.
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

-   O [Chain of Responsibility](chain-of-responsibility.html) é
    frequentemente usado em conjunto com o [Composite](composite.html).
    Neste caso, quando um componente folha recebe um pedido, ele pode
    passá-lo através de uma corrente de todos os componentes pai até a
    raiz do objeto árvore.

-   Handlers em uma [Chain of
    Responsibility](chain-of-responsibility.html) podem ser
    implementados como [comandos](command.html). Neste caso, você pode
    executar várias operações diferentes sobre o mesmo objeto contexto,
    representado por um pedido.

    Contudo, há outra abordagem, onde o próprio pedido é um objeto
    *comando*. Neste caso, você pode executar a mesma operação em uma
    série de diferentes contextos ligados em uma corrente.

-   O [Chain of Responsibility](chain-of-responsibility.html) e o
    [Decorator](decorator.html) têm estruturas de classe muito
    parecidas. Ambos padrões dependem de composição recursiva para
    passar a execução através de uma série de objetos. Contudo, há
    algumas diferenças cruciais.

    Os handlers do *CoR* podem executar operações arbitrárias
    independentemente uma das outras. Eles também podem parar o pedido
    de ser passado adiante em qualquer ponto. Por outro lado, vários
    *decoradores* podem estender o comportamento do objeto enquanto
    mantém ele consistente com a interface base. Além disso, os
    decoradores não tem permissão para quebrar o fluxo do pedido.
:::

::: {.section .implementations}
##  Exemplos de código {#implementations}

[![Chain of Responsibility em
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/csharp/example.html "Chain of Responsibility em C#"){.prog-lang-link}
[![Chain of Responsibility em
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/cpp/example.html "Chain of Responsibility em C++"){.prog-lang-link}
[![Chain of Responsibility em
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/go/example.html "Chain of Responsibility em Go"){.prog-lang-link}
[![Chain of Responsibility em
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/java/example.html "Chain of Responsibility em Java"){.prog-lang-link}
[![Chain of Responsibility em
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/php/example.html "Chain of Responsibility em PHP"){.prog-lang-link}
[![Chain of Responsibility em
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/python/example.html "Chain of Responsibility em Python"){.prog-lang-link}
[![Chain of Responsibility em
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/ruby/example.html "Chain of Responsibility em Ruby"){.prog-lang-link}
[![Chain of Responsibility em
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/rust/example.html "Chain of Responsibility em Rust"){.prog-lang-link}
[![Chain of Responsibility em
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/swift/example.html "Chain of Responsibility em Swift"){.prog-lang-link}
[![Chain of Responsibility em
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/typescript/example.html "Chain of Responsibility em TypeScript"){.prog-lang-link}
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

[Command []{.fa .fa-arrow-right}](command.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Padrões
comportamentais](behavioral-patterns.html){.btn .btn-default rel="prev"}
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
