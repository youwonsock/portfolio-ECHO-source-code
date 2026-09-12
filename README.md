# ECHO

소리를 시각화하는 **Echo 시스템**을 핵심 기믹으로 구성한 Unity 기반 3D 호러 퍼즐 어드벤처 게임입니다.

> 이 저장소는 전체 Unity 프로젝트가 아니라, 프로젝트에서 직접 구현한 주요 C# 스크립트와 셰이더를 선별해 공개한 포트폴리오용 소스 저장소입니다.

## 목차

- [프로젝트 개요](#프로젝트-개요)
- [프로젝트 요약](#프로젝트-요약)
- [사용 기술](#사용-기술)
- [클래스 구조 UML](#클래스-구조-uml)
- [기능 상세](#기능-상세)
- [저장소 구성](#저장소-구성)
- [실행 안내](#실행-안내)

## 프로젝트 개요

| 항목 | 내용 |
| --- | --- |
| 개발자 | 유원석 (You Won Sock) |
| GitHub | [youwonsock](https://github.com/youwonsock) |
| 이메일 | qazwsx233434@gmail.com |
| 개발 기간 | 2024.06 ~ 2024.08 |
| 장르 | 3D Horror Puzzle Adventure |
| 플랫폼 | Windows, Android |
| 프로젝트 목적 | 소리를 시각화하는 Echo 기믹과 이를 지원하는 클라이언트 시스템 구현 |
| 게임 엔진 | Unity |
| 개발 언어 | C#, ShaderLab/HLSL |
| 주요 패키지 | UniTask, Odin Inspector, TextMeshPro |
| 주요 기술 | Audio Mixer, ScriptableObject, Object Pooling, MVP, JSON Save/Load |
| 게임 페이지 | [Steam에서 ECHO 보기](https://store.steampowered.com/app/3132180/Echo/) |

## 프로젝트 요약

ECHO는 제한된 시야에서 플레이어와 NPC가 발생시키는 소리를 파동으로 표현하고, 파동이 닿은 공간과 오브젝트를 시각적으로 드러내는 게임입니다.

본 저장소에는 Echo 연출을 중심으로 게임 전반에서 재사용할 수 있도록 구현한 매니저, 오브젝트 풀, 사운드, UI, 설정 저장, 비동기 씬 전환 코드가 포함되어 있습니다.

주요 동작 흐름은 다음과 같습니다.

1. `GameManager`가 Object Pool, Sound, Scene, UI, Data 시스템을 초기화합니다.
2. 플레이어 또는 NPC의 소리 이벤트가 발생하면 `ObjectPool`에서 유형에 맞는 `Echo`를 가져옵니다.
3. `Echo`는 시간에 따라 반지름을 갱신하고 위치·유형 데이터를 `GameManager`에 전달합니다.
4. `GameManager`는 최대 25개의 Echo 데이터를 Ground·Number Material에 전달합니다.
5. Ground Shader와 Number Shader가 파동의 도달 범위에 따라 지면과 상호작용 오브젝트를 표현합니다.
6. `PlayerPresenter`가 플레이어 및 QTE 상태 변화를 UI에 전달합니다.
7. 그래픽·사운드·게임플레이 설정은 `DataManager`를 통해 JSON으로 저장하고 불러옵니다.

![ECHO 게임 화면](https://github.com/user-attachments/assets/099297dc-979a-4388-b759-40e1bb5abed4)

## 사용 기술

### Unity / C#

`GameManager`를 중심으로 게임 공통 시스템의 초기화와 생명주기를 관리합니다. 인터페이스 기반으로 Pooling, UI, Save/Load, Update 대상을 분리해 각 시스템의 책임을 명확하게 구성했습니다.

### UniTask

Echo 비활성화, 사용이 끝난 오브젝트와 AudioSource의 반환, 씬 로딩 대기처럼 프레임 간 비동기 처리가 필요한 로직에 사용했습니다. `CancellationTokenSource`를 통해 시스템 해제 시 대기 작업을 취소합니다.

### ShaderLab / HLSL

최대 25개의 Echo 위치와 반지름을 Material 배열로 전달합니다. Ground Shader는 월드 좌표와 파동 반지름의 거리를 비교해 지면 위의 원형 파동을 표현하고, Number Shader는 파동 범위에 들어온 오브젝트의 정점을 강조합니다.

### Object Pooling

`IPoolingAble` 인터페이스와 유형별 `Queue`를 사용해 Echo 오브젝트를 재사용합니다. 기본 인스턴스를 미리 생성하고, 풀이 비었을 때만 새 인스턴스를 생성해 반복적인 생성·파괴를 줄였습니다.

### Audio Mixer

BGM과 SFX 채널을 분리하고 2D·3D 재생을 지원합니다. SFX용 AudioSource와 AudioClip을 각각 Queue와 Dictionary로 관리해 반복 재생 비용을 줄였습니다.

### UI / MVP

`PlayerPresenter`가 플레이어·QTE 모델의 상태 변경 이벤트를 받아 `IProgressUIView`에 전달합니다. `UIManager`는 Scene UI와 Popup UI를 구분하고 Stack으로 팝업 순서와 생명주기를 관리합니다.

### JSON Save/Load

`ISave` 구현 객체를 `DataManager`에 등록하고 `Application.persistentDataPath`에 JSON 설정 파일을 저장합니다. 그래픽, 사운드, 카메라 FOV, 마우스 감도 설정을 다음 실행에서도 유지합니다.

## 클래스 구조 UML

UML 원본은 [`UML.plantuml`](UML.plantuml)에서 관리합니다.

> **이미지 플레이스홀더:** `UML.plantuml`을 렌더링한 클래스 구조 이미지

핵심 관계는 다음과 같습니다.

- `GameManager` → `ObjectPool`, `SoundManager`, `SceneManagerEX`, `UIManager`, `DataManager`
- `IPoolingAble` → `Echo`, `NullPoolingAble`
- `IUIBase` → Scene UI, Popup UI, Dependent UI
- `ISave`, `ISettingData` → Graphic, Sound, Gameplay Setting UI
- `PlayerPresenter` → Player/QTE Model과 Progress UI 연결

### 클래스별 역할

- `GameManager`: 공통 시스템 초기화와 Echo 데이터를 Material에 전달합니다.
- `Echo`: 시간에 따른 파동 확장, 위치·반지름 갱신, 자동 비활성화를 처리합니다.
- `ObjectPool`: PoolingType별 Echo 인스턴스를 생성·대여·회수합니다.
- `SoundManager`: BGM/SFX 재생, AudioClip 캐시, AudioSource 재사용, Mixer 볼륨을 관리합니다.
- `UIManager`: UI 생성, Canvas 정렬, Popup Stack과 Scene UI 생명주기를 관리합니다.
- `PlayerPresenter`: 플레이어와 QTE 상태 변경을 Progress UI에 반영합니다.
- `DataManager`: `ISave` 객체를 파일명별로 등록하고 설정 데이터를 저장·불러옵니다.
- `SceneManagerEX`: 중복 로딩을 방지하고 UniTask 기반으로 씬을 비동기 전환합니다.
- `UpdateManager`: `IUpdateable` 구현 객체의 Update 계열 작업을 중앙에서 호출합니다.

## 기능 상세

### Echo 파동 시스템

**목적**

소리의 발생 위치와 확산 범위를 게임의 핵심 시각 정보로 변환합니다.

**핵심 구현**

- 플레이어·NPC 및 걷기·달리기 Echo를 `PoolingType`으로 구분합니다.
- ScriptableObject에 Echo 유형, 확산 시간, 최대 크기를 데이터로 분리했습니다.
- `FixedUpdateWork`에서 시간에 따른 반지름을 계산합니다.
- 순환 ID를 사용해 최대 25개의 Echo 위치·반지름·발생 주체를 Shader 배열에 기록합니다.
- 확산 시간이 끝나면 Echo를 비활성화하고 Object Pool에 반환합니다.

> **스크린샷 플레이스홀더:** 플레이어와 NPC Echo가 동시에 확산되는 장면

### Ground Echo Shader

**목적**

Echo가 도달한 지면에 원형 파동을 표현해 보이지 않는 공간의 형태를 인지할 수 있게 합니다.

**핵심 구현**

- Vertex Shader에서 정점의 월드 좌표를 계산합니다.
- Pixel Shader에서 월드 좌표와 각 Echo 중심 사이의 거리를 비교합니다.
- 반지름과 `_EchoWidth`로 파동의 테두리 구간을 판정합니다.
- Echo 유형 데이터에 따라 플레이어와 NPC의 파동 색상을 구분합니다.
- 반복 횟수를 25회로 고정하고 `[unroll]`을 적용했습니다.

![Ground Echo Shader](https://github.com/user-attachments/assets/e6b74f9c-cbbd-47a6-97a4-282b92af6fed)

### Number Shader

**목적**

Echo 범위에 들어온 숫자·도형 오브젝트를 강조해 퍼즐 단서의 가시성을 높입니다.

**핵심 구현**

- 정점별 월드 좌표와 Echo 중심의 거리를 계산합니다.
- 여러 Echo 중 가장 가까운 파동의 영향을 선택합니다.
- `_AfterImageDistance`를 기준으로 강조 색상의 세기를 계산합니다.
- 플레이어와 NPC Echo를 서로 다른 색상으로 표현합니다.

![Number Shader](https://github.com/user-attachments/assets/6b051763-f68a-4d50-8b53-0218c22947e2)

### Echo Object Pool

**목적**

플레이 중 반복적으로 발생하는 Echo의 생성·파괴 비용을 줄입니다.

**핵심 구현**

- `IPoolingAble`로 활성화, 비활성화, 유형 반환 규약을 정의했습니다.
- Resources의 Prefab을 탐색해 PoolingType별 Queue를 구성합니다.
- 유형별 인스턴스를 기본 10개씩 미리 생성합니다.
- Queue가 비었을 때만 인스턴스를 추가 생성합니다.
- 비활성화가 확인된 객체를 UniTask로 대기한 뒤 해당 Queue에 반환합니다.
- 유효한 객체를 만들 수 없는 경우를 위해 Null Object를 사용합니다.

> **스크린샷 플레이스홀더:** Echo Object Pool 대여·회수 과정

### Sound Manager

**목적**

게임 전역에서 BGM과 2D·3D SFX를 일관된 방식으로 재생하고 재사용합니다.

**핵심 구현**

- Audio Mixer의 Master, BGM, SFX 그룹을 분리합니다.
- BGM 전용 AudioSource와 SFX용 AudioSource Queue를 구성합니다.
- SFX AudioClip을 경로별 Dictionary에 캐시합니다.
- SFX 재생이 끝나면 AudioSource를 Queue에 반환합니다.
- 풀을 초과해 임시 생성한 3D AudioSource는 재생 완료 후 제거합니다.
- `CancellationTokenSource`로 대기 중인 비동기 작업의 종료 시점을 관리합니다.

> **스크린샷 플레이스홀더:** Audio Mixer 및 3D SFX 동작 화면

### UI 관리와 Presenter

**목적**

UI 생성·정렬·해제를 중앙화하고 플레이어 로직과 UI 표시를 분리합니다.

**핵심 구현**

- Scene, Popup, Dependent 세 가지 UI 유형을 정의합니다.
- Popup UI를 Stack으로 관리하고 Canvas sorting order를 자동 지정합니다.
- 설정 창의 하위 UI는 부모 UI 생명주기에 종속되도록 구성합니다.
- `PlayerPresenter`가 `PropertyChanged` 이벤트를 받아 스태미나와 QTE 진행도를 갱신합니다.
- `IProgressUIView`로 Presenter가 구체적인 UI 클래스에 직접 의존하지 않도록 했습니다.

![UI 시스템](https://github.com/user-attachments/assets/539458bc-ff6b-4e6e-9ad0-a2e6ed717b24)

### 설정 저장 시스템

**목적**

사용자의 그래픽·사운드·조작 설정을 저장하고 다음 실행 시 동일하게 적용합니다.

**핵심 구현**

- `ISave` 구현 객체를 파일명 기준으로 `DataManager`에 등록합니다.
- 설정 구조체를 `JsonUtility`로 직렬화해 persistent data 경로에 저장합니다.
- 그래픽 설정에서 화면 모드, 16:9 해상도, 프레임 제한, VSync, Anti-Aliasing을 처리합니다.
- 사운드 설정에서 Master, BGM, SFX 볼륨과 음소거 상태를 처리합니다.
- 게임플레이 설정에서 카메라 FOV와 마우스 감도를 처리합니다.
- Android에서는 PC 전용 디스플레이 설정 UI를 조건부로 제외합니다.

> **스크린샷 플레이스홀더:** 그래픽·사운드·게임플레이 설정 화면

### 비동기 씬 전환과 Update 관리

**목적**

씬 전환 중 중복 요청을 막고, 반복 업데이트가 필요한 객체를 중앙에서 관리합니다.

**핵심 구현**

- MainMenu, Loading, InGame 씬을 열거형으로 관리합니다.
- Loading 씬을 거쳐 대상 씬을 비동기로 불러옵니다.
- 로딩 진행률이 활성화 가능한 시점에 도달하면 대상 씬으로 전환합니다.
- `IUpdateable` 구현 객체가 필요한 Update, FixedUpdate, LateUpdate 작업만 구독합니다.
- Echo와 QTE UI는 활성화 상태에 맞춰 Update 작업을 등록·해제합니다.

> **스크린샷 플레이스홀더:** Loading 씬과 대상 씬 전환 화면

## 저장소 구성

| 경로 | 내용 |
| --- | --- |
| [`Character`](Character) | Player 상태와 UI를 연결하는 Presenter |
| [`Echo`](Echo) | Echo 파동 동작 및 Pooling 구현 |
| [`Interface`](Interface) | Pooling, UI, Save, Setting, Update 인터페이스 |
| [`Manager`](Manager) | Game, Data, Object Pool, Scene, Sound, UI, Update Manager |
| [`Scene`](Scene) | 씬별 초기화·해제 생명주기 |
| [`Shader`](Shader) | Ground, Number Shader와 Echo Shader Graph |
| [`SO`](SO) | Echo 확산 설정 ScriptableObject |
| [`UI`](UI) | 설정, Pause, QTE, Stamina 등 UI 구현 |
| [`UML.plantuml`](UML.plantuml) | 클래스 구조 UML 원본 |

## 실행 안내

이 저장소는 포트폴리오 공개를 위해 선별한 소스 코드만 포함하므로 단독으로 빌드하거나 실행할 수 없습니다. 완성된 게임은 아래 Steam 페이지에서 확인할 수 있습니다.

- [ECHO Steam 페이지](https://store.steampowered.com/app/3132180/Echo/)
