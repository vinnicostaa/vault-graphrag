::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Decorator](../../decorator.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Decorator](../../../../images/patterns/cards/decorator-mini6adb.png?id=d30458908e315af195cb183bc52dbef9){srcset="/images/patterns/cards/decorator-mini-2x.png?id=3b58e540d7d28523080cad341ed9b2e9 2x"}
:::

# **Decorator** を Java で {#decorator-を-java-で .pattern-example-title-block-title}
::::

:::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Decorator** は[、]{.chpule2}[
]{.chpuri2}構造に関するパターンのひとつで[、]{.chpule2}[
]{.chpuri2}オブジェクトを*[デ]{.cjk}[コ]{.cjk}[レ]{.cjk}[ー]{.cjk}[タ]{.cjk}[ー]{.cjk}*と呼ばれる特別なラッパー・オブジェクト内に配置することにより[、]{.chpule2}[
]{.chpuri2}新しい振る舞いを動的に追加できます[。]{.chpule2}[ ]{.chpuri2}

対象のオブジェクトとデコレーターは同じインターフェースに従うため[、]{.chpule2}[
]{.chpuri2}デコレーターを使うと[、]{.chpule2}[
]{.chpuri2}オブジェクトを何重にも包み込むことができます[。]{.chpule2}[
]{.chpuri2}その結果として生成されるオブジェクトは[、]{.chpule2}[
]{.chpuri2}全部のラッパーの振る舞いを集積した振る舞いをします[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Decorator の詳細 ](../../decorator.html){.btn .btn-lg .btn-primary}
:::

::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [暗号化デコレーターと圧縮デコレーター](#example-0)
:::

::: en-file
 decorators
:::

:::: en-inside
::: en-file
  [Data­Source](#example-0--decorators-DataSource-java)
:::
::::

:::: en-inside
::: en-file
  [File­Data­Source](#example-0--decorators-FileDataSource-java)
:::
::::

:::: en-inside
::: en-file
  [Data­Source­Decorator](#example-0--decorators-DataSourceDecorator-java)
:::
::::

:::: en-inside
::: en-file
  [Encryption­Decorator](#example-0--decorators-EncryptionDecorator-java)
:::
::::

:::: en-inside
::: en-file
  [Compression­Decorator](#example-0--decorators-CompressionDecorator-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-txt)
:::
:::::::::::::::::::

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}** Decorator は[、]{.chpule2}[
]{.chpuri2}Java コードではかなり標準的で[、]{.chpule2}[
]{.chpuri2}特にストリームに関連するコードではよく使われます[。]{.chpule2}[
]{.chpuri2}

Java のコア・ライブラリーでの Decorator の使用例です[：]{.chpule2}[
]{.chpuri2}

-   [`java.io.InputStream`](http://docs.oracle.com/javase/8/docs/api/java/io/InputStream.html)[、]{.chpule2}[
    ]{.chpuri2}[`Output­Stream`](http://docs.oracle.com/javase/8/docs/api/java/io/OutputStream.html)[、]{.chpule2}[
    ]{.chpuri2}[`Reader`](http://docs.oracle.com/javase/8/docs/api/java/io/Reader.html)[、]{.chpule2}[
    ]{.chpuri2}[`Writer`](http://docs.oracle.com/javase/8/docs/api/java/io/Writer.html)
    には[、]{.chpule2}[
    ]{.chpuri2}自身と同じ型のオブジェクトを取るコンストラクターがあります[。]{.chpule2}[
    ]{.chpuri2}

-   [`java.util.Collections`](http://docs.oracle.com/javase/8/docs/api/java/util/Collections.html)[、]{.chpule2}[
    ]{.chpuri2}[`checked­XXX()`](http://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#checkedCollection-java.util.Collection-java.lang.Class-)
    メソッド[ ]{.chpule1}[（]{.chpuri1}以下同[）]{.ls-05}[、]{.chpule2}[
    ]{.chpuri2}[`synchronized­XXX()`](http://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#synchronizedCollection-java.util.Collection-)[、]{.chpule2}[
    ]{.chpuri2}[`unmodifiable­XXX()`](http://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#unmodifiableCollection-java.util.Collection-)[。]{.chpule2}[
    ]{.chpuri2}

-   [`javax.servlet.http.HttpServletRequestWrapper`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServletRequestWrapper.html)
    と
    [`Http­Servlet­Response­Wrapper`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServletResponseWrapper.html)

**見つけ方[：]{.chpule2}[ ]{.chpuri2}** Decorator は[、]{.chpule2}[
]{.chpuri2}そのクラスと同じクラスまたはインターフェースのオブジェクトをパラメーターとして取る生成メソッドかコンストラクターの存在により識別できます[。]{.chpule2}[
]{.chpuri2}
::::::::::::::::::::::

[]{#example-0}

## 暗号化デコレーターと圧縮デコレーター {#example-0-title}

この例では[、]{.chpule2}[
]{.chpuri2}コードを変更せずにオブジェクトの動作を変える方法を紹介します[。]{.chpule2}[
]{.chpuri2}

最初は[、]{.chpule2}[
]{.chpuri2}ビジネス・ロジックのクラスは平文ひらぶんのテキストのデータの読み書きしかできませんでした[。]{.chpule2}[
]{.chpuri2} 次に[、]{.chpule2}[
]{.chpuri2}ラップされたオブジェクトが標準操作を実行した後に新しい動作を追加するため[、]{.chpule2}[
]{.chpuri2}小さなラッパー・クラスをいくつか作成しました[。]{.chpule2}[
]{.chpuri2}

最初のラッパーはデータの暗号化と復号化を行い[、]{.chpule2}[
]{.chpuri2}二つ目のラッパーはデータの圧縮と解凍を行います[。]{.chpule2}[
]{.chpuri2}

デコレーターを別のデコレーターで包み込み[、]{.chpule2}[
]{.chpuri2}組み合わせることさえできます[。]{.chpule2}[ ]{.chpuri2}

### []{#example-0--decorators .anchor} **decorators** {#decorators .example-title}

#### []{#example-0--decorators-DataSource-java .anchor} **decorators/DataSource.java:** 読み書きの操作を定義する共通データ・インターフェース

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.decorator.example.decorators;

public interface DataSource {
    void writeData(String data);

    String readData();
}</code></pre>
</figure>

#### []{#example-0--decorators-FileDataSource-java .anchor} **decorators/FileDataSource.java:** 単純なデータの読み取り・書き取りクラス

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.decorator.example.decorators;

import java.io.*;

public class FileDataSource implements DataSource {
    private String name;

    public FileDataSource(String name) {
        this.name = name;
    }

    @Override
    public void writeData(String data) {
        File file = new File(name);
        try (OutputStream fos = new FileOutputStream(file)) {
            fos.write(data.getBytes(), 0, data.length());
        } catch (IOException ex) {
            System.out.println(ex.getMessage());
        }
    }

    @Override
    public String readData() {
        char[] buffer = null;
        File file = new File(name);
        try (FileReader reader = new FileReader(file)) {
            buffer = new char[(int) file.length()];
            reader.read(buffer);
        } catch (IOException ex) {
            System.out.println(ex.getMessage());
        }
        return new String(buffer);
    }
}</code></pre>
</figure>

#### []{#example-0--decorators-DataSourceDecorator-java .anchor} **decorators/DataSourceDecorator.java:** 抽象基底デコレーター

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.decorator.example.decorators;

public abstract class DataSourceDecorator implements DataSource {
    private DataSource wrappee;

    DataSourceDecorator(DataSource source) {
        this.wrappee = source;
    }

    @Override
    public void writeData(String data) {
        wrappee.writeData(data);
    }

    @Override
    public String readData() {
        return wrappee.readData();
    }
}</code></pre>
</figure>

#### []{#example-0--decorators-EncryptionDecorator-java .anchor} **decorators/EncryptionDecorator.java:** 暗号化デコレーター

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.decorator.example.decorators;

import java.util.Base64;

public class EncryptionDecorator extends DataSourceDecorator {

    public EncryptionDecorator(DataSource source) {
        super(source);
    }

    @Override
    public void writeData(String data) {
        super.writeData(encode(data));
    }

    @Override
    public String readData() {
        return decode(super.readData());
    }

    private String encode(String data) {
        byte[] result = data.getBytes();
        for (int i = 0; i &lt; result.length; i++) {
            result[i] += (byte) 1;
        }
        return Base64.getEncoder().encodeToString(result);
    }

    private String decode(String data) {
        byte[] result = Base64.getDecoder().decode(data);
        for (int i = 0; i &lt; result.length; i++) {
            result[i] -= (byte) 1;
        }
        return new String(result);
    }
}</code></pre>
</figure>

#### []{#example-0--decorators-CompressionDecorator-java .anchor} **decorators/CompressionDecorator.java:** 圧縮デコレーター

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.decorator.example.decorators;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.Base64;
import java.util.zip.Deflater;
import java.util.zip.DeflaterOutputStream;
import java.util.zip.InflaterInputStream;

public class CompressionDecorator extends DataSourceDecorator {
    private int compLevel = 6;

    public CompressionDecorator(DataSource source) {
        super(source);
    }

    public int getCompressionLevel() {
        return compLevel;
    }

    public void setCompressionLevel(int value) {
        compLevel = value;
    }

    @Override
    public void writeData(String data) {
        super.writeData(compress(data));
    }

    @Override
    public String readData() {
        return decompress(super.readData());
    }

    private String compress(String stringData) {
        byte[] data = stringData.getBytes();
        try {
            ByteArrayOutputStream bout = new ByteArrayOutputStream(512);
            DeflaterOutputStream dos = new DeflaterOutputStream(bout, new Deflater(compLevel));
            dos.write(data);
            dos.close();
            bout.close();
            return Base64.getEncoder().encodeToString(bout.toByteArray());
        } catch (IOException ex) {
            return null;
        }
    }

    private String decompress(String stringData) {
        byte[] data = Base64.getDecoder().decode(stringData);
        try {
            InputStream in = new ByteArrayInputStream(data);
            InflaterInputStream iin = new InflaterInputStream(in);
            ByteArrayOutputStream bout = new ByteArrayOutputStream(512);
            int b;
            while ((b = iin.read()) != -1) {
                bout.write(b);
            }
            in.close();
            iin.close();
            bout.close();
            return new String(bout.toByteArray());
        } catch (IOException ex) {
            return null;
        }
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** クライアント・コード

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.decorator.example;

import refactoring_guru.decorator.example.decorators.*;

public class Demo {
    public static void main(String[] args) {
        String salaryRecords = &quot;Name,Salary\nJohn Smith,100000\nSteven Jobs,912000&quot;;
        DataSourceDecorator encoded = new CompressionDecorator(
                                         new EncryptionDecorator(
                                             new FileDataSource(&quot;out/OutputDemo.txt&quot;)));
        encoded.writeData(salaryRecords);
        DataSource plain = new FileDataSource(&quot;out/OutputDemo.txt&quot;);

        System.out.println(&quot;- Input ----------------&quot;);
        System.out.println(salaryRecords);
        System.out.println(&quot;- Encoded --------------&quot;);
        System.out.println(plain.readData());
        System.out.println(&quot;- Decoded --------------&quot;);
        System.out.println(encoded.readData());
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>- Input ----------------
Name,Salary
John Smith,100000
Steven Jobs,912000
- Encoded --------------
Zkt7e1Q5eU8yUm1Qe0ZsdHJ2VXp6dDBKVnhrUHtUe0sxRUYxQkJIdjVLTVZ0dVI5Q2IwOXFISmVUMU5rcENCQmdxRlByaD4+
- Decoded --------------
Name,Salary
John Smith,100000
Steven Jobs,912000</code></pre>
</figure>

::: next
#### 次を読む

[Facade を Java で []{.fa
.fa-arrow-right}](../../facade/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Composite を Java
で](../../composite/java/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Decorator**

[![Decorator を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Decorator を C# で"){.prog-lang-link}
[![Decorator を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Decorator を C++ で"){.prog-lang-link}
[![Decorator を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Decorator を Go で"){.prog-lang-link}
[![Decorator を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Decorator を PHP で"){.prog-lang-link}
[![Decorator を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Decorator を Python で"){.prog-lang-link}
[![Decorator を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Decorator を Ruby で"){.prog-lang-link}
[![Decorator を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Decorator を Rust で"){.prog-lang-link}
[![Decorator を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Decorator を Swift で"){.prog-lang-link}
[![Decorator を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Decorator を TypeScript で"){.prog-lang-link}
::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
