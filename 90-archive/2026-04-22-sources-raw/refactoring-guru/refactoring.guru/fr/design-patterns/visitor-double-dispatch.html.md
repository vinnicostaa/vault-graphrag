:::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::: {.main-content-container .center-content-container}
:::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons
comportementaux](behavioral-patterns.html){.type} /
[Visiteur](visitor.html){.type}
:::

# Visiteur et double répartition {#visiteur-et-double-répartition .title}

Jetons un œil à la hiérarchie des classes des formes géométriques
suivantes (gare au pseudo-code) :

<figure class="code">
<pre class="code" lang="pseudocode"><code>interface Graphic is
    method draw()

class Shape implements Graphic is
    field id
    method draw()
    // ...

class Dot extends Shape is
    field x, y
    method draw()
    // ...

class Circle extends Dot is
    field radius
    method draw()
    // ...

class Rectangle extends Shape is
    field width, height
    method draw()
    // ...

class CompoundGraphic implements Graphic is
    field children: array of Graphic
    method draw()
    // ...</code></pre>
</figure>

Ce code fonctionne très bien et l'application tourne en production. Mais
un beau jour, vous décidez d'y ajouter une fonctionnalité d'export. Ce
code passerait pour un intrus si vous le rajoutiez dans ces classes.
Plutôt que de concevoir un comportement d'exportation dans chaque classe
de cette hiérarchie, vous décidez de créer une nouvelle classe à
l'extérieur, et d'y mettre toute la logique d'export. Dans cette classe,
vous allez écrire des méthodes pour exporter l'état public de chaque
objet sous la forme de chaînes de caractères dans un fichier XML :

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Exporter is
    method export(s: Shape) is
        print(&quot;Exporting shape&quot;)
    method export(d: Dot)
        print(&quot;Exporting dot&quot;)
    method export(c: Circle)
        print(&quot;Exporting circle&quot;)
    method export(r: Rectangle)
        print(&quot;Exporting rectangle&quot;)
    method export(cs: CompoundGraphic)
        print(&quot;Exporting compound&quot;)</code></pre>
</figure>

Le code semble bon, testons-le :

<figure class="code">
<pre class="code" lang="pseudocode"><code>class App() is
    method export(shape: Shape) is
        Exporter exporter = new Exporter()
        exporter.export(shape);

app.export(new Circle());
// Malheureusement, nous allons nous retrouver avec « Exporting
// shape » en sortie.</code></pre>
</figure>

Attendez un instant ! Mais pourquoi ?!

## Raisonner comme un compilateur

Remarque : Les informations suivantes sont valides pour tous les
langages de programmation orientés objet modernes (Java, C#, PHP, etc.).

### Liaison tardive/dynamique

Imaginez que vous êtes un compilateur. Vous devez décider comment
compiler le code suivant :

<figure class="code">
<pre class="code" lang="pseudocode"><code>method drawShape(shape: Shape) is
    shape.draw();</code></pre>
</figure>

Voyons voir\... La méthode `dessine` (draw) est définie dans la classe
`Forme` (Shape). Attendez un instant ! Il y a également quatre
sous-classes qui redéfinissent cette méthode. Pouvons-nous vraiment être
sûr à 100 % de l'implémentation à appeler ici ? Je n'en ai pas
l'impression. La seule façon d'en être absolument certain est de lancer
le programme et de vérifier la classe de l'objet passée à la méthode. La
seule chose dont nous pouvons être sûrs, c'est que l'objet **aura**
l'implémentation de la méthode `dessine`.

Le code machine va vérifier la classe du paramètre `s`, puis récupérer
l'implémentation de la classe appropriée.

Ce genre de vérification est appelé liaison tardive (ou dynamique) :

-   **Tardive** parce que nous associons l'objet et son implémentation
    après la compilation, pendant l'exécution.
-   **Dynamique** parce que chaque nouvel objet va potentiellement être
    associé à une implémentation différente.

### Liaison statique/anticipée

À présent, « compilons » le code suivant :

<figure class="code">
<pre class="code" lang="pseudocode"><code>method exportShape(shape: Shape) is
    Exporter exporter = new Exporter()
    exporter.export(shape);</code></pre>
</figure>

La deuxième ligne ne porte pas d'ambiguïté : la classe `Exportateur` n'a
pas de constructeur, donc nous instancions juste un objet. Qu'en est-il
de l'appel à la méthode `export` ? L'`Exportateur` possède cinq méthodes
dont le nom est identique, mais avec des paramètres différents. Laquelle
allons-nous appeler ? Il semble que nous allons devoir faire appel à une
liaison dynamique ici également.

Mais il y a un autre problème. Que se passe-t-il si une classe forme n'a
pas de méthode `export` associée dans la classe `Exportateur` ? Prenons
un objet `Ellipse` pour exemple. Le compilateur ne peut pas garantir
qu'une méthode surchargée existe en face de chaque méthode redéfinie. Le
compilateur ne peut pas autoriser une situation aussi ambiguë.

C'est pourquoi les développeurs de compilateurs privilégient le chemin
le plus sûr et utilisent la liaison statique (ou anticipée) pour
surcharger les méthodes :

-   **Anticipée** parce qu'elle est déjà établie à la compilation, avant
    le lancement du programme.
-   **Statique** parce qu'elle ne peut pas être modifiée pendant
    l'exécution du programme.

Revenons à notre exemple. Nous avons la certitude que le paramètre
appartiendra à la hiérarchie `Forme` : soit à la classe `Forme`, soit à
l'une de ses sous-classes. Nous savons également que la classe
`Exportateur` possède une implémentation basique de la méthode export
qui s'occupe de la classe `Forme` : `export(s: Shape)`.

C'est la seule implémentation qui peut être associée avec un code donné
sans provoquer d'ambiguïté. C'est pour cela que même si nous passons un
objet `Rectangle` dans la méthode `exportForme`, l'exportateur appellera
tout de même la méthode `export(s: Shape)`.

## Double répartition

La **Double répartition** est une technique qui permet d'utiliser une
liaison dynamique avec des méthodes surchargées. Voici comment s'y
prendre :

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Visitor is
    method visit(s: Shape) is
        print(&quot;Visited shape&quot;)
    method visit(d: Dot)
        print(&quot;Visited dot&quot;)

interface Graphic is
    method accept(v: Visitor)

class Shape implements Graphic is
    method accept(v: Visitor)
        // Le compilateur est certain que `this` est une `Forme`,
        // ce qui signifie que la méthode `visit(s: Shape)` peut être
        // appelée sans problème.
        v.visit(this)

class Dot extends Shape is
    method accept(v: Visitor)
        // Le compilateur est certain que `this` est un `Point`,
        // ce qui signifie que la méthode `visit(s: Dot)` peut être
        // appelée sans problème.
        v.visit(this)

Visitor v = new Visitor();
Graphic g = new Dot();

// La méthode `accepter` est redéfinie, mais pas surchargée. Le 
// compilateur l’associe dynamiquement. Par conséquent, la méthode 
// `accepter` sera appelée sur une classe correspondant à un objet qui 
// appelle une méthode (dans notre cas, la classe `Point`).

g.accept(v);

// Sortie : &quot;Visited dot&quot;</code></pre>
</figure>

### Postface

Bien que le patron de conception [Visiteur](visitor.html) soit basé sur
le principe de la double répartition, ce n'est pas son rôle principal.
Le visiteur vous permet d'ajouter des traitements « externes » à toute
une hiérarchie de classes sans modifier le code existant de ces classes.

::: next
#### Suivant

[Les patrons de conception en C# []{.fa
.fa-arrow-right}](csharp.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Visiteur](visitor.html){.btn .btn-default
rel="prev"}
:::
::::::
:::::::
::::::::
