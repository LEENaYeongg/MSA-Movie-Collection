아래는 첨부하신 **Requirements Analyst Agent Persona/System Prompt**를 기준으로 작성한 **“영화 컬렉션 관리 시스템 요구사항 분석서”**입니다. 이 Prompt는 요구사항 분석 Agent가 비즈니스 요구를 분석하고, MSA 설계에 필요한 명확한 요구사항 문서를 작성하도록 정의하고 있으며, 특히 **서비스 분리, 의존성 최소화, 서비스별 책임 경계 명확화**를 요구합니다. 

---

# 영화 컬렉션 관리 시스템 요구사항 분석서

## 1. 시스템 개요

### 1.1 시스템 목적

**영화 컬렉션 관리 시스템**은 사용자가 자신이 보유하거나 감상한 영화를 등록하고, 개인별 컬렉션으로 분류하며, 영화에 대한 평점·리뷰·시청 상태를 관리할 수 있도록 지원하는 서비스입니다.

이 시스템은 단일 애플리케이션 구조가 아니라 **Micro Service Architecture**를 적용하여 다음 목표를 달성합니다.

* 기능별 독립 서비스 구성
* 서비스별 책임 경계 명확화
* 서비스 간 직접 의존성 최소화
* 독립 배포 및 확장 가능성 확보
* 장애 발생 시 전체 시스템으로의 전파 최소화
* 향후 외부 영화 API, 추천 시스템, 알림 시스템 등 확장 가능성 확보

---

## 2. 사용자 유형

| 사용자 유형           | 설명                      | 주요 권한                             |
| ---------------- | ----------------------- | --------------------------------- |
| 일반 사용자           | 영화 컬렉션을 관리하는 개인 사용자     | 영화 조회, 컬렉션 생성, 리뷰 작성, 평점 등록, 즐겨찾기 |
| 관리자              | 영화 기본 데이터를 관리하는 운영자     | 영화 등록, 수정, 삭제, 부적절 리뷰 관리          |
| 외부 영화 데이터 제공 API | 영화 메타데이터를 제공하는 외부 시스템   | 영화 제목, 포스터, 배우, 감독, 장르 정보 제공      |
| 시스템 운영자          | 시스템 상태와 장애를 관리하는 운영 담당자 | 로그 확인, 모니터링, 장애 대응                |

---

## 3. 주요 유스케이스

| ID    | 유스케이스        | 사용자    | 설명                                |
| ----- | ------------ | ------ | --------------------------------- |
| UC-01 | 회원 가입        | 일반 사용자 | 이메일 또는 계정 정보로 회원 가입               |
| UC-02 | 로그인          | 일반 사용자 | 인증 후 Access Token 발급              |
| UC-03 | 영화 조회        | 일반 사용자 | 등록된 영화 목록 및 상세 정보 조회              |
| UC-04 | 영화 등록        | 관리자    | 신규 영화 메타데이터 등록                    |
| UC-05 | 컬렉션 생성       | 일반 사용자 | “내가 본 영화”, “보고 싶은 영화” 등 개인 컬렉션 생성 |
| UC-06 | 컬렉션에 영화 추가   | 일반 사용자 | 특정 영화를 개인 컬렉션에 추가                 |
| UC-07 | 영화 리뷰 작성     | 일반 사용자 | 감상평 작성                            |
| UC-08 | 영화 평점 등록     | 일반 사용자 | 영화별 개인 평점 등록                      |
| UC-09 | 영화 검색        | 일반 사용자 | 제목, 장르, 배우, 감독, 개봉연도 기준 검색        |
| UC-10 | 즐겨찾기 등록      | 일반 사용자 | 선호 영화 표시                          |
| UC-11 | 시청 상태 관리     | 일반 사용자 | 미시청, 시청 중, 시청 완료 등 상태 관리          |
| UC-12 | 리뷰 관리        | 관리자    | 부적절한 리뷰 숨김 또는 삭제                  |
| UC-13 | 외부 영화 데이터 연동 | 시스템    | 외부 API에서 영화 기본 정보 수집              |

---

## 4. 기능 요구사항

## 4.1 사용자 관리 요구사항

| ID         | 요구사항       | 설명                                 | 우선순위   | 책임 서비스       |
| ---------- | ---------- | ---------------------------------- | ------ | ------------ |
| FR-USER-01 | 회원 가입      | 사용자는 이메일, 이름, 비밀번호로 가입할 수 있어야 한다.  | High   | User Service |
| FR-USER-02 | 사용자 프로필 조회 | 사용자는 자신의 프로필을 조회할 수 있어야 한다.        | High   | User Service |
| FR-USER-03 | 사용자 프로필 수정 | 사용자는 닉네임, 프로필 이미지 등을 수정할 수 있어야 한다. | Medium | User Service |
| FR-USER-04 | 회원 탈퇴      | 사용자는 계정을 비활성화하거나 삭제 요청할 수 있어야 한다.  | Medium | User Service |

### 책임 경계

User Service는 **사용자 기본 정보**만 관리합니다.

User Service가 직접 관리하지 않는 정보:

* 로그인 토큰
* 영화 데이터
* 리뷰 데이터
* 컬렉션 데이터
* 평점 데이터

---

## 4.2 인증/인가 요구사항

| ID         | 요구사항  | 설명                              | 우선순위   | 책임 서비스                     |
| ---------- | ----- | ------------------------------- | ------ | -------------------------- |
| FR-AUTH-01 | 로그인   | 사용자는 이메일과 비밀번호로 로그인할 수 있어야 한다.  | High   | Auth Service               |
| FR-AUTH-02 | 토큰 발급 | 로그인 성공 시 Access Token을 발급해야 한다. | High   | Auth Service               |
| FR-AUTH-03 | 토큰 검증 | 보호 API 요청 시 토큰을 검증해야 한다.        | High   | Auth Service / API Gateway |
| FR-AUTH-04 | 권한 구분 | 일반 사용자와 관리자의 권한을 구분해야 한다.       | High   | Auth Service               |
| FR-AUTH-05 | 로그아웃  | 사용자는 로그아웃할 수 있어야 한다.            | Medium | Auth Service               |

### 책임 경계

Auth Service는 **인증과 권한 판단**만 담당합니다.

Auth Service가 직접 관리하지 않는 정보:

* 사용자 상세 프로필
* 영화 컬렉션
* 리뷰
* 영화 메타데이터

---

## 4.3 영화 정보 관리 요구사항

| ID          | 요구사항          | 설명                                            | 우선순위   | 책임 서비스                              |
| ----------- | ------------- | --------------------------------------------- | ------ | ----------------------------------- |
| FR-MOVIE-01 | 영화 등록         | 관리자는 영화 정보를 등록할 수 있어야 한다.                     | High   | Movie Service                       |
| FR-MOVIE-02 | 영화 수정         | 관리자는 영화 제목, 장르, 감독, 배우, 개봉연도 등을 수정할 수 있어야 한다. | High   | Movie Service                       |
| FR-MOVIE-03 | 영화 삭제         | 관리자는 잘못 등록된 영화 정보를 삭제하거나 비활성화할 수 있어야 한다.      | Medium | Movie Service                       |
| FR-MOVIE-04 | 영화 목록 조회      | 사용자는 영화 목록을 조회할 수 있어야 한다.                     | High   | Movie Service                       |
| FR-MOVIE-05 | 영화 상세 조회      | 사용자는 특정 영화의 상세 정보를 조회할 수 있어야 한다.              | High   | Movie Service                       |
| FR-MOVIE-06 | 외부 API 데이터 연동 | 외부 영화 API에서 영화 메타데이터를 가져올 수 있어야 한다.           | Medium | Movie Service / Integration Service |

### 책임 경계

Movie Service는 **영화의 기준 데이터**를 관리합니다.

Movie Service가 직접 관리하지 않는 정보:

* 사용자의 개인 컬렉션 포함 여부
* 사용자의 평점
* 사용자의 리뷰
* 즐겨찾기 여부
* 시청 상태

---

## 4.4 컬렉션 관리 요구사항

| ID        | 요구사항        | 설명                             | 우선순위   | 책임 서비스             |
| --------- | ----------- | ------------------------------ | ------ | ------------------ |
| FR-COL-01 | 컬렉션 생성      | 사용자는 개인 영화 컬렉션을 생성할 수 있어야 한다.  | High   | Collection Service |
| FR-COL-02 | 컬렉션 수정      | 사용자는 컬렉션 이름과 설명을 수정할 수 있어야 한다. | Medium | Collection Service |
| FR-COL-03 | 컬렉션 삭제      | 사용자는 자신의 컬렉션을 삭제할 수 있어야 한다.    | Medium | Collection Service |
| FR-COL-04 | 컬렉션 목록 조회   | 사용자는 자신의 컬렉션 목록을 조회할 수 있어야 한다. | High   | Collection Service |
| FR-COL-05 | 컬렉션에 영화 추가  | 사용자는 컬렉션에 영화를 추가할 수 있어야 한다.    | High   | Collection Service |
| FR-COL-06 | 컬렉션에서 영화 제거 | 사용자는 컬렉션에서 영화를 제거할 수 있어야 한다.   | High   | Collection Service |

### 책임 경계

Collection Service는 **사용자와 영화 간의 개인 컬렉션 관계**를 관리합니다.

Collection Service가 직접 관리하지 않는 정보:

* 영화 제목, 장르, 감독 등 영화 상세 메타데이터
* 리뷰 내용
* 평점 값
* 사용자 프로필 상세 정보

Collection Service는 영화 존재 여부 확인이 필요할 경우 **Movie Service API**를 호출하거나, 이벤트 기반으로 영화 요약 정보를 캐싱할 수 있습니다.

---

## 4.5 리뷰 및 평점 요구사항

| ID        | 요구사항      | 설명                               | 우선순위   | 책임 서비스         |
| --------- | --------- | -------------------------------- | ------ | -------------- |
| FR-REV-01 | 리뷰 작성     | 사용자는 영화에 리뷰를 작성할 수 있어야 한다.       | High   | Review Service |
| FR-REV-02 | 리뷰 수정     | 사용자는 자신이 작성한 리뷰를 수정할 수 있어야 한다.   | High   | Review Service |
| FR-REV-03 | 리뷰 삭제     | 사용자는 자신의 리뷰를 삭제할 수 있어야 한다.       | High   | Review Service |
| FR-REV-04 | 영화별 리뷰 조회 | 사용자는 특정 영화의 리뷰 목록을 조회할 수 있어야 한다. | High   | Review Service |
| FR-REV-05 | 평점 등록     | 사용자는 영화에 평점을 등록할 수 있어야 한다.       | High   | Review Service |
| FR-REV-06 | 평균 평점 계산  | 영화별 평균 평점을 제공할 수 있어야 한다.         | Medium | Review Service |
| FR-REV-07 | 부적절 리뷰 관리 | 관리자는 부적절한 리뷰를 숨김 처리할 수 있어야 한다.   | Medium | Review Service |

### 책임 경계

Review Service는 **리뷰와 평점 데이터**를 관리합니다.

Review Service가 직접 관리하지 않는 정보:

* 영화 상세 메타데이터
* 사용자 프로필 상세 정보
* 컬렉션 포함 여부

---

## 4.6 검색 요구사항

| ID           | 요구사항    | 설명                              | 우선순위   | 책임 서비스         |
| ------------ | ------- | ------------------------------- | ------ | -------------- |
| FR-SEARCH-01 | 제목 검색   | 사용자는 영화 제목으로 검색할 수 있어야 한다.      | High   | Search Service |
| FR-SEARCH-02 | 장르 검색   | 사용자는 장르 기준으로 영화를 검색할 수 있어야 한다.  | High   | Search Service |
| FR-SEARCH-03 | 감독 검색   | 사용자는 감독 이름으로 영화를 검색할 수 있어야 한다.  | Medium | Search Service |
| FR-SEARCH-04 | 배우 검색   | 사용자는 배우 이름으로 영화를 검색할 수 있어야 한다.  | Medium | Search Service |
| FR-SEARCH-05 | 개봉연도 검색 | 사용자는 개봉연도 기준으로 검색할 수 있어야 한다.    | Medium | Search Service |
| FR-SEARCH-06 | 복합 필터링  | 장르, 연도, 평점 등을 조합해 검색할 수 있어야 한다. | Medium | Search Service |

### 책임 경계

Search Service는 **조회 최적화된 검색 인덱스**를 관리합니다.

Search Service는 원천 데이터를 소유하지 않습니다.

* 영화 원천 데이터: Movie Service
* 리뷰/평점 원천 데이터: Review Service
* 사용자 컬렉션 원천 데이터: Collection Service

Search Service는 이벤트를 통해 필요한 검색 인덱스를 갱신하는 구조가 적합합니다.

---

## 4.7 즐겨찾기 및 시청 상태 요구사항

| ID        | 요구사항           | 설명                                      | 우선순위   | 책임 서비스                                |
| --------- | -------------- | --------------------------------------- | ------ | ------------------------------------- |
| FR-LIB-01 | 즐겨찾기 등록        | 사용자는 영화를 즐겨찾기에 추가할 수 있어야 한다.            | High   | Library Service 또는 Collection Service |
| FR-LIB-02 | 즐겨찾기 해제        | 사용자는 즐겨찾기를 해제할 수 있어야 한다.                | High   | Library Service 또는 Collection Service |
| FR-LIB-03 | 시청 상태 등록       | 사용자는 영화의 시청 상태를 설정할 수 있어야 한다.           | High   | Library Service                       |
| FR-LIB-04 | 시청 상태 변경       | 사용자는 미시청, 시청 중, 시청 완료 상태를 변경할 수 있어야 한다. | High   | Library Service                       |
| FR-LIB-05 | 개인 영화 라이브러리 조회 | 사용자는 자신의 영화 상태 목록을 조회할 수 있어야 한다.        | Medium | Library Service                       |

### 책임 경계

즐겨찾기와 시청 상태는 컬렉션과 유사하지만, 의미가 다릅니다.

* 컬렉션: 사용자가 만든 그룹
* 즐겨찾기: 선호 표시
* 시청 상태: 사용자별 영화 감상 진행 상태

따라서 초기 버전에서는 Collection Service에 포함할 수 있지만, 기능이 커질 경우 **Library Service**로 분리하는 것이 좋습니다.

---

## 5. 비기능 요구사항

| ID     | 항목      | 요구사항                                                           | MSA 관점               |
| ------ | ------- | -------------------------------------------------------------- | -------------------- |
| NFR-01 | 확장성     | 영화 조회, 검색, 리뷰 기능은 트래픽 증가에 따라 독립 확장 가능해야 한다.                    | 서비스별 Scale-out       |
| NFR-02 | 독립 배포   | User, Movie, Collection, Review, Search 서비스는 독립적으로 배포 가능해야 한다. | CI/CD 분리             |
| NFR-03 | 장애 격리   | Review Service 장애가 Movie Service 조회 전체를 중단시키면 안 된다.            | 장애 전파 차단             |
| NFR-04 | 응답 성능   | 일반 조회 API는 빠른 응답을 제공해야 한다.                                     | Search 인덱스 활용        |
| NFR-05 | 인증/인가   | 보호 API는 반드시 토큰 기반 인증을 거쳐야 한다.                                  | API Gateway + Auth   |
| NFR-06 | 데이터 정합성 | 서비스 간 데이터는 직접 DB 공유가 아니라 API 또는 이벤트로 동기화해야 한다.                 | Database per Service |
| NFR-07 | 로그 관리   | 서비스별 요청, 오류, 이벤트 로그를 남겨야 한다.                                   | 중앙 로그 수집             |
| NFR-08 | 모니터링    | 서비스 상태, 응답 시간, 장애율을 모니터링해야 한다.                                 | Observability        |
| NFR-09 | 테스트 용이성 | 서비스별 단위 테스트와 통합 테스트가 가능해야 한다.                                  | 독립 테스트               |
| NFR-10 | 보안      | 사용자 비밀번호는 암호화 저장되어야 하며, 민감 정보는 로그에 남기면 안 된다.                   | Auth/User 책임 분리      |

---

## 6. 도메인 개념 정의

| 도메인 개념         | 설명                  | 소유 서비스                                |
| -------------- | ------------------- | ------------------------------------- |
| User           | 시스템을 사용하는 회원        | User Service                          |
| AuthCredential | 로그인 인증 정보           | Auth Service                          |
| Role           | 사용자 권한              | Auth Service                          |
| Movie          | 영화 기준 정보            | Movie Service                         |
| Genre          | 영화 장르               | Movie Service                         |
| Actor          | 배우 정보               | Movie Service                         |
| Director       | 감독 정보               | Movie Service                         |
| Collection     | 사용자가 만든 영화 묶음       | Collection Service                    |
| CollectionItem | 컬렉션에 포함된 영화 항목      | Collection Service                    |
| Review         | 사용자가 작성한 영화 리뷰      | Review Service                        |
| Rating         | 사용자가 부여한 영화 평점      | Review Service                        |
| Favorite       | 사용자의 즐겨찾기 상태        | Library Service 또는 Collection Service |
| WatchStatus    | 미시청, 시청 중, 시청 완료 상태 | Library Service                       |
| SearchIndex    | 검색 최적화용 인덱스         | Search Service                        |
| Notification   | 알림 메시지              | Notification Service                  |

---

## 7. 서비스 분리 후보

## 7.1 1차 핵심 서비스

| 서비스                | 주요 책임                 | 소유 데이터         | 의존성                  |
| ------------------ | --------------------- | -------------- | -------------------- |
| API Gateway        | 외부 요청 진입점, 라우팅, 인증 필터 | 없음             | Auth Service         |
| Auth Service       | 로그인, 토큰 발급, 권한 검증     | 인증 정보, 권한      | User Service 최소 연동   |
| User Service       | 사용자 기본 정보 관리          | 사용자 프로필        | Auth Service와 분리     |
| Movie Service      | 영화 기준 정보 관리           | 영화, 장르, 배우, 감독 | 외부 영화 API            |
| Collection Service | 사용자 컬렉션 관리            | 컬렉션, 컬렉션 항목    | User ID, Movie ID 참조 |
| Review Service     | 리뷰 및 평점 관리            | 리뷰, 평점         | User ID, Movie ID 참조 |
| Search Service     | 영화 검색 및 필터링           | 검색 인덱스         | Movie/Review 이벤트 구독  |

---

## 7.2 확장 후보 서비스

| 서비스                    | 분리 시점                | 이유                      |
| ---------------------- | -------------------- | ----------------------- |
| Library Service        | 즐겨찾기, 시청 상태 기능이 커질 때 | 개인 영화 상태 관리 책임 분리       |
| Notification Service   | 알림 기능 추가 시           | 리뷰 알림, 컬렉션 공유 알림 등      |
| Recommendation Service | 추천 기능 추가 시           | 사용자 취향 기반 영화 추천         |
| Integration Service    | 외부 영화 API 연동이 복잡해질 때 | 외부 API 장애와 내부 핵심 서비스 분리 |
| Admin Service          | 관리자 기능이 커질 때         | 운영 기능과 사용자 기능 분리        |

---

## 8. 서비스 책임 경계

## 8.1 서비스별 책임 경계 원칙

| 원칙                                    | 설명                                                    |
| ------------------------------------- | ----------------------------------------------------- |
| 서비스는 하나의 명확한 비즈니스 책임을 가진다             | Movie Service는 영화 정보만, Review Service는 리뷰와 평점만 담당     |
| 서비스는 자신의 데이터만 직접 수정한다                 | 다른 서비스 DB 직접 접근 금지                                    |
| 서비스 간 연동은 API 또는 이벤트로 제한한다            | DB Join 대신 API 호출 또는 이벤트 기반 복제                        |
| 조회 성능을 위해 중복 데이터는 허용하되 원천 소유자를 명확히 한다 | Search Service는 Movie 데이터를 복제해도 원천은 Movie Service     |
| 장애 전파를 최소화한다                          | Review Service 장애가 Movie 조회를 중단시키지 않도록 설계             |
| 서비스 간 순환 의존성을 금지한다                    | Collection → Movie 호출은 가능하나 Movie → Collection 호출은 지양 |

---

## 8.2 서비스 간 의존성 최소화 방향

### 바람직하지 않은 구조

```text
Collection Service → Movie DB 직접 조회
Review Service → User DB 직접 조회
Search Service → Movie DB + Review DB 직접 Join
```

이 구조는 MSA 원칙에 맞지 않습니다.

### 권장 구조

```text
Collection Service → Movie Service API 또는 MovieCreated 이벤트 사용
Review Service → Movie ID, User ID만 저장
Search Service → MovieUpdated, ReviewCreated 이벤트를 구독하여 인덱스 갱신
API Gateway → Auth Service를 통해 토큰 검증
```

---

## 9. 요구사항 우선순위

## 9.1 MVP 필수 요구사항

| 우선순위 | 기능             |
| ---- | -------------- |
| High | 회원 가입          |
| High | 로그인 / 토큰 발급    |
| High | 영화 등록          |
| High | 영화 목록 조회       |
| High | 영화 상세 조회       |
| High | 컬렉션 생성         |
| High | 컬렉션에 영화 추가     |
| High | 리뷰 작성          |
| High | 평점 등록          |
| High | 제목/장르 기반 영화 검색 |

---

## 9.2 2차 개발 요구사항

| 우선순위   | 기능           |
| ------ | ------------ |
| Medium | 외부 영화 API 연동 |
| Medium | 복합 검색 필터     |
| Medium | 즐겨찾기         |
| Medium | 시청 상태 관리     |
| Medium | 평균 평점 계산     |
| Medium | 관리자 리뷰 관리    |
| Medium | 알림 기능        |

---

## 9.3 향후 확장 요구사항

| 우선순위 | 기능           |
| ---- | ------------ |
| Low  | 영화 추천        |
| Low  | 컬렉션 공유       |
| Low  | 팔로우 기반 소셜 기능 |
| Low  | 감상 통계 대시보드   |
| Low  | AI 기반 리뷰 요약  |
| Low  | 선호 장르 분석     |

---

## 10. 설계 단계로 전달할 핵심 사항

### 10.1 서비스 분해 기준

설계 단계에서는 다음 서비스를 우선 설계해야 합니다.

```text
api-gateway
auth-service
user-service
movie-service
collection-service
review-service
search-service
```

초기 MVP에서는 `notification-service`, `recommendation-service`, `integration-service`는 선택적으로 후순위 설계합니다.

---

### 10.2 서비스별 데이터 소유권

| 데이터        | 소유 서비스                                |
| ---------- | ------------------------------------- |
| 사용자 프로필    | User Service                          |
| 로그인 인증 정보  | Auth Service                          |
| 영화 메타데이터   | Movie Service                         |
| 컬렉션        | Collection Service                    |
| 컬렉션 항목     | Collection Service                    |
| 리뷰         | Review Service                        |
| 평점         | Review Service                        |
| 검색 인덱스     | Search Service                        |
| 즐겨찾기/시청 상태 | Library Service 또는 Collection Service |

---

### 10.3 서비스 간 통신 방향

| From               | To             | 방식                | 목적                |
| ------------------ | -------------- | ----------------- | ----------------- |
| API Gateway        | Auth Service   | Sync API          | 토큰 검증             |
| Collection Service | Movie Service  | Sync API 또는 Event | 영화 존재 여부 확인       |
| Review Service     | Movie Service  | Sync API 또는 Event | 리뷰 대상 영화 확인       |
| Movie Service      | Search Service | Event             | 영화 검색 인덱스 갱신      |
| Review Service     | Search Service | Event             | 평점/리뷰 기반 검색 정보 갱신 |
| User Service       | Auth Service   | Sync API 또는 Event | 사용자 생성 후 인증 정보 연결 |

---

### 10.4 이벤트 후보

| 이벤트               | 발행 서비스             | 구독 서비스                               | 목적                |
| ----------------- | ------------------ | ------------------------------------ | ----------------- |
| UserCreated       | User Service       | Auth Service, Notification Service   | 사용자 생성 후 인증/알림 처리 |
| MovieCreated      | Movie Service      | Search Service                       | 검색 인덱스 생성         |
| MovieUpdated      | Movie Service      | Search Service                       | 검색 인덱스 갱신         |
| MovieDeleted      | Movie Service      | Search Service, Collection Service   | 비활성 영화 처리         |
| CollectionCreated | Collection Service | Notification Service                 | 컬렉션 생성 알림         |
| ReviewCreated     | Review Service     | Search Service, Notification Service | 평점/리뷰 검색 반영       |
| RatingUpdated     | Review Service     | Search Service                       | 평균 평점 갱신          |

---

## 11. MSA 관점 전체 구성도

```mermaid
flowchart LR
    Client[Web / Mobile Client]

    Gateway[API Gateway]

    Auth[Auth Service]
    User[User Service]
    Movie[Movie Service]
    Collection[Collection Service]
    Review[Review Service]
    Search[Search Service]
    Notify[Notification Service<br/>Optional]
    ExternalAPI[External Movie API]

    UserDB[(User DB)]
    AuthDB[(Auth DB)]
    MovieDB[(Movie DB)]
    CollectionDB[(Collection DB)]
    ReviewDB[(Review DB)]
    SearchIndex[(Search Index)]

    Broker[(Message Broker)]

    Client --> Gateway

    Gateway --> Auth
    Gateway --> User
    Gateway --> Movie
    Gateway --> Collection
    Gateway --> Review
    Gateway --> Search

    Auth --> AuthDB
    User --> UserDB
    Movie --> MovieDB
    Collection --> CollectionDB
    Review --> ReviewDB
    Search --> SearchIndex

    Movie --> ExternalAPI

    Movie --> Broker
    Review --> Broker
    Collection --> Broker
    User --> Broker

    Broker --> Search
    Broker --> Notify
```

---

## 12. 확인 필요 사항

| 항목        | 확인 질문                                                 |
| --------- | ----------------------------------------------------- |
| 사용자 가입 방식 | 이메일 가입만 지원할 것인지, 소셜 로그인도 포함할 것인지 확인 필요                |
| 영화 데이터 출처 | 관리자가 직접 등록할 것인지, 외부 영화 API를 기본으로 사용할 것인지 확인 필요        |
| 컬렉션 공개 여부 | 개인 컬렉션만 지원할 것인지, 공개/공유 컬렉션도 지원할 것인지 확인 필요             |
| 리뷰 정책     | 리뷰 신고, 블라인드, 관리자 승인 기능이 필요한지 확인 필요                    |
| 평점 기준     | 5점 만점, 10점 만점, 별점 단위 등 기준 확인 필요                       |
| 검색 엔진     | 단순 DB 검색으로 시작할지, Elasticsearch/OpenSearch를 사용할지 확인 필요 |
| 추천 기능     | MVP에 포함할지, 향후 확장 기능으로 둘지 확인 필요                        |
| 알림 기능     | 이메일, SMS, Push 중 어떤 채널을 사용할지 확인 필요                    |
| 관리자 화면    | 별도 Admin Web이 필요한지 확인 필요                              |
| 배포 환경     | Docker Compose 수준인지, Kubernetes 기반인지 확인 필요            |

---

# 최종 정리

이 요구사항 분석의 핵심은 단순히 기능을 나누는 것이 아니라, **서비스가 어떤 데이터를 소유하고 어떤 책임을 갖는지 명확히 구분하는 것**입니다.

초기 MVP 기준으로는 다음 구조가 가장 적합합니다.

```text
API Gateway
 ├── Auth Service
 ├── User Service
 ├── Movie Service
 ├── Collection Service
 ├── Review Service
 └── Search Service
```

그리고 MSA 설계 단계에서는 반드시 다음 원칙을 유지해야 합니다.

```text
1. 서비스별 DB 독립
2. 다른 서비스 DB 직접 접근 금지
3. 서비스 간 API 또는 Event 기반 통신
4. 검색 서비스는 원천 데이터 소유자가 아니라 인덱스 소유자
5. 리뷰/평점은 Movie Service가 아니라 Review Service 책임
6. 컬렉션은 Movie 데이터가 아니라 User-Movie 관계 데이터
7. 장애 발생 시 핵심 조회 기능이 최대한 유지되도록 설계
```
