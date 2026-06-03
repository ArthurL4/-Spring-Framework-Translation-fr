{
page:23,
from:"https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/custom-autowire-configurer.html",
update:"2026-06-03",
by:["Arthur Leroux"],
}

# __*Utilisation de CustomAutowireConfigurer*__

[`CustomAutowireConfigurer`](https://docs.spring.io/spring-framework/docs/7.0.7/javadoc-api/org/springframework/beans/factory/annotation/CustomAutowireConfigurer.html) est un `BeanFactoryPostProcessor`  
qui vous permet d’enregistrer vos propres types d’annotations qualifiantes personnalisées, même si elles ne sont pas annotées avec l’annotation Spring `@Qualifier`.

L’exemple suivant montre comment utiliser `CustomAutowireConfigurer` :

```xml
<bean id="customAutowireConfigurer"
      class="org.springframework.beans.factory.annotation.CustomAutowireConfigurer">
    <property name="customQualifierTypes">
        <set>
            <value>example.CustomQualifier</value>
        </set>
    </property>
</bean>
```

L'`AutowireCandidateResolver` détermine les candidats à l’autowiring de la manière suivante :

- La valeur `autowire-candidate` de chaque définition de bean
- Tous les motifs `default-autowire-candidates` disponibles dans l’élément `<beans/>`
- La présence des annotations `@Qualifier` et de toute annotation personnalisée enregistrée avec `CustomAutowireConfigurer`

Lorsqu’il existe plusieurs beans candidats à l’autowiring, la sélection du bean « principal » se fait comme suit : si une seule définition de bean parmi les candidats possède l’attribut `primary` défini à `true`, 
elle est sélectionnée. Pour les configurations basées sur les annotations, voir [`Fine-tuning with @Primary or @Fallback `]().