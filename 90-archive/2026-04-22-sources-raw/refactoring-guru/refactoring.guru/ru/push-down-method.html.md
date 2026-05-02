:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Решение задач
обобщения](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Спуск метода {#спуск-метода .title}

::: aka
Также известен как: [Push Down Method]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Поведение, реализованное в суперклассе, используется только одним или
несколькими подклассами.
:::

::: after
### Решение

Переместите это поведение в подклассы.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Push Down Method -
Before](../images/refactoring/diagrams/Push%20Down%20Method%20-%20Beforea9e3.png?id=ed31b8dcbc3c98fc668ce95cff9f76a8){width="452"}
:::
:::::

::::: after
::: ba
После
:::

::: ba-container
![Push Down Method -
After](../images/refactoring/diagrams/Push%20Down%20Method%20-%20Aftere373.png?id=dcf9788fec0c13fb8e60fe494d481529){width="452"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

Метод, который планировали сделать универсальным для всех классов, по
факту используется только в одном подклассе. Такая ситуация может
возникнуть, когда планируемые фичи так и не были реализованы.

Кроме того, такая ситуация может возникнуть после извлечения (или
удаления) части функциональности из иерархии классов, после которого
метод остался используемым только в одном подклассе.

Если вы видите, что метод необходим более чем одному подклассу (но не
всем), возможно, стоит создать промежуточный подкласс и переместить
метод в него. Это позволит избежать дублирования кода, которое возникло
бы при спуске метода во все подклассы.

### Достоинства

-   Улучшает связность внутри классов. Метод находится там, где вы
    ожидаете его увидеть.

### Порядок рефакторинга

1.  Объявите метод в подклассе и скопируйте его код из суперкласса.

2.  Удалите метод из суперкласса.

3.  Найдите все места, где используется метод, и убедитесь, что он
    вызывается из нужного подкласса.

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

::::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Анти-рефакторинг

:::: dl
::: dt
[Подъём метода](pull-up-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Родственные рефакторинги

:::: dl
::: dt
[Спуск поля](push-down-field.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Помогает рефакторингу

:::: dl
::: dt
[Извлечение подкласса](extract-subclass.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Борется с запахом

:::: dl
::: dt
[Отказ от наследства](smells/refused-bequest.html)
:::
::::
:::::
:::::::::::::::

::: next
#### Читаем дальше

[Спуск поля []{.fa .fa-arrow-right}](push-down-field.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Подъём тела
конструктора](pull-up-constructor-body.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::
