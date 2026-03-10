# AI 활용 기록 002 - Claude Code Skill 분석

## 목적
Claude Code에서 제공하는 Skill 개념을 이해하고  
실제 개발 과정에서 어떻게 활용할 수 있는지 정리하기 위해 작성하였다.

---

## Claude Code Skill이란

Claude Code Skill은 특정 작업을 수행하기 위한  
instructions, scripts, resources 등을 묶어 둔 구조로  
AI가 반복적인 작업을 더 효율적으로 수행하도록 돕는 기능이다.

이를 통해 다음과 같은 작업을 빠르게 수행할 수 있다.

- 코드 생성
- 코드 리팩토링
- 오류 분석
- 문서 작성

Claude Code Skill을 활용하면 개발 workflow에서  
반복되는 작업을 빠르게 처리할 수 있다.

---

## Claude Code Skill의 장점

### 1. 반복 작업 자동화
같은 유형의 작업을 반복할 때  
일관된 방식으로 결과를 얻을 수 있다.

### 2. 개발 생산성 향상
코드 생성, 리팩토링, 문서 작성 등을 빠르게 수행할 수 있다.

### 3. 개발 workflow 정리
특정 작업 패턴을 Skill 형태로 정리하면  
AI 활용 방식이 체계화된다.

---

## Claude Code Skill 활용 예시 (MovieFinder 프로젝트)

Claude Code를 활용하여 Android 영화 검색 앱 **MovieFinder** 개발 과정에서  
코드 생성과 기능 구현을 진행하였다.

프로젝트 링크  
https://github.com/ChooJeongHo/MovieFinder

### 예시 작업 : 영화 검색 기능 구현

MovieFinder 앱에서 영화 검색 기능을 구현하기 위해  
Claude Code에게 Kotlin 기반 코드 생성을 요청하였다.

#### Claude Code 요청 예시

Kotlin Android 앱에서 영화 검색 API를 호출하고  
RecyclerView로 결과를 보여주는 구조를 만들어줘

#### 생성된 코드 예시

```kotlin
fun searchMovies(query: String) {
    viewModelScope.launch {
        try {
            val response = movieApi.searchMovies(query)
            _movies.value = response.results
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
}
```

#### 활용 결과

Claude Code를 통해 API 호출 구조와  
검색 결과 처리 코드를 빠르게 생성할 수 있었다.

이후 실제 앱 구조에 맞게 코드를 수정하고  
ViewModel 및 UI 구조와 연결하여 기능을 완성하였다.

---

## 느낀 점

Claude Code를 활용하면 초기 코드 작성 속도를 빠르게 높일 수 있다.

다만 생성된 코드를 그대로 사용하는 것보다는

- 코드 구조 검토
- 기능 동작 확인
- 추가 수정

과정을 거쳐 개발 workflow에 맞게 적용하는 것이 중요하다는 것을 확인하였다.
