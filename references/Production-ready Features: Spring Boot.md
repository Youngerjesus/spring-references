# Production-ready Features

https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.health

application 을 Production 에 배포할 때 monitor 와 manage 하기 위한 기능들을 제공해주는 것에 대해서 다룸.
- HTTP endpoint, JMX, Auditing, health, metric 을 통해서 가능하게 해줌.

## 1. Enabling Production-ready Features

사용하는 방법에 대해서 다룸

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
}
```


## 2. Endpoints

Spring Boot 의 endpoint 를 통해서 app 을 monitoring 하거나 interaction 할 수 있다. 
- 이런 endpoint 는 built-in (내장됨) 으로 되어 있다.
- 그리고 endpoint 들을 드러낼 것이나 가릴 것이냐를 선택할 수 있음.
- 대부분의 endpoint 는 HTTP 이고, `/actuator/health` uri 는 `health` 엔드포인트와 매핑되어있다. 어플리케이션의 health 정보를 준다.

## 2.8. Health Information

health endpoint 는 어플리케이션이 다운된 상태나, 트래픽을 받지 못하는 상태를 나타낼 수 있다. 

이 endpoint 는 다음 두 가지 Properties 에 의해서 설정된다. 

- `management.endpoint.health.show-details`
- `management.endpoint.health.show-components`

| Value | Description |
|-------|-------------|
| never | Details are never shown. |
| when-authorized | Details are shown only to authorized users. Authorized roles can be configured by using `management.endpoint.health.roles`. |
| always | Details are shown to all users. |


- default value 는 never 이다.
- 권한을 주는 역할에 대해서 설정하고 싶다면 이 프로퍼티를 보자. `management.endpoint.health.roles`

Health Information 은 [HealthContributorRegistry](https://github.com/spring-projects/spring-boot/blob/v3.0.5/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthContributorRegistry.java) 에 의해서 수집되어진다.
- 모든 `HealthContributor` 인스턴스는 ApplicationContext 에 의해서 정의되어 있다.
- 스프링 부트는 자동 설정된 `HealthContributors` 가 여러개 있다. 

`HealthContributor` 는 `HealthIndicator` 이거나 `CompositeHealthContributor` 중 하나다. 
- `HealthIndicator` 는 실제 앱의 health 정보를 준다. `Status` 포함해서.
- `CompositeHealthContributor` 는 다른 `HealthIndicator` 의 정보를 복합해서 준다.
  - 트리 구조로 합쳐서 전체 시스템의 health 정보를 준다.

마지막 health 정보는 `StatusAggregator` 에 의해서 나온다.
- `HealthIndicator` 로 부터 나온 `status` 정보를 정렬하고 health 정보를 주는 것. 
- `HealthIndicator` 가 준 정보가 없다면 `UNKNOWN` 으로 나올 수 있음.

### 2.8.1. Auto-configured HealthIndicators

다음 표들의 `HealthIndicator` 들은 자동 설정으로 등록된 애들이다. 

다음 설정을 통해서 enable 하거나, disable 할 수 있다. `key` 에 다음 테이블의 키를 넣으면 된다. 
- `management.health.key.enabled`

| Key          | Name                         | Description                                       |
|--------------|------------------------------|---------------------------------------------------|
| cassandra    | CassandraDriverHealthIndicator | Checks that a Cassandra database is up.           |
| couchbase    | CouchbaseHealthIndicator      | Checks that a Couchbase cluster is up.            |
| db           | DataSourceHealthIndicator     | Checks that a connection to DataSource can be obtained. |
| diskspace    | DiskSpaceHealthIndicator      | Checks for low disk space.                        |
| elasticsearch | ElasticsearchRestHealthIndicator | Checks that an Elasticsearch cluster is up.      |
| hazelcast    | HazelcastHealthIndicator      | Checks that a Hazelcast server is up.             |
| influxdb     | InfluxDbHealthIndicator       | Checks that an InfluxDB server is up.             |
| jms          | JmsHealthIndicator            | Checks that a JMS broker is up.                   |
| ldap         | LdapHealthIndicator           | Checks that an LDAP server is up.                 |
| mail         | MailHealthIndicator           | Checks that a mail server is up.                  |
| mongo        | MongoHealthIndicator          | Checks that a Mongo database is up.               |
| neo4j        | Neo4jHealthIndicator          | Checks that a Neo4j database is up.               |
| ping         | PingHealthIndicator           | Always responds with UP.                          |
| rabbit       | RabbitHealthIndicator         | Checks that a Rabbit server is up.                |
| redis        | RedisHealthIndicator          | Checks that a Redis server is up.                 |

- `management.health.defaults.enabled` 를 통해서 한번에 disable 할 수도 있다. 

추가적인 `HealthIndicator` 들도 사용가능하다. 이건 기본적으로 등록되어 있진 않음.

| Key          | Name                           | Description                                         |
|--------------|--------------------------------|-----------------------------------------------------|
| livenessstate | LivenessStateHealthIndicator   | Exposes the “Liveness” application availability state. |
| readinessstate | ReadinessStateHealthIndicator | Exposes the “Readiness” application availability state. |


2.8.5. Health Groups

`HealthIndicator` 들을 Health Group 으로 묶을 때 유용한다.

group 을 만들고 싶다면 다음과 같이 하면 된다. 

```properties
management.endpoint.health.group.custom.include=db
```

- 이렇게 설정해놓으면 다음과 같이 check 할 수 있다. `localhost:8080/actuator/health/custom.`

제외하고 싶다면 이렇게 하면 된다.

```properties
management.endpoint.health.group.custom.exclude=db
```



## 2.9. Kubernetes Probes

K8s 에 배포된 app 들은 그들의 [container probe](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes) 에 대한 정보를 줄 수도 있다. 
- K8s 설정에 따라서 kubelet 이 probe 를 호출할 것.

Spring Boot 는 기본적으로 [Application Availability State](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.spring-application.application-availability) 를 관리한다.
- Application Availability State 는 Liveness State 와 Rediness State 를 줄 수 있다.
  - 이 둘은 `AvailabilityState`  인터페이스의 상속이다.
  - readiness 는 `ReadinessStateHealthIndicator` 에서 관리하고 
  - liveness 는 `LivenessStateHealthIndicator` 에서 관리한다.
- 이 정보를 actuator 가 모우고 이를 K8s 에 probe 를 호출하면 전달해줄 수 있는 것.
- ApplicationAvailability 에 주입하면 커스텀한 state 를 얻을 수도 있다.

endpoint 는 다음과 같다. 
- global endpoint: `actuator/health`
- Readiness endpoint: `/actuator/health/readiness`
- liveness endpoint: `actuator/health/liveness`

k8s 의 설정은 다음과 같을 것.

```yaml
livenessProbe:
  httpGet:
    path: "/actuator/health/liveness"
    port: <actuator-port>
  failureThreshold: ...
  periodSeconds: ...

readinessProbe:
  httpGet:
    path: "/actuator/health/readiness"
    port: <actuator-port>
  failureThreshold: ...
  periodSeconds: ...
```

health Group 은 k8s 환경에선느 자동적으로 된다. 이를 어떤 환경에서도 적용하고 싶다면 이 설정을 쓰자. 

```properties
management.endpoint.health.probes.enabled
```

어플리케이션의 시작이 k8s 의 `liveness Period` 보다 크다면 k8s 에서 `startupProbe` 를 가능한 솔루션으로도 쓸 수 있다. 
- `initialDelaySeconds` 를 써도 괜찮을 것 같긴하다. 
- startupProbe 의 코드는 다음과 같다. 

```yaml
startupProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 30
  periodSeconds: 10
```

- `failureThreshold * periodSeconds` 만큼 최대 시간 확보를 할 수 있다. 이 중에 한번이라도 성공하면 startup probe 는 없어지고 liveness probe 가 시작된다. 

## 2.9.1. Checking External State With Kubernetes Probes

actuator 는 liveness 와 readiness 를 healthGroup 에 포함시킨다. 이걸 k8s 에서 probe 할 때 쓴다는 듯. 

그리고 추가적인 HealthGroup 도 포함할 수 있다. 다음 예는 추가적인 HealthIndicator 를 포함시킨 것.  

```properties
management.endpoint.health.group.readiness.include=readinessState,customCheck
```

- 이렇게 하면 포함시킨다. 스프링 부트는 다른 `HealthIndicator` 는 포함시키지 않는다. 기본적으로. 

liveness state 는 기본적으로 외부 시스템에 의존하지 않는다. spring boot 에서 application context 가 refresh 했다면 liveness 는 up 상태가 됨.
- 만약 외부 시스템에 의존적이라면 시스템 붕괴 전파 (cascading) 이 일어난다. 
- liveness state 가 BROKEN 이라면 k8s 는 재시작으로 문제를 해결하려고 한다.

readiness state 같은 경우도 외부 시스템을 포함시키는 걸 조심해야함.
- 어플리케이션에 꼭 포함되지 않아도 되는 거라면 외부 시스템은 제외해야하고. 어플리케이션도 같이 붕괴될 수 있음을 알아야한다.

## 2.9.2. Application Lifecycle and Probe States

application 의 `AvailabilityState` 의 라이프 사이클을 알아야 실제 k8s probe 도 결과를 정확히 얻어올 것.

스프링 부트는 어플리케이션이 start 될 때와 shutdown 될 때 `AvailabilityState` 가 변화한다.

### 스타트 될 때 

| Startup phase | LivenessState | ReadinessState | HTTP server | Notes |
| --- | --- | --- | --- | --- |
| Starting | BROKEN | REFUSING_TRAFFIC | Not started | Kubernetes checks the "liveness" Probe and restarts the application if it takes too long. |
| Started | CORRECT | REFUSING_TRAFFIC | Refuses requests | The application context is refreshed. The application performs startup tasks and does not receive traffic yet. |
| Ready | CORRECT | ACCEPTING_TRAFFIC | Accepts requests | Startup tasks are finished. The application is receiving traffic. |

### shutdown 될 때 

| Shutdown phase | Liveness State | Readiness State | HTTP server | Notes |
| --- | --- | --- | --- | --- |
| Running | CORRECT | ACCEPTING_TRAFFIC | Accepts requests | Shutdown has been requested. |
| Graceful shutdown | CORRECT | REFUSING_TRAFFIC | New requests are rejected | If enabled, graceful shutdown processes in-flight requests. |
| Shutdown complete | N/A | N/A | Server is shut down | The application context is closed and the application is shut down. |


