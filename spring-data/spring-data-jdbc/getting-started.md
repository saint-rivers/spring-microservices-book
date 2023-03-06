# Spring Data JDBC

## 1. Dependencies

We will use PostgreSQL for our database, and we only need one dependency to work with Spring Data JDBC.

Groovy - Gradle

```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-jdbc'
runtimeOnly 'org.postgresql:postgresql'
```

Maven

```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jdbc</artifactId>
    </dependency>
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>
```

## 2. Connecting to the database

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/postgres
spring.datasource.username=postgres
spring.datasource.password=password
spring.datasource.driver-class-name=org.postgresql.Driver
```

## 3. Creating an entity

// todo: explanation

Let's create a table in our database.

```postgresql
CREATE TABLE movie
(
    id          serial PRIMARY KEY,
    title       text NOT NULL,
    description text NOT NULL
);
```

Then create an entity to map our database table to a Java object.

```java
import lombok.AllArgsConstructor;
import lombok.Data;
import org.springframework.data.annotation.Id;

@Data // 1
@AllArgsConstructor // 2
public class Movie {
    @Id // 3
    private Long id;
    private String title;
    private String description;
}
```

There are a few things to note here:

1. Lombok annotation to generate getters, setters, no args constructor, etc
2. Spring Data JDBC needs an all args constructor when reading data from a result set
3. We need the `@Id` annotation to tell Spring Data JDBC that the annotated field
   is an auto-incremented value managed by the database.

However, we can be more verbose and explicit when creating our entity.
The code below is exactly the same.

```java
import lombok.AllArgsConstructor;
import lombok.Data;
import org.springframework.data.annotation.Id;
import org.springframework.data.relational.core.mapping.Column;
import org.springframework.data.relational.core.mapping.Table;

@Data
@AllArgsConstructor
@Table("movie")
public class Movie {

    @Id
    @Column("id")
    private Long id;

    @Column("title")
    private String title;

    @Column("description")
    private String description;
}
```

Annotating the class with `@Table`, the developer can map a table to a class,
even if their names do not match. The `@Column` achieves the same purpose,
but for mapping table columns to class properties.

> It should be noted that, by default, Spring Data JDBC will map postgres identifiers in snake_case to Java in
> PascalCase.

```java
import lombok.AllArgsConstructor;
import lombok.Data;
import org.springframework.data.annotation.Id;
import org.springframework.data.relational.core.mapping.Column;
import org.springframework.data.relational.core.mapping.Table;

@Data
@AllArgsConstructor
@Table("movie_listing")
public class Movie {

    @Id
    @Column("movie_id")
    private Long id;

    @Column("movie_title")
    private String title;

    @Column("short_description")
    private String description;
}
```

Create a Crud repository

```java
import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface MovieRepository extends CrudRepository<Movie, Long> {
}
```

In order to persist data to the database table, use the created repository.

```java

@Component
@Slf4j
@RequiredArgsConstructor
public class MovieAppRunner implements CommandLineRunner {

    private final MovieRepository movieRepository;

    @Override
    public void run(String... args) {
        Movie movie = new Movie(
                null,
                "The Matrix",
                "...",
        );
        movieRepository.save(movie);

        var movies = movieRepository.findAll();
        log.info(movies.toString());
    }
}
```

## References

[Spring Data JDBC - Reference Documentation](https://docs.spring.io/spring-data/jdbc/docs/current/reference/html)

[Introduction to Spring Data JDBC (YouTube)](https://www.youtube.com/watch?v=EaHlancPA14)

[Spring Data JDBC – Getting Started](https://thorben-janssen.com/spring-data-jdbc-getting-started)
