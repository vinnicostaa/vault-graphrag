::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Iterator](../../iterator.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Iterator](../../../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x"}
:::

# **Iterator** em Go {#iterator-em-go .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
O **Iterador** é um padrão de projeto comportamental que permite a
passagem sequencial através de uma estrutura de dados complexa sem expor
seus detalhes internos.

Graças ao Iterator, os clientes podem examinar elementos de diferentes
coleções de maneira semelhante usando uma única interface iterador.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Iterator ](../../iterator.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegação
:::

::: en-intro
 [Introdução](#)
:::

::: en-intro
 [Exemplo conceitual](#example-0)
:::

::: en-file
 [collection](#example-0--collection-go)
:::

::: en-file
 [user­Collection](#example-0--userCollection-go)
:::

::: en-file
 [iterator](#example-0--iterator-go)
:::

::: en-file
 [user­Iterator](#example-0--userIterator-go)
:::

::: en-file
 [user](#example-0--user-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::::
::::::::::::::::

[]{#example-0}

## Exemplo conceitual {#example-0-title}

A ideia principal por trás do padrão Iterator é extrair a lógica de
iteração de uma coleção em um objeto diferente denominado iterador. Este
iterador fornece um método genérico de iteração sobre uma coleção
independente de seu tipo.

#### []{#example-0--collection-go .anchor} **collection.go:** Coleção

<figure class="code">
<pre class="code" lang="go"><code>package main

type Collection interface {
    createIterator() Iterator
}</code></pre>
</figure>

#### []{#example-0--userCollection-go .anchor} **userCollection.go:** Coleção concreta

<figure class="code">
<pre class="code" lang="go"><code>package main

type UserCollection struct {
    users []*User
}

func (u *UserCollection) createIterator() Iterator {
    return &amp;UserIterator{
        users: u.users,
    }
}</code></pre>
</figure>

#### []{#example-0--iterator-go .anchor} **iterator.go:** Iterador

<figure class="code">
<pre class="code" lang="go"><code>package main

type Iterator interface {
    hasNext() bool
    getNext() *User
}</code></pre>
</figure>

#### []{#example-0--userIterator-go .anchor} **userIterator.go:** Iterador concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

type UserIterator struct {
    index int
    users []*User
}

func (u *UserIterator) hasNext() bool {
    if u.index &lt; len(u.users) {
        return true
    }
    return false

}
func (u *UserIterator) getNext() *User {
    if u.hasNext() {
        user := u.users[u.index]
        u.index++
        return user
    }
    return nil
}</code></pre>
</figure>

#### []{#example-0--user-go .anchor} **user.go:** Código cliente

<figure class="code">
<pre class="code" lang="go"><code>package main

type User struct {
    name string
    age  int
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Código cliente

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {

    user1 := &amp;User{
        name: &quot;a&quot;,
        age:  30,
    }
    user2 := &amp;User{
        name: &quot;b&quot;,
        age:  20,
    }

    userCollection := &amp;UserCollection{
        users: []*User{user1, user2},
    }

    iterator := userCollection.createIterator()

    for iterator.hasNext() {
        user := iterator.getNext()
        fmt.Printf(&quot;User is %+v\n&quot;, user)
    }
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>User is &amp;{name:a age:30}
User is &amp;{name:b age:20}</code></pre>
</figure>

::: next
#### Leia a seguir

[Mediator em Go []{.fa
.fa-arrow-right}](../../mediator/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Command em
Go](../../command/go/example.html){.btn .btn-default rel="prev"}
:::

## **Iterator** em outras linguagens

[![Iterator em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Iterator em C#"){.prog-lang-link}
[![Iterator em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Iterator em C++"){.prog-lang-link}
[![Iterator em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Iterator em Java"){.prog-lang-link}
[![Iterator em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Iterator em PHP"){.prog-lang-link}
[![Iterator em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Iterator em Python"){.prog-lang-link}
[![Iterator em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Iterator em Ruby"){.prog-lang-link}
[![Iterator em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Iterator em Rust"){.prog-lang-link}
[![Iterator em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Iterator em Swift"){.prog-lang-link}
[![Iterator em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Iterator em TypeScript"){.prog-lang-link}
::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pt-br" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Arquivo com exemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compre o eBook **Mergulho nos Padrões de Projeto** e tenha acesso ao
arquivo com dúzias de exemplos detalhados que podem ser abertos
diretamente em seu IDE.

[ Saiba mais...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::
