:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Proxy](../../proxy.html){.type} / [Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Proxy](../../../../images/patterns/cards/proxy-minid8d7.png?id=25890b11e7dc5af29625ccd0678b63a8){srcset="/images/patterns/cards/proxy-mini-2x.png?id=8638fac9dc08c992852492f9cb29d9c6 2x"}
:::

# **Proxy** em Ruby {#proxy-em-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
O **Proxy** é um padrão de projeto estrutural que fornece um objeto que
atua como um substituto para um objeto de serviço real usado por um
cliente. Um proxy recebe solicitações do cliente, realiza alguma tarefa
(controle de acesso, armazenamento em cache etc.) e passa a solicitação
para um objeto de serviço.

O objeto proxy tem a mesma interface que um serviço, o que o torna
intercambiável com um objeto real quando passado para um cliente.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Proxy ](../../proxy.html){.btn .btn-lg
.btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
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
 [main](#example-0--main-rb)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::

**Complexidade:** []{.fa-stars}

**Popularidade:** []{.fa-stars}

**Exemplos de uso:** Embora o padrão Proxy não seja um convidado
frequente na maioria das aplicações Ruby, ele ainda é muito útil em
alguns casos especiais. É insubstituível quando você deseja adicionar
alguns comportamentos adicionais a um objeto de alguma classe existente
sem alterar o código cliente.

**Identificação:** Proxies delegam todo o trabalho real para algum outro
objeto. Cada método de proxy deve, no final, se referir a um objeto de
serviço, a menos que o proxy seja uma subclasse de um serviço.
:::::::::::

[]{#example-0}

## Exemplo conceitual {#example-0-title}

Este exemplo ilustra a estrutura do padrão de projeto **Proxy**. Ele se
concentra em responder a estas perguntas:

-   De quais classes ele consiste?
-   Quais papéis essas classes desempenham?
-   De que maneira os elementos do padrão estão relacionados?

#### []{#example-0--main-rb .anchor} **main.rb:** Exemplo conceitual

<figure class="code">
<pre class="code" lang="ruby"><code># The Subject interface declares common operations for both RealSubject and the
# Proxy. As long as the client works with RealSubject using this interface,
# you&#39;ll be able to pass it a proxy instead of a real subject.
class Subject
  # @abstract
  def request
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# The RealSubject contains some core business logic. Usually, RealSubjects are
# capable of doing some useful work which may also be very slow or sensitive -
# e.g. correcting input data. A Proxy can solve these issues without any changes
# to the RealSubject&#39;s code.
class RealSubject &lt; Subject
  def request
    puts &#39;RealSubject: Handling request.&#39;
  end
end

# The Proxy has an interface identical to the RealSubject.
class Proxy &lt; Subject
  # @param [RealSubject] real_subject
  def initialize(real_subject)
    @real_subject = real_subject
  end

  # The most common applications of the Proxy pattern are lazy loading, caching,
  # controlling the access, logging, etc. A Proxy can perform one of these
  # things and then, depending on the result, pass the execution to the same
  # method in a linked RealSubject object.
  def request
    return unless check_access

    @real_subject.request
    log_access
  end

  # @return [Boolean]
  def check_access
    puts &#39;Proxy: Checking access prior to firing a real request.&#39;
    true
  end

  def log_access
    print &#39;Proxy: Logging the time of request.&#39;
  end
end

# The client code is supposed to work with all objects (both subjects and
# proxies) via the Subject interface in order to support both real subjects and
# proxies. In real life, however, clients mostly work with their real subjects
# directly. In this case, to implement the pattern more easily, you can extend
# your proxy from the real subject&#39;s class.
def client_code(subject)
  # ...

  subject.request

  # ...
end

puts &#39;Client: Executing the client code with a real subject:&#39;
real_subject = RealSubject.new
client_code(real_subject)

puts &quot;\n&quot;

puts &#39;Client: Executing the same client code with a proxy:&#39;
proxy = Proxy.new(real_subject)
client_code(proxy)</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Client: Executing the client code with a real subject:
RealSubject: Handling request.

Client: Executing the same client code with a proxy:
Proxy: Checking access prior to firing a real request.
RealSubject: Handling request.
Proxy: Logging the time of request.</code></pre>
</figure>

## **Proxy** em outras linguagens

[![Proxy em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Proxy em C#"){.prog-lang-link}
[![Proxy em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Proxy em C++"){.prog-lang-link}
[![Proxy em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Proxy em Go"){.prog-lang-link}
[![Proxy em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Proxy em Java"){.prog-lang-link}
[![Proxy em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Proxy em PHP"){.prog-lang-link}
[![Proxy em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Proxy em Python"){.prog-lang-link}
[![Proxy em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Proxy em Rust"){.prog-lang-link}
[![Proxy em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Proxy em Swift"){.prog-lang-link}
[![Proxy em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Proxy em TypeScript"){.prog-lang-link}
:::::::::::::::

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
:::::::::::::::::::::
::::::::::::::::::::::
