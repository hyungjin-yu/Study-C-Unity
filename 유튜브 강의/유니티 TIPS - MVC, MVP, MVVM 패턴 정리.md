#Unity #DesignPattern #MVC #MVP #MVVM #GameDev #CSharp #Programming #ObsidianNotes

### 📌 영상 정보

- **제목:** [유니티 TIPS] UI에 걸맞는 MVC, MVP, MVVM 패턴
- **채널:** Unity Korea
- **길이:** 약 38분
- **발행일:** 2024년 9월 11일
- **주제:** UI 개발에 적합한 아키텍처 패턴 비교 및 적용

---

### 🎯 핵심 개념

- **MVC (Model-View-Controller):**
- 가장 오래된 UI 아키텍처 패턴.
- Model(데이터), View(화면), Controller(로직)으로 분리.
- 단점: Controller와 View 간 결합도가 높아질 수 있음.

- **MVP (Model-View-Presenter):**
- Controller 대신 Presenter가 View와 Model을 연결.
- View는 단순히 UI 표시만 담당, Presenter가 로직을 관리.
- 테스트 용이성 증가.

- **MVVM (Model-View-ViewModel):**
- ViewModel이 데이터와 로직을 관리하며 View와 바인딩.
- View는 단순히 ViewModel을 반영.
- Unity UI Toolkit과 잘 맞음 → 데이터 바인딩 기능 활용 가능.

---

### 🛠️ 구현 방식

1. **MVC:**
- Controller가 사용자 입력 처리 → Model 업데이트 → View 반영.

2. **MVP:**
- Presenter가 Model과 View를 중재.
- View는 단순히 Presenter의 지시를 따름.

3. **MVVM:**
- ViewModel이 상태와 로직을 관리.
- View는 ViewModel과 바인딩되어 자동으로 업데이트.

---

### 🎮 Unity 활용 예시

- **MVC:**
- 간단한 UI 구조 (예: 메뉴 화면).

- **MVP:**
- 테스트가 중요한 UI 로직 (예: 설정 화면).

- **MVVM:**
- 복잡한 UI 시스템 (예: 인벤토리, 데이터 기반 UI).
- Unity UI Toolkit과 결합해 데이터 바인딩 구현.

---

### ✅ 장점

- **MVC:** 구조적 분리, 기본적인 UI 관리에 적합.
- **MVP:** 테스트 용이, View와 로직 분리 강화.
- **MVVM:** 데이터 바인딩으로 UI 자동 업데이트, 대규모 UI 시스템에 적합.

---

### ⚠️ 주의점

- MVC: Controller와 View 결합도가 높아질 수 있음.
- MVP: Presenter가 비대해질 수 있음.
- MVVM: 학습 곡선이 높고, 단순 UI에는 과도한 구조가 될 수 있음.

---

### 📚 결론

- **MVC → MVP → MVVM으로 발전하며 UI 관리가 점점 더 유연해짐**
- Unity UI Toolkit은 MVVM 패턴과 특히 잘 맞음
- 프로젝트 규모와 UI 복잡도에 따라 적절한 패턴을 선택하는 것이 중요