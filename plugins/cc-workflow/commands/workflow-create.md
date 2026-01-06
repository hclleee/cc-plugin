---
description: AI 기반 워크플로우 생성 - 자연어 설명으로 새 워크플로우 생성
---

# AI 워크플로우 생성

자연어 설명을 기반으로 AI가 워크플로우를 생성합니다.

## 실행 절차

1. **사용자 입력 수집**
   - AskUserQuestion 도구로 다음 정보 수집:
     - 워크플로우 이름 (필수)
     - 워크플로우 설명 (자연어로 원하는 동작 설명)

2. **workflow-generator 에이전트 호출**
   - Task 도구로 `workflow-generator` 서브에이전트 실행
   - 입력: 사용자가 제공한 설명
   - 출력: 워크플로우 JSON

3. **생성 결과 검증**
   - 필수 노드 확인 (start, end)
   - 연결 유효성 검사
   - 노드 수 제한 확인 (최대 50개)

4. **파일 저장**
   - `.claude/workflows/` 디렉토리 생성 (없는 경우)
   - `<워크플로우이름>.json` 파일로 저장
   - 성공 메시지 출력

5. **후속 안내**
   - 워크플로우 실행: `/workflow-run <이름>`
   - 워크플로우 수정: `/workflow-edit <이름>`

## 예시

**사용자 입력:**
```
이름: code-review
설명: PR이 올라오면 코드를 분석하고, 버그가 있으면 수정 제안을 하고,
      없으면 승인 코멘트를 남기는 워크플로우
```

**생성 결과:**
```
워크플로우 'code-review' 생성 완료!

노드 구성:
- start → subAgent(코드분석) → ifElse(버그여부)
  - 버그있음 → subAgent(수정제안) → end
  - 버그없음 → subAgent(승인코멘트) → end

파일: .claude/workflows/code-review.json

다음 단계:
- 실행: /workflow-run code-review
- 수정: /workflow-edit code-review
```

## 참조

- 워크플로우 스키마: workflow-schema 스킬 참조
- 생성 에이전트: agents/workflow-generator.md
