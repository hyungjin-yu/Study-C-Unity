#Unity #Coroutine #IEnumerator #GameObject #Lifecycle

## 📑 목차

1. 코루틴 시작 방법 3가지
2. 코루틴 중지 방법
3. GameObject와 코루틴 관계
4. yield 키워드
5. 핵심 포인트

## 📌 간단 요약

Unity 코루틴을 시작하는 3가지 방법, 중지 방법, GameObject와의 관계, 그리고 yield의 역할

---

## 1. 코루틴 시작 방법

### 방법 1: 코루틴 함수를 직접 인자로 넣기 ⭐

csharp

```csharp
IEnumerator MyCoroutine()
{
    yield return new WaitForSeconds(1f);
    Debug.Log("1초 후 실행");
}

void Start()
{
    // 가장 안전한 방법 - 실수 확률 낮음
    StartCoroutine(MyCoroutine());
}
```

**특징:**

- 컴파일 타임에 오류 검출
- 타입 안정성 보장
- 실수 확률 가장 낮음

---

### 방법 2: 코루틴 함수 이름(문자열)으로 시작

csharp

```csharp
IEnumerator MyCoroutine()
{
    yield return new WaitForSeconds(1f);
    Debug.Log("1초 후 실행");
}

void Start()
{
    // 문자열로 시작
    StartCoroutine("MyCoroutine");
}
```

#### 2-1. 리플렉션 연산 (비용이 큼)

**동작 원리:**

- 문자열로 함수 이름을 찾는 **리플렉션(Reflection)** 연산 발생
- 런타임에 메타데이터를 검색하여 함수 찾기
- 성능 비용이 상대적으로 큼

**단점:**

csharp

````csharp
// 오타가 있어도 컴파일 에러 발생 안함
StartCoroutine("MyCorutine"); // 오타! 런타임 에러
```

---

#### 2-2. 내부적인 (키-값) 컨테이너 매핑

**Unity 내부 동작:**
```
(함수 이름 문자열, 함수 주소) 매핑
"MyCoroutine" → MyCoroutine 함수의 메모리 주소
````

**장점:**

- 문자열로 중지 가능 (StopCoroutine)

---

### 방법 3: IEnumerator 변수에 담아 시작 ✅ (가장 추천)

csharp

```csharp
IEnumerator MyCoroutine()
{
    yield return new WaitForSeconds(1f);
    Debug.Log("1초 후 실행");
}

void Start()
{
    // IEnumerator 변수로 저장
    IEnumerator coroutine = MyCoroutine();
    
    // 변수를 통해 시작
    StartCoroutine(coroutine);
}
```

**장점:**

1. 타입 안정성
2. 코루틴 참조 보관 가능
3. 나중에 해당 코루틴만 정확히 중지 가능

---

### 세 가지 방법 비교

|방법|타입 안정성|성능|중지 방법|추천도|
|---|---|---|---|---|
|직접 호출|✅|빠름|참조 필요|⭐⭐⭐|
|문자열|❌|느림|문자열로 가능|⭐|
|변수 저장|✅|빠름|변수로 가능|⭐⭐⭐⭐⭐|

---

## 2. 코루틴 중지 방법

### 특정 코루틴 중지

#### 패턴 1: 변수로 중지 (추천)

csharp

```csharp
public class CoroutineExample : MonoBehaviour
{
    private Coroutine myCoroutine;
    
    void Start()
    {
        // 코루틴 참조 저장
        myCoroutine = StartCoroutine(MyCoroutine());
    }
    
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            // 저장된 참조로 중지
            if (myCoroutine != null)
            {
                StopCoroutine(myCoroutine);
                myCoroutine = null;
            }
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

#### 패턴 2: 문자열로 중지

csharp

```csharp
void Start()
{
    // 문자열로 시작
    StartCoroutine("MyCoroutine");
}

void Update()
{
    if (Input.GetKeyDown(KeyCode.Space))
    {
        // 문자열로 중지
        StopCoroutine("MyCoroutine");
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
```

> ⚠️ **중요:** 코루틴을 함수 이름(문자열)으로 멈출 때는 시작도 반드시 함수 이름(문자열)으로 해야 합니다!

**잘못된 예시:**

csharp

```csharp
// ❌ 이렇게 하면 중지 안됨!
void Start()
{
    StartCoroutine(MyCoroutine()); // 직접 호출로 시작
}

void Stop()
{
    StopCoroutine("MyCoroutine"); // 문자열로 중지 시도 - 실패!
}
```

---

### 모든 코루틴 중지: StopAllCoroutines()

csharp

```csharp
public class MultiCoroutineExample : MonoBehaviour
{
    void Start()
    {
        StartCoroutine(Coroutine1());
        StartCoroutine(Coroutine2());
        StartCoroutine(Coroutine3());
    }
    
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Escape))
        {
            // 현재 컴포넌트에서 실행 중인 모든 코루틴 중지
            StopAllCoroutines();
        }
    }
    
    IEnumerator Coroutine1()
    {
        while (true)
        {
            Debug.Log("코루틴 1");
            yield return new WaitForSeconds(1f);
        }
    }
    
    IEnumerator Coroutine2()
    {
        while (true)
        {
            Debug.Log("코루틴 2");
            yield return new WaitForSeconds(2f);
        }
    }
    
    IEnumerator Coroutine3()
    {
        while (true)
        {
            Debug.Log("코루틴 3");
            yield return new WaitForSeconds(3f);
        }
    }
}
```

**특징:**

- 현재 클래스(컴포넌트)에서 실행 중인 모든 코루틴 중지
- 다른 컴포넌트의 코루틴은 영향 없음

---

## 3. GameObject와 코루틴 관계

### 핵심 원칙

> **코루틴을 관리하는 주체는 GameObject입니다!**

---

### 시나리오 1: GameObject 비활성화

csharp

```csharp
public class GameObjectTest : MonoBehaviour
{
    void Start()
    {
        StartCoroutine(TestCoroutine());
        
        // 3초 후 GameObject 비활성화
        Invoke("DisableObject", 3f);
    }
    
    IEnumerator TestCoroutine()
    {
        for (int i = 0; i < 10; i++)
        {
            Debug.Log($"카운트: {i}");
            yield return new WaitForSeconds(1f);
        }
    }
    
    void DisableObject()
    {
        gameObject.SetActive(false);
        // 코루틴이 여기서 멈춤 (파괴됨)
    }
}
```

**결과:**

1. 코루틴 실행: 0, 1, 2 출력
2. 3초 후 GameObject 비활성화
3. **코루틴 멈춤 (파괴됨)**
4. GameObject 다시 활성화해도 **코루틴은 재시작되지 않음**

---

### 시나리오 2: 스크립트(컴포넌트) 비활성화

csharp

```csharp
public class ComponentTest : MonoBehaviour
{
    void Start()
    {
        StartCoroutine(TestCoroutine());
        
        // 3초 후 이 컴포넌트만 비활성화
        Invoke("DisableComponent", 3f);
    }
    
    IEnumerator TestCoroutine()
    {
        for (int i = 0; i < 10; i++)
        {
            Debug.Log($"카운트: {i}");
            yield return new WaitForSeconds(1f);
        }
    }
    
    void DisableComponent()
    {
        this.enabled = false;
        // 코루틴은 계속 실행됨!
    }
}
```

**결과:**

1. 코루틴 실행: 0, 1, 2 출력
2. 3초 후 컴포넌트 비활성화
3. **코루틴은 계속 실행됨!** (3, 4, 5... 계속 출력)
4. Update는 실행 안되지만 코루틴은 실행됨

---

### 시나리오 3: 다시 활성화

csharp

```csharp
public class ReactivateTest : MonoBehaviour
{
    private bool isRunning = false;
    
    void Start()
    {
        StartCoroutine(TestCoroutine());
    }
    
    IEnumerator TestCoroutine()
    {
        isRunning = true;
        
        for (int i = 0; i < 10; i++)
        {
            Debug.Log($"카운트: {i}");
            yield return new WaitForSeconds(1f);
        }
        
        isRunning = false;
    }
    
    void OnEnable()
    {
        Debug.Log($"활성화됨. 코루틴 실행 중: {isRunning}");
        
        // GameObject 재활성화 시 코루틴 재시작 필요
        if (!isRunning)
        {
            StartCoroutine(TestCoroutine());
        }
    }
}
```

---

### 관계 정리

|상황|Update()|FixedUpdate()|코루틴|
|---|---|---|---|
|GameObject 비활성화|❌ 멈춤|❌ 멈춤|❌ 멈춤 (파괴)|
|컴포넌트 비활성화|❌ 멈춤|❌ 멈춤|✅ 계속 실행|
|GameObject 재활성화|✅ 재시작|✅ 재시작|❌ 재시작 안됨|

---

### 실전 예시: 안전한 코루틴 관리

csharp

```csharp
public class SafeCoroutineManager : MonoBehaviour
{
    private Coroutine healthRegenCoroutine;
    
    void OnEnable()
    {
        // GameObject 활성화 시 코루틴 시작
        if (healthRegenCoroutine == null)
        {
            healthRegenCoroutine = StartCoroutine(RegenerateHealth());
        }
    }
    
    void OnDisable()
    {
        // GameObject 비활성화 시 명시적으로 중지
        if (healthRegenCoroutine != null)
        {
            StopCoroutine(healthRegenCoroutine);
            healthRegenCoroutine = null;
        }
    }
    
    IEnumerator RegenerateHealth()
    {
        while (true)
        {
            Debug.Log("체력 회복 중...");
            yield return new WaitForSeconds(1f);
        }
    }
}
```

---

## 4. yield 키워드

### 개념

**yield**: 반복자(Iterator)를 반환하는 문법

- 코루틴 내부가 실행되는 순서를 제어
- 실행을 일시 정지하고 다음 조건까지 대기

---

### yield return의 동작 순서

csharp

````csharp
IEnumerator UnderstandYield()
{
    Debug.Log("1. 코루틴 시작");
    
    yield return null; // 여기서 일시 정지, 다음 프레임에 재개
    
    Debug.Log("2. 다음 프레임 실행");
    
    yield return new WaitForSeconds(2f); // 2초 대기
    
    Debug.Log("3. 2초 후 실행");
    
    yield return new WaitForSeconds(1f); // 1초 대기
    
    Debug.Log("4. 1초 후 실행 (총 3초 후)");
}
```

**실행 순서:**
```
프레임 1: "1. 코루틴 시작" 출력 → yield return null
프레임 2: "2. 다음 프레임 실행" 출력 → yield return new WaitForSeconds(2f)
2초 대기...
프레임 N: "3. 2초 후 실행" 출력 → yield return new WaitForSeconds(1f)
1초 대기...
프레임 N+M: "4. 1초 후 실행 (총 3초 후)" 출력
````

---

### yield return의 다양한 사용

csharp

```csharp
IEnumerator YieldExamples()
{
    // 1. 다음 프레임까지 대기
    yield return null;
    
    // 2. 특정 시간 대기
    yield return new WaitForSeconds(1f);
    
    // 3. FixedUpdate 타이밍까지 대기
    yield return new WaitForFixedUpdate();
    
    // 4. 프레임 끝까지 대기
    yield return new WaitForEndOfFrame();
    
    // 5. 조건이 참이 될 때까지 대기
    yield return new WaitUntil(() => Input.GetKeyDown(KeyCode.Space));
    
    // 6. 조건이 거짓이 될 때까지 대기
    bool isMoving = true;
    yield return new WaitWhile(() => isMoving);
    
    // 7. 다른 코루틴이 끝날 때까지 대기
    yield return StartCoroutine(OtherCoroutine());
}

IEnumerator OtherCoroutine()
{
    Debug.Log("다른 코루틴 시작");
    yield return new WaitForSeconds(2f);
    Debug.Log("다른 코루틴 끝");
}
```

---

### yield의 내부 동작 이해

csharp

```csharp
IEnumerator CountdownExample()
{
    for (int i = 5; i > 0; i--)
    {
        Debug.Log($"카운트다운: {i}");
        yield return new WaitForSeconds(1f);
        // 여기서 함수가 멈췄다가, 1초 후 다시 for문의 다음 반복으로 돌아옴
    }
    
    Debug.Log("발사!");
}
```

**동작 순서:**

1. i = 5, "카운트다운: 5" 출력
2. `yield return new WaitForSeconds(1f)` → **함수 일시 정지**
3. 1초 대기...
4. **함수 재개**, for문 계속
5. i = 4, "카운트다운: 4" 출력
6. 반복...

---

### yield break (코루틴 조기 종료)

csharp

```csharp
IEnumerator ConditionalCoroutine()
{
    for (int i = 0; i < 10; i++)
    {
        if (i == 5)
        {
            Debug.Log("조건 충족, 코루틴 종료");
            yield break; // 코루틴 즉시 종료
        }
        
        Debug.Log($"카운트: {i}");
        yield return new WaitForSeconds(1f);
    }
    
    Debug.Log("이 줄은 실행되지 않음");
}
```

---

## 💡 핵심 포인트

1. **시작 방법**: IEnumerator 변수 저장 방식이 가장 추천
2. **문자열 시작/중지**: 반드시 쌍으로 사용해야 함
3. **GameObject 비활성화**: 코루틴 멈춤 및 파괴
4. **컴포넌트 비활성화**: 코루틴은 계속 실행
5. **관리 주체**: GameObject가 코루틴 관리
6. **재활성화**: 코루틴 자동 재시작 안됨, 수동으로 다시 시작해야 함
7. **yield**: 코루틴 실행 순서를 제어하는 반복자
8. **StopAllCoroutines**: 현재 컴포넌트의 모든 코루틴 중지
9. **성능**: 문자열 방식은 리플렉션으로 인해 비용 큼
10. **안전성**: 타입 안정성이 있는 방법 선택 권장