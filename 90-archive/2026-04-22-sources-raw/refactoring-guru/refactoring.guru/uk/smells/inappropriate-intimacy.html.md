:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} / [Заплутувальники
зв\'язками](../refactoring/smells/couplers.html){.type}
:::

# Недоречна близькість {#недоречна-близькість .title}

::: aka
Також відомий як: [Inappropriate Intimacy]{style="display:inline-block"}
:::

### Симптоми і ознаки

Один клас використовує службові поля і методи іншого класу.

<figure class="image">
<img
src="../../images/refactoring/content/smells/inappropriate-intimacy-017f9d.png?id=31bf185f4ff946f13e28d27d377a4b6c"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/inappropriate-intimacy-01.png?id=31bf185f4ff946f13e28d27d377a4b6c 500w,/images/refactoring/content/smells/inappropriate-intimacy-01-1.5x.png?id=c41452986a03eb3fb0af7992184f8b22 750w,/images/refactoring/content/smells/inappropriate-intimacy-01-2x.png?id=1857242bb9cf7b2ca50327897d256711 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

Дивіться уважно за класами, які проводять надто багато часу разом.
Хороші класи повинні знати один про одного якомога менше. Такі класи
легше підтримувати і повторно використовувати.

### Лікування

-   Найпростіший вихід - за допомогою [переміщення
    методу](../move-method.html) і [переміщення
    поля](../move-field.html) перенести частини одного класу в інший (в
    той, де вони використовуються). Проте це може спрацювати тільки в
    тому випадку, якщо оригінальний клас не використовує переміщувані
    поля і методи.

    <figure class="image">
    <img
    src="../../images/refactoring/content/smells/inappropriate-intimacy-024036.png?id=3f23c8df6eb8cf91b46e39fa912ff85c"
    style="aspect-ratio: 1.67;"
    srcset="/images/refactoring/content/smells/inappropriate-intimacy-02.png?id=3f23c8df6eb8cf91b46e39fa912ff85c 500w,/images/refactoring/content/smells/inappropriate-intimacy-02-1.5x.png?id=4c3b0852df877ca82d7fe12d61235b0a 750w,/images/refactoring/content/smells/inappropriate-intimacy-02-2x.png?id=f3ac66384b14f074ed36b6d9c3720b20 1000w"
    sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
    height="300" />
    </figure>

-   Іншим рішенням є [відокремлення залежних частин в окремий
    клас](../extract-class.html) і [приховання
    делегування](../hide-delegate.html) до цього класу.

-   Якщо між класами існує взаємна залежність, варто використати [заміну
    двонаправленого зв'язку на
    однонапрямлену](../change-bidirectional-association-to-unidirectional.html).

-   Якщо близькість виникає між підкласом і батьківським класом, краще
    розглянути можливість [заміни делегування
    наслідуванням](../replace-delegation-with-inheritance.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/inappropriate-intimacy-03b31e.png?id=de33e2285073feaabd1a81cffdcd386c"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/inappropriate-intimacy-03.png?id=de33e2285073feaabd1a81cffdcd386c 500w,/images/refactoring/content/smells/inappropriate-intimacy-03-1.5x.png?id=a2b1cf219e0a4a3056c4e7116b973d85 750w,/images/refactoring/content/smells/inappropriate-intimacy-03-2x.png?id=51bfcec36fb123f146857099555dc478 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Виграш

-   Покращує організацію коду.

-   Спрощує технічну підтримку і повторне використання коду.

::::: {.prom .banner-content .banner .banner-striped .banner-with-image data-id="Ref: Part of the ebook" creative-id="animated-uk" position="content_bottom"}
::: banner-image
Your browser does not support HTML video.
:::

::: banner-text
### Замучились читати? {#замучились-читати .title}

Збігайте за подушкою, в нас тут контенту на [7 годин]{.blue} читання.

Або спробуйте наш новий інтерактивний курс. Він набагато цікавіший за
банальний тест.

[ Дізнатися більше...](../refactoring/course.html){.btn .btn-secondary}
:::
:::::

::: next
#### Читаємо далі

[Ланцюжок викликів []{.fa .fa-arrow-right}](message-chains.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Заздрісні функції](feature-envy.html){.btn
.btn-default rel="prev"}
:::
::::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-uk" position="sidebar"}
::::: ban
::: {.image .product-image}
[Знижки!]{.banner-discount}
[![](../../images/refactoring/course/course-cover-uk4341.jpg?id=e53397ef67078e742fb132da37ca7cc8){width="300"
height="300" loading="lazy"}](../refactoring/course.html)
:::

::: {.banner-text .banner-text-uk}
Цей запах коду є частиною нашого інтерактивного **онлайн курсу з
рефакторингу**.

[ Подробиці...](../refactoring/course.html){.btn .btn-secondary
.btn-block}
:::
:::::
::::::
:::::::::::::::
::::::::::::::::
