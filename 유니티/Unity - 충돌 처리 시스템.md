
#Unity #Collision #Physics #Rigidbody #Collider

## 📌 간단 요약
Unity에서의 충돌 처리 메커니즘: AABB 충돌 방식, Collider와 Rigidbody 조합, 그리고 물리 엔진 설정 방법

---

## 1. AABB 충돌 (Axis-Aligned Bounding Box)

### 정의
- **AABB**: 축 방향과 평행한 경계 상자
- **용도**: 피격 범위에 대한 충돌 감지
- **특징**: 각도(orientation)가 없어 충돌 감지가 빠르고 간단함

---

## 2. Gizmos

### 기능
- Scene 영역에서만 보이는 시각적 도구
- 표시 항목: 격자, 빛, 카메라 등

### 특징
> ⚠️ **중요**: 오브젝트를 회전해도 Gizmos로 생성한 것은 회전하지 않음
```csharp
void OnDrawGizmos()
{
    // Scene 뷰에서만 보이는 충돌 범위 시각화
    Gizmos.color = Color.red;
    Gizmos.DrawWireCube(transform.position, new Vector3(1, 1, 1));
}
```

---

## 3. Collider (충돌체)

### 두 가지 충돌 형태

#### Trigger 충돌
- **특징**: 물리력을 무시하는 충돌
- **용도**: 아이템 획득, 구역 감지

#### Collision 충돌
- **특징**: 물리력이 적용되는 충돌
- **용도**: 벽, 바닥, 장애물

---

## 4. Collider 유형별 분류

### Collider + Rigidbody 조합

| Collider | Rigidbody | is Trigger | is Kinematic | 유형 |
|----------|-----------|------------|--------------|------|
| ✅ | ❌ | ❌ | - | **Static Collider** (정적 충돌체) |
| ✅ | ✅ | ❌ | ❌ | **Rigidbody Collider** (동적 충돌체) |
| ✅ | ✅ | ✅ | ❌ | **Rigidbody Trigger Collider** |
| ✅ | ❌ | ✅ | - | **Static Trigger Collider** |
| ✅ | ✅ | ❌ | ✅ | **Kinematic Rigidbody Collider** |
| ✅ | ✅ | ✅ | ✅ | **Kinematic Rigidbody Trigger Collider** |

### ⚠️ 중요 원칙
**Rigidbody를 하나의 오브젝트만 가지고 있어도 충돌 반응은 일어납니다.**
단, 두 오브젝트 모두 Collider는 반드시 가지고 있어야 합니다.

---

## 5. Rigidbody 컴포넌트

### 물리 속성

#### Mass (질량)
```csharp
rigidbody.mass = 1.0f; // 오브젝트의 무게
```

#### Drag (공기 저항)
```csharp
rigidbody.drag = 0.5f; // 이동 시 마찰력
```

#### Angular Drag (회전 저항)
```csharp
rigidbody.angularDrag = 0.05f; // 회전에 대한 저항
```

---

### 중심 설정

#### Automatic Center of Mass
- 무게 중심을 자동으로 계산

#### Automatic Tensor
- 회전 중심 축을 자동으로 설정

---

### 물리 옵션

#### Use Gravity (중력 사용)
```csharp
rigidbody.useGravity = true; // 중력 적용
```

#### is Kinematic (키네마틱)
- **특징**: 해당 오브젝트는 물리적 영향을 받지 않음
- **중요**: 상대방은 여전히 물리적 영향을 받음
- **용도**: 플레이어 캐릭터, 움직이는 플랫폼
```csharp
rigidbody.isKinematic = true; // 물리 엔진의 영향을 받지 않음
```

---

### 렌더링 보정

#### Interpolate (보간)
- **기능**: 프레임이 끊길 때 끊긴 구간을 부드럽게 채워줌
- **옵션**:
  - None: 보간 없음
  - Interpolate: 이전 프레임 기준 보간
  - Extrapolate: 다음 프레임 예측 보간
```csharp
rigidbody.interpolation = RigidbodyInterpolation.Interpolate;
```

---

### 충돌 감지 정밀도

#### Collision Detection
- **기능**: 충돌 검사 범위를 결정
- **용도**: 빠르게 움직이는 오브젝트의 충돌이 무시될 때 사용
- **⚠️ 주의**: 정밀도를 높이면 연산량이 증가함

**옵션:**
- **Discrete**: 기본값, 빠르지만 덜 정확
- **Continuous**: 연속적 충돌 감지
- **Continuous Dynamic**: 가장 정밀, 가장 무거움
```csharp
rigidbody.collisionDetectionMode = CollisionDetectionMode.Continuous;
```

---

### Constraints (제약 조건)

#### Freeze Position (위치 고정)
```csharp
// Z축 위치 고정 (2D 게임)
rigidbody.constraints = RigidbodyConstraints.FreezePositionZ;
```

#### Freeze Rotation (회전 고정)
```csharp
// X, Z축 회전 고정 (캐릭터가 넘어지지 않도록)
rigidbody.constraints = RigidbodyConstraints.FreezeRotationX | 
                        RigidbodyConstraints.FreezeRotationZ;
```

**캐릭터 설정 예시:**
```csharp
// 2D 플랫포머 캐릭터 기본 설정
void Start()
{
    Rigidbody rb = GetComponent<Rigidbody>();
    
    // Z축 위치 고정 + X, Z축 회전 고정
    rb.constraints = RigidbodyConstraints.FreezePositionZ |
                     RigidbodyConstraints.FreezeRotationX |
                     RigidbodyConstraints.FreezeRotationZ;
}
```

> 💡 **이유**: 물리력 때문에 캐릭터가 넘어지는 것을 방지

---

## 6. 충돌 감지 함수

### Trigger 충돌
```csharp
void OnTriggerEnter(Collider other)
{
    Debug.Log("트리거 진입: " + other.name);
}

void OnTriggerStay(Collider other)
{
    Debug.Log("트리거 체류 중");
}

void OnTriggerExit(Collider other)
{
    Debug.Log("트리거 이탈");
}
```

### Collision 충돌
```csharp
void OnCollisionEnter(Collision collision)
{
    Debug.Log("충돌 시작: " + collision.gameObject.name);
}

void OnCollisionStay(Collision collision)
{
    Debug.Log("충돌 지속 중");
}

void OnCollisionExit(Collision collision)
{
    Debug.Log("충돌 종료");
}
```

---

## 💡 핵심 포인트

1. **AABB 충돌**: 빠르고 효율적인 사각형 충돌 감지 방식
2. **충돌 조합**: Collider 하나 + Rigidbody 하나만 있어도 충돌 발생
3. **is Kinematic**: 물리 영향을 받지 않지만 다른 오브젝트에는 영향을 줌
4. **Constraints**: 2D 게임에서는 Z축 위치와 X/Z축 회전을 고정
5. **Collision Detection**: 빠른 오브젝트는 Continuous 사용
6. **Interpolate**: 부드러운 움직임을 위해 활성화 권장