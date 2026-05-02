::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} / [Template
Method](../../template-method.html){.type} /
[Rust](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Template
Method](../../../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x"}
:::

# **Template Method** em Rust {#template-method-em-rust .pattern-example-title-block-title}
::::

:::::::::: pattern-example-body
::: pattern-example-brief
O **Template Method** é um padrão de projeto comportamental que permite
definir o esqueleto de um algoritmo em uma classe base e permitir que as
subclasses substituam as etapas sem alterar a estrutura geral do
algoritmo.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Template Method ](../../template-method.html){.btn
.btn-lg .btn-primary}
:::

::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegação
:::

::: en-intro
 [Introdução](#)
:::

::: en-intro
 [Conceptual Example](#example-0)
:::

::: en-file
 [main](#example-0--main-rs)
:::
:::::::
::::::::::

[]{#example-0}

## Conceptual Example {#example-0-title}

#### []{#example-0--main-rs .anchor} **main.rs**

<figure class="code">
<pre class="code" lang="rust"><code>trait TemplateMethod {
    fn template_method(&amp;self) {
        self.base_operation1();
        self.required_operations1();
        self.base_operation2();
        self.hook1();
        self.required_operations2();
        self.base_operation3();
        self.hook2();
    }

    fn base_operation1(&amp;self) {
        println!(&quot;TemplateMethod says: I am doing the bulk of the work&quot;);
    }

    fn base_operation2(&amp;self) {
        println!(&quot;TemplateMethod says: But I let subclasses override some operations&quot;);
    }

    fn base_operation3(&amp;self) {
        println!(&quot;TemplateMethod says: But I am doing the bulk of the work anyway&quot;);
    }

    fn hook1(&amp;self) {}
    fn hook2(&amp;self) {}

    fn required_operations1(&amp;self);
    fn required_operations2(&amp;self);
}

struct ConcreteStruct1;

impl TemplateMethod for ConcreteStruct1 {
    fn required_operations1(&amp;self) {
        println!(&quot;ConcreteStruct1 says: Implemented Operation1&quot;)
    }

    fn required_operations2(&amp;self) {
        println!(&quot;ConcreteStruct1 says: Implemented Operation2&quot;)
    }
}

struct ConcreteStruct2;

impl TemplateMethod for ConcreteStruct2 {
    fn required_operations1(&amp;self) {
        println!(&quot;ConcreteStruct2 says: Implemented Operation1&quot;)
    }

    fn required_operations2(&amp;self) {
        println!(&quot;ConcreteStruct2 says: Implemented Operation2&quot;)
    }
}

fn client_code(concrete: impl TemplateMethod) {
    concrete.template_method()
}

fn main() {
    println!(&quot;Same client code can work with different concrete implementations:&quot;);
    client_code(ConcreteStruct1);
    println!();

    println!(&quot;Same client code can work with different concrete implementations:&quot;);
    client_code(ConcreteStruct2);
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>Same client code can work with different concrete implementations:
TemplateMethod says: I am doing the bulk of the work
ConcreteStruct1 says: Implemented Operation1
TemplateMethod says: But I let subclasses override some operations
ConcreteStruct1 says: Implemented Operation2
TemplateMethod says: But I am doing the bulk of the work anyway

Same client code can work with different concrete implementations:
TemplateMethod says: I am doing the bulk of the work
ConcreteStruct2 says: Implemented Operation1
TemplateMethod says: But I let subclasses override some operations
ConcreteStruct2 says: Implemented Operation2
TemplateMethod says: But I am doing the bulk of the work anyway</code></pre>
</figure>

::: next
#### Leia a seguir

[Visitor em Rust []{.fa
.fa-arrow-right}](../../visitor/rust/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Strategy em
Rust](../../strategy/rust/example.html){.btn .btn-default rel="prev"}
:::

## **Template Method** em outras linguagens

[![Template Method em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Template Method em C#"){.prog-lang-link}
[![Template Method em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Template Method em C++"){.prog-lang-link}
[![Template Method em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Template Method em Go"){.prog-lang-link}
[![Template Method em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Template Method em Java"){.prog-lang-link}
[![Template Method em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Template Method em PHP"){.prog-lang-link}
[![Template Method em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Template Method em Python"){.prog-lang-link}
[![Template Method em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Template Method em Ruby"){.prog-lang-link}
[![Template Method em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Template Method em Swift"){.prog-lang-link}
[![Template Method em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Template Method em TypeScript"){.prog-lang-link}
::::::::::::::::

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
::::::::::::::::::::::
:::::::::::::::::::::::
