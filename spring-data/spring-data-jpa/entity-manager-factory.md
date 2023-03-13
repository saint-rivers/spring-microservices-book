# Understanding the Entity Manager Factory

## Persistence Unit

As seen in code block 4.2, the persistence unit holds configurations pertaining to many properties such as:

- the persistence provider (which JPA implementation to use)
- the database driver to use
- the datasource url, username and password
- the SQL dialect to use
- other hibernate configurations used in development

Without Spring Boot, the file below needs to be declared in order to create a persistence unit.

Filename: `src/main/resources/persistence.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence
          http://xmlns.jcp.org/xml/ns/persistence/persistence_2_1.xsd"
    version="2.1">

    <persistence-unit name="TestDB">
	    <class>org.example.entity.Student</class>
	    <class>org.example.entity.EducationBackground</class>
	    <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <properties>
            <property name="javax.persistence.jdbc.url"
			          value="jdbc:h2:mem:studentmanagement" />
            <property name="javax.persistence.jdbc.user" value="sa" />
            <property name="javax.persistence.jdbc.password" value="" />
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver" />
            <property name="hibernate.show_sql" value="true" />
            <property name="hibernate.format_sql" value="true" />
        </properties>
    </persistence-unit>
</persistence>
```

## EntityManagerFactory

Why is the Persistence Unit required?

When an EntityManager needs to be injected, an EntityManagerFactory will create an EntityManager based on the predefined configuration declared in the Persistence Unit in the file `src/main/resources/persistence.xml`. The EntityManager will then have all the necessary details to manage database connections and the entities for that Persistence Context. There will only be one EntityManagerFactory for each Persistence Unit.

## References

https://www.developer.com/design/entity-manager-jpa-tutorial/

https://www.logicbig.com/tutorials/java-ee-tutorial/jpa/entity-context.html

https://stackoverflow.com/questions/19930152/what-is-persistence-context

https://stackoverflow.com/questions/21038706/persistenceunit-vs-persistencecontext

https://www.codejava.net/frameworks/spring/understand-spring-data-jpa-with-simple-example
