:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[상태](../../state.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![상태](../../../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x"}
:::

# PHP로 작성된 **상태** {#php로-작성된-상태 .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**상태**는 객체의 내부 상태가 변경될 때 해당 객체가 행동을 변경할 수
있도록 하는 행동 디자인 패턴입니다.

패턴은 상태 관련 행동들을 별도의 상태 클래스들로 추출하며 또 원래 객체가
자체적으로 작동하는 대신 위에 언급된 클래스들에 작업을 위임하도록
강제합니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 상태에 대하여 더 자세히 알아보세요 ](../../state.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
내비게이션
:::

::: en-intro
 [소개](#)
:::

::: en-intro
 [개념적인 예시](#example-0)
:::

::: en-file
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [실제 사례 예시](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 예시:** 상태 패턴은 `switch` 연산자를 기반으로 하는 크고 성가신
상태 머신들을 객체들로 바꾸기 위해 PHP에서 때때로 사용됩니다.

**식별:** 객체들의 상태에 따라 행동을 변경하는 메서드들이 있으면 패턴은
상태 패턴으로 초기 식별될 수 있으며 이 상태가 상태 객체들 자체를
포함하여 다른 객체들에 의해 제어되거나 대체될 수 있으면 해당 패턴은 상태
패턴입니다.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[개념적인 예시](#example-0){#example-0-tab .nav-item .nav-link .active
toggle="tab" role="tab" aria-controls="example-0" aria-selected="true"}
[실제 사례 예시](#example-1){#example-1-tab .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## 개념적인 예시 {#example-0-title}

이 예시는 **상태** 디자인 패턴의 구조를 보여주고 다음 질문에 중점을
둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

이 패턴의 구조를 배우면 실제 PHP 사용 사례를 기반으로 하는 다음 예시를
더욱 쉽게 이해할 수 있을 것입니다.

#### []{#example-0--index-php .anchor} **index.php:** 개념적인 예시

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\State\Conceptual;

/**
 * The Context defines the interface of interest to clients. It also maintains a
 * reference to an instance of a State subclass, which represents the current
 * state of the Context.
 */
class Context
{
    /**
     * @var State A reference to the current state of the Context.
     */
    private $state;

    public function __construct(State $state)
    {
        $this-&gt;transitionTo($state);
    }

    /**
     * The Context allows changing the State object at runtime.
     */
    public function transitionTo(State $state): void
    {
        echo &quot;Context: Transition to &quot; . get_class($state) . &quot;.\n&quot;;
        $this-&gt;state = $state;
        $this-&gt;state-&gt;setContext($this);
    }

    /**
     * The Context delegates part of its behavior to the current State object.
     */
    public function request1(): void
    {
        $this-&gt;state-&gt;handle1();
    }

    public function request2(): void
    {
        $this-&gt;state-&gt;handle2();
    }
}

/**
 * The base State class declares methods that all Concrete State should
 * implement and also provides a backreference to the Context object, associated
 * with the State. This backreference can be used by States to transition the
 * Context to another State.
 */
abstract class State
{
    /**
     * @var Context
     */
    protected $context;

    public function setContext(Context $context)
    {
        $this-&gt;context = $context;
    }

    abstract public function handle1(): void;

    abstract public function handle2(): void;
}

/**
 * Concrete States implement various behaviors, associated with a state of the
 * Context.
 */
class ConcreteStateA extends State
{
    public function handle1(): void
    {
        echo &quot;ConcreteStateA handles request1.\n&quot;;
        echo &quot;ConcreteStateA wants to change the state of the context.\n&quot;;
        $this-&gt;context-&gt;transitionTo(new ConcreteStateB());
    }

    public function handle2(): void
    {
        echo &quot;ConcreteStateA handles request2.\n&quot;;
    }
}

class ConcreteStateB extends State
{
    public function handle1(): void
    {
        echo &quot;ConcreteStateB handles request1.\n&quot;;
    }

    public function handle2(): void
    {
        echo &quot;ConcreteStateB handles request2.\n&quot;;
        echo &quot;ConcreteStateB wants to change the state of the context.\n&quot;;
        $this-&gt;context-&gt;transitionTo(new ConcreteStateA());
    }
}

/**
 * The client code.
 */
$context = new Context(new ConcreteStateA());
$context-&gt;request1();
$context-&gt;request2();</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Context: Transition to RefactoringGuru\State\Conceptual\ConcreteStateA.
ConcreteStateA handles request1.
ConcreteStateA wants to change the state of the context.
Context: Transition to RefactoringGuru\State\Conceptual\ConcreteStateB.
ConcreteStateB handles request2.
ConcreteStateB wants to change the state of the context.
Context: Transition to RefactoringGuru\State\Conceptual\ConcreteStateA.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## 실제 사례 예시 {#example-1-title}

이 예시에서 **상태** 패턴은 송장의 다양한 상태를 나타내는 데 사용됩니다.
이 접근법은 송장을 한 상태에서 다른 상태로 전환할 때 다양한 조건 검사를
구현하고, 각 상태의 로직을 별도의 클래스로 캡슐화할 수 있게 해줍니다.

#### []{#example-1--index-php .anchor} **index.php:** 개념적인 예시

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\State\RealWorld;

/**
 * State Design Pattern
 *
 * Intent: lets an object alter its behavior when its internal state changes. It
 * appears as if the object changed its class.
 */

/**
 * Invoice State Interface
 *
 * This interface defines the contract that all invoice states must implement.
 * It represents the State interface in the State pattern, ensuring that all
 * concrete states provide implementations for all possible events/transitions.
 *
 * The interface defines all possible events that can occur in the invoice
 * lifecycle, regardless of whether a particular state can handle them. This
 * approach ensures consistency across all states and makes the system more
 * maintainable.
 */
interface InvoiceState
{
    public function finalize(): void;
    public function pay(): void;
    public function cancel(): void;
    public function void(): void;
    public function getName(): string;
}


/**
 * Abstract Base State class
 *
 * This abstract class implements the InvoiceState interface and provides
 * default implementations for all state transition methods. The default
 * behavior is to throw exceptions for invalid transitions, following the &quot;fail-
 * fast&quot; principle.
 *
 * This approach allows concrete states to only override the methods for
 * transitions they actually support, keeping the code clean and focused. Any
 * attempt to perform an invalid transition will result in a clear exception
 * rather than silent failure.
 *
 * The abstract class also maintains a reference to the context (Invoice)
 * object, which is needed for performing state transitions.
 */
abstract class BaseInvoiceState implements InvoiceState
{
     /**
      * Reference to the context object (Invoice)
      *
      * Each state needs access to the context to perform state transitions.
      * This creates a bidirectional relationship between the state and context.
      */
    protected $invoice;

    public function __construct(Invoice $invoice)
    {
        $this-&gt;invoice = $invoice;
    }


    /**
     * Default implementation for finalize event
     *
     * By default, finalize is not allowed in most states. Only states that
     * support this transition will override this method.
     *
     * @throws InvalidStateTransitionException
     */
    public function finalize(): void
    {
        throw new InvalidStateTransitionException(&quot;Cannot finalize invoice in &quot; . $this-&gt;getName() . &quot; state&quot;);
    }

    /**
     * Default implementation for pay event
     *
     * By default, payment is not allowed in most states. Only states that
     * support this transition will override this method.
     *
     * @throws InvalidStateTransitionException
     */
    public function pay(): void
    {
        throw new InvalidStateTransitionException(&quot;Cannot pay invoice in &quot; . $this-&gt;getName() . &quot; state&quot;);
    }

    /**
     * Default implementation for cancel event
     *
     * By default, cancellation is not allowed in most states. Only states that
     * support this transition will override this method.
     *
     * @throws InvalidStateTransitionException
     */
    public function cancel(): void
    {
        throw new InvalidStateTransitionException(&quot;Cannot cancel invoice in &quot; . $this-&gt;getName() . &quot; state&quot;);
    }


    /**
     * Default implementation for void event
     *
     * By default, voiding is not allowed in most states. Only states that
     * support this transition will override this method.
     *
     * @throws InvalidStateTransitionException
     */
    public function void(): void
    {
        throw new InvalidStateTransitionException(&quot;Cannot void invoice in &quot; . $this-&gt;getName() . &quot; state&quot;);
    }

    /**
     * Abstract method to get the state name
     *
     * Each concrete state must implement this method to return its name. This
     * is used for logging, debugging, and display purposes.
     *
     * @return string The name of the current state
     */
    abstract public function getName(): string;
}

/**
 * Each Concrete State corresponds to a specific state.
 *
 * This Concrete State Represents a draft invoice.
 *
 * This is the initial state of every invoice. In this state, the invoice is
 * still being prepared and can only be finalized to move to the Open state. No
 * other operations are allowed in this state.
 */
class DraftInvoiceState extends BaseInvoiceState
{
    /**
     * Handle finalize event
     *
     * This is the only valid transition from Draft state. When an invoice is
     * finalized, it transitions to the Open state where it can be paid, voided,
     * or cancelled.
     */
    public function finalize(): void
    {
        echo &quot;Invoice #{$this-&gt;invoice-&gt;getId()} finalized - changing from Draft to Open\n&quot;;
        $this-&gt;invoice-&gt;setState(new OpenInvoiceState($this-&gt;invoice));
    }

    public function getName(): string
    {
        return &#39;draft&#39;;
    }
}


/**
 * This Concrete State Represents an open invoice.
 *
 * This state represents an invoice that has been finalized and is ready for
 * processing. From this state, the invoice can be:
 * - Paid (moves to Paid state)
 * - Voided (moves to Void state)
 * - Cancelled (moves to Uncollectable state)
 */
class OpenInvoiceState extends BaseInvoiceState
{
    /**
     * Handle pay event
     *
     * When payment is received, the invoice transitions to the Paid state. This
     * is a terminal state - no further operations are allowed.
     */
    public function pay(): void
    {
        echo &quot;Invoice #{$this-&gt;invoice-&gt;getId()} paid - changing from Open to Paid\n&quot;;
        $this-&gt;invoice-&gt;setState(new PaidInvoiceState($this-&gt;invoice));
    }

    /**
     * Handle void event
     *
     * When an invoice is voided, it transitions to the Void state. This is a
     * terminal state - no further operations are allowed.
     */
    public function void(): void
    {
        echo &quot;Invoice #{$this-&gt;invoice-&gt;getId()} voided - changing from Open to Void\n&quot;;
        $this-&gt;invoice-&gt;setState(new VoidInvoiceState($this-&gt;invoice));
    }

    /**
     * Handle cancel event
     *
     * When an invoice is cancelled, it transitions to the Uncollectable state.
     * From Uncollectable, the invoice can still be paid or voided.
     */
    public function cancel(): void
    {
        echo &quot;Invoice #{$this-&gt;invoice-&gt;getId()} cancelled - changing from Open to Uncollectable\n&quot;;
        $this-&gt;invoice-&gt;setState(new UncollectableInvoiceState($this-&gt;invoice));
    }

    public function getName(): string
    {
        return &#39;open&#39;;
    }
}

/**
 * This Concrete State Represents a paid invoice.
 *
 * This is a terminal state representing a paid invoice. Once an invoice is
 * paid, no further state transitions are allowed. All event methods use the
 * default implementation which throws exceptions.
 */
class PaidInvoiceState extends BaseInvoiceState
{
    public function getName(): string
    {
        return &#39;paid&#39;;
    }
}

/**
 * This Concrete State Represents a void invoice.
 *
 * This is a terminal state representing a voided invoice. Once an invoice is
 * voided, no further state transitions are allowed. All event methods use the
 * default implementation which throws exceptions.
 */
class VoidInvoiceState extends BaseInvoiceState
{
    public function getName(): string
    {
        return &#39;void&#39;;
    }
}

/**
 * This Concrete State Represents a collectable invoice.
 *
 * This state represents an invoice that has been cancelled but can still be
 * recovered. From this state, the invoice can be:
 * - Paid (moves to Paid state)
 * - Voided (moves to Void state)
 *
 * This provides a way to handle invoices that were cancelled but later can be
 * collected or definitively written off.
 */
class UncollectableInvoiceState extends BaseInvoiceState
{
    /**
     * Handle pay event
     *
     * Even though the invoice was cancelled, payment can still be received.
     * This transitions the invoice to the Paid state.
     */
    public function pay(): void
    {
        echo &quot;Invoice #{$this-&gt;invoice-&gt;getId()} paid - changing from Uncollectable to Paid\n&quot;;
        $this-&gt;invoice-&gt;setState(new PaidInvoiceState($this-&gt;invoice));
    }

    /**
     * Handle void event
     *
     * If the invoice is definitively uncollectable, it can be voided. This
     * transitions the invoice to the Void state.
     */
    public function void(): void
    {
        echo &quot;Invoice #{$this-&gt;invoice-&gt;getId()} voided - changing from Uncollectable to Void\n&quot;;
        $this-&gt;invoice-&gt;setState(new VoidInvoiceState($this-&gt;invoice));
    }

    public function getName(): string
    {
        return &#39;uncollectable&#39;;
    }
}

/**
 * Context class - Invoice
 *
 * This is the context class in the State pattern. It maintains a reference to
 * the current state object and delegates all state-specific behavior to the
 * current state. The context is unaware of the specific state classes and
 * interacts with them through the abstract InvoiceState interface.
 *
 * The context also maintains the invoice&#39;s data (id, amount, etc.) that remains
 * constant regardless of the state.
 */
class Invoice
{
    private $id;
    private $amount;

    /**
     * Current state object
     *
     * This is the key component of the State pattern. The context maintains a
     * reference to the current state object and delegates all state-specific
     * operations to this object.
     *
     * @var InvoiceState
     */
    private $state;

    private $createdAt;

    /**
     * Constructor
     *
     * Creates a new invoice. The invoice always starts in the Draft state as
     * per business requirements.
     */
    public function __construct(int $id, float $amount)
    {
        $this-&gt;id = $id;
        $this-&gt;amount = $amount;
        $this-&gt;createdAt = new \DateTime();
        // Initial state is draft This is where the State pattern begins - we
        // set the initial state
        $this-&gt;state = new DraftInvoiceState($this);
    }

    public function getId(): int
    {
        return $this-&gt;id;
    }


    /**
     * Set the current state
     *
     * This method is called by state objects to transition to a new state. It&#39;s
     * the mechanism that allows the State pattern to work - states can change
     * the context&#39;s state by calling this method.
     *
     * @param InvoiceState $state The new state object
     */
    public function setState(InvoiceState $state)
    {
        $this-&gt;state = $state;
    }

    /**
     * Get the current state object
     *
     * @return InvoiceState
     */
    public function getState(): InvoiceState
    {
        return $this-&gt;state;
    }

    /**
     * Get the current state name
     *
     * This is a convenience method that delegates to the current state object.
     *
     * @return string
     */
    public function getStateName(): string
    {
        return $this-&gt;state-&gt;getName();
    }

    /**
     * Event method: finalize
     *
     * This method delegates the finalize operation to the current state. This
     * is the core of the State pattern - the context doesn&#39;t know how to handle
     * the operation, so it delegates to the current state.
     */
    public function finalize()
    {
        $this-&gt;state-&gt;finalize();
    }

    /**
     * Event method: pay
     *
     * This method delegates the pay operation to the current state. The
     * behavior will vary depending on the current state.
     */
    public function pay()
    {
        $this-&gt;state-&gt;pay();
    }

    /**
     * Event method: cancel
     *
     * This method delegates the cancel operation to the current state. The
     * behavior will vary depending on the current state.
     */
    public function cancel()
    {
        $this-&gt;state-&gt;cancel();
    }

    /**
     * Event method: void
     *
     * This method delegates the void operation to the current state. The
     * behavior will vary depending on the current state.
     */
    public function void()
    {
        $this-&gt;state-&gt;void();
    }

    /**
     * Get invoice information
     *
     * Returns an array with all invoice information including current state.
     * This is useful for debugging, logging, or API responses.
     *
     * @return array
     */
    public function getInfo(): array
    {
        return [
            &#39;id&#39; =&gt; $this-&gt;id,
            &#39;amount&#39; =&gt; $this-&gt;amount,
            &#39;state&#39; =&gt; $this-&gt;getStateName(),
            &#39;created_at&#39; =&gt; $this-&gt;createdAt-&gt;format(&#39;Y-m-d H:i:s&#39;)
        ];
    }
}


/**
 * Custom exception for invalid state transitions
 *
 * This exception is thrown when an invalid state transition is attempted. It
 * provides clear error messages about what transition was attempted and why it
 * failed.
 */
class InvalidStateTransitionException extends \Exception
{
    /**
     * Constructor
     *
     * @param string $message Error message
     * @param int $code Error code
     * @param \Exception|null $previous Previous exception
     */
    public function __construct($message = &quot;&quot;, $code = 0, \Exception $previous = null)
    {
        parent::__construct($message, $code, $previous);
    }
}


/**
 * ============================================================================
 * USAGE EXAMPLE AND DEMONSTRATION
 * ============================================================================
 *
 * The following code demonstrates how to use the State pattern implementation
 * with various scenarios that show all possible state transitions.
 */

try {
    echo &quot;=== Invoice State Pattern Demo ===\n\n&quot;;

    // Create a new invoice (starts in draft state)
    $invoice = new Invoice(1001, 1500.00);
    echo &quot;Created invoice: &quot; . json_encode($invoice-&gt;getInfo()) . &quot;\n\n&quot;;

    // Scenario 1: Draft -&gt; Open -&gt; Paid
    echo &quot;--- Scenario 1: Draft -&gt; Open -&gt; Paid ---\n&quot;;
    $invoice-&gt;finalize(); // Draft -&gt; Open
    echo &quot;Current state: &quot; . $invoice-&gt;getStateName() . &quot;\n&quot;;

    $invoice-&gt;pay(); // Open -&gt; Paid
    echo &quot;Current state: &quot; . $invoice-&gt;getStateName() . &quot;\n&quot;;

    // Try to pay again (should fail)
    try {
        $invoice-&gt;pay();
    } catch (InvalidStateTransitionException $e) {
        echo &quot;Expected error: &quot; . $e-&gt;getMessage() . &quot;\n&quot;;
    }

    echo &quot;\n--- Scenario 2: Draft -&gt; Open -&gt; Void ---\n&quot;;
    $invoice2 = new Invoice(1002, 750.00);
    $invoice2-&gt;finalize(); // Draft -&gt; Open
    $invoice2-&gt;void(); // Open -&gt; Void
    echo &quot;Invoice 2 state: &quot; . $invoice2-&gt;getStateName() . &quot;\n&quot;;

    echo &quot;\n--- Scenario 3: Draft -&gt; Open -&gt; Uncollectable -&gt; Paid ---\n&quot;;
    $invoice3 = new Invoice(1003, 2000.00);
    $invoice3-&gt;finalize(); // Draft -&gt; Open
    $invoice3-&gt;cancel(); // Open -&gt; Uncollectable
    echo &quot;Invoice 3 state: &quot; . $invoice3-&gt;getStateName() . &quot;\n&quot;;

    $invoice3-&gt;pay(); // Uncollectable -&gt; Paid
    echo &quot;Invoice 3 final state: &quot; . $invoice3-&gt;getStateName() . &quot;\n&quot;;

    echo &quot;\n--- Scenario 4: Draft -&gt; Open -&gt; Uncollectable -&gt; Void ---\n&quot;;
    $invoice4 = new Invoice(1004, 500.00);
    $invoice4-&gt;finalize(); // Draft -&gt; Open
    $invoice4-&gt;cancel(); // Open -&gt; Uncollectable
    $invoice4-&gt;void(); // Uncollectable -&gt; Void
    echo &quot;Invoice 4 final state: &quot; . $invoice4-&gt;getStateName() . &quot;\n&quot;;

    echo &quot;\n--- Error Scenario: Invalid transition ---\n&quot;;
    $invoice5 = new Invoice(1005, 300.00);
    try {
        $invoice5-&gt;pay(); // Try to pay draft invoice (should fail)
    } catch (InvalidStateTransitionException $e) {
        echo &quot;Expected error: &quot; . $e-&gt;getMessage() . &quot;\n&quot;;
    }

    echo &quot;\n--- State Information ---\n&quot;;
    echo &quot;Invoice 1: &quot; . json_encode($invoice-&gt;getInfo()) . &quot;\n&quot;;
    echo &quot;Invoice 2: &quot; . json_encode($invoice2-&gt;getInfo()) . &quot;\n&quot;;
    echo &quot;Invoice 3: &quot; . json_encode($invoice3-&gt;getInfo()) . &quot;\n&quot;;
    echo &quot;Invoice 4: &quot; . json_encode($invoice4-&gt;getInfo()) . &quot;\n&quot;;
    echo &quot;Invoice 5: &quot; . json_encode($invoice5-&gt;getInfo()) . &quot;\n&quot;;
} catch (InvalidStateTransitionException $e) {
    echo &quot;Error: &quot; . $e-&gt;getMessage() . &quot;\n&quot;;
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>=== Invoice State Pattern Demo ===

Created invoice: {&quot;id&quot;:1001,&quot;amount&quot;:1500,&quot;state&quot;:&quot;draft&quot;,&quot;created_at&quot;:&quot;2025-07-12 13:14:15&quot;}

--- Scenario 1: Draft -&gt; Open -&gt; Paid ---
Invoice #1001 finalized - changing from Draft to Open
Current state: open
Invoice #1001 paid - changing from Open to Paid
Current state: paid
Expected error: Cannot pay invoice in paid state

--- Scenario 2: Draft -&gt; Open -&gt; Void ---
Invoice #1002 finalized - changing from Draft to Open
Invoice #1002 voided - changing from Open to Void
Invoice 2 state: void

--- Scenario 3: Draft -&gt; Open -&gt; Uncollectable -&gt; Paid ---
Invoice #1003 finalized - changing from Draft to Open
Invoice #1003 cancelled - changing from Open to Uncollectable
Invoice 3 state: uncollectable
Invoice #1003 paid - changing from Uncollectable to Paid
Invoice 3 final state: paid

--- Scenario 4: Draft -&gt; Open -&gt; Uncollectable -&gt; Void ---
Invoice #1004 finalized - changing from Draft to Open
Invoice #1004 cancelled - changing from Open to Uncollectable
Invoice #1004 voided - changing from Uncollectable to Void
Invoice 4 final state: void

--- Error Scenario: Invalid transition ---
Expected error: Cannot pay invoice in draft state

--- State Information ---
Invoice 1: {&quot;id&quot;:1001,&quot;amount&quot;:1500,&quot;state&quot;:&quot;paid&quot;,&quot;created_at&quot;:&quot;2025-07-12 13:14:15&quot;}
Invoice 2: {&quot;id&quot;:1002,&quot;amount&quot;:750,&quot;state&quot;:&quot;void&quot;,&quot;created_at&quot;:&quot;2025-07-12 13:14:15&quot;}
Invoice 3: {&quot;id&quot;:1003,&quot;amount&quot;:2000,&quot;state&quot;:&quot;paid&quot;,&quot;created_at&quot;:&quot;2025-07-12 13:14:15&quot;}
Invoice 4: {&quot;id&quot;:1004,&quot;amount&quot;:500,&quot;state&quot;:&quot;void&quot;,&quot;created_at&quot;:&quot;2025-07-12 13:14:15&quot;}
Invoice 5: {&quot;id&quot;:1005,&quot;amount&quot;:300,&quot;state&quot;:&quot;draft&quot;,&quot;created_at&quot;:&quot;2025-07-12 13:14:15&quot;}</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[개념적인 예시](#example-0){#example-0-tab-bottom .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [실제 사례 예시](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### 다음을 보세요

[PHP로 작성된 전략 []{.fa
.fa-arrow-right}](../../strategy/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} PHP로 작성된
옵서버](../../observer/php/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **상태**

[![C#으로 작성된
상태](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 상태"){.prog-lang-link}
[![C++로 작성된
상태](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 상태"){.prog-lang-link}
[![Go로 작성된
상태](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 상태"){.prog-lang-link}
[![자바로 작성된
상태](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 상태"){.prog-lang-link}
[![파이썬으로 작성된
상태](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 상태"){.prog-lang-link}
[![루비로 작성된
상태](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 상태"){.prog-lang-link}
[![러스트로 작성된
상태](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 상태"){.prog-lang-link}
[![스위프트로 작성된
상태](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 상태"){.prog-lang-link}
[![타입스크립트로 작성된
상태](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 상태"){.prog-lang-link}
:::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
