# One-to-one Mapping

[Spring Data JDBC](./getting-started.md)

## Objective
- create a one-to-one mapping between a movie and a movie rental
- meaning the movie entity will have a one-to-one relationship with a rental entity

## Creating the Entity

Firstly, we need to create a new table to store data about movie rentals.

```postgresql
CREATE TABLE IF NOT EXISTS movie
(
    id          serial PRIMARY KEY,
    title       text NOT NULL,
    description text NOT NULL
);

CREATE TABLE rental
(
    movie     integer PRIMARY KEY REFERENCES movie (id), -- 1
    duration  text,
    price     integer
);
```

Note that `movie` is an integer that will be our foreign key that references the `movie` table. Because we want this to be a one-to-one mapping, we should also make this attribute a `PRIMARY KEY` to make it unique, which will enforce the one-to-one nature. 

> It is important to note that the field must be identical to the table to be referenced. Otherwise, the mapping will not work.

First, we will need to create the Java class below:

```java
@Data
@AllArgsConstructor
public class Rental {
    private Duration duration;
    private Integer price;
}
```

Notice that there is no ID property in the class and no property annotated with `@Id`. This is because Spring Data JDBC tries to adhere more to Domain-Driven-Design concepts. More on that at ...

Now, we can add a `List<Rental>` to our Movie entity. Spring Data JDBC will be able to identify that the Rental entity needs to be mapped as one-to-one.

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

## Usage
