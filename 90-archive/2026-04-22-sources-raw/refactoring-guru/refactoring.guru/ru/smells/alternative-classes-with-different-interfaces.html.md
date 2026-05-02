:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} / [Нарушители
объектно-ориентированного
дизайна](../refactoring/smells/oo-abusers.html){.type}
:::

# Альтернативные классы с разными интерфейсами {#альтернативные-классы-с-разными-интерфейсами .title}

::: aka
Также известен как: [Alternative Classes with Different
Interfaces]{style="display:inline-block"}
:::

### Симптомы и признаки

Два класса выполняют одинаковые функции, но имеют разные названия
методов.

<figure class="image">
<img
src="../../images/refactoring/content/smells/alternative-classes-with-different-interfaces-011fe4.png?id=e5fccb2e5390e0a62b5c9f56029bd361"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/alternative-classes-with-different-interfaces-01.png?id=e5fccb2e5390e0a62b5c9f56029bd361 500w,/images/refactoring/content/smells/alternative-classes-with-different-interfaces-01-1.5x.png?id=41cee7e73c74ea76fd0e5a65157aef1d 750w,/images/refactoring/content/smells/alternative-classes-with-different-interfaces-01-2x.png?id=00f0e52d679514e0c16e836e7cee5c24 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

Программист, который создал один из классов, скорей всего, не знал о
том, что в программе уже существует аналогичный по функциям класс.

### Лечение

Постарайтесь привести интерфейс классов к общему знаменателю:

-   [Переименуйте методы](../rename-method.html) так, чтобы они стали
    одинаковыми во всех альтернативных классах.

-   Используйте [перемещение метода](../move-method.html), [добавление
    параметра](../add-parameter.html) и [параметризацию
    метода](../parameterize-method.html) для того, чтобы сигнатура и
    реализация методов стали одинаковыми.

-   Если только часть функциональности классов идентична, попробуйте
    [извлечь эту часть в общий суперкласс](../extract-superclass.html).
    Существующие классы в этом случае станут подклассами.

-   После того как вы определились с вариантом «лечения» и осуществили
    его, подумайте, возможно, один из классов теперь можно удалить.

### Выигрыш

-   Вы избавляетесь от ненужного дублирования кода, и, таким образом,
    уменьшаете его размер.

-   Повышается читабельность кода, улучшается его понимание. Вам больше
    не придется гадать, зачем создавался второй класс, выполняющий точно
    такие же функции, как и первый.

<figure class="image">
<img
src="../../images/refactoring/content/smells/alternative-classes-with-different-interfaces-02ece4.png?id=669874e082965799a70076a120288c6a"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/alternative-classes-with-different-interfaces-02.png?id=669874e082965799a70076a120288c6a 500w,/images/refactoring/content/smells/alternative-classes-with-different-interfaces-02-1.5x.png?id=720363b1a9f666cc0d41345ccf4e4de3 750w,/images/refactoring/content/smells/alternative-classes-with-different-interfaces-02-2x.png?id=db011d16b1dcea2e68d252eb435e63ef 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Не стоит трогать, если\...

-   Иногда объединить классы оказывается невозможно либо настолько
    сложно, что смысла заниматься этой работой нет. Один из
    примеров --- альтернативные классы находятся в двух разных
    библиотеках, каждая из которых имеет свою версию класса.

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

[Утяжелители изменений []{.fa
.fa-arrow-right}](../refactoring/smells/change-preventers.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Отказ от наследства](refused-bequest.html){.btn
.btn-default rel="prev"}
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
