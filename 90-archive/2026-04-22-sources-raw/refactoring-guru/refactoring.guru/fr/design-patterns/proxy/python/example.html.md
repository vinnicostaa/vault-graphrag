:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Procuration](../../proxy.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Procuration](../../../../images/patterns/cards/proxy-minid8d7.png?id=25890b11e7dc5af29625ccd0678b63a8){srcset="/images/patterns/cards/proxy-mini-2x.png?id=8638fac9dc08c992852492f9cb29d9c6 2x"}
:::

# **Procuration** en Python {#procuration-en-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
La **Procuration** est un patron de conception structurel qui fournit un
objet qui agit comme un substitut pour un objet du service utilisé par
un client. Une procuration reçoit les demandes d'un client, effectue des
tâches (contrôle des accès, mise en cache, etc.) et passe ensuite la
demande à un objet du service.

L'objet Procuration possède la même interface qu'un service, ce qui le
rend interchangeable avec un vrai objet lorsqu'il est passé à un client.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Procuration ](../../proxy.html){.btn
.btn-lg .btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
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
 [main](#example-0--main-py)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Complexité :** []{.fa-stars}

**Popularité :** []{.fa-stars}

**Exemples d'utilisation :** La procuration n'est pas souvent invitée en
Python, mais elle se montre très pratique dans certains cas. Elle est
incontournable lorsque vous voulez ajouter de nouveaux comportements à
un objet d'une classe existante sans modifier le code client.

**Identification :** Les procurations délèguent tout le travail à un
autre objet. Chaque méthode de la procuration devrait au final, faire
référence à un objet du service, sauf si la procuration est une
sous-classe d'un service.
:::::::::::

[]{#example-0}

## Exemple conceptuel {#example-0-title}

Dans cet exemple, nous allons voir la structure de la **Procuration**.
Nous allons répondre aux questions suivantes :

-   Que contiennent les classes ?
-   Quels rôles jouent-elles ?
-   Comment les éléments du patron sont-ils reliés ?

#### []{#example-0--main-py .anchor} **main.py:** Exemple conceptuel

<figure class="code">
<pre class="code" lang="python"><code>from abc import ABC, abstractmethod


class Subject(ABC):
    &quot;&quot;&quot;
    The Subject interface declares common operations for both RealSubject and
    the Proxy. As long as the client works with RealSubject using this
    interface, you&#39;ll be able to pass it a proxy instead of a real subject.
    &quot;&quot;&quot;

    @abstractmethod
    def request(self) -&gt; None:
        pass


class RealSubject(Subject):
    &quot;&quot;&quot;
    The RealSubject contains some core business logic. Usually, RealSubjects are
    capable of doing some useful work which may also be very slow or sensitive -
    e.g. correcting input data. A Proxy can solve these issues without any
    changes to the RealSubject&#39;s code.
    &quot;&quot;&quot;

    def request(self) -&gt; None:
        print(&quot;RealSubject: Handling request.&quot;)


class Proxy(Subject):
    &quot;&quot;&quot;
    The Proxy has an interface identical to the RealSubject.
    &quot;&quot;&quot;

    def __init__(self, real_subject: RealSubject) -&gt; None:
        self._real_subject = real_subject

    def request(self) -&gt; None:
        &quot;&quot;&quot;
        The most common applications of the Proxy pattern are lazy loading,
        caching, controlling the access, logging, etc. A Proxy can perform one
        of these things and then, depending on the result, pass the execution to
        the same method in a linked RealSubject object.
        &quot;&quot;&quot;

        if self.check_access():
            self._real_subject.request()
            self.log_access()

    def check_access(self) -&gt; bool:
        print(&quot;Proxy: Checking access prior to firing a real request.&quot;)
        return True

    def log_access(self) -&gt; None:
        print(&quot;Proxy: Logging the time of request.&quot;, end=&quot;&quot;)


def client_code(subject: Subject) -&gt; None:
    &quot;&quot;&quot;
    The client code is supposed to work with all objects (both subjects and
    proxies) via the Subject interface in order to support both real subjects
    and proxies. In real life, however, clients mostly work with their real
    subjects directly. In this case, to implement the pattern more easily, you
    can extend your proxy from the real subject&#39;s class.
    &quot;&quot;&quot;

    # ...

    subject.request()

    # ...


if __name__ == &quot;__main__&quot;:
    print(&quot;Client: Executing the client code with a real subject:&quot;)
    real_subject = RealSubject()
    client_code(real_subject)

    print(&quot;&quot;)

    print(&quot;Client: Executing the same client code with a proxy:&quot;)
    proxy = Proxy(real_subject)
    client_code(proxy)</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>Client: Executing the client code with a real subject:
RealSubject: Handling request.

Client: Executing the same client code with a proxy:
Proxy: Checking access prior to firing a real request.
RealSubject: Handling request.
Proxy: Logging the time of request.</code></pre>
</figure>

::: next
#### Suivant

[Chaîne de responsabilité en Python []{.fa
.fa-arrow-right}](../../chain-of-responsibility/python/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Poids mouche en
Python](../../flyweight/python/example.html){.btn .btn-default
rel="prev"}
:::

## **Procuration** dans les autres langues

[![Procuration en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Procuration en C#"){.prog-lang-link}
[![Procuration en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Procuration en C++"){.prog-lang-link}
[![Procuration en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Procuration en Go"){.prog-lang-link}
[![Procuration en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Procuration en Java"){.prog-lang-link}
[![Procuration en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Procuration en PHP"){.prog-lang-link}
[![Procuration en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Procuration en Ruby"){.prog-lang-link}
[![Procuration en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Procuration en Rust"){.prog-lang-link}
[![Procuration en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Procuration en Swift"){.prog-lang-link}
[![Procuration en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Procuration en TypeScript"){.prog-lang-link}
:::::::::::::::::

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
:::::::::::::::::::::::
::::::::::::::::::::::::
