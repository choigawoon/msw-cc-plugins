---
name: unity-setup
description: Unity 프로젝트 초기 세팅 (.gitignore, 패키지 설정, 프로젝트 구조)
triggers:
  - "Unity 프로젝트"
  - "gitignore"
  - ".gitignore"
  - "Unity 초기 설정"
  - "Unity 셋업"
  - "프로젝트 세팅"
  - "패키지 추가"
---

# Unity 프로젝트 초기 세팅 스킬

## 개요

Unity 프로젝트의 초기 설정을 자동화합니다. .gitignore 생성, 패키지 관리, 프로젝트 구조 확인 등을 수행합니다.

## 워크플로우

### 1단계: 프로젝트 확인

1. `ProjectSettings/ProjectVersion.txt`로 Unity 버전 확인
2. `Packages/manifest.json`으로 설치된 패키지 확인
3. 기존 .gitignore 존재 여부 확인

### 2단계: .gitignore 생성/업데이트

Unity 프로젝트용 .gitignore 표준 항목:

```gitignore
# Unity generated
/[Ll]ibrary/
/[Tt]emp/
/[Oo]bj/
/[Bb]uild/
/[Bb]uilds/
/[Ll]ogs/
/[Uu]serSettings/
/[Mm]emoryCaptures/
/[Rr]ecordings/

# Asset meta (keep)
!/[Aa]ssets/**/*.meta

# Crash reports
sysinfo.txt

# Build artifacts
*.apk
*.aab
*.unitypackage
*.app

# IDE
.vs/
*.csproj
*.unityproj
*.sln
*.suo
*.tmp
*.user
*.userprefs
*.pidb
*.booproj
*.svd
*.pdb
*.mdb
*.opendb
*.VC.db
.idea/
*.sln.iml
.vscode/

# OS
.DS_Store
*.swp
.Trashes
._*

# Plastic SCM
ignore.conf
*.private.0
*.private
```

### 3단계: 패키지 확인/추가

사용자 요구에 따라 `Packages/manifest.json`에 패키지 추가:

| 기능 | 패키지 |
|------|--------|
| UI Toolkit | `com.unity.modules.uielements` (내장) |
| TextMeshPro | `com.unity.textmeshpro` |
| Cinemachine | `com.unity.cinemachine` |
| Input System | `com.unity.inputsystem` |
| Addressables | `com.unity.addressables` |
| URP | `com.unity.render-pipelines.universal` |

### 4단계: Git 초기화

- `git init` (필요 시)
- 초기 커밋 생성

## 참조 예제

생성 전 반드시 해당 예제를 Read로 로드하여 패턴을 확인:

| 예제 | 경로 | 내용 |
|------|------|------|
| .gitignore 템플릿 | `references/examples/01-gitignore-template.md` | 표준 .gitignore, 패키지 체크리스트 |

## 프로젝트 구조 검증

확인할 항목:
- `Assets/` 디렉토리 존재
- `ProjectSettings/` 디렉토리 존재
- `Packages/manifest.json` 존재
- Unity 버전 호환성

## 규칙

1. **기존 .gitignore 존중**: 이미 있으면 병합 제안, 덮어쓰기 금지
2. **manifest.json 편집 시 주의**: 기존 패키지 제거하지 않고 추가만 수행
3. **버전 호환성**: Unity 버전에 맞는 패키지 버전 사용
