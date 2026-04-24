{
    page:17,
    from:"https://docs.spring.io/spring-framework/reference/core/beans/annotation-config.html",
    update:"2026-04-10",
    by:["Arthur Leroux"],
}
# _**Configuration du conteneur orientée annotation**_

_Spring_ fournit un support complet pour des configurations orientées annotation, opérant sur les métadonnées dans le **class** du composant elle-même, en utilisant les annotations sur les **classes** pertinentes ou les déclarations des champs. 

Comme indiqué dans [Example: The `AutowiredAnnotationBeanPostProcessor`](../Points%20d%20Extensions%20du%20Conteneur.md), _Spring_ utilise l'interface `BeanPostProcessor` en conjonction avec les annotations pour rendre le cœur du conteneur d'inversion de contrôle conscient des annotations spécifiques.

Par exemple, l'annotation [`@Autowired`]() fournit les mêmes possibilités que celles décrites dans [Autowiring Collaborators](../Dependances/Autocâblage%20des%20collaborateurs.md), mais avec un contrôle plus fin et une applicabilité plus large. De plus, _Spring_ fournit un support pour les annotations JSR-250, telles que `@PostConstruct` et `@PreDestroy`, mais également pour les annotations JSR-330 (Injection de dépendances pour Java) contenues dans le package `jakarta.inject`, telles que `@Inject` et `@Named`. Les détails concernant ces annotations peuvent être trouvés dans la [section concernée]().

> **NOTE**
> L'injection par annotation est effectuée avant l'injection de propriétés externes. Ainsi, une configuration externe (par exemple, un fichier de propriétés ou des **beans** XML) réécrit en réalité les annotations des propriétés lorsqu'elles sont câblées à travers des approches mixtes.

Techniquement, vous pouvez enregistrer les post-processeurs comme définitions de **beans** individuelles, mais ils sont déjà enregistrés implicitement dans un `AnnotationConfigApplicationContext`.

Dans une configuration orientée XML, vous pouvez inclure la configuration du tag suivant pour permettre un mélange et une correspondance avec une configuration orientée annotation :

"xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		https://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context
		https://www.springframework.org/schema/context/spring-context.xsd">

	<context:annotation-config/>

</beans>
"

L'élément `<context:annotation-config/>` enregistre implicitement les post-processeurs suivants :

* [`ConfigurationClassPostProcessor`](https://docs.spring.io/spring-framework/docs/7.0.6/javadoc-api/org/springframework/context/annotation/ConfigurationClassPostProcessor.html)
* [`AutowiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/7.0.6/javadoc-api/org/springframework/beans/factory/annotation/AutowiredAnnotationBeanPostProcessor.html)
* [`CommonAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/7.0.6/javadoc-api/org/springframework/context/annotation/CommonAnnotationBeanPostProcessor.html)
* [`PersistenceAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/7.0.6/javadoc-api/org/springframework/orm/jpa/support/PersistenceAnnotationBeanPostProcessor.html)
* [`EventListenerMethodProcessor`](https://docs.spring.io/spring-framework/docs/7.0.6/javadoc-api/org/springframework/context/event/EventListenerMethodProcessor.html)

> **NOTE**
>
> `<context:annotation-config/>` recherche seulement les annotations sur les **beans** dans le même **application context** dans lequel il a été défini. Cela signifie que, si vous placez `<context:annotation-config/>` dans un `WebApplicationContext` pour un `DispatcherServlet`, il vérifie uniquement les **beans** `@Autowired` dans vos contrôleurs, et non dans vos services. Voir [DispatcherServlet]() pour plus d'informations.