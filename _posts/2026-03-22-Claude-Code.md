---
layout: article
title: Anthropic Academy - Claude Code 공부 기록
tags: [claude, ai, tool]
aside:
  toc: true
---

## 들어가며

Anthropic에서 직접 만든 AI, Claude 학습 컨텐츠 [Anthropic Academy](https://www.anthropic.com/learn)가 오픈했다는 소식을 접했습니다👏

개인적으로, 그리고 업무에서 열심히 쓰고 있는 만큼 제대로 쓰기 위해 Claude Code 강의를 듣고 있습니다. 이 글은 강의를 들으며 정리한 내용입니다. 

<!--more-->

## Coding Assistant란 무엇인가?

Coding Assistant는 단순히 코드를 작성하는 도구가 아니라, 언어 모델을 활용해 복잡한 프로그래밍 작업을 수행하는 시스템이다.

### Tool Use

언어 모델 자체는 텍스트를 입력받아 텍스트를 반환하는 것만 가능하다. 파일을 읽거나 명령어를 실행하는 건 불가능. 이 한계를 극복하기 위해 **Tool Use**라는 구조를 사용한다.

언어 모델은 특정 형식의 텍스트를 생성할 뿐이지만, Coding Assistant가 이를 해석하고 실행함으로써 파일 읽기/쓰기, 명령어 실행 등이 가능해진다.

### Tool Use가 뛰어나면 좋은 점

+ **복잡한 작업 처리** — 여러 tool을 조합해 어려운 작업을 수행하고, 처음 보는 tool도 활용 가능
+ **확장성** — 새로운 tool을 쉽게 추가할 수 있고, 모델이 이를 적응적으로 사용
+ **보안** — 코드베이스 전체를 외부 서버에 보내지 않고도 탐색 가능 (인덱싱 불필요)

## 주요 명령어 및 기능

### Slash Commands

+ `/init` — 프로젝트 구조를 파악해서 **CLAUDE.md** 파일을 자동 생성. Claude Code 최초 실행 시 한 번 실행해두면 좋음.
+ `/compact` — 대화 내용을 요약하여 컨텍스트 윈도우를 확보. 작업 하나 끝나면 한 번 실행하면 좋음. 토큰이 부족해지면 자동으로 실행되기도 함.
+ `/clear` — 대화 기록을 완전히 초기화. `/compact`와 달리 요약 없이 깨끗하게 새 대화 시작.
+ `/cost` — 현재 세션에서 사용한 토큰 및 비용 확인.
+ `/help` — 사용 가능한 명령어 및 도움말 확인.

### 입력 단축키

+ `#` — CLAUDE.md 파일에 지침 추가하고 싶을 때 `#` 입력 후 작성.
+ `@` — 파일명 언급. 특정 파일을 컨텍스트에 포함시킬 때 사용.
+ `esc` — 실행 중단. 두 번 누르면 이전 질문을 수정할 수 있음.

### 모드

+ **Plan mode** — 코드 수정하기 전에 계획 먼저 세움. 넓은 영역을 여러 단계로 수정해야 할 때 좋음. `shift+tab`으로 토글하거나 프롬프트에 "plan"이라고 입력.
+ **Think mode** — 프롬프트에 직접 "think"라고 명시하면 됨. 복잡한 문제를 깊게 이해하고 트러블슈팅해야 할 때 좋음. 강도는 3단계: `think` → `think hard` → `think harder` (깊어질수록 더 많은 토큰을 사용해서 심층 추론).

### Custom Commands

자주 쓰는 프롬프트를 커스텀 슬래시 명령어로 만들 수 있다.

+ `.claude/commands/` 디렉토리 하위에 **마크다운 파일**을 생성하면, 파일명이 곧 명령어 이름이 됨
  - 예: `.claude/commands/review.md` → `/review`로 실행
+ 프롬프트 내에서 `$ARGUMENTS`를 사용하면 명령어 실행 시 인자를 받을 수 있음
  - 예: 파일 내용이 `$ARGUMENTS 파일을 리뷰해줘`이면, `/review src/main.js`로 호출 가능

### MCP 서버 활용

**MCP(Model Context Protocol)** 서버를 연결하면 Claude Code에 외부 tool을 추가할 수 있다. 예시로 Playwright MCP를 연결해보자.

**1단계: Playwright MCP 설치 및 등록**

```shell
claude mcp add playwright npx @playwright/mcp@latest
```

**2단계: Permission 추가**

MCP 서버의 tool을 사용하려면 permission 설정이 필요하다. `.claude/settings.local.json`에 추가한다.

```json
{
  "permissions": {
    "allow": ["mcp__playwright"],
    "deny": []
  }
}
```

**3단계: 사용**

Claude Code를 재시작하면 Playwright tool이 활성화된다. 이후 프롬프트에서 "이 페이지 열어서 스크린샷 찍어줘" 같은 요청이 가능해진다.

> 💡 근데 Claude에 눈이 달린 것도 아닌데, 브라우저를 연다고 UI를 어떻게 평가할까? 
> — Playwright MCP는 **스크린샷 캡처**와 **접근성 트리(Accessibility Snapshot)** 추출 tool을 제공한다. Claude는 멀티모달 모델이라 스크린샷 이미지를 "보고" 해석할 수 있고, 접근성 트리를 통해 DOM 구조와 ARIA 속성을 텍스트로 파악할 수도 있다. 브라우저를 여는 것 자체가 아니라, 브라우저에서 시각/구조 데이터를 뽑아오는 것이 핵심.

### Hooks

Hook은 Claude가 tool을 실행하기 **전/후**에 원하는 명령어를 자동으로 실행할 수 있는 기능이다.

**동작 원리**

일반적인 흐름: 사용자 질문 → Claude가 tool 사용 결정 → tool 실행 → 결과 반환. Hook은 이 과정에서 tool 실행 직전/직후에 끼어들어 추가 코드를 실행한다.

+ **PreToolUse** — tool 실행 **전**에 동작. 특정 파일 접근 차단 등 제어 용도.
+ **PostToolUse** — tool 실행 **후**에 동작. 포맷팅, 테스트 실행 등 후처리 용도.

**설정 위치**

+ `~/.claude/settings.json` — 전역 설정 (모든 프로젝트에 적용)
+ `.claude/settings.json` — 프로젝트 설정 (팀과 공유)
+ `.claude/settings.local.json` — 프로젝트 개인 설정 (커밋하지 않음)

Claude Code 내에서 `/hooks` 명령어로도 설정할 수 있다.

**활용 예시**

+ 파일 수정 후 자동 포맷팅 (prettier, eslint --fix 등)
+ 파일 변경 시 자동 테스트 실행
+ 특정 파일에 대한 읽기/수정 차단
+ 린터나 타입 체커 실행 후 결과를 Claude에 피드백
+ Claude가 접근/수정한 파일 로깅

정리하면, **PreToolUse**는 Claude가 하려는 것을 제어하고, **PostToolUse**는 Claude가 한 것을 보강하는 데 쓴다.

***

### 참고자료
- [Anthropic Academy](https://www.anthropic.com/learn)
