::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::: {.main-content-container .center-content-container}
::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} /
[Katalog](catalog.html){.type}
:::

# Wzorce behawioralne {#wzorce-behawioralne .title}

Wzorce behawioralne dotyczą algorytmów i rozdzielania odpowiedzialności
pomiędzy obiektami.

::: {.d-flex .flex-wrap .justify-content-between}
[[ [![Łańcuch
zobowiązań](../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x,/images/patterns/cards/chain-of-responsibility-mini-3x.png 3x"}]{.pattern-image}
[Łańcuch zobowiązań]{.pattern-name .with-aka} [Chain of
Responsibility]{.pattern-aka} ]{.pattern-card-top} [
]{.pattern-card-bottom}](chain-of-responsibility.html){.pattern-card-extended
.chain-of-responsibility}

Pozwala przekazywać żądania według łańcucha obiektów obsługujących.
Otrzymawszy żądanie, każdy z obiektów obsługujących decyduje o
zrealizowaniu żądania lub przekazaniu go do swojego następnika w
łańcuchu.

[[
[![Polecenie](../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x,/images/patterns/cards/command-mini-3x.png 3x"}]{.pattern-image}
[Polecenie]{.pattern-name .with-aka} [Command]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](command.html){.pattern-card-extended .command}

Zmienia żądanie w samodzielny obiekt zawierający wszystkie informacje o
tym żądaniu. Taka transformacja pozwala na parametryzowanie metod przy
przy użyciu różnych żądań. Oprócz tego umożliwia opóźnianie lub
kolejkowanie wykonywania żądań oraz pozwala na cofanie operacji.

[[
[![Iterator](../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x,/images/patterns/cards/iterator-mini-3x.png 3x"}]{.pattern-image}
[Iterator]{.pattern-name .with-aka} [Iterator]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](iterator.html){.pattern-card-extended
.iterator}

Pozwala przechodzić sekwencyjnie po elementach zbioru bez konieczności
eksponowania jego formy (lista, stos, drzewo, itp.).

[[
[![Mediator](../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x,/images/patterns/cards/mediator-mini-3x.png 3x"}]{.pattern-image}
[Mediator]{.pattern-name .with-aka} [Mediator]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](mediator.html){.pattern-card-extended
.mediator}

Pozwala zredukować chaos zależności pomiędzy obiektami. Wzorzec
ogranicza bezpośrednią komunikację pomiędzy obiektami i zmusza je do
współpracy wyłącznie za pośrednictwem obiektu mediatora.

[[
[![Pamiątka](../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x,/images/patterns/cards/memento-mini-3x.png 3x"}]{.pattern-image}
[Pamiątka]{.pattern-name .with-aka} [Memento]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](memento.html){.pattern-card-extended .memento}

Pozwala zapisywać i przywracać wcześniejszy stan obiektu bez ujawniania
szczegółów jego implementacji.

[[
[![Obserwator](../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x,/images/patterns/cards/observer-mini-3x.png 3x"}]{.pattern-image}
[Obserwator]{.pattern-name .with-aka} [Observer]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](observer.html){.pattern-card-extended
.observer}

Pozwala zdefiniować mechanizm subskrypcji by powiadamiać wiele obiektów
o zdarzeniach odbywających się w obserwowanym obiekcie.

[[
[![Stan](../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x,/images/patterns/cards/state-mini-3x.png 3x"}]{.pattern-image}
[Stan]{.pattern-name .with-aka} [State]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](state.html){.pattern-card-extended .state}

Pozwala obiektowi zmienić swoje zachowanie gdy zmieni się jego
wewnętrzny stan. Wygląda to tak, jakby obiekt zmienił swoją klasę.

[[
[![Strategia](../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x,/images/patterns/cards/strategy-mini-3x.png 3x"}]{.pattern-image}
[Strategia]{.pattern-name .with-aka} [Strategy]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](strategy.html){.pattern-card-extended
.strategy}

Pozwala zdefiniować rodzinę algorytmów, umieścić je w osobnych klasach i
uczynić obiekty tych klas wymienialnymi.

[[ [![Metoda
szablonowa](../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x,/images/patterns/cards/template-method-mini-3x.png 3x"}]{.pattern-image}
[Metoda szablonowa]{.pattern-name .with-aka} [Template
Method]{.pattern-aka} ]{.pattern-card-top} [
]{.pattern-card-bottom}](template-method.html){.pattern-card-extended
.template-method}

Definiuje szkielet algorytmu w klasie bazowej, ale umożliwia podklasom
nadpisanie poszczególnych etapów algorytmu bez konieczności zmiany jego
struktury.

[[
[![Odwiedzający](../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x,/images/patterns/cards/visitor-mini-3x.png 3x"}]{.pattern-image}
[Odwiedzający]{.pattern-name .with-aka} [Visitor]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](visitor.html){.pattern-card-extended .visitor}

Pozwala oddzielić algorytmy od obiektów na których pracują.
:::

::: next
#### Czytaj dalej

[Łańcuch zobowiązań []{.fa
.fa-arrow-right}](chain-of-responsibility.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Pełnomocnik](proxy.html){.btn .btn-default
rel="prev"}
:::
:::::::
::::::::
:::::::::
