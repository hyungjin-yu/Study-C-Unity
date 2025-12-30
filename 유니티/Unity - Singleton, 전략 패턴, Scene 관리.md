#Unity #DesignPattern #Singleton #Strategy #SceneManagement #Serialization

## 📑 목차

1. System.Serializable
2. 전략 패턴 (Strategy Pattern)
3. Singleton 패턴
4. DontDestroyOnLoad
5. 순수 C# 클래스와 Unity 연결
6. 실전 예시
7. 핵심 포인트

## 📌 간단 요약

Unity에서 자주 사용되는 Singleton 패턴과 전략 패턴, System.Serializable을 통한 Inspector 연동, 그리고 Scene 전환 시 오브젝트 유지 방법

---

## 1. System.Serializable

### 개념

**System.Serializable**: 클래스 내부의 변수를 Unity Inspector에서 제어 가능하게 만드는 특성(Attribute)

---

### 기본 사용법

```csharp
using System;
using UnityEngine;

// Serializable 속성 추가
[System.Serializable]
public class PlayerData
{
    public string playerName;
    public int level;
    public float health;
    public Vector3 position;
}

public class GameManager : MonoBehaviour
{
    // Inspector에서 편집 가능
    public PlayerData player;
    
    void Start()
    {
        Debug.Log($"플레이어: {player.playerName}, 레벨: {player.level}");
    }
}
```

**결과:**

- Inspector에서 PlayerData의 모든 필드 편집 가능
- 게임 실행 없이 데이터 조정 가능

---

### Serializable이 없을 때

```csharp
// ❌ Serializable 없음
public class EnemyData
{
    public string enemyName;
    public int damage;
}

public class Enemy : MonoBehaviour
{
    public EnemyData data; // Inspector에 표시 안됨!
    
    void Start()
    {
        // data는 null
        Debug.Log(data.enemyName); // NullReferenceException!
    }
}
```

---

### 실전 활용 예시

````csharp
[System.Serializable]
public class WeaponStats
{
    public string weaponName;
    public int damage;
    public float attackSpeed;
    public Sprite icon;
}

[System.Serializable]
public class CharacterStats
{
    public string characterName;
    public int maxHP;
    public int maxMP;
    public float moveSpeed;
    public WeaponStats weapon; // 중첩 가능
}

public class Character : MonoBehaviour
{
    public CharacterStats stats;
    
    void Start()
    {
        Debug.Log($"{stats.characterName}의 무기: {stats.weapon.weaponName}");
        Debug.Log($"공격력: {stats.weapon.damage}");
    }
}
```

**Inspector 구조:**
```
Character (Script)
├─ Stats
│  ├─ Character Name: "전사"
│  ├─ Max HP: 1000
│  ├─ Max MP: 100
│  ├─ Move Speed: 5.0
│  └─ Weapon
│     ├─ Weapon Name: "전설의 검"
│     ├─ Damage: 100
│     ├─ Attack Speed: 1.5
│     └─ Icon: [Sprite]
````

---

### List와 Array 직렬화

```csharp
[System.Serializable]
public class Item
{
    public string itemName;
    public int price;
}

public class Inventory : MonoBehaviour
{
    // 리스트도 Inspector에서 편집 가능
    public List<Item> items = new List<Item>();
    
    // 배열도 가능
    public Item[] itemArray;
    
    void Start()
    {
        foreach (Item item in items)
        {
            Debug.Log($"{item.itemName}: {item.price}골드");
        }
    }
}
```

---

## 2. 전략 패턴 (Strategy Pattern)

### 개념

**전략 패턴**: 알고리즘을 캡슐화하여 런타임에 동적으로 교체할 수 있게 하는 패턴

**장점:**

- 알고리즘을 독립적으로 변경 가능
- 새로운 전략 추가 용이
- if-else 분기문 제거

---

### 기본 구조

```csharp
// 1. 전략 인터페이스
public interface IAttackStrategy
{
    void Attack();
}

// 2. 구체적인 전략들
public class MeleeAttack : IAttackStrategy
{
    public void Attack()
    {
        Debug.Log("근접 공격!");
    }
}

public class RangedAttack : IAttackStrategy
{
    public void Attack()
    {
        Debug.Log("원거리 공격!");
    }
}

public class MagicAttack : IAttackStrategy
{
    public void Attack()
    {
        Debug.Log("마법 공격!");
    }
}

// 3. 컨텍스트 (전략 사용자)
public class Player : MonoBehaviour
{
    private IAttackStrategy attackStrategy;
    
    // 전략 설정
    public void SetAttackStrategy(IAttackStrategy strategy)
    {
        attackStrategy = strategy;
    }
    
    // 전략 실행
    public void PerformAttack()
    {
        if (attackStrategy != null)
        {
            attackStrategy.Attack();
        }
    }
}
```

**사용 예시:**

```csharp
void Start()
{
    Player player = GetComponent<Player>();
    
    // 전략 교체
    player.SetAttackStrategy(new MeleeAttack());
    player.PerformAttack(); // "근접 공격!"
    
    player.SetAttackStrategy(new MagicAttack());
    player.PerformAttack(); // "마법 공격!"
}
```

---

### 실전 예시: 적 AI 전략

```csharp
// 전략 인터페이스
public interface IEnemyBehavior
{
    void Execute(Transform enemy, Transform target);
}

// 공격적 행동
public class AggressiveBehavior : IEnemyBehavior
{
    public void Execute(Transform enemy, Transform target)
    {
        // 타겟을 향해 돌진
        Vector3 direction = (target.position - enemy.position).normalized;
        enemy.position += direction * 5f * Time.deltaTime;
    }
}

// 방어적 행동
public class DefensiveBehavior : IEnemyBehavior
{
    public void Execute(Transform enemy, Transform target)
    {
        // 타겟에서 후퇴
        Vector3 direction = (enemy.position - target.position).normalized;
        enemy.position += direction * 2f * Time.deltaTime;
    }
}

// 패트롤 행동
public class PatrolBehavior : IEnemyBehavior
{
    private Vector3[] patrolPoints;
    private int currentPoint = 0;
    
    public PatrolBehavior(Vector3[] points)
    {
        patrolPoints = points;
    }
    
    public void Execute(Transform enemy, Transform target)
    {
        // 패트롤 지점 순회
        Vector3 targetPoint = patrolPoints[currentPoint];
        enemy.position = Vector3.MoveTowards(enemy.position, targetPoint, 3f * Time.deltaTime);
        
        if (Vector3.Distance(enemy.position, targetPoint) < 0.1f)
        {
            currentPoint = (currentPoint + 1) % patrolPoints.Length;
        }
    }
}

// 적 AI
public class Enemy : MonoBehaviour
{
    private IEnemyBehavior currentBehavior;
    private Transform player;
    
    void Start()
    {
        player = GameObject.FindWithTag("Player").transform;
        
        // 기본 전략: 패트롤
        Vector3[] points = new Vector3[] 
        {
            new Vector3(0, 0, 0),
            new Vector3(10, 0, 0),
            new Vector3(10, 0, 10)
        };
        currentBehavior = new PatrolBehavior(points);
    }
    
    void Update()
    {
        float distanceToPlayer = Vector3.Distance(transform.position, player.position);
        
        // 거리에 따라 전략 변경
        if (distanceToPlayer < 3f)
        {
            currentBehavior = new AggressiveBehavior(); // 가까우면 공격
        }
        else if (distanceToPlayer < 7f)
        {
            float health = GetComponent<Health>().currentHP;
            if (health < 30)
                currentBehavior = new DefensiveBehavior(); // 체력 낮으면 후퇴
            else
                currentBehavior = new AggressiveBehavior();
        }
        else
        {
            // 멀면 패트롤
            // currentBehavior 유지
        }
        
        // 현재 전략 실행
        currentBehavior.Execute(transform, player);
    }
}
```

---

### 전략 패턴 vs if-else 비교

#### ❌ if-else 방식 (나쁜 예시)

```csharp
public class PlayerBad : MonoBehaviour
{
    public enum AttackType { Melee, Ranged, Magic }
    public AttackType currentAttack;
    
    public void Attack()
    {
        if (currentAttack == AttackType.Melee)
        {
            Debug.Log("근접 공격!");
        }
        else if (currentAttack == AttackType.Ranged)
        {
            Debug.Log("원거리 공격!");
        }
        else if (currentAttack == AttackType.Magic)
        {
            Debug.Log("마법 공격!");
        }
        
        // 새 공격 타입 추가하려면 여기 수정 필요
    }
}
```

#### ✅ 전략 패턴 방식 (좋은 예시)

```csharp
public class PlayerGood : MonoBehaviour
{
    private IAttackStrategy attackStrategy;
    
    public void SetAttackStrategy(IAttackStrategy strategy)
    {
        attackStrategy = strategy;
    }
    
    public void Attack()
    {
        attackStrategy?.Attack();
        
        // 새 공격 타입 추가해도 이 코드는 변경 불필요!
    }
}

// 새 전략 추가도 쉬움
public class PoisonAttack : IAttackStrategy
{
    public void Attack()
    {
        Debug.Log("독 공격!");
    }
}
```

---

## 3. Singleton 패턴

### 개념

**Singleton 패턴**: 객체가 하나만 존재하도록 보장하는 패턴

**사용 이유:**

- 게임 전체에서 하나만 필요한 객체 (GameManager, SoundManager 등)
- 어디서든 접근 가능한 전역 관리자
- 중복 생성 방지

---

### 기본 구조

```csharp
public class GameManager : MonoBehaviour
{
    // 1. private static 변수로 인스턴스 저장
    private static GameManager instance = null;
    
    // 2. public static 프로퍼티로 접근 제공
    public static GameManager Instance
    {
        get
        {
            // 인스턴스가 없으면 찾기
            if (instance == null)
            {
                instance = FindFirstObjectByType<GameManager>();
                
                // 그래도 없으면 생성
                if (instance == null)
                {
                    GameObject obj = new GameObject("GameManager");
                    instance = obj.AddComponent<GameManager>();
                }
            }
            
            return instance;
        }
    }
    
    // 3. Awake에서 중복 체크
    void Awake()
    {
        // 이미 인스턴스가 존재하면
        if (instance != null && instance != this)
        {
            // 중복된 자신을 파괴
            Destroy(gameObject);
            return;
        }
        
        // 최초 생성된 객체
        instance = this;
    }
}
```

**사용 방법:**

```csharp
public class Player : MonoBehaviour
{
    void Start()
    {
        // 어디서든 GameManager 접근 가능
        GameManager.Instance.AddScore(100);
    }
}
```

---

### instance = null의 의미

````csharp
private static GameManager instance = null;
```

**설명:**
- `null`: 아직 인스턴스가 생성되지 않았음을 의미
- 최초로 생성된 객체만 `instance`에 저장됨
- 이후 생성 시도는 모두 거부됨

**동작 과정:**
```
1. 게임 시작
   instance = null (아직 생성 안됨)

2. 첫 번째 GameManager 생성
   Awake() 호출
   instance == null? Yes
   instance = this (최초 객체 저장)

3. 두 번째 GameManager 생성 시도
   Awake() 호출
   instance == null? No
   instance != this? Yes
   Destroy(gameObject) (중복 제거)
````

---

### 완전한 Singleton 구현

```csharp
public class GameManager : MonoBehaviour
{
    private static GameManager instance = null;
    
    public static GameManager Instance
    {
        get
        {
            if (instance == null)
            {
                instance = FindFirstObjectByType<GameManager>();
                
                if (instance == null)
                {
                    GameObject obj = new GameObject("GameManager");
                    instance = obj.AddComponent<GameManager>();
                }
            }
            
            return instance;
        }
    }
    
    void Awake()
    {
        if (instance != null && instance != this)
        {
            Destroy(gameObject);
            return;
        }
        
        instance = this;
        
        // Scene 전환 시에도 파괴되지 않음
        DontDestroyOnLoad(gameObject);
    }
    
    // 싱글톤 해제
    void OnDestroy()
    {
        if (instance == this)
        {
            instance = null;
        }
    }
    
    // === 게임 매니저 기능들 ===
    
    public int score = 0;
    
    public void AddScore(int amount)
    {
        score += amount;
        Debug.Log($"점수: {score}");
    }
}
```

---

### 제네릭 Singleton 베이스 클래스

```csharp
public class Singleton<T> : MonoBehaviour where T : MonoBehaviour
{
    private static T instance;
    
    public static T Instance
    {
        get
        {
            if (instance == null)
            {
                instance = FindFirstObjectByType<T>();
                
                if (instance == null)
                {
                    GameObject obj = new GameObject(typeof(T).Name);
                    instance = obj.AddComponent<T>();
                }
            }
            
            return instance;
        }
    }
    
    protected virtual void Awake()
    {
        if (instance != null && instance != this)
        {
            Destroy(gameObject);
            return;
        }
        
        instance = this as T;
        DontDestroyOnLoad(gameObject);
    }
}

// 사용 예시
public class GameManager : Singleton<GameManager>
{
    public int score = 0;
    
    public void AddScore(int amount)
    {
        score += amount;
    }
}

public class SoundManager : Singleton<SoundManager>
{
    public void PlaySound(string soundName)
    {
        Debug.Log($"재생: {soundName}");
    }
}
```

**사용:**

```csharp
GameManager.Instance.AddScore(100);
SoundManager.Instance.PlaySound("Victory");
```

---

## 4. DontDestroyOnLoad()

### 개념

**DontDestroyOnLoad()**: Scene을 전환할 때 해당 게임 오브젝트를 파괴하지 않고 유지하는 함수

---

### 기본 사용법

````csharp
public class GameManager : MonoBehaviour
{
    void Awake()
    {
        // 이 오브젝트는 Scene 전환 시에도 파괴되지 않음
        DontDestroyOnLoad(gameObject);
    }
}
```

**동작:**
```
Scene1 (MainMenu)
├─ GameManager (DontDestroyOnLoad 적용)
└─ UI Canvas

Scene2 (GamePlay) 로드
├─ GameManager (유지됨!)  <- Scene1에서 온 것
├─ Player
└─ Enemies

Scene3 (GameOver) 로드
├─ GameManager (계속 유지!)  <- 여전히 같은 객체
└─ Result UI
````

---

### Scene 전환 예시

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

public class GameManager : MonoBehaviour
{
    private static GameManager instance;
    
    public static GameManager Instance => instance;
    
    public int score = 0;
    public int lives = 3;
    
    void Awake()
    {
        if (instance != null && instance != this)
        {
            Destroy(gameObject);
            return;
        }
        
        instance = this;
        DontDestroyOnLoad(gameObject); // Scene 전환 시 유지
    }
    
    public void LoadNextScene()
    {
        int currentScene = SceneManager.GetActiveScene().buildIndex;
        SceneManager.LoadScene(currentScene + 1);
        
        // GameManager는 파괴되지 않고 다음 Scene으로 이동
    }
    
    public void RestartGame()
    {
        score = 0;
        lives = 3;
        SceneManager.LoadScene(0); // 첫 Scene으로
    }
}
```

---

### 주의사항: 중복 생성 방지

````csharp
public class PersistentObject : MonoBehaviour
{
    void Awake()
    {
        // ❌ 잘못된 방법
        DontDestroyOnLoad(gameObject);
        
        // Scene마다 이 오브젝트가 있으면
        // Scene 전환할 때마다 계속 쌓임!
    }
}
```

**문제 상황:**
```
Scene1 로드 → PersistentObject 1개 생성
Scene2 로드 → PersistentObject 1개 더 생성 (총 2개)
Scene3 로드 → PersistentObject 1개 더 생성 (총 3개!)
````

**올바른 방법:**

```csharp
public class PersistentObject : MonoBehaviour
{
    private static PersistentObject instance;
    
    void Awake()
    {
        // ✅ 중복 체크 후 DontDestroyOnLoad
        if (instance != null)
        {
            Destroy(gameObject); // 중복 제거
            return;
        }
        
        instance = this;
        DontDestroyOnLoad(gameObject);
    }
}
```

---

### 실전 예시: BGM 매니저

```csharp
public class BGMManager : MonoBehaviour
{
    private static BGMManager instance;
    public static BGMManager Instance => instance;
    
    private AudioSource audioSource;
    public AudioClip menuBGM;
    public AudioClip gameBGM;
    public AudioClip bossBGM;
    
    void Awake()
    {
        if (instance != null)
        {
            Destroy(gameObject);
            return;
        }
        
        instance = this;
        DontDestroyOnLoad(gameObject);
        
        audioSource = GetComponent<AudioSource>();
    }
    
    public void PlayBGM(AudioClip clip)
    {
        if (audioSource.clip == clip)
            return; // 같은 BGM이면 그대로
        
        audioSource.Stop();
        audioSource.clip = clip;
        audioSource.Play();
    }
    
    // Scene 전환 시 BGM 변경
    void OnEnable()
    {
        SceneManager.sceneLoaded += OnSceneLoaded;
    }
    
    void OnDisable()
    {
        SceneManager.sceneLoaded -= OnSceneLoaded;
    }
    
    void OnSceneLoaded(Scene scene, LoadSceneMode mode)
    {
        // Scene에 따라 BGM 변경
        switch (scene.name)
        {
            case "MainMenu":
                PlayBGM(menuBGM);
                break;
            case "GamePlay":
                PlayBGM(gameBGM);
                break;
            case "BossStage":
                PlayBGM(bossBGM);
                break;
        }
    }
}
```

---

## 5. 순수 C# 클래스와 Unity 연결

### 순수 C# 클래스

```csharp
// MonoBehaviour를 상속받지 않는 순수 C# 클래스
public class GameData
{
    public string playerName;
    public int level;
    public int gold;
    
    public GameData(string name, int lv, int g)
    {
        playerName = name;
        level = lv;
        gold = g;
    }
    
    public void AddGold(int amount)
    {
        gold += amount;
    }
}
```

**특징:**

- GameObject에 붙일 수 없음
- Update, Start 등 Unity 생명주기 함수 없음
- `new` 키워드로 직접 생성
- 순수한 데이터 및 로직 담당

---

### Unity와 연결하기

```csharp
using UnityEngine;

// Unity 컴포넌트
public class Player : MonoBehaviour
{
    // 순수 C# 클래스 사용
    private GameData data;
    
    void Start()
    {
        // new로 생성
        data = new GameData("플레이어1", 1, 1000);
    }
    
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            data.AddGold(100);
            Debug.Log($"골드: {data.gold}");
        }
    }
}
```

---

### Serializable로 Inspector 연동

```csharp
[System.Serializable] // Inspector에서 편집 가능하게
public class GameData
{
    public string playerName;
    public int level;
    public int gold;
}

public class Player : MonoBehaviour
{
    // Inspector에서 편집 가능!
    public GameData data;
    
    void Start()
    {
        Debug.Log($"{data.playerName}: Lv.{data.level}, {data.gold}골드");
    }
}
```

---

### 전략 패턴과 순수 C# 조합

```csharp
// 순수 C# 인터페이스
public interface IDamageCalculator
{
    int Calculate(int baseDamage, int defense);
}

// 순수 C# 구현체들
public class PhysicalDamageCalculator : IDamageCalculator
{
    public int Calculate(int baseDamage, int defense)
    {
        return Mathf.Max(0, baseDamage - defense);
    }
}

public class MagicalDamageCalculator : IDamageCalculator
{
    public int Calculate(int baseDamage, int defense)
    {
        // 마법 데미지는 방어력 50% 무시
        return Mathf.Max(0, baseDamage - (defense / 2));
    }
}

public class TrueDamageCalculator : IDamageCalculator
{
    public int Calculate(int baseDamage, int defense)
    {
        // 고정 데미지는 방어력 무시
        return baseDamage;
    }
}

// Unity 컴포넌트
public class Character : MonoBehaviour
{
    public int attack = 100;
    public int defense = 50;
    
    private IDamageCalculator damageCalculator;
    
    void Start()
    {
        // 기본은 물리 데미지
        damageCalculator = new PhysicalDamageCalculator();
    }
    
    public void SetDamageType(string type)
    {
        switch (type)
        {
            case "Physical":
                damageCalculator = new PhysicalDamageCalculator();
                break;
            case "Magical":
                damageCalculator = new MagicalDamageCalculator();
                break;
            case "True":
                damageCalculator = new TrueDamageCalculator();
                break;
        }
    }
    
    public void AttackTarget(Character target)
    {
        int damage = damageCalculator.Calculate(attack, target.defense);
        Debug.Log($"{damage} 데미지!");
    }
}
```

---

## 6. 실전 종합 예시

### 완전한 게임 매니저 시스템

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;
using System;

// 순수 C# 데이터 클래스
[System.Serializable]
public class GameData
{
    public int score;
    public int highScore;
    public int lives;
    public int stage;
    
    public void ResetGame()
    {
        score = 0;
        lives = 3;
        stage = 1;
    }
}

// Singleton 게임 매니저
public class GameManager : MonoBehaviour
{
    // Singleton
    private static GameManager instance;
    public static GameManager Instance => instance;
    
    // 게임 데이터
    public GameData data = new GameData();
    
    // 이벤트
    public event Action<int> OnScoreChanged;
    public event Action<int> OnLivesChanged;
    public event Action OnGameOver;
    
    void Awake()
    {
        // Singleton 설정
        if (instance != null && instance != this)
        {
            Destroy(gameObject);
            return;
        }
        
        instance = this;
        DontDestroyOnLoad(gameObject);
        
        // 저장된 데이터 로드
        LoadData();
    }
    
    public void AddScore(int amount)
    {
        data.score += amount;
        
        if (data.score > data.highScore)
        {
            data.highScore = data.score;
            SaveData();
        }
        
        OnScoreChanged?.Invoke(data.score);
    }
    
    public void LoseLife()
    {
        data.lives--;
        OnLivesChanged?.Invoke(data.lives);
        
        if (data.lives <= 0)
        {
            GameOver();
        }
    }
    
    void GameOver()
    {
        OnGameOver?.Invoke();
        SaveData();
        Invoke("LoadGameOverScene", 2f);
    }
    
    void LoadGameOverScene()
    {
        SceneManager.LoadScene("GameOver");
    }
    
    public void StartNewGame()
    {
        data.ResetGame();
        SceneManager.LoadScene("GamePlay");
    }
    
    public void LoadNextStage()
    {
        data.stage++;
        SceneManager.LoadScene($"Stage{data.stage}");
    }
    
    void SaveData()
    {
        PlayerPrefs.SetInt("HighScore", data.highScore);
        PlayerPrefs.Save();
    }
    
    void LoadData()
    {
        data.highScore = PlayerPrefs.GetInt("HighScore", 0);
    }
}

// UI 매니저
public class UIManager : MonoBehaviour
{
    public Text scoreText;
    public Text livesText;
    public GameObject gameOverPanel;
    
    void Start()
    {
        // 이벤트 구독
        GameManager.Instance.OnScoreChanged += UpdateScore;
        GameManager.Instance.OnLivesChanged += UpdateLives;
        GameManager.Instance.OnGameOver += ShowGameOver;
        
        // 초기 표시
        UpdateScore(GameManager.Instance.data.score);
        UpdateLives(GameManager.Instance.data.lives);
    }
    
    void OnDestroy()
    {
        // 이벤트 구독 해제
        if (GameManager.Instance != null)
        {
            GameManager.Instance.OnScoreChanged -= UpdateScore;
            GameManager.Instance.OnLivesChanged -= UpdateLives;
            GameManager.Instance.OnGameOver -= ShowGameOver;
        }
    }
    
    void UpdateScore(int score)
    {
        scoreText.text = $"Score: {score}";
    }
    
    void UpdateLives(int lives)
    {
        livesText.text = $"Lives: {lives}";
    }
    
    void ShowGameOver()
    {
        gameOverPanel.SetActive(true);
    }
}

// 플레이어
public class Player : MonoBehaviour
{
    void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Coin"))
        {
            GameManager.Instance.AddScore(10);
            Destroy(other.gameObject);
        }
        
        if (other.CompareTag("Enemy"))
        {
            GameManager.Instance.LoseLife();
        }
    }
}
```

---

## 💡 핵심 포인트

1. **System.Serializable**: 순수 C# 클래스를 Inspector에서 편집 가능하게 만듦
2. **전략 패턴**: 알고리즘을 캡슐화하여 동적으로 교체 가능 
3. **Singleton**: 게임 전체에서 하나만 존재하는 관리자 객체 
4. **instance = null**: 최초 생성 객체만 저장, 중복 방지 
5. **DontDestroyOnLoad**: Scene 전환 시에도 오브젝트 유지 
6. **순수 C#**: 데이터와 로직을 Unity와 분리하여 관리 
7. **조합**: Singleton + DontDestroyOnLoad = 영구 관리자 
8. **중복 체크**: Awake에서 반드시 중복 확인 후 파괴 
9. **이벤트 활용**: Singleton과 이벤트로 느슨한 결합 
10. **데이터 분리**: Unity 컴포넌트와 순수 데이터 클래스 분리