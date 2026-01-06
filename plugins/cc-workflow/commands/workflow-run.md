---
description: 워크플로우 실행 - 저장된 워크플로우를 순차적으로 실행
---

# 워크플로우 실행

저장된 워크플로우를 로드하고 정의된 흐름대로 실행합니다.

## 실행 절차

1. **워크플로우 선택**
   - 인자로 워크플로우 이름이 제공되지 않으면:
     - `/workflow-list` 결과를 보여주고
     - 실행할 워크플로우 이름 입력 요청

2. **워크플로우 검증**
   - 파일 존재 확인
   - JSON 파싱
   - 필수 구조 검증 (start/end 노드 존재 여부)

3. **실행 시작 확인**
   ```
   워크플로우 'code-review' (v1.0.0)를 실행하시겠습니까?

   구조:
   start → analyze → has-bugs?
     ├─ Yes → suggest-fix → end
     └─ No → approve → end

   예상 노드: 5개
   ```

4. **workflow-executor 에이전트 호출**
   - Task 도구로 `workflow-executor` 서브에이전트 실행
   - 각 노드를 순차적으로 실행
   - 실행 로그 실시간 표시

5. **결과 요약**
   - 실행 완료 상태
   - 각 노드별 결과 요약
   - 총 실행 시간

## 사용 예시

```
/workflow-run code-review
```

## 출력 예시

```
## 워크플로우 실행: code-review

### 실행 로그

[1/5] start
  └─ 워크플로우 시작

[2/5] subAgent: analyze
  └─ 코드 분석 중...
  └─ 결과: 버그 2개, 경고 3개 발견

[3/5] ifElse: has-bugs
  └─ 조건 평가: 버그 존재 → Yes 분기

[4/5] subAgent: suggest-fix
  └─ 수정 제안 생성 중...
  └─ 결과: PR에 2개 코멘트 작성

[5/5] end
  └─ 워크플로우 완료

### 실행 결과
- 상태: 성공
- 실행 노드: 5개
- 실행 시간: 32초
- 최종 결과: 코드 리뷰 완료, 2개 수정 제안
```

## 사용자 상호작용

`askUserQuestion` 노드 도달 시:
```
[3/6] askUserQuestion: priority-select
  └─ 우선순위를 선택하세요:
     1. Critical - 즉시 처리
     2. Major - 금주 내 처리
     3. Minor - 백로그 추가
```

사용자 선택에 따라 해당 분기로 진행

## 에러 처리

```
## 실행 오류

[3/5] subAgent: analyze 에서 오류 발생
  └─ 오류: API 호출 실패

실행이 중단되었습니다.
- 실행된 노드: 2개
- 실패 노드: analyze

재시도하시겠습니까?
```

## 참조

- 실행 에이전트: `agents/workflow-executor.md`
- 스키마: `skills/workflow-schema/SKILL.md`
