---
tags:
  - Unity
  - Transform
  - Position
  - SerializeField
  - MonoBehaviour
  - GetComponent
  - Instantiate
  - 게임오브젝트
  - 컴포넌트
  - 직렬화
---
## Transform.position

### 개념
**transform.position을 바꾼다 = 월드 좌표를 바꾼다**

Transform의 position은 월드 좌표계 기준의 위치입니다.

### 사용법
```csharp
// X축으로 1 이동
transform.position = transform.position + new Vector3(1, 0, 0);

// 뒤쪽으로 이동
transform.position = transform.position + Vector3.back;
```

## Rotation 값 확인
```csharp
// 오브젝트의 회전값 출력
Debug.Log(transform.eulerAngles);
```

## SerializeField

### 개념
**private 필드를 직렬화**하여 Unity 인스펙터에 노출시키는 속성

### 사용 이유

1. **private 필드의 직렬화**: 캡슐화를 유지하면서 인스펙터에서 수정 가능
2. **직렬화 커스터마이징**: 특정 필드만 선택적으로 노출
3. **코드 유지보수**: public 없이도 인스펙터 사용 가능

### 문법
```csharp
[SerializeField]
private int health = 100;

[SerializeField]
private float speed = 5.0f;
```

## MonoBehaviour와 객체 생성

### 규칙

**MonoBehaviour 상속 클래스**:
- `new`로 생성하지 **않는다**
- Unity가 생명주기를 관리하므로 직접 생성 금지
- `new` 사용 시 경고 발생
```csharp
// ❌ 잘못된 사용
PlayerController player = new PlayerController();  // 경고!

// ✅ 올바른 사용
// AddComponent 또는 GetComponent 사용
```

**일반 클래스** (MonoBehaviour 미상속):
- `new`로 생성 **가능**
```csharp
// ✅ 정상 동작
MyDataClass data = new MyDataClass();
```

## 컴포넌트 접근 메서드

### GetComponent<T>()

**같은 게임 오브젝트**에서 컴포넌트를 찾아 참조를 가져옴
```csharp
Rigidbody rb = gameObject.GetComponent<Rigidbody>();
PlayerController player = GetComponent<PlayerController>();
```

### AddComponent<T>()

게임 오브젝트에 **컴포넌트를 추가**
```csharp
Rigidbody rb = gameObject.AddComponent<Rigidbody>();
```

### GetComponentInChildren<T>()

**자기 자신 + 모든 자식 오브젝트**에서 컴포넌트 검색
```csharp
// 자식 오브젝트까지 포함하여 검색
Renderer renderer = GetComponentInChildren<Renderer>();
```

## 다른 오브젝트의 컴포넌트 접근

### 방법 1: 인스펙터에서 미리 설정
```csharp
[SerializeField]
private GameObject targetObject;

void Start()
{
    // targetObject는 인스펙터에서 드래그앤드롭으로 설정
    PlayerController controller = targetObject.GetComponent<PlayerController>();
}
```

### 방법 2: 코드로 찾아서 접근
```csharp
GameObject target;

void Start()
{
    target = GameObject.Find("PlayerObject");
    PlayerController controller = target.GetComponent<PlayerController>();
}
```

## Instantiate (인스턴스화)

### 개념
게임 오브젝트를 **복사해서 생성**하는 함수

### 문법
```csharp
Instantiate(복사할_대상, 위치, 회전);
```

### 예시
```csharp
// 프리팹 복사
public GameObject bulletPrefab;

void Fire()
{
    // 총알 프리팹을 현재 위치에 생성
    Instantiate(bulletPrefab, transform.position, transform.rotation);
}
```
```csharp
// 특정 위치에 생성
Vector3 spawnPos = new Vector3(0, 5, 0);
Quaternion rotation = Quaternion.identity;
Instantiate(enemyPrefab, spawnPos, rotation);
```

### 반환값 활용
```csharp
// 생성된 객체의 참조 받기
GameObject newBullet = Instantiate(bulletPrefab, transform.position, transform.rotation);

// 생성된 객체 조작
newBullet.GetComponent<Rigidbody>().velocity = transform.forward * 10;
```

## 실전 예시

### 플레이어 이동과 발사
```csharp
public class PlayerController : MonoBehaviour
{
    [SerializeField]
    private float speed = 5.0f;
    
    [SerializeField]
    private GameObject bulletPrefab;
    
    void Update()
    {
        // 이동
        float h = Input.GetAxis("Horizontal");
        transform.position = transform.position + new Vector3(h * speed * Time.deltaTime, 0, 0);
        
        // 발사
        if (Input.GetKeyDown(KeyCode.Space))
        {
            Instantiate(bulletPrefab, transform.position, transform.rotation);
        }
    }
}
```

### 컴포넌트 간 통신
```csharp
public class GameManager : MonoBehaviour
{
    [SerializeField]
    private GameObject player;
    
    private PlayerController playerController;
    
    void Start()
    {
        // 플레이어의 컴포넌트 가져오기
        playerController = player.GetComponent<PlayerController>();
    }
    
    public void PowerUp()
    {
        // 플레이어 속도 증가
        playerController.IncreaseSpeed();
    }
}
```

## 핵심 정리

| 항목 | 설명 |
|------|------|
| transform.position | 월드 좌표 직접 변경 |
| SerializeField | private 필드 직렬화 |
| MonoBehaviour | new 사용 금지 (Unity 관리) |
| GetComponent | 같은 오브젝트의 컴포넌트 |
| GetComponentInChildren | 자식 포함 검색 |
| Instantiate | 오브젝트 복사 생성 |

## 관련 개념
- Transform 제어
- Unity 생명주기
- 컴포넌트 패턴
