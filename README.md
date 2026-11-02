# Base MD

그래픽스와 게임 개발 프로젝트를 AI와 함께 진행하기 위한 범용 문서 템플릿입니다.

게임, 렌더러/엔진, 셰이더·이펙트, 그래픽스 데모, 툴/에디터 등 실시간 그래픽스를 다루는 프로젝트라면 장르나 엔진(Unity, Unreal, Godot, 자체 엔진, OpenGL/Vulkan/DirectX/Metal/WebGPU 등)에 관계없이 사용할 수 있습니다.

이 구조의 목표는 AI가 매 작업마다 모든 문서를 읽지 않도록 하고, 사용자가 `docs/TASK_REQUEST.md`로 작업 범위와 참조 문서를 먼저 지정해 토큰 사용을 줄이는 것입니다.

## 기본 구조

```text
CLAUDE.md
.gitignore

design/
  PROJECT_BRIEF.md
  IMPLEMENTATION_PLAN.md

docs/
  AI_USAGE.md
  TASK_REQUEST.md
  WORKFLOW.md
  ARCHITECTURE.md
  PLANS.md
  SPEC.md
  VERIFY.md
  DECISIONS.md
```

## 사용 흐름

1. 새 작업이 생기면 `docs/TASK_REQUEST.md`에 작업 요청을 정리합니다.
2. AI는 먼저 `docs/TASK_REQUEST.md`만 확인합니다.
3. 참조 문서가 지정되어 있으면 그 문서만 우선 확인합니다.
4. 참조 문서가 비어 있으면 `docs/AI_USAGE.md`의 작업 유형별 최소 참조 규칙을 따릅니다.
5. 작업 후 필요한 문서만 갱신합니다.
6. 완료 이력은 커밋으로 남기고, 이후 작업에 영향을 주는 결정만 `docs/DECISIONS.md`에 기록합니다.

AI 도구는 프로젝트 루트의 `CLAUDE.md`를 자동으로 읽고, 이 흐름을 따르도록 안내받습니다.

## 새 프로젝트에 적용하기

1. 엔진/프레임워크로 프로젝트를 먼저 생성합니다.
2. 이 저장소의 파일을 프로젝트 루트에 복사합니다. (`.git` 폴더는 제외)
3. `.gitignore`에서 사용하지 않는 엔진 섹션을 정리합니다.
4. `design/PROJECT_BRIEF.md`로 기획을 정리합니다.
5. `docs/SPEC.md`의 공통 기술 기준과 `CLAUDE.md`의 프로젝트 정보를 채웁니다.
6. `design/IMPLEMENTATION_PLAN.md`와 `docs/PLANS.md`로 작업을 나눈 뒤 구현을 시작합니다.

## 작업 모드

- **일반 모드**: 장기 프로젝트용입니다. `docs/WORKFLOW.md`의 사용자 확인 지점을 모두 지킵니다.
- **스프린트 모드**: 하루 단위 테스트, 게임잼, 데모 마감처럼 시간이 정해진 작업용입니다. 허용 범위 안의 파일 생성과 문서 갱신은 확인 없이 진행하고, 범위·구조·외부 의존성 변경만 확인합니다.

`docs/TASK_REQUEST.md`의 `작업 모드`에서 선택합니다. 세부 규칙은 `docs/WORKFLOW.md`를 참고합니다.

## 주요 문서

- `CLAUDE.md`: AI가 작업을 시작할 때 자동으로 읽는 진입점입니다. 문서 사용 규칙과 프로젝트 기본 정보를 담습니다.

- `docs/TASK_REQUEST.md`: 사용자가 AI에게 전달하는 작업 요청서입니다.
- `docs/AI_USAGE.md`: AI가 문서를 적게 읽고 필요한 범위만 확인하기 위한 규칙입니다.
- `docs/WORKFLOW.md`: 작업 진행 순서와 사용자 확인이 필요한 지점을 정의합니다.
- `docs/DECISIONS.md`: 반복해서 확인할 필요가 있는 확정 결정을 짧게 기록합니다.
- `design/PROJECT_BRIEF.md`: 프로젝트 개요, 목표, 범위, 핵심 기능(게임이라면 GDD 역할)을 정리합니다.
- `design/IMPLEMENTATION_PLAN.md`: 기획을 실제 구현 순서와 마일스톤으로 나눕니다.
- `docs/PLANS.md`: 현재 작업 상태, 다음 작업, 보류 작업을 관리합니다.
- `docs/ARCHITECTURE.md`: 폴더, 코드 모듈, 렌더링 파이프라인, 에셋, 씬의 책임 범위를 정리합니다.
- `docs/SPEC.md`: 동작 사양과 기술 기준을 기록합니다. (게임 규칙·밸런스, 좌표계·색 공간·단위, 성능 예산 등)
- `docs/VERIFY.md`: 구현 후 확인할 검증 항목과 완료 기준을 정리합니다.

## 프로젝트 유형별 활용

| 문서 | 게임 프로젝트 | 그래픽스 프로젝트 |
| --- | --- | --- |
| `PROJECT_BRIEF.md` | 컨셉, 플레이 루프, 콘텐츠 (GDD) | 목표 기법, 대상 플랫폼/API, 결과물 형태 |
| `SPEC.md` | 조작, 승패, 점수, 밸런스 규칙 | 좌표계, 색 공간, 단위, 포맷, 성능 예산 |
| `ARCHITECTURE.md` | 시스템, 씬, 에셋 구성 | 렌더 패스, 리소스 관리, 셰이더 구성 |
| `VERIFY.md` | 플레이 테스트, 빌드 확인 | 시각 비교, 프레임 타임/GPU 프로파일링, 검증 레이어 |

사용하지 않는 항목은 비워 두거나 지워도 됩니다.

## 운영 원칙

- 모든 문서를 자동으로 읽지 않습니다.
- 작업 요청서에 적힌 참조 문서와 수정 허용 범위를 우선합니다.
- 큰 구조 변경, 여러 문서 동시 수정, 새 시스템 추가 전에는 사용자 확인을 거칩니다.
- 로그 문서는 따로 두지 않고 커밋 기록을 완료 이력으로 사용합니다.
- `DECISIONS.md`에는 작업 내역 전체가 아니라 앞으로의 개발 방향에 영향을 주는 결정만 남깁니다.
