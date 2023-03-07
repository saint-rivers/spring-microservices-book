# Spring Data JPA

## Introduction

Spring Data JPA is part of the Spring Data project, which provides libraries for data access for relational databases. Below are some of the benefits of including Spring Data JPA in your Spring Boot project.

1. Simplifies code for basic CRUD operations
<!-- 2. Provides  -->

## What is an ORM?

ORM stands for Object Relational Mapping which can be described as the process of connecting object code in OOP languages to tables in arelational database.

## What is JPA?

JPA (the Java Persistence API) is a specification (not an implementation) and is used to set a standard API for object-relational mapping. JPA itself is not a product and cannot persist anything into the database. It is simply a set of interfaces that requires implementation.

This can be accomplished by using the provided Java annotations or XML to map objects into one or more tables in a database.

Using JPA as the specification can make a code base modular and makes it easy to swap out for any other implementation, such as Hibernate.

## What is Hibernate?

Hibernate is an implementation of the JPA specification.

Hibernate provides HQL (Hibernate Query Language) which is an SQL inspired language for Java objects.
HQL can be converted to native SQL for each database. So if you're using Postgres, Hibernate will convert any queries in your code to PostgreSQL. Doing this, keeps your code implementation database agnostic and will continue to work, even if the developer decide to switch databases after production, unless they explicitly wrote PostgreSQL code (for example) in their implementation.
