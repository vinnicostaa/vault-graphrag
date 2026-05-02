:::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Monteur](../../builder.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Monteur](../../../../images/patterns/cards/builder-mini6d24.png?id=19b95fd05e6469679752c0554b116815){srcset="/images/patterns/cards/builder-mini-2x.png?id=de6d0938678b86903a1426dddfdd13bf 2x"}
:::

# **Monteur** en Java {#monteur-en-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
Le **Monteur** est un patron de conception de création qui permet de
construire des objets complexes étape par étape.

Le monteur n'est pas comme les autres patrons de création : les produits
n'ont pas besoin d'avoir une interface commune. Il est ainsi possible de
créer différents produits en utilisant le même procédé de fabrication.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Monteur ](../../builder.html){.btn
.btn-lg .btn-primary}
:::

:::::::::::::::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Construction d'une voiture étape par étape](#example-0)
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

**Complexité :** []{.fa-stars}

**Popularité :** []{.fa-stars}

**Exemples d'utilisation :** Le monteur est très connu dans le monde du
Java. Il se révèle très utile lorsque vous devez créer un objet
possédant de nombreuses configurations possibles.

Il est très largement utilisé dans les bibliothèques principales de
Java :

-   [`java.lang.StringBuilder#append()`](http://docs.oracle.com/javase/8/docs/api/java/lang/StringBuilder.html#append-boolean-)
    (`unsynchronized`)
-   [`java.lang.StringBuffer#append()`](http://docs.oracle.com/javase/8/docs/api/java/lang/StringBuffer.html#append-boolean-)
    (`synchronized`)
-   [`java.nio.ByteBuffer#put()`](http://docs.oracle.com/javase/8/docs/api/java/nio/ByteBuffer.html#put-byte-)
    (et aussi dans
    [`CharBuffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/CharBuffer.html#put-char-),
    [`ShortBuffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/ShortBuffer.html#put-short-),
    [`IntBuffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/IntBuffer.html#put-int-),
    [`LongBuffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/LongBuffer.html#put-long-),
    [`FloatBuffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/FloatBuffer.html#put-float-)
    et
    [`DoubleBuffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/DoubleBuffer.html#put-double-))
-   [`javax.swing.GroupLayout.Group#addComponent()`](http://docs.oracle.com/javase/8/docs/api/javax/swing/GroupLayout.Group.html#addComponent-java.awt.Component-)
-   Toutes les implémentations de
    [`java.lang.Appendable`](http://docs.oracle.com/javase/8/docs/api/java/lang/Appendable.html)

**Identification :** Le monteur peut être identifié à l'intérieur d'une
classe qui n'a qu'une seule méthode de création et plusieurs méthodes
permettant de configurer l'objet qui en résulte. Les méthodes du monteur
prennent souvent en charge le chaînage (par exemple,
`someBuilder.setValueA(1).setValueB(2).create()`).
:::::::::::::::::::::::::::::::::::::

[]{#example-0}

## Construction d'une voiture étape par étape {#example-0-title}

Dans cet exemple, le patron monteur permet de construire différents
modèles de voitures étape par étape.

On peut également y voir comment le monteur peut créer des produits de
différents types (manuel d'une voiture) en utilisant les mêmes étapes de
construction.

Le directeur contrôle l'ordre de la construction. Il sait exactement
quelles étapes appeler afin de produire un modèle de voiture précis. Il
manipule les monteurs uniquement à travers leur interface commune, ce
qui permet de passer différents types de monteurs au directeur.

L'objet monteur récupère le résultat final, car le directeur ne connait
pas le type du produit construit. Seul l'objet monteur sait exactement
ce qui est en train d'être monté.

### []{#example-0--builders .anchor} **builders** {#builders .example-title}

#### []{#example-0--builders-Builder-java .anchor} **builders/Builder.java:** Interface monteur commune

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example.builders;

import refactoring_guru.builder.example.cars.CarType;
import refactoring_guru.builder.example.components.Engine;
import refactoring_guru.builder.example.components.GPSNavigator;
import refactoring_guru.builder.example.components.Transmission;
import refactoring_guru.builder.example.components.TripComputer;

/**
 * Builder interface defines all possible ways to configure a product.
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

#### []{#example-0--builders-CarBuilder-java .anchor} **builders/CarBuilder.java:** Monteur d'une voiture

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example.builders;

import refactoring_guru.builder.example.cars.Car;
import refactoring_guru.builder.example.cars.CarType;
import refactoring_guru.builder.example.components.Engine;
import refactoring_guru.builder.example.components.GPSNavigator;
import refactoring_guru.builder.example.components.Transmission;
import refactoring_guru.builder.example.components.TripComputer;

/**
 * Concrete builders implement steps defined in the common interface.
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

#### []{#example-0--builders-CarManualBuilder-java .anchor} **builders/CarManualBuilder.java:** Monteur d'un manuel utilisateur

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example.builders;

import refactoring_guru.builder.example.cars.Manual;
import refactoring_guru.builder.example.cars.CarType;
import refactoring_guru.builder.example.components.Engine;
import refactoring_guru.builder.example.components.GPSNavigator;
import refactoring_guru.builder.example.components.Transmission;
import refactoring_guru.builder.example.components.TripComputer;

/**
 * Unlike other creational patterns, Builder can construct unrelated products,
 * which don&#39;t have the common interface.
 *
 * In this case we build a user manual for a car, using the same steps as we
 * built a car. This allows to produce manuals for specific car models,
 * configured with different features.
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

#### []{#example-0--cars-Car-java .anchor} **cars/Car.java:** Voiture (produit)

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example.cars;

import refactoring_guru.builder.example.components.Engine;
import refactoring_guru.builder.example.components.GPSNavigator;
import refactoring_guru.builder.example.components.Transmission;
import refactoring_guru.builder.example.components.TripComputer;

/**
 * Car is a product class.
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

#### []{#example-0--cars-Manual-java .anchor} **cars/Manual.java:** Manuel (produit)

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example.cars;

import refactoring_guru.builder.example.components.Engine;
import refactoring_guru.builder.example.components.GPSNavigator;
import refactoring_guru.builder.example.components.Transmission;
import refactoring_guru.builder.example.components.TripComputer;

/**
 * Car manual is another product. Note that it does not have the same ancestor
 * as a Car. They are not related.
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

#### []{#example-0--components-Engine-java .anchor} **components/Engine.java:** Caractéristique 1 du produit

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example.components;

/**
 * Just another feature of a car.
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

#### []{#example-0--components-GPSNavigator-java .anchor} **components/GPSNavigator.java:** Caractéristique 2 du produit

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example.components;

/**
 * Just another feature of a car.
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

#### []{#example-0--components-Transmission-java .anchor} **components/Transmission.java:** Caractéristique 3 du produit

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example.components;

/**
 * Just another feature of a car.
 */
public enum Transmission {
    SINGLE_SPEED, MANUAL, AUTOMATIC, SEMI_AUTOMATIC
}</code></pre>
</figure>

#### []{#example-0--components-TripComputer-java .anchor} **components/TripComputer.java:** Caractéristique 4 du produit

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example.components;

import refactoring_guru.builder.example.cars.Car;

/**
 * Just another feature of a car.
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

#### []{#example-0--director-Director-java .anchor} **director/Director.java:** Le directeur contrôle les monteurs

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example.director;

import refactoring_guru.builder.example.builders.Builder;
import refactoring_guru.builder.example.cars.CarType;
import refactoring_guru.builder.example.components.Engine;
import refactoring_guru.builder.example.components.GPSNavigator;
import refactoring_guru.builder.example.components.Transmission;
import refactoring_guru.builder.example.components.TripComputer;

/**
 * Director defines the order of building steps. It works with a builder object
 * through common Builder interface. Therefore it may not know what product is
 * being built.
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

#### []{#example-0--Demo-java .anchor} **Demo.java:** Code client

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example;

import refactoring_guru.builder.example.builders.CarBuilder;
import refactoring_guru.builder.example.builders.CarManualBuilder;
import refactoring_guru.builder.example.cars.Car;
import refactoring_guru.builder.example.cars.Manual;
import refactoring_guru.builder.example.director.Director;

/**
 * Demo class. Everything comes together here.
 */
public class Demo {

    public static void main(String[] args) {
        Director director = new Director();

        // Director gets the concrete builder object from the client
        // (application code). That&#39;s because application knows better which
        // builder to use to get a specific product.
        CarBuilder builder = new CarBuilder();
        director.constructSportsCar(builder);

        // The final product is often retrieved from a builder object, since
        // Director is not aware and not dependent on concrete builders and
        // products.
        Car car = builder.getResult();
        System.out.println(&quot;Car built:\n&quot; + car.getCarType());


        CarManualBuilder manualBuilder = new CarManualBuilder();

        // Director may know several building recipes.
        director.constructSportsCar(manualBuilder);
        Manual carManual = manualBuilder.getResult();
        System.out.println(&quot;\nCar manual built:\n&quot; + carManual.print());
    }

}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Résultat de l'exécution

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
#### Suivant

[Fabrique en Java []{.fa
.fa-arrow-right}](../../factory-method/java/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Fabrique abstraite en
Java](../../abstract-factory/java/example.html){.btn .btn-default
rel="prev"}
:::

## **Monteur** dans les autres langues

[![Monteur en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Monteur en C#"){.prog-lang-link}
[![Monteur en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Monteur en C++"){.prog-lang-link}
[![Monteur en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Monteur en Go"){.prog-lang-link}
[![Monteur en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Monteur en PHP"){.prog-lang-link}
[![Monteur en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Monteur en Python"){.prog-lang-link}
[![Monteur en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Monteur en Ruby"){.prog-lang-link}
[![Monteur en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Monteur en Rust"){.prog-lang-link}
[![Monteur en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Monteur en Swift"){.prog-lang-link}
[![Monteur en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Monteur en TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-fr" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive avec exemples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Achetez l'ebook **Plongée au cœur des patrons de conception** et obtenez
l'accès à une archive comprenant des dizaines d'exemples détaillés qui
peuvent être ouverts dans votre environnement de développement.

[ En savoir plus...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::
