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

Our movie and rental information is persisted, however, this setup presents a performance issue. If we enable logging by enabling the configuration below:

```properties
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
