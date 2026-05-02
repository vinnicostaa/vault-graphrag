::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} / [Padrões
comportamentais](behavioral-patterns.html){.type}
:::

# Command {#command .title}

::: aka
Também conhecido como:
[Comando, ]{style="display:inline-block"}[Ação, ]{style="display:inline-block"}[Action, ]{style="display:inline-block"}[Transação, ]{style="display:inline-block"}[Transaction]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

O **Command** é um padrão de projeto comportamental que transforma um
pedido em um objeto independente que contém toda a informação sobre o
pedido. Essa transformação permite que você parameterize métodos com
diferentes pedidos, atrase ou coloque a execução do pedido em uma fila,
e suporte operações que não podem ser feitas.

<figure class="image">
<img
src="../../images/patterns/content/command/command-pt-br9a1d.png?id=36096f8c2cd7783284eb80ce92db1a96"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/command/command-pt-br.png?id=36096f8c2cd7783284eb80ce92db1a96 640w,/images/patterns/content/command/command-pt-br-1.5x.png?id=2494268fd1573a0d8961f496091fa358 960w,/images/patterns/content/command/command-pt-br-2x.png?id=51156130ac5860b2b9146f63e872cce3 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Padrão de projeto Command" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagine que você está trabalhando em uma nova aplicação de editor de
texto. Sua tarefa atual é criar uma barra de tarefas com vários botões
para várias operações do editor. Você criou uma classe `Botão` muito
bacana que pode ser usada para botões na barra de tarefas, bem como para
botões genéricos de diversas caixas de diálogo.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem163e2.png?id=84189315a0e8d91da262792979005ab4"
style="aspect-ratio: 0.88;"
srcset="/images/patterns/diagrams/command/problem1.png?id=84189315a0e8d91da262792979005ab4 230w,/images/patterns/diagrams/command/problem1-1.5x.png?id=77bf0bc58e5c9c9b8369bc4f8dec457f 345w,/images/patterns/diagrams/command/problem1-2x.png?id=af4c4e9c1d1b4fa2c4104c5f6bb18719 460w"
sizes="(max-width: 720px) 100vw, 230px" loading="lazy" width="230"
alt="Problema resolvido pelo padrão Command" />
<figcaption><p>Todos os botões de uma aplicação são derivadas de uma
mesma classe.</p></figcaption>
</figure>

Embora todos esses botões pareçam similares, eles todos devem fazer
coisas diferentes. Aonde você deveria colocar o código para os vários
handlers de cliques desses botões? A solução mais simples é criar um
monte de subclasses para cada local que o botão for usado. Essas
subclasses conteriam o código que teria que ser executado em um clique
de botão.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem2f2f9.png?id=f0e33da1842b3a3ee3b4857de0b6ec93"
style="aspect-ratio: 2.11;"
srcset="/images/patterns/diagrams/command/problem2.png?id=f0e33da1842b3a3ee3b4857de0b6ec93 400w,/images/patterns/diagrams/command/problem2-1.5x.png?id=7ae15e2b07d5d076a878d99c3ed8ac32 600w,/images/patterns/diagrams/command/problem2-2x.png?id=5eea7d0f6247da011150bb63e423f405 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="Várias subclasses de botões" />
<figcaption><p>Várias subclasses de botões. O que pode
dar errado?</p></figcaption>
</figure>

Não demora muito e você percebe que essa abordagem é falha. Primeiro
você tem um enorme número de subclasses, e isso seria okay se você não
arriscasse quebrar o código dentro dessas subclasses cada vez que você
modificar a classe base `Botão`. Colocando em miúdos: seu código GUI se
torna absurdamente dependente de um código volátil da lógica do negócio.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem3-pt-br1820.png?id=65467aa7a8e9545ba387cf0f62ed52cc"
style="aspect-ratio: 4.36;"
srcset="/images/patterns/diagrams/command/problem3-pt-br.png?id=65467aa7a8e9545ba387cf0f62ed52cc 480w,/images/patterns/diagrams/command/problem3-pt-br-1.5x.png?id=bcbadbc34f2b0e9aa6ca4c8bd9385279 720w,/images/patterns/diagrams/command/problem3-pt-br-2x.png?id=f91f74e46951797c12cc37f61eb27d46 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Várias classes implementam a mesma funcionalidade" />
<figcaption><p>Várias classes implementam a
mesma funcionalidade.</p></figcaption>
</figure>

E aqui está a parte mais feia. Algumas operações, tais como copiar/colar
texto, precisariam ser invocadas de diversos lugares. Por exemplo, um
usuário poderia criar um pequeno botão "Copiar" na barra de ferramentas,
ou copiar alguma coisa através do menu de contexto, ou apenas apertando
`Crtl+C` no teclado.

Inicialmente, quando sua aplicação só tinha a barra de ferramentas, tudo
bem colocar a implementação de várias operações dentro das subclasses do
botão. Em outras palavras, ter o código de cópia de texto dentro da
subclasse `BotãoCópia` parecia certo. Mas então, quando você implementou
menus de contexto, atalhos, e outras coisas, você teve que ou duplicar o
código da operação em muitas classes ou fazer menus dependentes de
botões, o que é uma opção ainda pior.
:::

::: {.section .solution}
##  Solução {#solution}

Um bom projeto de software quase sempre se baseia no princípio da
separação de interesses, o que geralmente resulta em dividir a aplicação
em camadas. O exemplo mais comum: uma camada para a interface gráfica do
usuário e outra camada para a lógica do negócio. A camada GUI é
responsável por renderizar uma bonita imagem na tela, capturando
quaisquer dados e mostrando resultados do que o usuário e a aplicação
estão fazendo. Contudo, quando se trata de fazer algo importante, como
calcular a trajetória da lua ou compor um relatório anual, a camada GUI
delega o trabalho para a camada inferior da lógica do negócio.

Dentro do código pode parecer assim: um objeto GUI chama um método da
lógica do negócio, passando alguns argumentos. Este processo é
geralmente descrito como um objeto mandando um *pedido* para outro.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution1-pt-br5b1b.png?id=16cf91c77bfa26438a6992e25bd73a02"
style="aspect-ratio: 2.47;"
srcset="/images/patterns/diagrams/command/solution1-pt-br.png?id=16cf91c77bfa26438a6992e25bd73a02 470w,/images/patterns/diagrams/command/solution1-pt-br-1.5x.png?id=9142864b86ab2cea9ad8620cfa28e04b 705w,/images/patterns/diagrams/command/solution1-pt-br-2x.png?id=7409772f1849947aeaf868bcd0129fe0 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="A camada GUI pode acessar a camada da lógica do negócio diretamente" />
<figcaption><p>Os objetos GUI podem acessar os objetos da lógica do
negócio diretamente.</p></figcaption>
</figure>

O padrão Command sugere que os objetos GUI não enviem esses pedidos
diretamente. Ao invés disso, você deve extrair todos os detalhes do
pedido, tais como o objeto a ser chamado, o nome do método, e a lista de
argumentos em uma classe *comando* separada que tem apenas um método que
aciona esse pedido.

Objetos comando servem como links entre vários objetos GUI e de lógica
de negócio. De agora em diante, o objeto GUI não precisa saber qual
objeto de lógica do negócio irá receber o pedido e como ele vai ser
processado. O objeto GUI deve acionar o comando, que irá lidar com todos
os detalhes.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution2-pt-br8411.png?id=b88e0eda96b684286e09fdd76c1c7b45"
style="aspect-ratio: 2.89;"
srcset="/images/patterns/diagrams/command/solution2-pt-br.png?id=b88e0eda96b684286e09fdd76c1c7b45 550w,/images/patterns/diagrams/command/solution2-pt-br-1.5x.png?id=2fa2c7e4a738d85a443edbf715826f93 825w,/images/patterns/diagrams/command/solution2-pt-br-2x.png?id=1fe72ad5955eca0af48cca23e6997134 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Acessando a lógica de negócio através do comando." />
<figcaption><p>Acessando a lógica do negócio através
do comando.</p></figcaption>
</figure>

O próximo passo é fazer seus comandos implementarem a mesma interface.
Geralmente é apenas um método de execução que não pega parâmetros. Essa
interface permite que você use vários comandos com o mesmo remetente do
pedido, sem acoplá-lo com as classes concretas dos comandos. Como um
bônus, agora você pode trocar os objetos comando ligados ao remetente,
efetivamente mudando o comportamento do remetente no momento da
execução.

Você pode ter notado uma peça faltante nesse quebra cabeças, que são os
parâmetros do pedido. Um objeto GUI pode ter fornecido ao objeto da
camada de negócio com alguns parâmetros, como deveremos passar os
detalhes do pedido para o destinatário? Parece que o comando deve ser ou
pré configurado com esses dados, ou ser capaz de obtê-los por conta
própria.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution3-pt-brfb9f.png?id=fa9bc757cceb92fd03e9b8a49fd49a93"
style="aspect-ratio: 1.83;"
srcset="/images/patterns/diagrams/command/solution3-pt-br.png?id=fa9bc757cceb92fd03e9b8a49fd49a93 440w,/images/patterns/diagrams/command/solution3-pt-br-1.5x.png?id=9c8efe3022180a036fd0bb89d1b5e460 660w,/images/patterns/diagrams/command/solution3-pt-br-2x.png?id=b398836186fcdea28e04b915b85f8695 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="Os objetos GUI delegam o trabalho aos comandos" />
<figcaption><p>Os objetos GUI delegam o trabalho
aos comandos.</p></figcaption>
</figure>

Vamos voltar ao nosso editor de texto. Após aplicarmos o padrão Command,
nós não mais precisamos de todas aquelas subclasses de botões para
implementar vários comportamentos de cliques. É suficiente colocar
apenas um campo na classe `Botão` base que armazena a referência para um
objeto comando e faz o botão executar aquele comando com um clique.

Você vai implementar um monte de classes comando para cada possível
operação e ligá-los aos seus botões em particular, dependendo do
comportamento desejado para os botões.

Outros elementos GUI, tais como menus, atalhos, ou caixas de diálogo
inteiras, podem ser implementados da mesma maneira. Eles serão ligados a
um comando que será executado quando um usuário interage com um elemento
GUI. Como você provavelmente adivinhou, os elementos relacionados a
mesma operação serão ligados aos mesmos comandos, prevenindo a
duplicação de código.

Como resultado, os comandos se tornam uma camada intermédia conveniente
que reduz o acoplamento entre as camadas GUI e de lógica do negócio. E
isso é apenas uma fração dos benefícios que o padrão Command pode
oferecer.
:::

::: {.section .analogy}
##  Analogia com o mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/command/command-comic-16e6f.png?id=551df832f445080976f3116e0dc120c9"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/command/command-comic-1.png?id=551df832f445080976f3116e0dc120c9 600w,/images/patterns/content/command/command-comic-1-1.5x.png?id=82dc5c38ce372ed4dfd4a37c04faeb72 900w,/images/patterns/content/command/command-comic-1-2x.png?id=47b3c00b2cfbda7157a1ed9da6ce2948 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Fazendo um pedido em um restaurante" />
<figcaption><p>Fazendo um pedido em um restaurante.</p></figcaption>
</figure>

Após uma longa caminhada pela cidade, você chega a um restaurante bacana
e senta numa mesa perto da janela. Um garçom amigável se aproxima e
rapidamente recebe seu pedido, escrevendo-o em um pedaço de papel. O
garçom vai até a cozinha e prende o pedido em uma parede. Após algum
tempo, o pedido chega até o chef, que o lê e cozinha a refeição de
acordo. O cozinheiro coloca a refeição em uma bandeja junto com o
pedido. O garçom acha a bandeja, verifica o pedido para garantir que é
aquilo que você queria, e o traz para sua mesa.

O pedido no papel serve como um comando. Ele permanece em uma fila até
que o chef esteja pronto para serví-lo. O pedido contém todas as
informações relevantes necessárias para cozinhar a refeição. Ele permite
ao chef começar a cozinhar imediatamente ao invés de ter que ir até você
para clarificar os detalhes do pedido pessoalmente.
:::

::::: {.section .structure-container}
##  Estrutura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/command/structure44ec.png?id=1cd7833638f4c43630f4a84017d31195"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.7;"
srcset="/images/patterns/diagrams/command/structure.png?id=1cd7833638f4c43630f4a84017d31195 630w,/images/patterns/diagrams/command/structure-1.5x.png?id=6e5b68277c7747b22266e408891d5841 945w,/images/patterns/diagrams/command/structure-2x.png?id=1dfaa84354ffe49ef7ad46ce069482d2 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="Estrutura do padrão de projeto Command" /><img
src="../../images/patterns/diagrams/command/structure-indexed49b8.png?id=95529d7282dc7bc1c5bc443423b1cf4f"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/command/structure-indexed.png?id=95529d7282dc7bc1c5bc443423b1cf4f 640w,/images/patterns/diagrams/command/structure-indexed-1.5x.png?id=29d6c5c7a06da2747ed756c0ddad6169 960w,/images/patterns/diagrams/command/structure-indexed-2x.png?id=e4cc286a44768c7d060af33497da7df6 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Estrutura do padrão de projeto Command" />
</figure>
:::

1.  A classe **Remetente** (também conhecida como *invocadora*) é
    responsável por iniciar os pedidos. Essa classe deve ter um campo
    para armazenar a referência para um objeto comando. O remetente
    aciona aquele comando ao invés de enviar o pedido diretamente para o
    destinatário. Observe que o remetente não é responsável por criar o
    objeto comando. Geralmente ele é pré criado através de um construtor
    do cliente.

2.  A interface **Comando** geralmente declara apenas um único método
    para executar o comando.

3.  **Comandos Concretos** implementam vários tipos de pedidos. Um
    comando concreto não deve realizar o trabalho por conta própria, mas
    passar a chamada para um dos objetos da lógica do negócio. Contudo,
    para simplificar o código, essas classes podem ser fundidas.

    Os parâmetros necessários para executar um método em um objeto
    destinatário podem ser declarados como campos no comando concreto.
    Você pode tornar os objetos comando imutáveis ao permitir que apenas
    inicializem esses campos através do construtor.

4.  A classe **Destinatária** contém a lógica do negócio. Quase qualquer
    objeto pode servir como um destinatário. A maioria dos comandos
    apenas lida com os detalhes de como um pedido é passado para o
    destinatário, enquanto que o destinatário em si executa o verdadeiro
    trabalho.

5.  O **Cliente** cria e configura objetos comando concretos. O cliente
    deve passar todos os parâmetros do pedido, incluindo uma instância
    do destinatário, para o construtor do comando. Após isso, o comando
    resultante pode ser associado com um ou múltiplos destinatários.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Neste exemplo, o padrão **Command** ajuda a manter um registro da
história de operações executadas e torna possível reverter uma operação
se necessário.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/example31bf.png?id=1f42c8395fe54d0e409026b91881e2a0"
style="aspect-ratio: 1.07;"
srcset="/images/patterns/diagrams/command/example.png?id=1f42c8395fe54d0e409026b91881e2a0 640w,/images/patterns/diagrams/command/example-1.5x.png?id=435055a78a82c005eebb1bef869998ae 960w,/images/patterns/diagrams/command/example-2x.png?id=ed44dfd9b02eccf1ae7e2714d018ed36 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Exemplo de estrutura do padrão Command" />
<figcaption><p>Operações não executáveis em um editor
de texto.</p></figcaption>
</figure>

Os comandos que resultam de mudanças de estado do editor (por exemplo,
cortando e colando) fazem uma cópia de backup do estado do editor antes
de executarem uma operação associada com o comando. Após o comando ser
executado, ele é colocado em um histórico de comando (uma pilha de
objetos comando) junto com a cópia de backup do estado do editor naquele
momento. Mais tarde, se o usuário precisa reverter uma operação, a
aplicação pode pegar o comando mais recente do histórico, ler o backup
associado ao estado do editor, e restaurá-lo.

O código cliente (elementos GUI, histórico de comandos, etc.) não é
acoplado às classes comando concretas porque ele trabalha com comandos
através da interface comando. Essa abordagem permite que você introduza
novos comandos na aplicação sem quebrar o código existente.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// A classe comando base define a interface comum para todos
// comandos concretos.
abstract class Command is
    protected field app: Application
    protected field editor: Editor
    protected field backup: text

    constructor Command(app: Application, editor: Editor) is
        this.app = app
        this.editor = editor

    // Faz um backup do estado do editor.
    method saveBackup() is
        backup = editor.text

    // Restaura o estado do editor.
    method undo() is
        editor.text = backup

    // O método de execução é declarado abstrato para forçar
    // todos os comandos concretos a fornecer suas próprias
    // implementações. O método deve retornar verdadeiro ou
    // falso dependendo se o comando muda o estado do editor.
    abstract method execute()


// Comandos concretos vêm aqui.
class CopyCommand extends Command is
    // O comando copy (copiar) não é salvo no histórico já que
    // não muda o estado do editor.
    method execute() is
        app.clipboard = editor.getSelection()
        return false

class CutCommand extends Command is
    // O comando cut (cortar) muda o estado do editor, portanto
    // deve ser salvo no histórico. E ele será salvo desde que o
    // método retorne verdadeiro.
    method execute() is
        saveBackup()
        app.clipboard = editor.getSelection()
        editor.deleteSelection()
        return true

class PasteCommand extends Command is
    method execute() is
        saveBackup()
        editor.replaceSelection(app.clipboard)
        return true

// A operação undo (desfazer) também é um comando.
class UndoCommand extends Command is
    method execute() is
        app.undo()
        return false


// O comando global history (histórico) é apenas uma pilha.
class CommandHistory is
    private field history: array of Command

    // Último a entrar...
    method push(c: Command) is
        // Empurra o comando para o fim do vetor do histórico.

    // ...primeiro a sair.
    method pop():Command is
        // Obter o comando mais recente do histórico.


// A classe do editor tem verdadeiras operações de edição de
// texto. Ela faz o papel de destinatária: todos os comandos
// acabam delegando a execução para os métodos do editor.
class Editor is
    field text: string

    method getSelection() is
        // Retorna o texto selecionado.

    method deleteSelection() is
        // Deleta o texto selecionado.

    method replaceSelection(text) is
        // Insere os conteúdos da área de transferência na
        // posição atual.


// A classe da aplicação define as relações de objeto. Ela age
// como uma remetente: quando alguma coisa precisa ser feita,
// ela cria um objeto comando e executa ele.
class Application is
    field clipboard: string
    field editors: array of Editors
    field activeEditor: Editor
    field history: CommandHistory

    // O código que assinala comandos para objetos UI pode se
    // parecer como este.
    method createUI() is
        // ...
        copy = function() { executeCommand(
            new CopyCommand(this, activeEditor)) }
        copyButton.setCommand(copy)
        shortcuts.onKeyPress(&quot;Ctrl+C&quot;, copy)

        cut = function() { executeCommand(
            new CutCommand(this, activeEditor)) }
        cutButton.setCommand(cut)
        shortcuts.onKeyPress(&quot;Ctrl+X&quot;, cut)

        paste = function() { executeCommand(
            new PasteCommand(this, activeEditor)) }
        pasteButton.setCommand(paste)
        shortcuts.onKeyPress(&quot;Ctrl+V&quot;, paste)

        undo = function() { executeCommand(
            new UndoCommand(this, activeEditor)) }
        undoButton.setCommand(undo)
        shortcuts.onKeyPress(&quot;Ctrl+Z&quot;, undo)

    // Executa um comando e verifica se ele foi adicionado ao
    // histórico.
    method executeCommand(command) is
        if (command.execute())
            history.push(command)

    // Pega o comando mais recente do histórico e executa seu
    // método undo(desfazer). Observe que nós não sabemos a
    // classe desse comando. Mas nós não precisamos saber, já
    // que o comando sabe como desfazer sua própria ação.
    method undo() is
        command = history.pop()
        if (command != null)
            command.undo()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Aplicabilidade {#applicability}

::::::::: applicability
::: applicability-problem
Utilize o padrão Command quando você quer parametrizar objetos com
operações.
:::

::: applicability-solution
O padrão Command podem tornar uma chamada específica para um método em
um objeto separado. Essa mudança abre várias possibilidades de usos
interessantes: você pode passar comandos como argumentos do método,
armazená-los dentro de outros objetos, trocar comandos ligados no
momento de execução, etc.

Aqui está um exemplo: você está desenvolvendo um componente GUI como um
menu de contexto, e você quer que os usuários sejam capazes de
configurar os items do menu que aciona as operações quando um usuário
clica em um item.
:::

::: applicability-problem
Utilize o padrão Command quando você quer colocar operações em fila,
agendar sua execução, ou executá-las remotamente.
:::

::: applicability-solution
Como qualquer outro objeto, um comando pode ser serializado, o que
significa convertê-lo em uma string que pode ser facilmente escrita em
um arquivo ou base de dados. Mais tarde a string pode ser restaurada no
objeto comando inicial. Dessa forma você pode adiar e agendar execuções
do comando. Mas isso não é tudo! Da mesma forma, você pode colocar em
fila, fazer registro de log ou enviar comandos por uma rede.
:::

::: applicability-problem
Utilize o padrão Command quando você quer implementar operações
reversíveis.
:::

::: applicability-solution
Embora haja muitas formas de implementar o desfazer/refazer, o padrão
Command é talvez a mais popular de todas.

Para ser capaz de reverter operações, você precisa implementar o
histórico de operações realizadas. O histórico do comando é uma pilha
que contém todos os objetos comando executados junto com seus backups do
estado da aplicação relacionados.

Esse método tem duas desvantagens. Primeiro, se não for fácil salvar o
estado da aplicação por parte dela ser privada. Esse problema pode ser
facilmente mitigado com o padrão [Memento](memento.html).

Segundo, os backups de estado podem consumir uma considerável quantidade
de RAM. Portanto, algumas vezes você pode recorrer a uma implementação
alternativa: ao invés de restaurar a um estado passado, o comando faz a
operação inversa. A operação reversa também cobra um preço: ela pode ter
sua implementação difícil ou até impossível.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Como implementar {#checklist}

1.  Declare a interface comando com um único método de execução.

2.  Comece extraindo pedidos para dentro de classes concretas comando
    que implementam a interface comando. Cada classe deve ter um
    conjunto de campos para armazenar os argumentos dos pedidos junto
    com uma referência ao objeto destinatário real. Todos esses valores
    devem ser inicializados através do construtor do comando.

3.  Identifique classes que vão agir como *remetentes*. Adicione os
    campos para armazenar comandos nessas classes. Remetentes devem
    sempre comunicar-se com seus comandos através da interface comando.
    Remetentes geralmente não criam objetos comando por conta própria,
    mas devem obtê-los do código cliente.

4.  Mude os remetentes para que executem o comando ao invés de enviar o
    pedido para o destinatário diretamente.

5.  O cliente deve inicializar objetos na seguinte ordem:

    -   Crie os destinatários.
    -   Crie os comandos, e os associe com os destinatários se
        necessário.
    -   Crie os remetentes, e os associe com os comandos específicos.
:::

:::::: {.section .pros-cons}
##  Prós e contras {#pros-cons}

::::: row
::: col-sm-6
-    *Princípio de responsabilidade única*. Você pode desacoplar classes
    que invocam operações de classes que fazem essas operações.
-    *Princípio aberto/fechado*. Você pode introduzir novos comandos na
    aplicação sem quebrar o código cliente existente.
-    Você pode implementar desfazer/refazer.
-    Você pode implementar a execução adiada de operações.
-    Você pode montar um conjunto de comandos simples em um complexo.
:::

::: col-sm-6
-    O código pode ficar mais complicado uma vez que você está
    introduzindo uma nova camada entre remetentes e destinatários.
:::
:::::
::::::

::: {.section .relations}
##  Relações com outros padrões {#relations}

-   O [Chain of Responsibility](chain-of-responsibility.html),
    [Command](command.html), [Mediator](mediator.html) e
    [Observer](observer.html) abrangem várias maneiras de se conectar
    remetentes e destinatários de pedidos:

    -   O *Chain of Responsibility* passa um pedido sequencialmente ao
        longo de um corrente dinâmica de potenciais destinatários até
        que um deles atua no pedido.
    -   O *Command* estabelece conexões unidirecionais entre remetentes
        e destinatários.
    -   O *Mediator* elimina as conexões diretas entre remetentes e
        destinatários, forçando-os a se comunicar indiretamente através
        de um objeto mediador.
    -   O *Observer* permite que destinatários inscrevam-se ou cancelem
        sua inscrição dinamicamente para receber pedidos.

-   Handlers em uma [Chain of
    Responsibility](chain-of-responsibility.html) podem ser
    implementados como [comandos](command.html). Neste caso, você pode
    executar várias operações diferentes sobre o mesmo objeto contexto,
    representado por um pedido.

    Contudo, há outra abordagem, onde o próprio pedido é um objeto
    *comando*. Neste caso, você pode executar a mesma operação em uma
    série de diferentes contextos ligados em uma corrente.

-   Você pode usar o [Command](command.html) e o [Memento](memento.html)
    juntos quando implementando um "desfazer". Neste caso, os comandos
    são responsáveis pela realização de várias operações sobre um objeto
    alvo, enquanto que os mementos salvam o estado daquele objeto
    momentos antes de um comando ser executado.

-   O [Command](command.html) e o [Strategy](strategy.html) podem ser
    parecidos porque você pode usar ambos para parametrizar um objeto
    com alguma ação. Contudo, eles têm propósitos bem diferentes.

    -   Você pode usar o *Command* para converter qualquer operação em
        um objeto. Os parâmetros da operação se transformam em campos
        daquele objeto. A conversão permite que você atrase a execução
        de uma operação, transforme-a em uma fila, armazene o histórico
        de comandos, envie comandos para serviços remotos, etc.

    -   Por outro lado, o *Strategy* geralmente descreve diferentes
        maneiras de fazer a mesma coisa, permitindo que você troque
        esses algoritmos dentro de uma única classe contexto.

-   O [Prototype](prototype.html) pode ajudar quando você precisa salvar
    cópias de [comandos](command.html) no histórico.

-   Você pode tratar um [Visitor](visitor.html) como uma poderosa versão
    do padrão [Command](command.html). Seus objetos podem executar
    operações sobre vários objetos de diferentes classes.
:::

::: {.section .implementations}
##  Exemplos de código {#implementations}

[![Command em
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](command/csharp/example.html "Command em C#"){.prog-lang-link}
[![Command em
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](command/cpp/example.html "Command em C++"){.prog-lang-link}
[![Command em
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](command/go/example.html "Command em Go"){.prog-lang-link}
[![Command em
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](command/java/example.html "Command em Java"){.prog-lang-link}
[![Command em
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](command/php/example.html "Command em PHP"){.prog-lang-link}
[![Command em
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](command/python/example.html "Command em Python"){.prog-lang-link}
[![Command em
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](command/ruby/example.html "Command em Ruby"){.prog-lang-link}
[![Command em
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](command/rust/example.html "Command em Rust"){.prog-lang-link}
[![Command em
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](command/swift/example.html "Command em Swift"){.prog-lang-link}
[![Command em
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](command/typescript/example.html "Command em TypeScript"){.prog-lang-link}
:::

:::::: {#book-promo .banner-set2}
::::: {.prom .banner-content .banner-bg .banner-striped .banner-with-image data-id="DP: 1: Support our free website and own the eBook!" creative-id="standard-pt-br" position="content_bottom"}
::: banner-image
[![](../../images/patterns/banners/patterns-book-banner-3a29e.png?id=7d445df13c80287beaab234b4f3b698c){width="200"
height="200" loading="lazy"
srcset="/images/patterns/banners/patterns-book-banner-3-2x.png?id=0cc3f77ab421d1a5c02ee46488231c3a 2x"}](book.html)
:::

::: banner-text
### Apoie nosso website gratuito e ganhe o eBook! {#apoie-nosso-website-gratuito-e-ganhe-o-ebook .title}

-   22 padrões de projeto e 8 princípios explicados a fundo.
-   439 páginas bem estruturadas, fáceis de se ler e livres de jargões.
-   225 diagramas e ilustrações claras e úteis.
-   Um arquivo com exemplos de código em 11 línguas.
-   Suportado por todos os dispositivos: formatos PDF/EPUB/MOBI/KFX.

[ Saiba mais...](book.html){.btn .btn-secondary}
:::
:::::
::::::

::: next
#### Leia a seguir

[Iterator []{.fa .fa-arrow-right}](iterator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Chain of
Responsibility](chain-of-responsibility.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner-removable .banner-removable-patterns data-id="DP: Sidebar" creative-id="standard-sidebar-pt-br" position="sidebar"}
:::::: banner-inner
:::: image3d-book-right
::: {.image3d-cover style="background: #0b3752;"}
[![](../../images/patterns/book/web-cover-pt-brb3eb.png?id=cc7d821610ba9186e987ffc0f83476a8){width="250"
height="375" loading="lazy"
srcset="/images/patterns/book/web-cover-pt-br-2x.png?id=95380743a2833ed832101aa568c16373 2x"}](book.html)
:::
::::

::: {style="margin-top: 1rem"}
Este artigo é parte de nosso eBook\
**Mergulho nos Padrões de Projeto**.

[ Saiba mais...](book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::
