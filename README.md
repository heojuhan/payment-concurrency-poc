# Payment & Inventory Concurrency Control PoC

대용량 트래픽 환경에서 결제·재고 데이터 정합성을 보장하고, 팀 단위의 일관된 AI 개발 워크플로를 적용하기 위한 백엔드 아키텍처 검증 레포지토리입니다.

## Core Architecture
- **Stack:** Java 17, Spring Boot 3.x, Spring Data JPA, QueryDSL, Spring Batch, MySQL 8.0, Redis(Redisson), Apache Kafka
- **Concurrency Control:** DB 비관적 락 경합(V1) vs Redis(Redisson) Pub/Sub 분산 락 + Kafka Transactional Outbox 패턴(V2) 비교
- **Verification:** JUnit 5 멀티스레드 통합 테스트(Testcontainers) 및 `k6` 부하 테스트

## AI Engineering Workflow
- `CLAUDE.md`: 프로젝트 공통 아키텍처 제약 및 AI 위임/엔지니어 검증 경계 규칙
- `.claude/skills/sp-to-api/SKILL.md`: 레거시 DB 저장 프로시저(SP) 역분석 및 Spring API 전환 설계 스킬
- `.claude/skills/concurrency-test/SKILL.md`: 분산 락·배치 로직 JUnit 5 동시성 테스트 생성 스킬
- `.claude/skills/querydsl-explain/SKILL.md`: 레거시 SQL → QueryDSL 변환 및 실행 계획(`EXPLAIN`) 감사 스킬
- `docs/ai-verification-log.md`: AI 생성 코드의 결함(트랜잭션-락 역전, 인덱스 오추천, 이벤트 유실) 직접 교정 Diff 기록

## Roadmap
- [x] Step 1: Repository 구조, `CLAUDE.md` 가드레일 및 `.claude/skills` 3종 정의
- [ ] Step 2: Docker Compose 인프라 및 V1/V2 결제·재고 PoC + JUnit 5 동시성 테스트 구현
- [ ] Step 3: `k6` 부하 테스트 리포트 및 `docs/ai-verification-log.md` 작성