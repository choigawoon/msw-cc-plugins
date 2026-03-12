# 예제 3: uGUI ↔ UI Toolkit 양방향 드래그앤드롭 브릿지

두 UI 시스템 간 아이템 이동. 핵심 난제와 해결법을 정리.

## 파일 구조

```
Assets/DragDropBridge/
├── ItemData.cs                    # ScriptableObject 아이템 데이터
├── DraggableItem.cs               # uGUI 드래그 핸들러
├── UGUIToUIToolkitBridge.cs       # uGUI → UI Toolkit 브릿지
├── UIToolkitToUGUIBridge.cs       # UI Toolkit → uGUI 브릿지
├── DropZoneUI.uxml                # UI Toolkit 드롭 영역
├── DropZoneUI.uss
└── Editor/
    └── DragDropBridgeSetup.cs     # 씬 자동 구성
```

## 아키텍처

```
┌───────────────────┬───────────────────┐
│  uGUI Canvas      │  UI Toolkit Panel │
│  sortingOrder: 10 │  sortingOrder: 0  │
│                   │                   │
│  DraggableItem    │  DropZone         │
│  (IDragHandler)   │  (PointerDown)    │
│        │          │        │          │
│        ▼          │        ▼          │
│  OnEndDrag ───────┼→ TryDropItem()    │
│                   │                   │
│  CreateUGUIItem ←─┼── CompleteDrop()  │
└───────────────────┴───────────────────┘
```

## 난제 1: 좌표 변환

uGUI와 UI Toolkit은 좌표계가 다르다:
- **uGUI Screen**: 좌하단 원점 (Input.mousePosition)
- **UI Toolkit Panel**: 좌상단 원점

```csharp
// uGUI → UI Toolkit 좌표 변환
var localPos = RuntimePanelUtils.ScreenToPanel(
    uiDocument.rootVisualElement.panel, screenPosition);

// 영역 히트 테스트
var worldBound = dropZone.worldBound;
if (worldBound.Contains(localPos)) { /* 드롭 수락 */ }
```

## 난제 2: UI Toolkit 패널 밖 렌더링 불가

**문제**: UI Toolkit VisualElement는 자기 패널 영역 밖으로 그려지지 않음.
드래그 고스트가 경계에서 잘림.

**해결**: 드래그 시작 시 uGUI Canvas(sortingOrder=999)에 고스트 Image 생성.

```csharp
private void CreateGhost()
{
    var canvasGo = new GameObject("DragGhostCanvas");
    ghostCanvas = canvasGo.AddComponent<Canvas>();
    ghostCanvas.renderMode = RenderMode.ScreenSpaceOverlay;
    ghostCanvas.sortingOrder = 999;
    // CanvasScaler 안 붙임 — 픽셀 단위 직접 제어

    var ghostGo = new GameObject("Ghost", typeof(RectTransform));
    ghostGo.transform.SetParent(canvasGo.transform, false);
    ghostRect = ghostGo.GetComponent<RectTransform>();
    ghostRect.sizeDelta = new Vector2(100, 100);
    // ...
}
```

## 난제 3: UI Toolkit 패널 밖에서 이벤트 끊김

**문제**: 마우스가 UI Toolkit 패널 영역을 벗어나면 PointerMoveEvent가 안 옴.

**해결**: PointerDown만 UI Toolkit에서 받고, 이동/릴리즈는 Update()에서 Input API로 처리.

```csharp
void OnEnable()
{
    // PointerDown만 UI Toolkit에서 수신
    root.RegisterCallback<PointerDownEvent>(OnPointerDown, TrickleDown.TrickleDown);
}

void Update()
{
    if (!isDragging) return;

    // Update()에서 마우스 추적 — 패널 밖에서도 동작
    if (ghostRect != null)
        ghostRect.position = Input.mousePosition;

    if (Input.GetMouseButtonUp(0))
        CompleteDrop();
}
```

## uGUI → UI Toolkit 방향 (DraggableItem.cs)

```csharp
public class DraggableItem : MonoBehaviour,
    IBeginDragHandler, IDragHandler, IEndDragHandler
{
    public void OnEndDrag(PointerEventData eventData)
    {
        var bridge = FindFirstObjectByType<UGUIToUIToolkitBridge>();
        if (bridge != null && bridge.TryDropItem(
            eventData.position, itemName, itemColor))
        {
            Destroy(gameObject); // uGUI에서 제거
            return;
        }
        // 복귀
        transform.SetParent(originalParent);
        rectTransform.anchoredPosition = originalPosition;
    }
}
```

## UI Toolkit → uGUI 방향 (UIToolkitToUGUIBridge.cs)

```csharp
private void CompleteDrop()
{
    float normalizedX = Input.mousePosition.x / Screen.width;

    if (normalizedX < 0.48f) // uGUI 영역
    {
        CreateUGUIItem(dragItemName, dragItemColor);
        dragTarget.RemoveFromHierarchy(); // UI Toolkit에서 제거
    }
    else
    {
        dragTarget.style.opacity = 1f; // 복귀
    }
}
```

## 에디터 세팅 스크립트 패턴

두 시스템을 한 씬에 공존시키려면:

```csharp
// UI Toolkit
var uitkGo = new GameObject("UIToolkit");
var uiDoc = uitkGo.AddComponent<UIDocument>();
uiDoc.panelSettings = panelSettings; // sortingOrder: 0

// uGUI
var canvasGo = new GameObject("uGUI");
var canvas = canvasGo.AddComponent<Canvas>();
canvas.sortingOrder = 10; // UI Toolkit 위에 렌더링

// 양방향 브릿지
uitkGo.AddComponent<UGUIToUIToolkitBridge>();
uitkGo.AddComponent<UIToolkitToUGUIBridge>();
```

## 핵심 정리

| 문제 | 해결법 |
|------|--------|
| 좌표 변환 | `RuntimePanelUtils.ScreenToPanel()` |
| 패널 밖 렌더링 | uGUI Canvas(sortingOrder=999) 고스트 |
| 패널 밖 이벤트 끊김 | `Update()` + `Input.mousePosition` |
| sortingOrder 관리 | uGUI(10) > UI Toolkit(0), 고스트(999) |
| 아이템 이동 시 재생성 | Destroy + 상대 시스템에서 new 생성 |
