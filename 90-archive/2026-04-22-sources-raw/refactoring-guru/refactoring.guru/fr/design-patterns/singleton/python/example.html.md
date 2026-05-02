:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Singleton](../../singleton.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Singleton](../../../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x"}
:::

# **Singleton** en Python {#singleton-en-python .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
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

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Singleton naïf](#example-0)
:::

::: en-file
 [main](#example-0--main-py)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Singleton thread-safe](#example-1)
:::

::: en-file
 [main](#example-1--main-py)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Complexité :** []{.fa-stars}

**Popularité :** []{.fa-stars}

**Exemples d'utilisation :** Beaucoup de développeurs considèrent que le
singleton est un antipatron. C'est pourquoi il est de moins en moins
utilisé en Python.

**Identification :** Le singleton peut être reconnu par une méthode de
création statique qui retourne le même objet en cache.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Singleton naïf](#example-0){#example-0-tab .nav-item .nav-link .active
toggle="tab" role="tab" aria-controls="example-0" aria-selected="true"}
[Singleton thread-safe](#example-1){#example-1-tab .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Singleton naïf {#example-0-title}

Il est très facile d'implémenter un singleton bâclé. Il suffit de cacher
le constructeur et d'implémenter une méthode de création statique.

Cette même classe peut mal fonctionner dans un environnement
multithread. Plusieurs threads vont pouvoir appeler la méthode de
création simultanément et créer plusieurs instances de la classe
Singleton.

#### []{#example-0--main-py .anchor} **main.py:** Exemple conceptuel

<figure class="code">
<pre class="code" lang="python"><code>class SingletonMeta(type):
    &quot;&quot;&quot;
    The Singleton class can be implemented in different ways in Python. Some
    possible methods include: base class, decorator, metaclass. We will use the
    metaclass because it is best suited for this purpose.
    &quot;&quot;&quot;

    _instances = {}

    def __call__(cls, *args, **kwargs):
        &quot;&quot;&quot;
        Possible changes to the value of the `__init__` argument do not affect
        the returned instance.
        &quot;&quot;&quot;
        if cls not in cls._instances:
            instance = super().__call__(*args, **kwargs)
            cls._instances[cls] = instance
        return cls._instances[cls]


class Singleton(metaclass=SingletonMeta):
    def some_business_logic(self):
        &quot;&quot;&quot;
        Finally, any singleton should define some business logic, which can be
        executed on its instance.
        &quot;&quot;&quot;

        # ...


if __name__ == &quot;__main__&quot;:
    # The client code.

    s1 = Singleton()
    s2 = Singleton()

    if id(s1) == id(s2):
        print(&quot;Singleton works, both variables contain the same instance.&quot;)
    else:
        print(&quot;Singleton failed, variables contain different instances.&quot;)</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>Singleton works, both variables contain the same instance.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Singleton thread-safe {#example-1-title}

Pour régler ce problème, vous devez synchroniser les threads lors de la
première création de l'objet Singleton.

#### []{#example-1--main-py .anchor} **main.py:** Exemple conceptuel

<figure class="code">
<pre class="code" lang="python"><code>from threading import Lock, Thread


class SingletonMeta(type):
    &quot;&quot;&quot;
    This is a thread-safe implementation of Singleton.
    &quot;&quot;&quot;

    _instances = {}

    _lock: Lock = Lock()
    &quot;&quot;&quot;
    We now have a lock object that will be used to synchronize threads during
    first access to the Singleton.
    &quot;&quot;&quot;

    def __call__(cls, *args, **kwargs):
        &quot;&quot;&quot;
        Possible changes to the value of the `__init__` argument do not affect
        the returned instance.
        &quot;&quot;&quot;
        # Now, imagine that the program has just been launched. Since there&#39;s no
        # Singleton instance yet, multiple threads can simultaneously pass the
        # previous conditional and reach this point almost at the same time. The
        # first of them will acquire lock and will proceed further, while the
        # rest will wait here.
        with cls._lock:
            # The first thread to acquire the lock, reaches this conditional,
            # goes inside and creates the Singleton instance. Once it leaves the
            # lock block, a thread that might have been waiting for the lock
            # release may then enter this section. But since the Singleton field
            # is already initialized, the thread won&#39;t create a new object.
            if cls not in cls._instances:
                instance = super().__call__(*args, **kwargs)
                cls._instances[cls] = instance
        return cls._instances[cls]


class Singleton(metaclass=SingletonMeta):
    value: str = None
    &quot;&quot;&quot;
    We&#39;ll use this property to prove that our Singleton really works.
    &quot;&quot;&quot;

    def __init__(self, value: str) -&gt; None:
        self.value = value

    def some_business_logic(self):
        &quot;&quot;&quot;
        Finally, any singleton should define some business logic, which can be
        executed on its instance.
        &quot;&quot;&quot;


def test_singleton(value: str) -&gt; None:
    singleton = Singleton(value)
    print(singleton.value)


if __name__ == &quot;__main__&quot;:
    # The client code.

    print(&quot;If you see the same value, then singleton was reused (yay!)\n&quot;
          &quot;If you see different values, &quot;
          &quot;then 2 singletons were created (booo!!)\n\n&quot;
          &quot;RESULT:\n&quot;)

    process1 = Thread(target=test_singleton, args=(&quot;FOO&quot;,))
    process2 = Thread(target=test_singleton, args=(&quot;BAR&quot;,))
    process1.start()
    process2.start()</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>If you see the same value, then singleton was reused (yay!)
If you see different values, then 2 singletons were created (booo!!)

RESULT:

FOO
FOO</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Singleton naïf](#example-0){#example-0-tab-bottom .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Singleton
thread-safe](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Suivant

[Adaptateur en Python []{.fa
.fa-arrow-right}](../../adapter/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Prototype en
Python](../../prototype/python/example.html){.btn .btn-default
rel="prev"}
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
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Singleton en Go"){.prog-lang-link}
[![Singleton en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Singleton en Java"){.prog-lang-link}
[![Singleton en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Singleton en PHP"){.prog-lang-link}
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
:::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
