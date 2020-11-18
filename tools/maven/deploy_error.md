可以单独部署facade，不能deploy整个项目。

```
Caused by: org.apache.maven.wagon.TransferFailedException: Failed to transfer file: http://127.0.0.1:8081/repository/maven-releases/com/concord/study/travel/1.0.0/travel-1.0.0.pom. Return code is: 400, ReasonPhrase: Bad Request.
```
项目最外层pom
```
<modelVersion>4.0.0</modelVersion>
<groupId>com.concord.study</groupId>
<artifactId>travel</artifactId>
<version>1.0.0</version>
<packaging>pom</packaging>
```

facade层pom
```
<artifactId>travel-facade</artifactId>
<groupId>com.concord.study</groupId>
<version>1.0.0-SNAPSHOT</version>
<modelVersion>4.0.0</modelVersion>
```


### 原因如下，外层的pom定义的为RELEASE，无法deploy第二遍。

A couple things I can think of:

*   user credentials are wrong
*   url to server is wrong
*   user does not have access to the deployment repository
*   user does not have access to the specific repository target
*   artifact is already deployed with that version if it is a release (not -SNAPSHOT version)
*   the repository is not suitable for deployment of the respective artifact (e.g. release repo for snapshot version, proxy repo or group instead of a hosted repository)
