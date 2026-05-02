:::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::: {.main-content-container .center-content-container}
:::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons de
création](creational-patterns.html){.type}
:::

# Comparaison des fabriques {#comparaison-des-fabriques .title}

Cet article va vous exposer les différences entre :

1.  Fabrique
2.  Méthode de création
3.  Méthode (ou fabrique) de création statique
4.  Fabrique simple
5.  Patron de conception [Fabrique](factory-method.html)
6.  [Fabrique abstraite](abstract-factory.html)

Vous trouverez des références à ces termes un peu partout sur Internet.
Même s'ils se ressemblent, ils ont tous des significations différentes.
Beaucoup ne s'en rendent pas compte, ce qui peut semer la confusion et
mener à des incompréhensions.

Essayons donc de dénicher les différences et résolvons ce problème une
bonne fois pour toutes.

## 1. Fabrique

**Fabrique** est un terme ambigu utilisé pour représenter une fonction,
méthode ou classe qui est censée produire quelque chose. En général, les
fabriques produisent des objets. Elles peuvent également fabriquer des
fichiers, des enregistrements de bases de données, etc.

Par exemple, on peut faire référence à n'importe lequel des éléments
suivants en l'appelant « fabrique » :

-   Une fonction ou méthode qui crée l'interface graphique utilisateur
    (GUI) d'un programme.
-   Une classe qui crée des utilisateurs.
-   Une méthode statique qui appelle un constructeur de classe d'une
    manière un peu particulière.
-   Un des patrons de conception de création.

En général, lorsque quelqu'un parle d'une « fabrique », il est possible
de comprendre exactement de quoi il s'agit en s'aidant du contexte. En
cas de doute, n'hésitez pas à demander des précisions. Il est possible
que l'auteur ne connaisse même pas ces différences.

## 2. Méthode de création

La méthode de création est définie dans le livre [Refactoring To
Patterns](https://www.amazon.com/-/dp/0321213351) comme « une méthode
qui crée des objets ». Cela signifie que chaque résultat du patron de
conception fabrique est une « méthode de création », mais pas l'inverse.
Cela signifie également que vous pouvez remplacer le terme « méthode de
création » partout où Martin Fowler utilise « fonction factory »
(factory method) dans
[Refactoring](https://www.amazon.com/-/dp/2100801163) ou lorsque Joshua
Bloch écrit « méthode de fabrique statique » dans [Java
efficace](https://www.amazon.com/-/dp/2711748057).

En réalité, la méthode de création est juste un emballeur qui englobe un
appel au constructeur. Son nom importe peu, tant qu'il exprime vos
intentions. Il peut toutefois vous aider à isoler votre code des
modifications apportées au constructeur. Il peut même contenir une
certaine logique qui renvoie des objets existants au lieu d'en créer de
nouveaux.

Beaucoup de personnes vont appeler ce genre de méthodes des
« fabriques » juste parce qu'elles fabriquent de nouveaux objets. C'est
plutôt évident : la méthode crée des objets. Puisque toutes les
*fabriques* créent des objets, cette méthode doit logiquement être une
*fabrique*. Il est donc naturel d'être un peu confus lorsque l'on parle
du vrai patron de conception [Fabrique](factory-method.html).

Dans l'exemple suivant, `next` (suivant) est une méthode de création :

<figure class="code">
<pre class="code" lang="php"><code>class Number {
    private $value;

    public function __construct($value) {
        $this-&gt;value = $value;
    }

    public function next() {
        return new Number ($this-&gt;value + 1);
    }
}</code></pre>
</figure>

## 3. Méthode de création statique

Une **méthode de création statique** est une méthode de création
déclarée en `static`. En d'autres termes, elle peut être appelée dans
une classe et n'a pas besoin d'un objet pour être créée.

Ne soyez donc pas étonné lorsque quelqu'un appelle une telle méthode une
« méthode de fabrique statique ». C'est juste une mauvaise habitude. La
[Fabrique](factory-method.html) est un patron de conception qui repose
sur l'héritage. Si vous la rendez `static`, vous neutralisez l'utilité
du patron, car vous ne pouvez plus l'étendre dans des sous-classes.

Lorsqu'une méthode de création statique renvoie de nouveaux objets, elle
devient un autre constructeur possible.

Cela se révèle utile si :

-   Vous avez besoin de plusieurs constructeurs différents, mais leurs
    signatures sont identiques. Par exemple, avoir en même temps
    `Random(int max)` et `Random(int min)` est impossible en Java, C++,
    C# et dans bien d'autres langages. Le contournement le plus utilisé
    consiste à créer plusieurs méthodes statiques qui appellent le
    constructeur par défaut, et à affecter les valeurs appropriées par
    la suite.

-   Vous voulez réutiliser des objets existants plutôt que d'en
    instancier de nouveaux (se référer au patron de conception
    [Singleton](singleton.html)). Dans la majorité des langages de
    programmation, les constructeurs doivent retourner de nouvelles
    instances de classes. La méthode de création statique permet de
    contourner cette limitation. À l'intérieur de la méthode statique,
    votre code va décider s'il veut créer une instance toute fraiche en
    appelant le constructeur, ou s'il préfère renvoyer un objet existant
    depuis le cache.

Dans l'exemple suivant, `load` (charger) est une méthode de création
statique. Elle fournit une manière pratique de récupérer les
utilisateurs dans une base de données.

<figure class="code">
<pre class="code" lang="php"><code>class User {
    private $id, $name, $email, $phone;

    public function __construct($id, $name, $email, $phone) {
        $this-&gt;id = $id;
        $this-&gt;name = $name;
        $this-&gt;email = $email;
        $this-&gt;phone = $phone;
    }

    public static function load($id) {
        list($id, $name, $email, $phone) = DB::load_data(&#39;users&#39;, &#39;id&#39;, &#39;name&#39;, &#39;email&#39;, &#39;phone&#39;);
        $user = new User($id, $name, $email, $phone);
        return $user;
    }
}</code></pre>
</figure>

## 4. Patron *fabrique simple*

Le patron **Fabrique simple** []{.pop-i}[définie dans le livre [Tête la
première - Design
patterns](https://www.amazon.com/-/dp/2841773507).]{.pop-content} décrit
une classe ayant une méthode de création qui choisit la classe du
produit à instancier à l'aide d'un gros bloc conditionnel basé sur les
paramètres de la méthode, puis la renvoie.

Les *fabriques simples* sont souvent confondues avec les *fabriques*
générales ou avec un des patrons de création. Dans la majorité des cas,
une fabrique simple est une étape intermédiaire pour mettre en place un
patron de conception [Fabrique](factory-method.html) ou [Fabrique
abstraite](abstract-factory.html).

Une fabrique simple est souvent représentée par une unique méthode dans
une seule classe. Avec le temps, cette méthode peut devenir trop longue
et vous pourriez décider de transférer certaines parties dans des
sous-classes. Lorsque vous aurez rencontré cette situation plusieurs
fois, vous vous rendrez compte que tout ceci équivaut en fait à un
patron de conception *fabrique*.

Au passage, si vous déclarez une fabrique simple `abstract`, elle ne se
transforme pas en *fabrique abstraite* comme par magie.

Voici un exemple de *fabrique simple* :

<figure class="code">
<pre class="code" lang="php"><code>class UserFactory {
    public static function create($type) {
        switch ($type) {
            case &#39;user&#39;: return new User();
            case &#39;customer&#39;: return new Customer();
            case &#39;admin&#39;: return new Admin();
            default:
                throw new Exception(&#39;Wrong user type passed.&#39;);
        }
    }
}</code></pre>
</figure>

## 5. Patron de conception *fabrique*

La **Fabrique** []{.pop-i}[définie dans le GoF book [Design Patterns -
Catalogue de modèles de conception
réutilisables](https://www.amazon.com/-/dp/2711786447).]{.pop-content}
est un patron de création qui définit une interface pour créer des
objets, mais permet aux sous-classes de modifier le type de l'objet qui
sera créé.

Si vous voyez une méthode de création dans une classe de base et des
sous-classes qui l'étendent, il s'agit peut-être d'une fabrique.

<figure class="code">
<pre class="code" lang="php"><code>abstract class Department {
    public abstract function createEmployee($id);

    public function fire($id) {
        $employee = $this-&gt;createEmployee($id);
        $employee-&gt;paySalary();
        $employee-&gt;dismiss();
    }
}

class ITDepartment extends Department {
    public function createEmployee($id) {
        return new Programmer($id);
    }
}

class AccountingDepartment extends Department {
    public function createEmployee($id) {
        return new Accountant($id);
    }
}</code></pre>
</figure>

## 6. Patron *fabrique abstraite*

La **Fabrique Abstraite** []{.pop-i}[également définie dans le [GoF
book](https://www.amazon.com/-/dp/2711786447).]{.pop-content} est un
patron de conception de création qui permet de créer des familles
d'objets apparentés sans préciser leur classe concrète.

Mais que sont « des familles d'objets » ? Prenons cet ensemble de
classes comme exemple : `Transport` + `Moteur` + `Contrôles`. Plusieurs
variantes peuvent exister :

1.  `Voiture` + `MoteurCombustion` + `Volant`
2.  `Avion` + `MoteurRéaction` + `Manche`

Si votre programme fonctionne sans familles de produits, vous n'avez pas
besoin d'une fabrique abstraite.

Je vais me répéter, mais beaucoup confondent la *fabrique abstraite*
avec une classe fabrique toute simple déclarée `abstract`. Ne faites pas
cette erreur !

### Postface

Maintenant que vous savez les différencier, posez un regard nouveau sur
ces patrons :

-   [Fabrique](factory-method.html)
-   [Fabrique abstraite](abstract-factory.html)

::: next
#### Suivant

[Monteur []{.fa .fa-arrow-right}](builder.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Fabrique abstraite](abstract-factory.html){.btn
.btn-default rel="prev"}
:::
::::::
:::::::
::::::::
