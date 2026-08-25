---
tags:
  - FakeNull
  - MissingReferenceException
  - GC
  - 가비지컬렉션
  - 스터터링
  - Unity
  - 면접
---

## Fake Null

일반 C# 오브젝트는 진짜 null이면 참조 자체가 없는 상태. 하지만 Unity의 MonoBehaviour/GameObject는 `Destroy()`된 후에도 C# 참조 자체는 살아있고, Unity 엔진 쪽 네이티브 객체만 파괴됨.

이때 `== null` 비교가 Unity에서 오버로딩되어 있어서 `true`가 나오지만, 실제 C# 레벨에서는 null이 아닌 상태 — 이게 **fake null**.

파괴된 오브젝트 참조에 접근하면 `MissingReferenceException` 발생. 막으려면 사용 전에 항상 null 체크(또는 `if (obj)`)를 해야 함.

> **면접 답변**: null 체크는 왜 하나요? fake null이 뭔가요?
> "일반 C# 오브젝트는 진짜 null이면 참조 자체가 없는 건데, Unity의 MonoBehaviour나 GameObject는 Destroy()된 후에도 C# 참조 자체는 살아있고 Unity 엔진 쪽 네이티브 객체만 파괴됩니다. 이때 == null 비교가 Unity에서 오버로딩되어 있어서 true가 나오지만, 실제 C# 레벨에서는 null이 아닌 상태—이게 fake null입니다. 그래서 파괴된 오브젝트 참조에 접근하면 MissingReferenceException이 나고, 이걸 막으려면 사용 전에 항상 null 체크(또는 if (obj))를 해야 합니다."

## 가비지 컬렉션이 게임에서 문제인 이유

GC가 메모리를 정리하는 동안 짧게라도 실행이 멈추는 지점(stop-the-world 또는 부분 정지)이 생김. 게임은 매 프레임 16ms(60fps 기준) 안에 다 끝내야 하는데 GC가 그 타이밍에 걸리면 순간 프레임 드랍 = 스터터링 발생.

**피해야 할 패턴**:
- Update 안에서 매 프레임 `new`
- LINQ를 매 프레임 사용 (내부적으로 iterator/delegate를 계속 새로 생성)
- 박싱이 일어나는 코드 → [[박싱과 언박싱]]

**대응**: 오브젝트 풀링으로 할당 자체를 줄이기 → [[Unity - Object Pooling과 메모리 관리]]

> **면접 답변**: 가비지 컬렉션이 왜 게임에서 문제인가요?
> "GC가 메모리를 정리하는 동안 짧게라도 실행이 멈추는 지점(stop-the-world 또는 부분 정지)이 생기는데, 게임은 매 프레임 16ms(60fps 기준) 안에 다 끝내야 하니까 GC가 그 타이밍에 걸리면 순간 프레임 드랍, 즉 스터터링이 발생합니다. 그래서 Update 안에서 매 프레임 new를 하거나, LINQ 쓰거나, 박싱이 일어나는 코드를 피하고, 오브젝트 풀링으로 할당 자체를 줄이는 게 실무에서 중요한 최적화 습관입니다."

## 관련 개념
- [[박싱과 언박싱]]
- [[Unity - Object Pooling과 메모리 관리]]
- [[클래스와 구조체 (Class vs Struct)]]
