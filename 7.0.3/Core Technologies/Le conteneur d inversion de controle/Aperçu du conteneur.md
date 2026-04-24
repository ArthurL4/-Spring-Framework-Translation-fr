{
    page:5,
    from:"https://docs.spring.io/spring-framework/reference/core/beans/basics.html",
    update:"2026-02-07",
    by:["Arthur Leroux"],
}
# __*Aperçu du conteneur*__

L'interface `org.springframework.context.ApplicationContext` représente le conteneur d'inversion de contrôle de _Spring_ et est responsable de l'instanciation, de la configuration et de l'assemblage des **beans**. Le conteneur obtient ses instructions sur les composants à instancier, à configurer et à assembler en lisant les métadonnées de configuration. Les métadonnées de configuration peuvent être représentées sous la forme de classes annotées, de classes de configuration avec des méthodes **factory**, de fichiers XML externes ou encore de scripts **Groovy**. Quel que soit le format choisi, vous pouvez construire votre application et définir des relations riches et complexes entre vos composants.

Plusieurs implémentations de l'interface `ApplicationContext` font partie intégrante du noyau de _Spring_. Dans les applications autonomes, il est courant d'en créer une instance à l'aide des interfaces `AnnotationConfigApplicationContext` ou `ClassPathXmlApplicationContext`.

Dans la plupart des scénarios applicatifs, l'utilisation explicite de code n'est pas nécessaire pour instancier une ou plusieurs instances d'un conteneur d'inversion de contrôle avec _Spring_. Par exemple, dans une application orientée web, un simple fichier XML de description, déclaré dans le fichier `web.xml`, est suffisant (voir [Instanciation pratique d'un ApplicationContext pour les applications web](A COMPLETER). Dans le cas de l'utilisation de _Spring Boot_, l'**ApplicationContext** est chargé implicitement pour vous selon les conventions paramétrées.

Le diagramme suivant met en valeur une représentation de la façon dont _Spring_ fonctionne. Les classes de votre application sont combinées avec les métadonnées de configuration de telle manière que, une fois l'`ApplicationContext` créé et initialisé, vous disposez d'une application ou d'un environnement pleinement configuré et exécutable.

![Figure 1. The Spring IoC container](../../img/figure1-springIocContainer.png)


# __*Métadonnées de configuration*__

Comme le diagramme précédent le montre, le conteneur d'inversion de contrôle de _Spring_ consomme une forme de configuration de métadonnées. La configuration des métadonnées représente la manière dont vous, en tant que développeur de solutions informatiques, indiquez à _Spring_ comment instancier, configurer et assembler les composants de votre application.

Le conteneur d'inversion de contrôle de _Spring_ lui-même est totalement découplé du format dans lequel la configuration des métadonnées est écrite. De nos jours, de nombreux développeurs choisissent le format [Java-based configuration](À COMPLÉTER) pour leurs applications _Spring_ :

* [Annotation-based configuration](À COMPLÉTER) : permet de définir les **beans** en utilisant une configuration de métadonnées directement au sein des classes de votre application.
* [Java-based configuration](À COMPLÉTER) : permet de définir les **beans** en dehors des classes de votre application en utilisant des classes de configuration **Java-based**. Pour utiliser ces fonctionnalités, voir les annotations [@Configuration](À COMPLÉTER), [@Bean](À COMPLÉTER), [@Import](À COMPLÉTER) et [@DependsOn](À COMPLÉTER).

La configuration de _Spring_ consiste en au moins une, et généralement plusieurs, définitions de **beans** dont le conteneur assure la gestion. La configuration Java utilise généralement des méthodes annotées `@Bean` à l'intérieur d'une classe de configuration `@Configuration`, chaque méthode annotée représentant alors la définition d'un **bean**.

Les définitions de ces **beans** correspondent aux objets réels qui constituent votre application. En général, vous définissez une couche d'objets de service, une couche d'objets de persistance tels que des repositories ou des **Data Access Objects** (DAO), une couche pour vos contrôleurs web, ou encore une couche d'infrastructure pour des objets tels qu'un `EntityManagerFactory` pour JPA, des files JMS, et bien d'autres. Typiquement, le conteneur n'est pas destiné à gérer chaque objet du domaine métier dans le détail ; cette responsabilité relève plutôt des repositories et de la logique métier, qui se chargent de créer et de charger les objets du domaine.

# __*XML comme configuration DSL externe*__

La configuration des métadonnées **XML-based** définit les **beans** à l'intérieur de balises `<bean/>`, elles-mêmes regroupées au sein d'une balise parente `<beans/>`. L'exemple suivant montre la structure élémentaire d'une configuration de métadonnées **XML-based** :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		https://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="..." class="...">
		<!-- collaborators and configuration for this bean go here -->
	</bean>

	<bean id="..." class="...">
		<!-- collaborators and configuration for this bean go here -->
	</bean>

	<!-- more bean definitions go here -->

</beans>
```

1. L'attribut `id` est une chaîne de caractères identifiant de manière unique un **bean**.  
2. L'attribut `class` définit le type et la nature du **bean** et utilise le nom pleinement qualifié de la classe correspondante.

La valeur de l'attribut `id` peut être utilisée pour référencer le **bean** auprès des objets collaborateurs. La configuration XML permettant de référencer un **bean** auprès des objets collaborateurs n'est pas montrée ici. Voir [Dependencies](À COMPLÉTER) pour plus d'informations.

Pour instancier un conteneur, la localisation d'un ou de plusieurs chemins vers les fichiers XML de configuration doit être fournie au constructeur de `ClassPathXmlApplicationContext`, ce qui permet au conteneur de charger la configuration des métadonnées depuis plusieurs sources externes, telles qu'un fichier local, le `CLASSPATH` Java, et bien d'autres.

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

```kotlin
val context = ClassPathXmlApplicationContext("services.xml", "daos.xml")
```

> **NOTE**  
>
> Après en avoir appris davantage sur le conteneur d'inversion de contrôle de _Spring_, vous souhaiterez probablement en savoir plus sur l'abstraction `Resource` de _Spring_ (comme décrit dans [Resources](À COMPLÉTER)), laquelle fournit un mécanisme pratique pour lire un **InputStream** depuis une localisation définie à l'aide d'une syntaxe URI. En particulier, les chemins de `Resource` sont utilisés pour construire les **ApplicationContext**, comme expliqué dans [Application Contexts et chemins de Resource](À COMPLÉTER).

L'exemple suivant montre un fichier de configuration pour une couche d'objets de service (`services.xml`) :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		https://www.springframework.org/schema/beans/spring-beans.xsd">

	<!-- services -->

	<bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
		<property name="accountDao" ref="accountDao"/>
		<property name="itemDao" ref="itemDao"/>
		<!-- additional collaborators and configuration for this bean go here -->
	</bean>

	<!-- more bean definitions for services go here -->

</beans>
```

L'exemple suivant montre le fichier de configuration d'accès aux objets de données `daos.xml` :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		https://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="accountDao"
		class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
		<!-- additional collaborators and configuration for this bean go here -->
	</bean>

	<bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
		<!-- additional collaborators and configuration for this bean go here -->
	</bean>

	<!-- more bean definitions for data access objects go here -->

</beans>
```

Dans l'exemple précédent, la couche service consiste en une classe `PetStoreServiceImpl` et deux classes d'accès aux données de type `JpaAccountDao` et `JpaItemDao` (basées sur le standard du modèle **Object-Relational Mapping** de JPA). L'élément `property name` fait référence au nom de la propriété JavaBean, et l'élément `ref` exprime la dépendance entre les objets collaborateurs. Pour plus de détails sur la configuration des dépendances d'un objet, voir [Dependencies](À COMPLÉTER).

# __*Assembler une configuration des métadonnées basée XML*__

Il peut être utile d'utiliser le constructeur de la classe `ClassPathXmlApplicationContext` afin de charger les définitions des **beans** depuis des fichiers XML. Ce constructeur prend plusieurs localisations possibles d'une `Resource`, comme montré dans la [section précédente](#xml-comme-configuration-dsl-externe). Autrement, il est possible d'utiliser une ou plusieurs balises `<import/>` directement depuis les fichiers de définitions de **beans** pour charger d'autres définitions. L'exemple suivant montre comment procéder :

```xml
<beans>
	<import resource="services.xml"/>
	<import resource="resources/messageSource.xml"/>

	<bean id="bean1" class="..."/>
	<bean id="bean2" class="..."/>
</beans>
```

Dans l'exemple précédent, les définitions des **beans** externes sont chargées depuis les fichiers `services.xml` et `messageSource.xml`. Toutes les localisations sont relatives au fichier courant faisant l'importation ; ainsi, `services.xml` doit se trouver dans le même répertoire ou classpath que le fichier faisant l'importation, tandis que `messageSource.xml` doit être placé dans `resources`, se trouvant en dessous de la localisation du fichier importateur. Comme vous pouvez le voir, la barre oblique du début est omise. Cependant, dans la mesure où ces chemins sont relatifs, il est préférable de ne pas utiliser de barre oblique du tout. Le contenu des fichiers importés, incluant la balise parent `<beans/>`, doit être d'un format valide pour la définition d'un fichier de **beans** XML, selon le schéma de _Spring_.

> **NOTE**  
> Il est possible, mais non recommandé, de référencer ces fichiers dans un répertoire parent en utilisant un chemin relatif `"../"`.  
> Agir de la sorte crée une dépendance entre un fichier se trouvant à l'extérieur du périmètre de l’application. En particulier, ce type de référence n'est pas recommandé pour les URLs `classpath:` (par exemple, `classpath:../services.xml`), dans la mesure où la résolution du chemin au lancement va choisir la racine du classpath la plus proche et ensuite regarder à l'intérieur du répertoire parent. La modification de la configuration du classpath peut induire au choix d'un répertoire différent ou incorrect.
>
> Vous pouvez toujours utiliser le nom pleinement qualifié de la localisation de la ressource plutôt que le chemin relatif : par exemple `file:C:/config/services.xml` ou bien `classpath:/config/services.xml`. Cependant, soyez attentif à ne pas coupler la configuration de votre application à des chemins absolus. Il est généralement préférable de garder une abstraction pour de tels chemins absolus — par exemple, à l'aide d'espaces réservés comme `"${...}"`, qui sont résolus depuis le système de propriétés de la **JVM** à l'exécution.

L’espace de noms fournit lui-même la fonctionnalité de directive d’import. Pour des configurations avancées allant au-delà des fonctionnalités disponibles pour la définition des **beans**, une sélection d'espaces de noms est fournie par _Spring_ — par exemple, l'espace de noms `context` ou bien `util`.

# __*Utilisation du conteneur*__

L'interface `ApplicationContext` sert de **factory** avancée capable de maintenir un registre des différents **beans** et de leurs dépendances. En utilisant la méthode `T getBean(String name, Class<T> requiredType)`, vous pouvez retrouver les instances de vos **beans**.

L'interface `ApplicationContext` vous permet de lire les définitions de vos **beans** et d’y accéder, comme le montre l'exemple suivant :

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

```kotlin
import org.springframework.beans.factory.getBean

// create and configure beans
val context = ClassPathXmlApplicationContext("services.xml", "daos.xml")

// retrieve configured instance
val service = context.getBean<PetStoreService>("petStore")

// use configured instance
var userList = service.getUsernameList()
```

Avec une configuration **Groovy**, le chargement est très similaire. Il existe une classe d’implémentation de contexte différente pour **Groovy** (mais elle comprend également les fichiers de définition de **beans** XML). L'exemple suivant montre une configuration **Groovy** :

```java
ApplicationContext context = new GenericGroovyApplicationContext("services.groovy", "daos.groovy");
```

```kotlin
val context = GenericGroovyApplicationContext("services.groovy", "daos.groovy")
```

La variante la plus flexible est l'interface `GenericApplicationContext` conjointement avec des délégués de lecture — par exemple, avec `XmlBeanDefinitionReader` pour les fichiers XML, comme le montre l'exemple suivant :

```java
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```

```kotlin
val context = GenericApplicationContext()
XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml")
context.refresh()
```

Vous pouvez également utiliser l'interface `GroovyBeanDefinitionReader` pour les fichiers **Groovy**, comme le montre l'exemple suivant :

```java
GenericApplicationContext context = new GenericApplicationContext();
new GroovyBeanDefinitionReader(context).loadBeanDefinitions("services.groovy", "daos.groovy");
context.refresh();
```

```kotlin
val context = GenericApplicationContext()
GroovyBeanDefinitionReader(context).loadBeanDefinitions("services.groovy", "daos.groovy")
context.refresh()
```

Vous pouvez mélanger et associer de tels délégués de lecture au sein d’un même `ApplicationContext`, en lisant des définitions de **beans** depuis différentes sources de configuration.

Vous pouvez toujours utiliser la méthode `getBean` pour retrouver les instances de vos **beans**. L'interface `ApplicationContext` dispose de plusieurs méthodes pour retrouver des **beans**, mais, idéalement, votre code applicatif ne devrait pas les utiliser. En effet, votre code applicatif ne devrait comporter aucun appel à la méthode `getBean()` et ainsi n'avoir aucune dépendance aux API de _Spring_.

Par exemple, l'intégration de _Spring_ avec des frameworks web fournit une injection de dépendances pour différents composants du framework web, tels que des contrôleurs et des **beans** gérés par JSF, vous permettant ainsi de déclarer une dépendance sur un **bean** spécifique à travers les métadonnées (telles que l'annotation d’**autowiring**).
