# 066 - 릴리즈 파이프라인 자동화 — 버전 관리부터 GitHub Release까지

**`./scripts/release.sh patch` 한 줄로 버전 증가 → CHANGELOG → git 태그 → AAB 빌드 → GitHub Release 자동 완성**

---

## 목적

015일차 릴리즈 노트 자동 생성의 심화 버전.  
단순 릴리즈 노트를 넘어 **완전한 릴리즈 파이프라인** 구축.

---

## 생성된 파일

| 파일 | 역할 |
|------|------|
| `scripts/release.sh` | 로컬 릴리즈 오케스트레이터 (241줄) |
| `.github/workflows/release.yml` | 태그 push 시 서명 AAB 빌드 + GitHub Release 자동 생성 |
| `app/build.gradle.kts` | 환경변수 기반 릴리즈 서명 설정 추가 |
| `CHANGELOG.md` | 초기 v1.0.0 항목 |

---

## 사용법

```bash
# 패치 릴리즈 (1.0.0 → 1.0.1)
./scripts/release.sh patch

# 마이너 릴리즈 (1.0.0 → 1.1.0)
./scripts/release.sh minor

# 메이저 릴리즈 (1.0.0 → 2.0.0)
./scripts/release.sh major

# 시뮬레이션 (변경 없이 미리 보기)
./scripts/release.sh patch --dry-run

# 로컬 AAB 빌드 포함
./scripts/release.sh patch --build
```

---

## 릴리즈 파이프라인 흐름

```
./scripts/release.sh patch
    ↓
AndroidConfig.kt 버전 증가 (versionCode + versionName)
    ↓
CHANGELOG.md 자동 업데이트 (git log 기반)
    ↓
git commit + tag v1.0.1 생성
    ↓
git push + push tags
    ↓
GitHub Actions (release.yml) 자동 트리거
    ↓
서명된 AAB 빌드
    ↓
GitHub Release 자동 생성 + AAB 첨부
```

---

## GitHub Actions 트리거 조건

```yaml
on:
  push:
    tags:
      - 'v[0-9]+.[0-9]+.[0-9]+'
```

v로 시작하는 시맨틱 버전 태그만 트리거 → 잘못된 태그로 릴리즈 빌드 방지.

---

## 보안 처리

**환경변수 기반 릴리즈 서명:**
```kotlin
// RELEASE_KEYSTORE_PATH가 있으면 릴리즈 서명, 없으면 디버그 서명
val keystorePath = System.getenv("RELEASE_KEYSTORE_PATH")
signingConfig = if (releaseConfig.storeFile != null) releaseConfig
                else signingConfigs.getByName("debug")
```

**GitHub Actions 보안:**
- `github.ref_name`을 run 블록에 직접 삽입하지 않고 env 블록으로 분리
- 태그 패턴 제한으로 입력값 자체를 필터링

---

## GitHub Secrets 필요 항목

| 시크릿 | 생성 방법 |
|--------|---------|
| `RELEASE_KEYSTORE_BASE64` | `base64 -i release.jks \| pbcopy` |
| `RELEASE_STORE_PASSWORD` | 키스토어 비밀번호 |
| `RELEASE_KEY_ALIAS` | 키 별칭 |
| `RELEASE_KEY_PASSWORD` | 키 비밀번호 |
| `TMDB_READ_ACCESS_TOKEN` | TMDB 대시보드에서 발급 |

---

## 015일차 vs 066일차 비교

| 항목 | 015일차 | 066일차 |
|------|:-------:|:-------:|
| 목적 | 릴리즈 노트 자동 생성 | **완전한 릴리즈 파이프라인** |
| 버전 관리 | 수동 | **자동 증가** |
| AAB 빌드 | 수동 | **GitHub Actions 자동** |
| GitHub Release | 수동 | **자동 생성 + AAB 첨부** |
| 서명 | 수동 | **환경변수 기반 자동** |

---

## 느낀 점

릴리즈는 항상 귀찮고 실수하기 쉬운 작업이다. 버전 번호 바꾸고, CHANGELOG 쓰고, 태그 달고, 빌드하고, 업로드하는 과정을 하나씩 하다 보면 빠뜨리는 게 생긴다.

`./scripts/release.sh patch` 한 줄로 이 과정이 전부 자동화됐다. `--dry-run` 옵션으로 먼저 시뮬레이션해볼 수도 있어서 실수할 일이 없다.

**"릴리즈를 두려워하지 않게 만드는 것도 자동화의 역할이다."**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-06-29*
