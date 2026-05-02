:::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Singleton](../../singleton.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Singleton](../../../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x"}
:::

# **Singleton** en Go {#singleton-en-go .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
Le **Singleton** est un patron de conception de création qui s'assure de
l'existence d'un seul objet de son genre et fournit un unique point
d'accès vers cet objet.

Le singleton possède à peu près les mêmes avantages et inconvénients que
les variables globales. Même s'ils sont super utiles, ils réduisent la
modularité du code.

Vous ne pourrez pas utiliser une classe qui dépend d'un singleton dans
un autre contexte. Vous devrez également inclure complètement la classe
Singleton dans votre code. En général, on se rend compte de cette
limitation lorsque l'on crée des tests unitaires.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Singleton ](../../singleton.html){.btn
.btn-lg .btn-primary}
:::

::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Exemple conceptuel](#example-0)
:::

::: en-file
 [single](#example-0--single-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::

::: en-intro
 [Autre exemple](#example-1)
:::

::: en-file
 [sync­Once](#example-1--syncOnce-go)
:::

::: en-file
 [main](#example-1--main-go)
:::

::: en-file
 [output](#example-1--output-txt)
:::
:::::::::::::
::::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Exemple conceptuel](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Autre exemple](#example-1){#example-1-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Exemple conceptuel {#example-0-title}

En général, une instance de singleton est créée lors de la première
initialisation de la struct. Pour ce faire, nous définissons la méthode
`getInstance` dans la struct. Cette méthode sera responsable de la
création et du renvoi de l'instance du singleton. Une fois créée, cette
même instance sera retournée chaque fois que `getInstance` est appelée.

Qu'en est-il des goroutines ? La struct du singleton doit renvoyer la
même instance si plusieurs goroutines tentent d'y accéder. C'est la
raison pour laquelle on peut facilement se tromper et mal concevoir
notre singleton. L'exemple ci-dessous vous montre la bonne manière de le
mettre en place.

Quelques points intéressants à noter :

-   Il y a un test de nullité au début pour s'assurer que
    `singleInstance` est vide la première fois. Ce test permet de ne pas
    avoir à mettre en place des locks (coûteux) chaque fois que la
    méthode `getInstance` est appelée. Si ce test échoue, cela veut dire
    que l'attribut `singleInstance` a déjà été initialisé.

-   La struct `singleInstance` est créée à l'intérieur du lock.

-   Il y a un autre test de nullité après avoir placé le lock. Il permet
    de s'assurer qu'une seule des goroutines parmi celles qui passent la
    première vérification puisse créer l'instance du singleton. Sinon,
    toutes les goroutines vont créer leur propre instance du struct
    singleton.

#### []{#example-0--single-go .anchor} **single.go:** Singleton

<figure class="code">
<pre class="code" lang="go"><code>package main

import (
    &quot;fmt&quot;
    &quot;sync&quot;
)

var lock = &amp;sync.Mutex{}

type single struct {
}

var singleInstance *single

func getInstance() *single {
    if singleInstance == nil {
        lock.Lock()
        defer lock.Unlock()
        if singleInstance == nil {
            fmt.Println(&quot;Creating single instance now.&quot;)
            singleInstance = &amp;single{}
        } else {
            fmt.Println(&quot;Single instance already created.&quot;)
        }
    } else {
        fmt.Println(&quot;Single instance already created.&quot;)
    }

    return singleInstance
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Code client

<figure class="code">
<pre class="code" lang="go"><code>package main

import (
    &quot;fmt&quot;
)

func main() {

    for i := 0; i &lt; 30; i++ {
        go getInstance()
    }

    // Scanln is similar to Scan, but stops scanning at a newline and
    // after the final item there must be a newline or EOF.
    fmt.Scanln()
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>Creating single instance now.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Autre exemple {#example-1-title}

Il y a d'autres méthodes pour créer une instance de singleton en Go :

1.  Fonction `init`

Nous pouvons créer une instance unique à l'intérieur de la fonction
`init`. Ce n'est possible que si l'initialisation précoce de l'instance
nous l'accorde. La fonction `init` n'est appelée qu'une seule fois par
fichier dans un package, nous sommes donc certains qu'une seule instance
sera créée.

2.  `sync.Once`

`sync.Once` ne lancera le traitement qu'une seule fois. Voir le code
ci-dessous :

#### []{#example-1--syncOnce-go .anchor} **syncOnce.go:** Singleton

<figure class="code">
<pre class="code" lang="go"><code>package main

import (
    &quot;fmt&quot;
    &quot;sync&quot;
)

var once sync.Once

type single struct {
}

var singleInstance *single

func getInstance() *single {
    if singleInstance == nil {
        once.Do(
            func() {
                fmt.Println(&quot;Creating single instance now.&quot;)
                singleInstance = &amp;single{}
            })
    } else {
        fmt.Println(&quot;Single instance already created.&quot;)
    }

    return singleInstance
}</code></pre>
</figure>

#### []{#example-1--main-go .anchor} **main.go:** Code client

<figure class="code">
<pre class="code" lang="go"><code>package main

import (
    &quot;fmt&quot;
)

func main() {

    for i := 0; i &lt; 30; i++ {
        go getInstance()
    }

    // Scanln is similar to Scan, but stops scanning at a newline and
    // after the final item there must be a newline or EOF.
    fmt.Scanln()
}</code></pre>
</figure>

#### []{#example-1--output-txt .anchor} **output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>Creating single instance now.
Single instance already created.
Single instance already created.</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Exemple conceptuel](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Autre exemple](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### Suivant

[Adaptateur en Go []{.fa
.fa-arrow-right}](../../adapter/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Prototype en
Go](../../prototype/go/example.html){.btn .btn-default rel="prev"}
:::

## **Singleton** dans les autres langues

[![Singleton en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Singleton en C#"){.prog-lang-link}
[![Singleton en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Singleton en C++"){.prog-lang-link}
[![Singleton en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Singleton en Java"){.prog-lang-link}
[![Singleton en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Singleton en PHP"){.prog-lang-link}
[![Singleton en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Singleton en Python"){.prog-lang-link}
[![Singleton en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Singleton en Ruby"){.prog-lang-link}
[![Singleton en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Singleton en Rust"){.prog-lang-link}
[![Singleton en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Singleton en Swift"){.prog-lang-link}
[![Singleton en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Singleton en TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-fr" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive avec exemples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Achetez l'ebook **Plongée au cœur des patrons de conception** et obtenez
l'accès à une archive comprenant des dizaines d'exemples détaillés qui
peuvent être ouverts dans votre environnement de développement.

[ En savoir plus...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::
