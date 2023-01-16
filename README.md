volt gateway
===

# volt-gw

이 프로젝트는 고성능의 API G/W를 제공하기 위해 만들었습니다

* api gateway(proxy) : http, https, ws, wss
* ingress controller (kubernetes, k8s)
* ssh passthrough
* api management (ongoing)
* 모니터링 (ongoing)
* 통계 및 분석 (ongoing)

# Install and Run

## pre require

1. docker-compose

## run

```
git clone git@github.com:sanjuck/volt.git
cd volt

docker-compose up
```

gw테스트는 http://localhost:9080/todos 로 확인하실 수 있습니다.

현재 버전에는 미완성의 apim과 sso을 위한 keycloak가 통합되어 있습니다
* apim : http://localhost:9081
* keycloak: http://localhost:9082 (default id/pw : admin/admin)

# Gateway Configuration

## Server Configuration(conf/volt.conf)

```
servers:
  - protocol: https
    port: 9443

  - protocol: http
    port: 9080

auth:
  keycloak:
    key-server-url-pattern: "%s/auth/realms/%s/protocol/openid-connect/certs"

global:
  include-d-conf-pattern: d\.ya?ml$  # gateway configuration file format (default path: ./conf)
  host: localhost
  server:
    proxy-insecure-skip-verify: true
    proxy-timeout: 30
    websocket-timeout: 45
    passthru-timeout: 30
    dns-resolver: '8.8.8.8:53'
```

## Proxy Configuration

### prefix path
```
apis:
  - host: localhost
    path: /todos
    path-type: prefix
    target:
      path: /todos
      members:
      - host: jsonplaceholder.typicode.com
        port: 443
        protocol: https
```
GET https://localhost:9443/todos/1

```
{
  "userId": 1,
  "id": 1,
  "title": "delectus aut autem",
  "completed": false
}
```

### path variable
```
  - host: localhost
    path: /ctx/*
    target:
      path: /todos/{1}
      members:
      - host: jsonplaceholder.typicode.com
        port: 443
        protocol: https

  - host: localhost
    path: /ctx/**/*
    target:
      path: /todos/{2}
      members:
      - host: jsonplaceholder.typicode.com
        port: 443
        protocol: https
```

## tls cert

```
tlss:
  - host: localhost
    key: |
      -----BEGIN PRIVATE KEY-----
      ...
      -----END PRIVATE KEY-----
    crt: |
      -----BEGIN CERTIFICATE-----
      ...
      -----END CERTIFICATE-----
```

## passthrough

```
passthrus:
  - host: local.s.com
    members:
    - host: www.donga.com
      port: 443

```

# Monitoring

Gateway에 fluentbit에 통합되어 있습니다.

```mermaid
flowchart LR
    A[G/W fluentbit] .-> B[Fluentbit Collector] --> D[Grafana]
    A .-> C[(influxdb)] --> D
```

ex)
```
  volt-gateway:
    environment:
      TZ: "Asia/Seoul"
      FLUENTBIT_OUTPUT: "influxdb -p host=influxdb -p bucket=fluentbit -p sequence_tag=_seq"
      # FLUENTBIT_OUTPUT: "forward ..."
```
fluentbit output option documantaion : https://docs.fluentbit.io/manual/pipeline/outputs


# Docker (docker-compose)

[docker-compose.yaml 파일 참조](./docker-compose.yaml)

# K8S (ready)
