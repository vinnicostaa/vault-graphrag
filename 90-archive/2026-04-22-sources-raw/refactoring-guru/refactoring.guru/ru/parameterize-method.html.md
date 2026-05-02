::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Упрощение вызовов
методов](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Параметризация метода {#параметризация-метода .title}

::: aka
Также известен как: [Parameterize Method]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Несколько методов выполняют похожие действия, которые отличаются только
какими-то внутренними значениями, числами или операциями.
:::

::: after
### Решение

Объедините все эти методы в один с параметром, в который будет
передаваться отличающееся значение.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Parameterize Method -
Before](../images/refactoring/diagrams/Parameterize%20Method%20-%20Beforeb413.png?id=28369c5a763d56c4aa68ff0a9d2d6beb){width="171"}
:::
:::::

::::: after
::: ba
После
:::

::: ba-container
![Parameterize Method -
After](../images/refactoring/diagrams/Parameterize%20Method%20-%20After948e.png?id=fd1602ffde98a8fea3ad50cb63e14553){width="171"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

Если у вас есть схожие методы, скорей всего, в них присутствует
дублирующий код со всеми вытекающими недостатками.

Кроме того, если вам нужно будет добавить ещё одну вариацию
функциональности, вам придётся создавать ещё один метод. Вместо этого
можно бы было запустить существующий метод с другим параметром.

### Недостатки

-   Иногда при проведении рефакторинга можно переусердствовать, в
    результате чего у вас появится длинный и сложный общий метод вместо
    нескольких простых.

-   Кроме того, будьте осторожны, выделяя в параметр переключатель
    какой-то функциональности. В дальнейшем это может привести к
    созданию большого условного оператора, который надо будет лечить с
    помощью [замены параметра набором специализированных
    методов](replace-parameter-with-explicit-methods.html).

### Порядок рефакторинга

1.  Создайте новый метод с параметром и поместите в него общий для всех
    методов код, применяя [извлечение метода](extract-method.html).
    Обратите внимание, иногда общей оказывается только определённая
    часть методов. В этом случае рефакторинг сведётся к извлечению
    только этой общей части в новый метод.

2.  Отличающееся значение замените параметром в коде нового метода.

3.  Для каждого старого метода найдите места, где они вызываются, и
    поменяйте их вызовы на вызовы нового метода с параметром. После чего
    старый метод можно удалить.

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

:::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Анти-рефакторинг

:::: dl
::: dt
[Замена параметра набором специализированных
методов](replace-parameter-with-explicit-methods.html)
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Родственные рефакторинги

:::: dl
::: dt
[Извлечение метода](extract-method.html)
:::
::::

:::: dl
::: dt
[Создание шаблонного метода](form-template-method.html)
:::
::::
:::::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Борется с запахом

:::: dl
::: dt
[Дублирование кода](smells/duplicate-code.html)
:::
::::
:::::
::::::::::::::

::: next
#### Читаем дальше

[Замена параметра набором специализированных методов []{.fa
.fa-arrow-right}](replace-parameter-with-explicit-methods.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Разделение запроса и
модификатора](separate-query-from-modifier.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
