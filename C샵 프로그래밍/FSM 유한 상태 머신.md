---
tags:
  - FSM
  - 유한상태머신
  - 상태패턴
  - 게임AI
  - 디자인패턴
  - C샤프
  - State
  - Transition
  - 게임개발
  - AI시스템
  - 상태전이
  - StatePattern
---
# FSM (Finite State Machine)

## 개념
**유한한 개수의 상태**를 가지며, 특정 조건에 따라 **상태가 전환**되는 시스템

## 실생활 예시
- 신호등: 빨강 → 초록 → 노랑 → 빨강
- 자동판매기: 대기 → 돈 투입 → 상품 선택 → 상품 배출 → 대기
- 게임 메뉴: 메뉴 → 상점 → 전투 → 메뉴

## FSM 구성 요소

### 1. 상태 (State)
시스템이 가질 수 있는 **특정 조건이나 모드**

**예시 (게임 캐릭터)**:
- 대기(Idle)
- 이동(Move)
- 공격(Attack)
- 피격(Hit)
- 사망(Dead)

### 2. 전이 (Transition)
**상태에서 다른 상태로 변경**되는 것

**예시**:
- 대기 → 이동 (방향키 입력)
- 이동 → 공격 (공격 버튼 입력)
- 공격 → 대기 (공격 애니메이션 종료)

### 3. 이벤트 (Event)
전이를 **발생시키는 조건이나 입력**

**예시**:
- 키보드 입력
- 마우스 클릭
- 타이머 종료
- HP가 0 이하
- 충돌 감지

### 4. 액션 (Action)
**특정 상태에서 실행되는 동작**

**종류**:
- **Entry Action**: 상태 진입 시 실행
- **Update Action**: 상태 유지 중 매 프레임 실행
- **Exit Action**: 상태 종료 시 실행

## 구현 예시 (C#)

### 기본 FSM 구조
```csharp
// 상태 열거형
enum PlayerState
{
    Idle,
    Move,
    Attack,
    Dead
}

class Player
{
    private PlayerState currentState = PlayerState.Idle;
    
    public void Update()
    {
        switch (currentState)
        {
            case PlayerState.Idle:
                UpdateIdleState();
                break;
            case PlayerState.Move:
                UpdateMoveState();
                break;
            case PlayerState.Attack:
                UpdateAttackState();
                break;
            case PlayerState.Dead:
                UpdateDeadState();
                break;
        }
    }
    
    private void UpdateIdleState()
    {
        // 이벤트 체크
        if (Input.GetKey("W"))
        {
            TransitionToState(PlayerState.Move);
        }
        else if (Input.GetKey("Space"))
        {
            TransitionToState(PlayerState.Attack);
        }
    }
    
    private void TransitionToState(PlayerState newState)
    {
        // Exit Action
        ExitState(currentState);
        
        // 상태 전이
        currentState = newState;
        
        // Entry Action
        EnterState(currentState);
    }
    
    private void EnterState(PlayerState state)
    {
        switch (state)
        {
            case PlayerState.Move:
                Console.WriteLine("이동 시작");
                break;
            case PlayerState.Attack:
                Console.WriteLine("공격 시작");
                break;
        }
    }
    
    private void ExitState(PlayerState state)
    {
        switch (state)
        {
            case PlayerState.Move:
                Console.WriteLine("이동 종료");
                break;
            case PlayerState.Attack:
                Console.WriteLine("공격 종료");
                break;
        }
    }
}
```

### 인터페이스 기반 FSM
```csharp
// 상태 인터페이스
interface IState
{
    void Enter();
    void Update();
    void Exit();
}

// 구체적인 상태 클래스
class IdleState : IState
{
    public void Enter() 
    { 
        Console.WriteLine("대기 상태 진입"); 
    }
    
    public void Update() 
    { 
        // 대기 중 로직
    }
    
    public void Exit() 
    { 
        Console.WriteLine("대기 상태 종료"); 
    }
}

class MoveState : IState
{
    public void Enter() 
    { 
        Console.WriteLine("이동 시작"); 
    }
    
    public void Update() 
    { 
        // 이동 로직
    }
    
    public void Exit() 
    { 
        Console.WriteLine("이동 종료"); 
    }
}

// FSM 매니저
class StateMachine
{
    private IState currentState;
    
    public void ChangeState(IState newState)
    {
        currentState?.Exit();
        currentState = newState;
        currentState?.Enter();
    }
    
    public void Update()
    {
        currentState?.Update();
    }
}
```

## FSM 다이어그램 예시
```
[메인 메뉴] 
    ↓ (게임 시작)
[캐릭터 선택]
    ↓ (선택 완료)
[게임 플레이]
    ⇄ (상점 버튼)
[상점]
    ↓ (구매 완료)
[게임 플레이]
    ⇄ (전투 시작)
[전투]
    ↓ (전투 종료)
[게임 플레이]
    ↓ (ESC)
[메인 메뉴]
```

## 장점

1. **명확한 구조**: 각 상태와 전이가 명확히 정의됨
2. **유지보수 용이**: 상태별로 코드가 분리되어 수정 쉬움
3. **버그 감소**: 예상치 못한 상태 조합 방지
4. **확장성**: 새로운 상태 추가가 쉬움

## 단점

1. **상태 폭발**: 상태가 많아지면 관리 복잡
2. **코드 증가**: 각 상태마다 클래스/메서드 필요
3. **전이 복잡도**: 상태 간 전이 규칙이 복잡해질 수 있음

## 실전 활용

### 게임 AI
```csharp
enum EnemyState
{
    Patrol,    // 순찰
    Chase,     // 추적
    Attack,    // 공격
    Retreat    // 후퇴
}

class EnemyAI
{
    private EnemyState state = EnemyState.Patrol;
    
    public void Update()
    {
        switch (state)
        {
            case EnemyState.Patrol:
                if (PlayerDetected())
                    state = EnemyState.Chase;
                break;
                
            case EnemyState.Chase:
                if (PlayerInRange())
                    state = EnemyState.Attack;
                else if (!PlayerDetected())
                    state = EnemyState.Patrol;
                break;
                
            case EnemyState.Attack:
                if (!PlayerInRange())
                    state = EnemyState.Chase;
                else if (HealthLow())
                    state = EnemyState.Retreat;
                break;
                
            case EnemyState.Retreat:
                if (HealthRecovered())
                    state = EnemyState.Patrol;
                break;
        }
    }
}
```

### UI 메뉴 시스템
```csharp
enum MenuState
{
    MainMenu,
    Settings,
    Gameplay,
    Paused,
    GameOver
}

class MenuManager
{
    private MenuState currentMenu = MenuState.MainMenu;
    
    public void ShowMenu(MenuState menu)
    {
        // 이전 메뉴 숨기기
        HideMenu(currentMenu);
        
        // 새 메뉴 표시
        currentMenu = menu;
        DisplayMenu(currentMenu);
    }
}
```

## 최적화 팁

### 1. State Pattern 사용
복잡한 FSM은 디자인 패턴의 State Pattern 활용

### 2. 상태 캐싱
```csharp
// 매번 생성하지 않고 재사용
class StateCache
{
    private static IdleState idleState = new IdleState();
    private static MoveState moveState = new MoveState();
    
    public static IState GetIdleState() => idleState;
    public static IState GetMoveState() => moveState;
}
```

### 3. 전이 테이블 사용
```csharp
// 복잡한 전이 규칙을 테이블로 관리
Dictionary<(State from, Event evt), State> transitions = new()
{
    { (State.Idle, Event.MoveKey), State.Move },
    { (State.Idle, Event.AttackKey), State.Attack },
    { (State.Move, Event.StopKey), State.Idle },
    // ...
};
```

## 대안 기술

### Behavior Tree
- 더 복잡한 AI 로직에 적합
- 계층적 구조
- 재사용성 높음

### Utility AI
- 점수 기반 의사결정
- 더 유연한 행동 선택
- FSM보다 자연스러운 동작

## 관련 개념
- 디자인 패턴 - State Pattern
- 게임 AI 시스템