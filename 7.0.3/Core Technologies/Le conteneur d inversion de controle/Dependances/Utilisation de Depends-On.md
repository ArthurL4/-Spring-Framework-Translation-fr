{
    page:10,
    from:"https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-dependson.html",
    update:"2026-03-01",
    by:["Arthur Leroux"],
}

# _**Utiliser Depends-On**_

Si un **bean** est une dépendance d'un autre **bean**, cela veut généralement dire qu'un **bean** est utilisé comme propriété d'un autre. Typiquement, vous accomplissez cela avec [l'élément `<ref/>`](./Injection%20de%20Dépendances.md) dans une configuration orientée XML ou bien à travers [l'autowiring].

Cependant, parfois les dépendances entre les **beans** sont moins directes. Un exemple est quand un initialisateur **static** dans une **class** a besoin d'être déclenché, comme l'enregistrement d'un driver de base de données. L'attribut `depends-on` ou l'annotation `@DependsOn` peut explicitement forcer un ou plusieurs **beans** à être initialisés avant que le **bean** utilisant cet élément soit initialisé. L'exemple suivant utilise l'attribut `depends-on` pour exprimer une dépendance sur un seul **bean** :

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```

Pour exprimer une dépendance sur plusieurs **beans**, fournissez une liste de noms de **beans** comme valeur de l'attribut `depends-on` (les virgules, espaces et points-virgules sont des délimiteurs valides) :

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
	<property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

> **NOTE**
> L'attribut `depends-on` peut spécifier à la fois un ordre d'initialisation de la dépendance et, dans le cas d'un **bean** [singleton](), un ordre de destruction de dépendance. Les **beans** dépendants qui définissent une relation `depends-on` avec un **bean** donné sont détruits en premier, avant que le **bean** courant ne soit lui-même détruit. Ainsi, `depends-on` peut aussi contrôler l'ordre d'interruption.
