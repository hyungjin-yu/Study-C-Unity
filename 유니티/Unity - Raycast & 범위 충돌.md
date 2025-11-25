#Unity #Raycast #Physics #BitOperation #OverlapSphere

## 📌 간단 요약
Raycast를 활용한 선형 충돌 감지, 비트 연산을 통한 레이어 필터링, OverlapSphere를 이용한 범위 충돌 처리

---

## 1. Raycast 기본 개념

### RaycastHit 구조체
- **용도**: Ray가 충돌한 정보를 담는 구조체
- **사용 위치**: Update() 함수에서 사용

### Physics.Raycast()
```csharp
// 기본 형태
bool isHit = Physics.Raycast(발사위치, 방향, out hit정보, Ray길이);
```

**파라미터 설명:**
1. **발사 위치**: Ray를 발사할 위치 (Vector3)
2. **방향**: Ray가 나갈 방향 (Vector3)
3. **out hit정보**: 충돌 정보를 담을 RaycastHit (out 키워드)
4. **Ray 길이**: Ray가 도달할 수 있는 최대 거리 (float)

> 💡 **특징**: Collider를 감지하는 물리 기반 충돌

---

## 2. Raycast 실전 예시

### 기본 Raycast
```csharp
public class RaycastExample : MonoBehaviour
{
    void Update()
    {
        RaycastHit hit;
        
        // 앞쪽으로 10 거리만큼 Ray 발사
        if (Physics.Raycast(transform.position, transform.forward, out hit, 10f))
        {
            Debug.Log("충돌한 오브젝트: " + hit.collider.name);
            Debug.Log("충돌 지점: " + hit.point);
            Debug.Log("충돌 거리: " + hit.distance);
        }
    }
}
```

### 레이어 마스크 사용
```csharp
void Update()
{
    RaycastHit hit;
    int layerMask = 1 << LayerMask.NameToLayer("Enemy");
    
    // Enemy 레이어만 감지
    if (Physics.Raycast(transform.position, transform.forward, out hit, 10f, layerMask))
    {
        Debug.Log("적 발견: " + hit.collider.name);
    }
}
```

---

## 3. Debug.DrawRay()

### 기본 사용법
```csharp
// Ray 시각화 (Scene 뷰에서만 보임)
Debug.DrawRay(발사위치, 방향, 색상);
```

### 실전 예시
```csharp
void Update()
{
    RaycastHit hit;
    
    // Ray 발사
    if (Physics.Raycast(transform.position, transform.forward, out hit, 10f))
    {
        // 충돌 지점까지 빨간색 Ray
        Debug.DrawRay(transform.position, transform.forward * hit.distance, Color.red);
        
        // 충돌 지점에서 법선 방향으로 녹색 Ray
        Debug.DrawRay(hit.point, hit.normal, Color.green);
    }
    else
    {
        // 충돌 없으면 전체 거리 노란색 Ray
        Debug.DrawRay(transform.position, transform.forward * 10f, Color.yellow);
    }
}
```

---

## 4. out 키워드

### 개념
```csharp
// out: 함수에서 값을 가져오는 키워드
Physics.Raycast(transform.position, transform.forward, out hit, 10f);
```

**특징:**
- 함수 내부에서 값을 할당하여 외부로 가져옴
- 여러 개의 반환값이 필요할 때 유용

### 예시
```csharp
void GetHitInfo(out string name, out float distance)
{
    RaycastHit hit;
    if (Physics.Raycast(transform.position, transform.forward, out hit, 10f))
    {
        name = hit.collider.name;
        distance = hit.distance;
    }
    else
    {
        name = "None";
        distance = 0f;
    }
}

// 사용
void Update()
{
    string hitName;
    float hitDistance;
    GetHitInfo(out hitName, out hitDistance);
    Debug.Log($"{hitName}, {hitDistance}m");
}
```

---

## 5. transform.forward vs Vector3.forward

### transform.forward
```csharp
// 오브젝트 기준 앞쪽 (로컬 좌표)
Debug.DrawRay(transform.position, transform.forward * 5f, Color.red);
```

### Vector3.forward
```csharp
// 월드 기준 앞쪽 (항상 Z축 양의 방향)
Debug.DrawRay(transform.position, Vector3.forward * 5f, Color.blue);
```

| 방향 | 기준 | 회전 영향 |
|------|------|-----------|
| transform.forward | 로컬 (오브젝트) | ✅ 받음 |
| Vector3.forward | 월드 (고정) | ❌ 안받음 |

---

## 6. 비트 연산 (Bit Operation)

### 시프트 연산자 (<<, >>)
```csharp
// << : 왼쪽으로 비트 이동
int a = 1;      // 0000 0001
int b = a << 1; // 0000 0010 (2)
int c = a << 3; // 0000 1000 (8)

// >> : 오른쪽으로 비트 이동
int d = 8;      // 0000 1000
int e = d >> 1; // 0000 0100 (4)
int f = d >> 3; // 0000 0001 (1)
```

**레이어 마스크 예시:**
```csharp
// Layer 8번을 선택
int layerMask = 1 << 8; // 0000 0001을 왼쪽으로 8칸 이동
```

---

### OR 연산자 (|)
```csharp
// 비트 연산: 하나라도 참이면 참
int a = 5;  // 0000 0101
int b = 3;  // 0000 0011
int c = a | b; // 0000 0111 (7)
```

**레이어 마스크 예시:**
```csharp
// Enemy와 Boss 레이어 모두 선택
int enemyLayer = 1 << LayerMask.NameToLayer("Enemy");
int bossLayer = 1 << LayerMask.NameToLayer("Boss");
int layerMask = enemyLayer | bossLayer;

Physics.Raycast(transform.position, transform.forward, out hit, 10f, layerMask);
```

---

### AND 연산자 (&)
```csharp
// 비트 연산: 둘 다 참일 때만 참
int a = 5;  // 0000 0101
int b = 3;  // 0000 0011
int c = a & b; // 0000 0001 (1)
```

**레이어 체크 예시:**
```csharp
// 특정 레이어에 속하는지 확인
int targetLayer = 1 << gameObject.layer;
int layerMask = 1 << LayerMask.NameToLayer("Enemy");

if ((targetLayer & layerMask) != 0)
{
    Debug.Log("Enemy 레이어입니다");
}
```

---

### NOT 연산자 (~)
```csharp
// 비트 보수 연산자 (비트 뒤집기)
int a = 5;   // 0000 0101
int b = ~a;  // 1111 1010 (-6)
```

**레이어 마스크 예시:**
```csharp
// Player 레이어를 제외한 모든 레이어
int playerLayer = 1 << LayerMask.NameToLayer("Player");
int layerMask = ~playerLayer;

Physics.Raycast(transform.position, transform.forward, out hit, 10f, layerMask);
```

---

### 비트 연산 vs 논리 연산

| 연산자 | 비트 연산     | 논리 연산      |
| --- | --------- | ---------- |
| OR  | \| (한 개)  | \|\| (두 개) |
| AND | `&` (한 개) | `&&` (두 개) |
```csharp
// 논리 연산 (bool)
if (a > 5 && b < 10) { }

// 비트 연산 (int)
int mask = layer1 | layer2;
```

---

## 7. Physics.OverlapSphere() - 범위 충돌

### 개념
특정 위치를 중심으로 구 형태의 범위 내 모든 Collider를 감지

### 기본 사용법
```csharp
Collider[] colliders = Physics.OverlapSphere(중심위치, 반경, 레이어마스크);
```

### 실전 예시
```csharp
public class ExplosionDamage : MonoBehaviour
{
    public float explosionRadius = 10f;
    public int damage = 50;
    
    void Explode()
    {
        // Enemy 레이어만 감지
        int layerMask = 1 << LayerMask.NameToLayer("Enemy");
        
        // 범위 내 모든 Collider 찾기
        Collider[] colliders = Physics.OverlapSphere(transform.position, explosionRadius, layerMask);
        
        // 각 Collider에 대해 처리
        foreach (Collider col in colliders)
        {
            Enemy enemy = col.GetComponent<Enemy>();
            if (enemy != null)
            {
                enemy.TakeDamage(damage);
            }
        }
    }
}
```

---

### 여러 레이어 감지
```csharp
void DetectMultipleLayers()
{
    // Enemy와 Destructible 레이어 모두 감지
    int enemyLayer = 1 << LayerMask.NameToLayer("Enemy");
    int destructibleLayer = 1 << LayerMask.NameToLayer("Destructible");
    int layerMask = enemyLayer | destructibleLayer;
    
    Collider[] colliders = Physics.OverlapSphere(transform.position, 10f, layerMask);
    
    Debug.Log($"범위 내 오브젝트: {colliders.Length}개");
}
```

---

## 8. OnDrawGizmos() - 범위 시각화

### 기본 사용법
```csharp
private void OnDrawGizmos()
{
    // Scene 뷰에서만 보이는 시각화
    Gizmos.color = Color.blue;
    Gizmos.DrawWireSphere(transform.position, 10f);
}
```

### 충돌 범위 시각화 예시
```csharp
public class AreaDetector : MonoBehaviour
{
    public float detectionRadius = 10f;
    
    void Update()
    {
        Collider[] colliders = Physics.OverlapSphere(transform.position, detectionRadius);
        
        if (colliders.Length > 0)
        {
            Debug.Log($"감지된 오브젝트: {colliders.Length}개");
        }
    }
    
    // 범위 시각화
    private void OnDrawGizmos()
    {
        Gizmos.color = Color.yellow;
        Gizmos.DrawWireSphere(transform.position, detectionRadius);
    }
}
```

### 선택 시에만 표시
```csharp
// Inspector에서 선택했을 때만 표시
private void OnDrawGizmosSelected()
{
    Gizmos.color = Color.red;
    Gizmos.DrawWireSphere(transform.position, detectionRadius);
}
```

---

## 9. 종합 예시 - 적 감지 시스템
```csharp
public class PlayerDetector : MonoBehaviour
{
    public float rayDistance = 15f;
    public float detectionRadius = 5f;
    
    void Update()
    {
        // 1. Raycast로 전방 적 탐지
        DetectWithRaycast();
        
        // 2. OverlapSphere로 주변 적 탐지
        DetectWithSphere();
    }
    
    void DetectWithRaycast()
    {
        RaycastHit hit;
        int enemyLayer = 1 << LayerMask.NameToLayer("Enemy");
        
        if (Physics.Raycast(transform.position, transform.forward, out hit, rayDistance, enemyLayer))
        {
            Debug.Log($"전방 적 발견: {hit.collider.name}, 거리: {hit.distance}m");
            
            // 충돌 지점까지 빨간색 Ray
            Debug.DrawRay(transform.position, transform.forward * hit.distance, Color.red);
        }
        else
        {
            // 충돌 없으면 노란색 Ray
            Debug.DrawRay(transform.position, transform.forward * rayDistance, Color.yellow);
        }
    }
    
    void DetectWithSphere()
    {
        int enemyLayer = 1 << LayerMask.NameToLayer("Enemy");
        Collider[] enemies = Physics.OverlapSphere(transform.position, detectionRadius, enemyLayer);
        
        if (enemies.Length > 0)
        {
            Debug.Log($"주변 적 {enemies.Length}명 감지");
        }
    }
    
    // 범위 시각화
    private void OnDrawGizmos()
    {
        // Ray 범위
        Gizmos.color = Color.yellow;
        Gizmos.DrawLine(transform.position, transform.position + transform.forward * rayDistance);
        
        // Sphere 범위
        Gizmos.color = Color.blue;
        Gizmos.DrawWireSphere(transform.position, detectionRadius);
    }
}
```

---

## 💡 핵심 포인트

1. **Raycast**: 선형 충돌 감지, 정확한 타겟팅에 유용
2. **RaycastHit**: out 키워드로 충돌 정보 받아옴
3. **Debug.DrawRay**: Scene 뷰에서 Ray 시각화
4. **비트 연산**: 레이어 마스크 생성 및 필터링에 필수
5. **시프트 연산 (<<)**: `1 << layer` 형태로 레이어 선택
6. **OR 연산 (|)**: 여러 레이어 조합
7. **AND 연산 (&)**: 레이어 포함 여부 확인
8. **NOT 연산 (~)**: 특정 레이어 제외
9. **OverlapSphere**: 범위 내 모든 충돌체 감지
10. **OnDrawGizmos**: 범위 및 디버그 정보 시각화