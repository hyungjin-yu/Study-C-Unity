---
tags:
  - 코루틴
  - Awaitable
  - async
  - await
  - Task
  - CancellationToken
  - Unity
---

## 비교표

| | Coroutine | async/await (Awaitable, Unity 6~) |
|---|---|---|
| 반환 타입 | IEnumerator | Task / Unity 6의 Awaitable |
| 실행 주체 | MonoBehaviour의 StartCoroutine | 독립적으로 실행 가능 (MonoBehaviour 없어도 됨) |
| 예외 처리 | try/catch가 yield 지점에서 제대로 안 먹힘 | try/catch 정상 작동 |
| 취소 | StopCoroutine (수동 관리 번거로움) | CancellationToken으로 표준화 |
| 값 반환 | 못 함 | Task\<T\>로 반환 가능 |

## 코드 비교

```csharp
IEnumerator LoadRoutine()
{
    yield return new WaitForSeconds(1f);
    Debug.Log("done");
}

async Awaitable LoadAsync()
{
    await Awaitable.WaitForSecondsAsync(1f);
    Debug.Log("done");
}
```

## 실무 기준

- **Coroutine**: "몇 프레임 뒤 / 몇 초 뒤 실행" 같은 Unity 특화 타이밍 제어에 여전히 많이 사용. 자세한 사용법·yield 종류는 [[Unity - 코루틴 심화 (시작과 제어 방법)]], [[Unity - 설계 패턴 - 상속 vs 컴포지션 & 코루틴]] 참고
- **async/await**: 파일 I/O, 네트워크, Addressables 로딩처럼 진짜 비동기 작업에 적합
- 최근엔 Unity 6의 Awaitable이 Coroutine을 대체하는 방향으로 가고 있음

## 관련 개념
- [[Unity - 설계 패턴 - 상속 vs 컴포지션 & 코루틴]]
- [[Unity - 코루틴 심화 (시작과 제어 방법)]]
