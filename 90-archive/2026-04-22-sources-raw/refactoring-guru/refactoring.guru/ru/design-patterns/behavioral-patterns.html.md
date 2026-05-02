::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::: {.main-content-container .center-content-container}
::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} /
[Каталог](catalog.html){.type}
:::

# Поведенческие паттерны проектирования {#поведенческие-паттерны-проектирования .title}

Список поведенческих паттернов проектирования, которые решают задачи
эффективного и безопасного взаимодействия между объектами программы.

::: {.d-flex .flex-wrap .justify-content-between}
[[ [![Цепочка
обязанностей](../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x,/images/patterns/cards/chain-of-responsibility-mini-3x.png 3x"}]{.pattern-image}
[Цепочка обязанностей]{.pattern-name .with-aka .pattern-name-long}
[Chain of Responsibility]{.pattern-aka} ]{.pattern-card-top} [
]{.pattern-card-bottom}](chain-of-responsibility.html){.pattern-card-extended
.chain-of-responsibility}

Позволяет передавать запросы последовательно по цепочке обработчиков.
Каждый последующий обработчик решает, может ли он обработать запрос сам
и стоит ли передавать запрос дальше по цепи.

[[
[![Команда](../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x,/images/patterns/cards/command-mini-3x.png 3x"}]{.pattern-image}
[Команда]{.pattern-name .with-aka} [Command]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](command.html){.pattern-card-extended .command}

Превращает запросы в объекты, позволяя передавать их как аргументы при
вызове методов, ставить запросы в очередь, логировать их, а также
поддерживать отмену операций.

[[
[![Итератор](../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x,/images/patterns/cards/iterator-mini-3x.png 3x"}]{.pattern-image}
[Итератор]{.pattern-name .with-aka} [Iterator]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](iterator.html){.pattern-card-extended
.iterator}

Даёт возможность последовательно обходить элементы составных объектов,
не раскрывая их внутреннего представления.

[[
[![Посредник](../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x,/images/patterns/cards/mediator-mini-3x.png 3x"}]{.pattern-image}
[Посредник]{.pattern-name .with-aka} [Mediator]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](mediator.html){.pattern-card-extended
.mediator}

Позволяет уменьшить связанность множества классов между собой, благодаря
перемещению этих связей в один класс-посредник.

[[
[![Снимок](../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x,/images/patterns/cards/memento-mini-3x.png 3x"}]{.pattern-image}
[Снимок]{.pattern-name .with-aka} [Memento]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](memento.html){.pattern-card-extended .memento}

Позволяет сохранять и восстанавливать прошлые состояния объектов, не
раскрывая подробностей их реализации.

[[
[![Наблюдатель](../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x,/images/patterns/cards/observer-mini-3x.png 3x"}]{.pattern-image}
[Наблюдатель]{.pattern-name .with-aka .pattern-name-long}
[Observer]{.pattern-aka} ]{.pattern-card-top} [
]{.pattern-card-bottom}](observer.html){.pattern-card-extended
.observer}

Создаёт механизм подписки, позволяющий одним объектам следить и
реагировать на события, происходящие в других объектах.

[[
[![Состояние](../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x,/images/patterns/cards/state-mini-3x.png 3x"}]{.pattern-image}
[Состояние]{.pattern-name .with-aka} [State]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](state.html){.pattern-card-extended .state}

Позволяет объектам менять поведение в зависимости от своего состояния.
Извне создаётся впечатление, что изменился класс объекта.

[[
[![Стратегия](../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x,/images/patterns/cards/strategy-mini-3x.png 3x"}]{.pattern-image}
[Стратегия]{.pattern-name .with-aka} [Strategy]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](strategy.html){.pattern-card-extended
.strategy}

Определяет семейство схожих алгоритмов и помещает каждый из них в
собственный класс, после чего алгоритмы можно взаимозаменять прямо во
время исполнения программы.

[[ [![Шаблонный
метод](../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x,/images/patterns/cards/template-method-mini-3x.png 3x"}]{.pattern-image}
[Шаблонный метод]{.pattern-name .with-aka} [Template
Method]{.pattern-aka} ]{.pattern-card-top} [
]{.pattern-card-bottom}](template-method.html){.pattern-card-extended
.template-method}

Определяет скелет алгоритма, перекладывая ответственность за некоторые
его шаги на подклассы. Паттерн позволяет подклассам переопределять шаги
алгоритма, не меняя его общей структуры.

[[
[![Посетитель](../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x,/images/patterns/cards/visitor-mini-3x.png 3x"}]{.pattern-image}
[Посетитель]{.pattern-name .with-aka} [Visitor]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](visitor.html){.pattern-card-extended .visitor}

Позволяет добавлять в программу новые операции, не изменяя классы
объектов, над которыми эти операции могут выполняться.
:::

::: next
#### Читаем дальше

[Цепочка обязанностей []{.fa
.fa-arrow-right}](chain-of-responsibility.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Заместитель](proxy.html){.btn .btn-default
rel="prev"}
:::
:::::::
::::::::
:::::::::
