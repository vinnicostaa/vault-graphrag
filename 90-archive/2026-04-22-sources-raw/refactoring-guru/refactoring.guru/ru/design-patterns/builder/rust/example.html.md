::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Строитель](../../builder.html){.type} / [Rust](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Строитель](../../../../images/patterns/cards/builder-mini6d24.png?id=19b95fd05e6469679752c0554b116815){srcset="/images/patterns/cards/builder-mini-2x.png?id=de6d0938678b86903a1426dddfdd13bf 2x"}
:::

# **Строитель** на Rust {#строитель-на-rust .pattern-example-title-block-title}
::::

:::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Строитель** --- это порождающий паттерн проектирования, который
позволяет создавать объекты пошагово.

В отличие от других порождающих паттернов, Строитель позволяет
производить различные продукты, используя один и тот же процесс
строительства.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Строитель ](../../builder.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Car & car manual builders](#example-0)
:::

::: en-file
 builders
:::

:::: en-inside
::: en-file
  [mod](#example-0--builders-mod-rs)
:::
::::

:::: en-inside
::: en-file
  [car](#example-0--builders-car-rs)
:::
::::

:::: en-inside
::: en-file
  [car_manual](#example-0--builders-car_manual-rs)
:::
::::

::: en-file
 cars
:::

:::: en-inside
::: en-file
  [mod](#example-0--cars-mod-rs)
:::
::::

:::: en-inside
::: en-file
  [car](#example-0--cars-car-rs)
:::
::::

:::: en-inside
::: en-file
  [manual](#example-0--cars-manual-rs)
:::
::::

::: en-file
 [components](#example-0--components-rs)
:::

::: en-file
 [director](#example-0--director-rs)
:::

::: en-file
 [main](#example-0--main-rs)
:::
:::::::::::::::::::::::
::::::::::::::::::::::::::

[]{#example-0}

## Car & car manual builders {#example-0-title}

This slightly synthetic example illustrates how you can use the Builder
pattern to construct totally different products using the same building
process. For example, the trait `Builder` declares steps for assembling
a car. However, depending on the builder implementation, a constructed
object can be something different, for example, a car manual. The
resulting manual will contain instructions from each building step,
making it accurate and up-to-date.

The **Builder** design pattern is not the same as the **Fluent
Interface** idiom (that relies on *method chaining*), although Rust
developers sometimes use those terms interchangeably.

1.  **Fluent Interface** is a way to chain methods for constructing or
    modifying an object using the following approach:

    <figure class="code">
    <pre class="code" lang="rust"><code>let car = Car::default().places(5).gas(30)</code></pre>
    </figure>

    It\'s pretty elegant way to construct an object. Still, such a code
    may not be an instance of the Builder pattern.

2.  While the **Builder** pattern also suggests constructing object step
    by step, it also lets you build different types of products using
    the same construction process.

### []{#example-0--builders .anchor} **builders:** Builders {#builders-builders .example-title}

#### []{#example-0--builders-mod-rs .anchor} **builders/mod.rs**

<figure class="code">
<pre class="code" lang="rust"><code>mod car;
mod car_manual;

use crate::components::{CarType, Engine, GpsNavigator, Transmission};

/// Builder defines how to assemble a car.
pub trait Builder {
    type OutputType;
    fn set_car_type(&amp;mut self, car_type: CarType);
    fn set_seats(&amp;mut self, seats: u16);
    fn set_engine(&amp;mut self, engine: Engine);
    fn set_transmission(&amp;mut self, transmission: Transmission);
    fn set_gps_navigator(&amp;mut self, gps_navigator: GpsNavigator);
    fn build(self) -&gt; Self::OutputType;
}

pub use car::CarBuilder;
pub use car_manual::CarManualBuilder;</code></pre>
</figure>

#### []{#example-0--builders-car-rs .anchor} **builders/car.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use crate::{
    cars::Car,
    components::{CarType, Engine, GpsNavigator, Transmission},
};

use super::Builder;

pub const DEFAULT_FUEL: f64 = 5f64;

#[derive(Default)]
pub struct CarBuilder {
    car_type: Option&lt;CarType&gt;,
    engine: Option&lt;Engine&gt;,
    gps_navigator: Option&lt;GpsNavigator&gt;,
    seats: Option&lt;u16&gt;,
    transmission: Option&lt;Transmission&gt;,
}

impl Builder for CarBuilder {
    type OutputType = Car;

    fn set_car_type(&amp;mut self, car_type: CarType) {
        self.car_type = Some(car_type);
    }

    fn set_engine(&amp;mut self, engine: Engine) {
        self.engine = Some(engine);
    }

    fn set_gps_navigator(&amp;mut self, gps_navigator: GpsNavigator) {
        self.gps_navigator = Some(gps_navigator);
    }

    fn set_seats(&amp;mut self, seats: u16) {
        self.seats = Some(seats);
    }

    fn set_transmission(&amp;mut self, transmission: Transmission) {
        self.transmission = Some(transmission);
    }

    fn build(self) -&gt; Car {
        Car::new(
            self.car_type.expect(&quot;Please, set a car type&quot;),
            self.seats.expect(&quot;Please, set a number of seats&quot;),
            self.engine.expect(&quot;Please, set an engine configuration&quot;),
            self.transmission.expect(&quot;Please, set up transmission&quot;),
            self.gps_navigator,
            DEFAULT_FUEL,
        )
    }
}</code></pre>
</figure>

#### []{#example-0--builders-car_manual-rs .anchor} **builders/car_manual.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use crate::{
    cars::Manual,
    components::{CarType, Engine, GpsNavigator, Transmission},
};

use super::Builder;

#[derive(Default)]
pub struct CarManualBuilder {
    car_type: Option&lt;CarType&gt;,
    engine: Option&lt;Engine&gt;,
    gps_navigator: Option&lt;GpsNavigator&gt;,
    seats: Option&lt;u16&gt;,
    transmission: Option&lt;Transmission&gt;,
}

/// Builds a car manual instead of an actual car.
impl Builder for CarManualBuilder {
    type OutputType = Manual;

    fn set_car_type(&amp;mut self, car_type: CarType) {
        self.car_type = Some(car_type);
    }

    fn set_engine(&amp;mut self, engine: Engine) {
        self.engine = Some(engine);
    }

    fn set_gps_navigator(&amp;mut self, gps_navigator: GpsNavigator) {
        self.gps_navigator = Some(gps_navigator);
    }

    fn set_seats(&amp;mut self, seats: u16) {
        self.seats = Some(seats);
    }

    fn set_transmission(&amp;mut self, transmission: Transmission) {
        self.transmission = Some(transmission);
    }

    fn build(self) -&gt; Manual {
        Manual::new(
            self.car_type.expect(&quot;Please, set a car type&quot;),
            self.seats.expect(&quot;Please, set a number of seats&quot;),
            self.engine.expect(&quot;Please, set an engine configuration&quot;),
            self.transmission.expect(&quot;Please, set up transmission&quot;),
            self.gps_navigator,
        )
    }
}</code></pre>
</figure>

### []{#example-0--cars .anchor} **cars:** Products {#cars-products .example-title}

#### []{#example-0--cars-mod-rs .anchor} **cars/mod.rs**

<figure class="code">
<pre class="code" lang="rust"><code>mod car;
mod manual;

pub use car::Car;
pub use manual::Manual;</code></pre>
</figure>

#### []{#example-0--cars-car-rs .anchor} **cars/car.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use crate::components::{CarType, Engine, GpsNavigator, Transmission};

pub struct Car {
    car_type: CarType,
    seats: u16,
    engine: Engine,
    transmission: Transmission,
    gps_navigator: Option&lt;GpsNavigator&gt;,
    fuel: f64,
}

impl Car {
    pub fn new(
        car_type: CarType,
        seats: u16,
        engine: Engine,
        transmission: Transmission,
        gps_navigator: Option&lt;GpsNavigator&gt;,
        fuel: f64,
    ) -&gt; Self {
        Self {
            car_type,
            seats,
            engine,
            transmission,
            gps_navigator,
            fuel,
        }
    }

    pub fn car_type(&amp;self) -&gt; CarType {
        self.car_type
    }

    pub fn fuel(&amp;self) -&gt; f64 {
        self.fuel
    }

    pub fn set_fuel(&amp;mut self, fuel: f64) {
        self.fuel = fuel;
    }

    pub fn seats(&amp;self) -&gt; u16 {
        self.seats
    }

    pub fn engine(&amp;self) -&gt; &amp;Engine {
        &amp;self.engine
    }

    pub fn transmission(&amp;self) -&gt; &amp;Transmission {
        &amp;self.transmission
    }

    pub fn gps_navigator(&amp;self) -&gt; &amp;Option&lt;GpsNavigator&gt; {
        &amp;self.gps_navigator
    }
}</code></pre>
</figure>

#### []{#example-0--cars-manual-rs .anchor} **cars/manual.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use crate::components::{CarType, Engine, GpsNavigator, Transmission};

pub struct Manual {
    car_type: CarType,
    seats: u16,
    engine: Engine,
    transmission: Transmission,
    gps_navigator: Option&lt;GpsNavigator&gt;,
}

impl Manual {
    pub fn new(
        car_type: CarType,
        seats: u16,
        engine: Engine,
        transmission: Transmission,
        gps_navigator: Option&lt;GpsNavigator&gt;,
    ) -&gt; Self {
        Self {
            car_type,
            seats,
            engine,
            transmission,
            gps_navigator,
        }
    }
}

impl std::fmt::Display for Manual {
    fn fmt(&amp;self, f: &amp;mut std::fmt::Formatter&lt;&#39;_&gt;) -&gt; std::fmt::Result {
        writeln!(f, &quot;Type of car: {:?}&quot;, self.car_type)?;
        writeln!(f, &quot;Count of seats: {}&quot;, self.seats)?;
        writeln!(
            f,
            &quot;Engine: volume - {}; mileage - {}&quot;,
            self.engine.volume(),
            self.engine.mileage()
        )?;
        writeln!(f, &quot;Transmission: {:?}&quot;, self.transmission)?;
        match self.gps_navigator {
            Some(_) =&gt; writeln!(f, &quot;GPS Navigator: Functional&quot;)?,
            None =&gt; writeln!(f, &quot;GPS Navigator: N/A&quot;)?,
        };
        Ok(())
    }
}</code></pre>
</figure>

#### []{#example-0--components-rs .anchor} **components.rs:** Product components

<figure class="code">
<pre class="code" lang="rust"><code>#[derive(Copy, Clone, Debug)]
pub enum CarType {
    CityCar,
    SportsCar,
    Suv,
}

#[derive(Debug)]
pub enum Transmission {
    SingleSpeed,
    Manual,
    Automatic,
    SemiAutomatic,
}

pub struct Engine {
    volume: f64,
    mileage: f64,
    started: bool,
}

impl Engine {
    pub fn new(volume: f64, mileage: f64) -&gt; Self {
        Self {
            volume,
            mileage,
            started: false,
        }
    }

    pub fn on(&amp;mut self) {
        self.started = true;
    }

    pub fn off(&amp;mut self) {
        self.started = false;
    }

    pub fn started(&amp;self) -&gt; bool {
        self.started
    }

    pub fn volume(&amp;self) -&gt; f64 {
        self.volume
    }

    pub fn mileage(&amp;self) -&gt; f64 {
        self.mileage
    }

    pub fn go(&amp;mut self, mileage: f64) {
        if self.started() {
            self.mileage += mileage;
        } else {
            println!(&quot;Cannot go(), you must start engine first!&quot;);
        }
    }
}

pub struct GpsNavigator {
    route: String,
}

impl GpsNavigator {
    pub fn new() -&gt; Self {
        Self::from_route(
            &quot;221b, Baker Street, London  to Scotland Yard, 8-10 Broadway, London&quot;.into(),
        )
    }

    pub fn from_route(route: String) -&gt; Self {
        Self { route }
    }

    pub fn route(&amp;self) -&gt; &amp;String {
        &amp;self.route
    }
}</code></pre>
</figure>

#### []{#example-0--director-rs .anchor} **director.rs:** Directors

<figure class="code">
<pre class="code" lang="rust"><code>use crate::{
    builders::Builder,
    components::{CarType, Engine, GpsNavigator, Transmission},
};

/// Director knows how to build a car.
///
/// However, a builder can build a car manual instead of an actual car,
/// everything depends on the concrete builder.
pub struct Director;

impl Director {
    pub fn construct_sports_car(builder: &amp;mut impl Builder) {
        builder.set_car_type(CarType::SportsCar);
        builder.set_seats(2);
        builder.set_engine(Engine::new(3.0, 0.0));
        builder.set_transmission(Transmission::SemiAutomatic);
        builder.set_gps_navigator(GpsNavigator::new());
    }

    pub fn construct_city_car(builder: &amp;mut impl Builder) {
        builder.set_car_type(CarType::CityCar);
        builder.set_seats(2);
        builder.set_engine(Engine::new(1.2, 0.0));
        builder.set_transmission(Transmission::Automatic);
        builder.set_gps_navigator(GpsNavigator::new());
    }

    pub fn construct_suv(builder: &amp;mut impl Builder) {
        builder.set_car_type(CarType::Suv);
        builder.set_seats(4);
        builder.set_engine(Engine::new(2.5, 0.0));
        builder.set_transmission(Transmission::Manual);
        builder.set_gps_navigator(GpsNavigator::new());
    }
}</code></pre>
</figure>

#### []{#example-0--main-rs .anchor} **main.rs:** Client code

<figure class="code">
<pre class="code" lang="rust"><code>#![allow(unused)]

mod builders;
mod cars;
mod components;
mod director;

use builders::{Builder, CarBuilder, CarManualBuilder};
use cars::{Car, Manual};
use director::Director;

fn main() {
    let mut car_builder = CarBuilder::default();

    // Director gets the concrete builder object from the client
    // (application code). That&#39;s because application knows better which
    // builder to use to get a specific product.
    Director::construct_sports_car(&amp;mut car_builder);

    // The final product is often retrieved from a builder object, since
    // Director is not aware and not dependent on concrete builders and
    // products.
    let car: Car = car_builder.build();
    println!(&quot;Car built: {:?}\n&quot;, car.car_type());

    let mut manual_builder = CarManualBuilder::default();

    // Director may know several building recipes.
    Director::construct_city_car(&amp;mut manual_builder);

    // The final car manual.
    let manual: Manual = manual_builder.build();
    println!(&quot;Car manual built:\n{}&quot;, manual);
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>Car built: SportsCar

Car manual built:
Type of car: CityCar
Count of seats: 2
Engine: volume - 1.2; mileage - 0
Transmission: Automatic
GPS Navigator: Functional</code></pre>
</figure>

::: next
#### Читаем дальше

[Фабричный метод на Rust []{.fa
.fa-arrow-right}](../../factory-method/rust/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Абстрактная фабрика на
Rust](../../abstract-factory/rust/example.html){.btn .btn-default
rel="prev"}
:::

## **Строитель** на других языках программирования

[![Строитель на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Строитель на C#"){.prog-lang-link}
[![Строитель на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Строитель на C++"){.prog-lang-link}
[![Строитель на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Строитель на Go"){.prog-lang-link}
[![Строитель на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Строитель на Java"){.prog-lang-link}
[![Строитель на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Строитель на PHP"){.prog-lang-link}
[![Строитель на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Строитель на Python"){.prog-lang-link}
[![Строитель на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Строитель на Ruby"){.prog-lang-link}
[![Строитель на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Строитель на Swift"){.prog-lang-link}
[![Строитель на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Строитель на TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ru" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архив с примерами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Купи книгу **Погружение в Паттерны** и получи архив с десятками
детальных примеров, которые можно открывать прямо в IDE.

[ Узнать больше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
