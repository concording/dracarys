# [Spring Boot] Spring Data JPA vs MyBatis



You are using the Spring Boot project to build enterprise web applications. So, Are you using Spring Data JPA or MyBatis for your project?

Why do you choose Spring Data JPA or MyBatis? Here is my answer: depends on the size of your project. If your project is small you can choose one of them but when it’s medium or huge. You should consider using them carefully. In my option, we should use both Spring Data JPA and MyBatis. To achieve high performance, we should take strong points (advantages) of them. You can use Spring Data JPA for writing (INSERT, UPDATE, DELETE query) and use MyBatis for reading (SELECT query).

In this article, we’ll only focus on MySQL (RDMS) and two famous libraries: Spring Data JPA, MyBatis that most of my projects are using them.

Let’s start !!!

# What is Spring Data JPA?

Before talking about Spring Data JPA. It’s about time talking about Spring. Firstly, The Spring makes it easy to create Java enterprise applications. It’s open-source. Secondary, The Spring contains many projects: Spring Boot, Spring Framework, Spring Data, Spring Cloud, Spring Security, Spring Session, Spring AMQP, Spring Batch … All projects have been built on top of the Spring Framework. Most often, when people say “Spring”, they mean the entire family of projects.

Spring Data is a subproject in the Spring ecosystem. The Spring Data family includes many libraries such as Spring Data JPA, Spring Data Redis, Spring Data MongoDB, and so on. It helps developers develop faster and reduce errors and risks when communicating with the database.

JPA (Java Persistent API) just is Interface. Hibernate is the implementation of JPA.

![Image for post](https://miro.medium.com/max/60/1*DWD7ocJUfwxc0LcjG3ttSg.png?q=20)

![Image for post](https://miro.medium.com/max/2236/1*DWD7ocJUfwxc0LcjG3ttSg.png)

ORM Mapping (Hibernate)

# What is MyBatis?

MyBatis is one of the most commonly used open-source frameworks for implementing SQL database access in Java applications.

MyBatis-Spring integrates MyBatis seamlessly with Spring. This library allows MyBatis to participate in Spring transactions, takes care of building MyBatis mappers and SqlSessions and inject them into other beans, translates MyBatis exceptions into Spring DataAccessExceptions, and finally, it lets you build your application code free of dependencies on MyBatis, Spring or MyBatis-Spring.

# Compare Spring Data JPA and MyBatis

![Image for post](https://miro.medium.com/max/60/1*FHcYWYtBOiuxbt2ZsBZGig.png?q=20)

![Image for post](https://miro.medium.com/max/4284/1*FHcYWYtBOiuxbt2ZsBZGig.png)

Spring Data JPA vs MyBatis

# Integrate Spring Data JPA and MyBatis in one project

Some medium applications that they will use MySQL as the primary database and Master-Slave model(One master and n slaves). For example, a NEWS application can serve about 100 million users and collect information from a lot of content providers. Of course, they use a lot of storage such as MongoDB, Redis, and so on.

As I told before, you can use Spring Data JPA for writing (INSERT, UPDATE, DELETE query). It’s a good choice when we want to write something in the database. You will create less code that means decrease bugs. It will make your code more readable. In case, we need to join many tables(even though 3–4 tables) for report features. If you use Spring Data JPA we will make complex code with mapping the result. But with MyBatis we can do it easily by mapping mechanism. Of course, we have to add more code in this case. However, we will gain good reading performance. In my option, It is worth it.

# Some notes JPA

*   JPA supports a lot of useful Query Methods based on abstraction is Repository. For example, `save(), findById(), findAll(), count(), delete(), existesById()`, and so on.
*   Help us for defining Query Methods: `Query Lookup Strategies, Query Creation (find..By, count..By, Page<T>, Pageable, Sort, …), Limit Query Results, Repository Methods Returning Collections or Interables (Streamable, Interable, List, Set), Null Handling (Optional), Streaming query results (Stream<T>), Async query results (CompletableFuture<T>).`
*   It supports `Stored Procedures, Transactionality, Locking, and Auditing`.
*   It supports a criteria API (`Specifications`) that you can build queries programmatically.
*   You can write raw queries with `@Query` with parameter `nativeQuery` is true and write modifying queries with `@Modifying` annotation

More details: [https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#reference](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#reference)

# Some notes MyBatis

I highly recommend you using spring boot `mybatis-starter`. Ref: [https://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/.](https://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/) Otherwise, you can use two libraries: `mybatis` and `mybatis-spring` and config them manually.

To active/enable spring transaction, you have config DataSourceTransactionManager like the following:

More details: [http://mybatis.org/spring/index.html](http://mybatis.org/spring/index.html)

# Conclusion

*   Thank you for your reading. Here is my experience when we have worked with Spring Data JPA vs MyBatis. In some projects, I have used both Spring Data JPA and MyBatis to gain high performance.
*   If you have any doubt/questions, please comment here or tell me.

# References

*   [https://docs.spring.io/spring/docs/current/spring-framework-reference/data-access.html](https://docs.spring.io/spring/docs/current/spring-framework-reference/data-access.html)
*   [http://mybatis.org/spring/index.html](http://mybatis.org/spring/index.html)
