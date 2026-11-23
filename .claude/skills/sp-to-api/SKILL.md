---
name: sp-to-api
description: MSSQL/MySQL 레거시 저장 프로시저(SP)를 분석해 도메인 분기표, 트랜잭션 위험 구간, Spring Boot REST API 전환 설계 초안을 생성한다.
---

# Legacy Stored Procedure to Spring API Analyzer

## Execution Steps
1. **입출력 스펙 추출:** 입력 파라미터, 출력 파라미터, 리턴 코드, 호출하는 하위 SP/함수 목록을 표로 정리한다.
2. **비즈니스 분기 및 상태 전이 분해:** `IF/ELSE`, `CASE`, `CURSOR`, `WHILE` 루프를 번호 매긴 흐름으로 분해하고, 암묵적 예외 조건(NULL 처리, 음수 재고, 중복 키 등)을 명시한다.
3. **데이터 정합성 및 위험 구간 마킹:**
   - `BEGIN TRAN` / `COMMIT` / `ROLLBACK` 범위와 중첩 트랜잭션 여부 식별
   - `WITH (UPDLOCK)`, `HOLDLOCK`, `NOLOCK` 등 락 힌트 및 데드락 위험 테이블 표시
   - 외부 연동(알림, 포인트, 정산)과 결합된 구간을 Kafka 비동기 분리 후보로 마킹
4. **Spring Boot 전환 설계안 생성:**
   - Controller / Facade(분산 락) / Service(트랜잭션) / Repository 계층 분리안 제시
   - 요청/응답 DTO 레코드(Java 17 `record`) 초안 작성
   - Strangler Fig 점진 전환을 위한 기존 SP 병행 검증(Shadow/Dual-Read) 체크포인트 작성

## Strict Guardrails
- 원본 SP에 존재하지 않는 비즈니스 규칙을 추측해서 임의로 추가하지 않는다.
- 해석이 모호한 매직 넘버(상태 코드, 플래그 값)는 반드시 `[CONFIRM REQUIRED]` 태그로 표시한다.

## Output Template
1. Summary & Domain Boundary
2. Parameter & Dependency Table
3. Step-by-Step Business Logic & Edge Cases
4. Concurrency / Transaction Risk Audit
5. Proposed Spring Boot API & Package Structure
6. `[CONFIRM REQUIRED]` Open Questions