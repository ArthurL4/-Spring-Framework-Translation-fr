{
    page:18,
    from:"https://docs.spring.io/spring-framework/reference/core/beans/annotation-config.html",
    update:"2026-04-10",
    by:["Arthur Leroux"],
}

> **NOTE**
> L’annotation `@Inject` du JSR-330 peut être utilisée à la place de l’annotation `@Autowired` de _Spring_ dans les exemples inclus dans cette section. Voir [ici]() pour plus de détails.

Vous pouvez appliquer l’annotation `@Autowired` aux constructeurs, comme le montre l’exemple suivant:

```java
public class MovieRecommender {

	private final CustomerPreferenceDao customerPreferenceDao;

	@Autowired
	public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
		this.customerPreferenceDao = customerPreferenceDao;
	}

	// ...
}
```

```kotlin
class MovieRecommender @Autowired constructor(
	private val customerPreferenceDao: CustomerPreferenceDao)
```

> **TIP**
>
> Une annotation `@Autowired` sur un tel constructeur n’est pas nécessaire si le **bean** cible ne définit qu’un seul constructeur. Cependant, si plusieurs constructeurs sont disponibles et qu’il n’existe pas de constructeur primaire ou par défaut, au moins un des constructeurs doit être annoté avec `@Autowired` afin d’indiquer à _Spring_ lequel utiliser. Voir la discussion sur [constructor resolution]() pour plus de détails.

Vous pouvez appliquer l’annotation `@Autowired` aux méthodes **setter** classiques, comme le montre l’exemple suivant:

```java
public class SimpleMovieLister {

	private MovieFinder movieFinder;

	@Autowired
	public void setMovieFinder(MovieFinder movieFinder) {
		this.movieFinder = movieFinder;
	}

	// ...
}
```

```kotlin
class SimpleMovieLister {

	@set:Autowired
	lateinit var movieFinder: MovieFinder

	// ...

}
```

Vous pouvez appliquer `@Autowired` à des méthodes avec des noms arbitraires et plusieurs arguments, comme le montre l’exemple suivant:

```java
public class MovieRecommender {

	private MovieCatalog movieCatalog;

	private CustomerPreferenceDao customerPreferenceDao;

	@Autowired
	public void prepare(MovieCatalog movieCatalog,
			CustomerPreferenceDao customerPreferenceDao) {
		this.movieCatalog = movieCatalog;
		this.customerPreferenceDao = customerPreferenceDao;
	}

	// ...
}
```

```kotlin
class MovieRecommender {

	private lateinit var movieCatalog: MovieCatalog

	private lateinit var customerPreferenceDao: CustomerPreferenceDao

	@Autowired
	fun prepare(movieCatalog: MovieCatalog,
				customerPreferenceDao: CustomerPreferenceDao) {
		this.movieCatalog = movieCatalog
		this.customerPreferenceDao = customerPreferenceDao
	}

	// ...
}
```

Vous pouvez également appliquer `@Autowired` aux champs et même le combiner avec des constructeurs, comme le montre l’exemple suivant:

```java
public class MovieRecommender {

	private final CustomerPreferenceDao customerPreferenceDao;

	@Autowired
	private MovieCatalog movieCatalog;

	@Autowired
	public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
		this.customerPreferenceDao = customerPreferenceDao;
	}

	// ...
}
```

```kotlin
class MovieRecommender @Autowired constructor(
	private val customerPreferenceDao: CustomerPreferenceDao) {

	@Autowired
	private lateinit var movieCatalog: MovieCatalog

	// ...
}
```

> **TIP**
>
> Assurez-vous que vos composants cibles (par exemple, `MovieCatalog` ou `CustomerPreferenceDao`) soient cohérents avec le type utilisé pour votre point d’injection annoté avec `@Autowired`. Sinon, l’injection peut échouer avec une erreur à l’exécution indiquant qu’aucune correspondance de type n’a été trouvée.
>
> Pour les **beans** définis en XML ou les **classes** composants détectées via le scan du classpath, le conteneur connaît généralement le type concret dès le départ. Cependant, pour les méthodes factory `@Bean`, vous devez vous assurer que le type de retour déclaré est suffisamment expressif. Pour les composants qui implémentent plusieurs interfaces ou qui peuvent être référencés par leur type d’implémentation, déclarez le type de retour le plus spécifique dans votre méthode factory (au moins aussi spécifique que requis par le point d’injection faisant référence à ce **bean**).

> **Injection réflexive**
>
> `@Autowired` peut également considérer des références réflexives pour l’injection (c’est-à-dire des références qui pointent vers le **bean** actuellement injecté).
>
> Notez toutefois que ce type d’injection est un mécanisme de repli. Les dépendances régulières vers d’autres composants restent prioritaires. En ce sens, les références réflexives ne participent pas à la sélection normale des candidats à l’autocâblage et ne sont donc jamais prioritaires. Au contraire, elles ont toujours la plus faible priorité.
>
> En pratique, vous devriez utiliser les références réflexives uniquement en dernier recours — par exemple pour appeler d’autres méthodes sur la même instance, comme via le proxy transactionnel du **bean**. Comme alternative, envisagez de factoriser les méthodes concernées dans un **bean** délégué séparé dans un tel scénario.
>
> Une autre alternative consiste à utiliser `@Resource`, qui peut obtenir un proxy du **bean** courant via son nom unique.
>
> **NOTE**
>
> Tenter d’injecter les résultats des méthodes `@Bean` dans la même **class** `@Configuration` constitue également un scénario de référence réflexive.
>
> Dans ce cas, soit vous résolvez ces références de manière paresseuse dans la signature de la méthode lorsqu’elles sont réellement nécessaires (plutôt que d’effectuer un autocâblage sur un champ dans une **class** de configuration), soit vous déclarez les méthodes `@Bean` comme `static`, ce qui les découple de l’instance de la **class** de configuration qui les contient ainsi que de son cycle de vie.
>
> Sinon, ces **beans** sont uniquement considérés comme solutions de repli, les **beans** correspondants provenant d’autres **classes** de configuration étant choisis comme candidats principaux (lorsqu’ils sont disponibles).

Vous pouvez également demander à _Spring_ de fournir tous les **beans** d’un type particulier depuis l’`ApplicationContext` en ajoutant l’annotation `@Autowired` à un champ ou à une méthode qui attend un tableau de ce type, comme le montre l’exemple suivant:

```java
public class MovieRecommender {

	@Autowired
	private MovieCatalog[] movieCatalogs;

	// ...
}
```

```kotlin
class MovieRecommender {

	@Autowired
	private lateinit var movieCatalogs: Array<MovieCatalog>

	// ...
}
```

La même chose s’applique aux collections typées, comme le montre l’exemple suivant:

```java
public class MovieRecommender {

	private Set<MovieCatalog> movieCatalogs;

	@Autowired
	public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs) {
		this.movieCatalogs = movieCatalogs;
	}

	// ...
}
```

```kotlin
class MovieRecommender {

	@Autowired
	lateinit var movieCatalogs: Set<MovieCatalog>

	// ...
}
```

> **TIP**
>
> Vos **beans** cibles peuvent implémenter l’interface `org.springframework.core.Ordered` ou utiliser les annotations `@Order` ou l’annotation standard `@Priority` si vous souhaitez que les éléments d’un tableau ou d’une liste soient triés dans un ordre spécifique. Sinon, leur ordre suit l’ordre d’enregistrement des définitions des **beans** correspondants dans le conteneur.
>
> Vous pouvez déclarer l’annotation `@Order` au niveau de la classe cible ainsi que sur les méthodes `@Bean`, éventuellement pour des définitions individuelles de **beans** (dans le cas de plusieurs définitions utilisant la même classe). Les valeurs de `@Order` peuvent influencer les priorités aux points d’injection, mais elles n’influencent pas l’ordre de démarrage des singletons, qui est une préoccupation distincte déterminée par les relations de dépendance et les déclarations `@DependsOn`.
>
> Notez que les annotations `@Order` sur les classes de configuration influencent uniquement l’ordre d’évaluation au démarrage au sein de l’ensemble des classes de configuration. Elles n’affectent pas les méthodes `@Bean` qu’elles contiennent. Pour le tri au niveau des **beans**, chaque méthode `@Bean` doit posséder sa propre annotation `@Order`, qui s’applique parmi les correspondances possibles pour le type spécifique du **bean** (tel que retourné par la méthode factory).
>
> Notez que l’annotation standard `jakarta.annotation.Priority` n’est pas disponible au niveau des méthodes `@Bean`, car elle ne peut pas être déclarée sur des méthodes. Sa sémantique peut être reproduite à l’aide des valeurs de `@Order` combinées avec `@Primary` ou `@Fallback` sur un seul **bean** pour chaque type.

Même les instances typées de `Map` peuvent être autocâblées tant que la clé attendue est de type `String`. Les valeurs de la map sont tous les **beans** du type attendu, et leurs clés sont les noms des **beans** correspondants, comme le montre l’exemple suivant:

```java
public class MovieRecommender {

	private Map<String, MovieCatalog> movieCatalogs;

	@Autowired
	public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
		this.movieCatalogs = movieCatalogs;
	}

	// ...
}
```

```kotlin
class MovieRecommender {

	@Autowired
	lateinit var movieCatalogs: Map<String, MovieCatalog>

	// ...
}
```

Par défaut, l’autocâblage échoue lorsqu’aucun **bean** candidat correspondant n’est disponible pour un point d’injection donné. Dans le cas d’un tableau, d’une collection ou d’une map, au moins un élément correspondant est attendu.

Le comportement par défaut est de traiter les méthodes annotées et les champs comme des dépendances requises. Vous pouvez changer ce comportement comme montré dans l’exemple suivant, permettant au framework d’ignorer un point d’injection non satisfait en le marquant comme non requis (c’est-à-dire en définissant l’attribut `required` de `@Autowired` à `false`):

```java
public class SimpleMovieLister {

	private MovieFinder movieFinder;

	@Autowired(required = false)
	public void setMovieFinder(MovieFinder movieFinder) {
		this.movieFinder = movieFinder;
	}

	// ...
}
```

```kotlin
class SimpleMovieLister {

	@Autowired(required = false)
	var movieFinder: MovieFinder? = null

	// ...
}
```

> **NOTE**
>
> Une méthode non requise ne sera pas appelée du tout si sa dépendance (ou l’une de ses dépendances, dans le cas de plusieurs arguments) n’est pas disponible. Un champ non requis ne sera pas rempli dans de tels cas, laissant la valeur par défaut en place.
>
> En d’autres termes, appliquer l’attribut `required` à `false` indique que la propriété correspondante est optionnelle pour l’autocâblage, et que la propriété sera ignorée si elle ne peut pas être autocâblée. Cela permet aux propriétés d’être assignées à leur valeur par défaut, qui peuvent être optionnellement réécrites par l’injection de dépendances.

Un constructeur injecté et les arguments d’une méthode factory représentent un cas particulier, car l’attribut `required` dans `@Autowired` a une signification quelque peu différente en raison de l’algorithme de résolution des constructeurs de _Spring_, qui peut potentiellement avoir à gérer plusieurs constructeurs. Les constructeurs et les arguments des méthodes factory sont en pratique requis par défaut, mais avec quelques règles spéciales dans le scénario d’un constructeur unique, comme la résolution des points d’injection de plusieurs éléments (tableaux, collections, maps) se résolvant en instances vides si aucun **bean** correspondant n’est disponible. Cela permet un patron de conception courant où toutes les dépendances peuvent être déclarées dans un unique constructeur à multiples arguments — par exemple, déclaré comme un unique constructeur public sans annotation `@Autowired`.

> **NOTE**
>
> Un seul constructeur pour une **class** de **bean** donnée peut déclarer `@Autowired` avec l’attribut `required` appliqué à `true`, indiquant qu’il s’agit du constructeur à utiliser lors de la création du **bean** _Spring_. Par conséquent, si l’attribut `required` est laissé à sa valeur par défaut (`true`), un seul constructeur peut être annoté avec `@Autowired`. Si plusieurs constructeurs déclarent cette annotation, ils doivent tous définir `required=false` afin d’être considérés comme candidats à l’autocâblage (de manière analogue à `autowire=constructor` en XML). Le constructeur avec le plus grand nombre de dépendances pouvant être satisfaites par les **beans** présents dans le conteneur _Spring_ sera choisi. Si aucun candidat ne peut être satisfait, alors un constructeur primaire ou par défaut (s’il existe) sera utilisé. De manière similaire, si une **class** déclare plusieurs constructeurs mais qu’aucun n’est annoté avec `@Autowired`, alors un constructeur primaire ou par défaut (s’il existe) sera utilisé. Si une **class** ne déclare qu’un seul constructeur dès le départ, celui-ci sera toujours utilisé, même s’il n’est pas annoté. Notez qu’un constructeur annoté n’a pas besoin d’être public.

De manière alternative, vous pouvez exprimer la nature non requise d’une dépendance particulière à l’aide de l’interface `java.util.Optional`, comme le montre l’exemple suivant:

```java
public class SimpleMovieLister {

	@Autowired
	public void setMovieFinder(Optional<MovieFinder> movieFinder) {
		...
	}
}
```

Vous pouvez également utiliser l’annotation `@Nullable` au niveau d’un paramètre (quel que soit le package — par exemple `org.jspecify.annotation.Nullable` de JSpecify) ou simplement tirer parti du support de Kotlin:

```java
public class SimpleMovieLister {

	@Autowired
	public void setMovieFinder(@Nullable MovieFinder movieFinder) {
		...
	}
}
```

```kotlin
class SimpleMovieLister {

	@Autowired
	var movieFinder: MovieFinder? = null

	// ...
}
```

Vous pouvez aussi utiliser `@Autowired` pour des interfaces connues comme dépendances résolvables: `BeanFactory`, `ApplicationContext`, `Environment`, `ResourceLoader`, `ApplicationEventPublisher` et `MessageSource`. Ces interfaces, ainsi que leurs sous-interfaces telles que `ConfigurableApplicationContext` ou `ResourcePatternResolver`, sont automatiquement résolues sans configuration particulière. L’exemple suivant montre l’autocâblage d’un objet `ApplicationContext`:

```java
public class MovieRecommender {

	@Autowired
	private ApplicationContext context;

	public MovieRecommender() {
	}

	// ...
}
```

```kotlin
class MovieRecommender {

	@Autowired
	lateinit var context: ApplicationContext

	// ...
}
```

> **NOTE**
>
> Les annotations `@Autowired`, `@Inject`, `@Value` et `@Resource` sont gérées par _Spring_ via des implémentations de `BeanPostProcessor`. Cela signifie que vous ne pouvez pas appliquer ces annotations dans vos propres types `BeanPostProcessor` ou `BeanFactoryPostProcessor`.
>
> Ces types doivent être câblés explicitement en utilisant du XML ou une méthode `@Bean` de _Spring_.