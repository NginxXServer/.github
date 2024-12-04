# Quick Start

## DEMO

- Test Viedo : https://www.youtube.com/watch?v=hVOxU1w11hc
- Demo Viedo : https://www.youtube.com/watch?v=bwovJOpD3Dk

<br/>

## 버전 설명

### Version 1

- 리버스 프록시 서버와 HTTP 서버 모두 단일 프로세스, 순차 처리

### Version 2

-  리버스 프록시 서버는 멀티 스레드(Thread per Connection), 로드밸런싱(RR) 사용
- HTTP 서버는 멀티 프로세스로 분산 실행

### Version 3

- 리버스 프록시 서버는 멀티스레드 + epoll LT 사용

- HTTP 서버는 멀티프로세스 + epoll ET 사용

### Version 4

- 리버스 프록시 서버와 HTTP 서버 모두 Thread Pool 사용

<br/>

## Quick Start

**1. git clone**

- siege를 통한 테스트를 위해서는 [NginxX-Client](https://github.com/NginxXServer/NginxX-Client)를 받아야 합니다.

- 사용을 위해서는 [NginxX-ProxyServer](https://github.com/NginxXServer/NginxX-ProxyServer), [NginxX-HttpServer](https://github.com/NginxXServer/NginxX-HttpServer)를 받아야 합니다.

**2. Build**

- 테스트와 사용을 위해서는 build가 우선되어야 합니다.
- 현재 디렉토리 구조는 다음과 같습니다.
  ```
  server/
  ├── src/
  │   └── version1/
  │       ├── proxy/
  │       │   ├── proxy.h
  │       │   └── proxy.c
  │       ├── connection/
  │       │   ├── connection.h
  │       │   └── connection.c
  │       ├── monitoring/
  │       │   ├── health.h
  │       │   ├── health.c
  │       │   ├── metrics.h
  │       │   └── metrics.c
  │       ├── utils/
  │       │   ├── logger.h
  │       │   └── logger.c
  │       ├── main.c
  │       ├── Makefile
  │       └── README.md
  ```
- 버전 폴더에 들어가서 `make` 명령어를 통해 빌드할 수 있습니다.

**3. 실행**

1. `NginxX-HttpServer/src/version#` 디렉토리의 `make`를 통해 생성된 실행파일 `http_server` 실행
2. `NginxX-ProxyServer/src/version#` 디렉토리의 `make`를 통해 생성된 실행파일 `reverseProxy` 실행

- 두 서버를 모두 실행한 뒤 39071 포트로 요청
- 요청 예시 `http://10.198.138.213:39071/?docName=index.html`

**4. 테스트**

- [How to Test](https://github.com/NginxXServer/NginxX-Client/blob/main/README.md)
