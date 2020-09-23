## Spring 数据库连接池默认选择顺序

*   We prefer the Tomcat pooling DataSource for its performance and concurrency, so if that is available we always choose it.

*   Otherwise, if HikariCP is available we will use it.

*   If neither the Tomcat pooling datasource nor HikariCP are available and if Commons DBCP is available we will use it, but we don’t recommend it in production.

*   Lastly, if Commons DBCP2 is available we will use it.

If you use the spring-boot-starter-jdbc or spring-boot-starter-data-jpa ‘starters’ you will automatically get a dependency to tomcat-jdbc.

参考网址 [http://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-sql.html](http://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-sql.html)