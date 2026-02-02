# workflow-init: GitHub Workflow 초기화

GitHub Issues/Projects 기반 워크플로우를 초기화합니다.

## Purpose

이 명령어는 다음을 수행합니다:
1. 프로젝트 메타데이터 파악 (Milestones, Labels 등)
2. GitHub 레포 연결 (이슈 관리용)
3. PR 워크플로우 모드 선택
4. GitHub 리소스 생성 (Projects, Milestones, Labels, Custom Fields)
5. `.claude/github.json` 캐시 파일 생성

## Workflow

### Phase 1: 프로젝트 메타데이터 분석

**MUST READ** 다음 파일들을 순서대로 읽어서 프로젝트 구조 파악:

1. **루트 CLAUDE.md** (존재하는 경우)
   - Milestones 섹션에서 마일스톤 목록 추출
   - Project Overview, Tech Stack 정보 수집

2. **docs/ 디렉토리** (존재하는 경우)
   - 아키텍처 문서에서 주요 컴포넌트, 단계(Stage) 파악
   - 마일스톤 due date, 우선순위 정보 수집

3. **README.md** (존재하는 경우)
   - 프로젝트 설명, 주요 기능 파악

파악할 정보:
- Project name
- Milestones (이름, 설명, due date, 상태)
- 주요 Labels (feature, bug, epic, enhancement 등)
- Story Points 사용 여부 (Fibonacci: 1, 2, 3, 5, 8, 13, 21)

### Phase 2: GitHub 레포 연결

**gh CLI 인증 확인**:
```bash
gh auth status
```

인증되지 않은 경우:
```bash
gh auth login
```

**레포지토리 연결 옵션**:

사용자에게 질문:
```
GitHub 레포지토리를 어떻게 설정하시겠습니까?

1. 기존 레포 사용 (예: company/project-issues)
2. 새 private 레포 생성
```

옵션 1 선택 시:
```bash
gh repo view OWNER/REPO  # 존재 확인
```

옵션 2 선택 시:
```bash
gh repo create OWNER/REPO --private --description "Issue tracking for PROJECT"
```

### Phase 3: PR 워크플로우 모드 선택

**사용자에게 질문**:

```
소스 코드와 이슈 관리 방식을 선택하세요:

1. **GitHub Full Mode** (권장: 소스 코드도 GitHub)
   - 소스 코드 레포: github.com/owner/repo
   - 이슈/PR: GitHub에서 관리
   - PR 생성 및 머지: GitHub

2. **GitHub Issues-Only Mode** (하이브리드)
   - 소스 코드 레포: GitLab/Bitbucket/Azure DevOps 등
   - 이슈/프로젝트만: GitHub
   - 브랜치/머지: 로컬 git

현재 git remote:
[git remote -v 출력]

선택:
```

선택에 따라 `prMode` 설정:
- Option 1 → `"prMode": "github"`
- Option 2 → `"prMode": "local"`

**prMode별 동작**:

| 기능 | `github` 모드 | `local` 모드 |
|------|--------------|-------------|
| 이슈 생성 | GitHub | GitHub |
| PR 생성 | `gh pr create` | ❌ 비활성화 |
| 이슈 닫기 | PR 머지 시 자동 (`Closes #123`) | 수동 or 로컬 hook |
| 브랜치 연동 | GitHub 브랜치 | 로컬 git 브랜치 |

### Phase 4: GitHub 리소스 생성

**1. Projects 생성**

Phase 1에서 파악한 Milestones를 기반으로 Project(v2) 생성:

```bash
# 프로젝트 생성
gh project create --owner OWNER --title "PROJECT_NAME"

# 프로젝트 ID 가져오기
gh project list --owner OWNER --format json | jq '.projects[] | select(.title=="PROJECT_NAME") | .id'
```

**2. Milestones 생성**

```bash
gh api repos/OWNER/REPO/milestones \
  -f title="M0: Foundation" \
  -f description="공통 인프라, DB 스키마, 인증/인가" \
  -f due_on="2025-12-31T23:59:59Z" \
  -f state="open"
```

각 마일스톤에 대해 반복 실행.

**3. Labels 생성**

기본 라벨 세트:

| Label | Color | Description |
|-------|-------|-------------|
| `epic` | #8B5CF6 | Parent issue for feature breakdown |
| `sub-issue` | #06B6D4 | Sub-task of an epic |
| `feature` | #10B981 | New feature or request |
| `bug` | #EF4444 | Something isn't working |
| `enhancement` | #F59E0B | Improvement to existing feature |
| `documentation` | #6B7280 | Documentation updates |
| `refactor` | #EC4899 | Code refactoring |

```bash
gh label create "epic" --color "8B5CF6" --description "Parent issue for feature breakdown" --repo OWNER/REPO
```

**4. Custom Fields 생성** (GitHub Projects v2)

Story Points 필드 추가:

```bash
# Project ID 필요
gh api graphql -f query='
  mutation($projectId: ID!, $name: String!, $dataType: ProjectFieldType!) {
    createProjectField(input: {
      projectId: $projectId
      name: $name
      dataType: $dataType
    }) {
      projectField { id }
    }
  }
' -f projectId="PROJECT_ID" -f name="Story Points" -f dataType="NUMBER"
```

### Phase 5: github.json 생성

`.claude/github.json` 파일 생성:

```json
{
  "version": "1.0.0",
  "prMode": "github|local",
  "repository": {
    "owner": "company-name",
    "name": "project-name",
    "url": "https://github.com/company-name/project-name",
    "isPrivate": true
  },
  "sourceRepository": {
    "type": "gitlab|bitbucket|azure-devops|github",
    "url": "git@gitlab.company.com:team/project.git"
  },
  "project": {
    "id": "PVT_xxxxx",
    "number": 1,
    "title": "PROJECT_NAME",
    "url": "https://github.com/orgs/OWNER/projects/1"
  },
  "milestones": [
    {
      "number": 1,
      "title": "M0: Foundation",
      "description": "공통 인프라, DB 스키마, 인증/인가",
      "state": "open",
      "dueOn": "2025-12-31T23:59:59Z"
    }
  ],
  "labels": [
    {
      "name": "epic",
      "color": "8B5CF6",
      "description": "Parent issue for feature breakdown"
    }
  ],
  "customFields": {
    "storyPoints": {
      "id": "FIELD_ID",
      "name": "Story Points",
      "type": "NUMBER"
    }
  },
  "syncedAt": "2025-01-15T10:30:00Z"
}
```

**파일 경로**: `./.claude/github.json` (프로젝트 루트 기준)

### Phase 6: 완료 메시지

```
✅ GitHub Workflow 초기화 완료!

📋 생성된 리소스:
- Repository: github.com/OWNER/REPO
- Project: PROJECT_NAME (1개)
- Milestones: 5개
- Labels: 7개
- Custom Fields: Story Points

🔧 모드: [GitHub Full | Issues-Only]

📝 다음 명령어:
- /gh-sync        : GitHub 메타데이터 동기화
- /issue          : Epic + Sub-issues 생성
- /pr             : Pull Request 생성 (GitHub Full 모드만)

💾 캐시 파일: .claude/github.json
```

## Error Handling

1. **gh CLI 미설치**:
   ```
   ❌ gh CLI가 설치되지 않았습니다.
   설치: https://cli.github.com/
   ```

2. **인증 실패**:
   ```
   ❌ GitHub 인증이 필요합니다.
   실행: gh auth login
   ```

3. **레포 권한 부족**:
   ```
   ❌ OWNER/REPO에 대한 admin 권한이 필요합니다.
   Projects, Milestones, Labels를 생성하려면 admin 권한 필요.
   ```

4. **중복 리소스**:
   ```
   ⚠️ 마일스톤 "M0: Foundation"이 이미 존재합니다.
   [1] 건너뛰기  [2] 덮어쓰기  [3] 새 이름으로 생성
   ```

## Usage Examples

### Example 1: 새 프로젝트 초기화

```
User: /workflow-init

Claude:
1. CLAUDE.md 분석...
   - 프로젝트: SEMU-AI
   - 마일스톤: M0~M3 (4개)
   - Story Points: Fibonacci (1,2,3,5,8,13,21)

2. GitHub 레포 선택:
   [1] 기존 레포 사용
   [2] 새 레포 생성

User: 2

Claude: 새 레포 이름: imprun/semu-ai-issues

3. PR 모드 선택:
   [1] GitHub Full (소스도 GitHub)
   [2] Issues-Only (소스는 GitLab 등)

User: 2

Claude:
✅ 초기화 완료!
- Repository: github.com/imprun/semu-ai-issues (Private)
- Mode: Issues-Only
- Milestones: 4개 생성
- Labels: 7개 생성

.claude/github.json 생성됨.
```

### Example 2: 기존 레포 연결

```
User: /workflow-init

Claude: 기존 GitHub 레포를 입력하세요 (OWNER/REPO):

User: company/existing-project

Claude:
✅ company/existing-project 연결됨.
기존 마일스톤 5개 발견.

CLAUDE.md의 마일스톤과 병합하시겠습니까?
[1] 기존 유지  [2] CLAUDE.md로 덮어쓰기  [3] 병합

User: 3

Claude: ✅ 마일스톤 병합 완료 (8개).
```

## Implementation Notes

**Phase별 에이전트 사용**:
- Phase 1 (분석): `Explore` 에이전트로 docs 탐색
- Phase 2-5 (생성): 메인 에이전트에서 `gh` CLI 직접 실행
- 복잡한 GraphQL 쿼리: `api-documenter` 에이전트 활용 가능

**github.json 스키마**:
- `prMode`가 핵심 필드
- 다른 명령어들(`/issue`, `/pr`, `/task`)이 이 값을 읽어서 동작 변경
- `sourceRepository`는 참고용 (실제 git remote와 다를 수 있음)

**로컬 머지 워크플로우** (Issues-Only 모드):
```bash
# 작업 완료 후 로컬 머지
git checkout main
git merge feat/123-api --no-ff

# 이슈 수동 닫기
gh issue close 123 --comment "Merged locally in commit abc1234"
```

향후 개선: `.git/hooks/post-merge`에 자동 이슈 닫기 스크립트 추가 가능.

## Related Commands

- `/gh-sync`: GitHub 메타데이터를 `.claude/github.json`으로 동기화 (기존 명령어 확장)
- `/issue`: Epic + Sub-issues 생성 (`prMode`에 따라 PR 링크 여부 변경)
- `/pr`: Pull Request 생성 (`prMode=local`일 때 비활성화)
- `/task`: 작업 실행 (`prMode=local`일 때 PR 단계 건너뜀)

## See Also

- `.claude/templates/GH_PARENT_ISSUE_TEMPLATE.md`
- `.claude/templates/GH_SUB_ISSUE_TEMPLATE.md`
- `.claude/templates/GH_PR_TEMPLATE.md`
- GitHub CLI 문서: https://cli.github.com/manual/
- GitHub Projects v2 API: https://docs.github.com/en/issues/planning-and-tracking-with-projects/automating-your-project/using-the-api-to-manage-projects
