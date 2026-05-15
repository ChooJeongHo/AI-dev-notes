# 037 - MovieFinder 전용 커스텀 MCP 서버 직접 제작

**외부 MCP만 쓰던 것에서 한 단계 더 — 프로젝트 전용 MCP 서버를 직접 만들어 Claude Code에 등록**

---

## 목적

지금까지 GitHub MCP, Exa MCP, Context7 MCP 등 외부 MCP만 사용해왔다.  
이번엔 MovieFinder ADB 자동화 명령어들을 MCP 툴로 직접 묶어서 Claude Code에 등록.

---

## 구현한 툴 4종

| 툴 | 설명 | 주요 파라미터 |
|----|------|-------------|
| `capture_screenshot` | 탭 이동 후 PNG 저장 | tab, folder (기본: latest) |
| `tap_tab` | 하단 탭바 탭 이동 | tab |
| `get_performance` | gfxinfo reset → 스크롤 → 결과 반환 | scroll_count (기본: 3) |
| `install_app` | ./gradlew [clean] installDebug | clean (기본: false) |

---

## 기술 스택

- **Node.js** v26
- **@modelcontextprotocol/sdk** ^1.12.0
- **stdio 기반 JSON-RPC** 프로토콜

---

## 핵심 발견 — Claude Code가 보안 경고 후 자동 수정

```
// 처음 작성한 코드 (보안 위험)
exec(`adb shell input tap ${x} ${y}`)
    ↓
⚠️ Security Warning: child_process.exec() → 쉘 인젝션 위험
    ↓
Claude Code가 스스로 execFile로 전면 교체
    ↓
execFile('adb', ['shell', 'input', 'tap', x, y])
✅ 쉘 인젝션 차단
```

**보안 처리 목록:**
- 모든 ADB 호출: `execFile(cmd, [arg1, arg2])` 배열 분리 → 쉘 인젝션 차단
- `folder` 파라미터: `^[\w\-]+$` 정규식 검증 → 경로 순회(`../`) 차단
- Gradle: `spawn('./gradlew', tasks, { cwd })` → cd 없이 안전하게 실행

---

## MCP 등록 방법

```bash
# MovieFinder 디렉토리에서 실행
claude mcp add moviefinder node /Users/serveace/AndroidStudioProjects/MovieFinder/moviefinder-mcp-server.js
```

---

## 동작 확인

### 스크린샷 테스트
```
moviefinder MCP로 홈 탭 스크린샷 찍어줘
```
→ 앱이 꺼져있어도 자동으로 실행 후 캡처 완료

### 성능 측정 테스트
```
moviefinder MCP로 성능 측정해줘
```

| 지표 | 수치 | 평가 |
|------|------|------|
| Janky frames | 0.91% | ✅ 기준(1%) 이하 |
| 50th percentile | 14ms | ✅ 정상 |
| 90th percentile | 16ms | ✅ 60fps 경계 |
| 99th percentile | 28ms | ⚠️ 간헐적 1프레임 지연 |
| Slow bitmap uploads | 0 | ✅ 이미지 병목 없음 |
| GPU 99th percentile | 9ms | ✅ GPU 여유 충분 |

---

## 사용 전후 비교

**Before (커스텀 MCP 없을 때)**
```
adb shell am start -n com.choo.moviefinder/.MainActivity
adb shell input tap 135 2150
adb exec-out screencap -p > screenshot_test/latest/home.png
```

**After (커스텀 MCP 사용)**
```
moviefinder MCP로 홈 탭 스크린샷 찍어줘
```

---

## ★ Insight (Claude Code 발언)

> "MCP 서버는 stdio 기반 JSON-RPC 프로토콜로 동작합니다. Claude Code가 서버를 자식 프로세스로 띄우고 stdin/stdout으로 통신하므로, console.log를 출력에 쓰면 프로토콜이 깨집니다 — 디버그 출력은 반드시 stderr로 보내야 합니다."

---

## 느낀 점

GitHub MCP, Exa MCP 등 외부 MCP를 쓸 때는 "연결해서 쓰는 것"이었는데, 직접 만드니까 MCP가 어떻게 동작하는지 구조가 보였다.

특히 Claude Code가 보안 훅에 걸려서 `exec()` → `execFile()`로 자동 수정한 부분이 인상적이었다. 사람이 만들었다면 놓쳤을 수 있는 보안 취약점을 AI가 스스로 잡아냈다.

**쓸 줄 아는 것과 만들 줄 아는 것은 다르다. 커스텀 MCP를 만들 줄 안다는 건 포트폴리오에서 확실한 차별점이 된다.**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*테스트 기기: Samsung Galaxy (R3CX80BS9MX)*  
*작성일: 2026-05-15*
