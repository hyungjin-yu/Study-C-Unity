#Unity #DesignPattern #Composition #Coroutine #ComponentPattern

## 📌 간단 요약
is-a 관계(상속)의 한계점, has-a 관계(컴포지션)를 통한 컴포넌트화 설계, 그리고 코루틴을 활용한 비동기 처리 방법

---
## 📑 목차

1. is-a 관계 (상속)
2. is-a 관계의 한계
3. has-a 관계 (컴포지션)
4. 탐지 컴포넌트 예시
5. has-a의 장점
6. 코루틴 (Coroutine)
7. 컴포지션 + 코루틴 종합 예시
8. is-a vs has-a 비교
9. 핵심 포인트
10. 클래스 다이어그램
---

## 1. is-a 관계 (상속)

### 개념

**is-a 관계**: "A는 B이다" (A is a B)
- 클래스 상속을 통한 관계
- 자식 클래스가 부모 클래스의 특성을 물려받음

### 기본 예시
```csharp
// 부모 클래스
public class Character : MonoBehaviour
{
    public float hp = 100f;
    public float moveSpeed = 5f;
    
    public virtual void TakeDamage(float damage)
    {
        hp -= damage;
        if (hp <= 0)
        {
            Die();
        }
    }
    
    public virtual void Die()
    {
        Debug.Log("캐릭터 사망");
    }
}

// 자식 클래스 - Player는 Character이다
public class Player : Character
{
    public override void Die()
    {
        Debug.Log("플레이어 사망!");
        // 게임오버 처리
    }
}

// 자식 클래스 - Enemy는 Character이다
public class Enemy : Character
{
    public override void Die()
    {
        Debug.Log("적 사망!");
        // 점수 추가
    }
}
```

---

## 2. is-a 관계의 한계

### 문제점 1: 경직된 상속 구조
```csharp
public class Character : MonoBehaviour
{
    public float hp;
    public float moveSpeed;
}

public class FlyingEnemy : Character
{
    public float flyHeight;
    // 문제: 날아다니는 적인데 moveSpeed를 상속받음
    // 비행 속도와 이동 속도가 다를 수 있음
}

public class TurretEnemy : Character
{
    // 문제: 포탑은 움직이지 않는데 moveSpeed를 상속받음
    // 불필요한 변수 존재
}

public class BossEnemy : Character
{
    public float phase2Hp;
    public float specialAttackPower;
    // 문제: 보스만의 특수 기능이 많아짐
    // 상속 구조가 점점 복잡해짐
}
```

---

### 문제점 2: 다중 상속 불가
```csharp
// C#은 다중 상속을 지원하지 않음!

public class Flyable
{
    public void Fly() { }
}

public class Swimmable
{
    public void Swim() { }
}

// ❌ 불가능: 날면서 수영도 하는 캐릭터
public class FlyingFish : Flyable, Swimmable
{
    // 컴파일 에러!
}
```

---

### 문제점 3: 불필요한 기능 상속
```csharp
public class Enemy : Character
{
    public void Attack() { }
    public void Patrol() { }
    public void Chase() { }
}

public class PassiveEnemy : Enemy
{
    // 문제: 공격하지 않는 적인데 Attack()을 상속받음
    // Chase()도 필요 없음
    // 하지만 상속 구조상 어쩔 수 없이 가지고 있음
}
```

---

### 문제점 4: 변경의 어려움
```csharp
// 부모 클래스 변경 시 모든 자식 클래스 영향
public class Character : MonoBehaviour
{
    public float hp;
    
    // 나중에 방어력 추가하려면?
    public float defense; // 모든 자식 클래스에 영향!
}

// 100개의 자식 클래스가 있다면?
// 모두 테스트하고 수정해야 함
```

---

## 3. has-a 관계 (컴포지션)

### 개념

**has-a 관계**: "A는 B를 가지고 있다" (A has a B)
- 컴포넌트를 조합하여 기능 구현
- Unity의 핵심 설계 철학
- 유연하고 재사용 가능한 구조

---

### 컴포넌트화 기본 예시
```csharp
// HP 컴포넌트
public class HealthComponent : MonoBehaviour
{
    public float maxHp = 100f;
    private float currentHp;
    
    void Start()
    {
        currentHp = maxHp;
    }
    
    public void TakeDamage(float damage)
    {
        currentHp -= damage;
        if (currentHp <= 0)
        {
            Die();
        }
    }
    
    void Die()
    {
        Destroy(gameObject);
    }
}

// 이동 컴포넌트
public class MoveComponent : MonoBehaviour
{
    public float moveSpeed = 5f;
    
    public void Move(Vector3 direction)
    {
        transform.Translate(direction * moveSpeed * Time.deltaTime);
    }
}

// 플레이어는 HP와 이동 기능을 "가지고 있다"
// Player 오브젝트에 HealthComponent와 MoveComponent 추가
```

---

## 4. 탐지 컴포넌트 예시

### 기본 탐지 컴포넌트
```csharp
public class DetectionComponent : MonoBehaviour
{
    public float detectionRadius = 10f;
    public LayerMask targetLayer;
    public Transform target { get; private set; }
    
    void Update()
    {
        DetectTarget();
    }
    
    void DetectTarget()
    {
        Collider[] colliders = Physics.OverlapSphere(transform.position, detectionRadius, targetLayer);
        
        if (colliders.Length > 0)
        {
            // 가장 가까운 타겟 찾기
            target = GetClosestTarget(colliders);
        }
        else
        {
            target = null;
        }
    }
    
    Transform GetClosestTarget(Collider[] colliders)
    {
        Transform closest = null;
        float minDistance = Mathf.Infinity;
        
        foreach (Collider col in colliders)
        {
            float distance = Vector3.Distance(transform.position, col.transform.position);
            if (distance < minDistance)
            {
                minDistance = distance;
                closest = col.transform;
            }
        }
        
        return closest;
    }
    
    void OnDrawGizmos()
    {
        Gizmos.color = Color.yellow;
        Gizmos.DrawWireSphere(transform.position, detectionRadius);
    }
}
```

---

### 탐지 컴포넌트 활용
```csharp
public class Enemy : MonoBehaviour
{
    // has-a: 적은 탐지 기능을 "가지고 있다"
    private DetectionComponent detection;
    private MoveComponent movement;
    
    void Start()
    {
        detection = GetComponent<DetectionComponent>();
        movement = GetComponent<MoveComponent>();
    }
    
    void Update()
    {
        if (detection.target != null)
        {
            // 타겟 발견 시 추적
            Vector3 direction = (detection.target.position - transform.position).normalized;
            movement.Move(direction);
        }
    }
}
```

---

### 다양한 탐지 방식 컴포넌트
```csharp
// Raycast 탐지
public class RaycastDetectionComponent : MonoBehaviour
{
    public float rayDistance = 15f;
    public Transform target { get; private set; }
    
    void Update()
    {
        RaycastHit hit;
        if (Physics.Raycast(transform.position, transform.forward, out hit, rayDistance))
        {
            if (hit.collider.CompareTag("Player"))
            {
                target = hit.transform;
            }
        }
        else
        {
            target = null;
        }
        
        Debug.DrawRay(transform.position, transform.forward * rayDistance, Color.red);
    }
}

// 시야각 탐지
public class FOVDetectionComponent : MonoBehaviour
{
    public float detectionRadius = 10f;
    public float viewAngle = 90f;
    public LayerMask targetLayer;
    public Transform target { get; private set; }
    
    void Update()
    {
        DetectInFOV();
    }
    
    void DetectInFOV()
    {
        Collider[] colliders = Physics.OverlapSphere(transform.position, detectionRadius, targetLayer);
        
        foreach (Collider col in colliders)
        {
            Vector3 directionToTarget = (col.transform.position - transform.position).normalized;
            float angleToTarget = Vector3.Angle(transform.forward, directionToTarget);
            
            if (angleToTarget < viewAngle / 2)
            {
                // 시야각 내에 있음
                target = col.transform;
                return;
            }
        }
        
        target = null;
    }
}
```

---

## 5. has-a의 장점

### 1. 유연한 조합
```csharp
// 날아다니는 적
// FlyingEnemy 오브젝트에 추가:
// - HealthComponent
// - FlyMovementComponent
// - DetectionComponent
// - AttackComponent

// 포탑 적
// TurretEnemy 오브젝트에 추가:
// - HealthComponent
// - DetectionComponent
// - AttackComponent
// (이동 컴포넌트 없음)

// 보스 적
// BossEnemy 오브젝트에 추가:
// - HealthComponent (높은 HP)
// - MoveComponent
// - DetectionComponent
// - AttackComponent
// - SpecialAttackComponent (보스 전용)
```

---

### 2. 재사용성
```csharp
// 탐지 컴포넌트를 적, 아군, NPC 모두에게 사용 가능
public class Ally : MonoBehaviour
{
    private DetectionComponent detection; // 같은 컴포넌트 재사용!
    
    void Start()
    {
        detection = GetComponent<DetectionComponent>();
        detection.targetLayer = LayerMask.GetMask("Enemy"); // 레이어만 변경
    }
}

public class NPC : MonoBehaviour
{
    private DetectionComponent detection; // 같은 컴포넌트 재사용!
    
    void Start()
    {
        detection = GetComponent<DetectionComponent>();
        detection.targetLayer = LayerMask.GetMask("Player");
    }
}
```

---

### 3. 쉬운 수정과 확장
```csharp
// 탐지 컴포넌트만 수정하면 모든 오브젝트에 적용됨
public class DetectionComponent : MonoBehaviour
{
    public float detectionRadius = 10f;
    public LayerMask targetLayer;
    
    // 새로운 기능 추가
    public bool useLineOfSight = false; // 장애물 체크
    
    void DetectTarget()
    {
        Collider[] colliders = Physics.OverlapSphere(transform.position, detectionRadius, targetLayer);
        
        foreach (Collider col in colliders)
        {
            if (useLineOfSight)
            {
                // 시야 가림 체크
                if (!HasLineOfSight(col.transform))
                    continue;
            }
            
            target = col.transform;
            return;
        }
    }
    
    bool HasLineOfSight(Transform target)
    {
        Vector3 direction = target.position - transform.position;
        return !Physics.Raycast(transform.position, direction, direction.magnitude, LayerMask.GetMask("Obstacle"));
    }
}
```

---

## 6. 코루틴 (Coroutine)

### 개념

**코루틴**: 함수 실행을 일시 정지했다가 나중에 재개할 수 있는 기능
- `IEnumerator` 반환 타입 사용
- `yield return` 키워드로 대기
- 비동기적 작업 처리에 유용

---

### 기본 문법
```csharp
public class CoroutineExample : MonoBehaviour
{
    void Start()
    {
        // 코루틴 시작
        StartCoroutine(MyCoroutine());
    }
    
    IEnumerator MyCoroutine()
    {
        Debug.Log("코루틴 시작");
        
        // 1초 대기
        yield return new WaitForSeconds(1f);
        
        Debug.Log("1초 후 실행");
        
        // 다음 프레임까지 대기
        yield return null;
        
        Debug.Log("다음 프레임 실행");
    }
}
```

---

### yield return 종류
```csharp
IEnumerator YieldExamples()
{
    // 1. 다음 프레임까지 대기
    yield return null;
    
    // 2. 지정된 시간만큼 대기
    yield return new WaitForSeconds(2f);
    
    // 3. 실제 시간 기준 대기 (Time.timeScale 무시)
    yield return new WaitForSecondsRealtime(2f);
    
    // 4. 다음 FixedUpdate까지 대기
    yield return new WaitForFixedUpdate();
    
    // 5. 프레임 끝까지 대기
    yield return new WaitForEndOfFrame();
    
    // 6. 조건이 참이 될 때까지 대기
    yield return new WaitUntil(() => Input.GetKeyDown(KeyCode.Space));
    
    // 7. 조건이 거짓이 될 때까지 대기
    yield return new WaitWhile(() => isMoving);
    
    // 8. 다른 코루틴이 끝날 때까지 대기
    yield return StartCoroutine(OtherCoroutine());
}
```

---

### 실전 예시 1: 페이드 효과
```csharp
public class FadeEffect : MonoBehaviour
{
    public Material material;
    
    public void FadeOut()
    {
        StartCoroutine(FadeOutCoroutine());
    }
    
    IEnumerator FadeOutCoroutine()
    {
        Color color = material.color;
        
        // 2초 동안 서서히 투명해짐
        for (float t = 0; t < 2f; t += Time.deltaTime)
        {
            float alpha = Mathf.Lerp(1f, 0f, t / 2f);
            material.color = new Color(color.r, color.g, color.b, alpha);
            yield return null; // 매 프레임마다 실행
        }
        
        // 완전히 투명하게
        material.color = new Color(color.r, color.g, color.b, 0f);
    }
}
```

---

### 실전 예시 2: 연속 공격
```csharp
public class Enemy : MonoBehaviour
{
    public float attackInterval = 2f;
    public int attackCount = 3;
    
    void Start()
    {
        StartCoroutine(AttackRoutine());
    }
    
    IEnumerator AttackRoutine()
    {
        while (true)
        {
            // 3번 연속 공격
            for (int i = 0; i < attackCount; i++)
            {
                Attack();
                yield return new WaitForSeconds(0.5f); // 공격 간격
            }
            
            // 다음 공격까지 대기
            yield return new WaitForSeconds(attackInterval);
        }
    }
    
    void Attack()
    {
        Debug.Log("공격!");
    }
}
```

---

### 실전 예시 3: 스폰 시스템
```csharp
public class EnemySpawner : MonoBehaviour
{
    public GameObject enemyPrefab;
    public Transform[] spawnPoints;
    public float spawnInterval = 3f;
    
    void Start()
    {
        StartCoroutine(SpawnEnemies());
    }
    
    IEnumerator SpawnEnemies()
    {
        while (true)
        {
            // 랜덤 위치에서 적 생성
            Transform spawnPoint = spawnPoints[Random.Range(0, spawnPoints.Length)];
            Instantiate(enemyPrefab, spawnPoint.position, spawnPoint.rotation);
            
            // 다음 스폰까지 대기
            yield return new WaitForSeconds(spawnInterval);
        }
    }
}
```

---

### 실전 예시 4: 체력 회복
```csharp
public class HealthComponent : MonoBehaviour
{
    public float maxHp = 100f;
    public float currentHp;
    public float regenRate = 5f; // 초당 회복량
    
    void Start()
    {
        currentHp = maxHp;
        StartCoroutine(RegenerateHealth());
    }
    
    IEnumerator RegenerateHealth()
    {
        while (true)
        {
            if (currentHp < maxHp)
            {
                currentHp += regenRate * Time.deltaTime;
                currentHp = Mathf.Min(currentHp, maxHp);
            }
            
            yield return null; // 매 프레임 체크
        }
    }
}
```

---

### 코루틴 제어
```csharp
public class CoroutineControl : MonoBehaviour
{
    private Coroutine myCoroutine;
    
    void Start()
    {
        // 코루틴 시작하고 참조 저장
        myCoroutine = StartCoroutine(MyCoroutine());
    }
    
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            // 코루틴 중지
            if (myCoroutine != null)
            {
                StopCoroutine(myCoroutine);
                myCoroutine = null;
            }
        }
        
        if (Input.GetKeyDown(KeyCode.R))
        {
            // 모든 코루틴 중지
            StopAllCoroutines();
        }
    }
    
    IEnumerator MyCoroutine()
    {
        while (true)
        {
            Debug.Log("실행 중...");
            yield return new WaitForSeconds(1f);
        }
    }
}
```

---

## 7. 컴포지션 + 코루틴 종합 예시
```csharp
// 공격 컴포넌트
public class AttackComponent : MonoBehaviour
{
    public float attackCooldown = 2f;
    public int damage = 10;
    private bool canAttack = true;
    
    public void Attack(Transform target)
    {
        if (!canAttack) return;
        
        // 공격 실행
        HealthComponent targetHealth = target.GetComponent<HealthComponent>();
        if (targetHealth != null)
        {
            targetHealth.TakeDamage(damage);
        }
        
        // 쿨다운 시작
        StartCoroutine(AttackCooldown());
    }
    
    IEnumerator AttackCooldown()
    {
        canAttack = false;
        yield return new WaitForSeconds(attackCooldown);
        canAttack = true;
    }
}

// AI 컴포넌트
public class AIComponent : MonoBehaviour
{
    private DetectionComponent detection;
    private MoveComponent movement;
    private AttackComponent attack;
    
    public float attackRange = 2f;
    
    void Start()
    {
        // 필요한 컴포넌트 가져오기
        detection = GetComponent<DetectionComponent>();
        movement = GetComponent<MoveComponent>();
        attack = GetComponent<AttackComponent>();
        
        // AI 루틴 시작
        StartCoroutine(AIRoutine());
    }
    
    IEnumerator AIRoutine()
    {
        while (true)
        {
            if (detection.target != null)
            {
                float distance = Vector3.Distance(transform.position, detection.target.position);
                
                if (distance > attackRange)
                {
                    // 타겟이 멀면 이동
                    Vector3 direction = (detection.target.position - transform.position).normalized;
                    movement.Move(direction);
                }
                else
                {
                    // 타겟이 가까우면 공격
                    attack.Attack(detection.target);
                }
            }
            
            yield return new WaitForSeconds(0.1f); // 0.1초마다 체크
        }
    }
}
```

---

## 8. is-a vs has-a 비교

| 구분 | is-a (상속) | has-a (컴포지션) |
|------|-------------|------------------|
| 관계 | "~이다" | "~를 가지고 있다" |
| 구조 | 경직적 | 유연함 |
| 재사용 | 어려움 | 쉬움 |
| 확장 | 제한적 | 자유로움 |
| 수정 | 영향 범위 큼 | 영향 범위 작음 |
| Unity 권장 | ❌ | ✅ |

---

### 상속 사용이 적절한 경우
```csharp
// 매우 명확한 is-a 관계
public abstract class Weapon : MonoBehaviour
{
    public abstract void Fire();
}

public class Gun : Weapon
{
    public override void Fire()
    {
        // 총 발사
    }
}

public class Bow : Weapon
{
    public override void Fire()
    {
        // 활 발사
    }
}
```

---

### 컴포지션 사용이 적절한 경우 (대부분)
```csharp
// 다양한 조합이 필요한 경우
// 플레이어 오브젝트:
// - HealthComponent
// - MoveComponent
// - JumpComponent
// - AttackComponent
// - InventoryComponent

// 날아다니는 적 오브젝트:
// - HealthComponent
// - FlyMovementComponent
// - DetectionComponent
// - AttackComponent

// 포탑 오브젝트:
// - HealthComponent
// - DetectionComponent
// - AttackComponent
```

---

## 💡 핵심 포인트

1. **is-a 한계**: 상속은 경직되고 확장이 어려움
2. **has-a 장점**: 컴포지션은 유연하고 재사용 가능
3. **Unity 철학**: 컴포넌트 기반 설계가 핵심
4. **탐지 컴포넌트**: 다양한 오브젝트에 재사용 가능
5. **코루틴**: IEnumerator와 yield return 사용
6. **비동기 처리**: 시간 기반 작업에 코루틴 활용
7. **코루틴 제어**: StartCoroutine, StopCoroutine으로 관리
8. **조합의 힘**: 컴포넌트 + 코루틴으로 복잡한 AI 구현
9. **성능**: 코루틴은 Update보다 효율적
10. **유지보수**: 컴포넌트 단위 수정으로 영향 범위 최소화

---

```mermaid
classDiagram
    class GameObject {
        +string name
        +Transform transform
        +SetActive(bool)
    }
    
    class HealthComponent {
        +float maxHp
        +float currentHp
        +TakeDamage(float)
        +Die()
    }
    
    class MoveComponent {
        +float moveSpeed
        +Move(Vector3)
    }
    
    class DetectionComponent {
        +float detectionRadius
        +LayerMask targetLayer
        +Transform target
        +DetectTarget()
    }
    
    class AttackComponent {
        +float attackCooldown
        +int damage
        +bool canAttack
        +Attack(Transform)
        +AttackCooldown() IEnumerator
    }
    
    class AIComponent {
        +float attackRange
        +AIRoutine() IEnumerator
    }
    
    class Enemy {
    }
    
    class Player {
    }
    
    GameObject <|-- Enemy : is-a
    GameObject <|-- Player : is-a
    Enemy o-- HealthComponent : has-a
    Enemy o-- MoveComponent : has-a
    Enemy o-- DetectionComponent : has-a
    Enemy o-- AttackComponent : has-a
    Enemy o-- AIComponent : has-a
    Player o-- HealthComponent : has-a
    Player o-- MoveComponent : has-a
    AIComponent ..> DetectionComponent : uses
    AIComponent ..> MoveComponent : uses
    AIComponent ..> AttackComponent : uses
