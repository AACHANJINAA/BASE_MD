# Base MD

게임 개발 프로젝트를 AI와 함께 진행하기 위한 기본 문서 템플릿입니다.

이 구조의 목표는 AI가 매 작업마다 모든 문서를 읽지 않도록 하고, 사용자가 `docs/PLAY_REQUEST.md`로 작업 범위와 참조 문서를 먼저 지정해 토큰 사용을 줄이는 것입니다.

## 기본 구조

```text
GDD/
  Game_GDD.md
  Implemention_Plan.md

docs/
  AI_USAGE.md
  PLAY_REQUEST.md
  WORKFLOW.md
  ARCHITECTURE.md
  PLANS.md
  GAME_RULES.md
  Verify.md
  DECISIONS.md
```

## 사용 흐름

1. 새 작업이 생기면 `docs/PLAY_REQUEST.md`에 작업 요청을 정리합니다.
2. AI는 먼저 `docs/PLAY_REQUEST.md`만 확인합니다.
3. 참조 문서가 지정되어 있으면 그 문서만 우선 확인합니다.
4. 참조 문서가 비어 있으면 `docs/AI_USAGE.md`의 작업 유형별 최소 참조 규칙을 따릅니다.
5. 작업 후 필요한 문서만 갱신합니다.
6. 완료 이력은 커밋으로 남기고, 이후 작업에 영향을 주는 결정만 `docs/DECISIONS.md`에 기록합니다.

## 주요 문서

- `docs/PLAY_REQUEST.md`: 사용자가 AI에게 전달하는 작업 요청서입니다.
- `docs/AI_USAGE.md`: AI가 문서를 적게 읽고 필요한 범위만 확인하기 위한 규칙입니다.
- `docs/WORKFLOW.md`: 작업 진행 순서와 사용자 확인이 필요한 지점을 정의합니다.
- `docs/DECISIONS.md`: 반복해서 확인할 필요가 있는 확정 결정을 짧게 기록합니다.
- `GDD/Game_GDD.md`: 게임 컨셉, 목표 경험, 핵심 기획을 정리합니다.
- `GDD/Implemention_Plan.md`: 기획을 실제 구현 순서와 마일스톤으로 나눕니다.
- `docs/PLANS.md`: 현재 작업 상태, 다음 작업, 보류 작업을 관리합니다.
- `docs/ARCHITECTURE.md`: 폴더, 코드, 에셋, 씬의 책임 범위를 정리합니다.
- `docs/GAME_RULES.md`: 플레이 판정, 점수, 승패, 턴, 밸런스 규칙을 기록합니다.
- `docs/Verify.md`: 구현 후 확인할 검증 항목과 완료 기준을 정리합니다.

## 운영 원칙

- 모든 문서를 자동으로 읽지 않습니다.
- 작업 요청서에 적힌 참조 문서와 수정 허용 범위를 우선합니다.
- 큰 구조 변경, 여러 문서 동시 수정, 새 시스템 추가 전에는 사용자 확인을 거칩니다.
- 로그 문서는 따로 두지 않고 커밋 기록을 완료 이력으로 사용합니다.
- `DECISIONS.md`에는 작업 내역 전체가 아니라 앞으로의 개발 방향에 영향을 주는 결정만 남깁니다.
