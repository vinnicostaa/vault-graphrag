:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Polecenie](../../command.html){.type} / [C#](../../csharp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Polecenie](../../../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x"}
:::

# **Polecenie** w języku C# {#polecenie-w-języku-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Polecenie** to behawioralny wzorzec projektowy według którego żądania
lub proste działania są konwertowane na obiekty.

Wyżej wymieniona konwersja pozwala odkładać zadania w czasie,
uruchamiać je zdalnie, przechowywać ich historię, itp.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Polecenie ](../../command.html){.btn .btn-lg
.btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Przykład koncepcyjny](#example-0)
:::

::: en-file
 [Program](#example-0--Program-cs)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Złożoność:** []{.fa-stars}

**Popularność:** []{.fa-stars}

**Przykłady użycia:** Wzorzec Polecenie jest dość powszechny w kodzie
napisanym w C# jako alternatywa dla wywołania zwrotnego (ang. callback)
zakładającego przypisanie akcji elementom interfejsu użytkownika. Jest
stosowany także w celu kolejkowania zadań, śledzenia historii wykonanych
działań, itp.

**Identyfikacja:** Jeśli widzisz zestaw powiązanych klas odpowiadających
konkretnym czynnościom (jak "Kopiuj", "Wytnij", "Wyślij", "Drukuj",
itd.) --- może to oznaczać użycie wzorca Polecenie. Takie klasy powinny
implementować ten sam interfejs/klasę abstrakcyjną. Polecenia mogą
samodzielnie implementować poszczególne czynności lub delegować zadania
osobnym obiektom --- zwanym odbiorcami. Ostatnim elementem układanki
jest znalezienie obiektu wywołującego --- będzie to klasa przyjmująca
obiekty-polecenia w charakterze parametrów swoich metod lub
konstruktora.
:::::::::::

[]{#example-0}

## Przykład koncepcyjny {#example-0-title}

Poniższy przykład ilustruje strukturę wzorca **Polecenie** ze
szczególnym naciskiem na następujące kwestie:

-   Z jakich składa się klas?
-   Jakie role pełnią te klasy?
-   W jaki sposób elementy wzorca są ze sobą powiązane?

#### []{#example-0--Program-cs .anchor} **Program.cs:** Przykład koncepcyjny

<figure class="code">
<pre class="code" lang="csharp"><code>using System;

namespace RefactoringGuru.DesignPatterns.Command.Conceptual
{
    // The Command interface declares a method for executing a command.
    public interface ICommand
    {
        void Execute();
    }

    // Some commands can implement simple operations on their own.
    class SimpleCommand : ICommand
    {
        private string _payload = string.Empty;

        public SimpleCommand(string payload)
        {
            this._payload = payload;
        }

        public void Execute()
        {
            Console.WriteLine($&quot;SimpleCommand: See, I can do simple things like printing ({this._payload})&quot;);
        }
    }

    // However, some commands can delegate more complex operations to other
    // objects, called &quot;receivers.&quot;
    class ComplexCommand : ICommand
    {
        private Receiver _receiver;

        // Context data, required for launching the receiver&#39;s methods.
        private string _a;

        private string _b;

        // Complex commands can accept one or several receiver objects along
        // with any context data via the constructor.
        public ComplexCommand(Receiver receiver, string a, string b)
        {
            this._receiver = receiver;
            this._a = a;
            this._b = b;
        }

        // Commands can delegate to any methods of a receiver.
        public void Execute()
        {
            Console.WriteLine(&quot;ComplexCommand: Complex stuff should be done by a receiver object.&quot;);
            this._receiver.DoSomething(this._a);
            this._receiver.DoSomethingElse(this._b);
        }
    }

    // The Receiver classes contain some important business logic. They know how
    // to perform all kinds of operations, associated with carrying out a
    // request. In fact, any class may serve as a Receiver.
    class Receiver
    {
        public void DoSomething(string a)
        {
            Console.WriteLine($&quot;Receiver: Working on ({a}.)&quot;);
        }

        public void DoSomethingElse(string b)
        {
            Console.WriteLine($&quot;Receiver: Also working on ({b}.)&quot;);
        }
    }

    // The Invoker is associated with one or several commands. It sends a
    // request to the command.
    class Invoker
    {
        private ICommand _onStart;

        private ICommand _onFinish;

        // Initialize commands.
        public void SetOnStart(ICommand command)
        {
            this._onStart = command;
        }

        public void SetOnFinish(ICommand command)
        {
            this._onFinish = command;
        }

        // The Invoker does not depend on concrete command or receiver classes.
        // The Invoker passes a request to a receiver indirectly, by executing a
        // command.
        public void DoSomethingImportant()
        {
            Console.WriteLine(&quot;Invoker: Does anybody want something done before I begin?&quot;);
            if (this._onStart is ICommand)
            {
                this._onStart.Execute();
            }
            
            Console.WriteLine(&quot;Invoker: ...doing something really important...&quot;);
            
            Console.WriteLine(&quot;Invoker: Does anybody want something done after I finish?&quot;);
            if (this._onFinish is ICommand)
            {
                this._onFinish.Execute();
            }
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            // The client code can parameterize an invoker with any commands.
            Invoker invoker = new Invoker();
            invoker.SetOnStart(new SimpleCommand(&quot;Say Hi!&quot;));
            Receiver receiver = new Receiver();
            invoker.SetOnFinish(new ComplexCommand(receiver, &quot;Send email&quot;, &quot;Save report&quot;));

            invoker.DoSomethingImportant();
        }
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Invoker: Does anybody want something done before I begin?
SimpleCommand: See, I can do simple things like printing (Say Hi!)
Invoker: ...doing something really important...
Invoker: Does anybody want something done after I finish?
ComplexCommand: Complex stuff should be done by a receiver object.
Receiver: Working on (Send email.)
Receiver: Also working on (Save report.)</code></pre>
</figure>

::: next
#### Czytaj dalej

[Iterator w języku C# []{.fa
.fa-arrow-right}](../../iterator/csharp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Łańcuch zobowiązań w języku
C#](../../chain-of-responsibility/csharp/example.html){.btn .btn-default
rel="prev"}
:::

## **Polecenie** w innych językach

[![Polecenie w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Polecenie w języku C++"){.prog-lang-link}
[![Polecenie w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Polecenie w języku Go"){.prog-lang-link}
[![Polecenie w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Polecenie w języku Java"){.prog-lang-link}
[![Polecenie w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Polecenie w języku PHP"){.prog-lang-link}
[![Polecenie w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Polecenie w języku Python"){.prog-lang-link}
[![Polecenie w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Polecenie w języku Ruby"){.prog-lang-link}
[![Polecenie w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Polecenie w języku Rust"){.prog-lang-link}
[![Polecenie w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Polecenie w języku Swift"){.prog-lang-link}
[![Polecenie w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Polecenie w języku TypeScript"){.prog-lang-link}
:::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pl" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archiwum\
przykładów kodu](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Kupując eBook **Wzorce projektowe. Nowoczesny podręcznik** zyskujesz
dostęp do archiwum zawierającego mnóstwo szczegółowych przykładów, które
można od razu wkleić do swojego preferowanego zintegrowanego środowiska
programistycznego.

[ Dowiedz się więcej...](../../book.html){.btn .btn-secondary
.btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
