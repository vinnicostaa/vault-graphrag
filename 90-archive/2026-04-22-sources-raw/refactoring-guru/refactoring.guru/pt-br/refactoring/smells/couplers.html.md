::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::: {.page .text}
::: breadcrumb
[](../../../index.html){.home} /
[Refactoring](../../refactoring.html){.type} / [Code
Smells](../smells.html){.type}
:::

# Couplers {#couplers .title}

All the smells in this group contribute to excessive coupling between
classes or show what happens if coupling is replaced by excessive
delegation.

::::::::::::::: {.relations .link-list}
::::: dl
::: dt
[Feature Envy](../../smells/feature-envy.html)
:::

::: dd
A method accesses the data of another object more than its own data.
:::
:::::

::::: dl
::: dt
[Inappropriate Intimacy](../../smells/inappropriate-intimacy.html)
:::

::: dd
One class uses the internal fields and methods of another class.
:::
:::::

::::: dl
::: dt
[Message Chains](../../smells/message-chains.html)
:::

::: dd
In code you see a series of calls resembling `$a->b()->c()->d()`
:::
:::::

::::: dl
::: dt
[Middle Man](../../smells/middle-man.html)
:::

::: dd
If a class performs only one action, delegating work to another class,
why does it exist at all?
:::
:::::
:::::::::::::::

::: next
#### Leia a seguir

[Feature Envy []{.fa
.fa-arrow-right}](../../smells/feature-envy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Speculative
Generality](../../smells/speculative-generality.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::
::::::::::::::::::::
:::::::::::::::::::::
