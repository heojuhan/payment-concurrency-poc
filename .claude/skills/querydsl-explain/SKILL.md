---
name: querydsl-explain
description: 레거시 네이티브 SQL(MySQL/MSSQL)을 QueryDSL 코드로 변환하고, Zero-Offset 페이징 및 커버링 인덱스 관점의 실행 계획 검증 체크리스트를 출력한다.
---

# SQL to QueryDSL Converter & Execution Plan Auditor

## Execution Steps
1. **SQL 구조 분석:** JOIN 순서, WHERE 조건절 컬럼, ORDER BY, GROUP BY, 서브쿼리, 함수 래핑(인덱스 무효화 요인)을 분해한다.
2. **QueryDSL 변환:**
   - `JPAQueryFactory` 기반 Custom Repository 메서드로 변환
   - 동적 조건은 `BooleanExpression` 메서드로 분리해 `null` 안전성 보장
   - DTO 직접 조회 시 `Projections.constructor` 사용 (불필요한 엔티티 영속성 컨텍스트 적재 방지)
   - 대량 조회/배치 쿼리는 `OFFSET` 제거 후 마지막 PK/정렬 키 기준 Zero-Offset(`lt`/`gt`) 조건 적용
3. **인덱스 및 실행 계획(`EXPLAIN`) 감사 리포트 작성:**
   - 인덱스 무효화 패턴 탐지: 컬럼 함수 적용(`DATE(created_at)`), 암묵적 형변환, 앞쪽 와일드카드(`LIKE '%abc'`), `OR` 조건
   - 카디널리티(Cardinality) 순서 기준 최적 복합 인덱스 DDL 제안
   - 커버링 인덱스 적용 가능 여부 및 `Using filesort` / `Using temporary` 제거 방안 제시

## Strict Guardrails
- 실제 운영 테이블의 데이터 분포(카디널리티) 없이 성능 개선 수치를 단정하지 않는다.
- 실행 계획 검증용 `EXPLAIN ANALYZE` SQL을 반드시 함께 출력해 엔지니어가 로컬/스테이징 DB에서 실측하도록 유도한다.