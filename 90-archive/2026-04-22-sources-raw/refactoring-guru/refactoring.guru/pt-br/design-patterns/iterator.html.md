::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
comportamentais](behavioral-patterns.html){.type}
:::

# Iterator {#iterator .title}

::: aka
Também conhecido como: [Iterador]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

O **Iterator** é um padrão de projeto comportamental que permite a você
percorrer elementos de uma coleção sem expor as representações dele
(lista, pilha, árvore, etc.)

<figure class="image">
<img
src="../../images/patterns/content/iterator/iterator-enfc4d.png?id=d19123d71d355d01b0ede4be173eb695"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/iterator/iterator-en.png?id=d19123d71d355d01b0ede4be173eb695 640w,/images/patterns/content/iterator/iterator-en-1.5x.png?id=54aa477cafffe8f9b1b6e63c2e88c21b 960w,/images/patterns/content/iterator/iterator-en-2x.png?id=2a85705e8e5fab257802b2ce36d6d236 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Padrão de projeto Iterator" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Coleções são um dos tipos de dados mais usados em programação. Não
obstante, uma coleção é apenas um contêiner para um grupo de objetos.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/problem14cac.png?id=52ef2fe2d4920e3fed696c221fe757f2"
style="aspect-ratio: 4.9;"
srcset="/images/patterns/diagrams/iterator/problem1.png?id=52ef2fe2d4920e3fed696c221fe757f2 490w,/images/patterns/diagrams/iterator/problem1-1.5x.png?id=b46e39ade7de292fdcacc9790066c72f 735w,/images/patterns/diagrams/iterator/problem1-2x.png?id=1f4384c3c631be238bfc23d6eb84a0ef 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Vários tipos de coleções" />
<figcaption><p>Vários tipos de coleções.</p></figcaption>
</figure>

A maioria das coleções armazena seus elementos em listas simples.
Contudo, alguns deles são baseados em pilhas, árvores, grafos, e outras
estruturas complexas de dados.

Mas independente de quão complexa uma coleção é estruturada, ela deve
fornecer uma maneira de acessar seus elementos para que outro código
possa usá-los. Deve haver uma maneira de ir de elemento em elemento na
coleção sem ter que acessar os mesmos elementos repetidamente.

Isso parece uma tarefa fácil se você tem uma coleção baseada numa lista.
Você faz um loop em todos os elementos. Mas como você faz a travessia
dos elementos de uma estrutura de dados complexas sequencialmente, tais
como uma árvore. Por exemplo, um dia você pode apenas precisar percorrer
em profundidade em uma árvore. No dia seguinte você pode precisar
percorrer na amplitude. E na semana seguinte, você pode precisar algo
diferente, como um acesso aleatório aos elementos da árvore.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/problem29f71.png?id=f9c1a746c787320291c85fdc2a314192"
style="aspect-ratio: 3.75;"
srcset="/images/patterns/diagrams/iterator/problem2.png?id=f9c1a746c787320291c85fdc2a314192 600w,/images/patterns/diagrams/iterator/problem2-1.5x.png?id=7a003d020e789773e0c833d7b1df00e6 900w,/images/patterns/diagrams/iterator/problem2-2x.png?id=656b13c109d4d4960ea1f9fa993bae4c 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Vários algoritmos de travessia" />
<figcaption><p>A mesma coleção pode ser atravessada de
diferentes formas.</p></figcaption>
</figure>

Adicionando mais e mais algoritmos de travessia para uma coleção
gradualmente desfoca sua responsabilidade primária, que é um
armazenamento de dados eficiente. Além disso, alguns algoritmos podem
ser feitos para aplicações específicas, então incluí-los em uma coleção
genérica pode ser estranho.

Por outro lado, o código cliente que deveria trabalhar com várias
coleções pode não se importar com a maneira que elas armazenam seus
elementos. Contudo, uma vez que todas as coleções fornecem diferentes
maneiras de acessar seus elementos, você não tem outra opção além de
acoplar seu código com as classes de coleções específicas.
:::

::: {.section .solution}
##  Solução {#solution}

A ideia principal do padrão Iterator é extrair o comportamento de
travessia de uma coleção para um objeto separado chamado um *iterador*.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/solution19739.png?id=2f5fbcce6099d8ea09b2fbb83e3e7059"
style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/iterator/solution1.png?id=2f5fbcce6099d8ea09b2fbb83e3e7059 400w,/images/patterns/diagrams/iterator/solution1-1.5x.png?id=68612372ede6e5029403d38b381fdc05 600w,/images/patterns/diagrams/iterator/solution1-2x.png?id=dcebcad4c197bfa5f25f680382d0e5ac 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="Iterators implementam vários algoritmos de travessia" />
<figcaption><p>Iteradores implementam vários algoritmos de travessia.
Alguns objetos iterador podem fazer a travessia da mesma coleção ao
mesmo tempo.</p></figcaption>
</figure>

Além de implementar o algoritmo em si, um objeto iterador encapsula
todos os detalhes da travessia, tais como a posição atual e quantos
elementos faltam para chegar ao fim. Por causa disso, alguns iteradores
podem averiguar a mesma coleção ao mesmo tempo, independentemente um do
outro.

Geralmente, os iteradores fornecem um método primário para pegar
elementos de uma coleção. O cliente pode manter esse método funcionando
até que ele não retorne mais nada, o que significa que o iterador
atravessou todos os elementos.

Todos os iteradores devem implementar a mesma interface. Isso faz que o
código cliente seja compatível com qualquer tipo de coleção ou qualquer
algoritmo de travessia desde que haja um iterador apropriado. Se você
precisar de uma maneira especial para a travessia de uma coleção, você
só precisa criar uma nova classe iterador, sem ter que mudar a coleção
ou o cliente.
:::

::: {.section .analogy}
##  Analogia com o mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/iterator/iterator-comic-1-pt-br1216.png?id=5728f0044efc656229468032ba0f6833"
style="aspect-ratio: 2.13;"
srcset="/images/patterns/content/iterator/iterator-comic-1-pt-br.png?id=5728f0044efc656229468032ba0f6833 640w,/images/patterns/content/iterator/iterator-comic-1-pt-br-1.5x.png?id=8f2cb8f6468e223bc9113740780bd5dc 960w,/images/patterns/content/iterator/iterator-comic-1-pt-br-2x.png?id=8e100b67ee0fa6d311f9505e5cb0e229 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Várias maneiras de se caminhar por Roma" />
<figcaption><p>Várias maneiras de se caminhar por Roma.</p></figcaption>
</figure>

Você planeja visitar Roma por alguns dias e visitar todas suas
principais atrações e pontos de interesse. Mas uma vez lá, você poderia
gastar muito tempo andando em círculos, incapaz de achar até mesmo o
Coliseu.

Por outro lado, você pode comprar um guia virtual na forma de app para
seu smartphone e usá-lo como navegação. É inteligente e barato, e você
pode ficar em lugares interessantes por quanto tempo quiser.

Uma terceira alternativa é gastar um pouco da verba da viagem e
contratar um guia local que conhece a cidade como a palma de sua mão. O
guia poderia ser capaz de criar um passeio que se adeque a seus gostos,
mostrar todas as atrações, e contar um monte de histórias interessantes.
Isso seria mais divertido, mas, infelizmente, mais caro também.

Todas essas opções---direções aleatórias criadas em sua cabeça, o
navegador do smartphone, ou o guia humano---agem como iteradores sobre a
vasta coleção de locais e atrações de Roma.
:::

::::: {.section .structure-container}
##  Estrutura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/structure5247.png?id=35ea851f8f6bbe51d79eb91e6e6519d0"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.12;"
srcset="/images/patterns/diagrams/iterator/structure.png?id=35ea851f8f6bbe51d79eb91e6e6519d0 480w,/images/patterns/diagrams/iterator/structure-1.5x.png?id=19d4c2130ab2e3bd718f87e956fcd873 720w,/images/patterns/diagrams/iterator/structure-2x.png?id=b7b4708d3f279dd046eb1692f1cba710 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Estrutura do padrão de projeto Iterator" /><img
src="../../images/patterns/diagrams/iterator/structure-indexed3472.png?id=7bc28907ff6b480db6635a93ebaa10ff"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.33;"
srcset="/images/patterns/diagrams/iterator/structure-indexed.png?id=7bc28907ff6b480db6635a93ebaa10ff 520w,/images/patterns/diagrams/iterator/structure-indexed-1.5x.png?id=32fde4c4c1cbfbe07aa6e716e5ac2346 780w,/images/patterns/diagrams/iterator/structure-indexed-2x.png?id=d809428b2262e2013972fe69d2434473 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Estrutura do padrão de projeto Iterator" />
</figure>
:::

1.  A interface **Iterador** declara as operações necessárias para
    percorrer uma coleção: buscar o próximo elemento, pegar a posição
    atual, recomeçar a iteração, etc.

2.  **Iteradores Concretos** implementam algoritmos específicos para
    percorrer uma coleção. O objeto iterador deve monitorar o progresso
    da travessia por conta própria. Isso permite que diversos iteradores
    percorram a mesma coleção independentemente de cada um.

3.  A interface **Coleção** declara um ou mais métodos para obter os
    iteradores compatíveis com a coleção. Observe que o tipo do retorno
    dos métodos deve ser declarado como a interface do iterador para que
    as coleções concretas possam retornar vários tipos de iteradores.

4.  **Coleções Concretas** retornam novas instâncias de uma classe
    iterador concreta em particular cada vez que o cliente pede por uma.
    Você pode estar se perguntando, onde está o resto do código da
    coleção? Não se preocupe, ele deve ficar na mesma classe. É que
    esses detalhes não são cruciais para o padrão atual, então optamos
    por omiti-los.

5.  O **Cliente** trabalha tanto com as coleções como os iteradores
    através de suas interfaces. Dessa forma o cliente não fica acoplado
    a suas classes concretas, permitindo que você use várias coleções e
    iteradores com o mesmo código cliente.

    Tipicamente, os clientes não criam iteradores por conta própria, mas
    ao invés disso os obtêm das coleções. Ainda assim, em certos casos,
    o cliente pode criar um diretamente; por exemplo, quando o cliente
    define seu próprio iterador especial.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Neste exemplo, o padrão **Iterator** é usado para percorrer uma coleção
especial que encapsula acesso ao grafo social do Facebook. A coleção
fornece vários iteradores que podem percorrer perfis de várias maneiras.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/examplee0a2.png?id=f2a24ef3787bf80ed450709240506ff2"
style="aspect-ratio: 0.95;"
srcset="/images/patterns/diagrams/iterator/example.png?id=f2a24ef3787bf80ed450709240506ff2 600w,/images/patterns/diagrams/iterator/example-1.5x.png?id=cab0e1459ffc3721579a3e7d6a4e9022 900w,/images/patterns/diagrams/iterator/example-2x.png?id=73c3e55f75ca0250bd84e8a1d8cc88d2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Exemplo da estrutura do padrão Iterator" />
<figcaption><p>Exemplo de uma iteração sobre
perfis sociais.</p></figcaption>
</figure>

O iterador 'amigos' pode ser usado para verificar os amigos de um dado
perfil. O iterador \'colegas' faz a mesma coisa, exceto por omitir
amigos que não trabalham na mesma companhia como uma pessoa alvo. Ambos
iteradores implementam uma interface comum que permite os clientes
recuperar os perfis sem mergulhar nos detalhes de implementação como
autenticação e pedidos REST.

O código cliente não está acoplado à classes concretas porque funciona
com coleções e iteradores somente através de interfaces. Se você decidir
conectar sua aplicação com uma nova rede social, você simplesmente
precisa fornecer as novas classes de iteração e coleção sem mudar o
código existente.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// A interface da coleção deve declarar um método fábrica para
// produzir iteradores. Você pode declarar vários métodos se há
// diferentes tipos de iteração disponíveis em seu programa.
interface SocialNetwork is
    method createFriendsIterator(profileId):ProfileIterator
    method createCoworkersIterator(profileId):ProfileIterator


// Cada coleção concreta é acoplada a um conjunto de classes
// iterador concretas que ela retorna. Mas o cliente não é, uma
// vez que a assinatura desses métodos retorna interfaces de
// iterador.
class Facebook implements SocialNetwork is
    // ...o grosso do código da coleção deve vir aqui...

    // Código de criação do iterador.
    method createFriendsIterator(profileId) is
        return new FacebookIterator(this, profileId, &quot;friends&quot;)
    method createCoworkersIterator(profileId) is
        return new FacebookIterator(this, profileId, &quot;coworkers&quot;)


// A interface comum a todos os iteradores.
interface ProfileIterator is
    method getNext():Profile
    method hasMore():bool


// A classe iterador concreta.
class FacebookIterator implements ProfileIterator is
    // O iterador precisa de uma referência para a coleção que
    // ele percorre.
    private field facebook: Facebook
    private field profileId, type: string

    // Um objeto iterador percorre a coleção independentemente
    // de outros iteradores. Portanto ele tem que armazenar o
    // estado de iteração.
    private field currentPosition
    private field cache: array of Profile

    constructor FacebookIterator(facebook, profileId, type) is
        this.facebook = facebook
        this.profileId = profileId
        this.type = type

    private method lazyInit() is
        if (cache == null)
            cache = facebook.socialGraphRequest(profileId, type)

    // Cada classe iterador concreta tem sua própria
    // implementação da interface comum do iterador.
    method getNext() is
        if (hasMore())
            result = cache[currentPosition]
            currentPosition++
            return result

    method hasMore() is
        lazyInit()
        return currentPosition &lt; cache.length


// Aqui temos outro truque útil: você pode passar um iterador
// para uma classe cliente ao invés de dar acesso a ela à uma
// coleção completa. Dessa forma, você não expõe a coleção ao
// cliente.
//
// E tem outro benefício: você pode mudar o modo que o cliente
// trabalha com a coleção no tempo de execução ao passar a ele
// um iterador diferente. Isso é possível porque o código
// cliente não é acoplado às classes iterador concretas.
class SocialSpammer is
    method send(iterator: ProfileIterator, message: string) is
        while (iterator.hasMore())
            profile = iterator.getNext()
            System.sendEmail(profile.getEmail(), message)


// A classe da aplicação configura coleções e iteradores e então
// os passa ao código cliente.
class Application is
    field network: SocialNetwork
    field spammer: SocialSpammer

    method config() is
        if working with Facebook
            this.network = new Facebook()
        if working with LinkedIn
            this.network = new LinkedIn()
        this.spammer = new SocialSpammer()

    method sendSpamToFriends(profile) is
        iterator = network.createFriendsIterator(profile.getId())
        spammer.send(iterator, &quot;Very important message&quot;)

    method sendSpamToCoworkers(profile) is
        iterator = network.createCoworkersIterator(profile.getId())
        spammer.send(iterator, &quot;Very important message&quot;)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Aplicabilidade {#applicability}

::::::::: applicability
::: applicability-problem
Utilize o padrão Iterator quando sua coleção tiver uma estrutura de
dados complexa por debaixo dos panos, mas você quer esconder a
complexidade dela de seus clientes (seja por motivos de conveniência ou
segurança).
:::

::: applicability-solution
O iterador encapsula os detalhes de se trabalhar com uma estrutura de
dados complexa, fornecendo ao cliente vários métodos simples para
acessar os elementos da coleção. Embora essa abordagem seja muito
conveniente para o cliente, ela também protege a coleção de ações
descuidadas ou maliciosas que o cliente poderia fazer se estivesse
trabalhando com as coleções diretamente.
:::

::: applicability-problem
Utilize o padrão para reduzir a duplicação de código de travessia em sua
aplicação.
:::

::: applicability-solution
O código de algoritmos de iteração não triviais tendem a ser muito
pesados. Quando colocados dentro da lógica de negócio da aplicação, ele
pode desfocar a responsabilidade do codigo original e torná-lo um código
de difícil manutenção. Mover o código de travessia para os iteradores
designados pode ajudar você a tornar o código da aplicação mais enxuto e
limpo.
:::

::: applicability-problem
Utilize o Iterator quando você quer que seu código seja capaz de
percorrer diferentes estruturas de dados ou quando os tipos dessas
estruturas são desconhecidos de antemão.
:::

::: applicability-solution
O padrão fornece um par de interfaces genérica tanto para coleções como
para iteradores. Já que seu código agora usa essas interfaces, ele ainda
vai funcionar se você passar diversos tipos de coleções e iteradores que
implementam essas interfaces.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Como implementar {#checklist}

1.  Declare a interface do iterador. Ao mínimo, ela deve ter um método
    para buscar o próximo elemento de uma coleção. Mas por motivos de
    conveniência você pode adicionar alguns outros métodos, tais como
    recuperar o elemento anterior, saber a posição atual, e checar o fim
    da iteração.

2.  Declare a interface da coleção e descreva um método para buscar
    iteradores. O tipo de retorno deve ser igual à interface do
    iterador. Você pode declarar métodos parecidos se você planeja ter
    grupos distintos de iteradores.

3.  Implemente classes iterador concretas para as coleções que você quer
    percorrer com iteradores. Um objeto iterador deve ser ligado com uma
    única instância de coleção. Geralmente esse link é estabelecido
    através do construtor do iterador.

4.  Implemente a interface da coleção na suas classes de coleção. A
    ideia principal é fornecer ao cliente com um atalho para criar
    iteradores, customizados para uma classe coleção em particular. O
    objeto da coleção deve passar a si mesmo para o construtor do
    iterador para estabelecer um link entre eles.

5.  Vá até o código cliente e substitua todo o código de travessia de
    coleção com pelo uso de iteradores. O cliente busca um novo objeto
    iterador a cada vez que precisa iterar sobre os elementos de uma
    coleção.
:::

:::::: {.section .pros-cons}
##  Prós e contras {#pros-cons}

::::: row
::: col-sm-6
-    *Princípio de responsabilidade única*. Você pode limpar o código
    cliente e as coleções ao extrair os pesados algoritmos de travessia
    para classes separadas.
-    *Princípio aberto/fechado*. Você pode implementar novos tipos de
    coleções e iteradores e passá-los para o código existente sem
    quebrar coisa alguma.
-    Você pode iterar sobre a mesma coleção em paralelo porque cada
    objeto iterador contém seu próprio estado de iteração.
-    Pelas mesmas razões, você pode atrasar uma iteração e continuá-la
    quando necessário.
:::

::: col-sm-6
-    Aplicar o padrão pode ser um preciosismo se sua aplicação só
    trabalha com coleções simples.
-    Usar um iterador pode ser menos eficiente que percorrer elementos
    de algumas coleções especializadas diretamente.
:::
:::::
::::::

::: {.section .relations}
##  Relações com outros padrões {#relations}

-   Você pode usar [Iteradores](iterator.html) para percorrer árvores
    [Composite](composite.html).

-   Você pode usar o [Factory Method](factory-method.html) junto com o
    [Iterator](iterator.html) para permitir que uma coleção de
    subclasses retornem diferentes tipos de iteradores que são
    compatíveis com as coleções.

-   Você pode usar o [Memento](memento.html) junto com o
    [Iterator](iterator.html) para capturar o estado de iteração atual e
    revertê-lo se necessário.

-   Você pode usar o [Visitor](visitor.html) junto com o
    [Iterator](iterator.html) para percorrer uma estrutura de dados
    complexas e executar alguma operação sobre seus elementos, mesmo se
    eles todos tenham classes diferentes.
:::

::: {.section .implementations}
##  Exemplos de código {#implementations}

[![Iterator em
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](iterator/csharp/example.html "Iterator em C#"){.prog-lang-link}
[![Iterator em
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](iterator/cpp/example.html "Iterator em C++"){.prog-lang-link}
[![Iterator em
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](iterator/go/example.html "Iterator em Go"){.prog-lang-link}
[![Iterator em
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](iterator/java/example.html "Iterator em Java"){.prog-lang-link}
[![Iterator em
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](iterator/php/example.html "Iterator em PHP"){.prog-lang-link}
[![Iterator em
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](iterator/python/example.html "Iterator em Python"){.prog-lang-link}
[![Iterator em
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](iterator/ruby/example.html "Iterator em Ruby"){.prog-lang-link}
[![Iterator em
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](iterator/rust/example.html "Iterator em Rust"){.prog-lang-link}
[![Iterator em
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](iterator/swift/example.html "Iterator em Swift"){.prog-lang-link}
[![Iterator em
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](iterator/typescript/example.html "Iterator em TypeScript"){.prog-lang-link}
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

[Mediator []{.fa .fa-arrow-right}](mediator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Command](command.html){.btn .btn-default
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
