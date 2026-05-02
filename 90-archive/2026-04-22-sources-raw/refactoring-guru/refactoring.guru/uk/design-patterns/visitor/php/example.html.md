:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Відвідувач](../../visitor.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Відвідувач](../../../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x"}
:::

# **Відвідувач** на PHP {#відвідувач-на-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Відвідувач** --- це поведінковий патерн, який дозволяє додати нову
операцію для цілої ієрархії класів, не змінюючи код цих класів.

> Детальніше про те, чому Відвідувач не можна замінити звичайним
> перевантаженням методів читайте в статті [Відвідувач і Double
> Dispatch](../../visitor-double-dispatch.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Відвідувач ](../../visitor.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
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
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Життєвий приклад](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Складність:** []{.fa-stars}

**Популярність:** []{.fa-stars}

**Застосування:** Відвідувач нечасто зустрічається в PHP-коді внаслідок
своєї складності та особливостей реалізації.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Концептуальний приклад](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Життєвий приклад](#example-1){#example-1-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Концептуальний приклад {#example-0-title}

Цей приклад показує структуру патерна **Відвідувач**, а саме --- з яких
класів він складається, які ролі ці класи виконують і як вони
взаємодіють один з одним.

Після ознайомлення зі структурою, вам буде легше сприймати наступний
приклад, що розглядає реальний випадок використання патерна в світі PHP.

#### []{#example-0--index-php .anchor} **index.php:** Приклад структури патерна

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Visitor\Conceptual;

/**
 * The Component interface declares an `accept` method that should take the base
 * visitor interface as an argument.
 */
interface Component
{
    public function accept(Visitor $visitor): void;
}

/**
 * Each Concrete Component must implement the `accept` method in such a way that
 * it calls the visitor&#39;s method corresponding to the component&#39;s class.
 */
class ConcreteComponentA implements Component
{
    /**
     * Note that we&#39;re calling `visitConcreteComponentA`, which matches the
     * current class name. This way we let the visitor know the class of the
     * component it works with.
     */
    public function accept(Visitor $visitor): void
    {
        $visitor-&gt;visitConcreteComponentA($this);
    }

    /**
     * Concrete Components may have special methods that don&#39;t exist in their
     * base class or interface. The Visitor is still able to use these methods
     * since it&#39;s aware of the component&#39;s concrete class.
     */
    public function exclusiveMethodOfConcreteComponentA(): string
    {
        return &quot;A&quot;;
    }
}

class ConcreteComponentB implements Component
{
    /**
     * Same here: visitConcreteComponentB =&gt; ConcreteComponentB
     */
    public function accept(Visitor $visitor): void
    {
        $visitor-&gt;visitConcreteComponentB($this);
    }

    public function specialMethodOfConcreteComponentB(): string
    {
        return &quot;B&quot;;
    }
}

/**
 * The Visitor Interface declares a set of visiting methods that correspond to
 * component classes. The signature of a visiting method allows the visitor to
 * identify the exact class of the component that it&#39;s dealing with.
 */
interface Visitor
{
    public function visitConcreteComponentA(ConcreteComponentA $element): void;

    public function visitConcreteComponentB(ConcreteComponentB $element): void;
}

/**
 * Concrete Visitors implement several versions of the same algorithm, which can
 * work with all concrete component classes.
 *
 * You can experience the biggest benefit of the Visitor pattern when using it
 * with a complex object structure, such as a Composite tree. In this case, it
 * might be helpful to store some intermediate state of the algorithm while
 * executing visitor&#39;s methods over various objects of the structure.
 */
class ConcreteVisitor1 implements Visitor
{
    public function visitConcreteComponentA(ConcreteComponentA $element): void
    {
        echo $element-&gt;exclusiveMethodOfConcreteComponentA() . &quot; + ConcreteVisitor1\n&quot;;
    }

    public function visitConcreteComponentB(ConcreteComponentB $element): void
    {
        echo $element-&gt;specialMethodOfConcreteComponentB() . &quot; + ConcreteVisitor1\n&quot;;
    }
}

class ConcreteVisitor2 implements Visitor
{
    public function visitConcreteComponentA(ConcreteComponentA $element): void
    {
        echo $element-&gt;exclusiveMethodOfConcreteComponentA() . &quot; + ConcreteVisitor2\n&quot;;
    }

    public function visitConcreteComponentB(ConcreteComponentB $element): void
    {
        echo $element-&gt;specialMethodOfConcreteComponentB() . &quot; + ConcreteVisitor2\n&quot;;
    }
}

/**
 * The client code can run visitor operations over any set of elements without
 * figuring out their concrete classes. The accept operation directs a call to
 * the appropriate operation in the visitor object.
 */
function clientCode(array $components, Visitor $visitor)
{
    // ...
    foreach ($components as $component) {
        $component-&gt;accept($visitor);
    }
    // ...
}

$components = [
    new ConcreteComponentA(),
    new ConcreteComponentB(),
];

echo &quot;The client code works with all visitors via the base Visitor interface:\n&quot;;
$visitor1 = new ConcreteVisitor1();
clientCode($components, $visitor1);
echo &quot;\n&quot;;

echo &quot;It allows the same client code to work with different types of visitors:\n&quot;;
$visitor2 = new ConcreteVisitor2();
clientCode($components, $visitor2);</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>The client code works with all visitors via the base Visitor interface:
A + ConcreteVisitor1
B + ConcreteVisitor1

It allows the same client code to work with different types of visitors:
A + ConcreteVisitor2
B + ConcreteVisitor2</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Життєвий приклад {#example-1-title}

У цьому прикладі патерн **Відвідувач** допомагає впровадити функцію
звітності в існуючу ієрархію класів: `Компанія > Відділ > Співробітник`

Після реалізації Відвідувача ви можете легко додавати в програму інші
подібні поведінки не змінюючи існуючі класи.

#### []{#example-1--index-php .anchor} **index.php:** Приклад з реального світу

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Visitor\RealWorld;

/**
 * The Component interface declares a method of accepting visitor objects.
 *
 * In this method, a Concrete Component must call a specific Visitor&#39;s method
 * that has the same parameter type as that component.
 */
interface Entity
{
    public function accept(Visitor $visitor): string;
}

/**
 * The Company Concrete Component.
 */
class Company implements Entity
{
    private $name;

    /**
     * @var Department[]
     */
    private $departments;

    public function __construct(string $name, array $departments)
    {
        $this-&gt;name = $name;
        $this-&gt;departments = $departments;
    }

    public function getName(): string
    {
        return $this-&gt;name;
    }

    public function getDepartments(): array
    {
        return $this-&gt;departments;
    }

    // ...

    public function accept(Visitor $visitor): string
    {
        // See, the Company component must call the visitCompany method. The
        // same principle applies to all components.
        return $visitor-&gt;visitCompany($this);
    }
}

/**
 * The Department Concrete Component.
 */
class Department implements Entity
{
    private $name;

    /**
     * @var Employee[]
     */
    private $employees;

    public function __construct(string $name, array $employees)
    {
        $this-&gt;name = $name;
        $this-&gt;employees = $employees;
    }

    public function getName(): string
    {
        return $this-&gt;name;
    }

    public function getEmployees(): array
    {
        return $this-&gt;employees;
    }

    public function getCost(): int
    {
        $cost = 0;
        foreach ($this-&gt;employees as $employee) {
            $cost += $employee-&gt;getSalary();
        }

        return $cost;
    }

    // ...

    public function accept(Visitor $visitor): string
    {
        return $visitor-&gt;visitDepartment($this);
    }
}

/**
 * The Employee Concrete Component.
 */
class Employee implements Entity
{
    private $name;

    private $position;

    private $salary;

    public function __construct(string $name, string $position, int $salary)
    {
        $this-&gt;name = $name;
        $this-&gt;position = $position;
        $this-&gt;salary = $salary;
    }

    public function getName(): string
    {
        return $this-&gt;name;
    }

    public function getPosition(): string
    {
        return $this-&gt;position;
    }

    public function getSalary(): int
    {
        return $this-&gt;salary;
    }

    // ...

    public function accept(Visitor $visitor): string
    {
        return $visitor-&gt;visitEmployee($this);
    }
}

/**
 * The Visitor interface declares a set of visiting methods for each of the
 * Concrete Component classes.
 */
interface Visitor
{
    public function visitCompany(Company $company): string;

    public function visitDepartment(Department $department): string;

    public function visitEmployee(Employee $employee): string;
}

/**
 * The Concrete Visitor must provide implementations for every single class of
 * the Concrete Components.
 */
class SalaryReport implements Visitor
{
    public function visitCompany(Company $company): string
    {
        $output = &quot;&quot;;
        $total = 0;

        foreach ($company-&gt;getDepartments() as $department) {
            $total += $department-&gt;getCost();
            $output .= &quot;\n--&quot; . $this-&gt;visitDepartment($department);
        }

        $output = $company-&gt;getName() .
            &quot; (&quot; . money_format(&quot;%i&quot;, $total) . &quot;)\n&quot; . $output;

        return $output;
    }

    public function visitDepartment(Department $department): string
    {
        $output = &quot;&quot;;

        foreach ($department-&gt;getEmployees() as $employee) {
            $output .= &quot;   &quot; . $this-&gt;visitEmployee($employee);
        }

        $output = $department-&gt;getName() .
            &quot; (&quot; . money_format(&quot;%i&quot;, $department-&gt;getCost()) . &quot;)\n\n&quot; .
            $output;

        return $output;
    }

    public function visitEmployee(Employee $employee): string
    {
        return money_format(&quot;%#6n&quot;, $employee-&gt;getSalary()) .
            &quot; &quot; . $employee-&gt;getName() .
            &quot; (&quot; . $employee-&gt;getPosition() . &quot;)\n&quot;;
    }
}

/**
 * The client code.
 */

$mobileDev = new Department(&quot;Mobile Development&quot;, [
    new Employee(&quot;Albert Falmore&quot;, &quot;designer&quot;, 100000),
    new Employee(&quot;Ali Halabay&quot;, &quot;programmer&quot;, 100000),
    new Employee(&quot;Sarah Konor&quot;, &quot;programmer&quot;, 90000),
    new Employee(&quot;Monica Ronaldino&quot;, &quot;QA engineer&quot;, 31000),
    new Employee(&quot;James Smith&quot;, &quot;QA engineer&quot;, 30000),
]);
$techSupport = new Department(&quot;Tech Support&quot;, [
    new Employee(&quot;Larry Ulbrecht&quot;, &quot;supervisor&quot;, 70000),
    new Employee(&quot;Elton Pale&quot;, &quot;operator&quot;, 30000),
    new Employee(&quot;Rajeet Kumar&quot;, &quot;operator&quot;, 30000),
    new Employee(&quot;John Burnovsky&quot;, &quot;operator&quot;, 34000),
    new Employee(&quot;Sergey Korolev&quot;, &quot;operator&quot;, 35000),
]);
$company = new Company(&quot;SuperStarDevelopment&quot;, [$mobileDev, $techSupport]);

setlocale(LC_MONETARY, &#39;en_US&#39;);
$report = new SalaryReport();

echo &quot;Client: I can print a report for a whole company:\n\n&quot;;
echo $company-&gt;accept($report);

echo &quot;\nClient: ...or for different entities &quot; .
    &quot;such as an employee, a department, or the whole company:\n\n&quot;;
$someEmployee = new Employee(&quot;Some employee&quot;, &quot;operator&quot;, 35000);
$differentEntities = [$someEmployee, $techSupport, $company];
foreach ($differentEntities as $entity) {
    echo $entity-&gt;accept($report) . &quot;\r\n&quot;;
}

// $export = new JSONExport(); 
// echo $company-&gt;accept($export);</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Client: I can print a report for a whole company:

SuperStarDevelopment (USD550,000.00)

--Mobile Development (USD351,000.00)

    $100,000.00 Albert Falmore (designer)
    $100,000.00 Ali Halabay (programmer)
    $ 90,000.00 Sarah Konor (programmer)
    $ 31,000.00 Monica Ronaldino (QA engineer)
    $ 30,000.00 James Smith (QA engineer)

--Tech Support (USD199,000.00)

    $ 70,000.00 Larry Ulbrecht (supervisor)
    $ 30,000.00 Elton Pale (operator)
    $ 30,000.00 Rajeet Kumar (operator)
    $ 34,000.00 John Burnovsky (operator)
    $ 35,000.00 Sergey Korolev (operator)


Client: ...or for different entities such as an employee, a department, or the whole company:

35000 Some employee (operator)

Tech Support (199000)

   70000 Larry Ulbrecht (supervisor)
   30000 Elton Pale (operator)
   30000 Rajeet Kumar (operator)
   34000 John Burnovsky (operator)
   35000 Sergey Korolev (operator)

SuperStarDevelopment (550000)

--Mobile Development (351000)

   100000 Albert Falmore (designer)
   100000 Ali Halabay (programmer)
   90000 Sarah Konor (programmer)
   31000 Monica Ronaldino (QA engineer)
   30000 James Smith (QA engineer)

--Tech Support (199000)

   70000 Larry Ulbrecht (supervisor)
   30000 Elton Pale (operator)
   30000 Rajeet Kumar (operator)
   34000 John Burnovsky (operator)
   35000 Sergey Korolev (operator)</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Концептуальний приклад](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Життєвий
приклад](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Читаємо далі

[Патерни проектування на Python []{.fa
.fa-arrow-right}](../../python.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Шаблонний метод на
PHP](../../template-method/php/example.html){.btn .btn-default
rel="prev"}
:::

## **Відвідувач** іншими мовами програмування

[![Відвідувач на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Відвідувач на C#"){.prog-lang-link}
[![Відвідувач на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Відвідувач на C++"){.prog-lang-link}
[![Відвідувач на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Відвідувач на Go"){.prog-lang-link}
[![Відвідувач на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Відвідувач на Java"){.prog-lang-link}
[![Відвідувач на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Відвідувач на Python"){.prog-lang-link}
[![Відвідувач на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Відвідувач на Ruby"){.prog-lang-link}
[![Відвідувач на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Відвідувач на Rust"){.prog-lang-link}
[![Відвідувач на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Відвідувач на Swift"){.prog-lang-link}
[![Відвідувач на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Відвідувач на TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
