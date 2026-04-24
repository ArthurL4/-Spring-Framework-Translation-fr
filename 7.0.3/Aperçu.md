{
    page:1,
    from:"https://docs.spring.io/spring-framework/reference/overview.html",
    update:"2026-02-06",
    by:["Arthur Leroux"],
}
# __*Aperçu du Framework Spring*__

_Spring_ rend la création d'applications d'entreprise facile. Il fournit tout ce dont le développeur a besoin pour exploiter le langage Java dans un environnement d'entreprise, incluant un support pour _Groovy_ et _Kotlin_ comme langages alternatifs pour la JVM, et offre la flexibilité nécessaire pour créer différents types d'architectures selon les besoins de l'application. Depuis la version 6.0 du Framework _Spring_, celui-ci nécessite Java 17+.

_Spring_ supporte un large éventail de scénarios d’application. Dans le contexte d'une grande entreprise, les applications existent souvent sur une longue durée et doivent tourner sur un JDK et sur un serveur d'applications dont le cycle de maintenance dépasse le cadre direct du développeur. Certaines applications s’exécutent comme un simple fichier jar avec un serveur embarqué, possiblement dans un environnement cloud. D'autres encore sont des applications autonomes ne nécessitant pas de serveur.

_Spring_ est un projet open source. Il dispose d'une communauté large et active qui fournit en permanence des retours basés sur des cas d'usage concrets. Cela a permis à _Spring_ d’évoluer de manière pérenne au fil du temps.

## __*Qu'entendons-nous par "Spring"*__

Le terme _Spring_ peut avoir une signification variable selon le contexte. Il peut désigner le projet _Spring_ lui-même, qui est à l'origine de tout l'écosystème.  
Au fil du temps, d'autres projets _Spring_ ont été développés par-dessus le framework _Spring_. La plupart du temps, le terme "_Spring_" désigne l'ensemble des projets constituant l'écosystème Spring.  
La documentation présentée ici se concentre sur la fondation, c'est-à-dire le framework _Spring_ en lui-même.

Le framework _Spring_ est divisé en modules. Les applications peuvent alors choisir uniquement les modules dont elles ont besoin. Au cœur de l'écosystème _Spring_, se trouvent les modules du **Core Container**, incluant un modèle de configuration et un mécanisme d’injection de dépendances.  
Au-delà de cela, le framework _Spring_ fournit un support de base pour différents types d'architectures logicielles, incluant la messagerie, les transactions en base de données, la persistance des données, ainsi que la partie web.  
Il comprend également le **Servlet-based Spring MVC framework** et, parallèlement, le **Spring WebFlux reactive web framework**.

Une précision concernant les modules : les jars du framework _Spring_ autorisent le déploiement dans le module path (JPMS). Pour un usage dans le cadre d'applications modulaires, les jars du framework _Spring_ contiennent `Automatic-Module-Name` dans les entrées du manifest, définissant les noms de modules utilisés (`spring-core`, `spring-context`, etc.), indépendamment du nom des jars eux-mêmes. Les jars suivent la même convention de nommage avec `-` au lieu de `.` – par exemple, `spring-core` et `spring-context`. Bien entendu, les jars du framework _Spring_ fonctionnent parfaitement avec le classpath.

## __*L'histoire de Spring et du framework Spring*__

_Spring_ a vu le jour début 2003 en réponse à la complexité des premières spécifications J2EE.  
Bien que certains pensent que Java EE et son récent successeur Jakarta EE soient en compétition avec _Spring_, ils sont en réalité complémentaires.  
Le modèle de programmation de _Spring_ ne repose pas sur les spécifications de Jakarta EE, mais intègre avec soin des spécifications individuellement sélectionnées depuis le catalogue traditionnel de l'environnement EE:
* Servlet API [(JSR 340)](https://www.jcp.org/en/jsr/detail?id=340)
* WebSocket API [(JSR 356)](https://www.jcp.org/en/jsr/detail?id=356)
* Concurrency Utilities [(JSR 236)](https://www.jcp.org/en/jsr/detail?id=236)
* JSON Binding API [(JSR 367)](https://www.jcp.org/en/jsr/detail?id=367)
* Bean Validation [(JSR 303)](https://www.jcp.org/en/jsr/detail?id=303)
* JPA [(JSR 338)](https://www.jcp.org/en/jsr/detail?id=338)
* JMS [(JSR 914)](https://www.jcp.org/en/jsr/detail?id=914)
* mais également JTA/JCA paramétrages pour la coordination des transactions, si nécessaire.

Le framework _Spring_ supporte également Dependency Injection [(JSR 330)](https://www.jcp.org/en/jsr/detail?id=330) et Common Annotations [(JSR 250)](https://www.jcp.org/en/jsr/detail?id=250) comme spécifications, pour lesquelles les développeurs peuvent les choisir plutôt que des mécanismes spécifiques à _Spring_ et fournis par ce dernier. Initialement, ce sont ceux basés sur les packages `javax`.

Dès la version 6.0 du framework _Spring_, _Spring_ a été mise à jour pour suivre la version 9 de Jakarta EE (par exemple, Servlet 5.0+, JPA 3.0+), en se basant sur l’espace de noms `jakarta` au lieu des traditionnels packages `javax`. Avec une version minimale de EE à 9 et la version EE 10 déjà supportée, _Spring_ est préparé pour fournir un support tout-en-un pour les prochaines évolutions des APIs Jakarta EE.  
Le framework _Spring_ 6.0 est également compatible avec la version 10.1 de Tomcat, la version 11 de Jetty comme serveur web, ainsi qu’avec la version 6.1 de Hibernate ORM.

Au fil du temps, le rôle de Java/Jakarta EE dans le développement des applications a évolué. Au commencement de J2EE et de _Spring_, les applications étaient créées pour être déployées dans un serveur d'application. Aujourd'hui, avec l'aide de _Spring Boot_, les applications sont conçues dans un design DevOps et compatible cloud, avec le Servlet container embarqué et peu de configuration à faire.  
Depuis la version 5 du framework _Spring_, une application WebFlux n'utilise même pas l'API Servlet directement et peut démarrer sur des serveurs (tels que Jetty) qui ne sont pas des containers à Servlet.

_Spring_ continue d'innover et d'évoluer. Au-delà du framework _Spring_, il existe d'autres projets tels que _Spring Boot_, _Spring Security_, _Spring Data_, _Spring Cloud_, _Spring Batch_, parmi d'autres. Il est important de se souvenir que chaque projet a son propre répertoire pour accueillir le code source, les trackers d'événements et les dates de sortie.  
Voir [(spring.io/projects)](https://spring.io/projects) pour la liste complète des projets _Spring_.

## __*Philosophie de conception*__

Lorsque vous apprenez un framework, il est important de ne pas seulement savoir ce pourquoi il a été conçu, mais aussi quels principes il suit. Voici les principes guidant le framework _Spring_ :

* **Fournir un choix à tout niveau.** _Spring_ vous laisse la possibilité de différer les choix de conception le plus tard possible. Par exemple, vous pouvez basculer d'un fournisseur de persistance via la configuration sans changer votre code. La même logique s'applique à beaucoup d'autres infrastructures et à l'intégration avec des APIs tierces.  

* **Prendre en compte diverses perspectives.** _Spring_ est flexible et n'a pas de parti pris concernant la façon dont les choses devraient être faites. Il supporte un large panel de besoins applicatifs avec différentes perspectives.  

* **Maintenir une forte rétrocompatibilité.** Les évolutions de _Spring_ ont été minutieusement conçues pour limiter les changements incompatibles. _Spring_ repose sur un choix soigné de versions du JDK et de bibliothèques tierces afin de faciliter la maintenance des applications et des librairies qui dépendent de _Spring_.  

* **Prêter attention au design des APIs.** L’équipe _Spring_ consacre beaucoup de réflexion et de temps à produire des APIs intuitives, fiables et durables à travers les versions et les années.

* **Maintenir des standards élevés pour la qualité du code.** Le framework _Spring_ met l’accent sur une javadoc pertinente, à jour et précise. C’est l’un des rares projets à pouvoir revendiquer un code propre avec une structure sans dépendances circulaires entre les packages.

## __*Retours et Contributions*__

Concernant des questions d'utilisation, de diagnostic ou de débogage, nous suggérons d'utiliser Stack Overflow. Cliquez [ici](https://stackoverflow.com/questions/tagged/spring%20or%20spring-mvc%20or%20spring-aop) pour une liste de tags proposés. Si vous êtes raisonnablement certain qu'un problème existe dans le framework _Spring_ ou que vous souhaitez proposer une fonctionnalité, s'il vous plaît, utilisez [GitHub Issues](https://github.com/spring-projects/spring-framework/issues).

Si vous avez une solution à l'esprit ou une contribution à proposer, vous pouvez soumettre une pull request sur [GitHub](https://github.com/spring-projects/spring-framework). Cependant, gardez à l'esprit que pour des problèmes mineurs, nous nous attendons à ce qu'un ticket soit d'abord créé dans l’issue tracker, où les discussions ont lieu afin de conserver une trace pour des références futures.

Pour plus de détails, voir le guide suivant : [CONTRIBUTING](https://github.com/spring-projects/spring-framework/blob/main/CONTRIBUTING.md).

## __*Débutez*__

Si vous débutez avec _Spring_, vous pourriez vouloir commencer à utiliser le framework _Spring_ en créant une application basée sur [Spring Boot](https://spring.io/projects/spring-boot). Spring Boot fournit un environnement rapide (et orienté) pour créer une application _Spring_ de production prête à l’emploi. Il repose sur le framework _Spring_, favorise la convention plutôt que la configuration et est conçu pour être opérationnel au plus vite.

Vous pouvez utiliser [start.spring.io](https://start.spring.io) pour générer un projet simple ou suivre un des ["Getting Started" guides](https://spring.io/guides), tels que [Getting Started Building a RESTful Web Service](https://spring.io/guides/gs/rest-service). En plus de leur facilité de compréhension, ces guides sont orientés tâches, et la plupart d'entre eux sont basés sur Spring Boot. Ils couvrent également d'autres projets du portfolio _Spring_ que vous pourriez vouloir incorporer dans la résolution d'un problème particulier.


