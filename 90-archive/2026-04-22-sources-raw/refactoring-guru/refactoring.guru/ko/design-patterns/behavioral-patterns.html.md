::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::: {.main-content-container .center-content-container}
::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [목록](catalog.html){.type}
:::

# 행동 디자인 패턴 {#행동-디자인-패턴 .title}

이러한 패턴들은 알고리즘들 및 객체 간의 책임 할당과 관련이 있습니다.

::: {.d-flex .flex-wrap .justify-content-between}
[[ [![책임
연쇄](../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x,/images/patterns/cards/chain-of-responsibility-mini-3x.png 3x"}]{.pattern-image}
[책임 연쇄]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](chain-of-responsibility.html){.pattern-card-extended
.chain-of-responsibility}

일련의 핸들러들의 사슬을 따라 요청을 전달할 수 있게 해주는 행동 디자인
패턴입니다. 각 핸들러는 요청을 받으면 요청을 처리할지 아니면 체인의 다음
핸들러로 전달할지를 결정합니다.

[[
[![커맨드](../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x,/images/patterns/cards/command-mini-3x.png 3x"}]{.pattern-image}
[커맨드]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](command.html){.pattern-card-extended .command}

요청을 요청에 대한 모든 정보가 포함된 독립 실행형 객체로 변환합니다. 이
변환은 다양한 요청들이 있는 메서드들을 인수화할 수 있도록 하며, 요청의
실행을 지연 또는 대기열에 넣을 수 있도록 하고, 또 실행 취소할 수 있는
작업을 지원할 수 있도록 합니다.

[[
[![반복자](../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x,/images/patterns/cards/iterator-mini-3x.png 3x"}]{.pattern-image}
[반복자]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](iterator.html){.pattern-card-extended
.iterator}

컬렉션의 요소들의 기본 표현​(리스트, 스택, 트리 등)​을 노출하지 않고
그들을 하나씩 순회할 수 있도록 합니다.

[[
[![중재자](../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x,/images/patterns/cards/mediator-mini-3x.png 3x"}]{.pattern-image}
[중재자]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](mediator.html){.pattern-card-extended
.mediator}

객체 간의 혼란스러운 의존 관계들을 줄일 수 있습니다. 이 패턴은 객체 간의
직접 통신을 제한하고 중재자 객체를 통해서만 협력하도록 합니다.

[[
[![메멘토](../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x,/images/patterns/cards/memento-mini-3x.png 3x"}]{.pattern-image}
[메멘토]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](memento.html){.pattern-card-extended .memento}

객체의 구현 세부 사항을 공개하지 않으면서 해당 객체의 이전 상태를
저장하고 복원할 수 있게 해줍니다.

[[
[![옵서버](../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x,/images/patterns/cards/observer-mini-3x.png 3x"}]{.pattern-image}
[옵서버]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](observer.html){.pattern-card-extended
.observer}

여러 객체에 자신이 관찰 중인 객체에 발생하는 모든 이벤트에 대하여 알리는
구독 메커니즘을 정의할 수 있도록 합니다.

[[
[![상태](../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x,/images/patterns/cards/state-mini-3x.png 3x"}]{.pattern-image}
[상태]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](state.html){.pattern-card-extended .state}

객체의 내부 상태가 변경될 때 해당 객체가 그의 행동을 변경할 수 있도록
합니다. 객체가 행동을 변경할 때 객체가 클래스를 변경한 것처럼 보일 수
있습니다.

[[
[![전략](../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x,/images/patterns/cards/strategy-mini-3x.png 3x"}]{.pattern-image}
[전략]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](strategy.html){.pattern-card-extended
.strategy}

알고리즘들의 패밀리를 정의하고, 각 패밀리를 별도의 클래스들에 넣은 후
그들의 객체들을 상호교환할 수 있도록 합니다.

[[ [![템플릿
메서드](../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x,/images/patterns/cards/template-method-mini-3x.png 3x"}]{.pattern-image}
[템플릿 메서드]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](template-method.html){.pattern-card-extended
.template-method}

부모 클래스에서 알고리즘의 골격을 정의하지만, 해당 알고리즘의 구조를
변경하지 않고 자식 클래스들이 알고리즘의 특정 단계들을
오버라이드​(재정의)​할 수 있도록 합니다.

[[
[![비지터](../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x,/images/patterns/cards/visitor-mini-3x.png 3x"}]{.pattern-image}
[비지터]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](visitor.html){.pattern-card-extended .visitor}

알고리즘들을 그들이 작동하는 객체들로부터 분리할 수 있습니다.
:::

::: next
#### 다음을 보세요

[책임 연쇄 []{.fa .fa-arrow-right}](chain-of-responsibility.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 프록시](proxy.html){.btn .btn-default
rel="prev"}
:::
:::::::
::::::::
:::::::::
