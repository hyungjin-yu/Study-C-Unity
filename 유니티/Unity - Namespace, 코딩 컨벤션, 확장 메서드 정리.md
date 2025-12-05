


#CSharp #Namespace #ExtensionMethod #Unity #CodingConvention

## 📌 주요 내용

**1. using과 namespace**
- using은 해당 네임스페이스의 클래스를 간편하게 사용하기 위한 선언
- using: 네임스페이스를 선언하여 전체 경로 없이 클래스 사용 가능
- namespace: 클래스 이름 중복 방지 및 목적성 표시

**2. Unity 폴더 구조**

```
Assets/Scripts/
├── Player/
├── Enemy/
└── UI/
```

**3. namespace 목적**

- class 이름 중복 해결
- class의 목적성을 가시적으로 표현

**4. C# 코딩 컨벤션**

- 일관성 있는 코드 레이아웃
- 내용에 집중할 수 있는 코드 작성

**5. Inspector Attribute**

```csharp
[Header("체력")] public int hp;
[Tooltip("이동 속도")] public float speed;
```

**6. 확장 메서드**

```csharp
public static class Extensions {
    public static bool IsZero(this int value) {
        return value == 0;
    }
}
```

## 💡 핵심 요약

- using으로 네임스페이스 간편 사용
- namespace로 클래스 구조화
- 확장 메서드는 static class 내 static 메서드로 정의