:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Упрощение вызовов
методов](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Удаление параметра {#удаление-параметра .title}

::: aka
Также известен как: [Remove Parameter]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Параметр не используется в теле метода.
:::

::: after
### Решение

Удалите неиспользуемый параметр.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Remove Parameter -
Before](../images/refactoring/diagrams/Remove%20Parameter%20-%20Before1916.png?id=c49cf53fd9ebf39daa1ec27bb897a5cf){width="171"}
:::
:::::

::::: after
::: ba
После
:::

::: ba-container
![Remove Parameter -
After](../images/refactoring/diagrams/Remove%20Parameter%20-%20After4dba.png?id=93177c2a159087d73ebef60cc8c158a3){width="171"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

Каждый параметр в вызове метода заставляет человека, пишущего код
метода, размышлять о том, какая информация может оказаться в этом
параметре. И если параметр потом вовсе не используется в теле метода,
значит, весь этот мыслительный процесс уходит впустую.

Кроме того, дополнительные параметры --- это ещё и лишний код к
выполнению.

Порой мы добавляем параметры на будущее, предчувствуя скорые изменения в
методе, для которых потребуется этот параметр. Тем не менее, опыт
показывает, что лучше добавить параметр тогда, когда он действительно
понадобится, ведь ожидаемые изменения могут так и не наступить.

### Достоинства

-   Метод должен содержать только действительно необходимые параметры.

### Когда нельзя применить

-   Не стоит начинать этот рефакторинг, если метод имеет альтернативные
    реализации в подклассах или в суперклассе, и ваш параметр
    используется в этих реализациях.

### Порядок рефакторинга

1.  Проверьте, не определён ли метод в суперклассе или подклассе. Если
    так, используется ли там параметр? Если в какой-то из реализаций
    параметр используется, воздержитесь от рефакторинга.

2.  Следующий шаг важен, чтобы сохранить работоспособность программы во
    время рефакторинга. Итак, создайте новый метод, скопировав старый, и
    удалите из него требуемый параметр. Замените код старого метода
    вызовом нового метода.

3.  Найдите все обращения к старому методу и замените их обращениями к
    новому методу.

4.  Удалите старый метод. Этот шаг неосуществим, если старый метод
    является частью публичного интерфейса. В этом случае, старый метод
    нужно пометить как устаревший (`deprecated`).

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
[Добавление параметра](add-parameter.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Родственные рефакторинги

:::: dl
::: dt
[Переименование метода](rename-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Помогает рефакторингу

:::: dl
::: dt
[Замена параметра вызовом
метода](replace-parameter-with-method-call.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Борется с запахом

:::: dl
::: dt
[Теоретическая общность](smells/speculative-generality.html)
:::
::::
:::::
:::::::::::::::

::: next
#### Читаем дальше

[Разделение запроса и модификатора []{.fa
.fa-arrow-right}](separate-query-from-modifier.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Добавление параметра](add-parameter.html){.btn
.btn-default rel="prev"}
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
