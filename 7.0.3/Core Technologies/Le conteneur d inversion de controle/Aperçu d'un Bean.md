{
    page:6,
    from:"https://docs.spring.io/spring-framework/reference/core/beans/basics.html",
    update:"2026-02-22",
    by:["Arthur Leroux"],
}
# __*Aperçu d'un Bean*__

Un conteneur d'inversion de contrôle de _Spring_ gère un ou plusieurs **beans**. Ces **beans** sont créés à partir de la configuration des métadonnées que vous fournissez au conteneur (par exemple, dans un fichier XML `<beans/>`).  

Dans le conteneur, les définitions de ces **beans** sont représentées comme des objets `BeanDefinition`, lesquels contiennent (parmi d'autres informations) les métadonnées suivantes :

* Le nom pleinement qualifié de la classe : typiquement, l’implémentation concrète de la classe définie.  
* Les éléments de configuration du comportement du **bean** : comment le **bean** doit se comporter dans le conteneur (**scope**, **lifecycle**, **callback**, etc.).  
* Les références aux autres **beans** nécessaires au fonctionnement de ce **bean**. Ces références sont aussi appelées **collaborateurs** ou **dépendances**.  
* D'autres paramètres de configuration spécifiques à appliquer à l'objet nouvellement créé — par exemple, la taille maximale d’un pool ou le nombre de connexions à utiliser dans un **bean** gérant un pool de connexions.  

Ces métadonnées traduisent une liste de propriétés qui définissent chacune des définitions de **beans**. Le tableau suivant décrit ces propriétés :

| Propriété | Description / Référence |
|:---------:|:----------------------:|
| Class | [Instanciation des Beans](#instanciation-des-beans) |
| Name | [Nommage des Beans](#nommage-des-beans) |
| Scope | [Portée des Beans](A COMPLETER) |
| Constructor arguments | [Injection de dépendances](A COMPLETER) |
| Properties | [Injection de dépendances](A COMPLETER) |
| Autowiring mode | [Injection de dépendances](A COMPLETER) |
| Lazy initialization mode | [Instanciation paresseuse des Beans](A COMPLETER) |
| Initialization method | [Callbacks d'initialisation](A COMPLETER) |
| Destruction method | [Callbacks de destruction](A COMPLETER) |

---

En plus des définitions de **beans**, qui contiennent des informations sur la manière de créer un **bean**, les implémentations de `ApplicationContext` permettent l'enregistrement d'objets existants créés en dehors du conteneur (par l'utilisateur).  

Cela est possible via l'accès au `BeanFactory` de l'**ApplicationContext** avec la méthode `getAutowireCapableBeanFactory()`, qui retourne l’implémentation `DefaultListableBeanFactory`. Cette dernière supporte l’enregistrement de **singletons** via les méthodes `registerSingleton()` et l’enregistrement de définitions de beans avec `registerBeanDefinition()`.  

Cependant, dans les applications classiques, ces fonctionnalités sont principalement utilisées via les métadonnées de **beans** traditionnelles.

> **NOTE**  
> Les métadonnées des **beans** et les instances **singleton** ajoutées manuellement doivent être enregistrées le plus tôt possible. Cela permet au conteneur de résoudre correctement les injections et autres étapes d’introspection.  
> Bien que la réécriture de métadonnées existantes et de **singletons** soit partiellement supportée, l’enregistrement de nouveaux **beans** à l’exécution (parallèlement à l’accès direct à la **factory**) n’est pas officiellement supporté et peut provoquer des exceptions liées à la concurrence ou des états inconsistants dans le conteneur.

---

## __*Réécriture des beans*__

La réécriture des **beans** a lieu lorsqu’un **bean** est enregistré en utilisant un identifiant déjà alloué. Bien que la réécriture d’un **bean** soit possible, elle rend la configuration plus difficile à lire.

> **WARNING**  
> La réécriture des **beans** sera dépréciée dans une version future.

Pour désactiver totalement la réécriture des **beans**, vous pouvez définir la propriété `allowBeanDefinitionOverriding` à `false` sur l’`ApplicationContext` avant qu’il ne soit rafraîchi. Dans ce cas, une exception est levée si une réécriture est effectuée.

Par défaut, le conteneur journalise chaque tentative de réécriture d’un **bean** au niveau `INFO`, afin que vous puissiez adapter votre configuration en conséquence. Bien que cela ne soit pas recommandé, vous pouvez réduire ces logs au silence en définissant la propriété `allowBeanDefinitionOverriding` à `true`.

> **Java Configuration**  
> Si vous utilisez la configuration Java, une méthode annotée `@Bean` réécrira toujours silencieusement un **bean** détecté par scan de composants portant le même nom de composant, à condition que le type de retour de la méthode annotée `@Bean` corresponde à la classe de ce **bean**. Cela signifie que le conteneur appellera la méthode factory `@Bean` plutôt que d’utiliser un constructeur pré-déclaré dans la classe du **bean**.

> **Note**  
> Nous reconnaissons que la réécriture des **beans** dans des scénarios de test est pratique et qu’un support explicite est prévu à cet effet. Veuillez vous référer à [cette section](A COMPLETER) pour plus de détails.


## __*Nommage des beans*__

Chaque **bean** possède un ou plusieurs identifiants. Ces identifiants doivent être uniques dans le conteneur qui héberge le **bean**. Un **bean** possède généralement un seul identifiant. Cependant, si plusieurs identifiants sont nécessaires, les identifiants supplémentaires peuvent être considérés comme des alias.

Dans une configuration des métadonnées orientée XML, vous pouvez utiliser l'attribut `id`, l'attribut `name`, ou les deux, pour spécifier les identifiants des **beans**. L'attribut `id` permet de spécifier exactement un identifiant. Par convention, ces noms sont alphanumériques (`myBean`, `someService`, etc.), mais ils peuvent également contenir des caractères spéciaux. Si vous souhaitez introduire d'autres alias pour un **bean**, vous pouvez les spécifier dans l'attribut `name`, séparés par une virgule (`,`), un point-virgule (`;`) ou un espace. Bien que l'attribut `id` soit défini comme un type `xsd:string`, l'unicité des identifiants est assurée par le conteneur et non par les parseurs XML.

Vous n'êtes pas obligé de fournir un `name` ou un `id` pour un **bean**. Si vous ne les fournissez pas explicitement, le conteneur génère un nom unique pour ce **bean**. Cependant, si vous souhaitez référencer ce **bean** par nom, via l'utilisation de l'élément `ref` ou d'une recherche de type Service Locator, vous devez fournir un nom. Les raisons de ne pas fournir de nom sont liées à l'utilisation de [inner beans](A COMPLETER) et [autowiring collaborators](A COMPLETER).

> **Convention de nommage des beans**  
> La convention consiste à utiliser la convention standard Java pour les noms de champs d'instance lors du nommage des **beans**. Autrement dit, les noms des **beans** commencent par une lettre minuscule et suivent le format camel-case à partir de là. Par exemple : `accountManager`, `accountService`, `userDao`, `loginController`, etc.
>
> Le nommage cohérent des **beans** rend votre configuration plus lisible et compréhensible. De plus, si vous utilisez __Spring AOP__, cela facilite grandement l'application de recommandations à un ensemble de **beans** liés par leur nom.
>
> Avec le scannage des composants depuis le **classpath**, _Spring_ génère les noms des **beans** pour les composants sans nom en suivant les règles décrites ci-dessus : essentiellement, le nom simple de la classe est pris et sa première lettre mise en minuscule. Cependant, dans le cas inhabituel où le nom contient plus d'un caractère et que les deux premiers sont des majuscules, la casse initiale est conservée. Ce sont les mêmes règles définies par `java.beans.Introspector.decapitalize` (utilisé ici par _Spring_).

### __*Créer un alias pour un bean en dehors de la définition du bean*__

Dans la définition même d’un **bean**, vous pouvez fournir plusieurs noms pour celui-ci en utilisant une combinaison de l’attribut `id` (qui ne peut contenir qu’un seul nom) et d’un nombre quelconque d’autres noms dans l’attribut `name`.

Ces noms peuvent être des alias équivalents au **bean** courant et sont utiles dans certaines situations, par exemple pour permettre à chaque composant d’une application de se référer à une dépendance commune en utilisant un nom spécifique à son propre contexte.

Spécifier tous les alias à l’endroit où le **bean** est défini n’est pas toujours suffisant. Il peut être préférable d’introduire un alias pour un **bean** défini ailleurs.

C’est typiquement le cas dans les systèmes de grande taille où la configuration est répartie entre plusieurs sous-systèmes, chacun disposant de son propre ensemble de définitions d’objets.

Dans une configuration basée sur des métadonnées XML, vous pouvez utiliser l’élément `<alias/>` pour accomplir cela. L’exemple suivant montre comment procéder :

```xml
<alias name="fromName" alias="toName"/>
```

Dans ce cas, un **bean** (dans le même conteneur) nommé `fromName` peut également, après la déclaration de cet alias, être référencé sous le nom `toName`.

Par exemple, la configuration des métadonnées du sous-système **A** peut référencer un **DataSource** sous le nom `subsystemA-dataSource`.

La configuration des métadonnées du sous-système **B** peut référencer le **DataSource** sous le nom `subsystemB-dataSource`.

Lors de la composition de l’application principale utilisant ces deux sous-systèmes, celle-ci peut référencer le **DataSource** sous le nom `myApp-dataSource`.

Pour que ces trois noms référencent le même objet, vous pouvez ajouter les définitions d’alias suivantes dans la configuration des métadonnées :

```xml
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```

Ainsi, chaque composant et l’application principale peuvent référencer le **DataSource** via un nom unique, garanti sans collision avec d’autres définitions (créant ainsi un espace de noms), tout en pointant vers le même **bean**.

> **Configuration Java**  
> Si vous utilisez une configuration Java, l’annotation `@Bean` peut être utilisée pour déclarer des alias. Voir *Using the @Bean Annotation* pour plus de détails.


## __*Instanciation des Beans*__

La définition d'un **bean** est essentiellement une recette pour créer un ou plusieurs objets. Le conteneur consulte cette recette pour un **bean** nommé lorsqu’il est demandé et utilise les métadonnées de configuration encapsulées dans la définition du **bean** pour créer (ou acquérir) l'objet correspondant.

Si vous utilisez une configuration orientée XML, vous spécifiez le type (ou **class**) de l'objet à instancier dans l'attribut `class` de l'élément `<bean/>`.  
Cet attribut `class` (qui, en interne, correspond à la propriété `Class` d'une instance de `BeanDefinition`) est normalement obligatoire.  
(Pour les exceptions, voir [Instantiation by Using an Instance Factory Method](#instanciation-en-utilisant-une-instance-dune-méthode-factory) et [BeanDefinition Inheritance](A COMPLETER)). Vous pouvez utiliser la propriété `Class` de l'une de ces deux manières :

* Généralement, en spécifiant la **class** du **bean** à construire lorsque le conteneur crée le **bean** en appelant son constructeur par réflexion, ce qui est équivalent à du code Java utilisant l'opérateur `new`.  
* En spécifiant une classe contenant une méthode **static factory** qui sera invoquée pour créer l'objet, dans les cas moins courants où le conteneur appelle une méthode **static factory** sur une classe pour créer le **bean**. Le type de l'objet retourné par cette méthode peut être la même classe ou une classe entièrement différente.

> **Noms de classes imbriquées**  
> Si vous voulez configurer la définition d'un **bean** pour une **class** imbriquée, vous pouvez utiliser soit le nom binaire, soit le nom source de la **class** imbriquée.  
>
> Par exemple, si vous avez une **class** appelée `SomeThing` dans le **package** `com.example`, et que la **class** `SomeThing` contient une **class** **static** imbriquée appelée `OtherThing`, elles peuvent être séparées par un signe `$` ou un point `.`. Dans ce cas, l'attribut `class` dans la définition du **bean** devrait être `com.example.SomeThing$OtherThing` ou bien `com.example.SomeThing.OtherThing`.


### _**Instanciation avec un constructeur**_

Lorsque vous créez un **bean** par l'approche du constructeur, toutes les classes normales et compatibles avec _Spring_ peuvent être utilisées.  
Autrement dit, la **class** créée n'a pas besoin d’implémenter une interface spécifique ni d’être codée d’une manière particulière. Indiquer le nom de la **class** du **bean** suffit généralement. Cependant, selon le type de conteneur d’inversion de contrôle que vous utilisez pour un **bean** spécifique, vous pourriez avoir besoin d’un constructeur par défaut (sans argument).

Le conteneur d’inversion de contrôle de _Spring_ peut gérer virtuellement n’importe quelle **class** que vous souhaitez. Il n’est pas limité aux véritables **JavaBeans**. La plupart des utilisateurs de _Spring_ préfèrent utiliser des **JavaBeans** avec un constructeur par défaut et des **setters/getters** appropriés pour configurer les propriétés dans le conteneur. Vous pouvez également avoir plusieurs **beans** plus exotiques. Par exemple, si vous avez besoin d’un pool de connexions **legacy** qui ne correspond pas à la spécification d’un **JavaBean**, _Spring_ peut le gérer également.

Avec une configuration des métadonnées orientée XML, vous pouvez spécifier la **class** de vos **beans** comme suit :

```xml
<bean id="exampleBean" class="examples.ExampleBean"/>
<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

Pour plus de détails concernant le mécanisme pour fournir les arguments au constructeur (si nécessaire) et définir les propriétés des objets après leur construction, voir [Injecting Dependencies](A COMPLETER).

> **NOTE**  
> Dans le cas d’un constructeur avec arguments, le conteneur peut sélectionner le constructeur correspondant parmi plusieurs surcharges.  
> Cela dit, pour éviter les ambiguïtés, il est recommandé de garder les signatures de vos constructeurs aussi simples que possible.



### _**Instanciation avec une méthode static factory**_

Lors de la définition d'un **bean** qui sera créé via une méthode **static factory**, utilisez l'attribut `class` pour spécifier la **class** contenant la méthode **static factory** et l'attribut `factory-method` pour indiquer le nom de la méthode elle-même. Vous devriez être capable d'appeler cette méthode (avec des arguments optionnels, comme décrit plus tard) et de retourner un objet, lequel sera ensuite traité comme s'il avait été créé depuis un constructeur. Un usage courant de ce type de définition de **bean** est d'appeler des **factories statiques** depuis le code.

La définition de **bean** suivante définit un **bean** qui sera créé en appelant une **factory method**. La définition ne doit pas spécifier le type (**class**) de l'objet retourné mais plutôt la **class** contenant la **factory method**. Dans cet exemple, la méthode `createInstance()` doit être une méthode `static`. L'exemple suivant montre comment spécifier une méthode **factory** :

```xml
<bean id="clientService"
	class="examples.ClientService"
	factory-method="createInstance"/>
```

L'exemple suivant montre une **class** qui devrait fonctionner avec la définition précédente :

```java
public class ClientService {
	private static ClientService clientService = new ClientService();
	private ClientService() {}

	public static ClientService createInstance() {
		return clientService;
	}
}
```

```kotlin
class ClientService private constructor() {
	companion object {
		private val clientService = ClientService()
		@JvmStatic
		fun createInstance() = clientService
	}
}
```

Pour plus de détails concernant le mécanisme pour fournir les arguments à la méthode **factory** (si nécessaire) et définir les propriétés des objets après leur retour par la **factory**, voir [Dependencies and Configuration in Detail](A COMPLETER).

> **NOTE**  
> Dans le cas d'une méthode **factory** avec arguments, le conteneur peut sélectionner la méthode correspondante parmi plusieurs surcharges.  
> Cela dit, pour éviter les ambiguïtés, il est recommandé de garder les signatures de vos méthodes **factory** aussi simples que possible.

> **TIP**  
> Un problème typique dans le cas de surcharge d'une méthode **factory** est _Mockito_ avec ses nombreuses surcharges de la méthode `mock`. Choisissez la variante la plus spécifique de la méthode `mock` possible :
>
> ```xml
> <bean id="clientService" class="org.mockito.Mockito" factory-method="mock">
>	<constructor-arg type="java.lang.Class" value="examples.ClientService"/>
>	<constructor-arg type="java.lang.String" value="clientService"/>
> </bean>
> ```


### _**Instanciation en utilisant une instance d'une méthode factory**_

De manière similaire à l'instanciation via une [static factory method](#instanciation-avec-une-méthode-static-factory), l'instanciation avec une instance d'une méthode **factory** invoque une méthode non-`static` d'un **bean** existant dans le conteneur pour créer un nouveau **bean**. Pour utiliser ce mécanisme, laissez l'attribut `class` vide et, dans l'attribut `factory-bean`, spécifiez le nom d'un **bean** dans le conteneur courant (parent ou ancêtre) qui contient une instance de la méthode à invoquer pour créer l'objet. L'exemple suivant montre comment configurer un tel **bean** :

```xml
<!-- le bean factory, qui contient une méthode appelée createClientServiceInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
	<!-- injecter les dépendances requises par ce bean factory -->
</bean>

<!-- le bean à créer via le bean factory -->
<bean id="clientService"
	factory-bean="serviceLocator"
	factory-method="createClientServiceInstance"/>
```

L'exemple suivant montre la **class** correspondante :

```java
public class DefaultServiceLocator {

	private static ClientService clientService = new ClientServiceImpl();
	private static AccountService accountService = new AccountServiceImpl();

	public ClientService createClientServiceInstance() {
		return clientService;
	}

	public AccountService createAccountServiceInstance() {
		return accountService;
	}
}
```

```kotlin
class DefaultServiceLocator {
	companion object {
		private val clientService = ClientServiceImpl()
		private val accountService = AccountServiceImpl()
	}

	fun createClientServiceInstance(): ClientService {
		return clientService
	}

	fun createAccountServiceInstance(): AccountService {
		return accountService
	}
}
```

Une **factory class** peut aussi comporter plus d'une méthode **factory**, comme le montre l'exemple suivant:

```xml
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
	<!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
	factory-bean="serviceLocator"
	factory-method="createClientServiceInstance"/>

<bean id="accountService"
	factory-bean="serviceLocator"
	factory-method="createAccountServiceInstance"/>
```

L'exemple suivant montre la **class** correspondante :

```java
public class DefaultServiceLocator {

	private static ClientService clientService = new ClientServiceImpl();

	private static AccountService accountService = new AccountServiceImpl();

	public ClientService createClientServiceInstance() {
		return clientService;
	}

	public AccountService createAccountServiceInstance() {
		return accountService;
	}
}
```
```kotlin
class DefaultServiceLocator {
	companion object {
		private val clientService = ClientServiceImpl()
		private val accountService = AccountServiceImpl()
	}

	fun createClientServiceInstance(): ClientService {
		return clientService
	}

	fun createAccountServiceInstance(): AccountService {
		return accountService
	}
}
```

Cette approche montre que le **bean factory** lui-même peut être géré et configuré à travers l'injection de dépendances (DI). Voir [Dependencies and Configuration en détail](A COMPLETER).

> **NOTE**  
> Dans la documentation _Spring_, "factory bean" fait référence à un **bean** qui est configuré dans le conteneur _Spring_ et qui crée des objets via une [instance](#instanciation-en-utilisant-une-instance-dune-méthode-factory) ou une **factory** [static](#instanciation-avec-une-méthode-static-factory) méthode.  
> En contraste, `FactoryBean` (remarquez les lettres majuscules) fait référence à l'implémentation spécifique de _Spring_ de la **class** [FactoryBean](A COMPLETER).

---

### _**Déterminer le type d'un bean à l'exécution**_

Déterminer le type spécifique d'un **bean** à l'exécution n'est pas trivial. Une **class** spécifiée dans la définition des métadonnées d'un **bean** est juste une référence initiale à une **class**, potentiellement combinée avec une méthode **factory** déclarée, ou étant une **class** `FactoryBean` qui peut conduire à différents types d'un **bean** à l'exécution, ou peut ne pas être définie du tout dans le cas d'une instance créée par une méthode **factory** (résolue via l'attribut `factory-bean`). De plus, un proxy AOP peut encapsuler l'instance d'un **bean** avec un proxy orienté interface, limitant l'exposition du type réel du **bean** (juste ses interfaces implémentées).

La manière recommandée de retrouver le type réel à l'exécution pour un **bean** particulier est d'appeler `BeanFactory.getType` pour le nom spécifique du **bean**. Cette approche prend en compte tous les cas ci-dessus et retourne le type de l'objet que l'appel à `BeanFactory.getBean` va retourner pour le même nom de **bean**.