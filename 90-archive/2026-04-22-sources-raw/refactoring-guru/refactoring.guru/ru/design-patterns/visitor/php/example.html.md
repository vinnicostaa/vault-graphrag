:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Посетитель](../../visitor.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Посетитель](../../../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x"}
:::

# **Посетитель** на PHP {#посетитель-на-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Посетитель** --- это поведенческий паттерн, который позволяет добавить
новую операцию для целой иерархии классов, не изменяя код этих классов.

> Подробней о том, почему Посетитель нельзя заменить простой перегрузкой
> методов читайте в статье [Посетитель и Double
> Dispatch](../../visitor-double-dispatch.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Посетитель ](../../visitor.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Концептуальный пример](#example-0)
:::

::: en-file
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Пример из реальной жизни](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Посетитель нечасто встречается в PHP-коде из-за своей
сложности и нюансов реализазации.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Концептуальный пример](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Пример из реальной
жизни](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Посетитель**, а именно ---
из каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

После ознакомления со структурой, вам будет легче воспринимать второй
пример, который рассматривает реальный случай использования паттерна в
мире PHP.

#### []{#example-0--index-php .anchor} **index.php:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Visitor\Conceptual;

/**
 * Интерфейс Компонента объявляет метод accept, который в качестве аргумента
 * может получать любой объект, реализующий интерфейс посетителя.
 */
interface Component
{
    public function accept(Visitor $visitor): void;
}

/**
 * Каждый Конкретный Компонент должен реализовать метод accept таким образом,
 * чтобы он вызывал метод посетителя, соответствующий классу компонента.
 */
class ConcreteComponentA implements Component
{
    /**
     * Обратите внимание, мы вызываем visitConcreteComponentA, что соответствует
     * названию текущего класса. Таким образом мы позволяем посетителю узнать, с
     * каким классом компонента он работает.
     */
    public function accept(Visitor $visitor): void
    {
        $visitor-&gt;visitConcreteComponentA($this);
    }

    /**
     * Конкретные Компоненты могут иметь особые методы, не объявленные в их
     * базовом классе или интерфейсе. Посетитель всё же может использовать эти
     * методы, поскольку он знает о конкретном классе компонента.
     */
    public function exclusiveMethodOfConcreteComponentA(): string
    {
        return &quot;A&quot;;
    }
}

class ConcreteComponentB implements Component
{
    /**
     * То же самое здесь: visitConcreteComponentB =&gt; ConcreteComponentB
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
 * Интерфейс Посетителя объявляет набор методов посещения, соответствующих
 * классам компонентов. Сигнатура метода посещения позволяет посетителю
 * определить конкретный класс компонента, с которым он имеет дело.
 */
interface Visitor
{
    public function visitConcreteComponentA(ConcreteComponentA $element): void;

    public function visitConcreteComponentB(ConcreteComponentB $element): void;
}

/**
 * Конкретные Посетители реализуют несколько версий одного и того же алгоритма,
 * которые могут работать со всеми классами конкретных компонентов.
 *
 * Максимальную выгоду от паттерна Посетитель вы почувствуете, используя его со
 * сложной структурой объектов, такой как дерево Компоновщика. В этом случае
 * было бы полезно хранить некоторое промежуточное состояние алгоритма при
 * выполнении методов посетителя над различными объектами структуры.
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
 * Клиентский код может выполнять операции посетителя над любым набором
 * элементов, не выясняя их конкретных классов. Операция принятия направляет
 * вызов к соответствующей операции в объекте посетителя.
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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

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
## Пример из реальной жизни {#example-1-title}

В этом примере паттерн **Посетитель** помогает внедрить функцию
отчётности в существующую иерархию классов:
`Компания > Отдел > Сотрудник`

После реализации Посетителя вы можете легко добавлять в приложение
другие подобные поведения без изменения существующих классов.

#### []{#example-1--index-php .anchor} **index.php:** Пример из реальной жизни

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Visitor\RealWorld;

/**
 * Интерфейс Компонента объявляет метод принятия объектов-посетителей.
 *
 * В этом методе Конкретный Компонент вызывает конкретный метод Посетителя, с
 * тем же типом параметра, что и у компонента.
 */
interface Entity
{
    public function accept(Visitor $visitor): string;
}

/**
 * Конкретный Компонент Компании.
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
        // Смотрите, Компонент Компании должен вызвать метод visitCompany. Тот
        // же принцип применяется ко всем компонентам.
        return $visitor-&gt;visitCompany($this);
    }
}

/**
 * Конкретный Компонент Отдела.
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
 * Конкретный Компонент Сотрудника.
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
 * Интерфейс Посетителя объявляет набор методов посещения для каждого класса
 * Конкретного Компонента.
 */
interface Visitor
{
    public function visitCompany(Company $company): string;

    public function visitDepartment(Department $department): string;

    public function visitEmployee(Employee $employee): string;
}

/**
 * Конкретный Посетитель должен предоставить реализации для каждого из классов
 * Конкретных Компонентов.
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
 * Клиентский код.
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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат выполнения

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
[Концептуальный пример](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Пример из реальной
жизни](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Читаем дальше

[Паттерны проектирования на Python []{.fa
.fa-arrow-right}](../../python.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Шаблонный метод на
PHP](../../template-method/php/example.html){.btn .btn-default
rel="prev"}
:::

## **Посетитель** на других языках программирования

[![Посетитель на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Посетитель на C#"){.prog-lang-link}
[![Посетитель на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Посетитель на C++"){.prog-lang-link}
[![Посетитель на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Посетитель на Go"){.prog-lang-link}
[![Посетитель на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Посетитель на Java"){.prog-lang-link}
[![Посетитель на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Посетитель на Python"){.prog-lang-link}
[![Посетитель на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Посетитель на Ruby"){.prog-lang-link}
[![Посетитель на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Посетитель на Rust"){.prog-lang-link}
[![Посетитель на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Посетитель на Swift"){.prog-lang-link}
[![Посетитель на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Посетитель на TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ru" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архив с примерами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Купи книгу **Погружение в Паттерны** и получи архив с десятками
детальных примеров, которые можно открывать прямо в IDE.

[ Узнать больше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
