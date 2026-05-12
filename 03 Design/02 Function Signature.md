아래는 첨부된 **「영화 컬렉션 관리 시스템 MSA 설계서」**를 기준으로 확장한 **Function Signature 설계서**입니다.
이번 단계에서는 **코드 구현 없이**, 다음 단계의 Coding Agent가 클래스·메서드·DTO·Event Handler를 일관되게 구성할 수 있도록 **서비스별 기능 함수의 책임, 입력, 출력, 외부 의존성**을 정의합니다. 

---

# 영화 컬렉션 관리 시스템

# Function Signature 설계서

---

## 1. Function Signature 설계 목적

Function Signature는 각 마이크로서비스가 제공해야 하는 **핵심 기능 단위의 계약(Contract)** 을 정의합니다.

본 문서는 다음을 목적으로 합니다.

| 목적                    | 설명                                          |
| --------------------- | ------------------------------------------- |
| 서비스 책임 명확화            | 어떤 기능을 어느 서비스가 담당하는지 고정                     |
| Coding Agent 구현 기준 제공 | Controller, Service, Event Handler 작성 기준 제공 |
| 서비스 간 의존성 통제          | 내부 함수가 다른 서비스에 과도하게 의존하지 않도록 제어             |
| 테스트 설계 기준 제공          | 함수별 단위 테스트 및 통합 테스트 기준 마련                   |
| API/Event 설계와의 연결     | REST Endpoint와 내부 Application Function을 정렬  |

기존 설계서에서 정의한 원칙인 **서비스별 독립 책임**, **Database per Service**, **REST/Event 기반 연동**, **Search Service는 검색 인덱스만 소유**라는 기준을 그대로 유지합니다. 

---

# 2. Function Signature 작성 원칙

## 2.1 공통 원칙

| 원칙                          | 설명                         |
| --------------------------- | -------------------------- |
| Command / Query 분리          | 변경 기능과 조회 기능을 구분           |
| 입력은 명시적 Command/Query 객체 사용 | 원시 값 나열보다 목적이 분명한 입력 단위 사용 |
| 출력은 Result/View 객체로 반환      | 서비스 외부에 내부 Entity 직접 노출 금지 |
| 서비스 간 참조는 ID 기반             | `movieId`, `userId`만 보유    |
| 외부 서비스 호출은 최소화              | 필수 검증 외에는 이벤트 기반 반영 우선     |
| 이벤트 발행 여부 명시                | 상태 변경 함수는 후속 이벤트를 명확히 지정   |
| 실패 조건 명시                    | 권한 없음, 존재하지 않음, 중복 등록 등    |

---

## 2.2 Function Signature 표기 형식

아래 형식을 사용합니다.

| 항목                 | 의미                      |
| ------------------ | ----------------------- |
| Function Signature | 기능 함수 명세                |
| 입력                 | Command / Query / Event |
| 출력                 | Result / View / Ack     |
| 책임                 | 함수가 수행하는 단일 책임          |
| 선행 검증              | 수행 전 확인해야 할 조건          |
| 외부 의존성             | 다른 서비스 또는 Broker 호출 여부  |
| 후속 이벤트             | 상태 변경 후 발행해야 할 이벤트      |

---

# 3. 공통 Function Signature 유형

## 3.1 공통 유형 정의

| 유형                       | 용도                   |
| ------------------------ | -------------------- |
| Command Function         | 생성, 수정, 삭제 등 상태 변경   |
| Query Function           | 조회, 검색, 상세 정보 반환     |
| Validation Function      | 권한, 존재 여부, 정책 조건 검증  |
| Event Publisher Function | 상태 변경 후 이벤트 발행       |
| Event Handler Function   | 외부 이벤트 수신 후 내부 상태 반영 |

---

# 4. API Gateway Function Signature

API Gateway는 비즈니스 로직을 수행하지 않고, **라우팅·인증 컨텍스트 전달·공통 정책 적용**에 집중합니다. 설계서에서도 Gateway는 외부 진입점이며, 각 서비스의 내부 도메인 기능을 직접 소유하지 않는 것으로 정의되어 있습니다. 

## 4.1 Gateway 내부 기능 명세

| Function Signature     | 입력                              | 출력                     | 책임                                            |
| ---------------------- | ------------------------------- | ---------------------- | --------------------------------------------- |
| routeRequest           | ClientRequest                   | RoutedRequest          | 요청 경로를 내부 서비스로 전달                             |
| authenticateRequest    | AuthorizationHeader             | AuthContext            | 토큰 검증 결과를 사용자 컨텍스트로 변환                        |
| authorizeRequest       | AuthContext, AccessPolicy       | AuthorizationResult    | 역할 기반 접근 허용 여부 판단                             |
| enrichRequestContext   | RoutedRequest, AuthContext      | InternalServiceRequest | `X-User-Id`, `X-User-Role`, `X-Request-Id` 추가 |
| applyRateLimit         | ClientIdentity, RoutePolicy     | RateLimitDecision      | 초과 요청 차단                                      |
| normalizeErrorResponse | DownstreamError                 | StandardErrorResponse  | 서비스별 오류 응답 형식 표준화                             |
| writeAccessLog         | RequestContext, ResponseContext | LogAck                 | 공통 접근 로그 기록                                   |

---

# 5. Auth Service Function Signature

Auth Service는 **인증, 토큰, 권한 판단**만 담당합니다. 사용자 프로필이나 리뷰, 컬렉션 데이터는 관리하지 않습니다. 

---

## 5.1 Command Functions

| Function Signature      | 입력                   | 출력                           | 책임                                  | 후속 이벤트      |
| ----------------------- | -------------------- | ---------------------------- | ----------------------------------- | ----------- |
| loginUser               | LoginCommand         | TokenPairResult              | 이메일/비밀번호 검증 후 토큰 발급                 | 없음          |
| logoutUser              | LogoutCommand        | LogoutResult                 | Refresh Token 또는 세션 무효화             | 없음          |
| refreshAccessToken      | RefreshTokenCommand  | TokenPairResult              | Refresh Token 검증 후 Access Token 재발급 | 없음          |
| createCredentialForUser | UserCreatedEvent     | CredentialCreationResult     | 신규 사용자용 인증 레코드 생성                   | 없음          |
| deactivateCredential    | UserDeactivatedEvent | CredentialDeactivationResult | 탈퇴 사용자 인증 차단                        | 없음          |
| updateUserRole          | RoleUpdateCommand    | RoleUpdateResult             | 사용자 권한 변경                           | RoleUpdated |

---

## 5.2 Query / Validation Functions

| Function Signature | 입력                      | 출력                       | 책임           |
| ------------------ | ----------------------- | ------------------------ | ------------ |
| verifyAccessToken  | AccessTokenQuery        | TokenVerificationResult  | 토큰 유효성 검증    |
| getUserRole        | UserRoleQuery           | UserRoleResult           | 사용자 역할 반환    |
| validatePassword   | PasswordValidationQuery | PasswordValidationResult | 로그인 비밀번호 검증  |
| isTokenRevoked     | TokenRevocationQuery    | TokenRevocationResult    | 토큰 무효화 여부 확인 |

---

## 5.3 주요 실패 조건

| 조건                        | 설명                      |
| ------------------------- | ----------------------- |
| INVALID_CREDENTIAL        | 이메일 또는 비밀번호 불일치         |
| TOKEN_EXPIRED             | Access/Refresh Token 만료 |
| TOKEN_REVOKED             | 로그아웃 처리된 토큰             |
| USER_CREDENTIAL_NOT_FOUND | 인증 계정 없음                |
| ROLE_UPDATE_FORBIDDEN     | 권한 변경 권한 없음             |

---

# 6. User Service Function Signature

User Service는 **회원 프로필과 사용자 상태**를 관리합니다. 인증 정보는 Auth Service로 분리되어 있습니다. 

---

## 6.1 Command Functions

| Function Signature  | 입력                        | 출력                       | 책임               | 후속 이벤트          |
| ------------------- | ------------------------- | ------------------------ | ---------------- | --------------- |
| registerUser        | UserRegistrationCommand   | UserRegistrationResult   | 신규 사용자 프로필 생성    | UserCreated     |
| updateMyProfile     | ProfileUpdateCommand      | ProfileUpdateResult      | 본인 프로필 수정        | UserUpdated     |
| deactivateMyAccount | UserDeactivationCommand   | UserDeactivationResult   | 회원 탈퇴 또는 비활성화 처리 | UserDeactivated |
| updateProfileImage  | ProfileImageUpdateCommand | ProfileImageUpdateResult | 사용자 프로필 이미지 변경   | UserUpdated     |

---

## 6.2 Query Functions

| Function Signature | 입력                 | 출력                  | 책임                   |
| ------------------ | ------------------ | ------------------- | -------------------- |
| getMyProfile       | CurrentUserQuery   | UserProfileView     | 본인 프로필 조회            |
| getUserSummary     | UserSummaryQuery   | UserSummaryView     | 내부 서비스용 사용자 요약 정보 반환 |
| checkUserExists    | UserExistenceQuery | UserExistenceResult | 사용자 존재 여부 확인         |
| getUserStatus      | UserStatusQuery    | UserStatusResult    | 활성/비활성 상태 조회         |

---

## 6.3 주요 실패 조건

| 조건                       | 설명             |
| ------------------------ | -------------- |
| EMAIL_ALREADY_EXISTS     | 이미 가입된 이메일     |
| USER_NOT_FOUND           | 사용자 없음         |
| PROFILE_UPDATE_FORBIDDEN | 본인 외 프로필 수정 시도 |
| USER_ALREADY_DEACTIVATED | 이미 비활성화된 계정    |

---

# 7. Movie Service Function Signature

Movie Service는 영화의 **기준 데이터(System of Record)** 를 소유합니다. 컬렉션, 리뷰, 평점은 소유하지 않습니다. 

---

## 7.1 Command Functions

| Function Signature            | 입력                         | 출력                    | 책임                  | 후속 이벤트        |
| ----------------------------- | -------------------------- | --------------------- | ------------------- | ------------- |
| createMovie                   | MovieCreationCommand       | MovieCreationResult   | 신규 영화 등록            | MovieCreated  |
| updateMovie                   | MovieUpdateCommand         | MovieUpdateResult     | 영화 메타데이터 수정         | MovieUpdated  |
| deactivateMovie               | MovieDeletionCommand       | MovieDeletionResult   | 영화 삭제 또는 비활성화       | MovieDeleted  |
| importMovieFromExternalSource | ExternalMovieImportCommand | MovieImportResult     | 외부 API 기반 영화 데이터 등록 | MovieImported |
| updateMovieCast               | MovieCastUpdateCommand     | MovieCastUpdateResult | 배우/감독 관련 정보 갱신      | MovieUpdated  |

---

## 7.2 Query / Validation Functions

| Function Signature    | 입력                   | 출력                   | 책임                       |
| --------------------- | -------------------- | -------------------- | ------------------------ |
| getMovieList          | MovieListQuery       | MovieListView        | 영화 목록 조회                 |
| getMovieDetail        | MovieDetailQuery     | MovieDetailView      | 영화 상세 조회                 |
| checkMovieExists      | MovieExistenceQuery  | MovieExistenceResult | 다른 서비스에서 사용할 영화 존재 여부 확인 |
| getMovieSummary       | MovieSummaryQuery    | MovieSummaryView     | 검색/컬렉션 조합용 간단 정보 반환      |
| findMovieByExternalId | ExternalMovieIdQuery | MovieMatchResult     | 외부 API ID 기반 중복 여부 확인    |

---

## 7.3 주요 실패 조건

| 조건                        | 설명               |
| ------------------------- | ---------------- |
| MOVIE_NOT_FOUND           | 영화 없음            |
| MOVIE_ALREADY_EXISTS      | 동일 기준의 영화가 이미 존재 |
| INVALID_MOVIE_METADATA    | 필수 메타데이터 누락      |
| ADMIN_PERMISSION_REQUIRED | 관리자 권한 필요        |
| EXTERNAL_SOURCE_FAILURE   | 외부 영화 API 연동 실패  |

---

# 8. Collection Service Function Signature

Collection Service는 **사용자와 영화 간의 컬렉션 관계 데이터**를 소유합니다. 영화의 상세 정보는 직접 저장하지 않고, `movieId`를 참조합니다. 

---

## 8.1 Command Functions

| Function Signature        | 입력                            | 출력                           | 책임               | 외부 의존성              | 후속 이벤트                |
| ------------------------- | ----------------------------- | ---------------------------- | ---------------- | ------------------- | --------------------- |
| createCollection          | CollectionCreationCommand     | CollectionCreationResult     | 사용자 컬렉션 생성       | 없음                  | CollectionCreated     |
| updateCollection          | CollectionUpdateCommand       | CollectionUpdateResult       | 컬렉션명/설명/공개 상태 수정 | 없음                  | CollectionUpdated     |
| deleteCollection          | CollectionDeletionCommand     | CollectionDeletionResult     | 컬렉션 삭제           | 없음                  | CollectionDeleted     |
| addMovieToCollection      | CollectionItemAdditionCommand | CollectionItemAdditionResult | 컬렉션에 영화 추가       | Movie Service 존재 확인 | CollectionItemAdded   |
| removeMovieFromCollection | CollectionItemRemovalCommand  | CollectionItemRemovalResult  | 컬렉션에서 영화 제거      | 없음                  | CollectionItemRemoved |

---

## 8.2 Query Functions

| Function Signature       | 입력                              | 출력                               | 책임                 |
| ------------------------ | ------------------------------- | -------------------------------- | ------------------ |
| getMyCollections         | MyCollectionListQuery           | CollectionListView               | 로그인 사용자의 컬렉션 목록 조회 |
| getCollectionDetail      | CollectionDetailQuery           | CollectionDetailView             | 특정 컬렉션 상세 조회       |
| getCollectionItems       | CollectionItemListQuery         | CollectionItemListView           | 컬렉션 내 영화 ID 목록 조회  |
| checkCollectionOwnership | CollectionOwnershipQuery        | CollectionOwnershipResult        | 해당 컬렉션의 소유자 검증     |
| checkMovieAlreadyAdded   | CollectionMovieDuplicationQuery | CollectionMovieDuplicationResult | 동일 영화 중복 추가 여부 확인  |

---

## 8.3 주요 실패 조건

| 조건                            | 설명             |
| ----------------------------- | -------------- |
| COLLECTION_NOT_FOUND          | 컬렉션 없음         |
| COLLECTION_OWNERSHIP_REQUIRED | 본인 컬렉션이 아님     |
| MOVIE_NOT_FOUND               | 추가 대상 영화 없음    |
| MOVIE_ALREADY_IN_COLLECTION   | 이미 컬렉션에 추가된 영화 |
| COLLECTION_ITEM_NOT_FOUND     | 제거 대상 항목 없음    |

---

# 9. Review Service Function Signature

Review Service는 **리뷰와 평점의 원천 소유자**입니다. Movie Service와는 `movieId`만 공유하고, 영화 메타데이터를 직접 저장하지 않습니다. 

---

## 9.1 Review Command Functions

| Function Signature | 입력                      | 출력                     | 책임           | 외부 의존성              | 후속 이벤트          |
| ------------------ | ----------------------- | ---------------------- | ------------ | ------------------- | --------------- |
| createReview       | ReviewCreationCommand   | ReviewCreationResult   | 영화 리뷰 작성     | Movie Service 존재 확인 | ReviewCreated   |
| updateReview       | ReviewUpdateCommand     | ReviewUpdateResult     | 본인 리뷰 수정     | 없음                  | ReviewUpdated   |
| deleteReview       | ReviewDeletionCommand   | ReviewDeletionResult   | 본인 리뷰 삭제     | 없음                  | ReviewDeleted   |
| hideReviewByAdmin  | ReviewModerationCommand | ReviewModerationResult | 관리자 리뷰 숨김 처리 | 없음                  | ReviewModerated |

---

## 9.2 Rating Command Functions

| Function Signature   | 입력                    | 출력                   | 책임             | 외부 의존성              | 후속 이벤트        |
| -------------------- | --------------------- | -------------------- | -------------- | ------------------- | ------------- |
| createOrUpdateRating | RatingUpsertCommand   | RatingUpsertResult   | 평점 최초 등록 또는 수정 | Movie Service 존재 확인 | RatingUpdated |
| deleteRating         | RatingDeletionCommand | RatingDeletionResult | 사용자의 평점 삭제     | 없음                  | RatingDeleted |

---

## 9.3 Query Functions

| Function Signature      | 입력                      | 출력                    | 책임                |
| ----------------------- | ----------------------- | --------------------- | ----------------- |
| getReviewsByMovie       | MovieReviewListQuery    | ReviewListView        | 영화별 리뷰 목록 조회      |
| getMyReviews            | MyReviewListQuery       | ReviewListView        | 로그인 사용자의 리뷰 목록 조회 |
| getReviewDetail         | ReviewDetailQuery       | ReviewDetailView      | 리뷰 상세 조회          |
| getRatingSummaryByMovie | MovieRatingSummaryQuery | RatingSummaryView     | 평균 평점, 평점 수 조회    |
| getMyRatingForMovie     | MyMovieRatingQuery      | UserRatingView        | 사용자의 특정 영화 평점 조회  |
| checkReviewOwnership    | ReviewOwnershipQuery    | ReviewOwnershipResult | 리뷰 수정/삭제 권한 확인    |

---

## 9.4 주요 실패 조건

| 조건                                | 설명           |
| --------------------------------- | ------------ |
| REVIEW_NOT_FOUND                  | 리뷰 없음        |
| REVIEW_OWNERSHIP_REQUIRED         | 본인 리뷰가 아님    |
| MOVIE_NOT_FOUND                   | 리뷰 대상 영화 없음  |
| INVALID_RATING_SCORE              | 평점 범위 오류     |
| DUPLICATE_REVIEW_POLICY_VIOLATION | 중복 리뷰 정책 위반  |
| ADMIN_PERMISSION_REQUIRED         | 관리자 기능 권한 없음 |

---

# 10. Search Service Function Signature

Search Service는 원천 데이터를 소유하지 않고, Movie Service와 Review Service 이벤트를 받아 **검색 인덱스**를 구성합니다. 

---

## 10.1 Query Functions

| Function Signature              | 입력                        | 출력                    | 책임           |
| ------------------------------- | ------------------------- | --------------------- | ------------ |
| searchMoviesByKeyword           | MovieKeywordSearchQuery   | MovieSearchResultView | 제목 중심 키워드 검색 |
| filterMoviesByGenre             | MovieGenreSearchQuery     | MovieSearchResultView | 장르 기준 검색     |
| filterMoviesByActor             | MovieActorSearchQuery     | MovieSearchResultView | 배우 기준 검색     |
| filterMoviesByDirector          | MovieDirectorSearchQuery  | MovieSearchResultView | 감독 기준 검색     |
| filterMoviesByReleaseYear       | MovieYearSearchQuery      | MovieSearchResultView | 개봉연도 기준 검색   |
| filterMoviesByMinimumRating     | MovieRatingSearchQuery    | MovieSearchResultView | 최소 평점 기준 검색  |
| searchMoviesWithCompositeFilter | CompositeMovieSearchQuery | MovieSearchResultView | 복합 조건 검색     |
| getSearchIndexHealth            | SearchIndexHealthQuery    | SearchIndexHealthView | 검색 인덱스 상태 조회 |

---

## 10.2 Event Handler Functions

| Function Signature         | 입력 이벤트                       | 출력                   | 책임                  |
| -------------------------- | ---------------------------- | -------------------- | ------------------- |
| handleMovieCreated         | MovieCreatedEvent            | SearchIndexUpdateAck | 신규 영화 검색 인덱스 생성     |
| handleMovieUpdated         | MovieUpdatedEvent            | SearchIndexUpdateAck | 영화 검색 문서 갱신         |
| handleMovieDeleted         | MovieDeletedEvent            | SearchIndexUpdateAck | 비활성화 영화 검색 제거 또는 숨김 |
| handleReviewCreated        | ReviewCreatedEvent           | SearchIndexUpdateAck | 리뷰 수 반영             |
| handleReviewDeleted        | ReviewDeletedEvent           | SearchIndexUpdateAck | 리뷰 수 차감             |
| handleRatingUpdated        | RatingUpdatedEvent           | SearchIndexUpdateAck | 평균 평점 정보 갱신         |
| rebuildMovieSearchDocument | SearchDocumentRebuildCommand | SearchIndexUpdateAck | 특정 영화 검색 문서 재생성     |

---

## 10.3 주요 실패 조건

| 조건                        | 설명           |
| ------------------------- | ------------ |
| SEARCH_INDEX_UNAVAILABLE  | 검색 인덱스 연결 실패 |
| SEARCH_DOCUMENT_NOT_FOUND | 인덱스 문서 없음    |
| INVALID_SEARCH_FILTER     | 검색 필터 값 오류   |
| EVENT_PAYLOAD_INVALID     | 이벤트 데이터 부족   |

---

# 11. Notification Service Function Signature

Notification Service는 비즈니스 핵심 기능과 분리된 **이벤트 기반 부가 서비스**입니다. 장애가 발생해도 핵심 서비스는 계속 동작해야 합니다. 

---

## 11.1 Event Handler Functions

| Function Signature                    | 입력 이벤트                   | 출력                      | 책임                |
| ------------------------------------- | ------------------------ | ----------------------- | ----------------- |
| handleUserCreatedNotification         | UserCreatedEvent         | NotificationDispatchAck | 가입 환영 알림 생성       |
| handleCollectionCreatedNotification   | CollectionCreatedEvent   | NotificationDispatchAck | 컬렉션 생성 알림         |
| handleCollectionItemAddedNotification | CollectionItemAddedEvent | NotificationDispatchAck | 컬렉션 변경 알림         |
| handleReviewCreatedNotification       | ReviewCreatedEvent       | NotificationDispatchAck | 리뷰 등록 관련 알림       |
| handleRatingUpdatedNotification       | RatingUpdatedEvent       | NotificationDispatchAck | 평점 변경 알림, 필요 시 확장 |

---

## 11.2 Command / Query Functions

| Function Signature     | 입력                          | 출력                         | 책임           |
| ---------------------- | --------------------------- | -------------------------- | ------------ |
| dispatchNotification   | NotificationDispatchCommand | NotificationDispatchResult | 채널별 알림 발송    |
| saveNotificationLog    | NotificationLogCommand      | NotificationLogResult      | 알림 이력 저장     |
| getMyNotifications     | MyNotificationListQuery     | NotificationListView       | 사용자 알림 목록 조회 |
| markNotificationAsRead | NotificationReadCommand     | NotificationReadResult     | 알림 읽음 처리     |

---

# 12. 서비스 간 Outbound Port Function Signature

서비스 간 의존성을 최소화하기 위해, 내부 구현은 외부 서비스를 직접 알지 않고 **Outbound Port 수준의 계약**으로 분리하는 것이 적절합니다.

---

## 12.1 Collection Service → Movie Service

| Function Signature             | 입력                  | 출력                   | 사용 목적               |
| ------------------------------ | ------------------- | -------------------- | ------------------- |
| verifyMovieExistsForCollection | MovieExistenceQuery | MovieExistenceResult | 컬렉션에 추가 가능한 영화인지 확인 |

---

## 12.2 Review Service → Movie Service

| Function Signature         | 입력                  | 출력                   | 사용 목적              |
| -------------------------- | ------------------- | -------------------- | ------------------ |
| verifyMovieExistsForReview | MovieExistenceQuery | MovieExistenceResult | 리뷰/평점 대상 영화 유효성 확인 |

---

## 12.3 API Gateway → Auth Service

| Function Signature          | 입력               | 출력                      | 사용 목적         |
| --------------------------- | ---------------- | ----------------------- | ------------- |
| verifyAccessTokenForGateway | AccessTokenQuery | TokenVerificationResult | 외부 요청 인증 검증   |
| resolveUserRoleForGateway   | UserRoleQuery    | UserRoleResult          | 관리자 API 접근 제어 |

---

# 13. 이벤트 발행 Function Signature

상태 변경 함수는 이벤트 발행과 함께 설계되어야 합니다. 설계서에서 정의한 주요 이벤트는 Movie, Review, Collection, User 변화에 따라 Search 및 Notification Service로 전파됩니다. 

| Function Signature              | 입력                         | 출력              | 발행 이벤트              |
| ------------------------------- | -------------------------- | --------------- | ------------------- |
| publishUserCreatedEvent         | UserCreatedPayload         | EventPublishAck | UserCreated         |
| publishUserUpdatedEvent         | UserUpdatedPayload         | EventPublishAck | UserUpdated         |
| publishMovieCreatedEvent        | MovieCreatedPayload        | EventPublishAck | MovieCreated        |
| publishMovieUpdatedEvent        | MovieUpdatedPayload        | EventPublishAck | MovieUpdated        |
| publishMovieDeletedEvent        | MovieDeletedPayload        | EventPublishAck | MovieDeleted        |
| publishCollectionCreatedEvent   | CollectionCreatedPayload   | EventPublishAck | CollectionCreated   |
| publishCollectionItemAddedEvent | CollectionItemAddedPayload | EventPublishAck | CollectionItemAdded |
| publishReviewCreatedEvent       | ReviewCreatedPayload       | EventPublishAck | ReviewCreated       |
| publishReviewDeletedEvent       | ReviewDeletedPayload       | EventPublishAck | ReviewDeleted       |
| publishRatingUpdatedEvent       | RatingUpdatedPayload       | EventPublishAck | RatingUpdated       |

---

# 14. 서비스별 Function Signature 요약

## 14.1 핵심 Command 계열

| 서비스          | 핵심 Command Function                                               |
| ------------ | ----------------------------------------------------------------- |
| Auth         | loginUser, logoutUser, refreshAccessToken                         |
| User         | registerUser, updateMyProfile, deactivateMyAccount                |
| Movie        | createMovie, updateMovie, deactivateMovie                         |
| Collection   | createCollection, addMovieToCollection, removeMovieFromCollection |
| Review       | createReview, updateReview, deleteReview                          |
| Rating       | createOrUpdateRating, deleteRating                                |
| Notification | dispatchNotification                                              |

---

## 14.2 핵심 Query 계열

| 서비스          | 핵심 Query Function                                        |
| ------------ | -------------------------------------------------------- |
| Auth         | verifyAccessToken, getUserRole                           |
| User         | getMyProfile, getUserSummary                             |
| Movie        | getMovieList, getMovieDetail, checkMovieExists           |
| Collection   | getMyCollections, getCollectionDetail                    |
| Review       | getReviewsByMovie, getMyReviews, getRatingSummaryByMovie |
| Search       | searchMoviesByKeyword, searchMoviesWithCompositeFilter   |
| Notification | getMyNotifications                                       |

---

# 15. Coding Agent에게 전달할 Function Signature 구현 지침

다음 단계에서 Coding Agent는 아래 기준으로 상세 구현을 진행하면 됩니다.

| 지침                                      | 설명                                           |
| --------------------------------------- | -------------------------------------------- |
| Controller와 Application Function 1:1 대응 | REST Endpoint는 명확한 Application Function으로 연결 |
| Entity 직접 반환 금지                         | Result/View 객체 사용                            |
| 상태 변경 후 Event 발행                        | Movie/Review/Collection/User 변경은 이벤트와 연결     |
| 외부 서비스 호출은 Port로 분리                     | 직접 HTTP Client 의존을 Service 내부에 박지 않음         |
| Cross-Service 조회는 최소화                   | 필수 존재 검증만 동기 호출                              |
| Search는 Event Handler 중심                | 원천 DB 직접 조회 금지                               |
| Notification은 Event Consumer 중심         | 핵심 트랜잭션과 분리                                  |
| 테스트는 Function 단위로 설계                    | Command/Query/Event Handler별 테스트 케이스 작성      |

---

# 16. 최종 요약

이번 Function Signature 설계는 기존 MSA 설계서의 책임 경계를 함수 수준까지 구체화한 것입니다.

핵심 구조는 다음과 같습니다.

```text id="rk6u9q"
Auth Service
- 인증과 권한 함수

User Service
- 사용자 프로필 함수

Movie Service
- 영화 기준정보 함수

Collection Service
- 사용자-영화 컬렉션 관계 함수

Review Service
- 리뷰와 평점 함수

Search Service
- 검색 질의 함수 + 이벤트 반영 함수

Notification Service
- 이벤트 기반 알림 함수
```

가장 중요한 설계 원칙은 다음입니다.

```text id="rgwt8s"
1. 함수는 서비스 책임을 넘지 않는다.
2. 다른 서비스의 데이터를 직접 조작하지 않는다.
3. 상태 변경 함수는 후속 이벤트를 명확히 가진다.
4. 조회 함수와 변경 함수는 분리한다.
5. 외부 서비스 호출은 최소화하고 Port 형태로 격리한다.
```
