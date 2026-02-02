# Enhanced GitHub Pull Request Creation Template

모범 사례와 프로젝트 컨벤션을 따르는 뛰어난 GitHub 풀 리퀘스트를 생성하는 AI 어시스턴트입니다. 효과적인 코드 리뷰를 촉진하고 프로젝트 품질을 유지하는 포괄적인 PR 콘텐츠를 생성하는 것이 목표입니다.

## Input Parameters

<change_description>

# $ARGUMENTS

</change_description>

<repository_url>

# $REPO_URL

</repository_url>

<branch_info>
Source Branch: #$SOURCE_BRANCH
Target Branch: #$TARGET_BRANCH
</branch_info>

<related_issues>

# $RELATED_ISSUES (optional: comma-separated issue numbers)

</related_issues>

<pull_request_template_path>

# ~/.claude/templates/GH_PR_TEMPLATE.md

</pull_request_template_path>

<reviewers>
#$REVIEWERS (optional: comma-separated GitHub usernames)
</reviewers>

<labels>
#$LABELS (optional: comma-separated labels)
</labels>

<pr_type>

# $PR_TYPE (optional: feature|bugfix|hotfix|refactor|docs|test|chore)

</pr_type>

## Instructions

0. **prMode 체크 (최우선)**:
   - 현재 프로젝트에 `.claude/github.json`이 있으면 읽기
   - **`prMode` 필드 확인**:
     - `"github"`: GitHub PR 생성 (계속 진행)
     - `"local"`: **PR 생성 비활성화** → 에러 메시지 출력 후 종료

   **prMode: "local" 일 때 출력할 메시지**:
   ```
   ❌ PR 생성 불가: Issues-Only Mode

   현재 워크플로우는 Issues-Only Mode(`prMode: "local"`)입니다.
   이 모드에서는 GitHub PR을 사용하지 않고 로컬에서 직접 머지합니다.

   **로컬 머지 워크플로우:**
   1. 로컬 브랜치 작업 완료
   2. 실제 소스 레포로 푸시 (GitLab/Bitbucket 등)
   3. 로컬 머지:
      ```bash
      git checkout main
      git merge feat/<issue>-<name> --no-ff
      git push origin main
      ```
   4. GitHub 이슈 수동 닫기:
      ```bash
      gh issue close <number> --comment "Merged locally in commit <sha>"
      ```

   **GitHub PR을 사용하려면:**
   `/workflow-init`을 실행하여 `prMode`를 `"github"`로 변경하거나,
   `.claude/github.json`의 `prMode` 필드를 수동으로 `"github"`로 수정하세요.
   ```

   메시지 출력 후 **즉시 종료** (PR 생성 진행하지 않음)

1. 지정된 경로에서 풀 리퀘스트 템플릿을 먼저 읽기:
   - 템플릿 로드: ~/.claude/templates/GH_PR_TEMPLATE.md
   - 생성하는 모든 하위 이슈에 이 템플릿 구조 사용

2. 기능 분석 및 로드된 템플릿을 사용하여 하위 이슈 생성

## Comprehensive Analysis Process

### Phase 1: Template Detection & Repository Analysis

**필수 첫 단계**:

1. 다음 위치에서 기존 PR 템플릿 확인:
   - `.github/pull_request_template.md`
   - `.github/PULL_REQUEST_TEMPLATE.md`
   - `.github/PULL_REQUEST_TEMPLATE/` (여러 템플릿이 있는 디렉토리)
   - `docs/pull_request_template.md`
   - 루트 디렉토리 `pull_request_template.md`

2. **템플릿이 있는 경우**:
   - 주요 구조로 사용
   - 모든 섹션을 적절한 콘텐츠로 채우기
   - 정확한 형식 및 섹션 순서 유지
   - 필수 필드 절대 건너뛰지 않기

3. **템플릿이 없는 경우**:
   - 최근 PR을 분석하여 컨벤션 이해
   - 아래의 포괄적인 대체 템플릿 사용

4. Repository convention 분석:
   - 최근 머지된 PR 10-20개를 검토하여 패턴 파악
   - 제목 형식 확인 (conventional commits, 접두사, 이슈 번호)
   - 일반적인 레이블과 사용법 기록
   - 리뷰어 배정 패턴 이해
   - 머지 전략 확인 (squash, merge, rebase)
   - 필수 상태 검사 확인
   - CONTRIBUTING.md 가이드라인 검토
   - CODEOWNERS 파일 확인
   - CI/CD 요구사항 파악

### Phase 2: 변경사항 분류 및 영향도 분석

변경사항을 분석하여 다음을 결정:

1. **변경 타입**:
   - Feature: 새로운 기능 추가
   - Bugfix: 이슈 해결
   - Hotfix: 중요한 프로덕션 수정
   - Refactor: 기능 변경 없는 코드 개선
   - Docs: 문서 업데이트
   - Test: 테스트 추가 또는 수정
   - Chore: 유지보수 작업
   - Performance: 최적화 변경
   - Security: 보안 개선

2. **영향도 수준**:
   - **Critical**: Breaking changes, API 수정, 데이터 마이그레이션
   - **High**: 주요 기능, 대규모 리팩토링, 보안 수정
   - **Medium**: 표준 기능, 버그 수정, 성능 개선
   - **Low**: 문서, 사소한 수정, 코드 스타일 변경

3. **리스크 평가**:
   - Breaking changes 식별
   - 하위 호환성 검사
   - 의존성 업데이트 영향도
   - 데이터베이스 스키마 변경
   - 설정 수정
   - API 계약 변경

### Phase 3: 콘텐츠 생성 전략

분석을 바탕으로 다음을 생성:

1. **제목** (감지된 패턴 따르기):
   - Conventional: `type(scope): description`
   - GitHub: `[TYPE] Description (#issue)`
   - Descriptive: `Add/Fix/Update X for Y`
   - Jira: `[PROJ-123] Description`

2. **설명** (다음을 포함):
   - 명확한 문제 설명
   - 솔루션 접근 방식
   - 구현 세부사항
   - 사용자 영향도
   - 기술적 고려사항

3. **컨텍스트** 섹션:
   - 이 변경이 필요한 이유
   - 고려한 대안 접근 방식
   - 선택한 트레이드오프
   - 향후 고려사항

## 출력 구조

### SECTION 1: Repository 분석 요약

```markdown
## Repository 분석

### 템플릿 감지
- 템플릿 발견: [Yes/No]
- 템플릿 위치: [발견된 경로]
- 템플릿 타입: [standard/custom/multiple]

### 관찰된 컨벤션
- 제목 형식: [감지된 패턴]
- 일반적인 레이블: [목록]
- 리뷰어 패턴: [auto-assign/manual/CODEOWNERS]
- 머지 전략: [squash/merge/rebase]
- 필수 검사: [목록]

### 변경사항 분류
- 타입: [feature/bugfix/refactor/etc]
- 영향도: [critical/high/medium/low]
- 리스크 수준: [high/medium/low]
- Breaking changes: [yes/no]
