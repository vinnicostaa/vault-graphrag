::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::: {.page .list .text}
# Прийоми рефакторингу {#прийоми-рефакторингу .title}

:::::::: {.section-bordered .link-list style="border-top: none;"}
::::::: cell-flex
::: cell-image
![](../../images/refactoring/content/catalog/composing-methods1873.png?id=953854e802753495812cb9b2686765f7){width="160"
height="280"
srcset="/images/refactoring/content/catalog/composing-methods-2x.png?id=75620eee35ec53ff9e21955cba2c70c9 2x,/images/refactoring/content/catalog/composing-methods-3x.png 3x"}
:::

::::: cell-text
### [Складання методів](techniques/composing-methods.html) {#composing-methods}

::: annotation
Значна частина рефакторинга присвячується правильному складанню методів.
У більшості випадків, коренем усіх зол є занадто довгі методи.
Хитросплетіння коду всередині такого методу приховують логіку виконання
і роблять метод вкрай складним для розуміння, а значить і для змін.
Рефакторинги цієї групи покликані зменшити складність всередині методу,
прибрати дублювання коду і полегшити подальшу роботу з ним.
:::

::: catalog-list
-   [Extract Method](../extract-method.html)
-   [Inline Method](../inline-method.html)
-   [Extract Variable](../extract-variable.html)
-   [Inline Temp](../inline-temp.html)

<!-- -->

-   [Replace Temp with Query](../replace-temp-with-query.html)
-   [Split Temporary Variable](../split-temporary-variable.html)
-   [Remove Assignments to
    Parameters](../remove-assignments-to-parameters.html)

<!-- -->

-   [Replace Method with Method
    Object](../replace-method-with-method-object.html)
-   [Substitute Algorithm](../substitute-algorithm.html)
:::
:::::
:::::::
::::::::

:::::::: {.section-bordered .link-list style="border-top: none;"}
::::::: cell-flex
::: cell-image
![](../../images/refactoring/content/catalog/moving-features-between-objectsc45b.png?id=8ba49e26381112792e32172edf220524){width="160"
height="280"
srcset="/images/refactoring/content/catalog/moving-features-between-objects-2x.png?id=5503a838a78b344754e92cd116207d96 2x,/images/refactoring/content/catalog/moving-features-between-objects-3x.png 3x"}
:::

::::: cell-text
### [Переміщення функцій між об\'єктами](techniques/moving-features-between-objects.html) {#moving-features-between-objects}

::: annotation
Якщо ви розмістили функціональність по класах не найвдалішим чином - це
ще не привід впадати у відчай.

Рефакторинги цієї групи показують як безпечно переміщати
функціональність з одних класів в інші, створювати нові класи, а також
приховувати деталі реалізації з публічного доступу.
:::

::: catalog-list
-   [Move Method](../move-method.html)
-   [Move Field](../move-field.html)
-   [Extract Class](../extract-class.html)
-   [Inline Class](../inline-class.html)

<!-- -->

-   [Hide Delegate](../hide-delegate.html)
-   [Remove Middle Man](../remove-middle-man.html)

<!-- -->

-   [Introduce Foreign Method](../introduce-foreign-method.html)
-   [Introduce Local Extension](../introduce-local-extension.html)
:::
:::::
:::::::
::::::::

:::::::: {.section-bordered .link-list style="border-top: none;"}
::::::: cell-flex
::: cell-image
![](../../images/refactoring/content/catalog/organizing-datac280.png?id=0be19b5980545dccb976d377ec731d30){width="160"
height="280"
srcset="/images/refactoring/content/catalog/organizing-data-2x.png?id=881db3197ef8ea87bd55320073462caa 2x,/images/refactoring/content/catalog/organizing-data-3x.png 3x"}
:::

::::: cell-text
### [Організація даних](techniques/organizing-data.html) {#organizing-data}

::: annotation
Рефакторинги цієї групи покликані полегшити роботу з даними, замінивши
роботу з примітивними типами багатими функціональністю класами.

Крім того, важливим моментом є зменшення зв\'язаність між класами, що
покращує переносимість класів і шанси їх повторного використання.
:::

::: catalog-list
-   [Change Value to Reference](../change-value-to-reference.html)
-   [Change Reference to Value](../change-reference-to-value.html)
-   [Duplicate Observed Data](../duplicate-observed-data.html)
-   [Self Encapsulate Field](../self-encapsulate-field.html)
-   [Replace Data Value with
    Object](../replace-data-value-with-object.html)
-   [Replace Array with Object](../replace-array-with-object.html)

<!-- -->

-   [Change Unidirectional Association to
    Bidirectional](../change-unidirectional-association-to-bidirectional.html)
-   [Change Bidirectional Association to
    Unidirectional](../change-bidirectional-association-to-unidirectional.html)
-   [Encapsulate Field](../encapsulate-field.html)
-   [Encapsulate Collection](../encapsulate-collection.html)
-   [Replace Magic Number with Symbolic
    Constant](../replace-magic-number-with-symbolic-constant.html)

<!-- -->

-   [Replace Type Code with Class](../replace-type-code-with-class.html)
-   [Replace Type Code with
    Subclasses](../replace-type-code-with-subclasses.html)
-   [Replace Type Code with
    State/Strategy](../replace-type-code-with-state-strategy.html)
-   [Replace Subclass with Fields](../replace-subclass-with-fields.html)
:::
:::::
:::::::
::::::::

:::::::: {.section-bordered .link-list style="border-top: none;"}
::::::: cell-flex
::: cell-image
![](../../images/refactoring/content/catalog/simplifying-conditional-expressions908f.png?id=a551572d527946cd03b647098b67776d){width="160"
height="280"
srcset="/images/refactoring/content/catalog/simplifying-conditional-expressions-2x.png?id=a6ffc35feb77eac6108ee31655ae92d1 2x,/images/refactoring/content/catalog/simplifying-conditional-expressions-3x.png 3x"}
:::

::::: cell-text
### [Спрощення умовних виразів](techniques/simplifying-conditional-expressions.html) {#simplifying-conditional-expressions}

::: annotation
Логіка умовного виконання має тенденцію ставати складною, тому ряд
рефакторингів спрямований на те, щоб спростити її.
:::

::: catalog-list
-   [Consolidate Conditional
    Expression](../consolidate-conditional-expression.html)
-   [Consolidate Duplicate Conditional
    Fragments](../consolidate-duplicate-conditional-fragments.html)
-   [Decompose Conditional](../decompose-conditional.html)

<!-- -->

-   [Replace Conditional with
    Polymorphism](../replace-conditional-with-polymorphism.html)
-   [Remove Control Flag](../remove-control-flag.html)
-   [Replace Nested Conditional with Guard
    Clauses](../replace-nested-conditional-with-guard-clauses.html)

<!-- -->

-   [Introduce Null Object](../introduce-null-object.html)
-   [Introduce Assertion](../introduce-assertion.html)
:::
:::::
:::::::
::::::::

:::::::: {.section-bordered .link-list style="border-top: none;"}
::::::: cell-flex
::: cell-image
![](../../images/refactoring/content/catalog/simplifying-method-calls16bd.png?id=0af0ac74a5d0d7f8ac33a58b4a479ee6){width="160"
height="280"
srcset="/images/refactoring/content/catalog/simplifying-method-calls-2x.png?id=075a4082f9a85d01391184efa3c8f1a1 2x,/images/refactoring/content/catalog/simplifying-method-calls-3x.png 3x"}
:::

::::: cell-text
### [Спрощення викликів методів](techniques/simplifying-method-calls.html) {#simplifying-method-calls}

::: annotation
Ці рефакторинги роблять виклики методів простіше і ясніше для розуміння.
Це, в свою чергу, спрощує інтерфейси взаємодії між класами.
:::

::: catalog-list
-   [Add Parameter](../add-parameter.html)
-   [Remove Parameter](../remove-parameter.html)
-   [Rename Method](../rename-method.html)
-   [Separate Query from Modifier](../separate-query-from-modifier.html)
-   [Parameterize Method](../parameterize-method.html)

<!-- -->

-   [Introduce Parameter Object](../introduce-parameter-object.html)
-   [Preserve Whole Object](../preserve-whole-object.html)
-   [Remove Setting Method](../remove-setting-method.html)
-   [Replace Parameter with Explicit
    Methods](../replace-parameter-with-explicit-methods.html)
-   [Replace Parameter with Method
    Call](../replace-parameter-with-method-call.html)

<!-- -->

-   [Hide Method](../hide-method.html)
-   [Replace Constructor with Factory
    Method](../replace-constructor-with-factory-method.html)
-   [Replace Error Code with
    Exception](../replace-error-code-with-exception.html)
-   [Replace Exception with Test](../replace-exception-with-test.html)
:::
:::::
:::::::
::::::::

:::::::: {.section-bordered .link-list style="border-top: none;"}
::::::: cell-flex
::: cell-image
![](../../images/refactoring/content/catalog/dealing-with-generalizatione7ca.png?id=56357b115153175b2eb40563d936087c){width="160"
height="280"
srcset="/images/refactoring/content/catalog/dealing-with-generalization-2x.png?id=5187a4b8d010877a25761926c2f24a3c 2x,/images/refactoring/content/catalog/dealing-with-generalization-3x.png 3x"}
:::

::::: cell-text
### [Задачі узагальнення об\'єктів](techniques/dealing-with-generalization.html) {#dealing-with-generalization}

::: annotation
Узагальнення породжує власну групу рефакторингів, в основному
пов\'язаних з переміщенням функціональності по ієрархії успадкування
класів, створення нових класів та інтерфейсів, а також заміни
успадкування делегуванням і навпаки.
:::

::: catalog-list
-   [Pull Up Field](../pull-up-field.html)
-   [Pull Up Method](../pull-up-method.html)
-   [Pull Up Constructor Body](../pull-up-constructor-body.html)
-   [Push Down Field](../push-down-field.html)
-   [Push Down Method](../push-down-method.html)

<!-- -->

-   [Extract Subclass](../extract-subclass.html)
-   [Extract Superclass](../extract-superclass.html)
-   [Extract Interface](../extract-interface.html)
-   [Collapse Hierarchy](../collapse-hierarchy.html)

<!-- -->

-   [Form Template Method](../form-template-method.html)
-   [Replace Inheritance with
    Delegation](../replace-inheritance-with-delegation.html)
-   [Replace Delegation with
    Inheritance](../replace-delegation-with-inheritance.html)
:::
:::::
:::::::
::::::::
:::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::
