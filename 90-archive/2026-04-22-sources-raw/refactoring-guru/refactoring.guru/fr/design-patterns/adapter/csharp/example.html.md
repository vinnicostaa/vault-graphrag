:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Adaptateur](../../adapter.html){.type} / [C#](../../csharp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Adaptateur](../../../../images/patterns/cards/adapter-minid866.png?id=b2ee4f681fb589be5a0685b94692aebb){srcset="/images/patterns/cards/adapter-mini-2x.png?id=8274d99afbbe9c63bfbfd0d68ceeffc7 2x"}
:::

# **Adaptateur** en C# {#adaptateur-en-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
L'**Adaptateur** est un patron de conception structurel qui permet à des
objets incompatibles de collaborer.

L'adaptateur fait office d'emballeur entre les deux objets. Il récupère
les appels à un objet et les met dans un format et une interface
reconnaissables par le second objet.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Adaptateur ](../../adapter.html){.btn
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
 [Program](#example-0--Program-cs)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Complexité :** []{.fa-stars}

**Popularité :** []{.fa-stars}

**Exemples d'utilisation :** L'adaptateur est très répandu en C#. On le
retrouve souvent dans des systèmes basés sur du code hérité, dans
lesquels l'adaptateur fait fonctionner du code hérité avec des classes
modernes.

**Identification :** L'adaptateur peut être identifié grâce à son
constructeur qui prend une instance d'un type abstrait différent ou
d'une interface différente. Lorsque l'une des méthodes de l'adaptateur
est appelée, il traduit les paramètres dans un format approprié et
redirige l'appel vers une ou plusieurs méthodes de l'objet emballé.
:::::::::::

[]{#example-0}

## Exemple conceptuel {#example-0-title}

Dans cet exemple, nous allons voir la structure du patron de conception
**Adaptateur**. Nous allons répondre aux questions suivantes :

-   Que contiennent les classes ?
-   Quel rôle jouent-elles ?
-   Comment les éléments du patron sont-ils reliés ?

#### []{#example-0--Program-cs .anchor} **Program.cs:** Exemple conceptuel

<figure class="code">
<pre class="code" lang="csharp"><code>using System;

namespace RefactoringGuru.DesignPatterns.Adapter.Conceptual
{
    // The Target defines the domain-specific interface used by the client code.
    public interface ITarget
    {
        string GetRequest();
    }

    // The Adaptee contains some useful behavior, but its interface is
    // incompatible with the existing client code. The Adaptee needs some
    // adaptation before the client code can use it.
    class Adaptee
    {
        public string GetSpecificRequest()
        {
            return &quot;Specific request.&quot;;
        }
    }

    // The Adapter makes the Adaptee&#39;s interface compatible with the Target&#39;s
    // interface.
    class Adapter : ITarget
    {
        private readonly Adaptee _adaptee;

        public Adapter(Adaptee adaptee)
        {
            this._adaptee = adaptee;
        }

        public string GetRequest()
        {
            return $&quot;This is &#39;{this._adaptee.GetSpecificRequest()}&#39;&quot;;
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            Adaptee adaptee = new Adaptee();
            ITarget target = new Adapter(adaptee);

            Console.WriteLine(&quot;Adaptee interface is incompatible with the client.&quot;);
            Console.WriteLine(&quot;But with adapter client can call it&#39;s method.&quot;);

            Console.WriteLine(target.GetRequest());
        }
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>Adaptee interface is incompatible with the client.
But with adapter client can call it&#39;s method.
This is &#39;Specific request.&#39;</code></pre>
</figure>

::: next
#### Suivant

[Pont en C# []{.fa
.fa-arrow-right}](../../bridge/csharp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Singleton en
C#](../../singleton/csharp/example.html){.btn .btn-default rel="prev"}
:::

## **Adaptateur** dans les autres langues

[![Adaptateur en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Adaptateur en C++"){.prog-lang-link}
[![Adaptateur en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Adaptateur en Go"){.prog-lang-link}
[![Adaptateur en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Adaptateur en Java"){.prog-lang-link}
[![Adaptateur en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Adaptateur en PHP"){.prog-lang-link}
[![Adaptateur en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Adaptateur en Python"){.prog-lang-link}
[![Adaptateur en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Adaptateur en Ruby"){.prog-lang-link}
[![Adaptateur en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Adaptateur en Rust"){.prog-lang-link}
[![Adaptateur en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Adaptateur en Swift"){.prog-lang-link}
[![Adaptateur en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Adaptateur en TypeScript"){.prog-lang-link}
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
