::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
comportamentais](behavioral-patterns.html){.type}
:::

# Observer {#observer .title}

::: aka
Também conhecido como:
[Observador, ]{style="display:inline-block"}[Assinante do
evento, ]{style="display:inline-block"}[Event-Subscriber, ]{style="display:inline-block"}[Escutador, ]{style="display:inline-block"}[Listener]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

O **Observer** é um padrão de projeto comportamental que permite que
você defina um mecanismo de assinatura para notificar múltiplos objetos
sobre quaisquer eventos que aconteçam com o objeto que eles
estão observando.

<figure class="image">
<img
src="../../images/patterns/content/observer/observerd508.png?id=6088e31e1b0d4a417506a66614dcf065"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/observer/observer.png?id=6088e31e1b0d4a417506a66614dcf065 640w,/images/patterns/content/observer/observer-1.5x.png?id=aa64f9f336354462b57bbff5c09d8269 960w,/images/patterns/content/observer/observer-2x.png?id=d5a83e115528e9fd633f04ad2650f1db 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Padrão de projeto Observer" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagine que você tem dois tipos de objetos: um `Cliente` e uma `Loja`. O
cliente está muito interessado em uma marca particular de um produto
(digamos que seja um novo modelo de iPhone) que logo deverá estar
disponível na loja.

O cliente pode visitar a loja todos os dias e checar a disponibilidade
do produto. Mas enquanto o produto ainda está a caminho, a maioria
desses visitas serão em vão.

<figure class="image">
<img
src="../../images/patterns/content/observer/observer-comic-1-pt-br57f8.png?id=adfe141b54d9d26143d611158896597b"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/observer/observer-comic-1-pt-br.png?id=adfe141b54d9d26143d611158896597b 600w,/images/patterns/content/observer/observer-comic-1-pt-br-1.5x.png?id=c19c75332d10ba84a2b5c06aca514ae8 900w,/images/patterns/content/observer/observer-comic-1-pt-br-2x.png?id=3983224519de1ef00ebc73faba25db95 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Visitando a loja vs. enviando spam" />
<figcaption><p>Visitando a loja vs. enviando spam</p></figcaption>
</figure>

Por outro lado, a loja poderia mandar milhares de emails (que poderiam
ser considerados como spam) para todos os clientes cada vez que um novo
produto se torna disponível. Isso salvaria alguns clientes de
incontáveis viagens até a loja. Porém, ao mesmo tempo, irritaria outros
clientes que não estão interessados em novos produtos.

Parece que temos um conflito. Ou o cliente gasta tempo verificando a
disponibilidade do produto ou a loja gasta recursos notificando os
clientes errados.
:::

::: {.section .solution}
##  Solução {#solution}

O objeto que tem um estado interessante é quase sempre chamado de
*sujeito*, mas já que ele também vai notificar outros objetos sobre as
mudanças em seu estado, nós vamos chamá-lo de *publicador*. Todos os
outros objetos que querem saber das mudanças do estado do publicador são
chamados de *assinantes*.

O padrão Observer sugere que você adicione um mecanismo de assinatura
para a classe publicadora para que objetos individuais possam assinar ou
desassinar uma corrente de eventos vindo daquela publicadora. Nada tema!
Nada é complicado como parece. Na verdade, esse mecanismo consiste em 1)
um vetor para armazenar uma lista de referências aos objetos do
assinante e 2) alguns métodos públicos que permitem adicionar assinantes
e removê-los da lista.

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/solution1-pt-br8ccb.png?id=f51e1b724f0de948e6f9e2c3f4d029bc"
style="aspect-ratio: 2.61;"
srcset="/images/patterns/diagrams/observer/solution1-pt-br.png?id=f51e1b724f0de948e6f9e2c3f4d029bc 470w,/images/patterns/diagrams/observer/solution1-pt-br-1.5x.png?id=d8f5fcbf134ad4bf4e009181597335b3 705w,/images/patterns/diagrams/observer/solution1-pt-br-2x.png?id=b295a3804e5409b67a864070809adeca 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Mecanismo de assinatura" />
<figcaption><p>Um mecanismo de assinatura permite que objetos
individuais inscrevam-se a notificações de eventos.</p></figcaption>
</figure>

Agora, sempre que um evento importante acontece com a publicadora, ele
passa para seus assinantes e chama um método específico de notificação
em seus objetos.

Aplicações reais podem ter dúzias de diferentes classes assinantes que
estão interessadas em acompanhar eventos da mesma classe publicadora.
Você não iria querer acoplar a publicadora a todas essas classes. Além
disso, você pode nem estar ciente de algumas delas de antemão se a sua
classe publicadora deve ser usada por outras pessoas.

É por isso que é crucial que todos os assinantes implementem a mesma
interface e que a publicadora comunique-se com eles apenas através
daquela interface. Essa interface deve declarar o método de notificação
junto com um conjunto de parâmetros que a publicadora pode usar para
passar alguns dados contextuais junto com a notificação.

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/solution2-pt-br5f57.png?id=ffe67f7ad9191820105ff92a87859e5f"
style="aspect-ratio: 1.24;"
srcset="/images/patterns/diagrams/observer/solution2-pt-br.png?id=ffe67f7ad9191820105ff92a87859e5f 460w,/images/patterns/diagrams/observer/solution2-pt-br-1.5x.png?id=fdd701098cf08be414857efb442f6af4 690w,/images/patterns/diagrams/observer/solution2-pt-br-2x.png?id=ed8d8cc070892b4fe1fddf90bdc2596b 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Métodos de notificação" />
<figcaption><p>A publicadora notifica os assinantes chamando um método
específico de notificação em seus objetos.</p></figcaption>
</figure>

Se a sua aplicação tem diferentes tipos de publicadoras e você quer
garantir que seus assinantes são compatíveis com todas elas, você pode
ir além e fazer todas as publicadoras seguirem a mesma interface. Essa
interface precisa apenas descrever alguns métodos de inscrição. A
interface permitirá assinantes observar o estado das publicadoras sem se
acoplar a suas classes concretas.
:::

::: {.section .analogy}
##  Analogia com o mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/observer/observer-comic-2-pt-bref34.png?id=8714f51c7a460bf1744ab82edd02ed53"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/observer/observer-comic-2-pt-br.png?id=8714f51c7a460bf1744ab82edd02ed53 600w,/images/patterns/content/observer/observer-comic-2-pt-br-1.5x.png?id=e67e32e9725afb71f4d34f6b2e3ad36f 900w,/images/patterns/content/observer/observer-comic-2-pt-br-2x.png?id=efd856b55ea05267cde7e383729f0ee2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Assinaturas de revistas e jornais" />
<figcaption><p>Assinaturas de revistas e jornais.</p></figcaption>
</figure>

Se você assinar um jornal ou uma revista, você não vai mais precisar ir
até a banca e ver se a próxima edição está disponível. Ao invés disso a
publicadora manda novas edições diretamente para sua caixa de correio
após a publicação ou até mesmo com antecedência.

A publicadora mantém uma lista de assinantes e sabe em quais revistas
eles estão interessados. Os assinantes podem deixar essa lista a
qualquer momento quando desejarem que a publicadora pare de enviar novas
revistas para eles.
:::

::::: {.section .structure-container}
##  Estrutura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/observer/structurea121.png?id=365b7e2b8fbecc8948f34b9f8f16f33c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.97;"
srcset="/images/patterns/diagrams/observer/structure.png?id=365b7e2b8fbecc8948f34b9f8f16f33c 610w,/images/patterns/diagrams/observer/structure-1.5x.png?id=7f23a48c9a2d789844f912d3a0f289e6 915w,/images/patterns/diagrams/observer/structure-2x.png?id=228af9bded4d6ee6daf43a0e23cca9ff 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Estrutura do padrão de projeto Observer" /><img
src="../../images/patterns/diagrams/observer/structure-indexed8afd.png?id=2ca2c123503ede860740af2a22bc4b4d"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.88;"
srcset="/images/patterns/diagrams/observer/structure-indexed.png?id=2ca2c123503ede860740af2a22bc4b4d 620w,/images/patterns/diagrams/observer/structure-indexed-1.5x.png?id=20b9e7765e83ca75514c202968d9e9fa 930w,/images/patterns/diagrams/observer/structure-indexed-2x.png?id=910eec855bc41f05199e510494078926 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="Estrutura do padrão de projeto Observer" />
</figure>
:::

1.  A **Publicadora** manda eventos de interesse para outros objetos.
    Esses eventos ocorrem quando a publicadora muda seu estado ou
    executa algum comportamento. As publicadoras contêm uma
    infraestrutura de inscrição que permite novos assinantes se juntar
    aos atuais assinantes ou deixar a lista.

2.  Quando um novo evento acontece, a publicadora percorre a lista de
    assinantes e chama o método de notificação declarado na interface do
    assinante em cada objeto assinante.

3.  A interface do **Assinante** declara a interface de notificação. Na
    maioria dos casos ela consiste de um único método `atualizar`. O
    método pode ter vários parâmetros que permite que a publicadora
    passe alguns detalhes do evento junto com a atualização.

4.  **Assinantes Concretos** realizam algumas ações em resposta às
    notificações enviadas pela publicadora. Todas essas classes devem
    implementar a mesma interface para que a publicadora não fique
    acoplada à classes concretas.

5.  Geralmente, assinantes precisam de alguma informação contextual para
    lidar com a atualização corretamente. Por esse motivo, as
    publicadoras quase sempre passam algum dado de contexto como
    argumentos do método de notificação. A publicadora pode passar a si
    mesmo como um argumento, permitindo que o assinante recupere
    quaisquer dados diretamente.

6.  O **Cliente** cria a publicadora e os objetos assinantes
    separadamente e então registra os assinantes para as atualizações da
    publicadora.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Neste exemplo o padrão **Observer** permite que um objeto editor de
texto notifique outros objetos de serviço sobre mudanças em seu estado.

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/example04a2.png?id=6d0603ab5a00e4463b81d9639cd746a2"
style="aspect-ratio: 1.19;"
srcset="/images/patterns/diagrams/observer/example.png?id=6d0603ab5a00e4463b81d9639cd746a2 510w,/images/patterns/diagrams/observer/example-1.5x.png?id=67adccd1030a38dd263bfa09867f8dbe 765w,/images/patterns/diagrams/observer/example-2x.png?id=e2838e1562325e485fc7c2828a8ca445 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Exemplo da estrutura do padrão Observer" />
<figcaption><p>Notificando objetos sobre eventos que aconteceram com
outros objetos.</p></figcaption>
</figure>

A lista de assinantes é compilada dinamicamente: objetos podem começar
ou parar de ouvir às notificações durante a execução do programa,
dependendo do comportamento desejado pela sua aplicação.

Nesta implementação, a classe do editor não mantém a lista de assinatura
por si mesmo. Ele delega este trabalho para um objeto ajudante especial
devotado a fazer apenas isso. Você pode melhorar aquele objeto para
servir como um enviador de eventos centralizado, permitindo que qualquer
objeto aja como uma publicadora.

Adicionar novos assinantes ao programa não exige mudança nas classes
publicadoras existentes, desde que elas trabalhem com todos os
assinantes através da mesma interface.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// A classe publicadora base inclui o código de gerenciamento de
// inscrições e os métodos de notificação.
class EventManager is
    private field listeners: hash map of event types and listeners

    method subscribe(eventType, listener) is
        listeners.add(eventType, listener)

    method unsubscribe(eventType, listener) is
        listeners.remove(eventType, listener)

    method notify(eventType, data) is
        foreach (listener in listeners.of(eventType)) do
            listener.update(data)

// O publicador concreto contém a verdadeira lógica de negócio
// que é de interesse para alguns assinantes. Nós podemos
// derivar essa classe a partir do publicador base, mas isso nem
// sempre é possível na vida real devido a possibilidade do
// publicador concreto já ser uma subclasse. Neste caso, você
// pode remendar a lógica de inscrição com a composição, como
// fizemos aqui.
class Editor is
    public field events: EventManager
    private field file: File

    constructor Editor() is
        events = new EventManager()

    // Métodos da lógica de negócio podem notificar assinantes
    // acerca de mudanças.
    method openFile(path) is
        this.file = new File(path)
        events.notify(&quot;open&quot;, file.name)

    method saveFile() is
        file.write()
        events.notify(&quot;save&quot;, file.name)

    // ...


// Aqui é a interface do assinante. Se sua linguagem de
// programação suporta tipos funcionais, você pode substituir
// toda a hierarquia do assinante por um conjunto de funções.
interface EventListener is
    method update(filename)

// Assinantes concretos reagem a atualizações emitidas pelo
// publicador a qual elas estão conectadas.
class LoggingListener implements EventListener is
    private field log: File
    private field message: string

    constructor LoggingListener(log_filename, message) is
        this.log = new File(log_filename)
        this.message = message

    method update(filename) is
        log.write(replace(&#39;%s&#39;,filename,message))

class EmailAlertsListener implements EventListener is
    private field email: string
    private field message: string

    constructor EmailAlertsListener(email, message) is
        this.email = email
        this.message = message

    method update(filename) is
        system.email(email, replace(&#39;%s&#39;,filename,message))


// Uma aplicação pode configurar publicadores e assinantes
// durante o tempo de execução.
class Application is
    method config() is
        editor = new Editor()

        logger = new LoggingListener(
            &quot;/path/to/log.txt&quot;,
            &quot;Someone has opened the file: %s&quot;)
        editor.events.subscribe(&quot;open&quot;, logger)

        emailAlerts = new EmailAlertsListener(
            &quot;admin@example.com&quot;,
            &quot;Someone has changed the file: %s&quot;)
        editor.events.subscribe(&quot;save&quot;, emailAlerts)</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Aplicabilidade {#applicability}

::::::: applicability
::: applicability-problem
Utilize o padrão Observer quando mudanças no estado de um objeto podem
precisar mudar outros objetos, e o atual conjunto de objetos é
desconhecido de antemão ou muda dinamicamente.
:::

::: applicability-solution
Você pode vivenciar esse problema quando trabalhando com classes de
interface gráfica do usuário. Por exemplo, você criou classes de botões
customizados, e você quer deixar os clientes colocar algum código
customizado para seus botões para que ele ative sempre que usuário
aperta um botão.

O padrão Observer permite que qualquer objeto que implemente a interface
do assinante possa se inscrever para notificações de eventos em objetos
da publicadora. Você pode adicionar o mecanismo de inscrição em seus
botões, permitindo que o cliente coloque seu próprio código através de
classes assinantes customizadas.
:::

::: applicability-problem
Utilize o padrão quando alguns objetos em sua aplicação devem observar
outros, mas apenas por um tempo limitado ou em casos específicos.
:::

::: applicability-solution
A lista de inscrição é dinâmica, então assinantes podem entrar e sair da
lista sempre que quiserem.
:::
:::::::
::::::::

::: {.section .checklist}
##  Como implementar {#checklist}

1.  Olhe para sua lógica do negócio e tente quebrá-la em duas partes: a
    funcionalidade principal, independente de outros códigos, irá agir
    como publicadora; o resto será transformado em um conjunto de
    classes assinantes.

2.  Declare a interface do assinante. No mínimo, ela deve declarar um
    único método `atualizar`.

3.  Declare a interface da publicadora e descreva um par de métodos para
    adicionar um objeto assinante e removê-lo da lista. Lembre-se que
    publicadoras somente devem trabalhar com assinantes através da
    interface do assinante.

4.  Decida onde colocar a lista atual de assinantes e a implementação
    dos métodos de inscrição. Geralmente este código se parece o mesmo
    para todos os tipos de publicadoras, então o lugar óbvio para
    colocá-lo é dentro de uma classe abstrata derivada diretamente da
    interface da publicadora. Publicadoras concretas estendem aquela
    classe, herdando o comportamento de inscrição.

    Contudo, se você está aplicando o padrão para uma hierarquia de
    classe já existente, considere uma abordagem baseada na composição:
    coloque a lógica da inscrição dentro de um objeto separado, e faça
    todos as publicadoras reais usá-la.

5.  Crie as classes publicadoras concretas. A cada vez que algo
    importante acontece dentro de uma publicadora, ela deve notificar
    seus assinantes.

6.  Implemente os métodos de notificação de atualização nas classes
    assinantes concretas. A maioria dos assinantes precisarão de dados
    contextuais sobre o evento. Eles podem ser passados como argumentos
    do método de notificação.

    Mas há outra opção. Ao receber uma notificação, o assinante pode
    recuperar os dados diretamente da notificação. Neste caso, a
    publicadora deve passar a si mesma através do método de atualização.
    A opção menos flexível é ligar uma publicadora ao assinante
    permanentemente através do construtor.

7.  O cliente deve criar todas os assinantes necessários e registrá-los
    com suas publicadoras apropriadas.
:::

:::::: {.section .pros-cons}
##  Prós e contras {#pros-cons}

::::: row
::: col-sm-6
-    *Princípio aberto/fechado*. Você pode introduzir novas classes
    assinantes sem ter que mudar o código da publicadora (e vice versa
    se existe uma interface publicadora).
-    Você pode estabelecer relações entre objetos durante a execução.
:::

::: col-sm-6
-    Assinantes são notificados em ordem aleatória
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

[![Observer em
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](observer/csharp/example.html "Observer em C#"){.prog-lang-link}
[![Observer em
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](observer/cpp/example.html "Observer em C++"){.prog-lang-link}
[![Observer em
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](observer/go/example.html "Observer em Go"){.prog-lang-link}
[![Observer em
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](observer/java/example.html "Observer em Java"){.prog-lang-link}
[![Observer em
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](observer/php/example.html "Observer em PHP"){.prog-lang-link}
[![Observer em
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](observer/python/example.html "Observer em Python"){.prog-lang-link}
[![Observer em
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](observer/ruby/example.html "Observer em Ruby"){.prog-lang-link}
[![Observer em
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](observer/rust/example.html "Observer em Rust"){.prog-lang-link}
[![Observer em
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](observer/swift/example.html "Observer em Swift"){.prog-lang-link}
[![Observer em
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](observer/typescript/example.html "Observer em TypeScript"){.prog-lang-link}
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

[State []{.fa .fa-arrow-right}](state.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Memento](memento.html){.btn .btn-default
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
