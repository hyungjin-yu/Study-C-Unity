#Unity #Material #PhysicsMaterial #UI #Canvas #Collision

## 📌 간단 요약
Material 텍스처 설정, Physics Material을 통한 물리적 재질 구현, UI Canvas 시스템 활용 방법

---

## 1. Material 설정

### Tiling (텍스처 반복)
```csharp
// Material의 Tiling 값 설정
Material material = GetComponent<MeshRenderer>().material;
material.mainTextureScale = new Vector2(2, 2);
```

| Tiling 값 | 결과 | 설명 |
|-----------|------|------|
| (1, 1) | 타일 1개 | 텍스처 1개가 전체 표면 덮음 |
| (2, 2) | 타일 4개 | 가로 2개 × 세로 2개 = 4개 |
| (3, 3) | 타일 9개 | 가로 3개 × 세로 3개 = 9개 |
| (0.5, 0.5) | 그림 잘림 | 텍스처가 절반만 표시됨 |

> 💡 **팁**: 소수점을 입력하면 그림이 잘려서 들어감

---

### Emission (발광)
```csharp
// Emission 활성화 및 색상 설정
material.EnableKeyword("_EMISSION");
material.SetColor("_EmissionColor", Color.yellow);
```

**특징:**
- 텍스처의 밝기를 조절
- 빛이 물리적으로 나오는 것은 아님 (시각 효과만)

---

## 2. Physics Material (물리 재질)

### 개념
탄성과 마찰을 다루는 물리적 재질

### Bounciness (탄성력)
```csharp
// Physics Material 생성 및 적용
PhysicMaterial bouncyMaterial = new PhysicMaterial();
bouncyMaterial.bounciness = 0.8f; // 높을수록 많이 튀어오름
GetComponent<Collider>().material = bouncyMaterial;
```

**값 범위:** 0 (안 튐) ~ 1 (매우 많이 튐)

---

### Friction (마찰력)
```csharp
PhysicMaterial slipperyMaterial = new PhysicMaterial();
slipperyMaterial.dynamicFriction = 0.1f; // 낮을수록 많이 미끄러짐
slipperyMaterial.staticFriction = 0.1f;
```

**값 범위:** 0 (매우 미끄러움) ~ 1 (마찰 큼)

---

### Friction Combine (마찰력 계산 방식)
```csharp
bouncyMaterial.frictionCombine = PhysicMaterialCombine.Minimum;
bouncyMaterial.bounceCombine = PhysicMaterialCombine.Maximum;
```

| Combine 모드 | 설명 |
|--------------|------|
| Average | 평균값 사용 |
| Minimum | 최솟값 사용 |
| Maximum | 최댓값 사용 |
| Multiply | 곱셈 사용 |

> ⚠️ **중요 원칙**: 
> - **Friction 합산은 Minimum으로!** (더 미끄러운 쪽 우선)
> - **Bounciness 합산은 Maximum으로!** (더 튀는 쪽 우선)

---

### 실전 예시
```csharp
public class PhysicsMaterialExample : MonoBehaviour
{
    void Start()
    {
        // 매우 탄력적이고 미끄러운 재질 (공)
        PhysicMaterial ballMaterial = new PhysicMaterial();
        ballMaterial.bounciness = 0.9f;
        ballMaterial.dynamicFriction = 0.1f;
        ballMaterial.staticFriction = 0.1f;
        ballMaterial.frictionCombine = PhysicMaterialCombine.Minimum;
        ballMaterial.bounceCombine = PhysicMaterialCombine.Maximum;
        
        GetComponent<Collider>().material = ballMaterial;
    }
}
```

---

## 3. Rigidbody 관련 함수

### GetComponent<T>()
```csharp
// 자신의 T 타입 컴포넌트 가져오기
Rigidbody rb = GetComponent<Rigidbody>();
MeshRenderer renderer = GetComponent<MeshRenderer>();
```

---

### velocity (현재 이동 속도)
```csharp
void FixedUpdate()
{
    Rigidbody rb = GetComponent<Rigidbody>();
    
    // 현재 속도 확인
    Debug.Log($"속도: {rb.velocity}");
    
    // 속도 직접 설정
    rb.velocity = new Vector3(5, 0, 0);
}
```

> ⚠️ **중요**: Rigidbody 관련 코드는 **FixedUpdate()**에 작성하기!

---

### AddForce(Vec) - 힘 적용
```csharp
void FixedUpdate()
{
    Rigidbody rb = GetComponent<Rigidbody>();
    
    // Vec의 방향과 크기로 힘을 줌
    rb.AddForce(Vector3.forward * 10f);
}
```

**특징:**
- AddForce의 힘 방향으로 **계속 velocity가 증가함**
- 가속도 개념 (누적됨)

---

### ForceMode (힘을 주는 방식)
```csharp
// 질량 기반 연속 가속
rb.AddForce(Vector3.forward * 10f, ForceMode.Force);

// 질량 기반 순간 가속
rb.AddForce(Vector3.forward * 10f, ForceMode.Impulse);

// 질량 무시 연속 가속
rb.AddForce(Vector3.forward * 10f, ForceMode.Acceleration);

// 질량 무시 순간 가속
rb.AddForce(Vector3.forward * 10f, ForceMode.VelocityChange);
```

---

### AddTorque(Vec) - 회전력 적용
```csharp
void FixedUpdate()
{
    Rigidbody rb = GetComponent<Rigidbody>();
    
    // Vec 방향을 축으로 회전력 생성
    rb.AddTorque(Vector3.up * 10f); // Y축 기준 회전
}
```

> ⚠️ **주의**: Vec을 **축으로 삼기 때문에** 이동 방향에 주의해야 함!

**예시:**
```csharp
// Y축 회전 (위에서 보면 시계 반대 방향)
rb.AddTorque(Vector3.up * torquePower);

// X축 회전 (좌우로 구르기)
rb.AddTorque(Vector3.right * torquePower);

// Z축 회전 (앞뒤로 구르기)
rb.AddTorque(Vector3.forward * torquePower);
```

---

## 4. 오브젝트 재질 접근

### MeshRenderer를 통한 Material 접근
```csharp
public class MaterialController : MonoBehaviour
{
    MeshRenderer meshRenderer;
    Material material;
    
    void Start()
    {
        // MeshRenderer를 통해 재질 접근
        meshRenderer = GetComponent<MeshRenderer>();
        material = meshRenderer.material;
        
        // 재질 색상 변경
        material.color = Color.red;
    }
}
```

---

## 5. 충돌 이벤트 함수

### OnCollisionEnter (충돌 시작)
```csharp
void OnCollisionEnter(Collision collision)
{
    Debug.Log("충돌 시작: " + collision.gameObject.name);
    
    // 충돌 지점
    Debug.Log("충돌 위치: " + collision.contacts[0].point);
}
```

---

### OnCollisionStay (충돌 중)
```csharp
void OnCollisionStay(Collision collision)
{
    Debug.Log("충돌 지속 중");
}
```

---

### OnCollisionExit (충돌 끝)
```csharp
void OnCollisionExit(Collision collision)
{
    Debug.Log("충돌 종료: " + collision.gameObject.name);
}
```

---

### OnTriggerStay (Trigger 충돌 지속)
```csharp
void OnTriggerStay(Collider other)
{
    Debug.Log("Trigger 영역 내 체류 중");
}
```

---

## 6. 색상 클래스

### Color (기본 색상)
```csharp
// 0 ~ 1 범위
Color red = new Color(1f, 0f, 0f);
Color green = Color.green;

// 투명도 포함
Color transparent = new Color(1f, 0f, 0f, 0.5f);
```

---

### Color32 (255 색상)
```csharp
// 0 ~ 255 범위
Color32 red = new Color32(255, 0, 0, 255);
Color32 customColor = new Color32(128, 64, 200, 255);
```

**비교:**
```csharp
// 같은 빨간색 표현
Color red1 = new Color(1f, 0f, 0f);
Color32 red2 = new Color32(255, 0, 0, 255);
```

---

## 7. Collision 클래스 (충돌 정보)
```csharp
void OnCollisionEnter(Collision collision)
{
    // 충돌한 게임 오브젝트
    GameObject hitObject = collision.gameObject;
    
    // 충돌 지점들
    ContactPoint[] contacts = collision.contacts;
    foreach (ContactPoint contact in contacts)
    {
        Debug.Log($"충돌 위치: {contact.point}");
        Debug.Log($"충돌 법선: {contact.normal}");
    }
    
    // 상대 속도
    Debug.Log($"상대 속도: {collision.relativeVelocity}");
    
    // 충돌 임펄스 (충격량)
    Debug.Log($"충격량: {collision.impulse}");
}
```

---

## 8. UI Canvas 시스템

### Canvas (UI 도화지)

**역할:** UI가 그려지는 도화지 역할인 컴포넌트

**Render Mode:**
- **Screen Space - Overlay**: 화면 최상단에 표시
- **Screen Space - Camera**: 카메라 기준 표시
- **World Space**: 월드 공간에 배치

---

### 스크린 좌표계

**개념:**
- 게임이 표시되는 화면
- 해상도로 크기 결정
- 마우스 커서도 스크린 좌표계에 포함됨
```csharp
void Update()
{
    // 마우스 스크린 좌표
    Vector3 mousePos = Input.mousePosition;
    Debug.Log($"마우스 위치: {mousePos}");
    
    // 스크린 크기
    Debug.Log($"화면 크기: {Screen.width} x {Screen.height}");
}
```

---

### 폰트 사용 시 주의사항

> ⚠️ **중요**: 게임을 판매할 때, **폰트는 꼭 라이센스 확인**해야 함!

**무료 폰트 추천:**
- 눈누 (noonnu.cc)
- 구글 폰트
- Adobe Fonts

---

## 9. UI Image 설정

### Sprite 설정

> ⚠️ **필수**: 이미지를 첨부할 땐 **Sprite로 설정**해야 가능
```
1. 이미지 파일을 Unity로 드래그
2. Inspector에서 Texture Type을 Sprite (2D and UI)로 변경
3. Apply 클릭
```

---

### Preserve Aspect (비율 고정)
```csharp
// 코드로 Preserve Aspect 설정
Image image = GetComponent<Image>();
image.preserveAspect = true;
```

**효과:** 이미지 원본 비율 유지

---

### Set Native Size (실제 크기)
```csharp
// 코드로 Native Size 설정
Image image = GetComponent<Image>();
image.SetNativeSize();
```

**효과:** 이미지를 실제 픽셀 크기로 맞춤

---

### Image Type

#### Simple (단순 늘림)
```csharp
image.type = Image.Type.Simple;
```

**특징:** 가로세로 크기만큼 이미지 늘어남

---

#### Sliced (9-Sliced)
```csharp
image.type = Image.Type.Sliced;
```

**특징:**
- 이미지 양 끝을 잡고 늘리거나 줄임
- 모서리 부분은 왜곡되지 않음
- UI 버튼, 패널에 적합

**설정 방법:**
```
1. Sprite Editor 열기
2. Border 값 설정 (상하좌우)
3. Apply
```

---

#### Tiled (타일)
```csharp
image.type = Image.Type.Tiled;
```

**특징:**
- Width, Height만큼 이미지를 잘라서 타일처럼 붙임
- 패턴 배경에 적합

---

### Image Type 비교

| Type | 특징 | 용도 |
|------|------|------|
| Simple | 단순 늘림/줄임 | 아이콘, 로고 |
| Sliced | 모서리 유지하며 늘림 | 버튼, 패널 |
| Tiled | 타일처럼 반복 | 배경, 패턴 |
| Filled | 채워지는 효과 | HP바, 게이지 |

---

## 10. UI Button 이벤트

### OnClick() 이벤트
```csharp
public class ButtonController : MonoBehaviour
{
    public void OnButtonClick()
    {
        Debug.Log("버튼 클릭됨!");
    }
}
```

**Inspector 설정:**
```
1. Button 컴포넌트 선택
2. OnClick() 이벤트에 + 버튼 클릭
3. 오브젝트 드래그
4. 함수 선택 (OnButtonClick)
```

---

### 버튼 동작 원리

> 💡 **중요**: 버튼 동작은 **누르기 - 떼기**가 한 세트 동작
```csharp
public class ButtonBehavior : MonoBehaviour
{
    public void OnPointerDown()
    {
        Debug.Log("버튼 누름");
    }
    
    public void OnPointerUp()
    {
        Debug.Log("버튼 뗌");
    }
    
    public void OnClick()
    {
        Debug.Log("클릭 완료! (누르기 + 떼기)");
    }
}
```

---

## 11. 종합 예시 - 점프대 구현
```csharp
public class JumpPad : MonoBehaviour
{
    public float jumpForce = 20f;
    public Material normalMaterial;
    public Material activeMaterial;
    
    MeshRenderer meshRenderer;
    
    void Start()
    {
        meshRenderer = GetComponent<MeshRenderer>();
        
        // Physics Material 설정
        PhysicMaterial jumpMaterial = new PhysicMaterial();
        jumpMaterial.bounciness = 1f;
        jumpMaterial.bounceCombine = PhysicMaterialCombine.Maximum;
        GetComponent<Collider>().material = jumpMaterial;
    }
    
    void OnCollisionEnter(Collision collision)
    {
        Rigidbody rb = collision.gameObject.GetComponent<Rigidbody>();
        
        if (rb != null)
        {
            // 위쪽으로 강한 힘 적용
            rb.AddForce(Vector3.up * jumpForce, ForceMode.Impulse);
            
            // 재질 색상 변경
            meshRenderer.material = activeMaterial;
            
            // 0.2초 후 원래 색상으로
            Invoke("ResetMaterial", 0.2f);
        }
    }
    
    void ResetMaterial()
    {
        meshRenderer.material = normalMaterial;
    }
}
```

---

## 💡 핵심 포인트

1. **Material Tiling**: 소수점 사용 시 이미지 잘림 주의
2. **Emission**: 시각적 발광 효과만, 실제 빛은 아님
3. **Physics Material**: Friction은 Minimum, Bounciness는 Maximum
4. **Rigidbody 코드**: FixedUpdate()에 작성
5. **AddForce**: 속도가 계속 누적됨
6. **AddTorque**: Vec이 회전축이 됨 (방향 주의)
7. **재질 접근**: MeshRenderer를 통해서만 가능
8. **UI 이미지**: Sprite로 설정 필수
9. **Image Type**: Simple/Sliced/Tiled 용도에 맞게 선택
10. **폰트 라이센스**: 상업용 게임 제작 시 반드시 확인
11. **버튼 클릭**: 누르기 + 떼기 = 한 세트
12. **스크린 좌표**: 마우스 위치도 스크린 좌표계