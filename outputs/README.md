# outputs

ClickHouse 학습·실습 과정에서 생성된 산출물 모음입니다.

## 파일 목록

| 파일명 | 설명 | 생성일 | 버전 |
|--------|------|--------|------|
| `clickhouse_system_settings.xlsx` | `system.server_settings` 및 `system.settings` 전체 항목 추출 결과 | 2026-05-19 | v26.4.2.10 |

## 설명

### clickhouse_system_settings.xlsx
- **출처**: `SELECT * FROM system.server_settings` / `SELECT * FROM system.settings`
- **버전**: ClickHouse v26.4.2.10 (official build)
- **수집일**: 2026-05-19
- **내용**:
  - `system.server_settings`: 서버 레벨 설정 전체 목록 (name, value, default, changed, description, type, changeablewithoutrestart, isobsolete)
  - `system.settings`: 세션/쿼리 레벨 설정 전체 목록
- **활용**: 설정값 레퍼런스, 기본값 대비 변경 항목 파악, 운영 튜닝 참고용
