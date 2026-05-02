::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
comportamentais](behavioral-patterns.html){.type}
:::

# Strategy {#strategy .title}

::: aka
Também conhecido como: [Estratégia]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

O **Strategy** é um padrão de projeto comportamental que permite que
você defina uma família de algoritmos, coloque-os em classes separadas,
e faça os objetos deles intercambiáveis.

<figure class="image">
<img
src="../../images/patterns/content/strategy/strategy4ca6.png?id=379bfba335380500375881a3da6507e0"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/strategy/strategy.png?id=379bfba335380500375881a3da6507e0 640w,/images/patterns/content/strategy/strategy-1.5x.png?id=33ecebc66a9761454f2786a9b3e9bbe5 960w,/images/patterns/content/strategy/strategy-2x.png?id=1cee47d05a76fddf07dce9c67b700748 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Padrão de projeto Strategy" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Um dia você decide criar uma aplicação de navegação para viajantes
casuais. A aplicação estava centrada em um mapa bonito que ajudava os
usuários a se orientarem rapidamente em uma cidade.

Uma das funcionalidades mais pedidas para a aplicação era o planejamento
automático de rotas. Um usuário deveria ser capaz de entrar com um
endereço e ver a rota mais rápida no mapa.

A primeira versão da aplicação podia apenas construir rotas sobre
rodovias, e isso agradou muito quem viaja de carro. Porém aparentemente,
nem todo mundo dirige em suas férias. Então com a próxima atualização
você adicionou uma opção de construir rotas de caminhada. Logo após isso
você adicionou outra opção para permitir que as pessoas usem o
transporte público.

Contudo, isso foi apenas o começo. Mais tarde você planejou adicionar um
construtor de rotas para ciclistas. E mais tarde, outra opção para
construir rotas até todas as atrações turísticas de uma cidade.

<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/problem9b55.png?id=e5ca943e559a979bd31d20e232dc22b6"
style="aspect-ratio: 2.2;"
srcset="/images/patterns/diagrams/strategy/problem.png?id=e5ca943e559a979bd31d20e232dc22b6 330w,/images/patterns/diagrams/strategy/problem-1.5x.png?id=31d1042ffc28056845e45d5c616da2a9 495w,/images/patterns/diagrams/strategy/problem-2x.png?id=3974fb99c97aec525dd0ffcff2e48e78 660w"
sizes="(max-width: 720px) 100vw, 330px" loading="lazy" width="330"
alt="O código do navegador ficou muito inchado" />
<figcaption><p>O código do navegador ficou
muito inchado.</p></figcaption>
</figure>

Embora da perspectiva de negócio a aplicação tenha sido um sucesso, a
parte técnica causou a você muitas dores de cabeça. Cada vez que você
adicionava um novo algoritmo de roteamento, a classe principal do
navegador dobrava de tamanho. Em determinado momento, o monstro se
tornou algo muito difícil de se manter.

Qualquer mudança a um dos algoritmos, seja uma simples correção de bug
ou um pequeno ajuste no valor das ruas, afetava toda a classe,
aumentando a chance de criar um erro no código já existente.

Além disso, o trabalho em equipe se tornou ineficiente. Seus
companheiros de equipe, que foram contratados após ao bem sucedido
lançamento do produto, se queixavam que gastavam muito tempo resolvendo
conflitos de fusão. Implementar novas funcionalidades necessitava
mudanças na classe gigantesca, conflitando com os códigos criados por
outras pessoas.
:::

::: {.section .solution}
##  Solução {#solution}

O padrão Strategy sugere que você pegue uma classe que faz algo
específico em diversas maneiras diferentes e extraia todos esses
algoritmos para classes separadas chamadas *estratégias*.

A classe original, chamada *contexto*, deve ter um campo para armazenar
uma referência para um dessas estratégias. O contexto delega o trabalho
para um objeto estratégia ao invés de executá-lo por conta própria.

O contexto não é responsável por selecionar um algoritmo apropriado para
o trabalho. Ao invés disso, o cliente passa a estratégia desejada para o
contexto. Na verdade, o contexto não sabe muito sobre as estratégias.
Ele trabalha com todas elas através de uma interface genérica, que
somente expõe um único método para acionar o algoritmo encapsulado
dentro da estratégia selecionada.

Desta forma o contexto se torna independente das estratégias concretas,
então você pode adicionar novos algoritmos ou modificar os existentes
sem modificar o código do contexto ou outras estratégias.

<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/solution1435.png?id=0813a174b29a2ed5902d321aba28e5fc"
style="aspect-ratio: 2.04;"
srcset="/images/patterns/diagrams/strategy/solution.png?id=0813a174b29a2ed5902d321aba28e5fc 570w,/images/patterns/diagrams/strategy/solution-1.5x.png?id=ce3d4e57f4a2a06ebc96f2607b3d6691 855w,/images/patterns/diagrams/strategy/solution-2x.png?id=66b5ee048ea2ad25c4b20f180ebf94d7 1140w"
sizes="(max-width: 720px) 100vw, 570px" loading="lazy" width="570"
alt="Estratégias de planejamento de rotas" />
<figcaption><p>Estratégias de planejamento de rotas.</p></figcaption>
</figure>

Em nossa aplicação de navegação, cada algoritmo de roteamento pode ser
extraído para sua própria classe com um único método `construirRota`. O
método aceita uma origem e um destino e retorna uma coleção de pontos da
rota.

Mesmo dando os mesmos argumentos, cada classe de roteamento pode
construir uma rota diferente, a classe navegadora principal não se
importa qual algoritmo está selecionado uma vez que seu trabalho
primário é renderizar um conjunto de pontos num mapa. A classe tem um
método para trocar a estratégia ativa de rotas, então seus clientes, bem
como os botões na interface de usuário, podem substituir o comportamento
de rotas selecionado por um outro.
:::

::: {.section .analogy}
##  Analogia com o mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/strategy/strategy-comic-1-pt-br47f0.png?id=4a05476330ae772bad5889ece29e4f9c"
style="aspect-ratio: 2.13;"
srcset="/images/patterns/content/strategy/strategy-comic-1-pt-br.png?id=4a05476330ae772bad5889ece29e4f9c 640w,/images/patterns/content/strategy/strategy-comic-1-pt-br-1.5x.png?id=6e72257de2ec555255cf4f65b1caff3c 960w,/images/patterns/content/strategy/strategy-comic-1-pt-br-2x.png?id=2a49144307a58399f6ebe41450f55e5e 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Várias estratégias de transporte" />
<figcaption><p>Várias estratégias para se chegar
ao aeroporto.</p></figcaption>
</figure>

Imagine que você tem que chegar ao aeroporto. Você pode pegar um ônibus,
pedir um táxi, ou subir em sua bicicleta. Essas são suas estratégias de
transporte. Você pode escolher uma das estratégias dependendo de fatores
como orçamento ou restrições de tempo.
:::

::::: {.section .structure-container}
##  Estrutura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/structuree491.png?id=c6aa910c94960f35d100bfca02810ea1"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.19;"
srcset="/images/patterns/diagrams/strategy/structure.png?id=c6aa910c94960f35d100bfca02810ea1 440w,/images/patterns/diagrams/strategy/structure-1.5x.png?id=5b1c6376b06374719dcae95455b068d8 660w,/images/patterns/diagrams/strategy/structure-2x.png?id=5bd791857c3bab419bcf4fa86877439d 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="Estrutura do padrão de projeto Strategy" /><img
src="../../images/patterns/diagrams/strategy/structure-indexed64c7.png?id=ff55c5a6273cf82a5667f3cab5b605c7"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.15;"
srcset="/images/patterns/diagrams/strategy/structure-indexed.png?id=ff55c5a6273cf82a5667f3cab5b605c7 450w,/images/patterns/diagrams/strategy/structure-indexed-1.5x.png?id=3d6ba05b8a74a9fb8e3f88eb9ca1e30f 675w,/images/patterns/diagrams/strategy/structure-indexed-2x.png?id=9f8e2f838f16705775411e2b4616820e 900w"
sizes="(max-width: 720px) 100vw, 450px" loading="lazy" width="450"
alt="Estrutura do padrão de projeto Strategy" />
</figure>
:::

1.  O **Contexto** mantém uma referência para uma das estratégias
    concretas e se comunica com esse objeto através da interface da
    estratégia.

2.  A interface **Estratégia** é comum à todas as estratégias concretas.
    Ela declara um método que o contexto usa para executar uma
    estratégia.

3.  **Estratégias Concretas** implementam diferentes variações de um
    algoritmo que o contexto usa.

4.  O contexto chama o método de execução no objeto estratégia ligado
    cada vez que ele precisa rodar um algoritmo. O contexto não sabe
    qual tipo de estratégia ele está trabalhando ou como o algoritmo é
    executado.

5.  O **Cliente** cria um objeto estratégia específico e passa ele para
    o contexto. O contexto expõe um setter que permite o cliente mudar a
    estratégia associada com contexto durante a execução.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Neste exemplo, o contexto usa múltiplas **estratégias** para executar
várias operações aritméticas.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// A interface estratégia declara operações comuns a todas as
// versões suportadas de algum algoritmo. O contexto usa essa
// interface para chamar o algoritmo definido pelas estratégias
// concretas.
interface Strategy is
    method execute(a, b)

// Estratégias concretas implementam o algoritmo enquanto seguem
// a interface estratégia base. A interface faz delas
// intercomunicáveis no contexto.
class ConcreteStrategyAdd implements Strategy is
    method execute(a, b) is
        return a + b

class ConcreteStrategySubtract implements Strategy is
    method execute(a, b) is
        return a - b

class ConcreteStrategyMultiply implements Strategy is
    method execute(a, b) is
        return a * b

// O contexto define a interface de interesse para clientes.
class Context is
    // O contexto mantém uma referência para um dos objetos
    // estratégia. O contexto não sabe a classe concreta de uma
    // estratégia. Ele deve trabalhar com todas as estratégias
    // através da interface estratégia.
    private strategy: Strategy

    // Geralmente o contexto aceita uma estratégia através do
    // construtor, e também fornece um setter para que a
    // estratégia possa ser trocado durante o tempo de execução.
    method setStrategy(Strategy strategy) is
        this.strategy = strategy

    // O contexto delega algum trabalho para o objeto estratégia
    // ao invés de implementar múltiplas versões do algoritmo
    // por conta própria.
    method executeStrategy(int a, int b) is
        return strategy.execute(a, b)


// O código cliente escolhe uma estratégia concreta e passa ela
// para o contexto. O cliente deve estar ciente das diferenças
// entre as estratégia para que faça a escolha certa.
class ExampleApplication is
    method main() is
        Cria um objeto contexto.

        Lê o primeiro número.
        Lê o último número.
        Lê a ação desejada da entrada do usuário

        if (action == addition) then
            context.setStrategy(new ConcreteStrategyAdd())

        if (action == subtraction) then
            context.setStrategy(new ConcreteStrategySubtract())

        if (action == multiplication) then
            context.setStrategy(new ConcreteStrategyMultiply())

        result = context.executeStrategy(First number, Second number)

        Imprimir resultado.</code></pre>
</figure>
:::

:::::::::::: {.section .applicability-container}
##  Aplicabilidade {#applicability}

::::::::::: applicability
::: applicability-problem
Utilize o padrão Strategy quando você quer usar diferentes variantes de
um algoritmo dentro de um objeto e ser capaz de trocar de um algoritmo
para outro durante a execução.
:::

::: applicability-solution
O padrão Strategy permite que você altere indiretamente o comportamento
de um objeto durante a execução ao associá-lo com diferentes sub-objetos
que pode fazer sub-tarefas específicas em diferentes formas.
:::

::: applicability-problem
Utilize o Strategy quando você tem muitas classes parecidas que somente
diferem na forma que elas executam algum comportamento.
:::

::: applicability-solution
O padrão Strategy permite que você extraia o comportamento variante para
uma hierarquia de classe separada e combine as classes originais em uma,
portando reduzindo código duplicado.
:::

::: applicability-problem
Utilize o padrão para isolar a lógica do negócio de uma classe dos
detalhes de implementação de algoritmos que podem não ser tão
importantes no contexto da lógica.
:::

::: applicability-solution
O padrão Strategy permite que você isole o código, dados internos, e
dependências de vários algoritmos do restante do código. Vários clientes
podem obter uma simples interface para executar os algoritmos e
trocá-los durante a execução do programa.
:::

::: applicability-problem
Utilize o padrão quando sua classe tem um operador condicional muito
grande que troca entre diferentes variantes do mesmo algoritmo.
:::

::: applicability-solution
O padrão Strategy permite que você se livre dessa condicional ao extrair
todos os algoritmos para classes separadas, todos eles implementando a
mesma interface. O objeto original delega a execução de um desses
objetos, ao invés de implementar todas as variantes do algoritmo.
:::
:::::::::::
::::::::::::

::: {.section .checklist}
##  Como implementar {#checklist}

1.  Na classe contexto, identifique um algoritmo que é sujeito a
    frequentes mudanças. Pode ser também uma condicional enorme que
    seleciona e executa uma variante do mesmo algoritmo durante a
    execução do programa.

2.  Declare a interface da estratégia comum para todas as variantes do
    algoritmo.

3.  Um por um, extraia todos os algoritmos para suas próprias classes.
    Elas devem todas implementar a interface estratégia.

4.  Na classe contexto, adicione um campo para armazenar uma referência
    a um objeto estratégia. Forneça um setter para substituir valores
    daquele campo. O contexto deve trabalhar com o objeto estratégia
    somente através da interface estratégia. O contexto pode definir uma
    interface que deixa a estratégia acessar seus dados.

5.  Os Clientes do contexto devem associá-lo com uma estratégia
    apropriada que coincide com a maneira que esperam que o contexto
    atue em seu trabalho primário.
:::

:::::: {.section .pros-cons}
##  Prós e contras {#pros-cons}

::::: row
::: col-sm-6
-    Você pode trocar algoritmos usados dentro de um objeto durante a
    execução.
-    Você pode isolar os detalhes de implementação de um algoritmo do
    código que usa ele.
-    Você pode substituir a herança por composição.
-    *Princípio aberto/fechado*. Você pode introduzir novas estratégias
    sem mudar o contexto.
:::

::: col-sm-6
-    Se você só tem um par de algoritmos e eles raramente mudam, não há
    motivo real para deixar o programa mais complicado com novas classes
    e interfaces que vêm junto com o padrão.
-    Os Clientes devem estar cientes das diferenças entre as estratégias
    para serem capazes de selecionar a adequada.
-    Muitas linguagens de programação modernas tem suporte do tipo
    funcional que permite que você implemente diferentes versões de um
    algoritmo dentro de um conjunto de funções anônimas. Então você
    poderia usar essas funções exatamente como se estivesse usando
    objetos estratégia, mas sem inchar seu código com classes e
    interfaces adicionais.
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

-   O [Command](command.html) e o [Strategy](strategy.html) podem ser
    parecidos porque você pode usar ambos para parametrizar um objeto
    com alguma ação. Contudo, eles têm propósitos bem diferentes.

    -   Você pode usar o *Command* para converter qualquer operação em
        um objeto. Os parâmetros da operação se transformam em campos
        daquele objeto. A conversão permite que você atrase a execução
        de uma operação, transforme-a em uma fila, armazene o histórico
        de comandos, envie comandos para serviços remotos, etc.

    -   Por outro lado, o *Strategy* geralmente descreve diferentes
        maneiras de fazer a mesma coisa, permitindo que você troque
        esses algoritmos dentro de uma única classe contexto.

-   O [Decorator](decorator.html) permite que você mude a pele de um
    objeto, enquanto o [Strategy](strategy.html) permite que você mude
    suas entranhas.

-   O [Template Method](template-method.html) é baseado em herança: ele
    permite que você altere partes de um algoritmo ao estender essas
    partes em subclasses. O [Strategy](strategy.html) é baseado em
    composição: você pode alterar partes do comportamento de um objeto
    ao suprir ele como diferentes estratégias que correspondem a aquele
    comportamento. O *Template Method* funciona a nível de classe, então
    é estático. O *Strategy* trabalha a nível de objeto, permitindo que
    você troque os comportamentos durante a execução.

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

[![Strategy em
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](strategy/csharp/example.html "Strategy em C#"){.prog-lang-link}
[![Strategy em
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](strategy/cpp/example.html "Strategy em C++"){.prog-lang-link}
[![Strategy em
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](strategy/go/example.html "Strategy em Go"){.prog-lang-link}
[![Strategy em
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](strategy/java/example.html "Strategy em Java"){.prog-lang-link}
[![Strategy em
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](strategy/php/example.html "Strategy em PHP"){.prog-lang-link}
[![Strategy em
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](strategy/python/example.html "Strategy em Python"){.prog-lang-link}
[![Strategy em
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](strategy/ruby/example.html "Strategy em Ruby"){.prog-lang-link}
[![Strategy em
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](strategy/rust/example.html "Strategy em Rust"){.prog-lang-link}
[![Strategy em
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](strategy/swift/example.html "Strategy em Swift"){.prog-lang-link}
[![Strategy em
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](strategy/typescript/example.html "Strategy em TypeScript"){.prog-lang-link}
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

[Template Method []{.fa .fa-arrow-right}](template-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} State](state.html){.btn .btn-default rel="prev"}
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
