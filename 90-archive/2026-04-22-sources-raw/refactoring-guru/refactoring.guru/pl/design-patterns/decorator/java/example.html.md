::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Dekorator](../../decorator.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Dekorator](../../../../images/patterns/cards/decorator-mini6adb.png?id=d30458908e315af195cb183bc52dbef9){srcset="/images/patterns/cards/decorator-mini-2x.png?id=3b58e540d7d28523080cad341ed9b2e9 2x"}
:::

# **Dekorator** w języku Java {#dekorator-w-języku-java .pattern-example-title-block-title}
::::

:::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Dekorator** to strukturalny wzorzec pozwalający na dodawanie obiektom
nowych obowiązków w sposób dynamiczny --- poprzez "opakowywanie" ich w
specjalne obiekty posiadające potrzebną funkcjonalność.

Stosując dekoratory można opakowywać obiekty wielokrotnie, gdyż zarówno
obiekt docelowy jak i dekoratory są zgodne pod względem interfejsu.
Wynikowy obiekt będzie posiadał ułożoną w formie stosu połączoną
funkcjonalność wszystkich "opakowań".
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Dekorator ](../../decorator.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Dekoratory szyfrujące i kompresujące](#example-0)
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

**Złożoność:** []{.fa-stars}

**Popularność:** []{.fa-stars}

**Przykłady użycia:** Dekorator jest dość typowy w kodzie Java,
szczególnie tym dotyczącym strumieni.

Oto kilka przykładów dekoratorów w głównych bibliotekach Java:

-   Wszystkie klasy pochodne
    [`java.io.InputStream`](http://docs.oracle.com/javase/8/docs/api/java/io/InputStream.html),
    [`OutputStream`](http://docs.oracle.com/javase/8/docs/api/java/io/OutputStream.html),
    [`Reader`](http://docs.oracle.com/javase/8/docs/api/java/io/Reader.html)
    oraz
    [`Writer`](http://docs.oracle.com/javase/8/docs/api/java/io/Writer.html)
    mają konstruktory przyjmujące obiekty własnego typu.

-   [`java.util.Collections`](http://docs.oracle.com/javase/8/docs/api/java/util/Collections.html),
    metody
    [`checkedXXX()`](http://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#checkedCollection-java.util.Collection-java.lang.Class-),
    [`synchronizedXXX()`](http://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#synchronizedCollection-java.util.Collection-)
    oraz
    [`unmodifiableXXX()`](http://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#unmodifiableCollection-java.util.Collection-).

-   [`javax.servlet.http.HttpServletRequestWrapper`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServletRequestWrapper.html)
    oraz
    [`HttpServletResponseWrapper`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServletResponseWrapper.html)

**Identyfikacja:** Dekorator można poznać po metodach kreacyjnych lub
konstruktorach przyjmujących obiekty tej samej klasy lub interfejsu jako
bieżącą klasę.
::::::::::::::::::::::

[]{#example-0}

## Dekoratory szyfrujące i kompresujące {#example-0-title}

Poniższy przykład pokazuje jak można dostosować zachowanie obiektu bez
zmiany jego kodu.

Początkowo badana klasa z logiką biznesową mogła jedynie odczytywać i
zapisywać dane w formie zwykłego tekstu. Następnie utworzyliśmy wiele
mniejszych klas opakowujących, które dodają nową funkcjonalność po
wykonaniu standardowych działań opakowanego obiektu.

Pierwsza nakładka szyfruje i odszyfrowuje dane, a druga kompresuje i
dekompresuje dane.

Można nawet połączyć obie nakładki --- opakowując jedną z nich w drugą.

### []{#example-0--decorators .anchor} **decorators** {#decorators .example-title}

#### []{#example-0--decorators-DataSource-java .anchor} **decorators/DataSource.java:** Wspólny interfejs danych definiujący operacje odczytu i zapisu

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.decorator.example.decorators;

public interface DataSource {
    void writeData(String data);

    String readData();
}</code></pre>
</figure>

#### []{#example-0--decorators-FileDataSource-java .anchor} **decorators/FileDataSource.java:** Prosty obiekt odczytujący/zapisujący

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

#### []{#example-0--decorators-DataSourceDecorator-java .anchor} **decorators/DataSourceDecorator.java:** Abstrakcyjny dekorator bazowy

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

#### []{#example-0--decorators-EncryptionDecorator-java .anchor} **decorators/EncryptionDecorator.java:** Dekorator szyfrujący

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

#### []{#example-0--decorators-CompressionDecorator-java .anchor} **decorators/CompressionDecorator.java:** Dekorator kompresujący

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** Kod klienta

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

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Wynik działania

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
#### Czytaj dalej

[Fasada w języku Java []{.fa
.fa-arrow-right}](../../facade/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Kompozyt w języku
Java](../../composite/java/example.html){.btn .btn-default rel="prev"}
:::

## **Dekorator** w innych językach

[![Dekorator w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Dekorator w języku C#"){.prog-lang-link}
[![Dekorator w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Dekorator w języku C++"){.prog-lang-link}
[![Dekorator w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Dekorator w języku Go"){.prog-lang-link}
[![Dekorator w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Dekorator w języku PHP"){.prog-lang-link}
[![Dekorator w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Dekorator w języku Python"){.prog-lang-link}
[![Dekorator w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Dekorator w języku Ruby"){.prog-lang-link}
[![Dekorator w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Dekorator w języku Rust"){.prog-lang-link}
[![Dekorator w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Dekorator w języku Swift"){.prog-lang-link}
[![Dekorator w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Dekorator w języku TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pl" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archiwum\
przykładów kodu](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Kupując eBook **Wzorce projektowe. Nowoczesny podręcznik** zyskujesz
dostęp do archiwum zawierającego mnóstwo szczegółowych przykładów, które
można od razu wkleić do swojego preferowanego zintegrowanego środowiska
programistycznego.

[ Dowiedz się więcej...](../../book.html){.btn .btn-secondary
.btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
