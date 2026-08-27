---
title: AI Inference Server
date: 2026-08-27

tags: [Python, SQLAlchemy, Redis, RabbitMQ, SSE]
description: "SQLAlchemy와 PostgreSQL을 기반으로 비동기 작업 처리 시스템을 구축하고, Redis와 RabbitMQ를 활용해 실시간 상태 업데이트 및 안정적인 배포 환경을 구현하는 과정을 다룹니다."

category: project
thumbnail: ""
draft: false
---

<br>

# Concepts  

<br>

**SQLAlchemy**  

<br>

Python의 DB 라이브러리.  
SQL과 DB 작업을 Python 객체를 통해 조작할 수 있게 한다.  
Django ORM과 달리 독깁적인 라이브러리지만,  
- 어떤 Python 프로젝트에서도 사용할 수 있고  
- 복잡하거나 특수한 DB 요구사항에 더 잘 대응할 수 있다는 장점이 있다.  

<br>

**PostgreSQL**  

<br>

실제 데이터를 저장하는 ORDBMS(객체지향형DB)이다.  
MySQL에 비해서 SQL 표준을 더 잘 지원하고 기능이 더 강력하며  
쿼리가 복잡해질수록 성능이 더 잘 나오는 편이다.  
그러나 조회성 트랜잭션에서 파일베이스 아키텍처의 IO 한계로 인해 타 DBMS에 비해 우수한 성능을 내기 어렵다는 단점이 있다.  
Uber에서는 PostgreSQL을 메인으로 쓰다가 Scalability 문제를 극복하지 못하고 MySQL로 전환하기도 했단다.  

<br>

**CRUD (Create, Read, Update, Delete)**  

<br>

| CRUD   | 의미  | DB (SQL) | REST API (HTTP) |  
| ------ | --- | -------- | --------------- |  
| Create | 생성  | INSERT   | POST            |  
| Read   | 조회  | SELECT   | GET             |  
| Update | 수정  | UPDATE   | PUT/PATCH       |  
| Delete | 삭제  | DELETE   | DELETE          |  

<br>

**REST API vs RESTful API**  

<br>

1. REST API: 개념(아키텍처 스타일)에 초점을 맞춘 포괄적인 용어이다. HTTP 기반으로 클라이언트와 서버가 통신하는 API라면 일반적으로 이에 해당한다.  
2. RESTful API: 'REST의 원리를 잘 지켜서 설계된(Full)' 구현 상태를 강조한다. 예를 들어, URL에 행위(동사)를 포함하지 않고 오직 자원의 명사로만 구성하며 적절한 HTTP 메서드(GET, POST, PUT, DELETE)를 사용하는 등 엄격한 규칙을 따른 API이다.  

<br>

지금 만들고 있는 파이썬 기반의 AI 기반 서버 초기 구조다.  
```  
Browser
    ↓ HTTP
FastAPI
    ↓ SQLAlchemy
PostgreSQL
    ↓
Data
```  
- 예를 들어 `POST/users` 요청이 오면 `FastAPI`를 통해 `SQLALchemy`에서 `INSERT INTO users`라는 쿼리가 일어나 `PostgreSQL`에 쌓이는 것이다.  
- PostgreSQL Image는 Docker를 통해 구축했기 때문에 별도의 IDE를 통한 관리는 불필요 하다.  
- Uvicorn은 웹 서버고, FastAPI는 웹 프레임워크다. 그러니까 전달자와 객체의 관계인 셈이다.  

<br>

**ORM (Object-Relational Mapping)**  

<br>

[03-02 ORM(Object-Relational Mapping) 활용 가이드 - Amazing Python Universe](https://wikidocs.net/295889)  

<br>

ORM은 DB의 테이블을 Python의 클래스(Class)로, 테이블의 행을 클래스의 객체(Object)로 매핑하는 기술이다.  
- DB: 데이터를 구조화하고 저장하는 공간  
- 테이블: 데이터의 집합을 나타내는 구조  

<br>

다음과 같은 이점이 있다.  
- 생산성 향상: SQL 쿼리를 직접 작성하지 않고 파이썬 코드로 DB를 제어할 수 있다.  
- 코드 가독성 증가: DB 작업이 Python OOP 지향 문법으로 표현되어 코드가 더 직관적이고 이해하기 쉬워진다.  
- 유지보수 용이: DB 구조가 변경되어도 코드의 수정이 최소화된다.  

<br>

**HTTP**  

<br>

HTTP 요청 메서드: [2. GET / POST / PUT / PATCH / DELETE - FastAPI로 배우는 백엔드](https://wikidocs.net/312163)  

<br>

| CRUD   | HTTP 메서드    | 설명         |  
| ------ | ----------- | ---------- |  
| Create | POST        | 새로운 리소스 생성 |  
| Read   | GET         | 기존 리소스 조회  |  
| Update | PUT / PATCH | 기존 리소스 수정  |  
| Delete | DELETE      | 기존 리소스 삭제  |  

<br>

**Worker**  

<br>

메인 서버나 스레드가 처리하기 무거운 작업, 오래 걸리는 작업, 또는 독립적인 비동기 작업을 백그라운드에서 대신 처리해주는 독립적인 프로그램 단위  
- 역할 분리: 메인 서버는 사용자의 요청을 받고 응답하는 데 집중, 시간과 리소스가 많이 드는 작업은 워커(Worker) 프로세스나 스레드로 넘겨 처리 속도 유지  
- 비동기 큐 기반: 주로 메시지 큐(Message Queue)를 사용하여 메인 서버가 작업을 큐에 넣으면, 워커가 이를 가져와 처리하고 결과를 업데이트하는 방식  

<br>

시스템 아키텍처로 표현하자면 다음과 같다.  
- API 서버: 사용자의 명령을 접수하고 즉각적인 응답 반환, 복잡한 작업은 큐(RadditMQ, Redis 등)에 적재  
- 작업 큐(Job Queue): 대기 중인 작업 목록 관리  
- Worker Pool: 큐에 있는 작업을 하나씩 꺼내어 독립적으로 처리  

<br>

그러니까 FastAPI 서버와 워커 프로세스를 완전히 물리적으로 분리하고, 중간에 Redis같은 브로커를 두어 작업을 관리하는 방식이다.  
`uv` 환경에서는 Celery 외에도 비동기 친화적인 Taskiq 라이브러리가 자주 쓰인다.  
- 장점: 서버가 다운되어도 작업이 큐에 안전하게 보존되며, 워커 수 확장(Scale-out)이 자유롭다.  
- 단점: 인프라(Redis 등) 관리가 필요하고 아키텍처가 복잡해진다.  

<br>

실제로 `uv` 환경에서 다음과 같이 터미널을 분리하여 API 서버와 워커 프로세스를 각각 실행한다.  
- API 서버 실행 `uv run uvicorn main:app --reload`  
- 워커 실행: `uv run celery -A tasks.celery_app worker --loglevel=info`  

<br>

만약 워커가 DB나 무거운 외부 라이브러리(Pandas, PyTorch 등)를 사용한다면 API 서버와 워커의 `requirements`를 명확히 분리하여 베포 컨테이너 크기를 최적화 하는 것이 좋다.  

<br>

그리고 Celery는 기본적으로 Sync 방식 기반이기에 FastAPI의 `async def` 기반 DB와 연동할 때 설정이 번거로울 수 있다.  

<br>

**CI/CD**  

<br>

CI(지속적 통합)  
- 개발자들이 작성한 코드를 중앙 저장소(Git 등)에 병합하기 전에 자동으로 빌드하고 테스트하는 과정  
- 목적은 여러 개발자가 동시게 코드를 수정하더라도 충돌을 미리 방지하고 코드의 오류를 조기에 발견하기 위함이다.  

<br>

CD(지속적 베포)  
- CI를 통과한 코드를 자동으로 테스트 및 운영 환경에 베포하여 사용자가 항상 최신 소프트웨어를 사용할 수 있게 만드는 과정  
- 사람이 직접 수동으로 베포할 때 발생하는 실수를 줄이고 새로운 기능이나 업데이트를 빠르고 주기적으로 고객에게 제공한다.  

<br>

**Docker Compose**  

<br>

[도커 컴포즈(Docker compose) - 개념 정리 및 사용법 — SH's Devlog](https://seosh817.tistory.com/387#google_vignette)  

<br>

Docker Compose는 단일 서버에서 여러 개의 컨테이너를 하나의 서비스로 정의해 컨테이너의 묶음으로 관리할 수 있는 작업 환경  

<br>

예를 들어, 웹 어필리케이션을 테스트 하려면  
- 웹 서버 컨테이너  
- DB 컨테이너  
두 개의 컨테이너를 각각 생성해야 한다.  

<br>

Docker Compose는 여러 개의 컨테이너의 옵션과 환경을 정의한 파일을 읽어 컨테이너를 순차적으로 생성하는 방식으로 동작한다.  
필요한 파일은 다음과 같다. 기존에 사용하던 `docker run` 명령어를 `yaml` 파일로 변환한다고 생각하면 된다.  
- `docker-compose.yml`  

<br>

1. Docker Image: 응용프로그램의 종속성과 함께 응용프로그램 자체를 패키징 or 캡슐화하여 격리된 공간에서 프로세스를 동작시키는 기술  
2. Docker Container: 서비스 운영에 필요한 서버 프로그램, 소스코드 및 라이브러리, 컴파일 된 실행 파일을 묶는 형태  

<br>

| 단계  | 작업            | 사용하는 창                           |  
| --- | ------------- | -------------------------------- |  
| 1   | 코드 수정         | VSCode Editor                    |  
| 2   | 서버 실행 및 로그 확인 | Terminal 1 (`docker compose up`) |  
| 3   | API 테스트       | Browser (`/docs`)                |  
| 4   | DB 확인         | Terminal 3 (`psql`)              |  
| 5   | Git Commit    | Terminal 2                       |  

<br>

**Refactor: Repository Pattern**  

<br>

[FastAPI 디자인 패턴-레포지토리 패턴](https://databoom.tistory.com/entry/FastAPI-%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4-%EB%A0%88%ED%8F%AC%EC%A7%80%ED%86%A0%EB%A6%AC-%ED%8C%A8%ED%84%B411-6)  

<br>

레포지토리 패턴(Repository Pattern)은 DB 엑세스를 서비소 로직(Service Layer)에서 분리하는 패턴이다.  
- DB와 직접 상호작용하는 코드인 Query 부분을 레포지토리 파일에 따로 관리  
- 서비스 레이어는 이 레포지토리를 호출하여 데이터를 가져오는 방식  

<br>

다음과 같튼 효과를 얻을 수 있다.  
1. DB 종속성 제거: 서비스 레이어에서 직접 DB 모델을 다루지 않아 DB 변경 시 최소한의 코드만 수정하면 된다.  
2. 재사용성 증가: 동일한 DB 조회/저장 로직을 여러 서비스에서 재사용 가능하다.  
3. 테스트 용이성: 가짜(Faker) DB를 사용해 테스트가 가능하다. `Mock DB`  
4. 코드 가독성 향상: SQLALchemy 관련 로직이 분리되어 서비스 코드가 더 깔끔해진다.  

<br>

표준적인 FastAPI 아키텍처에서 보면:  
- Router: HTTP 요청/응답 처리  
- Service: 비즈니스 로직 처리  
- Repository: DB 접근/CRUD 처리  
- Schema: 요청/응답 데이터 계약  
위와 같다.  

<br>

**Git Commit Messages**  

<br>

Commit Message의 7가지 규칙은 다음과 같다:  
1. 제목과 본문을 빈 행으로 구분한다.  
2. 제목은 50글자 이내로 제한한다.  
3. 제목의 첫 글자는 대문자로 작성한다.  
4. 제목 끝에는 마침표를 넣지 않는다.  
5. 제목은 명령문으로 사용하며 과거형을 사용하지 않는다.  
6. 본문의 각 행은 72글자 내로 제한한다.  
7. 어떻게 보다는 무엇과 왜를 설명한다.  

<br>

| 타입 이름    | 내용                               |  
| -------- | -------------------------------- |  
| feat     | 새로운 기능에 대한 커밋                    |  
| fix      | 버그 수정에 대한 커밋                     |  
| build    | 빌드 관련 파일 수정 / 모듈 설치 또는 삭제에 대한 커밋 |  
| chore    | 그 외 자잘한 수정에 대한 커밋                |  
| ci       | ci 관련 설정 수정에 대한 커밋               |  
| docs     | 문서 수정에 대한 커밋                     |  
| style    | 코드 스타일 혹은 포맷 등에 관한 커밋            |  
| refactor | 코드 리팩토링에 대한 커밋                   |  
| test     | 테스트 코드 수정에 대한 커밋                 |  
| perf     | 성능 개선에 대한 커밋                     |  

<br>

**Dockerfile vs docker-compose.yml**  

<br>

[dockerFile과 docker-compose.yml 의 차이점](https://velog.io/@s2moon98/dockerFile%EA%B3%BC-docker-compose.yml-%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90)  

<br>

Dockerfile: 이미지 빌드  
docker-compose.yml: 앱이 실행되는 동안의 컨테이너 관리  

<br>

**Vibe Coding**  

<br>

[Vibe Coding 매뉴얼: AI 지원 개발을 위한 템플릿 - ROBOCO](https://roboco.io/posts/vibe-coding-manual/)  

<br>

"Vibe Coding 튜토리얼 및 모범 사례"에서 소개된 이 개념은 세 가지 핵심 기둥에 기반한다:  
1. 명세(Specification): 목표를 정의한다.(예: "로그인 기능이 있는 Twitter 클론 구축")  
2. 규칙(Rules): 명시적인 제약 조건을 설정한다.(예: "Python 사용, 복잡성 피하기")  
3. 감독(Oversight): 프로세스를 모니터링하고 조정하여 일관성을 보장한다.  

<br>

**전반적인 웹의 흐름**  

<br>

현대 웹 아키텍처는 과거의 단순한 '사용자-서버-DB' 구조를 넘어 엣지 컴퓨팅, 비동기 이벤트 기반 처리, 그리고 AI 프롬프트 및 데이터 파이프라인의 효율화를 중심으로 진화하고 있다.  

<br>


```math
\text{유저} \longrightarrow \text{브라우저} \overset{\text{HTTP}}{\longrightarrow} \underbrace{\text{WEB (Edge/CDN)}}_{\text{정적 자원 즉시 반환}} \longrightarrow \underbrace{\text{WAS (Serverless/API)}}_{\text{비즈니스 로직 수행}} \begin{cases} \longleftrightarrow \text{DB (Serverless/Vector)} \\ \overset{\text{Queue}}{\longrightarrow} \text{Worker (비동기 heavy 작업/AI 연산)} \end{cases}
```


<br>

현재 트렌드의 핵심은 "유저가 느끼는 속도는 브라우저와 WEB(Edge)단에서 최대한 빠르게 만들고, 무겁고 복잡한 연산은 WAS와 DB 부담을 줄이기 위해 Worker를 통해 뒤로 돌린다"로 요약할 수 있다.  

<br>

(1) 유저(User)  

<br>

서비스를 이용하는 최종 소비자, 브라우저를 통해 시스템에 이벤트를 발생시키는 주체이다.  

<br>

현대의 유저는 초저지연(Zero-Latency)과 개인화(Hyper-Personalization)를 요구한다.  
AI의 발전으로 유저의 단순 클릭뿐만 아니라  
- 음성  
- 시선 트래킹  
- LLM과의 대화형 인터랙션  
등이 새로운 유저 행동 패턴으로 자리 잡았다.  

<br>

(2) 브라우저(Browser)  

<br>

HTML, CSS, JavaScript를 해석해 유저에게 화면을 시각적으로 보여주는 클라이언트 소프트웨어이다.  

<br>

현재는 단순한 화면 뷰어를 넘어 하나의 거대한 애플리케이션 플랫폼이 되었다.  
- Edge 및 SSR/SSG: Next.js, Remix 같은 메타 프레임워크의 발전으로 서버(혹은 전 세계에 분산된 Edge 서버)에서 미리 렌더링된 화면을 브라우저가 받아와 초기 로딩 속도(LCP)를 극대화한다.  
- Browser-side AI: WebGPU의 발전으로 서버를 거치지 않고 브라우저 자체에서 가벼운 AI 모델을 구동하는 기술이 트렌드로 떠오르고 있다.  

<br>

(3) WEB(Web Server)  

<br>

클라이언트(브라우저)의 HTTP 요청을 가장 먼저 맞이하여 정적 자원(HTML, CSS, 이미지)을 반환하거나, 요청을 뒤의 WAS로 넘겨주는(Reverse Proxy) 역할을 한다. `Nginx, Apache`  

<br>

현재 트렌드는 클라우드 네이티브와 Edge(CDN)로의 융합이다.  
- 이제는 단일 물리 서버에 Nginx를 띄우기보다 Vercel, Cloudflare Pages, AWS CloudFront 같은 글로벌 CDN 및 Edge 네트워크가 WEB 서버의 역할을 대신하는 경우가 많다.  
- 유저와 가장 가까운 전 세계 정거장에서 정적 파일을 즉시 서빙하도록 되어있다.  

<br>

(4) WAS(Web Application Server)  

<br>

비즈니스 로직을 수행하고, 데이터베이스와 상호작용하여 동적인 결과물을 만들어내는 핵심 서버이다. `Spring Boot, Node.js, FastAPI, Tomcat`  

<br>

'서버리스(Serverless)'와 '서버 기능(Server Functions)'의 대중화:  
- 서버리스 & 컨테이너: 24시간 켜져 있는 거대한 Monolithic WAS 대신, 요청이 올 때만 잠깐 켜져서 로직을 수행하는 AWS Lambda 같은 Serverless 아키텍처나 가벼운 Microservices(MSA)가 주류이다.  
- 프론트-백엔드 경계 붕괴: Next.js의 Server Actions처럼 프런트엔드 코드 내에서 직접 백엔드(WAS) 함수를 호출하는 구조가 트렌드로 자리 잡아, 개발 생산성이 극대화되었다.  

<br>

(5) DB(Database)  

<br>

애플리케이션의 상태와 유저 데이터를 영구적으로 저장하고 관리하는 시스템입니다. `MySQL, PostgreSQL, MongoDB`  

<br>

현재 트렌드는 벡터 DB(Vector DB)의 부상과 분산 글로벌 DB이다.  
- Vector DB `Pgvector, Pinecone`: AI/LLM 트렌드에 맞춰 텍스트나 이미지의 의미를 숫자(벡터)로 저장하고 유사도를 검색하는 기능이 필수가 되었다.      
- Serverless DB: Supabase, PlanetScale처럼 트래픽에 따라 스토리지와 컴퓨팅 자원이 자동으로 늘어나는 서버리스 DB가 각광받고 있다.  

<br>

(6) Worker(Background Worker / Queue Worker)  

<br>

WAS가 메인 스레드에서 처리하기엔 너무 무겁거나 시간이 오래 걸리는 작업(메일 대량 발송, 이미지/비디오 인코딩 등)을 비동기로 넘겨받아 백그라운드에서 처리하는 독립된 주체이다. `Celery, BullMQ, AWS SQS`  

<br>

현재 트렌드는 이벤트 기반 아키텍처(EDA)와 AI 파이프라인의 핵심 축이다.  
- 유저가 AI 이미지 생성을 요청했을 때, WAS가 결과를 기다리면 브라우저 요청이 만료(Timeout)된다.  
- 따라서 WAS는 요청을 큐(Queue)에 넣고 유저에게 "접수 완료"를 바로 알린 뒤,  
- Worker가 뒤에서 무거운 GPU 연산이나 AI API 호출을 처리한 후 DB를 업데이트하거나 알림을 보낸다.  
- Modern 웹에서 사용자 경험(UX)을 유지하기 위한 필수 요소이다.  

<br>

**Polling vs Event-driven**  

<br>

시스템 내에 동작 중에 Polling 방식과 Event-driven 방식이 있다.  

<br>

Polling 방식:  

<br>

어떤 상태인지를 주기적으로 확인해보는 것  
예를 든다면 우편물이 왔는지를 매번 내가 가서 보는 것이다.  
이렇게 매번 오가는게 Polling이다.  
주기적으로 알아보는 만큼 오지 않았을 때 나가보는 동안 비효율이 발생을 한다.  

<br>

Event-driven 방식:  

<br>

어떤 상태가 되면 알려주는 것  
매번 가는 것이 아니라 우편물이 도착했을 때 문자를 보내는 것이다. 훨씬 효율적일 수 있다.  
이벤트 방식으로 해당 사람이 오면 알려 주는 방식이다.  

<br>

두 방식에는 차이가 있지만, 언듯 Polling 방식은 비효율적일 거 같다는 생각을 할 수 있다.  
하지만 정기적으로 뭔가를 감시하거나 검사를 해야 한다면 Polling 방식도 필요할 것이다.  
그러나 Event-driven 방식을 통해서 트리거를 발생 시켜서 인지를 하게 되면 효율적으로 처리 할 수 있다는 장점이 있다.  

<br>

**Redis: LPOP(Polling 방식) vs BLPOP(Event-driven 방식)**  

<br>

| 방식    | LPOP (직접 구현 필요)                                                                                           | BLPOP (내장 대기 활용)                                                                                            |  
| ----- | --------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |  
| 코드 예시 | while(true) {  <br>data = LPOP(key);  <br>if(data) process(data);  <br>else sleep(1); // 직접 쉬어줘야 함  <br>} | while(true) {  <br>// 데이터 올 때까지 여기서 멈춤  <br>data = BLPOP(key, timeout);  <br>if(data) process(data);  <br>} |  
| 장점    | 여러 작업을 동시에 체크하기 용이                                                                                        | 자원 효율성 극대화, 실시간 처리                                                                                          |  
| 단점    | 불필요한 네트워크/CPU 트래픽 발생                                                                                      | 타임아웃 동안 커넥션 점유 (Connection Block)                                                                           |  
1. LPOP (Left Pop): 즉각적인 응답을 중시하는 명령어  
- 동작: 리스트의 왼쪽(첫 번째) 아이템을 제거하고 반환한다.  
- 비어있을 때: 리스트가 비어 있으면 즉시 nil(C 코드에서는 NULL 형태)을 반환하고 끝난다.  
- 비유: 우체통을 열어봤는데 편지가 없으면 바로 문 닫고 돌아오는 것과 같다.  

<br>

2. BLPOP (Blocking Left Pop): 데이터가 들어올 때까지 대기하는 명령어  
- 동작: 리스트에 데이터가 있으면 LPOP처럼 즉시 반환한다.  
- 비어있을 때: 데이터가 생길 때까지, 혹은 설정한 타임아웃(Timeout) 시간이 지날 때까지 연결을 유지하며 기다린다.  
- 비유: 우체국 창구에서 편지가 올 때까지 자리에 앉아 기다리는 것과 같다.  

<br>

Redis에서 BLPOP `timeout=0`으로 설정한다는 것은 무한 대기하라는 뜻이다.  
그러나 그렇게 되면 Redis에 일감이 들어오기 전까지 해당 스레드는 완전 Lock이 걸려버린다.  
- 이때 서버를 종료하라는 신호 `SIGTERM, SIGINT`를 보내도 프로그램이 Blocking 상태에 갇혀 있어서 신호를 감지하지 못하고 Kill 될 때까지 좀비처럼 남아있게 된다.  
- 따라서 잠깐 동안 기다렸다가 풀려나서 `_running` Flag 즉, 종료 신호를 받았는지 확인하고, 다시 대기하는 루프를 돌리는 구조이다.  

<br>

Redis에서 BLPOP으로 설계를 할 때는 운영 환경을 생각했을 때, 다음과 같은 주의 사항을 고려해야한다.  

<br>

① 멱등성(Idempotency)과 무조건 실패 처리 ★   

<br>

멱등성(Idempotency): 연산이나 작업을 여러 번 반복하더라도 결과가 한 번만 실행했을 때와 같은 성질  
- 동일한 함수나 API 요청을 서버에 여러 번 전송해도, 최초 한 번 수행한 것과 동일한 결과가 나오는 것을 의미한다.  
- 네트워크 오류 등으로 중복 요청이 발생하더라도 시스템의 데이터 일관성을 유지하는 핵심 원칙이다.  

<br>

현재 코드는 `process_job` 내의 `try-except` 블록에서 오류가 나면 바로 `JobStatus.FAILED`로 업데이트합니다.  
- **문제 상황:** 만약 AI 모델을 호출하는 외부 API나 네트워크가 **1초 동안 잠깐 끊겼다면?** 이 작업은 영원히 실패 처리됩니다.  
- **해외/국내 트렌드 대응:** 보통 메세지 큐 시스템에서는 실패 시 **재시도(Retry) 메커니즘**을 둡니다. "최대 3번 재시도하고, 그래도 안 되면 실패(FAILED)나 데드 레터 큐(DLQ)로 보낸다"는 로직을 `JobService`나 워커에 추가하는 것을 고려해보세요.  

<br>

② Graceful Shutdown (우아한 종료)의 디테일  

<br>

코드에서 `_running = False`를 통해 현재 처리 중인 Job까지는 안전하게 완료하고 종료되도록(`Graceful`) 잘 짜여 있습니다.  
- **주의할 점:** 만약 `process_job` 내부의 AI 추론(실제 모델 호출)이 2초가 아니라 **10분**이 걸리는 무거운 작업이라면 어떻게 될까요? AWS나 Docker 인프라는 `SIGTERM`을 보낸 후 보통 10초~30초 동안 워커가 안 죽으면 `SIGKILL`로 강제 강등해 버립니다.  
- **대응:** 무거운 작업을 할 때는 타임아웃 제한을 두거나, 인프라(Docker/K8s)의 종료 대기 시간(Termination Grace Period)을 Job의 최대 예상 수행 시간보다 길게 잡아주어야 데이터 유실을 막을 수 있습니다.  

<br>

③ Connection Pool 및 세션 관리  

<br>

`_make_service` 함수에서 Job이 올 때마다 `SessionLocal()`을 생성하고 `finally`에서 `db.close()`로 닫아주고 있습니다. **매우 훌륭한 구현입니다.** * 워커는 한 번 켜지면 몇 달 동안 켜져 있을 수 있는 프로세스이기 때문에, 하나의 DB 세션을 공유하면 커넥션이 끊어지거나 오염될 수 있습니다. 지금처럼 매 Job마다 세션을 열고 닫는 것이 정석입니다.  
- 단, Redis 클라이언트(`redis_client`)도 내부적으로 **Connection Pool** 기반으로 동작하고 있는지 확인해 두는 것이 좋습니다.  
    
<br>


<br>

④ 워커 동시성 (Concurrency) 확장성  

<br>

현재 구조는 `while _running:` 루프 하나가 돌면서 **한 번에 딱 하나의 Job만** 처리합니다.  
- 만약 대기열에 Job이 100개 쌓이면, 앞의 Job이 끝날 때까지 뒤의 Job들은 무조건 대기해야 합니다.  
- **향후 확장 방향:** 멀티프로세싱(Python의 `multiprocessing`)을 도입하거나, 워커 프로세스 자체를 여러 개 띄우거나(Scale-out), `Celery`나 `FastAPI-Users` 환경에서 많이 쓰는 `BullMQ`(Node계열) 같은 검증된 패키지로 마이그레이션하는 시점이 올 수 있습니다. 현재 코드는 결합도가 낮아 가볍게 여러 프로세스로 띄우기 좋습니다.  

<br>

**DB: 트랜잭션**  

<br>

[트랜잭션개념 및 COMMIT/ROLLBACK 예제](https://infjin.tistory.com/137)  

<br>

트랜잭션은 한 개 이상의 SQL문으로 이루어진 작업을 위한 논리적인 작업 단위로써 분할할 수 없는 최소 수행단위이다.  
- DML과 DDL 명령어를 포함할 수 있다.  
- 연산의 집합이지만 논리적으론 하나로 보아야 한다.  
- DB의 일관성과 무결성을 유지하기 위해 사용된다.  

<br>

Database Language(DDL)  
- Database의 Schema를 정의하는 언어 `CREATE, ALTER, DROP, TRUNCATE 등`  
- DDL의 결과는 `data dictionary`에 Meta data로써 저장된다. 다음과 같은 정보들을 포함한다.  
- Database Schema  
- Integrity Constraints `Primary Key, Foreign Key`  
- Authorization  

<br>

Database Manipulatoin Language (DML) `SELECT, INSERT, UPDATE, DELETE`  
- 사용자가 적절한 Data Model로 구성된 Data를 접근/조작할 수 있도록 하는 언어  
- Query Language라고도 한다.  
- DML의 종류는 기본적으로 두 가지 정도가 있다.  
- Procedural DML(절차식 DML): Data를 어떻게 구할지도 요구  
- Declarative DML(비절차식 DML) `이게 더 쉬움`  

<br>

그러니까 DML은 엑셀의 셀을 바꾸는 명령어고 DDL은 나머지라고 생각하면 쉬울거 같다.  

<br>

트랜잭션의 4가지 특성(ACID)  
1. 원자성(Atomicity) : 트랜잭션의 연산은 모두 반영되든지(ALL) 혹은 전혀 반영되지 않아야한다(NOTHING).  
2. 일관성(Consistency) : 트랜잭션 실행 후 데이터베이스의 무결성은 유지되어야한다.  
3. 격리성,독립성(Isolation) : 트랜잭션 실행 중에는 다른 트랜잭션이 접근할 수 없다.  
4. 영속성(Durability) : 트랜잭션의 결과는 계속 유지되어야한다.  

<br>

TCL(Transaction Control Language): 트랜잭션을 제어하는 명령어 `COMMIT, ROLLBACK`  
- `COMMIT`: 수행한 트랜잭션 명령어를 DB에 반영할 때 사용하는 명령어다. `COMMIT`된 트랜잭션의 반영은 되돌릴 수 없으므로 신중하게 사용해야한다.  
- `ROLLBACK`: 트랜잭션을 취소할 때 사용한다. SQL을 수행하는 도중 TCL, DDL, DCL에 해당하는 작업이 없다면 이들은 하나의 트랜잭션에 속해있다. 트랜잭션을 반영하기 전에 `ROLLBACK` 명령어를 사용하면 트랜잭션에 포함된 DML의 수행을 취소한다.  

<br>

Q | 그럼 이 문제는 SQLAlchemy를 사용함으로써 원천 차단되는 거 아닌가? 내가 Repository 코드를 어떻게 설계하느냐에 따라 Service에 따른 병목이 발생할 수 있는건가?  
```  
하나의 트랜잭션에서 A를 저장하고 ai api를 호출해서 평균 10초동안 대기 후 결과를 받아와. 그리고 그 결과를 db에 저장해. 이 때 이 요청들을 하나의 트랜잭션에 넣으면 오랫동안 db 커넥션 풀을 점유해서 사용자가 몰렸을 때 서비스가 느려질 수 있는 문제가 있어.
```  
A| 원천 차단하지 않는다. 이건 SQLAlchemy의 문제가 아니라 트랜잭션 범위(Transaction Scope)의 문제이기 때문이다.  
- AI 추론 동안에는 DB Commection이 없는 게 굉장히 중요하다. 데이터의 무결성을 위해서  

<br>

**폴링 (Polling) & 서버-센트 이벤트 (SSE)**  

<br>

폴링(Polling)과 서버-센트 이벤트(Server-Sent Events, SSE)는 클라이언트와 서버 간의 실시간 또는 거의 실시간 데이터 통신을 위한 기술이다.  

<br>

서로 각기 다른 방식으로 데이터를 주고받으며, 웹 애플래케시연에서 널리 사용된다.  

<br>

1. 폴링 (Polling)  

<br>

클라이언트가 일정한 시간 간격으로 서버에 요청을 보내 데이터를 업데이트 하는 방식이다.  
- 클라이언트가 서버에 주기적으로 HTTP 요청을 보낸다.  
- 서버는 요청에 대해 현재 데이터를 응답한다.  
- 이 과정이 반복되며 클라이언트는 서버로부터 새로운 데이터를 받아 실시간처럼 처리한다.  

<br>

2. 서버-센트 이벤트 (SSE)  

<br>

서버에서 클라이언트로 실시간 데이터를 푸시하는 방식이다.  
클라이언트는 한 번의 연결로 지속적으로 서버로부터 이벤트를 수신할 수 있다.  
HTTP 프로토콜을 통해 단방향으로 이루어지고 HTML5에서 처음 도입된 기능이다.  
- 클라이언트는 서버에 `EventSource` 객체를 사용해 SSE 연결을 연다.  
- 서버는 데이터가 변경될 때마다 클라이언트로 이벤트를 푸시한다.  
- 연결이 열려 있는 동안 서버는 지속적으로 데이터를 보낼 수 있다.  

<br>

**RabbitMQ**  

<br>

| 구분        | RabbitMQ                           | Redis                                      |  
| --------- | ---------------------------------- | ------------------------------------------ |  
| 기본 목적     | 메시지 중개 및 라우팅 (AMQP 등 표준 지원)        | 인메모리 데이터 저장소 (Key-Value 등)                 |  
| 메시지 전달 보장 | 매우 높음 (확실한 승인(ACK), 트랜잭션 지원)       | 비교적 낮음 (기본적으로 휘발성이며, 유실 가능성 있음)            |  
| 메시지 라우팅   | 복잡하고 유연한 라우팅(Routing, Exchange) 제공 | 단순한 채널 발행-구독(Pub/Sub) 구조                   |  
| 메시지 소멸 여부 | 소비자가 처리하면 큐에서 삭제 (데이터 보존 가능)       | 소비자가 없으면 메시지가 즉시 사라짐 (Redis 5+ Streams 제외) |  
RabbitMQ는 다양한 조건과 라우팅 규칙에 따라 메시지를 안전하게 처리하고 분산 시스템을 통합할 때 사용한다.  
실패 시 데이터 유실이 치명적인 결제 시스템 및 주문처리나, 여러 Consumer에게 조건별로 데이터를 배분해야 하는 마이크로서비스 아키텍처(MSA)에서 사용한다.  

<br>

반면 Redis는 설정이 매우 간편하며, 실시간 데이터 엑세스 및 고성능, 빠른 속도가 최우선 과제일 때 주로 활용한다.  
잃어버려도 시스템에 큰 영향이 없는 채팅 서비스 또는 실시간 알림 시스템이나, 웹 서비스의 세션 관리, 데이터 캐싱과 가벼운 메시지 큐를 하나의 시스템으로 통합하려는 경우 사용한다.  

<br>

**MSQ & Monolithic**  

<br>

MSA(Microservice Architecture, 마이크로서비스 아키텍처)는 거대한 하나의 애플리케이션을 여러 개의 작고 독립적인 서비스 단위로 쪼개어 개발하고 배포하는 소프트웨어 설계 기법이다.  

<br>

| 구분    | 모놀리식 (Monolithic)                | MSA                           |  
| ----- | -------------------------------- | ----------------------------- |  
| 구조    | 모든 기능이 하나의 거대한 코드베이스와 DB로 결합됨    | 기능 단위로 서비스를 쪼개고 개별 DB를 가짐     |  
| 배포/확장 | 전체 시스템을 다시 빌드하고 배포해야 함           | 변경된 특정 서비스만 독립적으로 배포 및 확장 가능  |  
| 장점    | 초기 개발과 테스트가 단순함                  | 장애 격리 용이, 유연한 기술 도입, 빠른 배포    |  
| 단점    | 한 곳의 오류가 전체 장애로 이어짐, 부분적 확장이 어려움 | 네트워크 통신 오버헤드, 데이터 일관성 유지의 복잡성 |  

<br>

**k6**  

<br>

HTTP 요청을 대량으로 자동 생성해서 서버의 성능을 측정하는 부하 테스트(Load Testing) 도구  
우리가 Swagger로 테스트할 때는 사실상 사용자 한 명이 요청을 보내는 것에 그치나,  
k6를 사용하면 사용자 여러명이 동시에 요청을 보내는 것을 시뮬레이션 할 수 있다.  
1. Load Test: 사용자 50명 30초 동안 `POST/jobs` 계속 보내기, 응답시간/처리량/에러율 측정  
2. Stress Test: 10명 -> 100명 -> 500명 -> 1000명.. 언제 서버가 무너지기 시작하는가?  
3. Spike Test: 갑자기 0명에서 1000명으로 늘리는 테스트  
4. Soak Test: 50명이 2시간 동안 계속 요청  

<br>

Virtual User(VU) `vus`: 가상의 사용자 한 명 `vus = 1`  
Iteration: VU가 한 번 실행하는 작업  
Duration: `duration = 30`이면 30s 동안 계속 반복한다.  
Scenario: 사용자의 행동, 예를 들어 로그인 -> 상품 조회 -> 결제가 하나의 Scenario다.  
Metrics: k6는 마지막에 결과를 보여준다.  
- `http_req_duration`: 평균 응답시간  
- `http_reqs`: 총 요청 수  
- `iterations`: Scenario가 몇 번 실행됐는가  
- `checks`: 성공률  
- `failure`: 에러율  
- `RPS(Request Per Second)`: 1초에 처리한 요청 수  
- `Latency`: 사용사 -> 서버 -> 응답 까지 걸리는 시간  
- `Throughput`: 처리량  
- `P95`: 95%의 요청이 이 시간 안에 끝났다는 뜻이다.  

<br>

**AWS (Amazome Web Service)**  

<br>


<br>


<br>

# Phase  

<br>

## Phase 0  

<br>

Mini Milestone 마다 다음과 같은 작업을 반복한다.  

<br>

```  
1. 왜 필요한가?
        ↓
2. 아키텍처 설계
        ↓
3. AI에게 구현 Prompt 작성
        ↓
4. 코드 생성
        ↓
5. 코드 리뷰
        ↓
6. 동작 테스트
        ↓
7. 리팩토링
        ↓
8. 회고 + 문서화
```  

<br>

이를 위해 Milestone의 항목은 다음과 같이 구성한다.  

<br>

```  
# Phase 0.0

목표
현재 문제
설계
아키텍처
AI Prompt
구현 결과
테스트 방법 및 결과
```  

<br>

ChatGPT Milestone Prompt는 다음과 같이 구성한다.  
```  
Phase 2.4 Executor 및 Retry 기본 구조

목표
현재 문제
설계
아키텍처
AI Prompt

위 Milestone의 항목을 작성해주세요.

생성 조건:
- 현재 프로젝트 GPT Context 반영
- 딱 필요한 만큼만 적고 분량은 최소화
- AI Promt는 토큰 최적화 및 일관성 있게 작성, 현재 GPT 프로젝트 Context와 동일한 Agent Context 파일이 프로젝트 폴더 내에 이미 있으니 작성하지 않아도 됨
- 아래 템플릿을 사용하여 작성하되 다음을 지킬것:
    - 각 항목의 분량을 맞춘답시고(동일하게 세 줄, 네 줄 등) 내용을 일부러 늘리거나 줄이지 않기
    - 분량은 Github에 기록할 수 있을 정도로 최소화해서 작성 (어차피 너무 길면 나중에 읽기 힘듦)
    - 아키텍처 작성 시 화살표는 '↓' 사용
    - 항목 앞에 '#' 붙이기 금지지
    - AI Prompt는 Agent가 이해를 잘 할 수 있도록 영어로 구성하되 결과물에 대한 설명은 한국어로 출력하도록 구성

템플릿:
Phase 0.0 {Mimi Milestone name}

**목표**
- ..
- ..

**현재 문제**
- ..
- ..

**설계**
- ..
- ..

**아키텍처**
```plaintext  
'``  

<br>

**AI Prompt**  
```plaintext  
'``

```  

<br>

## Phase 1. CRUD  

<br>

후술  

<br>

## Phase 2. 비동기 작업 처리  

<br>

계획은 다음과 같다.  
- Phase 2.1 JobStatus Enum + State Machine  
- Phase 2.2 Redis Queue 연동  
- Phase 2.3 Worker 프로세스 구현  
- Phase 2.4 Executor 및 Retry 기본 구조  
- Phase 2.5 Logging과 안정성 검증  

<br>

### Phase 2.1 JobStatus Enum + State Machine  

<br>

**목표**  
- Job 상태를 Enum으로 통일하여 타입 안정성을 확보한다.  
- 상태 전이 규칙(State Machine)을 중앙에서 관리할 기반을 만든다.  

<br>

**현재 문제**  
- Job 상태가 문자열 기반이라 오타와 잘못된 상태 저장을 방지할 수 없다.  
- 상태 전이 규칙이 없어 계층마다 서로 다른 로직이 생길 수 있다.  

<br>

**설계**  
- JobStatus Enum을 정의하고 모든 계층에서 공통으로 사용한다.  
- 상태 변경은 State Machine을 통해서만 수행하도록 구조를 변경한다.  

<br>

**아키텍처**  
```plaintext  
Router
    ↓
Service
    ↓
State Machine
    ↓
Repository
    ↓
Database
```  

<br>

**AI Prompt**  
```plaintext  
Implement Phase 2.1 for the existing FastAPI backend project.

Requirements:
- Add a JobStatus Enum for all job states.
- Implement a State Machine that validates allowed state transitions.
- Keep the existing Layered Architecture.
- Router must not contain business logic.
- Repository must only access the database.
- Service must use the State Machine before changing job status.
- Design for future Worker integration without implementing Worker yet.
- Write readable and explicit code with type hints.
- Minimize unnecessary abstraction.
- Preserve the current project structure unless a small change is clearly justified.

Output requirements:
- 모든 설명은 한국어로 작성.
- 변경 이유와 Trade-off를 함께 설명.
- 수정한 파일 목록을 먼저 출력.
- 각 파일의 역할을 간단히 설명.
- 테스트 방법을 마지막에 제시.
```  

<br>

**구현 결과**  

<br>

| 파일                                  | 변경 내용                                                     |  
| ----------------------------------- | --------------------------------------------------------- |  
| `src/models/job.py`                 | `CANCELLED` 상태 추가                                         |  
| `src/exceptions.py`                 | `InvalidStateTransitionError` (HTTP 409) 추가               |  
| `src/services/job_state_machine.py` | _(신규)_ 허용 전이 테이블 + `validate_transition()`                |  
| `src/services/job.py`               | `create_job`은 PENDING 고정, `update_job`에서 State Machine 호출 |  
| `src/schemas/job.py`                | `JobCreate`에서 `status` 필드 제거                              |  

<br>

1. 상태 전이 규칙  
```  
PENDING ──→ RUNNING     (Worker가 Job 처리 시작)
PENDING ──→ CANCELLED   (사용자가 대기 중 취소)
RUNNING ──→ COMPLETED   (Worker 성공)
RUNNING ──→ FAILED      (Worker 실패)
RUNNING ──→ CANCELLED   (사용자가 실행 중 취소)

COMPLETED, FAILED, CANCELLED → 전환 없음 (종료 상태)
```  

<br>

2. 설계 결정 Trade-off  

<br>

`JobCreate`에서 `status` 제거 (Breaking Change)  
- 이유: 생성 시 PENDING이 아닌 상태를 설정하면 State Machine을 우회할 수 있음  
- Trade-off: 기존 API 클라이언트에서 `status` 필드를 보내면 Pydantic이 무시하므로 실제로는 호환됨 (추가 필드는 무시됨)  

<br>

State Machine을 별도 파일로 분리  
- 이유: 전이 규칙이 Service 로직과 분리되어 독립적으로 읽히고, Worker 추가 시 재사용 가능  
- Trade-off: 파일이 하나 늘어남. 하지만 규칙이 5개 상태 × N 전이로 늘어날 수 있어 분리 유지 비용 < 혼재 비용  

<br>

**테스트 방법 및 결과**  
```  
# 1. Job 생성 (항상 PENDING으로 시작)
curl -X POST http://localhost:8000/jobs \
  -H "Content-Type: application/json" \
  -d '{"user_id": 1}'
# → {"id":1, "user_id":1, "status":"PENDING"}

# 2. 유효한 전환: PENDING → RUNNING
curl -X PUT http://localhost:8000/jobs/1 \
  -H "Content-Type: application/json" \
  -d '{"status": "RUNNING"}'
# → {"id":1, "user_id":1, "status":"RUNNING"}

# 3. 유효한 전환: RUNNING → COMPLETED
curl -X PUT http://localhost:8000/jobs/1 \
  -H "Content-Type: application/json" \
  -d '{"status": "COMPLETED"}'
# → {"id":1, "user_id":1, "status":"COMPLETED"}

# 4. 잘못된 전환: COMPLETED → RUNNING (409 예상)
curl -X PUT http://localhost:8000/jobs/1 \
  -H "Content-Type: application/json" \
  -d '{"status": "RUNNING"}'
# → 409 {"detail": "'COMPLETED' 상태에서 'RUNNING'으로 전환할 수 없습니다. 허용된 전환: []"}

# 5. 잘못된 전환: PENDING → COMPLETED (409 예상)
curl -X PUT http://localhost:8000/jobs/2 \
  -H "Content-Type: application/json" \
  -d '{"status": "COMPLETED"}'
# → 409 {"detail": "'PENDING' 상태에서 'COMPLETED'로 전환할 수 없습니다. 허용된 전환: ['RUNNING', 'CANCELLED']"}
```  
- State Machine 정상 작동 확인 (상태 전이 가능 여부, 에러 상세 메세지 출력, DB 저장 등)  
- Enum 안정성 확인 (예: "ABC"와 같은 상태 입력 금지)  

<br>

### Phase 2.2 Redis Queue 연동  

<br>

**목표**  
* Job 생성 시 Redis Queue에 작업을 등록한다.  
* Queue를 Service 계층에서 관리하여 Worker와 API를 분리할 기반을 만든다.  

<br>

**현재 문제**  
* Job 생성 후 비동기 작업을 전달할 경로가 없다.  
* 향후 Worker와 API가 직접 결합될 가능성이 있다.  

<br>

**설계**  
* Queue 인터페이스를 추가하고 Redis 구현체를 분리한다.  
* Job 생성 후 상태를 QUEUED로 변경하고 Queue에 Job ID를 등록한다.  

<br>

**아키텍처**  
```plaintext  
Router
    ↓
Service
    ↓
Repository
    ↓
Database

Service
    ↓
Redis Queue
```  

<br>

**AI Prompt**  
```plaintext  
Implement Phase 2.2 for the existing FastAPI backend project.

Requirements:
- Integrate Redis as the job queue.
- Create a queue abstraction and a Redis implementation.
- Enqueue the Job ID after a job is successfully created.
- Keep Router free of queue logic.
- Keep Repository responsible only for database access.
- Queue operations must be handled by the Service layer.
- Do not implement Worker, dequeue logic, retry, or Pub/Sub.
- Design the queue interface for future Worker reuse.
- Write explicit code with type hints and minimal abstraction.

Output requirements:
- 모든 설명은 한국어로 작성.
- 변경 이유와 Trade-off를 함께 설명.
- 수정한 파일 목록을 먼저 출력.
- 각 파일의 역할을 간단히 설명.
- 테스트 방법을 마지막에 제시.
```  

<br>

**구현 결과**  

<br>

1. 새로운 파일 구조  
```  
src/
  queue/
    __init__.py         (빈 파일)
    base.py             (JobQueue Protocol)
    redis_queue.py      (RedisJobQueue + redis_client)
  services/
    job.py              (JobQueue 주입받아 create 후 enqueue)
  routers/
    job.py              (get_job_queue 의존성 추가)
```  

<br>

2. Job 생성 흐름 (Phase 2.2 이후)  
```  
POST /jobs
  → Router (HTTP 요청 수신, 의존성 주입)
  → JobService.create_job()
      → UserRepository.get_by_id()  (사용자 존재 확인)
      → JobRepository.create()      (DB에 PENDING 상태로 저장)
      → RedisJobQueue.enqueue()     (Redis List에 job_id RPUSH)
  → JobResponse 반환
```  

<br>

3. Worker 통합 시 예상 흐름 (Phase 2.3 이후)  
```  
Worker:
  redis.blpop("job_queue") → job_id 획득
  → JobService.update_job(job_id, RUNNING)
  → (AI 추론 실행)
  → JobService.update_job(job_id, COMPLETED or FAILED)
```  

<br>

`JobQueue` Protocol에 `dequeue()`를 추가하고 `RedisJobQueue`에 `blpop` 구현을 추가하면 Worker가 같은 인터페이스를 사용 가능.  

<br>

**테스트 방법 및 결과**  
```  
# 패키지 설치
pip install -e .

# Docker Compose로 전체 기동 (PostgreSQL + Redis + API)
docker-compose up -d

# 또는 로컬 개발 시 Redis만 Docker로 실행
docker run -d -p 6379:6379 redis:7
uvicorn main:app --reload

# Job 생성
curl -X POST http://localhost:8000/jobs \
  -H "Content-Type: application/json" \
  -d '{"user_id": 1}'
# → {"id": 1, "user_id": 1, "status": "PENDING"}

# Redis CLI로 큐 상태 확인
redis-cli LRANGE job_queue 0 -1
# → 1) "1"   (job_id가 큐에 있음)

# Job을 두 개 더 생성
curl -X POST http://localhost:8000/jobs -H "Content-Type: application/json" -d '{"user_id": 1}'
curl -X POST http://localhost:8000/jobs -H "Content-Type: application/json" -d '{"user_id": 1}'

redis-cli LRANGE job_queue 0 -1
# → 1) "1"
# → 2) "2"
# → 3) "3"   (FIFO 순서 확인)

redis-cli LLEN job_queue
# → (integer) 3   (큐 길이 확인)

# Redis를 중단한 상태에서 Job 생성 시도
docker stop ai-job-redis

curl -X POST http://localhost:8000/jobs \
  -H "Content-Type: application/json" \
  -d '{"user_id": 1}'
# → 500 (ConnectionError: Redis에 연결할 수 없음)
# 이는 의도된 동작: enqueue 실패 시 caller에게 오류를 전파
```  
- Job Create 시 Redis FIFO Queue에 작업이 들어가 있는 것을 확인할 수 있었다.  
- Redis 서버를 멈추면 Internal Server Error 코드 500이 뜨는 것도 확인하였다.  

<br>

### Phase 2.3 Worker 프로세스 구현  

<br>

Phase 2.3 Worker 프로세스 구현  

<br>

**목표**  
* Redis Queue를 소비하는 Worker 프로세스를 구현한다.  
* Worker가 Job 상태를 RUNNING → COMPLETED/FAILED로 변경하도록 한다.  

<br>

**현재 문제**  
* Queue에 등록된 Job을 처리하는 소비자가 없다.  
* API와 비동기 작업 실행이 아직 연결되지 않았다.  

<br>

**설계**  
* Worker를 FastAPI와 독립된 프로세스로 구현한다.  
* Worker는 Queue와 JobService만 사용하며 Repository를 직접 호출하지 않는다.  

<br>

**아키텍처**  
```plaintext  
Client
    ↓
Router
    ↓
Service
    ↓
Repository
    ↓
Database

Service
    ↓
Redis Queue
    ↓
Worker
    ↓
JobService
    ↓
Repository
    ↓
Database
```  

<br>

**AI Prompt**  
```plaintext  
Implement Phase 2.3 for the existing FastAPI backend project.

Requirements:
- Implement a standalone Worker process.
- Consume jobs from Redis Queue using the existing queue abstraction.
- Reuse JobService instead of accessing Repository directly.
- Change job status to RUNNING before processing.
- Simulate the job with a short delay instead of AI inference.
- Change job status to COMPLETED on success and FAILED on exception.
- Keep the Worker independent from FastAPI.
- Do not implement retry, executor abstraction, scheduling, or Pub/Sub.
- Write explicit code with type hints and minimal abstraction.

Output requirements:
- 모든 설명은 한국어로 작성.
- 변경 이유와 Trade-off를 함께 설명.
- 수정한 파일 목록을 먼저 출력.
- 각 파일의 역할을 간단히 설명.
- 테스트 방법을 마지막에 제시.
```  

<br>

**구현 결과**  

<br>

전체 Job 처리 흐름  
```  
[클라이언트]
  POST /jobs
    → API: DB에 Job 생성 (PENDING) + Redis에 job_id RPUSH

[Worker 프로세스 - 독립 실행]
  BLPOP job_queue (최대 5초 대기)
    ↓ job_id 수신
  새 DB Session 생성
  JobService.update_job(RUNNING)   ← State Machine 검증 통과
  time.sleep(2)                    ← AI 추론 시뮬레이션
  JobService.update_job(COMPLETED) ← 성공 시
  또는
  JobService.update_job(FAILED)    ← 예외 발생 시
  DB Session 종료
```  

<br>

**테스트 방법 및 결과**  
```  
docker-compose up -d

# 서비스 상태 확인
docker-compose ps
# api, postgres, redis, worker 모두 running이어야 함

docker-compose logs -f worker
# → Worker 시작. Redis 큐 대기 중...

# Job 생성
curl -X POST http://localhost:8000/jobs \
  -H "Content-Type: application/json" \
  -d '{"user_id": 1}'
# → {"id": 1, "user_id": 1, "status": "PENDING"}

# Worker 로그 확인
# → Job 1 처리 시작
# → Job 1 완료

# 2초 후 상태 확인
curl http://localhost:8000/jobs/1
# → {"id": 1, "user_id": 1, "status": "COMPLETED"}

# 터미널 1: API 서버
uvicorn main:app --reload

# 터미널 2: Worker (별도 프로세스)
python worker.py
# → Worker 시작. Redis 큐 대기 중...

# Worker 종료 시
Ctrl+C
# → 종료 신호 수신. 현재 Job 처리 완료 후 종료합니다.
# → Worker 종료

# 여러 Job 연속 생성
for i in {1..5}; do
  curl -s -X POST http://localhost:8000/jobs \
    -H "Content-Type: application/json" \
    -d '{"user_id": 1}' | python -m json.tool
done

# Worker 로그에서 순차 처리 확인 (FIFO 보장)
# Job 1 처리 시작 → 완료
# Job 2 처리 시작 → 완료
# ...
```  
- 모두 잘 돌아가는 것을 확인하였다.  
- 다만 worker.py가 자꾸 죽어 docker가 다시 살리는 에러가 반복되었다.  

<br>

**트러블 슈팅: Redis BLPOP 적용 후 Worker 프로세스 크래시 현상**  

<br>

문제 상황:  
- `BLPOP` 도입 후, 일감이 없는 유휴(Idle) 상태가 지속되면 Worker 프로세스가 주기적으로 `TimeoutError`를 뱉으며 크래시 발생.  
- Docker가 프로세스를 강제로 재시작하는 현상이 반복됨.  

<br>

원인 분석  
- 타임아웃 설정의 충돌:  
    - `BLPOP` 타임아웃은 5초로 설정되었으나, `redis-py` 클라이언트의 소켓 타임아웃(`socket_timeout`)은 기본값(`None`, 무한 대기) 또는 5초 이하로 설정됨.  
- TCP 연결 단절:  
    - 일감이 없는 5초 동안 Docker 네트워크 및 OS 레이어에서 이 연결을 '유휴 상태'로 판단해 TCP 소켓을 조용히 끊어버림.  
    - Redis 서버가 5초 후 정상적으로 `nil`(None) 응답을 보내도 이미 끊어진 소켓이라 수신하지 못함.  
- 지수 백오프(Exponential Backoff)로 인한 지연:  
    - 소켓 단절을 감지한 `redis-py` 내부 라이브러리가 자체적으로 여러 번 재시도(Retry)를 수행함.  
    - 이로 인해 실제 애플리케이션(`worker.py`)까지 에러가 전달되는 데 약 10~15초의 지연이 발생하며 최종 크래시로 이어짐.  

<br>

해결 방안 (아키텍처 및 코드 개선)  
- 공식 수식 적용: `socket_timeout`은 항상 Redis의 `BLPOP` 대기 시간보다 커야 정상 응답을 기다릴 수 있고, 실제 장애 상황에서만 발동함.
```math
\text{socket\_timeout (7초)} > \text{BLPOP timeout (5초)}
```
  
- 관심사 분리 (Separation of Concerns):  
    - 인프라 레이어(`redis_queue.py`)에서 예외를 삼키고 `None`을 반환하던 구조를 탈피, `TimeoutError`를 상위 애플리케이션 레이어로 그대로 전파(Propagation).  
    - `worker.py` 메인 루프에서 예외를 캐치하여 안전하게 `continue` 처리하도록 구조 개선.  

<br>

최종 결과  
- Graceful Recovery 구현: `TimeoutError`가 정상적으로 위로 터져 나오면서 `redis-py`의 ConnectionPool이 망가진 소켓을 자동으로 폐기함.  
- 다음 루프 시도 시 완전히 새로운 깨끗한 소켓으로 자동 재연결되어, 일감이 없거나 네트워크가 일시 단절되어도 Worker가 죽지 않고 영구적으로 대기·생존함.  

<br>

### Phase 2.4 Executor 및 Retry 기본 구조  

<br>

**목표**  
* Worker의 작업 실행 로직을 Executor로 분리한다.  
* 작업 실패 시 제한된 횟수만 재시도하는 구조를 추가한다.  

<br>

**현재 문제**  
* Worker가 Queue 소비와 작업 실행을 모두 담당하여 역할이 혼재되어 있다.  
* 작업 실패 시 즉시 FAILED 처리되어 일시적인 오류를 복구할 수 없다.  

<br>

**설계**  
* Worker는 Queue 소비와 상태 관리만 담당하고 실제 작업은 Executor에 위임한다.  
* Executor 실패 시 재시도 횟수를 확인한 후 Queue에 다시 등록하거나 FAILED로 종료한다.  

<br>

**아키텍처**  
```plaintext  
Worker
    ↓
Executor
    ↓
JobService
    ↓
Repository
    ↓
Database

Executor
    ↓
Retry Check
    ↓
Redis Queue
```  

<br>

**AI Prompt**  
```plaintext  
Implement Phase 2.4 for the existing FastAPI backend project.

Requirements:
- Extract job execution logic into a dedicated Executor.
- Keep Worker responsible only for queue consumption and job lifecycle.
- Add a retry counter with a configurable maximum retry limit.
- Re-enqueue failed jobs while the retry limit is not exceeded.
- Mark the job as FAILED after the retry limit is reached.
- Reuse the existing JobService and queue abstraction.
- Do not implement retry delay, backoff, retry queue, dead-letter queue, or scheduling.
- Keep the Layered Architecture and write explicit code with type hints.

Output requirements:
- 모든 설명은 한국어로 작성.
- 변경 이유와 Trade-off를 함께 설명.
- 수정한 파일 목록을 먼저 출력.
- 각 파일의 역할을 간단히 설명.
- 테스트 방법을 마지막에 제시.
```  

<br>

**구현 결과**  

<br>

| Phase 2.3 (이전)                                                                             | Phase 2.4 (이후)                                                                                                        |  
| ------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------- |  
| `worker.py`<br>- 큐 소비<br>- RUNNING 전환<br>- 추론 시뮬레이션<br>- COMPLETED/FAILED 전환<br>- (재시도 없음) | `src/executor.py (신규)`<br>- RUNNING 전환<br>- 추론 시뮬레이션<br>- COMPLETED 전환<br>- 실패 시 retry_count 증가<br>- 재큐잉 or FAILED 결정 |  

<br>

재시도 상태 전이  
```  
[최초 시도]
PENDING → RUNNING → (실패) → PENDING → 재큐잉 (retry_count=1)

[재시도 1~3회]
PENDING → RUNNING → (실패) → PENDING → 재큐잉 (retry_count=2, 3)

[한도 초과]
PENDING → RUNNING → (실패) → FAILED (retry_count=4, max=3 초과)
```  

<br>


<br>

**테스트 방법 및 결과**  
```  
-- PostgreSQL에서 실행
DROP TABLE IF EXISTS jobs CASCADE;
-- 앱 재시작 시 Base.metadata.create_all()이 새 컬럼 포함하여 테이블 재생성

curl -X POST http://localhost:8000/jobs \
  -H "Content-Type: application/json" \
  -d '{"user_id": 1}'
# → {"id":1, "user_id":1, "status":"PENDING", "retry_count":0}

# 2초 후
curl http://localhost:8000/jobs/1
# → {"id":1, "user_id":1, "status":"COMPLETED", "retry_count":0}

# time.sleep(INFERENCE_SIMULATION_SECONDS) 를 아래로 교체
raise RuntimeError("AI 모델 오류 시뮬레이션")

curl -X POST http://localhost:8000/jobs -H "Content-Type: application/json" -d '{"user_id": 1}'

# Worker 로그 확인
# → Job 1 처리 시작
# → Job 1 실행 실패: AI 모델 오류 시뮬레이션
# → Job 1 재시도 예정 (1/3). 큐에 다시 등록합니다.
# → Job 1 처리 시작  ← 재시도 1
# → ...
# → Job 1 최대 재시도 횟수 초과 (3회). FAILED 처리합니다.

curl http://localhost:8000/jobs/1
# → {"id":1, "user_id":1, "status":"FAILED", "retry_count":4}
#                                                         ^^^^
#                                       4 = 최초 실패(1) + 재시도 3회
```  
- 모두 정상 작동함을 확인했다.  

<br>

### Phase 2.5 Logging과 안정성 검증  

<br>

**목표**  
- API와 Worker의 로그를 일관된 형식으로 관리한다.  
- 예외와 종료 상황에서도 안정적으로 동작하도록 개선한다.  

<br>

**현재 문제**  
- SQL 로그와 애플리케이션 로그가 혼재되어 흐름을 파악하기 어렵다.  
- 종료 및 예외 처리 방식이 컴포넌트마다 일관되지 않다.  

<br>

**설계**  
- 공통 Logging 설정을 적용하고 필요한 로그만 출력한다.  
- Worker의 예외와 종료를 Graceful하게 처리하고 정상 동작을 검증한다.  

<br>

**아키텍처**  
```plaintext  
Client
    ↓
API
    ↓
Application Logger

Worker
    ↓
Application Logger

Application Logger
    ↓
Console
```  

<br>

**AI Prompt**  
```plaintext  
Implement Phase 2.5 for the existing FastAPI backend project.

Requirements:
- Create a shared logging configuration for API and Worker.
- Reduce unnecessary SQLAlchemy logs while keeping application logs readable.
- Use consistent log format and log levels.
- Improve exception handling and graceful shutdown for the Worker.
- Ensure database sessions and Redis connections are properly released.
- Do not add external logging systems or monitoring tools.
- Keep the existing Layered Architecture and write explicit code with type hints.

Output requirements:
- 모든 설명은 한국어로 작성.
- 변경 이유와 Trade-off를 함께 설명.
- 수정한 파일 목록을 먼저 출력.
- 각 파일의 역할을 간단히 설명.
- 테스트 방법을 마지막에 제시.
```  

<br>

## Phase 3. Redis Pub/Sub + SSE + 연결 관리  

<br>

Phase 3의 계획은 다음과 같다.  
- Phase 3.1 Redis Pub/Sub 연동  
- Phase 3.2 SSE(Server-Sent Events) 기반 실시간 상태 조회  
- Phase 3.3 SSE 연결 관리 및 안정성  

<br>

### Phase 3.1 Redis Pub/Sub 연동  

<br>

**목표**  
- Worker의 Job 상태 변경을 Redis Pub/Sub으로 실시간 전파한다.  
- API가 상태 변경 이벤트를 구독할 수 있는 기반을 구축한다.  

<br>

**현재 문제**  
- Job 상태는 DB에만 저장되어 API가 변경 시점을 즉시 알 수 없다.  
- 클라이언트가 실시간 상태 조회를 위해 Polling에 의존해야 한다.  

<br>

**설계**  
- Worker는 Job 상태 변경 후 Redis Pub/Sub에 이벤트를 Publish한다.  
- Queue(RPUSH/BLPOP)는 작업 전달, Pub/Sub은 상태 이벤트 전달만 담당한다.  

<br>

**아키텍처**  
```plaintext  
Worker
    ↓
JobService.update_job()
    ↓
PostgreSQL
    ↓
Redis Pub/Sub (Publish)

API
    ↓
Redis Pub/Sub (Subscribe)
```  

<br>

**AI Prompt**  
```plaintext  
Implement Phase 3.1: Redis Pub/Sub integration.

Requirements:
- Preserve the existing layered architecture.
- Do not change the Job Queue implementation.
- Publish a job status event whenever JobService successfully updates a job status.
- Keep Queue (Redis List) and Pub/Sub responsibilities separate.
- The API must be able to subscribe to job events in the next milestone.
- Minimize structural changes and unnecessary abstractions.
- Follow the existing project architecture and coding conventions defined in the project context.
- Explain design decisions in Korean.
```  

<br>

### Phase 3.2 SSE(Server-Sent Events) 기반 실시간 상태 조회  

<br>

Phase 3.2 SSE(Server-Sent Events) 기반 실시간 상태 조회  

<br>

**목표**  
- SSE를 통해 Job 상태를 클라이언트에 실시간으로 전달한다.  
- Polling 없이 하나의 연결로 상태 변경 이벤트를 수신한다.  

<br>

**현재 문제**  
- Worker가 Publish한 이벤트를 클라이언트에 전달하는 경로가 없다.  
- 클라이언트는 상태 확인을 위해 반복적으로 GET 요청을 보내야 한다.  

<br>

**설계**  
- `GET /jobs/{job_id}/events` SSE 엔드포인트를 추가한다.  
- API는 Redis Pub/Sub을 Subscribe하여 수신한 이벤트를 SSE로 그대로 전달한다.  

<br>

**아키텍처**  
```plaintext  
Client
    ↓
GET /jobs/{job_id}/events
    ↓
FastAPI SSE Endpoint
    ↓
Redis Pub/Sub (Subscribe)
    ↓
Worker Publish Event
```  

<br>

**AI Prompt**  
```plaintext  
Implement Phase 3.2: Server-Sent Events (SSE) for real-time job status updates.

Requirements:
- Preserve the existing layered architecture.
- Add an SSE endpoint: GET /jobs/{job_id}/events.
- Subscribe to Redis Pub/Sub and stream received job events to the connected client.
- Keep the HTTP connection open until the client disconnects.
- Do not implement heartbeat or connection management yet.
- Minimize structural changes and unnecessary abstractions.
- Follow the existing project architecture and coding conventions.

Output requirements:
- 모든 설명은 한국어로 작성.
- 변경 이유와 Trade-off를 함께 설명.
- 수정한 파일 목록을 먼저 출력.
- 각 파일의 역할을 간단히 설명.
- 테스트 방법을 마지막에 제시.
```  

<br>

### Phase 3.3 SSE 연결 관리 및 안정성  

<br>

Phase 3.3 SSE 연결 관리 및 안정성  

<br>

**목표**  
- 장시간 유지되는 SSE 연결을 안정적으로 관리한다.  
- 연결 종료 시 리소스를 정상적으로 해제한다.  

<br>

**현재 문제**  
- 클라이언트 연결 종료를 감지하고 정리하는 로직이 없다.  
- 장시간 유휴 연결은 끊길 수 있으며 Redis Subscription이 남을 수 있다.  

<br>

**설계**  
- Client Disconnect 시 SSE 종료 및 Redis Unsubscribe를 수행한다.  
- Heartbeat를 추가하여 연결을 유지하고 불필요한 리소스를 정리한다.  

<br>

**아키텍처**  
```plaintext  
Client
    ↓
SSE Connection
    ↓
FastAPI
    ↓
Disconnect Detection
    ↓
Redis Unsubscribe
    ↓
Connection Close
```  

<br>

**AI Prompt**  
```plaintext  
Implement Phase 3.3: SSE connection management and stability.

Requirements:
- Preserve the existing layered architecture.
- Detect client disconnection and terminate the SSE stream gracefully.
- Unsubscribe from Redis Pub/Sub when the connection closes.
- Add heartbeat messages to keep long-lived SSE connections alive.
- Ensure resources are cleaned up to prevent connection or subscription leaks.
- Minimize structural changes and unnecessary abstractions.
- Follow the existing project architecture and coding conventions.

Output requirements:
- 모든 설명은 한국어로 작성.
- 변경 이유와 Trade-off를 함께 설명.
- 수정한 파일 목록을 먼저 출력.
- 각 파일의 역할을 간단히 설명.
- 테스트 방법을 마지막에 제시.
```  

<br>

**Trade-off:** Redis Pub/Sub은 이벤트를 저장하지 않으므로 Subscriber가 늦게 연결되면 일부 상태 변경 이벤트를 놓칠 수 있다. 현재 프로젝트에서는 DB를 현재 상태의 기준(Source of Truth)으로 사용하고, SSE는 상태 변경 알림 역할만 담당한다.  

<br>

## Phase 4. 신뢰성 향상(RabbitMQ, Retry Queue, Dead Letter Queue)  

<br>

Worker가 죽어도 작업이 유실되지 않는가?  
계획은 다음과 같다.  
- Phase 4.1 RabbitMQ Queue 전환  
- Phase 4.2 ACK/NACK 기반 작업 보장  
- Phase 4.3 Dead Letter Queue(DLQ)  

<br>

### Phase 4.1 RabbitMQ Queue 전환  

<br>


<br>

## Phase 5. AWS EC2 배포  

<br>

Phase 3까지 진행된 것을 EC2에 간단히 올려보자.  
Docker Compose가 되어있으니 내가 해야하는 건 그냥  
1. 인스턴스를 올리고  
2. ssh로 접속한 뒤  
3. docker를 깔고  
4. docker compose를 하고  
5. 도메인으로 접속해보면 된다.  

<br>

처음 AWS를 만들었으니 먼저 `Root User`로 로그인을 하고 `IAM User`를 생성한 뒤 `AdministratorAccess`를 부여하여 앞으로는 IAM User로 계속 로그인을 하는게 안전상 옳다.  

<br>

Phase는 다음과 같다.  
- Phase 4.1 Create Accound  
- Phase 4.2 EC2 Git Clone & Docker Compose Up  
- Phase 4.3 CI/CD  

<br>

### Phase 4.1 Create Account  

<br>

1. Root 계정 생성 후 MFA 설정  

<br>

MFA란 2단계 인증이라고 생각하면 된다.  
나는 모바일 디바이스에 깔아놓은 Authenticator로 등록했다.  

<br>

2. IAM 계정 생성 후 AdmistratorAccess 권한 부여  

<br>

Root에서 무식하게 모든 작업을 하면 계정 털렸을 때 끝장난다.  
그러기 위해 IAM 계정을 생성한다.  
무늬만 부계고 정작 루트 권한은 거의 모두 부여한 계정이라고 생각하면된다.  

<br>

[How to Create an IAM User in AWS – Step-by-Step Guide | LinkedIn](https://www.linkedin.com/pulse/how-create-iam-user-aws-step-by-step-guide-sridivya-pulimi-ep6yc/)  

<br>

3. IAM에서 EC2 인스턴스 생성  

<br>

```text  
이름: ai-inference-server
AMI: Ubuntu Server 24.04 LTS (HVM), SSD Volume 2 Type
인스턴스 유형: t3.small
키 페어:
	- 이름: asw-key.pem
	- 유형: RSA
네트워크:
	- 방화벽 설정: SSH ✓ HTTPS ✓ HTTP ✓
스토리지 구성: 8GB, gp3
```  

<br>

4. ssh 접속 후 환경설정  

<br>

```Bash  
# 0. SSH 접속 (Windows PowerShell)
# EC2 서버에 SSH로 접속
# -i : 사용할 개인키(.pem)
# ubuntu : Ubuntu EC2의 기본 사용자
ssh -i .\your-key.pem ubuntu@<EC2_PUBLIC_IP_OR_DNS>

# 1. Ubuntu 패키지 목록 최신화
# 저장소(Repository)의 최신 패키지 목록을 받아온다.
# 실제 설치는 하지 않는다.
sudo apt update
# 설치되어 있는 패키지를 최신 버전으로 업그레이드한다.
# -y : 모든 질문에 자동으로 Yes
sudo apt upgrade -y

# 2. Docker 설치에 필요한 기본 패키지 확인
# HTTPS 통신, 다운로드(curl), GPG 검증을 위한 패키지
# 이미 설치되어 있다면 "0 newly installed"가 출력된다.
sudo apt install -y ca-certificates curl gnupg

# 3. Docker 공식 GPG Key 등록
# GPG Key를 저장할 디렉터리 생성
# 0755 = rwxr-xr-x
sudo install -m 0755 -d /etc/apt/keyrings
# Docker 공식 공개키 다운로드
# Ubuntu가 Docker 패키지가 진짜 공식 패키지인지 검증하기 위해 사용
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 4. Docker 공식 Repository 등록
# Docker 공식 저장소 등록
#
# dpkg --print-architecture
#   -> 현재 CPU 아키텍처 (amd64)
#
# VERSION_CODENAME
#   -> Ubuntu 코드명 (24.04 = noble)
#
# /etc/apt/sources.list.d/docker.list
#   -> Docker 저장소 정보를 저장하는 파일
echo "deb [arch=$`(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu `$(. /etc/os-release && echo $VERSION_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list
# 등록 확인
cat /etc/apt/sources.list.d/docker.list

# 5. 새 Repository의 패키지 목록 가져오기
# Docker Repository를 새로 등록했으므로
# apt가 Docker 패키지 목록도 함께 가져온다.
sudo apt update

# 6. Docker 설치
# docker-ce               : Docker Engine
# docker-ce-cli           : docker 명령어
# containerd.io           : 실제 컨테이너 실행
# docker-buildx-plugin    : 최신 이미지 빌드 기능
# docker-compose-plugin   : Docker Compose V2
sudo apt install -y \
docker-ce \
docker-ce-cli \
containerd.io \
docker-buildx-plugin \
docker-compose-plugin

# 7. 설치 확인
# Docker 버전 확인
docker --version
# Docker Compose V2 확인
docker compose version

# 8. Docker 정상 동작 테스트
# hello-world 이미지 실행
#
# 과정
# 1. Docker Hub에서 이미지 다운로드
# 2. 컨테이너 생성
# 3. 컨테이너 실행
# 4. Hello World 출력
sudo docker run hello-world

# 9. ubuntu 사용자에게 Docker 권한 부여
# ubuntu 사용자를 docker 그룹에 추가
# 앞으로 sudo 없이 docker 명령을 사용할 수 있게 된다.
sudo usermod -aG docker ubuntu

# 10. 그룹 적용
# SSH 종료
exit
# 다시 SSH 접속
ssh -i .\your-key.pem ubuntu@<EC2_PUBLIC_IP_OR_DNS>

# 11. sudo 없이 Docker 실행 확인
docker run hello-world
```  

<br>

### Phase 4.2 EC2 Git Clone & Docker Compose Up  

<br>

`git clone` 명령으로 내가 로컬에서 올려놓은 git 프로젝트를 받았다.  
참고로 외부에서 private repo clone을 하려면 repo 권한을 부여한 classic token을 만들어 `USER_NAME`과 `TOKEN_KEY`를 입력해야한다.  

<br>

그리고 그냥 compose up 하면 된다. 아직 Dockerfile로 image 분리를 안 했기 때문이다.  

<br>

이때 인스턴스의 보안 인바운드 규칙에 다음 포트 설정을 추가해서 네트워크 접근을 열어야 한다.  
그러니까 main 서버 접근 포트인 fast api의 포트인 `8000`을 열어주었다.  
```  
TCP | 8000 | 0.0.0.0/0
```  

<br>

테스트하여 정상 작동 확인한 후에 Dockerfile을 분리하여 리팩토링 해주었다.  

<br>

### Phase 4.3 CI/CD  

<br>

1. CI:   

<br>


<br>

2. CD:   

<br>

## 남은 Phase  

<br>

Phase 5. 데이터베이스 성능 분석  
Phase 6. 부하 테스트  
Phase 7. 측정 기반 성능 개선  
Phase 8. Monitoring & Dashboard  

<br>

# Issue Handling  

<br>

```  
Redis
Circuit Breaker

DB 지식
서버 트래픽 몰리면 어떻게 하느냐
MSA 환경에서 메시지가 유실되면 발생할 수 있는 문제를 어떻게 해결할 것이냐
멀티스레딩
비관적락
InnoDB Buffer Pool
기술을 안 바꿔서 발생한 문제

1. 무거운 추론 요청을 보냈을 때 Redis 큐에 작업이 쌓이고
2. 하나의 트랜잭션에서 A를 저장하고 ai api를 호출해서 평균 10초동안 대기 후 결과를 받아와. 그리고 그 결과를 db에 저장해. 이 때 이 요청들을 하나의 트랜잭션에 넣으면 오랫동안 db 커넥션 풀을 점유해서 사용자가 몰렸을 때 서비스가 느려질 수 있는 문제가 있어.

내가 실제로 필요한 숫자

내 유사한 웹사이트 : 100개 웹사이트에서 크롤링햇고. 실제로내가 크롤링 해보니까 1개당 평균 500개 게시글 있음
기존은 한국꺼만햇고. 나는 외국웹사이트까찌 확장 예정이 있었어. 따라서 1000개가 내 최대 웹사이트 크롤링 갯수라 가정했고. 따라서 1000 * 500 으로 50만개의 데이터가

트래픽을 만들어야 함
근데 우리가 못 만듦
그래서 크롤링 쪽이 나쁘지 않음

k6로 부하테스트
Prometheus + Grafana
prometheus : 내 서버에다가 주기적으로 요청해서 서버 데이터를 가져오는거야
grafana : prometheus가 가져온 시계열 데이터를 대시보드로 시각화하는 툴이야

결제 시스템
- 토스 테스트 계정 API

Java SpringBoot

CI 메인/도커허브/테스트
CD SSH/서버 접속/도커/빌드

데이터사이언티스트 → AX TF에서 데이터 엔지니어링쪽? 데이터수집?
```  

<br>

```  
이 프로젝트의 목표

비동기 AI 추론 서버를 구축하며 FastAPI 기반의 백엔드 아키텍처와 코드를 익히고,
트러블 슈팅을 통해 하반기 백엔드+AI 관련 직무에 사용할 수 있는 기술 스택과 스토리를 쌓는다.

절대 하지 않을 것

과한 기술적 구현
기술적 할루시네이션을 동반한 설계 및 구축

기술 선택 기준

실무에서 널리 사용되고 Trade-off를 체험할 수 있으며,
무거운 AI 추론에 유리한 아키텍처 및 라이브러리를 택한다.

코드 작성 원칙

각각의 파일마다 해당 파일의 역할을 간략히 설명하는 한 줄 주석을 맨 위에 작성한다.
그 외에는 정석적인 코드 작성 원칙을 따른다.

아키텍처 원칙



Phase별 목표

Phase1 백엔드 CRUD 학습
Phase2 Worker 추가를 위한 아키텍처 구축 (Job Queue 등)
Phase3 ..

Commit 규칙

PR 규칙

AI(Codex/ChatGPT)가 따라야 하는 규칙
```  