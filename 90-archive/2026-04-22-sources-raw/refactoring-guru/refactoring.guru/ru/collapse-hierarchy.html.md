::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Решение задач
обобщения](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Свёртывание иерархии {#свёртывание-иерархии .title}

::: aka
Также известен как: [Collapse Hierarchy]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас есть некая иерархия классов, в которой подкласс мало чем
отличается от суперкласса.
:::

::: after
### Решение

Слейте подкласс и суперкласс воедино.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Collapse Hierarchy -
Before](../images/refactoring/diagrams/Collapse%20Hierarchy%20-%20Beforea922.png?id=e95d97b3a9d564fbdba5ec0b76748f88){width="152"}
:::
:::::

::::: after
::: ba
После
:::

::: ba-container
![Collapse Hierarchy -
After](../images/refactoring/diagrams/Collapse%20Hierarchy%20-%20After1803.png?id=db86997e7b68eaac829168048cd02a8b){width="152"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

Развитие программы привело к тому, что подкласс и суперкласс стали очень
мало отличаться друг от друга. Какая-то фича была убрана из подкласса,
какой-то метод «переехал» в суперкласс, и вот вы уже имеете два
практически одинаковых класса.

### Достоинства

-   Уменьшается сложность программы. Меньше классов, меньше вещей,
    которые нужно держать в голове, меньше «движущихся частей», меньше
    вероятность сломать что-то при последующих изменениях в коде.

-   Навигация по коду становится проще, когда методы определены только в
    одном классе. Нужный метод не приходится искать по всей иерархии.

### Когда нельзя применить

-   Если в иерархии классов находится больше одного подкласса, то после
    проведения рефакторинга, оставшиеся подклассы должны стать
    наследниками класса, в котором была объединена иерархия.

-   Однако имейте в виду, что это может привести к нарушению *принципа
    подстановки Барбары Лисков*. Например, если в программе эмуляторе
    городского транспорта неверно свернуть суперкласс `Транспорт` в
    подкласс `Автомобиль`, класс `Самолёт` может оказаться наследником
    `Автомобиля`, а это уже неправильно.

### Порядок рефакторинга

1.  Выберите, какой класс убрать удобнее: суперкласс или подкласс.

2.  Используйте [подъём поля](pull-up-field.html) и [подъём
    метода](pull-up-method.html), если вы решили избавиться от
    подкласса. Используйте [спуск поля](push-down-field.html) и [спуск
    метода](push-down-method.html), если убран будет суперкласс.

3.  Замените все использования класса, который будет удалён, классом, в
    который переезжают поля и методы. Зачастую это будет код создания
    классов, указания типов параметров и переменных, а также
    документации в комментариях.

4.  Удалите пустой класс.

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

:::::::::::: row
:::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Родственные рефакторинги

::::: dl
::: dt
[Встраивание класса](inline-class.html)
:::

::: dd
*Свёртывание иерархии* является вариантом [встраивания
класса](inline-class.html), где все фичи переезжают в суперкласс или
подкласс.
:::
:::::
::::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Борется с запахом

:::: dl
::: dt
[Ленивый класс](smells/lazy-class.html)
:::
::::

:::: dl
::: dt
[Теоретическая общность](smells/speculative-generality.html)
:::
::::
:::::::
::::::::::::

::: next
#### Читаем дальше

[Создание шаблонного метода []{.fa
.fa-arrow-right}](form-template-method.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Извлечение
интерфейса](extract-interface.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::
