:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Memento](../../memento.html){.type} / [C#](../../csharp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Memento](../../../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x"}
:::

# **Memento** を C# で {#memento-を-c-で .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Memento** は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}オブジェクトの状態のスナップショットを作成し[、]{.chpule2}[
]{.chpuri2}それを将来復元します[。]{.chpule2}[ ]{.chpuri2}

Memento は[、]{.chpule2}[
]{.chpuri2}その対象オブジェクトの内部構造やスナップショットの内部に保存されるデータの機密を守ります[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Memento の詳細 ](../../memento.html){.btn .btn-lg .btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [概念的な例](#example-0)
:::

::: en-file
 [Program](#example-0--Program-cs)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}** Memento の原則は[、]{.chpule2}[
]{.chpuri2}シリアライゼーションを使って達成することが可能で[、]{.chpule2}[
]{.chpuri2}C# ではよく見かけます[。]{.chpule2}[
]{.chpuri2}これは[、]{.chpule2}[
]{.chpuri2}オブジェクトの状態のスナップショットを作る上で唯一の方法でも[、]{.chpule2}[
]{.chpuri2}最も効率のいい方法でもありませんが[、]{.chpule2}[
]{.chpuri2}オリジネーターの構造を他のオブジェクトから隠蔽しつつ状態のバックアップを取ることはできます[。]{.chpule2}[
]{.chpuri2}
:::::::::::

[]{#example-0}

## 概念的な例 {#example-0-title}

この例は[、]{.chpule2}[ ]{.chpuri2}**Memento**
デザインパターンの構造を説明するためのものです[。]{.chpule2}[
]{.chpuri2}以下の質問に答えることを目的としています[：]{.chpule2}[
]{.chpuri2}

-   どういうクラスからできているか[？]{.chpule2}[ ]{.chpuri2}
-   それぞれのクラスの役割は[？]{.chpule2}[ ]{.chpuri2}
-   パターンの要素同士はどう関係しているのか[？]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--Program-cs .anchor} **Program.cs:** 概念的な例

<figure class="code">
<pre class="code" lang="csharp"><code>using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;

namespace RefactoringGuru.DesignPatterns.Memento.Conceptual
{
    // The Originator holds some important state that may change over time. It
    // also defines a method for saving the state inside a memento and another
    // method for restoring the state from it.
    class Originator
    {
        // For the sake of simplicity, the originator&#39;s state is stored inside a
        // single variable.
        private string _state;

        public Originator(string state)
        {
            this._state = state;
            Console.WriteLine(&quot;Originator: My initial state is: &quot; + state);
        }

        // The Originator&#39;s business logic may affect its internal state.
        // Therefore, the client should backup the state before launching
        // methods of the business logic via the save() method.
        public void DoSomething()
        {
            Console.WriteLine(&quot;Originator: I&#39;m doing something important.&quot;);
            this._state = this.GenerateRandomString(30);
            Console.WriteLine($&quot;Originator: and my state has changed to: {_state}&quot;);
        }

        private string GenerateRandomString(int length = 10)
        {
            string allowedSymbols = &quot;abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ&quot;;
            string result = string.Empty;

            while (length &gt; 0)
            {
                result += allowedSymbols[new Random().Next(0, allowedSymbols.Length)];

                Thread.Sleep(12);

                length--;
            }

            return result;
        }

        // Saves the current state inside a memento.
        public IMemento Save()
        {
            return new ConcreteMemento(this._state);
        }

        // Restores the Originator&#39;s state from a memento object.
        public void Restore(IMemento memento)
        {
            if (!(memento is ConcreteMemento))
            {
                throw new Exception(&quot;Unknown memento class &quot; + memento.ToString());
            }

            this._state = memento.GetState();
            Console.Write($&quot;Originator: My state has changed to: {_state}&quot;);
        }
    }

    // The Memento interface provides a way to retrieve the memento&#39;s metadata,
    // such as creation date or name. However, it doesn&#39;t expose the
    // Originator&#39;s state.
    public interface IMemento
    {
        string GetName();

        string GetState();

        DateTime GetDate();
    }

    // The Concrete Memento contains the infrastructure for storing the
    // Originator&#39;s state.
    class ConcreteMemento : IMemento
    {
        private string _state;

        private DateTime _date;

        public ConcreteMemento(string state)
        {
            this._state = state;
            this._date = DateTime.Now;
        }

        // The Originator uses this method when restoring its state.
        public string GetState()
        {
            return this._state;
        }
        
        // The rest of the methods are used by the Caretaker to display
        // metadata.
        public string GetName()
        {
            return $&quot;{this._date} / ({this._state.Substring(0, 9)})...&quot;;
        }

        public DateTime GetDate()
        {
            return this._date;
        }
    }

    // The Caretaker doesn&#39;t depend on the Concrete Memento class. Therefore, it
    // doesn&#39;t have access to the originator&#39;s state, stored inside the memento.
    // It works with all mementos via the base Memento interface.
    class Caretaker
    {
        private List&lt;IMemento&gt; _mementos = new List&lt;IMemento&gt;();

        private Originator _originator = null;

        public Caretaker(Originator originator)
        {
            this._originator = originator;
        }

        public void Backup()
        {
            Console.WriteLine(&quot;\nCaretaker: Saving Originator&#39;s state...&quot;);
            this._mementos.Add(this._originator.Save());
        }

        public void Undo()
        {
            if (this._mementos.Count == 0)
            {
                return;
            }

            var memento = this._mementos.Last();
            this._mementos.Remove(memento);

            Console.WriteLine(&quot;Caretaker: Restoring state to: &quot; + memento.GetName());

            try
            {
                this._originator.Restore(memento);
            }
            catch (Exception)
            {
                this.Undo();
            }
        }

        public void ShowHistory()
        {
            Console.WriteLine(&quot;Caretaker: Here&#39;s the list of mementos:&quot;);

            foreach (var memento in this._mementos)
            {
                Console.WriteLine(memento.GetName());
            }
        }
    }
    
    class Program
    {
        static void Main(string[] args)
        {
            // Client code.
            Originator originator = new Originator(&quot;Super-duper-super-puper-super.&quot;);
            Caretaker caretaker = new Caretaker(originator);

            caretaker.Backup();
            originator.DoSomething();

            caretaker.Backup();
            originator.DoSomething();

            caretaker.Backup();
            originator.DoSomething();

            Console.WriteLine();
            caretaker.ShowHistory();

            Console.WriteLine(&quot;\nClient: Now, let&#39;s rollback!\n&quot;);
            caretaker.Undo();

            Console.WriteLine(&quot;\n\nClient: Once more!\n&quot;);
            caretaker.Undo();

            Console.WriteLine();
        }
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Originator: My initial state is: Super-duper-super-puper-super.

Caretaker: Saving Originator&#39;s state...
Originator: I&#39;m doing something important.
Originator: and my state has changed to: oGyQIIatlDDWNgYYqJATTmdwnnGZQj

Caretaker: Saving Originator&#39;s state...
Originator: I&#39;m doing something important.
Originator: and my state has changed to: jBtMDDWogzzRJbTTmEwOOhZrjjBULe

Caretaker: Saving Originator&#39;s state...
Originator: I&#39;m doing something important.
Originator: and my state has changed to: exoHyyRkbuuNEXOhhArKccUmexPPHZ

Caretaker: Here&#39;s the list of mementos:
12.06.2018 15:52:45 / (Super-dup...)
12.06.2018 15:52:46 / (oGyQIIatl...)
12.06.2018 15:52:46 / (jBtMDDWog...)

Client: Now, let&#39;s rollback!

Caretaker: Restoring state to: 12.06.2018 15:52:46 / (jBtMDDWog...)
Originator: My state has changed to: jBtMDDWogzzRJbTTmEwOOhZrjjBULe

Client: Once more!

Caretaker: Restoring state to: 12.06.2018 15:52:46 / (oGyQIIatl...)
Originator: My state has changed to: oGyQIIatlDDWNgYYqJATTmdwnnGZQj</code></pre>
</figure>

::: next
#### 次を読む

[Observer を C# で []{.fa
.fa-arrow-right}](../../observer/csharp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Mediator を C#
で](../../mediator/csharp/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Memento**

[![Memento を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Memento を C++ で"){.prog-lang-link}
[![Memento を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Memento を Go で"){.prog-lang-link}
[![Memento を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Memento を Java で"){.prog-lang-link}
[![Memento を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Memento を PHP で"){.prog-lang-link}
[![Memento を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Memento を Python で"){.prog-lang-link}
[![Memento を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Memento を Ruby で"){.prog-lang-link}
[![Memento を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Memento を Rust で"){.prog-lang-link}
[![Memento を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Memento を Swift で"){.prog-lang-link}
[![Memento を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Memento を TypeScript で"){.prog-lang-link}
:::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ja" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ コード例の書庫](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
**直撃！デザインパターン**を電子書籍で購入し、IDE
で開くことができる数多くの詳細な例を含む書庫にアクセス。

[ 詳細](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
