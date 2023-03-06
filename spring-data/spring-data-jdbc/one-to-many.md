# One-to-many Mapping

In the case where we want to be able to rent out a movie many times,
we would need a one-to-many mapping.

In order to make this work, we first need to give the `rental` table its own auto-incrementing ID, but we will not need to add an ID in our Java entity.

```postgresql
CREATE TABLE IF NOT EXISTS movie
(
    id          serial PRIMARY KEY,
    title       text NOT NULL,
    description text NOT NULL
);

CREATE TABLE rental
(
    id        serial PRIMARY KEY,
    movie     integer REFERENCES movie (id), -- 1
    duration  text,
    price     integer
);
```

Now, we can add a `Set<Rental>` to our Movie entity. Spring Data JDBC will be able to identify that the Rental entity needs to be mapped as one-to-one.

```java
@Data
@AllArgsConstructor
public class Movie {
    @Id
    private Long id;
    private String title;
    private String description;
    private Set<Rental> rentals = new HashSet<>();
}
```

Now that everything is set up, we can insert data as follows:

```java
@Component
@Slf4j
@RequiredArgsConstructor
public class MovieAppRunner implements CommandLineRunner {

    private final MovieRepository movieRepository;

    @Override
    public void run(String... args) {
        Set<Rental> rentals = new HashSet<>() {{
                    add(new Rental(Duration.ofDays(1), 2));
                    add(new Rental(Duration.ofDays(7), 10));
                }};

        Movie movie = new Movie(null, "The Matrix", "...",rentals);
        movieRepository.save(movie);

        Set<Movie> movies = movieRepository.findAll();
        log.info(movies.toString());
    }
}
```

> IDK what's happening here

Our movie and rental information is persisted, however, this setup presents a performance issue. If we enable logging by enabling the configuration below:

```properties
logging.level.org.springframework.jdbc.core=DEBUG
```

we can see that whenever we update our Rental information, all children of the movie gets deleted and reinserted. This is not ideal.

```text
// insert log file here
```

We can fix this issue by adding another attribute in our `rental` table.

```postgresql
CREATE TABLE rental
(
    id        serial PRIMARY KEY,
    movie     integer REFERENCES movie (id), -- 1
    movie_key integer,
    duration  text,
    price     integer
);
```

This attribute must be named as `[referenced_table_name]_key`
so that Spring Data JDBC can track each children by a separate id.

There's one more thing we need to change in order to make this work.
We need to change our `Set<Rental>` to a `List<Rental>`

```java
@Data
@AllArgsConstructor
public class Movie {
    @Id
    private Long id;
    private String title;
    private String description;
    private List<Rental> rentals = new ArrayList<>();
}
```

Again, we do not need to change anything in the `Rental` class.

## Under Construction

```text
Executing SQL update and returning generated keys
Executing prepared SQL statement [INSERT INTO "movie" ("description", "title") VALUES (?, ?)]
Executing SQL batch update [INSERT INTO "rental" ("duration", "movie", "movie_key", "price") VALUES (?, ?, ?, ?)]
Executing prepared SQL statement [INSERT INTO "rental" ("duration", "movie", "movie_key", "price") VALUES (?, ?, ?, ?)]

Executing prepared SQL update
Executing prepared SQL statement [UPDATE "movie" SET "title" = ?, "description" = ? WHERE "movie"."id" = ?]
Executing prepared SQL update
Executing prepared SQL statement [DELETE FROM "rental" WHERE "rental"."movie" = ?]
Executing SQL batch update [INSERT INTO "rental" ("duration", "movie", "movie_key", "price") VALUES (?, ?, ?, ?)]
Executing prepared SQL statement [INSERT INTO "rental" ("duration", "movie", "movie_key", "price") VALUES (?, ?, ?, ?)]
```

Spring Data provides a CrudRepository which simplifies and speeds up development time. However, it is clear from the logs above that it does not handle updating as efficiently as we would like.

But why is Spring Data handling it this way?

Simplicity. Spring Data aims to provide a simple implementation of our database and, currently, this is the only way this is handled by Spring Data.

To circumvent this implementation, we can use the JdbcTemplate to run a custom update statement.

Read more at [JdbcTemplate](./jdbc-template.md)

## Reference

[Explanation of Spring Data implementation](https://stackoverflow.com/questions/72503705/spring-data-jdbc-deletes-all-entries-and-inserts-them-again)
