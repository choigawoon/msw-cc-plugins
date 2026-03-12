# 예제 1: 기본 카운터 + 설정 UI

간단한 UI Toolkit 샘플. 카운터, 텍스트 입력, 설정 패널.

## 파일 구조

```
Assets/UIToolkitSample/
├── SampleUI.uxml
├── SampleUI.uss
├── SampleUIController.cs
└── Editor/
    └── UIToolkitSceneSetup.cs
```

## UXML

```xml
<ui:UXML xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:ui="UnityEngine.UIElements"
         xsi:noNamespaceSchemaLocation="../UIElementsSchema/UIElements.xsd">

    <Style src="SampleUI.uss" />

    <ui:VisualElement class="root-container">
        <ui:Label text="UI Toolkit Sample" class="title" />

        <!-- 카운터 카드 -->
        <ui:VisualElement class="card">
            <ui:Label text="Counter: 0" name="counter-label" class="counter-text" />
            <ui:VisualElement class="button-row">
                <ui:Button text="-" name="decrement-btn" class="btn btn-secondary" />
                <ui:Button text="+" name="increment-btn" class="btn btn-primary" />
            </ui:VisualElement>
        </ui:VisualElement>

        <!-- 인사 카드 -->
        <ui:VisualElement class="card">
            <ui:TextField label="Your Name" name="name-field" class="input-field" />
            <ui:Button text="Say Hello" name="hello-btn" class="btn btn-primary" />
            <ui:Label text="" name="greeting-label" class="greeting-text" />
        </ui:VisualElement>

        <!-- 설정 카드 -->
        <ui:VisualElement class="card">
            <ui:Label text="Settings" class="card-title" />
            <ui:Toggle label="Enable Sound" name="sound-toggle" value="true" />
            <ui:SliderInt label="Volume" name="volume-slider"
                          low-value="0" high-value="100" value="75" />
            <ui:DropdownField label="Difficulty" name="difficulty-dropdown"
                              index="1" choices="Easy,Normal,Hard" />
        </ui:VisualElement>
    </ui:VisualElement>
</ui:UXML>
```

## C# 컨트롤러

```csharp
using UnityEngine;
using UnityEngine.UIElements;

public class SampleUIController : MonoBehaviour
{
    private int counter;
    private Label counterLabel;
    private Label greetingLabel;

    void OnEnable()
    {
        var root = GetComponent<UIDocument>().rootVisualElement;

        // 카운터
        counterLabel = root.Q<Label>("counter-label");
        root.Q<Button>("increment-btn").clicked += () => UpdateCounter(counter + 1);
        root.Q<Button>("decrement-btn").clicked += () => UpdateCounter(counter - 1);

        // 인사
        var nameField = root.Q<TextField>("name-field");
        greetingLabel = root.Q<Label>("greeting-label");
        root.Q<Button>("hello-btn").clicked += () =>
        {
            var name = string.IsNullOrWhiteSpace(nameField.value) ? "World" : nameField.value;
            greetingLabel.text = $"Hello, {name}!";
        };

        // 설정 (값 변경 로그)
        root.Q<Toggle>("sound-toggle").RegisterValueChangedCallback(evt =>
            Debug.Log($"Sound: {evt.newValue}"));
        root.Q<SliderInt>("volume-slider").RegisterValueChangedCallback(evt =>
            Debug.Log($"Volume: {evt.newValue}"));
        root.Q<DropdownField>("difficulty-dropdown").RegisterValueChangedCallback(evt =>
            Debug.Log($"Difficulty: {evt.newValue}"));
    }

    private void UpdateCounter(int value)
    {
        counter = value;
        counterLabel.text = $"Counter: {counter}";
    }
}
```

## 핵심 패턴

- `<Style src="파일.uss" />` — UXML 안에서 USS 직접 참조
- `root.Q<T>("name")` — name 속성으로 요소 검색
- `.clicked += () => {}` — 버튼 이벤트
- `.RegisterValueChangedCallback()` — 값 변경 이벤트
