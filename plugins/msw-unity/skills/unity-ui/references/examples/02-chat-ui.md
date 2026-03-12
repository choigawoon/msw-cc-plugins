# 예제 2: 채팅 UI (사이드바 + 메시지 영역)

실제 앱 스크린샷을 재현한 복합 레이아웃 UI. 사이드바, 탭, 채팅 메시지, 입력창.

## 파일 구조

```
Assets/ChatUI/
├── ChatUI.uxml
├── ChatUI.uss
├── ChatUIController.cs
└── Editor/
    └── ChatUISceneSetup.cs
```

## UXML — 레이아웃 구조

```xml
<ui:UXML ...>
    <Style src="ChatUI.uss" />

    <ui:VisualElement class="root">
        <!-- 사이드바 -->
        <ui:VisualElement class="sidebar">
            <ui:Button text="+ 새 채팅" name="new-chat-btn" class="new-chat-btn" />
            <ui:Label text="즐겨찾기" class="section-label" />
            <ui:ScrollView class="favorites-scroll">
                <ui:Button text="항목1" class="sidebar-item fav-item" />
                <!-- ... -->
            </ui:ScrollView>
            <ui:VisualElement class="user-profile">
                <!-- 아바타 + 이름 -->
            </ui:VisualElement>
        </ui:VisualElement>

        <!-- 메인 -->
        <ui:VisualElement class="main-content">
            <!-- 탭 바 -->
            <ui:VisualElement class="top-bar">
                <ui:VisualElement class="tab-group">
                    <ui:Button text="탭1" class="tab-btn tab-active" />
                    <ui:Button text="탭2" class="tab-btn" />
                </ui:VisualElement>
            </ui:VisualElement>

            <!-- 채팅 영역 -->
            <ui:ScrollView name="chat-scroll" class="chat-area" />

            <!-- 입력 -->
            <ui:VisualElement class="input-area">
                <ui:TextField name="chat-input" class="chat-input" />
                <ui:Button text="↑" name="send-btn" class="send-btn" />
            </ui:VisualElement>
        </ui:VisualElement>
    </ui:VisualElement>
</ui:UXML>
```

## USS — 주요 스타일 패턴

```css
/* 사이드바 + 메인 수평 분할 */
.root { flex-direction: row; flex-grow: 1; }
.sidebar { width: 280px; }
.main-content { flex-grow: 1; }

/* 탭 활성화 */
.tab-active {
    background-color: rgb(255, 255, 255);
    -unity-font-style: bold;
}

/* 유저 메시지 오른쪽 정렬 */
.user-row { align-items: flex-end; }
.user-bubble { background-color: rgb(242, 240, 235); border-radius: 16px; }

/* 어시스턴트 메시지 왼쪽 정렬 */
.assistant-row { align-items: flex-start; }
```

## C# — 동적 메시지 추가 패턴

```csharp
private void SendMessage()
{
    var text = chatInput.value?.Trim();
    if (string.IsNullOrEmpty(text)) return;

    // VisualElement를 코드로 생성하여 ScrollView에 추가
    var row = new VisualElement();
    row.AddToClassList("message-row");
    row.AddToClassList("user-row");

    var bubble = new VisualElement();
    bubble.AddToClassList("message-bubble");
    bubble.AddToClassList("user-bubble");

    var label = new Label(text);
    label.AddToClassList("message-text");

    bubble.Add(label);
    row.Add(bubble);
    chatScroll.Add(row);

    // 스크롤 맨 아래로
    chatScroll.schedule.Execute(() =>
        chatScroll.scrollOffset = new Vector2(0, float.MaxValue));
}
```

## C# — 탭 전환 패턴

```csharp
var tabs = root.Query<Button>(className: "tab-btn").ToList();
foreach (var tab in tabs)
{
    tab.clicked += () =>
    {
        foreach (var t in tabs) t.RemoveFromClassList("tab-active");
        tab.AddToClassList("tab-active");
    };
}
```

## C# — Enter 키 입력 처리

```csharp
chatInput.RegisterCallback<KeyDownEvent>(evt =>
{
    if (evt.keyCode == KeyCode.Return && !evt.shiftKey)
    {
        evt.StopPropagation();
        evt.PreventDefault();
        SendMessage();
    }
});
```

## 핵심 패턴

- **flex-direction: row** — 사이드바/메인 수평 분할
- **ScrollView** — 스크롤 가능 영역 (채팅, 즐겨찾기)
- **AddToClassList / RemoveFromClassList** — 동적 스타일 전환
- **코드로 VisualElement 생성** — 런타임 동적 UI
- **schedule.Execute** — 다음 프레임에 스크롤 위치 설정
