:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Façade](../../facade.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Façade](../../../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x"}
:::

# **Façade** en Java {#façade-en-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
La **Façade** est un patron de conception structurel qui fournit une
interface simplifiée (mais limitée) à un système complexe de classes,
bibliothèques ou frameworks.

La façade permet non seulement de diminuer la complexité générale d'une
application, mais elle permet également de rassembler les dépendances
indésirables au même endroit.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Façade ](../../facade.html){.btn .btn-lg
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
 [Interface simple pour une bibliothèque de conversion vidéo
complexe](#example-0)
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

**Complexité :** []{.fa-stars}

**Popularité :** []{.fa-stars}

**Exemples d'utilisation :** La façade est régulièrement utilisée dans
les applications écrites en Java. Elle se révèle très pratique pour
gérer les bibliothèques complexes et les API.

Voici quelques exemples tirés des bibliothèques principales de Java :

-   [`javax.faces.context.FacesContext`](http://docs.oracle.com/javaee/7/api/javax/faces/context/FacesContext.html)
    utilise
    [`LifeCycle`](http://docs.oracle.com/javaee/7/api/javax/faces/lifecycle/Lifecycle.html),
    [`ViewHandler`](http://docs.oracle.com/javaee/7/api/javax/faces/application/ViewHandler.html),
    [`NavigationHandler`](http://docs.oracle.com/javaee/7/api/javax/faces/application/NavigationHandler.html)
    qui sont des classes cachées sous le capot, mais la majorité des
    clients n'en ont pas conscience.

-   [`javax.faces.context.ExternalContext`](http://docs.oracle.com/javaee/7/api/javax/faces/context/ExternalContext.html)
    utilise
    [`ServletContext`](http://docs.oracle.com/javaee/7/api/javax/servlet/ServletContext.html),
    [`HttpSession`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpSession.html),
    [`HttpServletRequest`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServletRequest.html),
    [`HttpServletResponse`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServletResponse.html)
    et d'autres à l'intérieur.

**Identification :** La façade peut être reconnue dans une classe qui a
une interface simple, mais délègue la majorité des tâches à d'autres. En
général, elle gère le cycle de vie complet des objets qu'elle utilise.
:::::::::::::::::::::::::::::

[]{#example-0}

## Interface simple pour une bibliothèque de conversion vidéo complexe {#example-0-title}

Dans cet exemple, la façade simplifie la communication avec un framework
complexe de conversion vidéo.

Elle fournit une seule classe dotée d'une seule méthode qui gère toute
la complexité de la configuration des bonnes classes du framework et de
la récupération du résultat dans le format attendu.

### []{#example-0--some_complex_media_library .anchor} **some_complex_media_library:** Bibliothèque de conversion vidéo complexe {#some_complex_media_library-bibliothèque-de-conversion-vidéo-complexe .example-title}

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

#### []{#example-0--facade-VideoConversionFacade-java .anchor} **facade/VideoConversionFacade.java:** La façade fournit une interface simple de conversion vidéo

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** Code client

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

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>VideoConversionFacade: conversion started.
CodecFactory: extracting ogg audio...
BitrateReader: reading file...
BitrateReader: writing file...
AudioMixer: fixing audio...
VideoConversionFacade: conversion completed.</code></pre>
</figure>

::: next
#### Suivant

[Poids mouche en Java []{.fa
.fa-arrow-right}](../../flyweight/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Décorateur en
Java](../../decorator/java/example.html){.btn .btn-default rel="prev"}
:::

## **Façade** dans les autres langues

[![Façade en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Façade en C#"){.prog-lang-link}
[![Façade en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Façade en C++"){.prog-lang-link}
[![Façade en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Façade en Go"){.prog-lang-link}
[![Façade en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Façade en PHP"){.prog-lang-link}
[![Façade en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Façade en Python"){.prog-lang-link}
[![Façade en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Façade en Ruby"){.prog-lang-link}
[![Façade en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Façade en Rust"){.prog-lang-link}
[![Façade en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Façade en Swift"){.prog-lang-link}
[![Façade en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Façade en TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::
