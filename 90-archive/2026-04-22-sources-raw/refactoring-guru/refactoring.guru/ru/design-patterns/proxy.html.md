::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Структурные
паттерны](structural-patterns.html){.type}
:::

# Заместитель {#заместитель .title}

::: aka
Также известен как: [Proxy]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть паттерна {#intent}

**Заместитель** --- это структурный паттерн проектирования, который
позволяет подставлять вместо реальных объектов специальные
объекты-заменители. Эти объекты перехватывают вызовы к оригинальному
объекту, позволяя сделать что-то *до* или *после* передачи
вызова оригиналу.

<figure class="image">
<img
src="../../images/patterns/content/proxy/proxyf258.png?id=efece4647fb11e3f7539291796327666"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/proxy/proxy.png?id=efece4647fb11e3f7539291796327666 640w,/images/patterns/content/proxy/proxy-1.5x.png?id=80a11690fa49172c517470a834289b60 960w,/images/patterns/content/proxy/proxy-2x.png?id=fb3d14e21c210a758d4777f4d93dce09 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Паттерн Заместитель" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Для чего вообще контролировать доступ к объектам? Рассмотрим такой
пример: у вас есть внешний ресурсоёмкий объект, который нужен не все
время, а изредка.

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/problem-ru25d8.png?id=06c6e7834de954b614b0c8ac366948a5"
style="aspect-ratio: 3.19;"
srcset="/images/patterns/diagrams/proxy/problem-ru.png?id=06c6e7834de954b614b0c8ac366948a5 510w,/images/patterns/diagrams/proxy/problem-ru-1.5x.png?id=8b3623dbc1c77f72cf97b1be88458728 765w,/images/patterns/diagrams/proxy/problem-ru-2x.png?id=7dd1c9e02c388f6d182dcbb96ded152d 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Проблема, которую решает Заместитель" />
<figcaption><p>Запросы к базе данных могут быть
очень медленными.</p></figcaption>
</figure>

Мы могли бы создавать этот объект не в самом начале программы, а только
тогда, когда он кому-то реально понадобится. Каждый клиент объекта
получил бы некий код отложенной инициализации. Но, вероятно, это привело
бы к множественному дублированию кода.

В идеале, этот код хотелось бы поместить прямо в служебный класс, но это
не всегда возможно. Например, код класса может находиться в закрытой
сторонней библиотеке.
:::

::: {.section .solution}
##  Решение {#solution}

Паттерн Заместитель предлагает создать новый класс-дублёр, имеющий тот
же интерфейс, что и оригинальный служебный объект. При получении запроса
от клиента объект-заместитель сам бы создавал экземпляр служебного
объекта и переадресовывал бы ему всю реальную работу.

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/solution-rudeb9.png?id=3f0693eb00bf83b5bbd3ce99c1257e82"
style="aspect-ratio: 3.19;"
srcset="/images/patterns/diagrams/proxy/solution-ru.png?id=3f0693eb00bf83b5bbd3ce99c1257e82 510w,/images/patterns/diagrams/proxy/solution-ru-1.5x.png?id=8df9931b4983477c28f6162b0eec4c48 765w,/images/patterns/diagrams/proxy/solution-ru-2x.png?id=ba367e6b357ae9b81ee5459d313955fd 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Решение с помощью Заместителя" />
<figcaption><p>Заместитель «притворяется» базой данных, ускоряя работу
за счёт ленивой инициализации и кеширования
повторяющихся запросов.</p></figcaption>
</figure>

Но в чём же здесь польза? Вы могли бы поместить в класс заместителя
какую-то промежуточную логику, которая выполнялась бы до (или после)
вызовов этих же методов в настоящем объекте. А благодаря одинаковому
интерфейсу, объект-заместитель можно передать в любой код, ожидающий
сервисный объект.
:::

::: {.section .analogy}
##  Аналогия из жизни {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/live-example66da.png?id=a268c57fdaf073ee81cf4dfc7239eae2"
style="aspect-ratio: 2.57;"
srcset="/images/patterns/diagrams/proxy/live-example.png?id=a268c57fdaf073ee81cf4dfc7239eae2 540w,/images/patterns/diagrams/proxy/live-example-1.5x.png?id=4b8ee7f90cf76180004e9f5d37661ee2 810w,/images/patterns/diagrams/proxy/live-example-2x.png?id=8b8083fa1954d2d92ca84a5a5636ead7 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Платёжная карта — это заместитель пачки наличных" />
<figcaption><p>Платёжной картой можно расплачиваться, как
и наличными.</p></figcaption>
</figure>

Платёжная карточка --- это заместитель пачки наличных. И карточка, и
наличные имеют общий интерфейс --- ими можно оплачивать товары. Для
покупателя польза в том, что не надо таскать с собой тонны наличных, а
владелец магазина рад, что ему не нужно делать дорогостоящую инкассацию
наличности в банк --- деньги поступают к нему на счёт напрямую.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/structure7fcd.png?id=f2478a82a84e1a1e512a8414bf1abd1c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/proxy/structure.png?id=f2478a82a84e1a1e512a8414bf1abd1c 370w,/images/patterns/diagrams/proxy/structure-1.5x.png?id=0db7bf3c1193b2a1961894349f1e07ad 555w,/images/patterns/diagrams/proxy/structure-2x.png?id=3d54eeca9af4aa373e989a73463539b5 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Структура классов паттерна Заместитель" /><img
src="../../images/patterns/diagrams/proxy/structure-indexed2b2c.png?id=e56d420f31925b3d41348c5036d3cd64"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.05;"
srcset="/images/patterns/diagrams/proxy/structure-indexed.png?id=e56d420f31925b3d41348c5036d3cd64 410w,/images/patterns/diagrams/proxy/structure-indexed-1.5x.png?id=5b24af632e76d81d24eab0b7c0be1a66 615w,/images/patterns/diagrams/proxy/structure-indexed-2x.png?id=ddf2b3b4807e52330c456c59fc52d504 820w"
sizes="(max-width: 720px) 100vw, 410px" loading="lazy" width="410"
alt="Структура классов паттерна Заместитель" />
</figure>
:::

1.  **Интерфейс сервиса** определяет общий интерфейс для сервиса и
    заместителя. Благодаря этому, объект заместителя можно использовать
    там, где ожидается объект сервиса.

2.  **Сервис** содержит полезную бизнес-логику.

3.  **Заместитель** хранит ссылку на объект сервиса. После того как
    заместитель заканчивает свою работу (например, инициализацию,
    логирование, защиту или другое), он передаёт вызовы вложенному
    сервису.

    Заместитель может сам отвечать за создание и удаление объекта
    сервиса.

4.  **Клиент** работает с объектами через интерфейс сервиса. Благодаря
    этому, его можно «одурачить», подменив объект сервиса объектом
    заместителя.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

В этом примере **Заместитель** помогает добавить в программу механизм
ленивой инициализации и кеширования результатов работы библиотеки
интеграции с YouTube.

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/example4e80.png?id=ddb1402479562b9e2c776933cc861bed"
style="aspect-ratio: 1.23;"
srcset="/images/patterns/diagrams/proxy/example.png?id=ddb1402479562b9e2c776933cc861bed 490w,/images/patterns/diagrams/proxy/example-1.5x.png?id=9b7608e1ef46a52d4ca1d7d89986e5b0 735w,/images/patterns/diagrams/proxy/example-2x.png?id=40f22d12d183b9285a380e4a072efb16 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Структура классов примера паттерна Заместитель" />
<figcaption><p>Пример кеширования результатов работы реального сервиса с
помощью заместителя.</p></figcaption>
</figure>

Оригинальный объект начинал загрузку по сети, даже если пользователь
запрашивал одно и то же видео. Заместитель же загружает видео только
один раз, используя для этого служебный объект, но в остальных случаях
возвращает закешированный файл.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Интерфейс удалённого сервиса.
interface ThirdPartyYouTubeLib is
    method listVideos()
    method getVideoInfo(id)
    method downloadVideo(id)

// Конкретная реализация сервиса. Методы этого класса
// запрашивают у YouTube различную информацию. Скорость запроса
// зависит не только от качества интернет-канала пользователя,
// но и от состояния самого YouTube. Значит, чем больше будет
// вызовов к сервису, тем менее отзывчивой станет программа.
class ThirdPartyYouTubeClass implements ThirdPartyYouTubeLib is
    method listVideos() is
        // Получить список видеороликов с помощью API YouTube.

    method getVideoInfo(id) is
        // Получить детальную информацию о каком-то видеоролике.

    method downloadVideo(id) is
        // Скачать видео с YouTube.

// С другой стороны, можно кешировать запросы к YouTube и не
// повторять их какое-то время, пока кеш не устареет. Но внести
// этот код напрямую в сервисный класс нельзя, так как он
// находится в сторонней библиотеке. Поэтому мы поместим логику
// кеширования в отдельный класс-обёртку. Он будет делегировать
// запросы к сервисному объекту, только если нужно
// непосредственно выслать запрос.
class CachedYouTubeClass implements ThirdPartyYouTubeLib is
    private field service: ThirdPartyYouTubeLib
    private field listCache, videoCache
    field needReset

    constructor CachedYouTubeClass(service: ThirdPartyYouTubeLib) is
        this.service = service

    method listVideos() is
        if (listCache == null || needReset)
            listCache = service.listVideos()
        return listCache

    method getVideoInfo(id) is
        if (videoCache == null || needReset)
            videoCache = service.getVideoInfo(id)
        return videoCache

    method downloadVideo(id) is
        if (!downloadExists(id) || needReset)
            service.downloadVideo(id)

// Класс GUI, который использует сервисный объект. Вместо
// реального сервиса, мы подсунем ему объект-заместитель. Клиент
// ничего не заметит, так как заместитель имеет тот же
// интерфейс, что и сервис.
class YouTubeManager is
    protected field service: ThirdPartyYouTubeLib

    constructor YouTubeManager(service: ThirdPartyYouTubeLib) is
        this.service = service

    method renderVideoPage(id) is
        info = service.getVideoInfo(id)
        // Отобразить страницу видеоролика.

    method renderListPanel() is
        list = service.listVideos()
        // Отобразить список превьюшек видеороликов.

    method reactOnUserInput() is
        renderVideoPage()
        renderListPanel()

// Конфигурационная часть приложения создаёт и передаёт клиентам
// объект заместителя.
class Application is
    method init() is
        YouTubeService = new ThirdPartyYouTubeClass()
        YouTubeProxy = new CachedYouTubeClass(YouTubeService)
        manager = new YouTubeManager(YouTubeProxy)
        manager.reactOnUserInput()</code></pre>
</figure>
:::

:::::::::::::: {.section .applicability-container}
##  Применимость {#applicability}

::::::::::::: applicability
::: applicability-problem
Ленивая инициализация (виртуальный прокси). Когда у вас есть тяжёлый
объект, грузящий данные из файловой системы или базы данных.
:::

::: applicability-solution
Вместо того, чтобы грузить данные сразу после старта программы, можно
сэкономить ресурсы и создать объект тогда, когда он действительно
понадобится.
:::

::: applicability-problem
Защита доступа (защищающий прокси). Когда в программе есть разные типы
пользователей, и вам хочется защищать объект от неавторизованного
доступа. Например, если ваши объекты --- это важная часть операционной
системы, а пользователи --- сторонние программы (хорошие или
вредоносные).
:::

::: applicability-solution
Прокси может проверять доступ при каждом вызове и передавать выполнение
служебному объекту, если доступ разрешён.
:::

::: applicability-problem
Локальный запуск сервиса (удалённый прокси). Когда настоящий сервисный
объект находится на удалённом сервере.
:::

::: applicability-solution
В этом случае заместитель транслирует запросы клиента в вызовы по сети в
протоколе, понятном удалённому сервису.
:::

::: applicability-problem
Логирование запросов (логирующий прокси). Когда требуется хранить
историю обращений к сервисному объекту.
:::

::: applicability-solution
Заместитель может сохранять историю обращения клиента к сервисному
объекту.
:::

::: applicability-problem
Кеширование объектов («умная» ссылка). Когда нужно кешировать результаты
запросов клиентов и управлять их жизненным циклом.
:::

::: applicability-solution
Заместитель может подсчитывать количество ссылок на сервисный объект,
которые были отданы клиенту и остаются активными. Когда все ссылки
освобождаются, можно будет освободить и сам сервисный объект (например,
закрыть подключение к базе данных).

Кроме того, Заместитель может отслеживать, не менял ли клиент сервисный
объект. Это позволит использовать объекты повторно и здóрово экономить
ресурсы, особенно если речь идёт о больших прожорливых сервисах.
:::
:::::::::::::
::::::::::::::

::: {.section .checklist}
##  Шаги реализации {#checklist}

1.  Определите интерфейс, который бы сделал заместитель и оригинальный
    объект взаимозаменяемыми.

2.  Создайте класс заместителя. Он должен содержать ссылку на сервисный
    объект. Чаще всего, сервисный объект создаётся самим заместителем. В
    редких случаях заместитель получает готовый сервисный объект от
    клиента через конструктор.

3.  Реализуйте методы заместителя в зависимости от его предназначения. В
    большинстве случаев, проделав какую-то полезную работу, методы
    заместителя должны передать запрос сервисному объекту.

4.  Подумайте о введении фабрики, которая решала бы, какой из объектов
    создавать --- заместитель или реальный сервисный объект. Но, с
    другой стороны, эта логика может быть помещена в создающий метод
    самого заместителя.

5.  Подумайте, не реализовать ли вам ленивую инициализацию сервисного
    объекта при первом обращении клиента к методам заместителя.
:::

:::::: {.section .pros-cons}
##  Преимущества и недостатки {#pros-cons}

::::: row
::: col-sm-6
-    Позволяет контролировать сервисный объект незаметно для клиента.
-    Может работать, даже если сервисный объект ещё не создан.
-    Может контролировать жизненный цикл служебного объекта.
:::

::: col-sm-6
-    Усложняет код программы из-за введения дополнительных классов.
-    Увеличивает время отклика от сервиса.
:::
:::::
::::::

::: {.section .relations}
##  Отношения с другими паттернами {#relations}

-   С [Адаптером](adapter.html) вы получаете доступ к существующему
    объекту через другой интерфейс. Используя [Заместитель](proxy.html),
    интерфейс остается неизменным. Используя
    [Декоратор](decorator.html), вы получаете доступ к объекту через
    расширенный интерфейс.

-   [Фасад](facade.html) похож на [Заместитель](proxy.html) тем, что
    замещает сложную подсистему и может сам её инициализировать. Но в
    отличие от *Фасада*, *Заместитель* имеет тот же интерфейс, что его
    служебный объект, благодаря чему их можно взаимозаменять.

-   [Декоратор](decorator.html) и [Заместитель](proxy.html) имеют схожие
    структуры, но разные назначения. Они похожи тем, что оба построены
    на принципе композиции и делегируют работу другим объектам. Паттерны
    отличаются тем, что *Заместитель* сам управляет жизнью сервисного
    объекта, а обёртывание *Декораторов* контролируется клиентом.
:::

::: {.section .implementations}
##  Примеры реализации паттерна {#implementations}

[![Заместитель на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](proxy/csharp/example.html "Заместитель на C#"){.prog-lang-link}
[![Заместитель на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](proxy/cpp/example.html "Заместитель на C++"){.prog-lang-link}
[![Заместитель на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](proxy/go/example.html "Заместитель на Go"){.prog-lang-link}
[![Заместитель на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](proxy/java/example.html "Заместитель на Java"){.prog-lang-link}
[![Заместитель на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](proxy/php/example.html "Заместитель на PHP"){.prog-lang-link}
[![Заместитель на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](proxy/python/example.html "Заместитель на Python"){.prog-lang-link}
[![Заместитель на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](proxy/ruby/example.html "Заместитель на Ruby"){.prog-lang-link}
[![Заместитель на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](proxy/rust/example.html "Заместитель на Rust"){.prog-lang-link}
[![Заместитель на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](proxy/swift/example.html "Заместитель на Swift"){.prog-lang-link}
[![Заместитель на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](proxy/typescript/example.html "Заместитель на TypeScript"){.prog-lang-link}
:::

:::::: {#book-promo .banner-set}
::::: {.prom .banner-content .banner-bg .banner-striped .banner-with-image data-id="DP: 1: Ne vtykay" creative-id="standard-ru" position="content_bottom"}
::: banner-image
[![](../../images/patterns/banners/patterns-book-banner-1-b158a.png?id=0877cab2f0102d98cd07b50af0e5beea){width="200"
height="200" loading="lazy"
srcset="/images/patterns/banners/patterns-book-banner-1-b-2x.png?id=5572aa55e5b09e59780aca9e0ea8e44b 2x"}](book.html)
:::

::: banner-text
### Не втыкай в транспорте {#не-втыкай-в-транспорте .title}

Лучше почитай нашу книгу о паттернах проектирования.

Теперь это удобно делать даже во время поездок в общественном
транспорте.

[ Узнать больше...](book.html){.btn .btn-secondary}
:::
:::::
::::::

::: next
#### Читаем дальше

[Поведенческие паттерны []{.fa
.fa-arrow-right}](behavioral-patterns.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Легковес](flyweight.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner-removable .banner-removable-patterns data-id="DP: Sidebar" creative-id="standard-sidebar-ru" position="sidebar"}
:::::: banner-inner
:::: image3d-book-right
::: {.image3d-cover style="background: #0b3752;"}
[![](../../images/patterns/book/web-cover-ruee5e.png?id=ea25feb28a6deb8aa4af6a633904a568){width="250"
height="375" loading="lazy"
srcset="/images/patterns/book/web-cover-ru-2x.png?id=95101b307f6dba409f28a417746ff3c1 2x"}](book.html)
:::
::::

::: {style="margin-top: 1rem"}
Эта статья является частью нашей электронной книги **Погружение в
Паттерны Проектирования**.

[ Узнать больше...](book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::
