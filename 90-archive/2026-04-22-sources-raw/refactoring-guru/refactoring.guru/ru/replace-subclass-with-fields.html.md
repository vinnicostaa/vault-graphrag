::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Организация
данных](refactoring/techniques/organizing-data.html){.type}
:::

# Замена подкласса полями {#замена-подкласса-полями .title}

::: aka
Также известен как: [Replace Subclass with
Fields]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас есть подклассы, которые отличаются только методами, возвращающими
данные-константы.
:::

::: after
### Решение

Замените методы полями в родительском классе и удалите подклассы.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Replace Subclass with Fields -
Before](../images/refactoring/diagrams/Replace%20Subclass%20with%20Fields%20-%20Beforec050.png?id=ea6525cc6b55e1a03fdb35def943c675){width="415"}
:::
:::::

::::: after
::: ba
После
:::

::: ba-container
![Replace Subclass with Fields -
After](../images/refactoring/diagrams/Replace%20Subclass%20with%20Fields%20-%20After22d1.png?id=bd1b29687aa333b3adbeb2bfb3e78614){width="152"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

Бывает так, что вам нужно развернуть действие рефакторинга избавления от
кодирования типа.

В одном из подобных случаев иерархия подклассов может отличаться только
значениями, возвращаемыми определёнными методами. Причём эти значения не
являются результатом вычисления, а жёстко прописаны либо в самих
методах, либо в полях, возвращаемых методами. Чтобы упростить
архитектуру классов, такая иерархия может быть свёрнута в один класс,
содержащий одно или несколько полей с нужными значениями в зависимости
от ситуации.

Нужда в этих изменениях могла возникнуть после перемещения большого
числа функциональностей из иерархии классов куда-то в другое место.
После этого текущая иерархия потеряла свою ценность, и подклассы стали
создавать только избыточную сложность.

### Достоинства

-   Упрощает архитектуру системы. Создание подклассов --- слишком
    избыточное решение, если все, что нужно сделать, это возвращать
    разные значения в нескольких методах.

### Порядок рефакторинга

1.  Примените к подклассам [замену конструктора фабричным
    методом](replace-constructor-with-factory-method.html).

2.  Замените вызовы конструкторов подклассов вызовами фабричного метода
    суперкласса.

3.  Объявите в суперклассе поля для хранения значений каждого из методов
    подклассов, возвращающих константные значения.

4.  Создайте защищённый конструктор суперкласса для инициализации новых
    полей.

5.  Создайте или модифицируйте имеющиеся конструкторы подклассов, чтобы
    они вызывали новый конструктор родительского класса и передавали в
    него соответствующие значения.

6.  Реализуйте каждый константный метод в родительском классе так, чтобы
    он возвращал значение соответствующего поля, а затем удалите метод
    из подкласса.

7.  Если конструктор подкласса имеет какую-то дополнительную
    функциональность, примените [встраивание метода](inline-method.html)
    для встраивания его конструктора в фабричный метод суперкласса.

8.  Удалите подкласс.

::::: {.prom .banner-content .banner .banner-striped .banner-with-image data-id="Ref: Part of the ebook" creative-id="animated-ru" position="content_bottom"}
::: banner-image
Your browser does not support HTML video.
:::

::: banner-text
### Устали читать? {#устали-читать .title}

Сбегайте за подушкой, у нас тут контента на [7 часов]{.blue} чтения.

Или попробуйте наш интерактивный курс. Он гораздо более интересный, чем
банальный текст.

[ Узнать больше...](refactoring/course.html){.btn .btn-secondary}
:::
:::::

:::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-12}
### Анти-рефакторинг

:::: dl
::: dt
[Замена кодирования типа
подклассами](replace-type-code-with-subclasses.html)
:::
::::
:::::
::::::

::: next
#### Читаем дальше

[Упрощение условных выражений []{.fa
.fa-arrow-right}](refactoring/techniques/simplifying-conditional-expressions.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Замена кодирования типа
состоянием/стратегией](replace-type-code-with-state-strategy.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-ru" position="sidebar"}
::::: ban
::: {.image .product-image}
[Скидки!]{.banner-discount}
[![](../images/refactoring/course/course-cover-rua60d.jpg?id=e9d6b4015f4c6c48cf06f7479874d8d7){width="300"
height="300" loading="lazy"}](refactoring/course.html)
:::

::: {.banner-text .banner-text-ru}
Этот рефакторинг --- малая часть интерактивного **онлайн курса по
рефакторингу**.

[ Узнать больше...](refactoring/course.html){.btn .btn-secondary
.btn-block}
:::
:::::
::::::
::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::
