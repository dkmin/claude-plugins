---
title: Cross-Verify 플러그인 분석 및 개선 계획
created_date: 2026-03-29 13:00
AI Model Name: Claude Opus 4.6
사용자 요청사항: cross-verify 플러그인 코드 리뷰 후 개선 계획 수립
내용 요약: Lead(Claude)→Sub-agents(Codex,Gemini) 패턴의 cross-verify 플러그인 v1.0.0 분석 결과 및 우선순위별 개선 로드맵
---

# Cross-Verify 플러그인 분석 및 개선 계획
Ref. : [[cross-verify-SKILL]] [[codex-reviewer]] [[gemini-reviewer]]
#status/planning #cross-verify #plugin #multi-ai

## 1. TL;DR

- Lead(Claude) + Sub-agents(Codex, Gemini) 오케스트레이션 패턴, v1.0.0 완성도 양호
- **Critical**: `codex exec --full-auto` 보안 이슈 + temp 파일 미정리
- **Quick Win**: 프롬프트 템플릿 추가만으로 비교 품질 대폭 향상 가능

## 2. 현재 아키텍처

```
User Request → SKILL.md(Lead/Claude)
                 ├─→ codex-reviewer (Agent, haiku) → Codex CLI
                 ├─→ gemini-reviewer (Agent, haiku) → Gemini CLI
                 └─→ Lead 자체 분석
                        ↓
               Synthesis (Consensus/Unique/Conflicts/Conclusion)
```

- **병렬**: Agent Teams 활성 시 → TeamCreate + 동시 Agent spawn
- **폴백**: Teams 미활성 시 → `run_in_background: true` 순차 실행
- **Degradation**: 3모델 → 2모델 → 1모델 자동 축소

## 3. 강점

| 항목 | 설명 |
|------|------|
| Graceful Degradation | CLI 유무에 따라 3→2→1 자동 전환 |
| 비용 최적화 | 서브에이전트 model=haiku (CLI 실행만 담당) |
| 에러 핸들링 | 5가지 시나리오 테이블 커버 |
| Synthesis 포맷 | 4단계 구조화된 비교 분석 출력 |

## 4. 개선 로드맵

### P0 - 배포 차단 (Critical)

| # | 이슈 | 해결 방향 |
|---|------|----------|
| 1 | `codex exec --full-auto` 보안 | read-only 플래그 조사, 없으면 sandbox/docker wrap 검토 |
| 2 | temp 파일 미정리 | trap/cleanup 로직 추가, `mktemp` 사용 후 종료 시 삭제 |

### P1 - 품질 향상 (Important)

| # | 이슈 | 해결 방향 |
|---|------|----------|
| 3 | 프롬프트 템플릿 부재 | SKILL.md에 요청 타입별 템플릿 섹션 추가 |
| 4 | CLI 플래그 하드코딩 | 환경변수 `CROSS_VERIFY_GEMINI_MODEL` 등으로 추출 |
| 5 | 타임아웃 불일치 | SKILL.md에 기본 180초 명시, agent .md와 동기화 |
| 6 | shutdown_request 실효성 | 완료 후 cleanup 불필요 확인 or TaskStop 전환 |

### P2 - 확장 (Nice to have)

| # | 항목 |
|---|------|
| 7 | 호출 전 비용 경고 메시지 |
| 8 | 결과 캐싱 (동일 질문 재사용) |
| 9 | 사용자 커스텀 모델 오버라이드 |

## 5. 실행 순서

```
Step 1: #1 full-auto 보안 수정 + #2 temp cleanup
Step 2: #3 프롬프트 템플릿 작성 (가장 높은 ROI)
Step 3: #4~#6 설정 외부화 및 일관성 정리
Step 4: #7~#9 사용성 개선
```
