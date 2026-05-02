:::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[빌더](../../builder.html){.type} / [자바](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![빌더](../../../../images/patterns/cards/builder-mini6d24.png?id=19b95fd05e6469679752c0554b116815){srcset="/images/patterns/cards/builder-mini-2x.png?id=de6d0938678b86903a1426dddfdd13bf 2x"}
:::

# 자바로 작성된 **빌더** {#자바로-작성된-빌더 .pattern-example-title-block-title}
::::

::::::::::::::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**빌더**는 복잡한 객체들을 단계별로 생성할 수 있도록 하는 생성 디자인
패턴입니다.

다른 생성 패턴과 달리 빌더 패턴은 제품들에 공통 인터페이스를 요구하지
않습니다. 이를 통해 같은 생성공정을 사용하여 다양한 제품을 생산할 수
있습니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 빌더에 대하여 더 자세히 알아보세요 ](../../builder.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
내비게이션
:::

::: en-intro
 [소개](#)
:::

::: en-intro
 [단계별 자동차 생성](#example-0)
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

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 예시들:** 빌더 패턴은 자바 개발자들에게 잘 알려진 패턴이며,
가능한 설정 옵션이 많은 객체를 만들어야 할 때 특히 유용합니다.

빌더는 자바 코어 라이브러리에서 널리 사용됩니다.

-   [`java.lang.StringBuilder#append()`](http://docs.oracle.com/javase/8/docs/api/java/lang/StringBuilder.html#append-boolean-)
    (`unsynchronized`)
-   [`java.lang.StringBuffer#append()`](http://docs.oracle.com/javase/8/docs/api/java/lang/StringBuffer.html#append-boolean-)
    (`synchronized`)
-   [`java.nio.ByteBuffer#put()`](http://docs.oracle.com/javase/8/docs/api/java/nio/ByteBuffer.html#put-byte-)
    (또 다음에도 사용됩니다:
    [`Char­Buffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/CharBuffer.html#put-char-),
    [`Short­Buffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/ShortBuffer.html#put-short-),
    [`Int­Buffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/IntBuffer.html#put-int-),
    [`Long­Buffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/LongBuffer.html#put-long-),
    [`Float­Buffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/FloatBuffer.html#put-float-)
    and
    [`Double­Buffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/DoubleBuffer.html#put-double-))
-   [`javax.swing.GroupLayout.Group#addComponent()`](http://docs.oracle.com/javase/8/docs/api/javax/swing/GroupLayout.Group.html#addComponent-java.awt.Component-)
-   [`java.lang.Appendable`](http://docs.oracle.com/javase/8/docs/api/java/lang/Appendable.html)의
    모든 구현

**식별법:** 빌더 패턴은 하나의 생성 메서드와 결과 객체를 설정하기 위한
여러 메서드가 있는 클래스가 있습니다. 또 빌더 메서드들은 자주 사슬식
연결을 지원합니다 (예:
`someBuilder.​setValueA(1).​setValueB(2).​create()`{style="white-space: normal; display:inline;"}).
:::::::::::::::::::::::::::::::::::::

[]{#example-0}

## 단계별 자동차 생성 {#example-0-title}

이 예시에서의 빌더 패턴은 다양한 자동차 모델을 단계별로 생성할 수
있습니다.

이 예시는 또 빌더가 어떻게 같은 생성 단계들을 사용하여 다양한 종류의
제품들​(예: 자동차 매뉴얼)​을 생성하는 방법을 보여줍니다.

디렉터는 생성 순서를 통제하며 특정 자동차 모델을 생성하기 위해 어떤 특정
생성 단계를 호출해야 하는지를 알고 있습니다. 또 디렉터는 빌더들의 공통
인터페이스를 통해서만 빌더들과 함께 작동하며 이는 다양한 유형의 빌더들을
디렉터에게 전달할 수 있도록 합니다.

최종 결과는 빌더 객체에서부터 가져옵니다. 왜냐하면 디렉터는 결과 제품의
유형을 알 수 없기 때문입니다. 빌더 객체만이 자신이 정확히 무엇을
생성하는지 압니다.

### []{#example-0--builders .anchor} **builders** {#builders .example-title}

#### []{#example-0--builders-Builder-java .anchor} **builders/Builder.java:** 공통 빌더 인터페이스

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

#### []{#example-0--builders-CarBuilder-java .anchor} **builders/CarBuilder.java:** 자동차의 빌더

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

#### []{#example-0--builders-CarManualBuilder-java .anchor} **builders/CarManualBuilder.java:** 자동차 매뉴얼의 빌더

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

#### []{#example-0--cars-Car-java .anchor} **cars/Car.java:** 자동차 제품

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

#### []{#example-0--cars-Manual-java .anchor} **cars/Manual.java:** 자동차 매뉴얼 제품

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

#### []{#example-0--components-Engine-java .anchor} **components/Engine.java:** 제품 특성 1

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

#### []{#example-0--components-GPSNavigator-java .anchor} **components/GPSNavigator.java:** 제품 특성 2

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

#### []{#example-0--components-Transmission-java .anchor} **components/Transmission.java:** 제품 특성 3

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.builder.example.components;

/**
 * Just another feature of a car.
 */
public enum Transmission {
    SINGLE_SPEED, MANUAL, AUTOMATIC, SEMI_AUTOMATIC
}</code></pre>
</figure>

#### []{#example-0--components-TripComputer-java .anchor} **components/TripComputer.java:** 제품 특성 4

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

#### []{#example-0--director-Director-java .anchor} **director/Director.java:** 디렉터는 빌더들을 제어합니다

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** 클라이언트 코드

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

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** 실행 결과

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
#### 다음을 보세요

[자바로 작성된 팩토리 메서드 []{.fa
.fa-arrow-right}](../../factory-method/java/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 자바로 작성된 추상
팩토리](../../abstract-factory/java/example.html){.btn .btn-default
rel="prev"}
:::

## 다른 언어로 작성된 **빌더**

[![C#으로 작성된
빌더](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 빌더"){.prog-lang-link}
[![C++로 작성된
빌더](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 빌더"){.prog-lang-link}
[![Go로 작성된
빌더](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 빌더"){.prog-lang-link}
[![PHP로 작성된
빌더](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 빌더"){.prog-lang-link}
[![파이썬으로 작성된
빌더](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 빌더"){.prog-lang-link}
[![루비로 작성된
빌더](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 빌더"){.prog-lang-link}
[![러스트로 작성된
빌더](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 빌더"){.prog-lang-link}
[![스위프트로 작성된
빌더](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 빌더"){.prog-lang-link}
[![타입스크립트로 작성된
빌더](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 빌더"){.prog-lang-link}
:::::::::::::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ko" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ 예시들을 포함한 저장소](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
전자책 **디자인 패턴에 뛰어들기**를 구매하면 IDE에서 바로 열 수 있는
수십 가지의 상세한 코드 예시들이 포함된 저장소를 사용할 수 있습니다.

[ 더 알아보기...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::
