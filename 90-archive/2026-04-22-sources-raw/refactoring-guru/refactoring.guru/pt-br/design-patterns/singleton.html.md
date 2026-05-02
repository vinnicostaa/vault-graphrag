::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
criacionais](creational-patterns.html){.type}
:::

# Singleton {#singleton .title}

::: aka
Também conhecido como: [Carta única]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

O **Singleton** é um padrão de projeto criacional que permite a você
garantir que uma classe tenha apenas uma instância, enquanto provê um
ponto de acesso global para essa instância.

<figure class="image">
<img
src="../../images/patterns/content/singleton/singleton4263.png?id=108a0b9b5ea5c4426e0afa4504491d6f"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/singleton/singleton.png?id=108a0b9b5ea5c4426e0afa4504491d6f 640w,/images/patterns/content/singleton/singleton-1.5x.png?id=29490ad6cd1426c63cf5f88243a1701c 960w,/images/patterns/content/singleton/singleton-2x.png?id=accb2cc7594f7a491ce01dddf0d2f876 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Padrão Singleton" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

O padrão Singleton resolve dois problemas de uma só vez, violando o
*princípio de responsabilidade única*:

1.  **Garantir que uma classe tenha apenas uma única instância**. Por
    que alguém iria querer controlar quantas instâncias uma classe tem?
    A razão mais comum para isso é para controlar o acesso a algum
    recurso compartilhado---por exemplo, uma base de dados ou um
    arquivo.

    Funciona assim: imagine que você criou um objeto, mas depois de um
    tempo você decidiu criar um novo. Ao invés de receber um objeto
    fresco, você obterá um que já foi criado.

    Observe que esse comportamento é impossível implementar com um
    construtor regular uma vez que uma chamada do construtor **deve**
    sempre retornar um novo objeto por design.

<figure class="image">
<img
src="../../images/patterns/content/singleton/singleton-comic-1-pt-br32ba.png?id=6c0be2bd018e2db71b94507b0c9ec8e5"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/singleton/singleton-comic-1-pt-br.png?id=6c0be2bd018e2db71b94507b0c9ec8e5 600w,/images/patterns/content/singleton/singleton-comic-1-pt-br-1.5x.png?id=d5c52196032d297a13ed2ccea5592b53 900w,/images/patterns/content/singleton/singleton-comic-1-pt-br-2x.png?id=81cca7d664b1b73385a6d872092cb3ff 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="O acesso global para um objeto" />
<figcaption><p>Clientes podem não se dar conta que estão lidando com o
mesmo objeto a todo momento.</p></figcaption>
</figure>

2.  **Fornece um ponto de acesso global para aquela instância**. Se
    lembra daquelas variáveis globais que você (tá bom, eu) usou para
    guardar alguns objetos essenciais? Embora sejam muito úteis, elas
    também são muito inseguras uma vez que qualquer código pode
    potencialmente sobrescrever os conteúdos daquelas variáveis e
    quebrar a aplicação.

    Assim como uma variável global, o padrão Singleton permite que você
    acesse algum objeto de qualquer lugar no programa. Contudo, ele
    também protege aquela instância de ser sobrescrita por outro código.

    Há outro lado para esse problema: você não quer que o código que
    resolve o problema #1 fique espalhado por todo seu programa. É muito
    melhor tê-lo dentro de uma classe, especialmente se o resto do seu
    código já depende dela.

Hoje em dia, o padrão Singleton se tornou tão popular que as pessoas
podem chamar algo de *singleton* mesmo se ele resolve apenas um dos
problemas listados.
:::

::: {.section .solution}
##  Solução {#solution}

Todas as implementações do Singleton tem esses dois passos em comum:

-   Fazer o construtor padrão privado, para prevenir que outros objetos
    usem o operador `new` com a classe singleton.
-   Criar um método estático de criação que age como um construtor. Esse
    método chama o construtor privado por debaixo dos panos para criar
    um objeto e o salva em um campo estático. Todas as chamadas
    seguintes para esse método retornam o objeto em cache.

Se o seu código tem acesso à classe singleton, então ele será capaz de
chamar o método estático da singleton. Então sempre que aquele método é
chamado, o mesmo objeto é retornado.
:::

::: {.section .analogy}
##  Analogia com o mundo real {#analogy}

O governo é um excelente exemplo de um padrão Singleton. Um país pode
ter apenas um governo oficial. Independentemente das identidades
pessoais dos indivíduos que formam governos, o título, "O Governo de X",
é um ponto de acesso global que identifica o grupo de pessoas no
command.
:::

::::: {.section .structure-container}
##  Estrutura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/singleton/structure-pt-br7755.png?id=151e5e19974d89c1382c5a92899784c4"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.48;"
srcset="/images/patterns/diagrams/singleton/structure-pt-br.png?id=151e5e19974d89c1382c5a92899784c4 430w,/images/patterns/diagrams/singleton/structure-pt-br-1.5x.png?id=c61d09a6b5bf29d6ed0ce554683913ab 645w,/images/patterns/diagrams/singleton/structure-pt-br-2x.png?id=aba43fac22976cfa26b70c541120c3b2 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="A estrutura do padrão Singleton" /><img
src="../../images/patterns/diagrams/singleton/structure-pt-br-indexed6cd4.png?id=df2a78131bc3c993ee50c74fb6b7768b"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.48;"
srcset="/images/patterns/diagrams/singleton/structure-pt-br-indexed.png?id=df2a78131bc3c993ee50c74fb6b7768b 430w,/images/patterns/diagrams/singleton/structure-pt-br-indexed-1.5x.png?id=5e63b97035c23af78b2bf03e20b24174 645w,/images/patterns/diagrams/singleton/structure-pt-br-indexed-2x.png?id=7cb433c0d8bbf0d182b9174cf8752083 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="A estrutura do padrão Singleton" />
</figure>
:::

1.  A classe **Singleton** declara o método estático `getInstance` que
    retorna a mesma instância de sua própria classe.

    O construtor da singleton deve ser escondido do código cliente.
    Chamando o método `getInstance` deve ser o único modo de obter o
    objeto singleton.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Neste exemplo, a classe de conexão com a base de dados age como um
**Singleton**. Essa classe não tem um construtor público, então a única
maneira de obter seu objeto é chamando o método `getInstance`. Esse
método coloca o primeiro objeto criado em cache e o retorna em todas as
chamadas subsequentes.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// A classe Database define o método `getInstance` que permite
// clientes acessar a mesma instância de uma conexão a base de
// dados através do programa.
class Database is
    // O campo para armazenar a instância singleton deve ser
    // declarado como estático.
    private static field instance: Database

    // O construtor do singleton devem sempre ser privado para
    // prevenir chamadas diretas de construção com o operador
    // `new`.
    private constructor Database() is
        // Algum código de inicialização, tal como uma conexão
        // com um servidor de base de dados.
        // ...

    // O método estático que controla acesso à instância do
    // singleton
    public static method getInstance() is
        if (Database.instance == null) then
            acquireThreadLock() and then
                // Certifique que a instância ainda não foi
                // inicializada por outra thread enquanto está
                // estiver esperando pela liberação do `lock`.
                if (Database.instance == null) then
                    Database.instance = new Database()
        return Database.instance

    // Finalmente, qualquer singleton deve definir alguma lógica
    // de negócio que deve ser executada em sua instância.
    public method query(sql) is
        // Por exemplo, todas as solicitações à base de dados de
        // uma aplicação passam por esse método. Portanto, você
        // pode colocar a lógica de throttling ou cache aqui.
        // ...

class Application is
    method main() is
        Database foo = Database.getInstance()
        foo.query(&quot;SELECT ...&quot;)
        // ...
        Database bar = Database.getInstance()
        bar.query(&quot;SELECT ...&quot;)
        // A variável `bar` vai conter o mesmo objeto que a
        // variável `foo`.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Aplicabilidade {#applicability}

::::::: applicability
::: applicability-problem
Utilize o padrão Singleton quando uma classe em seu programa deve ter
apenas uma instância disponível para todos seus clientes; por exemplo,
um objeto de base de dados único compartilhado por diferentes partes do
programa.
:::

::: applicability-solution
O padrão Singleton desabilita todos os outros meios de criar objetos de
uma classe exceto pelo método especial de criação. Esse método tanto
cria um novo objeto ou retorna um objeto existente se ele já tenha sido
criado.
:::

::: applicability-problem
Utilize o padrão Singleton quando você precisa de um controle mais
estrito sobre as variáveis globais.
:::

::: applicability-solution
Ao contrário das variáveis globais, o padrão Singleton garante que há
apenas uma instância de uma classe. Nada, a não ser a própria classe
singleton, pode substituir a instância salva em cache.

Observe que você sempre pode ajustar essa limitação e permitir a criação
de qualquer número de instâncias singleton. O único pedaço de código que
requer mudanças é o corpo do método `getInstance`.
:::
:::::::
::::::::

::: {.section .checklist}
##  Como implementar {#checklist}

1.  Adicione um campo privado estático na classe para o armazenamento da
    instância singleton.

2.  Declare um método de criação público estático para obter a instância
    singleton.

3.  Implemente a "inicialização preguiçosa" dentro do método estático.
    Ela deve criar um novo objeto na sua primeira chamada e colocá-lo no
    campo estático. O método deve sempre retornar aquela instância em
    todas as chamadas subsequentes.

4.  Faça o construtor da classe ser privado. O método estático da classe
    vai ainda ser capaz de chamar o construtor, mas não os demais
    objetos.

5.  Vá para o código cliente e substitua todas as chamadas diretas para
    o construtor do singleton com chamadas para seu método de criação
    estático.
:::

:::::: {.section .pros-cons}
##  Prós e contras {#pros-cons}

::::: row
::: col-sm-6
-    Você pode ter certeza que uma classe só terá uma única instância.
-    Você ganha um ponto de acesso global para aquela instância.
-    O objeto singleton é inicializado somente quando for pedido pela
    primeira vez.
:::

::: col-sm-6
-    Viola o *princípio de responsabilidade única*. O padrão resolve
    dois problemas de uma só vez.
-    O padrão Singleton pode mascarar um design ruim, por exemplo,
    quando os componentes do programa sabem muito sobre cada um.
-    O padrão requer tratamento especial em um ambiente multithreaded
    para que múltiplas threads não possam criar um objeto singleton
    várias vezes.
-    Pode ser difícil realizar testes unitários do código cliente do
    Singleton porque muitos frameworks de teste dependem de herança
    quando produzem objetos simulados. Já que o construtor da classe
    singleton é privado e sobrescrever métodos estáticos é impossível na
    maioria das linguagem, você terá que pensar em uma maneira criativa
    de simular o singleton. Ou apenas não escreva os testes. Ou não use
    o padrão Singleton.
:::
:::::
::::::

::: {.section .relations}
##  Relações com outros padrões {#relations}

-   Uma classe [fachada](facade.html) pode frequentemente ser
    transformada em uma [singleton](singleton.html) já que um único
    objeto fachada é suficiente na maioria dos casos.

-   O [Flyweight](flyweight.html) seria parecido com o
    [Singleton](singleton.html) se você, de algum modo, reduzisse todos
    os estados de objetos compartilhados para apenas um objeto
    flyweight. Mas há duas mudanças fundamentais entre esses padrões:

    1.  Deve haver apenas uma única instância singleton, enquanto que
        uma classe *flyweight* pode ter múltiplas instâncias com
        diferentes estados intrínsecos.
    2.  O objeto *singleton* pode ser mutável. Objetos flyweight são
        imutáveis.

-   As [Fábricas Abstratas](abstract-factory.html),
    [Construtores](builder.html), e [Protótipos](prototype.html) podem
    todos ser implementados como [Singletons](singleton.html).
:::

::: {.section .implementations}
##  Exemplos de código {#implementations}

[![Singleton em
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](singleton/csharp/example.html "Singleton em C#"){.prog-lang-link}
[![Singleton em
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](singleton/cpp/example.html "Singleton em C++"){.prog-lang-link}
[![Singleton em
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](singleton/go/example.html "Singleton em Go"){.prog-lang-link}
[![Singleton em
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](singleton/java/example.html "Singleton em Java"){.prog-lang-link}
[![Singleton em
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](singleton/php/example.html "Singleton em PHP"){.prog-lang-link}
[![Singleton em
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](singleton/python/example.html "Singleton em Python"){.prog-lang-link}
[![Singleton em
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](singleton/ruby/example.html "Singleton em Ruby"){.prog-lang-link}
[![Singleton em
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](singleton/rust/example.html "Singleton em Rust"){.prog-lang-link}
[![Singleton em
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](singleton/swift/example.html "Singleton em Swift"){.prog-lang-link}
[![Singleton em
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](singleton/typescript/example.html "Singleton em TypeScript"){.prog-lang-link}
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

[Padrões estruturais []{.fa
.fa-arrow-right}](structural-patterns.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Prototype](prototype.html){.btn .btn-default
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
