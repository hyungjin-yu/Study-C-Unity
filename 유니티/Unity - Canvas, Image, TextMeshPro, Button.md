#Unity #UI #Canvas #TextMeshPro #Button #EventSystem

## 📑 목차

1. Canvas 설정
2. Image 배치와 크기 조절
3. Panel과 Image 구조
4. TextMeshPro - UGUI
5. Button 이벤트 시스템
6. EventSystem 디버깅
7. 실전 예시
8. 핵심 포인트

## 📌 간단 요약

Unity UI 시스템의 Canvas 설정, Image/Panel 배치, TextMeshPro 사용법, 그리고 Button 이벤트 처리 방법

---

## 1. Canvas 설정

### Event Camera 설정

**중요 원칙:**

> Canvas의 Event Camera는 사용하지 않음

```csharp
// Canvas 컴포넌트 설정
Canvas canvas = GetComponent<Canvas>();

// Render Mode 설정
canvas.renderMode = RenderMode.ScreenSpaceOverlay;

// Event Camera는 None으로 설정 (사용 안함)
canvas.worldCamera = null;
```

**이유:**

- Screen Space - Overlay 모드에서는 Event Camera 불필요
- EventSystem이 자동으로 UI 이벤트 처리
- Event Camera 설정 시 불필요한 오버헤드 발생

---

### Canvas Render Mode

````csharp
public enum RenderMode
{
    ScreenSpaceOverlay,    // 화면 최상단, Event Camera 불필요 ✅
    ScreenSpaceCamera,     // 카메라 기준, Event Camera 필요
    WorldSpace            // 월드 공간, Event Camera 필요
}
````

**Screen Space - Overlay (권장):**
```
특징:
- 항상 화면 최상단에 표시
- 해상도에 자동 대응
- Event Camera 불필요
- UI 전용으로 최적화

용도:
- 게임 HUD
- 메뉴 화면
- 일반적인 UI
````

---

## 2. Image 크기 및 위치 조절

### Rect Transform 컴포넌트

**Image의 크기와 위치는 Rect Transform으로 조절 가능**

```csharp
public class ImageController : MonoBehaviour
{
    private RectTransform rectTransform;
    
    void Start()
    {
        rectTransform = GetComponent<RectTransform>();
        
        // 위치 조절
        rectTransform.anchoredPosition = new Vector2(100, 50);
        
        // 크기 조절
        rectTransform.sizeDelta = new Vector2(200, 100);
        
        // 회전
        rectTransform.rotation = Quaternion.Euler(0, 0, 45);
        
        // 스케일
        rectTransform.localScale = new Vector3(1.5f, 1.5f, 1);
    }
}
```

---

### Anchor 시스템

````csharp
public class AnchorExample : MonoBehaviour
{
    void Start()
    {
        RectTransform rt = GetComponent<RectTransform>();
        
        // 중앙 배치
        rt.anchorMin = new Vector2(0.5f, 0.5f);
        rt.anchorMax = new Vector2(0.5f, 0.5f);
        rt.anchoredPosition = Vector2.zero;
        
        // 좌측 상단
        rt.anchorMin = new Vector2(0, 1);
        rt.anchorMax = new Vector2(0, 1);
        rt.anchoredPosition = new Vector2(10, -10);
        
        // 우측 하단
        rt.anchorMin = new Vector2(1, 0);
        rt.anchorMax = new Vector2(1, 0);
        rt.anchoredPosition = new Vector2(-10, 10);
    }
}
````

**Anchor 프리셋:**
```
좌상단: (0, 1)
중앙상단: (0.5, 1)
우상단: (1, 1)

좌중앙: (0, 0.5)
정중앙: (0.5, 0.5)
우중앙: (1, 0.5)

좌하단: (0, 0)
중앙하단: (0.5, 0)
우하단: (1, 0)
````

---

### 동적 크기 조절 예시

````csharp
public class DynamicImageSize : MonoBehaviour
{
    public RectTransform imageRect;
    public float duration = 1f;
    
    // 크기 애니메이션
    public void AnimateSize(Vector2 targetSize)
    {
        StartCoroutine(AnimateSizeCoroutine(targetSize));
    }
    
    IEnumerator AnimateSizeCoroutine(Vector2 targetSize)
    {
        Vector2 startSize = imageRect.sizeDelta;
        float elapsed = 0f;
        
        while (elapsed < duration)
        {
            elapsed += Time.deltaTime;
            float t = elapsed / duration;
            
            imageRect.sizeDelta = Vector2.Lerp(startSize, targetSize, t);
            
            yield return null;
        }
        
        imageRect.sizeDelta = targetSize;
    }
    
    // 위치 애니메이션
    public void AnimatePosition(Vector2 targetPosition)
    {
        StartCoroutine(AnimatePositionCoroutine(targetPosition));
    }
    
    IEnumerator AnimatePositionCoroutine(Vector2 targetPosition)
    {
        Vector2 startPosition = imageRect.anchoredPosition;
        float elapsed = 0f;
        
        while (elapsed < duration)
        {
            elapsed += Time.deltaTime;
            float t = elapsed / duration;
            
            imageRect.anchoredPosition = Vector2.Lerp(startPosition, targetPosition, t);
            
            yield return null;
        }
        
        imageRect.anchoredPosition = targetPosition;
    }
}
````

---

## 3. Panel과 Image 구조

### 올바른 UI 구조

**권장 구조:**
```
Canvas
└── Panel (배경, 컨테이너)
    ├── Image (아이콘, 그림)
    ├── Image (캐릭터 초상화)
    └── Image (장식 요소)
````

**생성 순서:**

1. UI → Panel 생성
2. Panel을 선택한 상태에서
3. UI → Image 생성 (Panel의 자식으로 자동 생성)

---

### Panel의 역할

````csharp
public class UIPanel : MonoBehaviour
{
    public Image panelBackground;
    public List<Image> childImages;
    
    // Panel 전체 표시/숨김
    public void Show()
    {
        gameObject.SetActive(true);
    }
    
    public void Hide()
    {
        gameObject.SetActive(false);
    }
    
    // Panel 페이드 효과
    public void FadeIn(float duration)
    {
        StartCoroutine(FadeCoroutine(0, 1, duration));
    }
    
    public void FadeOut(float duration)
    {
        StartCoroutine(FadeCoroutine(1, 0, duration));
    }
    
    IEnumerator FadeCoroutine(float startAlpha, float endAlpha, float duration)
    {
        CanvasGroup canvasGroup = GetComponent<CanvasGroup>();
        if (canvasGroup == null)
            canvasGroup = gameObject.AddComponent<CanvasGroup>();
        
        float elapsed = 0f;
        
        while (elapsed < duration)
        {
            elapsed += Time.deltaTime;
            float t = elapsed / duration;
            
            canvasGroup.alpha = Mathf.Lerp(startAlpha, endAlpha, t);
            
            yield return null;
        }
        
        canvasGroup.alpha = endAlpha;
        
        if (endAlpha == 0)
        {
            gameObject.SetActive(false);
        }
    }
}
````

---

### 실전 예시: 인벤토리 UI
```
Canvas
└── InventoryPanel (Panel)
    ├── BackgroundImage (Image) - 반투명 검은 배경
    ├── TitleText (TextMeshProUGUI) - "인벤토리"
    ├── CloseButton (Button)
    └── ItemsContainer (Grid Layout Group)
        ├── ItemSlot1 (Panel)
        │   ├── ItemIcon (Image)
        │   └── ItemCount (TextMeshProUGUI)
        ├── ItemSlot2 (Panel)
        │   ├── ItemIcon (Image)
        │   └── ItemCount (TextMeshProUGUI)
        └── ...
````

**코드 구현:**

````csharp
public class InventoryUI : MonoBehaviour
{
    public GameObject inventoryPanel;
    public Transform itemsContainer;
    public GameObject itemSlotPrefab;
    
    public void OpenInventory()
    {
        inventoryPanel.SetActive(true);
        UpdateInventoryDisplay();
    }
    
    public void CloseInventory()
    {
        inventoryPanel.SetActive(false);
    }
    
    void UpdateInventoryDisplay()
    {
        // 기존 슬롯 제거
        foreach (Transform child in itemsContainer)
        {
            Destroy(child.gameObject);
        }
        
        // 아이템 표시
        List<Item> items = GameManager.Instance.inventory;
        
        foreach (Item item in items)
        {
            GameObject slot = Instantiate(itemSlotPrefab, itemsContainer);
            
            Image icon = slot.transform.Find("ItemIcon").GetComponent<Image>();
            icon.sprite = item.icon;
            
            TextMeshProUGUI count = slot.transform.Find("ItemCount").GetComponent<TextMeshProUGUI>();
            count.text = item.count.ToString();
        }
    }
}
````

---

## 4. TextMeshPro - UGUI

### 중요성

**TextMeshPro UGUI는 Unity UI에서 가장 중요한 텍스트 컴포넌트**

**기존 Text vs TextMeshPro:**

| 항목 | Text (Legacy) | TextMeshPro UGUI |
|------|---------------|------------------|
| 품질 | 낮음 | 매우 높음 ✅ |
| 성능 | 보통 | 우수 ✅ |
| 기능 | 제한적 | 풍부 ✅ |
| 권장 | ❌ | ✅ |

---

### TextMeshPro 생성
```
Hierarchy → UI → Text - TextMeshPro

최초 생성 시:
"Import TMP Essentials" 버튼 클릭 (필수)
"Import TMP Examples & Extras" (선택)
````

---

### 기본 사용법

```csharp
using TMPro;
using UnityEngine;

public class TextController : MonoBehaviour
{
    public TextMeshProUGUI scoreText;
    public TextMeshProUGUI timerText;
    public TextMeshProUGUI messageText;
    
    void Start()
    {
        // 텍스트 설정
        scoreText.text = "Score: 0";
        
        // 색상 변경
        scoreText.color = Color.yellow;
        
        // 폰트 크기
        scoreText.fontSize = 48;
        
        // 정렬
        scoreText.alignment = TextAlignmentOptions.Center;
    }
    
    public void UpdateScore(int score)
    {
        scoreText.text = $"Score: {score}";
    }
    
    public void UpdateTimer(float time)
    {
        int minutes = (int)(time / 60);
        int seconds = (int)(time % 60);
        timerText.text = $"{minutes:00}:{seconds:00}";
    }
    
    public void ShowMessage(string message, float duration)
    {
        messageText.text = message;
        messageText.gameObject.SetActive(true);
        
        Invoke("HideMessage", duration);
    }
    
    void HideMessage()
    {
        messageText.gameObject.SetActive(false);
    }
}
```

---

### TextMeshPro 고급 기능

```csharp
public class AdvancedTextEffects : MonoBehaviour
{
    public TextMeshProUGUI text;
    
    void Start()
    {
        // Rich Text 태그
        text.text = "<color=red>빨간</color> <b>굵은</b> <i>기울임</i>";
        
        // 그라데이션
        text.enableVertexGradient = true;
        text.colorGradient = new VertexGradient(
            Color.yellow,  // 상단 좌
            Color.yellow,  // 상단 우
            Color.red,     // 하단 좌
            Color.red      // 하단 우
        );
        
        // 외곽선
        text.outlineWidth = 0.2f;
        text.outlineColor = Color.black;
        
        // 그림자
        text.fontMaterial.EnableKeyword("UNDERLAY_ON");
    }
    
    // 타이핑 효과
    public IEnumerator TypeText(string fullText, float delay)
    {
        text.text = "";
        
        foreach (char c in fullText)
        {
            text.text += c;
            yield return new WaitForSeconds(delay);
        }
    }
    
    // 깜빡임 효과
    public IEnumerator BlinkText(int count)
    {
        for (int i = 0; i < count; i++)
        {
            text.gameObject.SetActive(false);
            yield return new WaitForSeconds(0.5f);
            
            text.gameObject.SetActive(true);
            yield return new WaitForSeconds(0.5f);
        }
    }
}
```

---

### 동적 텍스트 업데이트 최적화

```csharp
public class OptimizedTextUpdate : MonoBehaviour
{
    public TextMeshProUGUI scoreText;
    private int currentScore = 0;
    private int displayScore = 0;
    
    void Update()
    {
        // 부드러운 점수 증가 효과
        if (displayScore < currentScore)
        {
            displayScore += Mathf.CeilToInt((currentScore - displayScore) * 0.1f);
            scoreText.text = $"Score: {displayScore}";
        }
    }
    
    public void AddScore(int amount)
    {
        currentScore += amount;
    }
}
```

---

## 5. Button 이벤트 시스템

### onClick.AddListener()

**버튼 클릭 시 함수를 실행하는 방법**

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ButtonController : MonoBehaviour
{
    public Button startButton;
    public Button pauseButton;
    public Button quitButton;
    
    void Start()
    {
        // onClick에 함수 연결
        startButton.onClick.AddListener(OnStartButtonClicked);
        pauseButton.onClick.AddListener(OnPauseButtonClicked);
        quitButton.onClick.AddListener(OnQuitButtonClicked);
    }
    
    void OnStartButtonClicked()
    {
        Debug.Log("게임 시작!");
        GameManager.Instance.StartGame();
    }
    
    void OnPauseButtonClicked()
    {
        Debug.Log("일시 정지!");
        GameManager.Instance.PauseGame();
    }
    
    void OnQuitButtonClicked()
    {
        Debug.Log("게임 종료!");
        Application.Quit();
    }
    
    void OnDestroy()
    {
        // 리스너 제거 (메모리 누수 방지)
        if (startButton != null)
            startButton.onClick.RemoveListener(OnStartButtonClicked);
        if (pauseButton != null)
            pauseButton.onClick.RemoveListener(OnPauseButtonClicked);
        if (quitButton != null)
            quitButton.onClick.RemoveListener(OnQuitButtonClicked);
    }
}
```

---

### 파라미터가 있는 함수 연결

```csharp
public class ButtonWithParameters : MonoBehaviour
{
    public Button[] levelButtons;
    
    void Start()
    {
        // 레벨 버튼 동적 생성 및 연결
        for (int i = 0; i < levelButtons.Length; i++)
        {
            int levelIndex = i; // 클로저 문제 해결
            
            levelButtons[i].onClick.AddListener(() => LoadLevel(levelIndex));
        }
    }
    
    void LoadLevel(int levelIndex)
    {
        Debug.Log($"레벨 {levelIndex} 로드!");
        SceneManager.LoadScene($"Level{levelIndex}");
    }
}
```

**❌ 잘못된 방법 (클로저 문제):**

```csharp
// 이렇게 하면 안됨!
for (int i = 0; i < levelButtons.Length; i++)
{
    // i를 직접 사용하면 마지막 값만 캡처됨
    levelButtons[i].onClick.AddListener(() => LoadLevel(i));
}
```

---

### 람다식 활용

```csharp
public class LambdaButtonExample : MonoBehaviour
{
    public Button button;
    
    void Start()
    {
        // 간단한 동작은 람다식으로
        button.onClick.AddListener(() => 
        {
            Debug.Log("버튼 클릭!");
            PlaySound("ButtonClick");
            ShowEffect();
        });
    }
    
    void PlaySound(string soundName)
    {
        // 사운드 재생
    }
    
    void ShowEffect()
    {
        // 이펙트 표시
    }
}
```

---

### Button Interactable

```csharp
public class ButtonState : MonoBehaviour
{
    public Button button;
    
    // 버튼 활성화/비활성화
    public void SetButtonEnabled(bool enabled)
    {
        button.interactable = enabled;
    }
    
    // 조건부 활성화
    void Update()
    {
        // 골드가 충분하면 버튼 활성화
        int gold = GameManager.Instance.gold;
        button.interactable = (gold >= 100);
    }
}
```

---

## 6. EventSystem 디버깅

### 버튼이 안 눌릴 때 확인 사항

**핵심 원칙:**

> 버튼이 안 눌린다면, EventSystem에서 내가 의도한 버튼이 맞는지 확인해야 함

---

### 1. EventSystem 존재 확인

```csharp
public class EventSystemChecker : MonoBehaviour
{
    void Start()
    {
        // EventSystem이 있는지 확인
        EventSystem eventSystem = FindFirstObjectByType<EventSystem>();
        
        if (eventSystem == null)
        {
            Debug.LogError("EventSystem이 없습니다!");
            // 자동 생성
            GameObject obj = new GameObject("EventSystem");
            obj.AddComponent<EventSystem>();
            obj.AddComponent<StandaloneInputModule>();
        }
        else
        {
            Debug.Log("EventSystem 정상");
        }
    }
}
```

---

### 2. Raycast Target 확인

````csharp
public class RaycastTargetChecker : MonoBehaviour
{
    void Start()
    {
        Image image = GetComponent<Image>();
        
        if (image != null)
        {
            Debug.Log($"Raycast Target: {image.raycastTarget}");
            
            // 클릭 가능하게 설정
            image.raycastTarget = true;
        }
    }
}
````

**문제 상황:**
```
버튼의 Image 컴포넌트에서
Raycast Target이 체크 해제되어 있으면
클릭이 감지되지 않음!
````

---

### 3. Canvas Graphic Raycaster 확인

```csharp
public class CanvasChecker : MonoBehaviour
{
    void Start()
    {
        Canvas canvas = GetComponentInParent<Canvas>();
        
        if (canvas != null)
        {
            GraphicRaycaster raycaster = canvas.GetComponent<GraphicRaycaster>();
            
            if (raycaster == null)
            {
                Debug.LogError("GraphicRaycaster가 없습니다!");
                canvas.gameObject.AddComponent<GraphicRaycaster>();
            }
            else
            {
                Debug.Log("GraphicRaycaster 정상");
            }
        }
    }
}
```

---

### 4. 현재 클릭된 UI 확인

```csharp
using UnityEngine.EventSystems;

public class UIDebugger : MonoBehaviour
{
    void Update()
    {
        if (Input.GetMouseButtonDown(0))
        {
            // 현재 마우스 위치의 UI 확인
            PointerEventData pointerData = new PointerEventData(EventSystem.current)
            {
                position = Input.mousePosition
            };
            
            List<RaycastResult> results = new List<RaycastResult>();
            EventSystem.current.RaycastAll(pointerData, results);
            
            Debug.Log($"클릭된 UI 개수: {results.Count}");
            
            foreach (RaycastResult result in results)
            {
                Debug.Log($"- {result.gameObject.name}");
            }
        }
    }
}
```

---

### 5. Button 컴포넌트 확인

```csharp
public class ButtonDebugger : MonoBehaviour
{
    public Button button;
    
    void Start()
    {
        if (button == null)
        {
            Debug.LogError("Button이 할당되지 않았습니다!");
            return;
        }
        
        Debug.Log($"Button: {button.name}");
        Debug.Log($"Interactable: {button.interactable}");
        Debug.Log($"Listener Count: {button.onClick.GetPersistentEventCount()}");
        
        // 테스트 클릭
        button.onClick.AddListener(() => Debug.Log("버튼 클릭됨!"));
    }
}
```

---

### 6. CanvasGroup 차단 확인

```csharp
public class CanvasGroupChecker : MonoBehaviour
{
    void Start()
    {
        CanvasGroup[] canvasGroups = GetComponentsInParent<CanvasGroup>();
        
        foreach (CanvasGroup group in canvasGroups)
        {
            if (!group.interactable)
            {
                Debug.LogWarning($"{group.name}의 CanvasGroup이 비활성화되어 있습니다!");
            }
            
            if (group.blocksRaycasts == false)
            {
                Debug.LogWarning($"{group.name}의 Raycast가 차단되어 있습니다!");
            }
        }
    }
}
```

---

## 7. 실전 종합 예시

### 완전한 메뉴 UI 시스템

```csharp
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.SceneManagement;
using TMPro;
using System.Collections;

public class MainMenuUI : MonoBehaviour
{
    [Header("Panels")]
    public GameObject mainPanel;
    public GameObject settingsPanel;
    public GameObject creditsPanel;
    
    [Header("Buttons")]
    public Button startButton;
    public Button settingsButton;
    public Button creditsButton;
    public Button quitButton;
    public Button backButton;
    
    [Header("Text")]
    public TextMeshProUGUI titleText;
    public TextMeshProUGUI versionText;
    
    [Header("Settings")]
    public Slider volumeSlider;
    public Toggle fullscreenToggle;
    
    void Start()
    {
        // 초기 설정
        ShowMainPanel();
        
        // 버튼 이벤트 연결
        startButton.onClick.AddListener(OnStartGame);
        settingsButton.onClick.AddListener(OnShowSettings);
        creditsButton.onClick.AddListener(OnShowCredits);
        quitButton.onClick.AddListener(OnQuitGame);
        backButton.onClick.AddListener(OnBackToMain);
        
        // 설정 이벤트 연결
        volumeSlider.onValueChanged.AddListener(OnVolumeChanged);
        fullscreenToggle.onValueChanged.AddListener(OnFullscreenChanged);
        
        // 타이틀 애니메이션
        StartCoroutine(AnimateTitle());
        
        // 버전 표시
        versionText.text = $"v{Application.version}";
    }
    
    void ShowMainPanel()
    {
        mainPanel.SetActive(true);
        settingsPanel.SetActive(false);
        creditsPanel.SetActive(false);
    }
    
    void OnStartGame()
    {
        Debug.Log("게임 시작!");
        // 로딩 화면 표시
        StartCoroutine(LoadGameScene());
    }
    
    IEnumerator LoadGameScene()
    {
        // 페이드 아웃
        yield return StartCoroutine(FadeOut());
        
        // Scene 로드
        AsyncOperation asyncLoad = SceneManager.LoadSceneAsync("GamePlay");
        
        while (!asyncLoad.isDone)
        {
            // 로딩 진행률 표시 가능
            float progress = asyncLoad.progress;
            yield return null;
        }
    }
    
    void OnShowSettings()
    {
        mainPanel.SetActive(false);
        settingsPanel.SetActive(true);
        
        // 현재 설정 로드
        volumeSlider.value = PlayerPrefs.GetFloat("Volume", 1f);
        fullscreenToggle.isOn = Screen.fullScreen;
    }
    
    void OnShowCredits()
    {
        mainPanel.SetActive(false);
        creditsPanel.SetActive(true);
    }
    
    void OnBackToMain()
    {
        ShowMainPanel();
    }
    
    void OnQuitGame()
    {
        Debug.Log("게임 종료!");
        
#if UNITY_EDITOR
        UnityEditor.EditorApplication.isPlaying = false;
#else
        Application.Quit();
#endif
    }
    
    void OnVolumeChanged(float value)
    {
        AudioListener.volume = value;
        PlayerPrefs.SetFloat("Volume", value);
    }
    
    void OnFullscreenChanged(bool isFullscreen)
    {
        Screen.fullScreen = isFullscreen;
    }
    
    IEnumerator AnimateTitle()
    {
        // 타이틀 텍스트 타이핑 효과
        string fullTitle = "AWESOME GAME";
        titleText.text = "";
        
        foreach (char c in fullTitle)
        {
            titleText.text += c;
            yield return new WaitForSeconds(0.1f);
        }
        
        // 깜빡임 효과
        while (true)
        {
            titleText.color = Color.Lerp(Color.white, Color.yellow, Mathf.PingPong(Time.time, 1));
            yield return null;
        }
    }
    
    IEnumerator FadeOut()
    {
        CanvasGroup canvasGroup = GetComponent<CanvasGroup>();
        if (canvasGroup == null)
            canvasGroup = gameObject.AddComponent<CanvasGroup>();
        
        float duration = 1f;
        float elapsed = 0f;
        
        while (elapsed < duration)
        {
            elapsed += Time.deltaTime;
            canvasGroup.alpha = 1 - (elapsed / duration);
            yield return null;
        }
        
        canvasGroup.alpha = 0;
    }
    
    void OnDestroy()
    {
        // 모든 리스너 제거
        startButton.onClick.RemoveAllListeners();
        settingsButton.onClick.RemoveAllListeners();
        creditsButton.onClick.RemoveAllListeners();
        quitButton.onClick.RemoveAllListeners();
        backButton.onClick.RemoveAllListeners();
        
        volumeSlider.onValueChanged.RemoveAllListeners();
        fullscreenToggle.onValueChanged.RemoveAllListeners();
    }
}
```

---

### 동적 버튼 생성 시스템
```csharp
public class DynamicButtonCreator : MonoBehaviour
{
    public Transform buttonContainer;
    public GameObject buttonPrefab;
    
    void Start()
    {
        // 레벨 선택 버튼 동적 생성
        CreateLevelButtons(10);
    }
    
    void CreateLevelButtons(int levelCount)
    {
        for (int i = 1; i <= levelCount; i++)
        {
            // 버튼 생성
            GameObject buttonObj = Instantiate(buttonPrefab, buttonContainer);
            
            // 버튼 텍스트 설정
            TextMeshProUGUI buttonText = buttonObj.GetComponentInChildren<TextMeshProUGUI>();
            buttonText.text = $"Level {i}";
            
            // 버튼 이벤트 연결
            Button button = buttonObj.GetComponent<Button>();
            int levelIndex = i; // 클로저 문제 해결
            
            button.onClick.AddListener(() => OnLevelButtonClicked(levelIndex));
            
            // 잠금 상태 확인
            int unlockedLevel = PlayerPrefs.GetInt("UnlockedLevel", 1);
            button.interactable = (levelIndex <= unlockedLevel);
            
            if (!button.interactable)
            {
                // 잠금 아이콘 표시
                Transform lockIcon = buttonObj.transform.Find("LockIcon");
                if (lockIcon != null)
                    lockIcon.gameObject.SetActive(true);
            }
        }
    }
```