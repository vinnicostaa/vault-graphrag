:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Memento](../../memento.html){.type} / [Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Memento](../../../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x"}
:::

# **Memento** en Ruby {#memento-en-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Memento** es un patrón de diseño de comportamiento que permite tomar
instantáneas del estado de un objeto y restaurarlo en el futuro.

El patrón Memento no compromete la estructura interna del objeto con el
que trabaja, ni la información que se encuentra dentro de las
instantáneas.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Memento ](../../memento.html){.btn .btn-lg
.btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegación
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Ejemplo conceptual](#example-0)
:::

::: en-file
 [main](#example-0--main-rb)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::

**Complejidad:** []{.fa-stars}

**Popularidad:** []{.fa-stars}

**Ejemplos de uso:** El principio del patrón Memento puede cumplirse
utilizando la serialización, que es bastante habitual en Ruby. Aunque no
es la única forma ni la más efectiva de realizar instantáneas del estado
de un objeto, permite almacenar copias de seguridad del estado mientras
protege de otros objetos la estructura del originador.
:::::::::::

[]{#example-0}

## Ejemplo conceptual {#example-0-title}

Este ejemplo ilustra la estructura del patrón de diseño **Memento**. Se
centra en responder las siguientes preguntas:

-   ¿De qué clases se compone?
-   ¿Qué papeles juegan esas clases?
-   ¿De qué forma se relacionan los elementos del patrón?

#### []{#example-0--main-rb .anchor} **main.rb:** Ejemplo conceptual

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

#### []{#example-0--output-txt .anchor} **output.txt:** Resultado de la ejecución

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

## **Memento** en otros lenguajes

[![Memento en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Memento en C#"){.prog-lang-link}
[![Memento en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Memento en C++"){.prog-lang-link}
[![Memento en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Memento en Go"){.prog-lang-link}
[![Memento en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Memento en Java"){.prog-lang-link}
[![Memento en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Memento en PHP"){.prog-lang-link}
[![Memento en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Memento en Python"){.prog-lang-link}
[![Memento en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Memento en Rust"){.prog-lang-link}
[![Memento en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Memento en Swift"){.prog-lang-link}
[![Memento en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Memento en TypeScript"){.prog-lang-link}
:::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-es" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archivo con ejemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compra el libro electrónico **Sumérgete en los patrones de diseño** para
tener acceso al archivo con decenas de ejemplos detallados que puedes
abrir en tu IDE.

[ Saber más...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::
::::::::::::::::::::::
