#Unity #StringBuilder

유니티에서 문자열을 여러 번 수정하거나 합칠 때, 일반적인 더하기(`+`) 연산보다 성능이 뛰어난 **StringBuilder**를 사용하는 것이 좋습니다 [[5](https://discussions.unity.com/t/concatenating-strings-speed-test-which-is-faster/798660)].

### 🛠️ 기본 사용법

`System.Text` 네임스페이스를 추가한 후 다음과 같이 사용합니다.

```
using System.Text;

StringBuilder sb = new StringBuilder();
sb.Append("Score: ");
sb.Append(100);
sb.AppendLine(); // 줄바꿈
sb.AppendFormat("Level: {0}", 5);

string result = sb.ToString(); // 최종 문자열 변환
```

### 💡 언제 사용하나요?

1. **많은 문자열 결합**: 반복문(for, while) 안에서 문자열을 계속 합쳐야 할 때 필수입니다.
    
2. **가비지 컬렉션(GC) 방지**: 일반 문자열 결합은 매번 새로운 메모리를 할당하지만, StringBuilder는 내부 버퍼를 재사용하여 메모리 효율이 높습니다.
    
3. **주의사항**: 결합하는 문자열이 4~8개 이하로 적을 때는 일반적인 `+` 연산이 더 빠를 수 있습니다.