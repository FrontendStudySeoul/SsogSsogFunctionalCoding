## Intro
### 함수형 프로그래밍은 무엇인가요 ?
#### 함수형 프로그래밍이란?
- 수학 함수를 사용하고 부수효과를 피하는 것이 특징인 프로그래밍 패러다임
- 부수 효과 없이 순수 함수만 사용하는 프로그래밍 스타일

#### 부수효과
함수가 리턴값 이외에 하는 모든 일.
```tsx
let seungmoAge ;
function getSeungmo(){
//부수효과
seungmoAge = 27
//함수의 기존 목적
return "승모"
}
```
#### 순수 함수
인자에만 의존하고 부수 효과가 없는 함수.<br>
같은 인자를 넣으면 항상 같은 결과를 돌려주는 함수
```tsx
function add (a:number,b:number){
return a+b
}
```

### 실용적 측면에서 함수형 프로그래밍 정의의 문제점
- 부수 효과는 필요하다
- 함수형 프로그래밍은 부수 효과를 잘 다룰 수 있다.
- 함수평 프로그래밍은 실용적이다.

### 함수형 프로그래밍을 기술과 개념으로 보기
함수형 프로그래밍은 코드 어느곳에나 적용할 수 있는 유익한 기술이다.
<br>
함수형 프로그래밍은 액션,계산, 데이터로 구분한다.
<br>
**함수형 프로그래머는 액션보다 계산을 좋아하고 계산보다 데이터를 좋아합니다.**

### 함수형 프로그래머는 액션과 계산, 데이터를 구분합니다.
#### 액션
액션은 실행 시점이나 횟수 또눈 둘 다에 의존합니다.긴급한 메일을 오늘 보내는것과 다음 주에 보내는 것이 완전히 다르고 같은 메일을 10번 보내고 한번 보내는것 또한 다릅니다.
#### 계산
입력값으루 출력값을 만드는것. 계산은 테스트하기 쉽고 언제든지 몇 번을 불러도 안전합니다.
#### 데이터
데이터는 이벤트에 대해 기록한 사실입니다.데이터는 실행하는 코드만큼 복잡하지 않고 데이터 자체로 의미있습니다.<br>
또 같은 데이터를 여러 형태로 해석할 수 있습니다.

### 액션, 계산, 데이터를 구분하면 어떤 장점이 있나요?
최근 들어 인터넷과 전화, 노트북, 클라우드 서버와 같은 기기가 확산되면서 네트워크 통신을 하는 복잡한 소프트웨어가 필요하게 되었고, 그런 이유로 함수형 프로그래밍이 각광받게 되었습니다.
<br>
그래서 함수형 프로그래밍에서 실행 시점이나 횟수에 의존하는 코드를 줄이고(액션), 데이터나 계산으로 코드를 줄이려고 노력합니다.
<br>
그리고 액션을 코드 전체에 영향을 주지 않도록 격리시킵니다.

## 현실에서의 함수형 사고
함수형 프로그래밍에서는 두가지의 함수형 사고의 기술이 있다.
#### 파트 1: 액션과 계산, 데이터
실행 시점이나 횟수 또는 둘 다에 의존하는 액션과, 순수함수인 계산, 그리고 이벤트에 만들어진 데이터가 있다.
<br>
이를 잘 분리할 수 있는 계층형 설계가 중요하다.
#### 파트 2: 일급 추상
함수를 인자로 받는 일급함수를 사용해서 코드를 작성한다.

1. 런타임에서 언제나 들고다닐수 있고 인자를 넘길 수 있으며 원하는 시점에 평가할 수 있는 함수 .

```jsx
var f1 = function (a){
return a*a;
}
console.log(f1)
// 이처럼 함수가 변수에 담길 수 있다.
```

1. 함수가 함수를 인자로 받을 수 있다.

```jsx
function f3(f){
	return f()
}
f3(function(){return 10;})
```

```jsx
//add maker
function add_maker(a){
	return function(b){
		return a+b
	}
}
const add10 = add_maker(10);
add10(20) //30
```

함수를 갑승로 다루면서 순수함수를 다루면서 평가시점이 언제든지 가능해서 다양한 방법으로 평가가 가능한 방법론이 함수형 프로그래밍이다.

```jsx
function f4(f1,f2,f3){
return f3(f1()+f2())
}

f4(
function(){return 1}
function(){return 2}
function(a){return a*a}
)
```

함수가 함수를 받아서 원하는 값으로 평가하고 원하는 인자를 받아둔 함수에 컨트롤하는것.

### 변경 가능성에 따라 코드 나누기

<img width="459" alt="image" src="https://github.com/FrontendStudySeoul/SsogSsogFunctionalCoding/assets/103626175/66e5c625-f127-4e9a-af5d-8779f09bd1b5">
<br>
각 계층은 그 아래에 있는 계층을 기반으로 만들어 집니다. 그래서 각 계층에 있는 코드는 더 안정적인 기반 위에 작성할 수 있습니다.
<br>
그래서 아래에 있는 코드는 의존성은 강하지만 자주 바뀌지 않고 위에 있는 코드는 의존성이 약해 자주 바꿀 수 있습니다.

### 일급 추상
- 분산 시스템을 타임라인으로 시각화한다.
- 타임라인을 맞추기 위해 고차동작으로 구현한다.

#### 고차 동작(함수)
고차 함수(Higher order function)는 함수를 인자로 전달받거나 함수를 결과로 반환하는 함수를 말한다. 다시 말해, 고차 함수는 인자로 받은 함수를 필요한 시점에 호출하거나 클로저를 생성하여 반환한다.

## 참고 자료
[유인동 - 자바스크립트로 알아보는 함수형 프로그래밍](https://www.inflearn.com/course/%ED%95%A8%EC%88%98%ED%98%95-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D)
