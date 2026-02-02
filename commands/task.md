# 에이전트 오케스트레이션 및 추적을 통한 향상된 작업 해결

## Task: 에이전트 오케스트레이션으로 Issue #$ISSUE_NUMBER 해결

---

## 작업 추적 전략

이 명령어는 **두 가지 추적 시스템**을 병행 사용합니다:

| 시스템 | 용도 | 지속성 |
|--------|------|--------|
| **내장 TodoWrite** | 현재 세션 진행률 시각화 | 세션 내 |
| **Custom todos** | GitHub 이슈 연동, 에이전트 할당 기록 | 영구 저장 |

**규칙:**
- Phase 시작 시 → TodoWrite로 세부 단계 등록
- 에이전트 할당/완료 시 → Custom todos로 persistent 기록
- Phase 완료 시 → 양쪽 모두 업데이트

---

## prMode 확인 (최우선)

**MUST CHECK**: `.claude/github.json`의 `prMode` 필드를 먼저 확인:

```bash
# github.json 읽기
PRMODE=$(cat .claude/github.json | jq -r '.prMode // "github"')
```

**prMode별 워크플로우:**

| prMode | Phase 5 | Phase 6 |
|--------|---------|---------|
| `"github"` | GitHub PR 생성 | PR 리뷰 및 머지 |
| `"local"` | 로컬 머지 안내 | 이슈 수동 닫기 |

**Phase 구조 변경:**
- `prMode: "github"` → Phase 5: "PR 생성", Phase 6: "리뷰 및 완료"
- `prMode: "local"` → Phase 5: "로컬 머지", Phase 6: "이슈 완료"

---

### 초기 설정 및 이슈 등록

```bash
# Get issue details and initialize tracking
gh issue view $ISSUE_NUMBER --json title,body,state,labels,assignees,milestone
ISSUE_TITLE=$(gh issue view $ISSUE_NUMBER --json title -q .title)
ISSUE_LABELS=$(gh issue view $ISSUE_NUMBER --json labels -q '.labels[].name' | tr '\n' ',')
ISSUE_TYPE=$(echo $ISSUE_LABELS | grep -oP '(feat|fix|refactor|docs|test|chore)' | head -1)
BRANCH_NAME="${ISSUE_TYPE:-feat}/$ISSUE_NUMBER-$(echo $ISSUE_TITLE | tr '[:upper:]' '[:lower:]' | tr ' ' '-' | cut -c1-30)"
```

#### 내장 TodoWrite 초기화

Phase 전체 구조를 TodoWrite로 등록:

```
TodoWrite([
  { content: "Phase 1: 이슈 분석 및 에이전트 탐색", status: "in_progress", activeForm: "이슈 분석 중" },
  { content: "Phase 2: 작업 분해 및 에이전트 할당", status: "pending", activeForm: "작업 분해 중" },
  { content: "Phase 3: 에이전트 병렬 실행", status: "pending", activeForm: "에이전트 실행 중" },
  { content: "Phase 4: 통합 및 테스트", status: "pending", activeForm: "통합 테스트 중" },
  { content: "Phase 5: PR 생성", status: "pending", activeForm: "PR 생성 중" },
  { content: "Phase 6: 리뷰 및 완료", status: "pending", activeForm: "리뷰 진행 중" }
])
```

### 작업 환경 선택

AskUserQuestion 도구를 사용하여 사용자에게 질문합니다:

**Question**: "작업 환경을 선택하세요"

| Option | Description |
|--------|-------------|
| 현재 디렉토리 (권장: 싱글 에이전트) | 현재 디렉토리에서 브랜치 생성하여 작업 |
| Worktree (권장: 멀티 에이전트 병렬) | 독립된 worktree 디렉토리 생성하여 작업 |

Based on selection:

**Option 1: Current Directory (Single Agent)**
```bash
git checkout -b $BRANCH_NAME
```

**Option 2: Worktree (Multi-Agent Parallel)**
```bash
# Read worktree config from .claude/github.json
WORKTREE_BASE="~/.worktrees"
WORKTREE_DIR="$WORKTREE_BASE/$(echo $BRANCH_NAME | tr '/' '-')"

# Create worktree with new branch
git worktree add -b $BRANCH_NAME $WORKTREE_DIR origin/main

# Display worktree path for agent to work in
echo "Worktree created: $WORKTREE_DIR"
echo "Agent should work in: $(realpath $WORKTREE_DIR)"
```

After environment setup, continue with tracking:

```bash
# Initialize task orchestration tracking (Custom todos - persistent)
claude todos --add \
  --issue-number="$ISSUE_NUMBER" \
  --branch="$BRANCH_NAME" \
  --type="orchestration" \
  --title="$ISSUE_TITLE" \
  --labels="$ISSUE_LABELS" \
  --status="planning"
```

---

## Phase 1: 분석, 계획 수립 및 에이전트 탐색

### TodoWrite: Phase 1 세부 단계

```
TodoWrite([
  { content: "Phase 1: 이슈 분석 및 에이전트 탐색", status: "in_progress", activeForm: "이슈 분석 중" },
  { content: "1.1 이슈 컨텍스트 분석", status: "in_progress", activeForm: "컨텍스트 분석 중" },
  { content: "1.2 에이전트 탐색 및 역량 평가", status: "pending", activeForm: "에이전트 탐색 중" },
  { content: "Phase 2: 작업 분해 및 에이전트 할당", status: "pending", activeForm: "작업 분해 중" },
  ...
])
```

### 1.1 포괄적 이슈 분석

```bash
# Deep dive into issue context
gh issue view $ISSUE_NUMBER --comments
gh issue view $ISSUE_NUMBER --json linkedPullRequests,closedBy,projectItems

# Analyze related issues and PRs
RELATED_ISSUES=$(gh issue list --label "$(gh issue view $ISSUE_NUMBER --json labels -q '.labels[0].name')" --limit 5)

# Check for existing solutions or patterns
gh search issues "$ISSUE_TITLE" --repo $REPO_URL --limit 10
Analysis Checklist

 Problem scope and boundaries defined
 Acceptance criteria extracted
 Dependencies identified
 Complexity level assessed (simple/medium/complex/epic)
 Related issues/PRs reviewed
 Breaking changes identified
 Integration points mapped

1.2 Agent Discovery & Capability Assessment
bash# Discover available specialist agents
claude agent list --available
claude agent capabilities --match-issue="$ISSUE_NUMBER"

# Get agent recommendations based on issue
claude agent recommend --issue="$ISSUE_NUMBER" --format=table
Agent Planning Matrix
Task ComponentRequired ExpertiseRecommended AgentPriorityDependencies[Component 1][Expertise area][Agent name]HighNone[Component 2][Expertise area][Agent name]MediumComponent 1[Component 3][Expertise area][Agent name]LowComponent 2
bash# Update master tracking with planning complete
claude todos --update \
  --issue-number="$ISSUE_NUMBER" \
  --phase="analysis" \
  --agents-identified="[agent1,agent2,agent3]" \
  --complexity="medium" \
  --estimated-subtasks="5"
## Phase 2: 작업 분해 및 에이전트 할당

### TodoWrite: Phase 2 전환

```
// Phase 1 완료, Phase 2 시작
TodoWrite([
  { content: "Phase 1: 이슈 분석 및 에이전트 탐색", status: "completed", activeForm: "이슈 분석 완료" },
  { content: "Phase 2: 작업 분해 및 에이전트 할당", status: "in_progress", activeForm: "작업 분해 중" },
  { content: "2.1 작업 자동 분해", status: "in_progress", activeForm: "작업 분해 중" },
  { content: "2.2 에이전트 할당", status: "pending", activeForm: "에이전트 할당 중" },
  { content: "Phase 3: 에이전트 병렬 실행", status: "pending", activeForm: "에이전트 실행 중" },
  ...
])
```

### 2.1 지능형 작업 분해
bash# Auto-decompose based on issue analysis
claude task decompose \
  --issue="$ISSUE_NUMBER" \
  --strategy="parallel" \
  --max-subtasks="10" \
  --output="subtasks.json"

# Review and adjust decomposition
claude task review-decomposition --file="subtasks.json"
Decomposition Strategy

 Identify atomic, independent units of work
 Define clear interfaces between components
 Establish success criteria for each subtask
 Set up integration checkpoints
 Create validation criteria

2.2 Agent Assignment with Tracking
For each subtask, create tracked agent assignments:
bash# Template for agent assignment with tracking
claude agent assign \
  --agent="[agent-name]" \
  --parent-issue="$ISSUE_NUMBER" \
  --subtask-id="[ST-001]" \
  --description="[Specific task matching agent expertise]" \
  --priority="[high|medium|low]" \
  --dependencies="[ST-XXX,ST-YYY]" \
  --estimated-time="[2h|1d|3d]" \
  --success-criteria="[Clear acceptance criteria]"

# This creates a sub-todo linked to parent issue
# Returns: SUBTASK_TRACKING_ID
Example Agent Assignments:
bash# Backend API implementation
BACKEND_TASK_ID=$(claude agent assign \
  --agent="backend-specialist" \
  --parent-issue="$ISSUE_NUMBER" \
  --subtask-id="ST-001" \
  --description="Implement REST API endpoints for user authentication" \
  --priority="high" \
  --dependencies="none" \
  --estimated-time="1d" \
  --success-criteria="All endpoints return correct status codes, JWT tokens generated")

# Frontend UI components
FRONTEND_TASK_ID=$(claude agent assign \
  --agent="frontend-specialist" \
  --parent-issue="$ISSUE_NUMBER" \
  --subtask-id="ST-002" \
  --description="Create login and registration UI components" \
  --priority="high" \
  --dependencies="ST-001" \
  --estimated-time="1d" \
  --success-criteria="Responsive design, form validation, API integration")

# Testing suite
TEST_TASK_ID=$(claude agent assign \
  --agent="test-specialist" \
  --parent-issue="$ISSUE_NUMBER" \
  --subtask-id="ST-003" \
  --description="Write comprehensive test suite for auth system" \
  --priority="medium" \
  --dependencies="ST-001,ST-002" \
  --estimated-time="4h" \
  --success-criteria="80% code coverage, all edge cases tested")

# Documentation
DOCS_TASK_ID=$(claude agent assign \
  --agent="docs-specialist" \
  --parent-issue="$ISSUE_NUMBER" \
  --subtask-id="ST-004" \
  --description="Document API endpoints and usage examples" \
  --priority="low" \
  --dependencies="ST-001" \
  --estimated-time="2h" \
  --success-criteria="OpenAPI spec, README updated, examples provided")
2.3 Coordination Agent Assignment
bash# Assign coordination agent for complex multi-agent tasks
COORD_TASK_ID=$(claude agent assign \
  --agent="coordination-specialist" \
  --parent-issue="$ISSUE_NUMBER" \
  --subtask-id="ST-000" \
  --description="Coordinate integration of all components and validate end-to-end functionality" \
  --priority="critical" \
  --dependencies="all" \
  --estimated-time="continuous" \
  --success-criteria="All components integrated, no conflicts, passes E2E tests")

# Update master tracking
claude todos --update \
  --issue-number="$ISSUE_NUMBER" \
  --phase="assigned" \
  --subtasks="$BACKEND_TASK_ID,$FRONTEND_TASK_ID,$TEST_TASK_ID,$DOCS_TASK_ID,$COORD_TASK_ID" \
  --total-subtasks="5" \
  --status="in-progress"
## Phase 3: 실시간 추적을 통한 병렬 에이전트 실행

### TodoWrite: Phase 3 전환

```
// Phase 2 완료, Phase 3 시작 - 동적으로 에이전트별 subtask 추가
TodoWrite([
  { content: "Phase 1: 이슈 분석 및 에이전트 탐색", status: "completed", activeForm: "완료" },
  { content: "Phase 2: 작업 분해 및 에이전트 할당", status: "completed", activeForm: "완료" },
  { content: "Phase 3: 에이전트 병렬 실행", status: "in_progress", activeForm: "에이전트 실행 중" },
  { content: "3.1 Backend API (backend-specialist)", status: "in_progress", activeForm: "Backend 구현 중" },
  { content: "3.2 Frontend UI (frontend-specialist)", status: "pending", activeForm: "Frontend 구현 중" },
  { content: "3.3 Test Suite (test-specialist)", status: "pending", activeForm: "테스트 작성 중" },
  { content: "Phase 4: 통합 및 테스트", status: "pending", activeForm: "통합 테스트 중" },
  ...
])
```

### 3.1 에이전트 실행 시작
bash# Start all assigned agents in parallel
claude agent execute-all --parent-issue="$ISSUE_NUMBER" --mode="parallel"

# Or start specific agents
claude agent execute --task-id="$BACKEND_TASK_ID" --async
claude agent execute --task-id="$FRONTEND_TASK_ID" --async
3.2 Real-time Progress Monitoring
bash# Monitor all agent progress
claude agent monitor --parent-issue="$ISSUE_NUMBER" --format="dashboard"

# Get detailed status
claude todos --status --issue-number="$ISSUE_NUMBER" --show-subtasks --format="tree"

# Output example:
# 📋 Issue #123: Implement User Authentication [IN PROGRESS]
# ├── 🤖 ST-001: Backend API [✓ COMPLETE] (backend-specialist)
# ├── 🤖 ST-002: Frontend UI [🔄 IN PROGRESS - 75%] (frontend-specialist)
# ├── 🤖 ST-003: Test Suite [⏸ WAITING] (test-specialist)
# ├── 🤖 ST-004: Documentation [⏸ WAITING] (docs-specialist)
# └── 🤖 ST-000: Coordination [🔄 ACTIVE] (coordination-specialist)
3.3 Agent Communication & Conflict Resolution
bash# Check for conflicts between agents
claude agent check-conflicts --parent-issue="$ISSUE_NUMBER"

# Resolve conflicts through coordination agent
claude agent resolve-conflict \
  --conflict-id="[CONFLICT_ID]" \
  --strategy="merge|override|negotiate" \
  --coordinator="$COORD_TASK_ID"

# Send message between agents
claude agent message \
  --from="$BACKEND_TASK_ID" \
  --to="$FRONTEND_TASK_ID" \
  --type="interface-update" \
  --content="API endpoint signatures changed, see updated spec"
3.4 Checkpoint Validation
bash# Validate checkpoints as agents complete work
claude task validate-checkpoint \
  --task-id="$BACKEND_TASK_ID" \
  --checkpoint="api-implementation" \
  --criteria="endpoints-functional,auth-working,error-handling"

# Update subtask progress
claude todos --update-subtask \
  --parent-issue="$ISSUE_NUMBER" \
  --subtask-id="$BACKEND_TASK_ID" \
  --progress="100" \
  --status="complete" \
  --output="api-endpoints.json"
## Phase 4: 통합, 테스트 및 품질 보증

### TodoWrite: Phase 4 전환

```
TodoWrite([
  { content: "Phase 1-3", status: "completed", activeForm: "완료" },
  { content: "Phase 4: 통합 및 테스트", status: "in_progress", activeForm: "통합 테스트 중" },
  { content: "4.1 컴포넌트 통합", status: "in_progress", activeForm: "통합 중" },
  { content: "4.2 테스트 실행", status: "pending", activeForm: "테스트 실행 중" },
  { content: "4.3 품질 검증", status: "pending", activeForm: "품질 검증 중" },
  { content: "Phase 5: PR 생성", status: "pending", activeForm: "PR 생성 중" },
  ...
])
```

### 4.1 자동화된 통합
bash# Trigger integration when dependencies are met
claude agent integrate \
  --parent-issue="$ISSUE_NUMBER" \
  --strategy="incremental|big-bang" \
  --validate-interfaces="true"

# Track integration progress
claude todos --update \
  --issue-number="$ISSUE_NUMBER" \
  --phase="integration" \
  --integration-status="merging-components" \
  --conflicts="0"
4.2 Comprehensive Testing
bash# Run integrated test suite
claude test run \
  --scope="all" \
  --parent-issue="$ISSUE_NUMBER" \
  --include-agent-tests="true"

# Validate against acceptance criteria
claude test validate-acceptance \
  --issue="$ISSUE_NUMBER" \
  --criteria-file="acceptance-criteria.json"

# Update test results
claude todos --update \
  --issue-number="$ISSUE_NUMBER" \
  --phase="testing" \
  --test-results="passed" \
  --coverage="87%" \
  --passing-tests="145/145"
4.3 Quality Gate Validation
bash# Check all quality gates
claude quality check \
  --parent-issue="$ISSUE_NUMBER" \
  --gates="tests,coverage,linting,security,performance"

# Generate quality report
claude quality report \
  --issue="$ISSUE_NUMBER" \
  --output="quality-report.md" \
  --include-agent-metrics="true"
## Phase 5: 통합 및 제출

### TodoWrite: Phase 5 전환

**prMode에 따라 다른 Phase 5 구조 사용:**

#### GitHub Mode (`prMode: "github"`)

```
TodoWrite([
  { content: "Phase 1-4", status: "completed", activeForm: "완료" },
  { content: "Phase 5: PR 생성", status: "in_progress", activeForm: "PR 생성 중" },
  { content: "5.1 에이전트 결과물 통합", status: "in_progress", activeForm: "결과물 통합 중" },
  { content: "5.2 PR 생성 및 제출", status: "pending", activeForm: "PR 제출 중" },
  { content: "Phase 6: 리뷰 및 완료", status: "pending", activeForm: "리뷰 진행 중" }
])
```

#### Issues-Only Mode (`prMode: "local"`)

```
TodoWrite([
  { content: "Phase 1-4", status: "completed", activeForm: "완료" },
  { content: "Phase 5: 로컬 머지", status: "in_progress", activeForm: "로컬 머지 중" },
  { content: "5.1 에이전트 결과물 통합", status: "in_progress", activeForm: "결과물 통합 중" },
  { content: "5.2 소스 레포 푸시 및 로컬 머지", status: "pending", activeForm: "머지 중" },
  { content: "Phase 6: 이슈 완료", status: "pending", activeForm: "이슈 닫기 중" }
])
```

---

### 5.1 에이전트 기여 통합 (공통)

```bash
# Collect all agent outputs
claude agent collect-outputs \
  --parent-issue="$ISSUE_NUMBER" \
  --merge-strategy="unified" \
  --output-dir="./integrated"

# Generate consolidated commit
git add .
git commit -m "fix: #$ISSUE_NUMBER - $ISSUE_TITLE

Co-authored-by: backend-specialist <backend@agent>
Co-authored-by: frontend-specialist <frontend@agent>
Co-authored-by: test-specialist <test@agent>
Co-authored-by: docs-specialist <docs@agent>
Coordinated-by: coordination-specialist <coord@agent>"
```

---

### 5.2 제출 (prMode에 따라 분기)

#### GitHub Mode: PR 생성

```bash
# Generate PR with full agent tracking
claude pr create \
  --issue="$ISSUE_NUMBER" \
  --include-agent-reports="true" \
  --include-test-results="true" \
  --include-quality-metrics="true"

PR_NUMBER=$(gh pr view --json number -q .number)

# Update tracking with PR
claude todos --update \
  --issue-number="$ISSUE_NUMBER" \
  --pr-number="$PR_NUMBER" \
  --status="in-review" \
  --phase="review"
```

#### Issues-Only Mode: 로컬 머지

```bash
# 실제 소스 레포로 푸시 (GitLab/Bitbucket 등)
SOURCE_REPO=$(cat .claude/github.json | jq -r '.sourceRepository.url')
echo "Pushing to source repository: $SOURCE_REPO"

git push origin $BRANCH_NAME

# 로컬 머지 안내 출력
cat <<EOF

✅ 브랜치 푸시 완료: $BRANCH_NAME

🔀 로컬 머지 워크플로우:

1. **메인 브랜치로 전환**:
   git checkout main
   git pull origin main

2. **로컬 머지** (no-ff로 머지 커밋 생성):
   git merge $BRANCH_NAME --no-ff -m "Merge branch '$BRANCH_NAME'"

3. **소스 레포로 푸시**:
   git push origin main

4. **이슈 닫기** (Phase 6에서 자동 진행):
   gh issue close $ISSUE_NUMBER --comment "Merged locally in commit \$(git rev-parse HEAD)"

📌 현재 위치: Phase 5 완료, Phase 6으로 진행합니다.
EOF

# Update tracking
claude todos --update \
  --issue-number="$ISSUE_NUMBER" \
  --status="local-merged" \
  --phase="completion"
```
## Phase 6: 완료

### TodoWrite: Phase 6 전환 (최종)

**prMode에 따라 다른 Phase 6 구조 사용:**

#### GitHub Mode (`prMode: "github"`)

```
TodoWrite([
  { content: "Phase 1-5", status: "completed", activeForm: "완료" },
  { content: "Phase 6: 리뷰 및 완료", status: "in_progress", activeForm: "리뷰 진행 중" },
  { content: "6.1 리뷰 피드백 처리", status: "in_progress", activeForm: "피드백 처리 중" },
  { content: "6.2 머지 및 완료", status: "pending", activeForm: "머지 중" }
])
```

#### Issues-Only Mode (`prMode: "local"`)

```
TodoWrite([
  { content: "Phase 1-5", status: "completed", activeForm: "완료" },
  { content: "Phase 6: 이슈 완료", status: "in_progress", activeForm: "이슈 닫기 중" },
  { content: "6.1 이슈 닫기", status: "in_progress", activeForm: "이슈 닫는 중" },
  { content: "6.2 완료 리포트 생성", status: "pending", activeForm: "리포트 생성 중" }
])
```

---

### 6.1 완료 프로세스 (prMode에 따라 분기)

#### GitHub Mode: PR 리뷰

```bash
# Monitor review progress
gh pr view $PR_NUMBER --json reviews,checks

# Handle review feedback
claude agent handle-feedback \
  --pr="$PR_NUMBER" \
  --feedback-type="requested-changes" \
  --assign-to-agent="auto"
```

#### Issues-Only Mode: 이슈 닫기

```bash
# 커밋 SHA 가져오기
COMMIT_SHA=$(git rev-parse HEAD)

# GitHub 이슈 자동 닫기
gh issue close $ISSUE_NUMBER \
  --comment "✅ Merged locally in commit $COMMIT_SHA

**Branch**: $BRANCH_NAME
**Completed**: $(date -u +"%Y-%m-%dT%H:%M:%SZ")

**Agent Contributions**:
- backend-specialist
- frontend-specialist
- test-specialist
- docs-specialist

Merged to main branch in source repository."

echo "✅ Issue #$ISSUE_NUMBER closed successfully"
```

---

### 6.2 완료 및 리포트 생성 (공통)

#### GitHub Mode: PR 머지 후 완료

```bash
# After approval, merge
gh pr merge $PR_NUMBER --squash

# Complete all tracking
claude todos --complete \
  --issue-number="$ISSUE_NUMBER" \
  --pr="$PR_NUMBER" \
  --close-subtasks="true" \
  --generate-report="true"

# Generate completion report
claude task report \
  --issue="$ISSUE_NUMBER" \
  --include-metrics="true" \
  --output="completion-report.md"
```

#### Issues-Only Mode: 완료 리포트

```bash
# Complete all tracking
claude todos --complete \
  --issue-number="$ISSUE_NUMBER" \
  --close-subtasks="true" \
  --generate-report="true"

# Generate completion report
claude task report \
  --issue="$ISSUE_NUMBER" \
  --include-metrics="true" \
  --output="completion-report.md"

echo "✅ Task completed successfully in Issues-Only Mode"
```
Advanced Commands Reference
Task Orchestration Commands
bash# Task decomposition
claude task decompose --issue="$ISSUE_NUMBER"                    # Auto-decompose issue
claude task validate --parent-issue="$ISSUE_NUMBER"              # Validate all subtasks
claude task dependencies --issue="$ISSUE_NUMBER" --visualize    # Show dependency graph

# Agent orchestration
claude agent list --available                                    # List all agents
claude agent recommend --issue="$ISSUE_NUMBER"                   # Get recommendations
claude agent assign --agent="name" --subtask="description"       # Assign work
claude agent execute-all --parent-issue="$ISSUE_NUMBER"         # Start execution
claude agent monitor --parent-issue="$ISSUE_NUMBER"             # Monitor progress
claude agent collect-outputs --parent-issue="$ISSUE_NUMBER"     # Gather results
Tracking Commands
bash# Master issue tracking
claude todos --add --issue-number="$ISSUE_NUMBER"               # Initialize tracking
claude todos --status --issue-number="$ISSUE_NUMBER"            # Check status
claude todos --update --issue-number="$ISSUE_NUMBER"            # Update progress
claude todos --complete --issue-number="$ISSUE_NUMBER"          # Mark complete

# Subtask tracking
claude todos --add-subtask --parent="$ISSUE_NUMBER"             # Add subtask
claude todos --update-subtask --id="$SUBTASK_ID"                # Update subtask
claude todos --list-subtasks --parent="$ISSUE_NUMBER"           # List all subtasks
claude todos --status --show-subtasks --format="tree"           # Tree view
Integration Commands
bash# Integration management
claude integrate --parent-issue="$ISSUE_NUMBER"                  # Start integration
claude integrate validate --issue="$ISSUE_NUMBER"                # Validate integration
claude integrate conflicts --resolve --issue="$ISSUE_NUMBER"     # Resolve conflicts
Reporting Commands
bash# Generate reports
claude report progress --issue="$ISSUE_NUMBER"                   # Progress report
claude report agents --issue="$ISSUE_NUMBER"                     # Agent performance
claude report quality --issue="$ISSUE_NUMBER"                    # Quality metrics
claude report timeline --issue="$ISSUE_NUMBER"                   # Timeline view
Integration with Other Commands
1. GitHub Issue Creation (from issue template)
bash# Create parent issue and auto-decompose
claude issue create --template="feature" --auto-decompose="true"
ISSUE_NUMBER=$(gh issue list --limit 1 --json number -q '.[0].number')

# Start task orchestration immediately
claude task start --issue="$ISSUE_NUMBER" --auto-assign-agents="true"
2. Sub-Issue Creation (from sub-issue template)
bash# Create sub-issues from decomposition
claude issue create-sub \
  --parent="$ISSUE_NUMBER" \
  --from-decomposition="subtasks.json" \
  --assign-agents="true"
3. PR Creation (from PR template)
bash# Create PR with agent attribution
claude pr create \
  --issue="$ISSUE_NUMBER" \
  --template="comprehensive" \
  --include-agent-work="true" \
  --co-authors="all-agents"
Workflow Examples
Example 1: Simple Bug Fix
bash# Quick single-agent task
claude task quick \
  --issue="456" \
  --agent="bugfix-specialist" \
  --auto-complete="true"
Example 2: Complex Feature
bash# Full orchestration for complex feature
claude task orchestrate \
  --issue="789" \
  --complexity="high" \
  --agents="auto" \
  --parallel="true" \
  --monitor="dashboard"
Example 3: Epic with Multiple Issues
bash# Handle epic with sub-issues
claude task epic \
  --parent-issue="1000" \
  --create-sub-issues="true" \
  --assign-agents="true" \
  --coordinate="true"
Configuration File
Create .claude/task-config.yaml:
yamlorchestration:
  default_strategy: parallel
  max_agents: 10
  conflict_resolution: automatic
  checkpoint_validation: strict

tracking:
  update_frequency: on-change
  include_subtasks: true
  generate_reports: true

agents:
  auto_discover: true
  auto_assign: true
  load_balancing: enabled
  communication: websocket

integration:
  strategy: incremental
  validate_interfaces: true
  run_tests: always

quality:
  gates:
    - tests
    - coverage
    - linting
    - security
  minimum_coverage: 80
  required_approvals: 2
Status Dashboard
bash# Real-time dashboard
claude dashboard --issue="$ISSUE_NUMBER"

# Output:
┌─────────────────────────────────────────────────────────┐
│ Issue #123: Implement User Authentication              │
├─────────────────────────────────────────────────────────┤
│ Progress: ████████████████░░░░░░ 75%                   │
│ Phase: Integration                                      │
│ Agents: 5 active, 2 complete, 0 failed                 │
├─────────────────────────────────────────────────────────┤
│ Subtasks:                                              │
│ ✅ Backend API        (backend-specialist)    100%     │
│ ✅ Frontend UI        (frontend-specialist)   100%     │
│ 🔄 Testing           (test-specialist)       60%      │
│ ⏸️  Documentation     (docs-specialist)       0%       │
│ 🔄 Coordination      (coord-specialist)      Active    │
├─────────────────────────────────────────────────────────┤
│ Metrics:                                               │
│ • Time Elapsed: 4h 32m                                 │
│ • Est. Remaining: 2h 15m                               │
│ • Code Coverage: 84%                                   │
│ • Tests: 142/145 passing                               │
└─────────────────────────────────────────────────────────┘
---

## 이중 추적 시스템 동기화 규칙

### 언제 무엇을 업데이트하는가

| 이벤트 | 내장 TodoWrite | Custom todos |
|--------|---------------|--------------|
| Phase 시작 | 세부 단계 등록 | phase 상태 업데이트 |
| Subtask 완료 | 해당 항목 completed | subtask progress 100% |
| 에이전트 할당 | 동적으로 항목 추가 | agent 정보 기록 |
| Phase 완료 | 다음 Phase로 전환 | phase 상태 업데이트 |
| 전체 완료 | 모든 항목 completed | status: completed + report |

### 동기화 예시

```
// 에이전트 작업 완료 시
1. TodoWrite: "3.1 Backend API" → completed
2. Custom todos: --update-subtask --id="ST-001" --progress="100" --status="complete"

// Phase 전환 시
1. TodoWrite: 현재 Phase completed, 다음 Phase in_progress
2. Custom todos: --update --phase="integration"
```

### 우선순위 규칙

1. **사용자 가시성**: 내장 TodoWrite (실시간 진행률 표시)
2. **영구 기록**: Custom todos (세션 간 유지, GitHub 연동)
3. **충돌 시**: Custom todos의 상태가 정확한 소스 (Source of Truth)

---

## 요약

이 향상된 task 명령어는 다음을 제공합니다:

### 이중 추적 시스템 (NEW)

1. **내장 TodoWrite**: 세션 내 실시간 진행률 시각화
2. **Custom todos**: GitHub 이슈 연동 + 영구 상태 저장

### 핵심 기능

- **Comprehensive Agent Tracking**: Every agent assignment is tracked as a subtask
- **Hierarchical Todo System**: Parent issue with linked subtasks for each agent
- **Real-time Monitoring**: Dashboard and progress tracking for all agents
- **Conflict Resolution**: Automated detection and resolution of agent conflicts
- **Integration Management**: Coordinated integration of agent outputs
- **Quality Gates**: Automated validation at each phase
- **Full Command Integration**: Seamless work with issue, sub-issue, and PR commands
- **Detailed Reporting**: Progress, quality, and completion reports
- **Parallel Execution**: Efficient parallel agent execution with dependency management
- **Communication Protocol**: Inter-agent messaging and coordination

### 주요 장점

1. **완전한 에이전트 추적** 이중 todos 시스템과 통합
2. **Hierarchical task management** with parent/subtask relationships
3. **Real-time monitoring** via 내장 TodoWrite (사용자 가시성)
4. **Persistent state** via Custom todos (세션 간 유지)
5. **Full integration** with issue and PR creation commands
6. **Comprehensive reporting** at every phase
7. **Dashboard visualization** for status monitoring
8. **Quality gates** and validation checkpoints
9. **Inter-agent communication** protocols
10. **Configurable orchestration** strategies

The system now provides end-to-end traceability from issue analysis through agent delegation to final PR merge, with both real-time visibility and persistent tracking.
