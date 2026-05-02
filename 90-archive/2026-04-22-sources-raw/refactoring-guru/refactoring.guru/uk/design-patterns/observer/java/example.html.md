::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Спостерігач](../../observer.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Спостерігач](../../../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x"}
:::

# **Спостерігач** на Java {#спостерігач-на-java .pattern-example-title-block-title}
::::

:::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Спостерігач** --- це поведінковий патерн, який дозволяє об'єктам
повідомляти інші об'єкти про зміни свого стану.

При цьому спостерігачі можуть вільно підписуватися і відписуватись від
цих повідомлень.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Спостерігач ](../../observer.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навігація
:::

::: en-intro
 [Інтро](#)
:::

::: en-intro
 [Підписка та повідомлення](#example-0)
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

**Складність:** []{.fa-stars}

**Популярність:** []{.fa-stars}

**Застосування:** Спостерігач часто зустрічається в коді Java, особливо
там, де до відносин між компонентами застосовується модель подій.
Спостерігач дозволяє окремим компонентам реагувати на події, які
відбуваються в інших компонентах.

Приклади Спостерігача в стандартних бібліотеках Java:

-   [`java.util.Observer`](http://docs.oracle.com/javase/8/docs/api/java/util/Observer.html)/[`java.util.Observable`](http://docs.oracle.com/javase/8/docs/api/java/util/Observable.html)
    (рідко використовується у реальному житті)
-   Усі реалізації
    [`java.util.EventListener`](http://docs.oracle.com/javase/8/docs/api/java/util/EventListener.html)
    (практично у всьому Swing-е)
-   [`javax.servlet.http.HttpSessionBindingListener`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpSessionBindingListener.html)
-   [`javax.servlet.http.HttpSessionAttributeListener`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpSessionAttributeListener.html)
-   [`javax.faces.event.PhaseListener`](http://docs.oracle.com/javaee/7/api/javax/faces/event/PhaseListener.html)

**Ознаки застосування патерна:** Спостерігач визначається за механізмом
підписки та методами повідомлення, які викликають компоненти програми.
::::::::::::::::::::::::

[]{#example-0}

## Підписка та повідомлення {#example-0-title}

У цьому прикладі Спостерігач використовується для передачі подій між
об'єктами текстового редактора. Кожного разу, коли об'єкт редактора
змінює свій стан, він повідомляє своїх спостерігачів. Об'єкти
`EmailNotificationListener` та `LogOpenListener` стежать за цими
повідомленнями і виконують корисну роботу у відповідь.

Класи передплатників не пов'язані з класом редактора і, якщо буде
потрібно, можуть повторно використовуватись в інших програмах. Клас
`Editor` залежить тільки від загального інтерфейсу передплатників. Це
дозволяє додавати нові типи передплатників не змінюючи існуючого коду
редактора.

### []{#example-0--publisher .anchor} **publisher** {#publisher .example-title}

#### []{#example-0--publisher-EventManager-java .anchor} **publisher/EventManager.java:** Базовий видавець

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

#### []{#example-0--editor-Editor-java .anchor} **editor/Editor.java:** Конкретний видавець, зміни якого хочуть відстежувати спостерігачі

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

#### []{#example-0--listeners-EventListener-java .anchor} **listeners/EventListener.java:** Інтерфейс передплатників

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.observer.example.listeners;

import java.io.File;

public interface EventListener {
    void update(String eventType, File file);
}</code></pre>
</figure>

#### []{#example-0--listeners-EmailNotificationListener-java .anchor} **listeners/EmailNotificationListener.java:** Слухач, розсилаючий email-повідомлення

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

#### []{#example-0--listeners-LogOpenListener-java .anchor} **listeners/LogOpenListener.java:** Слухач, який записує лог операцій

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** Клієнтський код

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

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Save to log \path\to\log\file.txt: Someone has performed open operation with the following file: test.txt
Email to admin@example.com: Someone has performed save operation with the following file: test.txt</code></pre>
</figure>

::: next
#### Читаємо далі

[Стан на Java []{.fa
.fa-arrow-right}](../../state/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Знімок на
Java](../../memento/java/example.html){.btn .btn-default rel="prev"}
:::

## **Спостерігач** іншими мовами програмування

[![Спостерігач на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Спостерігач на C#"){.prog-lang-link}
[![Спостерігач на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Спостерігач на C++"){.prog-lang-link}
[![Спостерігач на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Спостерігач на Go"){.prog-lang-link}
[![Спостерігач на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Спостерігач на PHP"){.prog-lang-link}
[![Спостерігач на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Спостерігач на Python"){.prog-lang-link}
[![Спостерігач на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Спостерігач на Ruby"){.prog-lang-link}
[![Спостерігач на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Спостерігач на Rust"){.prog-lang-link}
[![Спостерігач на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Спостерігач на Swift"){.prog-lang-link}
[![Спостерігач на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Спостерігач на TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-uk" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архів з прикладами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Придбай книгу **Занурення в Патерни** та отримай архів з десятками
детальних прикладів, які можна відкривати прямо в IDE.

[ Дізнатися більше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::
