{
    page:4,
    from:"https://docs.spring.io/spring-framework/reference/core/beans/introduction.html",
    update:"2026-02-06",
    by:["Arthur Leroux"],
}

# __*Introduction au conteneur d'inversion de contrôle et aux Beans Spring*__

Ce chapitre couvre l'implémentation de l'inversion de contrôle (IoC) mise en œuvre par le framework _Spring_.  
**Dependency Injection** (DI) est une forme particulière de l'IoC, par laquelle les objets définissent leurs dépendances (c'est-à-dire les autres objets avec lesquels ils travaillent) uniquement à travers les arguments du constructeur, les arguments d'une méthode **factory**, ou bien par le biais de l'instanciation des propriétés sur les instances des objets après que ces derniers aient été construits ou retournés depuis une méthode **factory**. Le conteneur d'inversion de contrôle injecte par la suite ces dépendances lorsque le **bean** est créé. Ce processus est fondamentalement l'inverse (d'où le nom, inversion de contrôle) d'un **bean** contrôlant lui-même l'instanciation de ses dépendances en utilisant directement la construction de classes ou un mécanisme tel que le **Service Locator** pattern.

Les packages `org.springframework.beans` et `org.springframework.context` sont la base du conteneur d'inversion de contrôle du framework _Spring_. L'interface [BeanFactory](https://docs.spring.io/spring-framework/docs/7.0.3/javadoc-api/org/springframework/beans/factory/BeanFactory.html) fournit un mécanisme de configuration avancée capable de gérer n'importe quel type d'objet. L'interface [ApplicationContext](https://docs.spring.io/spring-framework/docs/7.0.3/javadoc-api/org/springframework/context/ApplicationContext.html) est une sous-interface de `BeanFactory`.  
Elle ajoute :  
* Une intégration simplifiée avec les fonctionnalités de Spring AOP  
* Un gestionnaire de **MessageResource** (dans le cas de l'**internationalisation**)  
* La publication des événements  
* Une couche applicative plus spécifique, telle que `WebApplicationContext` dans le cas d'applications orientées web  

En résumé, l'interface `BeanFactory` fournit la configuration du framework et les fonctionnalités essentielles, tandis que l'interface `ApplicationContext` rajoute des fonctionnalités plus spécifiques au monde de l'entreprise.  
L'interface `ApplicationContext` est un surensemble de l'interface `BeanFactory` et est utilisée exclusivement dans ce chapitre concernant la description du conteneur d'inversion de contrôle de _Spring_.  
Pour plus d'informations concernant l'usage de l'interface `BeanFactory` au lieu de l'interface `ApplicationContext`, voir la section couvrant le [BeanFactory API](https://docs.spring.io/spring-framework/reference/core/beans/beanfactory.html).

Avec _Spring_, les objets formant l'ossature de votre application et qui sont gérés par le conteneur d'inversion de contrôle de _Spring_ sont appelés des **beans**. Un **bean** est un objet qui est instancié, assemblé et géré par le conteneur d'inversion de contrôle de _Spring_. Sinon, un **bean** n'est qu'un objet parmi ceux de votre application. Les **beans**, ainsi que leurs dépendances, sont reflétés sous forme de métadonnées de configuration utilisées par le conteneur.


