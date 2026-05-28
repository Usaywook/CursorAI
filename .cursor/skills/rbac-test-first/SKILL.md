---
name: rbac-test-first
description: >-
  RBAC TDD 2단계(RED): Control·Boundary 레이어 실패 테스트 작성.
  권한 거부/허용 시나리오, 메뉴 가시성 테스트를 먼저 작성하고 pytest FAIL 확인.
  "실패 테스트", "권한 거부", "403 에러 테스트", "viewer delete", "RB-CTL", "RB-BND" 작업 시 사용.
---

# RBAC 실패 테스트 먼저 (RED Phase)

## 응답 선언

```
Phase: red | Layer: control|boundary | TestID: RB-CTL-01~0n / RB-BND-01~0n
```

## Control Track — 권한 검사 실패 테스트

### 테스트 체크리스트

```
Task Progress:
- [ ] RB-CTL-01: viewer가 DELETE → PermissionDeniedError 발생
- [ ] RB-CTL-02: admin이 DELETE → True 반환
- [ ] RB-CTL-03: editor가 CREATE → True 반환
- [ ] RB-CTL-04: viewer가 VIEW → True 반환
- [ ] RB-CTL-05: 역할 없는 User → 모든 권한 거부
- [ ] pytest tests/rbac/control → 전부 FAIL 확인
```

### AAA 패턴 예시

```python
# RB-CTL-01: viewer DELETE 거부
def test_rb_ctl_01_viewer_cannot_delete():
    # Arrange
    viewer_role = Role("viewer", frozenset({Permission.VIEW}))
    user = User(user_id="u1", roles=(viewer_role,))
    checker = PermissionChecker()
    # Act & Assert
    with pytest.raises(PermissionDeniedError) as exc_info:
        checker.require_permission(user, Permission.DELETE)
    assert exc_info.value.status_code == 403

# RB-CTL-02: admin DELETE 허용
def test_rb_ctl_02_admin_can_delete():
    # Arrange
    admin_role = Role("admin", frozenset({
        Permission.VIEW, Permission.CREATE,
        Permission.UPDATE, Permission.DELETE
    }))
    user = User(user_id="u2", roles=(admin_role,))
    checker = PermissionChecker()
    # Act
    result = checker.has_permission(user, Permission.DELETE)
    # Assert
    assert result is True
```

## Boundary Track — 메뉴 가시성 실패 테스트

### 테스트 체크리스트

```
Task Progress:
- [ ] RB-BND-01: viewer → ["home", "profile"] 메뉴만 보임
- [ ] RB-BND-02: admin → 전체 메뉴 보임
- [ ] RB-BND-03: 역할 없는 User → 빈 메뉴 또는 공개 메뉴만
- [ ] pytest tests/rbac/boundary → 전부 FAIL 확인
```

### AAA 패턴 예시

```python
# RB-BND-01: viewer 메뉴 가시성
def test_rb_bnd_01_viewer_sees_limited_menus():
    # Arrange
    viewer_role = Role("viewer", frozenset({Permission.VIEW}))
    user = User(user_id="u1", roles=(viewer_role,))
    visibility = MenuVisibility()
    # Act
    menus = visibility.get_visible_menus(user)
    # Assert
    assert "delete-user" not in menus
    assert "home" in menus
```

## 완료 조건

- [ ] `pytest tests/rbac/control` → RED (ImportError 또는 AssertionError)
- [ ] `pytest tests/rbac/boundary` → RED
- [ ] production 코드 아직 없음 (stub만 존재 가능)
- [ ] `test(control): RB-CTL-01~05 권한 거부/허용 실패 테스트` 커밋
- [ ] `test(boundary): RB-BND-01~03 메뉴 가시성 실패 테스트` 커밋
