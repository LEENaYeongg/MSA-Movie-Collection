아래는 첨부하신 **「01 Design Spec.md」**와 **「02 Function Signature.md」**를 기준으로 작성한 **“영화 컬렉션 관리 시스템 Code Skeleton”**입니다.
이번 단계에서는 **함수 내부 구현 로직은 작성하지 않고**, 다음 Coding 단계에서 바로 세부 구현을 시작할 수 있도록 **MSA 서비스별 디렉터리 구조, 계층 구조, 클래스, 함수 선언부, Port Interface, Event Handler Skeleton**만 작성했습니다.

특히 기존 설계 원칙인 **서비스 책임 경계 유지**, **Command/Query 분리**, **Outbound Port 기반 외부 의존성 격리**, **Event 기반 후속 처리**, **Entity 직접 반환 금지**를 그대로 반영했습니다.  

---

# 영화 컬렉션 관리 시스템

# Code Skeleton

---

# 1. Code Skeleton 작성 기준

## 1.1 적용 기준

본 Skeleton은 다음 기준으로 작성합니다.

| 항목             | 적용                                            |
| -------------- | --------------------------------------------- |
| Backend Style  | Python / FastAPI-style service skeleton       |
| Architecture   | Layered + Hexagonal Port 분리                   |
| 함수 본문          | 구현 없음, `...` placeholder만 사용                  |
| 함수 Description | 상세한 English Docstring 포함                      |
| 데이터 계약         | Command / Query / Result / View / Event 개념 유지 |
| 외부 의존성         | Port Interface로 격리                            |
| 서비스 간 연결       | REST Port 또는 Event Handler                    |
| 이벤트            | Publisher / Consumer Skeleton 분리              |

---

# 2. 전체 프로젝트 디렉터리 구조

```text
movie-collection-system/
├── shared/
│   ├── __init__.py
│   ├── contexts/
│   │   ├── auth_context.py
│   │   └── request_context.py
│   ├── contracts/
│   │   ├── api_response.py
│   │   ├── error_response.py
│   │   ├── event_envelope.py
│   │   └── pagination.py
│   ├── ports/
│   │   └── event_publisher_port.py
│   └── observability/
│       └── logging_contract.py
│
├── api-gateway/
│   └── app/
│       ├── main.py
│       ├── api/
│       │   └── proxy_routes.py
│       ├── application/
│       │   └── gateway_application_service.py
│       ├── contracts/
│       │   └── gateway_contracts.py
│       └── ports/
│           ├── auth_verification_port.py
│           └── downstream_routing_port.py
│
├── auth-service/
│   └── app/
│       ├── main.py
│       ├── api/
│       │   └── auth_routes.py
│       ├── application/
│       │   ├── auth_command_service.py
│       │   ├── auth_query_service.py
│       │   └── auth_event_handler.py
│       ├── contracts/
│       │   └── auth_contracts.py
│       ├── domain/
│       │   └── auth_entities.py
│       ├── ports/
│       │   ├── credential_repository_port.py
│       │   ├── token_repository_port.py
│       │   ├── token_provider_port.py
│       │   └── event_publisher_port.py
│       └── events/
│           └── auth_events.py
│
├── user-service/
│   └── app/
│       ├── main.py
│       ├── api/
│       │   └── user_routes.py
│       ├── application/
│       │   ├── user_command_service.py
│       │   └── user_query_service.py
│       ├── contracts/
│       │   └── user_contracts.py
│       ├── domain/
│       │   └── user_entities.py
│       ├── ports/
│       │   ├── user_repository_port.py
│       │   └── event_publisher_port.py
│       └── events/
│           └── user_events.py
│
├── movie-service/
│   └── app/
│       ├── main.py
│       ├── api/
│       │   ├── movie_routes.py
│       │   └── admin_movie_routes.py
│       ├── application/
│       │   ├── movie_command_service.py
│       │   ├── movie_query_service.py
│       │   └── movie_import_service.py
│       ├── contracts/
│       │   └── movie_contracts.py
│       ├── domain/
│       │   └── movie_entities.py
│       ├── ports/
│       │   ├── movie_repository_port.py
│       │   ├── external_movie_provider_port.py
│       │   └── event_publisher_port.py
│       └── events/
│           └── movie_events.py
│
├── collection-service/
│   └── app/
│       ├── main.py
│       ├── api/
│       │   └── collection_routes.py
│       ├── application/
│       │   ├── collection_command_service.py
│       │   └── collection_query_service.py
│       ├── contracts/
│       │   └── collection_contracts.py
│       ├── domain/
│       │   └── collection_entities.py
│       ├── ports/
│       │   ├── collection_repository_port.py
│       │   ├── movie_validation_port.py
│       │   └── event_publisher_port.py
│       └── events/
│           └── collection_events.py
│
├── review-service/
│   └── app/
│       ├── main.py
│       ├── api/
│       │   ├── review_routes.py
│       │   └── rating_routes.py
│       ├── application/
│       │   ├── review_command_service.py
│       │   ├── rating_command_service.py
│       │   └── review_query_service.py
│       ├── contracts/
│       │   └── review_contracts.py
│       ├── domain/
│       │   └── review_entities.py
│       ├── ports/
│       │   ├── review_repository_port.py
│       │   ├── rating_repository_port.py
│       │   ├── movie_validation_port.py
│       │   └── event_publisher_port.py
│       └── events/
│           └── review_events.py
│
├── search-service/
│   └── app/
│       ├── main.py
│       ├── api/
│       │   └── search_routes.py
│       ├── application/
│       │   ├── search_query_service.py
│       │   └── search_index_event_handler.py
│       ├── contracts/
│       │   └── search_contracts.py
│       ├── ports/
│       │   └── search_index_repository_port.py
│       └── events/
│           └── consumed_search_events.py
│
├── notification-service/
│   └── app/
│       ├── main.py
│       ├── api/
│       │   └── notification_routes.py
│       ├── application/
│       │   ├── notification_command_service.py
│       │   ├── notification_query_service.py
│       │   └── notification_event_handler.py
│       ├── contracts/
│       │   └── notification_contracts.py
│       ├── domain/
│       │   └── notification_entities.py
│       ├── ports/
│       │   ├── notification_repository_port.py
│       │   └── notification_dispatch_port.py
│       └── events/
│           └── consumed_notification_events.py
│
└── infra/
    ├── docker-compose.yml
    ├── broker/
    ├── database/
    ├── gateway/
    └── monitoring/
```

---

# 3. Shared Skeleton

---

## 3.1 `shared/contexts/auth_context.py`

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class AuthContext:
    """
    Represents the authenticated identity propagated across internal service boundaries.

    This context is expected to be resolved at the API Gateway layer after token
    verification and then forwarded to downstream services through trusted headers
    or internal request metadata.

    Responsibilities:
    - Carry the authenticated user's unique identifier.
    - Carry the resolved authorization role.
    - Prevent downstream services from repeatedly parsing raw access tokens.
    - Provide a stable input contract for application-level permission checks.

    Non-responsibilities:
    - This object does not verify JWT signatures.
    - This object does not load user profile information.
    - This object does not perform role-based access control by itself.
    """

    user_id: str
    role: str
```

---

## 3.2 `shared/contexts/request_context.py`

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class RequestContext:
    """
    Describes the request-level execution context shared across services.

    The context is used for observability, traceability, and consistent logging.
    It should be created at the edge of the system and propagated through every
    synchronous service call whenever possible.

    Responsibilities:
    - Carry a request identifier for distributed tracing.
    - Carry the originating client correlation identifier when available.
    - Provide a common metadata object for structured logs and error responses.

    Non-responsibilities:
    - This object does not contain business payload data.
    - This object does not perform tracing or logging by itself.
    """

    request_id: str
    correlation_id: str | None = None
```

---

## 3.3 `shared/contracts/event_envelope.py`

```python
from dataclasses import dataclass
from datetime import datetime
from typing import Generic, TypeVar


T = TypeVar("T")


@dataclass(frozen=True)
class EventEnvelope(Generic[T]):
    """
    Defines the canonical envelope for domain and integration events.

    Every event exchanged through the message broker should be wrapped by this
    envelope so that consumers can process events consistently regardless of
    their producing service.

    Responsibilities:
    - Standardize event identity and event type information.
    - Record the time at which the event was produced.
    - Identify the source service that emitted the event.
    - Carry the event-specific payload in a transport-neutral format.

    Non-responsibilities:
    - This contract does not persist event messages.
    - This contract does not guarantee delivery semantics.
    - This contract does not define broker-specific serialization details.
    """

    event_id: str
    event_type: str
    occurred_at: datetime
    source: str
    payload: T
```

---

## 3.4 `shared/ports/event_publisher_port.py`

```python
from typing import Protocol
from shared.contracts.event_envelope import EventEnvelope


class EventPublisherPort(Protocol):
    async def publish_event(self, event: EventEnvelope[object]) -> None:
        """
        Publishes a domain or integration event to the configured message broker.

        The application layer depends on this abstraction rather than directly
        depending on Kafka, RabbitMQ, or any specific messaging technology.

        Responsibilities:
        - Accept a fully prepared event envelope from the application layer.
        - Deliver the event to the infrastructure-specific publisher adapter.
        - Preserve application-level decoupling from broker implementation details.

        Non-responsibilities:
        - This port does not decide whether an event should be emitted.
        - This port does not construct event payloads.
        - This port does not implement retry, persistence, or outbox logic directly.
        """
        ...
```

---

# 4. API Gateway Code Skeleton

---

## 4.1 `api-gateway/app/application/gateway_application_service.py`

```python
from shared.contexts.auth_context import AuthContext
from shared.contexts.request_context import RequestContext


class GatewayApplicationService:
    async def route_request(self, client_request: object) -> object:
        """
        Routes an incoming external request to the appropriate downstream service.

        The function resolves the target service based on the external URL pattern,
        prepares the downstream request object, and delegates the final transport
        operation to the configured routing adapter.

        Responsibilities:
        - Match incoming request paths against gateway routing rules.
        - Resolve the appropriate downstream service target.
        - Preserve request metadata that must travel across service boundaries.
        - Keep business logic out of the gateway layer.

        Non-responsibilities:
        - This function does not execute domain logic.
        - This function does not directly modify user, movie, review, or collection data.
        """
        ...

    async def authenticate_request(self, authorization_header: str) -> AuthContext:
        """
        Resolves authentication information from the incoming authorization header.

        The function delegates token verification to the Auth Service through an
        outbound verification port and converts the result into an internal
        AuthContext object that downstream services can consume.

        Responsibilities:
        - Extract the bearer token or equivalent credential format.
        - Delegate token verification to the dedicated authentication boundary.
        - Produce a normalized AuthContext for internal request propagation.

        Non-responsibilities:
        - This function does not issue tokens.
        - This function does not access credential storage.
        - This function does not evaluate business-specific permissions.
        """
        ...

    async def authorize_request(
        self,
        auth_context: AuthContext,
        access_policy: object,
    ) -> object:
        """
        Evaluates whether an authenticated principal may access a protected route.

        This function applies gateway-level policy rules such as requiring an
        administrator role for administrative APIs or requiring any authenticated
        user role for protected user APIs.

        Responsibilities:
        - Compare the authenticated role against the route access policy.
        - Produce a decision object that indicates allow or deny.
        - Centralize first-level route authorization before downstream dispatch.

        Non-responsibilities:
        - This function does not verify JWT signatures.
        - This function does not determine resource ownership.
        - This function does not replace service-level authorization checks.
        """
        ...

    async def enrich_request_context(
        self,
        routed_request: object,
        auth_context: AuthContext | None,
        request_context: RequestContext,
    ) -> object:
        """
        Adds trusted internal context headers or metadata to the downstream request.

        The function ensures that downstream services receive consistent identity
        and observability information without re-parsing public-facing credentials.

        Responsibilities:
        - Add request identifier and correlation identifier metadata.
        - Add authenticated user identifier when available.
        - Add authenticated user role when available.
        - Produce a downstream-ready internal request representation.

        Non-responsibilities:
        - This function does not alter the business payload.
        - This function does not construct domain commands.
        """
        ...

    async def apply_rate_limit(self, client_identity: object, route_policy: object) -> object:
        """
        Applies gateway-level request throttling policies.

        The function checks whether the current client identity is allowed to
        continue based on route-specific quota rules and request frequency.

        Responsibilities:
        - Resolve the applicable rate-limiting policy.
        - Evaluate usage against the defined threshold.
        - Return a decision that can be translated into allow or reject behavior.

        Non-responsibilities:
        - This function does not execute downstream API calls.
        - This function does not perform application-specific validation.
        """
        ...

    async def normalize_error_response(self, downstream_error: object) -> object:
        """
        Converts downstream service errors into a gateway-standard response format.

        The gateway should provide a consistent external error contract even when
        individual services use their own internal exception hierarchies.

        Responsibilities:
        - Map internal or downstream error representations to a common response contract.
        - Preserve meaningful error codes and traceability identifiers.
        - Avoid leaking service internals or sensitive technical details.

        Non-responsibilities:
        - This function does not fix downstream errors.
        - This function does not retry failed domain commands.
        """
        ...

    async def write_access_log(
        self,
        request_context: RequestContext,
        response_context: object,
    ) -> None:
        """
        Records a structured access log entry for an external gateway transaction.

        The access log is used for traffic auditing, latency monitoring, and request
        tracing across the distributed system.

        Responsibilities:
        - Capture request identifiers and route information.
        - Capture response status and total gateway latency.
        - Provide a consistent audit point for external system access.

        Non-responsibilities:
        - This function does not persist business audit trails.
        - This function does not replace service-specific operational logs.
        """
        ...
```

---

# 5. Auth Service Code Skeleton

---

## 5.1 `auth-service/app/application/auth_command_service.py`

```python
class AuthCommandService:
    async def login_user(self, command: object) -> object:
        """
        Authenticates a user credential and issues a new access/refresh token pair.

        The command is expected to contain the login identifier and the raw secret
        required for authentication. The function validates the credential through
        dedicated domain and repository abstractions, then delegates token creation
        to the token provider port.

        Responsibilities:
        - Validate the submitted login identifier and secret.
        - Load the corresponding credential record.
        - Verify credential state such as active or deactivated status.
        - Issue a token pair for a successfully authenticated user.

        Non-responsibilities:
        - This function does not create user profiles.
        - This function does not manage movie, review, or collection data.
        - This function does not return raw credential storage records.
        """
        ...

    async def logout_user(self, command: object) -> object:
        """
        Revokes or invalidates the current refresh token or session record.

        The logout function creates an authentication-side invalidation boundary so
        that future token refresh attempts can be rejected according to the system's
        security policy.

        Responsibilities:
        - Resolve the token or session targeted for logout.
        - Mark the relevant refresh token or session as no longer valid.
        - Return a logout result that confirms the invalidation operation.

        Non-responsibilities:
        - This function does not destroy the user's profile.
        - This function does not revoke domain data owned by other services.
        """
        ...

    async def refresh_access_token(self, command: object) -> object:
        """
        Generates a renewed access token by validating an existing refresh token.

        The function preserves the separation between short-lived access tokens and
        longer-lived refresh tokens while enforcing token revocation and expiration
        policies.

        Responsibilities:
        - Validate the provided refresh token.
        - Confirm that the token is not expired or revoked.
        - Issue a renewed token pair or a renewed access token according to policy.

        Non-responsibilities:
        - This function does not authenticate by password.
        - This function does not modify user profile data.
        """
        ...

    async def update_user_role(self, command: object) -> object:
        """
        Changes the authorization role associated with a user credential record.

        This operation is intended for administration scenarios and must remain
        isolated inside the Auth Service because role ownership belongs to the
        authentication and authorization boundary.

        Responsibilities:
        - Validate that the requested role transition is allowed.
        - Update the user's effective role record.
        - Produce an event-friendly result for role changes when required.

        Non-responsibilities:
        - This function does not update profile names or contact information.
        - This function does not perform administrative UI logic.
        """
        ...
```

---

## 5.2 `auth-service/app/application/auth_query_service.py`

```python
class AuthQueryService:
    async def verify_access_token(self, query: object) -> object:
        """
        Verifies whether an access token is authentic, valid, and currently usable.

        This function is typically called by the API Gateway through an internal
        verification port and returns normalized identity claims that can be mapped
        into a trusted AuthContext.

        Responsibilities:
        - Validate token integrity and expiration.
        - Resolve subject identity and role claims.
        - Return a verification result suitable for gateway decisions.

        Non-responsibilities:
        - This function does not issue new tokens.
        - This function does not load a full user profile.
        """
        ...

    async def get_user_role(self, query: object) -> object:
        """
        Retrieves the current authorization role associated with a specific user.

        This function provides an internal, authentication-owned source of truth
        for role-based gateway and administration decisions.

        Responsibilities:
        - Load the effective role associated with the requested user identifier.
        - Return a role-oriented query result without exposing credential internals.

        Non-responsibilities:
        - This function does not manage permissions outside the Auth Service scope.
        - This function does not validate resource ownership in other services.
        """
        ...

    async def validate_password(self, query: object) -> object:
        """
        Evaluates whether a submitted raw secret matches the stored password hash.

        The method exists as an authentication-specific validation operation and
        should remain private to the Auth Service application boundary.

        Responsibilities:
        - Compare submitted credentials with secure stored credential material.
        - Return a structured validation decision.

        Non-responsibilities:
        - This function does not generate tokens.
        - This function does not mutate credential records.
        """
        ...

    async def is_token_revoked(self, query: object) -> object:
        """
        Determines whether a token or token family has been explicitly revoked.

        The function supports logout semantics and compromised-session containment
        by providing a dedicated revocation lookup abstraction.

        Responsibilities:
        - Resolve the token revocation state.
        - Return a clear revoked/not-revoked decision result.

        Non-responsibilities:
        - This function does not validate token signatures.
        - This function does not perform token refresh.
        """
        ...
```

---

## 5.3 `auth-service/app/application/auth_event_handler.py`

```python
class AuthEventHandler:
    async def create_credential_for_user(self, event: object) -> object:
        """
        Creates an authentication credential record when a new user account is announced.

        This handler consumes the UserCreated event emitted by User Service and creates
        the Auth Service-side credential representation required for future login.

        Responsibilities:
        - Interpret the user creation event payload.
        - Create the initial credential record for the new user identity.
        - Preserve independence between profile ownership and credential ownership.

        Non-responsibilities:
        - This handler does not create the user profile itself.
        - This handler does not emit user lifecycle events on behalf of User Service.
        """
        ...

    async def deactivate_credential(self, event: object) -> object:
        """
        Disables an authentication credential after the owning user account is deactivated.

        This handler consumes the UserDeactivated event and prevents future login or
        token refresh operations for the affected user identity.

        Responsibilities:
        - Resolve the credential record associated with the deactivated user.
        - Mark the credential state as unusable.
        - Preserve account lifecycle consistency across service boundaries.

        Non-responsibilities:
        - This handler does not delete user profile records.
        - This handler does not remove reviews or collections owned by the user.
        """
        ...
```

---

# 6. User Service Code Skeleton

---

## 6.1 `user-service/app/application/user_command_service.py`

```python
class UserCommandService:
    async def register_user(self, command: object) -> object:
        """
        Creates a new user profile within the User Service ownership boundary.

        This function establishes the profile-side representation of a platform user.
        Authentication-side records are intentionally not created here directly;
        instead, a UserCreated event is expected to trigger downstream credential
        initialization in the Auth Service.

        Responsibilities:
        - Validate profile registration input.
        - Enforce uniqueness constraints owned by the User Service.
        - Persist the new user profile.
        - Prepare the UserCreated event contract.

        Non-responsibilities:
        - This function does not hash or store passwords.
        - This function does not issue login tokens.
        - This function does not create collections or review records.
        """
        ...

    async def update_my_profile(self, command: object) -> object:
        """
        Updates mutable profile attributes for the authenticated user.

        The command should carry the actor identity and the requested profile changes.
        The function validates ownership implicitly through actor identity and applies
        only profile-domain modifications.

        Responsibilities:
        - Confirm that the requesting user is updating their own profile.
        - Validate mutable profile fields.
        - Persist the allowed profile changes.
        - Prepare a UserUpdated event if the update succeeds.

        Non-responsibilities:
        - This function does not change authentication role or password.
        - This function does not edit notification preferences unless modeled separately.
        """
        ...

    async def deactivate_my_account(self, command: object) -> object:
        """
        Deactivates the authenticated user's profile lifecycle state.

        The function represents user-driven account deactivation at the profile domain
        level and prepares a UserDeactivated event so dependent services may respond
        without requiring direct database coupling.

        Responsibilities:
        - Confirm that the actor owns the target profile.
        - Transition the profile into a deactivated state.
        - Prepare the lifecycle event for downstream consumers.

        Non-responsibilities:
        - This function does not directly revoke authentication tokens.
        - This function does not cascade-delete data from other microservices.
        """
        ...

    async def update_profile_image(self, command: object) -> object:
        """
        Updates the profile image reference for the authenticated user.

        The service is responsible for storing or referencing the final profile image
        location, while binary object storage concerns may be delegated to a separate
        infrastructure adapter if needed.

        Responsibilities:
        - Confirm ownership of the profile being updated.
        - Validate the new profile image reference contract.
        - Persist the updated profile image metadata.
        - Prepare a UserUpdated event after successful mutation.

        Non-responsibilities:
        - This function does not resize or transform images.
        - This function does not manage storage provider internals.
        """
        ...
```

---

## 6.2 `user-service/app/application/user_query_service.py`

```python
class UserQueryService:
    async def get_my_profile(self, query: object) -> object:
        """
        Retrieves the full profile view for the authenticated user.

        The result is intended for self-service profile screens and should expose a
        user-facing view contract rather than a persistence entity.

        Responsibilities:
        - Resolve the requested user profile by authenticated identity.
        - Return the profile fields approved for self-view consumption.

        Non-responsibilities:
        - This function does not expose credential secrets or token data.
        - This function does not fetch review or collection aggregates.
        """
        ...

    async def get_user_summary(self, query: object) -> object:
        """
        Retrieves a limited internal summary view for a specific user identity.

        This query is intended for service-to-service read use cases where downstream
        domains need a stable lightweight representation without owning the User model.

        Responsibilities:
        - Resolve a compact summary of a user's identity-facing information.
        - Return only fields explicitly approved for cross-service sharing.

        Non-responsibilities:
        - This function does not return sensitive credential data.
        - This function does not authorize resource ownership in foreign services.
        """
        ...

    async def check_user_exists(self, query: object) -> object:
        """
        Confirms whether a user profile exists in the User Service system of record.

        This query is suitable for internal validation scenarios that require existence
        awareness without loading a full user view.

        Responsibilities:
        - Resolve existence by user identifier.
        - Return a normalized existence result.

        Non-responsibilities:
        - This function does not create or modify user profiles.
        - This function does not validate authentication status.
        """
        ...

    async def get_user_status(self, query: object) -> object:
        """
        Returns the current lifecycle status of a user profile.

        The result may be used by internal logic that needs to understand whether a
        profile is active, suspended, or deactivated without depending on raw storage
        structures.

        Responsibilities:
        - Load the user status from the User Service domain record.
        - Return a status-specific result object.

        Non-responsibilities:
        - This function does not alter the lifecycle state.
        - This function does not revoke authentication tokens.
        """
        ...
```

---

# 7. Movie Service Code Skeleton

---

## 7.1 `movie-service/app/application/movie_command_service.py`

```python
class MovieCommandService:
    async def create_movie(self, command: object) -> object:
        """
        Creates a new movie metadata record as the authoritative movie-domain entity.

        This function is reserved for administrative or controlled ingestion workflows
        and represents the primary write entry point for movie catalog data.

        Responsibilities:
        - Validate required movie metadata.
        - Enforce movie-domain duplication rules.
        - Persist the movie record and related movie-domain associations.
        - Prepare a MovieCreated event for downstream indexing services.

        Non-responsibilities:
        - This function does not create reviews or collection items.
        - This function does not compute personalized recommendation data.
        """
        ...

    async def update_movie(self, command: object) -> object:
        """
        Updates existing movie metadata under the Movie Service ownership boundary.

        The update operation may affect title, description, release year, display media,
        and controlled domain associations such as genre or cast references.

        Responsibilities:
        - Confirm movie existence.
        - Validate administrative write permission at the application boundary.
        - Apply approved metadata changes.
        - Prepare a MovieUpdated event for search index synchronization.

        Non-responsibilities:
        - This function does not mutate review aggregates directly.
        - This function does not modify collection membership.
        """
        ...

    async def deactivate_movie(self, command: object) -> object:
        """
        Marks a movie as deleted or inactive without coupling directly to dependent services.

        Instead of directly editing Search, Collection, or Review Service records, the Movie
        Service emits a lifecycle event so downstream services can respond independently.

        Responsibilities:
        - Confirm the movie exists and is eligible for deactivation.
        - Apply the Movie Service-side inactive lifecycle state.
        - Prepare a MovieDeleted event.

        Non-responsibilities:
        - This function does not directly remove collection items.
        - This function does not directly delete search documents.
        """
        ...

    async def update_movie_cast(self, command: object) -> object:
        """
        Updates movie-domain cast and creator associations.

        The function maintains the Movie Service as the source of truth for actor and
        director relationships and prepares downstream index synchronization through
        the standard movie update event path.

        Responsibilities:
        - Validate cast update payloads.
        - Preserve movie-domain integrity rules.
        - Persist actor/director relationship changes.
        - Prepare a MovieUpdated event.

        Non-responsibilities:
        - This function does not modify review or rating data.
        - This function does not execute search index writes directly.
        """
        ...
```

---

## 7.2 `movie-service/app/application/movie_import_service.py`

```python
class MovieImportService:
    async def import_movie_from_external_source(self, command: object) -> object:
        """
        Imports a movie metadata candidate from an external movie information provider.

        This function coordinates external provider access, duplicate detection, and
        internal movie creation preparation while keeping vendor-specific integration
        details behind an outbound provider port.

        Responsibilities:
        - Resolve external movie metadata using the provider port.
        - Validate and normalize imported fields into internal movie-domain structures.
        - Detect duplicates using external identifiers or catalog matching rules.
        - Prepare a MovieImported or equivalent catalog lifecycle event.

        Non-responsibilities:
        - This function does not expose provider-native payloads as domain models.
        - This function does not bypass Movie Service validation rules.
        """
        ...
```

---

## 7.3 `movie-service/app/application/movie_query_service.py`

```python
class MovieQueryService:
    async def get_movie_list(self, query: object) -> object:
        """
        Retrieves a paginated list of movie catalog entries.

        The function returns a user-facing movie list view rather than persistence
        entities and serves as the standard read path for general movie browsing.

        Responsibilities:
        - Apply list paging and allowed catalog filters.
        - Resolve active movie records from the Movie Service-owned data store.
        - Return a normalized list view.

        Non-responsibilities:
        - This function does not provide full search relevance ranking.
        - This function does not aggregate review statistics owned by Review Service.
        """
        ...

    async def get_movie_detail(self, query: object) -> object:
        """
        Retrieves detailed metadata for a single movie catalog item.

        This query remains limited to Movie Service-owned metadata. Any review, rating,
        or personalized library data should be composed elsewhere rather than violating
        service ownership boundaries.

        Responsibilities:
        - Confirm movie existence.
        - Return movie-domain detail data approved for public consumption.

        Non-responsibilities:
        - This function does not fetch reviews.
        - This function does not fetch user collection membership.
        """
        ...

    async def check_movie_exists(self, query: object) -> object:
        """
        Confirms whether a movie exists and is available for cross-service referencing.

        Collection Service and Review Service may depend on this lightweight validation
        contract instead of reading Movie Service storage directly.

        Responsibilities:
        - Evaluate existence and availability by movie identifier.
        - Return a minimal existence result suitable for outbound validation consumers.

        Non-responsibilities:
        - This function does not return complete movie metadata.
        - This function does not create implicit movie records.
        """
        ...

    async def get_movie_summary(self, query: object) -> object:
        """
        Returns a lightweight movie summary view for composition-oriented read scenarios.

        The summary view may be consumed by frontend composition layers or internal
        service orchestration that requires display-safe movie labels without owning
        movie-domain data.

        Responsibilities:
        - Resolve a compact movie representation.
        - Avoid overfetching fields not needed by summary-level clients.

        Non-responsibilities:
        - This function does not provide search ranking.
        - This function does not attach rating or review summaries.
        """
        ...

    async def find_movie_by_external_id(self, query: object) -> object:
        """
        Finds whether an external provider identifier is already mapped to a movie record.

        This query supports safe ingestion workflows and prevents duplicate internal
        creation when the same external movie asset is imported multiple times.

        Responsibilities:
        - Resolve an internal movie match by external provider identity.
        - Return a duplication or matching result contract.

        Non-responsibilities:
        - This function does not perform the import itself.
        - This function does not modify external provider data.
        """
        ...
```

---

# 8. Collection Service Code Skeleton

---

## 8.1 `collection-service/app/application/collection_command_service.py`

```python
class CollectionCommandService:
    async def create_collection(self, command: object) -> object:
        """
        Creates a new user-owned movie collection.

        A collection is an organizational structure owned by the Collection Service and
        represents a user-defined grouping of movie identifiers rather than movie metadata
        itself.

        Responsibilities:
        - Validate collection creation input.
        - Persist a collection record under the authenticated owner.
        - Prepare a CollectionCreated event if required by downstream consumers.

        Non-responsibilities:
        - This function does not copy movie metadata into the collection domain.
        - This function does not manage reviews or ratings.
        """
        ...

    async def update_collection(self, command: object) -> object:
        """
        Updates mutable collection attributes such as title, description, or visibility.

        The function must enforce ownership because collections are user-owned resources
        and mutation rights belong to the collection owner.

        Responsibilities:
        - Verify collection existence.
        - Confirm collection ownership.
        - Apply permitted field updates.
        - Prepare a CollectionUpdated event.

        Non-responsibilities:
        - This function does not add or remove collection items.
        - This function does not alter underlying movie catalog data.
        """
        ...

    async def delete_collection(self, command: object) -> object:
        """
        Removes or deactivates an entire user-owned collection.

        The deletion affects only the Collection Service-owned aggregate and should not
        modify movie records, reviews, or ratings in other bounded contexts.

        Responsibilities:
        - Verify collection ownership.
        - Perform the approved collection removal operation.
        - Prepare a CollectionDeleted event.

        Non-responsibilities:
        - This function does not delete movies.
        - This function does not delete user accounts.
        """
        ...

    async def add_movie_to_collection(self, command: object) -> object:
        """
        Adds a referenced movie identifier to a user-owned collection.

        The function depends on a movie existence validation port rather than direct access
        to Movie Service storage, preserving microservice isolation while preventing
        invalid foreign references.

        Responsibilities:
        - Verify collection ownership.
        - Confirm that the target movie exists through the outbound validation port.
        - Enforce duplicate membership rules.
        - Persist the new collection item.
        - Prepare a CollectionItemAdded event.

        Non-responsibilities:
        - This function does not duplicate movie title or genre ownership.
        - This function does not create missing movies.
        """
        ...

    async def remove_movie_from_collection(self, command: object) -> object:
        """
        Removes an existing movie reference from a user-owned collection.

        The function changes only collection membership state and does not modify the
        referenced movie's lifecycle or availability in the Movie Service.

        Responsibilities:
        - Verify collection ownership.
        - Confirm that the collection item exists.
        - Remove the membership relationship.
        - Prepare a CollectionItemRemoved event.

        Non-responsibilities:
        - This function does not delete the movie record itself.
        - This function does not manipulate search index data.
        """
        ...
```

---

## 8.2 `collection-service/app/application/collection_query_service.py`

```python
class CollectionQueryService:
    async def get_my_collections(self, query: object) -> object:
        """
        Returns the collection list owned by the authenticated user.

        The result should remain focused on collection-domain information such as collection
        identity, title, description, visibility, and item counts when those counts are
        maintained locally.

        Responsibilities:
        - Filter collections by authenticated owner identity.
        - Return a paginated or ordered collection list view.

        Non-responsibilities:
        - This function does not automatically expand every collection item into movie detail.
        - This function does not fetch review summaries.
        """
        ...

    async def get_collection_detail(self, query: object) -> object:
        """
        Retrieves detailed collection-domain information for a single collection resource.

        The function should enforce either ownership or approved visibility rules before
        exposing collection contents.

        Responsibilities:
        - Resolve the target collection aggregate.
        - Validate access rights according to visibility rules.
        - Return a collection detail view.

        Non-responsibilities:
        - This function does not become a movie metadata source of truth.
        - This function does not modify collection state.
        """
        ...

    async def get_collection_items(self, query: object) -> object:
        """
        Retrieves the movie identifier membership list contained within a collection.

        This query returns Collection Service-owned membership data. Any expansion into
        full movie card content should occur through composition outside the collection
        domain boundary.

        Responsibilities:
        - Resolve collection access rights.
        - Return ordered or paginated collection item references.

        Non-responsibilities:
        - This function does not read Movie Service tables directly.
        - This function does not compute user review details.
        """
        ...

    async def check_collection_ownership(self, query: object) -> object:
        """
        Validates whether a user identity owns a specific collection resource.

        This query supports command-side permission checks and keeps ownership logic
        centralized within the Collection Service.

        Responsibilities:
        - Resolve the collection owner by collection identifier.
        - Compare ownership against the requesting user.

        Non-responsibilities:
        - This function does not modify collection records.
        - This function does not grant cross-service roles.
        """
        ...

    async def check_movie_already_added(self, query: object) -> object:
        """
        Determines whether a movie reference already exists inside a target collection.

        This validation prevents duplicate collection membership and protects collection
        consistency without consulting external services.

        Responsibilities:
        - Resolve the collection membership relation.
        - Return a duplication decision result.

        Non-responsibilities:
        - This function does not verify whether the movie still exists globally.
        - This function does not add the movie.
        """
        ...
```

---

## 8.3 `collection-service/app/ports/movie_validation_port.py`

```python
from typing import Protocol


class MovieValidationPort(Protocol):
    async def verify_movie_exists_for_collection(self, query: object) -> object:
        """
        Validates that a movie identifier is acceptable for collection membership.

        This port abstracts the synchronous dependency from Collection Service to
        Movie Service. The Collection application layer depends only on this contract,
        not on a concrete HTTP client or remote transport implementation.

        Responsibilities:
        - Submit a movie existence query to the external movie boundary.
        - Return a normalized existence decision usable by collection command logic.

        Non-responsibilities:
        - This port does not load full movie detail views.
        - This port does not persist collection data.
        """
        ...
```

---

# 9. Review Service Code Skeleton

---

## 9.1 `review-service/app/application/review_command_service.py`

```python
class ReviewCommandService:
    async def create_review(self, command: object) -> object:
        """
        Creates a new user-authored review for an existing movie reference.

        The function preserves bounded-context separation by verifying only the movie's
        existence through an outbound validation port while keeping review content,
        moderation state, and author identity inside the Review Service domain.

        Responsibilities:
        - Confirm that the target movie exists.
        - Validate review content and applicable duplicate-review policies.
        - Persist the review record.
        - Prepare a ReviewCreated event.

        Non-responsibilities:
        - This function does not modify movie metadata.
        - This function does not update search documents directly.
        """
        ...

    async def update_review(self, command: object) -> object:
        """
        Updates the content of an existing review owned by the requesting user.

        Review ownership is a Review Service concern and must be validated before
        mutation. The operation should prepare downstream synchronization when the
        changed review affects indexed or displayed review state.

        Responsibilities:
        - Confirm review existence.
        - Confirm author ownership.
        - Validate revised content.
        - Persist the updated review.
        - Prepare a ReviewUpdated event.

        Non-responsibilities:
        - This function does not reassign review ownership.
        - This function does not edit movie catalog data.
        """
        ...

    async def delete_review(self, command: object) -> object:
        """
        Deletes or deactivates a user-owned review.

        The review lifecycle change remains local to Review Service while any search
        aggregate consequences should be handled asynchronously through emitted events.

        Responsibilities:
        - Confirm review existence.
        - Confirm author ownership.
        - Apply the deletion or inactive state change.
        - Prepare a ReviewDeleted event.

        Non-responsibilities:
        - This function does not delete user accounts.
        - This function does not directly update search index storage.
        """
        ...

    async def hide_review_by_admin(self, command: object) -> object:
        """
        Applies an administrative moderation state to a review.

        This command supports policy-based review visibility control without changing
        ownership or rewriting author-submitted content unless moderation rules explicitly
        define such behavior.

        Responsibilities:
        - Confirm administrative authorization.
        - Resolve the target review.
        - Apply moderation visibility state.
        - Prepare a ReviewModerated event when relevant.

        Non-responsibilities:
        - This function does not delete the movie.
        - This function does not alter user identity records.
        """
        ...
```

---

## 9.2 `review-service/app/application/rating_command_service.py`

```python
class RatingCommandService:
    async def create_or_update_rating(self, command: object) -> object:
        """
        Creates the user's first rating for a movie or updates the existing rating value.

        The Review Service owns rating records and related rating mutation semantics.
        Movie validity is confirmed through an outbound validation boundary before the
        rating change is accepted.

        Responsibilities:
        - Verify the referenced movie exists.
        - Validate the rating score against permitted domain ranges.
        - Insert or update the user's rating record.
        - Prepare a RatingUpdated event.

        Non-responsibilities:
        - This function does not maintain movie metadata.
        - This function does not directly compute search index state.
        """
        ...

    async def delete_rating(self, command: object) -> object:
        """
        Removes the authenticated user's rating for a specific movie.

        The mutation affects Review Service-owned rating state and should notify
        asynchronous consumers so rating summaries can be recalculated downstream.

        Responsibilities:
        - Confirm that the user's rating exists.
        - Remove or deactivate the rating record.
        - Prepare a RatingDeleted event.

        Non-responsibilities:
        - This function does not delete the movie itself.
        - This function does not remove the user's reviews.
        """
        ...
```

---

## 9.3 `review-service/app/application/review_query_service.py`

```python
class ReviewQueryService:
    async def get_reviews_by_movie(self, query: object) -> object:
        """
        Retrieves review list views associated with a specific movie identifier.

        The query returns Review Service-owned review information and may support
        sorting, moderation visibility filtering, and pagination.

        Responsibilities:
        - Filter reviews by movie identifier.
        - Exclude hidden or non-displayable reviews according to policy.
        - Return a stable review list view contract.

        Non-responsibilities:
        - This function does not fetch movie metadata.
        - This function does not modify review records.
        """
        ...

    async def get_my_reviews(self, query: object) -> object:
        """
        Retrieves the review records authored by the authenticated user.

        The result supports user-facing history and review management screens while
        preserving Review Service ownership over review content.

        Responsibilities:
        - Filter reviews by actor user identifier.
        - Return a user-owned review list view.

        Non-responsibilities:
        - This function does not resolve full movie details.
        - This function does not change moderation state.
        """
        ...

    async def get_review_detail(self, query: object) -> object:
        """
        Retrieves detailed information for one review entity.

        The detail view is intended to expose review-domain fields and should avoid
        returning persistence-specific internals.

        Responsibilities:
        - Resolve the review by review identifier.
        - Apply visibility or access checks as required.
        - Return a review detail view contract.

        Non-responsibilities:
        - This function does not update review content.
        - This function does not load user profile details from User Service.
        """
        ...

    async def get_rating_summary_by_movie(self, query: object) -> object:
        """
        Returns aggregate rating information for a movie.

        The summary may include average score, rating count, and other Review Service-owned
        rating metrics that are useful for display or downstream indexing workflows.

        Responsibilities:
        - Aggregate ratings by movie identifier.
        - Return a normalized rating summary result.

        Non-responsibilities:
        - This function does not write search documents.
        - This function does not mutate individual ratings.
        """
        ...

    async def get_my_rating_for_movie(self, query: object) -> object:
        """
        Retrieves the authenticated user's own rating value for a movie.

        This query supports personalized display logic such as showing whether the user
        has already rated a given movie.

        Responsibilities:
        - Resolve the rating by user identifier and movie identifier.
        - Return a user-specific rating view when present.

        Non-responsibilities:
        - This function does not alter the rating.
        - This function does not return review content.
        """
        ...

    async def check_review_ownership(self, query: object) -> object:
        """
        Validates whether a review belongs to the specified user identity.

        This function centralizes review ownership logic and supports command-side
        authorization decisions within the Review Service.

        Responsibilities:
        - Resolve the author identity of the review.
        - Compare it with the requesting user identifier.
        - Return an ownership decision result.

        Non-responsibilities:
        - This function does not mutate review state.
        - This function does not manage global authorization roles.
        """
        ...
```

---

## 9.4 `review-service/app/ports/movie_validation_port.py`

```python
from typing import Protocol


class MovieValidationPort(Protocol):
    async def verify_movie_exists_for_review(self, query: object) -> object:
        """
        Validates whether a referenced movie can be used as the target of a review or rating.

        The Review Service depends on this port instead of directly reading Movie Service
        storage or re-implementing movie catalog logic.

        Responsibilities:
        - Submit movie existence validation to the external movie boundary.
        - Return a result that review and rating command services can enforce.

        Non-responsibilities:
        - This port does not return full movie details.
        - This port does not persist review or rating records.
        """
        ...
```

---

# 10. Search Service Code Skeleton

---

## 10.1 `search-service/app/application/search_query_service.py`

```python
class SearchQueryService:
    async def search_movies_by_keyword(self, query: object) -> object:
        """
        Executes keyword-oriented movie search against the Search Service-owned index.

        The query is intended for title-centric or general text matching and returns
        search result views rather than authoritative movie-domain entities.

        Responsibilities:
        - Interpret keyword search input.
        - Query the search index with supported ranking behavior.
        - Return normalized movie search result views.

        Non-responsibilities:
        - This function does not update the movie catalog.
        - This function does not read Movie Service tables directly.
        """
        ...

    async def filter_movies_by_genre(self, query: object) -> object:
        """
        Returns indexed movie search results filtered by genre criteria.

        Responsibilities:
        - Validate genre filtering input.
        - Execute the genre-oriented index lookup.
        - Return a search result view.

        Non-responsibilities:
        - This function does not create or edit genre definitions.
        - This function does not own canonical movie metadata.
        """
        ...

    async def filter_movies_by_actor(self, query: object) -> object:
        """
        Returns indexed movie search results filtered by actor name or actor key.

        Responsibilities:
        - Normalize actor-oriented search input.
        - Query the Search Service index.
        - Return ranked or paginated movie search results.

        Non-responsibilities:
        - This function does not edit actor relationships in Movie Service.
        """
        ...

    async def filter_movies_by_director(self, query: object) -> object:
        """
        Returns indexed movie search results filtered by director criteria.

        Responsibilities:
        - Validate director filter input.
        - Execute the corresponding search index request.
        - Return a normalized result view.

        Non-responsibilities:
        - This function does not own director reference data.
        """
        ...

    async def filter_movies_by_release_year(self, query: object) -> object:
        """
        Returns indexed movie search results filtered by release year constraints.

        Responsibilities:
        - Interpret supported year filter operators or exact-year values.
        - Query the search index.
        - Return paginated search result views.

        Non-responsibilities:
        - This function does not modify movie release metadata.
        """
        ...

    async def filter_movies_by_minimum_rating(self, query: object) -> object:
        """
        Returns indexed movie search results filtered by a minimum average rating threshold.

        Responsibilities:
        - Validate the numeric rating threshold.
        - Execute rating-aware index filtering.
        - Return consistent search results.

        Non-responsibilities:
        - This function does not recalculate rating aggregates.
        - This function does not read Review Service storage directly.
        """
        ...

    async def search_movies_with_composite_filter(self, query: object) -> object:
        """
        Executes a multi-condition search that combines keyword, catalog, and rating filters.

        This is the most expressive user-facing search contract and should rely solely
        on the Search Service-owned read model built from upstream events.

        Responsibilities:
        - Normalize composite filter input.
        - Translate filter combinations into the search index query model.
        - Return stable, paginated search result views.

        Non-responsibilities:
        - This function does not orchestrate runtime joins across remote services.
        """
        ...

    async def get_search_index_health(self, query: object) -> object:
        """
        Returns operational status information about the search index.

        This function supports monitoring and internal diagnostics by reporting whether
        the Search Service's backing index is reachable and usable.

        Responsibilities:
        - Evaluate search backend connectivity or readiness.
        - Return a health-oriented view result.

        Non-responsibilities:
        - This function does not rebuild indexes.
        - This function does not serve business search results.
        """
        ...
```

---

## 10.2 `search-service/app/application/search_index_event_handler.py`

```python
class SearchIndexEventHandler:
    async def handle_movie_created(self, event: object) -> object:
        """
        Creates a new search document when a movie creation event is consumed.

        The handler transforms the upstream movie event payload into the Search Service's
        internal read model without depending on direct database access to Movie Service.

        Responsibilities:
        - Parse the MovieCreated event payload.
        - Construct the initial movie search document.
        - Persist the document into the search index.

        Non-responsibilities:
        - This handler does not create the movie itself.
        - This handler does not validate administrative catalog permissions.
        """
        ...

    async def handle_movie_updated(self, event: object) -> object:
        """
        Updates an existing search document after movie metadata changes upstream.

        Responsibilities:
        - Parse the MovieUpdated event payload.
        - Resolve the associated search document.
        - Apply indexable movie metadata changes.

        Non-responsibilities:
        - This handler does not write to Movie Service storage.
        """
        ...

    async def handle_movie_deleted(self, event: object) -> object:
        """
        Removes or hides a movie search document after upstream movie deactivation.

        Responsibilities:
        - Parse the MovieDeleted event payload.
        - Apply the Search Service-side removal or inactive representation.
        - Preserve search consistency with catalog lifecycle.

        Non-responsibilities:
        - This handler does not decide the business deletion policy of Movie Service.
        """
        ...

    async def handle_review_created(self, event: object) -> object:
        """
        Applies review-driven indexed data changes after a new review is created.

        Depending on the agreed indexing model, this handler may update review counts or
        other display-level counters derived from Review Service events.

        Responsibilities:
        - Interpret the review creation payload.
        - Update review-related indexed summary values.

        Non-responsibilities:
        - This handler does not store the review body as authoritative content.
        - This handler does not moderate review text.
        """
        ...

    async def handle_review_deleted(self, event: object) -> object:
        """
        Applies index updates after a review is removed or hidden from effective counts.

        Responsibilities:
        - Interpret the review deletion payload.
        - Adjust review-related index values as specified by the read model.

        Non-responsibilities:
        - This handler does not restore or delete Review Service records.
        """
        ...

    async def handle_rating_updated(self, event: object) -> object:
        """
        Updates indexed rating summary fields when upstream rating information changes.

        Responsibilities:
        - Parse the rating update payload.
        - Refresh average rating or rating count fields represented in the search index.
        - Keep Search Service read models aligned with Review Service events.

        Non-responsibilities:
        - This handler does not own the rating calculation source of truth.
        """
        ...

    async def rebuild_movie_search_document(self, command: object) -> object:
        """
        Reconstructs a complete search document for a specific movie identifier.

        This operation is intended for maintenance, repair, or controlled reindexing flows
        when event-based incremental updates require recovery or reconciliation.

        Responsibilities:
        - Resolve the target search document reconstruction request.
        - Rebuild the read model using approved upstream data access strategy.
        - Persist the refreshed search document.

        Non-responsibilities:
        - This function does not redefine catalog ownership.
        - This function does not bypass service boundaries for arbitrary joins.
        """
        ...
```

---

# 11. Notification Service Code Skeleton

---

## 11.1 `notification-service/app/application/notification_event_handler.py`

```python
class NotificationEventHandler:
    async def handle_user_created_notification(self, event: object) -> object:
        """
        Creates a user-facing welcome notification in response to account creation.

        This handler reacts to lifecycle events without participating in the user creation
        transaction itself, preserving asynchronous decoupling.

        Responsibilities:
        - Interpret the UserCreated event payload.
        - Build a notification dispatch command or log entry.
        - Delegate channel delivery to the notification dispatch boundary.

        Non-responsibilities:
        - This handler does not create the user profile.
        - This handler does not create authentication credentials.
        """
        ...

    async def handle_collection_created_notification(self, event: object) -> object:
        """
        Processes collection creation notifications when such user messaging is enabled.

        Responsibilities:
        - Interpret the CollectionCreated event.
        - Prepare a notification record according to product policy.

        Non-responsibilities:
        - This handler does not create collections.
        - This handler does not modify collection visibility rules.
        """
        ...

    async def handle_collection_item_added_notification(self, event: object) -> object:
        """
        Processes collection membership update notifications.

        Responsibilities:
        - Interpret the CollectionItemAdded event payload.
        - Prepare any notification or audit-oriented communication required by policy.

        Non-responsibilities:
        - This handler does not add items to the collection itself.
        """
        ...

    async def handle_review_created_notification(self, event: object) -> object:
        """
        Processes notification behavior triggered by new review creation.

        Responsibilities:
        - Interpret the ReviewCreated event payload.
        - Determine whether product policy requires a notification.
        - Delegate the notification to the command layer or dispatch port.

        Non-responsibilities:
        - This handler does not moderate or persist authoritative review content.
        """
        ...

    async def handle_rating_updated_notification(self, event: object) -> object:
        """
        Processes optional user-facing notification behavior after rating changes.

        Responsibilities:
        - Interpret the RatingUpdated event.
        - Prepare any notification workflow that the product rules permit.

        Non-responsibilities:
        - This handler does not compute rating averages.
        - This handler does not update search indexes.
        """
        ...
```

---

## 11.2 `notification-service/app/application/notification_command_service.py`

```python
class NotificationCommandService:
    async def dispatch_notification(self, command: object) -> object:
        """
        Dispatches a prepared notification through one or more configured channels.

        Channel-specific execution such as email, push, or in-app delivery should remain
        behind outbound dispatch ports so the application layer stays transport-neutral.

        Responsibilities:
        - Validate notification dispatch input.
        - Resolve channel delivery strategy.
        - Delegate delivery to infrastructure-specific dispatch adapters.

        Non-responsibilities:
        - This function does not decide whether upstream business events should exist.
        - This function does not own user account or review state.
        """
        ...

    async def save_notification_log(self, command: object) -> object:
        """
        Persists the notification execution record for audit and display purposes.

        Responsibilities:
        - Store a normalized notification log record.
        - Preserve delivery status and reference metadata.

        Non-responsibilities:
        - This function does not execute channel delivery by itself.
        - This function does not load unrelated domain data.
        """
        ...

    async def mark_notification_as_read(self, command: object) -> object:
        """
        Marks a user-visible notification record as read.

        Responsibilities:
        - Validate notification ownership.
        - Transition the read-status marker.
        - Return a result suitable for frontend state refresh.

        Non-responsibilities:
        - This function does not resend notifications.
        - This function does not delete the event that originally triggered delivery.
        """
        ...
```

---

## 11.3 `notification-service/app/application/notification_query_service.py`

```python
class NotificationQueryService:
    async def get_my_notifications(self, query: object) -> object:
        """
        Retrieves the notification records that belong to the authenticated user.

        Responsibilities:
        - Filter notification logs by user identity.
        - Apply pagination and read/unread criteria when supported.
        - Return a notification list view.

        Non-responsibilities:
        - This function does not trigger new notifications.
        - This function does not fetch domain details from unrelated services.
        """
        ...
```

---

# 12. REST API Route Skeleton

아래는 **각 서비스 Controller가 Application Service와 1:1로 연결되는 방식**을 보여주는 Skeleton입니다.
Function Signature 설계서의 “Controller와 Application Function 1:1 대응” 원칙을 반영합니다. 

---

## 12.1 `movie-service/app/api/movie_routes.py`

```python
from fastapi import APIRouter


router = APIRouter(prefix="/movies", tags=["Movies"])


@router.get("")
async def get_movie_list_route(query: object) -> object:
    """
    Accepts a public movie list query request and delegates the operation to
    MovieQueryService.get_movie_list.

    This route should contain transport-level responsibilities only, such as:
    - Parsing HTTP query parameters.
    - Converting them into the application query contract.
    - Returning the application result through the API response standard.

    No movie-domain decision logic should be implemented inside this route.
    """
    ...


@router.get("/{movie_id}")
async def get_movie_detail_route(movie_id: str) -> object:
    """
    Accepts a public movie detail request and delegates the operation to
    MovieQueryService.get_movie_detail.

    The route is responsible only for adapting the HTTP path parameter into
    an application-level query object.
    """
    ...
```

---

## 12.2 `collection-service/app/api/collection_routes.py`

```python
from fastapi import APIRouter


router = APIRouter(prefix="/collections", tags=["Collections"])


@router.post("")
async def create_collection_route(command: object) -> object:
    """
    Accepts a protected collection creation request and delegates the operation to
    CollectionCommandService.create_collection.

    This route should not create domain entities directly and should not interact
    with persistence adapters.
    """
    ...


@router.post("/{collection_id}/items")
async def add_movie_to_collection_route(collection_id: str, command: object) -> object:
    """
    Accepts a protected request to add a movie reference to a collection and delegates
    the operation to CollectionCommandService.add_movie_to_collection.

    Resource ownership and movie validity must be enforced in the application layer,
    not in this route function.
    """
    ...
```

---

## 12.3 `review-service/app/api/review_routes.py`

```python
from fastapi import APIRouter


router = APIRouter(prefix="/reviews", tags=["Reviews"])


@router.post("")
async def create_review_route(command: object) -> object:
    """
    Accepts a protected review creation request and delegates the operation to
    ReviewCommandService.create_review.

    This route handles request adaptation only and must not implement review policies,
    movie existence validation, or event publishing logic.
    """
    ...


@router.get("")
async def get_reviews_by_movie_route(query: object) -> object:
    """
    Accepts a public movie review list request and delegates the operation to
    ReviewQueryService.get_reviews_by_movie.

    The route should preserve query-level transport mapping while leaving pagination,
    visibility rules, and filtering semantics to the application layer.
    """
    ...
```

---

## 12.4 `search-service/app/api/search_routes.py`

```python
from fastapi import APIRouter


router = APIRouter(prefix="/search", tags=["Search"])


@router.get("/movies")
async def search_movies_route(query: object) -> object:
    """
    Accepts a public search request and delegates the operation to the appropriate
    SearchQueryService function.

    The route is responsible for mapping HTTP query parameters into a structured
    composite search query contract.
    """
    ...
```

---

# 13. Outbound Port Skeleton

---

## 13.1 `api-gateway/app/ports/auth_verification_port.py`

```python
from typing import Protocol


class AuthVerificationPort(Protocol):
    async def verify_access_token_for_gateway(self, query: object) -> object:
        """
        Delegates access-token verification to the Auth Service.

        The API Gateway depends on this abstraction so that the transport mechanism
        used to contact Auth Service remains replaceable and testable.
        """
        ...

    async def resolve_user_role_for_gateway(self, query: object) -> object:
        """
        Retrieves the effective authorization role required for route-level policy checks.

        This abstraction prevents direct coupling between gateway authorization decisions
        and the Auth Service transport implementation.
        """
        ...
```

---

## 13.2 `movie-service/app/ports/external_movie_provider_port.py`

```python
from typing import Protocol


class ExternalMovieProviderPort(Protocol):
    async def fetch_movie_metadata(self, query: object) -> object:
        """
        Retrieves raw or normalized movie metadata from an external movie information source.

        The Movie Service import workflow depends on this abstraction rather than coupling
        directly to a vendor-specific API client.
        """
        ...
```

---

## 13.3 `notification-service/app/ports/notification_dispatch_port.py`

```python
from typing import Protocol


class NotificationDispatchPort(Protocol):
    async def send_notification(self, command: object) -> object:
        """
        Sends a prepared notification through a channel-specific infrastructure adapter.

        The concrete implementation may represent email, push notification, or another
        delivery mechanism, but the application layer should depend only on this contract.
        """
        ...
```

---

# 14. Event Publisher Skeleton

---

## 14.1 `movie-service/app/events/movie_events.py`

```python
class MovieEventPublisher:
    async def publish_movie_created_event(self, payload: object) -> None:
        """
        Publishes the MovieCreated event after successful movie creation.

        The event enables downstream services such as Search Service to construct or update
        their own read models without requiring direct access to Movie Service storage.
        """
        ...

    async def publish_movie_updated_event(self, payload: object) -> None:
        """
        Publishes the MovieUpdated event after authoritative movie metadata changes.

        The event is expected to trigger asynchronous synchronization in services that
        maintain derived movie views or indexes.
        """
        ...

    async def publish_movie_deleted_event(self, payload: object) -> None:
        """
        Publishes the MovieDeleted event after movie deactivation or deletion semantics
        are applied in Movie Service.

        Consumers should independently decide how to represent the movie's inactive state.
        """
        ...
```

---

## 14.2 `review-service/app/events/review_events.py`

```python
class ReviewEventPublisher:
    async def publish_review_created_event(self, payload: object) -> None:
        """
        Publishes the ReviewCreated event after successful review persistence.

        Downstream services may use this event to update indexed review counts,
        trigger notifications, or execute other decoupled read-model updates.
        """
        ...

    async def publish_review_deleted_event(self, payload: object) -> None:
        """
        Publishes the ReviewDeleted event after review lifecycle removal.

        The event allows derived aggregates to be reconciled without direct Review Service
        database access.
        """
        ...

    async def publish_rating_updated_event(self, payload: object) -> None:
        """
        Publishes the RatingUpdated event when rating information changes.

        Search Service may consume this event to refresh rating-related indexed fields.
        """
        ...
```

---

# 15. Service Main Application Skeleton

---

## 15.1 `movie-service/app/main.py`

```python
from fastapi import FastAPI


def create_app() -> FastAPI:
    """
    Creates and configures the Movie Service web application.

    This factory function should be responsible for:
    - Instantiating the FastAPI application object.
    - Registering API routers.
    - Registering exception handlers.
    - Registering startup/shutdown lifecycle hooks.
    - Exposing health-check endpoints if required.

    Business logic must not be implemented inside this function.
    """
    ...


app = create_app()
```

---

## 15.2 `collection-service/app/main.py`

```python
from fastapi import FastAPI


def create_app() -> FastAPI:
    """
    Creates and configures the Collection Service web application.

    This factory should configure HTTP routes, dependency injection wiring, and
    observability hooks while preserving a clean separation from collection-domain
    application services.
    """
    ...


app = create_app()
```

---

# 16. Code Skeleton 작성 시 유지해야 할 원칙

```text
1. 함수 본문에는 비즈니스 로직을 넣지 않는다.
2. Controller는 HTTP 변환만 담당한다.
3. Application Service가 실제 Use Case의 중심이 된다.
4. Domain Entity는 서비스 내부에서만 사용한다.
5. 외부 서비스 접근은 Port Interface로 감싼다.
6. 이벤트 발행은 상태 변경 Use Case와 연결하되 Infrastructure에 직접 종속되지 않는다.
7. Search Service는 Query + Event Handler 중심으로 구성한다.
8. Notification Service는 Event Consumer 중심으로 구성한다.
9. Collection과 Review는 Movie 존재 검증만 동기 호출한다.
10. 모든 함수는 향후 Test Agent가 테스트 케이스를 설계할 수 있을 정도로 Description을 명확히 가진다.
```

---

# 17. 최종 요약

이번 Code Skeleton은 이전 문서에서 정의한 설계를 실제 구현 구조로 옮기기 위한 **중간 개발 산출물**입니다.

```text
MSA Design Spec
        ↓
Function Signature
        ↓
Code Skeleton
        ↓
Concrete Implementation
        ↓
Test Cases / Integration Tests
```

이번 단계에서 확정된 주요 구조는 다음과 같습니다.

| 계층                  | 역할                            |
| ------------------- | ----------------------------- |
| API Route           | HTTP 요청/응답 어댑터                |
| Application Service | Use Case 실행                   |
| Domain              | 서비스 고유 엔티티                    |
| Port                | 외부 의존성 추상화                    |
| Event Publisher     | 상태 변경 이벤트 발행                  |
| Event Handler       | 외부 이벤트 수신 반영                  |
| Shared Contracts    | 공통 Context / Event / Error 계약 |

특히 **Collection Service → MovieValidationPort**, **Review Service → MovieValidationPort**, **Gateway → AuthVerificationPort**와 같은 Port 기반 설계는 기존 Function Signature 설계서의 **의존성 최소화 원칙**을 코드 구조에 직접 반영한 핵심 요소입니다. 
