:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Facade](../../facade.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Facade](../../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x"}
:::

# **Facade** in Java {#facade-in-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Facade** is a structural design pattern that provides a simplified
(but limited) interface to a complex system of classes, library or
framework.

While Facade decreases the overall complexity of the application, it
also helps to move unwanted dependencies to one place.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Facade ](../../facade.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Simple interface for a complex video conversion library](#example-0)
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

**Complexity:** []{.fa-stars}

**Popularity:** []{.fa-stars}

**Usage examples:** The Facade pattern is commonly used in apps written
in Java. It's especially handy when working with complex libraries and
APIs.

Here are some Facade examples in core Java libs:

-   [`javax.faces.context.FacesContext`](http://docs.oracle.com/javaee/7/api/javax/faces/context/FacesContext.html)
    uses
    [`LifeCycle`](http://docs.oracle.com/javaee/7/api/javax/faces/lifecycle/Lifecycle.html),
    [`ViewHandler`](http://docs.oracle.com/javaee/7/api/javax/faces/application/ViewHandler.html),
    [`NavigationHandler`](http://docs.oracle.com/javaee/7/api/javax/faces/application/NavigationHandler.html)
    classes under the hood, but most clients aren't aware of that.

-   [`javax.faces.context.ExternalContext`](http://docs.oracle.com/javaee/7/api/javax/faces/context/ExternalContext.html)
    uses
    [`ServletContext`](http://docs.oracle.com/javaee/7/api/javax/servlet/ServletContext.html),
    [`HttpSession`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpSession.html),
    [`HttpServletRequest`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServletRequest.html),
    [`HttpServletResponse`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServletResponse.html)
    and others inside.

**Identification:** Facade can be recognized in a class that has a
simple interface, but delegates most of the work to other classes.
Usually, facades manage the full life cycle of objects they use.
:::::::::::::::::::::::::::::

[]{#example-0}

## Simple interface for a complex video conversion library {#example-0-title}

In this example, the Facade simplifies communication with a complex
video conversion framework.

The Facade provides a single class with a single method that handles all
the complexity of configuring the right classes of the framework and
retrieving the result in a correct format.

### []{#example-0--some_complex_media_library .anchor} **some_complex_media_library:** Complex video conversion library {#some_complex_media_library-complex-video-conversion-library .example-title}

#### []{#example-0--some_complex_media_library-VideoFile-java .anchor} **some_complex_media_library/VideoFile.java**

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

#### []{#example-0--some_complex_media_library-Codec-java .anchor} **some_complex_media_library/Codec.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.facade.example.some_complex_media_library;

public interface Codec {
}</code></pre>
</figure>

#### []{#example-0--some_complex_media_library-MPEG4CompressionCodec-java .anchor} **some_complex_media_library/MPEG4CompressionCodec.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.facade.example.some_complex_media_library;

public class MPEG4CompressionCodec implements Codec {
    public String type = &quot;mp4&quot;;

}</code></pre>
</figure>

#### []{#example-0--some_complex_media_library-OggCompressionCodec-java .anchor} **some_complex_media_library/OggCompressionCodec.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.facade.example.some_complex_media_library;

public class OggCompressionCodec implements Codec {
    public String type = &quot;ogg&quot;;
}</code></pre>
</figure>

#### []{#example-0--some_complex_media_library-CodecFactory-java .anchor} **some_complex_media_library/CodecFactory.java**

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

#### []{#example-0--some_complex_media_library-BitrateReader-java .anchor} **some_complex_media_library/BitrateReader.java**

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

#### []{#example-0--some_complex_media_library-AudioMixer-java .anchor} **some_complex_media_library/AudioMixer.java**

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

#### []{#example-0--facade-VideoConversionFacade-java .anchor} **facade/VideoConversionFacade.java:** Facade provides simple interface of video conversion

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** Client code

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

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>VideoConversionFacade: conversion started.
CodecFactory: extracting ogg audio...
BitrateReader: reading file...
BitrateReader: writing file...
AudioMixer: fixing audio...
VideoConversionFacade: conversion completed.</code></pre>
</figure>

::: next
#### Read next

[Flyweight in Java []{.fa
.fa-arrow-right}](../../flyweight/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Decorator in
Java](../../decorator/java/example.html){.btn .btn-default rel="prev"}
:::

## **Facade** in Other Languages

[![Facade in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Facade in C#"){.prog-lang-link}
[![Facade in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Facade in C++"){.prog-lang-link}
[![Facade in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Facade in Go"){.prog-lang-link}
[![Facade in
PHP](../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Facade in PHP"){.prog-lang-link}
[![Facade in
Python](../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Facade in Python"){.prog-lang-link}
[![Facade in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Facade in Ruby"){.prog-lang-link}
[![Facade in
Rust](../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Facade in Rust"){.prog-lang-link}
[![Facade in
Swift](../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Facade in Swift"){.prog-lang-link}
[![Facade in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Facade in TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-en" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive with examples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Buy the eBook **Dive Into Design Patterns** and get the access to
archive with dozens of detailed examples that can be opened right in
your IDE.

[ Learn more...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::
