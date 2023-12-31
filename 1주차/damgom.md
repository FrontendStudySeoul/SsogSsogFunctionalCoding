## 함수형 프로그래밍
### 정의
부수효과 없이 순수 함수만을 사용하는 프로그래밍

### 실제 함수형 프로그래밍
- 부수효과는 실제로 필요하다
- 순수하지 않은 함수도 사용한다.

=> 부수효과를 잘 다루기 위한 방법을 알아햔다.

---

### 액션, 계산, 데이터
> 함수형 프로그래머는 액션, 계산, 데이터를 구분한다.

#### 정의
- `액션`: 시점, 횟수에 의존하는 비순수함수
  - 부수효과를 가지고 있다. 
  - UI 이벤트, 서버 통신, 상태 변경
  - 액션은 계산으로 변경하는 것이 다루기 쉽다. 
- `계산`: 순수함수
  - 부수효과를 가지고 있지 않다. 여러번 실행해도 같은 결과를 반환한다. 
- `데이터`: 값

=> 액션은 최대한 계산으로, 계산은 최대한 데이터로

### 일급 추상
- 함수에 함수를 넘겨 더 많은 함수를 재사용한다.
- 반응형 아키텍처, 어니언 아키텍처

---

## 현실에서 함수형 사고
### 액션, 계산, 데이터: 변경 가능성에 따라 코드 나누기
- 유지 보수를 쉽게 하기 위해 변경 가능성에 따라 계층을 나눈다.
#### 계층형 설계 원칙
```
비즈니스 규칙 (변경 ⬆️)
----------
도메인 규칙
----------
기술 스택 (변경 ⬇️)
```

### 일급 추상
#### 분산시스템에서 발생할 수 있는 문제점
> 분산시스템에서 타임라인을 사용하면 액션을 시각화할 수 있다.
> 액션은 실행 시점에 의존하기 때문에 순서가 중요하다.

1. 각 타임라인이 실행 순서가 보장되지 않는다.
   -> 순서를 맞춰야한다.
2. 액션이 걸리는 시간을 보장할 수 없다.
   -> 각 타임라인은 순서와 관계없이 만들어야한다. 



#### 해결책: 타임라인 커팅
타임라인 커팅: 여러 타임라인이 동시에 실행될 때 순서를 맞추는 방법
- 고차 동작으로 구현
- 각 타임라인은 독립적으로 동작하고 작업이 완료되면 다른 타임라인이 끝나기를 기다린다.
