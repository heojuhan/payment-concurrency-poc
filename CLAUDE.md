# Project Engineering Rules & AI Guardrails

## 1. Core Tech Stack
- Language / Framework: Java 17, Spring Boot 3.x, Spring Data JPA, QueryDSL, Spring Batch
- Storage / Messaging: MySQL 8.0, MSSQL (Legacy), Redis (Redisson), Apache Kafka
- Test: JUnit 5, Testcontainers

## 2. Delegation Boundary (AI 위임 vs 엔지니어 직접 통제)
- **AI 위임 허용:** 레거시 저장 프로시저(SP) 분기 문서화, DTO/Entity 보일러플레이트, JUnit 5 테스트 초안, 레거시 SQL → QueryDSL 1차 변환
- **엔지니어 필수 검증 (자동 머지 금지):** 트랜잭션 경계(`@Transactional`), 분산 락 획득/해제 순서, Kafka Transactional Outbox 정합성, 운영 DB 인덱스 및 실행 계획(`EXPLAIN`)

## 3. Hard Coding Constraints (위반 시 코드 생성 거부)
1. **분산 락 & 트랜잭션 순서 강제:** Redisson 락 획득/해제는 반드시 `@Transactional` 바깥 레이어(Facade 또는 AOP)에서 수행한다. 트랜잭션 커밋 전 락이 해제되는 코드를 절대 작성하지 않는다.
2. **비동기 이벤트 발행 규칙:** 결제/예약 완료 후 알림·포인트·권한 부여 등 후속 로직은 결제 트랜잭션 내부에 동기 호출하지 않고 Outbox 테이블 저장 후 Kafka로 비동기 발행한다.
3. **배치 페이징 규칙:** 대용량 정산 조회 시 `OFFSET` 기반 페이징 사용을 금지하고 PK/인덱스 기반 Zero-Offset(No-Offset) 페이징과 `JdbcTemplate` Bulk Insert를 우선 적용한다.
4. **캐시 방어 규칙:** Redis Look-Aside 캐시 적용 시 고정 TTL 사용을 금지하고 기본 TTL에 랜덤 지터(Jitter)를 추가해 캐시 스탬피드를 방지한다.
5. **JPA 금지 패턴:** 양방향 연관관계 무분별 추가, `FetchType.EAGER`, 루프 내부 단건 쿼리(N+1) 생성을 금지한다.

## 4. Output Format Requirement
- 모든 코드 제안 마지막에 아래 3줄의 자가 검증 항목을 반드시 출력한다.
  - `[Transaction]`: 락 범위와 트랜잭션 커밋 시점 충돌 여부
  - `[Concurrency]`: 동시 중복 요청(Double-Click) 및 멱등성 보장 여부
  - `[Query]`: Full Table Scan 또는 N+1 유발 가능성 여부