### 네트워크 인터페이스

1. OS 레벨 리다이렉션
   - 클라이언트가 HTTP 서버로 요청을 보내면 OS 레벨에서 자동으로 리버스 프록시 서버로 리다이렉션
   - `iptables -t nat -A PREROUTING -p tcp --dport 39020 -j REDIRECT --to-port 39071` 와 같이 iptable을 설정하는 권한이 필요함.
2. HTTP 서버 리다이렉션
   - HTTP 서버가 요청을 받으면 리버스 프록시 서버로 리다이렉션
   - 구현이 간단하지만 요청이 HTTP 서버까지 갔다가 리버스 프록시 서버로 돌아오므로 성능 저하 우려가 있음.
3. 클라이언트가 리버스 프록시로 직접 요청
   - 구현이 간단하지만 클라이언트가 리버스 프록시 서버의 주소(포트 번호)를 알아야 함.

### 패키지 설계

1. 리버스 프록시

   ```
   reverse-proxy/
   ├── src/
   │   └── version1/
   │       ├── proxy/
   │       │   ├── proxy.h
   │       │   └── proxy.c
   │       ├── connection/
   │       │   ├── connection.h
   │       │   └── connection.c
   │       ├── utils/
   │       │   ├── config.h
   │       │   ├── config.c
   │       │   ├── logger.h
   │       │   └── logger.c
   │       ├── main.c
   │       ├── Makefile         # 버전1의 Makefile
   │       └── README.md
   │   └── version2/
   ├── tests/
   │   └── version1/
   │       ├── proxy_test.c
   │       └── connection_test.c
   └── README.md
   ```

2. HTTP 서버

   ```
   http-server/
   ├── src/
   │   └── version1/
   │       ├── server/
   │       │   ├── server.h
   │       │   └── server.c
   │       ├── document/
   │       │   ├── document.h
   │       │   └── document.c
   │       ├── http/
   │       │   ├── parser.h
   │       │   └── parser.c
   │       ├── utils/
   │       │   ├── config.h
   │       │   ├── config.c
   │       │   ├── logger.h
   │       │   └── logger.c
   │       ├── main.c
   │       ├── Makefile         # 버전1의 Makefile
   │       └── README.md
   │   └── version2/
   ├── tests/
   │   └── version1/
   │       ├── server_test.c
   │       ├── document_test.c
   │       └── parser_test.c
   ├── docs/                    # 문서 저장
   └── README.md
   ```

3. 클라이언트

   ```
   client/
   ├── src/
   │   └── version1/
   │       ├── client/
   │       │   ├── client.h
   │       │   └── client.c
   │       ├── http/
   │       │   ├── request.h
   │       │   ├── request.c
   │       │   ├── response.h
   │       │   └── response.c
   │       ├── utils/
   │       │   ├── config.h
   │       │   ├── config.c
   │       │   ├── logger.h
   │       │   └── logger.c
   │       ├── main.c
   │       ├── Makefile         # 버전1의 Makefile
   │       └── README.md
   │   └── version2/
   ├── tests/
   │   └── version1/
   │       ├── client_test.c
   │       ├── request_test.c
   │       └── response_test.c
   └── README.md
   ```

### 인터페이스 설계
