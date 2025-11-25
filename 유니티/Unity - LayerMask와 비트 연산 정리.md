#Unity #LayerMask #비트연산 #게임개발 #프로그래밍
## LayerMask란?
특정 Layer의 Object에만 선택적으로 물리 연산을 적용하기 위한 시스템

## 비트 연산을 사용하는 이유
- 여러 Layer를 동시에 선택할 수 있음
- 32개의 비트를 체크 박스처럼 사용하는 비트 플래그 방식
- 성능 최적화: 불필요한 Object 검사 제외

## Layer 번호와 LayerMask 값
- 0번 Layer: 1 (0001)
- 1번 Layer: 2 (0010)
- 2번 Layer: 4 (0100)
- 3번 Layer: 8 (1000)
- 비트 시프트 공식: 1 << Layer 번호

## 사용 방법
### 단일 Layer
int layerMask = 1 << LayerMask.NameToLayer("Enemy");

### 여러 Layer 조합
int mask = (1 << LayerMask.NameToLayer("Enemy")) | (1 << LayerMask.NameToLayer("Boss"));

## Layer 관리 팁
- Unity Inspector: Edit → Project Settings → Tags and Layers
- Layer 번호로 이름 찾기: LayerMask.LayerToName(번호)
- Layer 이름으로 번호 찾기: LayerMask.NameToLayer("이름")
- Unity는 총 32개 Layer 지원 (0-31번)

## 핵심 개념
- Layer 이름: 가독성과 유지보수성 향상
- 비트 연산: 여러 Layer 동시 선택
- LayerMask: 필터링 도구, Layer 자체가 아님