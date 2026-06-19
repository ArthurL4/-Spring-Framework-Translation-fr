{
page:26,
from:"https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/resource.html",
update:"2026-06-03",
by:["Arthur Leroux"],
}

# _*Utilisation de @PostConstruct et @PreDestroy*_

Le **bean** `CommonAnnotationBeanPostProcessor` ne reconnaît pas seulement l'annotation `@Resource`, mais également les annotations de cycle de vie JSR-250 :
`jakarta.annotation.PostConstruct` et `jakarta.annotation.PreDestroy`.

Introduit dans _Spring_ 2.5, le support de ces annotations offre une alternative aux mécanismes de callback décrits dans [callbacks d'initialisation]() et [callbacks de destruction](). À condition que le **bean** `CommonAnnotationBeanPostProcessor` soit enregistré dans l'`ApplicationContext` de _Spring_, une méthode portant l'une de ces annotations est invoquée au même point du cycle de vie que la méthode d'interface de cycle de vie _Spring_ correspondante ou qu'une méthode de callback explicitement déclarée.

Dans l'exemple suivant, le cache est prérempli durant l'initialisation puis nettoyé lors de la destruction :

"
public class CachingMovieLister {

	@PostConstruct
	public void populateMovieCache() {
		// populates the movie cache upon initialization...
	}

	@PreDestroy
	public void clearMovieCache() {
		// clears the movie cache upon destruction...
	}
}
"

"
class CachingMovieLister {

	@PostConstruct
	fun populateMovieCache() {
		// populates the movie cache upon initialization...
	}

	@PreDestroy
	fun clearMovieCache() {
		// clears the movie cache upon destruction...
	}
}
"

Pour plus de détails concernant les effets de la combinaison de plusieurs mécanismes de cycle de vie, consultez [Combining Lifecycle Mechanisms]().

> !NOTE
> Comme `@Resource`, les annotations `@PostConstruct` et `@PreDestroy` faisaient partie des bibliothèques standard de Java du JDK 6 au JDK 8. Cependant, l'intégralité du package `javax.annotation` a été séparée des modules cœur de Java dans le JDK 9 puis finalement supprimée dans le JDK 11. Depuis Jakarta EE 9, ce package se trouve désormais `jakarta.annotation`. Si nécessaire, l'artefact `jakarta.annotation-api` peut être récupéré depuis Maven Central et ajouté au classpath de l'application comme n'importe quelle autre bibliothèque.