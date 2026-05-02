::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
comportamentais](behavioral-patterns.html){.type}
:::

# Memento {#memento .title}

::: aka
Também conhecido como:
[Lembrança, ]{style="display:inline-block"}[Retrato, ]{style="display:inline-block"}[Snapshot]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

O **Memento** é um padrão de projeto comportamental que permite que você
salve e restaure o estado anterior de um objeto sem revelar os detalhes
de sua implementação.

<figure class="image">
<img
src="../../images/patterns/content/memento/memento-pt-brd834.png?id=4ac00cfa2ce4742d5975410662f354f6"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/memento/memento-pt-br.png?id=4ac00cfa2ce4742d5975410662f354f6 640w,/images/patterns/content/memento/memento-pt-br-1.5x.png?id=4f0810185c70151dc7937a9efdd3dff5 960w,/images/patterns/content/memento/memento-pt-br-2x.png?id=d1d60fe956990dcda071efd908ab86d9 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Padrão de projeto Memento" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagine que você está criando uma aplicação de editor de texto. Além da
simples edição de texto, seu editor pode formatar o texto, inserir
imagens em linha, etc.

Em determinado momento você decide permitir que os usuários desfaçam
quaisquer operações realizadas no texto. Essa funcionalidade tem se
tornado tão comum nos últimos anos que, hoje em dia, as pessoas esperam
que toda aplicação a tenha. Para a implementação você decide usar a
abordagem direta. Antes de realizar qualquer operação, a aplicação grava
o estado de todos os objetos e salva eles em algum armazenamento. Mais
tarde, quando um usuário decide reverter a ação, a aplicação busca o
último retrato do histórico e usa ele para restaurar o estado de todos
os objetos.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/problem1-pt-br6c2f.png?id=3b7cdeb04e86fe63f342d42d341bb711"
style="aspect-ratio: 2.61;"
srcset="/images/patterns/diagrams/memento/problem1-pt-br.png?id=3b7cdeb04e86fe63f342d42d341bb711 470w,/images/patterns/diagrams/memento/problem1-pt-br-1.5x.png?id=404120903f01eb98f1668ec33afdedc5 705w,/images/patterns/diagrams/memento/problem1-pt-br-2x.png?id=3f0848d19580cde1fd191b608d382cfa 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Revertendo operaçõs no editor" />
<figcaption><p>Antes de executar uma operação, a aplicação salva um
retrato do estado dos objetos, que pode mais tarde ser usada para
restaurá-los a seu estado anterior.</p></figcaption>
</figure>

Vamos pensar sobre esses retratos de estado. Como exatamente você
produziria um? Você provavelmente teria que percorrer todos os campos de
um objeto e copiar seus valores para o armazenamento. Contudo, isso só
funcionaria se o objeto tiver poucas restrições de acesso a seu
conteúdo. Infelizmente, a maioria dos objetos reais não deixa os outros
espiarem dentro deles assim tão facilmente, escondendo todos os dados
significativos em campos privados.

Ignore esse problema por enquanto e vamos assumir que nossos objetos
comportam-se como hippies: preferindo relações abertas e mantendo seus
estado público. Embora essa abordagem resolveria o problema imediato e
permitiria que você produzisse retratos dos estados de seus objetos à
vontade, ela ainda tem vários problemas graves. No futuro, você pode
decidir refatorar algumas das classes do editor, ou adicionar e remover
alguns campos. Parece fácil, mas isso também exigiria mudar as classes
responsáveis por copiar o estado dos objetos afetados.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/problem2-pt-br5bab.png?id=639e955b098ae488a23f969165802a45"
style="aspect-ratio: 1.43;"
srcset="/images/patterns/diagrams/memento/problem2-pt-br.png?id=639e955b098ae488a23f969165802a45 300w,/images/patterns/diagrams/memento/problem2-pt-br-1.5x.png?id=aa2620d5cc27beb7def4ad7e30f1a632 450w,/images/patterns/diagrams/memento/problem2-pt-br-2x.png?id=97cc15b492b154cf39892daab78940d5 600w"
sizes="(max-width: 720px) 100vw, 300px" loading="lazy" width="300"
alt="Como fazer uma cópia de um estado privado de um objeto?" />
<figcaption><p>Como fazer uma cópia do estado privado de
um objeto?</p></figcaption>
</figure>

E tem mais. Vamos considerar os próprios "retratos" do estado do editor.
Que dados ele contém? No mínimo dos mínimos ele terá o próprio texto,
coordenadas do cursor, posição atual do scroll, etc. Para fazer um
retrato você teria que coletar todos esses valores e colocá-los em algum
tipo de contêiner.

O mais provável é que você vai armazenar muitos desses objetos
contêineres dentro de alguma lista que representaria o histórico.
Portanto os contêineres terminariam sendo objetos de uma classe. A
classe não teria muitos métodos, mas vários campos que espelhariam o
estado do editor. Para permitir que objetos sejam escritos e lidos no e
do retrato, você teria que provavelmente tornar seus campos públicos.
Isso iria expor todos os estados do editor, privados ou não. Outras
classes se tornariam dependentes de cada pequena mudança na classe do
retrato, o que poderia acontecer dentro dos campos privados e métodos
sem afetar as classes externas.

Parece que chegamos em um beco sem saída: você ou expõe todos os
detalhes internos das classes tornando-as frágeis, ou restringe o acesso
ao estado delas, tornando impossível produzir retratos. Existe alguma
outra maneira de implementar o \"desfazer\"?
:::

::: {.section .solution}
##  Solução {#solution}

Todos os problemas que vivenciamos foram causados por um encapsulamento
quebrado. Alguns objetos tentaram fazer mais do que podiam. Para coletar
os dados necessários para fazer uma ação, eles invadiram o espaço
privado de outros objetos ao invés de deixar esses objetos fazer a
verdadeira ação.

O padrão Memento delega a criação dos retratos do estado para o próprio
dono do estado, o objeto *originador*. Portanto, ao invés de outros
objetos tentarem copiar o estado do editor "a partir do lado de fora", a
própria classe do editor pode fazer o retrato já que tem acesso total a
seu próprio estado.

O padrão sugere armazenar a cópia do estado de um objeto em um objeto
especial chamado *memento*. Os conteúdos de um memento não são
acessíveis para qualquer outro objeto exceto aquele que o produziu.
Outros objetos podem se comunicar com mementos usando uma interface
limitada que pode permitir a recuperação dos metadados do retrato (data
de criação, nome a operação efetuada, etc.), mas não ao estado do objeto
original contido no retrato.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/solution-pt-br9716.png?id=fb2acd45818a5c8abdbf31f0b8a39e35"
style="aspect-ratio: 1.36;"
srcset="/images/patterns/diagrams/memento/solution-pt-br.png?id=fb2acd45818a5c8abdbf31f0b8a39e35 610w,/images/patterns/diagrams/memento/solution-pt-br-1.5x.png?id=eb9f2027704e4337e7d1d4f93c241352 915w,/images/patterns/diagrams/memento/solution-pt-br-2x.png?id=04cd22cedc221ba689cd5146f3438b98 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="O originador tem acesso total ao memento, enquanto que o cuidador pode acessar somente os metadados" />
<figcaption><p>O originador tem acesso total ao memento, enquanto que o
cuidador pode acessar somente os metadados</p></figcaption>
</figure>

Tal regra restritiva permite que você armazene mementos dentro de outros
objetos, geralmente chamados de *cuidadores*. Uma vez que o cuidador
trabalha com o memento apenas por meio de uma interface limitada, ele
não será capaz de mexer com o estado armazenado dentro do memento. Ao
mesmo tempo, o originador tem acesso total a todos os campos dentro do
memento, permitindo que ele o restaure ao seu estado anterior à vontade.

Em nosso exemplo de editor de texto, nós podemos criar uma classe
histórico separada para agir como a cuidadora. Uma pilha de mementos
armazenada dentro da cuidadora irá crescer a cada vez que o editor
estiver prestes a executar uma operação. Você pode até mesmo renderizar
essa pilha dentro do UI da aplicação, mostrando o histórico de operações
realizadas anteriormente para um usuário.

Quando um usuário aciona o desfazer, o histórico pega o memento mais
recente da pilha e o passa de volta para o editor, pedindo uma reversão.
Já que o editor tem acesso total ao memento, ele muda seu próprio estado
com os valores obtidos do memento.
:::

:::::::::: {.section .structure-container}
##  Estrutura {#structure}

::::::::: structure
#### Implementação baseada em classes aninhadas

A implementação clássica do padrão dependente do apoio para classes
aninhadas, disponível em muitas linguagens de programação populares
(tais como C++, C#, e Java).

:::: memento-classic
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure180c1.png?id=4b4a42363a005b617d4df06689787385"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.87;"
srcset="/images/patterns/diagrams/memento/structure1.png?id=4b4a42363a005b617d4df06689787385 580w,/images/patterns/diagrams/memento/structure1-1.5x.png?id=82cf757f153840c85d27bd63f3f3e133 870w,/images/patterns/diagrams/memento/structure1-2x.png?id=d4e77295e51c2417f22b7abb396d5977 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="Memento baseado em classes aninhadas" /><img
src="../../images/patterns/diagrams/memento/structure1-indexeda7c3.png?id=f79a8356b087ae6b004aec42b787ae2e"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.87;"
srcset="/images/patterns/diagrams/memento/structure1-indexed.png?id=f79a8356b087ae6b004aec42b787ae2e 580w,/images/patterns/diagrams/memento/structure1-indexed-1.5x.png?id=0687dc84dd4c98b4591a70ebd05c4d8e 870w,/images/patterns/diagrams/memento/structure1-indexed-2x.png?id=62fea7bdbc96420568869ea3bd25f6ad 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="Memento baseado em classes aninhadas" />
</figure>
:::
::::

1.  A classe **Originadora** pode produzir retratos de seu próprio
    estado, bem como restaurar seu estado de retratos quando necessário.

2.  O **Memento** é um objeto de valor que age como um retrato do estado
    da originadora. É uma prática comum fazer o memento imutável e
    passar os dados para ele apenas uma vez, através do construtor.

3.  A **Cuidadora** sabe não só "quando" e "por quê" capturar o estado
    da originadora, mas também quando o estado deve ser restaurado.

    Uma cuidadora pode manter registros do histórico da originadora
    armazenando os mementos em um pilha. Quando a originadora precisa
    voltar atrás no histórico, a cuidadora busca o memento mais do topo
    da pilha e o passa para o método de restauração da originadora.

4.  Nessa implementação, a classe memento está aninhada dentro da
    originadora. Isso permite que a originadora acesse os campos e
    métodos do memento, mesmo que eles tenham sido declarados privados.
    Por outro lado, a cuidadora tem um acesso muito limitado aos campos
    do memento, que permite ela armazenar os mementos em uma pilha, mas
    não permite mexer com seu estado.

#### Implementação baseada em uma interface intermediária

Há uma implementação alternativa, adequada para linguagens de
programação que não suportam classes aninhadas (sim, PHP, estou falando
de você).

:::: memento-empty-interface
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure21ec9.png?id=fcff71cb648389be2e302fbe55e2f847"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/memento/structure2.png?id=fcff71cb648389be2e302fbe55e2f847 560w,/images/patterns/diagrams/memento/structure2-1.5x.png?id=5c05495a7ca6545fcc57f70ea7ee904a 840w,/images/patterns/diagrams/memento/structure2-2x.png?id=aa7fb5d0f622d4344a2cb590f437f8c8 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Memento sem classes aninhadas" /><img
src="../../images/patterns/diagrams/memento/structure2-indexede045.png?id=2c98b4f64b03f2a30e159de31ca9f718"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/memento/structure2-indexed.png?id=2c98b4f64b03f2a30e159de31ca9f718 560w,/images/patterns/diagrams/memento/structure2-indexed-1.5x.png?id=1ba6e0f22bb613f3f1dcf86850c3b604 840w,/images/patterns/diagrams/memento/structure2-indexed-2x.png?id=2fb637daef1110dfa89f15b2d4627894 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Memento sem classes aninhadas" />
</figure>
:::
::::

1.  Na ausência de classes aninhadas, você pode restringir o acesso aos
    campos do memento ao estabelecer uma convenção para que cuidadoras
    possam trabalhar com um memento através apenas de uma interface
    intermediária explicitamente declarada, que só declararia os métodos
    relacionados aos metadados do memento.

2.  Por outro lado, as originadoras podem trabalhar com um objeto
    memento diretamente, acessando campos e métodos declarados na classe
    memento. O lado ruim dessa abordagem é que você precisa declarar
    todos os membros do memento como públicos.

#### Implementação com um encapsulamento ainda mais estrito

Há ainda outra implementação que é útil quando você não quer deixar a
mínima chance de outras classes acessarem o estado da originadora
através do memento.

:::: memento-safe
::: {.struct-image3 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure3f138.png?id=6c3ef6a916be8c8ec6d6659d19a6a79f"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/memento/structure3.png?id=6c3ef6a916be8c8ec6d6659d19a6a79f 590w,/images/patterns/diagrams/memento/structure3-1.5x.png?id=9bb6d9dd5567bc71d9e93efc9183ef3a 885w,/images/patterns/diagrams/memento/structure3-2x.png?id=988c37f92059457153d26ba3458d371e 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Memento com encapsulamento estrito" /><img
src="../../images/patterns/diagrams/memento/structure3-indexedadf8.png?id=17e84b0ef89a41bb3fb844c8d7a445ad"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/memento/structure3-indexed.png?id=17e84b0ef89a41bb3fb844c8d7a445ad 590w,/images/patterns/diagrams/memento/structure3-indexed-1.5x.png?id=c121c75333433b70b9a67b75e536214c 885w,/images/patterns/diagrams/memento/structure3-indexed-2x.png?id=fef9ae2a0151c105976075aafb8939dd 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Memento com encapsulamento estrito" />
</figure>
:::
::::

1.  Essa implementação permite ter múltiplos tipos de originadoras e
    mementos. Cada originadora trabalha com uma classe memento
    correspondente. Nem as originadoras nem os mementos expõem seu
    estado para ninguém.

2.  Cuidadoras são agora explicitamente restritas de mudar o estado
    armazenado nos mementos. Além disso, a classe cuidadora se torna
    independente da originadora porque o método de restauração agora
    está definido na classe memento.

3.  Cada memento se torna ligado à originadora que o produziu. A
    originadora passa a si mesmo para o construtor do memento, junto com
    os valores de seu estado. Graças a relação íntima entre essas
    classes, um memento pode restaurar o estado da sua originadora,
    desde que esta última tenha definido os setters apropriados.
:::::::::
::::::::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Este exemplo usa o padrão Memento junto com o padrão
[Command](command.html) para armazenar retratos do estado de um editor
de texto complexo e restaurá-lo para um estado anterior desses retratos
quando necessário

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/example4526.png?id=fb2196b065f374a1c2a64a0943463760"
style="aspect-ratio: 4.29;"
srcset="/images/patterns/diagrams/memento/example.png?id=fb2196b065f374a1c2a64a0943463760 600w,/images/patterns/diagrams/memento/example-1.5x.png?id=174b686f918a2c49e2545d5573c52d42 900w,/images/patterns/diagrams/memento/example-2x.png?id=41a73f3cc22bc3dd180f53e6968974d4 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Exemplo da estrutura do Memento" />
<figcaption><p>Salvando retratos do estado de um editor
de texto.</p></figcaption>
</figure>

Os objetos comando agem como cuidadores. Eles buscam o memento do editor
antes de executar operações relacionadas aos comandos. Quando um usuário
tenta desfazer o comando mais recente, o editor pode usar o memento
armazenando naquele comando para reverter a si mesmo para o estado
anterior.

A classe memento não declara qualquer campo público, getters, ou
setters. Portanto nenhum objeto pode alterar seus conteúdos. Os mementos
estão ligados ao objeto do editor que os criou. Isso permite que um
memento restaure o estado do editor ligado a ele passando os dados via
setters no objeto do editor. Já que mementos são ligados com objetos do
editor específicos, você pode fazer sua aplicação suportar diversas
janelas independentes do editor com uma pilha centralizada de desfazer.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// O originador tem alguns dados importantes que podem mudar com
// o tempo. Ele também define um método para salvar seu estado
// dentro de um memento e outro método para restaurar o estado
// dele.
class Editor is
    private field text, curX, curY, selectionWidth

    method setText(text) is
        this.text = text

    method setCursor(x, y) is
        this.curX = x
        this.curY = y

    method setSelectionWidth(width) is
        this.selectionWidth = width

    // Salva o estado atual dentro de um memento.
    method createSnapshot():Snapshot is
        // O memento é um objeto imutável; é por isso que o
        // originador passa seu estado para os parâmetros do
        // construtor do memento.
        return new Snapshot(this, text, curX, curY, selectionWidth)

// A classe memento armazena o estado anterior do editor.
class Snapshot is
    private field editor: Editor
    private field text, curX, curY, selectionWidth

    constructor Snapshot(editor, text, curX, curY, selectionWidth) is
        this.editor = editor
        this.text = text
        this.curX = x
        this.curY = y
        this.selectionWidth = selectionWidth

    // Em algum momento, um estado anterior do editor pode ser
    // restaurado usando um objeto memento.
    method restore() is
        editor.setText(text)
        editor.setCursor(curX, curY)
        editor.setSelectionWidth(selectionWidth)

// Um objeto comando pode agir como cuidador. Neste caso, o
// comando obtém o memento antes que ele mude o estado do
// originador. Quando o undo(desfazer) é solicitado, ele
// restaura o estado do originador a partir de um memento.
class Command is
    private field backup: Snapshot

    method makeBackup() is
        backup = editor.createSnapshot()

    method undo() is
        if (backup != null)
            backup.restore()
    // ...</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Aplicabilidade {#applicability}

::::::: applicability
::: applicability-problem
Utilize o padrão Memento quando você quer produzir retratos do estado de
um objeto para ser capaz de restaurar um estado anterior do objeto.
:::

::: applicability-solution
O padrão Memento permite que você faça cópias completas do estado de um
objeto, incluindo campos privados, e armazená-los separadamente do
objeto. Embora a maioria das pessoas vão lembrar desse padrão graças ao
caso "desfazer", ele também é indispensável quando se está lidando com
transações (isto é, se você precisa reverter uma operação quando se
depara com um erro).
:::

::: applicability-problem
Utilize o padrão quando o acesso direto para os campos/getters/setters
de um objeto viola seu encapsulamento.
:::

::: applicability-solution
O Memento faz o próprio objeto ser responsável por criar um retrato de
seu estado. Nenhum outro objeto pode ler o retrato, fazendo do estado
original do objeto algo seguro e confiável.
:::
:::::::
::::::::

::: {.section .checklist}
##  Como implementar {#checklist}

1.  Determine qual classe vai fazer o papel de originadora. É importante
    saber se o programa usa um objeto central deste tipo ou múltiplos
    objetos pequenos.

2.  Crie a classe memento. Um por um, declare o conjunto dos campos que
    espelham os campos declarados dentro da classe originadora.

3.  Faça a classe memento ser imutável. Um memento deve aceitar os dados
    apenas uma vez, através do construtor. A classe não deve ter
    setters.

4.  Se a sua linguagem de programação suporta classes aninhadas, aninhe
    o memento dentro da originadora. Se não, extraia uma interface em
    branco da classe memento e faça todos os outros objetos usá-la para
    se referir ao memento. Você pode adicionar algumas operações de
    metadados para a interface, mas nada que exponha o estado da
    originadora.

5.  Adicione um método para produção de mementos na classe originadora.
    A originadora deve passar seu estado para o memento através de um ou
    múltiplos argumentos do construtor do memento.

    O tipo de retorno do método deve ser o da interface que você extraiu
    na etapa anterior (assumindo que você extraiu alguma coisa). Por
    debaixo dos panos, o método de produção de memento deve funcionar
    diretamente com a classe memento.

6.  Adicione um método para restaurar o estado da classe originadora
    para sua classe. Ele deve aceitar o objeto memento como um
    argumento. Se você extraiu uma interface na etapa anterior, faça-a
    do tipo do parâmetro. Neste caso, você precisa converter o tipo do
    objeto que está vindo para a classe memento, uma vez que a
    originadora precisa de acesso total a aquele objeto.

7.  A cuidadora, estando ela representando um objeto comando, um
    histórico, ou algo completamente diferente, deve saber quando pedir
    novos mementos da originadora, como armazená-los, e quando restaurar
    a originadora com um memento em particular.

8.  O elo entre cuidadoras e originadoras deve ser movido para dentro da
    classe memento. Neste caso, cada memento deve se conectar com a
    originadora que criou ele. O método de restauração também deve ser
    movido para a classe memento. Contudo, isso tudo faria sentido
    somente se a classe memento estiver aninhada dentro da originadora
    ou a classe originadora fornece setters suficientes para
    sobrescrever seu estado.
:::

:::::: {.section .pros-cons}
##  Prós e contras {#pros-cons}

::::: row
::: col-sm-6
-    Você pode produzir retratos do estado de um objeto sem violar seu
    encapsulamento.
-    Você pode simplificar o código da originadora permitindo que a
    cuidadora mantenha o histórico do estado da originadora.
:::

::: col-sm-6
-    A aplicação pode consumir muita RAM se os clientes criarem mementos
    com muita frequência.
-    Cuidadoras devem acompanhar o ciclo de vida da originadora para
    serem capazes de destruir mementos obsoletos.
-    A maioria das linguagens de programação dinâmicas, tais como PHP,
    Python, e JavaScript, não conseguem garantir que o estado dentro do
    memento permaneça intacto.
:::
:::::
::::::

::: {.section .relations}
##  Relações com outros padrões {#relations}

-   Você pode usar o [Command](command.html) e o [Memento](memento.html)
    juntos quando implementando um "desfazer". Neste caso, os comandos
    são responsáveis pela realização de várias operações sobre um objeto
    alvo, enquanto que os mementos salvam o estado daquele objeto
    momentos antes de um comando ser executado.

-   Você pode usar o [Memento](memento.html) junto com o
    [Iterator](iterator.html) para capturar o estado de iteração atual e
    revertê-lo se necessário.

-   Algumas vezes o [Prototype](prototype.html) pode ser uma alternativa
    mais simples a um [Memento](memento.html). Isso funciona se o
    objeto, o estado no qual você quer armazenar na história, é
    razoavelmente intuitivo e não tem ligações para recursos externos,
    ou as ligações são fáceis de se restabelecer.
:::

::: {.section .implementations}
##  Exemplos de código {#implementations}

[![Memento em
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](memento/csharp/example.html "Memento em C#"){.prog-lang-link}
[![Memento em
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](memento/cpp/example.html "Memento em C++"){.prog-lang-link}
[![Memento em
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](memento/go/example.html "Memento em Go"){.prog-lang-link}
[![Memento em
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](memento/java/example.html "Memento em Java"){.prog-lang-link}
[![Memento em
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](memento/php/example.html "Memento em PHP"){.prog-lang-link}
[![Memento em
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](memento/python/example.html "Memento em Python"){.prog-lang-link}
[![Memento em
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](memento/ruby/example.html "Memento em Ruby"){.prog-lang-link}
[![Memento em
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](memento/rust/example.html "Memento em Rust"){.prog-lang-link}
[![Memento em
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](memento/swift/example.html "Memento em Swift"){.prog-lang-link}
[![Memento em
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](memento/typescript/example.html "Memento em TypeScript"){.prog-lang-link}
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

[Observer []{.fa .fa-arrow-right}](observer.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Mediator](mediator.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::
