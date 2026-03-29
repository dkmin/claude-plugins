---
name: start-agent-teams
description: GPT(Codex)와 Gemini를 Agent Teams teammate로 생성하여 병렬 분석 후 종합합니다. tmux split panes 지원. "에이전트 팀", "agent teams", "GPT랑 Gemini 동시에", "교차 검증", "cross verify", "멀티 AI 비교", "팀으로 분석해줘" 등의 요청에 활성화됩니다. 여러 AI 모델에 같은 질문을 보내고 비교하고 싶을 때 사용하세요.
---

# start-agent-teams — 멀티 AI 교차 검증 오케스트레이터

Agent Teams를 사용하여 GPT(Codex CLI)와 Gemini(Gemini CLI)를 tmux split panes에서 병렬 실행하고, Lead(Claude)가 결과를 종합합니다.

## 사전 요구사항

- `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` (settings.json에 설정)
- Codex CLI 또는 Gemini CLI 중 최소 하나 설치
- tmux 세션 안에서 실행 권장 (split panes 자동 활성화)

## 사용법

```
/start-agent-teams {질문}
/start-agent-teams {질문} @{파일경로}
/start-agent-teams 이 코드의 보안 취약점 분석해줘 @src/auth.py
```

## 워크플로우

### 1단계: CLI 확인 및 teammate 구성 결정

```bash
which codex 2>/dev/null && echo "CODEX_OK" || echo "CODEX_NOT_FOUND"
which gemini 2>/dev/null && echo "GEMINI_OK" || echo "GEMINI_NOT_FOUND"
```

| codex | gemini | teammate 구성 |
|-------|--------|--------------|
| OK | OK | codex-reviewer + gemini-reviewer + Lead |
| OK | 없음 | codex-reviewer + Lead |
| 없음 | OK | gemini-reviewer + Lead |
| 없음 | 없음 | 안내 후 중단 |

### 2단계: 컨텍스트 수집

| 조건 | 수집 대상 |
|------|----------|
| 파일 경로(@) 언급 | 해당 파일 내용 |
| git 변경사항 있음 | `git diff` (짧으면 포함) |
| 프로젝트 구조 질문 | 디렉토리 트리 |
| 일반 질문 | 질문만 |

### 3단계: 팀 생성

```
TeamCreate(team_name: "cross-verify", description: "AI 교차 검증")
```

### 4단계: Task 생성

사용 가능한 CLI에 따라 Task를 생성합니다:

```
TaskCreate(subject: "GPT 분석", description: "codex exec로 질문 분석")      ← codex OK일 때
TaskCreate(subject: "Gemini 분석", description: "gemini CLI로 질문 분석")   ← gemini OK일 때
TaskCreate(subject: "Lead 종합", description: "모든 결과를 교차 검증 종합")
```

### 5단계: Teammate 병렬 생성

단일 메시지에서 모든 teammate를 동시에 생성합니다.

**Codex Reviewer** (codex 설치 시):
```
Agent(
  description: "Codex CLI로 GPT 분석 실행",
  prompt: "당신은 codex-reviewer입니다.
    1. Task 'GPT 분석'을 claim하세요 (TaskUpdate: owner='codex-reviewer', status='in_progress')
    2. 다음 명령을 실행하세요:
       TMPFILE=$(mktemp /tmp/cv-codex-XXXXXXXX.txt)
       프롬프트를 $TMPFILE에 저장
       cat $TMPFILE | codex exec -m gpt-5.2 --full-auto - 2>/dev/null
       rm -f $TMPFILE
    3. 결과를 Lead에게 SendMessage로 전송
    4. Task를 completed로 마크

    프롬프트 내용:
    {구성된 프롬프트 + 컨텍스트}

    반드시 한국어로 답변하세요.",
  name: "codex-reviewer",
  team_name: "cross-verify",
  model: "haiku",
  mode: "dontAsk"
)
```

**Gemini Reviewer** (gemini 설치 시):
```
Agent(
  description: "Gemini CLI로 분석 실행",
  prompt: "당신은 gemini-reviewer입니다.
    1. Task 'Gemini 분석'을 claim하세요 (TaskUpdate: owner='gemini-reviewer', status='in_progress')
    2. 다음 명령을 실행하세요:
       TMPFILE=$(mktemp /tmp/cv-gemini-XXXXXXXX.txt)
       프롬프트를 $TMPFILE에 저장
       cat $TMPFILE | gemini -p - -m gemini-2.5-pro -o text 2>/dev/null
       rm -f $TMPFILE
    3. 결과를 Lead에게 SendMessage로 전송
    4. Task를 completed로 마크

    프롬프트 내용:
    {구성된 프롬프트 + 컨텍스트}

    반드시 한국어로 답변하세요.",
  name: "gemini-reviewer",
  team_name: "cross-verify",
  model: "haiku",
  mode: "dontAsk"
)
```

### 6단계: Lead 자체 분석

Teammate들이 작업하는 동안 Lead(Claude)가 동일한 질문에 대해 자체 분석을 수행합니다.

### 7단계: 종합

모든 teammate 결과를 수신한 후, 다음 형식으로 종합합니다:

```markdown
## 교차 검증 결과

### 합의 (모든 모델이 동의)
- 높은 신뢰도의 결론

### 고유 인사이트
- **GPT (gpt-5.2)**: 이 모델만의 발견
- **Gemini (gemini-2.5-pro)**: 이 모델만의 발견
- **Claude (Lead)**: 이 모델만의 발견

### 충돌 (의견 불일치)
- 항목별: 각 모델의 입장 + Lead의 판단

### 최종 결론
- 종합 권고사항
```

### 8단계: 팀 정리 (사용자 확인 필수)

종합 출력 후 사용자에게 확인합니다:

```
"teammate 정리할까요? (y/n)"
```

사용자가 승인하면:
```
SendMessage(to: "codex-reviewer", message: {type: "shutdown_request"})
SendMessage(to: "gemini-reviewer", message: {type: "shutdown_request"})
// 모든 teammate 종료 확인 후
TeamDelete()
```

사용자가 거부하면 teammate를 유지합니다. 사용자가 직접 pane에서 추가 질문을 할 수 있습니다.

> **중요**: Lead가 임의로 teammate를 종료하지 않습니다. 사용자가 pane에서 직접 대화할 수 있으므로 항상 확인 후 종료합니다.

## tmux 조작

| 동작 | 방법 |
|------|------|
| teammate pane 이동 | pane 클릭 또는 `Ctrl+B` → 방향키 |
| teammate에게 직접 메시지 | 해당 pane 클릭 → 타이핑 |
| Lead로 돌아가기 | Lead pane 클릭 |
| teammate 전환 (in-process) | `Shift+Down` |

## 에러 처리

| 시나리오 | 조치 |
|----------|------|
| 두 CLI 모두 미설치 | 설치 안내 후 중단 |
| TeamCreate 실패 | "settings.json에 CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 필요" 안내 |
| CLI 하나만 설치 | 설치된 CLI + Lead 2모델 비교로 진행 |
| Teammate 타임아웃 | 사용 가능한 결과만으로 종합 |
| 모델 에러 (404 등) | 해당 모델 건너뛰고 나머지로 종합 |

## 주의사항

- ChatGPT 계정은 `gpt-5.2`만 지원 (o3, gpt-4o 등 사용 불가)
- Gemini CLI는 반드시 `-m gemini-2.5-pro` 지정 (기본 모델 404)
- `gemini-3.1-pro`는 아직 미출시 (404)
- tmpfile은 고유 접두사 사용 (`cv-codex-`, `cv-gemini-`) 충돌 방지
- `codex exec --full-auto`는 sandbox workspace-write 모드 (파일 수정 가능)
