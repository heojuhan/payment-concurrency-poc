---
name: concurrency-test
description: 결제·예약 분산 락 및 Spring Batch 청크 로직에 대해 JUnit 5 기반 멀티스레드 동시성 및 엣지 케이스 테스트 코드를 생성한다.
---

# Concurrency & Batch JUnit 5 Test Generator

## Execution Steps
1. **대상 코드 분석:** 타겟 서비스/배치 클래스의 공유 자원(재고, 결제 상태, 멱등성 키, 청크 경계)과 예외 클래스를 파악한다.
2. **필수 테스트 시나리오 매트릭스 구성:**
   - **정상 케이스:** 단일 요청 성공 및 상태 전이·이벤트(Outbox) 저장 검증
   - **동시성 경합 케이스:** `ExecutorService` + `CountDownLatch`로 최소 100개 동시 요청 발사
     - 한정 수량 재고(예: 10개)에 100건 요청 시 정확히 10건 성공, 90건 실패, 최종 재고 `0` 검증
     - 동일 주문/결제 키 중복 요청(Double-Click) 시 단 1건만 결제 승인 검증
   - **장애·롤백 케이스:** 락 획득 타임아웃(`tryLock` 실패), 트랜잭션 내부 예외 발생 시 DB 롤백 및 락 정상 해제 검증
   - **배치 경계 케이스(배치 대상일 경우):** 청크 크기 배수 / 비배수 / 0건 / 중간 청크 실패 후 재시작 시 중복 집계 0건 검증
3. **테스트 코드 구현 규칙:**
   - Java 17, JUnit 5 (`@DisplayName` 한글 명시), AssertJ 사용
   - 동시성 테스트는 H2 인메모리 DB의 락 동작 왜곡을 피하기 위해 Testcontainers(MySQL + Redis) 기반 통합 테스트로 작성
   - 비동기/스레드 테스트에서 `Thread.sleep()` 하드코딩을 금지하고 `CountDownLatch` 또는 `Awaitility` 사용

## Strict Guardrails
- 테스트 통과를 위해 프로덕션 코드의 검증 로직이나 락 대기 시간을 임의로 우회·모킹하지 않는다.
- 모든 `assertThat`에는 실패 시 원인을 즉시 파악할 수 있도록 기대값과 실제 상태를 명확히 검증한다.