---
tags:
  - Transform
  - LocalScale
  - Position
  - Vector3
  - TimedeltaTime
  - Translate
  - Rotate
  - unity
---
## Transform 컴포넌트

게임 오브젝트의 **위치(Position), 회전(Rotation), 크기(Scale)**를 저장하는 컴포넌트

## 크기 설정 (Scale)

### localScale
부모 오브젝트 기준의 **상대적 크기**
```csharp
transform.localScale = new Vector3(1, 1, 1);
```

## 위치 제어 (Position)

### transform.position
**월드(글로벌) 좌표계** 기준 위치
- 게임 오브젝트의 방향과 무관한 순수 좌표 이동
```csharp
// 직접 좌표 변경
transform.position = transform.position + new Vector3(0, 0, 1);
```

### 프레임 독립적 이동
```csharp
// 1초에 Z축 양수 방향으로 speed만큼 이동
Vector3 vector = new Vector3(0, 0, speed) * Time.deltaTime;

// Vector3.forward 사용 (동일한 효과)
Vector3 vector = Vector3.forward * speed * Time.deltaTime;
```

## 이동 함수 - Translate()

### 문법
```csharp
transform.Translate(방향과크기(벡터), 좌표계);
```

### 좌표계 종류
- **Space.World**: 월드(글로벌) 좌표계
- **Space.Self**: 로컬 좌표계 (오브젝트 자신의 방향 기준)

### 예시
```csharp
// 로컬 좌표계 기준 앞으로 이동
transform.Translate(Vector3.forward * speed * Time.deltaTime, Space.Self);
```

## 회전 함수 - Rotate()

### 문법
```csharp
transform.Rotate(회전축과크기(벡터), 좌표계);
```

### 예시
```csharp
// Y축 기준으로 초당 35도 회전 (로컬 좌표계)
transform.Rotate(new Vector3(0, 35, 0) * Time.deltaTime, Space.Self);
```

## 초기화 팁

**Start() 함수에서 멤버 초기화**
- 게임 시작 시 한 번만 실행
- 초기 설정에 적합
```csharp
void Start()
{
    // 멤버 변수 초기화
    speed = 5.0f;
    transform.localScale = Vector3.one;
}
```

## 핵심 개념 정리

### Position vs Translate
- **position**: 글로벌 좌표 직접 변경
- **Translate**: 로컬/글로벌 선택 가능, 방향 기준 이동

### 좌표계 차이
- **글로벌(World)**: 월드 기준 절대 좌표
- **로컬(Self)**: 오브젝트 자신의 방향 기준

### Time.deltaTime
- 프레임 독립적 움직임 보장
- 초당 일정한 속도 유지