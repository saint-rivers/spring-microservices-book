# Entity Manager API

The EntityManager has been covered to some extent in sections [222] and [222], however, this section will focus on some basic CRUD operations with the EntityManager API. 

## Test Setup

For the test, a repository bean in required to declare the CRUD methods. 

```java
import jakarta.persistence.*;
import jakarta.transaction.Transactional;
import org.springframework.stereotype.Repository;

@Repository
public class StudentRepository {

    @PersistentContext
    private EntityManager entityManager;

    // crud operations
}
```

And for simplicity, the methods that will be created will be called from a Spring Boot Test. A simple Spring Boot Test can be created in the directory: `src/test/java/{groupId}/{artifactId}/`

For the example below, it was created in the `src/test/java/com/testing/hibernatedemo` directory.

```java
import com.testing.hibernatedemo.student.Contact;
import com.testing.hibernatedemo.student.Student;
import com.testing.hibernatedemo.student.StudentRepositoryImpl;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
public class StudentRepositoryUnitTest {

    @Autowired
    private StudentRepository studentRepository;

    // test methods here
}
```

## Create using persist()

To insert a new record into the database using the Entity Manager API, the persist() method is used and a transient entity (an entity with no ID) must be passed as an argument.

Class: StudentRepository.java

```java
@Transactional
public void insertStudent(Student student){
	entityManager.persist(student);
}
```

Class: StudentRepositoryUnitTest.java

```java
@Test  
void shouldInsertStudent() {  
	Student student = new Student(null, "Thomas", "Shelby", "thomas@gamil.com", 32);
	System.out.println(student);
	Long id = studentRepository.insertStudent(student);  
	System.out.println(student);
	assert id == 1L;  
}
```

```text
Student(id=null, firstName=Thomas, lastName=Shelby, email=thomas@gmail.com, age=32)
Student(id=1, firstName=Thomas, lastName=Shelby, email=thomas@gmail.com, age=32)
```

As seen in the demonstration in figure [222], after an entity has been persisted, the EntityManager will set the generated ID for that entity. This also signifies that the entity is no longer transient but is now persistent.

## Read an entity by ID

The Entity Manager API provides several overloaded find() methods to fetch an entity by its ID. This is the simplest way to fetch an entity, assuming the record exists.

Class: StudentRepository.java

```java
@Transactional
public Student findByStudentId(Long studentId) {
    Student fetchedStudent = entityManager.find(Student.class, studentId);
    System.out.println(fetchedStudent);
}
```

Class: StudentRepositoryUnitTest.java

```java
@Test  
void shouldInsertStudent() {  
	Student student = new Student(null, "Thomas", "Shelby", "thomas@gamil.com", 32);
	studentRepository.insertStudent(student);  

    student = studentRepository.findByStudentId(1L);
    assert student.getFirstName().equals("Thomas");
}
```

```text
Student(id=1, firstName=Thomas, lastName=Shelby, email=thomas@gmail.com, age=32)
```

## Create using merge()

Another way to insert a record into the database is to use the merge() method. This method also takes a single argument, which is the entity to be inserted.

Class: StudentRepository.java

```java
System.out.println(student);
entityManager.merge(student);
System.out.println(student);

Student fetchedStudent = entityManager.find(Student.class, 1L);
System.out.println(fetchedStudent);
```

Class: StudentRepositoryUnitTest.java

```text
Student(id=null, firstName=Thomas, lastName=Shelby, email=thomas@gmail.com, age=32)
Student(id=null, firstName=Thomas, lastName=Shelby, email=thomas@gmail.com, age=32)
Student(id=1, firstName=Thomas, lastName=Shelby, email=thomas@gmail.com, age=32)
```

## persist() vs. merge()

At first glance, these two functions seems to do the exact same thing, to create a new record in the database which did not previously exist. However, there are a few key differences.

Firstly, it would be helpful to look at the method declaration in the EntityManager interface. 

```java
public void persist(Object entity);
public <T> T merge(T entity);
```

As seen in the source code, persist() takes an Java Object as it's entity instead of a generic type as with the merge() method, mainly because the merge() method needs to return the entity as a persistent entity. 

There are several reasons to do this.

When using persist(), the entity's reference is passed in and the entity manager makes the entity persistent. This is why the `student` has an ID after being persisted.

On the other hand, merge() only copies the data from the entity and creates a new instance to be persisted. The initial `student` entity will remain as a transient entity. The merge() method will also return the newly created (and persistent) entity to be used. 

```java
Student student = new Student(null, "Thomas", "Shelby", "thomas@gamil.com", 32); // transient
Student persistedStudent = entityManager.merge(student); // new persisted entity

System.out.println("Transient: " + student);
System.out.println("Merged: " + persistedStudent);

persistedStudent.setFirstName("Arthur");

Student fetchedStudent = entityManager.find(Student.class, 1L);
System.out.println("Fetched: " + fetchedStudent);
```

```text
Transient: Student(id=null, firstName=Thomas, lastName=Shelby, email=thomas@gmail.com, age=32)
Merged: Student(id=1, firstName=Thomas, lastName=Shelby, email=thomas@gmail.com, age=32)
Fetched: Student(id=1, firstName=Arthur, lastName=Shelby, email=thomas@gmail.com, age=32)
```

## Updating an Entity

### method 1

Another notable point about the find() method is that it returns a persisted entity. That means, the entity can simply be updated by first fetching the instance, then making any desired changes to it and committing it afterwards.

Class: StudentRepository.java

```java
@Transactional
public void updateByStudentId(Long studentId){
	Student fetchedStudent = entityManager.find(Student.class, studentId);
	fetchedStudent.setFirstName("Polly");
	fetchedStudent.setLastName("Gray");
}
```

Class: StudentRepositoryUnitTest.java

```java
@Test  
void shouldUpdateStudentInformation() {  
	Student student = new Student(null, "Thomas", "Shelby", "thomas@gamil.com", 32);
	
	studentRepository.insertStudent(student);  
    Student fetchedStudent = entityManager.find(Student.class, 1L);
	System.out.println(fetchedStudent);

    studentRepository.updateByStudentId(1L);
    fetchedStudent = entityManager.find(Student.class, 1L);
	System.out.println(fetchedStudent);
}
```

```text
Student(id=1, firstName=John, lastName=Shelby, email=thomas@gmail.com, age=32)
Student(id=1, firstName=Polly, lastName=Gray, email=thomas@gmail.com, age=32)
```

### method 2

Alternatively, creating a new transient entity, but setting the ID to an existing one, then merging or persisting it will also update the record by the specified ID.

Class: StudentRepository.java

```java
@Transactional
public void updateByStudentId(Long studentId){
	Student updatedStudent = new Student(studentId, "Alfie", "Solomons", "alfie@gamil.com", 40);
	entityManager.persist(updatedStudent);
}
```

Class: StudentRepositoryUnitTest.java

```java
@Test  
void shouldUpdateStudentInformation() {  
	Student student = new Student(null, "Thomas", "Shelby", "thomas@gamil.com", 32);
	
	studentRepository.insertStudent(student);  
    Student fetchedStudent = entityManager.find(Student.class, 1L);
	System.out.println(fetchedStudent);

    studentRepository.updateByStudentId(1L);
    fetchedStudent = entityManager.find(Student.class, 1L);
	System.out.println(fetchedStudent);
}
```

```text
Student(id=1, firstName=Thomas, lastName=Shelby, email=thomas@gmail.com, age=32)
Student(id=1, firstName=Alfie, lastName=Solomons, email=alfie@gmail.com, age=40)
```

## Deleting an Entity

To perform deletion of an entity, use the remove() method to remove the entity from the persistence context and also delete the record from the database, while also making the instance ready for garbage collection.

Class: StudentRepository.java

```java
@Transactional
public void deleteByStudentId(Long studentId) {
	Student student = entityManager.find(Student.class, studentId)
    entityManager.remove(student);
}
```

It is important to run the delete and the find method in different transactions. Otherwise, the deletion will not be propagated and the record will still be searchable.

Class: StudentRepositoryUnitTest.java

```java
@Test  
void shouldDeleteInsertedStudent() {  
	Student student = new Student(null, "Thomas", "Shelby", "thomas@gamil.com", 32);
	studentRepository.insertStudent(student);  

    studentRepository.deleteByStudentId(1L);

    student = studentRepository.findByStudentId(1L);
    assert student == null;
}
```


If not record is found with the given ID, it should return null, instead of throwing an exception.

References: 

https://www.javaguides.net/2018/12/jpa-crud-example.html