::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} / [템플릿
메서드](../../template-method.html){.type} /
[자바](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![템플릿
메서드](../../../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x"}
:::

# 자바로 작성된 **템플릿 메서드** {#자바로-작성된-템플릿-메서드 .pattern-example-title-block-title}
::::

:::::::::::::::::: pattern-example-body
::: pattern-example-brief
**템플릿 메서드**는 기초 클래스에서 알고리즘의 골격을 정의할 수 있도록
하는 행동 디자인 패턴입니다. 또 이 패턴은 자식 클래스들이 전체
알고리즘의 구조를 변경하지 않고도 기본 알고리즘의 단계들을 오버라이드할
수 있도록 합니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 템플릿 메서드에 대하여 더 자세히 알아보세요
](../../template-method.html){.btn .btn-lg .btn-primary}
:::

::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
내비게이션
:::

::: en-intro
 [소개](#)
:::

::: en-intro
 [알고리즘의 표준 단계들의 오버라이드](#example-0)
:::

::: en-file
 networks
:::

:::: en-inside
::: en-file
  [Network](#example-0--networks-Network-java)
:::
::::

:::: en-inside
::: en-file
  [Facebook](#example-0--networks-Facebook-java)
:::
::::

:::: en-inside
::: en-file
  [Twitter](#example-0--networks-Twitter-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-txt)
:::
:::::::::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 사례들:** 템플릿 메서드 패턴은 자바 프레임워크들에서 매우
일반적이며 개발자들은 종종 이 패턴을 사용하여 프레임워크 사용자들에게
상속을 사용하여 표준 기능들을 확장하는 간단한 수단을 제공합니다.

다음은 코어 자바 라이브러리로부터 가져온 템플릿 메서드의 몇 가지
예시들입니다:

-   [`java.io.InputStream`](http://docs.oracle.com/javase/8/docs/api/java/io/InputStream.html),
    [`java.io.OutputStream`](http://docs.oracle.com/javase/8/docs/api/java/io/OutputStream.html),
    [`java.io.Reader`](http://docs.oracle.com/javase/8/docs/api/java/io/Reader.html)와
    [`java.io.Writer`](http://docs.oracle.com/javase/8/docs/api/java/io/Writer.html)의
    모든 비 추상적 메서드.

-   [`java.util.AbstractList`](http://docs.oracle.com/javase/8/docs/api/java/util/AbstractList.html),
    [`java.util.AbstractSet`](http://docs.oracle.com/javase/8/docs/api/java/util/AbstractSet.html)와
    [`java.util.AbstractMap`](http://docs.oracle.com/javase/8/docs/api/java/util/AbstractMap.html)의
    모든 비 추상적 메서드.

-   \[`javax.servlet.http.HttpServlet`\](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServlet.html)
    클래스에서 모든 doXXX() 메서드들은 HTTP 405 \'메서드 허용되지 않음\'
    에러를 기본값 응답으로 보냅니다. 그러나 당신은 이러한 모든 메서드들
    오버라이딩하여 다른 에러 메시지를 응답으로 보내게 할 수 있습니다.

**식별:** 템플릿 메서드는 기초 클래스에 추상적이거나 비어 있는 다른 여러
메서드들을 호출하는 메서드가 있습니다.
::::::::::::::::::

[]{#example-0}

## 알고리즘의 표준 단계들의 오버라이드 {#example-0-title}

이 예시에서 템플릿 메서드 패턴은 소셜 네트워크와 작업하는 알고리즘을
정의합니다. 특정 소셜 네트워크에 해당하는 자식 클래스들은 이러한
단계들을 소셜 네트워크에서 제공하는 API에 따라 구현합니다.

### []{#example-0--networks .anchor} **networks** {#networks .example-title}

#### []{#example-0--networks-Network-java .anchor} **networks/Network.java:** 기초 소셜 네트워크 클래스

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.template_method.example.networks;

/**
 * Base class of social network.
 */
public abstract class Network {
    String userName;
    String password;

    Network() {}

    /**
     * Publish the data to whatever network.
     */
    public boolean post(String message) {
        // Authenticate before posting. Every network uses a different
        // authentication method.
        if (logIn(this.userName, this.password)) {
            // Send the post data.
            boolean result =  sendData(message.getBytes());
            logOut();
            return result;
        }
        return false;
    }

    abstract boolean logIn(String userName, String password);
    abstract boolean sendData(byte[] data);
    abstract void logOut();
}</code></pre>
</figure>

#### []{#example-0--networks-Facebook-java .anchor} **networks/Facebook.java:** 구상 소셜 네트워크

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.template_method.example.networks;

/**
 * Class of social network
 */
public class Facebook extends Network {
    public Facebook(String userName, String password) {
        this.userName = userName;
        this.password = password;
    }

    public boolean logIn(String userName, String password) {
        System.out.println(&quot;\nChecking user&#39;s parameters&quot;);
        System.out.println(&quot;Name: &quot; + this.userName);
        System.out.print(&quot;Password: &quot;);
        for (int i = 0; i &lt; this.password.length(); i++) {
            System.out.print(&quot;*&quot;);
        }
        simulateNetworkLatency();
        System.out.println(&quot;\n\nLogIn success on Facebook&quot;);
        return true;
    }

    public boolean sendData(byte[] data) {
        boolean messagePosted = true;
        if (messagePosted) {
            System.out.println(&quot;Message: &#39;&quot; + new String(data) + &quot;&#39; was posted on Facebook&quot;);
            return true;
        } else {
            return false;
        }
    }

    public void logOut() {
        System.out.println(&quot;User: &#39;&quot; + userName + &quot;&#39; was logged out from Facebook&quot;);
    }

    private void simulateNetworkLatency() {
        try {
            int i = 0;
            System.out.println();
            while (i &lt; 10) {
                System.out.print(&quot;.&quot;);
                Thread.sleep(500);
                i++;
            }
        } catch (InterruptedException ex) {
            ex.printStackTrace();
        }
    }
}</code></pre>
</figure>

#### []{#example-0--networks-Twitter-java .anchor} **networks/Twitter.java:** 또 다른 소셜 네트워크

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.template_method.example.networks;

/**
 * Class of social network
 */
public class Twitter extends Network {

    public Twitter(String userName, String password) {
        this.userName = userName;
        this.password = password;
    }

    public boolean logIn(String userName, String password) {
        System.out.println(&quot;\nChecking user&#39;s parameters&quot;);
        System.out.println(&quot;Name: &quot; + this.userName);
        System.out.print(&quot;Password: &quot;);
        for (int i = 0; i &lt; this.password.length(); i++) {
            System.out.print(&quot;*&quot;);
        }
        simulateNetworkLatency();
        System.out.println(&quot;\n\nLogIn success on Twitter&quot;);
        return true;
    }

    public boolean sendData(byte[] data) {
        boolean messagePosted = true;
        if (messagePosted) {
            System.out.println(&quot;Message: &#39;&quot; + new String(data) + &quot;&#39; was posted on Twitter&quot;);
            return true;
        } else {
            return false;
        }
    }

    public void logOut() {
        System.out.println(&quot;User: &#39;&quot; + userName + &quot;&#39; was logged out from Twitter&quot;);
    }

    private void simulateNetworkLatency() {
        try {
            int i = 0;
            System.out.println();
            while (i &lt; 10) {
                System.out.print(&quot;.&quot;);
                Thread.sleep(500);
                i++;
            }
        } catch (InterruptedException ex) {
            ex.printStackTrace();
        }
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** 클라이언트 코드

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.template_method.example;

import refactoring_guru.template_method.example.networks.Facebook;
import refactoring_guru.template_method.example.networks.Network;
import refactoring_guru.template_method.example.networks.Twitter;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

/**
 * Demo class. Everything comes together here.
 */
public class Demo {
    public static void main(String[] args) throws IOException {
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        Network network = null;
        System.out.print(&quot;Input user name: &quot;);
        String userName = reader.readLine();
        System.out.print(&quot;Input password: &quot;);
        String password = reader.readLine();

        // Enter the message.
        System.out.print(&quot;Input message: &quot;);
        String message = reader.readLine();

        System.out.println(&quot;\nChoose social network for posting message.\n&quot; +
                &quot;1 - Facebook\n&quot; +
                &quot;2 - Twitter&quot;);
        int choice = Integer.parseInt(reader.readLine());

        // Create proper network object and send the message.
        if (choice == 1) {
            network = new Facebook(userName, password);
        } else if (choice == 2) {
            network = new Twitter(userName, password);
        }
        network.post(message);
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Input user name: Jhonatan
Input password: qswe
Input message: Hello, World!

Choose social network for posting message.
1 - Facebook
2 - Twitter
2

Checking user&#39;s parameters
Name: Jhonatan
Password: ****
..........

LogIn success on Twitter
Message: &#39;Hello, World!&#39; was posted on Twitter
User: &#39;Jhonatan&#39; was logged out from Twitter</code></pre>
</figure>

::: next
#### 다음을 보세요

[자바로 작성된 비지터 []{.fa
.fa-arrow-right}](../../visitor/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 자바로 작성된
전략](../../strategy/java/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **템플릿 메서드**

[![C#으로 작성된 템플릿
메서드](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 템플릿 메서드"){.prog-lang-link}
[![C++로 작성된 템플릿
메서드](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 템플릿 메서드"){.prog-lang-link}
[![Go로 작성된 템플릿
메서드](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 템플릿 메서드"){.prog-lang-link}
[![PHP로 작성된 템플릿
메서드](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 템플릿 메서드"){.prog-lang-link}
[![파이썬으로 작성된 템플릿
메서드](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 템플릿 메서드"){.prog-lang-link}
[![루비로 작성된 템플릿
메서드](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 템플릿 메서드"){.prog-lang-link}
[![러스트로 작성된 템플릿
메서드](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 템플릿 메서드"){.prog-lang-link}
[![스위프트로 작성된 템플릿
메서드](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 템플릿 메서드"){.prog-lang-link}
[![타입스크립트로 작성된 템플릿
메서드](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 템플릿 메서드"){.prog-lang-link}
::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::
