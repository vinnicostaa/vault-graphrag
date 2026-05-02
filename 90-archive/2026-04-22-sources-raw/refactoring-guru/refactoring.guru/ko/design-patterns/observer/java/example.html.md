::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[옵서버](../../observer.html){.type} / [자바](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![옵서버](../../../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x"}
:::

# 자바로 작성된 **옵서버** {#자바로-작성된-옵서버 .pattern-example-title-block-title}
::::

:::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**옵서버** 패턴은 일부 객체들이 다른 객체들에 자신의 상태 변경에 대해
알릴 수 있는 행동 디자인 패턴입니다.

옵서버 패턴은 구독자 인터페이스를 구현하는 모든 객체에 대한 이러한
이벤트들을 구독 및 구독 취소하는 방법을 제공합니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 옵서버에 대하여 더 자세히 알아보세요 ](../../observer.html){.btn
.btn-lg .btn-primary}
:::

::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
내비게이션
:::

::: en-intro
 [소개](#)
:::

::: en-intro
 [이벤트 구독](#example-0)
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

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 사례들:** 옵서버 패턴은 자바 코드, 특히 그래픽 사용자 인터페이스
컴포넌트들에서 매우 일반적입니다. 다른 객체들과의 클래스들과 결합하지
않고 해당 객체들에서 발생하는 이벤트들에 반응하는 방법을 제공합니다.

다음은 코어 자바 라이브러리로부터 가져온 패턴의 몇 가지 예시들입니다:

-   [`java.util.Observer`](http://docs.oracle.com/javase/8/docs/api/java/util/Observer.html)/[`java.util.Observable`](http://docs.oracle.com/javase/8/docs/api/java/util/Observable.html)
    (현실 세계에서 거의 사용되지 않습니다)
-   [`java.util.EventListener`](http://docs.oracle.com/javase/8/docs/api/java/util/EventListener.html)의
    모든 구현​(Swing 컴포넌트들에 매우 일반적임)
-   [`javax.servlet.http.HttpSessionBindingListener`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpSessionBindingListener.html)
-   [`javax.servlet.http.HttpSessionAttributeListener`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpSessionAttributeListener.html)
-   [`javax.faces.event.PhaseListener`](http://docs.oracle.com/javaee/7/api/javax/faces/event/PhaseListener.html)

**식별:** 옵서버 패턴은 들어오는 메서드들을 목록에 저장하는 구독
메서드로 초기 식별할 수 있으며, 만약 위 구독 메서드가 목록의 객체들을
순회하고 그들의 \'업데이트\' 메서드를 호출하면 해당 패턴은 옵서버
패턴으로 확정지을 수 있습니다.
::::::::::::::::::::::::

[]{#example-0}

## 이벤트 구독 {#example-0-title}

이 예시에서 옵서버 패턴은 텍스트 편집기의 객체 간의 간접적인 협업을
설정합니다. `Editor`​(편집기) 객체가 변경될 때마다 해당 객체는 그의
구독자들을 알립니다. `Email­Notification­Listener` 및 `Log­Open­Listener`는
그들의 기초 행동들을 실행하여 이러한 알림들에 반응합니다.

구독자 클래스들은 편집기 클래스에 결합하지 않으며 필요한 경우 다른
앱에서 재사용될 수 있습니다. `편집기` 클래스는 오로지 추상 구독자
인터페이스에만 의존합니다. 이렇게 하면 편집기의 코드를 변경하지 않고도
새 구독자 유형들을 추가할 수 있습니다.

### []{#example-0--publisher .anchor} **publisher** {#publisher .example-title}

#### []{#example-0--publisher-EventManager-java .anchor} **publisher/EventManager.java:** 기초 게시자

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

#### []{#example-0--editor-Editor-java .anchor} **editor/Editor.java:** 다른 객체들로 추적된 구상 게시자

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

#### []{#example-0--listeners-EventListener-java .anchor} **listeners/EventListener.java:** 공통 옵서버 인터페이스

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.observer.example.listeners;

import java.io.File;

public interface EventListener {
    void update(String eventType, File file);
}</code></pre>
</figure>

#### []{#example-0--listeners-EmailNotificationListener-java .anchor} **listeners/EmailNotificationListener.java:** 알림 수신 시 이메일 전송

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

#### []{#example-0--listeners-LogOpenListener-java .anchor} **listeners/LogOpenListener.java:** 알림 수신 시 로그에 메시지를 씁니다

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** 초기화 코드

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

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Save to log \path\to\log\file.txt: Someone has performed open operation with the following file: test.txt
Email to admin@example.com: Someone has performed save operation with the following file: test.txt</code></pre>
</figure>

::: next
#### 다음을 보세요

[자바로 작성된 상태 []{.fa
.fa-arrow-right}](../../state/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 자바로 작성된
메멘토](../../memento/java/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **옵서버**

[![C#으로 작성된
옵서버](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 옵서버"){.prog-lang-link}
[![C++로 작성된
옵서버](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 옵서버"){.prog-lang-link}
[![Go로 작성된
옵서버](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 옵서버"){.prog-lang-link}
[![PHP로 작성된
옵서버](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 옵서버"){.prog-lang-link}
[![파이썬으로 작성된
옵서버](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 옵서버"){.prog-lang-link}
[![루비로 작성된
옵서버](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 옵서버"){.prog-lang-link}
[![러스트로 작성된
옵서버](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 옵서버"){.prog-lang-link}
[![스위프트로 작성된
옵서버](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 옵서버"){.prog-lang-link}
[![타입스크립트로 작성된
옵서버](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 옵서버"){.prog-lang-link}
::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ko" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ 예시들을 포함한 저장소](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
전자책 **디자인 패턴에 뛰어들기**를 구매하면 IDE에서 바로 열 수 있는
수십 가지의 상세한 코드 예시들이 포함된 저장소를 사용할 수 있습니다.

[ 더 알아보기...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::
