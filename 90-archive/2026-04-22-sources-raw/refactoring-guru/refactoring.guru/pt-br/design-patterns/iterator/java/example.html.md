:::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Iterator](../../iterator.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Iterator](../../../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x"}
:::

# **Iterator** em Java {#iterator-em-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
O **Iterador** é um padrão de projeto comportamental que permite a
passagem sequencial através de uma estrutura de dados complexa sem expor
seus detalhes internos.

Graças ao Iterator, os clientes podem examinar elementos de diferentes
coleções de maneira semelhante usando uma única interface iterador.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Iterator ](../../iterator.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegação
:::

::: en-intro
 [Introdução](#)
:::

::: en-intro
 [Iterando nos perfis de redes sociais](#example-0)
:::

::: en-file
 iterators
:::

:::: en-inside
::: en-file
  [Profile­Iterator](#example-0--iterators-ProfileIterator-java)
:::
::::

:::: en-inside
::: en-file
  [Facebook­Iterator](#example-0--iterators-FacebookIterator-java)
:::
::::

:::: en-inside
::: en-file
  [Linked­In­Iterator](#example-0--iterators-LinkedInIterator-java)
:::
::::

::: en-file
 social_networks
:::

:::: en-inside
::: en-file
  [Social­Network](#example-0--social_networks-SocialNetwork-java)
:::
::::

:::: en-inside
::: en-file
  [Facebook](#example-0--social_networks-Facebook-java)
:::
::::

:::: en-inside
::: en-file
  [Linked­In](#example-0--social_networks-LinkedIn-java)
:::
::::

::: en-file
 profile
:::

:::: en-inside
::: en-file
  [Profile](#example-0--profile-Profile-java)
:::
::::

::: en-file
 spammer
:::

:::: en-inside
::: en-file
  [Social­Spammer](#example-0--spammer-SocialSpammer-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-txt)
:::
::::::::::::::::::::::::::::

**Complexidade:** []{.fa-stars}

**Popularidade:** []{.fa-stars}

**Exemplos de uso:** O padrão é muito comum no código Java. Muitos
frameworks e bibliotecas o usam para fornecer uma maneira padrão de
percorrer suas coleções.

Aqui estão alguns exemplos das principais bibliotecas Java:

-   Todas as implementações de
    [`java.util.Iterator`](http://docs.oracle.com/javase/8/docs/api/java/util/Iterator.html)
    (também
    [`java.util.Scanner`](http://docs.oracle.com/javase/8/docs/api/java/util/Scanner.html)).

-   Todas as implementações de
    [`java.util.Enumeration`](http://docs.oracle.com/javase/8/docs/api/java/util/Enumeration.html).

**Identificação:** O iterador é fácil de reconhecer pelos métodos de
navegação (como `next`, `previous` e outros). O código cliente que usa
iteradores pode não ter acesso direto à coleção que está sendo
percorrida.
:::::::::::::::::::::::::::::::

[]{#example-0}

## Iterando nos perfis de redes sociais {#example-0-title}

Neste exemplo, o padrão Iterator é usado para revisar perfis sociais de
uma coleção de rede social remota sem expor nenhum detalhe da
comunicação ao código cliente.

### []{#example-0--iterators .anchor} **iterators** {#iterators .example-title}

#### []{#example-0--iterators-ProfileIterator-java .anchor} **iterators/ProfileIterator.java:** Define interface de perfil

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.iterator.example.iterators;

import refactoring_guru.iterator.example.profile.Profile;

public interface ProfileIterator {
    boolean hasNext();

    Profile getNext();

    void reset();
}</code></pre>
</figure>

#### []{#example-0--iterators-FacebookIterator-java .anchor} **iterators/FacebookIterator.java:** Implementa iteração sobre perfis do Facebook

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.iterator.example.iterators;

import refactoring_guru.iterator.example.profile.Profile;
import refactoring_guru.iterator.example.social_networks.Facebook;

import java.util.ArrayList;
import java.util.List;

public class FacebookIterator implements ProfileIterator {
    private Facebook facebook;
    private String type;
    private String email;
    private int currentPosition = 0;
    private List&lt;String&gt; emails = new ArrayList&lt;&gt;();
    private List&lt;Profile&gt; profiles = new ArrayList&lt;&gt;();

    public FacebookIterator(Facebook facebook, String type, String email) {
        this.facebook = facebook;
        this.type = type;
        this.email = email;
    }

    private void lazyLoad() {
        if (emails.size() == 0) {
            List&lt;String&gt; profiles = facebook.requestProfileFriendsFromFacebook(this.email, this.type);
            for (String profile : profiles) {
                this.emails.add(profile);
                this.profiles.add(null);
            }
        }
    }

    @Override
    public boolean hasNext() {
        lazyLoad();
        return currentPosition &lt; emails.size();
    }

    @Override
    public Profile getNext() {
        if (!hasNext()) {
            return null;
        }

        String friendEmail = emails.get(currentPosition);
        Profile friendProfile = profiles.get(currentPosition);
        if (friendProfile == null) {
            friendProfile = facebook.requestProfileFromFacebook(friendEmail);
            profiles.set(currentPosition, friendProfile);
        }
        currentPosition++;
        return friendProfile;
    }

    @Override
    public void reset() {
        currentPosition = 0;
    }
}</code></pre>
</figure>

#### []{#example-0--iterators-LinkedInIterator-java .anchor} **iterators/LinkedInIterator.java:** Implementa iteração sobre perfis do LinkedIn

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.iterator.example.iterators;

import refactoring_guru.iterator.example.profile.Profile;
import refactoring_guru.iterator.example.social_networks.LinkedIn;

import java.util.ArrayList;
import java.util.List;

public class LinkedInIterator implements ProfileIterator {
    private LinkedIn linkedIn;
    private String type;
    private String email;
    private int currentPosition = 0;
    private List&lt;String&gt; emails = new ArrayList&lt;&gt;();
    private List&lt;Profile&gt; contacts = new ArrayList&lt;&gt;();

    public LinkedInIterator(LinkedIn linkedIn, String type, String email) {
        this.linkedIn = linkedIn;
        this.type = type;
        this.email = email;
    }

    private void lazyLoad() {
        if (emails.size() == 0) {
            List&lt;String&gt; profiles = linkedIn.requestRelatedContactsFromLinkedInAPI(this.email, this.type);
            for (String profile : profiles) {
                this.emails.add(profile);
                this.contacts.add(null);
            }
        }
    }

    @Override
    public boolean hasNext() {
        lazyLoad();
        return currentPosition &lt; emails.size();
    }

    @Override
    public Profile getNext() {
        if (!hasNext()) {
            return null;
        }

        String friendEmail = emails.get(currentPosition);
        Profile friendContact = contacts.get(currentPosition);
        if (friendContact == null) {
            friendContact = linkedIn.requestContactInfoFromLinkedInAPI(friendEmail);
            contacts.set(currentPosition, friendContact);
        }
        currentPosition++;
        return friendContact;
    }

    @Override
    public void reset() {
        currentPosition = 0;
    }
}</code></pre>
</figure>

### []{#example-0--social_networks .anchor} **social_networks** {#social_networks .example-title}

#### []{#example-0--social_networks-SocialNetwork-java .anchor} **social_networks/SocialNetwork.java:** Define interface comum de rede social

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.iterator.example.social_networks;

import refactoring_guru.iterator.example.iterators.ProfileIterator;

public interface SocialNetwork {
    ProfileIterator createFriendsIterator(String profileEmail);

    ProfileIterator createCoworkersIterator(String profileEmail);
}</code></pre>
</figure>

#### []{#example-0--social_networks-Facebook-java .anchor} **social_networks/Facebook.java:** Facebook

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.iterator.example.social_networks;

import refactoring_guru.iterator.example.iterators.FacebookIterator;
import refactoring_guru.iterator.example.iterators.ProfileIterator;
import refactoring_guru.iterator.example.profile.Profile;

import java.util.ArrayList;
import java.util.List;

public class Facebook implements SocialNetwork {
    private List&lt;Profile&gt; profiles;

    public Facebook(List&lt;Profile&gt; cache) {
        if (cache != null) {
            this.profiles = cache;
        } else {
            this.profiles = new ArrayList&lt;&gt;();
        }
    }

    public Profile requestProfileFromFacebook(String profileEmail) {
        // Here would be a POST request to one of the Facebook API endpoints.
        // Instead, we emulates long network connection, which you would expect
        // in the real life...
        simulateNetworkLatency();
        System.out.println(&quot;Facebook: Loading profile &#39;&quot; + profileEmail + &quot;&#39; over the network...&quot;);

        // ...and return test data.
        return findProfile(profileEmail);
    }

    public List&lt;String&gt; requestProfileFriendsFromFacebook(String profileEmail, String contactType) {
        // Here would be a POST request to one of the Facebook API endpoints.
        // Instead, we emulates long network connection, which you would expect
        // in the real life...
        simulateNetworkLatency();
        System.out.println(&quot;Facebook: Loading &#39;&quot; + contactType + &quot;&#39; list of &#39;&quot; + profileEmail + &quot;&#39; over the network...&quot;);

        // ...and return test data.
        Profile profile = findProfile(profileEmail);
        if (profile != null) {
            return profile.getContacts(contactType);
        }
        return null;
    }

    private Profile findProfile(String profileEmail) {
        for (Profile profile : profiles) {
            if (profile.getEmail().equals(profileEmail)) {
                return profile;
            }
        }
        return null;
    }

    private void simulateNetworkLatency() {
        try {
            Thread.sleep(2500);
        } catch (InterruptedException ex) {
            ex.printStackTrace();
        }
    }

    @Override
    public ProfileIterator createFriendsIterator(String profileEmail) {
        return new FacebookIterator(this, &quot;friends&quot;, profileEmail);
    }

    @Override
    public ProfileIterator createCoworkersIterator(String profileEmail) {
        return new FacebookIterator(this, &quot;coworkers&quot;, profileEmail);
    }

}</code></pre>
</figure>

#### []{#example-0--social_networks-LinkedIn-java .anchor} **social_networks/LinkedIn.java:** LinkedIn

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.iterator.example.social_networks;

import refactoring_guru.iterator.example.iterators.LinkedInIterator;
import refactoring_guru.iterator.example.iterators.ProfileIterator;
import refactoring_guru.iterator.example.profile.Profile;

import java.util.ArrayList;
import java.util.List;

public class LinkedIn implements SocialNetwork {
    private List&lt;Profile&gt; contacts;

    public LinkedIn(List&lt;Profile&gt; cache) {
        if (cache != null) {
            this.contacts = cache;
        } else {
            this.contacts = new ArrayList&lt;&gt;();
        }
    }

    public Profile requestContactInfoFromLinkedInAPI(String profileEmail) {
        // Here would be a POST request to one of the LinkedIn API endpoints.
        // Instead, we emulates long network connection, which you would expect
        // in the real life...
        simulateNetworkLatency();
        System.out.println(&quot;LinkedIn: Loading profile &#39;&quot; + profileEmail + &quot;&#39; over the network...&quot;);

        // ...and return test data.
        return findContact(profileEmail);
    }

    public List&lt;String&gt; requestRelatedContactsFromLinkedInAPI(String profileEmail, String contactType) {
        // Here would be a POST request to one of the LinkedIn API endpoints.
        // Instead, we emulates long network connection, which you would expect
        // in the real life.
        simulateNetworkLatency();
        System.out.println(&quot;LinkedIn: Loading &#39;&quot; + contactType + &quot;&#39; list of &#39;&quot; + profileEmail + &quot;&#39; over the network...&quot;);

        // ...and return test data.
        Profile profile = findContact(profileEmail);
        if (profile != null) {
            return profile.getContacts(contactType);
        }
        return null;
    }

    private Profile findContact(String profileEmail) {
        for (Profile profile : contacts) {
            if (profile.getEmail().equals(profileEmail)) {
                return profile;
            }
        }
        return null;
    }

    private void simulateNetworkLatency() {
        try {
            Thread.sleep(2500);
        } catch (InterruptedException ex) {
            ex.printStackTrace();
        }
    }

    @Override
    public ProfileIterator createFriendsIterator(String profileEmail) {
        return new LinkedInIterator(this, &quot;friends&quot;, profileEmail);
    }

    @Override
    public ProfileIterator createCoworkersIterator(String profileEmail) {
        return new LinkedInIterator(this, &quot;coworkers&quot;, profileEmail);
    }
}</code></pre>
</figure>

### []{#example-0--profile .anchor} **profile** {#profile .example-title}

#### []{#example-0--profile-Profile-java .anchor} **profile/Profile.java:** Perfis sociais

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.iterator.example.profile;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class Profile {
    private String name;
    private String email;
    private Map&lt;String, List&lt;String&gt;&gt; contacts = new HashMap&lt;&gt;();

    public Profile(String email, String name, String... contacts) {
        this.email = email;
        this.name = name;

        // Parse contact list from a set of &quot;friend:email@gmail.com&quot; pairs.
        for (String contact : contacts) {
            String[] parts = contact.split(&quot;:&quot;);
            String contactType = &quot;friend&quot;, contactEmail;
            if (parts.length == 1) {
                contactEmail = parts[0];
            }
            else {
                contactType = parts[0];
                contactEmail = parts[1];
            }
            if (!this.contacts.containsKey(contactType)) {
                this.contacts.put(contactType, new ArrayList&lt;&gt;());
            }
            this.contacts.get(contactType).add(contactEmail);
        }
    }

    public String getEmail() {
        return email;
    }

    public String getName() {
        return name;
    }

    public List&lt;String&gt; getContacts(String contactType) {
        if (!this.contacts.containsKey(contactType)) {
            this.contacts.put(contactType, new ArrayList&lt;&gt;());
        }
        return contacts.get(contactType);
    }
}</code></pre>
</figure>

### []{#example-0--spammer .anchor} **spammer** {#spammer .example-title}

#### []{#example-0--spammer-SocialSpammer-java .anchor} **spammer/SocialSpammer.java:** Aplicação de envio de mensagens

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.iterator.example.spammer;

import refactoring_guru.iterator.example.iterators.ProfileIterator;
import refactoring_guru.iterator.example.profile.Profile;
import refactoring_guru.iterator.example.social_networks.SocialNetwork;

public class SocialSpammer {
    public SocialNetwork network;
    public ProfileIterator iterator;

    public SocialSpammer(SocialNetwork network) {
        this.network = network;
    }

    public void sendSpamToFriends(String profileEmail, String message) {
        System.out.println(&quot;\nIterating over friends...\n&quot;);
        iterator = network.createFriendsIterator(profileEmail);
        while (iterator.hasNext()) {
            Profile profile = iterator.getNext();
            sendMessage(profile.getEmail(), message);
        }
    }

    public void sendSpamToCoworkers(String profileEmail, String message) {
        System.out.println(&quot;\nIterating over coworkers...\n&quot;);
        iterator = network.createCoworkersIterator(profileEmail);
        while (iterator.hasNext()) {
            Profile profile = iterator.getNext();
            sendMessage(profile.getEmail(), message);
        }
    }

    public void sendMessage(String email, String message) {
        System.out.println(&quot;Sent message to: &#39;&quot; + email + &quot;&#39;. Message body: &#39;&quot; + message + &quot;&#39;&quot;);
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** Código cliente

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.iterator.example;

import refactoring_guru.iterator.example.profile.Profile;
import refactoring_guru.iterator.example.social_networks.Facebook;
import refactoring_guru.iterator.example.social_networks.LinkedIn;
import refactoring_guru.iterator.example.social_networks.SocialNetwork;
import refactoring_guru.iterator.example.spammer.SocialSpammer;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

/**
 * Demo class. Everything comes together here.
 */
public class Demo {
    public static Scanner scanner = new Scanner(System.in);

    public static void main(String[] args) {
        System.out.println(&quot;Please specify social network to target spam tool (default:Facebook):&quot;);
        System.out.println(&quot;1. Facebook&quot;);
        System.out.println(&quot;2. LinkedIn&quot;);
        String choice = scanner.nextLine();

        SocialNetwork network;
        if (choice.equals(&quot;2&quot;)) {
            network = new LinkedIn(createTestProfiles());
        }
        else {
            network = new Facebook(createTestProfiles());
        }

        SocialSpammer spammer = new SocialSpammer(network);
        spammer.sendSpamToFriends(&quot;anna.smith@bing.com&quot;,
                &quot;Hey! This is Anna&#39;s friend Josh. Can you do me a favor and like this post [link]?&quot;);
        spammer.sendSpamToCoworkers(&quot;anna.smith@bing.com&quot;,
                &quot;Hey! This is Anna&#39;s boss Jason. Anna told me you would be interested in [link].&quot;);
    }

    public static List&lt;Profile&gt; createTestProfiles() {
        List&lt;Profile&gt; data = new ArrayList&lt;Profile&gt;();
        data.add(new Profile(&quot;anna.smith@bing.com&quot;, &quot;Anna Smith&quot;, &quot;friends:mad_max@ya.com&quot;, &quot;friends:catwoman@yahoo.com&quot;, &quot;coworkers:sam@amazon.com&quot;));
        data.add(new Profile(&quot;mad_max@ya.com&quot;, &quot;Maximilian&quot;, &quot;friends:anna.smith@bing.com&quot;, &quot;coworkers:sam@amazon.com&quot;));
        data.add(new Profile(&quot;bill@microsoft.eu&quot;, &quot;Billie&quot;, &quot;coworkers:avanger@ukr.net&quot;));
        data.add(new Profile(&quot;avanger@ukr.net&quot;, &quot;John Day&quot;, &quot;coworkers:bill@microsoft.eu&quot;));
        data.add(new Profile(&quot;sam@amazon.com&quot;, &quot;Sam Kitting&quot;, &quot;coworkers:anna.smith@bing.com&quot;, &quot;coworkers:mad_max@ya.com&quot;, &quot;friends:catwoman@yahoo.com&quot;));
        data.add(new Profile(&quot;catwoman@yahoo.com&quot;, &quot;Liza&quot;, &quot;friends:anna.smith@bing.com&quot;, &quot;friends:sam@amazon.com&quot;));
        return data;
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Please specify social network to target spam tool (default:Facebook):
1. Facebook
2. LinkedIn
&gt; 1

Iterating over friends...

Facebook: Loading &#39;friends&#39; list of &#39;anna.smith@bing.com&#39; over the network...
Facebook: Loading profile &#39;mad_max@ya.com&#39; over the network...
Sent message to: &#39;mad_max@ya.com&#39;. Message body: &#39;Hey! This is Anna&#39;s friend Josh. Can you do me a favor and like this post [link]?&#39;
Facebook: Loading profile &#39;catwoman@yahoo.com&#39; over the network...
Sent message to: &#39;catwoman@yahoo.com&#39;. Message body: &#39;Hey! This is Anna&#39;s friend Josh. Can you do me a favor and like this post [link]?&#39;

Iterating over coworkers...

Facebook: Loading &#39;coworkers&#39; list of &#39;anna.smith@bing.com&#39; over the network...
Facebook: Loading profile &#39;sam@amazon.com&#39; over the network...
Sent message to: &#39;sam@amazon.com&#39;. Message body: &#39;Hey! This is Anna&#39;s boss Jason. Anna told me you would be interested in [link].&#39;</code></pre>
</figure>

::: next
#### Leia a seguir

[Mediator em Java []{.fa
.fa-arrow-right}](../../mediator/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Command em
Java](../../command/java/example.html){.btn .btn-default rel="prev"}
:::

## **Iterator** em outras linguagens

[![Iterator em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Iterator em C#"){.prog-lang-link}
[![Iterator em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Iterator em C++"){.prog-lang-link}
[![Iterator em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Iterator em Go"){.prog-lang-link}
[![Iterator em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Iterator em PHP"){.prog-lang-link}
[![Iterator em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Iterator em Python"){.prog-lang-link}
[![Iterator em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Iterator em Ruby"){.prog-lang-link}
[![Iterator em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Iterator em Rust"){.prog-lang-link}
[![Iterator em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Iterator em Swift"){.prog-lang-link}
[![Iterator em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Iterator em TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::
