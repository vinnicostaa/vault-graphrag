:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} /
[Раздувальщики](../refactoring/smells/bloaters.html){.type}
:::

# Длинный метод {#длинный-метод .title}

::: aka
Также известен как: [Long Method]{style="display:inline-block"}
:::

### Симптомы и признаки

Метод содержит слишком большое число строк кода. Длина метода более
десяти строк должна начинать вас беспокоить.

<figure class="image">
<img
src="../../images/refactoring/content/smells/long-method-012095.png?id=ba3b4a6d8ef25a8f676543cee5e1e019"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/long-method-01.png?id=ba3b4a6d8ef25a8f676543cee5e1e019 500w,/images/refactoring/content/smells/long-method-01-1.5x.png?id=0aba5fa284aac80c0c3c909fca9ff51c 750w,/images/refactoring/content/smells/long-method-01-2x.png?id=fd9479c2221526f284c9b14fb94f9169 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

В метод всё время что-то добавляется, но ничего не выносится. Так как
писать код намного проще, чем читать, этот запах долго остаётся
незамеченным --- до тех пор пока метод не превратится в настоящего
монстра.

Стоит помнить, что человеку зачастую ментально сложнее создать новый
метод, чем дописать что-то в уже существующий: «Как же, мне нужно
добавить всего две строки, не буду же я создавать для этого целый
метод».

Таким образом, добавляется одна строка за другой, а в результате метод
превращается в большую тарелку спагетти.

### Лечение

Следует придерживаться такого правила: если ощущается необходимость
что-то прокомментировать внутри метода, этот код лучше выделить в новый
метод. Даже одну строку имеет смысл выделить в метод, если она нуждается
в разъяснениях. К тому же, если у метода хорошее название, то не нужно
будет смотреть в его код, чтобы понять, что он делает.

<figure class="image">
<img
src="../../images/refactoring/content/smells/long-method-027655.png?id=274350a92b305ae79848ab40b3bdb0cb"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/long-method-02.png?id=274350a92b305ae79848ab40b3bdb0cb 500w,/images/refactoring/content/smells/long-method-02-1.5x.png?id=dbb0cb724dd6beefe6474b86ae33913c 750w,/images/refactoring/content/smells/long-method-02-2x.png?id=beba19e840bf4763e85f006ef79cc89c 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

-   Для сокращения тела метода достаточно применить [извлечение
    метода](../extract-method.html).

-   Если локальные переменные и параметры препятствуют выделению метода,
    можно применить [замену временной переменной вызовом
    метода](../replace-temp-with-query.html), [замену параметров
    объектом](../introduce-parameter-object.html) и [передачу всего
    объекта](../preserve-whole-object.html).

-   Если предыдущие способы не помогли, можно попробовать выделить весь
    метод в отдельный объект с помощью [замены метода объектом
    методов](../replace-method-with-method-object.html).

-   Условные операторы и циклы свидетельствуют о возможности выделения
    кода в отдельный метод. Для работы с условными выражениями подходит
    [декомпозиция условных операторов](../decompose-conditional.html).
    Для работы с циклом --- [извлечение метода](../extract-method.html).

### Выигрыш

-   Из всех видов объектного кода дольше всего выживают классы с
    короткими методами. Чем длиннее ваш метод или функция, тем труднее
    будет её понять и поддерживать.

-   Кроме того, в длинных методах зачастую можно обнаружить «залежи»
    дублирования кода.

<figure class="image">
<img
src="../../images/refactoring/content/smells/long-method-03ca39.png?id=82ce2d388aa14bdae4e8f62b875f0259"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/long-method-03.png?id=82ce2d388aa14bdae4e8f62b875f0259 500w,/images/refactoring/content/smells/long-method-03-1.5x.png?id=fd71378acb56437e26c4b9b5654a3f28 750w,/images/refactoring/content/smells/long-method-03-2x.png?id=ba563da468cf42a704ff53da2e89b6d5 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Производительность

Многие волнуются, что увеличение числа методов может плохо сказаться на
производительности. В абсолютном большинстве случаев, это не является
реальной проблемой, так что **просто перестаньте об этом думать.**

Имея чистый и понятный код, вы с большей вероятностью натолкнётесь на
отличный способ реструктуризировать код программы и увеличить реальную
производительность, если такая надобность вообще будет.

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

[Большой класс []{.fa .fa-arrow-right}](large-class.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa
.fa-arrow-left} Раздувальщики](../refactoring/smells/bloaters.html){.btn
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
