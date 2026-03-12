# 예제 4: uGUI 도킹 윈도우 + UI Toolkit 오버레이 싱크

uGUI 도킹 시스템 안에 UI Toolkit을 겹쳐서 사용하는 패턴.
uGUI RectTransform이 위치/크기를 결정하고, UI Toolkit이 그 위에서 렌더링.

## 개념

```
uGUI Canvas (도킹 윈도우 시스템, sortingOrder: 0)
  ├─ Sidebar (uGUI)        ← 기존 uGUI 그대로
  ├─ DockSlot (uGUI)       ← 빈 RectTransform, 위치/크기만 정의
  ├─ RightPanel (uGUI)     ← 기존 uGUI 그대로
  └─ BottomBar (uGUI)      ← 기존 uGUI 그대로

UI Toolkit Panel (전체 화면, sortingOrder: 5)
  └─ root (picking-mode: Ignore)  ← 배경 투명, 이벤트 투과
      └─ dock-container            ← DockSlot 좌표를 매 프레임 추적
          ├─ title-bar             ← 드래그 이동 (UI Toolkit에서 처리)
          ├─ 콘텐츠 영역
          └─ resize-handle         ← 리사이즈 (UI Toolkit에서 처리)
```

## 파일 구조

```
Assets/DockSync/
├── DockSyncUI.uxml            # UI Toolkit 콘텐츠 + 타이틀바 + 리사이즈 핸들
├── DockSyncUI.uss
├── DockSync.cs                # 좌표 싱크 + 드래그/리사이즈 로직
└── Editor/
    └── DockSyncSetup.cs       # 씬 자동 구성
```

## 핵심 난제와 해결법

### 난제 1: UI Toolkit이 uGUI 이벤트를 가로챔

**문제**: UI Toolkit 패널이 전체 화면을 덮어서(sortingOrder 높음) uGUI 버튼 클릭이 안 됨.

**해결**: UXML root에 `picking-mode="Ignore"` 설정. dock-container만 이벤트 수신.

```xml
<ui:VisualElement name="root" class="root" picking-mode="Ignore">
    <ui:VisualElement name="dock-container" class="dock-container">
        <!-- 이 안의 요소만 클릭 가능 -->
    </ui:VisualElement>
</ui:VisualElement>
```

```css
.root {
    background-color: rgba(0, 0, 0, 0); /* 투명 필수 */
}
```

### 난제 2: 타이틀바/리사이즈가 UI Toolkit 아래에 가려짐

**문제**: uGUI에 타이틀바/리사이즈 핸들을 만들면 UI Toolkit 패널 뒤에 가려서 클릭 불가.

**해결**: 타이틀바/리사이즈를 UI Toolkit UXML에 포함시키고, DockSync.cs에서 처리.

```xml
<!-- 드래그 타이틀바 -->
<ui:VisualElement name="title-bar" class="title-bar">
    <ui:Label text=":: Drag to Move ::" picking-mode="Ignore" />
</ui:VisualElement>

<!-- 리사이즈 핸들 (우하단 절대 위치) -->
<ui:VisualElement name="resize-handle" class="resize-handle">
    <ui:Label text="⤡" picking-mode="Ignore" />
</ui:VisualElement>
```

### 난제 3: 좌표계 변환 — ScreenToPanel 대신 직접 매핑

**문제**: `GetWorldCorners()` + `RuntimePanelUtils.ScreenToPanel()` 조합이
CanvasScaler와 PanelSettings의 스케일링 차이로 위치가 어긋남.

**해결**: 둘 다 같은 reference resolution(1920x1080)을 쓰므로 직접 좌표 매핑.

```
uGUI: anchor(0.5, 0.5), anchoredPosition.y 위가 +
UI Toolkit: (0,0) = 좌상단, Y 아래가 +

변환 공식:
  left = refW/2 + anchoredPos.x - width/2
  top  = refH/2 - anchoredPos.y - height/2
```

```csharp
void LateUpdate()
{
    float refW = 1920f;
    float refH = 1080f;
    float w = dockSlot.sizeDelta.x;
    float h = dockSlot.sizeDelta.y;

    float left = refW / 2f + dockSlot.anchoredPosition.x - w / 2f;
    float top = refH / 2f - dockSlot.anchoredPosition.y - h / 2f;

    dockContainer.style.position = Position.Absolute;
    dockContainer.style.left = left;
    dockContainer.style.top = top;
    dockContainer.style.width = w;
    dockContainer.style.height = h;
}
```

**주의**: `RuntimePanelUtils.ScreenToPanel()`은 CanvasScaler 스케일링과 PanelSettings 스케일링의
미묘한 차이로 오프셋이 생길 수 있음. 같은 reference resolution이면 직접 매핑이 더 정확.

### 난제 4: 드래그 시 Y축 반전

**문제 없음**: `Input.mousePosition.y`(위가 +)와 `anchoredPosition.y`(위가 +)는 같은 방향.
`LateUpdate()`에서 Y를 뒤집어서 UI Toolkit에 반영하므로 자연스럽게 동작.

```csharp
void Update()
{
    if (isDragging)
    {
        Vector2 delta = ((Vector2)Input.mousePosition - dragStartMouse) / canvas.scaleFactor;
        dockSlot.anchoredPosition = dragStartPos + delta;
        // LateUpdate()가 자동으로 UI Toolkit 위치 업데이트
    }
}
```

## 드래그/리사이즈 전체 패턴

```csharp
// PointerDown은 UI Toolkit에서 수신
titleBar.RegisterCallback<PointerDownEvent>(evt =>
{
    isDragging = true;
    dragStartMouse = Input.mousePosition;
    dragStartPos = dockSlot.anchoredPosition;
    titleBar.CapturePointer(evt.pointerId);
    evt.StopPropagation();
});

titleBar.RegisterCallback<PointerUpEvent>(evt =>
{
    isDragging = false;
    titleBar.ReleasePointer(evt.pointerId);
});

// Update()에서 Input.mousePosition으로 추적
void Update()
{
    float scaleFactor = canvas.scaleFactor;

    if (isDragging)
    {
        Vector2 delta = ((Vector2)Input.mousePosition - dragStartMouse) / scaleFactor;
        dockSlot.anchoredPosition = dragStartPos + delta;
    }

    if (isResizing)
    {
        Vector2 delta = ((Vector2)Input.mousePosition - resizeStartMouse) / scaleFactor;
        var newSize = resizeStartSize + new Vector2(delta.x, -delta.y);
        newSize.x = Mathf.Max(250, newSize.x);
        newSize.y = Mathf.Max(200, newSize.y);
        dockSlot.sizeDelta = newSize;
    }
}
```

## 리사이즈 자동 반응

uGUI RectTransform의 sizeDelta가 바뀌면 `LateUpdate()`에서 자동으로 UI Toolkit도 따라감.
추가 코드 불필요.

```
리사이즈 핸들 드래그 → dockSlot.sizeDelta 변경
→ LateUpdate()에서 dockContainer.style.width/height 업데이트
→ UI Toolkit flexbox가 내부 콘텐츠 자동 리레이아웃
```

## 주의사항

1. **PanelSettings sortingOrder > Canvas sortingOrder** — UI Toolkit이 위에 렌더링되어야 DockSlot 영역을 덮음
2. **root picking-mode="Ignore"** — dock-container 밖 영역은 이벤트 투과 → uGUI 받음
3. **root background 투명** — `rgba(0,0,0,0)` 필수. 안 하면 uGUI 전체를 가림
4. **LateUpdate 사용** — Update 아닌 LateUpdate. uGUI 레이아웃 계산 완료 후 읽어야 정확
5. **CanvasScaler + PanelSettings 동일 설정** — 둘 다 ScaleWithScreenSize, 같은 referenceResolution
6. **타이틀바/리사이즈는 UI Toolkit에 배치** — uGUI에 두면 UI Toolkit 패널에 가려져서 클릭 불가
7. **카드 등 인터랙티브 요소는 Button으로** — VisualElement는 :active 피드백이 없음

## 인터랙션 브릿지 연동

아까 만든 양방향 드래그앤드롭 브릿지(03-ugui-uitoolkit-bridge.md)를 그대로 적용 가능:
- uGUI 사이드바에서 아이템 드래그 → dock-container 안으로 드롭
- dock-container 안에서 드래그 → uGUI 영역으로 드롭
- Update() + Input.mousePosition 패턴으로 패널 밖 이벤트 처리
