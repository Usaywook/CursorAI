---
name: rbac-minimal-impl
description: >-
  RBAC TDD 3단계(GREEN): 실패 테스트를 통과하는 최소 구현.
  Permission Enum, Role/User 데이터클래스, has_permission(), get_visible_menus() 딕셔너리 구현.
  "테스트 통과", "최소 구현", "GREEN", "is_visible", "has_permission" 작업 시 사용.
---

# RBAC 최소 구현 (GREEN Phase)

## 응답 선언

```
Phase: green | Layer: entity|control|boundary | TestID: RB-XXX-nn
```

## 구현 순서 (의존성 방향)

```
entity 먼저 → control → boundary
```

## Step 1 — Entity GREEN

```
Task Progress:
- [ ] src/rbac_tdd/exceptions/auth_errors.py — PermissionDeniedError
- [ ] src/rbac_tdd/entity/permission.py — Permission Enum
- [ ] src/rbac_tdd/entity/role.py — Role 데이터클래스
- [ ] src/rbac_tdd/entity/user.py — User 데이터클래스
- [ ] pytest tests/rbac/entity → 전부 GREEN
```

### 최소 구현 스켈레톤

```python
# entity/permission.py
from enum import Enum, auto

class Permission(Enum):
    VIEW   = auto()
    CREATE = auto()
    UPDATE = auto()
    DELETE = auto()
```

```python
# entity/role.py
from dataclasses import dataclass
from rbac_tdd.entity.permission import Permission

@dataclass(frozen=True)
class Role:
    name: str
    permissions: frozenset[Permission]

    def __post_init__(self) -> None:
        if not self.permissions:
            raise ValueError("Role must have at least one Permission")
```

```python
# entity/user.py
from dataclasses import dataclass, field
from rbac_tdd.entity.role import Role
from rbac_tdd.entity.permission import Permission

@dataclass(frozen=True)
class User:
    user_id: str
    roles: tuple[Role, ...] = field(default_factory=tuple)

    def has_role(self, role_name: str) -> bool:
        return any(r.name == role_name for r in self.roles)

    def all_permissions(self) -> frozenset[Permission]:
        result: set[Permission] = set()
        for role in self.roles:
            result |= role.permissions
        return frozenset(result)
```

## Step 2 — Control GREEN

```
Task Progress:
- [ ] src/rbac_tdd/control/permission_checker.py
- [ ] pytest tests/rbac/control → 전부 GREEN
```

### 최소 구현 스켈레톤

```python
# control/permission_checker.py
from rbac_tdd.entity.user import User
from rbac_tdd.entity.permission import Permission
from rbac_tdd.exceptions.auth_errors import PermissionDeniedError

class PermissionChecker:
    def has_permission(self, user: User, permission: Permission) -> bool:
        return permission in user.all_permissions()

    def require_permission(self, user: User, permission: Permission) -> None:
        if not self.has_permission(user, permission):
            raise PermissionDeniedError(user.user_id, permission)
```

## Step 3 — Boundary GREEN (메뉴 가시성 딕셔너리)

```
Task Progress:
- [ ] src/rbac_tdd/boundary/menu_visibility.py
- [ ] pytest tests/rbac/boundary → 전부 GREEN
```

### 최소 구현 스켈레톤

```python
# boundary/menu_visibility.py — 딕셔너리 기반 (REFACTOR 전 단계)
from rbac_tdd.entity.user import User
from rbac_tdd.entity.permission import Permission
from rbac_tdd.control.permission_checker import PermissionChecker

MENU_PERMISSION_MAP: dict[str, Permission] = {
    "home":        Permission.VIEW,
    "profile":     Permission.VIEW,
    "create-post": Permission.CREATE,
    "edit-post":   Permission.UPDATE,
    "delete-user": Permission.DELETE,
}

class MenuVisibility:
    def __init__(self) -> None:
        self._checker = PermissionChecker()

    def get_visible_menus(self, user: User) -> list[str]:
        return [
            menu for menu, perm in MENU_PERMISSION_MAP.items()
            if self._checker.has_permission(user, perm)
        ]
```

## 완료 조건

- [ ] `pytest tests/rbac` → 전체 GREEN
- [ ] 추가 기능·추상화 없음 (필요 최소만)
- [ ] `feat(entity): RB-ENT-01~05 Role·Permission·User 최소 구현` 커밋
- [ ] `feat(control): RB-CTL-01~05 has_permission 최소 구현` 커밋
- [ ] `feat(boundary): RB-BND-01~03 메뉴 가시성 딕셔너리 최소 구현` 커밋
