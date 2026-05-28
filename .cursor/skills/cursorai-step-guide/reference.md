# CursorAI 프로젝트 상세 참조

## 프로젝트별 파일 맵

### 단계 1 — `src/1-1.,MagicSquare_1004`

| 파일 | 목적 |
|------|------|
| `Prompting/README.md` | 3개 프롬프트의 흐름 및 사용 가이드 |
| `Prompting/01.*.md` | 문제 정의 프롬프트 (5 Why, 첫 TC 도출) |
| `Prompting/02.*.md` | Dual-Track TDD + 클린 아키텍처 설계 프롬프트 |
| `Prompting/03.*.md` | .cursorrules 작성 + User 엔티티 구현 프롬프트 |
| `Report/01.*.md` | 문제 정의 보고서 (1~16 배치, 합=34, 10줄 목표) |
| `Report/02.*.md` | 아키텍처 설계 (Grid, MagicSquareValidator, TwoCellSolver) |
| `Report/03.*.md` | Cursor Rules 결정 + User 구현 요약 |
| `src/entity/user.py` | 불변 User 데이터클래스 (user_id, display_name 50자) |
| `src/entity/exceptions/domain_errors.py` | DomainError, InvalidUserError |
| `tests/entity/test_user.py` | User AAA 테스트 10개 |

### 단계 2 — `src/1-2.MagicSquare_1004_개선`

| 파일 | 목적 |
|------|------|
| `src/magicsquare/entity/models/user.py` | email 추가 User (display_name 120자, rename()) |
| `tests/entity/models/test_user.py` | email 검증 포함 11개 테스트 |
| `pyproject.toml` | 정식 패키지 메타데이터 + Black 설정 |

### 단계 3 — `src/2.Refrigerator`

| 파일 | 목적 |
|------|------|
| `PRD_step1.md` | 이미지 → 재료 인식 기획 (OpenRouter Vision) |
| `PRD_step2.md` | 재료 → 레시피 생성 기획 (미구현) |
| `PRD_step3.md` | 인증·레시피 저장 기획 |
| `app/main.py` | FastAPI 진입점, /api/recognize |
| `app/openrouter_client.py` | Vision AI 호출 (재시도 포함) |
| `app/routers/auth.py` | 회원가입, 로그인 (JWT) |
| `app/routers/recipes.py` | 레시피 CRUD (소프트 삭제) |
| `static/index.html` | Step 1 UI (사진 업로드 + 재료 태그) |
| `static/recipes.html` | Step 3 UI (로그인 + 레시피 관리) |
| `scripts/test_openrouter.py` | API 연결 수동 테스트 |

### 단계 4 — `src/4.MagicSquare_1004_完`

| 경로 | 목적 |
|------|------|
| `cursorrules` | SSOT 규칙 파일 (ECB·TDD·금지패턴) |
| `cursor/rules/*.mdc` | `.cursor/rules/`에 복사 시 자동 활성화 |
| `cursor/agents/*.md` | `.cursor/agents/`에 복사 시 `@멘션` 가능 |
| `cursor/skills/magic-square-tdd/` | TDD 워크플로우 스킬 |
| `docs/PRD_MagicSquare.md` | 전체 SSOT 기획서 |
| `src/entity/services/` | EmptyCellLocator, MissingNumberFinder, MagicSquareValidator, TwoCellSolver |
| `src/entity/value_objects/` | CellCoordinate, EmptyCellPair, MagicConstant, SolutionResult 등 |
| `src/control/solve_partial_magic_square.py` | locate→find→solve 오케스트레이션 |
| `src/boundary/ui_boundary.py` | 검증→resolve→응답 반환 |
| `src/boundary/screen/main_window.py` | PyQt6 4×4 그리드 UI |
| `tests/test_gm_01_*.py` | Golden Master 5개 시나리오 E2E |

---

## 마방진 도메인 핵심 계약

```
입력:  4×4 int[][], 빈칸(0) 정확히 2개, 값은 0 또는 1~16, 중복 금지
출력:  int[6] = [row1, col1, val1, row2, col2, val2]  ← 좌표 1-index
```

**TwoCellSolver 전략**:
- Step A: smaller → firstEmpty, larger → secondEmpty → 마방진 성립 시 반환
- Step B: larger → firstEmpty, smaller → secondEmpty → 마방진 성립 시 반환
- 둘 다 실패 → `UnsolvableDomainError` → Boundary E006 에러 응답

**에러 코드**:
| 코드 | 의미 |
|------|------|
| INVALID_SIZE | None 또는 4×4 아닌 입력 |
| E002 | 빈칸이 정확히 2개가 아님 |
| E004 | 범위 밖 값 (1~16 또는 0 외) |
| E005 | 중복 숫자 |
| E006 | 해가 없는 경우 |

---

## TC 명세 요약 (`magic_square_tc.xlsx`)

| 분류 | 건수 | 대상 함수 |
|------|------|-----------|
| A. 빈칸 찾기 | 5개 | `find_blank_coords` |
| B. 마방진 검증 | 8개 | `is_magic_square` |
| C. 빈칸 채우기·경계값 | 5개 | `solution` |
| D. 오류·예외 | 6개 | 전체 |
| **합계** | **24개** | — |

---

## Cursor 에이전트 역할 (4.完 기준)

| Agent | 담당 | 주요 행동 |
|-------|------|-----------|
| `@backend-developer` | entity, control (Logic Track) | TDD RED/GREEN/REFACTOR 구현 |
| `@frontend-developer` | boundary UI (PyQt6) | 화면 컴포넌트 구현 |
| `@quality-assurance-engineer` | 커버리지·Golden Master | 회귀 분석, 80% 달성 |
| `@product-planning-manager` | 백로그·TC 관리 | 다음 RED 테스트 선정, DoD 정의 |
| `@ai-integration-expert` | OpenRouter, LLM 연동 | Vision/Text API 설계 |
| `@system-optimization-engineer` | 성능·구조 개선 | REFACTOR 단계 주도 |
| `@ux-design-advisor` | UI/UX 개선 | PyQt6 사용성 검토 |
| `@backup-report-github-manager` | 보고서·Git 관리 | Report 업데이트, 커밋 |

---

## 단계별 핵심 Cursor 프롬프트 패턴

### 문제 정의 (단계 1)
```
이 프로그램이 해결하는 문제는 무엇인가?
사용자가 겪는 불편함을 5 Why로 분석해줘.
첫 번째 테스트 가능한 요구사항을 하나만 도출해줘.
```

### TDD 설계 (단계 1)
```
Dual-Track TDD로 Logic Track과 UI Contract Track을 분리해서 설계해줘.
ECB 레이어별 책임을 정의하고 테스트 ID 체계를 만들어줘.
```

### 구현 (단계 4)
```
Phase: red | Layer: entity | Track: logic
D-VAL-01 테스트를 AAA 패턴으로 작성해줘. production 코드 없이 RED만.
```

### Golden Master
```
python scripts/generate_golden_master.py
pytest tests/test_gm_01_magic_square_golden_master.py -v
```
