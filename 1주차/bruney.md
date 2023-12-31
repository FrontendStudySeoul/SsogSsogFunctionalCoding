> Chapter 1: 쏙쏙 들어오는 함수형 코딩에 오신 것을 환영합니다.

# 함수형 프로그래밍이란?
> 이 책은 학문적인 내용보다는 실무에서 쓸 수 있는 내용을 다룸.

### 함수형 프로그래밍
1. 수학 함수를 사용하고 부수 효과를 피하는 것이 특징
2. 부수 효과 없이 순수 함수만 사용하는 프로그래밍 스타일

### 부수 효과
함수가 **리턴값 이외에 하는 모든 일**. 부수 효과는 함수를 부를 때마다 발생하기 때문에 문제가 될 수 있다. 즉, 의도치 않은 부분이 발생할 수 있다.

### 순수 함수
**인자에만 의존하고 부수 효과가 없는 함수.** 같은 인자를 넣으면 항상 같은 결과를 돌려주는 순수 함수를 **수학 함수**라고 볼 수 있음.

실제 함수형 프로그래머는 부수 효과와 숨수하지 않은 함수를 사용함..

## 실용적인 측면에서 함수형 프로그래밍 정의의 문제점
1. 부수 효과는 필요함: 정의에 따르면 피해야 하지만 부수 효과는 SW를 실행하는 이유.
2. 함수형 프로그래밍은 부수 효과를 잘 다룰 수 있다: 정의에는 **순수 함수만 쓰라**는 것처럼 되어 있지만, 함수형 프로그래머는 순수하지 않은 함수도 사용하고 순수하지 않은 함수를 잘 다룰 수 있는 기술이 많이 있음.
3. 함수형 프로그래밍은 실용적이다: 수학적이라 실제로 사용하지 않는 것처럼 느껴지지만 실제로 잘 만들어진 SW가 많이 있음. 정의는 함수형 프로그래밍을 시작하려는 사람에게 혼란을 준다.

## 함수형 프로그래밍 정의가 혼란스러운 관리자
정의에 따르면 부수 효과를 쓰지 말라는데 우리가 개발하려는 서비스가 부수 효과인 경우

## 함수형 프로그래밍을 학문적 지식이 아닌 기술과 개념으로 보기
함수형 프로그래밍은 많은 사람이 서로 다른 의미로 생각함. 학술적, 실용적으로도 많은 내용이 있음. 함수형 프로그래밍은 OOP, PP 모두 사용 가능.

## 액션과 계산, 데이터
**액션, 계산, 데이터를 구분하는 것은 함수형 프로그래밍의 기본 개념.**
- 액션: 호출하는 시점과 횟수에 의존하기 때문에 호출할 때, 조심해야 한다.
- 계산: 실행 가능, 실행하기 전까지 **어떻게 동작할지 알 수 없음.**
- 데이터: 실행 불가능, 정적이고 보이는 그대로

각각 장단점을 가지고 있기 때문에 잘 알고 적절하게 쓰는 것이 좋음. **가장 쓰기 좋은 것은 데이터.**

**함수형 프로그래밍은 분산 시스템에 잘 어울림.** 시간에 따라 바뀌는 값을 모델링할 때 동작 방법을 이해하는 것은 중요하지만 쉽지 않음. 실행 시점이나 횟수에 의존하는 코드를 없애면, 코드를 더 쉽게 이해할 수 있고 심각한 버그를 막을 수 있음.

- 데이터, 계산은 실행 시점, 횟수에 의존하지 않아서 이것들로 바굴수록 분산 시스템에 생기는 여러 문제를 해결함.
- 반면 액션은 실행 시점, 횟수에 의존적이지만 격리시키면 되고 안전하게 다룰 수 있는 기술이 있기 때문에 안심할 수 있음. 액션에서 계산으로 옮기면 다루기 쉬워짐.

## 액션
> 실행 시점이나 횟수 또는 둘 다에 의존.

잘 사용하는 방법
- 시간이 지남에 따라 안전하게 상태를 바꿀 수 있는 방법
- 순서를 보장하는 방법
- 액션이 정확히 한 번만 실행되게 보장하는 방법

## 계산
> 입력값으로 출력값을 만드는 것. 같은 입력값을 가지고 계산하면 항상 같은 결괏값이 나옴. 외부에 영향을 주지 않음. 테스트하기 쉽고 언제든지 몇 번을 불러도 안전함.

잘 사용하는 방법
- 정확성을 위한 정적 분석
- SW에서 쓸 수 있는 수학적 지식
- 테스트 전략

## 데이터
> 이벤트에 대해 기록한 사실. 실행하지 않아도 데이터 자체로 의미가 있음. 같은 데이터를 여러 형태로 해석할 수 있음.

잘 사용하는 방법
- 효율적으로 접근하기 위해 데이터를 구성하는 방법
- 데이터를 보관하기 위한 기술
- 데이터를 이용해 중요한 것을 발견하는 원칙

## 함수형 사고란?
함수형 프로그래머가 SW 문제를 해결하기 위해 사용하는 기술과 생각

아래 두 개념은 함수형 프로그래밍으로 실용적이고 튼튼한 프로그램을 만드는 기초가 됨.

1. 액션과 계산, 데이터
2. 일급 추상: 타임라인 다이어그램을 이용해 시간에 따라 변하는 액션을 추상화시킴. 함수를 인자로 받는 **일급 함수**를 사용함.

> Chapter 2: 현실에서의 함수형 사고

## 변경 가능성에 따라 코드 나누기
<img width="600" alt="계층화 설계" src="https://github.com/FrontendStudySeoul/SsogSsogFunctionalCoding/assets/62797441/4d9d9728-2c74-4650-a004-a55b50801797">

각 계층은 그 아래에 있는 계층을 기반으로 만들어 진다. 그래서 각 계층에 있는 코드는 더 안정적인 기반 위에 작성할 수 있다. 이런 구조로 SW를 만들면 코드를 쉽게 변경할 수 있다. 가장 위에 있는 코드는 의존성이 거의 없기 때문에 쉽게 바꿀 수 있다. 아래에 있는 코드들은 위에 있는 코드보다 의존성이 많아 바꾸기 어렵지만 자주 바뀌지 않는다.

함수형 프로그래머는 이 아키텍처 패턴이 계층을 만들기 때문에 **계층형 설계** 라고 부른다. 계층형 설계는 일반적으로 비즈니스 규칙, 도메인 규칙, 기술 스택 계층으로 나눈다. 테스트, 재사용, 유지보수가 쉽다.

## 일급 추상
<img width="600" alt="스크린샷 2023-09-03 오후 1 03 03" src="https://github.com/FrontendStudySeoul/SsogSsogFunctionalCoding/assets/62797441/23644129-21ea-4b89-b612-1519c9a63916">

타임라인 다이어그램을 사용하면 액션이 시간 순서에 따라 어떻게 실행되는지 볼 수 있다. 액션은 실행 시점에 의존하기 때문에 실행 순서가 중요하다.

## 분산 시스템을 타임라인으로 시각화하기
<img width="600" alt="스크린샷 2023-09-03 오후 1 06 03" src="https://github.com/FrontendStudySeoul/SsogSsogFunctionalCoding/assets/62797441/98da7388-6013-40d9-b94e-00edadc16eca">

로봇 한 대 -> 로봇 세 대가 동시에 일을 하면 더 빨라짐.(분산 시스템) 분산 시스템에서 독립된 액션의 실행 순서는 어떻게 될지 모름. 로봇은 각자 타임라인을 가지고 있음.

타임라인 다이어그램은 문제를 파악하는 데 도움이 됐지만 실행 순서가 섞이는 것은 어떻게 할 수 없음. 많은 피자가 잘못 나옴.

## 각각의 타임라인은 다른 순서로 실행됨.
기본적으로 타임라인은 서로 순서를 맞출 수 있는 기능이 없음. 다른 타임라인 작업이 끝날 때까지 기다리라는 표시가 없어서 순서대로 다음 단게를 그냥 진행함. 서로 다른 타임라인에 있는 액션 간 실행 순서는 보장할 수 없음.

**타임라인을 서로 맞추지 않은 분산 시스템은 예측 불가능한 순서로 실행됨.** 로봇들이 반죽, 치즈, 소스 준비가 끝나면 피자를 만들도록 해야 함.

## 어려운 경험을 통해 분산 시스템에 대해 배운 것.
순차적인 프로그램을 분산 시스템으로 바꾸는 것은 어렵다. 올바른 순서로 동작하는 프로그램을 만들려면 액션에 집중할 필요가 있다.
1. 기본적으로 타임라인은 서로 순서를 맞추지 않았다.
2. 액션이 실행되는 시간은 중요하지 않다: 소스 만드는 것이 제일 오래 걸리지만 항상 그렇지는 않다. 각 타임라인은 다른 타임라인의 순서와 관계없이 만들어야 한다.
3. 드물지만 타이밍이 어긋나는 경우는 실제 일어난다: 주문이 많이 들어오자 가끔 발생하던 오류는 더 많이 발생한다. 타임라인은 항상 옳바른 결과를 보장해야 한다.
4. 타임라인 다이어그램으로 시스템의 문제를 알 수 있다.

## 타임라인 커팅: 로봇이 서로를 기다릴 수 있게 하기
여러 타임라인이 동시에 진행될 때 서로 순서를 맞추는 방법. **고차 동작**으로 구현. 각 타임라인은 독립적으로 동작하고 작업이 완료되면 다른 타임라인 끝나기를 기다리기 때문에 어떤 타임라인이 먼저 끝나도 괜찮음.

이제 로봇은 다른 작업이 끝나기를 기다림. 모든 작업이 완료되면 로봇 한대가 나머지 피자를 완성함. 이렇게 하면 재료를 준비하는 작업(반죽 만들기, 치즈 갈기, 소스 만들기)은 순서가 중요하지 않음.

1. 타임라인 커팅으로 서로 다른 작업들을 쉽게 이해할 수 있음: 동시에 할 수 있는 재료 준비와 순서대로 해야 하는 피자 만들기를 분리.
2. 타임라인 다이어그램을 사용하면 시간에 따라 진행하는 작업을 쉽게 이해할 수 있음: 타임라인은 동시에 실행되는 분산 시스템을 시각화하기 좋음.
3. 타임라인 다이어그램은 유연함: 타임라인을 보고 쉽게 코드로 옮길 수 있고 동시에 진행되는 작업을 쉽게 모델링할 수 있음.

## 결론
1. 액션, 계산, 데이터를 구분해야 함.
2. 함수형 프로그래머는 유지보수를 잘 하기 위해 계층형 설계를 사용함. 각 계층은 코드의 변경 가능성에 따라 나눔.
3. 타임라인 다이어그램은 시간에 따라 변하는 액션을 시각화하는 방법임. 액션이 다른 액션과 어떻게 연결되는지 볼 수 있음.
4. 타임라인 커팅은 액션이 올바른 순서로 실행할 수 있도록 보장함.
