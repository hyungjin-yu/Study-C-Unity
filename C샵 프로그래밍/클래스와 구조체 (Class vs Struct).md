---
tags:
  - 클래스
  - 구조체
  - 값타입
  - 참조타입
  - 힙
  - 스택
  - C샤프
  - Unity
---

## 핵심 차이

| 구분 | class | struct |
|---|---|---|
| 저장 위치 | 힙(Heap) | 스택(Stack), 단 필드로 포함되면 그 컨테이너를 따라감 |
| 복사 방식 | 참조 복사 (같은 객체를 가리킴) | 값 복사 (통째로 복사됨) |
| 상속 | 가능 | 불가능 (인터페이스 구현만 가능) |
| null 가능 | 가능 | 불가능 (기본값 있음) |

## 예시로 보는 차이

```csharp
class PlayerData { public int hp; }
struct Vector2Int { public int x, y; }

PlayerData a = new PlayerData { hp = 100 };
PlayerData b = a;
b.hp = 50;
// a.hp도 50 — 같은 객체를 가리키니까

Vector2Int v1 = new Vector2Int { x = 1, y = 1 };
Vector2Int v2 = v1;
v2.x = 99;
// v1.x는 그대로 1 — 값이 복사됐으니까
```

## Unity가 Vector3/Quaternion/Color를 struct로 만든 이유

매 프레임 수백~수천 개가 생성/연산되는 타입인데, class였다면 힙 할당이 쌓여 GC가 자주 개입 → 프레임 드랍으로 이어짐. struct는 스택에 쌓이고 복사돼도 힙 할당이 없어서 GC 부담이 없음.

## 선택 기준 (면접 답변)

> Q. struct vs class, 언제 뭘 쓰나요?

값 자체가 중요하고 작고(대략 16바이트 이하) 불변에 가까운 데이터 — 좌표, 색상 같은 건 struct로 만든다. 복사돼도 상관없고 힙 할당이 없어서 GC 부담이 없기 때문. 반대로 정체성이 중요하고(같은 객체를 여러 곳에서 참조/공유해야 하고), 상속이 필요하거나 라이프사이클을 가진 것 — Enemy, Player, Inventory 같은 건 class로 만든다.

## 관련 개념
- [[박싱과 언박싱]] — 값 타입을 참조 타입처럼 다룰 때(예: struct를 인터페이스로 캐스팅) 박싱이 발생하는 이유
- [[제네릭과 컬렉션]] — `where T : struct` 제약 조건
- [[Fake Null과 GC 최적화 - Unity 면접 정리]]
