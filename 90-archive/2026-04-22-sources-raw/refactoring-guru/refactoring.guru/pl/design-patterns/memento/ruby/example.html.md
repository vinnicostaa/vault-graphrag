:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Pamiątka](../../memento.html){.type} / [Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Pamiątka](../../../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x"}
:::

# **Pamiątka** w języku Ruby {#pamiątka-w-języku-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Pamiątka** to behawioralny wzorzec projektowy umożliwiający
zapisywanie "migawek" stanu obiektu i późniejsze jego przywracanie.

Wzorzec Pamiątka nie wpływa na wewnętrzną strukturę obiektu z którym
współpracuje, ani na dane przechowywane w migawkach.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Pamiątka ](../../memento.html){.btn .btn-lg
.btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Przykład koncepcyjny](#example-0)
:::

::: en-file
 [main](#example-0--main-rb)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::

**Złożoność:** []{.fa-stars}

**Popularność:** []{.fa-stars}

**Przykłady użycia:** Zasada działania Pamiątki opiera się na
serializacji, która jest dość powszechnie stosowana w Ruby. Nie jest to
jedyny, czy najefektywniejszy sposób zapisywania migawki stanu obiektu,
ale pozwala na przechowywanie kopii zapasowych, chroniąc jednocześnie
strukturę obiektu źródłowego przed innymi obiektami.
:::::::::::

[]{#example-0}

## Przykład koncepcyjny {#example-0-title}

Poniższy przykład ilustruje strukturę wzorca **Pamiątka** ze szczególnym
naciskiem na następujące kwestie:

-   Z jakich składa się klas?
-   Jakie role pełnią te klasy?
-   W jaki sposób elementy wzorca są ze sobą powiązane?

#### []{#example-0--main-rb .anchor} **main.rb:** Przykład koncepcyjny

<figure class="code">
<pre class="code" lang="ruby"><code># The Originator holds some important state that may change over time. It also
# defines a method for saving the state inside a memento and another method for
# restoring the state from it.
class Originator
  # For the sake of simplicity, the originator&#39;s state is stored inside a single
  # variable.
  attr_accessor :state
  private :state

  # @param [String] state
  def initialize(state)
    @state = state
    puts &quot;Originator: My initial state is: #{@state}&quot;
  end

  # The Originator&#39;s business logic may affect its internal state. Therefore,
  # the client should backup the state before launching methods of the business
  # logic via the save() method.
  def do_something
    puts &#39;Originator: I\&#39;m doing something important.&#39;
    @state = generate_random_string(30)
    puts &quot;Originator: and my state has changed to: #{@state}&quot;
  end

  private def generate_random_string(length = 10)
    ascii_letters = [*&#39;a&#39;..&#39;z&#39;, *&#39;A&#39;..&#39;Z&#39;]
    (0...length).map { ascii_letters.sample }.join
  end

  # Saves the current state inside a memento.
  def save
    ConcreteMemento.new(@state)
  end

  # Restores the Originator&#39;s state from a memento object.
  def restore(memento)
    @state = memento.state
    puts &quot;Originator: My state has changed to: #{@state}&quot;
  end
end

# The Memento interface provides a way to retrieve the memento&#39;s metadata, such
# as creation date or name. However, it doesn&#39;t expose the Originator&#39;s state.
class Memento
  # @abstract
  #
  # @return [String]
  def name
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end

  # @abstract
  #
  # @return [String]
  def date
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

class ConcreteMemento &lt; Memento
  # @param [String] state
  def initialize(state)
    @state = state
    @date = Time.now.strftime(&#39;%F %T&#39;)
  end

  # The Originator uses this method when restoring its state.
  attr_reader :state

  # The rest of the methods are used by the Caretaker to display metadata.
  def name
    &quot;#{@date} / (#{@state[0, 9]}...)&quot;
  end

  # @return [String]
  attr_reader :date
end

# The Caretaker doesn&#39;t depend on the Concrete Memento class. Therefore, it
# doesn&#39;t have access to the originator&#39;s state, stored inside the memento. It
# works with all mementos via the base Memento interface.
class Caretaker
  # @param [Originator] originator
  def initialize(originator)
    @mementos = []
    @originator = originator
  end

  def backup
    puts &quot;\nCaretaker: Saving Originator&#39;s state...&quot;
    @mementos &lt;&lt; @originator.save
  end

  def undo
    return if @mementos.empty?

    memento = @mementos.pop
    puts &quot;Caretaker: Restoring state to: #{memento.name}&quot;

    begin
      @originator.restore(memento)
    rescue StandardError
      undo
    end
  end

  def show_history
    puts &#39;Caretaker: Here\&#39;s the list of mementos:&#39;

    @mementos.each { |memento| puts memento.name }
  end
end

originator = Originator.new(&#39;Super-duper-super-puper-super.&#39;)
caretaker = Caretaker.new(originator)

caretaker.backup
originator.do_something

caretaker.backup
originator.do_something

caretaker.backup
originator.do_something

puts &quot;\n&quot;
caretaker.show_history

puts &quot;\nClient: Now, let&#39;s rollback!\n&quot;
caretaker.undo

puts &quot;\nClient: Once more!\n&quot;
caretaker.undo</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Originator: My initial state is: Super-duper-super-puper-super.

Caretaker: Saving Originator&#39;s state...
Originator: I&#39;m doing something important.
Originator: and my state has changed to: CHYzYSIWbqvWkCzIHOqTyEJWfQlFMn

Caretaker: Saving Originator&#39;s state...
Originator: I&#39;m doing something important.
Originator: and my state has changed to: vbkhwCeAEQBpLwQLlhmpcvUnwzxVnT

Caretaker: Saving Originator&#39;s state...
Originator: I&#39;m doing something important.
Originator: and my state has changed to: SBWlQnAEPLsitiOQAZbGlXHZAeWBoW

Caretaker: Here&#39;s the list of mementos:
2023-08-11 15:02:35 / (Super-dup...)
2023-08-11 15:02:35 / (CHYzYSIWb...)
2023-08-11 15:02:35 / (vbkhwCeAE...)

Client: Now, let&#39;s rollback!
Caretaker: Restoring state to: 2023-08-11 15:02:35 / (vbkhwCeAE...)
Originator: My state has changed to: vbkhwCeAEQBpLwQLlhmpcvUnwzxVnT

Client: Once more!
Caretaker: Restoring state to: 2023-08-11 15:02:35 / (CHYzYSIWb...)
Originator: My state has changed to: CHYzYSIWbqvWkCzIHOqTyEJWfQlFMn</code></pre>
</figure>

## **Pamiątka** w innych językach

[![Pamiątka w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Pamiątka w języku C#"){.prog-lang-link}
[![Pamiątka w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Pamiątka w języku C++"){.prog-lang-link}
[![Pamiątka w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Pamiątka w języku Go"){.prog-lang-link}
[![Pamiątka w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Pamiątka w języku Java"){.prog-lang-link}
[![Pamiątka w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Pamiątka w języku PHP"){.prog-lang-link}
[![Pamiątka w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Pamiątka w języku Python"){.prog-lang-link}
[![Pamiątka w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Pamiątka w języku Rust"){.prog-lang-link}
[![Pamiątka w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Pamiątka w języku Swift"){.prog-lang-link}
[![Pamiątka w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Pamiątka w języku TypeScript"){.prog-lang-link}
:::::::::::::::

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
:::::::::::::::::::::
::::::::::::::::::::::
