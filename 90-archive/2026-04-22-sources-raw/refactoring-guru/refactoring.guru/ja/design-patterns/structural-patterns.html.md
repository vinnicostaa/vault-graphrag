::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::: {.main-content-container .center-content-container}
::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[カタログ](catalog.html){.type}
:::

# 構造に関するデザインパターン {#構造に関するデザインパターン .title}

構造に関するデザインパターンは[、]{.chpule2}[
]{.chpuri2}構造の柔軟性と効率を維持しつつ[、]{.chpule2}[
]{.chpuri2}オブジェクトとクラスを大きな構造に束ねる方法を説明します[。]{.chpule2}[
]{.chpuri2}

::: {.d-flex .flex-wrap .justify-content-between}
[[
[![Adapter](../../images/patterns/cards/adapter-minid866.png?id=b2ee4f681fb589be5a0685b94692aebb){srcset="/images/patterns/cards/adapter-mini-2x.png?id=8274d99afbbe9c63bfbfd0d68ceeffc7 2x,/images/patterns/cards/adapter-mini-3x.png 3x"}]{.pattern-image}
[Adapter]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](adapter.html){.pattern-card-extended .adapter}

非互換なインターフェースのオブジェクト同士の協働を可能とします[。]{.chpule2}[
]{.chpuri2}

[[
[![Bridge](../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x,/images/patterns/cards/bridge-mini-3x.png 3x"}]{.pattern-image}
[Bridge]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](bridge.html){.pattern-card-extended .bridge}

巨大なクラスや密接に関連したクラスの集まりを[、]{.chpule2}[
]{.chpuri2}抽象部分と実装部分の二つの階層に分離し[、]{.chpule2}[
]{.chpuri2}それぞれが独立して開発できるようにします[。]{.chpule2}[
]{.chpuri2}

[[
[![Composite](../../images/patterns/cards/composite-minib543.png?id=a369d98d18b417f255d04568fd0131b8){srcset="/images/patterns/cards/composite-mini-2x.png?id=3f7f811fefeb0b64f6774746eb42af09 2x,/images/patterns/cards/composite-mini-3x.png 3x"}]{.pattern-image}
[Composite]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](composite.html){.pattern-card-extended
.composite}

オブジェクトからツリー構造を組み立て[、]{.chpule2}[
]{.chpuri2}その木構造がまるで独立したオブジェクトであるかのように扱えるようにします[。]{.chpule2}[
]{.chpuri2}

[[
[![Decorator](../../images/patterns/cards/decorator-mini6adb.png?id=d30458908e315af195cb183bc52dbef9){srcset="/images/patterns/cards/decorator-mini-2x.png?id=3b58e540d7d28523080cad341ed9b2e9 2x,/images/patterns/cards/decorator-mini-3x.png 3x"}]{.pattern-image}
[Decorator]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](decorator.html){.pattern-card-extended
.decorator}

ある振る舞いを含む特別なラッパー・オブジェクトの中にオブジェクトを配置することで[、]{.chpule2}[
]{.chpuri2}それらのオブジェクトに新しい振る舞いを付け加えます[。]{.chpule2}[
]{.chpuri2}

[[
[![Facade](../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x,/images/patterns/cards/facade-mini-3x.png 3x"}]{.pattern-image}
[Facade]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](facade.html){.pattern-card-extended .facade}

ライブラリー[、]{.chpule2}[ ]{.chpuri2}フレームワーク[、]{.chpule2}[
]{.chpuri2}その他のクラスの複雑な組み合わせに対し[、]{.chpule2}[
]{.chpuri2}簡素化されたインターフェースを提供します[。]{.chpule2}[
]{.chpuri2}

[[
[![Flyweight](../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x,/images/patterns/cards/flyweight-mini-3x.png 3x"}]{.pattern-image}
[Flyweight]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](flyweight.html){.pattern-card-extended
.flyweight}

複数のオブジェクト間で共通する部分を各自で持つ代わりに共有することによって[、]{.chpule2}[
]{.chpuri2}利用可能な RAM
により多くのオブジェクトを収められるようにします[。]{.chpule2}[
]{.chpuri2}

[[
[![Proxy](../../images/patterns/cards/proxy-minid8d7.png?id=25890b11e7dc5af29625ccd0678b63a8){srcset="/images/patterns/cards/proxy-mini-2x.png?id=8638fac9dc08c992852492f9cb29d9c6 2x,/images/patterns/cards/proxy-mini-3x.png 3x"}]{.pattern-image}
[Proxy]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](proxy.html){.pattern-card-extended .proxy}

他のオブジェクトの代理[、]{.chpule2}[
]{.chpuri2}代用を提供します[。]{.chpule2}[
]{.chpuri2}プロキシーは[、]{.chpule2}[
]{.chpuri2}元のオブジェクトへのアクセスを制御し[、]{.chpule2}[
]{.chpuri2}元のオブジェクトへリクエストが行く前か後に別の何かを行うようにすることができます[。]{.chpule2}[
]{.chpuri2}
:::

::: next
#### 次を読む

[Adapter []{.fa .fa-arrow-right}](adapter.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Singleton](singleton.html){.btn .btn-default
rel="prev"}
:::
:::::::
::::::::
:::::::::
