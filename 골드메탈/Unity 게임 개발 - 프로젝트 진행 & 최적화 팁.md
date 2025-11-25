
#Unity #GameDevelopment #Optimization #LateUpdate #BestPractice

## 📌 간단 요약
게임 개발 프로세스, LateUpdate 활용법, Find 함수 사용 시 주의사항 및 최적화 방법

---

## 1. 게임 개발 프로세스

### 올바른 개발 순서
```
1단계: 계획하기
  ├── 게임 이름
  ├── 장르
  ├── 목표
  ├── 구성
  └── 구상도

2단계: 플레이어 스크립트 작성
  ├── Rigidbody 설정
  ├── Jump 기능
  ├── Move 기능
  └── 기타 기본 기능
```

---

### 1단계: 계획하기 예시
```markdown
# 게임 기획서

## 게임 이름
- "점프 어드벤처"

## 장르
- 2D 플랫포머, 액션

## 목표
- 장애물을 피해 최대한 오래 생존하기
- 높은 점수 달성

## 구성
1. 플레이어 캐릭터
2. 회전하는 바닥
3. 떨어지는 탄환
4. UI (시간, 점수, 게임오버)

## 구상도
[간단한 게임 화면 스케치]
- 중앙: 회전 플랫폼
- 위: 탄환 생성기
- 상단: UI 표시
```

---

### 2단계: 플레이어 스크립트 작성
```csharp
public class PlayerController : MonoBehaviour
{
    // 1. 기본 컴포넌트
    Rigidbody rb;
    
    // 2. 이동 관련 변수
    public float moveSpeed = 5f;
    
    // 3. 점프 관련 변수
    public float jumpPower = 10f;
    private bool isGrounded = true;
    
    void Start()
    {
        // Rigidbody 가져오기
        rb = GetComponent<Rigidbody>();
    }
    
    void Update()
    {
        // 입력 처리
        Move();
        Jump();
    }
    
    void FixedUpdate()
    {
        // 물리 처리는 여기서
    }
    
    void Move()
    {
        float xInput = Input.GetAxis("Horizontal");
        float zInput = Input.GetAxis("Vertical");
        
        Vector3 moveDirection = new Vector3(xInput, 0, zInput);
        transform.Translate(moveDirection * moveSpeed * Time.deltaTime);
    }
    
    void Jump()
    {
        if (Input.GetKeyDown(KeyCode.Space) && isGrounded)
        {
            rb.AddForce(Vector3.up * jumpPower, ForceMode.Impulse);
            isGrounded = false;
        }
    }
    
    void OnCollisionEnter(Collision collision)
    {
        if (collision.gameObject.CompareTag("Ground"))
        {
            isGrounded = true;
        }
    }
}
```

---

## 2. 비활성화 구간의 함수 실행

### 중요 원칙

> ⚠️ **주의**: 비활성화 구간에는 컴포넌트 함수가 실행되지 않을 수 있습니다.

### 비활성화 시 동작하지 않는 함수들
```csharp
public class LifecycleTest : MonoBehaviour
{
    void Awake()
    {
        // ✅ 오브젝트 생성 시 실행 (비활성화 상태여도 실행됨)
    }
    
    void OnEnable()
    {
        // ✅ 활성화될 때마다 실행
    }
    
    void Start()
    {
        // ❌ 비활성화 상태면 실행 안됨
    }
    
    void Update()
    {
        // ❌ 비활성화 상태면 실행 안됨
    }
    
    void FixedUpdate()
    {
        // ❌ 비활성화 상태면 실행 안됨
    }
    
    void LateUpdate()
    {
        // ❌ 비활성화 상태면 실행 안됨
    }
    
    void OnDisable()
    {
        // ✅ 비활성화될 때 실행
    }
}
```

---

### 활성화/비활성화 예시
```csharp
public class ObjectManager : MonoBehaviour
{
    public GameObject player;
    
    void Start()
    {
        // 오브젝트 비활성화
        player.SetActive(false);
        // 이 상태에서는 player의 Update, FixedUpdate 등이 실행되지 않음
        
        // 3초 후 활성화
        Invoke("ActivatePlayer", 3f);
    }
    
    void ActivatePlayer()
    {
        player.SetActive(true);
        // 활성화되면 OnEnable → Start → Update 순서로 실행
    }
}
```

---

## 3. LateUpdate() 함수

### 개념

**LateUpdate()**: 모든 Update()가 끝난 후 실행되는 함수

### 실행 순서
```
Update() (모든 오브젝트)
    ↓
FixedUpdate() (물리 업데이트)
    ↓
LateUpdate() (모든 오브젝트)
```

---

### 주요 용도

#### 1. UI 업데이트
```csharp
public class UIManager : MonoBehaviour
{
    public Text scoreText;
    private int score;
    
    void Update()
    {
        // 게임 로직에서 점수 변경
        if (Input.GetKeyDown(KeyCode.Space))
        {
            score += 10;
        }
    }
    
    void LateUpdate()
    {
        // 모든 계산이 끝난 후 UI 업데이트
        scoreText.text = "Score: " + score;
    }
}
```

---

#### 2. 카메라 업데이트 (가장 중요!)
```csharp
public class CameraFollow : MonoBehaviour
{
    public Transform player;
    public Vector3 offset = new Vector3(0, 5, -10);
    
    void LateUpdate()
    {
        // 플레이어의 모든 이동이 끝난 후 카메라 이동
        transform.position = player.position + offset;
    }
}
```

> 💡 **이유**: Update()에서 카메라를 움직이면 플레이어가 움직이기 전의 위치를 따라가서 떨림 현상 발생

---

#### 3. 부드러운 추적
```csharp
public class SmoothFollow : MonoBehaviour
{
    public Transform target;
    public float smoothSpeed = 0.125f;
    public Vector3 offset;
    
    void LateUpdate()
    {
        // 타겟의 최종 위치를 기준으로 부드럽게 따라감
        Vector3 desiredPosition = target.position + offset;
        Vector3 smoothedPosition = Vector3.Lerp(transform.position, desiredPosition, smoothSpeed);
        transform.position = smoothedPosition;
        
        // 타겟을 바라보기
        transform.LookAt(target);
    }
}
```

---

### LateUpdate vs Update 비교

| 함수 | 실행 시점 | 용도 |
|------|-----------|------|
| Update | 매 프레임 시작 | 입력 처리, 게임 로직 |
| FixedUpdate | 고정 시간 간격 | 물리 계산 |
| LateUpdate | 모든 Update 후 | 카메라, UI, 후처리 |

---

### 실전 예시: 3인칭 카메라
```csharp
public class ThirdPersonCamera : MonoBehaviour
{
    public Transform player;
    public float distance = 5f;
    public float height = 2f;
    public float rotationSpeed = 5f;
    
    private float currentX = 0f;
    private float currentY = 0f;
    
    void Update()
    {
        // 마우스 입력 받기
        currentX += Input.GetAxis("Mouse X") * rotationSpeed;
        currentY -= Input.GetAxis("Mouse Y") * rotationSpeed;
        currentY = Mathf.Clamp(currentY, -35f, 60f);
    }
    
    void LateUpdate()
    {
        // 플레이어 이동이 끝난 후 카메라 위치 계산
        Vector3 direction = new Vector3(0, 0, -distance);
        Quaternion rotation = Quaternion.Euler(currentY, currentX, 0);
        
        transform.position = player.position + rotation * direction + Vector3.up * height;
        transform.LookAt(player.position + Vector3.up * height);
    }
}
```

---

## 4. Find 함수 최적화

### FindGameObjectWithTag vs FindWithTag

#### ❌ FindGameObjectWithTag (잘못된 사용)
```csharp
// .transform이 없음
GameObject player = GameObject.FindGameObjectWithTag("Player");
Transform playerTransform = player.transform; // 추가 단계 필요
```

---

#### ✅ FindWithTag (올바른 사용)
```csharp
// .transform이 있음
Transform playerTransform = GameObject.FindWithTag("Player").transform;
```

---

### Find 계열 함수 비교

| 함수 | 반환 타입 | .transform | 성능 |
|------|-----------|------------|------|
| FindGameObjectWithTag | GameObject | ❌ | 느림 |
| FindWithTag | GameObject | ❌ | 느림 |
| FindFirstObjectByType<T> | T | 타입 따라 다름 | 매우 느림 |
| FindObjectsByType<T> | T[] | 타입 따라 다름 | 매우 느림 |

---

### ⚠️ Find 함수 사용 시 주의사항

> **중요**: Find 계열 함수는 **부하를 초래**할 수 있어 **지양**하기!

#### 나쁜 예시 ❌
```csharp
public class BadExample : MonoBehaviour
{
    void Update()
    {
        // 매 프레임마다 찾기 - 매우 비효율적!
        GameObject player = GameObject.FindWithTag("Player");
        transform.LookAt(player.transform);
    }
}
```

---

#### 좋은 예시 ✅
```csharp
public class GoodExample : MonoBehaviour
{
    private Transform player;
    
    void Start()
    {
        // 한 번만 찾기
        player = GameObject.FindWithTag("Player").transform;
    }
    
    void Update()
    {
        // 저장된 참조 사용
        if (player != null)
        {
            transform.LookAt(player);
        }
    }
}
```

---

### Find 최적화 전략

#### 1. Start()에서 한 번만 찾기
```csharp
public class OptimizedFind : MonoBehaviour
{
    private Transform player;
    private GameManager gameManager;
    
    void Start()
    {
        // 게임 시작 시 한 번만 찾기
        player = GameObject.FindWithTag("Player").transform;
        gameManager = FindFirstObjectByType<GameManager>();
    }
    
    void Update()
    {
        // 저장된 참조 사용
        if (player != null)
        {
            float distance = Vector3.Distance(transform.position, player.position);
        }
    }
}
```

---

#### 2. 싱글톤 패턴 사용
```csharp
public class GameManager : MonoBehaviour
{
    public static GameManager Instance;
    
    void Awake()
    {
        // 싱글톤 설정
        if (Instance == null)
        {
            Instance = this;
        }
        else
        {
            Destroy(gameObject);
        }
    }
}

// 다른 스크립트에서 사용
public class OtherScript : MonoBehaviour
{
    void Update()
    {
        // Find 없이 바로 접근
        GameManager.Instance.AddScore(10);
    }
}
```

---

#### 3. Inspector에서 직접 할당
```csharp
public class DirectReference : MonoBehaviour
{
    // Inspector에서 드래그 앤 드롭으로 할당
    public Transform player;
    public GameManager gameManager;
    
    void Update()
    {
        // Find 없이 바로 사용
        if (player != null)
        {
            transform.LookAt(player);
        }
    }
}
```

> 💡 **가장 권장**: Inspector에서 직접 할당하는 방법이 가장 안전하고 빠름!

---

#### 4. 이벤트 시스템 사용
```csharp
// 이벤트 매니저
public class EventManager : MonoBehaviour
{
    public static event Action<Transform> OnPlayerSpawned;
    
    public static void PlayerSpawned(Transform player)
    {
        OnPlayerSpawned?.Invoke(player);
    }
}

// 플레이어
public class Player : MonoBehaviour
{
    void Start()
    {
        EventManager.PlayerSpawned(transform);
    }
}

// 적
public class Enemy : MonoBehaviour
{
    private Transform player;
    
    void OnEnable()
    {
        EventManager.OnPlayerSpawned += OnPlayerSpawned;
    }
    
    void OnDisable()
    {
        EventManager.OnPlayerSpawned -= OnPlayerSpawned;
    }
    
    void OnPlayerSpawned(Transform spawnedPlayer)
    {
        player = spawnedPlayer; // Find 없이 플레이어 참조 획득
    }
}
```

---

## 5. 성능 비교

### Find 함수 처리 시간 비교
```csharp
public class PerformanceTest : MonoBehaviour
{
    void Start()
    {
        System.Diagnostics.Stopwatch sw = new System.Diagnostics.Stopwatch();
        
        // FindGameObjectWithTag 테스트
        sw.Start();
        for (int i = 0; i < 1000; i++)
        {
            GameObject.FindGameObjectWithTag("Player");
        }
        sw.Stop();
        Debug.Log($"FindGameObjectWithTag: {sw.ElapsedMilliseconds}ms");
        
        // FindFirstObjectByType 테스트
        sw.Restart();
        for (int i = 0; i < 1000; i++)
        {
            FindFirstObjectByType<Player>();
        }
        sw.Stop();
        Debug.Log($"FindFirstObjectByType: {sw.ElapsedMilliseconds}ms");
        
        // 직접 참조 테스트
        Transform player = GameObject.FindWithTag("Player").transform;
        sw.Restart();
        for (int i = 0; i < 1000; i++)
        {
            Vector3 pos = player.position;
        }
        sw.Stop();
        Debug.Log($"Direct Reference: {sw.ElapsedMilliseconds}ms");
    }
}
```

**결과 예시:**
- FindGameObjectWithTag: ~500ms
- FindFirstObjectByType: ~800ms
- Direct Reference: ~1ms (500배 빠름!)

---

## 💡 핵심 포인트

1. **개발 순서**: 계획 → 플레이어 → 나머지 순서로 진행
2. **비활성화**: SetActive(false) 상태에서는 Update 계열 함수 실행 안됨
3. **LateUpdate**: 카메라와 UI 업데이트에 필수적으로 사용
4. **카메라 추적**: 반드시 LateUpdate()에서 처리 (떨림 방지)
5. **Find 함수**: Update()에서 절대 사용 금지
6. **Find 최적화**: Start()에서 한 번만 찾거나 Inspector에서 직접 할당
7. **싱글톤**: 자주 접근하는 매니저는 싱글톤 패턴 사용
8. **직접 참조**: Inspector 할당이 가장 안전하고 빠름
9. **FindWithTag**: FindGameObjectWithTag보다 간결함
10. **성능**: Find 함수는 직접 참조보다 수백 배 느림