# 액션에서 계산 뺴내기

### 해당 장에서 살펴볼 내용
- **어떻게 함수로 정보가 들어가고 나오는지 살펴보기**
- **테스트하기 쉽고 재사용성이 좋은 코드를 만들기 위한 함수형 기술**
- **액션에서 계산을 뺴내는 방법**


### 예시 : MegaMart.com
MegaMart는 온라인 쇼핑몰입니다.<br>
경쟁력을 유지하고 있는 중요한 기능중 하나는 쇼핑중에 장바구니에 담겨 있는 제품의 금액 합꼐를 볼 수 있습니다.
<img width="627" alt="image" src="https://github.com/FrontendStudySeoul/SsogSsogFunctionalCoding/assets/103626175/05b27363-2f15-4491-a016-5942b8cd6e63">

```tsx
var shopping_cart = []; // 장바구니 제품과 금액 합계를 담고 있는 전역변수
var shopping_cart_total = 0; // 장바구니 금액 합계를 담고 있는 전역변수

function add_item_to_cart(name, price) {
  shopping_cart.push({
    name: name,
    price: price,
  });
  calc_cart_total();
}

function calc_cart_total() {
  shopping_cart_total = 0;
  for(var i = 0; i < shopping_cart.length; i++) {
    shopping_cart_total += item.price;
  }
  set_cart_total_dom(); // 금액 합계를 반영하기 위해 DOM 업데이트
}
```

<img width="627" alt="image" src="https://github.com/FrontendStudySeoul/SsogSsogFunctionalCoding/assets/103626175/12091313-fa65-4254-9ecf-6b0d73c376bb">


여기서 새로운 요구사항이 들어옵니다.
구매 합계가 20 달러 이상이면 무료 배송을 해줄 수 있습니다.
그래서 20달러 이상이면 구매하기 버튼 옆에 무료 배송 아이콘이 표시 됩니다.

```tsx
function update_shipping_icons() {
  var buy_buttons = get_buy_buttons_dom();
  for(var i = 0; i < buy_buttons.length; i++) {
    var button = buy_bottons[i];
    var item = button.item;
    if(item.price + shopping_cart_total >= 20)
      button.show_free_shipping_icon();
    else
      button.hide_free_shipping_icon();
  }
}

function calc_cart_total() {
  shopping_cart_total = 0;
  for(var i = 0; i < shopping_cart.length; i++) {
    shopping_cart_total += item.price;
  }
  set_cart_total_dom(); // 금액 합계를 반영하기 위해 DOM 업데이트
  update_shipping_icons(); // 아이콘을 업데이트하는 코드 추가
}
```
마지막 요구사항으로 금앱 합계가 바뀔 떄마다 세금을 다시 계산하는 기능이 생깁니다.
```tsx
function update_tax_dom() {
  set_tax_dom(shopping_cart_total + 0.10); // 세금 계산 후 DOM을 바로 업데이트
}

function calc_cart_total() {
  shopping_cart_total = 0;
  for(var i = 0; i < shopping_cart.length; i++) {
    var item = shopping_cart[i];
    shopping_cart_total += item.price;
  }
  set_cart_total_dom(); // 금액 합계를 반영하기 위해 DOM 업데이트
  update_shipping_icons(); // 아이콘을 업데이트하는 코드 추가
  update_tax_dom(); // 페이지에 세금을 업데이트하기 위한 코드 추가
}
```
앞에선 하나의 기능에 dom을 바꾸는 함수와 비즈니스 로직이 같이 들어가있기 때문에 테스트가 너무 힘듭니다. <br>
그리고 세금을 계산하는 로직에서 shopping_cart_total라는 전역변수가 생겼는데 이 또한 코드의 가독성과 테스트를 힘들게 만듭니다.

<br>
이를 잘 해주려면 어떻게 코드를 리팩토링 해야할까요 ? (질문)

1. 전역변수애 의존 x
2. Dom 로직을 외부로 이동
3. 합수는 항상 결과값을 리턴

### 재사용하기 쉽게 만들기

추후 웹페이지에서 보여주는 코드를 다른팀에서 사용하려하지만 해당 코드를 직접 바꾸지 않는 이상 재사용할 수 없었습니다.
- 장바구니 정보를 전역변수에서 읽어오는 코드지만 다른팀은 DB에서 직접 정보를 읽어야한다.
- 결과를 보여주기 위해 DOM을 바꾸는 코드로 돼있지만 다른팀은 해당 정보로 영수증이나 운송장을 출력해야한다.


### 액션과 계산, 데이터를 구분하기
1. 전역변수는 action이다.
2. 전역변수를 바꾸는것 또한 action이다.
3. dom을 읽는것은 action이다.
4. dom을 바꾸는것 또한 action

### 함수에는 입력과 출력이 있다.
모든 함수는 입력과 출력이 있습니다.
<br>
입력은 함수가 계산을 하기 위한 외부 정보입니다. 출력은 함수 밖으로 나오는 정보나 어떤 동작입니다.
<br>
즉 결과를 얻으려면 항상 입력이 필요합니다.

```tsx
var total =0;
function add_to_total(amount){// 인자는 입력
console.log(total);// 콘솔에 찍는것은 출력, 전역변수를 읽는것은 입력
total+=amount;// 전역변수를 바꾸는건 출력
return total;// 리턴값은 출력
}
```
위처럼 입력과 출력은 명시적일수도 있지만 암묵적일 수 있습니다.<br>
함수에 암묵적 입력과 출력이 있으면 액션이므로 두개를 없애주는게 계산으로 바꾸는 것입니다.

### 액션에서 계산 빼내기
액션에 있는 계산을 뺴냅니다.

앞서 전역변수는 암묵적인 값이라고 했기에 이부분을 고칩니다.

[함수 추출하기 리팩토링 2판 8 - 1](https://github.com/FrontendStudySeoul/RefactoringV2/blob/main/4%EC%A3%BC%EC%B0%A8/moseung.md)

```tsx
//리팩토링 전
function calc_cart_total() {
  shopping_cart_total = 0;
  for(var i = 0; i < shopping_cart.length; i++) {
    shopping_cart_total += item.price;
  }
  set_cart_total_dom(); // 금액 합계를 반영하기 위해 DOM 업데이트
}
//리팩토링 후
//함수 내의 모든 값이 순수 입력으로 계산되고 순수 출력인 return 구문이 생겼다.
function calc_cart_total(cart) {
  var total = 0;
  for(var i = 0; i < cart.length; i++) {
    total += item.price;
  }
  return total
}
```

### 액션에서 또 다른 계산 빼내기
```tsx
//리팩토링 전
function add_item_to_cart(name, price) {
  shopping_cart.push({
    name: name,
    price: price,
  });
  calc_cart_total();
}
//리팩토링 후
//전역 변수(암묵적 입력)에 의존적인 코드가 사라지고 명시적인 입력과 출력인 계산 함수로 바뀌었다.
//입력값을 카피온라이트를 이용해서 원본 값을 보존합니다.
function add_item_to_cart(cart,name, price) {
var new_cart = cart.slice()
  new_cart.push({
    name: name,
    price: price,
  });
return new_cart;
}
```



### 테스트와 재사용성은 입출력과 관련 있습니다.
1. 돔 업데이트와 비즈니스 규칙은 분리되어야합니다.
- 앞서 돔 관련 로직은 암묵적 출력이라고 했기때문에 비즈니스 로직과 분리가 되어야 테스트가 가능하다.
2. 전역변수가 없어야 합니다.
- 전역변수 또한 앞서 암묵적인 값이 될 수 있기때문에 없어야 합니다.
3. 함수는 결괎값을 리턴해야 합니다.
- 함수에 명시적인 출력과 입력으로 관리하기 위해 함수가 결괎값을 리턴해야합니다.

### 정리

#### chatGPT
```
Q : 
함수형 프로그래밍에서 액션을 데이터와 계산으로 분리하는데 이렇게하면 실제로 코드가 더 길어지는 현상이 발생하기도해
근데 이렇게 되면 오히려 가독성이 떨어지는걸수도 있는게 아닐까 ?
코드가 더 길어지더라고 액션과 데이터를 분리하는게 맞는걸가 ?

A :
함수형 프로그래밍에서 액션(로직)과 데이터를 분리하는 주요 목적은 "순수성"과 "재사용성"을 추구하기 위함입니다.

순수성: 함수형 프로그래밍에서는 순수 함수(pure function)를 사용하려고 노력합니다.
순수 함수는 동일한 입력에 대해 항상 동일한 출력을 반환하며, 외부 상태에 영향을 주지 않습니다. 이렇게 함으로써 코드의 예측 가능성이 높아지고 디버깅이 쉬워집니다.
재사용성: 액션(로직)과 데이터를 분리하면, 해당 로직을 다양한 데이터에 대해 재사용할 수 있게 됩니다. 이는 코드의 중복을 줄이고 유지보수를 용이하게 합니다.
그러나 말씀하신대로, 이런 접근 방식은 때때로 코드가 길어지는 결과를 초래할 수 있습니다. 그리고 이것이 반드시 가독성을 해치는 것은 아닙니다.
가독성은 코드의 길이만큼이나, 그 구조와 작명, 주석 등 여러 요소에 의해 결정됩니다.

따라서 액션과 데이터를 분리하는 것이 항상 옳다고 말하기 보다는, 개발 상황과 요구 사항에 따라 적절히 판단해야 한다고 볼 수 있습니다.

예를 들어 작은 스크립트나 단일 기능만 수행하는 코드에서는 굳이 함수형 프로그래밍 원칙을 엄격하게 지키려 할 필요가 없습니다.
반면 큰 프로젝트나 팀으로 작업할 때, 특히 복잡도가 높거나 변경 가능성이 있는 경우에는 함수형 프로그래밍 원칙을 따르면 유익할 수 있습니다.
```

```
Q:
재사용성을 높이고 쉽게 테스트 할 수 있는 것이 함수평 프로그래밍으로 얻을 수 있는 전부입니까?
A:
동시성이나 설계, 데이터 모델링 측면에서 좋은 점들을 가져갈 수 있습니다.
```

```
Q:
다른곳에서 중복해서 쓰지 않더라도 계싼으로 분리하는 것이 중요한 것입니까?
A:
함수형 프로그래밍의 목적 중에 어떤 것을 분리해서 더 작게 만들려고 하는 것도 있습니다. 작은 것은 테스트하기 쉽고 재사용하기 쉽고 이해하기 쉽기 때문입니다.
```

#### 계산 추출의 단계
1. 계산 코드를 찾아 뺴냅니다.
2. 새 함수에 압묵적 입력과 출력을 찾습니다.
3. 암묵적 입력은 인자(명시적 입력)로 암묵적 출력은 리턴값(명시적 출력)으로 바꿉니다.

아래 함수를 어떻게 **계산을 추출**할 수 있을까요?
```tsx
function update_tax_dom() {
  set_tax_dom(shoppping_cart_total * 0.10);
}
```

```tsx
function update_tax_dom() {
  set_tax_dom(get_total_with_tax);
}

function get_total_with_tax (total){
  const TAX_RULE = 0.10;
  return total * TAX_RULE;
}
```
