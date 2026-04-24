{
    page:12,
    from:"https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-autowire.html",
    update:"2026-03-02",
    by:["Arthur Leroux"],
}

# _**Autocâblage des Collaborateurs**_

Le conteneur _Spring_ peut autocâbler les relations entre les **beans** collaborants. Vous pouvez laisser _Spring_ résoudre les collaborateurs (les autres **beans**) automatiquement pour vos **beans** en inspectant le contenu de l'`ApplicationContext`. L'autocâblage a les avantages suivants :

* L'autocâblage peut réduire significativement le besoin de spécifier les propriétés ou les arguments du constructeur. (D'autres mécanismes comme le **bean** template, [discutés plus loin dans ce chapitre]() sont aussi de valeur en ce sens.)
* L'autocâblage peut mettre à jour une configuration en même temps que vos objets évoluent. Par exemple, si vous avez besoin d'ajouter une dépendance à une **classe**, cette dépendance peut être satisfaite automatiquement sans avoir besoin de modifier la configuration. Ainsi, l'autocâblage peut être particulièrement utile durant le développement, sans renier l'option de basculer explicitement le câblage quand le code devient plus stable.

Lors de l'utilisation d'une configuration des métadonnées orientée XML (voir [Dependency Injection](./Injection%20de%20Dépendances.md)), vous pouvez spécifier le mode d'autocâblage pour la définition d'un **bean** avec l'attribut `autowire` d'un élément `<bean/>`. La fonctionnalité d'autocâblage possède quatre modes. Vous spécifiez l'autocâblage par **bean** et pouvez ainsi choisir quel mode utiliser pour l'autocâblage. La table suivante décrit les quatre modes :

|Mode|Explication|
|---|---|
|`no`|(Par défaut) Pas d'autocâblage. Les références des **beans** doivent être définies par les éléments `ref`. Changer le paramétrage par défaut n'est pas recommandé pour des déploiements plus larges, parce que spécifier les collaborateurs donne un plus grand contrôle et plus de clarté. Dans une certaine mesure, cela documente la structure d'un système.|
|`byName`|L'autocâblage par nom de propriété. _Spring_ recherche un **bean** avec le même nom que la propriété qui a besoin d'être injectée. Par exemple, si une définition de **bean** est appliquée pour l'autocâblage par nom et contient une propriété `master` (c'est-à-dire qu'il possède une méthode `setMaster(...)`), _Spring_ recherche une définition de **bean** nommée `master` et l'utilise pour appliquer la propriété.|
|`byType`|Permet d'autocâbler une propriété si exactement un **bean** du type de la propriété existe dans le conteneur. Si plus d'un bean existe, une exception fatale est lancée, indiquant que vous ne pouvez pas utiliser l'autocâblage `byType` pour ce **bean**. S'il n'existe pas de **beans** correspondant, rien ne se passe (la propriété n'est pas appliquée).|
|`constructor`|De manière analogue à `byType`, mais appliqué aux arguments du constructeur. S'il n'y a pas exactement un seul **bean** du type de l'argument du constructeur dans le conteneur, une erreur fatale est levée.|

Avec les modes d'autocâblage `byType` et `constructor`, vous pouvez câbler des tableaux et des collections typées. Dans de tels cas, tous les candidats à l'autocâblage dans le conteneur qui correspondent au type attendu sont fournis pour satisfaire la dépendance. Vous pouvez autocâbler les instances de `Map` fortement typées si le type de clé attendu est `String`. La valeur des instances d'un autocâblage d'une `Map` consiste en toutes les instances des **beans** qui correspondent au type attendu, et les clés des instances d'une `Map` correspondent au nom des **beans**.

## _**Limitations et désavantages de l'autocâblage**_

L'autocâblage fonctionne bien lorsqu'il est utilisé de manière consistante à travers le projet. Si l'autocâblage n'est pas utilisé en général, il pourrait être confus de l'utiliser pour câbler seulement une ou deux définitions.

Considérez les limitations et désavantages de l'autocâblage :

* Les dépendances explicites dans les configurations `property` et `constructor-arg` réécrivent toujours l'autocâblage. Vous ne pouvez pas autocâbler les propriétés simples comme les types primitifs, `String` et `Class` (et les tableaux de telles propriétés). Cette limitation fait partie de la conception.
* L'autocâblage est moins exact que le câblage explicite. Bien que, comme indiqué dans le tableau précédent, _Spring_ fasse attention à ne pas deviner dans le cas d'une ambiguïté qui pourrait avoir des résultats inattendus, les relations entre vos objets gérés par _Spring_ ne sont plus explicitement documentées.
* Les informations de câblage peuvent ne pas être disponibles pour les outils qui peuvent générer de la documentation depuis un conteneur _Spring_.
* Plusieurs définitions de **beans** dans le conteneur peuvent correspondre au type spécifié par la méthode **setter** ou à l'argument du constructeur pour être autocâblées. Pour les tableaux, les collections ou les instances de `Map`, cela n'est pas nécessairement un problème. Cependant, pour les dépendances qui attendent une seule valeur, cette ambiguïté n'est pas résolue arbitrairement. Si aucune définition de **bean** n'est disponible, une exception est levée.

Dans ce dernier scénario, vous avez plusieurs options :

* Abandonner l'autocâblage en faveur d'un câblage explicite.
* Éviter l'autocâblage pour la définition d'un **bean** en appliquant l'attribut `autowire-candidate` à `false`, comme décrit dans la [section suivante]().
* Désigner une seule définition de **bean** comme candidate principale en appliquant l'attribut `primary` à `true` dans ses éléments `<bean/>`.
* Implémenter un contrôle plus granulaire disponible avec la configuration par annotation, comme décrit dans le chapitre [Annotation-based Container Configuration]().

## _**Exclusion d'un Bean de l'autocâblage**_

Pour chaque **bean**, vous pouvez exclure un **bean** de l'autocâblage. Dans le format XML de _Spring_, appliquez l'attribut `autowire-candidate` de l'élément `<bean/>` à `false` ; avec l'annotation `@Bean`, l'attribut est appelé `autowireCandidate`. Le conteneur rend spécifiquement la définition de ce **bean** indisponible pour l'infrastructure d'autocâblage, incluant les points d'injection orientés annotation telle que [@Autowired]().

> **NOTE**  
> L'attribut `autowire-candidate` est conçu pour affecter seulement l'autocâblage par type. Cela n'affecte pas explicitement les références par nom, lesquelles sont prises en compte même si le **bean** spécifié n'est pas marqué comme candidat à l'autocâblage. Cela a pour conséquence que l'autocâblage par nom est malgré tout injecté si un **bean** correspondant par nom existe.

Vous pouvez limiter l'autocâblage des candidats en fonction d'une correspondance d'un pattern sur les noms des **beans**. Le super-élément `<beans/>` accepte un ou plusieurs patterns dans l'attribut `default-autowire-candidates`. Par exemple, pour limiter le statut des candidats à l'autocâblage à tous les **beans** dont le nom se termine par `Repository`, fournissez la valeur `*Repository`. Pour fournir plusieurs patterns, définissez-les dans une liste séparée par des virgules. Une valeur explicite `true` ou `false` pour l'attribut `autowire-candidate` d'un **bean** a toujours la priorité. Pour de tels **beans**, les règles de correspondance par pattern ne s'appliquent pas.

Ces techniques sont utiles pour les **beans** que vous ne souhaitez jamais injecter dans d'autres **beans** par autocâblage. Cela ne veut pas dire qu'un **bean** exclu ne peut pas lui-même être configuré en utilisant l'autocâblage. Plutôt, le **bean** lui-même n'est pas candidat pour l'autocâblage d'autres **beans**.

> **NOTE**  
> Depuis la version 6.2, les méthodes annotées `@Bean` supportent deux variantes pour le flag des candidats à l'autocâblage : `autowireCandidate` et `defaultCandidate`.  
>
> Lors de l'utilisation des [qualifiers](), un **bean** marqué avec `defaultCandidate=false` n'est disponible que pour les injections où un qualifier supplémentaire est présent.  
> Cela est utile pour des délégués restreints, qui sont destinés à être injectables dans certaines zones mais qui ne doivent pas interférer avec les **beans** du même type ailleurs. De tels **beans** ne seront jamais injectés uniquement par type, mais par type en combinaison avec un qualifier spécifique.  
>
> En contraste, `autowireCandidate=false` se comporte exactement comme l'attribut `autowire-candidate` indiqué plus haut : un tel **bean** ne sera jamais injecté par type.