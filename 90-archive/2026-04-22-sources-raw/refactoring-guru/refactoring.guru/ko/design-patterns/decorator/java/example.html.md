::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[데코레이터](../../decorator.html){.type} /
[자바](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![데코레이터](../../../../images/patterns/cards/decorator-mini6adb.png?id=d30458908e315af195cb183bc52dbef9){srcset="/images/patterns/cards/decorator-mini-2x.png?id=3b58e540d7d28523080cad341ed9b2e9 2x"}
:::

# 자바로 작성된 **데코레이터** {#자바로-작성된-데코레이터 .pattern-example-title-block-title}
::::

:::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**데코레이터**는 구조 패턴이며 새로운 행동들을 특수 래퍼 객체들 내에
넣어서 이러한 행동들을 객체들에 동적으로 추가할 수 있도록 합니다.

데코레이터를 사용하여 객체들을 제한 없이 래핑할 수 있습니다. 왜냐하면
대상 객체들과 데코레이터들은 같은 인터페이스를 따르기 때문입니다. 결과
객체는 모든 래퍼의 스태킹된 행동을 가질 것입니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 데코레이터에 대하여 더 자세히 알아보세요 ](../../decorator.html){.btn
.btn-lg .btn-primary}
:::

::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
내비게이션
:::

::: en-intro
 [소개](#)
:::

::: en-intro
 [인코딩 및 압축 데코레이터들](#example-0)
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

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 예시들:** 데코레이터는 자바 코드, 특히 스트림과 관련된 코드에서
꽤 표준적입니다.

다음은 코어 자바 라이브러리로부터 가져온 데코레이터의 몇 가지
예시들입니다:

-   [`java.io.InputStream`](http://docs.oracle.com/javase/8/docs/api/java/io/InputStream.html),
    [`Output­Stream`](http://docs.oracle.com/javase/8/docs/api/java/io/OutputStream.html),
    [`Reader`](http://docs.oracle.com/javase/8/docs/api/java/io/Reader.html)
    와
    [`Writer`](http://docs.oracle.com/javase/8/docs/api/java/io/Writer.html)의
    모든 자식 클래스들은 같은 유형의 객체들을 수락하는 생성자들이
    있습니다.

-   [`java.util.Collections`](http://docs.oracle.com/javase/8/docs/api/java/util/Collections.html)와
    그의 메서드들:
    [`checked­XXX()`](http://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#checkedCollection-java.util.Collection-java.lang.Class-),
    [`synchronized­XXX()`](http://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#synchronizedCollection-java.util.Collection-),
    [`unmodifiable­XXX()`](http://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#unmodifiableCollection-java.util.Collection-).

-   [`javax.servlet.http.HttpServletRequestWrapper`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServletRequestWrapper.html)와
    [`Http­Servlet­Response­Wrapper`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServletResponseWrapper.html)

**식별:** 데코레이터는 같은 클래스의 객체 또는 인터페이스를 현재
클래스로 수락하는 생성 메서드들 또는 생성자들로 인식할 수 있습니다.
::::::::::::::::::::::

[]{#example-0}

## 인코딩 및 압축 데코레이터들 {#example-0-title}

이 예시는 코드를 변경하지 않고 객체의 행동을 조정하는 방법을 보여줍니다.

처음에 비즈니스 로직 클래스는 일반 텍스트로만 데이터를 읽고 쓸 수
있었으나 그다음 표준 작업을 래핑 된 객체 내에 실행 후 새로운 행동을
추가하는 몇 개의 작은 래퍼 클래스들을 만들었습니다.

첫 번째 래퍼는 데이터를 암호화 및 해독하고 두 번째 래퍼는 데이터를 압축
및 추출합니다.

또 한 데코레이터를 다른 데코레이터로 래핑하여 이러한 래퍼들을 결합할
수도 있습니다.

### []{#example-0--decorators .anchor} **decorators** {#decorators .example-title}

#### []{#example-0--decorators-DataSource-java .anchor} **decorators/DataSource.java:** 읽기 및 쓰기 작업을 정의하는 공통 데이터 인터페이스

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.decorator.example.decorators;

public interface DataSource {
    void writeData(String data);

    String readData();
}</code></pre>
</figure>

#### []{#example-0--decorators-FileDataSource-java .anchor} **decorators/FileDataSource.java:** 간단한 데이터 판독기-작성기

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

#### []{#example-0--decorators-DataSourceDecorator-java .anchor} **decorators/DataSourceDecorator.java:** 추상 기초 데코레이터

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

#### []{#example-0--decorators-EncryptionDecorator-java .anchor} **decorators/EncryptionDecorator.java:** 암호화 데코레이터

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

#### []{#example-0--decorators-CompressionDecorator-java .anchor} **decorators/CompressionDecorator.java:** 압축 데코레이터

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** 클라이언트 코드

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

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** 실행 결과

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
#### 다음을 보세요

[자바로 작성된 퍼사드 []{.fa
.fa-arrow-right}](../../facade/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 자바로 작성된
복합체](../../composite/java/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **데코레이터**

[![C#으로 작성된
데코레이터](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 데코레이터"){.prog-lang-link}
[![C++로 작성된
데코레이터](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 데코레이터"){.prog-lang-link}
[![Go로 작성된
데코레이터](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 데코레이터"){.prog-lang-link}
[![PHP로 작성된
데코레이터](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 데코레이터"){.prog-lang-link}
[![파이썬으로 작성된
데코레이터](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 데코레이터"){.prog-lang-link}
[![루비로 작성된
데코레이터](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 데코레이터"){.prog-lang-link}
[![러스트로 작성된
데코레이터](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 데코레이터"){.prog-lang-link}
[![스위프트로 작성된
데코레이터](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 데코레이터"){.prog-lang-link}
[![타입스크립트로 작성된
데코레이터](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 데코레이터"){.prog-lang-link}
::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
