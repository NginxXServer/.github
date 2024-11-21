## 💻 프로젝트 개요

### 주요 기능

1. 기본적인 HTTP 요청, 응답 처리
   - GET 요청에 대해서 클라이언트의 입력에 따른 document를 반환
2. HTTP 메소드 구분
   - 단, content-type은 JSON만 다룸
3. 로드밸런서, 리버스 프록시
4. 서버 헬스 체크

### 특장점

1. epoll 기반의 이벤트 처리
2. 스레드 풀 사용
3. 리버스 프록시 및 로드밸런싱
   - RR : RoundRobin 방식으로 단순하게 모든 서버에 균등하게 요청을 분배함. 모든 서버의 성능이 같은 swist 서버 환경의 특성상 적합하다고 판단
   - LC : 실시간 서버 부하 상태를 반영하여 효율적으로 부하를 분산하지만 구현이 복잡함
4. 주기적인 서버 모니터링

## 💡 프로젝트 평가

**wrk**

- 다른 웹 서버 벤치마크에 비해 가벼우면서도 마이크로초 단위의 정밀한 타이밍 측정이 가능하고 통계 데이터, 상세한 에러 리포팅을 제공

- wrk를 통한 Throughput, 응답 시간, 에러율, 동시 사용자 수 평가

- 정적 부하 테스트, 최대 부하 테스트, 스파이크 테스트 진행

- wrk를 활용한 테스트 자동화 구현 예정
- wrk의 결과를 바탕으로 각 버전별 그래프를 생성하여 시각적으로 성능 변화를 확인하고 최대한 성능을 개선하는 것이 목표

## 🗂️ System Architecture

![system architecture](architecture.png)
[인터페이스 설계](./interface.md)

## 📜 Development History

### Version 1

- HTTP 서버, 리버스 프록시 서버, 클라이언트를 개발 및 통합 테스트 진행

- [v1 개발 과정 보기](../v1/version1.md)

### Version 2

## 🔗 Repository

- [HTTP Server](https://github.com/NginxXServer/NginxX-HttpServer)
- [Reverse Proxy Server](https://github.com/NginxXServer/NginxX-ProxyServer)
- [Client](https://github.com/NginxXServer/NginxX-Client)
