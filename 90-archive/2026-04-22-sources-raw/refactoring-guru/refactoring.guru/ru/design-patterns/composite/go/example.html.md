::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Компоновщик](../../composite.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Компоновщик](../../../../images/patterns/cards/composite-minib543.png?id=a369d98d18b417f255d04568fd0131b8){srcset="/images/patterns/cards/composite-mini-2x.png?id=3f7f811fefeb0b64f6774746eb42af09 2x"}
:::

# **Компоновщик** на Go {#компоновщик-на-go .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Компоновщик** --- это структурный паттерн, который позволяет создавать
дерево объектов и работать с ним так же, как и с единичным объектом.

Компоновщик давно стал синонимом всех задач, связанных с построением
дерева объектов. Все операции компоновщика основаны на рекурсии и
«суммировании» результатов на ветвях дерева.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Компоновщик ](../../composite.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Концептуальный пример](#example-0)
:::

::: en-file
 [component](#example-0--component-go)
:::

::: en-file
 [folder](#example-0--folder-go)
:::

::: en-file
 [file](#example-0--file-go)
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

## Концептуальный пример {#example-0-title}

Давайте попробуем понять паттерн Компоновщик, используя для примера
файловую систему ОС. Внутри нее есть два типа объектов: файлы и папки. В
некоторых случаях они должны восприниматься как одно и то же. Здесь нам
пригодится паттерн проектирования Компоновщик.

Представьте, что вам нужно провести поиск по конкретному ключевому слову
в вашей файловой системе. Такая операция поиска применяется в равной
степени и к файлам, и к папкам. В случае файла, она только проверит
содержимое файла, а в случае папки -- обработает все файлы этой папки
для нахождения ключевого слова.

#### []{#example-0--component-go .anchor} **component.go:** Интерфейс компонента

<figure class="code">
<pre class="code" lang="go"><code>package main

type Component interface {
    search(string)
}</code></pre>
</figure>

#### []{#example-0--folder-go .anchor} **folder.go:** Компоновщик

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Folder struct {
    components []Component
    name       string
}

func (f *Folder) search(keyword string) {
    fmt.Printf(&quot;Serching recursively for keyword %s in folder %s\n&quot;, keyword, f.name)
    for _, composite := range f.components {
        composite.search(keyword)
    }
}

func (f *Folder) add(c Component) {
    f.components = append(f.components, c)
}</code></pre>
</figure>

#### []{#example-0--file-go .anchor} **file.go:** Лист

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type File struct {
    name string
}

func (f *File) search(keyword string) {
    fmt.Printf(&quot;Searching for keyword %s in file %s\n&quot;, keyword, f.name)
}

func (f *File) getName() string {
    return f.name
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Клиентский код

<figure class="code">
<pre class="code" lang="go"><code>package main

func main() {
    file1 := &amp;File{name: &quot;File1&quot;}
    file2 := &amp;File{name: &quot;File2&quot;}
    file3 := &amp;File{name: &quot;File3&quot;}

    folder1 := &amp;Folder{
        name: &quot;Folder1&quot;,
    }

    folder1.add(file1)

    folder2 := &amp;Folder{
        name: &quot;Folder2&quot;,
    }
    folder2.add(file2)
    folder2.add(file3)
    folder2.add(folder1)

    folder2.search(&quot;rose&quot;)
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Serching recursively for keyword rose in folder Folder2
Searching for keyword rose in file File2
Searching for keyword rose in file File3
Serching recursively for keyword rose in folder Folder1
Searching for keyword rose in file File1</code></pre>
</figure>

::: next
#### Читаем дальше

[Декоратор на Go []{.fa
.fa-arrow-right}](../../decorator/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Мост на Go](../../bridge/go/example.html){.btn
.btn-default rel="prev"}
:::

## **Компоновщик** на других языках программирования

[![Компоновщик на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Компоновщик на C#"){.prog-lang-link}
[![Компоновщик на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Компоновщик на C++"){.prog-lang-link}
[![Компоновщик на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Компоновщик на Java"){.prog-lang-link}
[![Компоновщик на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Компоновщик на PHP"){.prog-lang-link}
[![Компоновщик на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Компоновщик на Python"){.prog-lang-link}
[![Компоновщик на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Компоновщик на Ruby"){.prog-lang-link}
[![Компоновщик на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Компоновщик на Rust"){.prog-lang-link}
[![Компоновщик на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Компоновщик на Swift"){.prog-lang-link}
[![Компоновщик на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Компоновщик на TypeScript"){.prog-lang-link}
::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ru" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архив с примерами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Купи книгу **Погружение в Паттерны** и получи архив с десятками
детальных примеров, которые можно открывать прямо в IDE.

[ Узнать больше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::
:::::::::::::::::::::::::::
