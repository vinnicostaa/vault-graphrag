:::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
:::

# Коли рефакторити {#коли-рефакторити .title}

::: {style="float:right; margin-left: 10px;"}
<figure class="image">
<img
src="../../images/refactoring/content/pages/r1198e.svg?id=0a20f72cdd91691d0e6b29f2c5519e4c"
data-fetchpriority="high" />
</figure>
:::

#### Правило трьох

1.  Роблячи щось в перший раз, ви просто це робите.

2.  Роблячи щось аналогічне вдруге, ви мружитеся від необхідності
    повторення, але все-таки повторюєте те ж саме.

3.  Роблячи щось схоже в третій раз, ви починаєте рефакторинг.

::: {style="float:right; margin-left: 10px;"}
<figure class="image">
<img
src="../../images/refactoring/content/pages/r2d073.svg?id=5b3806c58901d111cee12fb60df9e0ac"
loading="lazy" />
</figure>
:::

#### Коли робите нову фічу

-   Рефакторинг допомагає зрозуміти чужий код. Якщо код, в який потрібно
    додати нову фічу, недостатньо ясний, рефакторинг дозволяє зробити
    його більш очевидними для вас і для того, хто буде працювати з ним в
    майбутньому.

-   Рефакторинг полегшує написання нового коду. Після рефакторингу
    додавання нової фічі відбувається значно простіше і займає менше
    часу.

::: {style="float:right; margin-left: 10px;"}
<figure class="image">
<img
src="../../images/refactoring/content/pages/r37b3a.svg?id=b3ab8426bd85943ecedccbaf02181342"
loading="lazy" />
</figure>
:::

#### Коли виправляєте баги

Помилки --- як таргани, люблять жити в темних затхлих місцях вашого
коду. Спробуйте навести лад в коді і помилки знайдуться самі по собі.

Крім того, вам не доведеться створювати спеціальні завдання для
рефакторингу, які так не люблять бачити в звітах менеджери.

::: {style="float:right; margin-left: 10px;"}
<figure class="image">
<img
src="../../images/refactoring/content/pages/r4d93f.svg?id=68c5945ac20e360e480c1254ac94be82"
loading="lazy" />
</figure>
:::

#### Під час код-рев'ю

Коли ви робите рев'ю нової фічі, можливо це останній шанс почистити код,
перед тим як він стане доступним публічно.

Найкраще проводити таке рев'ю разом з автором коду. У цьому випадку, ви
будете пропонувати автору зміни, а потім разом вирішувати, наскільки
складно зробити ту чи іншу зміну. Водночас, невеличкі зміни можна буде
здійснювати дуже швидко.

::: next
#### Читаємо далі

[Як рефакторити код []{.fa .fa-arrow-right}](how-to.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Технічний борг](technical-debt.html){.btn
.btn-default rel="prev"}
:::
::::::::::
:::::::::::
::::::::::::
