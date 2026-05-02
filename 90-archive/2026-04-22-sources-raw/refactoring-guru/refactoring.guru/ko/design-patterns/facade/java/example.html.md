:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[퍼사드](../../facade.html){.type} / [자바](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![퍼사드](../../../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x"}
:::

# 자바로 작성된 **퍼사드** {#자바로-작성된-퍼사드 .pattern-example-title-block-title}
::::

::::::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**퍼사드**는 구조 디자인 패턴이며 간단한 (그러나 제한된 인터페이스를
클래스, 라이브러리 또는 프레임워크로 구성된 복잡한 시스템에 제공합니다.

퍼사드는 앱의 전반적인 복잡성을 줄이는 동시에 원치 않는 의존성들을 한
곳으로 옮기는 것을 돕습니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 퍼사드에 대하여 더 자세히 알아보세요 ](../../facade.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
내비게이션
:::

::: en-intro
 [소개](#)
:::

::: en-intro
 [복잡한 비디오 변환 라이브러리를 위한 간단한 인터페이스](#example-0)
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

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 사례들:** 퍼사드 패턴은 일반적으로 자바로 작성된 앱에서 사용되며
또 복잡한 라이브러리 및 API와 작업할 때 특히 편리합니다.

다음은 코어 자바 라이브러리로부터 가져온 몇 가지 퍼사드 예시들입니다:

-   [`javax.faces.context.FacesContext`](http://docs.oracle.com/javaee/7/api/javax/faces/context/FacesContext.html)는
    내부적으로
    [`Life­Cycle`](http://docs.oracle.com/javaee/7/api/javax/faces/lifecycle/Lifecycle.html),
    [`View­Handler`](http://docs.oracle.com/javaee/7/api/javax/faces/application/ViewHandler.html),
    [`Navigation­Handler`](http://docs.oracle.com/javaee/7/api/javax/faces/application/NavigationHandler.html)
    클래스들을 사용하나, 대부분 클라이언트는 이러한 사실을 모릅니다.

-   [`javax.faces.context.ExternalContext`](http://docs.oracle.com/javaee/7/api/javax/faces/context/ExternalContext.html)는
    [`Servlet­Context`](http://docs.oracle.com/javaee/7/api/javax/servlet/ServletContext.html),
    [`Http­Session`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpSession.html),
    [`Http­Servlet­Request`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServletRequest.html),
    [`Http­Servlet­Response`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServletResponse.html)
    등을 내부에서 사용합니다.

**식별:** 퍼사드는 인터페이스가 간단하나 대부분의 작업을 다른 클래스들에
위임하는 클래스의 존재여부로 식별할 수 있으며 일반적으로 퍼사드들은
사용하는 객체들의 전체 수명 주기를 관리합니다.
:::::::::::::::::::::::::::::

[]{#example-0}

## 복잡한 비디오 변환 라이브러리를 위한 간단한 인터페이스 {#example-0-title}

이 예시에서 퍼사드 패턴은 복잡한 비디오 변환 프레임워크와의 통신을
단순화합니다.

퍼사드는 하나의 메서드를 가진 단일 메서드를 제공하며 이 메서드는
프레임워크의 올바른 클래스들의 설정 그리고 결과를 가져오는 작업에 대한
모든 복잡성을 처리합니다.

### []{#example-0--some_complex_media_library .anchor} **some_complex_media_library:** 복잡한 비디오 변환 라이브러리 {#some_complex_media_library-복잡한-비디오-변환-라이브러리 .example-title}

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

#### []{#example-0--facade-VideoConversionFacade-java .anchor} **facade/VideoConversionFacade.java:** 퍼사드는 비디오 변환의 간단한 인터페이스를 제공합니다

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** 클라이언트 코드

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

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>VideoConversionFacade: conversion started.
CodecFactory: extracting ogg audio...
BitrateReader: reading file...
BitrateReader: writing file...
AudioMixer: fixing audio...
VideoConversionFacade: conversion completed.</code></pre>
</figure>

::: next
#### 다음을 보세요

[자바로 작성된 플라이웨이트 []{.fa
.fa-arrow-right}](../../flyweight/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 자바로 작성된
데코레이터](../../decorator/java/example.html){.btn .btn-default
rel="prev"}
:::

## 다른 언어로 작성된 **퍼사드**

[![C#으로 작성된
퍼사드](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 퍼사드"){.prog-lang-link}
[![C++로 작성된
퍼사드](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 퍼사드"){.prog-lang-link}
[![Go로 작성된
퍼사드](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 퍼사드"){.prog-lang-link}
[![PHP로 작성된
퍼사드](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 퍼사드"){.prog-lang-link}
[![파이썬으로 작성된
퍼사드](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 퍼사드"){.prog-lang-link}
[![루비로 작성된
퍼사드](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 퍼사드"){.prog-lang-link}
[![러스트로 작성된
퍼사드](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 퍼사드"){.prog-lang-link}
[![스위프트로 작성된
퍼사드](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 퍼사드"){.prog-lang-link}
[![타입스크립트로 작성된
퍼사드](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 퍼사드"){.prog-lang-link}
:::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::
