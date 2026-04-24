{
    page:9,
    from:"https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-properties-detailed.html",
    update:"2026-03-01",
    by:["Arthur Leroux"],
}

# __*Dépendances et Configuration en détail*__

Comme mentionné dans la [section précédente](./Injection%20de%20Dépendances.md), vous pouvez définir les propriétés et les arguments des constructeurs soit comme références à d'autres **beans** gérés (collaborateurs), soit comme valeurs définies en ligne. La configuration des métadonnées orientée XML de *Spring* prend en charge différents types sous-éléments dans `<property/>` et `<constructor-arg/>` à cet effet.

## __*Valeurs directes (Primitives, Strings et autres)*__

L'attribut `value` de l'élément `<property/>` spécifie une propriété ou un argument de constructeur sous forme de représentation textuelle lisible (une chaîne de caractères). Le [service de conversion]() de *Spring* est utilisé pour convertir ce `String` vers le type réel de la propriété ou de l’argument. L'exemple suivant montre différentes valeurs appliquées :

```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
	<!-- results in a setDriverClassName(String) call -->
	<property name="driverClassName" value="com.mysql.jdbc.Driver"/>
	<property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
	<property name="username" value="root"/>
	<property name="password" value="misterkaoli"/>
</bean>
```

L'exemple suivant montre l'utilisation de l’espace de noms `p` pour définir une configuration XML encore plus succincte :

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
	https://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
		destroy-method="close"
		p:driverClassName="com.mysql.jdbc.Driver"
		p:url="jdbc:mysql://localhost:3306/mydb"
		p:username="root"
		p:password="misterkaoli"/>

</beans>
```

La structure XML précédente est plus succincte. Cependant, les fautes de frappes sont découvertes à l'exécution plutôt qu'au moment de la conception, sauf si vous utilisez un IDE (tel que [IntelliJ IDEA](https://www.jetbrains.com/idea/) ou [Spring Tools](https://spring.io/tools)) qui prend en charge la complétion automatique des propriétés lors de la création de vos définitions de **bean**. Une telle assistance IDE est fortement recommandée.

Vous pouvez également configurer une instance de `java.util.Properties`, comme suit :

```xml
<bean id="mappings"
	class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">

	<!-- typed as a java.util.Properties -->
	<property name="properties">
		<value>
			jdbc.driver.className=com.mysql.jdbc.Driver
			jdbc.url=jdbc:mysql://localhost:3306/mydb
		</value>
	</property>
</bean>
```

Le conteneur *Spring* convertit le texte contenu dans l'élément `<value/>` en une instance de `java.util.Properties` en utilisant le mécanisme **JavaBeans** `PropertyEditor`. Il s'agit d'un raccourci pratique et c’est l’un des rares cas où l’équipe *Spring* recommande explicitement l’utilisation de l’élément imbriqué `<value/>` plutôt que de l’attribut `value`.

### _**L'élément idref**_

L'élément `idref` est simplement une manière fiable de transmettre l'`id` (une valeur de type `String` — et non une référence) à un autre **bean** du conteneur via un élément `<constructor-arg/>` ou `<property/>`. L'exemple suivant montre comment l'utiliser :

```xml
<bean id="collaborator" class="..." />

<bean id="client" class="...">
	<property name="targetName">
		<idref bean="collaborator" />
	</property>
</bean>
```

La définition du **bean** précédente est exactement équivalente (à l'exécution) au fragment de code suivant :

```xml
<bean id="collaborator" class="..." />

<bean id="client" class="...">
	<property name="targetName" value="collaborator" />
</bean>
```

La première forme est préférable à la seconde, car l'utilisation de l'élément `<idref/>` permet au conteneur de valider, au moment du déploiement, que le bean nommé et référencé existe bien. Dans le second exemple, aucune validation n'est effectuée sur la valeur transmise à la propriété `targetName` du **bean** `client`. Les fautes de frappe sont donc découvertes (et entraînent des erreurs fatales) lorsque le **bean** `client` est effectivement instancié. Si le **bean** `client` est un **bean** [prototype](), cette faute de frappe et l'exception qui en résulte peuvent ne pas être découvertes avant longtemps après le déploiement du conteneur.

> **NOTE**
>
> Un cas courant (au moins dans les versions antérieures à *Spring* 2.0) où l'élément `<idref/>` apporte une réelle valeur ajoutée est la configuration des [AOP interceptors]() dans la définition d'un **bean** `ProxyFactoryBean`. Utiliser l'élément `<idref/>` lors de la spécification des noms des interceptors permet de s'assurer que l'ID d'un interceptor n'est pas mal orthographié.


## _**Références à d'autres Beans (Collaborateurs)**_

L'élément `ref` est l'élément final à utiliser à l'intérieur des définitions `<constructor-arg/>` ou `<property/>`. Ici, vous positionnez la valeur de la propriété spécifiée d'un **bean** pour être une référence à un autre **bean** (un collaborateur) géré par le conteneur. Le **bean** référencé est une dépendance du **bean** dont la propriété doit être appliquée, et il est initialisé à la demande quand nécessaire, avant que la propriété ne soit appliquée. (Si le collaborateur est un **bean** singleton, il pourrait déjà être initialisé par le conteneur.) Toutes les références sont finalement des références à un autre objet. Le **scoping** et la validation dépendent de l'ID ou du nom de l'autre objet que vous spécifiez à travers l'attribut `bean` ou `parent`.

Spécifier le **bean** cible via l'attribut `bean` du tag `<ref/>` est la forme la plus courante et permet de créer une référence à n'importe quel **bean** dans le même conteneur ou dans un conteneur parent, indépendamment du fait qu'il se trouve dans le même fichier XML. La valeur de l'attribut `bean` peut être la même que l'attribut `id` du **bean** cible ou correspondre à l'une des valeurs de l'attribut `name` du **bean** cible. L'exemple suivant montre comment utiliser l'élément `ref` :

```xml
<ref bean="someBean"/>
```

Spécifier le **bean** cible via l'attribut `parent` crée une référence à un autre **bean** situé dans un conteneur parent du conteneur courant. La valeur de l'attribut `parent` peut correspondre à l'attribut `id` du **bean** cible ou à l'une des valeurs de l'attribut `name` du **bean** cible. Le **bean** cible doit se trouver dans un conteneur parent du conteneur actuel. Vous devriez utiliser cette variante principalement lorsque vous disposez d'une hiérarchie de conteneurs et que vous souhaitez envelopper un **bean** existant dans le conteneur parent à l'aide d'un **proxy** qui porte le même nom que le **bean** parent. Les deux extraits suivants montrent comment utiliser l'attribut `parent` :

```xml
<!-- dans le conteneur parent -->
<bean id="accountService" class="com.something.SimpleAccountService">
	<!-- insérer les dépendances nécessaires ici -->
</bean>
```

```xml
<!-- dans le conteneur enfant (descendant), le nom du bean est le même que dans le parent -->
<bean id="accountService"
	class="org.springframework.aop.framework.ProxyFactoryBean">
	<property name="target">
		<ref parent="accountService"/> <!-- notez comment on fait référence au bean parent -->
	</property>
	<!-- insérer les autres configurations et dépendances nécessaires ici -->
</bean>
```

> **NOTE**
>
> L'attribut `local` sur l'élément `ref` n'est plus supporté dans les schémas XSD des **beans** 4.0, puisqu'il n'apportait plus de valeur ajoutée à une référence régulière avec `bean`. Changez vos références existantes de `ref local` à `ref bean` lors de la migration vers le schéma 4.0.


## _**Beans Internes**_

Un élément `<bean/>` à l'intérieur des éléments `<property/>` et `<constructor-arg/>` définit un **bean** interne, comme le montre l'exemple suivant :

<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>

Une définition de **bean** interne ne nécessite pas d'ID ou de nom. S'ils sont spécifiés, le conteneur ne les utilise pas en tant qu'identifiant. Le conteneur ignore également le flag `scope` à la création, parce que les **beans** internes sont toujours anonymes et sont toujours créés avec le **bean** externe. Il n'est pas possible d'accéder à un **bean** interne de façon indépendante ou de les injecter en tant que collaborateurs dans d'autres **beans**, hormis le **bean** encapsulant.

En tant que cas limite, il est possible de recevoir un **callback** de destruction depuis un **scope** personnalisé — par exemple, pour un **request scope** d'un **bean** interne contenu dans un **bean** singleton. La création d'un **bean** interne est liée à son **bean** contenant, mais les **callbacks** de destruction le laissent participer dans le cycle de vie du **scope** **request**.  
Cela n'est pas un scénario commun. Les **beans** internes partagent généralement simplement le même **scope** que le **bean** englobant.

## _**Collections**_

Les éléments `<list/>`, `<set/>`, `<map/>` et `<props/>` appliquent les propriétés et arguments d'une `Collection` Java de type `List`, `Set`, `Map` et `Properties`, respectivement. L'exemple suivant montre comment les utiliser:

```xml
<bean id="moreComplexObject" class="example.ComplexObject">
	<!-- results in a setAdminEmails(java.util.Properties) call -->
	<property name="adminEmails">
		<props>
			<prop key="administrator">administrator@example.org</prop>
			<prop key="support">support@example.org</prop>
			<prop key="development">development@example.org</prop>
		</props>
	</property>
	<!-- results in a setSomeList(java.util.List) call -->
	<property name="someList">
		<list>
			<value>a list element followed by a reference</value>
			<ref bean="myDataSource" />
		</list>
	</property>
	<!-- results in a setSomeMap(java.util.Map) call -->
	<property name="someMap">
		<map>
			<entry key="an entry" value="just some string"/>
			<entry key="a ref" value-ref="myDataSource"/>
		</map>
	</property>
	<!-- results in a setSomeSet(java.util.Set) call -->
	<property name="someSet">
		<set>
			<value>just some string</value>
			<ref bean="myDataSource" />
		</set>
	</property>
</bean>
```

La contenu de la clé ou de la valeur d'une **map**, ou le contenu d'un **set**, peut aussi être n'importe quel type suivant:

```xml
bean | ref | idref | list | set | map | props | value | null
```

### Fusion de collections

Le conteneur **Spring** supporte également la fusion de collections. Un développeur d'application peut définir un élément parent `<list/>`, `<map/>`, `<set/>` ou `<props/>` et avoir des éléments enfants `<list/>`, `<map/>`, `<set/>` ou `<props/>` hérités, avec des valeurs réécrites depuis une collection parente. Autrement dit, les valeurs de la collection enfant sont le résultat de la fusion des éléments de la collection parente et enfant, les éléments de la collection enfant réécrivant les valeurs présentes dans la collection parente.

Cette section sur la fusion traite du mécanisme parent-enfant des **beans**. Les lecteurs qui ne sont pas familiers avec les définitions de **bean** parent et enfant pourraient vouloir lire [l'article correspondant](#) avant de continuer.

L'exemple suivant démontre une fusion de collections :

```xml
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- La fusion est spécifiée sur la définition de la collection enfant -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
</beans>
```

Remarquez l'utilisation de l'attribut `merge="true"` dans l'élément `<props/>` de la propriété `adminEmails` de la définition du **bean** `child`. Lorsque le **bean** `child` est résolu et instancié par le conteneur, l'instance résultante possède une collection `adminEmails` de type `Properties` contenant le résultat de la fusion de la collection enfant `adminEmails` avec celle de la collection parente `adminEmails`.

L'exemple suivant montre le résultat :

```
administrator=administrator@example.com
sales=sales@example.com
support=support@example.co.uk
```

La valeur de la collection `Properties` de l'enfant hérite de toutes les propriétés du tag `<props/>` du parent, et la valeur de `support` de l'enfant réécrit celle de la collection parente.

Ce comportement de fusion s'applique de façon similaire aux collections de type `<list/>`, `<map/>` et `<set/>`. Dans le cas spécifique d'un élément `<list/>`, les sémantiques associées au type de collection `List` (c'est-à-dire, la notion d'une collection ordonnée d'éléments) sont maintenues : les valeurs parentes précèdent toutes les valeurs de la liste enfant. Dans le cas des collections de type `Map`, `Set` et `Properties`, l'ordre n'a pas d'importance. Ainsi, les sémantiques sans ordre sont de fait pour les types de collection qui sont initialement héritées par les implémentations de `Map`, `Set` et `Properties` que le conteneur utilise en interne.


### _**Limitation de la fusion de collections**_

Vous ne pouvez pas fusionner différents types de collections (tels qu’une `Map` et une `List`). Si vous essayez de le faire, une `Exception` est levée. L’attribut `merge` doit être spécifié dans la définition de l’élément enfant le plus bas dans la hiérarchie d’héritage. Spécifier l’attribut `merge` sur une définition de collection parente est redondant et ne produit pas la fusion attendue.


### _**Collections fortement typées**_

Grâce au support des types génériques de Java, vous pouvez utiliser des collections fortement typées. Cela signifie qu’il est possible de déclarer une `Collection` de telle façon qu’elle ne contienne (par exemple) que des éléments `String`. Si vous utilisez _Spring_ pour injecter une `Collection` fortement typée dans un **bean**, vous pouvez tirer parti du mécanisme de conversion de types de _Spring_ afin que les éléments de votre `Collection` fortement typée soient convertis vers le type approprié avant d’être ajoutées à la `Collection`. La classe Java suivante et la définition de **bean** montrent comment procéder :

```java
public class SomeClass {

	private Map<String, Float> accounts;

	public void setAccounts(Map<String, Float> accounts) {
		this.accounts = accounts;
	}
}
```xml
<beans>
	<bean id="something" class="x.y.SomeClass">
		<property name="accounts">
			<map>
				<entry key="one" value="9.99"/>
				<entry key="two" value="2.75"/>
				<entry key="six" value="3.99"/>
			</map>
		</property>
	</bean>
</beans>
```

Lorsque la propriété accounts du bean something est préparée pour l’injection, les informations génériques concernant le type des éléments de la Map<String, Float> fortement typée sont disponibles par réflexion. Ainsi, l’infrastructure de conversion de types de Spring reconnaît les différentes valeurs comme étant de type Float, et les valeurs String (9.99, 2.75 et 3.99) sont converties vers le type réel Float.`Float`.



## _**Valeurs String nulles et vides**_

_Spring_ traite les arguments vides des propriétés et autres éléments comme des `String` vides. La configuration XML suivante applique la propriété `email` avec la valeur d’un `String` vide (`""`) :

```xml
<bean class="ExampleBean">
	<property name="email" value=""/>
</bean>
```

L’exemple précédent est équivalent au code Java suivant :

```java
exampleBean.setEmail("");
```

L’élément <null/> gère les valeurs nulles. L’exemple suivant l’illustre :

```xml
<bean class="ExampleBean">
	<property name="email">
		<null/>
	</property>
</bean>
```

La configuration précédente est équivalente au code Java suivant :

exampleBean.setEmail(null);


## _**Raccourcis XML avec l'espace de noms p**_

L'espace de noms **p** vous permet d'utiliser l'attribut `bean` d'un élément (au lieu des éléments imbriqués `<property/>`) pour définir les valeurs de vos propriétés collaborantes `beans`, ou les deux.

_Spring_ supporte une configuration extensible grâce aux [namespaces]() basés sur une **Schema Definition** XML. Le format de configuration des **beans** discuté dans ce chapitre est défini dans un document **XML Schema**. Cependant, l'espace de noms **p** n'est pas défini dans un fichier **XSD** et existe uniquement dans le cœur de _Spring_.

L'exemple suivant montre deux codes XML (le premier utilise le format standard XML et le second utilise l'espace de noms **p**) qui aboutissent au même résultat :

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		https://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean name="classic" class="com.example.ExampleBean">
		<property name="email" value="someone@somewhere.com"/>
	</bean>

	<bean name="p-namespace" class="com.example.ExampleBean"
		p:email="someone@somewhere.com"/>
</beans>
```

L'exemple montre un attribut appelé email dans l'espace de noms p dans la définition du bean. Cela indique à Spring d'inclure une déclaration de propriété. Comme indiqué précédemment, l'espace de noms p ne possède pas de Schema Definition, mais vous pouvez appliquer le nom de l'attribut au nom de la propriété.

L'exemple suivant inclut deux définitions de beans supplémentaires, qui sont toutes deux des références à un autre bean :

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		https://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean name="john-classic" class="com.example.Person">
		<property name="name" value="John Doe"/>
		<property name="spouse" ref="jane"/>
	</bean>

	<bean name="john-modern"
		class="com.example.Person"
		p:name="John Doe"
		p:spouse-ref="jane"/>

	<bean name="jane" class="com.example.Person">
		<property name="name" value="Jane Doe"/>
	</bean>
</beans>
```

Cet exemple n'inclut pas seulement une valeur de propriété utilisant l'espace de noms p, mais aussi un format spécial pour déclarer les références aux propriétés. Alors que la première définition de bean utilise <property name="spouse" ref="jane"/> pour créer une référence du bean john vers le bean jane, la seconde définition de bean utilise p:spouse-ref="jane" comme attribut pour faire exactement la même chose. Dans ce cas, spouse est le nom de la propriété, tandis que la partie -ref indique qu'il ne s'agit pas d'une valeur brute, mais d'une référence à un autre bean.



> **NOTE**
> L'espace de noms **p** n'est pas aussi flexible que le format standard XML. Par exemple, le format pour déclarer des références à des propriétés entre en conflit avec les propriétés dont le nom se termine par Ref, alors que le format standard XML fonctionne parfaitement.  
> Nous recommandons de choisir votre approche avec soin et de communiquer cela aux membres de votre équipe afin d'éviter de produire des documents XML utilisant les trois approches en même temps.



## Raccourcis XML avec l'espace de noms `c`

De façon similaire aux [raccourcis XML avec l'espace de noms `p`](#raccourcis-xml-avec-lespace-de-noms-p), l'espace de noms **`c`**, introduit dans **Spring 3.1**, permet d'utiliser des attributs en ligne pour configurer les arguments d'un constructeur, plutôt que des éléments imbriqués `constructor-arg`.

L'exemple suivant utilise l'espace de noms `c:` pour faire la même chose que dans la section [Injection de dépendances par constructeur](./Dependances%20et%20Configuration%20en%20Detail.md) :

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns\:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns\:c="http://www.springframework.org/schema/c"
    xsi\:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="beanTwo" class="x.y.ThingTwo"/>
    <bean id="beanThree" class="x.y.ThingThree"/>

    <!-- Déclaration traditionnelle avec noms d'arguments optionnels -->
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg name="thingTwo" ref="beanTwo"/>
        <constructor-arg name="thingThree" ref="beanThree"/>
        <constructor-arg name="email" value="something@somewhere.com"/>
    </bean>

    <!-- Déclaration avec l'espace de noms c: et noms d'arguments -->
    <bean id="beanOne" class="x.y.ThingOne" c\:thingTwo-ref="beanTwo"
        c\:thingThree-ref="beanThree" c\:email="something@somewhere.com"/>

</beans>
```

L'espace de noms **`c`** utilise la même convention que **`p`** (un suffixe `-ref` pour les références aux beans) afin d'appliquer les arguments du constructeur par leur nom.
De même, il doit être déclaré dans un fichier XML, bien qu'il ne soit pas défini dans un schéma XSD (il est géré directement par le cœur de Spring).

Pour les rares cas où le nom des arguments du constructeur n'est pas disponible (par exemple, si le bytecode a été compilé sans l'option `-parameters`), il est possible de recourir aux arguments par indice, comme suit :

```xml
<!-- Déclaration avec l'espace de noms c: et indices -->
<bean id="beanOne" class="x.y.ThingOne" c:_0-ref="beanTwo" c:_1-ref="beanThree"
    c:_2="something@somewhere.com"/>
```

> **NOTE**
>
> En raison de la grammaire XML, la notation par indice nécessite un `_` précédant le numéro, car les noms d'attributs XML ne peuvent pas commencer par un chiffre (même si certains IDE le permettent).
> Une notation par indice est également disponible pour les éléments `<constructor-arg>`, mais elle est rarement utilisée, l'ordre de déclaration étant généralement suffisant.

En pratique, le [mécanisme d'injection de dépendances par constructeur](./Injection%20de%20Dépendances.md) est assez efficace pour reconnaître les arguments. À moins d'une nécessité absolue, il est donc recommandé d'utiliser la notation par nom dans votre configuration.


## _**Noms de propriétés calculées**_

Vous pouvez calculer ou imbriquer des noms de propriétés lorsque vous appliquez les propriétés de vos **beans**, tant que tous les composants du chemin, **excepté** le nom final de la propriété, ne sont pas `null`. Considérez la définition de **bean** suivante :

```xml
<bean id="something" class="things.ThingOne">
	<property name="fred.bob.sammy" value="123" />
</bean>
```

Le **bean** `something` possède une propriété `fred`, qui possède une propriété `bob`, qui possède une propriété `sammy`, et cette propriété finale `sammy` va être appliquée à une valeur de `123`. Afin que cela fonctionne, la propriété `fred` de `something` et la propriété `bob` de `fred` ne doivent pas être `null` après que le **bean** soit construit. Sinon, une exception `NullPointerException` est levée.
