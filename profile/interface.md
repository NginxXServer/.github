### 네트워크 인터페이스

1. OS 레벨 리다이렉션
   - 클라이언트가 HTTP 서버로 요청을 보내면 OS 레벨에서 자동으로 리버스 프록시 서버로 리다이렉션
   - `iptables -t nat -A PREROUTING -p tcp --dport 39020 -j REDIRECT --to-port 39071` 와 같이 iptable을 설정하는 권한이 필요함.
2. HTTP 서버 리다이렉션
   - HTTP 서버가 요청을 받으면 리버스 프록시 서버로 리다이렉션
   - 구현이 간단하지만 요청이 HTTP 서버까지 갔다가 리버스 프록시 서버로 돌아오므로 성능 저하 우려가 있음.
3. **클라이언트가 리버스 프록시로 직접 요청**
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
   │     ├── monitoring/
   │      │   ├── health.h
   │      │   ├── health.c
   │      │   ├── metrics.h
   │      │   └── metrics.c
   │       ├── utils/
   │       │   ├── config.h
   │       │   ├── config.c
   │       │   ├── logger.h
   │       │   └── logger.c
   │       ├── main.c
   │       ├── Makefile
   │       └── README.md
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

1. 서버 응답

   ```json
   (1)문서가 존재하는 경우
   HTTP/1.1 200 OK
   Content-Type: application/json

   {
       "docName": "example",
       "content": "This is the content of the example document."
   }

   (2)문서가 존재하지 않는 경우
   HTTP/1.1 404 Not Found
   Content-Type: application/json

   {
       "error": "Document not found"
   }

   에러코드는 4xx(클라이언트), 5xx(서버) 우선 구현
   ```

2. 로그 형식

   ```python
   [2024-11-21 14:30:15][INFO] 서버 시작: 포트 8080
   [2024-11-21 14:30:20][INFO] Client IP: 192.168.1.100, Status: 200, Response: {"docName": "example", "content": "..."}
   [2024-11-21 14:30:25][ERROR] Client IP: 192.168.1.101, Status: 404, Response: {"error": "Document not found"}
   ```

3. 리버스 프록시 서버 로그 형식

   ```python
   [2024-11-21 14:30:15][INFO] Backend server pool initialized with 5 servers
   [2024-11-21 14:30:20][INFO] New connection from 192.168.1.100
   [2024-11-21 14:30:20][INFO] Selected backend server 127.0.0.1:39020

   [2024-11-21 14:30:20][INFO] Client IP: 192.168.1.100, Status: 200, Response: {"docName": "example", "content": "..."}
   [2024-11-21 14:30:20][INFO] [METRIC][SERVER 127.0.0.1:39020] Active: 3, Total: 150, Failures: 2, Avg Response: 45.5ms
   [2024-11-21 14:30:20][INFO] [METRIC][SYSTEM] Total Requests: 500, Total Failures: 5, Avg Response: 48.2ms

   [2024-11-21 14:30:25][INFO] New connection from 192.168.1.101
   [2024-11-21 14:30:25][INFO] Selected backend server 127.0.0.1:39021
   [2024-11-21 14:30:25][ERROR] Client IP: 192.168.1.101, Status: 404, Response: {"error": "Document not found"}
   [2024-11-21 14:30:25][INFO] [METRIC][SERVER 127.0.0.1:39021] Active: 2, Total: 145, Failures: 3, Avg Response: 47.8ms
   [2024-11-21 14:30:25][INFO] [METRIC][SYSTEM] Total Requests: 501, Total Failures: 6, Avg Response: 48.3ms

   [2024-11-21 14:30:30][ERROR] [STATUS] Server 127.0.0.1:39021 marked as unhealthy
   ```

4. 버퍼의 크기
   - 클라이언트 요청 : 8192
   - 서버의 응답 : 9999
