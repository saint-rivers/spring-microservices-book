# Entity Manager & Persistence Context

An EntityManager instance is associated with a persistence context. A persistence context is a set of entity instances in which for any persistent entity identity there is a unique entity instance. Within the persistence context, the entity instances and their lifecycle are managed. The EntityManager API is used to create and remove persistent entity instances, to find entities by their primary key, and to query over entities.

The set of entities that can be managed by a given EntityManager instance is defined by a persistence unit. A persistence unit defines the set of all classes that are related or grouped by the application, and which must be colocated in their mapping to a single database.

```java
EntityTransaction transaction = entityManager.getTransaction(); 
transaction.begin(); 

// code here

transaction.commit();
entityManager.close();
```

When working with an entity, a transaction is required to perform any changes to the data in the table.
The `entityManager.getTransaction().begin()` method first pulls the transaction from current persistence context and then begins the transaction using `begin()` method.

After the desired changes are made to the entity, `entityManager.getTransaction.commit()` method is used to fetch the transaction and then to commit the same transaction. This will commit all the changes to database.

After a commit, the EntityManager needs to be closed to detach all entities in the context using the `entityManager.close()`.
Otherwise, the entities in the persistent context will remain in the persistent state.

## Code

In the code below, we can inject the EntityManager simply by annotation it with `@PersistentContext`.
As one EntityManager is associated with a single persistent context, this means that a single bean is created by a proxy for each persistent context. 

```java
import jakarta.persistence.*;
import org.springframework.stereotype.Repository;

@Repository
public class StudentRepository {

    @PersistentContext
    private EntityManager entityManager;

    public void insertStudent(){
        EntityTransaction transaction = entityManager.getTransaction(); 
        transaction.begin(); 

        var student = new Student(null, "Josh", "Dun", "joshd@gmail.com", 30);
        entityManager.persist(student);

        transaction.commit();
        entityManager.close();
    }
}
```

The code above is explicit but contains boilerplate code and can make it difficult to read. 
JPA provides the `@Transactional` annotation which will run the declared function in a transaction 
without having it write it manually, making the code base cleaner.

```java
import jakarta.persistence.*;
import jakarta.transaction.Transactional;
import org.springframework.stereotype.Repository;

@Repository
public class StudentRepository {

    @PersistentContext
    private EntityManager entityManager;

    @Transactional
    public void insertStudent(){
        var student = new Student(null, "Josh", "Dun", "joshd@gmail.com", 30);
        entityManager.persist(student);
    }
}
```


## References:

https://www.digitalocean.com/community/tutorials/jpa-entitymanager-hibernate 

https://access.redhat.com/documentation/en-us/red_hat_jboss_web_server/1.0/html-single/hibernate_entity_manager_reference_guide/index

https://stackoverflow.com/questions/65725054/spring-boot-create-many-entity-manager

https://jakarta.ee/specifications/persistence/2.2/apidocs/javax/persistence/entitymanager

https://blog.devgenius.io/hibernate-tutorial-2-context-entity-593b828f4f28 







