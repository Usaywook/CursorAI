---
name: rbac-git-workflow
description: >-
  RBAC TDD 5단계(GIT): 브랜치 생성, 단계별 커밋(RED/GREEN/REFACTOR), C2C Traceability 확인, PR 생성.
  "브랜치 만들어줘", "단계별 커밋", "PR 요청", "C2C 추적", "git commit RBAC" 작업 시 사용.
---

# RBAC Git 워크플로우 (GIT Phase)

## 전체 흐름

```
1. 브랜치 생성
2. RED 커밋 (테스트 파일)
3. GREEN 커밋 (최소 구현)
4. REFACTOR 커밋 (구조 개선)
5. C2C Traceability 확인
6. PR 생성
```

## Step 1 — 브랜치 생성

```bash
git checkout main
git pull origin main
git checkout -b feat/rbac-tdd-implementation
```

## Step 2 — RED 커밋 (단계별)

```bash
# 2-1. Entity 실패 테스트
git add tests/rbac/entity/
git commit -m "$(cat <<'EOF'
test(entity): RB-ENT-01~05 Role·Permission·User 실패 테스트

TestID: RB-ENT-01, RB-ENT-02, RB-ENT-03, RB-ENT-04, RB-ENT-05
Phase: red
Layer: entity

- Role 생성·Permission 할당 실패 테스트
- 빈 permissions Role 거부 실패 테스트
- User Role 할당 실패 테스트
EOF
)"

# 2-2. Control 실패 테스트
git add tests/rbac/control/
git commit -m "$(cat <<'EOF'
test(control): RB-CTL-01~05 권한 거부/허용 실패 테스트

TestID: RB-CTL-01, RB-CTL-02, RB-CTL-03, RB-CTL-04, RB-CTL-05
Phase: red
Layer: control

- viewer DELETE → PermissionDeniedError(403) 실패 테스트
- admin DELETE → True 반환 실패 테스트
EOF
)"

# 2-3. Boundary 실패 테스트
git add tests/rbac/boundary/
git commit -m "$(cat <<'EOF'
test(boundary): RB-BND-01~03 메뉴 가시성 실패 테스트

TestID: RB-BND-01, RB-BND-02, RB-BND-03
Phase: red
Layer: boundary

- viewer 메뉴 가시성 제한 실패 테스트
- admin 전체 메뉴 허용 실패 테스트
EOF
)"
```

## Step 3 — GREEN 커밋 (단계별)

```bash
# 3-1. Entity 최소 구현
git add src/rbac_tdd/entity/ src/rbac_tdd/exceptions/
git commit -m "$(cat <<'EOF'
feat(entity): RB-ENT-01~05 Role·Permission·User 최소 구현

TestID: RB-ENT-01, RB-ENT-02, RB-ENT-03, RB-ENT-04, RB-ENT-05
Phase: green
Layer: entity

- Permission Enum (VIEW/CREATE/UPDATE/DELETE)
- Role frozen dataclass (name, permissions, 최소 1개 필수)
- User frozen dataclass (user_id, roles, all_permissions())
EOF
)"

# 3-2. Control 최소 구현
git add src/rbac_tdd/control/
git commit -m "$(cat <<'EOF'
feat(control): RB-CTL-01~05 has_permission 최소 구현

TestID: RB-CTL-01, RB-CTL-02, RB-CTL-03, RB-CTL-04, RB-CTL-05
Phase: green
Layer: control

- PermissionChecker.has_permission()
- PermissionChecker.require_permission() → PermissionDeniedError
EOF
)"

# 3-3. Boundary 최소 구현
git add src/rbac_tdd/boundary/
git commit -m "$(cat <<'EOF'
feat(boundary): RB-BND-01~03 메뉴 가시성 딕셔너리 최소 구현

TestID: RB-BND-01, RB-BND-02, RB-BND-03
Phase: green
Layer: boundary

- MENU_PERMISSION_MAP 딕셔너리
- MenuVisibility.get_visible_menus()
EOF
)"
```

## Step 4 — REFACTOR 커밋 (단계별)

```bash
git add src/rbac_tdd/entity/role.py
git commit -m "$(cat <<'EOF'
refactor(entity): RB-ENT 역할 계층 구조 (parent 상속) 도입

TestID: RB-ENT-01~05
Phase: refactor
Layer: entity

- Role.parent → 상위 Role 참조
- Role.effective_permissions() → 상속 포함 권한 집합
EOF
)"

git add src/rbac_tdd/boundary/auth_middleware.py src/rbac_tdd/control/role_repository.py
git commit -m "$(cat <<'EOF'
refactor(boundary): RB-BND-02 JWT claim 기반 Role 읽기 구조

TestID: RB-BND-02
Phase: refactor
Layer: boundary, control

- RoleRepository 인터페이스 (control)
- JWTAuthMiddleware.decode_user() (boundary)
EOF
)"
```

## Step 5 — C2C Traceability 최종 확인

```bash
pytest tests/rbac --tb=short                          # 전체 GREEN
pytest --cov=src/rbac_tdd --cov-report=term-missing   # 커버리지 80%+
git log --oneline feat/rbac-tdd-implementation        # 단계별 커밋 이력
```

## Step 6 — PR 생성

```bash
git push -u origin feat/rbac-tdd-implementation
gh pr create \
  --title "feat(rbac): RBAC TDD 구현 — C2C Traceability RED→GREEN→REFACTOR" \
  --body "$(cat <<'EOF'
## 개요

Role·Permission·User ECB Entity 기반 RBAC를 Dual-Track TDD로 구현.
역할 개념→실패 테스트→최소 구현→리팩터링까지 의미 소실 없이 추적 가능.

## C2C Traceability

| 역할 개념 | TestID | 테스트 함수 | 구현 파일 | Phase |
|-----------|--------|-------------|-----------|-------|
| viewer는 DELETE 불가 | RB-CTL-01 | test_rb_ctl_01_viewer_cannot_delete | control/permission_checker.py | GREEN |
| admin은 DELETE 가능 | RB-CTL-02 | test_rb_ctl_02_admin_can_delete | control/permission_checker.py | GREEN |
| 메뉴 가시성 제어 | RB-BND-01 | test_rb_bnd_01_viewer_sees_limited_menus | boundary/menu_visibility.py | GREEN |
| JWT Role 읽기 | RB-BND-02 | test_rb_bnd_02_jwt_role_extraction | boundary/auth_middleware.py | REFACTOR |
| 역할 계층 구조 | RB-ENT-01 | test_rb_ent_01_role_hierarchy | entity/role.py | REFACTOR |

## 단계별 커밋 이력

| Phase | TestID | 설명 |
|-------|--------|------|
| RED | RB-ENT-01~05 | Role·Permission·User 실패 테스트 |
| RED | RB-CTL-01~05 | 권한 거부/허용 실패 테스트 |
| RED | RB-BND-01~03 | 메뉴 가시성 실패 테스트 |
| GREEN | RB-ENT-01~05 | Entity 최소 구현 |
| GREEN | RB-CTL-01~05 | has_permission 최소 구현 |
| GREEN | RB-BND-01~03 | 메뉴 가시성 딕셔너리 구현 |
| REFACTOR | RB-ENT | 역할 계층 구조 |
| REFACTOR | RB-BND-02 | JWT claim Role 읽기 |

## 테스트 결과

- pytest tests/rbac: 전체 GREEN
- 커버리지: 80% 이상

## 체크리스트

- [ ] 모든 테스트 GREEN
- [ ] ECB 의존성 방향 준수
- [ ] C2C Traceability 표 완성
- [ ] 커버리지 80%+
EOF
)"
```
