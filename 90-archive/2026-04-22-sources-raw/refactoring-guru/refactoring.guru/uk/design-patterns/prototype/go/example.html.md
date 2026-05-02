::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Прототип](../../prototype.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Прототип](../../../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x"}
:::

# **Прототип** на Go {#прототип-на-go .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Прототип** --- це породжуючий патерн, який дозволяє копіювати об'єкти
будь-якої складності без прив'язки до їхніх конкретних класів.

Усі класи-Прототипи мають спільний інтерфейс. Тому ви можете копіювати
об'єкти, не звертаючи уваги на їхні конкретні типи та бути завжди
впевненими в тому, що отримаєте точну копію. Клонування здійснюється
самим об'єктом-прототипу, що дозволяє йому скопіювати значення всіх
полів, навіть приватних.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Прототип ](../../prototype.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навігація
:::

::: en-intro
 [Інтро](#)
:::

::: en-intro
 [Концептуальний приклад](#example-0)
:::

::: en-file
 [inode](#example-0--inode-go)
:::

::: en-file
 [file](#example-0--file-go)
:::

::: en-file
 [folder](#example-0--folder-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::
::::::::::::::

[]{#example-0}

## Концептуальний приклад {#example-0-title}

Давайте спробуємо розібрати патерн Прототип, використовуючи за приклад
файлову систему ОС. Файлова система є рекурсивною --- папки містять
файли та інші папки, які, в свою чергу, можуть містити файли та папки, і
так далі.

Кожен файл і папка можуть бути представлені інтерфейсом `inode`. Він має
функцію `clone`.

Обидві структури файлу й папки --- `file` і` folder` --- реалізують
функції `print` і` clone`, оскільки вони мають тип `inode`. Також,
зверніть увагу на функцію `clone` в` file` і `folder`. Функція `clone` в
обох випадках повертає копію відповідного файлу або папки. Під час
клонування ми додаємо ключове слово «\_clone» в поле імені.

#### []{#example-0--inode-go .anchor} **inode.go:** Інтерфейс прототипа

<figure class="code">
<pre class="code" lang="go"><code>package main

type Inode interface {
    print(string)
    clone() Inode
}</code></pre>
</figure>

#### []{#example-0--file-go .anchor} **file.go:** Конкретний прототип

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type File struct {
    name string
}

func (f *File) print(indentation string) {
    fmt.Println(indentation + f.name)
}

func (f *File) clone() Inode {
    return &amp;File{name: f.name + &quot;_clone&quot;}
}</code></pre>
</figure>

#### []{#example-0--folder-go .anchor} **folder.go:** Конкретний прототип

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Folder struct {
    children []Inode
    name     string
}

func (f *Folder) print(indentation string) {
    fmt.Println(indentation + f.name)
    for _, i := range f.children {
        i.print(indentation + indentation)
    }
}

func (f *Folder) clone() Inode {
    cloneFolder := &amp;Folder{name: f.name + &quot;_clone&quot;}
    var tempChildren []Inode
    for _, i := range f.children {
        copy := i.clone()
        tempChildren = append(tempChildren, copy)
    }
    cloneFolder.children = tempChildren
    return cloneFolder
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Клієнтський код

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {
    file1 := &amp;File{name: &quot;File1&quot;}
    file2 := &amp;File{name: &quot;File2&quot;}
    file3 := &amp;File{name: &quot;File3&quot;}

    folder1 := &amp;Folder{
        children: []Inode{file1},
        name:     &quot;Folder1&quot;,
    }

    folder2 := &amp;Folder{
        children: []Inode{folder1, file2, file3},
        name:     &quot;Folder2&quot;,
    }
    fmt.Println(&quot;\nPrinting hierarchy for Folder2&quot;)
    folder2.print(&quot;  &quot;)

    cloneFolder := folder2.clone()
    fmt.Println(&quot;\nPrinting hierarchy for clone Folder&quot;)
    cloneFolder.print(&quot;  &quot;)
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Printing hierarchy for Folder2
  Folder2
    Folder1
        File1
    File2
    File3

Printing hierarchy for clone Folder
  Folder2_clone
    Folder1_clone
        File1_clone
    File2_clone
    File3_clone</code></pre>
</figure>

::: next
#### Читаємо далі

[Одинак на Go []{.fa
.fa-arrow-right}](../../singleton/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Фабричний метод на
Go](../../factory-method/go/example.html){.btn .btn-default rel="prev"}
:::

## **Прототип** іншими мовами програмування

[![Прототип на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Прототип на C#"){.prog-lang-link}
[![Прототип на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Прототип на C++"){.prog-lang-link}
[![Прототип на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Прототип на Java"){.prog-lang-link}
[![Прототип на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Прототип на PHP"){.prog-lang-link}
[![Прототип на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Прототип на Python"){.prog-lang-link}
[![Прототип на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Прототип на Ruby"){.prog-lang-link}
[![Прототип на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Прототип на Rust"){.prog-lang-link}
[![Прототип на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Прототип на Swift"){.prog-lang-link}
[![Прототип на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Прототип на TypeScript"){.prog-lang-link}
::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-uk" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архів з прикладами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Придбай книгу **Занурення в Патерни** та отримай архів з десятками
детальних прикладів, які можна відкривати прямо в IDE.

[ Дізнатися більше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::
:::::::::::::::::::::::::::
