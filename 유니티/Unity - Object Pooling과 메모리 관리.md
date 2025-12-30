### Object Pooling

`#메모리최적화 #GC #메모리단편화`

Object Pooling은 객체를 미리 여러 개 생성해두고 활성화/비활성화를 반복하여 메모리 할당과 해제의 빈도를 줄이는 최적화 기법입니다.

### 핵심 개념

**Destroy와 GC의 시간 차이**

- `Destroy()` 호출 후 즉시 삭제되지 않음
- `OnDestroy`는 파괴 예약 상태일 뿐, 실제 메모리 해제는 GC가 수행
- 메모리 해제 시점과 OnDestroy 호출 시점이 다를 수 있음

**유니티의 보헴 GC 특성**

- 비세대(Non-Generational), 비압축(Non-Compacting) 방식
- 메모리 단편화 발생 → Object Pooling의 필요성 증대

**메모리 단편화(Fragmentation)**

```
메모리 상태: [사용중] [빈공간] [사용중] [빈공간] [사용중]
문제: 충분한 공간이 있어도 연속적인 공간이 없어 할당 불가
```

### Object Pooling 구현 시나리오

```csharp
public class ObjectPool : MonoBehaviour
{
    private Queue<GameObject> pool = new Queue<GameObject>();
    public GameObject prefab;
    public int poolSize = 10;

    void Start()
    {
        for (int i = 0; i < poolSize; i++)
        {
            GameObject obj = Instantiate(prefab);
            obj.SetActive(false);
            pool.Enqueue(obj);
        }
    }

    public GameObject GetObject()
    {
        if (pool.Count > 0)
        {
            GameObject obj = pool.Dequeue();
            obj.SetActive(true);
            return obj;
        }
        return Instantiate(prefab);
    }

    public void ReturnObject(GameObject obj)
    {
        obj.SetActive(false);
        pool.Enqueue(obj);
    }
}
```

Pool에서 객체를 꺼낼 때 활성화되므로 `OnEnable()`에서 초기화해야 합니다.

### 주요 포인트

|단계|설명|
|---|---|
|Pool 생성|사용할 객체를 여러 개 미리 생성|
|비활성화|SetActive(false)로 Pool에 저장|
|사용|Pool에서 꺼내어 SetActive(true)|
|반환|사용 완료 후 다시 비활성화 하여 Pool 반환|

---

## Dictionary

`#자료구조 #키값쌍`

Dictionary는 키(Key)와 값(Value)이 1:1로 대응되는 자료구조로, 빠른 검색과 동적 데이터 관리에 유용합니다.

**사용 문법**

````csharp
Dictionary<string, int> scores = new Dictionary<string, int>();
scores["Player1"] = 100;
scores["Player2"] = 85;

if (scores.ContainsKey("Player1"))
{
    scores["Player1"] = 120;
}
```

키를 이용한 O(1) 시간에 값 접근 가능합니다.

---

## 마킹 앤 스윕 (Mark and Sweep)
`#GC알고리즘 #보헴GC`

마킹 앤 스윕은 유니티의 보헴 GC가 사용하는 메모리 관리 알고리즘입니다.
```
[1단계: Marking] 
루트에서 도달 가능한 모든 객체 표시
  ↓
[2단계: Sweeping]
표시되지 않은 객체의 메모리 해제
````

비압축 방식이므로 메모리 단편화가 발생하며, 이를 해결하기 위해 Object Pooling이 중요합니다.

---

## 🎯 핵심 요약

|항목|내용|
|---|---|
|**Object Pooling**|객체 재사용으로 메모리 할당/해제 빈도 감소|
|**메모리 단편화**|비압축 GC로 인한 메모리 비효율 문제|
|**초기화 위치**|`Start()` 대신 `OnEnable()`에서 초기화|
|**Dictionary**|키-값 쌍으로 빠른 검색(O(1))|
|**GC 알고리즘**|Mark and Sweep으로 사용 객체만 표시 후 나머지 해제|

**결론**: 자주 생성/삭제되는 객체(총알, 이펙트, UI 등)는 Object Pooling으로 관리하여 메모리 단편화를 방지하고 게임 성능을 최적화하세요.