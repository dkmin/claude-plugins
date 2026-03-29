---
name: cross-verify
description: AI 교차 검증 스킬. Codex, Gemini, Claude의 답변을 비교합니다. "cross verify", "교차 검증", "다른 AI한테도 물어봐", "멀티 AI 비교" 등의 요청에 활성화됩니다.
---

# AI 교차 검증 스킬

주어진 질문에 대해 Codex CLI, Gemini CLI, Claude(리드)의 답변을 수집하고 교차 검증한 뒤, 종합 분석을 제공합니다.

## 사전 요구사항

다음 CLI 도구 중 최소 하나가 설치되어 있어야 합니다:
- **Codex CLI**: `npm install -g @openai/codex`
- **Gemini CLI**: `npm install -g @anthropic-ai/gemini` 또는 Google 공식 설치 방법

## 워크플로우

### 1단계: 프롬프트 준비

1. 사용자 요청에서 검증할 질문/작업을 추출
2. 코드 관련 질문이면 자동으로 컨텍스트 수집:
   - `git diff` (변경사항이 있는 경우)
   - 관련 파일 내용
   - 프로젝트 구조
3. 각 에이전트용 프롬프트 구성:
   - 질문 본문
   - 수집된 컨텍스트
   - 사용자 언어로 응답하라는 지시

### 2단계: CLI 사용 가능 여부 확인

에이전트를 생성하기 전에 사용 가능한 CLI를 확인합니다:

```bash
which codex 2>/dev/null && echo "CODEX_OK" || echo "CODEX_NOT_FOUND"
which gemini 2>/dev/null && echo "GEMINI_OK" || echo "GEMINI_NOT_FOUND"
```

- 둘 다 미설치: 최소 하나의 CLI(codex 또는 gemini)가 필요하다고 안내 후 중단
- 하나만 설치: 2모델 비교 진행 (설치된 CLI + Claude)
- 둘 다 설치: 3모델 비교 진행

### 3단계: 에이전트 생성

#### 권장: Agent Teams (병렬 실행)

먼저 팀 생성을 시도합니다:

```
TeamCreate(team_name: "cross-verify", description: "AI 교차 검증")
```

TeamCreate 실패 시 (Agent Teams 미활성):
1. 표시: "Agent Teams를 사용하려면 settings.json env에 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`이 필요합니다. 활성화할까요?"
2. 사용자가 거부하면 순차적 Agent 생성으로 폴백 (아래 참조)

성공 시, **단일 메시지**로 에이전트를 병렬 생성합니다:

Codex 리뷰어 (codex 설치 시에만):
```
Agent(
  description: "Codex CLI 분석 실행",
  prompt: "<컨텍스트가 포함된 구성 프롬프트>",
  name: "gpt-agent",
  subagent_type: "gpt-agent",
  team_name: "cross-verify",
  model: "haiku",
  mode: "dontAsk"
)
```

Gemini 리뷰어 (gemini 설치 시에만):
```
Agent(
  description: "Gemini CLI 분석 실행",
  prompt: "<컨텍스트가 포함된 구성 프롬프트>",
  name: "gem-agent",
  subagent_type: "gem-agent",
  team_name: "cross-verify",
  model: "haiku",
  mode: "dontAsk"
)
```

#### 폴백: 순차적 Agent 생성

Teams를 사용할 수 없는 경우, `run_in_background: true`로 순차 생성합니다:

```
Agent(
  description: "Codex CLI 분석 실행",
  prompt: "<구성 프롬프트>",
  name: "gpt-agent",
  subagent_type: "gpt-agent",
  model: "haiku",
  mode: "dontAsk",
  run_in_background: true
)
```

#### 리드(Claude) 분석

에이전트가 작업하는 동안, 리드가 동일한 질문에 대해 자체 분석을 수행합니다.

### 4단계: 종합

모든 결과를 수신한 후, 다음 형식으로 종합합니다:

```markdown
## 교차 검증 결과

### 합의 (모든 모델이 동의)
- 높은 신뢰도의 결론

### 고유 인사이트
- **Codex (GPT)**: 이 모델만의 발견
- **Gemini**: 이 모델만의 발견
- **Claude**: 이 모델만의 발견

### 충돌 (의견 불일치)
- 항목별: 각 모델의 입장 + 리드의 판단

### 최종 결론
- 종합 권고사항
```

### 5단계: 정리

출력 후, 팀원에게 종료 메시지를 전송합니다:
```
SendMessage(to: "gpt-agent", message: {type: "shutdown_request"})
SendMessage(to: "gem-agent", message: {type: "shutdown_request"})
```

## 에러 처리

| 시나리오 | 조치 |
|----------|------|
| CLI 미설치 | 해당 모델 건너뛰고 나머지로 진행 |
| API 에러 (ModelNotFoundError 등) | 해당 모델 건너뛰고 결과에 기록 |
| 타임아웃 (에이전트 무응답) | 사용 가능한 결과로 종합 |
| 두 CLI 모두 실패 | 리드 분석과 에러 컨텍스트를 비교 |
| TeamCreate 실패 | 활성화 안내 표시, 순차적 Agent로 폴백 |

## 프롬프트 구성 규칙

| 요청 유형 | 포함할 컨텍스트 |
|-----------|----------------|
| 코드 리뷰 | git diff + 관련 파일 내용 |
| 아키텍처 질문 | 프로젝트 구조 + 주요 설정 파일 |
| 버그 분석 | 에러 로그 + 관련 코드 |
| 일반 기술 질문 | 질문만 |
