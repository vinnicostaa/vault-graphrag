::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Obserwator](../../observer.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Obserwator](../../../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x"}
:::

# **Obserwator** w języku Java {#obserwator-w-języku-java .pattern-example-title-block-title}
::::

:::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Obserwator** to behawioralny wzorzec projektowy pozwalający obiektom
powiadamiać inne obiekty o zmianach swojego stanu.

Obserwator daje możliwość subskrypcji lub zrezygnowania z subskrypcji
zdarzeń dowolnego obiektu implementującego interfejs subskrybenta.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Obserwator ](../../observer.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Subskrybowanie zdarzeń](#example-0)
:::

::: en-file
 publisher
:::

:::: en-inside
::: en-file
  [Event­Manager](#example-0--publisher-EventManager-java)
:::
::::

::: en-file
 editor
:::

:::: en-inside
::: en-file
  [Editor](#example-0--editor-Editor-java)
:::
::::

::: en-file
 listeners
:::

:::: en-inside
::: en-file
  [Event­Listener](#example-0--listeners-EventListener-java)
:::
::::

:::: en-inside
::: en-file
  [Email­Notification­Listener](#example-0--listeners-EmailNotificationListener-java)
:::
::::

:::: en-inside
::: en-file
  [Log­Open­Listener](#example-0--listeners-LogOpenListener-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-txt)
:::
:::::::::::::::::::::

**Złożoność:** []{.fa-stars}

**Popularność:** []{.fa-stars}

**Przykłady użycia:** Wzorzec Obserwator jest dość powszechny w kodzie
Java, szczególnie w komponentach graficznego interfejsu użytkownika.
Pozwala reagować na zdarzenia dotyczące innych obiektów bez konieczności
sprzęgania z klasami tych obiektów.

Oto parę przykładów użycia wzorca w głównych bibliotekach Java:

-   [`java.util.Observer`](http://docs.oracle.com/javase/8/docs/api/java/util/Observer.html)/[`java.util.Observable`](http://docs.oracle.com/javase/8/docs/api/java/util/Observable.html)
    (rzadko stosowane w prawdziwym życiu)
-   Wszystkie implementacje
    [`java.util.EventListener`](http://docs.oracle.com/javase/8/docs/api/java/util/EventListener.html)
    (praktycznie we wszystkich komponentach Swing)
-   [`javax.servlet.http.HttpSessionBindingListener`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpSessionBindingListener.html)
-   [`javax.servlet.http.HttpSessionAttributeListener`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpSessionAttributeListener.html)
-   [`javax.faces.event.PhaseListener`](http://docs.oracle.com/javaee/7/api/javax/faces/event/PhaseListener.html)

**Identyfikacja:** Wzorzec Obserwator można poznać po obecności metod
służących subskrypcji, które przechowują obiekty w strukturze listy i po
wywołaniach metod aktualizacji obiektów z tej listy.
::::::::::::::::::::::::

[]{#example-0}

## Subskrybowanie zdarzeń {#example-0-title}

W poniższym przykładzie, wzorzec Obserwator umożliwia pośrednią
współpracę pomiędzy obiektami edytora tekstu. Za każdym razem, gdy
obiekt `Editor` ulega zmianie, informuje o tym zdarzeniu subskrybentów.
Metody `EmailNotificationListener` oraz `LogOpenListener` reagują na te
powiadomienia przystępując do wykonywania swoich głównych zadań.

Klasy subskrybentów nie są sprzęgnięte z klasą edytora i mogą zostać
wykorzystane ponownie w innych aplikacjach, jeśli zachodzi taka
potrzeba. Klasa `Editor` zależy tylko od abstrakcyjnego interfejsu
subskrybenta. Pozwala to dodawać kolejne typy subskrybentów bez
konieczności zmiany kodu edytora.

### []{#example-0--publisher .anchor} **publisher** {#publisher .example-title}

#### []{#example-0--publisher-EventManager-java .anchor} **publisher/EventManager.java:** Bazowy publikujący

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.observer.example.publisher;

import refactoring_guru.observer.example.listeners.EventListener;

import java.io.File;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class EventManager {
    Map&lt;String, List&lt;EventListener&gt;&gt; listeners = new HashMap&lt;&gt;();

    public EventManager(String... operations) {
        for (String operation : operations) {
            this.listeners.put(operation, new ArrayList&lt;&gt;());
        }
    }

    public void subscribe(String eventType, EventListener listener) {
        List&lt;EventListener&gt; users = listeners.get(eventType);
        users.add(listener);
    }

    public void unsubscribe(String eventType, EventListener listener) {
        List&lt;EventListener&gt; users = listeners.get(eventType);
        users.remove(listener);
    }

    public void notify(String eventType, File file) {
        List&lt;EventListener&gt; users = listeners.get(eventType);
        for (EventListener listener : users) {
            listener.update(eventType, file);
        }
    }
}</code></pre>
</figure>

### []{#example-0--editor .anchor} **editor** {#editor .example-title}

#### []{#example-0--editor-Editor-java .anchor} **editor/Editor.java:** Konkretny publikujący, obserwowany przez inne obiekty

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.observer.example.editor;

import refactoring_guru.observer.example.publisher.EventManager;

import java.io.File;

public class Editor {
    public EventManager events;
    private File file;

    public Editor() {
        this.events = new EventManager(&quot;open&quot;, &quot;save&quot;);
    }

    public void openFile(String filePath) {
        this.file = new File(filePath);
        events.notify(&quot;open&quot;, file);
    }

    public void saveFile() throws Exception {
        if (this.file != null) {
            events.notify(&quot;save&quot;, file);
        } else {
            throw new Exception(&quot;Please open a file first.&quot;);
        }
    }
}</code></pre>
</figure>

### []{#example-0--listeners .anchor} **listeners** {#listeners .example-title}

#### []{#example-0--listeners-EventListener-java .anchor} **listeners/EventListener.java:** Wspólny interfejs obserwatora

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.observer.example.listeners;

import java.io.File;

public interface EventListener {
    void update(String eventType, File file);
}</code></pre>
</figure>

#### []{#example-0--listeners-EmailNotificationListener-java .anchor} **listeners/EmailNotificationListener.java:** Wysyła emaile otrzymawszy powiadomienie

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.observer.example.listeners;

import java.io.File;

public class EmailNotificationListener implements EventListener {
    private String email;

    public EmailNotificationListener(String email) {
        this.email = email;
    }

    @Override
    public void update(String eventType, File file) {
        System.out.println(&quot;Email to &quot; + email + &quot;: Someone has performed &quot; + eventType + &quot; operation with the following file: &quot; + file.getName());
    }
}</code></pre>
</figure>

#### []{#example-0--listeners-LogOpenListener-java .anchor} **listeners/LogOpenListener.java:** Zapisuje komunikat do dziennika otrzymawszy powiadomienie

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.observer.example.listeners;

import java.io.File;

public class LogOpenListener implements EventListener {
    private File log;

    public LogOpenListener(String fileName) {
        this.log = new File(fileName);
    }

    @Override
    public void update(String eventType, File file) {
        System.out.println(&quot;Save to log &quot; + log + &quot;: Someone has performed &quot; + eventType + &quot; operation with the following file: &quot; + file.getName());
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** Kod inicjalizacyjny

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.observer.example;

import refactoring_guru.observer.example.editor.Editor;
import refactoring_guru.observer.example.listeners.EmailNotificationListener;
import refactoring_guru.observer.example.listeners.LogOpenListener;

public class Demo {
    public static void main(String[] args) {
        Editor editor = new Editor();
        editor.events.subscribe(&quot;open&quot;, new LogOpenListener(&quot;/path/to/log/file.txt&quot;));
        editor.events.subscribe(&quot;save&quot;, new EmailNotificationListener(&quot;admin@example.com&quot;));

        try {
            editor.openFile(&quot;test.txt&quot;);
            editor.saveFile();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Save to log \path\to\log\file.txt: Someone has performed open operation with the following file: test.txt
Email to admin@example.com: Someone has performed save operation with the following file: test.txt</code></pre>
</figure>

::: next
#### Czytaj dalej

[Stan w języku Java []{.fa
.fa-arrow-right}](../../state/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Pamiątka w języku
Java](../../memento/java/example.html){.btn .btn-default rel="prev"}
:::

## **Obserwator** w innych językach

[![Obserwator w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Obserwator w języku C#"){.prog-lang-link}
[![Obserwator w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Obserwator w języku C++"){.prog-lang-link}
[![Obserwator w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Obserwator w języku Go"){.prog-lang-link}
[![Obserwator w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Obserwator w języku PHP"){.prog-lang-link}
[![Obserwator w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Obserwator w języku Python"){.prog-lang-link}
[![Obserwator w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Obserwator w języku Ruby"){.prog-lang-link}
[![Obserwator w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Obserwator w języku Rust"){.prog-lang-link}
[![Obserwator w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Obserwator w języku Swift"){.prog-lang-link}
[![Obserwator w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Obserwator w języku TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::
