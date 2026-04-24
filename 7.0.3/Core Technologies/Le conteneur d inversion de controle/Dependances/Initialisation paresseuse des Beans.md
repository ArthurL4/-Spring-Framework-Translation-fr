{
    page:11,
    from:"https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-lazy-init.html",
    update:"2026-03-02",
    by:["Arthur Leroux"],
}

# _**Initialisation paresseuse des Beans**_

Par défaut, les implémentations de `ApplicationContext` créent et configurent au plus tôt tous les **beans** [singletons]() au moment du processus d'initialisation. Généralement, cette pré-instanciation est préférable, parce que les erreurs dans la configuration ou dans l'environnement sont découvertes immédiatement, contrairement à des heures ou des jours plus tard. Quand ce comportement n'est pas souhaité, vous pouvez empêcher la pré-instanciation d'un **bean singleton** en marquant la définition du **bean** comme étant initialisée de façon paresseuse. Une initialisation paresseuse d'un **bean** indique au conteneur d'inversion de contrôle de créer l'instance du **bean** lorsqu'il est requis pour la première fois, plutôt qu'au démarrage.

Ce comportement est contrôlé par l'annotation `@Lazy` ou bien par l'attribut `lazy-init` en XML dans l'élément `<bean/>`, comme le montre l'exemple suivant :

```java
@Bean
@Lazy
ExpensiveToCreateBean lazy() {
	return new ExpensiveToCreateBean();
}

@Bean
AnotherBean notLazy() {
	return new AnotherBean();
}
```

```kotlin
@Bean
@Lazy
fun lazy(): ExpensiveToCreateBean {
	return ExpensiveToCreateBean()
}

@Bean
fun notLazy(): AnotherBean {
	return AnotherBean()
}
```

```xml
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>

<bean name="notLazy" class="com.something.AnotherBean"/>
```

Lorsque la configuration précédente est utilisée par un `ApplicationContext`, le **bean** paresseux n'est pas pré-instancié quand l'`ApplicationContext` démarre, alors que ceux qui ne sont pas paresseux sont pré-instanciés.

Cependant, lorsqu'un **bean** paresseux est une dépendance d'un **bean** singleton qui n'est pas paresseux, l'`ApplicationContext` crée le **bean** paresseux au démarrage, parce qu'il doit satisfaire la dépendance du singleton. Le **bean** initialisé paresseusement est injectée dans les **beans** singleton partout ailleurs où ils ne sont pas initialisés paresseusement.

Vous pouvez également contrôler l'initialisation paresseuse d'un ensemble de **beans** en utilisant l'annotation `@Lazy` sur votre **classe** annotée `@Configuration` ou bien en XML en utilisant l'attribut `default-lazy-init` dans l'élément `<beans/>`, comme le montre l'exemple suivant :

```java
@Configuration
@Lazy
public class LazyConfiguration {
	// No bean will be pre-instantiated...
}
```

```kotlin
@Configuration
@Lazy
class LazyConfiguration {
	// No bean will be pre-instantiated...
}
```

```xml
<beans default-lazy-init="true">

	<!-- No bean will be pre-instantiated... -->
</beans>
```