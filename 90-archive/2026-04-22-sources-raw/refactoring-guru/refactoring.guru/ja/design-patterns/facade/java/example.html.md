:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Facade](../../facade.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Facade](../../../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x"}
:::

# **Facade** を Java で {#facade-を-java-で .pattern-example-title-block-title}
::::

::::::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Facade** は[、]{.chpule2}[
]{.chpuri2}構造に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}複雑なクラスのシステム[、]{.chpule2}[
]{.chpuri2}ライブラリー[、]{.chpule2}[
]{.chpuri2}またはフレームワークに対して単純な[
]{.chpule1}[（]{.chpuri1}しかし限定された[）]{.chpule2}[
]{.chpuri2}インターフェースを提供します[。]{.chpule2}[ ]{.chpuri2}

Facade は[、]{.chpule2}[
]{.chpuri2}アプリケーションの全体としての複雑さを軽減しますが[、]{.chpule2}[
]{.chpuri2}それと同時に望ましくない依存性を一箇所に集めるのにも役立ちます[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Facade の詳細 ](../../facade.html){.btn .btn-lg .btn-primary}
:::

:::::::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [複雑なビデオ変換ライブラリー用の単純なインターフェース](#example-0)
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

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}** Facade パターンは[、]{.chpule2}[
]{.chpuri2}Java のアプリでよく見かけます[。]{.chpule2}[
]{.chpuri2}複雑なライブラリーや API を相手にする時[、]{.chpule2}[
]{.chpuri2}特に役に立ちます[。]{.chpule2}[ ]{.chpuri2}

Java のコラ・ライブラリー内での Facade の使用例[：]{.chpule2}[
]{.chpuri2}

-   [`javax.faces.context.FacesContext`](http://docs.oracle.com/javaee/7/api/javax/faces/context/FacesContext.html)
    は[、]{.chpule2}[ ]{.chpuri2}実は内部で
    [`Life­Cycle`](http://docs.oracle.com/javaee/7/api/javax/faces/lifecycle/Lifecycle.html)[、]{.chpule2}[
    ]{.chpuri2}[`View­Handler`](http://docs.oracle.com/javaee/7/api/javax/faces/application/ViewHandler.html)[、]{.chpule2}[
    ]{.chpuri2}[`Navigation­Handler`](http://docs.oracle.com/javaee/7/api/javax/faces/application/NavigationHandler.html)
    クラスを使用していますが[、]{.chpule2}[
    ]{.chpuri2}ほとんどのクライアントはそれに気づいていません[。]{.chpule2}[
    ]{.chpuri2}

-   [`javax.faces.context.ExternalContext`](http://docs.oracle.com/javaee/7/api/javax/faces/context/ExternalContext.html)
    は内部で[、]{.chpule2}[
    ]{.chpuri2}[`Servlet­Context`](http://docs.oracle.com/javaee/7/api/javax/servlet/ServletContext.html)[、]{.chpule2}[
    ]{.chpuri2}[`Http­Session`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpSession.html)[、]{.chpule2}[
    ]{.chpuri2}[`Http­Servlet­Request`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServletRequest.html)[、]{.chpule2}[
    ]{.chpuri2}[`Http­Servlet­Response`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServletResponse.html)
    他を使用します[。]{.chpule2}[ ]{.chpuri2}

**見つけ方[：]{.chpule2}[
]{.chpuri2}**単純なインターフェースのクラスがほとんどの作業を他のクラスに委任していたら[、]{.chpule2}[
]{.chpuri2}Facade パターンの使用が識別できます[。]{.chpule2}[
]{.chpuri2}通常[、]{.chpule2}[ ]{.chpuri2}ファサードは[、]{.chpule2}[
]{.chpuri2}それが使うオブジェクトのライフサイクルを完全に管理します[。]{.chpule2}[
]{.chpuri2}
:::::::::::::::::::::::::::::

[]{#example-0}

## 複雑なビデオ変換ライブラリー用の単純なインターフェース {#example-0-title}

この例では[、]{.chpule2}[ ]{.chpuri2}Facade
パターンを使い[、]{.chpule2}[
]{.chpuri2}複雑なビデオ変換フレームワークとのやりとりを簡素化します[。]{.chpule2}[
]{.chpuri2}

Facade を使用した結果のクラスは[、]{.chpule2}[
]{.chpuri2}フレームワークの適切なクラスを構成し[、]{.chpule2}[
]{.chpuri2}正しい形式で結果を取得するのに必要な複雑な処理すべてを行うメソッド一つからできています[。]{.chpule2}[
]{.chpuri2}

### []{#example-0--some_complex_media_library .anchor} **some_complex_media_library:** 複雑なビデオ変換ライブラリー {#some_complex_media_library-複雑なビデオ変換ライブラリー .example-title}

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

#### []{#example-0--facade-VideoConversionFacade-java .anchor} **facade/VideoConversionFacade.java:** Facade で[、]{.chpule2}[ ]{.chpuri2}ビデオ変換の単純なインターフェースを作成

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** クライアント・コード

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

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>VideoConversionFacade: conversion started.
CodecFactory: extracting ogg audio...
BitrateReader: reading file...
BitrateReader: writing file...
AudioMixer: fixing audio...
VideoConversionFacade: conversion completed.</code></pre>
</figure>

::: next
#### 次を読む

[Flyweight を Java で []{.fa
.fa-arrow-right}](../../flyweight/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Decorator を Java
で](../../decorator/java/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Facade**

[![Facade を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Facade を C# で"){.prog-lang-link}
[![Facade を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Facade を C++ で"){.prog-lang-link}
[![Facade を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Facade を Go で"){.prog-lang-link}
[![Facade を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Facade を PHP で"){.prog-lang-link}
[![Facade を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Facade を Python で"){.prog-lang-link}
[![Facade を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Facade を Ruby で"){.prog-lang-link}
[![Facade を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Facade を Rust で"){.prog-lang-link}
[![Facade を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Facade を Swift で"){.prog-lang-link}
[![Facade を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Facade を TypeScript で"){.prog-lang-link}
:::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::
