#CSharp #Delegate #Action #Func #Event #Lambda #Callback

## 📑 목차

1. Delegate 기본 개념
2. Delegate 체이닝
3. 콜백(Callback)
4. Lambda 표현식
5. Action
6. Func
7. Event
8. UnityEvent
9. 실전 예시
10. 핵심 포인트

## 📌 간단 요약

C# Delegate의 개념부터 Action, Func, Event까지 함수를 객체처럼 다루는 방법과 Unity에서의 활용법

---

## 1. Delegate 기본 개념

### 정의

**Delegate**: 함수도 변수처럼 담아서 쓸 수 있게 해주는 문법

```csharp
// Delegate 선언
delegate 리턴타입 Delegate명(매개변수);
```

**핵심:**

- Delegate는 객체
- 함수의 참조를 저장
- 함수 포인터와 유사하지만 타입 안전성 보장

---

### 기본 사용법

```csharp
// 1. Delegate 선언
public delegate void MyDelegate(string message);

// 2. Delegate와 매칭되는 함수 작성
public void PrintMessage(string message)
{
    Debug.Log(message);
}

public void ShowMessage(string message)
{
    Debug.Log($"[알림] {message}");
}

// 3. 사용
void Start()
{
    // Delegate 변수 생성
    MyDelegate del;
    
    // 함수 할당
    del = PrintMessage;
    
    // 함수 호출
    del("안녕하세요"); // "안녕하세요" 출력
    
    // 다른 함수로 교체
    del = ShowMessage;
    del("반갑습니다"); // "[알림] 반갑습니다" 출력
}
```

---

### 반환값이 있는 Delegate

```csharp
// int를 반환하는 Delegate
public delegate int Calculator(int a, int b);

public int Add(int a, int b)
{
    return a + b;
}

public int Multiply(int a, int b)
{
    return a * b;
}

void Start()
{
    Calculator calc = Add;
    int result = calc(5, 3); // 8
    Debug.Log(result);
    
    calc = Multiply;
    result = calc(5, 3); // 15
    Debug.Log(result);
}
```

---

## 2. Delegate 체이닝

### 개념

**체이닝**: Delegate에 여러 함수를 이어붙이는 것 (Multicast Delegate)

```csharp
public delegate void GameEvent();

public void SaveGame()
{
    Debug.Log("게임 저장");
}

public void ShowNotification()
{
    Debug.Log("알림 표시");
}

public void PlaySound()
{
    Debug.Log("사운드 재생");
}

void Start()
{
    GameEvent onGameEnd = SaveGame;
    
    // += 로 함수 추가 (체이닝)
    onGameEnd += ShowNotification;
    onGameEnd += PlaySound;
    
    // 한 번 호출하면 등록된 모든 함수 실행
    onGameEnd();
    
    // 출력:
    // "게임 저장"
    // "알림 표시"
    // "사운드 재생"
}
```

---

### 체이닝 관리

```csharp
public delegate void OnDamage(int damage);

void Start()
{
    OnDamage damageHandler = null;
    
    // 함수 추가
    damageHandler += TakeDamage;
    damageHandler += UpdateHealthUI;
    damageHandler += PlayHitSound;
    
    // 모든 함수 실행
    damageHandler(10);
    
    // 특정 함수 제거
    damageHandler -= PlayHitSound;
    
    // 남은 함수들만 실행
    damageHandler(5);
}

void TakeDamage(int damage)
{
    Debug.Log($"데미지: {damage}");
}

void UpdateHealthUI(int damage)
{
    Debug.Log("UI 업데이트");
}

void PlayHitSound(int damage)
{
    Debug.Log("사운드 재생");
}
```

---

### Invoke를 통한 브로드캐스팅

**Invoke**: Delegate에 등록된 모든 함수를 한 번에 호출 (브로드캐스팅)

```csharp
public delegate void BroadcastEvent();

public class EventBroadcaster : MonoBehaviour
{
    public BroadcastEvent onLevelComplete;
    
    void Start()
    {
        // 여러 리스너 등록
        onLevelComplete += ShowVictoryScreen;
        onLevelComplete += GiveReward;
        onLevelComplete += UnlockNextLevel;
        onLevelComplete += PlayVictoryMusic;
    }
    
    public void CompleteLe()
    {
        // Invoke로 모든 함수 브로드캐스팅
        onLevelComplete?.Invoke();
        
        // 또는 그냥 호출해도 동일
        // onLevelComplete();
    }
    
    void ShowVictoryScreen() { Debug.Log("승리 화면"); }
    void GiveReward() { Debug.Log("보상 지급"); }
    void UnlockNextLevel() { Debug.Log("다음 레벨 해금"); }
    void PlayVictoryMusic() { Debug.Log("승리 음악"); }
}
```

---

## 3. 콜백 (Callback)

### 개념

**콜백**: 함수를 인자로 넣는 것

```csharp
public delegate void Callback();

public void DoSomethingAsync(Callback onComplete)
{
    Debug.Log("작업 시작");
    
    // 작업 완료 후 콜백 호출
    onComplete();
}

void Start()
{
    // 함수를 인자로 전달
    DoSomethingAsync(OnTaskComplete);
}

void OnTaskComplete()
{
    Debug.Log("작업 완료!");
}
```

---

### 실전 예시: 비동기 작업

```csharp
public delegate void DownloadCallback(bool success, string data);

public class Downloader : MonoBehaviour
{
    public void DownloadFile(string url, DownloadCallback callback)
    {
        StartCoroutine(DownloadCoroutine(url, callback));
    }
    
    IEnumerator DownloadCoroutine(string url, DownloadCallback callback)
    {
        Debug.Log($"다운로드 시작: {url}");
        
        // 다운로드 시뮬레이션
        yield return new WaitForSeconds(2f);
        
        // 성공
        bool success = true;
        string data = "다운로드된 데이터";
        
        // 콜백 호출
        callback(success, data);
    }
}

// 사용
void Start()
{
    Downloader downloader = GetComponent<Downloader>();
    
    // 콜백 함수 전달
    downloader.DownloadFile("http://example.com/data", OnDownloadComplete);
}

void OnDownloadComplete(bool success, string data)
{
    if (success)
    {
        Debug.Log($"다운로드 성공: {data}");
    }
    else
    {
        Debug.Log("다운로드 실패");
    }
}
```

---

## 4. Lambda 표현식

### 개념

**Lambda**: 이름이 없는 함수(무명 메서드)를 만드는 것

**문법:**

```csharp
(매개변수) => { 실행할 것 };
```

---

### 기본 사용법

```csharp
public delegate void SimpleDelegate();

void Start()
{
    // 일반 함수
    SimpleDelegate del1 = NormalFunction;
    del1();
    
    // Lambda 표현식
    SimpleDelegate del2 = () => { Debug.Log("Lambda!"); };
    del2();
    
    // 한 줄이면 중괄호 생략 가능
    SimpleDelegate del3 = () => Debug.Log("간단한 Lambda");
    del3();
}

void NormalFunction()
{
    Debug.Log("일반 함수");
}
```

---

### 매개변수가 있는 Lambda

```csharp
public delegate void IntDelegate(int value);

void Start()
{
    IntDelegate del = (x) => Debug.Log($"값: {x}");
    del(10); // "값: 10"
    
    // 매개변수가 하나면 괄호 생략 가능
    IntDelegate del2 = x => Debug.Log($"값: {x}");
    del2(20);
}
```

---

### 여러 줄 Lambda

```csharp
public delegate int MathOperation(int a, int b);

void Start()
{
    MathOperation multiply = (a, b) => 
    {
        Debug.Log($"계산 중: {a} * {b}");
        int result = a * b;
        Debug.Log($"결과: {result}");
        return result;
    };
    
    int answer = multiply(5, 3); // 15
}
```

---

### ⚠️ Lambda 사용 권장하지 않는 이유

**문제점:**

1. **찾을 수 없음**: 디버깅 시 함수 이름이 없어서 추적 어려움
2. **제거 불가**: Lambda는 Delegate 방식으로 뺄 수 없음

```csharp
public delegate void MyDelegate();

void Start()
{
    MyDelegate del = () => Debug.Log("Lambda");
    
    // ❌ 제거 불가능!
    del -= () => Debug.Log("Lambda"); // 다른 인스턴스로 인식됨
    
    // ✅ 일반 함수는 제거 가능
    MyDelegate del2 = NormalFunction;
    del2 -= NormalFunction; // 제거 성공
}

void NormalFunction()
{
    Debug.Log("일반 함수");
}
```

**올바른 Lambda 사용 (변수에 저장):**

```csharp
void Start()
{
    // Lambda를 변수에 저장하면 제거 가능
    Action lambda = () => Debug.Log("Lambda");
    
    MyDelegate del = lambda;
    del += lambda;
    
    // 제거 가능
    del -= lambda; // 성공!
}
```

---

## 5. Action

### 개념

**Action**: Delegate를 압축해서 사용할 수 있는 내장 타입

**특징:**

- 리턴 타입은 **void만 가능**
- 매개변수 0~16개까지 지원
- Delegate 선언 없이 바로 사용 가능

---

### 매개변수 없는 Action

```csharp
// ❌ 기존 방식 (Delegate 선언 필요)
public delegate void MyDelegate();
MyDelegate del = SomeFunction;

// ✅ Action 사용 (선언 불필요)
Action action = SomeFunction;

void SomeFunction()
{
    Debug.Log("실행!");
}
```

---

### 매개변수 있는 Action<T>
```csharp
// Action<매개변수 타입>

// 매개변수 1개
Action<int> printInt = (x) => Debug.Log(x);
printInt(10); // "10"

// 매개변수 2개
Action<string, int> printInfo = (name, age) => 
{
    Debug.Log($"{name}은 {age}살입니다");
};
printInfo("철수", 20); // "철수은 20살입니다"

// 매개변수 3개
Action<int, int, int> printSum = (a, b, c) => 
{
    Debug.Log($"합계: {a + b + c}");
};
printSum(1, 2, 3); // "합계: 6"
```

---

### Action 최대 16개 매개변수
```csharp
// 최대 16개까지 가능
Action<int, int, int, int, int, int, int, int,
       int, int, int, int, int, int, int, int> action16;

// 실용적인 예시 (3-4개 정도가 적당)
Action<string, int, float, bool> gameEvent = (name, score, time, isWin) =>
{
    Debug.Log($"{name}: {score}점, {time}초, 승리: {isWin}");
};

gameEvent("플레이어", 1000, 120.5f, true);
```

---

### Action 실전 예시
```csharp
public class EventManager : MonoBehaviour
{
    // 매개변수 없는 이벤트
    public Action OnGameStart;
    public Action OnGameEnd;
    
    // 매개변수 있는 이벤트
    public Action<int> OnScoreChanged;
    public Action<int, int> OnHealthChanged; // (현재HP, 최대HP)
    public Action<string> OnMessageReceived;
    
    public void StartGame()
    {
        OnGameStart?.Invoke();
    }
    
    public void EndGame()
    {
        OnGameEnd?.Invoke();
    }
    
    public void AddScore(int amount)
    {
        OnScoreChanged?.Invoke(amount);
    }
    
    public void UpdateHealth(int current, int max)
    {
        OnHealthChanged?.Invoke(current, max);
    }
}

// 사용
public class GameUI : MonoBehaviour
{
    void Start()
    {
        EventManager em = EventManager.Instance;
        
        // 이벤트 구독
        em.OnGameStart += () => Debug.Log("게임 시작!");
        em.OnScoreChanged += (score) => Debug.Log($"점수: {score}");
        em.OnHealthChanged += (cur, max) => Debug.Log($"HP: {cur}/{max}");
    }
}
```

---

## 6. Func

### 개념

**Func**: 리턴 타입이 **void가 아닌** Delegate용 내장 타입

**문법:**
```csharp
Func<매개변수1, 매개변수2, ..., 리턴타입>
```

**주의:** 마지막 타입이 항상 리턴 타입!

---

### Delegate vs Func 비교
```csharp
// ❌ 기존 방식
public delegate int MyDelegate(float value);
MyDelegate del = SomeFunction;

// ✅ Func 사용
Func<float, int> func = SomeFunction;

int SomeFunction(float value)
{
    return (int)value;
}
```

---

### Func 기본 사용법
```csharp
// 매개변수 없이 int 반환
Func<int> getNumber = () => 42;
int num = getNumber(); // 42

// float 받아서 int 반환
Func<float, int> floatToInt = (f) => (int)f;
int result = floatToInt(3.14f); // 3

// 두 개 받아서 하나 반환
Func<int, int, int> add = (a, b) => a + b;
int sum = add(5, 3); // 8

// string 두 개 받아서 bool 반환
Func<string, string, bool> isEqual = (a, b) => a == b;
bool equal = isEqual("hello", "world"); // false
```

---

### Func 실전 예시
```csharp
public class Calculator : MonoBehaviour
{
    // 계산 함수들을 Func로 저장
    private Func<int, int, int> operation;
    
    void Start()
    {
        // 더하기로 설정
        SetOperation("add");
        Debug.Log(Calculate(5, 3)); // 8
        
        // 곱하기로 변경
        SetOperation("multiply");
        Debug.Log(Calculate(5, 3)); // 15
    }
    
    public void SetOperation(string type)
    {
        switch (type)
        {
            case "add":
                operation = (a, b) => a + b;
                break;
            case "subtract":
                operation = (a, b) => a - b;
                break;
            case "multiply":
                operation = (a, b) => a * b;
                break;
            case "divide":
                operation = (a, b) => b != 0 ? a / b : 0;
                break;
        }
    }
    
    public int Calculate(int a, int b)
    {
        return operation(a, b);
    }
}
```

---

### Func를 사용한 조건 검사
```csharp
public class ItemFilter : MonoBehaviour
{
    public List<Item> items;
    
    // Func로 조건 전달
    public List<Item> FilterItems(Func<Item, bool> condition)
    {
        List<Item> result = new List<Item>();
        
        foreach (Item item in items)
        {
            if (condition(item))
            {
                result.Add(item);
            }
        }
        
        return result;
    }
    
    void Start()
    {
        // 희귀 아이템만 필터링
        List<Item> rareItems = FilterItems(item => item.rarity == Rarity.Rare);
        
        // 가격이 100 이상인 아이템
        List<Item> expensiveItems = FilterItems(item => item.price >= 100);
        
        // 레벨 10 이상 장비
        List<Item> highLevelItems = FilterItems(item => item.level >= 10);
    }
}
```

---

## 7. Event

### 개념

**event**: Delegate를 수식해주는 키워드

**사용 가능:**

- event Action
- event Delegate
- event Func

---

### event의 제약사항

**event가 붙으면:**

1. 외부에서 수정 불가능 (= 할당 불가)
2. 외부에서 직접 호출 불가능
3. **추가(+=)와 해제(-=)만 가능**
```csharp
public class EventExample : MonoBehaviour
{
    // 일반 Action
    public Action normalAction;
    
    // event Action
    public event Action eventAction;
    
    void TestFromInside()
    {
        // 내부에서는 둘 다 가능
        normalAction = SomeFunction;
        normalAction();
        
        eventAction = SomeFunction;
        eventAction();
    }
}

public class ExternalClass : MonoBehaviour
{
    void TestFromOutside()
    {
        EventExample ex = GetComponent<EventExample>();
        
        // ✅ 일반 Action - 모두 가능
        ex.normalAction = SomeFunction;  // 할당 가능
        ex.normalAction();               // 호출 가능
        ex.normalAction += SomeFunction; // 추가 가능
        ex.normalAction -= SomeFunction; // 제거 가능
        
        // ⚠️ event Action - 제한됨
        // ex.eventAction = SomeFunction;  // ❌ 할당 불가
        // ex.eventAction();                // ❌ 호출 불가
        ex.eventAction += SomeFunction;  // ✅ 추가만 가능
        ex.eventAction -= SomeFunction;  // ✅ 제거만 가능
    }
    
    void SomeFunction() { }
}
```

---

### event를 사용하는 이유

**캡슐화와 안전성:**
```csharp
public class DangerousManager : MonoBehaviour
{
    // ❌ event 없는 경우 - 위험!
    public Action OnGameOver;
    
    public void GameOver()
    {
        OnGameOver?.Invoke();
    }
}

public class BadUser : MonoBehaviour
{
    void Start()
    {
        DangerousManager dm = GetComponent<DangerousManager>();
        
        // 😱 외부에서 모든 이벤트를 제거할 수 있음!
        dm.OnGameOver = null;
        
        // 😱 다른 사람의 리스너를 덮어씌울 수 있음!
        dm.OnGameOver = MyFunction;
        
        // 😱 외부에서 임의로 발동시킬 수 있음!
        dm.OnGameOver();
    }
    
    void MyFunction() { }
}
```

**event로 보호:**
```csharp
public class SafeManager : MonoBehaviour
{
    // ✅ event로 보호
    public event Action OnGameOver;
    
    public void GameOver()
    {
        OnGameOver?.Invoke(); // 내부에서만 호출 가능
    }
}

public class GoodUser : MonoBehaviour
{
    void Start()
    {
        SafeManager sm = GetComponent<SafeManager>();
        
        // ✅ 추가만 가능
        sm.OnGameOver += MyFunction;
        
        // ❌ 불가능한 동작들
        // sm.OnGameOver = null;     // 컴파일 에러
        // sm.OnGameOver = MyFunction; // 컴파일 에러
        // sm.OnGameOver();           // 컴파일 에러
    }
    
    void MyFunction() { }
}
```

---

### event 실전 예시
```csharp
public class Player : MonoBehaviour
{
    // 이벤트 선언
    public event Action OnDeath;
    public event Action<int> OnDamaged;
    public event Action<int> OnHealed;
    
    private int hp = 100;
    
    public void TakeDamage(int damage)
    {
        hp -= damage;
        
        // 이벤트 발동 (내부에서만 가능)
        OnDamaged?.Invoke(damage);
        
        if (hp <= 0)
        {
            hp = 0;
            OnDeath?.Invoke();
        }
    }
    
    public void Heal(int amount)
    {
        hp += amount;
        OnHealed?.Invoke(amount);
    }
}

// 이벤트 구독자들
public class UIManager : MonoBehaviour
{
    void Start()
    {
        Player player = FindFirstObjectByType<Player>();
        
        // 이벤트 구독
        player.OnDeath += ShowGameOverScreen;
        player.OnDamaged += UpdateHealthBar;
        player.OnHealed += ShowHealEffect;
    }
    
    void ShowGameOverScreen()
    {
        Debug.Log("게임 오버 화면 표시");
    }
    
    void UpdateHealthBar(int damage)
    {
        Debug.Log($"체력바 업데이트: -{damage}");
    }
    
    void ShowHealEffect(int amount)
    {
        Debug.Log($"힐 이펙트: +{amount}");
    }
}

public class SoundManager : MonoBehaviour
{
    void Start()
    {
        Player player = FindFirstObjectByType<Player>();
        
        // 같은 이벤트에 여러 구독자 가능
        player.OnDeath += PlayDeathSound;
        player.OnDamaged += PlayHitSound;
    }
    
    void PlayDeathSound()
    {
        Debug.Log("사망 사운드 재생");
    }
    
    void PlayHitSound(int damage)
    {
        Debug.Log("피격 사운드 재생");
    }
}
```

---

## 8. UnityEvent

### UnityEvent는 권장하지 않음

**이유:**

1. 성능이 Action/event보다 느림
2. Inspector 연결 방식이 디버깅 어려움
3. 코드 추적이 어려움
4. 타입 안정성 낮음

---

### UnityEvent vs event Action 비교
```csharp
using UnityEngine.Events;

public class ComparisonExample : MonoBehaviour
{
    // ❌ UnityEvent (권장 안함)
    public UnityEvent onClickUnityEvent;
    
    // ✅ event Action (권장)
    public event Action OnClickAction;
    
    public void TriggerEvents()
    {
        // UnityEvent
        onClickUnityEvent?.Invoke();
        
        // event Action
        OnClickAction?.Invoke();
    }
}
```

**문제점:**
```csharp
// UnityEvent는 Inspector에서 연결하면
// 어떤 함수가 연결되어 있는지 코드로 확인 어려움
public UnityEvent onSomething;

// event Action은 코드로 명확히 확인 가능
public event Action OnSomething;

void Start()
{
    OnSomething += Function1;
    OnSomething += Function2;
    // 어떤 함수들이 연결되어 있는지 명확
}
```

---

## 9. 실전 종합 예시

### 게임 이벤트 시스템
```csharp
public class GameEventSystem : MonoBehaviour
{
    // Singleton
    public static GameEventSystem Instance { get; private set; }
    
    // 게임 이벤트들 (event로 보호)
    public event Action OnGameStart;
    public event Action OnGamePause;
    public event Action OnGameResume;
    public event Action OnGameOver;
    
    public event Action<int> OnScoreChanged;
    public event Action<int, int> OnHealthChanged;
    public event Action<string> OnItemCollected;
    public event Action<int> OnLevelUp;
    
    void Awake()
    {
        if (Instance == null)
        {
            Instance = this;
            DontDestroyOnLoad(gameObject);
        }
        else
        {
            Destroy(gameObject);
        }
    }
    
    // 이벤트 발동 함수들 (내부에서만)
    public void StartGame()
    {
        Debug.Log("[GameEvent] 게임 시작");
        OnGameStart?.Invoke();
    }
    
    public void PauseGame()
    {
        Debug.Log("[GameEvent] 게임 일시정지");
        Time.timeScale = 0;
        OnGamePause?.Invoke();
    }
    
    public void ResumeGame()
    {
        Debug.Log("[GameEvent] 게임 재개");
        Time.timeScale = 1;
        OnGameResume?.Invoke();
    }
    
    public void GameOver()
    {
        Debug.Log("[GameEvent] 게임 오버");
        OnGameOver?.Invoke();
    }
    
    public void ChangeScore(int newScore)
    {
        OnScoreChanged?.Invoke(newScore);
    }
    
    public void ChangeHealth(int current, int max)
    {
        OnHealthChanged?.Invoke(current, max);
    }
    
    public void CollectItem(string itemName)
    {
        OnItemCollected?.Invoke(itemName);
    }
    
    public void LevelUp(int newLevel)
    {
        OnLevelUp?.Invoke(newLevel);
    }
}
```

---

### 구독자 클래스들
```csharp
// UI 매니저
public class UIManager : MonoBehaviour
{
    public TextMeshProUGUI scoreText;
    public Slider healthBar;
    public GameObject pausePanel;
    public GameObject gameOverPanel;
    
    void OnEnable()
    {
        var events = GameEventSystem.Instance;
        
        // 이벤트 구독
        events.OnGameStart += HandleGameStart;
        events.OnGamePause += HandleGamePause;
        events.OnGameResume += HandleGameResume;
        events.OnGameOver += HandleGameOver;
        events.OnScoreChanged += UpdateScore;
        events.OnHealthChanged += UpdateHealthBar;
        events.OnItemCollected += ShowItemNotification;
        events.OnLevelUp += ShowLevelUpEffect;
    }
    
    void OnDisable()
    {
        var events = GameEventSystem.Instance;
        if (events == null) return;
        
        // 이벤트 구독 해제
        events.OnGameStart -= HandleGameStart;
        events.OnGamePause -= HandleGamePause;
        events.OnGameResume -= HandleGameResume;
        events.OnGameOver -= HandleGameOver;
        events.OnScoreChanged -= UpdateScore;
        events.OnHealthChanged -= UpdateHealthBar;
        events.OnItemCollected -= ShowItemNotification;
        events.OnLevelUp -= ShowLevelUpEffect;
    }
    
    void HandleGameStart()
    {
        pausePanel.SetActive(false);
        gameOverPanel.SetActive(false);
    }
    
    void HandleGamePause()
    {
        pausePanel.SetActive(true);
    }
    
    void HandleGameResume()
    {
        pausePanel.SetActive(false);
    }
    
    void HandleGameOver()
    {
        gameOverPanel.SetActive(true);
    }
    
    void UpdateScore(int score)
    {
        scoreText.text = $"Score: {score}";
    }
    
    void UpdateHealthBar(int current, int max)
    {
        healthBar.value = (float)current / max;
    }
    
    void ShowItemNotification(string itemName)
    {
        Debug.Log($"아이템 획득: {itemName}");
    }
    
    void ShowLevelUpEffect(int level)
    {
        Debug.Log($"레벨업! Lv.{level}");
    }
}
```
### Func를 활용한 전략 패턴
```csharp
public class DamageCalculator : MonoBehaviour
{
    // Func로 데미지 계산 전략 저장
    private Func<int, int, int> damageFormula;
    
    void Start()
    {
        // 기본 전략: 물리 데미지
        SetDamageType("physical");
    }
    
    public void SetDamageType(string type)
    {
        switch (type)
        {
            case "physical":
                // 공격력 - 방어력
                damageFormula = (attack, defense) => 
                    Mathf.Max(0, attack - defense);
                break;
                
            case "magical":
                // 방어력 50% 무시
                damageFormula = (attack, defense) => 
                    Mathf.Max(0, attack - (defense / 2));
                break;
                
            case "true":
                // 방어력 완전 무시
                damageFormula = (attack, defense) => attack;
                break;
                
            case "percentage":
                // 공격력의 일정 비율
                damageFormula = (attack, defense) => 
                    (int)(attack * 0.1f);
                break;
        }
    }
    
    public int Calculate(int attack, int defense)
    {
        return damageFormula(attack, defense);
    }
}

// 사용
void Start()
{
    DamageCalculator calc = GetComponent<DamageCalculator>();
    
    calc.SetDamageType("physical");
    Debug.Log(calc.Calculate(100, 30)); // 70
    
    calc.SetDamageType("magical");
    Debug.Log(calc.Calculate(100, 30)); // 85
    
    calc.SetDamageType("true");
    Debug.Log(calc.Calculate(100, 30)); // 100
}
```

---

## 💡 핵심 포인트

1. **Delegate**: 함수를 변수처럼 저장하는 객체
2. **체이닝**: += 로 여러 함수 연결 가능
3. **콜백**: 함수를 인자로 전달
4. **Invoke**: 등록된 모든 함수 브로드캐스팅
5. **Lambda**: 무명 함수, 디버깅 어려워서 권장 안함
6. **Action**: void 반환, 매개변수 0~16개
7. **Func**: void 아닌 반환값, 마지막이 리턴 타입
8. **event**: += -= 만 가능, 외부 수정/호출 차단
9. **UnityEvent**: 성능과 디버깅 이유로 권장 안함
10. **구독 해제**: OnDisable에서 -= 로 메모리 누수 방지