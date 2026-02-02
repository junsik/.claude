# GitHub 메타데이터 캐시 동기화

`.claude/github.json` 캐시 파일을 GitHub API에서 최신 정보로 갱신합니다.

## 사용법

```bash
/gh-sync
```

## 동작

1. GitHub API를 호출하여 최신 정보 수집
2. `.claude/github.json` 파일 업데이트
3. 변경 사항 요약 출력

## 수집 정보

### Repository 정보
```bash
gh api repos/{owner}/{repo} --jq '{name, full_name, default_branch, html_url}'
```

### Milestones
```bash
gh api repos/{owner}/{repo}/milestones --jq '.[] | {number, title, state, open_issues, closed_issues}'
```

### Labels
```bash
gh label list --json name,color,description
```

### Project Fields (GitHub Projects v2)
```bash
gh project view {number} --owner {owner} --format json
```

## 갱신 절차

1. 현재 `.claude/github.json` 백업
2. 각 API 호출 및 데이터 수집
3. JSON 구조로 포맷팅
4. `lastUpdated` 타임스탬프 갱신
5. 파일 저장
6. 변경 사항 diff 출력

## 캐시 구조

```json
{
  "lastUpdated": "2026-02-02T00:00:00Z",
  "repository": { ... },
  "project": { ... },
  "milestones": [ ... ],
  "labels": { ... },
  "worktree": { ... },
  "conventions": { ... }
}
```

## 주의사항

- GitHub CLI (`gh`)가 인증되어 있어야 합니다
- Organization 프로젝트 접근 권한이 필요합니다
- 네트워크 오류 시 기존 캐시 유지
