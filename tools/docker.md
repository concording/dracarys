
### docker install failed 
`sudo systemctl start docker`

```
-- Defined-By: systemd
-- Support: https://access.redhat.com/support
--
-- Automatic restarting of the unit containerd.service has been scheduled, as the result for
-- the configured Restart= setting for the unit.
Sep 30 10:08:19 VM-0-17-centos systemd[1]: Stopped containerd container runtime.
-- Subject: Unit containerd.service has finished shutting down
-- Defined-By: systemd
-- Support: https://access.redhat.com/support
--
-- Unit containerd.service has finished shutting down.
Sep 30 10:08:19 VM-0-17-centos systemd[1]: Starting containerd container runtime...
-- Subject: Unit containerd.service has begun start-up
-- Defined-By: systemd
-- Support: https://access.redhat.com/support
--
-- Unit containerd.service has begun starting up.
Sep 30 10:08:19 VM-0-17-centos containerd[7581]: /usr/bin/containerd: symbol lookup error: /usr/bin/containerd: undefined symbol: seccomp_api_set
Sep 30 10:08:19 VM-0-17-centos systemd[1]: containerd.service: Main process exited, code=exited, status=127/n/a
Sep 30 10:08:19 VM-0-17-centos systemd[1]: containerd.service: Failed with result 'exit-code'.
Sep 30 10:08:19 VM-0-17-centos systemd[1]: Failed to start containerd container runtime.
-- Subject: Unit containerd.service has failed
-- Defined-By: systemd
-- Support: https://access.redhat.com/support
--
-- Unit containerd.service has failed.
--
-- The result is failed.
Sep 30 10:08:20 VM-0-17-centos systemd[1]: containerd.service: Service RestartSec=100ms expired, scheduling restart.
Sep 30 10:08:20 VM-0-17-centos systemd[1]: containerd.service: Scheduled restart job, restart counter is at 5.
-- Subject: Automatic restarting of a unit has been scheduled
-- Defined-By: systemd
-- Support: https://access.redhat.com/support
--
-- Automatic restarting of the unit containerd.service has been scheduled, as the result for
-- the configured Restart= setting for the unit.
Sep 30 10:08:20 VM-0-17-centos systemd[1]: Stopped containerd container runtime.
-- Subject: Unit containerd.service has finished shutting down
-- Defined-By: systemd
-- Support: https://access.redhat.com/support
--
-- Unit containerd.service has finished shutting down.
Sep 30 10:08:20 VM-0-17-centos systemd[1]: containerd.service: Start request repeated too quickly.
Sep 30 10:08:20 VM-0-17-centos systemd[1]: containerd.service: Failed with result 'exit-code'.
Sep 30 10:08:20 VM-0-17-centos systemd[1]: Failed to start containerd container runtime.
-- Subject: Unit containerd.service has failed
-- Defined-By: systemd
-- Support: https://access.redhat.com/support
--
-- Unit containerd.service has failed.
--
-- The result is failed.
```


`yum update`

### docker start failed

`docker run --name some-zookeeper --restart always -d zookeeper`

`failed to start daemon: pid file found, ensure docker is not running or delete /var/run/docker.pid`

`sudo usermod -aG docker $USER`

### docker command

`docker exec -it some-zookeeper /bin/bash`


### docker mysql 
`docker stop mysql`

`docker rm mysql`

`docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7`

`docker exec -it  mysql mysql -uroot -p`

 [mysql 8.0 客户端无法连接](https://pythonspeed.com/articles/docker-connection-refused/)

### docker storage
[storage](https://docs.docker.com/storage/)
 
