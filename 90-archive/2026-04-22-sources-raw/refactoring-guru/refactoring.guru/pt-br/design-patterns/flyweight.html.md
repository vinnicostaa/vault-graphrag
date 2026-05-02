:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
estruturais](structural-patterns.html){.type}
:::

# Flyweight {#flyweight .title}

::: aka
Também conhecido como: [Peso
mosca, ]{style="display:inline-block"}[Cache]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

O **Flyweight** é um padrão de projeto estrutural que permite a você
colocar mais objetos na quantidade de RAM disponível ao compartilhar
partes comuns de estado entre os múltiplos objetos ao invés de manter
todos os dados em cada objeto.

<figure class="image">
<img
src="../../images/patterns/content/flyweight/flyweight7be2.png?id=e34fbacb847dd609b5e68aaf252c4db0"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/flyweight/flyweight.png?id=e34fbacb847dd609b5e68aaf252c4db0 640w,/images/patterns/content/flyweight/flyweight-1.5x.png?id=b32df2297473b8a7577e1900e57325ac 960w,/images/patterns/content/flyweight/flyweight-2x.png?id=6a8f17d9550c75c3d648a605c4d31b45 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Padrão de projeto Flyweight" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Para se divertir após longas horas de trabalho você decide criar um jogo
simples: os jogadores estarão se movendo em um mapa e atirado uns aos
outros. Você escolhe implementar um sistema de partículas realístico e
faz dele uma funcionalidade distinta do jogo. Uma grande quantidades de
balas, mísseis, e estilhaços de explosões devem voar por todo o mapa e
entregar adrenalina para o jogador.

Ao completar, você sobe suas últimas mudanças, constrói o jogo e manda
ele para um amigo para um test drive. Embora o jogo tenha rodado
impecavelmente na sua máquina, seu amigo não foi capaz de jogar por
muito tempo. No computador dele, o jogo continuava quebrando após alguns
minutos de gameplay. Após algumas horas pesquisando nos registros do
jogo você descobre que ele quebrou devido a uma quantidade insuficiente
de RAM. Acontece que a máquina do seu amigo é muito menos poderosa que o
seu computador, e é por isso que o problema apareceu facilmente na
máquina dele.

O verdadeiro problema está relacionado ao seu sistema de partículas.
Cada partícula, tais como uma bala, um míssil, ou um estilhaço era
representado por um objeto separado contendo muita informação. Em algum
momento, quando a destruição na tela do jogadora era tanta, as novas
partículas criadas não cabiam mais no RAM restante, então o programa
quebrava.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/problem-pt-br8125.png?id=6cfa1ce50e6af299e9c7e761813fda07"
style="aspect-ratio: 2.54;"
srcset="/images/patterns/diagrams/flyweight/problem-pt-br.png?id=6cfa1ce50e6af299e9c7e761813fda07 660w,/images/patterns/diagrams/flyweight/problem-pt-br-1.5x.png?id=65ad901689841371ce9f8f66f200210b 990w,/images/patterns/diagrams/flyweight/problem-pt-br-2x.png?id=60c62ec6e5911201a0bcfbf3e42bc3cd 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="Problema do padrão Flyweight" />
</figure>
:::

::: {.section .solution}
##  Solução {#solution}

Olhando de perto a classe `Partícula`, você pode notar que a cor e o
campo sprite consomem muita memória se comparado aos demais campos. E o
pior é que esses dois campos armazenam dados quase idênticos para todas
as partículas. Por exemplo, todas as balas têm a mesma cor e sprite.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution1-pt-brd23c.png?id=b473e0d7b210adeb3d5432e373489321"
style="aspect-ratio: 2.06;"
srcset="/images/patterns/diagrams/flyweight/solution1-pt-br.png?id=b473e0d7b210adeb3d5432e373489321 640w,/images/patterns/diagrams/flyweight/solution1-pt-br-1.5x.png?id=683e5fab96ba47eb9bad8ad767f4d0ac 960w,/images/patterns/diagrams/flyweight/solution1-pt-br-2x.png?id=bfa98655ca7381482a68634359cf5163 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Solução do padrão Flyweight" />
</figure>

Outras partes do estado de uma partícula, tais como coordenadas, vetor
de movimento e velocidade, são únicos para cada partícula. Afinal de
contas, todos os valores desses campos mudam com o tempo. Esses dados
representam todo o contexto de mudança na qual a partícula existe,
enquanto que a cor e o sprite permanecem constante para cada partícula.

Esse dado constante de um objeto é usualmente chamado de *estado
intrínseco*. Ele vive dentro do objeto; outros objetos só podem lê-lo,
não mudá-lo. O resto do estado do objeto, quase sempre alterado "pelo
lado de fora" por outros objetos é chamado *estado extrínseco*.

O padrão Flyweight sugere que você pare de armazenar o estado extrínseco
dentro do objeto. Ao invés disso, você deve passar esse estado para
métodos específicos que dependem dele. Somente o estado intrínseco fica
dentro do objeto, permitindo que você o reutilize em diferentes
contextos. Como resultado, você vai precisar de menos desses objetos uma
vez que eles diferem apenas em seu estado intrínseco, que tem menos
variações que o extrínseco.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution3-pt-br3c86.png?id=6cb98174409f3c9e4356288399e5c457"
style="aspect-ratio: 0.76;"
srcset="/images/patterns/diagrams/flyweight/solution3-pt-br.png?id=6cb98174409f3c9e4356288399e5c457 424w,/images/patterns/diagrams/flyweight/solution3-pt-br-1.5x.png?id=1651ee2ef4cf7b234e4e93005e15c65c 636w,/images/patterns/diagrams/flyweight/solution3-pt-br-2x.png?id=484ae7afce740a2644ca1ab75e0cab4d 848w"
sizes="(max-width: 720px) 100vw, 424px" loading="lazy" width="424"
alt="Solução do padrão Flyweight" />
</figure>

Vamos voltar ao nosso jogo. Assumindo que extraímos o estado extrínseco
de nossa classe de partículas, somente três diferentes objetos serão
suficientes para representar todas as partículas no jogo: uma bala, um
míssil, e um pedaço de estilhaço. Como você provavelmente já adivinhou,
um objeto que apenas armazena o estado intrínseco é chamado de um
flyweight.

#### Armazenamento do estado extrínseco

Para onde vai o estado extrínseco então? Algumas classes ainda devem ser
capazes de armazená-lo, certo? Na maioria dos casos, ele é movido para o
objeto contêiner, que agrega os objetos antes de aplicarmos o padrão.

No nosso caso, esse seria o objeto principal `Jogo` que armazena todas
as partículas no campo `partículas`. Para mover o estado extrínseco para
essa classe você precisa criar diversos campos array para armazenar
coordenadas, vetores, e a velocidade de cada partícula individual. Mas
isso não é tudo. Você vai precisar de outra array para armazenar
referências ao flyweight específico que representa a partícula. Essas
arrays devem estar em sincronia para que você possa acessar todos os
dados de uma partícula usando o mesmo índice.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution2-pt-br00b3.png?id=2138f9da653564e8baf8fdab5fe2c9e0"
style="aspect-ratio: 1.94;"
srcset="/images/patterns/diagrams/flyweight/solution2-pt-br.png?id=2138f9da653564e8baf8fdab5fe2c9e0 640w,/images/patterns/diagrams/flyweight/solution2-pt-br-1.5x.png?id=1b5b34681cc01fd1d1ded552d81d368c 960w,/images/patterns/diagrams/flyweight/solution2-pt-br-2x.png?id=40718a9d82a43ffd7403aefccb2d0caf 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Solução padrão flyweight" />
</figure>

Uma solução mais elegante é criar uma classe de contexto separada que
armazenaria o estado extrínseco junto com a referência para o objeto
flyweight. Essa abordagem precisaria apenas de uma única array na classe
contêiner.

Calma aí! Não vamos precisar de tantos objetos contextuais como tínhamos
no começo? Tecnicamente, sim, mas nesse caso, esses objetos são muito
menores que antes. Os campos mais pesados foram movidos para alguns
poucos objetos flyweight. Agora, milhares de objetos contextuais podem
reutilizar um único objeto flyweight pesado ao invés de armazenar
milhares de cópias de seus dados.

#### Flyweight e a imutabilidade

Já que o mesmo objeto flyweight pode ser usado em diferentes contextos,
você tem que certificar-se que seu estado não pode ser modificado. Um
flyweight deve inicializar seu estado apenas uma vez, através dos
parâmetros do construtor. Ele não deve expor qualquer setter ou campos
públicos para outros objetos.

#### Fábrica Flyweight

Para um acesso mais conveniente para vários flyweights, você pode criar
um método fábrica que gerencia um conjunto de objetos flyweight
existentes. O método aceita o estado intrínseco do flyweight desejado
por um cliente, procura por um objeto flyweight existente que coincide
com esse estado, e retorna ele se for encontrado. Se não for, ele cria
um novo flyweight e o adiciona ao conjunto.

Há várias opções onde esse método pode ser colocado. O lugar mais óbvio
é um contêiner de flyweights. Alternativamente você pode criar uma nova
classe fábrica. Ou você pode fazer o método fábrica ser estático e
colocá-lo dentro da própria classe do flyweight.
:::

::::: {.section .structure-container}
##  Estrutura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/structure7b00.png?id=c1e7e1748f957a4792822f902bc1d420"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/flyweight/structure.png?id=c1e7e1748f957a4792822f902bc1d420 640w,/images/patterns/diagrams/flyweight/structure-1.5x.png?id=34cf08f6c2b09dcd1c521c7512cc52b6 960w,/images/patterns/diagrams/flyweight/structure-2x.png?id=a7c8347421bde16435fc90a706f5dd34 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Estrutura do padrão de projeto Flyweight" /><img
src="../../images/patterns/diagrams/flyweight/structure-indexed086a.png?id=aa490792baa26b04464dacbc1f4a19cd"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.54;"
srcset="/images/patterns/diagrams/flyweight/structure-indexed.png?id=aa490792baa26b04464dacbc1f4a19cd 630w,/images/patterns/diagrams/flyweight/structure-indexed-1.5x.png?id=b22a5eab6aacbbd03c66c34802b240ca 945w,/images/patterns/diagrams/flyweight/structure-indexed-2x.png?id=205e2f7d08b4ac0695f445a9db8989c4 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="Estrutura do padrão de projeto Flyweight" />
</figure>
:::

1.  O padrão Flyweight é somente uma optimização. Antes de aplicá-lo,
    certifique-se que seu programa tem mesmo um problema de consumo de
    RAM relacionado a existência de múltiplos objetos similares na
    memória ao mesmo tempo. Certifique-se que o problema não possa ser
    resolvido por outra forma relevante.

2.  A classe **Flyweight** contém a porção do estado do objeto original
    que pode ser compartilhada entre múltiplos objetos. O mesmo objeto
    flyweight pode ser usado em muitos contextos diferentes. O estado
    armazenado dentro de um flyweight é chamado "intrínseco". O estado
    passado pelos métodos flyweight é chamado "extrínseco".

3.  A classe **Contexto** contém o estado extrínseco, único para todos
    os objetos originais. Quando um contexto é pareado com um dos
    objetos flyweight, ele representa o estado completo do objeto
    original.

4.  Geralmente, o comportamento do objeto original permanece na classe
    flyweight. Nesse caso, quem chamar o método do flyweight deve também
    passar os dados apropriados do estado extrínseco nos parâmetros do
    método. Por outro lado, o comportamento pode ser movido para a
    classe contexto, que usaria o flyweight meramente como um objeto de
    dados.

5.  O **Cliente** calcula ou armazena o estado extrínseco dos
    flyweights. Da perspectiva do cliente, um flyweight é um objeto
    modelo que pode ser configurado no momento da execução ao passar
    alguns dados de contexto nos parâmetros de seus métodos.

6.  A **Fábrica Flyweight** gerencia um conjunto de flyweights
    existentes. Com a fábrica os clientes não precisam criar flyweights
    diretamente. Ao invés disso, eles chamam a fábrica, passam os dados
    de estado intrínseco para o flyweight desejado. A fábrica procura
    por flyweights já criados e então retorna um existe que coincide o
    critério de busca ou cria um novo se nada for encontrado.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Neste exemplo o padrão **Flyweight** ajuda a reduzir o uso de memória
quando renderizando milhões de objetos árvore em uma tela.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/example5053.png?id=0818d078c1a79f373e96397f37b7ee06"
style="aspect-ratio: 1.54;"
srcset="/images/patterns/diagrams/flyweight/example.png?id=0818d078c1a79f373e96397f37b7ee06 540w,/images/patterns/diagrams/flyweight/example-1.5x.png?id=449fa5906e277c134870c07e33dd4b62 810w,/images/patterns/diagrams/flyweight/example-2x.png?id=9423640fe3688a64201389b6e7aa1f48 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Exemplo do padrão Flyweight" />
</figure>

O padrão extrai o estado intrínseco repetido para uma classe `Árvore`
principal e o move para dentro da classe flyweight `TipoÁrvore`.

Agora ao invés de armazenar os mesmos dados em múltiplos objetos, ele
armazena apenas alguns objetos flyweight e os liga aos objetos `Árvore`
apropriados que agem como contexto. O código cliente cria novos objetos
árvore usando a fábrica flyweight, que encapsula a complexidade de busca
pelo objeto correto e o reutiliza se necessário.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// A classe flyweight contém uma parte do estado de uma árvore
// Esses campos armazenam valores que são únicos para cada
// árvore em particular. Por exemplo, você não vai encontrar
// coordenadas da árvore aqui. Já que esses dados geralmente são
// GRANDES, você gastaria muita memória mantendo-os em cada
// objeto árvore. Ao invés disso, nós podemos extrair a textura,
// cor e outros dados repetitivos em um objeto separado os quais
// muitas árvores individuais podem referenciar.
class TreeType is
    field name
    field color
    field texture
    constructor TreeType(name, color, texture) { ... }
    method draw(canvas, x, y) is
        // 1. Cria um bitmap de certo tipo, cor e textura.
        // 2. Desenha o bitmap em uma tela nas coordenadas X e
        // Y.

// A fábrica flyweight decide se reutiliza o flyweight existente
// ou cria um novo objeto.
class TreeFactory is
    static field treeTypes: collection of tree types
    static method getTreeType(name, color, texture) is
        type = treeTypes.find(name, color, texture)
        if (type == null)
            type = new TreeType(name, color, texture)
            treeTypes.add(type)
        return type

// O objeto contextual contém a parte extrínseca do estado da
// árvore. Uma aplicação pode criar bilhões desses estados, já
// que são muito pequenos:
// apenas dois números inteiros para coordenadas e um campo de
// referência.
class Tree is
    field x,y
    field type: TreeType
    constructor Tree(x, y, type) { ... }
    method draw(canvas) is
        type.draw(canvas, this.x, this.y)

// As classes Tree (Árvore) e Forest (Floresta) são os clientes
// flyweight. Você pode uni-las se não planeja desenvolver mais
// a classe Tree.
class Forest is
    field trees: collection of Trees

    method plantTree(x, y, name, color, texture) is
        type = TreeFactory.getTreeType(name, color, texture)
        tree = new Tree(x, y, type)
        trees.add(tree)

    method draw(canvas) is
        foreach (tree in trees) do
            tree.draw(canvas)</code></pre>
</figure>
:::

:::::: {.section .applicability-container}
##  Aplicabilidade {#applicability}

::::: applicability
::: applicability-problem
Utilize o padrão Flyweight apenas quando seu programa deve suportar um
grande número de objetos que mal cabem na RAM disponível.
:::

::: applicability-solution
O benefício de aplicar o padrão depende muito de como e onde ele é
usado. Ele é mais útil quando:

-   uma aplicação precisa gerar um grande número de objetos similares
-   isso drena a RAM disponível no dispositivo alvo
-   os objetos contém estados duplicados que podem ser extraídos e
    compartilhados entre múltiplos objetos
:::
:::::
::::::

::: {.section .checklist}
##  Como implementar {#checklist}

1.  Divida os campos de uma classe que irá se tornar um flyweight em
    duas partes:

    -   o estado intrínseco: os campos que contém dados imutáveis e
        duplicados para muitos objetos
    -   o estado extrínseco: os campos que contém dados contextuais
        únicos para cada objeto

2.  Deixe os campos que representam o estado intrínseco dentro da
    classe, mas certifique-se que eles sejam imutáveis. Eles só podem
    obter seus valores iniciais dentro do construtor.

3.  Examine os métodos que usam os campos do estado extrínseco. Para
    cada campo usado no método, introduza um novo parâmetro e use-o ao
    invés do campo.

4.  Opcionalmente, crie uma classe fábrica para gerenciar o conjunto de
    flyweights. Ela deve checar por um flyweight existente antes de
    criar um novo. Uma vez que a fábrica está rodando, os clientes devem
    pedir flyweights apenas através dela. Eles devem descrever o
    flyweight desejado ao passar o estado intrínseco para a fábrica.

5.  O cliente deve armazenar ou calcular valores para o estado
    extrínseco (contexto) para ser capaz de chamar métodos de objetos
    flyweight. Por conveniência, o estado extrínseco junto com o campo
    de referência flyweight podem ser movidos para uma classe de
    contexto separada.
:::

:::::: {.section .pros-cons}
##  Prós e contras {#pros-cons}

::::: row
::: col-sm-6
-    Você pode economizar muita RAM, desde que seu programa tenha muitos
    objetos similares.
:::

::: col-sm-6
-    Você pode estar trocando RAM por ciclos de CPU quando parte dos
    dados de contexto precisa ser recalculado cada vez que alguém chama
    um método flyweight.
-    O código fica muito mais complicado. Novos membros de equipe sempre
    se perguntarão por que o estado de uma entidade foi separado de tal
    forma.
:::
:::::
::::::

::: {.section .relations}
##  Relações com outros padrões {#relations}

-   Você pode implementar nós folha compartilhados da árvore
    [Composite](composite.html) como [Flyweights](flyweight.html) para
    salvar RAM.

-   O [Flyweight](flyweight.html) mostra como fazer vários pequenos
    objetos, enquanto o [Facade](facade.html) mostra como fazer um único
    objeto que represente um subsistema inteiro.

-   O [Flyweight](flyweight.html) seria parecido com o
    [Singleton](singleton.html) se você, de algum modo, reduzisse todos
    os estados de objetos compartilhados para apenas um objeto
    flyweight. Mas há duas mudanças fundamentais entre esses padrões:

    1.  Deve haver apenas uma única instância singleton, enquanto que
        uma classe *flyweight* pode ter múltiplas instâncias com
        diferentes estados intrínsecos.
    2.  O objeto *singleton* pode ser mutável. Objetos flyweight são
        imutáveis.
:::

::: {.section .implementations}
##  Exemplos de código {#implementations}

[![Flyweight em
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](flyweight/csharp/example.html "Flyweight em C#"){.prog-lang-link}
[![Flyweight em
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](flyweight/cpp/example.html "Flyweight em C++"){.prog-lang-link}
[![Flyweight em
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](flyweight/go/example.html "Flyweight em Go"){.prog-lang-link}
[![Flyweight em
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](flyweight/java/example.html "Flyweight em Java"){.prog-lang-link}
[![Flyweight em
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](flyweight/php/example.html "Flyweight em PHP"){.prog-lang-link}
[![Flyweight em
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](flyweight/python/example.html "Flyweight em Python"){.prog-lang-link}
[![Flyweight em
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](flyweight/ruby/example.html "Flyweight em Ruby"){.prog-lang-link}
[![Flyweight em
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](flyweight/rust/example.html "Flyweight em Rust"){.prog-lang-link}
[![Flyweight em
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](flyweight/swift/example.html "Flyweight em Swift"){.prog-lang-link}
[![Flyweight em
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](flyweight/typescript/example.html "Flyweight em TypeScript"){.prog-lang-link}
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

[Proxy []{.fa .fa-arrow-right}](proxy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Facade](facade.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::
