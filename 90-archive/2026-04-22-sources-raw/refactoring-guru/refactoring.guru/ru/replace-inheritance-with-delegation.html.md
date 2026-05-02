:::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Решение задач
обобщения](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Замена наследования делегированием {#замена-наследования-делегированием .title}

::: aka
Также известен как: [Replace Inheritance with
Delegation]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас есть подкласс, который использует только часть методов суперкласса
или не хочет наследовать его данные.
:::

::: after
### Решение

Создайте поле и поместите в него объект суперкласса, делегируйте
выполнение методов объекту-суперклассу, уберите наследование.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Replace Inheritance with Delegation -
Before](../images/refactoring/diagrams/Replace%20Inheritance%20with%20Delegation%20-%20Before7f4b.png?id=ea9631c50eaae9a19a51f039b44e35bf){width="152"}
:::
:::::

::::: after
::: ba
После
:::

::: ba-container
![Replace Inheritance with Delegation -
After](../images/refactoring/diagrams/Replace%20Inheritance%20with%20Delegation%20-%20Afterf4ed.png?id=7c6d23661792658feae42d051b2fd7ea){width="453"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

Замена наследования композицией может значительно улучшить дизайн
классов, если:

-   Ваш подкласс нарушает *принцип замещения Барбары Лисков*. Другими
    словами, наследование возникло только ради объединения общего кода,
    но не потому, что подкласс «является» (is-a) расширением
    суперкласса.

-   Подкласс использует только часть методов суперкласса. В этом случае,
    это только вопрос времени, пока кто-то не вызовет метод суперкласса,
    который он не должен был вызывать.

Суть рефакторинга сводится к тому, чтобы разделить оба класса, и сделать
суперкласс помощником подкласса, а не его родителем. Вместо того чтобы
наследовать все методы суперкласса, подкласс будет иметь только
необходимые методы, которые будут делегировать выполнение методам
объекта-суперкласса.

### Достоинства

-   Класс не содержит лишних методов, которые достались ему в наследство
    от суперкласса.

-   В поле-делегат можно подставлять разные объекты, имеющие различные
    реализации функциональности. По сути, вы получаете реализацию
    паттерна проектирования [Стратегия](design-patterns/strategy.html).

### Недостатки

-   Приходится писать очень много простых делегирующих методов.

### Порядок рефакторинга

1.  Создайте поле в подклассе для содержания суперкласса. На первом
    этапе поместите в него текущий объект.

2.  Измените методы подкласса так, чтобы они использовали объект
    суперкласса, вместо `this`.

3.  Для методов, которые были унаследованы из суперкласса и которые
    вызываются в клиентском коде, в подклассе нужно создать простые
    делегирующие методы.

4.  Уберите объявление наследования из подкласса.

5.  Измените код инициализации поля, в котором хранится бывший
    суперкласс, созданием нового объекта.

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

::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Анти-рефакторинг

:::: dl
::: dt
[Замена делегирования
наследованием](replace-delegation-with-inheritance.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Реализует паттерн проектирования

:::: dl
::: dt
[Стратегия](design-patterns/strategy.html)
:::
::::
:::::
:::::::::

::: next
#### Читаем дальше

[Замена делегирования наследованием []{.fa
.fa-arrow-right}](replace-delegation-with-inheritance.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Создание шаблонного
метода](form-template-method.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::
