# Sessions

Claude Code 대화 세션 기록.

## 2026-03-12: Unity UI Toolkit 세션

**파일**: `2026-03-12_unity-uitoolkit-session.jsonl`

이 세션에서 수행한 작업:

1. Unity 2022 프로젝트 .gitignore 생성 및 초기 커밋
2. UI Toolkit 사용 가능 여부 확인 (com.unity.modules.uielements)
3. 기본 UI Toolkit 샘플 (카운터, 인사, 설정 패널)
4. 스크린샷 기반 채팅 UI 재현 (사이드바, 탭, 동적 메시지)
5. uGUI ↔ UI Toolkit 양방향 드래그앤드롭 브릿지
   - 좌표 변환 (RuntimePanelUtils.ScreenToPanel)
   - 패널 밖 렌더링 (uGUI 고스트 Canvas)
   - 패널 밖 이벤트 (Update + Input.mousePosition)
6. uGUI 도킹 윈도우 + UI Toolkit 오버레이 싱크
   - 직접 좌표 매핑 (ScreenToPanel 대신)
   - picking-mode: Ignore로 이벤트 투과
   - 타이틀바/리사이즈를 UI Toolkit에 배치
7. msw-unity 플러그인 생성 (3 skills, 6 reference examples)
