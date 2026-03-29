# claude-plugins

> DevBrother Claude Code 플러그인 마켓플레이스

## Overview

Claude Code용 플러그인 모음. AI 에이전트 간 협업·교차 검증 도구를 제공한다.

## 디렉토리 구조

```
claude-plugins/
├── .claude-plugin/marketplace.json   ← 마켓플레이스 메타
├── plugins/
│   ├── cross-verify/                 ← AI 교차 검증 (Codex+Gemini+Claude)
│   │   ├── .claude-plugin/plugin.json
│   │   └── skills/cross-verify/
│   │       ├── SKILL.md              ← 오케스트레이터
│   │       ├── SKILL-ko.md           ← 한국어 번역
│   │       └── agents/
│   │           ├── codex-reviewer.md
│   │           └── gemini-reviewer.md
│   ├── req-gpt/                      ← GPT 서브에이전트 호출 #260329-15
│   │   ├── .claude-plugin/plugin.json
│   │   └── skills/req-gpt/
│   │       └── SKILL.md
│   └── req-gem/                      ← Gemini 서브에이전트 호출 #260329-15
│       ├── .claude-plugin/plugin.json
│       └── skills/req-gem/
│           └── SKILL.md
├── docs/                             ← Zettel 분석 카드
└── README.md
```

## 플러그인 목록

| 플러그인 | 설명 | 기본 모델 |
|---------|------|----------|
| cross-verify | Codex(GPT) + Gemini + Claude 교차 검증 | haiku (서브에이전트) |
| req-gpt | Codex CLI로 GPT 호출 | gpt-5.2 |
| req-gem | Gemini CLI로 Gemini 호출 | gemini-2.5-pro | #260329-15

## 개발 참고

- 플러그인 구조: `plugins/{name}/skills/{name}/SKILL.md`
- 에이전트 정의: `plugins/{name}/skills/{name}/agents/*.md`
- 마켓플레이스 등록: `.claude-plugin/marketplace.json`에 추가
- Codex CLI: `codex exec -m {model} --full-auto "{prompt}"`
- Gemini CLI: `gemini -p "{prompt}" -m gemini-2.5-pro -o text`

## 주의사항

- ChatGPT 계정은 `gpt-5.2`만 지원 (o3, gpt-4o 등 사용 불가)
- `codex exec --full-auto`는 sandbox workspace-write 모드 (read-only 아님)
- Gemini CLI는 반드시 `-m` 플래그로 모델 지정 (기본 모델 404 에러)
- `gemini-3.1-pro`는 아직 미출시 (404), `gemini-2.5-pro` 사용 #260329-15
