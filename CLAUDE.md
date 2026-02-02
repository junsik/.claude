# CLAUDE.md

Claude Code (claude.ai/code)가 이 저장소의 코드를 작업할 때 참고하는 가이드 파일입니다.

## Repository Overview

개인 Claude Code 설정 디렉토리로, 명령 템플릿, 특화된 AI 에이전트, GitHub 워크플로우 자동화 도구를 포함합니다.

## Key Architecture

### Command Templates (`commands/`)

GitHub 작업을 위한 정교한 프롬프트 템플릿:

- **`issue.md`**: 하위 이슈 분해가 포함된 다단계 이슈 생성 워크플로우
  - 저장소 컨벤션 및 기존 템플릿 분석
  - 복잡한 기능을 배정 가능한 하위 작업으로 분해
  - 의존성 그래프 및 통합 지점 생성
  - Template path: `~/.claude/templates/GH_SUB_ISSUE_TEMPLATE.md`

- **`pr.md`**: 포괄적인 풀 리퀘스트 생성 워크플로우
  - 기존 PR 템플릿 감지 및 사용 (`.github/pull_request_template.md`)
  - 최근 PR 분석을 통한 컨벤션 파악 (title format, labels, merge strategy)
  - 변경 분류 및 리스크 평가 수행
  - Template path: `~/.claude/templates/GH_PR_TEMPLATE.md`

- **`todos.md`**: 에이전트 오케스트레이션이 포함된 고급 할일 추적
  - 멀티 에이전트 워크플로우 오케스트레이션 지원
  - 단계별 진행 추적 (analysis, implementation, integration)
  - Tree view 및 status dashboard 출력

### Specialized Agents (`agents/` - Git Submodule)

Claude 모델 계층(Haiku/Sonnet/Opus)에 최적화된 도메인별 전문가를 제공합니다:

- Architecture & Design
- Programming Languages
- Infrastructure & Operations
- Security & Quality
- AI/ML & Data
- Documentation & Business

`agents/README.md`에서 다음 내용을 확인하세요:

- 전체 에이전트 카탈로그 및 기능
- 모델 분배 및 선택 기준
- 멀티 에이전트 오케스트레이션 패턴
- 사용 예시 및 모범 사례

### Templates (`templates/`)

GitHub 전용 템플릿:

- `GH_PR_TEMPLATE.md`: 체크리스트가 포함된 표준 PR 구조
- `GH_PARENT_ISSUE_TEMPLATE.md`: 작업 분류 및 스토리 포인트가 포함된 상위 이슈/에픽 형식
- `GH_SUB_ISSUE_TEMPLATE.md`: 범위, 의존성, 인터페이스가 포함된 하위 이슈 형식
- `GH_USER_STORY_TEMPLATE.md`: Gherkin 문법의 BDD 사용자 스토리

## Directory Structure

```
.claude/
├── commands/           # GitHub 작업용 프롬프트 템플릿
├── agents/            # 특화된 AI 서브에이전트 정의
├── templates/         # GitHub PR 및 이슈 템플릿
├── projects/          # 프로젝트 경로별 세션 히스토리 (.jsonl)
├── shell-snapshots/   # 쉘 세션 지속성
├── todos/            # 개별 작업 추적 (.json)
├── statsig/          # 분석 캐시
├── plugins/          # Claude Code 플러그인
└── ide/              # IDE 통합
```

## GitHub Workflow Commands

### Issue Creation

`commands/issue.md` 워크플로우 사용:

1. 템플릿 읽기:
   - Parent issue: `~/.claude/templates/GH_PARENT_ISSUE_TEMPLATE.md`
   - Sub-issues: `~/.claude/templates/GH_SUB_ISSUE_TEMPLATE.md`
2. 저장소 컨벤션 분석 (CONTRIBUTING.md, 기존 이슈)
3. **Claude Code 스킬 및 도구 식별**:
   - `~/.claude/skills/`에서 도메인 전문성을 위한 사용 가능한 스킬 검토
   - 기존 스킬 적용 가능 여부 판단 (독립형 또는 에이전트와 함께)
   - 고도로 기술적/도메인 특화 작업을 위한 새 커스텀 스킬 생성 권장
   - 고품질 특화 출력을 위해 하위 이슈에 스킬 매핑
4. 복잡한 기능을 의존성이 있는 하위 이슈로 분해
5. Fibonacci 스토리 포인트로 복잡도 추정 (1, 2, 3, 5, 8, 13, 21)
6. 의존성 그래프 및 에이전트/팀 배정 생성
7. 작업 분류 테이블 및 통합 지점이 포함된 상위 에픽 생성

**진행 상황 추적:**
- 상위 이슈 설명을 편집하여 상태 업데이트
- "Completed" 열의 체크박스로 하위 이슈 진행 상황 추적
- 단일 진실 공급원 - 상태 업데이트에 댓글 사용 금지

### Pull Request Creation

`commands/pr.md` 워크플로우 사용:

1. `~/.claude/templates/GH_PR_TEMPLATE.md`에서 템플릿 읽기
2. 기존 PR 템플릿 확인 (`.github/pull_request_template.md`)
3. 최근 10-20개 PR 분석하여 제목 형식 및 컨벤션 파악
4. 변경 유형(feature/bugfix/refactor 등) 및 영향 수준 분류
5. 컨텍스트 및 테스트 증거가 포함된 포괄적인 PR 생성

### Todo Management

`commands/todos.md` 워크플로우 사용:

- Initialize: `claude todos --init --project="NAME" --repo="URL"`
- Add issue: `claude todos --add --issue="123" --type="orchestration"`
- Update: `claude todos --update --issue="123" --phase="integration" --progress="75"`
- Status: `claude todos --status [--tree]`

## Agent Orchestration Patterns

**순차 처리 (Sequential Processing):**

```
backend-architect → frontend-developer → test-automator → security-auditor
```

**병렬 실행 (Parallel Execution):**

```
performance-engineer + database-optimizer → 병합된 분석
```

**검증 파이프라인 (Validation Pipeline):**

```
payment-integration → security-auditor → 검증된 구현
```

## Session Management

- **Project sessions**: `projects/`에 `.jsonl` 파일로 추적
- **Shell persistence**: `shell-snapshots/`에 명령 히스토리 스냅샷
- **Todo state**: `todos/`에 개별 JSON 파일로 세션 간 추적
- **Privacy**: 개인 데이터는 gitignore, 공유 템플릿만 버전 관리

## Settings

`settings.json` 포함 내용:

- `alwaysThinkingEnabled: true` - 복잡한 작업을 위한 확장 추론

## Documentation Guidelines

문서 수정 시 변경 이력 형식으로 작성하지 않는다:
- 새로 추가된 내용을 **굵게** 강조하지 않음
- ✅, ➡️ 등 변경 표시용 이모지 사용 금지
- "(NEW)", "(추가됨)", "(변경됨)" 같은 태그 사용 금지
- 문서는 항상 최종 상태로 자연스럽게 작성

## Usage Notes

1. **Command templates**는 특정 파일 경로를 참조 - 항상 템플릿을 먼저 로드
2. **Repository analysis**는 필수 - CONTRIBUTING.md, 기존 이슈/PR 확인
3. **Agent selection**은 작업에 따라 자동으로 진행되거나 명시적 호출 가능
4. **Todo tracking**은 멀티 에이전트 조정을 위해 오케스트레이션 모드 사용
5. **Templates paths**는 고정:
   - Parent issues: `~/.claude/templates/GH_PARENT_ISSUE_TEMPLATE.md`
   - Sub-issues: `~/.claude/templates/GH_SUB_ISSUE_TEMPLATE.md`
   - Pull requests: `~/.claude/templates/GH_PR_TEMPLATE.md`
   - User stories: `~/.claude/templates/GH_USER_STORY_TEMPLATE.md`
