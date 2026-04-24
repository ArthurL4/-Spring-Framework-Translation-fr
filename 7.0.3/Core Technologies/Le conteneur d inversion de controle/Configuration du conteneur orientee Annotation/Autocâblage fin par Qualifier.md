{
    page:20,
    from:"https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired-qualifiers.html",
    update:"2026-04-19",
    by:["Arthur Leroux"],
}

# _**Ajustement fin de l’autocâblage avec @Qualifier**_

`@Primary` et `@Fallback` sont des moyens efficaces d’utiliser l’autocâblage par type lorsqu’il existe plusieurs instances et qu’un candidat *primary* (ou non-*fallback*) peut être déterminé.

Lorsque vous avez besoin de davantage de contrôle sur le processus de sélection, vous pouvez utiliser l’annotation `@Qualifier` de _Spring_. Vous pouvez associer des valeurs de **qualifier** à des arguments spécifiques, réduisant ainsi l’ensemble des types correspondants afin qu’un **bean** précis soit sélectionné pour chaque argument. Dans le cas le plus simple, il peut s’agir d’une valeur descriptive, comme illustré dans l’exemple suivant :

```java
public class MovieRecommender {

	@Autowired
	@Qualifier("main")
	private MovieCatalog movieCatalog;

	// ...
}
```

```kotlin
class MovieRecommender {

	@Autowired
	@Qualifier("main")
	private lateinit var movieCatalog: MovieCatalog

	// ...
}
```

Vous pouvez également spécifier l’annotation `@Qualifier` sur des arguments individuels du constructeur ou des paramètres de méthode, comme le montre l’exemple suivant :

```java
public class MovieRecommender {

	private final MovieCatalog movieCatalog;

	private final CustomerPreferenceDao customerPreferenceDao;

	@Autowired
	public void prepare(@Qualifier("main") MovieCatalog movieCatalog,
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
	fun prepare(@Qualifier("main") movieCatalog: MovieCatalog,
				customerPreferenceDao: CustomerPreferenceDao) {
		this.movieCatalog = movieCatalog
		this.customerPreferenceDao = customerPreferenceDao
	}

	// ...
}
```

L’exemple suivant montre les définitions de **beans** correspondantes :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		https://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context
		https://www.springframework.org/schema/context/spring-context.xsd">

	<context:annotation-config/>

	<bean class="example.SimpleMovieCatalog">
		<qualifier value="main"/>

		<!-- inject any dependencies required by this bean -->
	</bean>

	<bean class="example.SimpleMovieCatalog">
		<qualifier value="action"/>

		<!-- inject any dependencies required by this bean -->
	</bean>

	<bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

1. Le **bean** avec la valeur `main` du **qualifier** est câblé avec l’argument du constructeur qualifié avec la même valeur.  
2. Le **bean** avec la valeur `action` du **qualifier** est câblé avec l’argument du constructeur qualifié avec la même valeur.

Pour une correspondance de repli, le nom du **bean** est considéré comme la valeur par défaut du **qualifier**. Ainsi, vous pouvez définir le **bean** avec un `id` de `main` au lieu d’un élément **qualifier** imbriqué, ce qui conduit à la même correspondance. Cependant, bien que vous puissiez utiliser cette convention pour référencer des **beans** spécifiques par leur nom, `@Autowired` reste fondamentalement une injection orientée par type avec une sémantique optionnelle de **qualifier**. Cela signifie que les valeurs de **qualifier**, même en utilisant le nom du **bean** comme repli, conservent une fonction de filtrage au sein des types correspondants. Elles n’expriment pas sémantiquement une référence à un unique **bean** identifié par son `id`. De bonnes valeurs de **qualifier** sont `main`, `EMEA` ou `persistent`, exprimant des caractéristiques d’un composant spécifique indépendantes du `id` du **bean**, lequel peut être généré automatiquement dans le cas d’une définition anonyme, comme dans l’exemple précédent.

Les **qualifiers** s’appliquent également aux collections typées, comme évoqué précédemment — par exemple, à `Set<MovieCatalog>`. Dans ce cas, tous les **beans** correspondants, selon les **qualifiers** déclarés, sont injectés dans la collection. Cela implique que les **qualifiers** n’ont pas besoin d’être uniques. Ils constituent plutôt un critère de filtrage. Par exemple, vous pouvez définir plusieurs **beans** `MovieCatalog` avec la même valeur de **qualifier** `"action"` ; tous seront injectés dans un `Set<MovieCatalog>` annoté avec `@Qualifier("action")`.

> **TIP**
>
> L’utilisation des noms de **beans** comme valeurs de **qualifier** dans la correspondance des candidats par type ne nécessite pas d’annotation `@Qualifier` au point d’injection. S’il n’y a pas d’autre indicateur de résolution (tel qu’un **qualifier**, un marqueur *primary* ou *fallback*), dans une situation de dépendance non unique, _Spring_ fait correspondre le nom du point d’injection (c’est-à-dire le nom du champ ou du paramètre) avec les noms des **beans** cibles et sélectionne le candidat portant le même nom (que ce soit le nom du **bean** ou un alias associé).
>
> Depuis la version 6.1, cela nécessite que le flag Java `-parameters` du compilateur soit présent. Depuis la version 6.2, le conteneur applique une résolution rapide par correspondance directe du nom du **bean**, en contournant l’algorithme de correspondance par type lorsque le nom du paramètre correspond au nom du **bean** et qu’aucune condition de type, **qualifier**, *primary* ou *fallback* ne redéfinit la correspondance. Il est donc recommandé que les noms de vos paramètres correspondent aux noms des **beans** cibles.


Comme alternative à l’injection par nom, considérez l’annotation `JSR-250` `@Resource`, qui est sémantiquement définie pour identifier un composant cible spécifique par son nom unique, le type déclaré n’étant pas pertinent pour le processus de correspondance. `@Autowired`, en revanche, a une sémantique différente : après la sélection des **beans** candidats par type, la valeur `String` du **qualifier** est prise en compte uniquement parmi ces candidats (par exemple, faire correspondre un **qualifier** `account` avec des **beans** marqués du même **qualifier**).

Pour les **beans** qui sont eux-mêmes définis comme une collection, une `Map` ou un tableau, `@Resource` constitue une solution adaptée, permettant de référencer une collection ou un tableau spécifique de **beans** par un nom unique. Cela dit, vous pouvez également faire correspondre des collections, des `Map` et des tableaux typés via l’algorithme de correspondance par type de `@Autowired` de _Spring_, tant que l’information de type des éléments est conservée dans la signature de retour de la méthode `@Bean` ou via la hiérarchie d’héritage de la collection. Dans ce cas, vous pouvez utiliser les valeurs des **qualifiers** pour filtrer parmi ces collections typées, comme expliqué dans le paragraphe précédent.

`@Autowired` prend également en charge les références réflexives pour l’injection (c’est-à-dire les références vers le **bean** actuellement injecté). Voir [Self Injection]().

`@Autowired` s’applique aux champs, aux constructeurs et aux méthodes avec plusieurs arguments, permettant un filtrage via des annotations **qualifier** au niveau des paramètres. En revanche, `@Resource` est uniquement pris en charge pour les champs et les propriétés **setter** (méthodes à un seul argument). Par conséquent, il est recommandé d’utiliser des **qualifiers** si votre cible d’injection est un constructeur ou une méthode à plusieurs paramètres.

Vous pouvez également créer votre propre annotation **qualifier** personnalisée. Pour ce faire, définissez une annotation et annotez-la avec `@Qualifier`, comme illustré dans l’exemple suivant :

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Genre {

	String value();
}
```

```kotlin
@Target(AnnotationTarget.FIELD, AnnotationTarget.VALUE_PARAMETER)
@Retention(AnnotationRetention.RUNTIME)
@Qualifier
annotation class Genre(val value: String)
```

Ensuite, vous pouvez utiliser ce **qualifier** personnalisé sur les champs et paramètres autocâblés, comme illustré dans l’exemple suivant :

```java
public class MovieRecommender {

	@Autowired
	@Genre("Action")
	private MovieCatalog actionCatalog;

	private MovieCatalog comedyCatalog;

	@Autowired
	public void setComedyCatalog(@Genre("Comedy") MovieCatalog comedyCatalog) {
		this.comedyCatalog = comedyCatalog;
	}

	// ...
}
```

```kotlin
class MovieRecommender {

	@Autowired
	@Genre("Action")
	private lateinit var actionCatalog: MovieCatalog

	private lateinit var comedyCatalog: MovieCatalog

	@Autowired
	fun setComedyCatalog(@Genre("Comedy") comedyCatalog: MovieCatalog) {
		this.comedyCatalog = comedyCatalog
	}

	// ...
}
```

Par la suite, vous pouvez fournir les métadonnées correspondantes dans les définitions de **beans** candidats. Vous pouvez ajouter des éléments `<qualifier/>` comme sous-éléments de `<bean/>`, puis spécifier les attributs `type` et `value` afin de correspondre à vos annotations **qualifier** personnalisées. Le type est recherché à partir du nom pleinement qualifié de la **classe** de l’annotation. Alternativement, pour un usage simplifié, s’il n’existe aucun risque de conflit de nom, vous pouvez utiliser le nom court de la **classe**. L’exemple suivant illustre les deux approches :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		https://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context
		https://www.springframework.org/schema/context/spring-context.xsd">

	<context:annotation-config/>

	<bean class="example.SimpleMovieCatalog">
		<qualifier type="Genre" value="Action"/>
		<!-- inject any dependencies required by this bean -->
	</bean>

	<bean class="example.SimpleMovieCatalog">
		<qualifier type="example.Genre" value="Comedy"/>
		<!-- inject any dependencies required by this bean -->
	</bean>

	<bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

Dans [Classpath Scanning and Managed Components](), vous pouvez voir une alternative basée sur les annotations pour fournir les métadonnées de **qualifier** au lieu du XML. Plus précisément, voir [Providing Qualifier Metadata with Annotations]().

Dans certains cas, utiliser une annotation sans valeur peut suffire. Cela peut être utile lorsque l’annotation sert un objectif plus générique et peut être appliquée à plusieurs types de dépendances. Par exemple, vous pouvez fournir un catalogue hors ligne qui peut être utilisé lorsqu’aucune connexion Internet n’est disponible. Tout d’abord, définissez une annotation simple, comme illustré ci-dessous :

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Offline {
}
```

```kotlin
@Target(AnnotationTarget.FIELD, AnnotationTarget.VALUE_PARAMETER)
@Retention(AnnotationRetention.RUNTIME)
@Qualifier
annotation class Offline
```

Ensuite, ajoutez l’annotation au champ ou à la propriété à autocâbler, comme montré dans l’exemple suivant :

```java
public class MovieRecommender {

	@Autowired
	@Offline
	private MovieCatalog offlineCatalog;

	// ...
}
```

```kotlin
class MovieRecommender {

	@Autowired
	@Offline
	private lateinit var offlineCatalog: MovieCatalog

	// ...
}
```

1. Cette ligne ajoute l’annotation `@Offline`.

La définition du **bean** nécessite alors uniquement un **qualifier** `type`, comme illustré dans l’exemple suivant :

```xml
<bean class="example.SimpleMovieCatalog">
	<qualifier type="Offline"/>
	<!-- inject any dependencies required by this bean -->
</bean>
```

1. Cet élément spécifie le **qualifier**.

Vous pouvez également définir des annotations **qualifier** personnalisées qui acceptent des attributs nommés en plus ou à la place d’un simple attribut `value`. Si plusieurs attributs sont spécifiés sur un champ ou un paramètre à autocâbler, une définition de **bean** doit correspondre à l’ensemble de ces attributs pour être considérée comme un candidat valide. Par exemple, considérez la définition d’annotation suivante :

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface MovieQualifier {

	String genre();

	Format format();
}
```

```kotlin
@Target(AnnotationTarget.FIELD, AnnotationTarget.VALUE_PARAMETER)
@Retention(AnnotationRetention.RUNTIME)
@Qualifier
annotation class MovieQualifier(val genre: String, val format: Format)
```

Dans ce cas, `Format` est une **enum**, définie comme suit :

```java
public enum Format {
	VHS, DVD, BLURAY
}
```

```kotlin
enum class Format {
	VHS, DVD, BLURAY
}
```

Les champs à autocâbler sont annotés avec le **qualifier** personnalisé et incluent les valeurs pour les deux attributs `genre` et `format`, comme illustré dans l’exemple suivant :

```java
public class MovieRecommender {

	@Autowired
	@MovieQualifier(format=Format.VHS, genre="Action")
	private MovieCatalog actionVhsCatalog;

	@Autowired
	@MovieQualifier(format=Format.VHS, genre="Comedy")
	private MovieCatalog comedyVhsCatalog;

	@Autowired
	@MovieQualifier(format=Format.DVD, genre="Action")
	private MovieCatalog actionDvdCatalog;

	@Autowired
	@MovieQualifier(format=Format.BLURAY, genre="Comedy")
	private MovieCatalog comedyBluRayCatalog;

	// ...
}
```

```kotlin
class MovieRecommender {

	@Autowired
	@MovieQualifier(format = Format.VHS, genre = "Action")
	private lateinit var actionVhsCatalog: MovieCatalog

	@Autowired
	@MovieQualifier(format = Format.VHS, genre = "Comedy")
	private lateinit var comedyVhsCatalog: MovieCatalog

	@Autowired
	@MovieQualifier(format = Format.DVD, genre = "Action")
	private lateinit var actionDvdCatalog: MovieCatalog

	@Autowired
	@MovieQualifier(format = Format.BLURAY, genre = "Comedy")
	private lateinit var comedyBluRayCatalog: MovieCatalog

	// ...
}
```

Enfin, les définitions de **beans** doivent contenir les valeurs correspondantes du **qualifier**. L’exemple suivant montre également que vous pouvez utiliser des méta-attributs (`<meta/>`) du **bean** au lieu des éléments `<qualifier/>`. Si les deux sont présents, `<qualifier/>` et ses attributs sont prioritaires. Toutefois, le mécanisme d’autocâblage peut utiliser les valeurs fournies dans les éléments `<meta/>` comme solution de repli lorsqu’aucun **qualifier** n’est défini, comme dans les deux dernières définitions de **beans** ci-dessous :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		https://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context
		https://www.springframework.org/schema/context/spring-context.xsd">

	<context:annotation-config/>

	<bean class="example.SimpleMovieCatalog">
		<qualifier type="MovieQualifier">
			<attribute key="format" value="VHS"/>
			<attribute key="genre" value="Action"/>
		</qualifier>
		<!-- inject any dependencies required by this bean -->
	</bean>

	<bean class="example.SimpleMovieCatalog">
		<qualifier type="MovieQualifier">
			<attribute key="format" value="VHS"/>
			<attribute key="genre" value="Comedy"/>
		</qualifier>
		<!-- inject any dependencies required by this bean -->
	</bean>

	<bean class="example.SimpleMovieCatalog">
		<meta key="format" value="DVD"/>
		<meta key="genre" value="Action"/>
		<!-- inject any dependencies required by this bean -->
	</bean>

	<bean class="example.SimpleMovieCatalog">
		<meta key="format" value="BLURAY"/>
		<meta key="genre" value="Comedy"/>
		<!-- inject any dependencies required by this bean -->
	</bean>

</beans>
```