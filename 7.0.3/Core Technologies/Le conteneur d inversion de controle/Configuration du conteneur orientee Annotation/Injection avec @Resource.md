{
page:24,
from:"https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/resource.html",
update:"2026-06-03",
by:["Arthur Leroux"],
}

# _**Injection avec @Resource**_

_Spring_ supporte également l’injection en utilisant l’annotation `@Resource` du JSR-250 (`jakarta.annotation.Resource`) sur les champs ou les propriétés du **bean** via les méthodes **setter**. C’est un pattern courant dans Jakarta EE : par exemple, dans les **beans** gérés par JSF et les endpoints JAX-WS. _Spring_ supporte également ce pattern pour les objets gérés par _Spring_.

L’annotation `@Resource` prend un attribut `name`. Par défaut, _Spring_ interprète cette valeur comme le nom d’un **bean** à injecter. En d’autres termes, cela suit une sémantique d'injection par nom, comme le montre l’exemple suivant :

```
java id="yk7q8a"
public class SimpleMovieLister {

	private MovieFinder movieFinder;

	@Resource(name="myMovieFinder")
	public void setMovieFinder(MovieFinder movieFinder) {
		this.movieFinder = movieFinder;
	}
}
```

```
kotlin id="p3m8xq"
class SimpleMovieLister {

	@Resource(name="myMovieFinder")
	private lateinit var movieFinder: MovieFinder
}
```

Si aucun nom n’est explicitement spécifié, le nom par défaut est dérivé du nom du champ ou de la méthode **setter**. Dans le cas d’un champ, cela correspond au nom du champ. Dans le cas d’une méthode **setter**, cela correspond au nom de la propriété du **bean**. L’exemple suivant montre que le **bean** nommé `movieFinder` est injecté dans sa méthode **setter** :

```
java id="m1d8rv"
public class SimpleMovieLister {

	private MovieFinder movieFinder;

	@Resource
	public void setMovieFinder(MovieFinder movieFinder) {
		this.movieFinder = movieFinder;
	}
}
```
```
kotlin id="q9x2ld"
class SimpleMovieLister {

	@set:Resource
	private lateinit var movieFinder: MovieFinder
}
```

> !NOTE  
> Le nom fourni avec l’annotation est résolu comme le nom d’un **bean** par l’`ApplicationContext`, dont le `CommonAnnotationBeanPostProcessor` est responsable.  
> Les noms peuvent être résolus via JNDI si vous configurez explicitement `SimpleJndiBeanFactory` de _Spring_. Cependant, nous recommandons de s’appuyer sur le comportement par défaut et d’utiliser les capacités de recherche JNDI de _Spring_ afin de préserver le niveau d’indirection.

Dans le cas particulier d'utilisation de `@Resource` sans nom spécifié, et de façon similaire à `@Autowired`, `@Resource` recherche d’abord une correspondance par nom, puis bascule vers une correspondance par type, et résout les dépendances bien connues : les interfaces `BeanFactory`, `ApplicationContext`, `ResourceLoader`, `ApplicationEventPublisher` et `MessageSource`.

Ainsi, dans l’exemple suivant, le champ `customerPreferenceDao` recherche d’abord un **bean** nommé `customerPreferenceDao`, puis bascule sur une correspondance par type premier pour le type `CustomerPreferenceDao` :

```
java id="t7n5pq"
public class MovieRecommender {

	@Resource
	private CustomerPreferenceDao customerPreferenceDao;

	@Resource
	private ApplicationContext context;

	public MovieRecommender() {
	}

	// ...
}
```
```
kotlin id="v8k2zn"
class MovieRecommender {

	@Resource
	private lateinit var customerPreferenceDao: CustomerPreferenceDao

	@Resource
	private lateinit var context: ApplicationContext

	// ...
}
```