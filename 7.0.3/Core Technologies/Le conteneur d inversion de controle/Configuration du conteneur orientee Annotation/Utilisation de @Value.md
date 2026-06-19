{
page:25,
from:"https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/resource.html",
update:"2026-06-03",
by:["Arthur Leroux"],
}

# _*Utilisation de @Value*_

`@Value` est typiquement utilisé pour injecter des propriétés externalisées.

```java
@Component
public class MovieRecommender {

	private final String catalog;

	public MovieRecommender(@Value("${catalog.name}") String catalog) {
		this.catalog = catalog;
	}
}
```

```kotlin
@Component
class MovieRecommender(@Value("\${catalog.name}") private val catalog: String)
```

Avec la configuration suivante :

```java
@Configuration
@PropertySource("classpath:application.properties")
public class AppConfig { }
```

```kotlin
@Configuration
@PropertySource("classpath:application.properties")
class AppConfig
```

Et le fichier de configuration `application.properties` :

```text
catalog.name=MovieCatalog
```

Dans ce cas, le paramètre `catalog` et le champ seront équivalant à la valeur `MovieCatalog`.

Spring fournit par défaut un résolveur embarqué de valeurs tolérant. Il essaiera de résoudre la valeur de la propriété et, s'il ne peut pas la résoudre, le nom de la propriété (par exemple `${catalog.name}`) sera injecté en tant que valeur. Si vous souhaitez conserver un contrôle strict sur l'absence d'une valeur, vous devriez déclarer un **bean** `PropertySourcesPlaceholderConfigurer`, comme le montre l'exemple suivant :

```java
@Configuration
public class AppConfig {

	@Bean
	public static PropertySourcesPlaceholderConfigurer propertyPlaceholderConfigurer() {
		return new PropertySourcesPlaceholderConfigurer();
	}
}
```

```java
@Configuration
class AppConfig {

	@Bean
	fun propertyPlaceholderConfigurer() = PropertySourcesPlaceholderConfigurer()
}
```

> !NOTE
> Lors de la configuration d'un `PropertySourcesPlaceholderConfigurer` avec une configuration Java-based, la méthode `@Bean` doit être `static`.

L'utilisation de la configuration ci-dessus garantit que l'initialisation de _Spring_ échoue si un placeholder `${}` ne peut pas être résolu. Il est également possible d'utiliser des méthodes telles que `setPlaceholderPrefix()`, `setPlaceholderSuffix()`, `setValueSeparator()` ou `setEscapeCharacter()` pour personnaliser la syntaxe du placeholder. En complément, le caractère d'échappement peut être modifié ou désactivé globalement en appliquant la propriété `spring.placeholder.escapeCharacter.default` via la JVM (ou via le mécanisme [`SpringProperties`]()).

> !NOTE
> _Spring Boot_ configure par défaut un **bean** `PropertySourcesPlaceholderConfigurer` qui récupère les propriétés depuis les fichiers `application.properties` et `application.yml`.

Le support de conversion disponible dans _Spring_ permet de gérer automatiquement les conversions de types simples (par exemple de `Integer` vers `int`). Plusieurs valeurs séparées par des virgules peuvent également être automatiquement converties en tableau de `String` sans effort supplémentaire.

Il est possible de fournir une valeur par défaut comme suit :

```java
@Component
public class MovieRecommender {

	private final String catalog;

	public MovieRecommender(@Value("${catalog.name:defaultCatalog}") String catalog) {
		this.catalog = catalog;
	}
}
```

```kotlin
@Component
class MovieRecommender(@Value("\${catalog.name:defaultCatalog}") private val catalog: String)
```

Un **bean** `BeanPostProcessor` utilise un **bean** `ConversionService` en arrière-plan pour gérer le processus de conversion de la valeur `String` présente dans `@Value` vers le type cible. Si vous souhaitez fournir votre propre mécanisme de conversion de types, vous pouvez déclarer votre propre **bean** `ConversionService`, comme le montre l'exemple suivant :

```java
@Configuration
public class AppConfig {

	@Bean
	public ConversionService conversionService() {
		DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService();
		conversionService.addConverter(new MyCustomConverter());
		return conversionService;
	}
}
```

```kotlin
@Configuration
class AppConfig {

	@Bean
	fun conversionService(): ConversionService {
		return DefaultFormattingConversionService().apply {
			addConverter(MyCustomConverter())
		}
	}
}
"

Lorsque `@Value` contient une [expression `SpEL`](), la valeur est calculée dynamiquement à l'exécution, comme le montre l'exemple suivant :

```java
@Component
public class MovieRecommender {

	private final String catalog;

	public MovieRecommender(@Value("#{systemProperties['user.catalog'] + 'Catalog'}") String catalog) {
		this.catalog = catalog;
	}
}
```

```kotlin
@Component
class MovieRecommender(
	@Value("#{systemProperties['user.catalog'] + 'Catalog'}") private val catalog: String
)
"

SpEL permet également d'utiliser des structures de données plus complexes :

```java
@Component
public class MovieRecommender {

	private final Map<String, Integer> countOfMoviesPerCatalog;

	public MovieRecommender(
			@Value("#{{'Thriller': 100, 'Comedy': 300}}") Map<String, Integer> countOfMoviesPerCatalog) {
		this.countOfMoviesPerCatalog = countOfMoviesPerCatalog;
	}
}
```

```kotlin
@Component
class MovieRecommender(
	@Value("#{{'Thriller': 100, 'Comedy': 300}}")
	private val countOfMoviesPerCatalog: Map<String, Int>
)
```