# ClickHouse Study & Operations To-Do

ClickHouse 유지보수 및 고객 지원을 위한 학습·실습용 To-Do 리스트입니다.
환경은 baremetal/VM 기준입니다.

---

## 1단계. 단일 노드 설치·기본기

**목표:** 한 대에 제대로 꺔고, 디렉토리·로그·기본 SQL까지 손에 익히기.

### OS/VM 준비
- [ ] ClickHouse 전용 VM 1대 준비 (고정 IP, hostname, 시간 동기화).
- [ ] OS 패키지 업데이트 및 기본 도구 설치.

### ClickHouse 설치
- [ ] ClickHouse 공식 문서에서 설치 방법 선택 (rpm/deb 패키지 또는 quick install 스크립트).
- [ ] `clickhouse-server`, `clickhouse-client` 설치.
- [ ] `clickhouse-server` 서비스 기동 및 `systemctl status`로 정상 기동 확인.
- [ ] 방화벽에서 9000(TCP), 8123(HTTP) 포트 확인/허용.

### 디렉토리·파일 구조 파악
- [ ] `/etc/clickhouse-server/` 구조 확인 (`config.xml`, `users.xml` 등).
- [ ] `/var/lib/clickhouse/` 데이터 디렉토리 구조 확인.
- [ ] `/var/log/clickhouse-server/` 로그 파일 위치 확인.

### 기본 SQL 및 접속
- [ ] `clickhouse-client`로 접속 후 `SELECT 1;` 실행.
- [ ] `SHOW DATABASES;`, `CREATE DATABASE testdb;`, `USE testdb;` 실행.

### 테스트 테이블·쿼리
- [ ] `testdb`에 간단한 MergeTree 테이블 생성.
  - 예: `user_actions(user_id UInt64, action String, action_time DateTime) ENGINE = MergeTree() ORDER BY (user_id, action_time);`
- [ ] `INSERT` 10~100건 넣고 `SELECT`, `WHERE`, `GROUP BY`, `ORDER BY`, `LIMIT` 쿼리 실행.

### 기본 설정 훑어보기
- [ ] `config.xml`에서 `listen_host`, `tcp_port`, `http_port` 항목 확인.
- [ ] `users.xml`에서 기본 유저 `default` 설정 확인 (필요시 비밀번호·쿼리 제한 간단히 설정).

---

## 2단계. 아키텍처·내부 구조 이해

**목표:** "왜 이렇게 설계돼 있냀"를 이해해서 HA/튜닝·장애분석이 자연스럽게 이어지게 만들기.

### 공식 아키텍처 문서 읽기
- [ ] 공식 문서 **Architecture / 아키텍처 개요** 1회 정독.
- [ ] 아래 키워드별 메모 작성:
  - 콼럼 저장 구조, 파트(part)·세그먼트 구조.
  - 벡터화 실행 모델 개념.
  - 백그라운드 머지/컴팩션 동작.

### MergeTree 엔진 개념 잡기
- [ ] MergeTree 엔진 설명 부분 정독:
  - 파티션 키, 정렬 키, 샘플링 키, TTL 개념을 각각 요약.
- [ ] `SHOW CREATE TABLE`로 테스트 테이블 DDL 확인.

### system.* 메타 테이블 탐색
- [ ] `system.databases`, `system.tables` 조회.
- [ ] `system.parts` 조회해서 파트 단위 저장 구조 확인.
- [ ] `system.merges` 조회해서 머지 작업 상태 확인.

---

## 3단계. 샤딩·복제·클러스터 개념 정리

**목표:** HA 구성할 때 개념부터 헷갈리지 않도록 정리.

### Shard / Replica / Cluster 개념
- [ ] Shard / Replica / Cluster 개념 정리:
  - Shard = 수평 분할 단위.
  - Replica = 고가용성/읽기 스케일용 복제본.
- [ ] `remote_servers`, `macros`, `clusters` 설정 구조 개념 메모.

### ReplicatedMergeTree 이해
- [ ] ReplicatedMergeTree 동작 방식 정리:
  - Keeper(ZooKeeper/ClickHouse Keeper)가 관리하는 메타데이터/로그 구조.
  - 파트 생성/복제 흐름.

### Distributed 테이블 이해
- [ ] Distributed 테이블 개념 정리:
  - 여러 샤드/리플리카를 묶어서 하나의 논리 테이블로 쿼리 라우팅.

### 수평 확장 문서 훑기
- [ ] 수평 확장 관련 공식 문서 정독 후 요약:
  - 샤딩 전략, 복제 전략의 기본 패턴 정리.

---

## 4단계. baremetal/VM 2~3노드 클러스터 실습

**목표:** 직접 샤드·리플리카 구성해보고 HA 동작을 확인.

### VM 추가 준비
- [ ] ClickHouse용 VM 추가 1~2대 준비 (총 2~3대).
- [ ] 모든 노드 hostname, IP, `/etc/hosts` 정리.

### 각 노드에 ClickHouse 설치
- [ ] 각 VM에 1단계와 같은 방식으로 ClickHouse 설치.
- [ ] 각 노드에서 `clickhouse-client`로 `SELECT 1;` 확인.

### 클러스터 설정 작성
- [ ] 각 노드 `config.xml`에 `remote_servers` 섹션 정의 (1 shard 2 replica 구성).
- [ ] `zookeeper` 또는 `keeper_server` 섹션 설정.
- [ ] 각 노드 `macros`에 `shard`, `replica` 값 설정.

### ReplicatedMergeTree / Distributed 테이블 실습
- [ ] 노드 A에서 ReplicatedMergeTree 테이블 생성.
- [ ] 노드 B(리플리카)에서 동일한 DDL로 생성.
- [ ] 한 노드에서 INSERT 후 다른 노드에서 SELECT로 복제 확인.
- [ ] Distributed 테이블 생성 후 분산 쿼리 동작 확인.

### 장애 테스트 (기초)
- [ ] 데이터 노드 1대 stop → Distributed 테이블로 조회/쓰기 시나리오 확인.
- [ ] 다시 start 후 복제 상태 정상화 확인 (`system.replicas`).

---

## 5단계. 구성·튜닝 포인트 익히기

**목표:** 설정 파일을 읽고 손댈 수 있는 수준까지.

### config.xml 주요 섹션
- [ ] `config.xml` 주요 섹션 읽고 정리:
  - 서버 포트, 경로, 로그 설정.
  - `max_connections`, `max_concurrent_queries`, 메모리 limit 관련 옵션.
- [ ] 위 설정 몇 가지 수정 후 재기동 → 적용 여부 확인.

### users.xml / RBAC 설정
- [ ] 기본 `default` 유저 확인 및 패스워드·쿼리 제한 추가 테스트.
- [ ] read-only 유저 1개 생성, 권한 차이 테스트.

### 테이블 설계 패턴 실습
- [ ] 서로 다른 파티션/정렬 키를 사용하는 MergeTree 테이블 2~3개 생성.
- [ ] 동일 데이터 넣고 쿼리 성능·파트 수·머지 패턴 비교.
- [ ] TTL 설정으로 오래된 데이터 자동 삭제/아카이브 테스트.

---

## 6단계. 트러블슈팅 패턴 연습

**목표:** 장애 났을 때 어디부터 볼지 기본 흐름 만들기.

### 공식 문서 요약
- [ ] 공식 Troubleshooting 문서에서 다음 케이스 발췌·요약:
  - 메모리 초과 (`Memory limit exceeded`).
  - 쿼리 타임아웃.
  - 디스크 부족.
  - 복제 지연.
  - 파트 손상 (broken parts).

### 의도적 장애 시나리오 실습
- [ ] 디스크 부족 상황 만들기 → 에러 메시지, 로그, 동작 확인.
- [ ] 복제 노드 1대 장시간 stop 후 다시 start → 복제 catch-up 동작 확인.

### system.* 기반 진단 연습
- [ ] `system.query_log`에서 슬로우 쿼리 조회.
- [ ] `system.parts`에서 파트 수·크기·상태 확인.
- [ ] `system.replicas`에서 복제 lag, queue length 확인.

---

## 7단계. 운영 런북 초안 작성

**목표:** 고객 지원·운영 시 바로 쓸 수 있는 체크리스트 문서 만들기.

### 장애 대응 플로우 문서화
- [ ] "쿼리가 느리다" 상황 대응 플로우:
  - 리소스 사용량 확인.
  - `system.query_log`, `system.parts`, `system.merges` 확인 순서 정의.
- [ ] "노드 한 대 다운" 상황 대응 플로우:
  - `system.replicas` 확인, 재시작 순서 정리.
  - Keeper 장애 시 처리 순서 포함.

### 정기 점검 루틴 정의
- [ ] 정기적으로 확인할 항목 리스트:
  - 파트 수, merges 상태, replicas lag, 디스크 사용량.

### 백업/복구 전략 초안
- [ ] 백업 전략 후보 정리 (파일 시스템 스냅샷 vs clickhouse-backup 도구 등).
- [ ] 단일 노드 기준 간단 백업/복원 테스트 절차 문서화.

---

## 변경 이력

| 날짜 | 버전 | 변경 내용 | 비고 |
|------|------|-----------|------|
| 2026-05-14 | v1.0 | README 최초 작성: 7단계 학습·실습 To-Do 리스트 추가 | ClickHouse v26.4.2.10 |
| 2026-05-19 | v1.1 | `outputs/` 산출물 폴더 생성, `outputs/README.md` 추가 | system.server_settings·system.settings 추출 결과 xlsx 포함 |
