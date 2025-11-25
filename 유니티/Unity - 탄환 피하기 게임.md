
#Unity #GameLogic #PlayerController #BulletSystem #GameManager #UI

## 📌 간단 요약
플레이어 제어, 탄환 시스템, UI 관리, 게임 매니저를 통한 완전한 2D 슈팅 게임 구현 방법

---

## 1. 플레이어 로직

### 기본 설정 순서
```
1. 플레이어 오브젝트에 Rigidbody 생성
2. PlayerController script 생성
3. Rigidbody 변수, speed float 변수 선언
4. Update()에서 Input으로 이동 설정
5. Die() 함수 생성
```

### 이동 개선 로직
```csharp
public class PlayerController : MonoBehaviour
{
    Rigidbody rb;
    public float speed = 5f;
    
    void Start()
    {
        rb = GetComponent<Rigidbody>();
    }
    
    void Update()
    {
        // 1. 수평/수직 입력값 감지
        float xInput = Input.GetAxis("Horizontal");
        float zInput = Input.GetAxis("Vertical");
        
        // 2. 실제 이동 속도 결정
        Vector3 newVelocity = new Vector3(xInput, 0, zInput) * speed;
        
        // 3. Rigidbody 속도에 대입
        rb.velocity = newVelocity;
    }
    
    void Die()
    {
        // 플레이어 사망 처리
        gameObject.SetActive(false);
    }
}
```

---

## 2. 탄환 발사 시스템

### 탄환 오브젝트 설정
```
1. 탄환 오브젝트 생성
2. Rigidbody 추가
3. Use Gravity 해제
4. Bullet script 생성
```

### 탄환 Script
```csharp
public class Bullet : MonoBehaviour
{
    Rigidbody rb;
    public float speed = 10f;
    
    void Start()
    {
        rb = GetComponent<Rigidbody>();
        rb.velocity = transform.forward * speed;
        
        // 3초 후 파괴
        Destroy(gameObject, 3f);
    }
    
    // 충돌 시 탄환 파괴
    void OnTriggerEnter(Collider other)
    {
        // 플레이어와 충돌했는지 확인
        PlayerController player = other.GetComponent<PlayerController>();
        
        if (player != null)
        {
            player.Die(); // 플레이어 사망 처리
            Destroy(gameObject); // 탄환 파괴
        }
    }
}
```

---

## 3. 탄환 생성기 로직

### BulletSpawner Script
```csharp
public class BulletSpawner : MonoBehaviour
{
    public GameObject bulletPrefab; // 원본 프리팹
    public float spawnRateMin = 0.5f; // 최소 생성 간격
    public float spawnRateMax = 2f; // 최대 생성 간격
    
    private Transform target; // 조준할 대상
    private float spawnRate; // 실제 생성 간격
    private float timeAfterSpawn; // 마지막 생성 이후 시간
    
    void Start()
    {
        // 누적 시간 초기화
        timeAfterSpawn = 0f;
        
        // 랜덤 생성 간격 설정
        spawnRate = Random.Range(spawnRateMin, spawnRateMax);
        
        // 플레이어를 타겟으로 설정
        target = FindFirstObjectByType<PlayerController>().transform;
    }
    
    void Update()
    {
        // 시간 누적
        timeAfterSpawn += Time.deltaTime;
        
        // 생성 간격 도달 시
        if (timeAfterSpawn >= spawnRate)
        {
            // 시간 리셋
            timeAfterSpawn = 0f;
            
            // 새 탄환 생성
            GameObject bullet = Instantiate(bulletPrefab, transform.position, transform.rotation);
            
            // 탄환이 타겟을 바라보도록 회전
            bullet.transform.LookAt(target);
            
            // 다음 생성 간격 랜덤 설정
            spawnRate = Random.Range(spawnRateMin, spawnRateMax);
        }
    }
}
```

---

## 4. 바닥 회전 로직

### Rotator Script
```csharp
public class Rotator : MonoBehaviour
{
    public float rotationSpeed = 30f;
    
    void Update()
    {
        // Y축 기준 회전
        transform.Rotate(0, rotationSpeed * Time.deltaTime, 0);
    }
}
```

---

## 5. UI Text 시스템

### UI 설정 순서
```
1. Hierarchy > UI > Legacy > Text
2. Anchor 프리셋에서 Alt + 중앙 상단 클릭 (스냅핑)
3. Rect Transform의 Pos Y를 -30으로 변경
4. Font Size를 42로 변경
5. Horizontal/Vertical Overflow를 Overflow로 변경
6. Shadow 컴포넌트 추가
```

### GameOver Text 추가
```
1. Time Text 복제
2. GameOver Text로 이름 변경
3. Text를 "Press R to Restart"로 변경
4. Anchor 프리셋에서 Alt + 정중앙 클릭
```

---

## 6. 게임 매니저 로직

### GameManager Script
```csharp
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.SceneManagement;

public class GameManager : MonoBehaviour
{
    public GameObject gameoverText; // 게임오버 텍스트
    public Text timeText; // 생존 시간 텍스트
    public Text recordText; // 최고 기록 텍스트
    
    private float surviveTime; // 생존 시간
    private bool isGameover; // 게임오버 상태
    
    void Start()
    {
        // 초기화
        surviveTime = 0f;
        isGameover = false;
    }
    
    void Update()
    {
        // 게임오버가 아니면 시간 증가
        if (!isGameover)
        {
            surviveTime += Time.deltaTime;
            timeText.text = "Time: " + (int)surviveTime;
        }
        else
        {
            // 게임오버 상태에서 R키 누르면 재시작
            if (Input.GetKeyDown(KeyCode.R))
            {
                SceneManager.LoadScene("SampleScene");
            }
        }
    }
    
    public void EndGame()
    {
        // 게임오버 상태로 전환
        isGameover = true;
        
        // 게임오버 텍스트 활성화
        gameoverText.SetActive(true);
        
        // 이전 최고 기록 가져오기
        float bestTime = PlayerPrefs.GetFloat("BestTime");
        
        // 현재 생존 시간이 최고 기록보다 크면
        if (surviveTime > bestTime)
        {
            bestTime = surviveTime;
            // 새로운 최고 기록 저장
            PlayerPrefs.SetFloat("BestTime", bestTime);
        }
        
        // 최고 기록 표시
        recordText.text = "Best Time: " + (int)bestTime;
    }
}
```

### PlayerController에 GameManager 연동
```csharp
public class PlayerController : MonoBehaviour
{
    // ... 기존 코드 ...
    
    void Die()
    {
        gameObject.SetActive(false);
        
        // GameManager 찾아서 EndGame 호출
        GameManager gameManager = FindFirstObjectByType<GameManager>();
        gameManager.EndGame();
    }
}
```

---

## 7. 주요 함수 & 개념

### transform.forward
```csharp
// 해당 오브젝트의 Z축 방향을 나타내는 Vector3
rb.velocity = transform.forward * speed;
```

---

### Random.Range()
```csharp
// int 타입: 최댓값 미포함 (0, 1, 2 중 하나)
int randomInt = Random.Range(0, 3);

// float 타입: 최댓값 포함 (0f ~ 3f 사이)
float randomFloat = Random.Range(0f, 3f);
```

---

### FindFirstObjectByType<T>()
```csharp
// Scene에서 해당 타입의 오브젝트를 찾아 반환
PlayerController player = FindFirstObjectByType<PlayerController>();
```
> ⚠️ **주의**: 처리 비용이 크므로 Start()에서만 사용!

---

### FindObjectsByType<T>()
```csharp
// 해당 타입의 모든 오브젝트를 배열로 반환
Enemy[] enemies = FindObjectsByType<Enemy>();
```

---

### LookAt()
```csharp
// 입력받은 Transform을 바라보도록 회전 변경
bullet.transform.LookAt(target);
```

---

### SetActive()
```csharp
// 게임 오브젝트 활성화/비활성화
gameObject.SetActive(false); // 비활성화
gameoverText.SetActive(true); // 활성화
```

---

### Input.GetAxis()

| 입력 | 반환값 |
|------|--------|
| 아무것도 안누름 | 0 |
| 음의 방향 (A, 왼쪽, 아래) | -1.0 |
| 양의 방향 (D, 오른쪽, 위) | 1.0 |
| 조이스틱 살짝 기울임 | -1.0 ~ 1.0 사이 값 |
```csharp
// Horizontal: A/D 또는 좌우 방향키
float xInput = Input.GetAxis("Horizontal");

// Vertical: W/S 또는 상하 방향키
float zInput = Input.GetAxis("Vertical");
```

---

### AddForce vs velocity (linearVelocity)

| 방식 | 특징 | 용도 |
|------|------|------|
| AddForce | 힘을 누적, 속력 점진적 증가 | 자연스러운 가속 |
| velocity | 이전 속도 무시, 즉시 변경 | 관성 무시, 정확한 이동 |
```csharp
// AddForce: 힘 누적
rb.AddForce(Vector3.forward * speed);

// velocity: 즉시 속도 변경
rb.velocity = new Vector3(xInput, 0, zInput) * speed;
```

---

## 8. UI Canvas 설정

### Anchor 프리셋 스냅핑
- **Alt 키 + 프리셋 클릭**: UI 게임 오브젝트가 해당 방향 모서리에 달라붙음
- **용도**: 화면 크기가 변해도 UI 위치 유지

### Text UI 코딩으로 변경
```csharp
Text timeText;

void Update()
{
    timeText.text = "Time: " + (int)surviveTime;
}
```

---

## 9. Scene 관리

### SceneManager.LoadScene()
```csharp
using UnityEngine.SceneManagement;

// Scene 로드
SceneManager.LoadScene("SampleScene");
```

> ⚠️ **주의**: File > Build Settings에 Scene이 추가되어 있어야 함!

---

## 10. PlayerPrefs (데이터 저장)

### 개념
- **키-값** 단위로 데이터를 로컬에 저장
- 게임을 껐다 켜도 데이터 유지

### Float 값 저장/불러오기
```csharp
// 저장
PlayerPrefs.SetFloat("BestTime", bestTime);

// 불러오기
float bestTime = PlayerPrefs.GetFloat("BestTime");
```

### Int 값 저장/불러오기
```csharp
// 저장
PlayerPrefs.SetInt("Score", score);

// 불러오기
int score = PlayerPrefs.GetInt("Score");
```

### String 값 저장/불러오기
```csharp
// 저장
PlayerPrefs.SetString("PlayerName", "Player1");

// 불러오기
string name = PlayerPrefs.GetString("PlayerName");
```

### 기본값
- GetInt(), GetFloat(): 0
- GetString(): ""

### 키 존재 확인
```csharp
// 해당 키로 저장된 값이 있는지 확인
if (PlayerPrefs.HasKey("BestTime"))
{
    float bestTime = PlayerPrefs.GetFloat("BestTime");
}
else
{
    // 처음 실행, 기본값 설정
    PlayerPrefs.SetFloat("BestTime", 0f);
}
```

---

## 💡 핵심 포인트

1. **플레이어 이동**: velocity를 사용하면 관성 무시하고 정확한 이동 가능
2. **탄환 시스템**: Prefab + Spawner 패턴으로 효율적 관리
3. **UI Anchor**: Alt + 프리셋으로 스냅핑하여 반응형 UI 구현
4. **게임 매니저**: 게임 전체 상태를 하나의 오브젝트로 관리
5. **PlayerPrefs**: 최고 기록 등 간단한 데이터 저장에 유용
6. **FindFirstObjectByType**: Start()에서만 사용하여 성능 최적화
7. **Scene 로드**: Build Settings에 Scene 추가 필수
8. **Random.Range**: int와 float의 동작 방식 차이 주의