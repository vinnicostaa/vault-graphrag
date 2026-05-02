:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Посередник](../../mediator.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Посередник](../../../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x"}
:::

# **Посередник** на Java {#посередник-на-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Посередник** --- це поведінковий патерн, який спрощує комунікацію між
компонентами системи.

Посередник прибирає прямі зв'язки між окремими компонентами, змушуючи їх
спілкуватися один з одним через себе.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Посередник ](../../mediator.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навігація
:::

::: en-intro
 [Інтро](#)
:::

::: en-intro
 [Редактор нотаток](#example-0)
:::

::: en-file
 components
:::

:::: en-inside
::: en-file
  [Component](#example-0--components-Component-java)
:::
::::

:::: en-inside
::: en-file
  [Add­Button](#example-0--components-AddButton-java)
:::
::::

:::: en-inside
::: en-file
  [Delete­Button](#example-0--components-DeleteButton-java)
:::
::::

:::: en-inside
::: en-file
  [Filter](#example-0--components-Filter-java)
:::
::::

:::: en-inside
::: en-file
  [List](#example-0--components-List-java)
:::
::::

:::: en-inside
::: en-file
  [Save­Button](#example-0--components-SaveButton-java)
:::
::::

:::: en-inside
::: en-file
  [Text­Box](#example-0--components-TextBox-java)
:::
::::

:::: en-inside
::: en-file
  [Title](#example-0--components-Title-java)
:::
::::

::: en-file
 mediator
:::

:::: en-inside
::: en-file
  [Mediator](#example-0--mediator-Mediator-java)
:::
::::

:::: en-inside
::: en-file
  [Editor](#example-0--mediator-Editor-java)
:::
::::

:::: en-inside
::: en-file
  [Note](#example-0--mediator-Note-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-png)
:::
::::::::::::::::::::::::::::::::

**Складність:** []{.fa-stars}

**Популярність:** []{.fa-stars}

**Застосування:** Мабуть, найпопулярнішим застосуванням Посередника в
Java-коді є зв'язок кількох компонентів GUI однієї програми.

Приклади Посередника в стандартних бібліотеках Java:

-   [`java.util.Timer`](http://docs.oracle.com/javase/8/docs/api/java/util/Timer.html)
    (усі методи `scheduleXXX()`)
-   [`java.util.concurrent.Executor#execute()`](http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executor.html#execute-java.lang.Runnable-)
-   [`java.util.concurrent.ExecutorService`](http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html)
    (методи `invokeXXX()` та `submit()`)
-   [`java.util.concurrent.ScheduledExecutorService`](http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ScheduledExecutorService.html)
    (усі методи `scheduleXXX()`)
-   [`java.lang.reflect.Method#invoke()`](http://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Method.html#invoke-java.lang.Object-java.lang.Object...-)
:::::::::::::::::::::::::::::::::::

[]{#example-0}

## Редактор нотаток {#example-0-title}

Цей приклад показує, як організувати множину елементів інтерфейсу за
допомогою посередника так, щоб вони не знали і не залежали один від
одного.

### []{#example-0--components .anchor} **components:** Класи-колеги {#components-класи-колеги .example-title}

#### []{#example-0--components-Component-java .anchor} **components/Component.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.mediator.example.components;

import refactoring_guru.mediator.example.mediator.Mediator;

/**
 * Common component interface.
 */
public interface Component {
    void setMediator(Mediator mediator);
    String getName();
}</code></pre>
</figure>

#### []{#example-0--components-AddButton-java .anchor} **components/AddButton.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.mediator.example.components;

import refactoring_guru.mediator.example.mediator.Mediator;
import refactoring_guru.mediator.example.mediator.Note;

import javax.swing.*;
import java.awt.event.ActionEvent;

/**
 * Concrete components don&#39;t talk with each other. They have only one
 * communication channel–sending requests to the mediator.
 */
public class AddButton extends JButton implements Component {
    private Mediator mediator;

    public AddButton() {
        super(&quot;Add&quot;);
    }

    @Override
    public void setMediator(Mediator mediator) {
        this.mediator = mediator;
    }

    @Override
    protected void fireActionPerformed(ActionEvent actionEvent) {
        mediator.addNewNote(new Note());
    }

    @Override
    public String getName() {
        return &quot;AddButton&quot;;
    }
}</code></pre>
</figure>

#### []{#example-0--components-DeleteButton-java .anchor} **components/DeleteButton.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.mediator.example.components;

import refactoring_guru.mediator.example.mediator.Mediator;

import javax.swing.*;
import java.awt.event.ActionEvent;

/**
 * Concrete components don&#39;t talk with each other. They have only one
 * communication channel–sending requests to the mediator.
 */
public class DeleteButton extends JButton  implements Component {
    private Mediator mediator;

    public DeleteButton() {
        super(&quot;Del&quot;);
    }

    @Override
    public void setMediator(Mediator mediator) {
        this.mediator = mediator;
    }

    @Override
    protected void fireActionPerformed(ActionEvent actionEvent) {
        mediator.deleteNote();
    }

    @Override
    public String getName() {
        return &quot;DelButton&quot;;
    }
}</code></pre>
</figure>

#### []{#example-0--components-Filter-java .anchor} **components/Filter.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.mediator.example.components;

import refactoring_guru.mediator.example.mediator.Mediator;
import refactoring_guru.mediator.example.mediator.Note;

import javax.swing.*;
import java.awt.event.KeyEvent;
import java.util.ArrayList;

/**
 * Concrete components don&#39;t talk with each other. They have only one
 * communication channel–sending requests to the mediator.
 */
public class Filter extends JTextField implements Component {
    private Mediator mediator;
    private ListModel listModel;

    public Filter() {}

    @Override
    public void setMediator(Mediator mediator) {
        this.mediator = mediator;
    }

    @Override
    protected void processComponentKeyEvent(KeyEvent keyEvent) {
        String start = getText();
        searchElements(start);
    }

    public void setList(ListModel listModel) {
        this.listModel = listModel;
    }

    private void searchElements(String s) {
        if (listModel == null) {
            return;
        }

        if (s.equals(&quot;&quot;)) {
            mediator.setElementsList(listModel);
            return;
        }

        ArrayList&lt;Note&gt; notes = new ArrayList&lt;&gt;();
        for (int i = 0; i &lt; listModel.getSize(); i++) {
            notes.add((Note) listModel.getElementAt(i));
        }
        DefaultListModel&lt;Note&gt; listModel = new DefaultListModel&lt;&gt;();
        for (Note note : notes) {
            if (note.getName().contains(s)) {
                listModel.addElement(note);
            }
        }
        mediator.setElementsList(listModel);
    }

    @Override
    public String getName() {
        return &quot;Filter&quot;;
    }
}</code></pre>
</figure>

#### []{#example-0--components-List-java .anchor} **components/List.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.mediator.example.components;

import refactoring_guru.mediator.example.mediator.Mediator;
import refactoring_guru.mediator.example.mediator.Note;

import javax.swing.*;

/**
 * Concrete components don&#39;t talk with each other. They have only one
 * communication channel–sending requests to the mediator.
 */
@SuppressWarnings(&quot;unchecked&quot;)
public class List extends JList implements Component {
    private Mediator mediator;
    private final DefaultListModel LIST_MODEL;

    public List(DefaultListModel listModel) {
        super(listModel);
        this.LIST_MODEL = listModel;
        setModel(listModel);
        this.setLayoutOrientation(JList.VERTICAL);
        Thread thread = new Thread(new Hide(this));
        thread.start();
    }

    @Override
    public void setMediator(Mediator mediator) {
        this.mediator = mediator;
    }

    public void addElement(Note note) {
        LIST_MODEL.addElement(note);
        int index = LIST_MODEL.size() - 1;
        setSelectedIndex(index);
        ensureIndexIsVisible(index);
        mediator.sendToFilter(LIST_MODEL);
    }

    public void deleteElement() {
        int index = this.getSelectedIndex();
        try {
            LIST_MODEL.remove(index);
            mediator.sendToFilter(LIST_MODEL);
        } catch (ArrayIndexOutOfBoundsException ignored) {}
    }

    public Note getCurrentElement() {
        return (Note)getSelectedValue();
    }

    @Override
    public String getName() {
        return &quot;List&quot;;
    }

    private class Hide implements Runnable {
        private List list;

        Hide(List list) {
            this.list = list;
        }

        @Override
        public void run() {
            while (true) {
                try {
                    Thread.sleep(300);
                } catch (InterruptedException ex) {
                    ex.printStackTrace();
                }
                if (list.isSelectionEmpty()) {
                    mediator.hideElements(true);
                } else {
                    mediator.hideElements(false);
                }
            }
        }
    }
}</code></pre>
</figure>

#### []{#example-0--components-SaveButton-java .anchor} **components/SaveButton.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.mediator.example.components;

import refactoring_guru.mediator.example.mediator.Mediator;

import javax.swing.*;
import java.awt.event.ActionEvent;

/**
 * Concrete components don&#39;t talk with each other. They have only one
 * communication channel–sending requests to the mediator.
 */
public class SaveButton extends JButton implements Component {
    private Mediator mediator;

    public SaveButton() {
        super(&quot;Save&quot;);
    }

    @Override
    public void setMediator(Mediator mediator) {
        this.mediator = mediator;
    }

    @Override
    protected void fireActionPerformed(ActionEvent actionEvent) {
        mediator.saveChanges();
    }

    @Override
    public String getName() {
        return &quot;SaveButton&quot;;
    }
}</code></pre>
</figure>

#### []{#example-0--components-TextBox-java .anchor} **components/TextBox.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.mediator.example.components;

import refactoring_guru.mediator.example.mediator.Mediator;

import javax.swing.*;
import java.awt.event.KeyEvent;

/**
 * Concrete components don&#39;t talk with each other. They have only one
 * communication channel–sending requests to the mediator.
 */
public class TextBox extends JTextArea implements Component {
    private Mediator mediator;

    @Override
    public void setMediator(Mediator mediator) {
        this.mediator = mediator;
    }

    @Override
    protected void processComponentKeyEvent(KeyEvent keyEvent) {
        mediator.markNote();
    }

    @Override
    public String getName() {
        return &quot;TextBox&quot;;
    }
}</code></pre>
</figure>

#### []{#example-0--components-Title-java .anchor} **components/Title.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.mediator.example.components;

import refactoring_guru.mediator.example.mediator.Mediator;

import javax.swing.*;
import java.awt.event.KeyEvent;

/**
 * Concrete components don&#39;t talk with each other. They have only one
 * communication channel–sending requests to the mediator.
 */
public class Title extends JTextField implements Component {
    private Mediator mediator;

    @Override
    public void setMediator(Mediator mediator) {
        this.mediator = mediator;
    }

    @Override
    protected void processComponentKeyEvent(KeyEvent keyEvent) {
        mediator.markNote();
    }

    @Override
    public String getName() {
        return &quot;Title&quot;;
    }
}</code></pre>
</figure>

### []{#example-0--mediator .anchor} **mediator** {#mediator .example-title}

#### []{#example-0--mediator-Mediator-java .anchor} **mediator/Mediator.java:** Загальний інтерфейс посередника

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.mediator.example.mediator;

import refactoring_guru.mediator.example.components.Component;

import javax.swing.*;

/**
 * Common mediator interface.
 */
public interface Mediator {
    void addNewNote(Note note);
    void deleteNote();
    void getInfoFromList(Note note);
    void saveChanges();
    void markNote();
    void clear();
    void sendToFilter(ListModel listModel);
    void setElementsList(ListModel list);
    void registerComponent(Component component);
    void hideElements(boolean flag);
    void createGUI();
}</code></pre>
</figure>

#### []{#example-0--mediator-Editor-java .anchor} **mediator/Editor.java:** Конкретний посередник

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.mediator.example.mediator;

import refactoring_guru.mediator.example.components.*;
import refactoring_guru.mediator.example.components.Component;
import refactoring_guru.mediator.example.components.List;

import javax.swing.*;
import javax.swing.border.LineBorder;
import java.awt.*;

/**
 * Concrete mediator. All chaotic communications between concrete components
 * have been extracted to the mediator. Now components only talk with the
 * mediator, which knows who has to handle a request.
 */
public class Editor implements Mediator {
    private Title title;
    private TextBox textBox;
    private AddButton add;
    private DeleteButton del;
    private SaveButton save;
    private List list;
    private Filter filter;

    private JLabel titleLabel = new JLabel(&quot;Title:&quot;);
    private JLabel textLabel = new JLabel(&quot;Text:&quot;);
    private JLabel label = new JLabel(&quot;Add or select existing note to proceed...&quot;);
  
    /**
     * Here the registration of components by the mediator.
     */
    @Override
    public void registerComponent(Component component) {
        component.setMediator(this);
        switch (component.getName()) {
            case &quot;AddButton&quot;:
                add = (AddButton)component;
                break;
            case &quot;DelButton&quot;:
                del = (DeleteButton)component;
                break;
            case &quot;Filter&quot;:
                filter = (Filter)component;
                break;
            case &quot;List&quot;:
                list = (List)component;
                this.list.addListSelectionListener(listSelectionEvent -&gt; {
                    Note note = (Note)list.getSelectedValue();
                    if (note != null) {
                        getInfoFromList(note);
                    } else {
                        clear();
                    }
                });
                break;
            case &quot;SaveButton&quot;:
                save = (SaveButton)component;
                break;
            case &quot;TextBox&quot;:
                textBox = (TextBox)component;
                break;
            case &quot;Title&quot;:
                title = (Title)component;
                break;
        }
    }

    /**
     * Various methods to handle requests from particular components.
     */
    @Override
    public void addNewNote(Note note) {
        title.setText(&quot;&quot;);
        textBox.setText(&quot;&quot;);
        list.addElement(note);
    }

    @Override
    public void deleteNote() {
        list.deleteElement();
    }

    @Override
    public void getInfoFromList(Note note) {
        title.setText(note.getName().replace(&#39;*&#39;, &#39; &#39;));
        textBox.setText(note.getText());
    }

    @Override
    public void saveChanges() {
        try {
            Note note = (Note) list.getSelectedValue();
            note.setName(title.getText());
            note.setText(textBox.getText());
            list.repaint();
        } catch (NullPointerException ignored) {}
    }

    @Override
    public void markNote() {
        try {
            Note note = list.getCurrentElement();
            String name = note.getName();
            if (!name.endsWith(&quot;*&quot;)) {
                note.setName(note.getName() + &quot;*&quot;);
            }
            list.repaint();
        } catch (NullPointerException ignored) {}
    }

    @Override
    public void clear() {
        title.setText(&quot;&quot;);
        textBox.setText(&quot;&quot;);
    }

    @Override
    public void sendToFilter(ListModel listModel) {
        filter.setList(listModel);
    }

    @SuppressWarnings(&quot;unchecked&quot;)
    @Override
    public void setElementsList(ListModel list) {
        this.list.setModel(list);
        this.list.repaint();
    }

    @Override
    public void hideElements(boolean flag) {
        titleLabel.setVisible(!flag);
        textLabel.setVisible(!flag);
        title.setVisible(!flag);
        textBox.setVisible(!flag);
        save.setVisible(!flag);
        label.setVisible(flag);
    }

    @Override
    public void createGUI() {
        JFrame notes = new JFrame(&quot;Notes&quot;);
        notes.setSize(960, 600);
        notes.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
        JPanel left = new JPanel();
        left.setBorder(new LineBorder(Color.BLACK));
        left.setSize(320, 600);
        left.setLayout(new BoxLayout(left, BoxLayout.Y_AXIS));
        JPanel filterPanel = new JPanel();
        filterPanel.add(new JLabel(&quot;Filter:&quot;));
        filter.setColumns(20);
        filterPanel.add(filter);
        filterPanel.setPreferredSize(new Dimension(280, 40));
        JPanel listPanel = new JPanel();
        list.setFixedCellWidth(260);
        listPanel.setSize(320, 470);
        JScrollPane scrollPane = new JScrollPane(list);
        scrollPane.setPreferredSize(new Dimension(275, 410));
        listPanel.add(scrollPane);
        JPanel buttonPanel = new JPanel();
        add.setPreferredSize(new Dimension(85, 25));
        buttonPanel.add(add);
        del.setPreferredSize(new Dimension(85, 25));
        buttonPanel.add(del);
        buttonPanel.setLayout(new FlowLayout());
        left.add(filterPanel);
        left.add(listPanel);
        left.add(buttonPanel);
        JPanel right = new JPanel();
        right.setLayout(null);
        right.setSize(640, 600);
        right.setLocation(320, 0);
        right.setBorder(new LineBorder(Color.BLACK));
        titleLabel.setBounds(20, 4, 50, 20);
        title.setBounds(60, 5, 555, 20);
        textLabel.setBounds(20, 4, 50, 130);
        textBox.setBorder(new LineBorder(Color.DARK_GRAY));
        textBox.setBounds(20, 80, 595, 410);
        save.setBounds(270, 535, 80, 25);
        label.setFont(new Font(&quot;Verdana&quot;, Font.PLAIN, 22));
        label.setBounds(100, 240, 500, 100);
        right.add(label);
        right.add(titleLabel);
        right.add(title);
        right.add(textLabel);
        right.add(textBox);
        right.add(save);
        notes.setLayout(null);
        notes.getContentPane().add(left);
        notes.getContentPane().add(right);
        notes.setResizable(false);
        notes.setLocationRelativeTo(null);
        notes.setVisible(true);
    }
}</code></pre>
</figure>

#### []{#example-0--mediator-Note-java .anchor} **mediator/Note.java:** Клас нотаток

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.mediator.example.mediator;

/**
 * Note class.
 */
public class Note {
    private String name;
    private String text;

    public Note() {
        name = &quot;New note&quot;;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setText(String text) {
        this.text = text;
    }

    public String getName() {
        return name;
    }

    public String getText() {
        return text;
    }

    @Override
    public String toString() {
        return name;
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** Код ініціалізації

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.mediator.example;

import refactoring_guru.mediator.example.components.*;
import refactoring_guru.mediator.example.mediator.Editor;
import refactoring_guru.mediator.example.mediator.Mediator;

import javax.swing.*;

/**
 * Demo class. Everything comes together here.
 */
public class Demo {
    public static void main(String[] args) {
        Mediator mediator = new Editor();

        mediator.registerComponent(new Title());
        mediator.registerComponent(new TextBox());
        mediator.registerComponent(new AddButton());
        mediator.registerComponent(new DeleteButton());
        mediator.registerComponent(new SaveButton());
        mediator.registerComponent(new List(new DefaultListModel()));
        mediator.registerComponent(new Filter());

        mediator.createGUI();
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-png .anchor} **OutputDemo.png:** Результат виконання

![](../../../../images/patterns/examples/java/mediator/OutputDemo67e0.png?id=c7ac651c6472d68d5c9459551f29ec9c)

::: next
#### Читаємо далі

[Знімок на Java []{.fa
.fa-arrow-right}](../../memento/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Ітератор на
Java](../../iterator/java/example.html){.btn .btn-default rel="prev"}
:::

## **Посередник** іншими мовами програмування

[![Посередник на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Посередник на C#"){.prog-lang-link}
[![Посередник на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Посередник на C++"){.prog-lang-link}
[![Посередник на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Посередник на Go"){.prog-lang-link}
[![Посередник на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Посередник на PHP"){.prog-lang-link}
[![Посередник на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Посередник на Python"){.prog-lang-link}
[![Посередник на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Посередник на Ruby"){.prog-lang-link}
[![Посередник на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Посередник на Rust"){.prog-lang-link}
[![Посередник на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Посередник на Swift"){.prog-lang-link}
[![Посередник на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Посередник на TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-uk" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архів з прикладами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Придбай книгу **Занурення в Патерни** та отримай архів з десятками
детальних прикладів, які можна відкривати прямо в IDE.

[ Дізнатися більше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::
