::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::: {.main-content-container .center-content-container}
::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[カタログ](catalog.html){.type}
:::

# 振る舞いに関するデザインパターン {#振る舞いに関するデザインパターン .title}

振る舞いに関するデザインパターンは[、]{.chpule2}[
]{.chpuri2}アルゴリズムやオブジェクト間の責任の分担に関するものです[。]{.chpule2}[
]{.chpuri2}

::: {.d-flex .flex-wrap .justify-content-between}
[[ [![Chain of
Responsibility](../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x,/images/patterns/cards/chain-of-responsibility-mini-3x.png 3x"}]{.pattern-image}
[Chain of Responsibility]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](chain-of-responsibility.html){.pattern-card-extended
.chain-of-responsibility}

ハンドラーの連鎖に沿ってリクエストを渡すことができます[。]{.chpule2}[
]{.chpuri2}各ハンドラーは[、]{.chpule2}[
]{.chpuri2}リクエストを受け取ると[、]{.chpule2}[
]{.chpuri2}リクエストを処理するか[、]{.chpule2}[
]{.chpuri2}連鎖内の次のハンドラーに渡すかを決めます[。]{.chpule2}[
]{.chpuri2}

[[
[![Command](../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x,/images/patterns/cards/command-mini-3x.png 3x"}]{.pattern-image}
[Command]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](command.html){.pattern-card-extended .command}

リクエストを[、]{.chpule2}[
]{.chpuri2}それに関するすべての情報を含む独立したオブジェクトに転換します[。]{.chpule2}[
]{.chpuri2}この転換により[、]{.chpule2}[
]{.chpuri2}リクエストをメソッドの引数として渡したり[、]{.chpule2}[
]{.chpuri2}リクエストの実行を遅らせたり[、]{.chpule2}[
]{.chpuri2}待ち行列に入れたり[、]{.chpule2}[
]{.chpuri2}取り消し操作を行なうことが可能になります[。]{.chpule2}[
]{.chpuri2}

[[
[![Iterator](../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x,/images/patterns/cards/iterator-mini-3x.png 3x"}]{.pattern-image}
[Iterator]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](iterator.html){.pattern-card-extended
.iterator}

リスト[、]{.chpule2}[ ]{.chpuri2}スタック[、]{.chpule2}[
]{.chpuri2}ツリーなどの実際のデータ表現を表に出さずにコレクションの要素を探索することができます[。]{.chpule2}[
]{.chpuri2}

[[
[![Mediator](../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x,/images/patterns/cards/mediator-mini-3x.png 3x"}]{.pattern-image}
[Mediator]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](mediator.html){.pattern-card-extended
.mediator}

オブジェクト間の混沌とした依存性を削減します[。]{.chpule2}[
]{.chpuri2}パターンは[、]{.chpule2}[
]{.chpuri2}オブジェクト間の直接の通信を制限し[、]{.chpule2}[
]{.chpuri2}メディエーター・オブジェクトを介してのみの共同作業を強制します[。]{.chpule2}[
]{.chpuri2}

[[
[![Memento](../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x,/images/patterns/cards/memento-mini-3x.png 3x"}]{.pattern-image}
[Memento]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](memento.html){.pattern-card-extended .memento}

オブジェクトの以前の状態を保存し復元することを[、]{.chpule2}[
]{.chpuri2}実装の詳細を明かさずに行います[。]{.chpule2}[ ]{.chpuri2}

[[
[![Observer](../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x,/images/patterns/cards/observer-mini-3x.png 3x"}]{.pattern-image}
[Observer]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](observer.html){.pattern-card-extended
.observer}

複数のオブジェクトが観察しているオブジェクトに何かイベントが発生した時にそのイベントについて観察しているオブジェクトへ通知を行うサブスクリプションの仕組みを定義することができます[。]{.chpule2}[
]{.chpuri2}

[[
[![State](../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x,/images/patterns/cards/state-mini-3x.png 3x"}]{.pattern-image}
[State]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](state.html){.pattern-card-extended .state}

オブジェクトの内部状態が変化した時に[、]{.chpule2}[
]{.chpuri2}その挙動を変化させます[。]{.chpule2}[
]{.chpuri2}それは[、]{.chpule2}[
]{.chpuri2}あたかもそのオブジェクトのクラスが変わったかのように見えます[。]{.chpule2}[
]{.chpuri2}

[[
[![Strategy](../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x,/images/patterns/cards/strategy-mini-3x.png 3x"}]{.pattern-image}
[Strategy]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](strategy.html){.pattern-card-extended
.strategy}

アルゴリズムのファミリーを定義し[、]{.chpule2}[
]{.chpuri2}それぞれのアルゴリズムを別個のクラスとし[、]{.chpule2}[
]{.chpuri2}それらのオブジェクトを交換可能にします[。]{.chpule2}[
]{.chpuri2}

[[ [![Template
Method](../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x,/images/patterns/cards/template-method-mini-3x.png 3x"}]{.pattern-image}
[Template Method]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](template-method.html){.pattern-card-extended
.template-method}

スーパークラス内でアルゴリズムの骨格を定義しておき[、]{.chpule2}[
]{.chpuri2}サブクラスは構造を変えることなくアルゴリズムの特定のステップを上書きします[。]{.chpule2}[
]{.chpuri2}

[[
[![Visitor](../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x,/images/patterns/cards/visitor-mini-3x.png 3x"}]{.pattern-image}
[Visitor]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](visitor.html){.pattern-card-extended .visitor}

アルゴリズムをその動作対象となるオブジェクトから切り離します[。]{.chpule2}[
]{.chpuri2}
:::

::: next
#### 次を読む

[Chain of Responsibility []{.fa
.fa-arrow-right}](chain-of-responsibility.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Proxy](proxy.html){.btn .btn-default rel="prev"}
:::
:::::::
::::::::
:::::::::
