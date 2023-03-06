# Spring Data JPA vs. Spring Data JDBC

While comparing both persistence libraries, it's important to note that they share some similarities, namely that both are under the Spring Data project and will be using some of the same dependencies.


## Developer Experience

Spring Data JPA provides an additional layer of abstraction, with the main purpose of providing implementation that the developer does not have to manage, making it much faster to develop using Spring Data JPA. The drawback to this abstraction layer is that code will be implicit and sometimes, it will not be running the most efficient SQL commands and it may not be the most predictable when it comes to development.

However, some developers opt to use Spring Data JDBC because it does not have that abstraction layer and the code has to be more explicit, making for a more verbose and precise code base.

## Performance

## Exception Handling

## Database Dependencies

Spring Data JPA is database-agnostic, meaning writing code for Spring Data JPA will not depend on which database vendor is being used. Queries can be written in JPQL (Java Persistence Query Language), which can be mapped to whichever SQL dialect is specified (i.e. PostgreSQL, MySQL, etc).

On the other hand, developers have to implement their code with the SQL dialect in mind and implement their queries based on the database used. 

## Database Schema

And unlike with JPA/Hibernate, Spring Data JDBC does not provide an option to generate database tables based off of the entities created. Therefore, the developer must create scripts to create the necessary tables or have it managed by a database administrator. 

The pros are that database management can be externalized and managed outside of the API, which can be desirable if we're follwing the `Separation of Concerns` pricipal.

The downside to this is that development time can be much slower as the developer has to first create the database schema and personally ensure that it maps to the entity correctly. And they would usually have to write their own SQL statements. 