# 064 - Context7 MCP 심화 — 공식 문서 기반 라이브러리 개선점 자동 발견

**공식 문서와 실제 코드를 대조해서 8개 개선점 발견 — 훈련 데이터 컷오프 이후 최신 API 패턴까지 반영**

---

## 목적

011일차 Context7 MCP 기본 활용의 심화 버전.  
Paging3, Hilt, Coil, Room 공식 문서를 실시간으로 참조해서 현재 코드와 대조 분석.

---

## Context7 MCP란?

라이브러리 공식 문서를 벡터 인덱싱해서 쿼리 기반으로 관련 스니펫을 반환.  
Claude의 훈련 데이터 컷오프 이후 API 변경사항도 반영된다는 게 핵심.

---

## 발견된 개선점 8건

### 🔴 높음

**H-1 — Workers에 @HiltWorker 미적용**
```kotlin
// 현재: EntryPoint 우회 방식
class ReleaseNotificationWorker(context, workerParams) : CoroutineWorker(...)

// 개선: @HiltWorker로 직접 주입
@HiltWorker
class ReleaseNotificationWorker @AssistedInject constructor(
    @Assisted context: Context,
    @Assisted workerParams: WorkerParameters,
    private val reminderScheduler: ReleaseNotificationScheduler
) : CoroutineWorker(context, workerParams)
```

**R-2 — Entity @ColumnInfo(defaultValue) 누락**  
Room 스키마 검증 시 Kotlin 기본값과 SQLite 스키마 기본값 불일치 → IllegalStateException 위험

---

### 🟡 중간

**P-3 — System.currentTimeMillis() 15곳 이상 사용**  
CLAUDE.md에 kotlinx-datetime으로 교체 명시됐는데 데이터 레이어 전체에 남아있음  
→ `Clock.System.now().toEpochMilliseconds()`로 교체 시 테스트에서 FakeClock 주입 가능

**P-2 — PagingConfig.maxSize 미설정**
```kotlin
// 현재: 무제한 메모리 누적
PagingConfig(pageSize = PAGE_SIZE, prefetchDistance = PREFETCH_DISTANCE)

// 개선: 메모리 상한 설정
PagingConfig(
    pageSize = PAGE_SIZE,
    prefetchDistance = PREFETCH_DISTANCE,
    maxSize = PAGE_SIZE * 3  // viewport 밖 페이지 자동 drop
)
```

---

### 🟢 낮음

**C-1 — 디스크 캐시 maxSizeBytes → maxSizePercent**
```kotlin
// 현재: 50MB 하드코딩
.maxSizeBytes(50L * 1024 * 1024)

// 개선: 기기 저장공간의 2%로 자동 적응
.maxSizePercent(0.02)
```

**P-1 — APPEND 시 remoteKeyDao.getRemoteKey() 이중 조회**  
동일 category로 DB를 2번 조회 → 변수를 스코프 밖으로 올려서 1회로 절감

**R-1 — MemoDao 기본값 제거**  
Room은 @Query 메서드의 Kotlin 기본값을 별도 처리 안 함 → 혼란 유발

**H-2 — @ViewModelScoped 미활용**  
DetailViewModel Delegate들의 snackbarChannel 공유를 더 명확하게 표현 가능

---

## 우선순위 요약

| 우선순위 | 항목 | 영향 |
|---------|------|------|
| 🔴 높음 | H-1 @HiltWorker 미적용 | Workers DI 불완전, 테스트 불가 |
| 🔴 높음 | R-2 @ColumnInfo(defaultValue) 누락 | 스키마 해시 불일치 위험 |
| 🟡 중간 | P-3 System.currentTimeMillis() 15곳+ | 테스트 가능성, 일관성 |
| 🟡 중간 | P-2 PagingConfig.maxSize 미설정 | 장시간 스크롤 시 메모리 누수 |
| 🟢 낮음 | C-1 maxSizeBytes → maxSizePercent | 기기 적응형 캐시 |
| 🟢 낮음 | P-1 APPEND 이중 DB 조회 | 미세 성능 개선 |
| 🟢 낮음 | R-1 DAO 기본값 제거 | 코드 명확성 |
| 💡 선택 | H-2 @ViewModelScoped 도입 | 아키텍처 정리 |

---

## 011일차 vs 064일차 비교

| 항목 | 011일차 | 064일차 |
|------|:-------:|:-------:|
| 목적 | ML Kit API 참조로 구현 | **코드 vs 공식 문서 대조 분석** |
| 사용 방식 | 구현 도중 참조 | **감사 도구로 활용** |
| 결과물 | 기능 구현 | **8개 개선점 발견** |
| 훈련 데이터 한계 극복 | 부분적 | **최신 API 패턴 반영** |

---

## ★ Insight (Claude Code 발언)

> "Context7은 라이브러리별 공식 문서를 벡터 인덱싱해서 쿼리 기반으로 관련 스니펫을 반환합니다. 훈련 데이터 컷오프 이후의 API 변경사항도 반영됩니다."

---

## 느낀 점

Exa MCP가 "최신 버전 정보를 웹에서 검색"한다면, Context7 MCP는 "공식 문서 자체를 읽어서 현재 코드와 대조"한다.

특히 @HiltWorker 미적용과 @ColumnInfo(defaultValue) 누락은 공식 문서를 직접 참조하지 않으면 발견하기 어려운 이슈였다.

**"AI가 공식 문서를 읽고 내 코드를 리뷰한다" — 사람이 문서를 읽지 않아도 모범 사례를 알 수 있다.**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-06-25*
