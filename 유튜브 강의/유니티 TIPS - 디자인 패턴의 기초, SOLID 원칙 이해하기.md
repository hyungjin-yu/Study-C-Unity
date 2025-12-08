https://www.youtube.com/watch?v=J6F8plGUqv8&list=PL412Ym60h6uufFJlxsKcYkOtRu-PIsLuA

#Unity #SOLID #DesignPatterns #OOP #CleanCode #GameDevelopment #ProgrammingPrinciples

## 📌 영상 타임라인
1. **00:00** GoF와 SOLID 원칙 소개  
2. **02:19** S. 단일 책임 원칙  
3. **05:34** O. 개방-폐쇄 원칙  
4. **09:20** L. 리스코프 치환 원칙  
5. **16:57** I. 인터페이스 분리 원칙  
6. **19:20** D. 의존 역전 원칙  
7. **25:25** 추상 클래스 vs 인터페이스  
8. **31:51** 유니티의 SOLID 원칙  
9. **33:24** 마무리  

---

## 🎯 SOLID 원칙 요약

### S. Single Responsibility Principle (단일 책임 원칙)
- 클래스는 **하나의 책임만** 가져야 한다.  
- Unity 예시: `PlayerMovement` → 이동만 담당, `PlayerAttack` → 공격만 담당.  
- 장점: 가독성 ↑, 유지보수성 ↑.  

---

### O. Open/Closed Principle (개방-폐쇄 원칙)
- **확장에는 열려 있고, 수정에는 닫혀 있어야 한다.**  
- Unity 예시: 무기 시스템 확장 시 기존 코드 수정 없이 상속/인터페이스로 추가.  
- 장점: 새로운 기능 추가 시 안정성 확보.  

---

### L. Liskov Substitution Principle (리스코프 치환 원칙)
- **자식 클래스는 부모 클래스의 행위를 대체할 수 있어야 한다.**  
- Unity 예시: `EnemyBase` → `ZombieEnemy`, `RobotEnemy`가 동일하게 동작 가능해야 함.  
- 잘못된 상속은 버그와 유지보수 문제를 유발.  

---

### I. Interface Segregation Principle (인터페이스 분리 원칙)
- **큰 인터페이스를 작은 인터페이스로 분리**해야 한다.  
- Unity 예시: `IAttack`, `IMove`, `IHeal` 등으로 나누어 필요 없는 기능 강제 방지.  
- 장점: 유연한 구조, 불필요한 의존성 제거.  

---

### D. Dependency Inversion Principle (의존 역전 원칙)
- **상위 모듈은 하위 모듈에 의존하지 않고, 추상화에 의존해야 한다.**  
- Unity 예시: `GameManager` → `Player` 직접 참조 X, `IPlayer` 인터페이스를 통해 접근.  
- 장점: 결합도 ↓, 테스트 용이성 ↑.  

---

## 📝 추상 클래스 vs 인터페이스
- **추상 클래스**: 공통된 기본 구현 제공 가능.  
- **인터페이스**: 기능 계약만 정의, 다중 구현 가능.  
- Unity에서는 상황에 따라 선택적으로 사용.  

---

## 🎮 유니티에서 SOLID 적용
- **MonoBehaviour 스크립트 분리**: 하나의 스크립트에 모든 기능 몰아넣지 않기.  
- **인터페이스 활용**: 다양한 오브젝트에 공통 기능 적용.  
- **DI 프레임워크(Zenject 등)**: DIP 구현에 도움.  
- **테스트 코드 작성**: SOLID 원칙 적용 시 단위 테스트가 쉬워짐.  

---

## ✅ 결론
- SOLID 원칙은 **디자인 패턴 이해의 기초**.  
- Unity 개발에서 구조적 사고를 돕고, 장기적으로 **유지보수성과 확장성**을 보장.  
- 팀 프로젝트에서 코드 품질을 유지하는 핵심 철학.