::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} / [Factory
Method](../../factory-method.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Factory
Method](../../../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x"}
:::

# **Factory Method** を Java で {#factory-method-を-java-で .pattern-example-title-block-title}
::::

:::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Factory Method** は[、]{.chpule2}[
]{.chpuri2}生成に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}具象クラスを指定することなく[、]{.chpule2}[
]{.chpuri2}プロダクト[ ]{.chpule1}[（]{.chpuri1}訳注[：]{.chpule2}[
]{.chpuri2}本パターンでは[、]{.chpule2}[
]{.chpuri2}生成されるモノのことを一般にプロダクトと呼びます[）]{.chpule2}[
]{.chpuri2}のオブジェクトを生成することを可能とします[。]{.chpule2}[
]{.chpuri2}

Factory Method では[、]{.chpule2}[
]{.chpuri2}オブジェクトの生成において[、]{.chpule2}[
]{.chpuri2}直接のコンストラクター呼び出し[
]{.chpule1}[（]{.chpuri1}`new` 演算子[）]{.chpule2}[
]{.chpuri2}代わりに使用すべきメソッドを定義します[。]{.chpule2}[
]{.chpuri2}サブクラスにおいてこのメソッドを上書きすることにより[、]{.chpule2}[
]{.chpuri2}生成されるオブジェクトのクラスを変更します[。]{.chpule2}[
]{.chpuri2}

> もし各種ファクトリー系のパターンやコンセプトの違いで迷った場合は[、]{.chpule2}[
> ]{.chpuri2}[ファクトリーの比較](../../factory-comparison.html)
> をご覧ください[。]{.chpule2}[ ]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Factory Method の詳細 ](../../factory-method.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [クロス・プラットフォーム GUI 要素の作成](#example-0)
:::

::: en-file
 buttons
:::

:::: en-inside
::: en-file
  [Button](#example-0--buttons-Button-java)
:::
::::

:::: en-inside
::: en-file
  [Html­Button](#example-0--buttons-HtmlButton-java)
:::
::::

:::: en-inside
::: en-file
  [Windows­Button](#example-0--buttons-WindowsButton-java)
:::
::::

::: en-file
 factory
:::

:::: en-inside
::: en-file
  [Dialog](#example-0--factory-Dialog-java)
:::
::::

:::: en-inside
::: en-file
  [Html­Dialog](#example-0--factory-HtmlDialog-java)
:::
::::

:::: en-inside
::: en-file
  [Windows­Dialog](#example-0--factory-WindowsDialog-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-txt)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-png)
:::
:::::::::::::::::::::::

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}** Factory Method
パターンは[、]{.chpule2}[ ]{.chpuri2}Java
コードでは広く使われます[。]{.chpule2}[
]{.chpuri2}コードに高度の柔軟性を持たせたい時にとても役に立ちます[。]{.chpule2}[
]{.chpuri2}

このパターンは[、]{.chpule2}[ ]{.chpuri2}以下の Java
コア・ライブラリーで使われています[：]{.chpule2}[ ]{.chpuri2}

-   [`java.util.Calendar#getInstance()`](http://docs.oracle.com/javase/8/docs/api/java/util/Calendar.html#getInstance--)
-   [`java.util.ResourceBundle#getBundle()`](http://docs.oracle.com/javase/8/docs/api/java/util/ResourceBundle.html#getBundle-java.lang.String-)
-   [`java.text.NumberFormat#getInstance()`](http://docs.oracle.com/javase/8/docs/api/java/text/NumberFormat.html#getInstance--)
-   [`java.nio.charset.Charset#forName()`](http://docs.oracle.com/javase/8/docs/api/java/nio/charset/Charset.html#forName-java.lang.String-)
-   [`java.net.URLStreamHandlerFactory#createURLStreamHandler(String)`](http://docs.oracle.com/javase/8/docs/api/java/net/URLStreamHandlerFactory.html)[
    ]{.chpule1}[（]{.chpuri1}プロトコルに応じて異なるシングルトン・オブジェクトを返却[）]{.chpule2}[
    ]{.chpuri2}
-   [`java.util.EnumSet#of()`](https://docs.oracle.com/javase/8/docs/api/java/util/EnumSet.html#of(E))
-   [`javax.xml.bind.JAXBContext#createMarshaller()`](https://docs.oracle.com/javase/8/docs/api/javax/xml/bind/JAXBContext.html#createMarshaller--)
    および他の類似メソッド[。]{.chpule2}[ ]{.chpuri2}

**見つけ方[：]{.chpule2}[ ]{.chpuri2}**
具象クラスで具象オブジェクトを作成し[、]{.chpule2}[
]{.chpuri2}それを抽象型またはインターフェースのオブジェクトとして返すような生成メソッドの存在により[、]{.chpule2}[
]{.chpuri2}Factory Method を識別できます[。]{.chpule2}[ ]{.chpuri2}
::::::::::::::::::::::::::

[]{#example-0}

## クロス・プラットフォーム GUI 要素の作成 {#example-0-title}

この例では[、]{.chpule2}[ ]{.chpuri2}Button はプロダクト[、]{.chpule2}[
]{.chpuri2}ダイアログはクリエーターとして機能します[。]{.chpule2}[
]{.chpuri2}

ダイアログは[、]{.chpule2}[ ]{.chpuri2}種類によって[、]{.chpule2}[
]{.chpuri2}それ独自の要素を必要とします[。]{.chpule2}[
]{.chpuri2}ダイアログの種類に応じてサブクラスを作成し[、]{.chpule2}[
]{.chpuri2}ファクトリー・メソッドを上書きするのはこのためです[。]{.chpule2}[
]{.chpuri2}

これで[、]{.chpule2}[
]{.chpuri2}ダイアログの種類に応じて適切なボタンのクラスのインスタンスが生成されます[。]{.chpule2}[
]{.chpuri2}基底ダイアログは[、]{.chpule2}[
]{.chpuri2}共通のインターフェースを使ってプロダクトと連携するため[、]{.chpule2}[
]{.chpuri2}多くの変更にもかかわらずコードは動作し続けます[。]{.chpule2}[
]{.chpuri2}

### []{#example-0--buttons .anchor} **buttons** {#buttons .example-title}

#### []{#example-0--buttons-Button-java .anchor} **buttons/Button.java:** 共通プロダクト・インターフェース

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.factory_method.example.buttons;

/**
 * Common interface for all buttons.
 */
public interface Button {
    void render();
    void onClick();
}</code></pre>
</figure>

#### []{#example-0--buttons-HtmlButton-java .anchor} **buttons/HtmlButton.java:** 具象プロダクト

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.factory_method.example.buttons;

/**
 * HTML button implementation.
 */
public class HtmlButton implements Button {

    public void render() {
        System.out.println(&quot;&lt;button&gt;Test Button&lt;/button&gt;&quot;);
        onClick();
    }

    public void onClick() {
        System.out.println(&quot;Click! Button says - &#39;Hello World!&#39;&quot;);
    }
}</code></pre>
</figure>

#### []{#example-0--buttons-WindowsButton-java .anchor} **buttons/WindowsButton.java:** もう一つの具象プロダクト

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.factory_method.example.buttons;

import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

/**
 * Windows button implementation.
 */
public class WindowsButton implements Button {
    JPanel panel = new JPanel();
    JFrame frame = new JFrame();
    JButton button;

    public void render() {
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        JLabel label = new JLabel(&quot;Hello World!&quot;);
        label.setOpaque(true);
        label.setBackground(new Color(235, 233, 126));
        label.setFont(new Font(&quot;Dialog&quot;, Font.BOLD, 44));
        label.setHorizontalAlignment(SwingConstants.CENTER);
        panel.setLayout(new FlowLayout(FlowLayout.CENTER));
        frame.getContentPane().add(panel);
        panel.add(label);
        onClick();
        panel.add(button);

        frame.setSize(320, 200);
        frame.setVisible(true);
        onClick();
    }

    public void onClick() {
        button = new JButton(&quot;Exit&quot;);
        button.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                frame.setVisible(false);
                System.exit(0);
            }
        });
    }
}</code></pre>
</figure>

### []{#example-0--factory .anchor} **factory** {#factory .example-title}

#### []{#example-0--factory-Dialog-java .anchor} **factory/Dialog.java:** 基底クリエーター

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.factory_method.example.factory;

import refactoring_guru.factory_method.example.buttons.Button;

/**
 * Base factory class. Note that &quot;factory&quot; is merely a role for the class. It
 * should have some core business logic which needs different products to be
 * created.
 */
public abstract class Dialog {

    public void renderWindow() {
        // ... other code ...

        Button okButton = createButton();
        okButton.render();
    }

    /**
     * Subclasses will override this method in order to create specific button
     * objects.
     */
    public abstract Button createButton();
}</code></pre>
</figure>

#### []{#example-0--factory-HtmlDialog-java .anchor} **factory/HtmlDialog.java:** 具象クリエーター

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.factory_method.example.factory;

import refactoring_guru.factory_method.example.buttons.Button;
import refactoring_guru.factory_method.example.buttons.HtmlButton;

/**
 * HTML Dialog will produce HTML buttons.
 */
public class HtmlDialog extends Dialog {

    @Override
    public Button createButton() {
        return new HtmlButton();
    }
}</code></pre>
</figure>

#### []{#example-0--factory-WindowsDialog-java .anchor} **factory/WindowsDialog.java:** もう一つの具象クリエーター

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.factory_method.example.factory;

import refactoring_guru.factory_method.example.buttons.Button;
import refactoring_guru.factory_method.example.buttons.WindowsButton;

/**
 * Windows Dialog will produce Windows buttons.
 */
public class WindowsDialog extends Dialog {

    @Override
    public Button createButton() {
        return new WindowsButton();
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** クライアント・コード

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.factory_method.example;

import refactoring_guru.factory_method.example.factory.Dialog;
import refactoring_guru.factory_method.example.factory.HtmlDialog;
import refactoring_guru.factory_method.example.factory.WindowsDialog;

/**
 * Demo class. Everything comes together here.
 */
public class Demo {
    private static Dialog dialog;

    public static void main(String[] args) {
        configure();
        runBusinessLogic();
    }

    /**
     * The concrete factory is usually chosen depending on configuration or
     * environment options.
     */
    static void configure() {
        if (System.getProperty(&quot;os.name&quot;).equals(&quot;Windows 10&quot;)) {
            dialog = new WindowsDialog();
        } else {
            dialog = new HtmlDialog();
        }
    }

    /**
     * All of the client code should work with factories and products through
     * abstract interfaces. This way it does not care which factory it works
     * with and what kind of product it returns.
     */
    static void runBusinessLogic() {
        dialog.renderWindow();
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** 実行結果[ ]{.chpule1}[（]{.chpuri1}Html­Dialog[）]{.chpule2}[ ]{.chpuri2}

<figure class="code">
<pre class="code" lang="output"><code>&lt;button&gt;Test Button&lt;/button&gt;
Click! Button says - &#39;Hello World!&#39;</code></pre>
</figure>

#### []{#example-0--OutputDemo-png .anchor} **OutputDemo.png:** 実行結果[ ]{.chpule1}[（]{.chpuri1}Windows­Dialog[）]{.chpule2}[ ]{.chpuri2}

![](../../../../images/patterns/examples/java/factory-method/OutputDemof025.png?id=36afce413161f6650321896d3023fb65)

::: next
#### 次を読む

[Prototype を Java で []{.fa
.fa-arrow-right}](../../prototype/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Builder を Java
で](../../builder/java/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Factory Method**

[![Factory Method を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Factory Method を C# で"){.prog-lang-link}
[![Factory Method を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Factory Method を C++ で"){.prog-lang-link}
[![Factory Method を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Factory Method を Go で"){.prog-lang-link}
[![Factory Method を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Factory Method を PHP で"){.prog-lang-link}
[![Factory Method を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Factory Method を Python で"){.prog-lang-link}
[![Factory Method を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Factory Method を Ruby で"){.prog-lang-link}
[![Factory Method を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Factory Method を Rust で"){.prog-lang-link}
[![Factory Method を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Factory Method を Swift で"){.prog-lang-link}
[![Factory Method を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Factory Method を TypeScript で"){.prog-lang-link}
::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
