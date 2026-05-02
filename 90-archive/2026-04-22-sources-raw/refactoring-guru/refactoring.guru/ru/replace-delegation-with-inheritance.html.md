:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Решение задач
обобщения](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Замена делегирования наследованием {#замена-делегирования-наследованием .title}

::: aka
Также известен как: [Replace Delegation with
Inheritance]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Класс содержит множество простых делегирующих методов ко всем методам
другого класса.
:::

::: after
### Решение

Сделайте класс наследником делегата, после чего делегирующие методы
потеряют смысл.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Replace Delegation with Inheritance -
Before](../images/refactoring/diagrams/Replace%20Delegation%20with%20Inheritance%20-%20Before7d55.png?id=b408bcc35ce5a341e675818d6c1124a2){width="453"}
:::
:::::

::::: after
::: ba
После
:::

::: ba-container
![Replace Delegation with Inheritance -
After](../images/refactoring/diagrams/Replace%20Delegation%20with%20Inheritance%20-%20After3a88.png?id=633a1c3ff0e87033e10408df57ac6118){width="152"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

Делегирование является более гибким подходом, чем наследование, так как
позволяет изменять поведение класса на лету, заменяя объект к которому
происходит делегирование. Тем не менее, применение делегирования
перестаёт быть выгодным, если вы делегируете действия только одному
классу, причём всем его публичным методам.

Если в этом случае заменить делегирование наследованием, вы избавите
класс от множества делегирующих методов, а себя от необходимости
создавать их для каждого нового метода класса-делегата.

### Достоинства

-   Уменьшает количество кода. Вам больше не нужны все эти делегирующие
    методы.

### Когда нельзя применить

-   Не применяйте рефакторинг, если класс содержит делегирование только
    к части публичных методов класса-делегата. Этим вы нарушите *принцип
    замещения Барбары Лисков*.

-   Этот рефакторинг может быть применён только если класс ещё не имеет
    родителей.

### Порядок рефакторинга

1.  Сделайте класс подклассом класса-делегата.

2.  В поле, содержащее ссылку на объект-делегат, поставьте текущий
    объект.

3.  Один за другим удаляйте методы с простым делегированием. Если у них
    отличались названия, используйте [переименование
    метода](rename-method.html) чтобы привести все методы к одному
    названию.

4.  Замените все обращения к полю-делегату обращениями к текущему
    объекту.

5.  Удалите поле-делегат.

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
::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Анти-рефакторинг

:::: dl
::: dt
[Замена наследования
делегированием](replace-inheritance-with-delegation.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Родственные рефакторинги

:::: dl
::: dt
[Удаление посредника](remove-middle-man.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Борется с запахом

:::: dl
::: dt
[Неуместная близость](smells/inappropriate-intimacy.html)
:::
::::
:::::
::::::::::::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Замена наследования
делегированием](replace-inheritance-with-delegation.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::
