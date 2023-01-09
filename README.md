volt gateway
===

# volt-gw

이 프로젝트는 개인프로젝트로써, 고성능의 API G/W를 제공하기 위해 만들어 졌습니다.\
기본적으로 아래와 같은 기능이 제공이 됩니다.

* api gateway(proxy) : http, https, ws, wss
* ingress controller (kubernetes, k8s)
* ssh passthrough
* api management (통합 예정)

admin을 통해 쉬운 설정이 가능한 apim, 그리고 통계 및 분석이 통합될 예정입니다.

# Install and Run

## docker

### pre require

1. docker-compose

### run

```
git clone git@github.com:sanjuck/volt.git
cd volt

docker-compose up
```

gw테스트는 http://localhost:9080/todos 로 확인하실 수 있습니다. \
\
현재 버전에는 미완성의 apim과 sso을 위한 keycloak가 통합되어 있습니다.\
apim : http://localhost:9081 \
keycloak: http://localhost:9082 (default id/pw : admin/admin) \

# server configuration

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

# gateway configuration

## proxy

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

# docker (docker-compose)

```
version: "3.7"
services:
  volt-gateway:
    image: sanjuck/volt-gateway:latest
    network_mode: bridge
    volumes:
      - ./conf/:/app/conf/
      - ./logger.yaml:/app/logger.yaml
    ports:
      - "9443:9443"
      - "9080:9080"
```

# k8s (ready)
