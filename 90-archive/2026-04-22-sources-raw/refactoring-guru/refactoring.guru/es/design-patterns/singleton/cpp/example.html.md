:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Singleton](../../singleton.html){.type} / [C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Singleton](../../../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x"}
:::

# **Singleton** en C++ {#singleton-en-c .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Singleton** es un patrón de diseño creacional que garantiza que tan
solo exista un objeto de su tipo y proporciona un único punto de acceso
a él para cualquier otro código.

El patrón tiene prácticamente los mismos pros y contras que las
variables globales. Aunque son muy útiles, rompen la modularidad de tu
código.

No se puede utilizar una clase que dependa del Singleton en otro
contexto. Tendrás que llevar también la clase Singleton. La mayoría de
las veces, esta limitación aparece durante la creación de pruebas de
unidad.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Singleton ](../../singleton.html){.btn
.btn-lg .btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegación
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Singleton ingenuo](#example-0)
:::

::: en-file
 [main](#example-0--main-cc)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Singleton con seguridad en los hilos](#example-1)
:::

::: en-file
 [main](#example-1--main-cc)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Complejidad:** []{.fa-stars}

**Popularidad:** []{.fa-stars}

**Ejemplos de uso:** Muchos desarrolladores consideran el patrón
Singleton un antipatrón. Por este motivo, su uso está en declive en el
código C++.

**Identificación:** El patrón Singleton se puede reconocer por un método
de creación estático, que devuelve el mismo objeto guardado en caché.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Singleton ingenuo](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Singleton con seguridad en los
hilos](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Singleton ingenuo {#example-0-title}

Es muy fácil implementar un Singleton descuidado. Tan solo necesitas
esconder el constructor e implementar un método de creación estático.

La misma clase se comporta de forma incorrecta en un entorno multihilo.
Los múltiples hilos pueden llamar al método de creación de forma
simultánea y obtener varias instancias de la clase Singleton.

#### []{#example-0--main-cc .anchor} **main.cc:** Ejemplo conceptual

<figure class="code">
<pre class="code" lang="cpp"><code>/**
 * The Singleton class defines the `GetInstance` method that serves as an
 * alternative to constructor and lets clients access the same instance of this
 * class over and over.
 */
class Singleton
{

    /**
     * The Singleton&#39;s constructor should always be private to prevent direct
     * construction calls with the `new` operator.
     */

protected:
    Singleton(const std::string value): value_(value)
    {
    }

    static Singleton* singleton_;

    std::string value_;

public:

    /**
     * Singletons should not be cloneable.
     */
    Singleton(Singleton &amp;other) = delete;
    /**
     * Singletons should not be assignable.
     */
    void operator=(const Singleton &amp;) = delete;
    /**
     * This is the static method that controls the access to the singleton
     * instance. On the first run, it creates a singleton object and places it
     * into the static field. On subsequent runs, it returns the client existing
     * object stored in the static field.
     */

    static Singleton *GetInstance(const std::string&amp; value);
    /**
     * Finally, any singleton should define some business logic, which can be
     * executed on its instance.
     */
    void SomeBusinessLogic()
    {
        // ...
    }

    std::string value() const{
        return value_;
    } 
};

Singleton* Singleton::singleton_= nullptr;;

/**
 * Static methods should be defined outside the class.
 */
Singleton *Singleton::GetInstance(const std::string&amp; value)
{
    /**
     * This is a safer way to create an instance. instance = new Singleton is
     * dangeruous in case two instance threads wants to access at the same time
     */
    if(singleton_==nullptr){
        singleton_ = new Singleton(value);
    }
    return singleton_;
}

void ThreadFoo(){
    // Following code emulates slow initialization.
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    Singleton* singleton = Singleton::GetInstance(&quot;FOO&quot;);
    std::cout &lt;&lt; singleton-&gt;value() &lt;&lt; &quot;\n&quot;;
}

void ThreadBar(){
    // Following code emulates slow initialization.
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    Singleton* singleton = Singleton::GetInstance(&quot;BAR&quot;);
    std::cout &lt;&lt; singleton-&gt;value() &lt;&lt; &quot;\n&quot;;
}


int main()
{
    std::cout &lt;&lt;&quot;If you see the same value, then singleton was reused (yay!\n&quot; &lt;&lt;
                &quot;If you see different values, then 2 singletons were created (booo!!)\n\n&quot; &lt;&lt;
                &quot;RESULT:\n&quot;;   
    std::thread t1(ThreadFoo);
    std::thread t2(ThreadBar);
    t1.join();
    t2.join();

    return 0;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>If you see the same value, then singleton was reused (yay!
If you see different values, then 2 singletons were created (booo!!)

RESULT:
BAR
FOO</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Singleton con seguridad en los hilos {#example-1-title}

Para arreglar el problema, debes sincronizar hilos durante la primera
creación del objeto Singleton.

#### []{#example-1--main-cc .anchor} **main.cc:** Ejemplo conceptual

<figure class="code">
<pre class="code" lang="cpp"><code>/**
 * The Singleton class defines the `GetInstance` method that serves as an
 * alternative to constructor and lets clients access the same instance of this
 * class over and over.
 */
class Singleton
{

    /**
     * The Singleton&#39;s constructor/destructor should always be private to
     * prevent direct construction/desctruction calls with the `new`/`delete`
     * operator.
     */
private:
    static Singleton * pinstance_;
    static std::mutex mutex_;

protected:
    Singleton(const std::string value): value_(value)
    {
    }
    ~Singleton() {}
    std::string value_;

public:
    /**
     * Singletons should not be cloneable.
     */
    Singleton(Singleton &amp;other) = delete;
    /**
     * Singletons should not be assignable.
     */
    void operator=(const Singleton &amp;) = delete;
    /**
     * This is the static method that controls the access to the singleton
     * instance. On the first run, it creates a singleton object and places it
     * into the static field. On subsequent runs, it returns the client existing
     * object stored in the static field.
     */

    static Singleton *GetInstance(const std::string&amp; value);
    /**
     * Finally, any singleton should define some business logic, which can be
     * executed on its instance.
     */
    void SomeBusinessLogic()
    {
        // ...
    }
    
    std::string value() const{
        return value_;
    } 
};

/**
 * Static methods should be defined outside the class.
 */

Singleton* Singleton::pinstance_{nullptr};
std::mutex Singleton::mutex_;

/**
 * The first time we call GetInstance we will lock the storage location
 *      and then we make sure again that the variable is null and then we
 *      set the value. RU:
 */
Singleton *Singleton::GetInstance(const std::string&amp; value)
{
    std::lock_guard&lt;std::mutex&gt; lock(mutex_);
    if (pinstance_ == nullptr)
    {
        pinstance_ = new Singleton(value);
    }
    return pinstance_;
}

void ThreadFoo(){
    // Following code emulates slow initialization.
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    Singleton* singleton = Singleton::GetInstance(&quot;FOO&quot;);
    std::cout &lt;&lt; singleton-&gt;value() &lt;&lt; &quot;\n&quot;;
}

void ThreadBar(){
    // Following code emulates slow initialization.
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    Singleton* singleton = Singleton::GetInstance(&quot;BAR&quot;);
    std::cout &lt;&lt; singleton-&gt;value() &lt;&lt; &quot;\n&quot;;
}

int main()
{   
    std::cout &lt;&lt;&quot;If you see the same value, then singleton was reused (yay!\n&quot; &lt;&lt;
                &quot;If you see different values, then 2 singletons were created (booo!!)\n\n&quot; &lt;&lt;
                &quot;RESULT:\n&quot;;   
    std::thread t1(ThreadFoo);
    std::thread t2(ThreadBar);
    t1.join();
    t2.join();
    
    return 0;
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>If you see the same value, then singleton was reused (yay!
If you see different values, then 2 singletons were created (booo!!)

RESULT:
FOO
FOO</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Singleton ingenuo](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Singleton con seguridad en los
hilos](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Leer siguiente

[Adapter en C++ []{.fa
.fa-arrow-right}](../../adapter/cpp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Prototype en
C++](../../prototype/cpp/example.html){.btn .btn-default rel="prev"}
:::

## **Singleton** en otros lenguajes

[![Singleton en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Singleton en C#"){.prog-lang-link}
[![Singleton en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Singleton en Go"){.prog-lang-link}
[![Singleton en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Singleton en Java"){.prog-lang-link}
[![Singleton en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Singleton en PHP"){.prog-lang-link}
[![Singleton en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Singleton en Python"){.prog-lang-link}
[![Singleton en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Singleton en Ruby"){.prog-lang-link}
[![Singleton en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Singleton en Rust"){.prog-lang-link}
[![Singleton en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Singleton en Swift"){.prog-lang-link}
[![Singleton en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Singleton en TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
