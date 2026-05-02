:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} / [Нарушители
объектно-ориентированного
дизайна](../refactoring/smells/oo-abusers.html){.type}
:::

# Операторы switch {#операторы-switch .title}

::: aka
Также известен как: [Switch Statements]{style="display:inline-block"}
:::

### Симптомы и признаки

У вас есть сложный оператор `switch` или последовательность `if`-ов.

<figure class="image">
<img
src="../../images/refactoring/content/smells/switch-statements-013ae1.png?id=1c5a538d451c3d2b2e43d0d4f53b994b"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/switch-statements-01.png?id=1c5a538d451c3d2b2e43d0d4f53b994b 500w,/images/refactoring/content/smells/switch-statements-01-1.5x.png?id=77cd896e1d6bcf499db5eb5acdda01e4 750w,/images/refactoring/content/smells/switch-statements-01-2x.png?id=da7f9cad4c4bb8bf5d6910aba08680be 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

Одним из очевидных признаков объектно-ориентированного кода служит
сравнительно редкое использование операторов типа `switch` или `case`.
Часто один и тот же блок `switch` оказывается разбросанным по разным
местам программы. При добавлении в него нового варианта приходится
искать все эти блоки `switch` и модифицировать их.

Как правило, заметив блок `switch`, следует подумать о полиморфизме.

### Лечение

-   Чтобы изолировать `switch` и поместить его в нужный класс может
    понадобиться [извлечение метода](../extract-method.html) и
    [перемещение метода](../move-method.html).

-   Если `switch` переключается по коду типа, например, переключается
    режим выполнения программы, то следует использовать [замену
    кодирования типа
    подклассами](../replace-type-code-with-subclasses.html) или [замену
    кодирования типа
    состоянием/стратегией](../replace-type-code-with-state-strategy.html).

-   После настройки структуры наследования следует использовать [замену
    условного оператора
    полиморфизмом](../replace-conditional-with-polymorphism.html).

-   Если вариантов в операторе не очень много и все они приводят к
    вызову одного и того же метода с разными параметрами, введение
    полиморфизма будет избыточным. В этом случае стоит задуматься о
    разбиении этого метода на несколько разных, которые будут выполнять
    каждый свои функции, для чего нужно применить [замену параметра
    набором специализированных
    методов](../replace-parameter-with-explicit-methods.html).

-   Если одним из вариантов условного оператора является `null`,
    используйте [введение Null-объекта](../introduce-null-object.html).

### Выигрыш

-   Улучшает организацию кода.

<figure class="image">
<img
src="../../images/refactoring/content/smells/switch-statements-029112.png?id=29ec9ad9c6d760d2b940a68a1d23c4be"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/switch-statements-02.png?id=29ec9ad9c6d760d2b940a68a1d23c4be 500w,/images/refactoring/content/smells/switch-statements-02-1.5x.png?id=ba92702333e88eed74ee9866f9ac9d0b 750w,/images/refactoring/content/smells/switch-statements-02-2x.png?id=b7c9163ebe366c956f1aa64c4b3312e4 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Не стоит трогать, если\...

-   Когда оператор `switch` выполняет простые действия, нет никакого
    смысла что-то менять в коде.

-   Зачастую оператор `switch` используется в фабричных паттернах
    проектирования ([Фабричный
    метод](../design-patterns/factory-method.html), [Абстрактная
    фабрика](../design-patterns/abstract-factory.html)) для выбора
    создаваемого класса.

::::: {.prom .banner-content .banner .banner-striped .banner-with-image data-id="Ref: Part of the ebook" creative-id="animated-ru" position="content_bottom"}
::: banner-image
Your browser does not support HTML video.
:::

::: banner-text
### Устали читать? {#устали-читать .title}

Сбегайте за подушкой, у нас тут контента на [7 часов]{.blue} чтения.

Или попробуйте наш интерактивный курс. Он гораздо более интересный, чем
банальный текст.

[ Узнать больше...](../refactoring/course.html){.btn .btn-secondary}
:::
:::::

::: next
#### Читаем дальше

[Временное поле []{.fa .fa-arrow-right}](temporary-field.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Нарушители объектно-ориентированного
дизайна](../refactoring/smells/oo-abusers.html){.btn .btn-default
rel="prev"}
:::
::::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-ru" position="sidebar"}
::::: ban
::: {.image .product-image}
[Скидки!]{.banner-discount}
[![](../../images/refactoring/course/course-cover-rua60d.jpg?id=e9d6b4015f4c6c48cf06f7479874d8d7){width="300"
height="300" loading="lazy"}](../refactoring/course.html)
:::

::: {.banner-text .banner-text-ru}
Этот запах кода --- малая часть интерактивного **онлайн курса по
рефакторингу**.

[ Узнать больше...](../refactoring/course.html){.btn .btn-secondary
.btn-block}
:::
:::::
::::::
:::::::::::::::
::::::::::::::::
