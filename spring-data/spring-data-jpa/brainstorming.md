# Brainstorming

## Entity

- an instance of that can be persisted into the database

## EntityManager

- An object that is tasked with managing the states of entities
- by providing functions to manage the states of the provided entities

## Persistent Context

- a set of entity instances, under the purview of an entity manager

## EntityManagerFactory

- creates entity managers based on the configuration specified in the persistence unit
- the configurations will include datasource url, database drivers, etc

## Persistence Unit

As seen in code block 4.2, the persistence unit holds configurations pertaining to many properties such as:

- the persistence provider (which JPA implementation to use)
- the database driver to use
- the datasource url, username and password
- the SQL dialect to use
- other hibernate configurations used in development

```xml
<persistence xmlns="http://java.sun.com/xml/ns/persistence"
             xmlns_xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi_schemaLocation="http://java.sun.com/xml/ns/
                persistence
             http://java.sun.com/xml/ns/persistence/
                persistence_2_0.xsd"
             version="2.0">
   <persistence-unit name="BooksPU"
      transaction-type="RESOURCE_LOCAL">
   <!-- Persistence provider -->
   <provider>org.hibernate.jpa
      .HibernatePersistenceProvider</provider>
   <!-- Entity classes -->
   <class>org.example.entity.Book</class>
   <properties>
      <property name="javax.persistence.jdbc.driver"
      value="org.h2.Driver" />
      <property name="javax.persistence.jdbc.url"
      value="jdbc:h2:mem:bookstore" />
      <property name="javax.persistence.jdbc.user"
      value="sa" />
      <property name="javax.persistence.jdbc.password"
      value="" />
      <property name="hibernate.dialect"
      value="org.hibernate.dialect.H2Dialect"/>
      <property name="hibernate.hbm2ddl.auto"
      value="update" />
      <property name="show_sql"
      value="true"/>
      <property name="hibernate.temp.use_jdbc_metadata_defaults"
      value="false"/>
      <property name="hibernate.format_sql"
      value="true"/>
      <property name="hibernate.use_sql_comments"
      value="true"/>
   </properties>
   </persistence-unit>
</persistence>
```

## References

https://www.developer.com/design/entity-manager-jpa-tutorial/

https://www.logicbig.com/tutorials/java-ee-tutorial/jpa/entity-context.html

https://stackoverflow.com/questions/19930152/what-is-persistence-context

https://stackoverflow.com/questions/21038706/persistenceunit-vs-persistencecontext
