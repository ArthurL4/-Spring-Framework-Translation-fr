{
    page: 16,
    from: "https://docs.spring.io/spring-framework/reference/core/beans/child-bean-definitions.html",
    update: "2026-04-02",
    by: ["Arthur Leroux"],
}

# _**Points d'extension du conteneur**_

Généralement, un développeur d'application n'a pas besoin de sous-classer les classes d'implémentation de `ApplicationContext`. À la place, le conteneur d'inversion de contrôle peut être étendu en s'appuyant sur des implémentations d'interfaces d'intégration spécifiques. Les sections suivantes décrivent ces interfaces d'intégration.

## _**Personnaliser les beans avec un BeanPostProcessor**_

L'interface `BeanPostProcessor` définit des méthodes de callback que vous pouvez implémenter afin de fournir votre propre logique d'instanciation, de résolution des dépendances, et plus encore (ou pour surcharger celle du conteneur par défaut). Si vous souhaitez implémenter une logique personnalisée après que le conteneur _Spring_ a terminé d'instancier, de configurer et d'initialiser un **bean**, vous pouvez enregistrer une ou plusieurs implémentations de `BeanPostProcessor`.

Vous pouvez configurer plusieurs instances de `BeanPostProcessor`, et contrôler l'ordre dans lequel elles s'exécutent en définissant la propriété `order`. Cette propriété ne peut être utilisée que si le `BeanPostProcessor` implémente l'interface `Ordered`. Si vous écrivez votre propre `BeanPostProcessor`, vous devriez également envisager d'implémenter l'interface `Ordered`. Pour plus de détails, consultez la Javadoc des interfaces `BeanPostProcessor` et `Ordered`.  
Voir également la note sur l'enregistrement programmatique des instances de `BeanPostProcessor`.

> **NOTE**
>
> Les instances de `BeanPostProcessor` opèrent sur des **beans** (objets). Le conteneur d'inversion de contrôle de _Spring_ instancie d'abord un **bean**, puis les instances de `BeanPostProcessor` appliquent leur traitement.
>
> Les instances de `BeanPostProcessor` sont spécifiques à un conteneur. Cela est pertinent uniquement si vous utilisez une hiérarchie de conteneurs. Si vous définissez un `BeanPostProcessor` dans un conteneur, il ne traite que les **beans** de ce conteneur. En d'autres termes, les **beans** définis dans un conteneur ne sont pas traités par un `BeanPostProcessor` défini dans un autre conteneur, même si les deux conteneurs font partie d'une même hiérarchie.
>
> Pour modifier la définition d'un **bean** (c'est-à-dire le plan de configuration qui le décrit), vous devez plutôt utiliser un `BeanFactoryPostProcessor`, comme décrit dans la section correspondante.

L'interface `org.springframework.beans.factory.config.BeanPostProcessor` contient exactement deux méthodes. Lorsqu'une classe est enregistrée comme post-processeur dans le conteneur, pour chaque instance de **bean** créée, le post-processeur reçoit un callback avant que les méthodes d'initialisation du conteneur (telles que `InitializingBean.afterPropertiesSet()` ou toute méthode `init`) ne soient appelées, et également après ces callbacks d'initialisation.

Le post-processeur peut effectuer n'importe quelle action sur l'instance du **bean**, y compris ignorer complètement le traitement. Un `BeanPostProcessor` vérifie généralement la présence d'interfaces de callback ou peut encapsuler un **bean** dans un proxy. Certaines classes d'infrastructure AOP de _Spring_ sont implémentées comme des `BeanPostProcessor` afin de fournir une logique de proxy.

Un `ApplicationContext` détecte automatiquement tout **bean** défini dans la configuration qui implémente l'interface `BeanPostProcessor`. L'`ApplicationContext` enregistre ces **beans** comme post-processeurs afin qu'ils soient appelés lors de la création des **beans**. Un `BeanPostProcessor` peut être déclaré comme n'importe quel autre **bean**.

Notez que, lors de la déclaration d'un `BeanPostProcessor` via une méthode factory annotée `@Bean` dans une classe de configuration, le type de retour de la méthode doit être la classe d'implémentation elle-même ou au minimum l'interface `org.springframework.beans.factory.config.BeanPostProcessor`. Cela permet à l'`ApplicationContext` de détecter correctement sa nature. Sinon, il ne pourra pas être détecté automatiquement par type avant sa création complète. Étant donné que les `BeanPostProcessor` doivent être instanciés tôt afin de s'appliquer à l'initialisation des autres **beans**, cette détection précoce est essentielle.

De plus, lors de l'enregistrement d'un `BeanPostProcessor` via une méthode `@Bean`, il est recommandé de déclarer cette méthode comme `static` et idéalement sans dépendances. Cela évite une initialisation précoce de la classe de configuration et des autres **beans**, ce qui pourrait les rendre inéligibles à un post-traitement complet (comme l'auto-proxying).

> **NOTE**
>
> _Enregistrement programmatique des instances de `BeanPostProcessor`_
>
> Bien qu'il soit recommandé d'utiliser l'enregistrement automatique via l'`ApplicationContext`, il est possible d'enregistrer programmatiquement des instances via `ConfigurableBeanFactory` avec la méthode `addBeanPostProcessor`. Cela peut être utile pour appliquer des conditions logiques avant l'enregistrement ou pour dupliquer un post-processeur.

> **NOTE**
>
> _Instances de `BeanPostProcessor` et initialisation précoce_
>
> Les classes implémentant `BeanPostProcessor` sont particulières et sont traitées différemment par le conteneur. Toutes les instances de `BeanPostProcessor` ainsi que les **beans** qu'elles référencent directement sont instanciées dans une phase spéciale du cycle de vie de l'`ApplicationContext`.
>
> Ensuite, toutes les instances de `BeanPostProcessor` sont enregistrées dans un ordre déterminé et appliquées aux **beans** suivants.
>
> Étant donné que l'auto-proxying AOP est implémenté via des `BeanPostProcessor`, ni ces derniers ni les **beans** qu'ils référencent directement ne sont éligibles à l'auto-proxying et ne bénéficient donc pas du tissage d'aspects.
>
> Plus généralement, tout **bean** instancié durant cette phase précoce ne pourra pas bénéficier d'un post-traitement complet.
>
> Dans ces cas, vous pouvez observer un log de niveau WARN similaire :
>
> _**Bean 'someBean' of type [org.example.SomeType] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying).**_
>
> Pour réduire ce phénomène, déclarez les `BeanPostProcessor` via des méthodes `static` sans dépendances. Si des dépendances sont injectées (via `@Autowired` ou `@Resource`), _Spring_ peut être amené à instancier d'autres **beans** trop tôt lors de la résolution des dépendances, les rendant inéligibles à l'auto-proxying.
>
> Par exemple, si une dépendance annotée `@Resource` ne correspond pas directement à un nom de **bean**, _Spring_ peut effectuer une résolution par type, ce qui entraîne un accès anticipé à d'autres **beans**.

Les exemples suivants montrent comment écrire, enregistrer et utiliser des instances de `BeanPostProcessor` dans un `ApplicationContext`.

### _**Exemple : Hello World, BeanPostProcessor-style**_

Le premier exemple illustre un usage basique. Il montre une implémentation personnalisée d’un `BeanPostProcessor` qui invoque la méthode `toString()` de chaque **bean** dès qu’il est créé par le conteneur, puis affiche le résultat dans la console système.

Le code suivant montre la définition de cette implémentation personnalisée de `BeanPostProcessor` :

```java
package scripting;

import org.springframework.beans.factory.config.BeanPostProcessor;

public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {

	// simply return the instantiated bean as-is
	public Object postProcessBeforeInitialization(Object bean, String beanName) {
		return bean; // we could potentially return any object reference here...
	}

	public Object postProcessAfterInitialization(Object bean, String beanName) {
		System.out.println(\"Bean '\" + beanName + \"' created : \" + bean);
		return bean;
	}
}
```

```kotlin
package scripting

import org.springframework.beans.factory.config.BeanPostProcessor

class InstantiationTracingBeanPostProcessor : BeanPostProcessor {

	// simply return the instantiated bean as-is
	override fun postProcessBeforeInitialization(bean: Any, beanName: String): Any? {
		return bean // we could potentially return any object reference here...
	}

	override fun postProcessAfterInitialization(bean: Any, beanName: String): Any? {
		println(\"Bean '$beanName' created : $bean\")
		return bean
	}
}
```

Vous pouvez enregistrer `InstantiationTracingBeanPostProcessor` avec une configuration Java en utilisant une méthode `static` annotée `@Bean` (recommandé afin d’éviter des initialisations précoces de la **classe** de configuration et d’autres **beans**) :

```java
@Configuration
public class AppConfig {

	@Bean
	public static InstantiationTracingBeanPostProcessor instantiationTracingBeanPostProcessor() {
		return new InstantiationTracingBeanPostProcessor();
	}

	// ... other bean definitions
}
```

```kotlin
@Configuration
class AppConfig {

	@Bean
	companion object {
		@JvmStatic
		fun instantiationTracingBeanPostProcessor() = InstantiationTracingBeanPostProcessor()
	}

	// ... other bean definitions
}
```

De manière alternative, `InstantiationTracingBeanPostProcessor` peut être enregistré via un élément `<bean>` avec une configuration XML :

```xml
<?xml version=\"1.0\" encoding=\"UTF-8\"?>
<beans xmlns=\"http://www.springframework.org/schema/beans\"
	xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"
	xmlns:lang=\"http://www.springframework.org/schema/lang\"
	xsi:schemaLocation=\"http://www.springframework.org/schema/beans
		https://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/lang
		https://www.springframework.org/schema/lang/spring-lang.xsd\">

	<lang:groovy id=\"messenger\"
			script-source=\"classpath:org/springframework/scripting/groovy/Messenger.groovy\">
		<lang:property name=\"message\" value=\"Fiona Apple Is Just So Dreamy.\"/>
	</lang:groovy>

	<!--
	when the above bean (messenger) is instantiated, this custom
	BeanPostProcessor implementation will output the fact to the system console
	-->
	<bean class=\"scripting.InstantiationTracingBeanPostProcessor\"/>

</beans>
```

Remarquez que `InstantiationTracingBeanPostProcessor` est seulement défini. Il ne possède même pas de nom et, comme c’est un **bean**, il peut être injecté comme dépendance dans n’importe quel autre **bean**. (La configuration précédente définit également un **bean** basé par un script Groovy.)

L’application Java suivante exécute le code et la configuration précédents :

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.scripting.Messenger;

public final class Boot {

	public static void main(final String[] args) throws Exception {
		ApplicationContext ctx = new ClassPathXmlApplicationContext(\"scripting/beans.xml\");
		Messenger messenger = ctx.getBean(\"messenger\", Messenger.class);
		System.out.println(messenger);
	}

}
```

La sortie de l’application précédente ressemble à ceci :

```
Bean 'messenger' created : org.springframework.scripting.groovy.GroovyMessenger@272961
org.springframework.scripting.groovy.GroovyMessenger@272961
```

### _**Exemple : L'AutowiredAnnotationBeanPostProcessor**_

L'utilisation d'interfaces de callback ou d'annotations, en conjonction avec une implémentation personnalisée de `BeanPostProcessor`, est une manière courante d'étendre le conteneur d'inversion de contrôle de _Spring_. Un exemple est l'interface `AutowiredAnnotationBeanPostProcessor` de _Spring_, une implémentation de `BeanPostProcessor` fournie avec la distribution de _Spring_, qui réalise l'autocâblage des champs annotés, des méthodes **setters** et des méthodes de configuration arbitraires.

## _**Personnaliser une configuration des métadonnées avec un BeanFactoryPostProcessor**_

Le prochain point d'extension que nous examinons est l'interface `org.springframework.beans.factory.config.BeanFactoryPostProcessor`. La sémantique de cette interface est similaire à celle de `BeanPostProcessor`, avec une différence majeure : `BeanFactoryPostProcessor` opère sur les configurations des métadonnées des **beans**. Autrement dit, le conteneur d'inversion de contrôle de _Spring_ permet à un `BeanFactoryPostProcessor` de lire la configuration des métadonnées et potentiellement de la modifier avant que le conteneur n'instancie tout **bean**, à l'exception des instances de `BeanFactoryPostProcessor`.

Vous pouvez configurer plusieurs instances de `BeanFactoryPostProcessor`, et vous pouvez contrôler l'ordre dans lequel elles s'exécutent en définissant la propriété `order`. Cependant, vous ne pouvez appliquer cette propriété que si le `BeanFactoryPostProcessor` implémente l'interface `Ordered`. Si vous écrivez votre propre `BeanFactoryPostProcessor`, vous devriez également envisager d'implémenter l'interface `Ordered`. Consultez la Javadoc des interfaces [BeanFactoryPostProcessor](https://docs.spring.io/spring-framework/docs/7.0.6/javadoc-api/org/springframework/beans/factory/config/BeanFactoryPostProcessor.html) et [Ordered](https://docs.spring.io/spring-framework/docs/7.0.6/javadoc-api/org/springframework/core/Ordered.html) pour plus de détails.

> **NOTE**
>
> Si vous souhaitez modifier les instances réelles des **beans** (c'est-à-dire les objets créés à partir de la configuration des métadonnées), vous devez plutôt utiliser un `BeanPostProcessor` (décrit plus tôt dans [Personnaliser les beans avec un BeanPostProcessor](#personnaliser-les-beans-avec-un-beanpostprocessor)). Bien qu'il soit techniquement possible de travailler avec les instances des **beans** dans un `BeanFactoryPostProcessor` (par exemple en utilisant `BeanFactory.getBean()`), cela entraîne des instanciations prématurées de **beans**, ce qui viole le cycle de vie standard du conteneur. Cela peut provoquer des effets de bord indésirables, comme contourner le post-traitement des **beans**.
>
> De plus, les instances de `BeanFactoryPostProcessor` ont une portée limitée au conteneur. Cela n'est pertinent que si vous utilisez une hiérarchie de conteneurs. Si vous définissez un `BeanFactoryPostProcessor` dans un conteneur, il ne sera appliqué que dans ce conteneur. Les définitions de **beans** dans un conteneur ne sont pas post-traitées par les instances de `BeanFactoryPostProcessor` d'un autre conteneur, même si les deux conteneurs font partie de la même hiérarchie.

Un **bean** de post-traitement est automatiquement exécuté lorsqu'il est déclaré dans un `ApplicationContext`, afin d'appliquer les modifications à la configuration des métadonnées définie par le conteneur. _Spring_ inclut un certain nombre de **beans** de post-traitement prédéfinis, tels que `PropertyOverrideConfigurer` et `PropertySourcesPlaceholderConfigurer`. Vous pouvez également utiliser un `BeanFactoryPostProcessor` personnalisé, par exemple pour enregistrer des éditeurs de propriétés personnalisés.

Un `ApplicationContext` détecte automatiquement tout **bean** qui y est déployé et qui implémente l'interface `BeanFactoryPostProcessor`. Il utilise ces **beans** comme des post-processeurs de factory au moment approprié. Vous pouvez déployer ces **beans** post-processeurs comme n'importe quel autre **bean**.

Lors de l'enregistrement d'un `BeanFactoryPostProcessor` à l'aide d'une méthode **factory** annotée `@Bean` dans une **classe** annotée `@Configuration`, déclarez la méthode comme `static` afin d'éviter des conflits de cycle de vie avec le traitement des annotations (telles que `@Autowired`, `@Value` et `@PostConstruct`) dans la **classe** de configuration. Consultez la section "BeanFactoryPostProcessor-returning `@Bean` methods" dans la Javadoc de `@Bean` pour plus de détails.

Pour toute méthode non statique `@Bean` avec un type de retour `BeanFactoryPostProcessor`, vous devriez voir un message de log de niveau INFO similaire au suivant :

""
@Bean method MyConfig.myBfpp is non-static and returns an object assignable to Spring’s BeanFactoryPostProcessor interface. This will result in a failure to process annotations such as @Autowired, @Resource, and @PostConstruct within the method’s declaring @Configuration class. Add the 'static' modifier to this method to avoid these container lifecycle issues; see @Bean javadoc for complete details.
""

> **NOTE**
>
> Comme pour `BeanPostProcessor`, il est généralement déconseillé de configurer un `BeanFactoryPostProcessor` pour une initialisation paresseuse. Si aucun autre **bean** ne référence un `Bean(Factory)PostProcessor`, ce post-processeur ne sera jamais instancié. Ainsi, le marquer pour une initialisation paresseuse sera ignoré, et le `Bean(Factory)PostProcessor` sera instancié précocement, même si vous définissez l'attribut `default-lazy-init` à `true` sur vos éléments `<beans />`.


### _**Exemple : Substitution de propriétés Placeholder avec PropertySourcesPlaceholderConfigurer**_

Vous pouvez utiliser la **classe** `PropertySourcesPlaceholderConfigurer` pour externaliser les valeurs des propriétés d'une définition de **bean** dans un fichier séparé en utilisant le format standard Java `Properties`. Cela permet à la personne déployant l'application de personnaliser les propriétés spécifiques à l'environnement, telles que les URLs de base de données et les mots de passe, sans la complexité ni le risque de modifier les fichiers XML principaux du conteneur.  

Considérez le fragment de configuration des métadonnées orientée XML suivant, où un `DataSource` avec des valeurs de placeholder est défini :

```xml
<bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
	<property name="locations" value="classpath:com/something/jdbc.properties"/>
</bean>

<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
	<property name="driverClassName" value="${jdbc.driverClassName}"/>
	<property name="url" value="${jdbc.url}"/>
	<property name="username" value="${jdbc.username}"/>
	<property name="password" value="${jdbc.password}"/>
</bean>
```

Cet exemple montre des propriétés configurées à partir d'un fichier Properties externe. À l'exécution, un `PropertySourcesPlaceholderConfigurer` est appliqué aux métadonnées et remplace certaines propriétés du `DataSource`. Les valeurs à remplacer sont spécifiées sous forme de placeholders `${property-name}`, suivant le style **Ant**, **log4j** et **JSP EL**. Les valeurs réelles proviennent d'un autre fichier au format standard Java Properties :

```properties
jdbc.driverClassName=org.hsqldb.jdbcDriver
jdbc.url=jdbc:hsqldb:hsql://production:9002
jdbc.username=sa
jdbc.password=root
```

Ainsi, le placeholder `${jdbc.username}` est remplacé à l'exécution par la valeur `'sa'`, et la même logique s'applique à tous les autres placeholders présents dans le fichier de propriétés. La **classe** `PropertySourcesPlaceholderConfigurer` vérifie les placeholders dans la plupart des propriétés et attributs d'une définition de **bean**.  

De plus, vous pouvez configurer le préfixe, le suffixe, le séparateur par défaut et le caractère d'échappement du placeholder. Par défaut, le caractère d'échappement peut être modifié ou désactivé globalement en appliquant la propriété système JVM `spring.placeholder.escapeCharacter.default` (ou via le mécanisme [SpringProperties]()). Avec l’espace de noms `context`, vous pouvez configurer un placeholder à l’aide d’un élément de configuration dédié. Vous pouvez fournir un ou plusieurs emplacements sous forme de liste séparée par des virgules dans l’attribut `location`, comme dans l’exemple suivant :

```xml
<context:property-placeholder location="classpath:com/something/jdbc.properties"/>
```

La **classe** `PropertySourcesPlaceholderConfigurer` ne se limite pas aux fichiers Properties que vous spécifiez. Par défaut, si elle ne trouve pas une propriété dans les fichiers indiqués, elle la cherche dans l'`Environment` de Spring ainsi que dans les propriétés système régulières de Java.  

> **WARNING**  
> Un seul élément devrait être défini pour une application donnée avec les propriétés nécessaires. Plusieurs placeholders peuvent être configurés tant qu’ils ont des syntaxes distinctes (${...}).  
> Si vous devez modulariser la source des propriétés utilisées pour le remplacement, ne créez pas plusieurs placeholders. Créez plutôt votre propre **bean** `PropertySourcesPlaceholderConfigurer` qui regroupe les propriétés à utiliser.

> **NOTE**  
> Vous pouvez utiliser la **classe** `PropertySourcesPlaceholderConfigurer` pour substituer des noms de **classes**, ce qui est utile si vous devez choisir une implémentation de **classe** particulière à l’exécution. L’exemple suivant montre comment procéder :

```xml
<bean class="org.springframework.beans.factory.config.PropertySourcesPlaceholderConfigurer">
	<property name="locations">
		<value>classpath:com/something/strategy.properties</value>
	</property>
	<property name="properties">
		<value>custom.strategy.class=com.something.DefaultStrategy</value>
	</property>
</bean>

<bean id="serviceStrategy" class="${custom.strategy.class}"/>
```

Si la **classe** ne peut pas être résolue à l’exécution pour devenir une **classe** valide, la création du **bean** échoue. Cela se produit durant la phase `preInstantiateSingletons()` d’un `ApplicationContext` pour un **bean** non paresseux.

### _Exemple : Le PropertyOverrideConfigurer_

La **classe** `PropertyOverrideConfigurer`, un autre **BeanFactoryPostProcessor**, ressemble à `PropertySourcesPlaceholderConfigurer`, mais contrairement à ce dernier, les définitions initiales peuvent avoir des valeurs par défaut ou ne pas avoir de valeur du tout pour les propriétés des **beans**. Si un fichier `Properties` écrasant ne possède pas d'entrée pour une certaine propriété de **bean**, la définition du contexte par défaut est utilisée.

Notez que la définition du **bean** n'est pas consciente d'être réécrite ; il n'est donc pas immédiatement évident, depuis la définition du fichier XML, que le configureur écrasant est en train d'être utilisé. Dans le cas de multiples instances `PropertyOverrideConfigurer` qui définissent des valeurs pour la même propriété d'un **bean**, la dernière l'emporte, à cause du mécanisme de réécriture.

Les lignes de configuration d'un fichier de propriétés suivent le format suivant :

```
beanName.property=value
```

Le code suivant montre un exemple de ce format :

```
dataSource.driverClassName=com.mysql.jdbc.Driver
dataSource.url=jdbc:mysql:mydb
```

Cet exemple de fichier peut être utilisé avec la définition d'un conteneur qui contient un **bean** appelé `"dataSource"` et qui a des propriétés `driverClassName` et `url`.

Les noms composés des propriétés sont aussi supportés, tant que chaque composant du chemin, à l'exception de la propriété finale étant réécrite, est déjà non null (présumé initialisé par les constructeurs). Dans l'exemple suivant, la propriété `sammy` de la propriété `bob` de la propriété `fred` du **bean** `tom` est appliquée avec la valeur scalaire `123` :

```
tom.fred.bob.sammy=123
```

> **NOTE**
>
> Les valeurs réécrites spécifiées sont toujours des valeurs littérales. Elles ne sont pas traduites en références de **bean**. Cette convention s'applique également lorsque la valeur originale dans la définition XML du **bean** indique une référence de **bean**.

Avec l'espace de noms `context` introduit dans _Spring_ 2.5, il est possible de configurer la réécriture d'une propriété avec un élément de configuration dédié, comme le montre l'exemple suivant :

```
<context:property-override location='classpath:override.properties'/>
```

## _**Customiser la Logique d'Instanciation avec un FactoryBean**_

Vous pouvez implémenter l'interface `org.springframework.beans.factory.FactoryBean` pour les objets qui sont eux-mêmes des factories.

L'interface `FactoryBean` est un point de connectivité dans la logique d'instanciation du conteneur d'inversion de contrôle de _Spring_. Si vous avez du code d'initialisation complexe qui est mieux exprimé en Java plutôt qu'en XML, vous pouvez créer votre propre `FactoryBean`, écrire l'initialisation complexe dans cette **classe**, et ensuite connecter votre `FactoryBean` personnalisée dans le conteneur.

L'interface `FactoryBean<T>` fournit trois méthodes :

* `T getObject()` : Retourne une instance de l'objet que cette factory crée. L'instance peut possiblement être partagée, selon que cette factory retourne un singleton ou un prototype.
* `boolean isSingleton()` : Retourne `true` si ce `FactoryBean` retourne des singletons, ou `false` dans le cas contraire. L'implémentation par défaut de cette méthode retourne `true`.
* `Class<?> getObjectType()` : Retourne le type de l'objet retourné par la méthode `getObject()` ou `null` si le type n'est pas connu à l'avance.

Le concept et l'interface de `FactoryBean` sont utilisés dans de nombreux endroits dans le Framework _Spring_. Plus de 50 implémentations de l'interface `FactoryBean` sont fournies avec _Spring_ lui-même.

Lorsque vous avez besoin de questionner un conteneur pour obtenir une instance réelle de `FactoryBean` elle-même plutôt que le **bean** qu'il produit, préfixez l'`id` du **bean** avec le symbole esperluette (`&`) lors de l'appel à la méthode `getBean()` de l'`ApplicationContext`. Ainsi, pour un `FactoryBean` donné avec un `id` `myBean`, invoquer `getBean("myBean")` sur le conteneur retourne le produit de `FactoryBean`, tandis que l'invocation de `getBean("&myBean")` retourne l'instance `FactoryBean` elle-même.