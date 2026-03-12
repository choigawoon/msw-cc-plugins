---
description: Unity 씬 구성 및 GameObject 배치 자동화 (에디터 스크립트 기반). Use when the user asks to create a scene, add GameObjects, or set up a test scene.
---

# Unity 씬 구성 스킬

## 개요

Unity 씬에 GameObject를 배치하고 컴포넌트를 구성합니다. 씬 YAML을 직접 편집하지 않고 에디터 스크립트(MenuItem)를 생성하여 안전하게 씬을 구성합니다.

## 핵심 원칙

**씬 YAML 직접 편집 금지** — Unity 씬 파일(.unity)은 GUID 참조가 복잡하므로, 반드시 에디터 스크립트를 통해 구성합니다.

## 워크플로우

### 1단계: 요구사항 파악

- 씬 이름
- 배치할 오브젝트 (카메라, 라이트, UI, 게임 오브젝트 등)
- 필요한 컴포넌트 (Rigidbody, Collider, 커스텀 스크립트 등)

### 2단계: 빈 씬 파일 생성

최소한의 씬 YAML 생성 (설정만 포함, GameObject 없음):

```yaml
%YAML 1.1
%TAG !u! tag:unity3d.com,2011:
--- !u!29 &1
OcclusionCullingSettings:
  ...
--- !u!104 &2
RenderSettings:
  ...
--- !u!157 &3
LightmapSettings:
  ...
--- !u!196 &4
NavMeshSettings:
  ...
--- !u!1660057539 &9
SceneRoots:
  m_ObjectHideFlags: 0
  m_Roots: []
```

### 3단계: 에디터 세팅 스크립트 생성

```csharp
using UnityEditor;
using UnityEngine;

public static class {SceneName}Setup
{
    [MenuItem("Tools/Setup {SceneName}")]
    public static void SetupScene()
    {
        // 카메라 생성
        // 라이트 생성
        // 게임 오브젝트 생성 + 컴포넌트 추가
        // Prefab 인스턴스화 (AssetDatabase.FindAssets 사용)
    }
}
```

### 4단계: 안내

1. Unity 에디터에서 새 씬 열기 또는 생성된 씬 열기
2. `Tools > Setup {SceneName}` 메뉴 실행
3. Play로 테스트

## 참조 예제

생성 전 반드시 해당 예제를 Read로 로드하여 패턴을 확인:

| 예제 | 경로 | 내용 |
|------|------|------|
| 에디터 씬 구성 | `references/examples/01-editor-scene-setup.md` | MenuItem 패턴, GUID 검색, Undo, 혼합 씬 |

## 에디터 스크립트 패턴

### 카메라 생성
```csharp
var camGo = new GameObject("Main Camera");
camGo.tag = "MainCamera";
var cam = camGo.AddComponent<Camera>();
cam.orthographic = true;
cam.orthographicSize = 5;
cam.transform.position = new Vector3(0, 1, -10);
cam.backgroundColor = new Color(0.12f, 0.12f, 0.18f);
```

### Prefab 인스턴스화
```csharp
var guids = AssetDatabase.FindAssets("PrefabName t:Prefab");
if (guids.Length > 0)
{
    var prefab = AssetDatabase.LoadAssetAtPath<GameObject>(
        AssetDatabase.GUIDToAssetPath(guids[0]));
    var instance = (GameObject)PrefabUtility.InstantiatePrefab(prefab);
    instance.transform.position = Vector3.zero;
    Undo.RegisterCreatedObjectUndo(instance, "Create PrefabName");
}
```

### 컴포넌트 추가
```csharp
var go = new GameObject("MyObject");
go.AddComponent<Rigidbody2D>();
go.AddComponent<BoxCollider2D>();
go.AddComponent<MyCustomScript>();
Undo.RegisterCreatedObjectUndo(go, "Create MyObject");
```

## 규칙

1. **Undo 등록 필수**: 모든 생성 오브젝트에 `Undo.RegisterCreatedObjectUndo` 호출
2. **Selection 설정**: 마지막 생성 오브젝트를 `Selection.activeGameObject`로 설정
3. **에디터 폴더**: 에디터 스크립트는 반드시 `Editor/` 폴더에 배치
4. **GUID 하드코딩 금지**: `AssetDatabase.FindAssets`로 동적 검색
