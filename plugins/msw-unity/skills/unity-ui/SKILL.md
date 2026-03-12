---
name: unity-ui
description: Unity UI Toolkit 기반 UI 생성 (UXML, USS, C# 컨트롤러, 에디터 세팅 스크립트 자동 생성)
triggers:
  - "UI 만들어"
  - "UI Toolkit"
  - "UXML"
  - "USS"
  - "UIDocument"
  - "화면 만들어"
  - "메뉴 만들어"
  - "HUD"
  - "인벤토리 UI"
---

# Unity UI Toolkit UI 생성 스킬

## 개요

Unity UI Toolkit을 사용하여 런타임 UI를 생성합니다. UXML(레이아웃), USS(스타일), C# 컨트롤러, 에디터 세팅 스크립트를 자동으로 생성합니다.

## 사전 조건

- Unity 2021.2+ 프로젝트 (UI Toolkit 런타임 지원)
- `com.unity.modules.uielements` 패키지가 manifest.json에 포함되어 있어야 함
- ProjectSettings/ProjectVersion.txt로 Unity 버전 확인

## 워크플로우

### 1단계: 프로젝트 확인

1. `ProjectSettings/ProjectVersion.txt`에서 Unity 버전 확인
2. `Packages/manifest.json`에서 `com.unity.modules.uielements` 존재 확인
3. 조건 미충족 시 사용자에게 안내

### 2단계: 요구사항 파악

사용자에게 질문:
- UI 이름 (예: InventoryUI, MainMenuUI)
- UI 유형 (HUD, 메뉴, 팝업, 설정 등)
- 필요한 요소 (버튼, 텍스트, 슬라이더, 토글, 리스트 등)

### 3단계: 파일 생성

다음 파일들을 `Assets/{UIName}/` 디렉토리에 생성:

#### UXML 파일 (`{UIName}.uxml`)
```xml
<ui:UXML xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:ui="UnityEngine.UIElements"
         xsi:noNamespaceSchemaLocation="../UIElementsSchema/UIElements.xsd">
    <Style src="{UIName}.uss" />
    <!-- UI 요소들 -->
</ui:UXML>
```

#### USS 파일 (`{UIName}.uss`)
- 다크 테마 기본 스타일 제공
- 반응형 레이아웃 (flexbox 기반)
- 카드, 버튼, 입력 필드 등 공통 스타일

#### C# 컨트롤러 (`{UIName}Controller.cs`)
```csharp
using UnityEngine;
using UnityEngine.UIElements;

public class {UIName}Controller : MonoBehaviour
{
    void OnEnable()
    {
        var root = GetComponent<UIDocument>().rootVisualElement;
        // Q<T>()로 요소 바인딩
        // 이벤트 핸들러 등록
    }
}
```

#### 에디터 세팅 스크립트 (`Editor/{UIName}SceneSetup.cs`)
```csharp
using UnityEditor;
using UnityEngine;
using UnityEngine.UIElements;

public static class {UIName}SceneSetup
{
    [MenuItem("Tools/Setup {UIName}")]
    public static void SetupScene()
    {
        // PanelSettings 에셋 생성/검색
        // UIDocument 컴포넌트 추가 + UXML 연결
        // 컨트롤러 스크립트 추가
    }
}
```

### 4단계: 사용자 안내

생성 완료 후 안내:
1. Unity 에디터로 돌아가서 컴파일 대기
2. 테스트 씬 열기 (또는 새 씬)
3. 메뉴 `Tools > Setup {UIName}` 클릭
4. Play 버튼으로 테스트

## 핵심 규칙

1. **USS 인라인 참조**: UXML에서 `<Style src="{UIName}.uss" />` 방식으로 USS를 직접 참조. PanelSettings themeStyleSheet 사용 금지
2. **에디터 스크립트 필수**: Unity 씬 YAML을 직접 조작하지 않고, 에디터 스크립트(MenuItem)로 씬 구성 자동화
3. **PanelSettings 자동 생성**: 에디터 스크립트에서 PanelSettings 에셋이 없으면 자동 생성 (scaleMode: ScaleWithScreenSize, referenceResolution: 1920x1080)
4. **네이밍 규칙**: PascalCase (예: MainMenuUI, InventoryPanel)
5. **GUID 참조 금지**: 씬 파일에 GUID를 하드코딩하지 않음 — 에디터 API(AssetDatabase.FindAssets)로 런타임 검색

## USS 스타일 가이드

- flexbox 기반 레이아웃 사용
- border-radius로 라운드 처리
- :hover pseudo-class로 인터랙션
- 색상: Catppuccin Mocha 팔레트 기본 사용
  - 배경: rgb(30, 30, 46)
  - 카드: rgba(49, 50, 68, 0.9)
  - 텍스트: rgb(205, 214, 244)
  - 액센트: rgb(137, 180, 250)
  - 성공: rgb(166, 227, 161)

## 참조 예제

생성 전 반드시 해당 예제를 Read로 로드하여 패턴을 확인:

| 예제 | 경로 | 내용 |
|------|------|------|
| 기본 카운터 | `references/examples/01-basic-counter.md` | 버튼, 입력, 토글, 슬라이더 |
| 채팅 UI | `references/examples/02-chat-ui.md` | 사이드바, 탭, 동적 메시지, 스크롤 |
| uGUI↔UIToolkit 브릿지 | `references/examples/03-ugui-uitoolkit-bridge.md` | 양방향 드래그앤드롭, 좌표변환, 고스트 |
| uGUI 도킹+UIToolkit 싱크 | `references/examples/04-dock-sync.md` | 도킹 윈도우 추적, 리사이즈 연동, 투명 오버레이 |

## UI Toolkit 주요 요소 레퍼런스

| 요소 | UXML 태그 | C# 접근 |
|------|-----------|---------|
| 버튼 | `<ui:Button>` | `root.Q<Button>("name")` |
| 레이블 | `<ui:Label>` | `root.Q<Label>("name")` |
| 텍스트 입력 | `<ui:TextField>` | `root.Q<TextField>("name")` |
| 토글 | `<ui:Toggle>` | `root.Q<Toggle>("name")` |
| 슬라이더 | `<ui:Slider>` / `<ui:SliderInt>` | `root.Q<Slider>("name")` |
| 드롭다운 | `<ui:DropdownField>` | `root.Q<DropdownField>("name")` |
| 스크롤뷰 | `<ui:ScrollView>` | `root.Q<ScrollView>("name")` |
| 리스트뷰 | `<ui:ListView>` | `root.Q<ListView>("name")` |
| 프로그레스바 | `<ui:ProgressBar>` | `root.Q<ProgressBar>("name")` |

## uGUI 도킹 + UI Toolkit 오버레이 패턴

기존 uGUI 도킹 윈도우 시스템에 UI Toolkit을 연동할 때:

1. **uGUI DockSlot** — 빈 RectTransform 패널. 크기/위치만 정의
2. **UI Toolkit dock-container** — DockSlot의 스크린 좌표를 `LateUpdate()`에서 매 프레임 추적
3. **root 배경 투명** — `background-color: rgba(0,0,0,0)` 필수
4. **PanelSettings sortingOrder** — uGUI Canvas보다 높게 설정
5. **pickingMode: Ignore** — dock-container 외 영역은 이벤트 투과시켜 uGUI로 전달
6. **인터랙션 브릿지** — 03-ugui-uitoolkit-bridge.md 패턴 적용

핵심 좌표 변환 (같은 reference resolution일 때 직접 매핑 — ScreenToPanel보다 정확):
```csharp
// LateUpdate()에서 매 프레임
// uGUI: anchor(0.5, 0.5), Y 위가 +
// UI Toolkit: (0,0)=좌상단, Y 아래가 +
float refW = 1920f, refH = 1080f;
float left = refW / 2f + dockSlot.anchoredPosition.x - dockSlot.sizeDelta.x / 2f;
float top  = refH / 2f - dockSlot.anchoredPosition.y - dockSlot.sizeDelta.y / 2f;
dockContainer.style.left = left;
dockContainer.style.top = top;
dockContainer.style.width = dockSlot.sizeDelta.x;
dockContainer.style.height = dockSlot.sizeDelta.y;
```

**주의**: `RuntimePanelUtils.ScreenToPanel()`은 CanvasScaler와 PanelSettings 스케일링 차이로 오프셋이 생길 수 있음. 같은 reference resolution이면 직접 매핑이 더 정확.
7. **타이틀바/리사이즈는 UI Toolkit에 배치** — uGUI에 두면 UI Toolkit 패널에 가려져 클릭 불가

상세 예제: `references/examples/04-dock-sync.md`

## 이벤트 바인딩 패턴

```csharp
// 버튼 클릭
root.Q<Button>("btn").clicked += () => { };

// 값 변경
root.Q<Toggle>("toggle").RegisterValueChangedCallback(evt => { });
root.Q<Slider>("slider").RegisterValueChangedCallback(evt => { });
root.Q<TextField>("field").RegisterValueChangedCallback(evt => { });

// ListView 바인딩
var listView = root.Q<ListView>("list");
listView.makeItem = () => new Label();
listView.bindItem = (element, index) => { };
listView.itemsSource = dataList;
```
