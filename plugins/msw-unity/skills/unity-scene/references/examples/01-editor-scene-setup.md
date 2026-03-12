# 예제: 에디터 스크립트로 씬 자동 구성

씬 YAML을 직접 편집하지 않고 에디터 MenuItem 스크립트로 안전하게 구성하는 패턴.

## 왜 에디터 스크립트인가?

Unity 씬 파일(.unity)은 YAML 직렬화이지만:
- **GUID 참조** — 모든 에셋/스크립트가 GUID로 연결됨. 에디터가 자동 생성하는 값이라 수동 지정 불가
- **fileID** — 같은 파일 내 오브젝트 간 참조. 충돌 위험
- **serializedVersion** — Unity 버전마다 다름

**결론**: 씬 YAML 직접 편집 = missing script, broken reference 위험. 에디터 API 사용이 안전.

## 기본 패턴

```csharp
using UnityEditor;
using UnityEngine;
using UnityEngine.UIElements;

public static class MySceneSetup
{
    [MenuItem("Tools/Setup My Scene")]
    public static void SetupScene()
    {
        // 1. 에셋 검색 (GUID 하드코딩 대신 동적 검색)
        var uxmlGuids = AssetDatabase.FindAssets("MyUI t:VisualTreeAsset");
        var uxml = AssetDatabase.LoadAssetAtPath<VisualTreeAsset>(
            AssetDatabase.GUIDToAssetPath(uxmlGuids[0]));

        // 2. PanelSettings 생성 또는 검색
        var panelGuids = AssetDatabase.FindAssets("t:PanelSettings",
            new[] { "Assets/MyUI" });
        PanelSettings panelSettings;
        if (panelGuids.Length > 0)
        {
            panelSettings = AssetDatabase.LoadAssetAtPath<PanelSettings>(
                AssetDatabase.GUIDToAssetPath(panelGuids[0]));
        }
        else
        {
            panelSettings = ScriptableObject.CreateInstance<PanelSettings>();
            panelSettings.scaleMode = PanelScaleMode.ScaleWithScreenSize;
            panelSettings.referenceResolution = new Vector2Int(1920, 1080);
            AssetDatabase.CreateAsset(panelSettings, "Assets/MyUI/PanelSettings.asset");
        }

        // 3. GameObject 생성 + 컴포넌트 연결
        var go = new GameObject("MyUI");
        var uiDoc = go.AddComponent<UIDocument>();
        uiDoc.panelSettings = panelSettings;
        uiDoc.visualTreeAsset = uxml;
        go.AddComponent<MyUIController>();

        // 4. Undo 등록 (Ctrl+Z로 되돌리기 가능)
        Undo.RegisterCreatedObjectUndo(go, "Create MyUI");

        // 5. 선택 상태로
        Selection.activeGameObject = go;

        Debug.Log("Setup complete! Press Play to test.");
    }
}
```

## uGUI + UI Toolkit 혼합 씬

```csharp
[MenuItem("Tools/Setup Hybrid Scene")]
public static void SetupHybridScene()
{
    // UI Toolkit (sortingOrder 낮음)
    var uitkGo = new GameObject("UIToolkit_Layer");
    var uiDoc = uitkGo.AddComponent<UIDocument>();
    // panelSettings.sortingOrder = 0;

    // uGUI Canvas (sortingOrder 높음 — 위에 렌더링)
    var canvasGo = new GameObject("uGUI_Layer");
    var canvas = canvasGo.AddComponent<Canvas>();
    canvas.renderMode = RenderMode.ScreenSpaceOverlay;
    canvas.sortingOrder = 10;
    canvasGo.AddComponent<CanvasScaler>();
    canvasGo.AddComponent<GraphicRaycaster>();

    // EventSystem (uGUI 필수)
    if (FindFirstObjectByType<EventSystem>() == null)
    {
        var esGo = new GameObject("EventSystem");
        esGo.AddComponent<EventSystem>();
        esGo.AddComponent<StandaloneInputModule>();
    }
}
```

## Prefab 인스턴스화

```csharp
var guids = AssetDatabase.FindAssets("MyPrefab t:Prefab");
if (guids.Length > 0)
{
    var prefab = AssetDatabase.LoadAssetAtPath<GameObject>(
        AssetDatabase.GUIDToAssetPath(guids[0]));
    var instance = (GameObject)PrefabUtility.InstantiatePrefab(prefab);
    instance.transform.position = Vector3.zero;
    Undo.RegisterCreatedObjectUndo(instance, "Create MyPrefab");
}
```

## 필수 규칙

1. **`AssetDatabase.FindAssets`** — GUID 하드코딩 금지, 항상 동적 검색
2. **`Undo.RegisterCreatedObjectUndo`** — 모든 생성 오브젝트에 Undo 등록
3. **`Editor/` 폴더** — 에디터 스크립트는 반드시 Editor 폴더에 배치
4. **`Selection.activeGameObject`** — 마지막 생성 오브젝트를 선택 상태로
