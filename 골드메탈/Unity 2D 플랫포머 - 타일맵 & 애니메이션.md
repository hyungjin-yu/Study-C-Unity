#Unity #2DPlatformer #Tilemap #Animation #GoldMetal
## 📌 간단 요약
2D 플랫포머 게임 제작을 위한 타일맵 시스템 구축, 물리 이동 처리, 그리고 애니메이터 제어 방법

---

## 1. 타일맵 시스템 (Tilemap System)

### 핵심 컴포넌트
- **Tile Palette**: 타일을 사용하기 위해 모아둔 프리팹 모음
- **TileMap**: 타일을 일정하게 배치하는 컴포넌트
- **TileMap Collider 2D**: 타일맵에 맞춰 자동 생성되는 콜라이더
- **Sprite Editor**: 타일의 물리 모양 편집 도구

### 설정 순서 (중요!)
```
1. Tile Palette 생성
2. TileMap 생성
3. TileMap Collider 2D 추가
4. Sprite Editor로 물리 모양 편집
```

> ⚠️ **주의사항**: Tile Palette에서 먼저 타일을 삭제한 후, 물리 모양을 편집하는 것이 훨씬 안전합니다.

---

## 2. 캐릭터 이동 처리

### AddForce의 한계
- `AddForce`로 이동 시 높은 경사로에서 문제 발생
- 걷는 애니메이션이 제대로 재생되지 않음

### 해결 방법: Velocity 사용
```csharp
// AddForce 대신 velocity 직접 제어
rigidbody2D.velocity = new Vector2(speed, rigidbody2D.velocity.y);
```

---

## 3. 재귀 함수 (Recursive Function)

### 정의
자기 자신을 스스로 호출하는 함수

### ⚠️ 위험성
**딜레이 없이 재귀 함수를 사용하는 것은 매우 위험합니다!**
- 무한 루프로 게임 크래시 가능
- 스택 오버플로우 발생 위험

---

## 4. Invoke 함수

### Invoke()
```csharp
// 2초 후에 함수 실행
Invoke("FunctionName", 2.0f);
```
- **기능**: 주어진 시간이 지난 뒤, 지정된 함수를 실행
- **매개변수**: (함수명, 지연시간)

### CancelInvoke()
```csharp
// 모든 Invoke 중단
CancelInvoke();

// 특정 함수의 Invoke만 중단
CancelInvoke("FunctionName");
```
- **기능**: 현재 작동 중인 모든 Invoke 함수를 멈춤

### 예시 코드
```csharp
void Start()
{
    // 3초 후 Damage 함수 실행
    Invoke("Damage", 3.0f);
}

void Damage()
{
    Debug.Log("플레이어가 데미지를 입었습니다!");
}

void OnDestroy()
{
    // 오브젝트 파괴 시 모든 Invoke 취소
    CancelInvoke();
}
```

---

## 5. 애니메이터 - Trigger 파라미터

### Trigger란?
- **역할**: 방아쇠 역할의 매개변수
- **특징**: 값이 없음 (bool, int, float과 다름)
- **용도**: 일회성 애니메이션 전환

### Any State → Exit
```
Any State → 애니메이션 → Exit
```
- **의미**: 현재 상태와 상관없이 애니메이션 실행 후 복귀
- **사용 예시**: 피격 모션, 점프 착지 등

### 예시 코드
```csharp
Animator animator;

void Start()
{
    animator = GetComponent<Animator>();
}

void Attack()
{
    // Trigger 파라미터 활성화 (일회성)
    animator.SetTrigger("Attack");
}

void Jump()
{
    // Any State에서 점프 애니메이션 실행 후 원래 상태로 복귀
    animator.SetTrigger("Jump");
}
```

---

## 💡 핵심 포인트

1. **타일맵 편집**: 항상 Palette에서 먼저 삭제 → Sprite Editor 편집 순서
2. **이동 처리**: 경사로 문제는 velocity로 해결
3. **재귀 함수**: 반드시 종료 조건과 딜레이 추가
4. **Invoke**: 시간 기반 함수 실행에 유용하며, 정리 시 CancelInvoke 필수
5. **Trigger**: 일회성 애니메이션 전환에 최적화된 파라미터