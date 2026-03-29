---
name: req-gem
description: Gemini에게 질문을 보내고 답변을 받는 스킬. Gemini CLI를 통해 Gemini를 서브에이전트처럼 호출합니다. "/req-gem", "Gemini한테 물어봐", "Gemini 의견", "ask gemini", "gemini로 질문" 등의 요청에 활성화됩니다.
---

# req-gem — Gemini 서브에이전트 호출 스킬

Gemini CLI를 통해 Google Gemini 모델에 질문을 보내고 응답을 받습니다.

## 사용법

```
/req-gem {질문}                           ← 기본 모델(gemini-2.5-pro)로 질문
/req-gem gemini-2.5-flash {질문}          ← 모델 지정
/req-gem gemini-2.5-pro 이 코드 리뷰해줘  ← 모델 + 질문
```

## 인자 파싱

`$ARGUMENTS`에서 첫 번째 토큰이 알려진 모델명이면 모델로 사용하고, 나머지를 질문으로 처리합니다.

알려진 모델명 패턴:
- `gemini-*` (gemini-3.1-pro, gemini-2.5-pro, gemini-2.5-flash 등)

첫 토큰이 모델명이 아니면 전체를 질문으로 사용하고 기본 모델 `gemini-2.5-pro`를 적용합니다.

## 워크플로우

### 1단계: CLI 확인

```bash
which gemini 2>/dev/null || echo "GEMINI_NOT_INSTALLED"
```

미설치 시: "Gemini CLI가 필요합니다: `npm install -g @google/gemini-cli`" 안내 후 중단.

### 2단계: 컨텍스트 수집

질문이 코드 관련이면 자동으로 수집:

| 조건 | 수집 대상 |
|------|----------|
| git 변경사항 있음 | `git diff` |
| 특정 파일 언급 | 해당 파일 내용 |
| 프로젝트 구조 질문 | 디렉토리 트리 |
| 일반 질문 | 질문만 |

### 3단계: 프롬프트 구성

```
[사용자 질문]

---
[컨텍스트 (있는 경우)]

반드시 한국어로 답변하세요.
```

### 4단계: Gemini CLI 실행

짧은 프롬프트 (< 1000자):
```bash
gemini -p "{prompt}" -m {model} -o text 2>/dev/null
```

긴 프롬프트 (>= 1000자):
```bash
TMPFILE=$(mktemp /tmp/req-gem-XXXXXX.txt)
cat > "$TMPFILE" << 'PROMPT_EOF'
{prompt content}
PROMPT_EOF
cat "$TMPFILE" | gemini -p - -m {model} -o text 2>/dev/null
rm -f "$TMPFILE"
```

- Bash timeout: 180000 (3분)
- `-o text`로 텍스트 출력
- `2>/dev/null`로 stderr MCP 경고 억제

### 5단계: 결과 출력

Gemini 응답을 다음 형식으로 출력:

```markdown
## Gemini 응답 ({model})

{gemini 출력 원문}
```

응답을 요약하거나 수정하지 않습니다. 사용자가 추가 분석을 요청하면 그때 Claude가 해석합니다.

## 에러 처리

| 시나리오 | 조치 |
|----------|------|
| Gemini CLI 미설치 | 설치 안내 후 중단 |
| 모델명 오류 / 404 | `gemini-2.5-pro`로 자동 폴백 후 재시도 |
| 타임아웃 (3분 초과) | 타임아웃 안내, 짧은 질문으로 재시도 제안 |
| API 인증 에러 | `gemini` 실행 후 브라우저 인증 안내 |
| 기타 에러 | 에러 메시지 원문 표시 |
