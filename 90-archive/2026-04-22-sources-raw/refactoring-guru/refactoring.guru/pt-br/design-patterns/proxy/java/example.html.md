::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Proxy](../../proxy.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Proxy](../../../../images/patterns/cards/proxy-minid8d7.png?id=25890b11e7dc5af29625ccd0678b63a8){srcset="/images/patterns/cards/proxy-mini-2x.png?id=8638fac9dc08c992852492f9cb29d9c6 2x"}
:::

# **Proxy** em Java {#proxy-em-java .pattern-example-title-block-title}
::::

:::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
O **Proxy** é um padrão de projeto estrutural que fornece um objeto que
atua como um substituto para um objeto de serviço real usado por um
cliente. Um proxy recebe solicitações do cliente, realiza alguma tarefa
(controle de acesso, armazenamento em cache etc.) e passa a solicitação
para um objeto de serviço.

O objeto proxy tem a mesma interface que um serviço, o que o torna
intercambiável com um objeto real quando passado para um cliente.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Proxy ](../../proxy.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegação
:::

::: en-intro
 [Introdução](#)
:::

::: en-intro
 [Proxy de cache](#example-0)
:::

::: en-file
 some_cool_media_library
:::

:::: en-inside
::: en-file
  [Third­Party­You­Tube­Lib](#example-0--some_cool_media_library-ThirdPartyYouTubeLib-java)
:::
::::

:::: en-inside
::: en-file
  [Third­Party­You­Tube­Class](#example-0--some_cool_media_library-ThirdPartyYouTubeClass-java)
:::
::::

:::: en-inside
::: en-file
  [Video](#example-0--some_cool_media_library-Video-java)
:::
::::

::: en-file
 proxy
:::

:::: en-inside
::: en-file
  [You­Tube­Cache­Proxy](#example-0--proxy-YouTubeCacheProxy-java)
:::
::::

::: en-file
 downloader
:::

:::: en-inside
::: en-file
  [You­Tube­Downloader](#example-0--downloader-YouTubeDownloader-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-txt)
:::
:::::::::::::::::::::

**Complexidade:** []{.fa-stars}

**Popularidade:** []{.fa-stars}

**Exemplos de uso:** Embora o padrão Proxy não seja um convidado
frequente na maioria das aplicações Java, ele ainda é muito útil em
alguns casos especiais. É insubstituível quando você deseja adicionar
alguns comportamentos adicionais a um objeto de alguma classe existente
sem alterar o código cliente.

Alguns exemplos de proxies em bibliotecas Java padrão:

-   [`java.lang.reflect.Proxy`](http://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Proxy.html)
-   [`java.rmi.*`](http://docs.oracle.com/javase/8/docs/api/java/rmi/package-summary.html)
-   [`javax.ejb.EJB`](http://docs.oracle.com/javaee/7/api/javax/ejb/EJB.html)
    (\[veja comentários\]
    (http://stackoverflow.com/questions/25514361/when-using-ejb-does-each-managed-bean-get-its-own-ejb-instance))
-   [`javax.inject.Inject`](http://docs.oracle.com/javaee/7/api/javax/inject/Inject.html)
    (\[veja comentários\]
    (http://stackoverflow.com/questions/29651008/field-getobj-returns-all-nulls-on-injected-cdi-managed-beans-while-manually-i/29672591#29672591))
-   [`javax.persistence.PersistenceContext`](http://docs.oracle.com/javaee/7/api/javax/persistence/PersistenceContext.html)

**Identificação:** Proxies delegam todo o trabalho real para algum outro
objeto. Cada método de proxy deve, no final, se referir a um objeto de
serviço, a menos que o proxy seja uma subclasse de um serviço.
::::::::::::::::::::::::

[]{#example-0}

## Proxy de cache {#example-0-title}

Neste exemplo, o padrão Proxy ajuda a implementar a inicialização
preguiçosa e o cache em uma biblioteca de terceiros de integração
ineficiente do YouTube.

O proxy é inestimável quando você precisa adicionar alguns
comportamentos adicionais a uma classe cujo código não pode ser
alterado.

### []{#example-0--some_cool_media_library .anchor} **some_cool_media_library** {#some_cool_media_library .example-title}

#### []{#example-0--some_cool_media_library-ThirdPartyYouTubeLib-java .anchor} **some_cool_media_library/ThirdPartyYouTubeLib.java:** Interface do serviço remoto

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.proxy.example.some_cool_media_library;

import java.util.HashMap;

public interface ThirdPartyYouTubeLib {
    HashMap&lt;String, Video&gt; popularVideos();

    Video getVideo(String videoId);
}</code></pre>
</figure>

#### []{#example-0--some_cool_media_library-ThirdPartyYouTubeClass-java .anchor} **some_cool_media_library/ThirdPartyYouTubeClass.java:** Implementação do serviço remoto

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.proxy.example.some_cool_media_library;

import java.util.HashMap;

public class ThirdPartyYouTubeClass implements ThirdPartyYouTubeLib {

    @Override
    public HashMap&lt;String, Video&gt; popularVideos() {
        connectToServer(&quot;http://www.youtube.com&quot;);
        return getRandomVideos();
    }

    @Override
    public Video getVideo(String videoId) {
        connectToServer(&quot;http://www.youtube.com/&quot; + videoId);
        return getSomeVideo(videoId);
    }

    // -----------------------------------------------------------------------
    // Fake methods to simulate network activity. They as slow as a real life.

    private int random(int min, int max) {
        return min + (int) (Math.random() * ((max - min) + 1));
    }

    private void experienceNetworkLatency() {
        int randomLatency = random(5, 10);
        for (int i = 0; i &lt; randomLatency; i++) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException ex) {
                ex.printStackTrace();
            }
        }
    }

    private void connectToServer(String server) {
        System.out.print(&quot;Connecting to &quot; + server + &quot;... &quot;);
        experienceNetworkLatency();
        System.out.print(&quot;Connected!&quot; + &quot;\n&quot;);
    }

    private HashMap&lt;String, Video&gt; getRandomVideos() {
        System.out.print(&quot;Downloading populars... &quot;);

        experienceNetworkLatency();
        HashMap&lt;String, Video&gt; hmap = new HashMap&lt;String, Video&gt;();
        hmap.put(&quot;catzzzzzzzzz&quot;, new Video(&quot;sadgahasgdas&quot;, &quot;Catzzzz.avi&quot;));
        hmap.put(&quot;mkafksangasj&quot;, new Video(&quot;mkafksangasj&quot;, &quot;Dog play with ball.mp4&quot;));
        hmap.put(&quot;dancesvideoo&quot;, new Video(&quot;asdfas3ffasd&quot;, &quot;Dancing video.mpq&quot;));
        hmap.put(&quot;dlsdk5jfslaf&quot;, new Video(&quot;dlsdk5jfslaf&quot;, &quot;Barcelona vs RealM.mov&quot;));
        hmap.put(&quot;3sdfgsd1j333&quot;, new Video(&quot;3sdfgsd1j333&quot;, &quot;Programing lesson#1.avi&quot;));

        System.out.print(&quot;Done!&quot; + &quot;\n&quot;);
        return hmap;
    }

    private Video getSomeVideo(String videoId) {
        System.out.print(&quot;Downloading video... &quot;);

        experienceNetworkLatency();
        Video video = new Video(videoId, &quot;Some video title&quot;);

        System.out.print(&quot;Done!&quot; + &quot;\n&quot;);
        return video;
    }

}</code></pre>
</figure>

#### []{#example-0--some_cool_media_library-Video-java .anchor} **some_cool_media_library/Video.java:** Arquivo de vídeo

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.proxy.example.some_cool_media_library;

public class Video {
    public String id;
    public String title;
    public String data;

    Video(String id, String title) {
        this.id = id;
        this.title = title;
        this.data = &quot;Random video.&quot;;
    }
}</code></pre>
</figure>

### []{#example-0--proxy .anchor} **proxy** {#proxy .example-title}

#### []{#example-0--proxy-YouTubeCacheProxy-java .anchor} **proxy/YouTubeCacheProxy.java:** Proxy de cache

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.proxy.example.proxy;

import refactoring_guru.proxy.example.some_cool_media_library.ThirdPartyYouTubeClass;
import refactoring_guru.proxy.example.some_cool_media_library.ThirdPartyYouTubeLib;
import refactoring_guru.proxy.example.some_cool_media_library.Video;

import java.util.HashMap;

public class YouTubeCacheProxy implements ThirdPartyYouTubeLib {
    private ThirdPartyYouTubeLib youtubeService;
    private HashMap&lt;String, Video&gt; cachePopular = new HashMap&lt;String, Video&gt;();
    private HashMap&lt;String, Video&gt; cacheAll = new HashMap&lt;String, Video&gt;();

    public YouTubeCacheProxy() {
        this.youtubeService = new ThirdPartyYouTubeClass();
    }

    @Override
    public HashMap&lt;String, Video&gt; popularVideos() {
        if (cachePopular.isEmpty()) {
            cachePopular = youtubeService.popularVideos();
        } else {
            System.out.println(&quot;Retrieved list from cache.&quot;);
        }
        return cachePopular;
    }

    @Override
    public Video getVideo(String videoId) {
        Video video = cacheAll.get(videoId);
        if (video == null) {
            video = youtubeService.getVideo(videoId);
            cacheAll.put(videoId, video);
        } else {
            System.out.println(&quot;Retrieved video &#39;&quot; + videoId + &quot;&#39; from cache.&quot;);
        }
        return video;
    }

    public void reset() {
        cachePopular.clear();
        cacheAll.clear();
    }
}</code></pre>
</figure>

### []{#example-0--downloader .anchor} **downloader** {#downloader .example-title}

#### []{#example-0--downloader-YouTubeDownloader-java .anchor} **downloader/YouTubeDownloader.java:** App baixadora de mídia

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.proxy.example.downloader;

import refactoring_guru.proxy.example.some_cool_media_library.ThirdPartyYouTubeLib;
import refactoring_guru.proxy.example.some_cool_media_library.Video;

import java.util.HashMap;

public class YouTubeDownloader {
    private ThirdPartyYouTubeLib api;

    public YouTubeDownloader(ThirdPartyYouTubeLib api) {
        this.api = api;
    }

    public void renderVideoPage(String videoId) {
        Video video = api.getVideo(videoId);
        System.out.println(&quot;\n-------------------------------&quot;);
        System.out.println(&quot;Video page (imagine fancy HTML)&quot;);
        System.out.println(&quot;ID: &quot; + video.id);
        System.out.println(&quot;Title: &quot; + video.title);
        System.out.println(&quot;Video: &quot; + video.data);
        System.out.println(&quot;-------------------------------\n&quot;);
    }

    public void renderPopularVideos() {
        HashMap&lt;String, Video&gt; list = api.popularVideos();
        System.out.println(&quot;\n-------------------------------&quot;);
        System.out.println(&quot;Most popular videos on YouTube (imagine fancy HTML)&quot;);
        for (Video video : list.values()) {
            System.out.println(&quot;ID: &quot; + video.id + &quot; / Title: &quot; + video.title);
        }
        System.out.println(&quot;-------------------------------\n&quot;);
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** Código de inicialização

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.proxy.example;

import refactoring_guru.proxy.example.downloader.YouTubeDownloader;
import refactoring_guru.proxy.example.proxy.YouTubeCacheProxy;
import refactoring_guru.proxy.example.some_cool_media_library.ThirdPartyYouTubeClass;

public class Demo {

    public static void main(String[] args) {
        YouTubeDownloader naiveDownloader = new YouTubeDownloader(new ThirdPartyYouTubeClass());
        YouTubeDownloader smartDownloader = new YouTubeDownloader(new YouTubeCacheProxy());

        long naive = test(naiveDownloader);
        long smart = test(smartDownloader);
        System.out.print(&quot;Time saved by caching proxy: &quot; + (naive - smart) + &quot;ms&quot;);

    }

    private static long test(YouTubeDownloader downloader) {
        long startTime = System.currentTimeMillis();

        // User behavior in our app:
        downloader.renderPopularVideos();
        downloader.renderVideoPage(&quot;catzzzzzzzzz&quot;);
        downloader.renderPopularVideos();
        downloader.renderVideoPage(&quot;dancesvideoo&quot;);
        // Users might visit the same page quite often.
        downloader.renderVideoPage(&quot;catzzzzzzzzz&quot;);
        downloader.renderVideoPage(&quot;someothervid&quot;);

        long estimatedTime = System.currentTimeMillis() - startTime;
        System.out.print(&quot;Time elapsed: &quot; + estimatedTime + &quot;ms\n&quot;);
        return estimatedTime;
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Connecting to http://www.youtube.com... Connected!
Downloading populars... Done!

-------------------------------
Most popular videos on YouTube (imagine fancy HTML)
ID: sadgahasgdas / Title: Catzzzz.avi
ID: asdfas3ffasd / Title: Dancing video.mpq
ID: 3sdfgsd1j333 / Title: Programing lesson#1.avi
ID: mkafksangasj / Title: Dog play with ball.mp4
ID: dlsdk5jfslaf / Title: Barcelona vs RealM.mov
-------------------------------

Connecting to http://www.youtube.com/catzzzzzzzzz... Connected!
Downloading video... Done!

-------------------------------
Video page (imagine fancy HTML)
ID: catzzzzzzzzz
Title: Some video title
Video: Random video.
-------------------------------

Connecting to http://www.youtube.com... Connected!
Downloading populars... Done!

-------------------------------
Most popular videos on YouTube (imagine fancy HTML)
ID: sadgahasgdas / Title: Catzzzz.avi
ID: asdfas3ffasd / Title: Dancing video.mpq
ID: 3sdfgsd1j333 / Title: Programing lesson#1.avi
ID: mkafksangasj / Title: Dog play with ball.mp4
ID: dlsdk5jfslaf / Title: Barcelona vs RealM.mov
-------------------------------

Connecting to http://www.youtube.com/dancesvideoo... Connected!
Downloading video... Done!

-------------------------------
Video page (imagine fancy HTML)
ID: dancesvideoo
Title: Some video title
Video: Random video.
-------------------------------

Connecting to http://www.youtube.com/catzzzzzzzzz... Connected!
Downloading video... Done!

-------------------------------
Video page (imagine fancy HTML)
ID: catzzzzzzzzz
Title: Some video title
Video: Random video.
-------------------------------

Connecting to http://www.youtube.com/someothervid... Connected!
Downloading video... Done!

-------------------------------
Video page (imagine fancy HTML)
ID: someothervid
Title: Some video title
Video: Random video.
-------------------------------

Time elapsed: 9354ms
Connecting to http://www.youtube.com... Connected!
Downloading populars... Done!

-------------------------------
Most popular videos on YouTube (imagine fancy HTML)
ID: sadgahasgdas / Title: Catzzzz.avi
ID: asdfas3ffasd / Title: Dancing video.mpq
ID: 3sdfgsd1j333 / Title: Programing lesson#1.avi
ID: mkafksangasj / Title: Dog play with ball.mp4
ID: dlsdk5jfslaf / Title: Barcelona vs RealM.mov
-------------------------------

Connecting to http://www.youtube.com/catzzzzzzzzz... Connected!
Downloading video... Done!

-------------------------------
Video page (imagine fancy HTML)
ID: catzzzzzzzzz
Title: Some video title
Video: Random video.
-------------------------------

Retrieved list from cache.

-------------------------------
Most popular videos on YouTube (imagine fancy HTML)
ID: sadgahasgdas / Title: Catzzzz.avi
ID: asdfas3ffasd / Title: Dancing video.mpq
ID: 3sdfgsd1j333 / Title: Programing lesson#1.avi
ID: mkafksangasj / Title: Dog play with ball.mp4
ID: dlsdk5jfslaf / Title: Barcelona vs RealM.mov
-------------------------------

Connecting to http://www.youtube.com/dancesvideoo... Connected!
Downloading video... Done!

-------------------------------
Video page (imagine fancy HTML)
ID: dancesvideoo
Title: Some video title
Video: Random video.
-------------------------------

Retrieved video &#39;catzzzzzzzzz&#39; from cache.

-------------------------------
Video page (imagine fancy HTML)
ID: catzzzzzzzzz
Title: Some video title
Video: Random video.
-------------------------------

Connecting to http://www.youtube.com/someothervid... Connected!
Downloading video... Done!

-------------------------------
Video page (imagine fancy HTML)
ID: someothervid
Title: Some video title
Video: Random video.
-------------------------------

Time elapsed: 5875ms
Time saved by caching proxy: 3479ms</code></pre>
</figure>

::: next
#### Leia a seguir

[Chain of Responsibility em Java []{.fa
.fa-arrow-right}](../../chain-of-responsibility/java/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Flyweight em
Java](../../flyweight/java/example.html){.btn .btn-default rel="prev"}
:::

## **Proxy** em outras linguagens

[![Proxy em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Proxy em C#"){.prog-lang-link}
[![Proxy em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Proxy em C++"){.prog-lang-link}
[![Proxy em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Proxy em Go"){.prog-lang-link}
[![Proxy em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Proxy em PHP"){.prog-lang-link}
[![Proxy em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Proxy em Python"){.prog-lang-link}
[![Proxy em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Proxy em Ruby"){.prog-lang-link}
[![Proxy em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Proxy em Rust"){.prog-lang-link}
[![Proxy em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Proxy em Swift"){.prog-lang-link}
[![Proxy em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Proxy em TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pt-br" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Arquivo com exemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compre o eBook **Mergulho nos Padrões de Projeto** e tenha acesso ao
arquivo com dúzias de exemplos detalhados que podem ser abertos
diretamente em seu IDE.

[ Saiba mais...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::
