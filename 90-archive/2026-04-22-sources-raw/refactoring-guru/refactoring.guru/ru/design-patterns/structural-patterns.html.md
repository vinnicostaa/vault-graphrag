::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::: {.main-content-container .center-content-container}
::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} /
[Каталог](catalog.html){.type}
:::

# Структурные паттерны проектирования {#структурные-паттерны-проектирования .title}

Список структурных паттернов проектирования, которые отвечают за
построение удобных в поддержке иерархий классов.

::: {.d-flex .flex-wrap .justify-content-between}
[[
[![Адаптер](../../images/patterns/cards/adapter-minid866.png?id=b2ee4f681fb589be5a0685b94692aebb){srcset="/images/patterns/cards/adapter-mini-2x.png?id=8274d99afbbe9c63bfbfd0d68ceeffc7 2x,/images/patterns/cards/adapter-mini-3x.png 3x"}]{.pattern-image}
[Адаптер]{.pattern-name .with-aka} [Adapter]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](adapter.html){.pattern-card-extended .adapter}

Позволяет объектам с несовместимыми интерфейсами работать вместе.

[[
[![Мост](../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x,/images/patterns/cards/bridge-mini-3x.png 3x"}]{.pattern-image}
[Мост]{.pattern-name .with-aka} [Bridge]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](bridge.html){.pattern-card-extended .bridge}

Разделяет один или несколько классов на две отдельные иерархии ---
абстракцию и реализацию, позволяя изменять их независимо друг от друга.

[[
[![Компоновщик](../../images/patterns/cards/composite-minib543.png?id=a369d98d18b417f255d04568fd0131b8){srcset="/images/patterns/cards/composite-mini-2x.png?id=3f7f811fefeb0b64f6774746eb42af09 2x,/images/patterns/cards/composite-mini-3x.png 3x"}]{.pattern-image}
[Компоновщик]{.pattern-name .with-aka} [Composite]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](composite.html){.pattern-card-extended
.composite}

Позволяет сгруппировать множество объектов в древовидную структуру, а
затем работать с ней так, как будто это единичный объект.

[[
[![Декоратор](../../images/patterns/cards/decorator-mini6adb.png?id=d30458908e315af195cb183bc52dbef9){srcset="/images/patterns/cards/decorator-mini-2x.png?id=3b58e540d7d28523080cad341ed9b2e9 2x,/images/patterns/cards/decorator-mini-3x.png 3x"}]{.pattern-image}
[Декоратор]{.pattern-name .with-aka} [Decorator]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](decorator.html){.pattern-card-extended
.decorator}

Позволяет динамически добавлять объектам новую функциональность,
оборачивая их в полезные «обёртки».

[[
[![Фасад](../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x,/images/patterns/cards/facade-mini-3x.png 3x"}]{.pattern-image}
[Фасад]{.pattern-name .with-aka} [Facade]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](facade.html){.pattern-card-extended .facade}

Предоставляет простой интерфейс к сложной системе классов, библиотеке
или фреймворку.

[[
[![Легковес](../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x,/images/patterns/cards/flyweight-mini-3x.png 3x"}]{.pattern-image}
[Легковес]{.pattern-name .with-aka} [Flyweight]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](flyweight.html){.pattern-card-extended
.flyweight}

Позволяет вместить бóльшее количество объектов в отведённую оперативную
память. Легковес экономит память, разделяя общее состояние объектов
между собой, вместо хранения одинаковых данных в каждом объекте.

[[
[![Заместитель](../../images/patterns/cards/proxy-minid8d7.png?id=25890b11e7dc5af29625ccd0678b63a8){srcset="/images/patterns/cards/proxy-mini-2x.png?id=8638fac9dc08c992852492f9cb29d9c6 2x,/images/patterns/cards/proxy-mini-3x.png 3x"}]{.pattern-image}
[Заместитель]{.pattern-name .with-aka} [Proxy]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](proxy.html){.pattern-card-extended .proxy}

Позволяет подставлять вместо реальных объектов специальные
объекты-заменители. Эти объекты перехватывают вызовы к оригинальному
объекту, позволяя сделать что-то *до* или *после* передачи вызова
оригиналу.
:::

::: next
#### Читаем дальше

[Адаптер []{.fa .fa-arrow-right}](adapter.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Одиночка](singleton.html){.btn .btn-default
rel="prev"}
:::
:::::::
::::::::
:::::::::
