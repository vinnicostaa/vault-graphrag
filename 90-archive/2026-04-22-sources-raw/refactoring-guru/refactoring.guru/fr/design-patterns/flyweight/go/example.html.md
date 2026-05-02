:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} / [Poids
mouche](../../flyweight.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Poids
mouche](../../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# **Poids mouche** en Go {#poids-mouche-en-go .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
Le **poids mouche** est un patron de conception structurel qui permet à
des programmes de limiter leur consommation de mémoire malgré un très
grand nombre d'objets.

Ce patron est obtenu en partageant des parties de l'état d'un objet à
plusieurs autres objets. En d'autres termes, le poids mouche économise
de la RAM en mettant en cache les données identiques chez différents
objets.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Poids mouche ](../../flyweight.html){.btn
.btn-lg .btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Exemple conceptuel](#example-0)
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

## Exemple conceptuel {#example-0-title}

Dans une partie de Counter-Strike, les terroristes et les
contre-terroristes sont habillés différemment. Pour faire plus simple,
disons que chaque camp n'a qu'un seul type de tenue. L'objet Tenue est
imbriqué dans l'objet joueur comme ci-dessous.

Voici la struct pour un joueur. Nous pouvons observer que la tenue est
imbriquée dans la struct joueur :

<figure class="code">
<pre class="code" lang="go"><code>type player struct {
    dress      dress
    playerType string // Peut être un T ou CT
    lat        int
    long       int
}</code></pre>
</figure>

Prenons 5 terroristes et 5 contre-terroristes, donc 10 joueurs au total.
Nous avons maintenant 2 possibilités pour les tenues.

1.  Chacun des 10 objets joueur crée un objet tenue différent et
    l'imbrique. Au total, 10 objets tenue seront créés.

2.  Nous créons deux objets tenue :

    -   Un unique objet tenue pour terroriste. Il sera partagé entre les
        5 terroristes.
    -   Un unique objet tenue pour contre-terroriste. Il sera partagé
        entre les 5 contre-terroristes.

Comme vous pouvez le constater, dans la première approche, un total de
10 objets tenue sont créés, alors que dans la seconde, on se contente de
deux. La seconde approche est celle suggérée par le patron de conception
poids mouche. Les deux objets tenue que nous avons créés s'appellent les
objets poids mouche.

Le patron de conception poids mouche retire les morceaux communs et crée
des objets poids mouche. Ces objets (tenues) peuvent ensuite être
partagés entre de multiples objets (joueurs). Nous réduisons ainsi
drastiquement le nombre d'objets tenue et de plus, même si vous ajoutez
plus de joueurs, les deux objets tenue restent suffisants.

Dans le patron poids mouche, nous stockons les objets poids mouche dans
l'attribut de la table de hachage. Dès que d'autres objets qui partagent
ces objets poids mouche sont créés, ces derniers sont récupérés depuis
la table de hachage.

Regardons quelles parties de ce dispositif seront les états intrinsèques
et extrinsèques :

-   *État intrinsèque* : La tenue fait partie de l'état intrinsèque, car
    elle peut être partagée entre plusieurs terroristes ou
    contre-terroristes.

-   *État extrinsèque* : La position de chaque joueur et son arme font
    partie de l'état extrinsèque, car elles sont différentes pour chaque
    objet.

#### []{#example-0--dressFactory-go .anchor} **dressFactory.go:** Fabrique poids mouche

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

#### []{#example-0--dress-go .anchor} **dress.go:** Interface poids mouche

<figure class="code">
<pre class="code" lang="go"><code>package main

type Dress interface {
    getColor() string
}</code></pre>
</figure>

#### []{#example-0--terroristDress-go .anchor} **terroristDress.go:** Objet poids mouche concret

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

#### []{#example-0--counterTerroristDress-go .anchor} **counterTerroristDress.go:** Objet poids mouche concret

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

#### []{#example-0--player-go .anchor} **player.go:** Contexte

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

#### []{#example-0--game-go .anchor} **game.go:** Code client

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

#### []{#example-0--main-go .anchor} **main.go:** Code client

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

#### []{#example-0--output-txt .anchor} **output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>DressColorType: ctDress
DressColor: green
DressColorType: tDress
DressColor: red</code></pre>
</figure>

::: next
#### Suivant

[Procuration en Go []{.fa
.fa-arrow-right}](../../proxy/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Façade en Go](../../facade/go/example.html){.btn
.btn-default rel="prev"}
:::

## **Poids mouche** dans les autres langues

[![Poids mouche en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Poids mouche en C#"){.prog-lang-link}
[![Poids mouche en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Poids mouche en C++"){.prog-lang-link}
[![Poids mouche en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Poids mouche en Java"){.prog-lang-link}
[![Poids mouche en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Poids mouche en PHP"){.prog-lang-link}
[![Poids mouche en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Poids mouche en Python"){.prog-lang-link}
[![Poids mouche en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Poids mouche en Ruby"){.prog-lang-link}
[![Poids mouche en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Poids mouche en Rust"){.prog-lang-link}
[![Poids mouche en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Poids mouche en Swift"){.prog-lang-link}
[![Poids mouche en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Poids mouche en TypeScript"){.prog-lang-link}
:::::::::::::::::::::::

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
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
