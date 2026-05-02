:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} / [Ланцюжок
обов\'язків](../../chain-of-responsibility.html){.type} /
[Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Ланцюжок
обов\'язків](../../../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x"}
:::

# **Ланцюжок обов\'язків** на Ruby {#ланцюжок-обовязків-на-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Ланцюжок обов'язків** --- це поведінковий патерн, що дозволяє
передавати запит ланцюжком потенційних обробників до тих пір, поки один
з них не обробить його.

Позбавляє від жорсткої прив'язки відправника запиту до одержувача,
дозволяючи динамічно вибудовувати ланцюг з різних обробників.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Ланцюжок обов\'язків
](../../chain-of-responsibility.html){.btn .btn-lg .btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
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
 [main](#example-0--main-rb)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::

**Складність:** []{.fa-stars}

**Популярність:** []{.fa-stars}

**Застосування:** Патерн зустрічається в Ruby не так вже й часто,
оскільки для його застосування потрібен ланцюг об'єктів, наприклад,
зв'язаний список.

**Ознаки застосування патерна:** Ланцюжок обов'язків можна визначити за
списками обробників або перевірок, через які пропускаються запити.
Особливо, якщо порядок проходження обробників є важливим.
:::::::::::

[]{#example-0}

## Концептуальний приклад {#example-0-title}

Цей приклад показує структуру патерна **Ланцюжок обов'язків**, а
саме --- з яких класів він складається, які ролі ці класи виконують і як
вони взаємодіють один з одним.

#### []{#example-0--main-rb .anchor} **main.rb:** Приклад структури патерна

<figure class="code">
<pre class="code" lang="ruby"><code># The Handler interface declares a method for building the chain of handlers. It
# also declares a method for executing a request.
class Handler
  # @abstract
  #
  # @param [Handler] handler
  def next_handler=(handler)
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end

  # @abstract
  #
  # @param [String] request
  #
  # @return [String, nil]
  def handle(request)
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# The default chaining behavior can be implemented inside a base handler class.
class AbstractHandler &lt; Handler
  # @return [Handler]
  attr_writer :next_handler

  # @param [Handler] handler
  #
  # @return [Handler]
  def next_handler(handler)
    @next_handler = handler
    # Returning a handler from here will let us link handlers in a convenient
    # way like this:
    # monkey.next_handler(squirrel).next_handler(dog)
    handler
  end

  # @abstract
  #
  # @param [String] request
  #
  # @return [String, nil]
  def handle(request)
    return @next_handler.handle(request) if @next_handler

    nil
  end
end

# All Concrete Handlers either handle a request or pass it to the next handler
# in the chain.
class MonkeyHandler &lt; AbstractHandler
  # @param [String] request
  #
  # @return [String, nil]
  def handle(request)
    if request == &#39;Banana&#39;
      &quot;Monkey: I&#39;ll eat the #{request}&quot;
    else
      super(request)
    end
  end
end

class SquirrelHandler &lt; AbstractHandler
  # @param [String] request
  #
  # @return [String, nil]
  def handle(request)
    if request == &#39;Nut&#39;
      &quot;Squirrel: I&#39;ll eat the #{request}&quot;
    else
      super(request)
    end
  end
end

class DogHandler &lt; AbstractHandler
  # @param [String] request
  #
  # @return [String, nil]
  def handle(request)
    if request == &#39;MeatBall&#39;
      &quot;Dog: I&#39;ll eat the #{request}&quot;
    else
      super(request)
    end
  end
end

# The client code is usually suited to work with a single handler. In most
# cases, it is not even aware that the handler is part of a chain.
def client_code(handler)
  [&#39;Nut&#39;, &#39;Banana&#39;, &#39;Cup of coffee&#39;].each do |food|
    puts &quot;\nClient: Who wants a #{food}?&quot;
    result = handler.handle(food)
    if result
      print &quot;  #{result}&quot;
    else
      print &quot;  #{food} was left untouched.&quot;
    end
  end
end

monkey = MonkeyHandler.new
squirrel = SquirrelHandler.new
dog = DogHandler.new

monkey.next_handler(squirrel).next_handler(dog)

# The client should be able to send a request to any handler, not just the first
# one in the chain.
puts &#39;Chain: Monkey &gt; Squirrel &gt; Dog&#39;
client_code(monkey)
puts &quot;\n\n&quot;

puts &#39;Subchain: Squirrel &gt; Dog&#39;
client_code(squirrel)</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Chain: Monkey &gt; Squirrel &gt; Dog

Client: Who wants a Nut?
  Squirrel: I&#39;ll eat the Nut
Client: Who wants a Banana?
  Monkey: I&#39;ll eat the Banana
Client: Who wants a Cup of coffee?
  Cup of coffee was left untouched.

Subchain: Squirrel &gt; Dog

Client: Who wants a Nut?
  Squirrel: I&#39;ll eat the Nut
Client: Who wants a Banana?
  Banana was left untouched.
Client: Who wants a Cup of coffee?
  Cup of coffee was left untouched.</code></pre>
</figure>

## **Ланцюжок обов\'язків** іншими мовами програмування

[![Ланцюжок обов\'язків на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Ланцюжок обов'язків на C#"){.prog-lang-link}
[![Ланцюжок обов\'язків на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Ланцюжок обов'язків на C++"){.prog-lang-link}
[![Ланцюжок обов\'язків на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Ланцюжок обов'язків на Go"){.prog-lang-link}
[![Ланцюжок обов\'язків на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Ланцюжок обов'язків на Java"){.prog-lang-link}
[![Ланцюжок обов\'язків на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Ланцюжок обов'язків на PHP"){.prog-lang-link}
[![Ланцюжок обов\'язків на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Ланцюжок обов'язків на Python"){.prog-lang-link}
[![Ланцюжок обов\'язків на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Ланцюжок обов'язків на Rust"){.prog-lang-link}
[![Ланцюжок обов\'язків на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Ланцюжок обов'язків на Swift"){.prog-lang-link}
[![Ланцюжок обов\'язків на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Ланцюжок обов'язків на TypeScript"){.prog-lang-link}
:::::::::::::::

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
:::::::::::::::::::::
::::::::::::::::::::::
