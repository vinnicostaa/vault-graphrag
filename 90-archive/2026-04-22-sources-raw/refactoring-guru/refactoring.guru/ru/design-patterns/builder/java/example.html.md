:::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Строитель](../../builder.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Строитель](../../../../images/patterns/cards/builder-mini6d24.png?id=19b95fd05e6469679752c0554b116815){srcset="/images/patterns/cards/builder-mini-2x.png?id=de6d0938678b86903a1426dddfdd13bf 2x"}
:::

# **Строитель** на Java {#строитель-на-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::::::::::::::::: pattern-example-body
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

:::::::::::::::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Пошаговое производство автомобилей](#example-0)
:::

::: en-file
 builders
:::

:::: en-inside
::: en-file
  [Builder](#example-0--builders-Builder-java)
:::
::::

:::: en-inside
::: en-file
  [Car­Builder](#example-0--builders-CarBuilder-java)
:::
::::

:::: en-inside
::: en-file
  [Car­Manual­Builder](#example-0--builders-CarManualBuilder-java)
:::
::::

::: en-file
 cars
:::

:::: en-inside
::: en-file
  [Car](#example-0--cars-Car-java)
:::
::::

:::: en-inside
::: en-file
  [Manual](#example-0--cars-Manual-java)
:::
::::

:::: en-inside
::: en-file
  [Car­Type](#example-0--cars-CarType-java)
:::
::::

::: en-file
 components
:::

:::: en-inside
::: en-file
  [Engine](#example-0--components-Engine-java)
:::
::::

:::: en-inside
::: en-file
  [GPSNavigator](#example-0--components-GPSNavigator-java)
:::
::::

:::: en-inside
::: en-file
  [Transmission](#example-0--components-Transmission-java)
:::
::::

:::: en-inside
::: en-file
  [Trip­Computer](#example-0--components-TripComputer-java)
:::
::::

::: en-file
 director
:::

:::: en-inside
::: en-file
  [Director](#example-0--director-Director-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-txt)
:::
::::::::::::::::::::::::::::::::::

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Паттерн можно часто встретить в Java-коде, особенно
там, где требуется пошаговое создание продуктов или конфигурация сложных
объектов.

Паттерн широко используется в стандартных библиотеках Java:

-   [`java.lang.StringBuilder#append()`](http://docs.oracle.com/javase/8/docs/api/java/lang/StringBuilder.html#append-boolean-)
    (`unsynchronized`)
-   [`java.lang.StringBuffer#append()`](http://docs.oracle.com/javase/8/docs/api/java/lang/StringBuffer.html#append-boolean-)
    (`synchronized`)
-   [`java.nio.ByteBuffer#put()`](http://docs.oracle.com/javase/8/docs/api/java/nio/ByteBuffer.html#put-byte-)
    (также в
    [`CharBuffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/CharBuffer.html#put-char-),
    [`ShortBuffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/ShortBuffer.html#put-short-),
    [`IntBuffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/IntBuffer.html#put-int-),
    [`LongBuffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/LongBuffer.html#put-long-),
    [`FloatBuffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/FloatBuffer.html#put-float-)
    и
    [`DoubleBuffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/DoubleBuffer.html#put-double-))
-   [`javax.swing.GroupLayout.Group#addComponent()`](http://docs.oracle.com/javase/8/docs/api/javax/swing/GroupLayout.Group.html#addComponent-java.awt.Component-)
-   Все реализации
    [`java.lang.Appendable`](http://docs.oracle.com/javase/8/docs/api/java/lang/Appendable.html)

**Признаки применения паттерна:** Строителя можно узнать в классе,
который имеет один создающий метод и несколько методов настройки
создаваемого продукта. Обычно, методы настройки вызывают для удобства
цепочкой (например, `someBuilder.setValueA(1).setValueB(2).create()`).
:::::::::::::::::::::::::::::::::::::

[]{#example-0}

## Пошаговое производство автомобилей {#example-0-title}

В этом примере Строитель позволяет пошагово конструировать различные
конфигурации автомобилей.

Кроме того, здесь показано как с помощью Строителя может создавать
другой продукт на основе тех же шагов строительства. Для этого мы имеем
два конкретных строителя --- `CarBuilder` и `CarManualBuilder`.

Порядок строительства продуктов заключён в Директоре. Он знает какие
шаги строителей нужно вызвать, чтобы получить ту или иную конфигурацию
продукта. Он работает со строителями только через общий интерфейс, что
позволяет взаимозаменять объекты строителей для получения разных
продуктов.

### []{#example-0--builders .anchor} **builders** {#builders .example-title}

#### []{#example-0--builders-Builder-java .anchor} **builders/Builder.java:** Общий интерфейс строителей

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example.builders;

import refactoring_guru.builder.example.cars.CarType;
import refactoring_guru.builder.example.components.Engine;
import refactoring_guru.builder.example.components.GPSNavigator;
import refactoring_guru.builder.example.components.Transmission;
import refactoring_guru.builder.example.components.TripComputer;

/**
 * Интерфейс Строителя объявляет все возможные этапы и шаги конфигурации
 * продукта.
 */
public interface Builder {
    void setCarType(CarType type);
    void setSeats(int seats);
    void setEngine(Engine engine);
    void setTransmission(Transmission transmission);
    void setTripComputer(TripComputer tripComputer);
    void setGPSNavigator(GPSNavigator gpsNavigator);
}</code></pre>
</figure>

#### []{#example-0--builders-CarBuilder-java .anchor} **builders/CarBuilder.java:** Строитель автомобилей

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example.builders;

import refactoring_guru.builder.example.cars.Car;
import refactoring_guru.builder.example.cars.CarType;
import refactoring_guru.builder.example.components.Engine;
import refactoring_guru.builder.example.components.GPSNavigator;
import refactoring_guru.builder.example.components.Transmission;
import refactoring_guru.builder.example.components.TripComputer;

/**
 * Конкретные строители реализуют шаги, объявленные в общем интерфейсе.
 */
public class CarBuilder implements Builder {
    private CarType type;
    private int seats;
    private Engine engine;
    private Transmission transmission;
    private TripComputer tripComputer;
    private GPSNavigator gpsNavigator;

    public void setCarType(CarType type) {
        this.type = type;
    }

    @Override
    public void setSeats(int seats) {
        this.seats = seats;
    }

    @Override
    public void setEngine(Engine engine) {
        this.engine = engine;
    }

    @Override
    public void setTransmission(Transmission transmission) {
        this.transmission = transmission;
    }

    @Override
    public void setTripComputer(TripComputer tripComputer) {
        this.tripComputer = tripComputer;
    }

    @Override
    public void setGPSNavigator(GPSNavigator gpsNavigator) {
        this.gpsNavigator = gpsNavigator;
    }

    public Car getResult() {
        return new Car(type, seats, engine, transmission, tripComputer, gpsNavigator);
    }
}</code></pre>
</figure>

#### []{#example-0--builders-CarManualBuilder-java .anchor} **builders/CarManualBuilder.java:** Строитель руководств пользователя

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example.builders;

import refactoring_guru.builder.example.cars.Manual;
import refactoring_guru.builder.example.cars.CarType;
import refactoring_guru.builder.example.components.Engine;
import refactoring_guru.builder.example.components.GPSNavigator;
import refactoring_guru.builder.example.components.Transmission;
import refactoring_guru.builder.example.components.TripComputer;

/**
 * В отличие от других создающих паттернов, Строители могут создавать совершенно
 * разные продукты, не имеющие общего интерфейса.
 *
 * В данном случае мы производим руководство пользователя автомобиля с помощью
 * тех же шагов, что и сами автомобили. Это устройство позволит создавать
 * руководства под конкретные модели автомобилей, содержащие те или иные фичи.
 */
public class CarManualBuilder implements Builder{
    private CarType type;
    private int seats;
    private Engine engine;
    private Transmission transmission;
    private TripComputer tripComputer;
    private GPSNavigator gpsNavigator;

    @Override
    public void setCarType(CarType type) {
        this.type = type;
    }

    @Override
    public void setSeats(int seats) {
        this.seats = seats;
    }

    @Override
    public void setEngine(Engine engine) {
        this.engine = engine;
    }

    @Override
    public void setTransmission(Transmission transmission) {
        this.transmission = transmission;
    }

    @Override
    public void setTripComputer(TripComputer tripComputer) {
        this.tripComputer = tripComputer;
    }

    @Override
    public void setGPSNavigator(GPSNavigator gpsNavigator) {
        this.gpsNavigator = gpsNavigator;
    }

    public Manual getResult() {
        return new Manual(type, seats, engine, transmission, tripComputer, gpsNavigator);
    }
}</code></pre>
</figure>

### []{#example-0--cars .anchor} **cars** {#cars .example-title}

#### []{#example-0--cars-Car-java .anchor} **cars/Car.java:** Продукт-автомобиль

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example.cars;

import refactoring_guru.builder.example.components.Engine;
import refactoring_guru.builder.example.components.GPSNavigator;
import refactoring_guru.builder.example.components.Transmission;
import refactoring_guru.builder.example.components.TripComputer;

/**
 * Автомобиль — это класс продукта.
 */
public class Car {
    private final CarType carType;
    private final int seats;
    private final Engine engine;
    private final Transmission transmission;
    private final TripComputer tripComputer;
    private final GPSNavigator gpsNavigator;
    private double fuel = 0;

    public Car(CarType carType, int seats, Engine engine, Transmission transmission,
               TripComputer tripComputer, GPSNavigator gpsNavigator) {
        this.carType = carType;
        this.seats = seats;
        this.engine = engine;
        this.transmission = transmission;
        this.tripComputer = tripComputer;
        if (this.tripComputer != null) {
            this.tripComputer.setCar(this);
        }
        this.gpsNavigator = gpsNavigator;
    }

    public CarType getCarType() {
        return carType;
    }

    public double getFuel() {
        return fuel;
    }

    public void setFuel(double fuel) {
        this.fuel = fuel;
    }

    public int getSeats() {
        return seats;
    }

    public Engine getEngine() {
        return engine;
    }

    public Transmission getTransmission() {
        return transmission;
    }

    public TripComputer getTripComputer() {
        return tripComputer;
    }

    public GPSNavigator getGpsNavigator() {
        return gpsNavigator;
    }
}</code></pre>
</figure>

#### []{#example-0--cars-Manual-java .anchor} **cars/Manual.java:** Продукт-руководство

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example.cars;

import refactoring_guru.builder.example.components.Engine;
import refactoring_guru.builder.example.components.GPSNavigator;
import refactoring_guru.builder.example.components.Transmission;
import refactoring_guru.builder.example.components.TripComputer;

/**
 * Руководство автомобиля — это второй продукт. Заметьте, что руководство и сам
 * автомобиль не имеют общего родительского класса. По сути, они независимы.
 */
public class Manual {
    private final CarType carType;
    private final int seats;
    private final Engine engine;
    private final Transmission transmission;
    private final TripComputer tripComputer;
    private final GPSNavigator gpsNavigator;

    public Manual(CarType carType, int seats, Engine engine, Transmission transmission,
                  TripComputer tripComputer, GPSNavigator gpsNavigator) {
        this.carType = carType;
        this.seats = seats;
        this.engine = engine;
        this.transmission = transmission;
        this.tripComputer = tripComputer;
        this.gpsNavigator = gpsNavigator;
    }

    public String print() {
        String info = &quot;&quot;;
        info += &quot;Type of car: &quot; + carType + &quot;\n&quot;;
        info += &quot;Count of seats: &quot; + seats + &quot;\n&quot;;
        info += &quot;Engine: volume - &quot; + engine.getVolume() + &quot;; mileage - &quot; + engine.getMileage() + &quot;\n&quot;;
        info += &quot;Transmission: &quot; + transmission + &quot;\n&quot;;
        if (this.tripComputer != null) {
            info += &quot;Trip Computer: Functional&quot; + &quot;\n&quot;;
        } else {
            info += &quot;Trip Computer: N/A&quot; + &quot;\n&quot;;
        }
        if (this.gpsNavigator != null) {
            info += &quot;GPS Navigator: Functional&quot; + &quot;\n&quot;;
        } else {
            info += &quot;GPS Navigator: N/A&quot; + &quot;\n&quot;;
        }
        return info;
    }
}</code></pre>
</figure>

#### []{#example-0--cars-CarType-java .anchor} **cars/CarType.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example.cars;

public enum CarType {
    CITY_CAR, SPORTS_CAR, SUV
}</code></pre>
</figure>

### []{#example-0--components .anchor} **components** {#components .example-title}

#### []{#example-0--components-Engine-java .anchor} **components/Engine.java:** Разные фичи продуктов

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example.components;

/**
 * Одна из фишек автомобиля.
 */
public class Engine {
    private final double volume;
    private double mileage;
    private boolean started;

    public Engine(double volume, double mileage) {
        this.volume = volume;
        this.mileage = mileage;
    }

    public void on() {
        started = true;
    }

    public void off() {
        started = false;
    }

    public boolean isStarted() {
        return started;
    }

    public void go(double mileage) {
        if (started) {
            this.mileage += mileage;
        } else {
            System.err.println(&quot;Cannot go(), you must start engine first!&quot;);
        }
    }

    public double getVolume() {
        return volume;
    }

    public double getMileage() {
        return mileage;
    }
}</code></pre>
</figure>

#### []{#example-0--components-GPSNavigator-java .anchor} **components/GPSNavigator.java:** Разные фичи продуктов

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example.components;

/**
 * Одна из фишек автомобиля.
 */
public class GPSNavigator {
    private String route;

    public GPSNavigator() {
        this.route = &quot;221b, Baker Street, London  to Scotland Yard, 8-10 Broadway, London&quot;;
    }

    public GPSNavigator(String manualRoute) {
        this.route = manualRoute;
    }

    public String getRoute() {
        return route;
    }
}</code></pre>
</figure>

#### []{#example-0--components-Transmission-java .anchor} **components/Transmission.java:** Разные фичи продуктов

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example.components;

/**
 * Одна из фишек автомобиля.
 */
public enum Transmission {
    SINGLE_SPEED, MANUAL, AUTOMATIC, SEMI_AUTOMATIC
}</code></pre>
</figure>

#### []{#example-0--components-TripComputer-java .anchor} **components/TripComputer.java:** Разные фичи продуктов

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example.components;

import refactoring_guru.builder.example.cars.Car;

/**
 * Одна из фишек автомобиля.
 */
public class TripComputer {

    private Car car;

    public void setCar(Car car) {
        this.car = car;
    }

    public void showFuelLevel() {
        System.out.println(&quot;Fuel level: &quot; + car.getFuel());
    }

    public void showStatus() {
        if (this.car.getEngine().isStarted()) {
            System.out.println(&quot;Car is started&quot;);
        } else {
            System.out.println(&quot;Car isn&#39;t started&quot;);
        }
    }
}</code></pre>
</figure>

### []{#example-0--director .anchor} **director** {#director .example-title}

#### []{#example-0--director-Director-java .anchor} **director/Director.java:** Директор управляет строителями

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example.director;

import refactoring_guru.builder.example.builders.Builder;
import refactoring_guru.builder.example.cars.CarType;
import refactoring_guru.builder.example.components.Engine;
import refactoring_guru.builder.example.components.GPSNavigator;
import refactoring_guru.builder.example.components.Transmission;
import refactoring_guru.builder.example.components.TripComputer;

/**
 * Директор знает в какой последовательности заставлять работать строителя. Он
 * работает с ним через общий интерфейс Строителя. Из-за этого, он может не
 * знать какой конкретно продукт сейчас строится.
 */
public class Director {

    public void constructSportsCar(Builder builder) {
        builder.setCarType(CarType.SPORTS_CAR);
        builder.setSeats(2);
        builder.setEngine(new Engine(3.0, 0));
        builder.setTransmission(Transmission.SEMI_AUTOMATIC);
        builder.setTripComputer(new TripComputer());
        builder.setGPSNavigator(new GPSNavigator());
    }

    public void constructCityCar(Builder builder) {
        builder.setCarType(CarType.CITY_CAR);
        builder.setSeats(2);
        builder.setEngine(new Engine(1.2, 0));
        builder.setTransmission(Transmission.AUTOMATIC);
        builder.setTripComputer(new TripComputer());
        builder.setGPSNavigator(new GPSNavigator());
    }

    public void constructSUV(Builder builder) {
        builder.setCarType(CarType.SUV);
        builder.setSeats(4);
        builder.setEngine(new Engine(2.5, 0));
        builder.setTransmission(Transmission.MANUAL);
        builder.setGPSNavigator(new GPSNavigator());
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** Клиентский код

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example;

import refactoring_guru.builder.example.builders.CarBuilder;
import refactoring_guru.builder.example.builders.CarManualBuilder;
import refactoring_guru.builder.example.cars.Car;
import refactoring_guru.builder.example.cars.Manual;
import refactoring_guru.builder.example.director.Director;

/**
 * Демо-класс. Здесь всё сводится воедино.
 */
public class Demo {

    public static void main(String[] args) {
        Director director = new Director();

        // Директор получает объект конкретного строителя от клиента
        // (приложения). Приложение само знает какой строитель использовать,
        // чтобы получить нужный продукт.
        CarBuilder builder = new CarBuilder();
        director.constructSportsCar(builder);

        // Готовый продукт возвращает строитель, так как Директор чаще всего не
        // знает и не зависит от конкретных классов строителей и продуктов.
        Car car = builder.getResult();
        System.out.println(&quot;Car built:\n&quot; + car.getCarType());


        CarManualBuilder manualBuilder = new CarManualBuilder();

        // Директор может знать больше одного рецепта строительства.
        director.constructSportsCar(manualBuilder);
        Manual carManual = manualBuilder.getResult();
        System.out.println(&quot;\nCar manual built:\n&quot; + carManual.print());
    }

}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Результаты выполнения

<figure class="code">
<pre class="code" lang="output"><code>Car built:
SPORTS_CAR

Car manual built:
Type of car: SPORTS_CAR
Count of seats: 2
Engine: volume - 3.0; mileage - 0.0
Transmission: SEMI_AUTOMATIC
Trip Computer: Functional
GPS Navigator: Functional</code></pre>
</figure>

::: next
#### Читаем дальше

[Фабричный метод на Java []{.fa
.fa-arrow-right}](../../factory-method/java/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Абстрактная фабрика на
Java](../../abstract-factory/java/example.html){.btn .btn-default
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
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Строитель на Rust"){.prog-lang-link}
[![Строитель на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Строитель на Swift"){.prog-lang-link}
[![Строитель на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Строитель на TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::
