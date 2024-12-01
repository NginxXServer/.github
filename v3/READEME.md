### 기한: ~ 11.25

# Team

- 웹브라우저를 통한 이미지, html 테스팅 진행
- wrk를 통한 통합 테스팅 진행

# 김지유

- HTTP 서버 epoll 이벤트 처리 기반 전환
  - epoll 초기화
    서버 시작 시 `epoll_create`를 호출하여 epoll 인스턴스 생성.
    서버 소켓을 `EPOLLIN` 이벤트로 `epoll_ctl`에 등록.
  - 이벤트 대기 및 처리
    기존 `accept` 루프를 `epoll_wait` 기반으로 변경.
- 클라이언트 요청 비동기 처리
  - 읽기/쓰기 이벤트 처리
  - 멀티플렉싱 처리
- Edge Trigger 방식 사용

# 조준화

- 버전2 오류 해결
  - HTTP를 온전히 읽지 못하는 오류 해결
- 웹브라우저를 통한 HTTP 서버 접근 방식 조사  
  proxy 서버에 접근 :  
  http://10.198.138.213:39071/?docName=a  
  http 서버에 접근 :  
  http://10.198.138.212:39020/?docName=a
- 기존 멀티스레드 코드 제거
  - thread-per-connection 방식 제거
- epoll 기반 구조로 전환
  - epoll 인스턴스 생성 및 초기화
  - listen socket을 epoll에 등록
  - non-blocking socket으로 설정
- 기존 기능들을 epoll 구조에 맞게 수정
- Level Trigger 방식 사용
- **연결 흐름 정리**

  1. 클라이언트 연결 요청 → epoll이 감지
  2. handle_client() 호출

     - 서버 선택
     - 백엔드 소켓 생성 (non-blocking으로 설정)
     - 백엔드 연결 시작
     - 즉시 리턴하여 다음 클라이언트 연결을 받을 준비

  3. 백엔드 연결 완료 → epoll이 감지

     - 요청 전송 시작
     - 즉시 리턴

  4. 백엔드로부터 응답 수신 가능 → epoll이 감지

     - 응답 수신
     - 클라이언트에 전송
     - 즉시 리턴

# 정한울

- 로드밸런싱 RR 알고리즘 개발 (ver RR)
- 로드밸런싱 LC 알고리즘 개발 (ver LC)
  - 헬스체크 기능 활용
  - 현재 처리하고 있는 요청이 가장 적은 서버 선택
