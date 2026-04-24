{
    page:13,
    from:"https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-method-injection.html",
    update:"2026-09-02",
    by:["Arthur Leroux"],
}

# _**Injection par Méthode**_

Dans les cas les plus courants, la plupart des **beans** dans le conteneur sont des [singletons](). Quand un **singleton** a besoin de collaborer avec un autre **singleton bean** ou qu’un **non-singleton** a besoin de collaborer avec un autre **non-singleton bean**, vous maniez généralement la dépendance en définissant un **bean** comme étant une propriété d'un autre. Un problème apparaît quand les cycles de vie des **beans** sont différents. Supposons qu’un **bean A** ait besoin d'utiliser un **non-singleton** (**prototype**) **bean B**, peut-être à chaque invocation de méthode sur **A**. Le conteneur crée le **singleton bean A** seulement une fois, et obtient ainsi une seule opportunité d'initialiser les propriétés. Le conteneur ne peut pas fournir au **bean A** une nouvelle instance du **bean B** à chaque fois que cela est requis.

Une solution consiste à renoncer partiellement à l'inversion de contrôle. Vous pouvez [rendre un bean A conscient du conteneur]() en implémentant l'interface `ApplicationContextAware`, et en [appelant getBean("B") sur le conteneur]() pour demander une (typiquement nouvelle) instance du **bean B** à chaque fois que le **bean A** en a besoin. L'exemple suivant montre cette approche :

```java
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

/**
 * A class that uses a stateful Command-style class to perform
 * some processing.
 */
public class CommandManager implements ApplicationContextAware {

	private ApplicationContext applicationContext;

	public Object process(Map commandState) {
		// grab a new instance of the appropriate Command
		Command command = createCommand();
		// set the state on the (hopefully brand new) Command instance
		command.setState(commandState);
		return command.execute();
	}

	protected Command createCommand() {
		// notice the Spring API dependency!
		return this.applicationContext.getBean("command", Command.class);
	}

	public void setApplicationContext(
			ApplicationContext applicationContext) throws BeansException {
		this.applicationContext = applicationContext;
	}
}
```

L'exemple précédent n'est pas désirable, car le code métier est conscient et couplé au framework _Spring_. L'injection par méthode, une fonctionnalité quelque peu avancée du conteneur d'inversion de contrôle de _Spring_, vous permet de gérer ce cas proprement.

Vous pouvez lire plus à propos des motivations pour l'injection par méthodes dans [l'entrée de ce blog](https://spring.io/blog/2004/08/06/method-injection).

## _**Injection par recherche de méthode**_

L'injection par recherche de méthode est la capacité du conteneur à réecrire sur les **beans** gérés par le conteneur et retourner le résultat de la recherche pour un autre **bean** nommé dans le conteneur. La recherche implique typiquement un **bean prototype**, tel que le scénario décrit dans [la section précédente](#injection-par-méthode). Le framework _Spring_ implémente cette injection par méthode en utilisant la génération de **bytecode** depuis la bibliothèque CGLIB pour générer dynamiquement une sous-classe qui réecrrit la méthode.

> **NOTE**
>
> * Pour que cette sous-classe dynamique fonctionne, la **class** que le conteneur à **beans** de _Spring_ sous-classe ne doit pas être `final` et la méthode qui doit être réecrite ne doit pas être `final` non plus.
> * Les tests unitaires d'une **class** qui ont une méthode `abstract` nécessitent que vous sous-classiez la **class** vous même et que vous fournissiez une implémentation stub de la méthode `abstract`.
> * Une limitation ultérieure est que la recherche par méthodes ne fonctionne pas avec les **factory** méthodes et en particulier avec les méthodes `@Bean` dans les **classes** de configuration, dans la mesure où, dans ce cas, le conteneur n'est pas en charge de la création d'une instance et donc ne peut pas créer une sous-classe à l'exécution à la volé.

Dans le cas d'une **class** `CommandManager` dans l'exemple précédent, le conteneur _Spring_ réecrit dynamiquement l'implémentation de la méthode `createCommand()`. La **class** `CommandManager` n'a aucune dépendance avec _Spring_, comme la réecriture de l'exemple le montre :

```java
package fiona.apple;

// no more Spring imports!

public abstract class CommandManager {

    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```

Dans la **class** cliente qui contient la méthode à être injectée (`CommandManager` dans ce cas), la méthode qui doit être injectée nécessite une signature du format suivant :

```xml
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```

Si la méthode est `abstract`, la sous-classe dynamiquement générée implémente la méthode. Sinon, la sous-classe dynamiquement générée réecrit la méthode concrète définie dans la **class** originale. Considérez l'exemple suivant :

```xml
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandManager uses myCommand prototype bean -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="myCommand"/>
</bean>
```

Le **bean** identifié comme `commandManager` appelle sa propre méthode `createCommand()` dès lors qu'il a besoin d'une nouvelle instance du **bean** `myCommand`. Vous devez être attentif de déployer le **bean** `myCommand` comme un prototype si c'est ce qui est nécessaire. Si c'est un [singleton](), la même instance du **bean** `myCommand` is retournée à chaque fois.

Aussi, dans le format orientée annotation et le modèle des composants, vous pouvez déclarer une recherche par méthode à travers l'annotation `@Lookup`, comme le montre l'exemple suivant :

```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup("myCommand")
    protected abstract Command createCommand();
}
```

```kotlin
abstract class CommandManager {

    fun process(commandState: Any): Any {
        val command = createCommand()
        command.state = commandState
        return command.execute()
    }

    @Lookup("myCommand")
    protected abstract fun createCommand(): Command
}
```

Ou, de façon plus idiomatique, vous pouvez vous reposer sur le **bean** cible pour résoudre le type de retour declaré de la méthode de recherche :

```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup
    protected abstract Command createCommand();
}
```

```kotlin
abstract class CommandManager {

    fun process(commandState: Any): Any {
        val command = createCommand()
        command.state = commandState
        return command.execute()
    }

    @Lookup
    protected abstract fun createCommand(): Command
}
```

> **TIP**
>
> Un autre moyen d'accéder différents **beans** cibles selon leur scope est avec un point d'injection `ObjectFactory/Provider`. Voir [Scoped Beans as Dependencies]().
>
> Vous pourriez trouver le `ServiceLocatorFactoryBean` (dans le package `org.springframework.beans.factory.config`) être utile.


## _**Remplacement arbitaire de méthode**_

Une façon moins utile d'injection par méthode que l'injection par recherche de méthode est la capacité de remplacer des méthodes arbitraires dans un **bean** géré par une autre implémentation de méthode. Vous pouvez sans risque passer le reste de cette section à moins que vous n'ayez besoin de cette fonctionnalité.

Avec une configuration des métadonnées orientée XML, vous pouvez utiliser l'élément `replaced-method` pour remplacer l'implémentation d'une méthode existante par une autre, pour un **bean** déployé. Considérez la **class** suivante, qui possède une méthode appelée `computeValue` que nous voulons réécrire :

```java
public class MyValueCalculator {

	public String computeValue(String input) {
		// some real code...
	}

	// some other methods...
}
```

```kotlin
class MyValueCalculator {

	fun computeValue(input: String): String {
		// some real code...
	}

	// some other methods...
}
```

Une **class** qui implémente l'interface `org.springframework.beans.factory.support.MethodReplacer` fournit la nouvelle définition de la méthode, comme le montre l'exemple suivant :

```java
/**
 * meant to be used to override the existing computeValue(String)
 * implementation in MyValueCalculator
 */
public class ReplacementComputeValue implements MethodReplacer {

	public Object reimplement(Object o, Method m, Object[] args) throws Throwable {
		// get the input value, work with it, and return a computed result
		String input = (String) args[0];
		...
		return ...;
	}
}
```

```kotlin
/**
 * meant to be used to override the existing computeValue(String)
 * implementation in MyValueCalculator
 */
class ReplacementComputeValue : MethodReplacer {

	override fun reimplement(obj: Any, method: Method, args: Array<out Any>): Any {
		// get the input value, work with it, and return a computed result
		val input = args[0] as String
		...
		return ...
	}
}
```

La définition du **bean** pour déployer la **class** originale et spécifier la méthode à réécrire devrait ressembler à l'exemple suivant :

```xml
<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
	<!-- arbitrary method replacement -->
	<replaced-method name="computeValue" replacer="replacementComputeValue">
		<arg-type>String</arg-type>
	</replaced-method>
</bean>

<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>
```

Vous pouvez utiliser un ou plusieurs éléments `<arg-type/>` dans l'élément `<replaced-method/>` pour indiquer la signature de la méthode qui doit être réécrite. La signature des arguments est nécessaire seulement si la méthode est surchargée et que plusieurs variantes existent dans la **class**.

Pour faciliter l'usage, le type `String` pour un argument peut être une sous-chaîne du type pleinement qualifié. Par exemple, `java.lang.String` fonctionne avec :

```java
java.lang.String
String
Str
```

Parce que le nombre d'arguments est souvent suffisant pour distinguer plusieurs choix possibles, ce raccourci peut vous éviter beaucoup d'écriture, en vous permettant d'utiliser seulement la chaîne la plus courte qui correspond à un type d'argument.

