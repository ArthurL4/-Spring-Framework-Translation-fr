{
    page: 16,
    from: "https://docs.spring.io/spring-framework/reference/core/beans/child-bean-definitions.html",
    update: "2026-03-30",
    by: ["Arthur Leroux"],
}

# _*Héritage de la définition d'un Bean*_

Une définition de **bean** peut contenir de nombreuses informations de configuration, incluant les arguments du constructeur, les valeurs des propriétés, et des informations spécifiques au conteneur, telles que la méthode d'initialisation, le nom d'une méthode **static factory**, et plus encore. Une définition de **bean** enfant hérite des données de configuration depuis la définition d'un parent. La définition de l'enfant peut réécrire certaines valeurs ou en ajouter d'autres au besoin. Utiliser des définitions de **beans** parent et enfant peut réduire beaucoup de duplication. Effectivement, c'est une forme de modèle.

Si vous travaillez avec une interface `ApplicationContext` programmatiquement, les définitions de **beans** enfants sont représentées par la **classe** `ChildBeanDefinition`. La plupart des utilisateurs ne travaillent pas avec elles à ce niveau. Plutôt, ils configurent les définitions des **beans** de manière déclarative dans une **classe** telle que `ClassPathXmlApplicationContext`.

Lorsque vous utilisez une configuration des métadonnées orientée XML, vous pouvez indiquer une définition de **bean** enfant en utilisant l'attribut `parent`, en spécifiant le **bean** parent comme valeur de cet attribut. L'exemple suivant montre comment faire :

```xml
<bean id="inheritedTestBean" abstract="true"
    class="org.springframework.beans.TestBean">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass"
    class="org.springframework.beans.DerivedTestBean"
    parent="inheritedTestBean" init-method="initialize">
    <property name="name" value="override"/>
    <!-- la valeur de la propriété age de 1 sera héritée du parent -->
</bean>
```

Une définition de **bean** enfant utilise la **classe** du **bean** depuis la définition du parent si aucune n’est spécifiée, mais peut aussi la réécrire. Dans ce dernier cas, la **classe** du **bean** enfant doit être compatible avec le parent (c'est-à-dire qu'elle doit accepter les valeurs des propriétés du parent).

Une définition de **bean** enfant hérite de la portée, des valeurs des arguments du constructeur, des valeurs des propriétés, et des réécritures des méthodes depuis le parent, avec l'option d'ajouter de nouvelles valeurs. Toute portée, méthode d'initialisation, méthode de destruction ou méthode `static` **factory** configurée dans la définition enfant réécrit la configuration correspondante du parent.

Les autres configurations sont toujours prises depuis la définition enfant : `depends-on`, mode d'autowiring, `dependency-check`, `singleton` et `lazy-init`.

L'exemple précédent marque explicitement la définition du **bean** parent comme **abstract** en utilisant l'attribut `abstract`. Si la définition parente ne spécifie pas de **classe**, marquer explicitement la définition du **bean** parent comme `abstract` est requis, comme le montre l'exemple suivant :

```xml
<bean id="inheritedTestBeanWithoutClass" abstract="true">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithClass" class="org.springframework.beans.DerivedTestBean"
    parent="inheritedTestBeanWithoutClass" init-method="initialize">
    <property name="name" value="override"/>
    <!-- age héritera de la valeur 1 depuis la définition du bean parent -->
</bean>
```

Le **bean** parent ne peut être initialisé par lui-même car il est incomplet et est explicitement marqué comme `abstract`. Lorsqu'une définition est `abstract`, elle n’est utilisable que comme modèle pour des définitions enfants. Essayer d'utiliser un tel **bean** parent `abstract` directement, en le référençant comme propriété d’un autre **bean** ou en appelant explicitement `getBean()` avec l'ID du **bean** parent retourne une erreur. De façon similaire, la méthode interne du conteneur `preInstantiateSingletons()` ignore les définitions de **bean** marquées comme **abstract**.

> **NOTE**
>
> L'`ApplicationContext` préinstancie tous les singletons par défaut. Ainsi, si vous avez une définition de **bean** (parente) destinée uniquement comme modèle et que cette définition spécifie une **classe**, vous devez vous assurer d’appliquer l'attribut _abstract_ à `true`. Sinon, l'ApplicationContext tentera de pré-instancier le **bean** `abstract`.