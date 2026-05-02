:::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::: {.main-content-container .center-content-container}
:::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
comportamentais](behavioral-patterns.html){.type} /
[Visitor](visitor.html){.type}
:::

# Visitor e Double Dispatch {#visitor-e-double-dispatch .title}

Vamos dar uma olhada na seguinte hierarquia de classe para formas
geométricas (cuidado com o pseudocódigo):

<figure class="code">
<pre class="code" lang="pseudocode"><code>interface Graphic is
    method draw()

class Shape implements Graphic is
    field id
    method draw()
    // ...

class Dot extends Shape is
    field x, y
    method draw()
    // ...

class Circle extends Dot is
    field radius
    method draw()
    // ...

class Rectangle extends Shape is
    field width, height
    method draw()
    // ...

class CompoundGraphic implements Graphic is
    field children: array of Graphic
    method draw()
    // ...</code></pre>
</figure>

O código funciona bem e a aplicação está em produção. Mas um dia você
decide criar uma funcionalidade de exportação. O código da exportação
seria alienígena se colocado nessas classes. Então ao invés de acionar a
exportação para todas as classes dessa hierarquia você decidiu criar uma
nova classe, externa à hierarquia e colocar toda a lógica de exportação
lá dentro. A classe obteria métodos para a exportação do estado público
de cada objeto em strings XML:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Exporter is
    method export(s: Shape) is
        print(&quot;Exportando forma&quot;)
    method export(d: Dot)
        print(&quot;Exportando Ponto&quot;)
    method export(c: Circle)
        print(&quot;Exportando círculo&quot;)
    method export(r: Rectangle)
        print(&quot;Exportando retângulo&quot;)
    method export(cs: CompoundGraphic)
        print(&quot;Exportando composto&quot;)</code></pre>
</figure>

O código parece bom, mas vamos testá-lo:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class App() is
    method export(shape: Shape) is
        Exporter exporter = new Exporter()
        exporter.export(shape);

app.export(new Circle());
// Infelizmente, isso irá imprimir &quot;Exportando forma&quot;.</code></pre>
</figure>

Espera aí! Por quê?!

## Pensando como um compilador

Nota: a seguinte informação é verdadeira para a maioria das linguagens
de programação modernas orientadas aos objetos (Java, C#, PHP, e
outras).

### Vinculação tardia/dinâmica

Finja que você é um compilador. Você tem que decidir como compilar o
seguinte código:

<figure class="code">
<pre class="code" lang="pseudocode"><code>method drawShape(shape: Shape) is
    shape.draw();</code></pre>
</figure>

Vejamos\... o método `draw` definido na classe `Shape`. Espera um
segundo, mas também há quatro subclasses que sobrescrevem esse método.
Podemos decidir com segurança qual das implementações chamar aqui?
Parece que não. A única maneira de saber com certeza é rodar o programa
e checar a classe de um objeto passado para o método. A única coisa que
sabemos de certeza é que o objeto **terá** a implementação do método
`draw`.

Então o código máquina resultante irá checar a classe pelo parâmetro `s`
e pegar a implementação `draw` da classe apropriada.

Esse tipo de checagem dinâmica é chamada de vinculação tardia (ou
dinâmica):

-   **Tardia**, porque nós ligamos o objeto e sua implementação após a
    compilação no tempo de execução.
-   **Dinâmica**, porque cada novo objeto pode precisar estar ligado à
    uma implementação diferente.

### Vinculação antecipada/estática

Agora, vamos "compilar" o seguinte código:

<figure class="code">
<pre class="code" lang="pseudocode"><code>method exportShape(shape: Shape) is
    Exporter exporter = new Exporter()
    exporter.export(shape);</code></pre>
</figure>

Tudo fica claro com a segunda linha: a classe `Exporter` não tem um
construtor, então nós apenas instanciamos um objeto. E a chamada para o
`export`? A `Exporter` tem cinco métodos com o mesmo nome que diferem
por tipos de parâmetro. Qual delas chamar? Parece que nós vamos precisar
de uma vinculação dinâmica aqui também.

Mas há outro problema. E se houver uma classe de "forma" que não tem o
método `export` apropriado na classe `Exporter`? Por exemplo, um objeto
`Ellipse`. O compilador não pode garantir que o método de
sobrecarregamento adequado exista em contraste com os métodos
sobrescritos. Uma situação ambígua aparece que o compilador não pode
permitir.

Portanto, desenvolvedores de compiladores usam um caminho seguro e usam
a vinculação antecipada (ou estática) para métodos sobrecarregados:

-   **Antecipada** porque acontece durante o tempo de compilação, antes
    do programa ser rodado.
-   **Estática** porque não pode ser alterada no tempo de execução.

Vamos voltar ao nosso exemplo. Nós temos certeza que o argumento que
está vindo será da hierarquia `Shape`: seja da classe `Shape` ou uma de
suas subclasses. Nós também sabemos que a classe `Exporter` tem uma
implementação básica da exportação que suporta a classe `Shape`:
`export(s: Shape)`.

Essa é a única implementação que pode ser ligada de forma segura a um
código sem tornar as coisas ambíguas. É por isso que mesmo que passamos
um objeto `Rectangle` em `exportShape`, o exportador ainda vai chamar um
método `export(s: Shape)`.

## Double Dispatch (Despacho Duplo)

O **double dispatch** é um truque que permite usar a vinculação dinâmica
junto com método sobrecarregados. Veja como é feito:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Visitor is
    method visit(s: Shape) is
        print(&quot;Forma visitada&quot;)
    method visit(d: Dot)
        print(&quot;Ponto visitado&quot;)

interface Graphic is
    method accept(v: Visitor)

class Shape implements Graphic is
    method accept(v: Visitor)
        // O compilador sabe com certeza que `this` é uma `Shape`.
        // O que significa que o `visit(s: Shape)` pode ser chamado com segurança.
        v.visit(this)

class Dot extends Shape is
    method accept(v: Visitor)
        // O compilador sabe que `this` é um `Dot`.
        // O que significa que o `visit(s: Dot)` pode ser chamado com segurança.
        v.visit(this)


Visitor v = new Visitor();
Graphic g = new Dot();

// O método `accept` foi sobrescrito, não sobrecarregado. O compilador
// vinculou ele dinamicamente. Portanto o `accept` será executado em uma
// classe que corresponda a um método de chamada de objeto (no nosso caso,
// a classe `Dot`).
g.accept(v);

// Saída: &quot;Ponto visitado&quot;</code></pre>
</figure>

### Posfácio

Mesmo que o padrão [Visitor](visitor.html) seja construído no princípio
do double dispatch (despacho duplo), esse não é seu propósito principal.
O Visitor permite que você adicione operações "externas" para toda uma
hierarquia de classe sem mudar o código existente dessas classes.

::: next
#### Leia a seguir

[Padrões de Projeto em C# []{.fa .fa-arrow-right}](csharp.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Visitor](visitor.html){.btn .btn-default
rel="prev"}
:::
::::::
:::::::
::::::::
