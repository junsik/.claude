# Claude Code 명령어 레퍼런스

모든 Claude Code 슬래시 명령어와 워크플로우에 대한 포괄적인 문서입니다.

## 목차

- [개요](#개요)
- [GitHub 워크플로우 명령어](#github-워크플로우-명령어)
  - [/issue - Issue 생성](#issue---github-issue-생성)
  - [/pr - Pull Request 생성](#pr---github-pull-request-생성)
  - [/user-story - BDD User Story](#user-story---bdd-user-story-생성)
- [작업 관리 명령어](#작업-관리-명령어)
  - [/task - Task 오케스트레이션](#task---task-오케스트레이션)
  - [/todos - Todo 추적](#todos---todo-추적)
- [콘텐츠 생성 명령어](#콘텐츠-생성-명령어)
  - [/nlm-research - 연구 생성기](#nlm-research---notebooklm-연구-생성기)
  - [/tiktok-tech - TikTok 스크립트](#tiktok-tech---tiktok-기술-뉴스-다이제스트)
- [개발 명령어](#개발-명령어)
  - [/prompt - Prompt Engineering](#prompt---prompt-engineering)

---

## 개요

이 명령어들은 다단계 작업을 오케스트레이션하고, GitHub와 통합하며, 복잡한 작업을 위해 전문 에이전트를 활용하는 정교한 워크플로우 템플릿입니다.

**주요 기능:**
- 진행 상황 추적이 포함된 다단계 워크플로우
- GitHub CLI 통합
- 템플릿 기반 이슈/PR 생성
- 에이전트 오케스트레이션 지원
- 시맨틱 버저닝 강제
- BDD/Gherkin 문법 지원

---

## GitHub 워크플로우 명령어

### `/issue` - GitHub Issue 생성

지능적인 하위 이슈 분해 및 에이전트 배정을 통해 포괄적인 GitHub 이슈를 생성합니다.

**사용 템플릿:** `~/.claude/templates/GH_PARENT_ISSUE_TEMPLATE.md`, `~/.claude/templates/GH_SUB_ISSUE_TEMPLATE.md`

**사용법:**
```bash
/issue Implement user authentication with OAuth2 support
```

**워크플로우 단계:**

1. **Repository 조사**
   - 프로젝트 구조 및 코드베이스 검토
   - 기존 이슈의 컨벤션 분석
   - CONTRIBUTING.md 및 템플릿 확인
   - 코딩 스타일 및 패턴 분석

2. **Skill & 도구 분석**
   - 사용 가능한 Claude Code skill 목록 확인
   - Skill을 하위 작업에 매핑
   - 전문 작업을 위한 새 skill 생성 권장
   - Skill-작업 매핑 문서화

3. **모범 사례 조사**
   - 최신 GitHub 이슈 모범 사례 검색
   - 효과적인 분해 전략 연구
   - 작업 분류 구조(WBS) 검토

4. **이슈 분류 및 분해**
   - 이슈 타입 결정 (feature/bug/improvement)
   - 논리적 구성 요소 식별
   - 하위 이슈 간 의존성 매핑
   - Fibonacci 스토리 포인트로 추정 (1, 2, 3, 5, 8, 13, 21)

5. **상위 이슈 구조화**
   - 작업 분류 테이블이 포함된 포괄적인 설명 생성
   - 진행 상황 추적을 위한 "Completed" 컬럼 포함
   - 통합 지점 정의
   - 명확한 상태 업데이트 지침 추가

6. **하위 이슈 생성**
   - 전문 에이전트/팀 멤버 배정
   - 명확한 범위 및 인터페이스 정의
   - 의존성 체인 설정
   - 필요한 Claude Code skill 명시

7. **품질 보증**
   - 완전성 및 실행 가능성 검증
   - 컨벤션과의 일치 확인
   - 의존성 및 배정 검증

**스토리 포인트:**
- Fibonacci 시퀀스 사용: 1, 2, 3, 5, 8, 13, 21
- 시간 기반 추정 절대 사용 금지
- 복잡도 및 노력을 나타냄

**예시:**
```bash
/issue Add payment processing with Stripe integration and webhook handling
```

**출력:**
- 작업 분류 테이블이 포함된 상위 이슈
- 명확한 범위를 가진 여러 하위 이슈
- 의존성 그래프
- 에이전트/팀 배정
- 통합 지점 정의

---

### `/pr` - GitHub Pull Request 생성

Repository 컨벤션을 따르는 잘 구조화된 풀 리퀘스트를 포괄적인 분석과 함께 생성합니다.

**사용 템플릿:** `~/.claude/templates/GH_PR_TEMPLATE.md`

**사용법:**
```bash
/pr Add user authentication with OAuth2 and JWT tokens
```

**워크플로우 단계:**

1. **템플릿 감지 및 Repository 분석**
   - `.github/`에서 기존 PR 템플릿 확인
   - 최근 10-20개 PR을 분석하여 컨벤션 파악
   - 제목 형식 패턴 식별
   - CONTRIBUTING.md 및 CODEOWNERS 검토
   - 머지 전략 및 CI/CD 요구사항 확인

2. **변경사항 분류 및 영향도 분석**
   - **변경 타입**: feature, bugfix, hotfix, refactor, docs, test, chore, performance, security
   - **영향도 수준**: Critical, High, Medium, Low
   - **리스크 평가**: Breaking changes, 하위 호환성, 의존성 업데이트

3. **콘텐츠 생성**
   - 감지된 패턴을 따르는 제목
   - 명확한 문제 설명 및 솔루션 접근 방식
   - 구현 세부사항 및 사용자 영향도
   - 기술적 고려사항 및 트레이드오프

**감지된 제목 형식:**
- Conventional: `type(scope): description`
- GitHub: `[TYPE] Description (#issue)`
- Descriptive: `Add/Fix/Update X for Y`
- Jira: `[PROJ-123] Description`

**예시:**
```bash
/pr feat(auth): implement OAuth2 authentication with Google and GitHub providers
```

**출력:**
- Repository 분석 요약
- 모든 섹션이 포함된 완전한 PR 설명
- 테스트 증거 및 검증 단계
- 리스크 평가 및 롤백 계획

---

### `/user-story` - BDD User Story 생성

Gherkin 문법, 시맨틱 버저닝, GitHub Projects 통합을 갖춘 포괄적인 BDD user story를 생성합니다.

**Template Used:** `~/.claude/templates/GH_USER_STORY_TEMPLATE.md`

**Usage:**
```bash
/user-story Create admin dashboard with analytics
```

**Workflow Phases:**

1. **Load Template & Context**
   - Reads user story template
   - Analyzes existing stories for conventions
   - Reviews project documentation

2. **Gather Story Overview**
   - Feature name and persona
   - User goal and benefit
   - Formats as: "As a [persona] I want [goal] So that [benefit]"

3. **Collect Version Information**
   - Version number (X.Y.Z format)
   - Change type (Feature/Bug Fix/Breaking Change)
   - Validates semantic versioning
   - Adds version label (e.g., `v2.1.0`)

4. **Define Gherkin Scenarios**
   - Primary happy path
   - Alternative paths
   - Error/edge cases
   - Proper Given/When/Then syntax

5. **Collect Business Context**
   - Problem statement
   - Priority and user segment
   - Expected usage frequency
   - Business value metrics

6. **Define Technical Context**
   - Dependencies and integration points
   - Data requirements
   - UI/UX needs
   - Accessibility requirements

7. **Specify Testing Strategy**
   - Test coverage requirements
   - Performance criteria
   - Definition of Done

8. **Collect Metadata**
   - Story points estimate
   - Epic linkage
   - Sprint target
   - Labels and related stories

9. **GitHub Project Integration**
   - Creates issue in specified repository
   - Adds to GitHub Project automatically
   - Includes version badge and labels

**Semantic Version Validation:**
```
Valid:   1.0.0, v2.3.4, 0.1.0, 10.20.30
Invalid: 1.2, 01.2.3, 1.2.3.4, abc, 1.2.x
```

**Version Impact:**
- **Feature**: Minor bump (1.0.0 → 1.1.0)
- **Bug Fix**: Patch bump (1.0.0 → 1.0.1)
- **Breaking Change**: Major bump (1.0.0 → 2.0.0)

**Example Gherkin:**
```gherkin
Feature: Admin Analytics Dashboard

  Scenario: View real-time user metrics
    Given I am logged in as an administrator
    And I navigate to the analytics dashboard
    When the page loads
    Then I should see current active users
    And I should see today's registration count
    And the data should refresh every 30 seconds
```

**출력:**
- Gherkin 시나리오가 포함된 완전한 BDD user story
- 버전 배지 및 레이블
- GitHub 이슈 링크
- Project 추가 확인

---

## 작업 관리 명령어

### `/task` - Task 오케스트레이션

에이전트 오케스트레이션, 병렬 실행, 포괄적인 추적을 통한 향상된 작업 해결.

**Usage:**
```bash
/task #123
```

**Workflow Phases:**

1. **Initial Setup & Issue Registration**
   - Fetches issue details via `gh`
   - Creates feature branch
   - Initializes orchestration tracking

2. **Analysis, Planning & Agent Discovery**
   - Comprehensive issue analysis
   - Agent capability assessment
   - Planning matrix creation
   - Complexity estimation

3. **Task Decomposition & Agent Assignment**
   - Intelligent task breakdown
   - Agent assignment with tracking
   - Subtask IDs (ST-001, ST-002, etc.)
   - Dependency mapping

4. **Parallel Agent Execution**
   - Launches agents in parallel
   - Real-time progress monitoring
   - Inter-agent communication
   - Conflict detection and resolution

5. **Integration & Testing**
   - Automated integration
   - Comprehensive test suite
   - Quality gate validation
   - Acceptance criteria verification

6. **Consolidated PR Creation**
   - Aggregates agent contributions
   - Co-author attribution
   - Test results and metrics
   - Quality reports

7. **Review & Completion**
   - Review monitoring
   - Feedback handling
   - Merge and tracking completion
   - Report generation

**Agent Assignment Example:**
```bash
BACKEND_TASK_ID=$(claude agent assign \
  --agent="backend-specialist" \
  --parent-issue="123" \
  --subtask-id="ST-001" \
  --description="Implement REST API endpoints" \
  --priority="high" \
  --dependencies="none" \
  --estimated-time="1d" \
  --success-criteria="All endpoints return correct status codes")
```

**Status Dashboard:**
```
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
└─────────────────────────────────────────────────────────┘
```

**Advanced Commands:**
```bash
# Task decomposition
claude task decompose --issue="123"

# Agent orchestration
claude agent list --available
claude agent recommend --issue="123"
claude agent execute-all --parent-issue="123" --mode="parallel"

# Progress monitoring
claude agent monitor --parent-issue="123" --format="dashboard"
claude todos --status --issue="123" --show-subtasks --format="tree"

# Integration
claude integrate --parent-issue="123" --strategy="incremental"

# Reporting
claude report progress --issue="123"
claude report quality --issue="123"
```

---

### `/todos` - Todo 추적

에이전트 오케스트레이션 지원 및 풍부한 포맷팅을 갖춘 고급 todo 추적.

**Usage:**
```bash
# Initialize project tracking
/todos --init --project="MyProject" --repo="https://github.com/user/repo"

# Add issue
/todos --add --issue="123" --title="Implement Auth" --branch="feature/auth" --type="orchestration" --priority="high"

# Update progress
/todos --update --issue="123" --phase="integration" --progress="75" --agents="backend,frontend,test"

# Add subtask
/todos --add-subtask --parent="123" --id="ST-001" --agent="backend-specialist" --task="Implement API endpoints"

# Update subtask
/todos --update-subtask --parent="123" --id="ST-001" --progress="100" --status="complete"

# View status
/todos --status [--issue="123"] [--tree]

# Complete
/todos --complete --issue="123" --pr="456"
```

**Output Format (todos.md):**
```markdown
# TODOs

> Project: MyProject | Updated: 2024-01-16 14:30:00
> Active: 4 | Review: 2 | Completed: 5

## 🚀 In Progress

### [#123] Implement User Authentication [ORCHESTRATED]

- **Branch**: `feature/auth`
- **Priority**: High | **Progress**: 75%
- **Phase**: Integration

#### Agents (3/5 complete)

- ✅ ST-001: Backend API (backend-specialist)
- ✅ ST-002: Frontend UI (frontend-specialist)
- 🔄 ST-003: Testing (test-specialist) - 60%
- ⏸️ ST-004: Docs (docs-specialist)
- 🔄 ST-000: Coordination (coord-specialist)

#### Tasks

- [x] Analysis complete
- [x] Agents assigned
- [x] Backend implemented
- [ ] Testing (in progress)
- [ ] Documentation
- [ ] Create PR
```

**Tree View:**
```
📁 MyProject (4 active, 2 review, 5 completed)
├── 🚀 In Progress
│   ├── #123: User Authentication [75%]
│   │   ├── ✅ Backend API
│   │   ├── ✅ Frontend UI
│   │   ├── 🔄 Testing (60%)
│   │   └── ⏸️ Documentation
│   └── #124: Memory Leak [60%]
├── 🔄 In Review
│   └── #122: Documentation (1/2)
└── ⏸️ Blocked
    └── #125: Payment Integration
```

**Status Codes:**
- 🚀 In Progress
- 🔄 In Review
- ⏸️ Blocked
- ✅ Completed
- 📋 Planned

**Priority Levels:**
- 🔴 Critical
- 🟠 High
- 🟡 Medium
- 🟢 Low

---

## 콘텐츠 생성 명령어

## 개발 명령어

### `/prompt` - Prompt Engineering

고급 프롬프트 엔지니어링 기법을 사용하여 효과적이고 잘 구조화된 프롬프트를 생성합니다.

**Usage:**
```bash
/prompt task="Create a REST API client" audience="Claude Sonnet" style="technical" format="code"
```

**Parameters:**
- `task` (required): The task or goal
- `audience`: Target (e.g., "Claude Sonnet", "developer")
- `style`: Output style (e.g., "technical", "conversational")
- `format`: Expected format (e.g., "code", "markdown", "json")

**Prompt Engineering Framework:**

1. **Context Analysis**
   - Core objective identification
   - Domain knowledge requirements
   - Edge cases and ambiguities
   - Constraints and requirements

2. **Prompt Structure Design**
   - Role assignment
   - Clear objective statement
   - Context provision
   - Requirements & constraints
   - Examples (when helpful)
   - Output format specification
   - Thinking process (for complex tasks)

3. **Optimization Techniques**
   - Specificity over vagueness
   - Task decomposition
   - Chain-of-thought reasoning
   - Few-shot learning
   - Positive constraints
   - Validation steps
   - XML tags for structure

4. **Quality Checklist**
   - Clear and unambiguous objective
   - Sufficient context
   - Specific requirements
   - Well-defined output format
   - Concise language
   - Edge cases addressed

**Output Template:**
```markdown
# [Prompt Title]

## Role & Context
[Assign role, provide context]

## Objective
[Clear goal statement]

## Requirements
[Specific requirements]

## Constraints
[Limitations and boundaries]

## Output Format
[Detailed specification]

## Examples
[Input/output examples]

## Thinking Process
[Step-by-step reasoning]

## Quality Criteria
[Success evaluation]
```

**Advanced Patterns:**
- Multi-agent pattern
- Validation loop
- Progressive disclosure
- Socratic method
- Meta-prompting

---

## 팁 및 모범 사례

**GitHub 워크플로우:**
- 항상 repository 컨벤션을 먼저 검토
- Fibonacci 스토리 포인트 사용 (1, 2, 3, 5, 8, 13, 21)
- 진행 상황 추적을 위해 상위 이슈 설명 업데이트
- 상태 업데이트에 댓글 사용 금지
- 관련 이슈 및 PR 연결

**작업 관리:**
- 작업 시작 전 추적 초기화
- 진행 상황을 정기적으로 업데이트
- 복잡한 오케스트레이션에는 트리 뷰 사용
- 에이전트 충돌 모니터링
- 완료 보고서 생성

**콘텐츠 생성:**
- 더 나은 출력을 위해 포괄적인 입력 제공
- 날짜 및 소스 포함
- 특정 연구 타입 사용
- 접근성을 위해 오디오 요청

**Prompt Engineering:**
- 대상 및 형식에 대해 구체적으로 명시
- 복잡한 패턴에는 예시 포함
- 구조화를 위해 XML 태그 사용
- 추론을 위해 chain-of-thought 요청

---

## 관련 문서

- 템플릿: `~/.claude/templates/`
- Skill: `~/.claude/skills/`
- 에이전트: `~/.claude/agents/`
- GitHub CLI: `gh help`

