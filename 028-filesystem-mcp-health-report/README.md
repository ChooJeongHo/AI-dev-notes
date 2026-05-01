# 028 - Filesystem MCP로 프로젝트 건강도 리포트 자동 생성

**14개 툴로 파일 읽기/쓰기/탐색/검색까지 — Filesystem MCP 첫 실험**

---

## 목적

연결만 되고 한 번도 실험하지 않았던 Filesystem MCP를 드디어 사용.  
Claude Code 없이도 파일 시스템을 직접 읽고 쓸 수 있는 Filesystem MCP로 MovieFinder 프로젝트 건강도 리포트 자동 생성.

---

## Filesystem MCP 14개 툴

| 카테고리 | 툴 | 설명 |
|---------|---|------|
| 읽기 | read_file, read_text_file, read_media_file, read_multiple_files | 파일 읽기 |
| 탐색 | list_directory, list_directory_with_sizes, directory_tree, list_allowed_directories | 구조 파악 |
| 쓰기 | write_file, edit_file, create_directory, move_file | 파일 생성/수정 |
| 검색 | search_files, get_file_info | 검색/메타데이터 |

---

## 실험 내용

### 1단계 - 프로젝트 건강도 분석

```
Filesystem MCP로 MovieFinder 프로젝트 전체를 분석해서
파일별 크기, 가장 큰 파일 TOP 10, 프로젝트 구조
프로젝트 건강도 리포트를 만들어줘.
```

**분석 결과:**

| 지표 | 수치 |
|------|------|
| 총 파일 수 | 340+ 파일 |
| 총 코드 라인 | ~20,900줄 |
| Kotlin 소스 | 233개 파일 / 14,327줄 |
| XML Resources | 102개 파일 / 5,967줄 |

**가장 큰 파일 TOP 5:**

| 순위 | 파일 | 라인 수 | 비고 |
|------|------|---------|------|
| 1 | SearchFragment.kt | 777줄 | ⚠️ 분리 고려 |
| 2 | DetailFragment.kt | 688줄 | ⚠️ Delegate 추가 여지 |
| 3 | fragment_settings.xml | 605줄 | ⚠️ 가장 큰 레이아웃 |
| 4 | FavoriteFragment.kt | 545줄 | ⚠️ 복잡한 필터/탭 로직 |
| 5 | fragment_detail.xml | 528줄 | 점진적 로딩 반영 |

**레이어별 코드 분포:**
```
Presentation  59파일  7,758줄  ████████████████████  54%
Data          62파일  2,892줄  ████████              20%
Domain        91파일  1,631줄  █████                 11%
Core          17파일    955줄  ███                    7%
DI             4파일    716줄  ██                     5%
```

**건강도 평가:**
- ✅ Clean Architecture 계층 분리
- ✅ 의존성 방향 준수
- ✅ 테스트 커버리지 (509 단위 + 23 UI)
- ✅ 보안 (Certificate Pinning, R8)
- ⚠️ SearchFragment(777줄), DetailFragment(688줄) 분리 여지

### 2단계 - 리포트를 파일로 저장

```
Filesystem MCP로 방금 분석한 건강도 리포트를
/Users/serveace/AndroidStudioProjects/MovieFinder/PROJECT_HEALTH.md 파일로 저장해줘.
```

→ `PROJECT_HEALTH.md` 로컬 파일 생성 완료

---

## 예상치 못한 발견 — Compose + Detekt 충돌

커밋 시도 시 Detekt pre-commit 훅에서 10개 위반 발생:

```
Function names should match the pattern: [a-z][a-zA-Z0-9]* [FunctionNaming]
```

**원인:** Compose `@Composable` 함수는 PascalCase가 관례인데 Detekt는 camelCase를 요구

**해결:** `detekt.yml`에 한 줄 추가
```yaml
FunctionNaming:
  ignoreAnnotated: ['Composable', 'Preview']  # 추가

LongMethod:
  ignoreAnnotated: ['Composable']  # 추가
```

**교훈:** XML → Compose 마이그레이션 시 Detekt 설정도 함께 업데이트 필요

---

## Filesystem MCP vs Claude Code 기본 파일 도구

| 구분 | Claude Code 기본 | Filesystem MCP |
|------|----------------|----------------|
| 파일 읽기 | ✅ | ✅ |
| 파일 쓰기 | ✅ | ✅ |
| 크기 포함 목록 | Bash 필요 | `list_directory_with_sizes` |
| 트리 구조 | Bash 필요 | `directory_tree` |
| 샌드박스 | 없음 | 허용된 디렉토리만 |
| Claude Code 없이 사용 | ❌ | **✅ 가능** |

핵심 차이: Filesystem MCP는 **Claude Code 없이도 어떤 AI 도구에서든 파일 시스템을 다룰 수 있어요**

---

## 느낀 점

Filesystem MCP는 Claude Code와 함께 쓸 때는 큰 차별점이 없었다. Claude Code 자체가 이미 파일을 읽고 쓸 수 있기 때문.

하지만 `list_directory_with_sizes`, `directory_tree` 같은 툴이 Bash 없이도 구조화된 데이터를 바로 반환해준다는 점이 편리했다.

그리고 예상치 못한 수확이 있었다. Compose 마이그레이션 후 Detekt 설정을 업데이트해야 한다는 걸 커밋 실패로 직접 배웠다. 설정 파일 한 곳(detekt.yml)에서 Compose 어노테이션을 예외 처리하는 게 파일마다 `@Suppress`를 붙이는 것보다 훨씬 깔끔하다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*생성 파일: PROJECT_HEALTH.md*  
*작성일: 2026-05-01*
