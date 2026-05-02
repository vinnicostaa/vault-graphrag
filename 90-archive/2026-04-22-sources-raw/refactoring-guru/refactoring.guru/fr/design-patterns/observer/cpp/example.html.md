:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Observateur](../../observer.html){.type} / [C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Observateur](../../../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x"}
:::

# **Observateur** en C++ {#observateur-en-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
L'**Observateur** est un patron de conception comportemental qui permet
à certains objets d'envoyer des notifications concernant leur état à
d'autres objets.

Ce patron fournit la possibilité aux objets qui implémentent une
interface de souscription, de s'inscrire et de se désinscrire de ces
événements.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Observateur ](../../observer.html){.btn
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
 [main](#example-0--main-cc)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Complexité :** []{.fa-stars}

**Popularité :** []{.fa-stars}

**Exemples d'utilisation :** L'observateur est assez répandu en C++,
surtout dans les composants GUI. Il fournit une manière de réagir aux
événements qui se produisent chez d'autres objets sans se coupler à
leurs classes.

**Identification :** Ce patron peut être reconnu dans les méthodes de
souscription qui stockent des objets dans une liste et par les appels
des objets de cette liste à la méthode update.
:::::::::::

[]{#example-0}

## Exemple conceptuel {#example-0-title}

Dans cet exemple, nous allons voir la structure de l'**Observateur**.
Nous allons répondre aux questions suivantes :

-   Que contiennent les classes ?
-   Quels rôles jouent-elles ?
-   Comment les éléments du patron sont-ils reliés ?

#### []{#example-0--main-cc .anchor} **main.cc:** Exemple conceptuel

<figure class="code">
<pre class="code" lang="cpp"><code>/**
 * Observer Design Pattern
 *
 * Intent: Lets you define a subscription mechanism to notify multiple objects
 * about any events that happen to the object they&#39;re observing.
 *
 * Note that there&#39;s a lot of different terms with similar meaning associated
 * with this pattern. Just remember that the Subject is also called the
 * Publisher and the Observer is often called the Subscriber and vice versa.
 * Also the verbs &quot;observe&quot;, &quot;listen&quot; or &quot;track&quot; usually mean the same thing.
 */

#include &lt;iostream&gt;
#include &lt;list&gt;
#include &lt;string&gt;

class IObserver {
 public:
  virtual ~IObserver(){};
  virtual void Update(const std::string &amp;message_from_subject) = 0;
};

class ISubject {
 public:
  virtual ~ISubject(){};
  virtual void Attach(IObserver *observer) = 0;
  virtual void Detach(IObserver *observer) = 0;
  virtual void Notify() = 0;
};

/**
 * The Subject owns some important state and notifies observers when the state
 * changes.
 */

class Subject : public ISubject {
 public:
  virtual ~Subject() {
    std::cout &lt;&lt; &quot;Goodbye, I was the Subject.\n&quot;;
  }

  /**
   * The subscription management methods.
   */
  void Attach(IObserver *observer) override {
    list_observer_.push_back(observer);
  }
  void Detach(IObserver *observer) override {
    list_observer_.remove(observer);
  }
  void Notify() override {
    std::list&lt;IObserver *&gt;::iterator iterator = list_observer_.begin();
    HowManyObserver();
    while (iterator != list_observer_.end()) {
      (*iterator)-&gt;Update(message_);
      ++iterator;
    }
  }

  void CreateMessage(std::string message = &quot;Empty&quot;) {
    this-&gt;message_ = message;
    Notify();
  }
  void HowManyObserver() {
    std::cout &lt;&lt; &quot;There are &quot; &lt;&lt; list_observer_.size() &lt;&lt; &quot; observers in the list.\n&quot;;
  }

  /**
   * Usually, the subscription logic is only a fraction of what a Subject can
   * really do. Subjects commonly hold some important business logic, that
   * triggers a notification method whenever something important is about to
   * happen (or after it).
   */
  void SomeBusinessLogic() {
    this-&gt;message_ = &quot;change message message&quot;;
    Notify();
    std::cout &lt;&lt; &quot;I&#39;m about to do some thing important\n&quot;;
  }

 private:
  std::list&lt;IObserver *&gt; list_observer_;
  std::string message_;
};

class Observer : public IObserver {
 public:
  Observer(Subject &amp;subject) : subject_(subject) {
    this-&gt;subject_.Attach(this);
    std::cout &lt;&lt; &quot;Hi, I&#39;m the Observer \&quot;&quot; &lt;&lt; ++Observer::static_number_ &lt;&lt; &quot;\&quot;.\n&quot;;
    this-&gt;number_ = Observer::static_number_;
  }
  virtual ~Observer() {
    std::cout &lt;&lt; &quot;Goodbye, I was the Observer \&quot;&quot; &lt;&lt; this-&gt;number_ &lt;&lt; &quot;\&quot;.\n&quot;;
  }

  void Update(const std::string &amp;message_from_subject) override {
    message_from_subject_ = message_from_subject;
    PrintInfo();
  }
  void RemoveMeFromTheList() {
    subject_.Detach(this);
    std::cout &lt;&lt; &quot;Observer \&quot;&quot; &lt;&lt; number_ &lt;&lt; &quot;\&quot; removed from the list.\n&quot;;
  }
  void PrintInfo() {
    std::cout &lt;&lt; &quot;Observer \&quot;&quot; &lt;&lt; this-&gt;number_ &lt;&lt; &quot;\&quot;: a new message is available --&gt; &quot; &lt;&lt; this-&gt;message_from_subject_ &lt;&lt; &quot;\n&quot;;
  }

 private:
  std::string message_from_subject_;
  Subject &amp;subject_;
  static int static_number_;
  int number_;
};

int Observer::static_number_ = 0;

void ClientCode() {
  Subject *subject = new Subject;
  Observer *observer1 = new Observer(*subject);
  Observer *observer2 = new Observer(*subject);
  Observer *observer3 = new Observer(*subject);
  Observer *observer4;
  Observer *observer5;

  subject-&gt;CreateMessage(&quot;Hello World! :D&quot;);
  observer3-&gt;RemoveMeFromTheList();

  subject-&gt;CreateMessage(&quot;The weather is hot today! :p&quot;);
  observer4 = new Observer(*subject);

  observer2-&gt;RemoveMeFromTheList();
  observer5 = new Observer(*subject);

  subject-&gt;CreateMessage(&quot;My new car is great! ;)&quot;);
  observer5-&gt;RemoveMeFromTheList();

  observer4-&gt;RemoveMeFromTheList();
  observer1-&gt;RemoveMeFromTheList();

  delete observer5;
  delete observer4;
  delete observer3;
  delete observer2;
  delete observer1;
  delete subject;
}

int main() {
  ClientCode();
  return 0;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>Hi, I&#39;m the Observer &quot;1&quot;.
Hi, I&#39;m the Observer &quot;2&quot;.
Hi, I&#39;m the Observer &quot;3&quot;.
There are 3 observers in the list.
Observer &quot;1&quot;: a new message is available --&gt; Hello World! :D
Observer &quot;2&quot;: a new message is available --&gt; Hello World! :D
Observer &quot;3&quot;: a new message is available --&gt; Hello World! :D
Observer &quot;3&quot; removed from the list.
There are 2 observers in the list.
Observer &quot;1&quot;: a new message is available --&gt; The weather is hot today! :p
Observer &quot;2&quot;: a new message is available --&gt; The weather is hot today! :p
Hi, I&#39;m the Observer &quot;4&quot;.
Observer &quot;2&quot; removed from the list.
Hi, I&#39;m the Observer &quot;5&quot;.
There are 3 observers in the list.
Observer &quot;1&quot;: a new message is available --&gt; My new car is great! ;)
Observer &quot;4&quot;: a new message is available --&gt; My new car is great! ;)
Observer &quot;5&quot;: a new message is available --&gt; My new car is great! ;)
Observer &quot;5&quot; removed from the list.
Observer &quot;4&quot; removed from the list.
Observer &quot;1&quot; removed from the list.
Goodbye, I was the Observer &quot;5&quot;.
Goodbye, I was the Observer &quot;4&quot;.
Goodbye, I was the Observer &quot;3&quot;.
Goodbye, I was the Observer &quot;2&quot;.
Goodbye, I was the Observer &quot;1&quot;.
Goodbye, I was the Subject.</code></pre>
</figure>

::: next
#### Suivant

[État en C++ []{.fa .fa-arrow-right}](../../state/cpp/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Mémento en
C++](../../memento/cpp/example.html){.btn .btn-default rel="prev"}
:::

## **Observateur** dans les autres langues

[![Observateur en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Observateur en C#"){.prog-lang-link}
[![Observateur en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Observateur en Go"){.prog-lang-link}
[![Observateur en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Observateur en Java"){.prog-lang-link}
[![Observateur en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Observateur en PHP"){.prog-lang-link}
[![Observateur en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Observateur en Python"){.prog-lang-link}
[![Observateur en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Observateur en Ruby"){.prog-lang-link}
[![Observateur en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Observateur en Rust"){.prog-lang-link}
[![Observateur en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Observateur en Swift"){.prog-lang-link}
[![Observateur en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Observateur en TypeScript"){.prog-lang-link}
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
