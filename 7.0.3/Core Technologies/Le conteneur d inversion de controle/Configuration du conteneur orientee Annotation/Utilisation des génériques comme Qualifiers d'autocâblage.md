{
    page:21,
    from:"https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired-qualifiers.html",
    update:"2026-04-19",
    by:["Arthur Leroux"],
}


# _**Utilisation des génériques comme qualifiers**_

En plus de l’annotation `@Qualifier`, vous pouvez utiliser les types génériques de Java comme une forme implicite de qualification. Par exemple, supposons que vous ayez la configuration suivante :

```java
@Configuration
public class MyConfiguration {

	@Bean
	public StringStore stringStore() {
		return new StringStore();
	}

	@Bean
	public IntegerStore integerStore() {
		return new IntegerStore();
	}
}
```

```kotlin
@Configuration
class MyConfiguration {

	@Bean
	fun stringStore() = StringStore()

	@Bean
	fun integerStore() = IntegerStore()
}
```

Supposons que les **beans** précédents implémentent une interface générique (c’est-à-dire `Store<String>` et `Store<Integer>`). Vous pouvez alors utiliser `@Autowired` sur l’interface `Store`, le type générique servant de **qualifier**, comme illustré dans l’exemple suivant :

```java
@Autowired
private Store<String> s1; // <String> qualifier, injecte le bean stringStore

@Autowired
private Store<Integer> s2; // <Integer> qualifier, injecte le bean integerStore
```

```kotlin
@Autowired
private lateinit var s1: Store<String> // <String> qualifier, injecte le bean stringStore

@Autowired
private lateinit var s2: Store<Integer> // <Integer> qualifier, injecte le bean integerStore
```

Les **qualifiers** basés sur les génériques s’appliquent également lors de l’autocâblage des listes, des `Map` et des tableaux. L’exemple suivant montre l’autocâblage d’une `List` générique :

```java
// Injecte tous les beans Store ayant le générique <Integer>
// Les beans Store<String> ne seront pas inclus dans cette liste
@Autowired
private List<Store<Integer>> s;
```

```kotlin
// Injecte tous les beans Store ayant le générique <Integer>
// Les beans Store<String> ne seront pas inclus dans cette liste
@Autowired
private lateinit var s: List<Store<Integer>>
```