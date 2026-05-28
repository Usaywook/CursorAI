---
name: cursorai-step-guide
description: >-
  CursorAI 학습 프로젝트(MagicSquare 1-1·1-2·Refrigerator·4.完)의 단계별 진행 절차 안내.
  각 단계에서 무엇을 해야 하고 어떻게 하는지, Cursor 기능(Rules·Agents·Skills·Auto-run)을
  어떻게 활용하는지 가이드한다.
  "다음에 뭐해야 해", "어떻게 시작해", "이 단계 순서가 뭐야" 등의 질문에 사용.
---

# CursorAI 학습 프로젝트 — 단계별 진행 가이드

## 전체 학습 흐름

```
[단계 1] 1-1 워밍업    → [단계 2] 1-2 개선    → [단계 3] Refrigerator    → [단계 4] 4.完
문제 정의·설계·엔티티      패키지 리팩터         실전 웹앱 구현             전체 완성·Golden Master
```

상세 절차는 [reference.md](reference.md) 참조.

---

## 단계 1 — `src/1-1.,MagicSquare_1004` (워밍업)

**목표**: 문제 정의 → Dual-Track TDD 설계 → Cursor Rules 작성 → 첫 엔티티 구현

```
Task Progress:
- [ ] 1-1. Prompting/01 스크립트로 AI와 문제 정의 대화 → Report/01 생성
- [ ] 1-2. Prompting/02 스크립트로 Dual-Track TDD 설계 → Report/02 생성
- [ ] 1-3. Prompting/03 스크립트로 .cursorrules 설계 → Report/03 생성
- [ ] 1-4. src/entity/user.py User 엔티티 RED 테스트 작성
- [ ] 1-5. pytest 실행 → RED 확인
- [ ] 1-6. User 엔티티 GREEN 구현
- [ ] 1-7. pytest GREEN 확인
```

**Cursor 기능 활용**:
- `Prompting/README.md`의 프롬프트를 Cursor Chat에 그대로 붙여넣기
- Report 생성 후 `.cursorrules` 파일로 저장 → 이후 세션에 자동 적용

---

## 단계 2 — `src/1-2.MagicSquare_1004_개선` (코드 개선)

**목표**: 1-1 코드를 namespaced 패키지로 리팩터, User에 email 추가

```
Task Progress:
- [ ] 2-1. 1-1과 1-2의 패키지 구조 차이 비교 (entity/ vs magicsquare/entity/models/)
- [ ] 2-2. pyproject.toml에 [project] 메타데이터·Black 설정 추가
- [ ] 2-3. email 필드 추가 + 검증 로직 (소문자 정규화, shape 검증)
- [ ] 2-4. rename() 메서드 구현 (with_display_name 대체)
- [ ] 2-5. 기존 10개 테스트 유지 + email 검증 테스트 추가
- [ ] 2-6. pytest 전체 GREEN 확인
```

**핵심 차이점**:
- import 경로: `from entity.user import User` → `from magicsquare.entity.models.user import User`
- 예외: 별도 `exceptions/` → `user.py` 내부 `InvalidUserError(ValueError)`

---

## 단계 3 — `src/2.Refrigerator` (실전 웹앱)

**목표**: FastAPI + OpenRouter Vision + JWT 인증 3단계 웹앱 구현

### Step 1 — 이미지 인식 (구현됨)

```
Task Progress:
- [ ] 3-1-1. .env 파일 생성: OPENROUTER_API_KEY=sk-or-...
- [ ] 3-1-2. pip install -r requirements.txt
- [ ] 3-1-3. python -m uvicorn app.main:app --port 8000
- [ ] 3-1-4. http://127.0.0.1:8000 → 냉장고 사진 업로드 테스트
- [ ] 3-1-5. scripts/test_openrouter.py로 API 연결 확인
```

### Step 2 — 레시피 생성 (미구현 — 다음 과제)

```
Task Progress:
- [ ] 3-2-1. PRD_step2.md 읽기 (deepseek 모델, 응답 스키마 확인)
- [ ] 3-2-2. app/routers/recipes_gen.py 생성 (POST /api/generate-recipes)
- [ ] 3-2-3. openrouter_client.py에 call_text() 함수 추가
- [ ] 3-2-4. index.html → recipes.html 연결 (Step1 결과 → Step2 입력)
```

### Step 3 — 인증·저장 (구현됨)

```
Task Progress:
- [ ] 3-3-1. recipes.html에서 회원가입/로그인 동작 확인
- [ ] 3-3-2. 레시피 저장·목록·삭제 API 확인
- [ ] 3-3-3. pytest 테스트 작성 (현재 테스트 없음)
```

---

## 단계 4 — `src/4.MagicSquare_1004_完` (완성 캡스톤)

**목표**: ECB 전체 + Dual-Track TDD + PyQt UI + Golden Master

```
Task Progress:
- [ ] 4-1. cursor/rules/ → .cursor/rules/ 복사 (규칙 활성화)
- [ ] 4-2. cursor/agents/ → .cursor/agents/ 복사 (에이전트 활성화)
- [ ] 4-3. pip install -e ".[gui]" (PyQt6 포함 설치)
- [ ] 4-4. pytest tests/entity → Logic Track GREEN 확인
- [ ] 4-5. pytest tests/boundary → UI Track 확인 (일부 RED 정상)
- [ ] 4-6. RED 스켈레톤 테스트 하나씩 GREEN으로 올리기
- [ ] 4-7. pytest --cov=src → 커버리지 80% 달성
- [ ] 4-8. pytest tests/test_gm_01_* → Golden Master 확인
- [ ] 4-9. python boundary.screen.app → PyQt UI 실행
```

**Cursor 에이전트 활용** (`.cursor/agents/` 복사 후):
- `@backend-developer` : entity, control Logic Track 구현
- `@quality-assurance-engineer` : 커버리지·Golden Master 분석
- `@product-planning-manager` : 다음 RED 테스트 선정

---

## Cursor 기능 빠른 참조

| 기능 | 설정 위치 | 효과 |
|------|-----------|------|
| Rules (자동 주입) | `.cursor/rules/*.mdc` | 모든 AI 대화에 규칙 자동 포함 |
| Agents (역할 호출) | `.cursor/agents/*.md` | `@에이전트명`으로 역할 전환 |
| Skills (절차 가이드) | `.cursor/skills/*/SKILL.md` | `/스킬명`으로 워크플로우 안내 |
| Auto-run (YOLO) | Cursor Settings → Agents | 터미널 명령 자동 실행 |
| Auto-save | settings.json | 파일 자동 저장 |

---

## 단계 전환 체크리스트

다음 단계로 넘어가기 전 확인:

- [ ] 현재 단계 pytest 전체 GREEN
- [ ] Report 또는 PRD에 설계 내용 정리됨
- [ ] README 또는 Report에 다음 단계 TODO 기록됨
- [ ] (4.完만) 커버리지 80% 이상, Golden Master 통과
