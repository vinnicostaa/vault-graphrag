:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.page .text}
# [ПАТЕРНИ]{.dp1-h-1} [ПРОЕКТУВАННЯ]{.dp1-h-2} [на]{.dp-lang-in} [Rust]{.dp-lang} {#патерни-проектування-на-rust .dp1-h .dp-abs .dp-c .dp-h1}

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: dp-text
## Каталог **Rust** прикладів {#каталог-rust-прикладів .dp-h2}

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: dp-double-cols
#### Породжувальні патерни {#породжувальні-патерни .dp-h4 .dp-creational-title}

:::::::: {.dp-pattern-item .dp-creational}
::: dp-pattern-card
![Абстрактна
фабрика](../../images/patterns/cards/abstract-factory-minid1c5.png?id=4c3927c446313a38ce77dfee38111e27){srcset="/images/patterns/cards/abstract-factory-mini-2x.png?id=22236aaa65ff52cbde1c713216d52c1f 2x,/images/patterns/cards/abstract-factory-mini-3x.png 3x"}
:::

#### Абстрактна фабрика []{.dp-popularity} {#абстрактна-фабрика .dp-pattern-title}

::: dp-pattern-intent
Дає змогу створювати сімейства пов'язаних об'єктів, не прив'язуючись до
конкретних класів створюваних об'єктів.
:::

::::: dp-pattern-links
<div>

[ Головний розділ](abstract-factory.html){.dp-pattern-link}

</div>

<div>

[ Приклад
коду](abstract-factory/rust/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-creational}
::: dp-pattern-card
![Будівельник](../../images/patterns/cards/builder-mini6d24.png?id=19b95fd05e6469679752c0554b116815){srcset="/images/patterns/cards/builder-mini-2x.png?id=de6d0938678b86903a1426dddfdd13bf 2x,/images/patterns/cards/builder-mini-3x.png 3x"}
:::

#### Будівельник []{.dp-popularity} {#будівельник .dp-pattern-title}

::: dp-pattern-intent
Дає змогу створювати складні об\'єкти крок за кроком. Будівельник дає
можливість використовувати один і той самий код будівництва для
отримання різних відображень об\'єктів.
:::

::::: dp-pattern-links
<div>

[ Головний розділ](builder.html){.dp-pattern-link}

</div>

<div>

[ Приклад коду](builder/rust/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::::: {.dp-pattern-item .dp-creational}
::: dp-pattern-card
![Фабричний
метод](../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x,/images/patterns/cards/factory-method-mini-3x.png 3x"}
:::

#### Фабричний метод []{.dp-popularity} {#фабричний-метод .dp-pattern-title}

::: dp-pattern-intent
Визначає загальний інтерфейс для створення об\'єктів у суперкласі,
дозволяючи підкласам змінювати тип створюваних об\'єктів.
:::

::::::: dp-pattern-links
<div>

[ Головний розділ](factory-method.html){.dp-pattern-link}

</div>

<div>

<div>

[ Приклад коду
1](factory-method/rust/example.html#example-0){.dp-pattern-link}

</div>

<div>

[ Приклад коду
2](factory-method/rust/example.html#example-1){.dp-pattern-link}

</div>

</div>
:::::::
::::::::::

:::::::: {.dp-pattern-item .dp-creational}
::: dp-pattern-card
![Прототип](../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x,/images/patterns/cards/prototype-mini-3x.png 3x"}
:::

#### Прототип []{.dp-popularity} {#прототип .dp-pattern-title}

::: dp-pattern-intent
Дає змогу копіювати об\'єкти, не вдаючись у подробиці їхньої реалізації.
:::

::::: dp-pattern-links
<div>

[ Головний розділ](prototype.html){.dp-pattern-link}

</div>

<div>

[ Приклад коду](prototype/rust/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-creational}
::: dp-pattern-card
![Одинак](../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x,/images/patterns/cards/singleton-mini-3x.png 3x"}
:::

#### Одинак []{.dp-popularity} {#одинак .dp-pattern-title}

::: dp-pattern-intent
Гарантує, що клас має лише один екземпляр, та надає глобальну точку
доступу до нього.
:::

::::: dp-pattern-links
<div>

[ Головний розділ](singleton.html){.dp-pattern-link}

</div>

<div>

[ Приклад коду](singleton/rust/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

#### Структурні патерни {#структурні-патерни .dp-h4 .dp-structural-title}

:::::::: {.dp-pattern-item .dp-structural}
::: dp-pattern-card
![Адаптер](../../images/patterns/cards/adapter-minid866.png?id=b2ee4f681fb589be5a0685b94692aebb){srcset="/images/patterns/cards/adapter-mini-2x.png?id=8274d99afbbe9c63bfbfd0d68ceeffc7 2x,/images/patterns/cards/adapter-mini-3x.png 3x"}
:::

#### Адаптер []{.dp-popularity} {#адаптер .dp-pattern-title}

::: dp-pattern-intent
Дає змогу об'єктам із несумісними інтерфейсами працювати разом.
:::

::::: dp-pattern-links
<div>

[ Головний розділ](adapter.html){.dp-pattern-link}

</div>

<div>

[ Приклад коду](adapter/rust/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-structural}
::: dp-pattern-card
![Міст](../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x,/images/patterns/cards/bridge-mini-3x.png 3x"}
:::

#### Міст []{.dp-popularity} {#міст .dp-pattern-title}

::: dp-pattern-intent
Розділяє один або кілька класів на дві окремі ієрархії --- абстракцію та
реалізацію, дозволяючи змінювати код в одній гілці класів, незалежно від
іншої.
:::

::::: dp-pattern-links
<div>

[ Головний розділ](bridge.html){.dp-pattern-link}

</div>

<div>

[ Приклад коду](bridge/rust/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-structural}
::: dp-pattern-card
![Компонувальник](../../images/patterns/cards/composite-minib543.png?id=a369d98d18b417f255d04568fd0131b8){srcset="/images/patterns/cards/composite-mini-2x.png?id=3f7f811fefeb0b64f6774746eb42af09 2x,/images/patterns/cards/composite-mini-3x.png 3x"}
:::

#### Компонувальник []{.dp-popularity} {#компонувальник .dp-pattern-title}

::: dp-pattern-intent
Дає змогу згрупувати декілька об\'єктів у деревоподібну структуру, а
потім працювати з нею так, ніби це одиничний об\'єкт.
:::

::::: dp-pattern-links
<div>

[ Головний розділ](composite.html){.dp-pattern-link}

</div>

<div>

[ Приклад коду](composite/rust/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-structural}
::: dp-pattern-card
![Декоратор](../../images/patterns/cards/decorator-mini6adb.png?id=d30458908e315af195cb183bc52dbef9){srcset="/images/patterns/cards/decorator-mini-2x.png?id=3b58e540d7d28523080cad341ed9b2e9 2x,/images/patterns/cards/decorator-mini-3x.png 3x"}
:::

#### Декоратор []{.dp-popularity} {#декоратор .dp-pattern-title}

::: dp-pattern-intent
Дає змогу динамічно додавати об\'єктам нову функціональність, загортаючи
їх у корисні «обгортки».
:::

::::: dp-pattern-links
<div>

[ Головний розділ](decorator.html){.dp-pattern-link}

</div>

<div>

[ Приклад коду](decorator/rust/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-structural}
::: dp-pattern-card
![Фасад](../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x,/images/patterns/cards/facade-mini-3x.png 3x"}
:::

#### Фасад []{.dp-popularity} {#фасад .dp-pattern-title}

::: dp-pattern-intent
Надає простий інтерфейс до складної системи класів, бібліотеки або
фреймворку.
:::

::::: dp-pattern-links
<div>

[ Головний розділ](facade.html){.dp-pattern-link}

</div>

<div>

[ Приклад коду](facade/rust/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-structural}
::: dp-pattern-card
![Легковаговик](../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x,/images/patterns/cards/flyweight-mini-3x.png 3x"}
:::

#### Легковаговик []{.dp-popularity} {#легковаговик .dp-pattern-title}

::: dp-pattern-intent
Дає змогу вмістити більшу кількість об\'єктів у відведеній оперативній
пам\'яті. Легковаговик заощаджує пам\'ять, розподіляючи спільний стан
об\'єктів між собою, замість зберігання однакових даних у кожному
об\'єкті.
:::

::::: dp-pattern-links
<div>

[ Головний розділ](flyweight.html){.dp-pattern-link}

</div>

<div>

[ Приклад коду](flyweight/rust/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-structural}
::: dp-pattern-card
![Замісник](../../images/patterns/cards/proxy-minid8d7.png?id=25890b11e7dc5af29625ccd0678b63a8){srcset="/images/patterns/cards/proxy-mini-2x.png?id=8638fac9dc08c992852492f9cb29d9c6 2x,/images/patterns/cards/proxy-mini-3x.png 3x"}
:::

#### Замісник []{.dp-popularity} {#замісник .dp-pattern-title}

::: dp-pattern-intent
Дає змогу підставляти замість реальних об\'єктів спеціальні
об\'єкти-замінники. Ці об\'єкти перехоплюють виклики до оригінального
об\'єкта, дозволяючи зробити щось *до* чи *після* передачі виклику
оригіналові.
:::

::::: dp-pattern-links
<div>

[ Головний розділ](proxy.html){.dp-pattern-link}

</div>

<div>

[ Приклад коду](proxy/rust/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

#### Поведінкові патерни {#поведінкові-патерни .dp-h4 .dp-behavioral-title}

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: dp-double-cols
:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Ланцюжок
обов\'язків](../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x,/images/patterns/cards/chain-of-responsibility-mini-3x.png 3x"}
:::

#### Ланцюжок обов\'язків []{.dp-popularity} {#ланцюжок-обовязків .dp-pattern-title}

::: dp-pattern-intent
Дає змогу передавати запити послідовно ланцюжком обробників. Кожен
наступний обробник вирішує, чи може він обробити запит сам і чи варто
передавати запит далі ланцюжком.
:::

::::: dp-pattern-links
<div>

[ Головний розділ](chain-of-responsibility.html){.dp-pattern-link}

</div>

<div>

[ Приклад
коду](chain-of-responsibility/rust/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Команда](../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x,/images/patterns/cards/command-mini-3x.png 3x"}
:::

#### Команда []{.dp-popularity} {#команда .dp-pattern-title}

::: dp-pattern-intent
Перетворює запити на об\'єкти, дозволяючи передавати їх як аргументи під
час виклику методів, ставити запити в чергу, логувати їх, а також
підтримувати скасування операцій.
:::

::::: dp-pattern-links
<div>

[ Головний розділ](command.html){.dp-pattern-link}

</div>

<div>

[ Приклад коду](command/rust/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Ітератор](../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x,/images/patterns/cards/iterator-mini-3x.png 3x"}
:::

#### Ітератор []{.dp-popularity} {#ітератор .dp-pattern-title}

::: dp-pattern-intent
Дає змогу послідовно обходити елементи складових об\'єктів, не
розкриваючи їхньої внутрішньої організації.
:::

::::: dp-pattern-links
<div>

[ Головний розділ](iterator.html){.dp-pattern-link}

</div>

<div>

[ Приклад коду](iterator/rust/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Посередник](../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x,/images/patterns/cards/mediator-mini-3x.png 3x"}
:::

#### Посередник []{.dp-popularity} {#посередник .dp-pattern-title}

::: dp-pattern-intent
Дає змогу зменшити зв\'язаність великої кількості класів між собою,
завдяки переміщенню цих зв\'язків до одного класу-посередника.
:::

::::: dp-pattern-links
<div>

[ Головний розділ](mediator.html){.dp-pattern-link}

</div>

<div>

[ Приклад коду](mediator/rust/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Знімок](../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x,/images/patterns/cards/memento-mini-3x.png 3x"}
:::

#### Знімок []{.dp-popularity} {#знімок .dp-pattern-title}

::: dp-pattern-intent
Дає змогу зберігати та відновлювати минулий стан об\'єктів, не
розкриваючи подробиць їхньої реалізації.
:::

::::: dp-pattern-links
<div>

[ Головний розділ](memento.html){.dp-pattern-link}

</div>

<div>

[ Приклад коду](memento/rust/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Спостерігач](../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x,/images/patterns/cards/observer-mini-3x.png 3x"}
:::

#### Спостерігач []{.dp-popularity} {#спостерігач .dp-pattern-title}

::: dp-pattern-intent
Створює механізм підписки, що дає змогу одним об'єктам стежити й
реагувати на події, які відбуваються в інших об'єктах.
:::

::::: dp-pattern-links
<div>

[ Головний розділ](observer.html){.dp-pattern-link}

</div>

<div>

[ Приклад коду](observer/rust/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Стан](../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x,/images/patterns/cards/state-mini-3x.png 3x"}
:::

#### Стан []{.dp-popularity} {#стан .dp-pattern-title}

::: dp-pattern-intent
Дає змогу об\'єктам змінювати поведінку в залежності від їхнього стану.
Ззовні створюється враження, ніби змінився клас об\'єкта.
:::

::::: dp-pattern-links
<div>

[ Головний розділ](state.html){.dp-pattern-link}

</div>

<div>

[ Приклад коду](state/rust/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Стратегія](../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x,/images/patterns/cards/strategy-mini-3x.png 3x"}
:::

#### Стратегія []{.dp-popularity} {#стратегія .dp-pattern-title}

::: dp-pattern-intent
Визначає сімейство схожих алгоритмів і розміщує кожен з них у власному
класі. Після цього алгоритми можна заміняти один на інший прямо під час
виконання програми.
:::

::::: dp-pattern-links
<div>

[ Головний розділ](strategy.html){.dp-pattern-link}

</div>

<div>

[ Приклад коду](strategy/rust/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Шаблонний
метод](../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x,/images/patterns/cards/template-method-mini-3x.png 3x"}
:::

#### Шаблонний метод []{.dp-popularity} {#шаблонний-метод .dp-pattern-title}

::: dp-pattern-intent
Визначає кістяк алгоритму, перекладаючи відповідальність за деякі його
кроки на підкласи. Патерн дозволяє підкласам перевизначати кроки
алгоритму, не змінюючи його загальної структури.
:::

::::: dp-pattern-links
<div>

[ Головний розділ](template-method.html){.dp-pattern-link}

</div>

<div>

[ Приклад
коду](template-method/rust/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Відвідувач](../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x,/images/patterns/cards/visitor-mini-3x.png 3x"}
:::

#### Відвідувач []{.dp-popularity} {#відвідувач .dp-pattern-title}

::: dp-pattern-intent
Дає змогу додавати до програми нові операції, не змінюючи класи
об\'єктів, над якими ці операції можуть виконуватися.
:::

::::: dp-pattern-links
<div>

[ Головний розділ](visitor.html){.dp-pattern-link}

</div>

<div>

[ Приклад коду](visitor/rust/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
