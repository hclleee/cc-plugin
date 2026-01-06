---
description: 워크플로우 구조를 트리 형태로 시각화
---

# 워크플로우 시각화

워크플로우 JSON을 읽어 터미널에서 보기 쉬운 트리 구조로 출력합니다.

## 실행 절차

1. **워크플로우 파일 확인**
   - 인자로 워크플로우 이름이 주어지면 `.claude/workflows/<name>.json` 로드
   - 인자가 없으면 AskUserQuestion으로 목록에서 선택

2. **JSON 파싱 및 구조 분석**
   - nodes 배열에서 모든 노드 추출
   - connections 배열에서 연결 관계 파악
   - 합류점(여러 경로가 모이는 노드) 식별

3. **트리 구조 생성**
   - start 노드부터 시작
   - 분기점에서 들여쓰기로 각 경로 표시
   - 합류점은 `[LABEL]`로 참조하고 하단에 정의
   - 순환이 있으면 `⟲` 심볼로 표시

## 출력 형식

### 노드 타입별 심볼

| 타입 | 심볼 | 설명 |
|------|------|------|
| start | ○ | 시작점 |
| end | ● | 종료점 |
| prompt | ◇ | 메시지 출력 |
| subAgent | ◈ | AI 서브에이전트 |
| askUserQuestion | ◆ | 사용자 질문 |
| ifElse | ◊ | 2방향 분기 |
| switch | ❖ | 다중 분기 |
| skill | ★ | 스킬 호출 |
| mcp | ⬡ | MCP 도구 호출 |
| subAgentFlow | ◎ | 중첩 워크플로우 |

### 분기 표시

- askUserQuestion: `├─[1] "선택지1"`, `├─[2] "선택지2"`, ...
- ifElse: `├─✓ "조건 충족"`, `└─✗ "조건 미충족"`
- switch: `├─"케이스1"`, `├─"케이스2"`, `└─"기타" (default)`

### 합류점 참조

여러 경로가 같은 노드로 합류하면:
- 본문에서 `→ [LABEL]`로 참조
- 하단에 `├── [LABEL] 노드내용 → 다음노드` 형태로 정의

### 순환 표시

이전 노드로 돌아가는 연결이 있으면:
- `→ ○ start ⟲` 형태로 순환 표시

## 출력 예시

```
[workflow-test v1.0.0] 9 nodes

○ start
│
├── ◇ prompt "워크플로우 테스트를 시작합니다"
│
├── ◆ ask "테스트 유형을 선택하세요"
│   │
│   ├─[1] "AI 분석 테스트"
│   │     └── ◈ subagent "프로젝트 구조 분석" → [A]
│   │
│   ├─[2] "조건 분기 테스트"
│   │     └── ◊ ifelse "분석 성공 여부"
│   │           ├─✓ "성공한 경우" → [A]
│   │           └─✗ "실패한 경우" → [B]
│   │
│   └─[3] "다중 분기 테스트"
│         └── ❖ switch "결과 유형 판단"
│               ├─"유형 A" → [A]
│               ├─"유형 B" → [A]
│               └─"기타" (default) → [B]
│
├── [A] ◇ prompt "테스트 성공!" → ● end
└── [B] ◇ prompt "테스트 실패..." → ● end

범례: ○start ●end ◇prompt ◆ask ◈subAgent ◊ifElse ❖switch ★skill ⬡mcp ◎flow
```

## 구현 로직

### 1. 합류점 식별

```
incomingCount = {} // 각 노드로 들어오는 연결 수
for connection in connections:
    incomingCount[connection.to]++

mergePoints = nodes where incomingCount > 1
```

### 2. 레이블 생성

```
labels = {}
labelIndex = 'A'
for node in mergePoints:
    if node.type != 'end':
        labels[node.id] = labelIndex++
    else:
        labels[node.id] = 'end'
```

### 3. 트리 순회 (DFS)

```
visited = {}
function traverse(nodeId, indent):
    if visited[nodeId]:
        return "→ [" + labels[nodeId] + "] ⟲"

    visited[nodeId] = true
    node = getNode(nodeId)

    if labels[nodeId]:
        // 합류점은 참조만 출력
        return "→ [" + labels[nodeId] + "]"

    output = formatNode(node)

    if isBranchNode(node):
        for branch in getBranches(node):
            output += indent + formatBranch(branch)
            output += traverse(branch.target, indent + "│   ")
    else:
        nextNode = getNextNode(nodeId)
        if nextNode:
            output += traverse(nextNode, indent)

    return output
```

### 4. 합류점 정의 출력

```
for label, nodeId in labels:
    node = getNode(nodeId)
    output += "├── [" + label + "] " + formatNode(node) + " → " + getNextPath(nodeId)
```

## 에러 처리

- 워크플로우 파일 없음: "워크플로우를 찾을 수 없습니다: <name>"
- JSON 파싱 실패: "워크플로우 파일이 손상되었습니다"
- start 노드 없음: "시작 노드가 없습니다"
