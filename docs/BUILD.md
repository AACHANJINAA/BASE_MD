# Build

이 문서는 엔진별로 빌드·컴파일·테스트를 효율적으로 실행하는 방법을 정리합니다.

목표는 두 가지입니다.

- 반복 작업의 대기 시간을 줄입니다.
- AI가 빌드할 때 출력 전체를 읽지 않게 해서 토큰 사용을 줄입니다.

사용하지 않는 엔진 섹션은 지워도 됩니다. 경로의 `<...>`는 프로젝트에 맞게 바꿉니다.

## 공통 원칙

- **빌드 출력은 걸러서 봅니다.** 에러, 경고, 최종 결과 줄만 확인하고 진행 로그는 읽지 않습니다.
- **첫 번째 에러부터 고칩니다.** 뒤따르는 에러는 대부분 첫 에러의 연쇄입니다. 고친 뒤 다시 빌드해서 남은 에러를 확인합니다.
- **같은 실패를 반복하지 않습니다.** 같은 에러로 두 번 실패하면 재시도하지 말고 원인과 대안을 사용자에게 보고합니다.
- **에디터 상태를 먼저 확인합니다.** 에디터가 켜져 있으면 외부 빌드가 실패하거나 충돌할 수 있습니다. 엔진별 규칙을 따릅니다.
- **읽지 않을 폴더**: 빌드 산출물, 캐시, 엔진 소스, 바이너리 에셋은 요청이 없으면 열지 않습니다.
- **API 확인**: 엔진 소스나 생성 코드를 열기보다 엔진 버전을 기준으로 판단하고, 불확실하면 사용자에게 확인합니다.

### 셸별 필터 문법

```bash
# Bash / Git Bash
<빌드 명령> 2>&1 | grep -E "<패턴>" | head -40
```

```powershell
# PowerShell
<빌드 명령> 2>&1 | Select-String -Pattern "<패턴>" | Select-Object -First 40
```

---

## Unreal Engine

### 읽지 않을 폴더

`Binaries/`, `Intermediate/`, `Saved/`(로그 확인 시 제외), `DerivedDataCache/`, `Plugins/*/Binaries/`, `Plugins/*/Intermediate/`, 엔진 설치 폴더, `.uasset`/`.umap` (바이너리)

### 수정 종류별 빌드 방법

| 수정 내용 | 방법 | 비고 |
| --- | --- | --- |
| `.cpp` 함수 본문만 수정 | 에디터에서 Live Coding (`Ctrl+Alt+F11`) | 가장 빠름. 사용자가 직접 실행 |
| 헤더 수정 (`UPROPERTY`, `UFUNCTION`, `UCLASS`, 멤버 추가·삭제) | **에디터 종료 → 외부 빌드 → 에디터 재시작** | Live Coding으로 반영하면 블루프린트나 에셋이 깨질 수 있음 |
| 새 클래스·파일 추가 | 에디터 종료 → 프로젝트 파일 재생성 → 외부 빌드 | |
| `.Build.cs`, `.uproject`, 플러그인 변경 | 에디터 종료 → 프로젝트 파일 재생성 → 외부 빌드 | |

### 에디터 상태 규칙

- 에디터가 켜져 있고 Live Coding이 활성화된 상태에서는 외부 빌드가 `Unable to build while Live Coding is active` 에러로 실패합니다.
- 이 에러가 나오면 재시도하지 말고 사용자에게 에디터를 닫아 달라고 요청합니다.

### 외부 빌드 명령 (Windows)

```bash
"<UE 설치 경로>/Engine/Build/BatchFiles/Build.bat" <프로젝트명>Editor Win64 Development -Project="<프로젝트 경로>/<프로젝트명>.uproject" -WaitMutex 2>&1 | grep -E "error|warning C[0-9]+|Result:" | head -40
```

- 타깃 이름은 `<프로젝트명>Editor`입니다. (에디터용 빌드)
- 마지막의 `Result: Succeeded` 또는 `Result: Failed`로 성공 여부를 판단합니다.
- 에러 패턴: `error C####` (컴파일), `error LNK####` (링크), `.h(줄): Error:` (UHT, 헤더 매크로 문제)

### 프로젝트 파일 재생성

- 가장 간단한 방법: `.uproject` 우클릭 → `Generate Visual Studio project files`
- 명령으로 실행:

```bash
"<UE 설치 경로>/Engine/Binaries/DotNET/UnrealBuildTool/UnrealBuildTool.exe" -projectfiles -project="<프로젝트 경로>/<프로젝트명>.uproject" -game -progress 2>&1 | tail -5
```

### 로그 위치

- 빌드 로그: `%LOCALAPPDATA%/UnrealBuildTool/Log.txt`
- 에디터·런타임 로그: `<프로젝트>/Saved/Logs/<프로젝트명>.log`
- 로그는 통째로 읽지 말고 필요한 줄만 검색합니다.

```bash
grep -E "Error|Warning|LogTemp" "<프로젝트>/Saved/Logs/<프로젝트명>.log" | tail -40
```

### 자동화 테스트 (선택)

```bash
"<UE 설치 경로>/Engine/Binaries/Win64/UnrealEditor-Cmd.exe" "<프로젝트 경로>/<프로젝트명>.uproject" -ExecCmds="Automation RunTests <테스트 이름>;Quit" -unattended -nullrhi -nosplash -log 2>&1 | grep -E "Test Completed|Result=|Error" | head -40
```

### 빌드 시간 줄이기

- 헤더에서는 전방 선언(`class UFoo;`)을 쓰고 `#include`는 `.cpp`로 옮깁니다.
- 여러 곳에서 include되는 헤더는 자주 수정하지 않습니다. 수정하면 연쇄적으로 재컴파일됩니다.
- 튜닝 수치는 `UPROPERTY(EditAnywhere)`로 노출해서 빌드 없이 에디터에서 조정합니다.
- 첫 빌드는 오래 걸리므로 작업 시작 직후 한 번 미리 빌드해 둡니다.

---

## Unity

### 읽지 않을 폴더·파일

`Library/`, `Temp/`, `Obj/`, `Logs/`, `Build/`, `UserSettings/`, `.meta` 파일, **`.unity`·`.prefab`·`.asset` YAML 파일**

씬과 프리팹 YAML은 텍스트라서 열 수는 있지만 매우 길어 토큰을 크게 소모합니다. 수정이 꼭 필요할 때만 관련 부분을 검색해서 확인합니다.

### 수정 종류별 확인 방법

| 상황 | 방법 | 비고 |
| --- | --- | --- |
| 에디터가 켜져 있음 (일반적인 경우) | 파일 저장 → 에디터가 자동 재컴파일 → `Editor.log`에서 에러만 검색 | 가장 빠름 |
| 에디터가 꺼져 있음 | 배치 모드로 컴파일 확인 | 프로젝트를 여는 시간이 걸림 |
| 테스트 실행 | 배치 모드 `-runTests` | |
| 실행 파일 빌드 | 배치 모드 `-executeMethod` | 빌드 스크립트 필요 |

### 에디터 상태 규칙

- 같은 프로젝트가 에디터에서 열려 있으면 배치 모드 실행이 실패합니다. (프로젝트 잠금)
- 에디터가 켜져 있을 때는 배치 모드 대신 `Editor.log`를 확인합니다.
- 에디터 창이 포커스를 받아야 재컴파일이 시작될 수 있습니다. 로그에 변화가 없으면 사용자에게 에디터 창을 한 번 클릭해 달라고 요청합니다.

### 에디터 로그에서 컴파일 에러 확인

- Windows: `%LOCALAPPDATA%/Unity/Editor/Editor.log`
- macOS: `~/Library/Logs/Unity/Editor.log`

```bash
grep -E "error CS[0-9]+|warning CS[0-9]+" "$LOCALAPPDATA/Unity/Editor/Editor.log" | tail -30
```

- 에러 형식: `Assets/Scripts/Foo.cs(12,5): error CS0103: ...` (파일, 줄, 열이 함께 나옵니다)
- 로그에는 이전 컴파일 결과도 남아 있으므로 `tail`로 최근 결과만 봅니다.

### 배치 모드 컴파일 확인 (에디터가 꺼져 있을 때)

```bash
"<Unity 설치 경로>/Editor/Unity.exe" -batchmode -nographics -quit -projectPath "<프로젝트 경로>" -logFile "<프로젝트 경로>/Logs/batch.log"; echo "exit: $?"
grep -E "error CS[0-9]+" "<프로젝트 경로>/Logs/batch.log" | head -30
```

- 종료 코드가 0이 아니면 실패입니다. 컴파일 에러가 있으면 실패합니다.
- 설치 경로는 Unity Hub 기준으로 보통 `C:/Program Files/Unity/Hub/Editor/<버전>/Editor/Unity.exe`입니다.

### 테스트 실행 (Unity Test Framework)

```bash
"<Unity.exe>" -batchmode -nographics -projectPath "<프로젝트 경로>" -runTests -testPlatform EditMode -testResults "<프로젝트 경로>/Logs/test-results.xml" -logFile "<프로젝트 경로>/Logs/test.log"; echo "exit: $?"
grep -oE 'result="[A-Za-z]+"|total="[0-9]+"|passed="[0-9]+"|failed="[0-9]+"' "<프로젝트 경로>/Logs/test-results.xml" | head -5
```

- `-runTests`를 쓸 때는 `-quit`를 붙이지 않습니다.
- 플레이 모드 테스트는 `-testPlatform PlayMode`를 씁니다.

### 실행 파일 빌드 (선택)

`Assets/Editor/BuildScript.cs`에 `BuildPipeline.BuildPlayer`를 호출하는 정적 메서드를 만든 뒤 실행합니다.

```bash
"<Unity.exe>" -batchmode -nographics -quit -projectPath "<프로젝트 경로>" -executeMethod BuildScript.Build -logFile "<프로젝트 경로>/Logs/build.log"; echo "exit: $?"
```

### 컴파일 시간 줄이기

- Assembly Definition(`.asmdef`)으로 코드를 나누면 수정한 어셈블리만 재컴파일됩니다.
- `Enter Play Mode Options`에서 도메인 리로드를 끄면 플레이 진입이 빨라집니다. 대신 static 변수 초기화를 직접 처리해야 합니다.
- 튜닝 수치는 `[SerializeField]`나 ScriptableObject로 노출해서 재컴파일 없이 조정합니다.
