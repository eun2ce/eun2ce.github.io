---
title: "[git] 커밋(commit) 잘 남기는 방법"
date: 2024-12-04 16:00:00 +0900
categories: [ "programming", "git" ]
tags: [ "git", "commit" ,"programming" ]
pin: false
math: false
mermaid: false
---

## 커밋(commit) 구조

```
타입(스코프): 주제(제목) // Header (헤더)

본문 // Body (바디)

바닥글 // Footer
```

### 커밋 메시지 규약

| 타입 이름    | 내용                        |
|----------|---------------------------|
| feat     | 새로운 기능 추가                 |
| fix      | 버그 수정                     |
| build    | 빌드 관련 파일 수정 / 모듈 설치 or 삭제 |
| chore    | 그 외 자잘한 수정                |
| ci       | ci 관련 설정 수정               |
| docs     | 문서 수정                     |
| style    | 코드 스타일 혹은 포맷              |
| refactor | 코드 리팩터링                   |
| test     | 테스트 코드 수정                 |
| perf     | 성능 개선                     |

모든 구조를 지키며 커밋 할 필요는 없습니다. (필요 시 사용)  
개발자들간의 약속이기때문에 header 정도는 지키며 커밋하는 습관을 들이는 것이 좋습니다.
