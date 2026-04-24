{
    page: 15,
    from: "https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html",
    update: "2026-03-30",
    by: ["Arthur Leroux"],
}

# _**Personnaliser la Nature d'un Bean**_

Le framework _Spring_ fournit un certain nombre d’interfaces que vous pouvez utiliser pour personnaliser la nature d’un **bean**. Cette section les regroupe comme suit :

* [Callback du cycle de vie]()
* [`ApplicationContextAware` et `BeanNameAware`]()
* [Autres interfaces `Aware`]()

## _**Callback du cycle de vie**_

Pour interagir avec la gestion du cycle de vie des **beans** du conteneur, vous pouvez implémenter les interfaces `InitializingBean` et `DisposableBean`. Le conteneur appelle la méthode `afterPropertiesSet()` pour la première et `destroy()` pour la seconde afin de permettre au **bean** d’exécuter certaines actions lors de l’initialisation et de la destruction de vos **beans**.

> **TIP**
>
> Les annotations JSR-250 `@PostConstruct` et `@PreDestroy` sont généralement considérées comme une meilleure pratique pour recevoir des callbacks du cycle de vie dans les applications modernes _Spring_. Utiliser ces annotations signifie que vos **beans** ne sont couplés à aucune interface de _Spring_. Pour plus de détails, voir [Using `PostConstruct` et `PreDestroy()`]().
>
> Si vous ne souhaitez pas utiliser les annotations JSR-250 mais que vous voulez tout de même supprimer ce couplage, vous pouvez utiliser les métadonnées des définitions de **beans** `init-method` et `destroy-method`.

En interne, le framework _Spring_ utilise les implémentations de `BeanPostProcessor` pour traiter toutes les interfaces de callback qu’il peut trouver et appeler les méthodes appropriées. Si vous avez besoin de fonctionnalités personnalisées ou d’autres comportements du cycle de vie que _Spring_ n’offre pas par défaut, vous pouvez implémenter `BeanPostProcessor` vous-même. Pour plus d’informations, voir [Container Extension Points]().

En plus des callbacks d’initialisation et de destruction, les objets gérés par _Spring_ peuvent également implémenter l’interface `Lifecycle`, de sorte que ces objets puissent participer au démarrage et à l’arrêt du processus, comme le fait le propre cycle de vie du conteneur.

Les interfaces de callback du cycle de vie sont décrites dans cette section.

### _**Initialisation des callbacks**_

L’interface `org.springframework.beans.factory.InitializingBean` permet à un **bean** de gérer le travail d’initialisation après que le conteneur a appliqué toutes les propriétés nécessaires sur le **bean**. L’interface `InitializingBean` possède une unique méthode :

```java
void afterPropertiesSet() throws Exception;
```

Nous recommandons de ne pas utiliser l’interface `InitializingBean`, car cela couple inutilement le code à _Spring_. À la place, nous suggérons d’utiliser l’annotation `@PostConstruct` ou bien de spécifier une méthode d’initialisation POJO.

Dans le cas d’une configuration des métadonnées orientée XML, vous pouvez utiliser l’attribut `init-method` pour spécifier le nom de la méthode, qui doit avoir une signature `void` sans argument. Avec une configuration Java, vous pouvez utiliser l’attribut `initMethod` de `@Bean`. Voir [Receiving Lifecycle Callbacks]().

Considérez l’exemple suivant :

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```

```java
public class ExampleBean {

    public void init() {
        // do some initialization work
    }
}
```

```kotlin
class ExampleBean {

    fun init() {
        // do some initialization work
    }
}
```

L’exemple précédent a presque le même effet que l’exemple suivant (qui consiste en deux extraits de code) :

```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```java
public class AnotherExampleBean implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        // do some initialization work
    }
}
```

```kotlin
class AnotherExampleBean : InitializingBean {

    override fun afterPropertiesSet() {
        // do some initialization work
    }
}
```

Cependant, le premier des deux exemples précédents ne couple pas le code à _Spring_.

> **NOTE**
>
> Soyez attentif au fait que `@PostConstruct` et les méthodes d’initialisation, en général, sont exécutées dans le verrou de création des singletons du conteneur. L’instance du **bean** est seulement considérée comme pleinement initialisée et prête à être publiée aux autres beans après avoir été retournée depuis la méthode `@PostConstruct`.
>
> De telles méthodes d’initialisation individuelles ont uniquement pour but de valider l’état de la configuration et, éventuellement, de préparer certaines structures de données basées sur cette configuration, mais pas d’effectuer davantage d’activités impliquant des accès à des **beans** externes. Sinon, il existe un risque de blocage (_deadlock_) lors de l’initialisation.
>
> Pour les scénarios où des activités coûteuses post-initialisation doivent être déclenchées (par exemple, des étapes de préparation asynchrones d’une base de données), votre **bean** devrait soit implémenter `SmartInitializingSingleton.afterSingletonsInstantiated()`, soit se baser sur l’événement de rafraîchissement du contexte : en implémentant `ApplicationListener<ContextRefreshedEvent>` ou en déclarant son équivalent via `@EventListener(ContextRefreshedEvent.class)`.
>
> Ces variantes interviennent après toutes les initialisations régulières des singletons, et donc en dehors de tout verrou de création de **beans**.
>
> En alternative, vous pouvez implémenter l’interface `(Smart)Lifecycle` et vous intégrer à la gestion globale du cycle de vie du conteneur, incluant un mécanisme d’auto-démarrage, une phase d’arrêt (_pre-destroy_), ainsi que d’éventuels callbacks de type arrêt/redémarrage (voir plus bas).


### _**Callbacks de destruction**_

Implémenter l’interface `org.springframework.beans.factory.DisposableBean` permet à un **bean** d’obtenir un callback lorsque le conteneur qui le contient est détruit. L’interface `DisposableBean` spécifie une unique méthode :

```java
void destroy() throws Exception;
```

Nous recommandons de ne pas utiliser l’interface `DisposableBean`, car cela couple inutilement le code à _Spring_. À la place, nous suggérons d’utiliser l’annotation `@PreDestroy` ou bien de spécifier une méthode générique supportée par les définitions de **beans**.

Dans le cas d’une configuration des métadonnées orientée XML, vous pouvez utiliser l’attribut `destroy-method` sur l’élément `<bean/>`. Avec une configuration Java, vous pouvez utiliser l’attribut `destroyMethod` de `@Bean`. Voir [Receiving Lifecycle Callbacks]. Considérez la définition suivante :

```xml
<bean id="exampleDestructionBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```

```java
public class ExampleBean {

    public void cleanup() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

```kotlin
class ExampleBean {

    fun cleanup() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

La définition précédente a presque exactement le même effet que la définition suivante :

```xml
<bean id="exampleDestructionBean" class="examples.AnotherExampleBean"/>
```

```java
public class AnotherExampleBean implements DisposableBean {

    @Override
    public void destroy() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

```kotlin
class AnotherExampleBean : DisposableBean {

    override fun destroy() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

Cependant, la première des deux définitions précédentes ne couple pas le code à _Spring_.

Remarquez que _Spring_ supporte également l’inférence des méthodes de destruction, en détectant une méthode publique `close` ou `shutdown`. Il s’agit du comportement par défaut pour les méthodes annotées `@Bean` dans les **classes** de configuration Java, et cela correspond automatiquement aux implémentations `java.lang.AutoCloseable` ou `java.io.Closeable`, afin de ne pas coupler la logique de destruction à _Spring_.

> **TIP**
>
> Pour l’inférence de la méthode de destruction en XML, vous pouvez assigner à l’attribut `destroy-method` d’un élément `<bean/>` une valeur spéciale `(inferred)`, qui indique à _Spring_ de détecter automatiquement une méthode `close` ou `shutdown` sur une définition de **bean** spécifique.  
> Vous pouvez aussi appliquer cette valeur spéciale `(inferred)` à l’attribut `default-destroy-method` d’un élément `<beans>` pour étendre ce comportement à l’ensemble des définitions de **beans** (voir *Default Initialization and Destroy Methods*).

> **NOTE**
>
> Pour des phases d’arrêt étendues, vous pouvez implémenter l’interface `Lifecycle` afin de recevoir un signal d’arrêt anticipé avant que les méthodes de destruction de n’importe quel **bean** singleton ne soient appelées.  
> Vous pouvez également implémenter `SmartLifecycle` pour définir une phase d’arrêt limitée dans le temps, durant laquelle le conteneur attendra que ces arrêts soient terminés avant de passer aux méthodes de destruction.


### _**Méthodes d'initialisation et de destruction par défaut**_

Lorsque vous écrivez des méthodes de callback d'initialisation et de destruction qui n'utilisent pas les interfaces de callback spécifiques de _Spring_ `InitializingBean` et `DisposableBean`, vous écrivez généralement des méthodes avec des noms tels que `init()`, `initialize()`, `dispose()`, et d'autres. Idéalement, les noms de telles méthodes de callback de cycle de vie sont standardisés à travers le projet afin que tous les développeurs utilisent les mêmes noms et que la cohérence soit assurée.

Vous pouvez configurer le conteneur de _Spring_ pour "rechercher" les noms des méthodes de callback d'initialisation et de destruction sur chaque **bean**. Cela signifie que vous, en tant que développeur d'applications, pouvez écrire vos **classes** applicatives et utiliser un callback d'initialisation appelé `init()`, sans avoir à configurer un attribut `init-method` pour chaque définition de **bean**. Le conteneur d'inversion de contrôle de _Spring_ appelle la méthode lorsque le **bean** est créé (en accord avec le contrat standard des callbacks du cycle de vie [décrit précédemment]()). Cette fonctionnalité renforce également une convention de nommage cohérente pour les méthodes de callback d'initialisation et de destruction.

Supposons que vos méthodes de callback d'initialisation soient nommées `init()` et vos méthodes de callback de destruction soient nommées `destroy()`. Votre **classe** ressemble alors à l'exemple suivant :

```java
public class DefaultBlogService implements BlogService {

    private BlogDao blogDao;

    public void setBlogDao(BlogDao blogDao) {
        this.blogDao = blogDao;
    }

    // this is (unsurprisingly) the initialization callback method
    public void init() {
        if (this.blogDao == null) {
            throw new IllegalStateException("The [blogDao] property must be set.");
        }
    }
}
```

```kotlin
class DefaultBlogService : BlogService {

    private var blogDao: BlogDao? = null

    // this is (unsurprisingly) the initialization callback method
    fun init() {
        if (blogDao == null) {
            throw IllegalStateException("The [blogDao] property must be set.")
        }
    }
}
```

Vous pouvez ensuite utiliser cette **classe** dans un **bean**, comme ceci :

```xml
<beans default-init-method="init">

    <bean id="blogService" class="com.something.DefaultBlogService">
        <property name="blogDao" ref="blogDao" />
    </bean>

</beans>
```

La présence de l'attribut `default-init-method` sur l'élément parent `<beans/>` fait que le conteneur d'inversion de contrôle de _Spring_ reconnaît une méthode appelée `init` sur la **classe** du **bean** comme une méthode de callback d'initialisation. Lorsque le **bean** est créé et assemblé, si la **classe** du **bean** possède une telle méthode, elle est invoquée au moment approprié.

Vous pouvez configurer une méthode de callback de destruction de façon similaire (en XML), en utilisant l'attribut `default-destroy-method` sur l'élément parent `<beans/>`.

Lorsque des **classes** de **beans** ont déjà des méthodes de callback nommées différemment de la convention, vous pouvez réécrire le comportement par défaut en spécifiant (en XML) le nom de la méthode en utilisant les attributs `init-method` et `destroy-method` de `<bean/>`.

Le conteneur de _Spring_ garantit qu'un callback d'initialisation configuré est appelé immédiatement après qu'un **bean** ait été fourni avec toutes ses dépendances. Ainsi, la méthode d'initialisation est appelée sur la référence brute du **bean**, ce qui signifie que les intercepteurs AOP et autres ne sont pas encore appliqués au **bean**. Le **bean** cible est pleinement créé dans un premier temps, puis un proxy AOP (par exemple) avec sa chaîne d'intercepteurs est appliqué. Si le **bean** cible et le proxy sont définis séparément, votre code peut même interagir avec le **bean** cible brut, en contournant le proxy. Il serait donc incohérent d'appliquer les intercepteurs à la méthode `init`, car cela couplerait le cycle de vie du **bean** cible au proxy ou aux intercepteurs et créerait des comportements inattendus lorsque votre code interagit directement avec le **bean** cible brut.

### _**Combiner les mécanismes de cycle de vie**_

Depuis _Spring_ 2.5, vous avez trois options pour contrôler le comportement du cycle de vie d'un **bean** :

* Les interfaces de callback `InitializingBean` et `DisposableBean`
* Des méthodes personnalisées `init()` et `destroy()`
* Les [annotations @PostConstruct et @PreDestroy]()

1. Vous pouvez combiner ces mécanismes pour contrôler un bean donné.

> **NOTE**
>
> Si plusieurs mécanismes du cycle de vie sont configurés pour un **bean** et que chaque mécanisme est configuré avec un nom de méthode différent, alors chaque méthode configurée est lancée dans l'ordre indiqué après cette note. Cependant, si le même nom de méthode est configuré — par exemple, `init()` pour une méthode d'initialisation — pour plus d'un de ces mécanismes de cycle de vie, cette méthode est exécutée une seule fois, comme expliqué dans la [section précédente](#méthodes-dinitialisation-et-de-destruction-par-défaut).

Plusieurs mécanismes du cycle de vie pour le même **bean**, avec différentes méthodes d'initialisation, sont appelés comme suit :

1. Les méthodes annotées avec `@PostConstruct`
2. `afterPropertiesSet` tel que défini par l'interface de callback `InitializingBean`
3. Une méthode personnalisée configurée `init()`

Les méthodes de destruction sont appelées dans le même ordre :

1. Les méthodes annotées avec `@PreDestroy`
2. `destroy()` tel que défini par l'interface `DisposableBean`
3. Une méthode personnalisée configurée `destroy()`

---

### _**Callbacks de lancement et d'arrêt**_

L'interface `Lifecycle` définit les méthodes essentielles pour n'importe quel objet qui a ses propres prérequis de cycle de vie (tels que lancer et arrêter des processus en arrière-plan) :

```java
public interface Lifecycle {

	void start();

	void stop();

	boolean isRunning();
}
```

N'importe quel objet géré par _Spring_ peut implémenter l'interface `Lifecycle`. Ainsi, lorsque l'`ApplicationContext` lui-même reçoit des signaux de lancement et d'arrêt (par exemple, pour un scénario stop/restart à l'exécution), il propage ces appels à toutes les implémentations `Lifecycle` définies dans ce contexte. Cela s'effectue par délégation à un `LifecycleProcessor`, comme montré dans le code suivant :

```java
public interface LifecycleProcessor extends Lifecycle {

	void onRefresh();

	void onClose();
}
```

Remarquez que l'interface `LifecycleProcessor` elle-même est une extension de l'interface `Lifecycle`. Elle ajoute également deux autres méthodes pour réagir lorsque le contexte est rafraîchi ou fermé.

> **TIP**
>
> Remarquez que l'interface régulière `org.springframework.context.Lifecycle` est un contrat simple pour des notifications explicites de lancement et d'arrêt et n'implique aucun auto-lancement au moment du rafraîchissement du contexte. Pour un contrôle plus fin sur l'auto-lancement et pour un arrêt **graceful** d'un **bean** spécifique (incluant les phases de lancement et d'arrêt), considérez plutôt l'implémentation de l'interface étendue `org.springframework.context.SmartLifecycle`.
>
> Par ailleurs, remarquez s'il vous plaît que les notifications d'arrêt ne sont pas garanties d'arriver avant la destruction. Lors d'un arrêt régulier, tous les **beans** `Lifecycle` reçoivent d'abord une notification d'arrêt avant que le callback de destruction générale ne soit propagé. Cependant, lors d'un rafraîchissement à chaud durant la vie du contexte ou lors de tentatives d'arrêt pendant un rafraîchissement, seules les méthodes de destruction sont appelées.

L'ordre d'invocation du lancement et de l'arrêt peut être important. Si une relation **"depends-on"** existe entre des objets, le composant dépendant démarre après sa dépendance et s'arrête avant celle-ci. Cependant, parfois, les dépendances directes sont inconnues. Vous pouvez seulement savoir quels objets d'un certain type doivent démarrer avant ceux d'un autre type. Dans ces cas, l'interface `SmartLifecycle` définit une autre option, à savoir la méthode `getPhase()`, telle que définie dans son interface parente `Phased`. Le code suivant montre la définition de l'interface `Phased` :

```java
public interface Phased {

	int getPhase();
}
```

Le code suivant montre la définition de l'interface `SmartLifecycle` :

```java
public interface SmartLifecycle extends Lifecycle, Phased {

	boolean isAutoStartup();

	void stop(Runnable callback);
}
```

Lors du lancement, les objets avec la plus petite phase démarrent en premier. Lors de l'arrêt, l'ordre inverse est suivi. Ainsi, un objet qui implémente `SmartLifecycle` dont la méthode `getPhase()` retourne `Integer.MIN_VALUE` devrait être parmi les premiers au lancement et parmi les derniers à l'arrêt. À l'autre extrémité du spectre, une phase évaluée à `Integer.MAX_VALUE` indique que l'objet doit être lancé en dernier et arrêté en premier (probablement parce qu'il dépend d'autres processus en cours d'exécution).

Lors de la prise en compte des phases, il est également important de savoir que la phase par défaut pour tout objet `Lifecycle` "standard" qui n'implémente pas `SmartLifecycle` est 0. Ainsi, toute phase négative indique qu'un objet doit démarrer avant les composants standards (et s'arrêter après eux). L'inverse est vrai pour toute phase positive.

La méthode d'arrêt définie par `SmartLifecycle` accepte un callback. Toute implémentation doit invoquer la méthode `run()` du callback une fois le processus d'arrêt terminé. Cela permet un arrêt asynchrone si nécessaire, puisque l'implémentation par défaut de l'interface `LifecycleProcessor`, `DefaultLifecycleProcessor`, attend jusqu'à l'expiration du délai configuré pour chaque groupe d'objets dans une phase donnée avant d'invoquer le callback. La valeur par défaut du délai est de 30 secondes.

Vous pouvez redéfinir l'instance par défaut du processeur de cycle de vie en déclarant un **bean** nommé `lifecycleProcessor` dans le contexte. Si vous souhaitez seulement modifier le délai d'expiration, définir l'exemple suivant devrait suffire :

```xml
<bean id="lifecycleProcessor" class="org.springframework.context.support.DefaultLifecycleProcessor">
	<!-- timeout value in milliseconds -->
	<property name="timeoutPerShutdownPhase" value="10000"/>
</bean>
```

Comme indiqué précédemment, l'interface `LifecycleProcessor` définit également des méthodes de callback pour le rafraîchissement et la fermeture du contexte. La dernière méthode déclenche l'arrêt du processus comme si `stop()` avait été appelée explicitement, mais cela se produit lorsque le contexte est en train de s'arrêter.

Le callback de rafraîchissement, quant à lui, permet une autre fonctionnalité des **beans** `SmartLifecycle`. Quand le contexte est rafraîchi (après que tous les objets ont été instanciés et initialisés), ce callback est appelé. À ce moment, le processeur par défaut du cycle de vie vérifie la valeur booléenne retournée par la méthode `isAutoStartup()` pour chaque objet `SmartLifecycle`. Si elle retourne `true`, cet objet est démarré à ce moment, plutôt que d'attendre une invocation explicite du contexte ou de sa propre méthode `start()`.

Contrairement au rafraîchissement du contexte, le lancement du contexte ne se produit pas automatiquement pour une implémentation standard. La valeur de `getPhase()` ainsi que les relations **"depends-on"** déterminent l'ordre de lancement, comme décrit précédemment.

### _**Arrêter le conteneur d'inversion de contrôle de _Spring_ proprement dans les applications non-web**_

> **NOTE**
>
> Cette section s'applique seulement aux applications non-web. L'implémentation de l'`ApplicationContext` de _Spring_ orientée web possède déjà du code en place pour un arrêt propre du conteneur d'inversion de contrôle lorsque l'application web correspondante est arrêtée.

Si vous utilisez le conteneur d'inversion de contrôle de _Spring_ dans un environnement d'application non-web (par exemple, dans une application riche côté client), enregistrez un **hook d'arrêt** auprès de la JVM. Cela assure un arrêt propre et appelle les méthodes de destruction pertinentes sur vos **beans** singleton afin que toutes les ressources soient libérées. Vous devez encore configurer et implémenter ces callbacks de destruction correctement.

Pour enregistrer un arrêt propre, appelez la méthode `registerShutdownHook()` qui est déclarée dans l'interface `ConfigurableApplicationContext`, comme le montre l'exemple suivant :

```java
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Boot {

	public static void main(final String[] args) throws Exception {
		ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");

		// Ajouter un hook d'arrêt pour le contexte ci-dessus...
		ctx.registerShutdownHook();

		// L'application s'exécute ici...

		// La méthode main se termine, le hook est appelé avant l'arrêt de l'application...
	}
}
```

```kotlin
import org.springframework.context.support.ClassPathXmlApplicationContext

fun main() {
	val ctx = ClassPathXmlApplicationContext("beans.xml")

	// Ajouter un hook d'arrêt pour le contexte ci-dessus...
	ctx.registerShutdownHook()

	// L'application s'exécute ici...

	// La fonction main se termine, le hook est appelé avant l'arrêt de l'application...
}
```

---

### _**Sûreté des threads et visibilité**_

Le conteneur principal de _Spring_ met à disposition les instances des singletons créées de façon **thread-safe**, en gardant l'accès protégé par un verrou singleton et en garantissant la visibilité dans les autres threads.

Par conséquent, les **classes de beans** fournies par l'application ne doivent pas se préoccuper de la visibilité de leur état d'initialisation. Les champs de configuration réguliers ne doivent pas être marqués `volatile` tant qu'ils sont modifiés uniquement durant la phase d'initialisation. La publication sécurisée garantit un état similaire à `final` même pour les champs configurés via setter qui sont mutables durant la phase initiale. Si ces champs sont modifiés après la phase de création et les publications initiales, ils doivent être déclarés comme `volatile` ou protégés par un verrou commun lors de leur accès.

Remarquez que les accès concurrents à ces configurations d'état dans les instances des **beans** singleton — par exemple, les instances de **controllers** ou de **repositories** — sont parfaitement **thread-safe** après une telle publication initiale sécurisée par le conteneur. Cela inclut également les instances singleton de `FactoryBean`, qui sont traitées sous le verrou singleton général.

Pour les callbacks de destruction, la configuration d'état reste **thread-safe**, mais tout état d'exécution accumulé entre l'initialisation et la destruction devrait être conservé dans des structures **thread-safe** (ou dans des champs `volatile` pour des cas simples), conformément aux directives Java courantes.

Une intégration plus avancée de `Lifecycle`, comme montré ci-dessus, implique une modification mutable de l'état, par exemple le champ `runnable`, qui doit être déclaré comme `volatile`. Bien que les callbacks du cycle de vie suivent un certain ordre — par exemple, un callback de démarrage est garanti de survenir uniquement après une initialisation complète, et un callback d'arrêt uniquement après un démarrage initial — il existe un cas particulier pour un arrêt avant l'exécution des callbacks de destruction : il est fortement recommandé que l'état interne de n'importe lequel de ces **beans** permette un callback de destruction immédiat, même si l'arrêt survient durant un arrêt extraordinaire après un démarrage annulé ou dans le cas d'un arrêt par expiration du délai dans un autre **bean**.


## _**ApplicationContextAware et BeanNameAware**_

Lorsqu'un `ApplicationContext` crée une instance d'un objet qui implémente l'interface `org.springframework.context.ApplicationContextAware`, l'instance est fournie avec une référence à cette `ApplicationContext`. Le code suivant montre la définition de l'interface `ApplicationContextAware` :

```java
public interface ApplicationContextAware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```

Ainsi, les **beans** peuvent manipuler programmatiquement l'`ApplicationContext` qui les crée, soit via l'interface `ApplicationContextAware`, soit en castant la référence vers une sous-classe connue de cette interface (comme `ConfigurableApplicationContext`, qui expose des fonctionnalités supplémentaires).  

Un usage possible : rechercher dynamiquement d’autres **beans** dans le contexte. Cependant, cette pratique **couple fortement votre code à Spring**, et va à l’encontre du principe d’inversion de contrôle (IoC).  

D’autres méthodes de l'`ApplicationContext` offrent l’accès aux fichiers ressources, à la publication d’événements et à la résolution de messages (`MessageSource`). Ces fonctionnalités sont détaillées dans [Additional Capabilities of the `ApplicationContext`]().  

**Alternative à ApplicationContextAware :** l’auto-câblage (`@Autowired`) d’un champ ou d’un paramètre de constructeur de type `ApplicationContext` :

```java
@Autowired
private ApplicationContext context;
```

---

Lorsqu'un `ApplicationContext` crée une classe qui implémente l'interface `org.springframework.beans.factory.BeanNameAware`, la classe est fournie avec le nom défini dans sa définition de bean. La définition de l’interface `BeanNameAware` est la suivante :

```java
public interface BeanNameAware {

    void setBeanName(String name) throws BeansException;
}
```

- Le callback est invoqué **après l’injection des propriétés normales**, mais **avant un callback d’initialisation** comme `InitializingBean.afterPropertiesSet()` ou une méthode init personnalisée.
- Usage typique : identification dynamique d’un bean, logging ou enregistrement dans un registre.

---

## _**Autres interfaces Aware**_

À côté de `ApplicationContextAware` et `BeanNameAware`, Spring offre un large éventail d’interfaces de callback `Aware`, permettant aux beans de déclarer qu’ils ont besoin de services ou d’infrastructure du conteneur.  

| Nom | Dépendance injectée | Utilité / contexte |
|-----|------------------|------------------|
| `ApplicationContextAware` | `ApplicationContext` | Accès programmatique au contexte |
| `ApplicationEventPublisherAware` | `ApplicationEventPublisher` | Publier des événements dans le contexte |
| `BeanClassLoaderAware` | ClassLoader | Accès au class loader utilisé pour charger les classes des beans |
| `BeanFactoryAware` | `BeanFactory` | Obtenir le BeanFactory créateur du bean |
| `LoadTimeWeaverAware` | LoadTimeWeaver | Tissage des classes au chargement (AspectJ, etc.) |
| `MessageSourceAware` | `MessageSource` | Résolution de messages i18n avec paramètres |
| `NotificationPublisherAware` | `NotificationPublisher` | Publication de notifications JMX |
| `ResourceLoaderAware` | `ResourceLoader` | Accès aux fichiers et ressources |
| `ServletConfigAware` | `ServletConfig` | Accès à la configuration du servlet (Spring MVC) |
| `ServletContextAware` | `ServletContext` | Accès au contexte du servlet (Spring MVC) |

---


Chaque interface Aware fournit **une dépendance précise**, comme son nom l’indique.
Elles **attachent le bean à l’API Spring**, donc leur usage doit rester limité aux beans qui ont besoin d’un accès programmatique au conteneur.
Pour la majorité des beans, il est recommandé d’utiliser **l’injection par constructeur, par propriété ou par annotation** plutôt qu’une interface Aware.





