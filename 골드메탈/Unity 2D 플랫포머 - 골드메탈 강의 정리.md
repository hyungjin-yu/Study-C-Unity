---
tags:
  - Unity
  - 2D플랫포머
  - 골드메탈
  - BoxCollider
  - Rigidbody
  - Animator
  - RayCast
  - 물리엔진
---
### Box Collider
**부딪힐 수 있는 충돌 컴포넌트**
```csharp
// Collider 크기를 이미지에 맞게 조정
BoxCollider2D collider = GetComponent<BoxCollider2D>();
collider.size = new Vector2(1f, 2f);
```

### Rigidbody 2D
**중력을 생성하는 물리 컴포넌트**

**주요 속성**:
- **Mass**: 질량 (기본값 1)
- **Linear Drag**: 공기 저항, 이동 속도 감소 (권장: 1~2)
- **Gravity Scale**: 오브젝트에 적용되는 중력 비율
- **Freeze Rotation**: 오브젝트의 회전을 멈추는 옵션
```csharp
Rigidbody2D rb = GetComponent<Rigidbody2D>();

// 속도 설정
rb.velocity = new Vector2(5f, rb.velocity.y);

// 점프
rb.AddForce(Vector2.up * 10f, ForceMode2D.Impulse);

// 회전 고정
rb.constraints = RigidbodyConstraints2D.FreezeRotation;
```

### Default Contact Offset
**충돌 여백 설정** (Edit > Project Settings > Physics 2D)
- 충돌 감지의 정확도 조절

## 이동 구현

### 정규화 벡터 (normalized)
**벡터의 크기를 1로 만든 단위벡터**

대각선 이동 시 속도가 빨라지는 문제 해결
```csharp
void Update()
{
    float h = Input.GetAxis("Horizontal");
    float v = Input.GetAxis("Vertical");
    
    // ❌ 대각선 이동 시 √2배 빨라짐
    Vector2 moveInput = new Vector2(h, v);
    
    // ✅ 정규화로 속도 일정하게 유지
    Vector2 normalizedInput = moveInput.normalized;
    
    transform.position += (Vector3)(normalizedInput * speed * Time.deltaTime);
}
```

### 좌우 반전 (Flip)
**스프라이트를 뒤집는 옵션**
```csharp
SpriteRenderer spriteRenderer;

void Start()
{
    spriteRenderer = GetComponent<SpriteRenderer>();
}

void Update()
{
    float h = Input.GetAxis("Horizontal");
    
    if (h > 0)
        spriteRenderer.flipX = false;  // 오른쪽
    else if (h < 0)
        spriteRenderer.flipX = true;   // 왼쪽
}
```

## 애니메이션

### Animator
**애니메이션을 관리하는 컴포넌트**

**구성 요소**:
- **Key Frame**: 애니메이션 값을 가진 프레임
- **State**: 애니메이션 상태 단위 (Idle, Walk, Jump 등)
- **Transition**: 애니메이션 상태를 옮겨가는 통로
- **Parameters**: 상태를 바꿀 때 필요한 변수

### 2D 애니메이션 설정 (중요!)

**3가지 필수 설정**:
1. **Has Exit Time 끄기**: 즉시 전환 가능하게
2. **겹구간 닫기**: Transition Duration을 0으로
3. **매개변수 설정**: Bool, Float, Trigger 등
```csharp
Animator anim;

void Start()
{
    anim = GetComponent<Animator>();
}

void Update()
{
    float h = Input.GetAxis("Horizontal");
    
    // Bool 파라미터 설정
    anim.SetBool("isWalking", h != 0);
    
    // Float 파라미터 설정
    anim.SetFloat("Speed", Mathf.Abs(h));
    
    // Trigger 파라미터 (점프)
    if (Input.GetButtonDown("Jump"))
        anim.SetTrigger("Jump");
}
```

## 점프 구현

### 바닥 체크 - RayCast
**오브젝트 검색을 위해 Ray를 쏘는 방식**
```csharp
public LayerMask groundLayer;  // 인스펙터에서 설정
private bool isGrounded;

void Update()
{
    // 바닥 체크
    CheckGround();
    
    // 점프
    if (Input.GetButtonDown("Jump") && isGrounded)
    {
        GetComponent<Rigidbody2D>().velocity = Vector2.up * jumpPower;
    }
}

void CheckGround()
{
    // Ray 발사
    RaycastHit2D hit = Physics2D.Raycast(
        transform.position,      // 시작 위치
        Vector2.down,            // 방향
        1.5f,                    // 거리
        groundLayer              // 검사할 레이어
    );
    
    // 에디터에서 Ray 시각화
    Debug.DrawRay(transform.position, Vector2.down * 1.5f, Color.red);
    
    // 충돌 확인
    isGrounded = hit.collider != null;
}
```

### LayerMask
**물리 효과를 구분하는 정수값**

사용법:
1. GameObject에 Layer 설정 (예: "Ground")
2. 스크립트에서 LayerMask 변수 선언
3. 인스펙터에서 검사할 Layer 선택
```csharp
public LayerMask whatIsGround;

void CheckCollision()
{
    // 특정 레이어만 검사
    Collider2D[] colliders = Physics2D.OverlapCircleAll(
        transform.position, 
        0.5f, 
        whatIsGround
    );
}
```

## 수학 함수 - Mathf

### Mathf.Abs()
**절대값 반환**
```csharp
float speed = -5f;
float absSpeed = Mathf.Abs(speed);  // 5f

// 활용: 이동 속도 크기만 확인
if (Mathf.Abs(rb.velocity.x) > maxSpeed)
{
    rb.velocity = new Vector2(
        Mathf.Sign(rb.velocity.x) * maxSpeed, 
        rb.velocity.y
    );
}
```

### 기타 유용한 Mathf 함수
```csharp
// 값 제한
float hp = Mathf.Clamp(currentHp, 0, maxHp);

// 선형 보간
float smooth = Mathf.Lerp(current, target, 0.1f);

// 반올림
int rounded = Mathf.RoundToInt(3.7f);  // 4
```

## Update 함수

### 단발적 변화는 Update()
**키 입력, 점프 등 즉각적인 반응**
```csharp
void Update()
{
    // 키 입력 감지
    if (Input.GetButtonDown("Jump"))
    {
        Jump();
    }
    
    // 애니메이션 상태 변경
    anim.SetBool("isMoving", isMoving);
}
```

### 물리 변화는 FixedUpdate()
**Rigidbody 조작**
```csharp
void FixedUpdate()
{
    // 물리 기반 이동
    float h = Input.GetAxis("Horizontal");
    rb.velocity = new Vector2(h * speed, rb.velocity.y);
}
```

## 물리 설정

### Physics 2D 설정
**Edit > Project Settings > Physics 2D**

- **Gravity**: 중력 값 (기본: -9.81)
- **Default Contact Offset**: 충돌 여백
- **Velocity Iterations**: 속도 정확도
- **Position Iterations**: 위치 정확도

### Gravity Scale 활용
```csharp
Rigidbody2D rb;

void Jump()
{
    // 일반 점프
    rb.gravityScale = 3f;
    rb.velocity = Vector2.up * jumpPower;
}

void FloatJump()
{
    // 떠있는 점프
    rb.gravityScale = 1f;
    rb.velocity = Vector2.up * jumpPower;
}
```

## 최적화

### Batch Count
**그래픽을 그리기 위해 메모리와 CPU를 사용한 횟수**

- Stats 창에서 확인 가능
- Batch 수를 줄이면 성능 향상
- Sprite Atlas 사용으로 최적화

## 실전 예시

### 2D 플레이어 컨트롤러
```csharp
using UnityEngine;

public class Player2DController : MonoBehaviour
{
    [Header("이동")]
    public float moveSpeed = 5f;
    public float jumpPower = 10f;
    
    [Header("컴포넌트")]
    private Rigidbody2D rb;
    private SpriteRenderer spriteRenderer;
    private Animator anim;
    
    [Header("바닥 체크")]
    public Transform groundCheck;
    public LayerMask groundLayer;
    private bool isGrounded;
    
    void Start()
    {
        rb = GetComponent<Rigidbody2D>();
        spriteRenderer = GetComponent<SpriteRenderer>();
        anim = GetComponent<Animator>();
        
        // Rigidbody 설정
        rb.constraints = RigidbodyConstraints2D.FreezeRotation;
        rb.drag = 1.5f;
    }
    
    void Update()
    {
        // 바닥 체크
        CheckGround();
        
        // 점프
        if (Input.GetButtonDown("Jump") && isGrounded)
        {
            rb.velocity = new Vector2(rb.velocity.x, jumpPower);
            anim.SetTrigger("Jump");
        }
        
        // 애니메이션
        float speedAbs = Mathf.Abs(rb.velocity.x);
        anim.SetFloat("Speed", speedAbs);
        anim.SetBool("isGrounded", isGrounded);
    }
    
    void FixedUpdate()
    {
        // 이동
        float h = Input.GetAxis("Horizontal");
        rb.velocity = new Vector2(h * moveSpeed, rb.velocity.y);
        
        // 좌우 반전
        if (h > 0)
            spriteRenderer.flipX = false;
        else if (h < 0)
            spriteRenderer.flipX = true;
    }
    
    void CheckGround()
    {
        // Raycast로 바닥 체크
        RaycastHit2D hit = Physics2D.Raycast(
            groundCheck.position,
            Vector2.down,
            0.1f,
            groundLayer
        );
        
        // 디버그 표시
        Debug.DrawRay(groundCheck.position, Vector2.down * 0.1f, 
            hit.collider != null ? Color.green : Color.red);
        
        isGrounded = hit.collider != null;
    }
}
```

## 핵심 정리

| 항목 | 설명 | 예시 |
|------|------|------|
| Box Collider | 충돌 영역 | 플레이어 몸체 |
| Rigidbody2D | 물리 시뮬레이션 | 중력, 속도 |
| normalized | 단위 벡터 | 대각선 이동 보정 |
| RayCast | 오브젝트 검색 | 바닥 체크 |
| Animator | 애니메이션 관리 | Idle, Walk, Jump |
| LayerMask | 레이어 구분 | Ground, Player |

## 관련 개념
- Unity 물리 엔진
- 2D 게임 개발
- 캐릭터 컨트롤러