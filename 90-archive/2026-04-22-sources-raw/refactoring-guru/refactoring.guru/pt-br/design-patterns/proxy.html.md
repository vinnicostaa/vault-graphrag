:::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
estruturais](structural-patterns.html){.type}
:::

# Proxy {#proxy .title}

::: {.section .intent}
##  Propósito {#intent}

O **Proxy** é um padrão de projeto estrutural que permite que você
forneça um substituto ou um espaço reservado para outro objeto. Um proxy
controla o acesso ao objeto original, permitindo que você faça algo ou
antes ou depois do pedido chegar ao objeto original.

<figure class="image">
<img
src="../../images/patterns/content/proxy/proxyf258.png?id=efece4647fb11e3f7539291796327666"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/proxy/proxy.png?id=efece4647fb11e3f7539291796327666 640w,/images/patterns/content/proxy/proxy-1.5x.png?id=80a11690fa49172c517470a834289b60 960w,/images/patterns/content/proxy/proxy-2x.png?id=fb3d14e21c210a758d4777f4d93dce09 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Padrão de projeto Proxy" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Por que eu iria querer controlar o acesso a um objeto? Aqui está um
exemplo: você tem um objeto grande que consome muitos recursos do
sistema. Você precisa dele de tempos em tempos, mas não sempre.

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/problem-pt-br5abe.png?id=77c37b2d6146eb75eedba6e453a9c0f3"
style="aspect-ratio: 3.19;"
srcset="/images/patterns/diagrams/proxy/problem-pt-br.png?id=77c37b2d6146eb75eedba6e453a9c0f3 510w,/images/patterns/diagrams/proxy/problem-pt-br-1.5x.png?id=22eae17c45205ea7742babd1bf21eea3 765w,/images/patterns/diagrams/proxy/problem-pt-br-2x.png?id=b45cd2900db20501e2f19df14342fd18 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Problema resolvido pelo padrão Proxy" />
<figcaption><p>Solicitações para bases de dados podem ser
bem lentas.</p></figcaption>
</figure>

Você poderia implementar uma inicialização preguiçosa: criar esse objeto
apenas quando ele é realmente necessário. Todos os clientes do objeto
teriam que executar algum código adiado de inicialização. Infelizmente,
isso provavelmente resultaria em muito código duplicado.

Em um mundo ideal, gostaríamos que você colocasse esse código
diretamente dentro da classe do nosso objeto, mas isso nem sempre é
possível. Por exemplo, a classe pode fazer parte de uma biblioteca
fechada de terceiros.
:::

::: {.section .solution}
##  Solução {#solution}

O padrão Proxy sugere que você crie uma nova classe proxy com a mesma
interface do objeto do serviço original. Então você atualiza sua
aplicação para que ela passe o objeto proxy para todos os clientes do
objeto original. Ao receber uma solicitação de um cliente, o proxy cria
um objeto do serviço real e delega a ele todo o trabalho.

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/solution-pt-brce6a.png?id=639d07cfce75d09b50a35ebb53056f9d"
style="aspect-ratio: 3.19;"
srcset="/images/patterns/diagrams/proxy/solution-pt-br.png?id=639d07cfce75d09b50a35ebb53056f9d 510w,/images/patterns/diagrams/proxy/solution-pt-br-1.5x.png?id=a18348202a8287d54df55160a82e5986 765w,/images/patterns/diagrams/proxy/solution-pt-br-2x.png?id=0982f2534d698d7f4ce4e37dfe9f1a9d 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Solução com o padrão Proxy" />
<figcaption><p>O proxy se disfarça de objeto de base de dados. Ele pode
lidar com inicializações preguiçosas e caches de resultados sem que o
cliente ou a base de dados fiquem sabendo.</p></figcaption>
</figure>

Mas qual é o benefício? Se você precisa executar alguma coisa tanto
antes como depois da lógica primária da classe, o proxy permite que você
faça isso sem mudar aquela classe. Uma vez que o proxy implementa a
mesma interface que a classe original, ele pode ser passado para
qualquer cliente que espera um objeto do serviço real.
:::

::: {.section .analogy}
##  Analogia com o mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/live-example66da.png?id=a268c57fdaf073ee81cf4dfc7239eae2"
style="aspect-ratio: 2.57;"
srcset="/images/patterns/diagrams/proxy/live-example.png?id=a268c57fdaf073ee81cf4dfc7239eae2 540w,/images/patterns/diagrams/proxy/live-example-1.5x.png?id=4b8ee7f90cf76180004e9f5d37661ee2 810w,/images/patterns/diagrams/proxy/live-example-2x.png?id=8b8083fa1954d2d92ca84a5a5636ead7 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Um cartão de crédito é um proxy para uma porção de dinheiro" />
<figcaption><p>Cartões de crédito podem ser usados como pagamentos assim
como o dinheiro.</p></figcaption>
</figure>

Um cartão de crédito é um proxy para uma conta bancária, que é um proxy
para uma porção de dinheiro. Ambos implementam a mesma interface porque
não há necessidade de carregar uma porção de dinheiro por aí. Um cliente
se sente bem porque não precisa ficar carregando montanhas de dinheiro
por aí. Um dono de loja também fica feliz uma vez que a renda da
transação é adicionada eletronicamente para sua conta sem o risco de
perdê-la no depósito ou de ser roubado quando estiver indo ao banco.
:::

::::: {.section .structure-container}
##  Estrutura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/structure7fcd.png?id=f2478a82a84e1a1e512a8414bf1abd1c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/proxy/structure.png?id=f2478a82a84e1a1e512a8414bf1abd1c 370w,/images/patterns/diagrams/proxy/structure-1.5x.png?id=0db7bf3c1193b2a1961894349f1e07ad 555w,/images/patterns/diagrams/proxy/structure-2x.png?id=3d54eeca9af4aa373e989a73463539b5 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Estrutura do padrão de projeto Proxy" /><img
src="../../images/patterns/diagrams/proxy/structure-indexed2b2c.png?id=e56d420f31925b3d41348c5036d3cd64"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.05;"
srcset="/images/patterns/diagrams/proxy/structure-indexed.png?id=e56d420f31925b3d41348c5036d3cd64 410w,/images/patterns/diagrams/proxy/structure-indexed-1.5x.png?id=5b24af632e76d81d24eab0b7c0be1a66 615w,/images/patterns/diagrams/proxy/structure-indexed-2x.png?id=ddf2b3b4807e52330c456c59fc52d504 820w"
sizes="(max-width: 720px) 100vw, 410px" loading="lazy" width="410"
alt="Estrutura do padrão de projeto Proxy" />
</figure>
:::

1.  A **Interface do Serviço** declara a interface do Serviço. O proxy
    deve seguir essa interface para ser capaz de se disfarçar como um
    objeto do serviço.

2.  O **Serviço** é uma classe que fornece alguma lógica de negócio
    útil.

3.  A classe **Proxy** tem um campo de referência que aponta para um
    objeto do serviço. Após o proxy finalizar seu processamento (por
    exemplo: inicialização preguiçosa, acesso, acessar controle, colocar
    em cache, etc.), ele passa o pedido para o objeto do serviço.

    Geralmente os proxies gerenciam todo o ciclo de vida dos seus
    objetos de serviço.

4.  O **Cliente** deve trabalhar tanto com os serviços e proxies através
    da mesma interface. Dessa forma você pode passar uma proxy para
    qualquer código que espera um objeto do serviço.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Este exemplo ilustra como o padrão **Proxy** pode ajudar a introduzir
uma inicialização preguiçosa e cache para um biblioteca de integração
terceirizada do YouTube.

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/example4e80.png?id=ddb1402479562b9e2c776933cc861bed"
style="aspect-ratio: 1.23;"
srcset="/images/patterns/diagrams/proxy/example.png?id=ddb1402479562b9e2c776933cc861bed 490w,/images/patterns/diagrams/proxy/example-1.5x.png?id=9b7608e1ef46a52d4ca1d7d89986e5b0 735w,/images/patterns/diagrams/proxy/example-2x.png?id=40f22d12d183b9285a380e4a072efb16 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Exemplo de estrutura de um padrão Proxy" />
<figcaption><p>Colocando em cache os resultados de um serviço com
um proxy.</p></figcaption>
</figure>

A biblioteca fornece a nós com uma classe de download de vídeo. Contudo,
ela é muito ineficiente. Se a aplicação cliente pedir o mesmo vídeo
múltiplas vezes, a biblioteca apenas baixa de novo e de novo, ao invés
de colocar ele em cache e reutilizar o primeiro arquivo de download.

A classe proxy implementa a mesma interface que a classe baixadora
original e delega-a todo o trabalho. Contudo, ela mantém um registro dos
arquivos baixados e retorna o resultado em cache quando a aplicação pede
o mesmo vídeo múltiplas vezes.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// A interface de um serviço remoto.
interface ThirdPartyYouTubeLib is
    method listVideos()
    method getVideoInfo(id)
    method downloadVideo(id)

// A implementação concreta de um serviço conector. Métodos
// dessa classe podem pedir informações do YouTube. A velocidade
// do pedido depende da conexão do usuário com a internet, bem
// como do YouTube. A aplicação irá ficar lenta se muitos
// pedidos forem feitos ao mesmo tempo, mesmo que todos peçam a
// mesma informação.
class ThirdPartyYouTubeClass implements ThirdPartyYouTubeLib is
    method listVideos() is
        // Envia um pedido API para o YouTube.

    method getVideoInfo(id) is
        // Obtém metadados sobre algum vídeo.

    method downloadVideo(id) is
        // Baixa um arquivo de vídeo do YouTube.

// Para salvar largura de banda, nós podemos colocar os
// resultados do pedido em cache e mantê-los por determinado
// tempo. Mas pode ser impossível colocar tal código diretamente
// na classe de serviço. Por exemplo, ele pode ter sido
// fornecido como parte de uma biblioteca de terceiros e/ou
// definida como `final`. É por isso que nós colocamos o código
// do cache em uma nova classe proxy que implementa a mesma
// interface que a classe de serviço. Ela delega ao objeto do
// serviço somente quando os pedidos reais foram enviados.
class CachedYouTubeClass implements ThirdPartyYouTubeLib is
    private field service: ThirdPartyYouTubeLib
    private field listCache, videoCache
    field needReset

    constructor CachedYouTubeClass(service: ThirdPartyYouTubeLib) is
        this.service = service

    method listVideos() is
        if (listCache == null || needReset)
            listCache = service.listVideos()
        return listCache

    method getVideoInfo(id) is
        if (videoCache == null || needReset)
            videoCache = service.getVideoInfo(id)
        return videoCache

    method downloadVideo(id) is
        if (!downloadExists(id) || needReset)
            service.downloadVideo(id)

// A classe GUI, que é usada para trabalhar diretamente com um
// objeto de serviço, permanece imutável desde que trabalhe com
// o objeto de serviço através de uma interface. Nós podemos
// passar um objeto proxy com segurança ao invés de um objeto
// real de serviço uma vez que ambos implementam a mesma
// interface.
class YouTubeManager is
    protected field service: ThirdPartyYouTubeLib

    constructor YouTubeManager(service: ThirdPartyYouTubeLib) is
        this.service = service

    method renderVideoPage(id) is
        info = service.getVideoInfo(id)
        // Renderiza a página do vídeo.

    method renderListPanel() is
        list = service.listVideos()
        // Renderiza a lista de miniaturas do vídeo.

    method reactOnUserInput() is
        renderVideoPage()
        renderListPanel()

// A aplicação pode configurar proxies de forma fácil e rápida.
class Application is
    method init() is
        aYouTubeService = new ThirdPartyYouTubeClass()
        aYouTubeProxy = new CachedYouTubeClass(aYouTubeService)
        manager = new YouTubeManager(aYouTubeProxy)
        manager.reactOnUserInput()</code></pre>
</figure>
:::

:::::::::::::::: {.section .applicability-container}
##  Aplicabilidade {#applicability}

::::::::::::::: applicability
Há dúzias de maneiras de utilizar o padrão Proxy. Vamos ver os usos mais
populares.

::: applicability-problem
Inicialização preguiçosa (proxy virtual). Este é quando você tem um
objeto do serviço peso-pesado que gasta recursos do sistema por estar
sempre rodando, mesmo quando você precisa dele de tempos em tempos.
:::

::: applicability-solution
Ao invés de criar um objeto quando a aplicação inicializa, você pode
atrasar a inicialização do objeto para um momento que ele é realmente
necessário.
:::

::: applicability-problem
Controle de acesso (proxy de proteção). Este é quando você quer que
apenas clientes específicos usem o objeto do serviço; por exemplo,
quando seus objetos são partes cruciais de um sistema operacional e os
clientes são várias aplicações iniciadas (incluindo algumas maliciosas).
:::

::: applicability-solution
O proxy pode passar o pedido para o objeto de serviço somente se as
credenciais do cliente coincidem com certos critérios.
:::

::: applicability-problem
Execução local de um serviço remoto (proxy remoto). Este é quando o
objeto do serviço está localizado em um servidor remoto.
:::

::: applicability-solution
Neste caso, o proxy passa o pedido do cliente pela rede, lidando com
todos os detalhes sujos pertinentes a se trabalhar com a rede.
:::

::: applicability-problem
Registros de pedidos (proxy de registro). Este é quando você quer manter
um histórico de pedidos ao objeto do serviço.
:::

::: applicability-solution
O proxy pode fazer o registro de cada pedido antes de passar ao serviço.
:::

::: applicability-problem
Cache de resultados de pedidos (proxy de cache). Este é quando você
precisa colocar em cache os resultados de pedidos do cliente e gerenciar
o ciclo de vida deste cache, especialmente se os resultados são muito
grandes.
:::

::: applicability-solution
O proxy pode implementar o armazenamento em cache para pedidos
recorrentes que sempre acabam nos mesmos resultados. O proxy pode usar
como parâmetros dos pedidos as chaves de cache.
:::

::: applicability-problem
Referência inteligente. Este é para quando você precisa ser capaz de se
livrar de um objeto peso-pesado assim que não há mais clientes que o
usam.
:::

::: applicability-solution
O proxy pode manter um registro de clientes que obtiveram uma referência
ao objeto serviço ou seus resultados. De tempos em tempos, o proxy pode
verificar com os clientes se eles ainda estão ativos. Se a lista cliente
ficar vazia, o proxy pode remover o objeto serviço e liberar os recursos
de sistema que ficaram empatados.

O proxy pode também fiscalizar se o cliente modificou o objeto do
serviço. Então os objetos sem mudança podem ser reutilizados por outros
clientes.
:::
:::::::::::::::
::::::::::::::::

::: {.section .checklist}
##  Como implementar {#checklist}

1.  Se não há uma interface do serviço pré existente, crie uma para
    fazer os objetos proxy e serviço intercomunicáveis. Extrair a
    interface da classe serviço nem sempre é possível, porque você
    precisaria mudar todos os clientes do serviço para usar aquela
    interface. O plano B é fazer do proxy uma subclasse da classe
    serviço e, dessa forma, ele herdará a interface do serviço.

2.  Crie a classe proxy. Ela deve ter um campo para armazenar uma
    referência ao serviço. Geralmente proxies criam e gerenciam todo o
    ciclo de vida de seus serviços. Em raras ocasiões, um serviço é
    passado ao proxy através do construtor pelo cliente.

3.  Implemente os métodos proxy de acordo com o propósito deles. Na
    maioria dos casos, após realizar algum trabalho, o proxy deve
    delegar o trabalho para o objeto do serviço.

4.  Considere introduzir um método de criação que decide se o cliente
    obtém um proxy ou serviço real. Isso pode ser um simples método
    estático na classe do proxy ou um método factory todo implementado.

5.  Considere implementar uma inicialização preguiçosa para o objeto do
    serviço.
:::

:::::: {.section .pros-cons}
##  Prós e contras {#pros-cons}

::::: row
::: col-sm-6
-    Você pode controlar o objeto do serviço sem os clientes ficarem
    sabendo.
-    Você pode gerenciar o ciclo de vida de um objeto do serviço quando
    os clientes não se importam mais com ele.
-    O proxy trabalha até mesmo se o objeto do serviço ainda não está
    pronto ou disponível.
-    *Princípio aberto/fechado*. Você pode introduzir novos proxies sem
    mudar o serviço ou clientes.
:::

::: col-sm-6
-    O código pode ficar mais complicado uma vez que você precisa
    introduzir uma série de novas classes.
-    A resposta de um serviço pode ter atrasos.
:::
:::::
::::::

::: {.section .relations}
##  Relações com outros padrões {#relations}

-   Com [Adapter](adapter.html), você acessa um objeto existente por
    meio de uma interface diferente. Com [Proxy](proxy.html), a
    interface permanece a mesma. Com [Decorator](decorator.html), você
    acessa o objeto por meio de uma interface aprimorada.

-   O [Facade](facade.html) é parecido como o [Proxy](proxy.html) no
    quesito que ambos colocam em buffer uma entidade complexa e
    inicializam ela sozinhos. Ao contrário do *Facade*, o *Proxy* tem a
    mesma interface que seu objeto de serviço, o que os torna
    intermutáveis.

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

[![Proxy em
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](proxy/csharp/example.html "Proxy em C#"){.prog-lang-link}
[![Proxy em
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](proxy/cpp/example.html "Proxy em C++"){.prog-lang-link}
[![Proxy em
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](proxy/go/example.html "Proxy em Go"){.prog-lang-link}
[![Proxy em
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](proxy/java/example.html "Proxy em Java"){.prog-lang-link}
[![Proxy em
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](proxy/php/example.html "Proxy em PHP"){.prog-lang-link}
[![Proxy em
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](proxy/python/example.html "Proxy em Python"){.prog-lang-link}
[![Proxy em
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](proxy/ruby/example.html "Proxy em Ruby"){.prog-lang-link}
[![Proxy em
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](proxy/rust/example.html "Proxy em Rust"){.prog-lang-link}
[![Proxy em
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](proxy/swift/example.html "Proxy em Swift"){.prog-lang-link}
[![Proxy em
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](proxy/typescript/example.html "Proxy em TypeScript"){.prog-lang-link}
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

[Padrões comportamentais []{.fa
.fa-arrow-right}](behavioral-patterns.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Flyweight](flyweight.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::
