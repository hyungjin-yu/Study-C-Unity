#Unity #Collider #Rigidbody #Physics #ForceMode

## 📌 간단 요약
복합 콜라이더를 통한 효율적인 충돌 관리와 다양한 ForceMode를 활용한 물리 힘 적용 방법

---

## 1. 복합 콜라이더 (Compound Collider)

### 개념
**최상위 부모에 Rigidbody가 있다면, 자식들은 모두 Rigidbody가 없어도 Rigidbody가 있는 것으로 판정됩니다.**

### 구조 예시
```
부모 오브젝트 (Rigidbody ✅)
├── 자식 1 (Collider만 있음)
├── 자식 2 (Collider만 있음)
└── 자식 3 (Collider만 있음)
```

### 장점
- 충돌의 주체를 부모 하나로 관리하면 편함
- 성능 최적화 (Rigidbody 하나만 계산)
- 복잡한 모양의 충돌체를 여러 개의 간단한 Collider로 구성 가능

---

## 2. Rigidbody 우선 순위

### 우선순위 규칙
```
1순위: 자기 자신의 Rigidbody
2순위: 부모 오브젝트의 Rigidbody
3순위: Rigidbody가 없는 자식 (Static Collider로 판정)
```

### 예시
```csharp
// 자식 오브젝트에서 Rigidbody 찾기
Rigidbody rb = GetComponent<Rigidbody>(); // 1순위: 자신
if (rb == null)
{
    rb = GetComponentInParent<Rigidbody>(); // 2순위: 부모
}
```

---

## 3. 좌표계 기준 함수

### AddForce (월드 좌표 기준)
```csharp
// 월드 좌표 기준으로 힘을 가함
rigidbody.AddForce(Vector3.forward * 10f);
// 오브젝트가 회전해도 항상 월드의 Z축 방향으로 힘이 가해짐
```

### AddRelativeForce (로컬 좌표 기준)
```csharp
// 로컬 좌표 기준으로 힘을 가함
rigidbody.AddRelativeForce(Vector3.forward * 10f);
// 오브젝트가 바라보는 방향(로컬 Z축)으로 힘이 가해짐
```

### 차이점 비교
| 함수 | 기준 | 사용 예시 |
|------|------|-----------|
| AddForce | 월드 좌표 | 중력, 바람, 폭발 등 |
| AddRelativeForce | 로컬 좌표 | 차량 전진, 캐릭터 이동 등 |

---

## 4. ForceMode (힘의 적용 방식)

### ForceMode.Force
```csharp
rigidbody.AddForce(Vector3.forward * 10f, ForceMode.Force);
```
- **특징**: 질량 기반의 **연속적인** 가속
- **공식**: `F = ma` (힘 = 질량 × 가속도)
- **용도**: 지속적인 추진력 (로켓 엔진, 지속적인 밀기)

---

### ForceMode.Impulse
```csharp
rigidbody.AddForce(Vector3.forward * 10f, ForceMode.Impulse);
```
- **특징**: 질량 기반의 **순간적인** 가속
- **공식**: `Impulse = m × Δv` (충격량 = 질량 × 속도 변화)
- **용도**: 점프, 충격, 폭발

---

### ForceMode.Acceleration
```csharp
rigidbody.AddForce(Vector3.forward * 10f, ForceMode.Acceleration);
```
- **특징**: 질량과 **상관없이** 연속적인 가속
- **공식**: `a = F/m` (가속도는 질량 무시)
- **용도**: 중력, 자석 끌림

---

### ForceMode.VelocityChange
```csharp
rigidbody.AddForce(Vector3.forward * 10f, ForceMode.VelocityChange);
```
- **특징**: 질량과 **상관없이** 순간적인 가속
- **공식**: 직접적인 속도 변화
- **용도**: 순간 이동, 텔레포트 효과

---

## 5. ForceMode 비교표

| ForceMode | 질량 영향 | 적용 방식 | 사용 시점 |
|-----------|----------|-----------|-----------|
| Force | ✅ 있음 | 연속적 | FixedUpdate |
| Impulse | ✅ 있음 | 순간적 | 한 번 |
| Acceleration | ❌ 없음 | 연속적 | FixedUpdate |
| VelocityChange | ❌ 없음 | 순간적 | 한 번 |

---

## 6. 실전 예시 코드

### 점프 구현 (Impulse)
```csharp
void Jump()
{
    // 질량 기반 순간 가속 - 무거운 캐릭터는 낮게, 가벼운 캐릭터는 높게 점프
    rigidbody.AddForce(Vector3.up * jumpPower, ForceMode.Impulse);
}
```

### 로켓 추진 (Force)
```csharp
void FixedUpdate()
{
    // 질량 기반 연속 가속 - 무거우면 천천히 가속
    rigidbody.AddRelativeForce(Vector3.forward * thrustPower, ForceMode.Force);
}
```

### 중력 구현 (Acceleration)
```csharp
void FixedUpdate()
{
    // 질량 무시 연속 가속 - 모든 물체가 같은 속도로 낙하
    rigidbody.AddForce(Vector3.down * 9.8f, ForceMode.Acceleration);
}
```

### 대시 구현 (VelocityChange)
```csharp
void Dash()
{
    // 질량 무시 순간 가속 - 모든 캐릭터가 같은 속도로 대시
    rigidbody.AddForce(transform.forward * dashSpeed, ForceMode.VelocityChange);
}
```

---

## 7. 복합 콜라이더 설정 예시

### 플레이어 캐릭터 구조
```
Player (Rigidbody + Capsule Collider)
├── Head (Sphere Collider)
├── Body (Box Collider)
└── Weapon (Box Collider, is Trigger)
```
```csharp
// 부모의 Rigidbody 하나로 모든 자식의 충돌 관리
public class Player : MonoBehaviour
{
    Rigidbody rb;
    
    void Start()
    {
        rb = GetComponent<Rigidbody>();
        
        // 모든 자식 Collider가 이 Rigidbody를 공유함
        Debug.Log("자식 Collider 수: " + GetComponentsInChildren<Collider>().Length);
    }
}
```

---

## 💡 핵심 포인트

1. **복합 콜라이더**: 부모에 Rigidbody 하나만 있으면 자식들도 동일한 물리 계산 공유
2. **우선 순위**: 자신 → 부모 → 없음 순서로 Rigidbody 탐색
3. **좌표계**: AddForce(월드), AddRelativeForce(로컬) 구분하여 사용
4. **Force vs Impulse**: 연속적 vs 순간적 차이
5. **질량 영향**: Force/Impulse는 질량 영향, Acceleration/VelocityChange는 질량 무시
6. **사용 시점**: Force/Acceleration은 FixedUpdate, Impulse/VelocityChange는 이벤트 시점