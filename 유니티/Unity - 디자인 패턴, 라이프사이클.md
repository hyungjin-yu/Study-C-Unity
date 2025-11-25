#Unity #CSharp #DesignPattern #Lifecycle #GameDevelopment

## 📌 간단 요약
Unity에서 사용되는 핵심 개념: Time.deltaTime을 활용한 프레임 독립적 움직임, 프로토타입/템플릿 메서드 패턴, 그리고 Unity 오브젝트의 생명주기 관리

## 1. Time.deltaTime
- **정의**: 이전 프레임과 현재 프레임 사이의 시간(초 단위)
- **용도**: 프레임 속도에 관계없이 일정한 움직임 구현

## 2. 프로토타입 패턴 (Prototype Pattern)
- **개념**: 기존 객체를 복제하여 새 객체 생성
- **구현**: `ICloneable` 인터페이스 사용
- **예시**: 하스스톤의 '배후자' 카드처럼 복제 생성

## 3. 템플릿 메서드 패턴 (Template Method Pattern)
- **개념**: 부모 클래스에서 알고리즘 구조 정의, 자식 클래스에서 세부 구현
- **핵심**: Hook 메서드를 통한 유연한 확장

### 예시 코드
```csharp
abstract class Item
{
    protected abstract void EffectItem(); // Hook 메서드
    
    public void Use()
    {
        ActiveItemUseEffect();
        ActiveItemUseSound();
        EffectItem(); // 자식 클래스가 구현
        Debug.Log("아이템 사용 후 처리");
    }
    
    private void ActiveItemUseEffect()
    {
        Debug.Log("아이템 사용 이펙트 출력");
    }
    
    private void ActiveItemUseSound()
    {
        Debug.Log("아이템 사용 사운드 출력");
    }
}

class Potion : Item
{
    protected override void EffectItem()
    {
        Debug.Log("포션 사용");
    }
}

// 사용 예시
void Start()
{
    Item item = new Potion();
    item.Use();
}
```

## 4. Unity 라이프사이클 (Lifecycle)

### 실행 순서
```
Awake → OnEnable → Start → [Update Loop] → OnDisable → OnDestroy
```

### 메서드 상세

| 메서드 | 호출 시점 | 용도 |
|--------|-----------|------|
| `Awake()` | 오브젝트 생성 시 | 초기화 (Start보다 먼저 실행) |
| `OnEnable()` | 활성화 시 | 오브젝트가 활성화될 때마다 실행 |
| `Start()` | 첫 프레임 | 게임 시작 시 초기화 (한 번만) |
| `OnDisable()` | 비활성화 시 | 오브젝트 비활성화 시 정리 작업 |
| `OnDestroy()` | 파괴 시 | 오브젝트 파괴 시 최종 정리 |

### 이벤트 루프
Unity가 내부적으로 지속 실행하는 Update 사이클로, 매 프레임마다 반복됩니다.
```csharp
// 예시 코드
private void Awake()
{
    Debug.Log("Awake - 오브젝트가 생성 될 때 호출");
}

private void OnEnable()
{
    Debug.Log("OnEnable - 활성화 될 때 호출");
}

private void Start()
{
    Debug.Log("Start - 첫프레임 때 한번 초기화");
}

private void OnDisable()
{
    Debug.Log("OnDisable - 비활성화 될 때 호출");
}

private void OnDestroy()
{
    Debug.Log("OnDestroy - 오브젝트 파괴 될 때 호출");
}
```