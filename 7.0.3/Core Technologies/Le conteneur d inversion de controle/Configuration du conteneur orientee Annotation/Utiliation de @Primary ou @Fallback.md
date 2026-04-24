{
    page:19,
    from:"https://docs.spring.io/spring-framework/reference/core/beans/annotation-config.html",
    update:"2026-04-18",
    by:["Arthur Leroux"],
}

# _**Autocâblage avec @Primary ou @Fallback**_

Parce que l’autocâblage par type peut conduire à de multiples candidats, il est souvent nécessaire d’avoir plus de contrôle sur le processus de sélection. Une manière d’accomplir cela avec _Spring_ est d’utiliser l’annotation `@Primary`. `@Primary` indique qu’un **bean** en particulier doit être préféré lorsque plusieurs **beans** sont candidats à l’autocâblage pour une dépendance unique. S’il existe exactement un seul **bean** marqué comme *primary* parmi les candidats, il devient alors la valeur injectée.

Considérez la configuration suivante qui définit `firstMovieCatalog` comme le **bean principal** de type `MovieCatalog` :

```java
@Configuration
public class MovieConfiguration {

	@Bean
	@Primary
	public MovieCatalog firstMovieCatalog() { ... }

	@Bean
	public MovieCatalog secondMovieCatalog() { ... }

	// ...
}
```

```kotlin
@Configuration
class MovieConfiguration {

	@Bean
	@Primary
	fun firstMovieCatalog(): MovieCatalog { ... }

	@Bean
	fun secondMovieCatalog(): MovieCatalog { ... }

	// ...
}
```

Depuis la version 6.2, il existe également une annotation `@Fallback` permettant de marquer certains **beans** comme secondaires par rapport aux autres lors de l’injection. S’il ne reste qu’un seul **bean** non marqué comme *fallback*, celui-ci est effectivement considéré comme *primary* :

```java
@Configuration
public class MovieConfiguration {

	@Bean
	public MovieCatalog firstMovieCatalog() { ... }

	@Bean
	@Fallback
	public MovieCatalog secondMovieCatalog() { ... }

	// ...
}
```

```kotlin
@Configuration
class MovieConfiguration {

	@Bean
	fun firstMovieCatalog(): MovieCatalog { ... }

	@Bean
	@Fallback
	fun secondMovieCatalog(): MovieCatalog { ... }

	// ...
}
```

Avec les deux variantes de configuration précédentes, le `MovieRecommender` suivant est autocâblé avec `firstMovieCatalog` :

```java
public class MovieRecommender {

	@Autowired
	private MovieCatalog movieCatalog;

	// ...
}
```

```kotlin
public class MovieRecommender {

	@Autowired
	private MovieCatalog movieCatalog;

	// ...
}
```

La configuration XML correspondante des définitions de **beans** est la suivante :

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

	<bean class="example.SimpleMovieCatalog" primary="true">
		<!-- inject any dependencies required by this bean -->
	</bean>

	<bean class="example.SimpleMovieCatalog">
		<!-- inject any dependencies required by this bean -->
	</bean>

	<bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```