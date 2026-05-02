::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
comportamentais](behavioral-patterns.html){.type}
:::

# Template Method {#template-method .title}

::: aka
Também conhecido como: [Método padrão]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

O **Template Method** é um padrão de projeto comportamental que define o
esqueleto de um algoritmo na superclasse mas deixa as subclasses
sobrescreverem etapas específicas do algoritmo sem modificar
sua estrutura.

<figure class="image">
<img
src="../../images/patterns/content/template-method/template-method4c1b.png?id=eee9461742f832814f19612ccf472819"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/template-method/template-method.png?id=eee9461742f832814f19612ccf472819 640w,/images/patterns/content/template-method/template-method-1.5x.png?id=2b4c179aaa02f5c45a59324b904cd130 960w,/images/patterns/content/template-method/template-method-2x.png?id=4e164dc41be4dcfa122628864c2be210 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Padrão Template Method" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagine que você está criando uma aplicação de mineração de dados que
analisa documentos corporativos. Os usuários alimentam a aplicação com
documentos em vários formatos (PDF, DOC, CSV), e ela tenta extrair dados
significativos desses documentos para um formato uniforme.

A primeira versão da aplicação podia funcionar somente com arquivos DOC.
Na versão seguinte, ela era capaz de suportar arquivos CSV. Um mês
depois, você a "ensinou" a extrair dados de arquivos PDF.

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/problem6793.png?id=60fa4f735c467ac1c9438231a1782807"
style="aspect-ratio: 1.35;"
srcset="/images/patterns/diagrams/template-method/problem.png?id=60fa4f735c467ac1c9438231a1782807 620w,/images/patterns/diagrams/template-method/problem-1.5x.png?id=83fa009f7727bdcc69335b946919f0d8 930w,/images/patterns/diagrams/template-method/problem-2x.png?id=fc8b434afec7b6135043d0d2f48189f0 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="Classes de mineração de dados continham muito código duplicado" />
<figcaption><p>Classes de mineração de dados continham muito
código duplicado.</p></figcaption>
</figure>

Em algum momento você percebeu que todas as três classes tem muito
código parecido. Embora o código para lidar com vários formatos seja
inteiramente diferente em todas as classes, o código para processamento
de dados e análise é quase idêntico. Não seria bacana se livrar da
duplicação de código, deixando a estrutura do algoritmo intacta?

Havia outro problema relacionado com o código cliente que usou essas
classes. Ele tinha muitas condicionais que pegavam um curso de ação
apropriado dependendo da classe do objeto processador. Se todas as três
classes processantes tiverem uma interface comum ou uma classe base,
você poderia eliminar as condicionais no código cliente e usar
polimorfismo quando chamar métodos em um objeto sendo processado.
:::

::: {.section .solution}
##  Solução {#solution}

O padrão do Template Method sugere que você quebre um algoritmo em uma
série de etapas, transforme essas etapas em métodos, e coloque uma série
de chamadas para esses métodos dentro de um único *método padrão*. As
etapas podem ser tanto `abstratas`, ou ter alguma implementação padrão.
Para usar o algoritmo, o cliente deve fornecer sua própria subclasse,
implementar todas as etapas abstratas, e sobrescrever algumas das
opcionais se necessário (mas não o próprio método padrão).

Vamos ver como isso vai funcionar com nossa aplicação de mineração de
dados. Nós podemos criar uma classe base para todos os três algoritmos
de processamento. Essa classe define um método padrão que consiste de
uma série de chamadas para várias etapas de processamento de documentos.

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/solution-pt-brdeff.png?id=e78192c33847daf89b998a546bc0c72e"
style="aspect-ratio: 1.43;"
srcset="/images/patterns/diagrams/template-method/solution-pt-br.png?id=e78192c33847daf89b998a546bc0c72e 600w,/images/patterns/diagrams/template-method/solution-pt-br-1.5x.png?id=ce4c4271b7be7e0e53a742184bd178b2 900w,/images/patterns/diagrams/template-method/solution-pt-br-2x.png?id=43455a54fec3147ce15e27f8c50f602f 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="O Template Method define o esqueleto do algoritmo" />
<figcaption><p>O método padrão quebra o algoritmo em etapas, permitindo
que subclasses sobrescrevam essas etapas mas não o
método atual.</p></figcaption>
</figure>

A princípio nós podemos declarar todos os passos como `abstratos`,
forçando as subclasses a fornecer suas próprias implementações para
esses métodos. No nosso caso, as subclasses já tem todas as
implementações necessárias, então a única coisa que precisamos fazer é
ajustar as assinaturas dos métodos para coincidirem com os da
superclasse.

Agora vamos ver o que podemos fazer para nos livrarmos do código
duplicado. Parece que o código para abrir/fechar arquivos e
extrair/analisar os dados são diferentes para vários formatos de dados,
então não tem porque tocar nesses métodos. Contudo, a implementação
dessas etapas, tais como analisar os dados brutos e compor relatórios, é
muito parecida, então eles podem ser erguidos para a classe base, onde
as subclasses podem compartilhar o código.

Como você pode ver, nós temos dois tipos de etapas:

-   *etapas abstratas* devem ser implementadas por cada subclasse
-   *etapas opcionais* já tem alguma implementação padrão, mas ainda
    podem ser sobrescritas se necessário.

Existe outro tipo de etapa chamado *ganchos*(hooks). Um gancho é uma
etapa opcional com um corpo vazio. Um método padrão poderia funcionar
até mesmo se um hook não for sobrescrito. Geralmente os hooks são
colocados antes e depois de etapas cruciais de algoritmos, fornecendo às
subclasses com pontos de extensão adicionais para um algoritmo.
:::

::: {.section .analogy}
##  Analogia com o mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/live-exampleea7a.png?id=2485d52852f87da06c9cc0e2fd257d6a"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/template-method/live-example.png?id=2485d52852f87da06c9cc0e2fd257d6a 590w,/images/patterns/diagrams/template-method/live-example-1.5x.png?id=92c5fd9345329da09e3233302158678c 885w,/images/patterns/diagrams/template-method/live-example-2x.png?id=89083a3dcd1fe2b627b9b6e6ff4986dc 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Construções de moradia em massa" />
<figcaption><p>Um típico plano de arquitetura pode ser levemente
alterado para melhor servir às necessidades do cliente.</p></figcaption>
</figure>

A abordagem do Template Method pode ser usada na construção em massa de
moradias. O plano arquitetônico para construir uma casa padrão pode
conter diversos pontos de extensões que permitiriam um dono em potencial
ajustar alguns detalhes na casa resultante.

Cada etapa de construção, tais como estabelecer as fundações,
enquadramento, construindo paredes, instalando encanamento e fiação
elétrica para água e eletricidade, etc. pode ser levemente mudado para
se ter uma casa resultante um pouco diferente das outras.
:::

::::: {.section .structure-container}
##  Estrutura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/structure1c3b.png?id=924692f994bff6578d8408d90f6fc459"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.89;"
srcset="/images/patterns/diagrams/template-method/structure.png?id=924692f994bff6578d8408d90f6fc459 340w,/images/patterns/diagrams/template-method/structure-1.5x.png?id=613d78ad8ec08536ec70f4e0976b5a1a 510w,/images/patterns/diagrams/template-method/structure-2x.png?id=25082d6d6a76f51c6b64d8aeeaffdbb5 680w"
sizes="(max-width: 720px) 100vw, 340px" loading="lazy" width="340"
alt="Estrutura do padrão Template Method" /><img
src="../../images/patterns/diagrams/template-method/structure-indexedb94b.png?id=4ced6107519bc66710d2f05c0f4097a1"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.92;"
srcset="/images/patterns/diagrams/template-method/structure-indexed.png?id=4ced6107519bc66710d2f05c0f4097a1 350w,/images/patterns/diagrams/template-method/structure-indexed-1.5x.png?id=58e3a9092786c824eb738a7771702093 525w,/images/patterns/diagrams/template-method/structure-indexed-2x.png?id=86f28789cdcc5a4c415d6a1100de56fc 700w"
sizes="(max-width: 720px) 100vw, 350px" loading="lazy" width="350"
alt="Estrutura do padrão Template Method" />
</figure>
:::

1.  A **Classe Abstrata** declara métodos que agem como etapas de um
    algoritmo, bem como o próprio método padrão que chama esses métodos
    em uma ordem específica. Os passos podem ser declarados como
    `abstratos` ou ter alguma implementação padrão.

2.  As **Classes Concretas** podem sobrescrever todas as etapas, mas não
    o próprio método padrão.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Neste exemplo, o padrão **Template Method** fornece um "esqueleto" para
várias ramificações de inteligência artificial de um jogo de estratégia
simples.

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/example32c6.png?id=c0ce5cc8070925a1cd345fac6afa16b6"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/template-method/example.png?id=c0ce5cc8070925a1cd345fac6afa16b6 430w,/images/patterns/diagrams/template-method/example-1.5x.png?id=d85c6b81c900f46d55688406170600bc 645w,/images/patterns/diagrams/template-method/example-2x.png?id=d8b309659c4b722237ec97733da935bd 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="Exemplo da estrutura do padrão Template Method" />
<figcaption><p>Classes IA de um jogo simples.</p></figcaption>
</figure>

Todas as raças do jogo tem quase o mesmo tipo de unidades e construções.
Portanto você pode reutilizar a mesma estrutura de IA para várias raças,
enquanto é capaz de sobrescrever alguns dos detalhes. Com essa
abordagem, você pode sobrescrever a IA dos Orcs para torná-la mais
agressiva, fazer os humanos mais orientados a defesa e fazer os monstros
serem incapazes de construir qualquer coisa. Adicionando uma nova raça
ao jogo irá necessitar a criação de uma nova subclasse IA e a
sobrescrição dos métodos padrão declarados na classe IA base.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// A classe abstrata define um método padrão que contém um
// esqueleto de algum algoritmo composto de chamadas, geralmente
// para operações abstratas primitivas. Subclasses concretas
// implementam essas operações, mas deixam o método padrão em si
// intacto.
class GameAI is
    // O método padrão define o esqueleto de um algoritmo.
    method turn() is
        collectResources()
        buildStructures()
        buildUnits()
        attack()

    // Algumas das etapas serão implementadas diretamente na
    // classe base.
    method collectResources() is
        foreach (s in this.builtStructures) do
            s.collect()

    // E algumas delas podem ser definidas como abstratas.
    abstract method buildStructures()
    abstract method buildUnits()

    // Uma classe pode ter vários métodos padrão.
    method attack() is
        enemy = closestEnemy()
        if (enemy == null)
            sendScouts(map.center)
        else
            sendWarriors(enemy.position)

    abstract method sendScouts(position)
    abstract method sendWarriors(position)

// Classes concretas têm que implementar todas as operações
// abstratas da classe base, mas não podem sobrescrever o método
// padrão em si.
class OrcsAI extends GameAI is
    method buildStructures() is
        if (there are some resources) then
            // Construir fazendas, depois quartéis, e então uma
            // fortaleza.

    method buildUnits() is
        if (there are plenty of resources) then
            if (there are no scouts)
                // Construir peão, adicionar ele ao grupo de
                // scouts (batedores).
            else
                // Construir um bruto, adicionar ele ao grupo
                // dos guerreiros.

    // ...

    method sendScouts(position) is
        if (scouts.length &gt; 0) then
            // Enviar batedores para posição.


    method sendWarriors(position) is
        if (warriors.length &gt; 5) then
            // Enviar guerreiros para posição.

// As subclasses também podem sobrescrever algumas operações com
// uma implementação padrão.
class MonstersAI extends GameAI is
    method collectResources() is
        // Monstros não coletam recursos.

    method buildStructures() is
        // Monstros não constroem estruturas.

    method buildUnits() is
        // Monstros não constroem unidades.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Aplicabilidade {#applicability}

::::::: applicability
::: applicability-problem
Utilize o padrão Template Method quando você quer deixar os clientes
estender apenas etapas particulares de um algoritmo, mas não todo o
algoritmo e sua estrutura.
:::

::: applicability-solution
O Template Method permite que você transforme um algoritmo monolítico em
uma série de etapas individuais que podem facilmente ser estendidas por
subclasses enquanto ainda mantém intacta a estrutura definida em uma
superclasse.
:::

::: applicability-problem
Utilize o padrão quando você tem várias classes que contém algoritmos
quase idênticos com algumas diferenças menores. Como resultado, você
pode querer modificar todas as classes quando o algoritmo muda.
:::

::: applicability-solution
Quando você transforma tal algoritmo em um Template Method, você também
pode erguer as etapas com implementações similares para dentro de uma
superclasse, eliminando duplicação de código. Códigos que variam entre
subclasses podem permanecer dentro das subclasses.
:::
:::::::
::::::::

::: {.section .checklist}
##  Como implementar {#checklist}

1.  Analise o algoritmo alvo para ver se você quer quebrá-lo em etapas.
    Considere quais etapas são comuns a todas as subclasses e quais
    permanecerão únicas.

2.  Crie a classe abstrata base e declare o método padrão e o conjunto
    de métodos abstratos representando as etapas do algoritmo. Contorne
    a estrutura do algoritmo no método padrão ao executar as etapas
    correspondentes. Considere tornar o método padrão como `final` para
    prevenir subclasses de sobrescrevê-lo.

3.  Tudo bem se todas as etapas terminarem sendo abstratas. Contudo,
    alguns passos podem se beneficiar de ter uma implementação padrão.
    Subclasses não tem que implementar esses métodos.

4.  Pense em adicionar ganchos entre as etapas cruciais do algoritmo.

5.  Para cada variação do algoritmo, crie uma nova subclasse concreta.
    Ela *deve* implementar todas as etapas abstratas, mas *pode* também
    sobrescrever algumas das opcionais.
:::

:::::: {.section .pros-cons}
##  Prós e contras {#pros-cons}

::::: row
::: col-sm-6
-    Você pode deixar clientes sobrescrever apenas certas partes de um
    algoritmo grande, tornando-os menos afetados por mudanças que
    acontece por outras partes do algoritmo.
-    Você pode elevar o código duplicado para uma superclasse.
:::

::: col-sm-6
-    Alguns clientes podem ser limitados ao fornecer o esqueleto de um
    algoritmo.
-    Você pode violar o *princípio de substituição de Liskov* ao
    suprimir uma etapa padrão de implementação através da subclasse.
-    Implementações do padrão Template Method tendem a ser mais difíceis
    de se manter quanto mais etapas eles tiverem.
:::
:::::
::::::

::: {.section .relations}
##  Relações com outros padrões {#relations}

-   O [Factory Method](factory-method.html) é uma especialização do
    [Template Method](template-method.html). Ao mesmo tempo, o *Factory
    Method* pode servir como uma etapa em um *Template Method* grande.

-   O [Template Method](template-method.html) é baseado em herança: ele
    permite que você altere partes de um algoritmo ao estender essas
    partes em subclasses. O [Strategy](strategy.html) é baseado em
    composição: você pode alterar partes do comportamento de um objeto
    ao suprir ele como diferentes estratégias que correspondem a aquele
    comportamento. O *Template Method* funciona a nível de classe, então
    é estático. O *Strategy* trabalha a nível de objeto, permitindo que
    você troque os comportamentos durante a execução.
:::

::: {.section .implementations}
##  Exemplos de código {#implementations}

[![Template Method em
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](template-method/csharp/example.html "Template Method em C#"){.prog-lang-link}
[![Template Method em
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](template-method/cpp/example.html "Template Method em C++"){.prog-lang-link}
[![Template Method em
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](template-method/go/example.html "Template Method em Go"){.prog-lang-link}
[![Template Method em
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](template-method/java/example.html "Template Method em Java"){.prog-lang-link}
[![Template Method em
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](template-method/php/example.html "Template Method em PHP"){.prog-lang-link}
[![Template Method em
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](template-method/python/example.html "Template Method em Python"){.prog-lang-link}
[![Template Method em
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](template-method/ruby/example.html "Template Method em Ruby"){.prog-lang-link}
[![Template Method em
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](template-method/rust/example.html "Template Method em Rust"){.prog-lang-link}
[![Template Method em
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](template-method/swift/example.html "Template Method em Swift"){.prog-lang-link}
[![Template Method em
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](template-method/typescript/example.html "Template Method em TypeScript"){.prog-lang-link}
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

[Visitor []{.fa .fa-arrow-right}](visitor.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Strategy](strategy.html){.btn .btn-default
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
