:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Фасад](../../facade.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Фасад](../../../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x"}
:::

# **Фасад** на Java {#фасад-на-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Фасад** --- це структурний патерн, який надає простий (але урізаний)
інтерфейс до складної системи об'єктів, бібліотеки або фреймворку.

Крім того, що Фасад дозволяє знизити загальну складність програми, він
також допомагає винести код, який залежить від зовнішньої системи, в
одне місце.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Фасад ](../../facade.html){.btn .btn-lg .btn-primary}
:::

:::::::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навігація
:::

::: en-intro
 [Інтро](#)
:::

::: en-intro
 [Простий інтерфейс до складної бібліотеки відеоконвертації](#example-0)
:::

::: en-file
 some_complex_media_library
:::

:::: en-inside
::: en-file
  [Video­File](#example-0--some_complex_media_library-VideoFile-java)
:::
::::

:::: en-inside
::: en-file
  [Codec](#example-0--some_complex_media_library-Codec-java)
:::
::::

:::: en-inside
::: en-file
  [MPEG4Compression­Codec](#example-0--some_complex_media_library-MPEG4CompressionCodec-java)
:::
::::

:::: en-inside
::: en-file
  [Ogg­Compression­Codec](#example-0--some_complex_media_library-OggCompressionCodec-java)
:::
::::

:::: en-inside
::: en-file
  [Codec­Factory](#example-0--some_complex_media_library-CodecFactory-java)
:::
::::

:::: en-inside
::: en-file
  [Bitrate­Reader](#example-0--some_complex_media_library-BitrateReader-java)
:::
::::

:::: en-inside
::: en-file
  [Audio­Mixer](#example-0--some_complex_media_library-AudioMixer-java)
:::
::::

::: en-file
 facade
:::

:::: en-inside
::: en-file
  [Video­Conversion­Facade](#example-0--facade-VideoConversionFacade-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-txt)
:::
::::::::::::::::::::::::::

**Складність:** []{.fa-stars}

**Популярність:** []{.fa-stars}

**Застосування:** Патерн часто зустрічається в клієнтських додатках,
написаних на Java, які використовують складні бібліотеки або API.

Приклади Фасадів в стандартних бібліотеках Java:

-   [`javax.faces.context.FacesContext`](http://docs.oracle.com/javaee/7/api/javax/faces/context/FacesContext.html)
    використовує «під капотом» класи
    [`LifeCycle`](http://docs.oracle.com/javaee/7/api/javax/faces/lifecycle/Lifecycle.html),
    [`ViewHandler`](http://docs.oracle.com/javaee/7/api/javax/faces/application/ViewHandler.html),
    [`NavigationHandler`](http://docs.oracle.com/javaee/7/api/javax/faces/application/NavigationHandler.html)
    і багато інших, але клієнти про це навіть не знають (що не заважає
    замінити ці класи іншими за допомогою ін'єкцій).

-   [`javax.faces.context.ExternalContext`](http://docs.oracle.com/javaee/7/api/javax/faces/context/ExternalContext.html)
    використовує всередині класи
    [`ServletContext`](http://docs.oracle.com/javaee/7/api/javax/servlet/ServletContext.html),
    [`HttpSession`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpSession.html),
    [`HttpServletRequest`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServletRequest.html),
    [`HttpServletResponse`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServletResponse.html)
    і так далі.

**Ознаки застосування патерна:** Фасад впізнається у класі, який має
простий інтерфейс, але делегує основну частину роботи іншим класам.
Найчастіше, фасади самі стежать за життєвим циклом об'єктів складної
системи.
:::::::::::::::::::::::::::::

[]{#example-0}

## Простий інтерфейс до складної бібліотеки відеоконвертації {#example-0-title}

У цьому прикладі Фасад спрощує роботу клієнта зі складною бібліотекою
відеоконвертації.

Фасад надає користувачеві лише один простий метод, приховуючи за собою
цілу систему з відеокодеками, аудіомікшерами та іншими, не менш
складними об'єктами.

### []{#example-0--some_complex_media_library .anchor} **some_complex_media_library:** Складна бібліотека відеоконвертації {#some_complex_media_library-складна-бібліотека-відеоконвертації .example-title}

#### []{#example-0--some_complex_media_library-VideoFile-java .anchor} **some_complex_media_library/VideoFile.java:** Клас відеофайлу

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.facade.example.some_complex_media_library;

public class VideoFile {
    private String name;
    private String codecType;

    public VideoFile(String name) {
        this.name = name;
        this.codecType = name.substring(name.indexOf(&quot;.&quot;) + 1);
    }

    public String getCodecType() {
        return codecType;
    }

    public String getName() {
        return name;
    }
}</code></pre>
</figure>

#### []{#example-0--some_complex_media_library-Codec-java .anchor} **some_complex_media_library/Codec.java:** Інтерфейс кодека

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.facade.example.some_complex_media_library;

public interface Codec {
}</code></pre>
</figure>

#### []{#example-0--some_complex_media_library-MPEG4CompressionCodec-java .anchor} **some_complex_media_library/MPEG4CompressionCodec.java:** Кодек MPEG4

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.facade.example.some_complex_media_library;

public class MPEG4CompressionCodec implements Codec {
    public String type = &quot;mp4&quot;;

}</code></pre>
</figure>

#### []{#example-0--some_complex_media_library-OggCompressionCodec-java .anchor} **some_complex_media_library/OggCompressionCodec.java:** Ogg Кодек

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.facade.example.some_complex_media_library;

public class OggCompressionCodec implements Codec {
    public String type = &quot;ogg&quot;;
}</code></pre>
</figure>

#### []{#example-0--some_complex_media_library-CodecFactory-java .anchor} **some_complex_media_library/CodecFactory.java:** Фабрика відеокодеків

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.facade.example.some_complex_media_library;

public class CodecFactory {
    public static Codec extract(VideoFile file) {
        String type = file.getCodecType();
        if (type.equals(&quot;mp4&quot;)) {
            System.out.println(&quot;CodecFactory: extracting mpeg audio...&quot;);
            return new MPEG4CompressionCodec();
        }
        else {
            System.out.println(&quot;CodecFactory: extracting ogg audio...&quot;);
            return new OggCompressionCodec();
        }
    }
}</code></pre>
</figure>

#### []{#example-0--some_complex_media_library-BitrateReader-java .anchor} **some_complex_media_library/BitrateReader.java:** Bitrate-конвертер

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.facade.example.some_complex_media_library;

public class BitrateReader {
    public static VideoFile read(VideoFile file, Codec codec) {
        System.out.println(&quot;BitrateReader: reading file...&quot;);
        return file;
    }

    public static VideoFile convert(VideoFile buffer, Codec codec) {
        System.out.println(&quot;BitrateReader: writing file...&quot;);
        return buffer;
    }
}</code></pre>
</figure>

#### []{#example-0--some_complex_media_library-AudioMixer-java .anchor} **some_complex_media_library/AudioMixer.java:** Мікшування аудіо

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.facade.example.some_complex_media_library;

import java.io.File;

public class AudioMixer {
    public File fix(VideoFile result){
        System.out.println(&quot;AudioMixer: fixing audio...&quot;);
        return new File(&quot;tmp&quot;);
    }
}</code></pre>
</figure>

### []{#example-0--facade .anchor} **facade** {#facade .example-title}

#### []{#example-0--facade-VideoConversionFacade-java .anchor} **facade/VideoConversionFacade.java:** Фасад бібліотеки роботи з відео

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.facade.example.facade;

import refactoring_guru.facade.example.some_complex_media_library.*;

import java.io.File;

public class VideoConversionFacade {
    public File convertVideo(String fileName, String format) {
        System.out.println(&quot;VideoConversionFacade: conversion started.&quot;);
        VideoFile file = new VideoFile(fileName);
        Codec sourceCodec = CodecFactory.extract(file);
        Codec destinationCodec;
        if (format.equals(&quot;mp4&quot;)) {
            destinationCodec = new MPEG4CompressionCodec();
        } else {
            destinationCodec = new OggCompressionCodec();
        }
        VideoFile buffer = BitrateReader.read(file, sourceCodec);
        VideoFile intermediateResult = BitrateReader.convert(buffer, destinationCodec);
        File result = (new AudioMixer()).fix(intermediateResult);
        System.out.println(&quot;VideoConversionFacade: conversion completed.&quot;);
        return result;
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** Клієнтський код

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.facade.example;

import refactoring_guru.facade.example.facade.VideoConversionFacade;

import java.io.File;

public class Demo {
    public static void main(String[] args) {
        VideoConversionFacade converter = new VideoConversionFacade();
        File mp4Video = converter.convertVideo(&quot;youtubevideo.ogg&quot;, &quot;mp4&quot;);
        // ...
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>VideoConversionFacade: conversion started.
CodecFactory: extracting ogg audio...
BitrateReader: reading file...
BitrateReader: writing file...
AudioMixer: fixing audio...
VideoConversionFacade: conversion completed.</code></pre>
</figure>

::: next
#### Читаємо далі

[Легковаговик на Java []{.fa
.fa-arrow-right}](../../flyweight/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Декоратор на
Java](../../decorator/java/example.html){.btn .btn-default rel="prev"}
:::

## **Фасад** іншими мовами програмування

[![Фасад на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Фасад на C#"){.prog-lang-link}
[![Фасад на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Фасад на C++"){.prog-lang-link}
[![Фасад на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Фасад на Go"){.prog-lang-link}
[![Фасад на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Фасад на PHP"){.prog-lang-link}
[![Фасад на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Фасад на Python"){.prog-lang-link}
[![Фасад на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Фасад на Ruby"){.prog-lang-link}
[![Фасад на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Фасад на Rust"){.prog-lang-link}
[![Фасад на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Фасад на Swift"){.prog-lang-link}
[![Фасад на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Фасад на TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::
