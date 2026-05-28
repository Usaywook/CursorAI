---
name: rbac-refactor
description: >-
  RBAC TDD 4단계(REFACTOR): 딕셔너리 기반 메뉴 가시성을 DB 기반 동적 권한으로 개선,
  JWT claim에서 Role 읽기, 역할 계층 구조 도입. 전체 테스트 GREEN 유지 필수.
  "리팩토링", "DB 기반 권한", "JWT Role", "역할 계층", "REFACTOR" 작업 시 사용.
---

# RBAC 리팩터링 (REFACTOR Phase)

## 응답 선언

```
Phase: refactor | Layer: entity|control|boundary | TestID: RB-XXX-nn
```

## 전제 조건

- [ ] `pytest tests/rbac` → 전체 GREEN 상태에서만 시작
- [ ] 동작·공개 API 불변 (테스트 수정 없이 리팩터링)

## 리팩터링 목표

```
딕셔너리 하드코딩 → DB·설정 기반 동적 권한
단일 Role → 역할 계층 구조 (admin > editor > viewer)
하드코딩 User → JWT claim에서 Role 읽기
```

## Step 1 — 역할 계층 구조 도입

```
Task Progress:
- [ ] Role에 parent 관계 추가 (admin inherits editor, editor inherits viewer)
- [ ] User.all_permissions() → 상속 포함 권한 집합 반환
- [ ] pytest → GREEN 유지
```

```python
# entity/role.py — 계층 구조 추가
@dataclass(frozen=True)
class Role:
    name: str
    permissions: frozenset[Permission]
    parent: "Role | None" = None   # 상속 관계

    def effective_permissions(self) -> frozenset[Permission]:
        """자신 + 부모 Role의 Permission 합집합."""
        inherited = self.parent.effective_permissions() if self.parent else frozenset()
        return self.permissions | inherited
```

## Step 2 — JWT Claim 기반 Role 읽기

```
Task Progress:
- [ ] boundary/auth_middleware.py — JWT 디코드 → Role 조회 → User 생성
- [ ] RoleRepository 인터페이스 정의 (control 레이어)
- [ ] InMemoryRoleRepository 구현 (테스트용)
- [ ] pytest → GREEN 유지
```

```python
# control/role_repository.py — 인터페이스
from abc import ABC, abstractmethod
from rbac_tdd.entity.role import Role

class RoleRepository(ABC):
    @abstractmethod
    def find_by_name(self, name: str) -> Role | None: ...
```

```python
# boundary/auth_middleware.py — JWT → User 변환
import jwt
from rbac_tdd.control.role_repository import RoleRepository
from rbac_tdd.entity.user import User

class JWTAuthMiddleware:
    def __init__(self, repo: RoleRepository, secret: str) -> None:
        self._repo = repo
        self._secret = secret

    def decode_user(self, token: str) -> User:
        payload = jwt.decode(token, self._secret, algorithms=["HS256"])
        role_names: list[str] = payload.get("roles", [])
        roles = tuple(
            r for name in role_names
            if (r := self._repo.find_by_name(name)) is not None
        )
        return User(user_id=payload["sub"], roles=roles)
```

## Step 3 — 메뉴 가시성 DB 기반으로 전환

```
Task Progress:
- [ ] MenuPermissionRepository 인터페이스 정의
- [ ] MenuVisibility가 하드코딩 dict 대신 Repository 사용
- [ ] pytest → GREEN 유지
- [ ] 커버리지 80% 이상 확인
```

## 완료 조건

- [ ] `pytest tests/rbac` → 전체 GREEN
- [ ] `pytest --cov=src/rbac_tdd --cov-report=term-missing` → 80% 이상
- [ ] `refactor(entity): RB-ENT 역할 계층 구조 도입` 커밋
- [ ] `refactor(boundary): RB-BND-02 JWT claim Role 읽기 구조` 커밋
- [ ] `refactor(control): RB-CTL DB 기반 동적 권한 Repository` 커밋
