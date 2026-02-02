---
description: Gherkin 문법으로 BDD user story 생성 (사양만, 프로젝트/마일스톤 연결 없음)
---

Gherkin 문법을 사용하여 포괄적인 BDD (Behavior-Driven Development) user story를 생성합니다. 다단계 워크플로우를 따라 필요한 모든 정보를 수집하고 GitHub에 완전한 user story 사양을 생성하세요.

**참고:** User story는 사양이지 실행 가능한 작업 항목이 아닙니다. User story 생성 후 `/issue` 명령어를 사용하여 Epic + 하위 이슈로 분해하고 마일스톤 및 프로젝트 로드맵에 연결하세요.

## Phase 1: 템플릿 로드 및 컨텍스트 이해

먼저 user story 템플릿을 읽습니다:

```bash
cat ~/.claude/templates/GH_USER_STORY_TEMPLATE.md
```

Repository에서 작업하는 경우 기존 컨텍스트 분석:
- 컨벤션을 이해하기 위해 기존 user story 또는 이슈 확인
- CONTRIBUTING.md 또는 프로젝트 문서 검토
- 레이블링 패턴을 위한 최근 이슈 검토

## Phase 2: 스토리 개요 수집

AskUserQuestion 도구를 사용하여 핵심 user story 정보를 수집합니다:

**질문 항목:**
1. 기능 또는 기능성의 이름은 무엇인가요?
2. 사용자 페르소나는 누구인가요? (예: "관리자", "최종 사용자", "개발자")
3. 사용자가 달성하고자 하는 것은 무엇인가요?
4. 사용자가 받을 혜택 또는 가치는 무엇인가요?

**스토리 형식:**
```
As a [페르소나]
I want [목표]
So that [혜택]
```

## Phase 2.5: 버전 정보 수집

AskUserQuestion 도구를 사용하여 시맨틱 버저닝 세부 정보를 수집합니다:

**질문 항목:**
1. 이것이 도입될 버전은 무엇인가요? (형식: X.Y.Z 또는 vX.Y.Z)
2. 이것은 어떤 유형의 변경인가요?
   - **Feature**: 새로운 기능 (minor 버전 증가: 1.0.0 → 1.1.0)
   - **Bug Fix**: 기존 기능 수정 (patch 버전 증가: 1.0.0 → 1.0.1)
   - **Breaking Change**: 사용자 조치 필요 (major 버전 증가: 1.0.0 → 2.0.0)

**Semantic Version Validation:**
- Validate the version follows semver format: `MAJOR.MINOR.PATCH`
- Each component must be a non-negative integer
- No leading zeros (except 0 itself)
- Optional 'v' prefix is acceptable (e.g., v1.2.0 or 1.2.0)

**Validation Rules:**
```
Valid:   1.0.0, v2.3.4, 0.1.0, 10.20.30
Invalid: 1.2, 01.2.3, 1.2.3.4, abc, 1.2.x
```

**If validation fails:**
- Show the invalid format with clear error message
- Explain the semver format requirements
- Ask the user to provide a valid version number
- Suggest the appropriate version bump based on change type

**Version Impact Analysis:**
- For **Breaking Changes**: Explain what makes this breaking and why it requires a major version bump
- For **Features**: Describe the new capability being added
- For **Bug Fixes**: Explain what issue is being resolved

**레이블에 추가:**
버전 레이블을 다음 형식으로 포함: `v[X.Y.Z]` (예: `v2.1.0`)

## Phase 3: Gherkin 시나리오 정의

AskUserQuestion 도구를 사용하여 상세한 Gherkin 시나리오를 생성합니다. 최소 3개의 시나리오가 필요합니다:

**시나리오 1: 주요 성공 경로**
질문: "주요 성공 시나리오를 단계별로 설명해주세요"
- 초기 상태는 무엇인가요? (Given)
- 사용자가 취하는 행동은 무엇인가요? (When)
- 무엇이 발생해야 하나요? (Then)

**시나리오 2: 대안 경로**
질문: "대안이 되는 유효한 경로 또는 변형을 설명해주세요"
- 다른 시작 조건 또는 행동
- 다르지만 여전히 성공적인 결과

**시나리오 3: 오류/엣지 케이스**
질문: "오류 시나리오 또는 엣지 케이스를 설명해주세요"
- 유효하지 않은 입력 또는 조건
- 오류는 어떻게 처리되어야 하나요?
- 사용자는 어떤 피드백을 받아야 하나요?

**추가 시나리오 (해당하는 경우)**
질문: "다룰 시나리오가 더 있나요?" (검증, 권한, 데이터 경계 등)

Format each scenario in proper Gherkin syntax:
```gherkin
Feature: [Feature name]

  Scenario: [Descriptive scenario name]
    Given [precondition]
    And [additional context]
    When [action]
    And [additional action]
    Then [expected outcome]
    And [additional verification]
```

## Phase 4: 비즈니스 컨텍스트 수집

AskUserQuestion 도구를 사용하여 다음을 수집합니다:

1. **문제 설명**: 이것이 해결하는 문제는 무엇인가요?
2. **우선순위**: High, Medium, 또는 Low?
3. **사용자 세그먼트**: 어떤 사용자가 영향을 받나요?
4. **예상 사용 빈도**: 얼마나 자주 사용될 것인가요?
5. **비즈니스 가치**: 수익 영향, 비용 절감, 사용자 만족도 지표?

## Phase 5: 기술적 컨텍스트 정의

AskUserQuestion 도구를 사용하여 다음을 식별합니다:

1. **의존성**: 먼저 무엇이 준비되어야 하나요?
   - 다른 기능, API, 서비스 또는 인프라
2. **통합 지점**: 어떤 시스템/구성 요소와 상호 작용하나요?
3. **데이터 요구사항**: 어떤 데이터 엔티티 또는 필드가 필요한가요?
4. **UI/UX 요구사항**: 디자인 요구사항, 와이어프레임 또는 목업이 있나요?
5. **접근성 요구사항**: 표준 준수를 넘어서는 특정 접근성 요구사항이 있나요?

## Phase 6: 테스트 전략 명시

AskUserQuestion 도구를 사용하여 다음을 정의합니다:

1. **테스트 커버리지**: 어떤 특정 테스트 케이스가 필요한가요?
   - Unit tests
   - Integration tests
   - E2E tests (Gherkin 형식)
2. **성능 기준**:
   - 응답 시간 요구사항
   - 처리량 기대치
   - 데이터 볼륨 고려사항
3. **완료 정의**: 템플릿 기본값을 넘어서는 추가 DoD 기준이 있나요?

## Phase 7: 메타데이터 수집

AskUserQuestion 도구를 사용하여 다음을 수집합니다:

1. **스토리 포인트**: 추정치 (또는 TBD)
2. **Epic**: 상위 epic 링크 (해당하는 경우)
3. **Sprint**: 목표 sprint (알고 있는 경우)
4. **레이블**: 구성 요소, 우선순위 및 기타 관련 레이블
5. **관련 스토리**: 관련되거나 의존하는 스토리 링크

## Phase 8: Repository 정보

AskUserQuestion 도구를 사용하여 질문합니다:

1. **Repository**: 이 이슈를 어떤 repository에 생성해야 하나요?
   - `.claude/github.json`이 있으면 `repository.fullName`을 기본값으로 사용

## Phase 9: 이슈 생성 및 생성

1. **Validate Version One Final Time**: Before creating the issue, validate the version number:
   ```
   Pattern: ^v?([0-9]+)\.([0-9]+)\.([0-9]+)$
   - Strip optional 'v' prefix
   - Ensure each component is numeric
   - Ensure no leading zeros (except 0 itself)
   ```

2. **Fill in the template** with all gathered information:
   - Replace `[X.Y.Z]` with the validated version number
   - Replace `[Feature (Minor) / Bug Fix (Patch) / Breaking Change (Major)]` with the change type
   - Fill in the version impact explanation
   - Ensure version badge shows correct version
   - Add version label to the labels list

3. **Create a complete, formatted user story document**

4. **Create the GitHub issue**:
   ```bash
   gh issue create --repo [REPO] --title "[User Story Title]" --body-file [temp-file-with-content] --label "user-story,bdd,v[X.Y.Z],[other-labels]"
   ```

## Phase 10: 확인 및 다음 단계

사용자에게 다음을 표시합니다:
- 생성된 이슈 링크
- 버전 정보 (버전 번호 및 변경 타입)
- 주요 시나리오가 포함된 user story 요약
- 쉬운 참조를 위한 버전 배지

**다음 단계 제안:**
```
User Story가 생성되었습니다.
이제 `/issue` 명령어로 이 스토리를 Epic + 하위 이슈로 분해하고
마일스톤/프로젝트 로드맵에 연결하세요.
```

## 중요 사항:

- **점진적으로 질문하기** - 모든 질문을 한 번에 하지 말고 단계적으로 진행
- **Semver 검증 강제** - 진행하기 전에 버전 형식 (X.Y.Z)을 엄격히 검증
- **Gherkin 문법 검증** - Given/When/Then 구조가 올바른지 확인
- **구체적으로** - 모호한 플레이스홀더 피하기; 구체적인 세부사항 수집
- **테스트 가능성 고려** - 시나리오는 자동화된 테스트를 작성할 수 있을 만큼 명확해야 함
- **엣지 케이스 고려** - 오류 시나리오 및 경계 조건 추구
- **적절한 레이블 사용** - 항상 `user-story`, `bdd`, `v[X.Y.Z]` 레이블 포함
- **버전 일관성** - 버전이 템플릿 본문, 배지, 레이블에 나타나는지 확인
- **프로젝트/마일스톤 없음** - user story는 사양임; `/issue`를 사용하여 분해하고 로드맵에 연결

## GitHub CLI 명령어 참조:

```bash
# 이슈 (user story) 생성
gh issue create --repo [OWNER/REPO] --title "TITLE" --body "BODY" --label "user-story,bdd,v[X.Y.Z]"
```

템플릿을 로드하고 스토리 개요에 대한 첫 번째 질문 세트를 시작하세요.