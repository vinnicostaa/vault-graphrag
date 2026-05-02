:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Singleton](../../singleton.html){.type} /
[Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Singleton](../../../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x"}
:::

# **Singleton** w języku Ruby {#singleton-w-języku-ruby .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Singleton** to kreacyjny wzorzec projektowy gwarantujący istnienie
tylko jednego obiektu danego rodzaju. Udostępnia też pojedynczy punkt
dostępowy do takiego obiektu z dowolnego miejsca w programie.

Singleton charakteryzuje się prawie takimi samymi zaletami i wadami jak
zmienne globalne i chociaż jest bardzo poręczny, to jednak psuje
modularność kodu.

Nie można przenieść klasy zależnej od Singletona i użyć jej w innym
kontekście bez równoczesnego przeniesienia tego drugiego. To
ograniczenie zazwyczaj ujawnia się na etapie tworzenia testów
jednostkowych.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Singleton ](../../singleton.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Implementacja naiwna](#example-0)
:::

::: en-file
 [main](#example-0--main-rb)
:::

::: en-file
 [output](#example-0--output-txt)
:::

::: en-intro
 [Singleton z bezpieczeństwem wątków](#example-1)
:::

::: en-file
 [main](#example-1--main-rb)
:::

::: en-file
 [output](#example-1--output-txt)
:::
:::::::::::

**Złożoność:** []{.fa-stars}

**Popularność:** []{.fa-stars}

**Przykłady użycia:** Wielu twórców oprogramowania uważa Singleton za
antywzorzec, przez to jego wykorzystanie w kodzie Ruby maleje.

**Identyfikacja:** Singleton można rozpoznać po statycznej metodzie
kreacyjnej zwracającej jakiś obiekt którego instancja jest
przechowywana w pamięci podręcznej.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Implementacja naiwna](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Singleton z bezpieczeństwem
wątków](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Implementacja naiwna {#example-0-title}

Łatwo jest zaimplementować wzorzec Singleton niechlujnie --- wystarczy
ukryć konstruktor i zaimplementować statyczną metodę kreacyjną.

Ta sama klasa będzie działać nieprawidłowo w środowisku
wielowątkowym --- różne wątki mogą wywołać metodę kreacyjną w tym samym
momencie, otrzymując wiele instancji klasy Singleton.

#### []{#example-0--main-rb .anchor} **main.rb:** Przykład koncepcyjny

<figure class="code">
<pre class="code" lang="ruby"><code># The Singleton class defines the `instance` method that lets clients access the
# unique singleton instance.
class Singleton
  @instance = new

  private_class_method :new

  # The static method that controls the access to the singleton instance.
  #
  # This implementation let you subclass the Singleton class while keeping just
  # one instance of each subclass around.
  def self.instance
    @instance
  end

  # Finally, any singleton should define some business logic, which can be
  # executed on its instance.
  def some_business_logic
    # ...
  end
end

# The client code.

s1 = Singleton.instance
s2 = Singleton.instance

if s1.equal?(s2)
  print &#39;Singleton works, both variables contain the same instance.&#39;
else
  print &#39;Singleton failed, variables contain different instances.&#39;
end</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Singleton works, both variables contain the same instance.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Singleton z bezpieczeństwem wątków {#example-1-title}

Aby pozbyć się wyżej wymienionego problemu, trzeba zsynchronizować
wątki w momencie pierwszego tworzenia obiektu Singleton.

#### []{#example-1--main-rb .anchor} **main.rb:** Przykład koncepcyjny

<figure class="code">
<pre class="code" lang="ruby"><code># The Singleton class defines the `instance` method that lets clients access the
# unique singleton instance.
class Singleton
  attr_reader :value

  @instance_mutex = Mutex.new

  private_class_method :new

  def initialize(value)
    @value = value
  end

  # The static method that controls the access to the singleton instance.
  #
  # This implementation let you subclass the Singleton class while keeping just
  # one instance of each subclass around.
  def self.instance(value)
    return @instance if @instance

    @instance_mutex.synchronize do
      @instance ||= new(value)
    end

    @instance
  end

  # Finally, any singleton should define some business logic, which can be
  # executed on its instance.
  def some_business_logic
    # ...
  end
end

# @param [String] value
def test_singleton(value)
  singleton = Singleton.instance(value)
  puts singleton.value
end

# The client code.

puts &quot;If you see the same value, then singleton was reused (yay!)\n&quot;\
     &quot;If you see different values, then 2 singletons were created (booo!!)\n\n&quot;\
     &quot;RESULT:\n\n&quot;

process1 = Thread.new { test_singleton(&#39;FOO&#39;) }
process2 = Thread.new { test_singleton(&#39;BAR&#39;) }
process1.join
process2.join</code></pre>
</figure>

#### []{#example-1--output-txt .anchor} **output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>If you see the same value, then singleton was reused (yay!)
If you see different values, then 2 singletons were created (booo!!)

RESULT:

FOO
FOO</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Implementacja naiwna](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Singleton z bezpieczeństwem
wątków](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

## **Singleton** w innych językach

[![Singleton w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Singleton w języku C#"){.prog-lang-link}
[![Singleton w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Singleton w języku C++"){.prog-lang-link}
[![Singleton w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Singleton w języku Go"){.prog-lang-link}
[![Singleton w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Singleton w języku Java"){.prog-lang-link}
[![Singleton w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Singleton w języku PHP"){.prog-lang-link}
[![Singleton w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Singleton w języku Python"){.prog-lang-link}
[![Singleton w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Singleton w języku Rust"){.prog-lang-link}
[![Singleton w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Singleton w języku Swift"){.prog-lang-link}
[![Singleton w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Singleton w języku TypeScript"){.prog-lang-link}
:::::::::::::::::::::::

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
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
