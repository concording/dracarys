
### spring boot Actuator
https://docs.spring.io/spring-boot/docs/current/reference/html/index.html

### prometheus
https://prometheus.io/docs/prometheus/latest/getting_started/

```
management.server.port=9144
management.endpoints.web.exposure.include=health,metrics,prometheus
management.metrics.tags.application=test
management.metrics.distribution.percentiles-histogram.http.server.requests=true
```
