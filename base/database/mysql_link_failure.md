```
2021-08-21 10:13:55.284[ERROR][traceId: ][com.alibaba.druid.pool.DruidDataSource][run][2507][Druid-ConnectionPool-Create-114588098] -> create connection SQLException, url: jdbc:mysql://********:3306/test_pay?useUnicode=true&characterEncoding=utf8&allowMultiQueries=true, errorCode 0, state 08S01
com.mysql.cj.jdbc.exceptions.CommunicationsException: Communications link failure

The last packet sent successfully to the server was 0 milliseconds ago. The driver has not received any packets from the server.
	at com.mysql.cj.jdbc.exceptions.SQLError.createCommunicationsException(SQLError.java:174)
	at com.mysql.cj.jdbc.exceptions.SQLExceptionsMapping.translateException(SQLExceptionsMapping.java:64)
	at com.mysql.cj.jdbc.ConnectionImpl.createNewIO(ConnectionImpl.java:835)
	at com.mysql.cj.jdbc.ConnectionImpl.<init>(ConnectionImpl.java:455)
	at com.mysql.cj.jdbc.ConnectionImpl.getInstance(ConnectionImpl.java:240)
	at com.mysql.cj.jdbc.NonRegisteringDriver.connect(NonRegisteringDriver.java:199)
	at com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1570)
	at com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1636)
	at com.alibaba.druid.pool.DruidDataSource$CreateConnectionThread.run(DruidDataSource.java:2505)
Caused by: com.mysql.cj.exceptions.CJCommunicationsException: Communications link failure

```


&useSSL=false&allowMultiQueries=true&serverTimezone=Asia/Shanghai



