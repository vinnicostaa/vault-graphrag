:::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::: {.main-content-container .center-content-container}
:::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
criacionais](creational-patterns.html){.type}
:::

# Comparação Factory {#comparação-factory .title}

Este artigo irá mostrar as diferenças entre:

1.  Factory (Fábrica)
2.  Creation method (Método de criação)
3.  Static creation method (Método de criação estática)
4.  Simple factory (Fábrica simples)
5.  Padrão [Factory Method](factory-method.html)
6.  Padrão [Abstract Factory](abstract-factory.html)

Você pode encontrar referências a todos esses termos na web. Embora
pareçam similares, eles todos têm significados diferentes. Muitas
pessoas não se dão conta disso, o que leva a confusão e equívocos.

Então vamos tentar entender a diferença e resolver esse problema de uma
vez por todas.

## 1. Factory *(Fábrica)*

**Fábrica** é um termo ambíguo que se refere a uma função, método ou
classe que deve produzir alguma coisa. Mais comumente, fábricas produzem
objetos. Mas elas também podem produzir arquivos, registros em bases de
dados, etc.

Por exemplo, qualquer uma destas coisas pode ser casualmente
referenciada como uma "fábrica":

-   uma função ou método que cria a interface de um programa;
-   uma classe que cria usuários;
-   um método estático que chama uma classe construtora de uma certa
    maneira;
-   um dos padrões de projeto criacionais.

Geralmente, quando alguém diz uma palavra "fábrica", o significado exato
deve ficar claro a partir de um contexto. Mas se estiver em dúvida,
apenas pergunte. Há uma chance que o autor seja ignorante.

## 2. Creation method *(Método de criação)*

O **método de criação** definido no livro [Refatoração para
Padrões](https://www.amazon.com/-/dp/B019HIO22C) como "um método que
cria objetos". Isso significa que qualquer resultado de um padrão
Factory Method é um "método de criação" mas não necessariamente o
inverso. Também quer dizer que você pode substituir o termo "método de
criação" sempre que Martin Fowler usar o termo "método fábrica" em
[Refactoring](https://www.amazon.com/-/dp/0201485672) e sempre que
Joshua Bloch usar o termo "método fábrica estático" em [Effective
Java](https://www.amazon.com/-/dp/0134685997).

Na verdade, o *método de criação* é apenas um wrapper ao redor de uma
chamada de construtor. Ele apenas tem um nome que melhor expressa suas
intenções. Por outro lado, ele pode ajudar a isolar seu código de
mudanças do construtor. Ele pode até conter certa lógica que retornaria
objetos existentes ao invés de criar novos objetos.

Muitas pessoas chamariam tais métodos "métodos fábrica" apenas porque
ele produz novos objetos: A lógica é simples: o método cria objetos e já
que todas as *fábricas* criam objetos, esse método deve claramente ser
uma *método fábrica*. Naturalmente, há muita confusão quando se trata do
verdadeiro padrão [Factory Method](factory-method.html).

No seguinte exemplo, `next` é um método de criação:

<figure class="code">
<pre class="code" lang="php"><code>class Number {
    private $value;

    public function __construct($value) {
        $this-&gt;value = $value;
    }

    public function next() {
        return new Number ($this-&gt;value + 1);
    }
}</code></pre>
</figure>

## 3. Static creation method *(Método de criação estática)*

O **método de criação estático** é um método de criação declarado como
`static`. Em outras palavras, ele pode ser chamado em uma classe e não
precisa que um objeto seja criado.

Não se confunda quando alguém chama métodos como esse de "método fábrica
estático". Isso é um mau hábito. O [Factory Method](factory-method.html)
é um padrão de projeto que se baseia em herança. Se você torná-lo
estático, você não pode mais estendê-lo em subclasses, que é contrário
ao propósito do padrão.

Quando um método de criação estático retorna novos objetos ele se torna
um construtor alternativo.

Ele pode ser útil quando:

-   Você precisa ter diferentes construtores para diferentes propósitos
    mas as assinaturas deles coincidem. Por exemplo, tendo ambos
    `Random(int max)` e `Random(int min)` é possível em Java C++, C#, e
    muitas outras linguagens. E o modo mais popular de fazer isso é
    criar vários métodos estáticos que chamam o construtor padrão e
    configuram valores apropriados depois.

-   Você quer reutilizar objetos existentes, ao invés de instanciar
    novos objetos (veja o padrão [Singleton](singleton.html)). Os
    construtores na maioria das linguagens de programação precisam
    retornar novas instâncias de classe. O método de criação estático é
    uma alternativa a essa limitação. Dentro de um método estático, seu
    código pode decidir se quer criar uma instância nova chamando o
    construtor ou retornar um objeto existente de algum cache.

No seguinte exemplo, o método `load` é um método criação estático. Ele
fornece uma maneira conveniente de buscar usuários de uma base de dados.

<figure class="code">
<pre class="code" lang="php"><code>class User {
    private $id, $name, $email, $phone;

    public function __construct($id, $name, $email, $phone) {
        $this-&gt;id = $id;
        $this-&gt;name = $name;
        $this-&gt;email = $email;
        $this-&gt;phone = $phone;
    }

    public static function load($id) {
        list($id, $name, $email, $phone) = DB::load_data(&#39;users&#39;, &#39;id&#39;, &#39;name&#39;, &#39;email&#39;, &#39;phone&#39;);
        $user = new User($id, $name, $email, $phone);
        return $user;
    }
}</code></pre>
</figure>

## 4. Simple factory *(Fábrica simples)*

O padrão **fábrica simples** []{.pop-i}[Definido no livro [Use a
Cabeça!: Padrões de
Projetos](https://www.amazon.com.br/-/dp/8576081741/).]{.pop-content}
descreve uma classe que tem um método de criação com uma condicional
grande que, baseada nos parâmetros do método, escolhe qual classe
produto instanciar e então retornar.

As pessoas geralmente confundem *fábricas simples* com *fábricas* gerais
ou com um dos padrões de projeto criacional. Na maioria dos casos, uma
fábrica simples é uma etapa intermediária na introdução dos padrões
[Factory Method](factory-method.html) ou [Abstract
Factory](abstract-factory.html).

Fábricas simples geralmente não têm subclasses. Mas após extrair
subclasses de uma fábrica simples, ela começa a se parecer como um
padrão *Factory Method* clássico.

A propósito, se você declarar uma fábrica simples como `abstract`, ela
não se torna um padrão *Abstract Factory* magicamente.

Aqui temos um exemplo da *fábricas simples*:

<figure class="code">
<pre class="code" lang="php"><code>class UserFactory {
    public static function create($type) {
        switch ($type) {
            case &#39;user&#39;: return new User();
            case &#39;customer&#39;: return new Customer();
            case &#39;admin&#39;: return new Admin();
            default:
                throw new Exception(&#39;Wrong user type passed.&#39;);
        }
    }
}</code></pre>
</figure>

## 5. Padrão *Factory Method*

O **Factory Method** []{.pop-i}[Definido no livro GoF [Padrões de
Projeto --- Soluções Reutilizáveis de Software Orientado a
Objetos](https://www.amazon.com/gp/product/8573076100/).]{.pop-content}
é um padrão de projeto criacional que fornece uma interface para criação
de objetos mas permite que subclasses alterem o tipo de um objeto que
será criado.

Se você tem um método de criação na classe base e subclasses que
estendem ele, você pode estar olhando para o Factory Method.

<figure class="code">
<pre class="code" lang="php"><code>abstract class Department {
    public abstract function createEmployee($id);

    public function fire($id) {
        $employee = $this-&gt;createEmployee($id);
        $employee-&gt;paySalary();
        $employee-&gt;dismiss();
    }
}

class ITDepartment extends Department {
    public function createEmployee($id) {
        return new Programmer($id);
    }
}

class AccountingDepartment extends Department {
    public function createEmployee($id) {
        return new Accountant($id);
    }
}</code></pre>
</figure>

## 6. Padrão *Abstract Factory*

O **Abstract Factory** []{.pop-i}[Também definido no [livro
GoF](https://www.amazon.com/gp/product/8573076100/).]{.pop-content} é um
padrão de projeto criacional que permite produzir famílias de objetos
relacionados ou dependentes sem especificar suas classes concretas.

Quais são as \"famílias de objetos\"? Por exemplo, tome este conjunto de
classes: `Transporte` + `Motor` + `Controles`. Pode haver muitas
variantes deles:

1.  `Carro` + `MotorCombustão` + `Volante`
2.  `Avião` + `Turbina` + `Jugo`

Se o seu programa não opera com famílias de produtos, então você não
precisa de um Abstract Factory.

E novamente, muitas pessoas confundem o padrão *Abstract Factory* com a
classe fábrica simples declarada como `abstract`. Não faça isso!

### Posfácio

Agora que você sabe a diferença, vá dar uma olhada nos padrões de
projeto:

-   [Factory Method](factory-method.html)
-   [Abstract Factory](abstract-factory.html)

::: next
#### Leia a seguir

[Builder []{.fa .fa-arrow-right}](builder.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Abstract Factory](abstract-factory.html){.btn
.btn-default rel="prev"}
:::
::::::
:::::::
::::::::
