# cc-workflow Plugin

Claude Code 워크플로우 관리 플러그인

## 개요

이 플러그인은 AI 기반 워크플로우를 생성, 편집, 실행할 수 있는 기능을 제공합니다.
워크플로우는 `.claude/workflows/*.json` 파일로 저장됩니다.

## 명령어

| 명령어 | 설명 |
|--------|------|
| `/workflow-list` | 저장된 워크플로우 목록 조회 |
| `/workflow-create` | AI로 새 워크플로우 생성 |
| `/workflow-edit <name>` | 기존 워크플로우 수정 |
| `/workflow-run <name>` | 워크플로우 실행 |
| `/workflow-delete <name>` | 워크플로우 삭제 |

## 워크플로우 구조

워크플로우는 노드와 연결로 구성됩니다:

```
start → [처리노드들] → end
```

### 지원 노드 타입 (10종)

- **start/end**: 시작/종료점
- **prompt**: 메시지 표시
- **subAgent**: AI 태스크 실행
- **askUserQuestion**: 사용자 질문 (2-4 선택지)
- **ifElse**: 2방향 분기
- **switch**: 다방향 분기 (2-10)
- **skill**: Claude Code 스킬 호출
- **mcp**: MCP 도구 호출
- **subAgentFlow**: 중첩 워크플로우

## 사용 예시

### 워크플로우 생성

```
/workflow-create

이름: code-review
설명: PR 코드를 분석하고 버그가 있으면 수정 제안, 없으면 승인
```

### 워크플로우 실행

```
/workflow-run code-review
```

### 워크플로우 수정

```
/workflow-edit code-review
→ "분석 전에 PR 정보 조회 단계 추가해줘"
```

## 파일 구조

```
.claude/workflows/
├── code-review.json
├── deploy-check.json
└── bug-triage.json
```

## 스키마 참조

상세 스키마는 `skills/workflow-schema/SKILL.md` 참조
