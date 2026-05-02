:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.page .list .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](../refactoring.html){.type}
:::

# Code Smells {#code-smells .title}

--- What? How can code \"smell\"??\
--- Well it doesn\'t have a nose\... but it definitely can stink!

:::::::: {.section-bordered .link-list}
::::::: cell-flex
::: cell-image
![](../images/refactoring/content/catalog/bloatersae0b.png?id=32a44a371122874ebd1e8a2cbb9202b9){width="160"
height="280"
srcset="/images/refactoring/content/catalog/bloaters-2x.png?id=21e2800a3c877cc37b95103bcf6ec5df 2x,/images/refactoring/content/catalog/bloaters-3x.png 3x"}
:::

::::: cell-text
### [Bloaters](smells/bloaters.html)

::: annotation
Bloaters are code, methods and classes that have increased to such
gargantuan proportions that they are hard to work with. Usually these
smells do not crop up right away, rather they accumulate over time as
the program evolves (and especially when nobody makes an effort to
eradicate them).
:::

::: catalog-list
-   [Long Method](../smells/long-method.html)
-   [Large Class](../smells/large-class.html)

<!-- -->

-   [Primitive Obsession](../smells/primitive-obsession.html)
-   [Long Parameter List](../smells/long-parameter-list.html)

<!-- -->

-   [Data Clumps](../smells/data-clumps.html)
:::
:::::
:::::::
::::::::

:::::::: {.section-bordered .link-list}
::::::: cell-flex
::: cell-image
![](../images/refactoring/content/catalog/oo-abusersd646.png?id=dee31050499d8d6b5a2d5b2e84e68cc8){width="160"
height="280"
srcset="/images/refactoring/content/catalog/oo-abusers-2x.png?id=d42468d27ca548c870a47b2ed08c0a16 2x,/images/refactoring/content/catalog/oo-abusers-3x.png 3x"}
:::

::::: cell-text
### [Object-Orientation Abusers](smells/oo-abusers.html) {#oo-abusers}

::: annotation
All these smells are incomplete or incorrect application of
object-oriented programming principles.
:::

::: catalog-list
-   [Alternative Classes with Different
    Interfaces](../smells/alternative-classes-with-different-interfaces.html)

<!-- -->

-   [Refused Bequest](../smells/refused-bequest.html)
-   [Switch Statements](../smells/switch-statements.html)

<!-- -->

-   [Temporary Field](../smells/temporary-field.html)
:::
:::::
:::::::
::::::::

:::::::: {.section-bordered .link-list}
::::::: cell-flex
::: cell-image
![](../images/refactoring/content/catalog/change-preventers1331.png?id=db5f332e55fd4b993e15c419baf1db41){width="160"
height="280"
srcset="/images/refactoring/content/catalog/change-preventers-2x.png?id=94d4444b1a3414ac3704f4ab53062f08 2x,/images/refactoring/content/catalog/change-preventers-3x.png 3x"}
:::

::::: cell-text
### [Change Preventers](smells/change-preventers.html)

::: annotation
These smells mean that if you need to change something in one place in
your code, you have to make many changes in other places too. Program
development becomes much more complicated and expensive as a result.
:::

::: catalog-list
-   [Divergent Change](../smells/divergent-change.html)

<!-- -->

-   [Parallel Inheritance
    Hierarchies](../smells/parallel-inheritance-hierarchies.html)

<!-- -->

-   [Shotgun Surgery](../smells/shotgun-surgery.html)
:::
:::::
:::::::
::::::::

:::::::: {.section-bordered .link-list}
::::::: cell-flex
::: cell-image
![](../images/refactoring/content/catalog/dispensables12b8.png?id=b1072dc9efcf8c0374ddbd7e0b8d496f){width="160"
height="280"
srcset="/images/refactoring/content/catalog/dispensables-2x.png?id=a316a726f68f8778cdfd38e850d00b8b 2x,/images/refactoring/content/catalog/dispensables-3x.png 3x"}
:::

::::: cell-text
### [Dispensables](smells/dispensables.html)

::: annotation
A dispensable is something pointless and unneeded whose absence would
make the code cleaner, more efficient and easier to understand.
:::

::: catalog-list
-   [Comments](../smells/comments.html)
-   [Duplicate Code](../smells/duplicate-code.html)

<!-- -->

-   [Data Class](../smells/data-class.html)
-   [Dead Code](../smells/dead-code.html)

<!-- -->

-   [Lazy Class](../smells/lazy-class.html)
-   [Speculative Generality](../smells/speculative-generality.html)
:::
:::::
:::::::
::::::::

:::::::: {.section-bordered .link-list style="border-bottom: none;"}
::::::: cell-flex
::: cell-image
![](../images/refactoring/content/catalog/couplers15c1.png?id=1a0e96c005372053d5823ccb5282ae7d){width="160"
height="280"
srcset="/images/refactoring/content/catalog/couplers-2x.png?id=86e33d80273d564bd4d48a554167a7c9 2x,/images/refactoring/content/catalog/couplers-3x.png 3x"}
:::

::::: cell-text
### [Couplers](smells/couplers.html)

::: annotation
All the smells in this group contribute to excessive coupling between
classes or show what happens if coupling is replaced by excessive
delegation.
:::

::: catalog-list
-   [Feature Envy](../smells/feature-envy.html)
-   [Inappropriate Intimacy](../smells/inappropriate-intimacy.html)

<!-- -->

-   [Incomplete Library Class](../smells/incomplete-library-class.html)
-   [Message Chains](../smells/message-chains.html)

<!-- -->

-   [Middle Man](../smells/middle-man.html)
:::
:::::
:::::::
::::::::
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::
