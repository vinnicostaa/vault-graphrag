:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Memento](../../memento.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Memento](../../../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x"}
:::

# **Memento** em PHP {#memento-em-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
O **Memento** é um padrão de projeto comportamental que permite tirar um
"retrato" do estado de um objeto e restaurá-lo no futuro.

O Memento não compromete a estrutura interna do objeto com o qual
trabalha, nem os dados mantidos dentro dos retratos.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Memento ](../../memento.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegação
:::

::: en-intro
 [Introdução](#)
:::

::: en-intro
 [Exemplo conceitual](#example-0)
:::

::: en-file
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Exemplo do mundo real](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Complexidade:** []{.fa-stars}

**Popularidade:** []{.fa-stars}

**Exemplos de uso:** A aplicabilidade real do padrão Memento no PHP é
muito questionável. Na maioria dos casos, você pode facilitar a cópia do
estado de um objeto simplesmente usando serialização.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Exemplo conceitual](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Exemplo do mundo real](#example-1){#example-1-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Exemplo conceitual {#example-0-title}

Este exemplo ilustra a estrutura do padrão de projeto **Memento**. Ele
se concentra em responder a estas perguntas:

-   De quais classes ele consiste?
-   Quais papéis essas classes desempenham?
-   De que maneira os elementos do padrão estão relacionados?

Depois de aprender sobre a estrutura do padrão, será mais fácil entender
o exemplo a seguir, com base em um caso de uso PHP do mundo real.

#### []{#example-0--index-php .anchor} **index.php:** Exemplo conceitual

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Memento\Conceptual;

/**
 * The Originator holds some important state that may change over time. It also
 * defines a method for saving the state inside a memento and another method for
 * restoring the state from it.
 */
class Originator
{
    /**
     * @var string For the sake of simplicity, the originator&#39;s state is stored
     * inside a single variable.
     */
    private $state;

    public function __construct(string $state)
    {
        $this-&gt;state = $state;
        echo &quot;Originator: My initial state is: {$this-&gt;state}\n&quot;;
    }

    /**
     * The Originator&#39;s business logic may affect its internal state. Therefore,
     * the client should backup the state before launching methods of the
     * business logic via the save() method.
     */
    public function doSomething(): void
    {
        echo &quot;Originator: I&#39;m doing something important.\n&quot;;
        $this-&gt;state = $this-&gt;generateRandomString(30);
        echo &quot;Originator: and my state has changed to: {$this-&gt;state}\n&quot;;
    }

    private function generateRandomString(int $length = 10): string
    {
        return substr(
            str_shuffle(
                str_repeat(
                    $x = &#39;abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ&#39;,
                    ceil($length / strlen($x))
                )
            ),
            1,
            $length,
        );
    }

    /**
     * Saves the current state inside a memento.
     */
    public function save(): Memento
    {
        return new ConcreteMemento($this-&gt;state);
    }

    /**
     * Restores the Originator&#39;s state from a memento object.
     */
    public function restore(Memento $memento): void
    {
        $this-&gt;state = $memento-&gt;getState();
        echo &quot;Originator: My state has changed to: {$this-&gt;state}\n&quot;;
    }
}

/**
 * The Memento interface provides a way to retrieve the memento&#39;s metadata, such
 * as creation date or name. However, it doesn&#39;t expose the Originator&#39;s state.
 */
interface Memento
{
    public function getName(): string;

    public function getDate(): string;
}

/**
 * The Concrete Memento contains the infrastructure for storing the Originator&#39;s
 * state.
 */
class ConcreteMemento implements Memento
{
    private $state;

    private $date;

    public function __construct(string $state)
    {
        $this-&gt;state = $state;
        $this-&gt;date = date(&#39;Y-m-d H:i:s&#39;);
    }

    /**
     * The Originator uses this method when restoring its state.
     */
    public function getState(): string
    {
        return $this-&gt;state;
    }

    /**
     * The rest of the methods are used by the Caretaker to display metadata.
     */
    public function getName(): string
    {
        return $this-&gt;date . &quot; / (&quot; . substr($this-&gt;state, 0, 9) . &quot;...)&quot;;
    }

    public function getDate(): string
    {
        return $this-&gt;date;
    }
}

/**
 * The Caretaker doesn&#39;t depend on the Concrete Memento class. Therefore, it
 * doesn&#39;t have access to the originator&#39;s state, stored inside the memento. It
 * works with all mementos via the base Memento interface.
 */
class Caretaker
{
    /**
     * @var Memento[]
     */
    private $mementos = [];

    /**
     * @var Originator
     */
    private $originator;

    public function __construct(Originator $originator)
    {
        $this-&gt;originator = $originator;
    }

    public function backup(): void
    {
        echo &quot;\nCaretaker: Saving Originator&#39;s state...\n&quot;;
        $this-&gt;mementos[] = $this-&gt;originator-&gt;save();
    }

    public function undo(): void
    {
        if (!count($this-&gt;mementos)) {
            return;
        }
        $memento = array_pop($this-&gt;mementos);

        echo &quot;Caretaker: Restoring state to: &quot; . $memento-&gt;getName() . &quot;\n&quot;;
        try {
            $this-&gt;originator-&gt;restore($memento);
        } catch (\Exception $e) {
            $this-&gt;undo();
        }
    }

    public function showHistory(): void
    {
        echo &quot;Caretaker: Here&#39;s the list of mementos:\n&quot;;
        foreach ($this-&gt;mementos as $memento) {
            echo $memento-&gt;getName() . &quot;\n&quot;;
        }
    }
}

/**
 * Client code.
 */
$originator = new Originator(&quot;Super-duper-super-puper-super.&quot;);
$caretaker = new Caretaker($originator);

$caretaker-&gt;backup();
$originator-&gt;doSomething();

$caretaker-&gt;backup();
$originator-&gt;doSomething();

$caretaker-&gt;backup();
$originator-&gt;doSomething();

echo &quot;\n&quot;;
$caretaker-&gt;showHistory();

echo &quot;\nClient: Now, let&#39;s rollback!\n\n&quot;;
$caretaker-&gt;undo();

echo &quot;\nClient: Once more!\n\n&quot;;
$caretaker-&gt;undo();</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Originator: My initial state is: Super-duper-super-puper-super. 

Caretaker: Saving Originator&#39;s state...
Originator: I&#39;m doing something important.
Originator: and my state has changed to: srGIngezAEboNPDjBkuvymJKUtMSFX

Caretaker: Saving Originator&#39;s state...
Originator: I&#39;m doing something important.
Originator: and my state has changed to: UwCZQaHJOiERLlchyVuMbXNtpqTgWF

Caretaker: Saving Originator&#39;s state...
Originator: I&#39;m doing something important.
Originator: and my state has changed to: incqsdoJXkbDUuVOvRFYyKBgfzwZCQ

Caretaker: Here&#39;s the list of mementos:
2018-06-04 14:50:39 / (Super-dup...)
2018-06-04 14:50:39 / (srGIngezA...)
2018-06-04 14:50:39 / (UwCZQaHJO...)

Client: Now, let&#39;s rollback!

Caretaker: Restoring state to: 2018-06-04 14:50:39 / (UwCZQaHJO...)
Originator: My state has changed to: UwCZQaHJOiERLlchyVuMbXNtpqTgWF

Client: Once more!

Caretaker: Restoring state to: 2018-06-04 14:50:39 / (srGIngezA...)
Originator: My state has changed to: srGIngezAEboNPDjBkuvymJKUtMSFX</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Exemplo do mundo real {#example-1-title}

Neste exemplo, o padrão **Memento** é usado para implementar o fluxo de
gerenciamento de configurações de um aplicativo web em PHP. Muitos
aplicativos permitem que os usuários personalizem suas preferências e,
às vezes, eles querem desfazer alterações para restaurar a configuração
anterior. O padrão Memento pode ser usado para fornecer essa
funcionalidade capturando e armazenando o estado do objeto de
configuração antes que qualquer alteração seja feita.

#### []{#example-1--index-php .anchor} **index.php:** Exemplo do mundo real

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Memento\RealWorld;

/**
 * The Originator holds some important state that may change over time. It also
 * defines a method for saving the state inside a memento and another method for
 * restoring the state from it.
 *
 * In this example, the ConfigManager acts as the Originator. It manages
 * application configuration settings that can be modified during runtime. The
 * configuration might include database settings, feature flags, UI themes, SEO
 * settings, and other application parameters.
 *
 * The ConfigManager&#39;s business logic may affect its internal state. Therefore,
 * the client should backup the state before launching methods of the business
 * logic via the save() method.
 */
class ConfigManager
{
    /**
     * @var array For simplicity, the configuration state is stored inside an
     * array. In real applications, this might be a more complex structure with
     * validation, type checking, and nested configurations.
     */
    private $config;

    /**
     * Constructor initializes the configuration manager with default settings.
     *
     * @param array $initialConfig The initial configuration values
     */
    public function __construct(array $initialConfig)
    {
        $this-&gt;config = $initialConfig;
        echo &quot;ConfigManager: Initialized with &quot; . count($initialConfig) . &quot; config items.\n&quot;;
    }

    /**
     * The ConfigManager&#39;s business logic may affect its internal state.
     * Therefore, the client should backup the state before launching methods of
     * the business logic via the save() method.
     *
     * This method simulates updating configuration values, which is a common
     * operation in web applications (admin panels, user preferences, etc.).
     *
     * @param array $newValues New configuration values to merge with existing
     * config
     */
    public function updateConfig(array $newValues): void
    {
        echo &quot;ConfigManager: Updating configuration with new values...\n&quot;;
        $this-&gt;config = array_merge($this-&gt;config, $newValues);
        echo &quot;ConfigManager: Configuration updated. Current config has &quot; . count($this-&gt;config) . &quot; items.\n&quot;;
    }

    /**
     * Retrieves the current configuration state.
     *
     * @return array The current configuration
     */
    public function getConfig(): array
    {
        return $this-&gt;config;
    }

    /**
     * Saves the current state inside a memento.
     *
     * This method creates a snapshot of the current configuration state and
     * returns it wrapped in a memento object. The memento contains all the
     * information needed to restore the configuration to its current state
     * later.
     *
     * @return ConfigSnapshot A memento containing the current config state
     */
    public function save(): ConfigSnapshot
    {
        echo &quot;ConfigManager: Saving current configuration state...\n&quot;;
        return new ConfigSnapshot($this-&gt;config);
    }

    /**
     * Restores the ConfigManager&#39;s state from a memento object.
     *
     * This method takes a memento and restores the configuration to the state
     * that was saved in that memento. This is useful for implementing undo
     * functionality or rolling back failed configuration changes.
     *
     * @param ConfigSnapshot $snapshot The memento to restore from
     */
    public function restore(ConfigSnapshot $snapshot): void
    {
        $this-&gt;config = $snapshot-&gt;getState();
        echo &quot;ConfigManager: Configuration restored from snapshot.\n&quot;;
    }

    /**
     * Additional business method: Reset to defaults
     *
     * Resets the configuration to default values. This is another operation
     * that might benefit from creating a backup before execution.
     */
    public function resetToDefaults(): void
    {
        echo &quot;ConfigManager: Resetting configuration to defaults...\n&quot;;
        $this-&gt;config = [
            &#39;maintenance_mode&#39; =&gt; false,
            &#39;theme&#39; =&gt; &#39;light&#39;,
            &#39;debug&#39; =&gt; false
        ];
        echo &quot;ConfigManager: Configuration reset to defaults.\n&quot;;
    }

    /**
     * Additional business method: Enable maintenance mode
     *
     * Quickly enables maintenance mode across the application.
     */
    public function enableMaintenanceMode(): void
    {
        echo &quot;ConfigManager: Enabling maintenance mode...\n&quot;;
        $this-&gt;config[&#39;maintenance_mode&#39;] = true;
        $this-&gt;config[&#39;maintenance_message&#39;] = &#39;System under maintenance. Please try again later.&#39;;
        echo &quot;ConfigManager: Maintenance mode enabled.\n&quot;;
    }
}

/**
 * The Memento interface provides a way to retrieve the memento&#39;s metadata, such
 * as creation date or name. However, it doesn&#39;t expose the Originator&#39;s state.
 *
 * This interface ensures that external classes can work with mementos without
 * having direct access to the stored state. Only the originator should be able
 * to extract the actual state data.
 */
interface ConfigMemento
{
    /**
     * Returns a user-friendly name for this memento.
     *
     * @return string A descriptive name for this snapshot
     */
    public function getName(): string;

    /**
     * Returns the date when this memento was created.
     *
     * @return string The creation timestamp
     */
    public function getDate(): string;
}

/**
 * The Concrete Memento contains the infrastructure for storing the Originator&#39;s
 * state.
 *
 * This class stores a snapshot of the configuration state along with metadata
 * about when the snapshot was created. The actual state is stored privately and
 * can only be accessed by the originator through the getState() method.
 */
class ConfigSnapshot implements ConfigMemento
{
    /**
     * @var array The configuration state at the time this snapshot was created
     */
    private $state;

    /**
     * @var string The timestamp when this snapshot was created
     */
    private $date;

    /**
     * Constructor stores the provided state and records the current timestamp.
     *
     * @param array $state The configuration state to store
     */
    public function __construct(array $state)
    {
        $this-&gt;state = $state;
        $this-&gt;date = date(&#39;Y-m-d H:i:s&#39;);
        echo &quot;ConfigSnapshot: Created snapshot with &quot; . count($state) . &quot; config items.\n&quot;;
    }

    /**
     * The Originator uses this method when restoring its state.
     *
     * This method provides access to the stored state data. It should only be
     * called by the ConfigManager when restoring configuration.
     *
     * @return array The stored configuration state
     */
    public function getState(): array
    {
        return $this-&gt;state;
    }

    /**
     * The rest of the methods are used by the Caretaker to display metadata.
     *
     * Returns a descriptive name that includes the timestamp and a preview of
     * the configuration content.
     *
     * @return string A user-friendly name for this snapshot
     */
    public function getName(): string
    {
        $configCount = count($this-&gt;state);
        $maintenanceStatus = $this-&gt;state[&#39;maintenance_mode&#39;] ?? &#39;unknown&#39;;
        return $this-&gt;date . &quot; / ({$configCount} items, maintenance: {$maintenanceStatus})&quot;;
    }

    /**
     * Returns the creation date of this snapshot.
     *
     * @return string The timestamp when this snapshot was created
     */
    public function getDate(): string
    {
        return $this-&gt;date;
    }
}

/**
 * The Caretaker doesn&#39;t depend on the Concrete Memento class. Therefore, it
 * doesn&#39;t have access to the originator&#39;s state, stored inside the memento. It
 * works with all mementos via the base Memento interface.
 *
 * The ConfigHistory class manages a collection of configuration snapshots and
 * provides undo functionality. It demonstrates how the caretaker can manage
 * mementos without knowing their internal structure.
 */
class ConfigHistory
{
    /**
     * @var ConfigSnapshot[] Array of stored configuration snapshots
     */
    private $snapshots = [];

    /**
     * @var ConfigManager Reference to the configuration manager
     */
    private $configManager;

    /**
     * Constructor establishes the relationship with the originator.
     *
     * @param ConfigManager $configManager The configuration manager to work
     * with
     */
    public function __construct(ConfigManager $configManager)
    {
        $this-&gt;configManager = $configManager;
        echo &quot;ConfigHistory: History manager initialized.\n&quot;;
    }

    /**
     * Creates a backup of the current configuration state.
     *
     * This method asks the originator to create a memento and stores it in the
     * history. This should be called before making changes that might need to
     * be undone.
     */
    public function backup(): void
    {
        echo &quot;\nConfigHistory: Creating backup of current configuration...\n&quot;;
        $this-&gt;snapshots[] = $this-&gt;configManager-&gt;save();
        echo &quot;ConfigHistory: Backup created. Total backups: &quot; . count($this-&gt;snapshots) . &quot;\n&quot;;
    }

    /**
     * Restores the configuration to the most recent backup.
     *
     * This method retrieves the most recent memento from the history and asks
     * the originator to restore its state from that memento.
     */
    public function undo(): void
    {
        if (!count($this-&gt;snapshots)) {
            echo &quot;ConfigHistory: No backups available for undo.\n&quot;;
            return;
        }

        $memento = array_pop($this-&gt;snapshots);

        echo &quot;ConfigHistory: Restoring configuration to: &quot; . $memento-&gt;getName() . &quot;\n&quot;;
        try {
            $this-&gt;configManager-&gt;restore($memento);
            echo &quot;ConfigHistory: Undo completed successfully.\n&quot;;
        } catch (\Exception $e) {
            echo &quot;ConfigHistory: Undo failed, trying previous backup...\n&quot;;
            $this-&gt;undo();
        }
    }

    /**
     * Displays the history of all saved configuration snapshots.
     *
     * This method shows all available backups using only the information
     * available through the memento interface, without accessing the actual
     * configuration data.
     */
    public function showHistory(): void
    {
        echo &quot;\nConfigHistory: Available configuration backups:\n&quot;;
        if (empty($this-&gt;snapshots)) {
            echo &quot;No backups available.\n&quot;;
        } else {
            foreach ($this-&gt;snapshots as $index =&gt; $memento) {
                echo &quot;[{$index}] &quot; . $memento-&gt;getName() . &quot;\n&quot;;
            }
        }
        echo &quot;\n&quot;;
    }

    /**
     * Clears all stored backups.
     *
     * This method removes all mementos from the history, which might be useful
     * when starting fresh or to free up memory.
     */
    public function clearHistory(): void
    {
        $count = count($this-&gt;snapshots);
        $this-&gt;snapshots = [];
        echo &quot;ConfigHistory: Cleared {$count} backups from history.\n&quot;;
    }

    /**
     * Gets the number of available backups.
     *
     * @return int The number of stored backups
     */
    public function getBackupCount(): int
    {
        return count($this-&gt;snapshots);
    }
}



/**
 * ============================================================================
 * CLIENT CODE - DEMONSTRATION AND USAGE EXAMPLES
 * ============================================================================
 */

echo &quot;=== Configuration Manager with Memento Pattern Demo ===\n\n&quot;;

/**
 * Example 1: Basic configuration management with backup/restore
 */
echo &quot;--- Example 1: Basic Configuration Management ---\n&quot;;

// Initialize configuration manager with default settings
$config = new ConfigManager([
    &#39;maintenance_mode&#39; =&gt; false,
    &#39;theme&#39; =&gt; &#39;light&#39;,
    &#39;seo&#39; =&gt; [&#39;title&#39; =&gt; &#39;My Website&#39;, &#39;description&#39; =&gt; &#39;Welcome to my site!&#39;],
    &#39;debug&#39; =&gt; false,
    &#39;max_users&#39; =&gt; 1000
]);

// Create history manager
$history = new ConfigHistory($config);

echo &quot;\nInitial configuration:\n&quot;;
print_r($config-&gt;getConfig());

/**
 * Example 2: Making changes with backups
 */
echo &quot;\n--- Example 2: Making Changes with Backups ---\n&quot;;

// Create backup before making changes
$history-&gt;backup();

// Update theme settings
$config-&gt;updateConfig([
    &#39;theme&#39; =&gt; &#39;dark&#39;,
    &#39;theme_options&#39; =&gt; [&#39;sidebar&#39; =&gt; &#39;collapsed&#39;, &#39;font_size&#39; =&gt; &#39;large&#39;]
]);

echo &quot;\nAfter theme update:\n&quot;;
print_r($config-&gt;getConfig());

// Create another backup
$history-&gt;backup();

// Enable maintenance mode
$config-&gt;enableMaintenanceMode();

echo &quot;\nAfter enabling maintenance mode:\n&quot;;
print_r($config-&gt;getConfig());

/**
 * Example 3: Demonstrating undo functionality
 */
echo &quot;\n--- Example 3: Undo Functionality ---\n&quot;;

// Show current history
$history-&gt;showHistory();

// Undo last change (maintenance mode)
echo &quot;Undoing maintenance mode activation...\n&quot;;
$history-&gt;undo();

echo &quot;\nAfter first undo:\n&quot;;
print_r($config-&gt;getConfig());

// Undo theme changes
echo &quot;\nUndoing theme changes...\n&quot;;
$history-&gt;undo();

echo &quot;\nAfter second undo (back to original):\n&quot;;
print_r($config-&gt;getConfig());

/**
 * Example 4: Multiple configuration scenarios
 */
echo &quot;\n--- Example 4: Multiple Configuration Scenarios ---\n&quot;;

// Scenario A: SEO Configuration
$history-&gt;backup();
echo &quot;\nScenario A: Updating SEO settings...\n&quot;;
$config-&gt;updateConfig([
    &#39;seo&#39; =&gt; [
        &#39;title&#39; =&gt; &#39;Best Products Online&#39;,
        &#39;description&#39; =&gt; &#39;Find the best products at great prices!&#39;,
        &#39;keywords&#39; =&gt; &#39;products, online, shopping, deals&#39;
    ],
    &#39;analytics&#39; =&gt; [&#39;google_id&#39; =&gt; &#39;GA-123456&#39;, &#39;facebook_pixel&#39; =&gt; &#39;FB-789012&#39;]
]);

echo &quot;SEO configuration updated:\n&quot;;
print_r($config-&gt;getConfig());

// Scenario B: Performance Settings
$history-&gt;backup();
echo &quot;\nScenario B: Updating performance settings...\n&quot;;
$config-&gt;updateConfig([
    &#39;cache_enabled&#39; =&gt; true,
    &#39;cache_duration&#39; =&gt; 3600,
    &#39;compression&#39; =&gt; &#39;gzip&#39;,
    &#39;max_users&#39; =&gt; 2000
]);

echo &quot;Performance settings updated:\n&quot;;
print_r($config-&gt;getConfig());

// Scenario C: Emergency rollback
echo &quot;\nScenario C: Emergency rollback to SEO-only changes...\n&quot;;
$history-&gt;undo(); // Remove performance changes
echo &quot;Rolled back performance changes:\n&quot;;
print_r($config-&gt;getConfig());

/**
 * Example 5: Reset and recovery
 */
echo &quot;\n--- Example 5: Reset and Recovery ---\n&quot;;

// Save current state before reset
$history-&gt;backup();

// Reset configuration
$config-&gt;resetToDefaults();

echo &quot;\nAfter reset to defaults:\n&quot;;
print_r($config-&gt;getConfig());

// Restore previous configuration
echo &quot;\nRestoring previous configuration...\n&quot;;
$history-&gt;undo();

echo &quot;\nAfter restoration:\n&quot;;
print_r($config-&gt;getConfig());

/**
 * Example 6: History management
 */
echo &quot;\n--- Example 6: History Management ---\n&quot;;

// Show complete history
$history-&gt;showHistory();

echo &quot;Total backups available: &quot; . $history-&gt;getBackupCount() . &quot;\n&quot;;

// Clear history
echo &quot;\nClearing history...\n&quot;;
$history-&gt;clearHistory();

// Show history after clearing
$history-&gt;showHistory();

/**
 * Example 7: Real-world workflow simulation
 */
echo &quot;\n--- Example 7: Real-World Workflow Simulation ---\n&quot;;

// Simulate a typical configuration update workflow
echo &quot;Simulating typical admin workflow...\n\n&quot;;

// Step 1: Admin wants to update site for promotion
$history-&gt;backup();
echo &quot;Step 1: Preparing for Black Friday promotion...\n&quot;;
$config-&gt;updateConfig([
    &#39;promotion_banner&#39; =&gt; &#39;Black Friday Sale - 50% Off!&#39;,
    &#39;theme&#39; =&gt; &#39;dark&#39;,
    &#39;special_offers&#39; =&gt; [&#39;discount&#39; =&gt; 50, &#39;code&#39; =&gt; &#39;BLACKFRIDAY50&#39;]
]);

// Step 2: Update SEO for promotion
$history-&gt;backup();
echo &quot;\nStep 2: Updating SEO for promotion visibility...\n&quot;;
$config-&gt;updateConfig([
    &#39;seo&#39; =&gt; [
        &#39;title&#39; =&gt; &#39;Black Friday Sale - 50% Off Everything!&#39;,
        &#39;description&#39; =&gt; &#39;Huge Black Friday discounts on all products. Limited time offer!&#39;,
        &#39;keywords&#39; =&gt; &#39;black friday, sale, discount, deals, promotion&#39;
    ]
]);

// Step 3: Something goes wrong, need to rollback SEO changes only
echo &quot;\nStep 3: SEO changes caused issues, rolling back SEO only...\n&quot;;
$history-&gt;undo();

echo &quot;Final configuration after workflow:\n&quot;;
print_r($config-&gt;getConfig());

echo &quot;\nFinal history state:\n&quot;;
$history-&gt;showHistory();</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Resultado da execução

<figure class="code">
<pre class="code" lang="output"><code>=== Configuration Manager with Memento Pattern Demo ===

--- Example 1: Basic Configuration Management ---
ConfigManager: Initialized with 5 config items.
ConfigHistory: History manager initialized.

Initial configuration:
Array
(
    [maintenance_mode] =&gt; 
    [theme] =&gt; light
    [seo] =&gt; Array
        (
            [title] =&gt; My Website
            [description] =&gt; Welcome to my site!
        )

    [debug] =&gt; 
    [max_users] =&gt; 1000
)

--- Example 2: Making Changes with Backups ---

ConfigHistory: Creating backup of current configuration...
ConfigManager: Saving current configuration state...
ConfigSnapshot: Created snapshot with 5 config items.
ConfigHistory: Backup created. Total backups: 1
ConfigManager: Updating configuration with new values...
ConfigManager: Configuration updated. Current config has 6 items.

After theme update:
Array
(
    [maintenance_mode] =&gt; 
    [theme] =&gt; dark
    [seo] =&gt; Array
        (
            [title] =&gt; My Website
            [description] =&gt; Welcome to my site!
        )

    [debug] =&gt; 
    [max_users] =&gt; 1000
    [theme_options] =&gt; Array
        (
            [sidebar] =&gt; collapsed
            [font_size] =&gt; large
        )

)

ConfigHistory: Creating backup of current configuration...
ConfigManager: Saving current configuration state...
ConfigSnapshot: Created snapshot with 6 config items.
ConfigHistory: Backup created. Total backups: 2
ConfigManager: Enabling maintenance mode...
ConfigManager: Maintenance mode enabled.

After enabling maintenance mode:
Array
(
    [maintenance_mode] =&gt; 1
    [theme] =&gt; dark
    [seo] =&gt; Array
        (
            [title] =&gt; My Website
            [description] =&gt; Welcome to my site!
        )

    [debug] =&gt; 
    [max_users] =&gt; 1000
    [theme_options] =&gt; Array
        (
            [sidebar] =&gt; collapsed
            [font_size] =&gt; large
        )

    [maintenance_message] =&gt; System under maintenance. Please try again later.
)

--- Example 3: Undo Functionality ---

ConfigHistory: Available configuration backups:
[0] 2025-07-28 15:04:16 / (5 items, maintenance: )
[1] 2025-07-28 15:04:16 / (6 items, maintenance: )

Undoing maintenance mode activation...
ConfigHistory: Restoring configuration to: 2025-07-28 15:04:16 / (6 items, maintenance: )
ConfigManager: Configuration restored from snapshot.
ConfigHistory: Undo completed successfully.

After first undo:
Array
(
    [maintenance_mode] =&gt; 
    [theme] =&gt; dark
    [seo] =&gt; Array
        (
            [title] =&gt; My Website
            [description] =&gt; Welcome to my site!
        )

    [debug] =&gt; 
    [max_users] =&gt; 1000
    [theme_options] =&gt; Array
        (
            [sidebar] =&gt; collapsed
            [font_size] =&gt; large
        )

)

Undoing theme changes...
ConfigHistory: Restoring configuration to: 2025-07-28 15:04:16 / (5 items, maintenance: )
ConfigManager: Configuration restored from snapshot.
ConfigHistory: Undo completed successfully.

After second undo (back to original):
Array
(
    [maintenance_mode] =&gt; 
    [theme] =&gt; light
    [seo] =&gt; Array
        (
            [title] =&gt; My Website
            [description] =&gt; Welcome to my site!
        )

    [debug] =&gt; 
    [max_users] =&gt; 1000
)

--- Example 4: Multiple Configuration Scenarios ---

ConfigHistory: Creating backup of current configuration...
ConfigManager: Saving current configuration state...
ConfigSnapshot: Created snapshot with 5 config items.
ConfigHistory: Backup created. Total backups: 1

Scenario A: Updating SEO settings...
ConfigManager: Updating configuration with new values...
ConfigManager: Configuration updated. Current config has 6 items.
SEO configuration updated:
Array
(
    [maintenance_mode] =&gt; 
    [theme] =&gt; light
    [seo] =&gt; Array
        (
            [title] =&gt; Best Products Online
            [description] =&gt; Find the best products at great prices!
            [keywords] =&gt; products, online, shopping, deals
        )

    [debug] =&gt; 
    [max_users] =&gt; 1000
    [analytics] =&gt; Array
        (
            [google_id] =&gt; GA-123456
            [facebook_pixel] =&gt; FB-789012
        )

)

ConfigHistory: Creating backup of current configuration...
ConfigManager: Saving current configuration state...
ConfigSnapshot: Created snapshot with 6 config items.
ConfigHistory: Backup created. Total backups: 2

Scenario B: Updating performance settings...
ConfigManager: Updating configuration with new values...
ConfigManager: Configuration updated. Current config has 9 items.
Performance settings updated:
Array
(
    [maintenance_mode] =&gt; 
    [theme] =&gt; light
    [seo] =&gt; Array
        (
            [title] =&gt; Best Products Online
            [description] =&gt; Find the best products at great prices!
            [keywords] =&gt; products, online, shopping, deals
        )

    [debug] =&gt; 
    [max_users] =&gt; 2000
    [analytics] =&gt; Array
        (
            [google_id] =&gt; GA-123456
            [facebook_pixel] =&gt; FB-789012
        )

    [cache_enabled] =&gt; 1
    [cache_duration] =&gt; 3600
    [compression] =&gt; gzip
)

Scenario C: Emergency rollback to SEO-only changes...
ConfigHistory: Restoring configuration to: 2025-07-28 15:04:16 / (6 items, maintenance: )
ConfigManager: Configuration restored from snapshot.
ConfigHistory: Undo completed successfully.
Rolled back performance changes:
Array
(
    [maintenance_mode] =&gt; 
    [theme] =&gt; light
    [seo] =&gt; Array
        (
            [title] =&gt; Best Products Online
            [description] =&gt; Find the best products at great prices!
            [keywords] =&gt; products, online, shopping, deals
        )

    [debug] =&gt; 
    [max_users] =&gt; 1000
    [analytics] =&gt; Array
        (
            [google_id] =&gt; GA-123456
            [facebook_pixel] =&gt; FB-789012
        )

)

--- Example 5: Reset and Recovery ---

ConfigHistory: Creating backup of current configuration...
ConfigManager: Saving current configuration state...
ConfigSnapshot: Created snapshot with 6 config items.
ConfigHistory: Backup created. Total backups: 2
ConfigManager: Resetting configuration to defaults...
ConfigManager: Configuration reset to defaults.

After reset to defaults:
Array
(
    [maintenance_mode] =&gt; 
    [theme] =&gt; light
    [debug] =&gt; 
)

Restoring previous configuration...
ConfigHistory: Restoring configuration to: 2025-07-28 15:04:16 / (6 items, maintenance: )
ConfigManager: Configuration restored from snapshot.
ConfigHistory: Undo completed successfully.

After restoration:
Array
(
    [maintenance_mode] =&gt; 
    [theme] =&gt; light
    [seo] =&gt; Array
        (
            [title] =&gt; Best Products Online
            [description] =&gt; Find the best products at great prices!
            [keywords] =&gt; products, online, shopping, deals
        )

    [debug] =&gt; 
    [max_users] =&gt; 1000
    [analytics] =&gt; Array
        (
            [google_id] =&gt; GA-123456
            [facebook_pixel] =&gt; FB-789012
        )

)

--- Example 6: History Management ---

ConfigHistory: Available configuration backups:
[0] 2025-07-28 15:04:16 / (5 items, maintenance: )

Total backups available: 1

Clearing history...
ConfigHistory: Cleared 1 backups from history.

ConfigHistory: Available configuration backups:
No backups available.


--- Example 7: Real-World Workflow Simulation ---
Simulating typical admin workflow...


ConfigHistory: Creating backup of current configuration...
ConfigManager: Saving current configuration state...
ConfigSnapshot: Created snapshot with 6 config items.
ConfigHistory: Backup created. Total backups: 1
Step 1: Preparing for Black Friday promotion...
ConfigManager: Updating configuration with new values...
ConfigManager: Configuration updated. Current config has 8 items.

ConfigHistory: Creating backup of current configuration...
ConfigManager: Saving current configuration state...
ConfigSnapshot: Created snapshot with 8 config items.
ConfigHistory: Backup created. Total backups: 2

Step 2: Updating SEO for promotion visibility...
ConfigManager: Updating configuration with new values...
ConfigManager: Configuration updated. Current config has 8 items.

Step 3: SEO changes caused issues, rolling back SEO only...
ConfigHistory: Restoring configuration to: 2025-07-28 15:04:16 / (8 items, maintenance: )
ConfigManager: Configuration restored from snapshot.
ConfigHistory: Undo completed successfully.
Final configuration after workflow:
Array
(
    [maintenance_mode] =&gt; 
    [theme] =&gt; dark
    [seo] =&gt; Array
        (
            [title] =&gt; Best Products Online
            [description] =&gt; Find the best products at great prices!
            [keywords] =&gt; products, online, shopping, deals
        )

    [debug] =&gt; 
    [max_users] =&gt; 1000
    [analytics] =&gt; Array
        (
            [google_id] =&gt; GA-123456
            [facebook_pixel] =&gt; FB-789012
        )

    [promotion_banner] =&gt; Black Friday Sale - 50% Off!
    [special_offers] =&gt; Array
        (
            [discount] =&gt; 50
            [code] =&gt; BLACKFRIDAY50
        )

)

Final history state:

ConfigHistory: Available configuration backups:
[0] 2025-07-28 15:04:16 / (6 items, maintenance: )</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Exemplo conceitual](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Exemplo do mundo
real](#example-1){#example-1-tab-bottom .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Leia a seguir

[Observer em PHP []{.fa
.fa-arrow-right}](../../observer/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Mediator em
PHP](../../mediator/php/example.html){.btn .btn-default rel="prev"}
:::

## **Memento** em outras linguagens

[![Memento em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Memento em C#"){.prog-lang-link}
[![Memento em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Memento em C++"){.prog-lang-link}
[![Memento em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Memento em Go"){.prog-lang-link}
[![Memento em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Memento em Java"){.prog-lang-link}
[![Memento em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Memento em Python"){.prog-lang-link}
[![Memento em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Memento em Ruby"){.prog-lang-link}
[![Memento em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Memento em Rust"){.prog-lang-link}
[![Memento em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Memento em Swift"){.prog-lang-link}
[![Memento em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Memento em TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pt-br" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Arquivo com exemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compre o eBook **Mergulho nos Padrões de Projeto** e tenha acesso ao
arquivo com dúzias de exemplos detalhados que podem ser abertos
diretamente em seu IDE.

[ Saiba mais...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
