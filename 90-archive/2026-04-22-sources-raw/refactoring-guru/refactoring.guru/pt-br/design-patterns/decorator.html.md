::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
estruturais](structural-patterns.html){.type}
:::

# Decorator {#decorator .title}

::: aka
Também conhecido como:
[Decorador, ]{style="display:inline-block"}[Envoltório, ]{style="display:inline-block"}[Wrapper]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

O **Decorator** é um padrão de projeto estrutural que permite que você
acople novos comportamentos para objetos ao colocá-los dentro de
invólucros de objetos que contém os comportamentos.

<figure class="image">
<img
src="../../images/patterns/content/decorator/decoratore8f5.png?id=710c66670c7123e0928d3b3758aea79e"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/decorator/decorator.png?id=710c66670c7123e0928d3b3758aea79e 640w,/images/patterns/content/decorator/decorator-1.5x.png?id=72aafc603a289fc64e028e83e8aa843b 960w,/images/patterns/content/decorator/decorator-2x.png?id=736ab07b1d8920ab2c7a70c9cb1305cc 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Padrão de projeto Decorator" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagine que você está trabalhando em um biblioteca de notificação que
permite que outros programas notifiquem seus usuários sobre eventos
importantes.

A versão inicial da biblioteca foi baseada na classe `Notificador` que
tinha apenas alguns poucos campos, um construtor, e um único método
`enviar`. O método podia aceitar um argumento de mensagem de um cliente
e enviar a mensagem para uma lista de emails que eram passadas para o
notificador através de seu construtor. Uma aplicação de terceiros que
agia como cliente deveria criar e configurar o objeto notificador uma
vez, e então usá-lo a cada vez que algo importante acontecesse.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem1-pt-brd355.png?id=56c7f877dbf344107a9162b81b7dca3a"
style="aspect-ratio: 2.45;"
srcset="/images/patterns/diagrams/decorator/problem1-pt-br.png?id=56c7f877dbf344107a9162b81b7dca3a 540w,/images/patterns/diagrams/decorator/problem1-pt-br-1.5x.png?id=a9d50a7299551706ff903993f34e1090 810w,/images/patterns/diagrams/decorator/problem1-pt-br-2x.png?id=f3f8ce46f9ae80241cb41ace3340b51c 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Estrutura da biblioteca antes de aplicar o padrão Decorator" />
<figcaption><p>Um programa poderia usar a classe notificador para enviar
notificações sobre eventos importantes para um conjunto predefinido
de emails.</p></figcaption>
</figure>

Em algum momento você se dá conta que os usuários da biblioteca esperam
mais que apenas notificações por email. Muitos deles gostariam de
receber um SMS acerca de problemas críticos. Outros gostariam de ser
notificados no Facebook, e, é claro, os usuários corporativos adorariam
receber notificações do Slack.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem2a2f8.png?id=ba5d5e106ea8c4848d60e230feca9135"
style="aspect-ratio: 2.59;"
srcset="/images/patterns/diagrams/decorator/problem2.png?id=ba5d5e106ea8c4848d60e230feca9135 440w,/images/patterns/diagrams/decorator/problem2-1.5x.png?id=4f15b6208533188dd09e7da7dd6b509a 660w,/images/patterns/diagrams/decorator/problem2-2x.png?id=28b2c8509b4e78c031d728424b876ebc 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="Estrutura da biblioteca após implementar outros tipos de notificação" />
<figcaption><p>Cada tipo de notificação é implementada em uma subclasse
do notificador.</p></figcaption>
</figure>

Quão difícil isso seria? Você estende a classe `Notificador` e coloca os
métodos de notificação adicionais nas novas subclasses. Agora o cliente
deve ser instanciado à classe de notificação que deseja e usar ela para
todas as futura notificações.

Mas então alguém, com razão, pergunta a você, "Por que você não usa
diversos tipos de notificação de uma só vez? Se a sua casa pegar fogo,
você provavelmente vai querer ser notificado por todos os canais."

Você tenta resolver esse problema criando subclasses especiais que
combinam diversos tipos de métodos de notificação dentro de uma classe.
Contudo, rapidamente você nota que isso irá inflar o código imensamente,
e não só da biblioteca, o código cliente também.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem34033.png?id=f3b3e7a107d870871f2c3167adcb7ccb"
style="aspect-ratio: 1.85;"
srcset="/images/patterns/diagrams/decorator/problem3.png?id=f3b3e7a107d870871f2c3167adcb7ccb 630w,/images/patterns/diagrams/decorator/problem3-1.5x.png?id=f4ef9d367df838c77121e9f84260b09c 945w,/images/patterns/diagrams/decorator/problem3-2x.png?id=abb7a87b521ce97d7661dd9c0b988cc3 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="Estrutura da biblioteca após criar combinações de classes" />
<figcaption><p>Combinação explosiva de subclasses.</p></figcaption>
</figure>

Você precisa encontrar outra maneira de estruturar classes de
notificação para que o número delas não quebre um recorde do Guinness
acidentalmente.
:::

::: {.section .solution}
##  Solução {#solution}

Estender uma classe é a primeira coisa que vem à mente quando você
precisa alterar o comportamento de um objeto. Contudo, a herança vem com
algumas ressalvas sérias que você precisa estar ciente.

-   A herança é estática. Você não pode alterar o comportamento de um
    objeto existente durante o tempo de execução. Você só pode
    substituir todo o objeto por outro que foi criado de uma subclasse
    diferente.
-   As subclasses só podem ter uma classe pai. Na maioria das
    linguagens, a herança não permite que uma classe herde
    comportamentos de múltiplas classes ao mesmo tempo.

Uma das maneiras de superar essas ressalvas é usando *Agregação* ou
*Composição* []{.pop-i}[*Agregação*: objeto A contém objetos B; B pode
viver sem A.\
*Composição*: objeto A consiste de objetos B; A gerencia o ciclo de vida
de B; B não pode viver sem A.]{.pop-content} ao invés de *Herança*.
Ambas alternativas funcionam quase da mesma maneira: um objeto *tem uma*
referência com outro e delega alguma funcionalidade, enquanto que na
herança, o próprio objeto *é* capaz de fazer a função, herdando o
comportamento da sua superclasse.

Com essa nova abordagem você pode facilmente substituir o objeto
"auxiliador" por outros, mudando o comportamento do contêiner durante o
tempo de execução. Um objeto pode usar o comportamento de várias
classes, ter referências a múltiplos objetos, e delegar qualquer tipo de
trabalho a eles. A agregação/composição é o princípio chave por trás de
muitos padrões de projeto, incluindo o Decorator. Falando nisso, vamos
voltar à discussão desse padrão.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution1-pt-brfe50.png?id=2678803c5fbd7265a1f993d1c514d250"
style="aspect-ratio: 3.44;"
srcset="/images/patterns/diagrams/decorator/solution1-pt-br.png?id=2678803c5fbd7265a1f993d1c514d250 550w,/images/patterns/diagrams/decorator/solution1-pt-br-1.5x.png?id=c651fed6cbb914634fcf46c3b5e087a9 825w,/images/patterns/diagrams/decorator/solution1-pt-br-2x.png?id=82b00a77a649c8ad5ff2e3867d45771a 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Herança vs. Agregação" />
<figcaption><p>Herança vs. Agregação</p></figcaption>
</figure>

"Envoltório" (ing. "wrapper") é o apelido alternativo para o padrão
Decorator que expressa claramente a ideia principal dele. Um
*envoltório* é um objeto que pode ser ligado com outro objeto *alvo*. O
envoltório contém o mesmo conjunto de métodos que o alvo e delega a ele
todos os pedidos que recebe. Contudo, o envoltório pode alterar o
resultado fazendo alguma coisa ou antes ou depois de passar o pedido
para o alvo.

Quando um simples envoltório se torna um verdadeiro decorador? Como
mencionei, o envoltório implementa a mesma interface que o objeto
envolvido. É por isso que da perspectiva do cliente esses objetos são
idênticos. Faça o campo de referência do envoltório aceitar qualquer
objeto que segue aquela interface. Isso lhe permitirá cobrir um objeto
em múltiplos envoltórios, adicionando o comportamento combinado de todos
os envoltórios a ele.

No nosso exemplo de notificações vamos deixar o simples comportamento de
notificação por email dentro da classe `Notificador` base, mas
transformar todos os métodos de notificação em decoradores.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution2e4ab.png?id=cbee4a27080ce3a0bf773482613e1347"
style="aspect-ratio: 1.39;"
srcset="/images/patterns/diagrams/decorator/solution2.png?id=cbee4a27080ce3a0bf773482613e1347 640w,/images/patterns/diagrams/decorator/solution2-1.5x.png?id=d2fa0c0b9bf5cba0e38be7aff7e7b7a8 960w,/images/patterns/diagrams/decorator/solution2-2x.png?id=7775f76b94dbd5cd25f711ce81f59262 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="A solução com o padrão Decorator" />
<figcaption><p>Vários métodos de notificação se
tornam decoradores.</p></figcaption>
</figure>

O código cliente vai precisar envolver um objeto notificador básico em
um conjunto de decoradores que coincidem com as preferências do cliente.
Os objetos resultantes serão estruturados como uma pilha.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution3-pt-br3871.png?id=70a468aff2bd17b4d2d0ad3ce5acc484"
style="aspect-ratio: 1.03;"
srcset="/images/patterns/diagrams/decorator/solution3-pt-br.png?id=70a468aff2bd17b4d2d0ad3ce5acc484 300w,/images/patterns/diagrams/decorator/solution3-pt-br-1.5x.png?id=4cf0214e09551a2bb2c73e3b4cb415e3 450w,/images/patterns/diagrams/decorator/solution3-pt-br-2x.png?id=4b4f3cb97b49a58cf62a3f8b7d9ca39c 600w"
sizes="(max-width: 720px) 100vw, 300px" loading="lazy" width="300"
alt="As aplicações pode configurar pilhas complexas de notificações decorators" />
<figcaption><p>As aplicações pode configurar pilhas complexas de
notificações decoradores</p></figcaption>
</figure>

O último decorador na pilha seria o objeto que o cliente realmente
trabalha. Como todos os decoradores implementam a mesma interface que o
notificador base, o resto do código cliente não quer saber se ele
funciona com o objeto "puro" do notificador ou do decorador.

Podemos utilizar a mesma abordagem para vários comportamentos tais como
formatação de mensagens ou compor uma lista de recipientes. O cliente
pode decorar o objeto com quaisquer decoradores customizados, desde que
sigam a mesma interface que os demais.
:::

::: {.section .analogy}
##  Analogia com o mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/decorator/decorator-comic-1cbff.png?id=80d95baacbfb91f5bcdbdc7814b0c64d"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/decorator/decorator-comic-1.png?id=80d95baacbfb91f5bcdbdc7814b0c64d 600w,/images/patterns/content/decorator/decorator-comic-1-1.5x.png?id=490ef9754d7554c2046b69f6f213c6da 900w,/images/patterns/content/decorator/decorator-comic-1-2x.png?id=ba869f621b6e0ea173fdc2b535fc7eed 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Exemplo do padrão Decorator" />
<figcaption><p>Você tem um efeito combinado de usar múltiplas peças
de roupa.</p></figcaption>
</figure>

Vestir roupas é um exemplo de usar decoradores. Quando você está com
frio, você se envolve com um suéter. Se você ainda sente frio com um
suéter, você pode vestir um casaco por cima. Se está chovendo, você pode
colocar uma capa de chuva. Todas essas vestimentas "estendem" seu
comportamento básico mas não são parte de você, e você pode facilmente
remover uma peça de roupa sempre que não precisar mais dela.
:::

::::: {.section .structure-container}
##  Estrutura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/structure128f.png?id=8c95d894aecce5315cc1b12093a7ea0c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.92;"
srcset="/images/patterns/diagrams/decorator/structure.png?id=8c95d894aecce5315cc1b12093a7ea0c 480w,/images/patterns/diagrams/decorator/structure-1.5x.png?id=4b2cd91d4483d9e3bba725f0e45d281d 720w,/images/patterns/diagrams/decorator/structure-2x.png?id=3cfa1f10417a4ef0c12580bc4a63b80d 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Estrutura do padrão de projeto Decorator" /><img
src="../../images/patterns/diagrams/decorator/structure-indexeda93c.png?id=09401b230a58f2249e4c9a1195d485a0"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.98;"
srcset="/images/patterns/diagrams/decorator/structure-indexed.png?id=09401b230a58f2249e4c9a1195d485a0 500w,/images/patterns/diagrams/decorator/structure-indexed-1.5x.png?id=dccc54182965078ccb4cfdeee41acbe5 750w,/images/patterns/diagrams/decorator/structure-indexed-2x.png?id=2733e7d0e322bfb2f150ccf8a878dbf6 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="Estrutura do padrão de projeto Decorator" />
</figure>
:::

1.  O **Componente** declara a interface comum tanto para os envoltórios
    como para os objetos envolvidos.

2.  O **Componente Concreto** é uma classe de objetos sendo envolvidos.
    Ela define o comportamento básico, que pode ser alterado por
    decoradores.

3.  A classe **Decorador Base** tem um campo para referenciar um objeto
    envolvido. O tipo do campo deve ser declarado assim como a interface
    do componente para que possa conter ambos os componentes concretos e
    os decoradores. O decorador base delega todas as operações para o
    objeto envolvido.

4.  Os **Decoradores Concretos** definem os comportamentos adicionais
    que podem ser adicionados aos componentes dinamicamente. Os
    decoradores concretos sobrescrevem métodos do decorador base e
    executam seus comportamentos tanto antes como depois de chamarem o
    método pai.

5.  O **Cliente** pode envolver componentes em múltiplas camadas de
    decoradors, desde que trabalhe com todos os objetos através da
    interface do componente.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Neste exemplo, o padrão **Decorator** lhe permite comprimir e encriptar
dados sensíveis independentemente do código que verdadeiramente usa
esses dados.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/example42cb.png?id=eec9dc488f00c85f50e764323baa723e"
style="aspect-ratio: 0.98;"
srcset="/images/patterns/diagrams/decorator/example.png?id=eec9dc488f00c85f50e764323baa723e 540w,/images/patterns/diagrams/decorator/example-1.5x.png?id=40ccaba4f5a8e6a2ebac697f04b9f10a 810w,/images/patterns/diagrams/decorator/example-2x.png?id=4891323a27d5601a174eec366187d833 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Exemplo de estrutura do padrão Decorator" />
<figcaption><p>Exemplo da encriptação e compressão
com decoradores.</p></figcaption>
</figure>

A aplicação envolve o objeto da fonte de dados com um par de
decoradores. Ambos invólucros mudam a maneira que os dados são escritos
e lidos no disco:

-   Antes dos dados serem **escritos no disco**, os decoradores
    encriptam e comprimem eles. A classe original escreve os dados
    protegidos e encriptados para o arquivo sem saber da mudança.

-   Logo antes dos dados serem **lidos do disco**, ele passa pelos
    mesmos decoradores que descomprimem e decodificam eles.

Os decoradores e a classe da fonte de dados implementam a mesma
interface, que os torna intercomunicáveis dentro do código cliente.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// A interface componente define operações que podem ser
// alteradas por decoradores.
interface DataSource is
    method writeData(data)
    method readData():data

// Componentes concretos fornecem uma implementação padrão para
// as operações. Pode haver diversas variações dessas classes em
// um programa.
class FileDataSource implements DataSource is
    constructor FileDataSource(filename) { ... }

    method writeData(data) is
        // Escreve dados no arquivo.

    method readData():data is
        // Lê dados de um arquivo.

// A classe decorador base segue a mesma interface que os outros
// componentes. O propósito primário dessa classe é definir a
// interface que envolve todos os decoradores concretos. A
// implementação padrão do código de envolvimento pode também
// incluir um campo para armazenar um componente envolvido e os
// meios para inicializá-lo.
class DataSourceDecorator implements DataSource is
    protected field wrappee: DataSource

    constructor DataSourceDecorator(source: DataSource) is
        wrappee = source

    // O decorador base simplesmente delega todo o trabalho para
    // o componente envolvido. Comportamentos extra podem ser
    // adicionados em decoradores concretos.
    method writeData(data) is
        wrappee.writeData(data)

    // Decoradores concretos podem chamar a implementação pai da
    // operação ao invés de chamar o objeto envolvido
    // diretamente. Essa abordagem simplifica a extensão de
    // classes decorador.
    method readData():data is
        return wrappee.readData()

// Decoradores concretos devem chamar métodos no objeto
// envolvido, mas podem adicionar algo próprio para o resultado.
// Os decoradores podem executar o comportamento adicional tanto
// antes como depois da chamada ao objeto envolvido.
class EncryptionDecorator extends DataSourceDecorator is
    method writeData(data) is
        // 1. Encriptar os dados passados.
        // 2. Passar dados encriptados para o método writeData
        // do objeto envolvido.

    method readData():data is
        // 1. Obter os dados do método readData do objeto
        // envolvido.
        // 2. Tentar decifrá-lo se for encriptado.
        // 3. Retornar o resultado.

// Você pode envolver objetos em diversas camadas de
// decoradores.
class CompressionDecorator extends DataSourceDecorator is
    method writeData(data) is
        // 1. Comprimir os dados passados.
        // 2. Passar os dados comprimidos para o método
        // writeData do objeto envolvido.

    method readData():data is
        // 1. Obter dados do método readData do objeto
        // envolvido.
        // 2. Tentar descomprimi-lo se for comprimido.
        // 3. Retornar o resultado.

// Opção 1. Um exemplo simples de uma montagem decorador.
class Application is
    method dumbUsageExample() is
        source = new FileDataSource(&quot;somefile.dat&quot;)
        source.writeData(salaryRecords)
        // O arquivo alvo foi escrito com dados simples.

        source = new CompressionDecorator(source)
        source.writeData(salaryRecords)
        // O arquivo alvo foi escrito com dados comprimidos.

        source = new EncryptionDecorator(source)
        // A variável fonte agora contém isso:
        // Encryption &gt; Compression &gt; FileDataSource
        source.writeData(salaryRecords)
        // O arquivo foi escrito com dados comprimidos e
        // encriptados.


// Opção 2. Código cliente que usa uma fonte de dados externa.
// Objetos SalaryManager não sabem e nem se importam sobre as
// especificações de armazenamento de dados. Eles trabalham com
// uma fonte de dados pré configurada recebida pelo configurador
// da aplicação.
class SalaryManager is
    field source: DataSource

    constructor SalaryManager(source: DataSource) { ... }

    method load() is
        return source.readData()

    method save() is
        source.writeData(salaryRecords)
    // ...Outros métodos úteis...


// A aplicação pode montar diferentes pilhas de decoradores no
// tempo de execução, dependendo da configuração ou ambiente.
class ApplicationConfigurator is
    method configurationExample() is
        source = new FileDataSource(&quot;salary.dat&quot;)
        if (enabledEncryption)
            source = new EncryptionDecorator(source)
        if (enabledCompression)
            source = new CompressionDecorator(source)

        logger = new SalaryManager(source)
        salary = logger.load()
    // ...</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Aplicabilidade {#applicability}

::::::: applicability
::: applicability-problem
Utilize o padrão Decorator quando você precisa ser capaz de projetar
comportamentos adicionais para objetos em tempo de execução sem quebrar
o código que usa esses objetos.
:::

::: applicability-solution
O Decorator lhe permite estruturar sua lógica de negócio em camadas,
criar um decorador para cada camada, e compor objetos com várias
combinações dessa lógica durante a execução. O código cliente pode
tratar de todos esses objetos da mesma forma, como todos seguem a mesma
interface comum.
:::

::: applicability-problem
Utilize o padrão quando é complicado ou impossível estender o
comportamento de um objeto usando herança.
:::

::: applicability-solution
Muitas linguagens de programação tem a palavra chave `final` que pode
ser usada para prevenir a extensão de uma classe. Para uma classe final,
a única maneira de reutilizar seu comportamento existente seria envolver
a classe com seu próprio invólucro usando o padrão Decorator.
:::
:::::::
::::::::

::: {.section .checklist}
##  Como implementar {#checklist}

1.  Certifique-se que seu domínio de negócio pode ser representado como
    um componente primário com múltiplas camadas opcionais sobre ele.

2.  Descubra quais métodos são comuns tanto para o componente primário e
    para as camadas opcionais. Crie uma interface componente e declare
    aqueles métodos ali.

3.  Crie uma classe componente concreta e defina o comportamento base
    nela.

4.  Crie uma classe decorador base. Ela deve ter um campo para armazenar
    uma referência ao objeto envolvido. O campo deve ser declarado com o
    tipo da interface componente para permitir uma ligação entre os
    componentes concretos e decoradores. O decorador base deve delegar
    todo o trabalho para o objeto envolvido.

5.  Certifique-se que todas as classes implementam a interface
    componente.

6.  Crie decoradores concretos estendendo-os a partir do decorador base.
    Um decorador concreto deve executar seu comportamento antes ou
    depois da chamada para o método pai (que sempre delega para o objeto
    envolvido).

7.  O código cliente deve ser responsável por criar decoradores e
    compô-los do jeito que o cliente precisa.
:::

:::::: {.section .pros-cons}
##  Prós e contras {#pros-cons}

::::: row
::: col-sm-6
-    Você pode estender o comportamento de um objeto sem fazer um nova
    subclasse.
-    Você pode adicionar ou remover responsabilidades de um objeto no
    momento da execução.
-    Você pode combinar diversos comportamentos ao envolver o objeto com
    múltiplos decoradores.
-    *Princípio de responsabilidade única*. Você pode dividir uma classe
    monolítica que implementa muitas possíveis variantes de um
    comportamento em diversas classes menores.
:::

::: col-sm-6
-    É difícil remover um invólucro de uma pilha de invólucros.
-    É difícil implementar um decorador de tal maneira que seu
    comportamento não dependa da ordem do pilha de decoradores.
-    A configuração inicial do código de camadas pode ficar bastante
    feia.
:::
:::::
::::::

::: {.section .relations}
##  Relações com outros padrões {#relations}

-   O [Adapter](adapter.html) fornece uma interface completamente
    diferente para acessar um objeto existente. Por outro lado, com o
    padrão [Decorator](decorator.html), a interface permanece a mesma ou
    é estendida. Além disso, o *Decorator* oferece suporte à composição
    recursiva, o que não é possível quando você usa o *Adapter*.

-   Com [Adapter](adapter.html), você acessa um objeto existente por
    meio de uma interface diferente. Com [Proxy](proxy.html), a
    interface permanece a mesma. Com [Decorator](decorator.html), você
    acessa o objeto por meio de uma interface aprimorada.

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

-   O [Composite](composite.html) e o [Decorator](decorator.html) tem
    diagramas estruturais parecidos já que ambos dependem de composição
    recursiva para organizar um número indefinido de objetos.

    Um *Decorador* é como um *Composite* mas tem apenas um componente
    filho. Há outra diferença significativa: o *Decorador* adiciona
    responsabilidades adicionais ao objeto envolvido, enquanto que o
    *Composite* apenas "soma" o resultado de seus filhos.

    Contudo, os padrões também podem cooperar: você pode usar o
    *Decorador* para estender o comportamento de um objeto específico na
    árvore [Composite](composite.html)

-   Projetos que fazem um uso pesado de [Composite](composite.html) e do
    [Decorator](decorator.html) podem se beneficiar com frequência do
    uso do [Prototype](prototype.html). Aplicando o padrão permite que
    você clone estruturas complexas ao invés de reconstruí-las do zero.

-   O [Decorator](decorator.html) permite que você mude a pele de um
    objeto, enquanto o [Strategy](strategy.html) permite que você mude
    suas entranhas.

-   O [Decorator](decorator.html) e o [Proxy](proxy.html) têm estruturas
    semelhantes, mas propósitos muito diferentes. Alguns padrões são
    construídos no princípio de composição, onde um objeto deve delegar
    parte do trabalho para outro. A diferença é que o *Proxy* geralmente
    gerencia o ciclo de vida de seu objeto serviço por conta própria,
    enquanto que a composição do *decoradores* é sempre controlada pelo
    cliente.
:::

::: {.section .implementations}
##  Exemplos de código {#implementations}

[![Decorator em
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](decorator/csharp/example.html "Decorator em C#"){.prog-lang-link}
[![Decorator em
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](decorator/cpp/example.html "Decorator em C++"){.prog-lang-link}
[![Decorator em
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](decorator/go/example.html "Decorator em Go"){.prog-lang-link}
[![Decorator em
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](decorator/java/example.html "Decorator em Java"){.prog-lang-link}
[![Decorator em
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](decorator/php/example.html "Decorator em PHP"){.prog-lang-link}
[![Decorator em
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](decorator/python/example.html "Decorator em Python"){.prog-lang-link}
[![Decorator em
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](decorator/ruby/example.html "Decorator em Ruby"){.prog-lang-link}
[![Decorator em
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](decorator/rust/example.html "Decorator em Rust"){.prog-lang-link}
[![Decorator em
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](decorator/swift/example.html "Decorator em Swift"){.prog-lang-link}
[![Decorator em
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](decorator/typescript/example.html "Decorator em TypeScript"){.prog-lang-link}
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

[Facade []{.fa .fa-arrow-right}](facade.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Composite](composite.html){.btn .btn-default
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
