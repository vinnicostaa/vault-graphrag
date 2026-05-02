:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Flyweight](../../flyweight.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Flyweight](../../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# **Flyweight** en Go {#flyweight-en-go .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Flyweight** es un patrón de diseño estructural que permite a los
programas soportar grandes cantidades de objetos manteniendo un bajo uso
de memoria.

El patrón lo logra compartiendo partes del estado del objeto entre
varios objetos. En otras palabras, el Flyweight ahorra memoria RAM
guardando en caché la misma información utilizada por distintos objetos.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Flyweight ](../../flyweight.html){.btn
.btn-lg .btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegación
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Ejemplo conceptual](#example-0)
:::

::: en-file
 [dress­Factory](#example-0--dressFactory-go)
:::

::: en-file
 [dress](#example-0--dress-go)
:::

::: en-file
 [terrorist­Dress](#example-0--terroristDress-go)
:::

::: en-file
 [counter­Terrorist­Dress](#example-0--counterTerroristDress-go)
:::

::: en-file
 [player](#example-0--player-go)
:::

::: en-file
 [game](#example-0--game-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::::::::
:::::::::::::::::

[]{#example-0}

## Ejemplo conceptual {#example-0-title}

En un juego de contraataque, el terrorista y el contraterrorista tienen
distintos tipos de vestimenta. Por simplificar, asumamos que terrorista
y contraterrorista tienen un tipo de vestimenta cada uno. El objeto
vestimenta está integrado en el objeto jugador, como se ve a
continuación.

A continuación se encuentra la estructura de un jugador. Podemos ver que
el objeto vestimenta está integrado en la estructura del jugador:

<figure class="code">
<pre class="code" lang="go"><code>type player struct {
    dress      dress
    playerType string // Puede ser T o CT
    lat        int
    long       int
}</code></pre>
</figure>

Digamos que hay 5 terroristas y 5 contraterroristas, de modo que hay un
total de 10 jugadores. Hay dos opciones en lo que respecta a la
vestimenta.

1.  Cada uno de los 10 objetos jugador crea un objeto vestimenta
    diferente y lo integra. Se creará un total de 10 objetos de
    vestimenta.

2.  Creamos dos objetos de vestimenta:

    -   Objeto único de vestimenta de terrorista: Lo compartirán 5
        terroristas.
    -   Objeto único de vestimenta de contraterrorista: Lo compartirán 5
        contraterroristas.

Como puedes ver en la solución 1, se crea un total de 10 objetos de
vestimenta, mientras que en la solución 2 solo se crean 2 objetos de
vestimenta. La segunda solución es la que empleamos en el patrón de
diseño Flyweight. Los dos objetos de vestimenta que creamos se denominan
objetos flyweight.

El patrón Flyweight extrae las partes comunes y crea objetos flyweight.
Estos objetos flyweight (vestimenta) pueden ser compartidos por muchos
objetos (jugador). Esto reduce drásticamente el número de objetos de
vestimenta y lo bueno es que, aunque crees más jugadores, será
suficiente con dos objetos de vestimenta.

En el patrón Flyweight almacenamos los objetos flyweight en el campo
mapa. Cuando se crean los otros objetos que comparten los objetos
flyweight, los objetos flyweight se extraen del mapa.

Veamos qué partes de este sistema serán estados intrínsecos y
extrínsecos:

-   *Estado intrínseco*: Vestimenta en el estado intrínseco, ya que
    puede ser compartido por varios objetos de terrorista y
    contraterrorista.

-   *Estado extrínseco*: La ubicación y el arma del jugador son un
    estado extrínseco, ya que son diferentes para cada objeto.

#### []{#example-0--dressFactory-go .anchor} **dressFactory.go:** Fábrica flyweight

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

const (
    //TerroristDressType terrorist dress type
    TerroristDressType = &quot;tDress&quot;
    //CounterTerrroristDressType terrorist dress type
    CounterTerrroristDressType = &quot;ctDress&quot;
)

var (
    dressFactorySingleInstance = &amp;DressFactory{
        dressMap: make(map[string]Dress),
    }
)

type DressFactory struct {
    dressMap map[string]Dress
}

func (d *DressFactory) getDressByType(dressType string) (Dress, error) {
    if d.dressMap[dressType] != nil {
        return d.dressMap[dressType], nil
    }

    if dressType == TerroristDressType {
        d.dressMap[dressType] = newTerroristDress()
        return d.dressMap[dressType], nil
    }
    if dressType == CounterTerrroristDressType {
        d.dressMap[dressType] = newCounterTerroristDress()
        return d.dressMap[dressType], nil
    }

    return nil, fmt.Errorf(&quot;Wrong dress type passed&quot;)
}

func getDressFactorySingleInstance() *DressFactory {
    return dressFactorySingleInstance
}</code></pre>
</figure>

#### []{#example-0--dress-go .anchor} **dress.go:** Interfaz flyweight

<figure class="code">
<pre class="code" lang="go"><code>package main

type Dress interface {
    getColor() string
}</code></pre>
</figure>

#### []{#example-0--terroristDress-go .anchor} **terroristDress.go:** Objeto concreto flyweight

<figure class="code">
<pre class="code" lang="go"><code>package main

type TerroristDress struct {
    color string
}

func (t *TerroristDress) getColor() string {
    return t.color
}

func newTerroristDress() *TerroristDress {
    return &amp;TerroristDress{color: &quot;red&quot;}
}</code></pre>
</figure>

#### []{#example-0--counterTerroristDress-go .anchor} **counterTerroristDress.go:** Objeto concreto flyweight

<figure class="code">
<pre class="code" lang="go"><code>package main

type CounterTerroristDress struct {
    color string
}

func (c *CounterTerroristDress) getColor() string {
    return c.color
}

func newCounterTerroristDress() *CounterTerroristDress {
    return &amp;CounterTerroristDress{color: &quot;green&quot;}
}</code></pre>
</figure>

#### []{#example-0--player-go .anchor} **player.go:** Contexto

<figure class="code">
<pre class="code" lang="go"><code>package main

type Player struct {
    dress      Dress
    playerType string
    lat        int
    long       int
}

func newPlayer(playerType, dressType string) *Player {
    dress, _ := getDressFactorySingleInstance().getDressByType(dressType)
    return &amp;Player{
        playerType: playerType,
        dress:      dress,
    }
}

func (p *Player) newLocation(lat, long int) {
    p.lat = lat
    p.long = long
}</code></pre>
</figure>

#### []{#example-0--game-go .anchor} **game.go**

<figure class="code">
<pre class="code" lang="go"><code>package main

type game struct {
    terrorists        []*Player
    counterTerrorists []*Player
}

func newGame() *game {
    return &amp;game{
        terrorists:        make([]*Player, 1),
        counterTerrorists: make([]*Player, 1),
    }
}

func (c *game) addTerrorist(dressType string) {
    player := newPlayer(&quot;T&quot;, dressType)
    c.terrorists = append(c.terrorists, player)
    return
}

func (c *game) addCounterTerrorist(dressType string) {
    player := newPlayer(&quot;CT&quot;, dressType)
    c.counterTerrorists = append(c.counterTerrorists, player)
    return
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Código cliente

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {
    game := newGame()

    //Add Terrorist
    game.addTerrorist(TerroristDressType)
    game.addTerrorist(TerroristDressType)
    game.addTerrorist(TerroristDressType)
    game.addTerrorist(TerroristDressType)

    //Add CounterTerrorist
    game.addCounterTerrorist(CounterTerrroristDressType)
    game.addCounterTerrorist(CounterTerrroristDressType)
    game.addCounterTerrorist(CounterTerrroristDressType)

    dressFactoryInstance := getDressFactorySingleInstance()

    for dressType, dress := range dressFactoryInstance.dressMap {
        fmt.Printf(&quot;DressColorType: %s\nDressColor: %s\n&quot;, dressType, dress.getColor())
    }
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>DressColorType: ctDress
DressColor: green
DressColorType: tDress
DressColor: red</code></pre>
</figure>

::: next
#### Leer siguiente

[Proxy en Go []{.fa .fa-arrow-right}](../../proxy/go/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Facade en Go](../../facade/go/example.html){.btn
.btn-default rel="prev"}
:::

## **Flyweight** en otros lenguajes

[![Flyweight en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Flyweight en C#"){.prog-lang-link}
[![Flyweight en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Flyweight en C++"){.prog-lang-link}
[![Flyweight en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Flyweight en Java"){.prog-lang-link}
[![Flyweight en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Flyweight en PHP"){.prog-lang-link}
[![Flyweight en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Flyweight en Python"){.prog-lang-link}
[![Flyweight en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Flyweight en Ruby"){.prog-lang-link}
[![Flyweight en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Flyweight en Rust"){.prog-lang-link}
[![Flyweight en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Flyweight en Swift"){.prog-lang-link}
[![Flyweight en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Flyweight en TypeScript"){.prog-lang-link}
:::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-es" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archivo con ejemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compra el libro electrónico **Sumérgete en los patrones de diseño** para
tener acceso al archivo con decenas de ejemplos detallados que puedes
abrir en tu IDE.

[ Saber más...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
