# GitHub Issue Creation Template with Sub-Issue Management

잘 구조화된 GitHub 이슈를 생성하는 AI 어시스턴트입니다. 기능 요청, 버그 리포트, 개선 아이디어를 위한 이슈를 작성하며, 복잡한 이슈를 팀 멤버나 에이전트에게 배정 가능한 하위 이슈로 분해하는 기능을 제공합니다.

<feature_description>
$ARGUMENTS
(파일 경로가 전달되면 해당 파일을 구현 계획으로 읽어 Issue 분해의 기초로 사용)
</feature_description>

## Instructions

0. **캐시된 프로젝트 메타데이터 먼저 확인:**
   - 현재 프로젝트에 `.claude/github.json`이 있으면 읽기
   - 캐시된 `milestones`, `labels`, `project` 필드 및 `conventions` 사용
   - 반복적인 GitHub API 호출 불필요
   - 캐시가 없거나 오래된 경우 Phase 1 리서치로 대체

1. 필수 템플릿 먼저 읽기:
   - Parent issue: ~/.claude/templates/GH_PARENT_ISSUE_TEMPLATE.md
   - Sub-issues: ~/.claude/templates/GH_SUB_ISSUE_TEMPLATE.md
   - 상위 이슈 및 모든 하위 이슈 생성 시 이 템플릿 구조 사용

2. 현재 저장소 컨텍스트 분석 (이미 프로젝트 디렉토리에 있음)

3. Claude Code 스킬 및 도구 식별:
   - `~/.claude/`에서 적용 가능한 도메인 전문성을 위한 사용 가능한 스킬 검토
   - 기존 스킬을 독립적으로 또는 에이전트와 함께 사용할 수 있는지 판단
   - 새로운 커스텀 스킬이 필요할 수 있는 고도로 기술적/도메인 특화 작업 식별
   - 고품질 출력이 필요한 특화 워크플로우를 위한 스킬 생성 권장

4. 기능 분해 및 로드된 템플릿을 사용하여 이슈 생성

5. 추정을 위해 스토리 포인트 사용:
   - Fibonacci 시퀀스 적용: 1, 2, 3, 5, 8, 13, 21
   - 시간 기반 추정(시간/일) 절대 사용 금지
   - 스토리 포인트는 다른 작업 대비 복잡도 및 노력을 나타냄

## Issue Management Policy

### Issue Type Connection Rules

| Issue Type | Milestone | Project | Start Date | End Date | Issue Type 필드 | Label |
|------------|:---------:|:-------:|:----------:|:--------:|:---------------:|:-----:|
| User Story | ❌ | ❌ | - | - | - | - |
| Epic (Parent) | ✅ | ✅ | 생성일 | Story Points 기반 | feat/docs/etc | epic |
| Sub-issue | ❌ | ❌ | - | - | feat/fix/etc | - |

**Epic 구분**: `epic` label로 구분, Issue Type은 작업 유형(feat, docs 등) 선택
**Epic만 Project/Milestone 연결**: Sub-issue는 Epic에 종속되므로 별도 연결 불필요.

### Milestone 처리

1. `.claude/github.json`에서 기존 마일스톤 목록 확인
2. 적합한 마일스톤이 있으면 선택
3. **적합한 마일스톤이 없으면**: AskUserQuestion으로 새 마일스톤 생성 여부 확인
   - 생성 시 제목과 Due Date 입력받아 `gh api` 로 생성

### Project 연결 (Epic만)

Epic을 Project에 추가할 때 필수 필드:
- **Start Date**: 생성일
- **End Date**: 마일스톤 Due Date
- **Issue Type**: `Epic`

### Issue Closing Policy

| Issue Type | Closing Method |
|------------|---------------|
| **Sub-issue** | PR 본문에 `Closes #번호` → 머지 시 자동 닫힘 |
| **Epic** | 모든 Sub-issue 완료 후 수동: `gh issue close <번호> --reason completed` |

**PR 필수 규칙:**
- 커밋 메시지에 이슈 번호는 권장 (강제 아님)
- PR 본문에 `Closes #이슈번호` **필수**
- PR 템플릿 체크리스트로 확인

## TODO List

작업을 체계적으로 완료하기 위해 다음 단계를 따르세요:

### Phase 1: Repository Research

- [ ] 현재 프로젝트 구조 및 코드베이스 검토
- [ ] `gh` CLI를 사용하여 기존 이슈 검토 및 패턴/컨벤션 이해
- [ ] CONTRIBUTING.md, ISSUE_TEMPLATE.md 또는 .github/ISSUE_TEMPLATE/ 파일 찾기
- [ ] 프로젝트의 코딩 스타일 및 명명 규칙 식별
- [ ] 이슈 제출을 위한 특정 요구사항 기록
- [ ] README.md에서 프로젝트 컨텍스트 및 기여 가이드라인 검토
- [ ] `gh`를 사용하여 labels, milestones, project boards 확인
- [ ] 프로젝트가 이슈 의존성 및 하위 작업을 처리하는 방식 분석
- [ ] 프로젝트가 GitHub Projects 또는 유사한 작업 추적 도구를 사용하는지 확인
- [ ] `gh api`를 사용하여 기여자 및 전문 분야 식별

### Phase 2: Skill & Tooling Analysis

- [ ] `~/.claude/skills/` 디렉토리에서 사용 가능한 Claude Code 스킬 나열
- [ ] 도메인 관련성을 위해 스킬 설명 및 기능 검토
- [ ] 기능 요구사항에 적용 가능한 기존 스킬 식별
- [ ] 스킬을 독립적으로 사용할지 에이전트와 결합할지 결정
- [ ] 각 하위 작업의 기술적 복잡도 및 도메인 특수성 분석
- [ ] 커스텀 스킬이 출력 품질을 향상시킬 수 있는 격차 식별
- [ ] 고도로 특화된 워크플로우(예: 재무 모델링, 법적 준수, 의료 데이터 처리)를 위한 새 스킬 생성 권장
- [ ] 이슈 분해에 스킬-작업 매핑 문서화

### Phase 3: Best Practices Research

- [ ] GitHub 이슈 작성의 현재 모범 사례 검색
- [ ] 명확성, 완전성 및 실행 가능성 원칙에 집중
- [ ] 영감을 위해 인기 프로젝트의 잘 작성된 이슈 예시 찾기
- [ ] 효과적인 이슈 템플릿 및 구조 조사
- [ ] 재현 가능한 단계 및 명확한 인수 조건의 중요성 이해
- [ ] 이슈 분해 및 작업 의존성에 대한 모범 사례 조사
- [ ] 소프트웨어 프로젝트를 위한 효과적인 작업 분류 구조(WBS) 연구
- [ ] GitHub의 작업 목록 문법 및 이슈 연결 메커니즘 학습

### Phase 4: Issue Classification & Decomposition

- [ ] 기능 요청, 버그 리포트 또는 개선인지 판단
- [ ] 따라야 할 적절한 이슈 템플릿 식별 (사용 가능한 경우)
- [ ] 저장소 컨벤션에 따라 관련 라벨 선택
- [ ] 우선순위 및 복잡도 수준 평가
- [ ] 이슈를 하위 이슈로 분해해야 하는지 분석
- [ ] 독립적으로 작업할 수 있는 논리적 컴포넌트 또는 모듈 식별
- [ ] 잠재적 하위 이슈 간 의존성 결정
- [ ] Fibonacci 시퀀스(1, 2, 3, 5, 8, 13, 21)를 사용하여 각 하위 작업의 스토리 포인트 추정
- [ ] 전문성에 따라 하위 이슈를 적절한 팀 멤버/에이전트에게 매핑

### Phase 5: Parent Issue Structure Creation

- [ ] 명확하고 설명적인 제목 작성 (50-72자 권장)
- [ ] 다음 구조에 따라 포괄적인 설명 작성:
- [ ] **Summary**: 요청/이슈의 간략한 개요
- [ ] **Problem Statement**: 어떤 문제를 해결하는가?
- [ ] **Proposed Solution**: 요청된 기능/수정에 대한 상세 설명
- [ ] **Task Breakdown**: 진행 상황 추적을 위한 "Completed" 열이 포함된 하위 이슈 테이블
- [ ] **Status Update Instructions**: 댓글이 아닌 이슈 설명에서 완료 상태를 업데이트하라는 명확한 메모 포함
- [ ] **Acceptance Criteria**: 완료를 위한 명확하고 테스트 가능한 조건
- [ ] **Additional Context**: 스크린샷, 예시 또는 관련 이슈
- [ ] **Implementation Notes**: 기술적 고려사항 (해당하는 경우)
- [ ] 하위 이슈에 연결되는 작업 추적 섹션 생성
- [ ] 하위 이슈 간 통합 지점 정의

### Phase 6: Sub-Issue Creation

- [ ] 식별된 각 하위 작업에 대해 상세한 하위 이슈 구조 생성
- [ ] 전문 분야와 함께 적절한 특화 에이전트/팀 멤버 배정
- [ ] Fibonacci 시퀀스를 사용하여 각 하위 이슈에 스토리 포인트 포함
- [ ] 각 하위 이슈에 대한 명확한 범위 경계 설정
- [ ] 하위 이슈 간 인터페이스 및 핸드오프 지점 정의
- [ ] 의존성 체인 및 차단 관계 설정
- [ ] 하위 이슈 완료를 위한 타임라인/로드맵 생성
- [ ] 컴포넌트 간 통합 테스트 요구사항 정의
- [ ] 에이전트 간 전환을 위한 핸드오프 체크리스트 문서화
- [ ] 각 하위 이슈에 필요한 Claude Code 스킬 명시 (해당하는 경우)
- [ ] 도메인 특화 작업을 위해 새 커스텀 스킬을 생성해야 하는 위치 기록

### Phase 7: Quality Assurance

- [ ] 상위 이슈가 실행 가능하고 구체적인지 확인
- [ ] 모든 필요한 정보가 포함되어 있는지 검증
- [ ] 적절한 형식 및 가독성 확인
- [ ] 프로젝트 컨벤션과의 정렬 확인
- [ ] 적절한 라벨, 담당자 및 마일스톤 추가
- [ ] 하위 이슈 의존성이 올바르게 매핑되었는지 검증
- [ ] 분해에서 중요한 작업이 누락되지 않았는지 확인
- [ ] 팀 멤버 배정이 전문성과 일치하는지 검증
- [ ] 통합 지점이 잘 정의되어 있는지 확인

## Output

Create the parent issue and all sub-issues following the loaded templates from:
- `~/.claude/templates/GH_PARENT_ISSUE_TEMPLATE.md`
- `~/.claude/templates/GH_SUB_ISSUE_TEMPLATE.md`

### Epic 생성 (Milestone + Project 연결)

```bash
# 1. 마일스톤 확인 및 Due Date 가져오기
MILESTONE_TITLE="<milestone title>"
MILESTONE_DUE=$(gh api repos/{owner}/{repo}/milestones \
  --jq ".[] | select(.title == \"$MILESTONE_TITLE\") | .due_on")

# 2. Epic 생성 (epic label + 작업 유형 label 포함)
EPIC_URL=$(gh issue create \
  --title "epic: <description>" \
  --body "[epic body]" \
  --label "epic,<type:feat|docs|etc>,<other labels>" \
  --milestone "$MILESTONE_TITLE" \
  | grep -oP 'https://[^\s]+')

# 3. Project에 추가
ITEM_ID=$(gh project item-add 18 \
  --owner imprun \
  --url "$EPIC_URL" --format json | jq -r '.id')

# 4. Issue Type = 작업 유형 (feat, docs, refactor 등)
gh project item-edit \
  --project-id PVT_kwDODjwv6s4BN9Jx \
  --id $ITEM_ID \
  --field-id PVTSSF_lADODjwv6s4BN9Jxzg8zXFw \
  --single-select-option-id <feat:10ec1f73|docs:84e36a99|etc>

# 5. Start Date = 오늘
gh project item-edit \
  --project-id PVT_kwDODjwv6s4BN9Jx \
  --id $ITEM_ID \
  --field-id PVTF_lADODjwv6s4BN9Jxzg8zXHI \
  --date "$(date +%Y-%m-%d)"

# 6. End Date = Story Points 기반 계산
# Sub-issue story points 합산 (예: 1+2+3+5 = 11 points)
TOTAL_POINTS=<sum of sub-issue story points>
# 1 story point = 0.5일 가정 (조정 가능)
DAYS=$(echo "scale=0; $TOTAL_POINTS * 0.5 / 1" | bc)
END_DATE=$(date -d "+${DAYS} days" +%Y-%m-%d)

gh project item-edit \
  --project-id PVT_kwDODjwv6s4BN9Jx \
  --id $ITEM_ID \
  --field-id PVTF_lADODjwv6s4BN9Jxzg8zXH4 \
  --date "$END_DATE"
```

### Sub-issue 생성 (Milestone/Project 연결 없음)

```bash
# Epic의 GraphQL Node ID 가져오기
EPIC_NODE_ID=$(gh api repos/{owner}/{repo}/issues/{epic_number} --jq '.node_id')

# Sub-issue 생성
SUB_ISSUE_URL=$(gh issue create \
  --title "feat: <sub-issue description>" \
  --body "[sub-issue body]" \
  --label "<labels>" \
  | grep -oP 'https://[^\s]+')

SUB_ISSUE_NUMBER=$(echo $SUB_ISSUE_URL | grep -oP '\d+$')

# Sub-issue의 Node ID 가져오기
SUB_ISSUE_NODE_ID=$(gh api repos/{owner}/{repo}/issues/$SUB_ISSUE_NUMBER --jq '.node_id')

# Parent Relationship 설정
gh api graphql -f query='
  mutation($issueId: ID!, $parentId: ID!) {
    updateIssue(input: {id: $issueId, parentIssueId: $parentId}) {
      issue {
        id
        number
        title
      }
    }
  }' -f issueId="$SUB_ISSUE_NODE_ID" -f parentId="$EPIC_NODE_ID"
```

**Parent Relationship**: GitHub의 issue relationships 기능으로 Epic과 Sub-issue를 자동 연결
