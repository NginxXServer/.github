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
   │   ├── main.c              # 프로그램 시작점
   │   ├── proxy/
   │   │   ├── proxy.h        # 프록시 서버 핵심 기능
   │   │   └── proxy.c
   │   ├── connection/
   │   │   ├── connection.h   # 연결 관리 (백엔드 서버와의 연결)
   │   │   └── connection.c
   │   └── utils/
   │       ├── config.h       # 설정 관리 (포트 번호 등)
   │       ├── config.c
   │       ├── logger.h       # 로깅
   │       └── logger.c
   ├── tests/
   │   ├── proxy_test.c
   │   └── connection_test.c
   ├── Makefile
   └── README.md
   ```

2. HTTP 서버

   ```
   http-server/
   ├── src/
   │   ├── main.c              # 프로그램 시작점
   │   ├── server/
   │   │   ├── server.h       # HTTP 서버 핵심 기능
   │   │   └── server.c
   │   ├── document/
   │   │   ├── document.h     # 문서 조회 기능
   │   │   └── document.c
   │   ├── http/
   │   │   ├── parser.h       # HTTP 요청 파싱
   │   │   └── parser.c
   │   └── utils/
   │       ├── config.h       # 설정 (포트, 문서 디렉토리 경로 등)
   │       ├── config.c
   │       ├── logger.h
   │       └── logger.c
   ├── tests/
   │   ├── server_test.c
   │   ├── document_test.c    # 문서 조회 테스트
   │   └── parser_test.c
   ├── data/                   # 문서 저장 디렉토리
   ├── Makefile
   └── README.md
   ```

3. 클라이언트

   ```
   client/
   ├── src/
   │   ├── main.c              # 프로그램 시작점
   │   ├── client/
   │   │   ├── client.h       # 클라이언트 핵심 기능
   │   │   └── client.c
   │   ├── http/
   │   │   ├── request.h      # HTTP 요청 생성
   │   │   ├── request.c
   │   │   ├── response.h     # HTTP 응답 처리
   │   │   └── response.c
   │   └── utils/
   │       ├── config.h       # 설정 (서버 주소, 포트 등)
   │       ├── config.c
   │       ├── logger.h
   │       └── logger.c
   ├── tests/
   │   ├── client_test.c
   │   ├── request_test.c
   │   └── response_test.c
   ├── Makefile
   └── README.md
   ```

### 인터페이스 설계
