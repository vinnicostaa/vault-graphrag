::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Decorator](../../decorator.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Decorator](../../../images/patterns/cards/decorator-mini6adb.png?id=d30458908e315af195cb183bc52dbef9){srcset="/images/patterns/cards/decorator-mini-2x.png?id=3b58e540d7d28523080cad341ed9b2e9 2x"}
:::

# **Decorator** in Java {#decorator-in-java .pattern-example-title-block-title}
::::

:::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Decorator** is a structural pattern that allows adding new behaviors
to objects dynamically by placing them inside special wrapper objects,
called *decorators*.

Using decorators you can wrap objects countless number of times since
both target objects and decorators follow the same interface. The
resulting object will get a stacking behavior of all wrappers.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Decorator ](../../decorator.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Encoding and compression decorators](#example-0)
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

**Complexity:** []{.fa-stars}

**Popularity:** []{.fa-stars}

**Usage examples:** The Decorator is pretty standard in Java code,
especially in code related to streams.

Here are some examples of Decorator in core Java libraries:

-   All subclasses of
    [`java.io.InputStream`](http://docs.oracle.com/javase/8/docs/api/java/io/InputStream.html),
    [`OutputStream`](http://docs.oracle.com/javase/8/docs/api/java/io/OutputStream.html),
    [`Reader`](http://docs.oracle.com/javase/8/docs/api/java/io/Reader.html)
    and
    [`Writer`](http://docs.oracle.com/javase/8/docs/api/java/io/Writer.html)
    have constructors that accept objects of their own type.

-   [`java.util.Collections`](http://docs.oracle.com/javase/8/docs/api/java/util/Collections.html),
    methods
    [`checkedXXX()`](http://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#checkedCollection-java.util.Collection-java.lang.Class-),
    [`synchronizedXXX()`](http://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#synchronizedCollection-java.util.Collection-)
    and
    [`unmodifiableXXX()`](http://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#unmodifiableCollection-java.util.Collection-).

-   [`javax.servlet.http.HttpServletRequestWrapper`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServletRequestWrapper.html)
    and
    [`HttpServletResponseWrapper`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServletResponseWrapper.html)

**Identification:** Decorator can be recognized by creation methods or
constructors that accept objects of the same class or interface as a
current class.
::::::::::::::::::::::

[]{#example-0}

## Encoding and compression decorators {#example-0-title}

This example shows how you can adjust the behavior of an object without
changing its code.

Initially, the business logic class could only read and write data in
plain text. Then we created several small wrapper classes that add new
behavior after executing standard operations in a wrapped object.

The first wrapper encrypts and decrypts data, and the second one
compresses and extracts data.

You can even combine these wrappers by wrapping one decorator with
another.

### []{#example-0--decorators .anchor} **decorators** {#decorators .example-title}

#### []{#example-0--decorators-DataSource-java .anchor} **decorators/DataSource.java:** A common data interface, which defines read and write operations

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.decorator.example.decorators;

public interface DataSource {
    void writeData(String data);

    String readData();
}</code></pre>
</figure>

#### []{#example-0--decorators-FileDataSource-java .anchor} **decorators/FileDataSource.java:** Simple data reader-writer

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

#### []{#example-0--decorators-DataSourceDecorator-java .anchor} **decorators/DataSourceDecorator.java:** Abstract base decorator

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

#### []{#example-0--decorators-EncryptionDecorator-java .anchor} **decorators/EncryptionDecorator.java:** Encryption decorator

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

#### []{#example-0--decorators-CompressionDecorator-java .anchor} **decorators/CompressionDecorator.java:** Compression decorator

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** Client code

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

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Execution result

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
#### Read next

[Facade in Java []{.fa
.fa-arrow-right}](../../facade/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Composite in
Java](../../composite/java/example.html){.btn .btn-default rel="prev"}
:::

## **Decorator** in Other Languages

[![Decorator in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Decorator in C#"){.prog-lang-link}
[![Decorator in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Decorator in C++"){.prog-lang-link}
[![Decorator in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Decorator in Go"){.prog-lang-link}
[![Decorator in
PHP](../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Decorator in PHP"){.prog-lang-link}
[![Decorator in
Python](../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Decorator in Python"){.prog-lang-link}
[![Decorator in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Decorator in Ruby"){.prog-lang-link}
[![Decorator in
Rust](../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Decorator in Rust"){.prog-lang-link}
[![Decorator in
Swift](../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Decorator in Swift"){.prog-lang-link}
[![Decorator in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Decorator in TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
