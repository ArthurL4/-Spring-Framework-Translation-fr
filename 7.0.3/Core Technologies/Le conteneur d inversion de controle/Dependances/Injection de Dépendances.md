{
    page:8,
    from:"https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html",
    update:"2026-02-23",
    by:["Arthur Leroux"],
}

# __*Injection de Dépendances*__

L'injection de dépendances (DI) est le procédé par lequel les objets définissent leurs dépendances (c'est-à-dire les autres objets avec lesquels ils travaillent) seulement à travers un constructeur avec arguments, des arguments à une méthode **factory**, ou bien des propriétés de l'instance de l'objet qui sont mises en place après que l'instance ait été créée ou retournée par une méthode **factory**. Le conteneur injecte ensuite ces dépendances quand il crée le **bean**. Ce processus est fondamentalement l'inverse (d'où le nom d'inversion de contrôle) du **bean** contrôlant lui-même l'instanciation ou la localisation de ses dépendances en utilisant directement la construction de **classes** ou du pattern **Service Locator**.

Le code est plus propre avec le principe d'injection de dépendances, et le découplement du code est plus effectif quand les objets sont fournis avec leurs dépendances. L'objet ne recherche pas ses dépendances et ne connaît pas la localisation ou les **classes** des dépendances. Il en résulte que vos **classes** deviennent plus faciles à tester, particulièrement quand les dépendances sont sur des **interfaces** ou des **abstract classes**, permettant le **stub** ou le **mock** des implémentations qui doivent être dans des tests unitaires.

L'injection de dépendances existe dans deux variantes majeures : [Constructor-based dependency injection](#injection-de-dépendances-par-constructeur) et [Setter-based dependency injection]()

## _**Injection de dépendances par constructeur**_

L'injection de dépendances par constructeur est accomplie par le conteneur en invoquant un constructeur avec un certain nombre d'arguments, chacun représentant une dépendance. Appeler une méthode `static` **factory** avec des arguments spécifiques pour construire le **bean** est presque équivalent. Cette discussion traite donc les arguments d’un constructeur et ceux d’une méthode `static` **factory** de manière équivalente.

L'exemple suivant montre une **classe** qui peut uniquement être injectée par constructeur :

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private final MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

```kotlin
// a constructor so that the Spring container can inject a MovieFinder
class SimpleMovieLister(private val movieFinder: MovieFinder) {
    // business logic that actually uses the injected MovieFinder is omitted...
}
```

Remarquez qu'il n'y a rien de spécial concernant cette **classe**. C'est un **POJO** qui n'a aucune dépendance envers une interface, une **classe** ou une annotation spécifique au conteneur.

### _**Résolution des arguments d'un constructeur**_

La correspondance lors de la résolution des arguments d'un constructeur s’effectue en utilisant le type de l’argument. S’il n’existe aucune ambiguïté possible dans les arguments du constructeur d’une définition de **bean**, l’ordre dans lequel les arguments du constructeur sont définis dans la définition du **bean** correspond à l’ordre dans lequel ces arguments sont fournis au constructeur approprié lorsque le **bean** est instancié.

Considérez la **classe** suivante :

```java
package x.y;

public class ThingOne {

    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        // ...
    }
}
```

```kotlin
package x.y

class ThingOne(thingTwo: ThingTwo, thingThree: ThingThree)
```

Supposons que les **classes** `ThingTwo` et `ThingThree` ne soient pas reliées par héritage : aucune ambiguïté potentielle n’existe alors. La configuration suivante fonctionne donc parfaitement et vous n’avez pas besoin de spécifier explicitement les indices ou les types des arguments du constructeur dans l’élément `<constructor-arg/>`.

```xml
<beans>
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>

    <bean id="beanTwo" class="x.y.ThingTwo"/>

    <bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```

Lorsqu’un autre **bean** est référencé, le type est connu et la correspondance peut avoir lieu (comme dans l’exemple précédent). Lorsqu’un type simple est utilisé, tel que `<value>true</value>`, _Spring_ ne peut pas déterminer le type de la valeur et ne peut donc pas effectuer la correspondance par type sans aide.

Considérez la **classe** suivante :

```java
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private final int years;

    // The Answer to Life, the Universe, and Everything
    private final String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

```kotlin
package examples

class ExampleBean(
    private val years: Int, // Number of years to calculate the Ultimate Answer
    private val ultimateAnswer: String // The Answer to Life, the Universe, and Everything
)
```

#### _**Correspondance des arguments d'un constructeur par type**_

Dans le scénario précédent, le conteneur peut utiliser la correspondance par type avec des types simples si vous spécifiez explicitement le type de l’argument du constructeur via l’attribut `type`, comme le montre l’exemple suivant :

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

#### _**Correspondance des arguments d'un constructeur par indice**_

Vous pouvez utiliser l’attribut `index` pour spécifier explicitement l’indice des arguments du constructeur, comme le montre l’exemple suivant :

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```

En plus de lever les ambiguïtés liées à plusieurs valeurs simples, la spécification de l’indice permet également de résoudre les ambiguïtés lorsqu’un constructeur possède deux arguments du même type.

> **NOTE**  
> L’indice commence à 0.

#### _**Correspondance des arguments d'un constructeur par nom**_

Vous pouvez également utiliser le nom des paramètres du constructeur pour lever toute ambiguïté, comme le montre l’exemple suivant :

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

Gardez à l’esprit que, pour que cela fonctionne, votre code doit être compilé avec l’option `-parameters` activée afin que _Spring_ puisse récupérer les noms des paramètres du constructeur. Si vous ne pouvez pas ou ne souhaitez pas compiler votre code avec l’option `-parameters`, vous pouvez utiliser l’annotation `@ConstructorProperties` du JDK pour nommer explicitement les arguments du constructeur. La **classe** suivante ressemblerait alors à ceci :

```java
package examples;

import java.beans.ConstructorProperties;

public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

```kotlin
package examples

import java.beans.ConstructorProperties

class ExampleBean
@ConstructorProperties("years", "ultimateAnswer")
constructor(val years: Int, val ultimateAnswer: String)
```

## _**Injection de dépendances par setter**_

L’injection de dépendances par **setter** est effectuée par le conteneur en appelant les méthodes **setter** sur vos **beans** après avoir invoqué un constructeur sans argument ou une méthode `static` **factory** sans argument afin d’instancier le **bean**.

L’exemple suivant montre une **classe** qui ne peut être injectée que par **setter**. Cette classe est conventionnelle en Java. C’est un **POJO** qui n’a aucune dépendance envers une interface, une **classe** ou une annotation spécifique au conteneur.

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

```kotlin
class SimpleMovieLister {

    // a late-initialized property so that the Spring container can inject a MovieFinder
    lateinit var movieFinder: MovieFinder

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

L’`ApplicationContext` prend en charge l’injection de dépendances à la fois par constructeur et par **setter** pour les **beans** qu’il gère. Il prend également en charge l’injection par **setter** après que des dépendances ont déjà été injectées via le constructeur.

Vous configurez les dépendances sous la forme d’un `BeanDefinition`, utilisé conjointement avec des instances de `PropertyEditor` afin de convertir les propriétés d’un format vers un autre. Cependant, la plupart des utilisateurs de _Spring_ ne manipulent pas ces classes directement (c’est-à-dire de manière programmatique), mais travaillent plutôt avec des définitions de **beans** XML, des composants annotés (c’est-à-dire des **classes** annotées avec `@Component`, `@Controller`, etc.), ou des méthodes annotées `@Bean` dans des **classes** annotées `@Configuration` pour une approche basée sur Java. Ces sources sont ensuite converties en interne en instances de `BeanDefinition` et utilisées pour charger une instance complète d’un conteneur d’inversion de contrôle _Spring_.

## **Injection par constructeur ou par setter ?**
Bien qu’il soit possible de combiner l’injection de dépendances par constructeur et par **setter**, il est recommandé d’utiliser principalement les constructeurs pour les dépendances obligatoires et les **setters** ou méthodes de configuration pour les dépendances optionnelles. Notez que l’annotation `@Autowired` appliquée à une méthode **setter** peut rendre la propriété obligatoire ; cependant, l’injection par constructeur avec validation programmatique reste préférable.

L’équipe _Spring_ recommande généralement l’injection par constructeur, car elle permet d’implémenter les composants de l’application comme des objets immuables et garantit que les dépendances nécessaires ne soient pas `null`. De plus, les composants injectés par constructeur sont toujours retournés au code appelant dans un état complètement initialisé. En revanche, un constructeur avec de nombreux arguments est souvent le signe d’un mauvais design, indiquant que la **classe** possède trop de responsabilités et devrait être scindée afin de mieux respecter la séparation des responsabilités.

L’injection par **setter** devrait être principalement utilisée pour des dépendances optionnelles pouvant raisonnablement être initialisées avec une valeur
par défaut dans la **classe**. Dans ce cas, une vérification de non-nullité doit être effectuée là où la dépendance est utilisée. Un avantage de l’injection par **setter** est qu’elle permet de reconfigurer ou de réinjecter les objets après leur création. La gestion via JMX MBeans constitue donc un cas d’usage pertinent pour l’injection par **setter**.

Utilisez le style d’injection de dépendances le plus adapté à une **classe** donnée. Parfois, lorsque vous travaillez avec des **classes** externes dont vous ne possédez pas le code source, le choix peut s’imposer à vous. Par exemple, si une **classe** externe n’expose aucun **setter**, l’injection par constructeur peut être la seule solution viable.

## _**Processus de résolution des dépendances**_

Le conteneur résout les dépendances d'un **bean** de la façon suivante :

* L'`ApplicationContext` est créé et initialisé avec la configuration des métadonnées qui décrivent tous les **beans**. La configuration des métadonnées peut être spécifiée par XML, en Java ou par des annotations.
* Pour chaque **bean**, ses dépendances sont exprimées sous la forme de propriétés, d'arguments du constructeur ou d'arguments d'une méthode `static` **factory** (si vous utilisez cela au lieu d'un constructeur normal). Ces dépendances sont fournies au **bean** quand le **bean** est effectivement créé.
* Chaque propriété ou argument d'un constructeur est une définition réelle de la valeur à appliquer, ou une référence à un autre **bean** dans un conteneur.
* Chaque propriété ou argument d'un constructeur qui est une valeur est converti depuis son format spécifique au type courant de cette propriété ou argument du constructeur. Par défaut, _Spring_ peut convertir une valeur fournie dans un format **String** vers n'importe quel type primitif, tel que `int`, `long`, `String`, `boolean` et autres.

Le conteneur de _Spring_ valide la configuration pour chaque **bean** au moment où le conteneur est créé. Cependant, les propriétés des **beans** elles-mêmes ne sont pas appliquées jusqu'à ce que le **bean** soit réellement créé. Les **beans** qui ont un scope **singleton** et configurés pour être pré-instanciés (par défaut) sont créés quand le conteneur est créé. Les scopes sont définis dans [Bean Scopes](). Sinon, le **bean** est créé seulement quand il est requis. La création d'un **bean** peut potentiellement causer un graphe de **beans** qui doivent être également créés, car les dépendances du **bean**, ainsi que les dépendances des dépendances (et ainsi de suite), sont créées et assignées. Remarquez que la mauvaise correspondance dans cette résolution parmi les dépendances peut n'apparaître que plus tard — c'est-à-dire durant la première création d'un **bean** affecté.

> **Dépendances circulaires**  
> Si vous utilisez de manière prédominante l'injection par constructeur, il est possible de créer un scénario irré-soluble de dépendance circulaire.
>
> Par exemple : une **Class A** nécessite une instance de la **Class B** à travers l'injection par constructeur, et la **Class B** nécessite une instance de la **Class A** à travers l'injection par constructeur. Si vous configurez les **beans** pour les **classes A et B** à être injectés l'un dans l'autre, le conteneur d'inversion de contrôle de _Spring_ détecte cette dépendance circulaire à l'exécution et lance une exception `BeanCurrentlyInCreationException`.
>
> Une solution possible est d'éditer le code source des **classes** pour qu'elles soient configurées par **setter** plutôt que par constructeur. Alternativement, évitez l'injection par constructeur et utilisez seulement l'injection par **setter**. En d'autres mots, bien que cela ne soit pas recommandé, vous pouvez configurer des dépendances circulaires avec l'injection par **setter**.
>
> Contrairement au cas typique (sans dépendance circulaire), une dépendance circulaire entre un **bean** A et un **bean** B force un des **beans** à être injecté dans l'autre avant d'être lui-même complètement initialisé (un scénario classique de l'œuf et la poule).

Vous pouvez généralement faire confiance à _Spring_ pour faire les choses correctement. Il détecte les problèmes de configuration, tels que les références à un **bean** non existant et les dépendances circulaires, au chargement du conteneur. _Spring_ applique les propriétés et résout les dépendances aussi tard que possible, quand le **bean** est effectivement créé. Cela signifie qu'un conteneur _Spring_ qui a chargé correctement peut plus tard générer une exception quand vous demandez un objet s'il y a un problème de création de cet objet ou de l'une de ses dépendances — par exemple, le **bean** lance une exception pour une propriété manquante ou invalide. Cela retarde potentiellement la visibilité des problèmes de configuration, et c'est pour cela que par défaut les implémentations de l'`ApplicationContext` pré-instancient les **beans singleton**. Cela a un coût en termes de temps et de mémoire au démarrage pour créer ces **beans** avant qu'ils soient réellement utilisés, mais vous découvrez les problèmes de configuration quand l'`ApplicationContext` est créé, pas plus tard. Vous pouvez encore réécrire le comportement par défaut de façon à ce que les **beans singleton** soient initialisés de manière paresseuse, plutôt que d'être pré-instanciés.

S'il n'existe pas de dépendance circulaire, quand un ou plusieurs **beans** collaborateurs sont injectés dans un **bean** dépendant, chaque **bean** collaborateur est totalement configuré avant d'être injecté dans le **bean** dépendant. Cela signifie que si un **bean** A a une dépendance sur un **bean** B, le conteneur d'inversion de contrôle de _Spring_ configure complètement le **bean** B avant d'invoquer la méthode **setter** sur le **bean** A. En d'autres mots, le **bean** est instancié (s'il n'est pas un **singleton** pré-instancié), ses dépendances sont appliquées, et les méthodes pertinentes du cycle de vie (telles que la [configured init method]() ou la [initializingBean callback method]()) sont invoquées.

## _**Exemples d'injection de Dépendances**_

L'exemple suivant utilise une configuration des métadonnées orientée XML pour l'injection de dépendances par **setter**. Une petite partie de la configuration XML spécifie certaines définitions de **beans** comme suit :

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

L'exemple suivant montre la **class** ExampleBean correspondante :

```java
public class ExampleBean {

    private AnotherBean beanOne;
    private YetAnotherBean beanTwo;
    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }
}
```

```kotlin
class ExampleBean {
    lateinit var beanOne: AnotherBean
    lateinit var beanTwo: YetAnotherBean
    var i: Int = 0
}
```

Dans l'exemple précédent, les **setters** sont déclarés pour correspondre aux propriétés spécifiées dans le fichier XML. L'exemple suivant utilise l'injection par constructeur :

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

L'exemple suivant montre la **class** ExampleBean correspondante :

```java
public class ExampleBean {

    private AnotherBean beanOne;
    private YetAnotherBean beanTwo;
    private int i;

    public ExampleBean(
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }
}
```

```kotlin
class ExampleBean(
    private val beanOne: AnotherBean,
    private val beanTwo: YetAnotherBean,
    private val i: Int)
```

Les arguments du constructeur spécifiés dans la définition du **bean** sont utilisés comme arguments pour le constructeur de ExampleBean. Maintenant, considérez une variante de cet exemple, où, au lieu d'utiliser un constructeur, _Spring_ appelle une méthode **static factory** pour retourner une instance de l'objet :

```xml
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

L'exemple suivant montre la **class** ExampleBean correspondante :

```java
public class ExampleBean {

    // a private constructor
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean(...);
        // some other operations...
        return eb;
    }
}
```

Les arguments de la méthode **static factory** sont fournis par l'élément `<constructor-arg/>`, exactement de la même manière que si c'était un constructeur qui était utilisé. Le type de **class** retourné par la méthode **static factory** ne doit pas être du même type que la **class** contenant la méthode **static factory** (bien que, dans cet exemple, ce soit le cas). Une instance (non-static) d'une méthode **factory** peut être utilisée de manière équivalente (hormis l'usage de l'attribut factory-bean au lieu de l'attribut class), aussi, nous ne discuterons pas de ces détails ici.
