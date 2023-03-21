# 프로젝트 컨벤션

프로젝트를 진행하면서 컨벤션을 정하고 진행했어요. 몇가지를 소개하려고 합니다.

## Branch

기능 단위로 `Branch`를 구성합니다.

- Branch 이름에는 prefix로 `feature/*` 를 붙입니다.
- 이슈와 관련된 주제를 작업한 경우에는 `feature/{issue-number}-{title}` 형식으로 네이밍합니다.
- 일반적인 기능 추가인 경우는 `feature/{title}` 형식을 따릅니다.

## Commit

- 각 액션마다 하나의 `Commit`을 작성합니다.
- Commit 메시지는 작업 내용을 이해할 수 있도록 작성하며,
- `{작업 유형}: {작업 설명}` 형식으로 작성합니다.

### 작업 유형

- feat: 새로운 기능 추가
- fix: 버그 수정
- refactor: 기능 개선, 코드 리팩토링
- style: 코드 스타일 수정
- docs: 문서 추가, 수정
- test: 테스트 코드 추가, 수정
- chore: 개발 환경 추가, 수정.

## Code

- [Airbnb](https://github.com/airbnb/javascript) 컨벤션에 따라 `Eslint`와 `Prettier`를 설정했습니다.

## Husky

- git hook을 활용하여 `pre-commit` 시점에서 Eslint와 Prettier 규칙을 체크합니다.

## CI

- 최종적으로 `Pull Request` 시점에 Eslint와 Prettier 규칙을 체크합니다.
