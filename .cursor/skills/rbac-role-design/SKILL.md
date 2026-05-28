---
name: rbac-role-design
description: >-
  RBAC TDD 1단계(RED): Role·Permission·User ECB Entity 설계.
  역할 개념을 도메인 데이터 구조로 번역하고 실패 테스트(RB-ENT-*)를 작성한다.
  "역할 설계", "Permission 구조", "RBAC Entity", "RB-ENT" 작업 시 사용.
---

# RBAC Role 설계 (RED Phase)

## 응답 선언

```
Phase: red | Layer: entity | TestID: RB-ENT-01~0n
```

## 1. 디렉터리 생성

```bash
mkdir -p src/rbac_tdd/entity src/rbac_tdd/exceptions
mkdir -p tests/rbac/entity
touch src/rbac_tdd/__init__.py src/rbac_tdd/entity/__init__.py
touch src/rbac_tdd/exceptions/__init__.py src/rbac_tdd/exceptions/auth_errors.py
touch tests/__init__.py tests/rbac/__init__.py tests/rbac/entity/__init__.py
```

## 2. Entity 설계 원칙

```
Permission (Enum)  →  Role (dataclass)  →  User (dataclass)
    값 객체              집합 관계              역할 보유자
```

- `Permission`: VIEW · CREATE · UPDATE · DELETE (Enum, 확장 가능)
- `Role`: name + `permissions: frozenset[Permission]` (불변)
- `User`: user_id + `roles: tuple[Role, ...]` (불변)

## 3. 실패 테스트 작성 체크리스트

```
Task Progress:
- [ ] RB-ENT-01: Role 생성 (name, permissions 할당)
- [ ] RB-ENT-02: Permission.DELETE Role에 포함 여부 확인
- [ ] RB-ENT-03: 빈 permissions Role 거부 (최소 1개 필수)
- [ ] RB-ENT-04: User에 Role 할당
- [ ] RB-ENT-05: User.has_role("admin") 조회
- [ ] pytest tests/rbac/entity → 전부 FAIL 확인
```

## 4. 테스트 파일 위치

```
tests/rbac/entity/test_rb_ent_01_role_creation.py
tests/rbac/entity/test_rb_ent_02_permission_assignment.py
```

## 5. AAA 테스트 패턴

```python
# RB-ENT-01: Role 생성
def test_rb_ent_01_role_can_be_created_with_permissions():
    # Arrange
    permissions = frozenset({Permission.VIEW, Permission.CREATE})
    # Act
    role = Role(name="editor", permissions=permissions)
    # Assert
    assert role.name == "editor"
    assert Permission.VIEW in role.permissions
```

## 6. 완료 조건

- [ ] `pytest tests/rbac/entity` → RED (ImportError 또는 AssertionError)
- [ ] production 코드(`src/rbac_tdd/entity/`) 아직 없음
- [ ] TestID RB-ENT-01~05 커버리지 확인
- [ ] `test(entity): RB-ENT-01~05 역할·권한 설계 실패 테스트` 커밋
