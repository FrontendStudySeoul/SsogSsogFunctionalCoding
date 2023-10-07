# Chapter 17. 타임라인 조율하기

## 시간을 일급으로 다루기

> 액션의 순서와 타이밍을 맞추기 위해 타임라인을 다이어그램으로 그려 시각화해보고 Cut의 개념을 이용해 해결해보자.

## 버그 발견

최적화를 하고자 로직을 분리했지만 버그가 발생했습니다.

```tsx
// 이전
function calculateCartTotal(cart: Cart, callback: (total: number) => void) {
  let total = 0;
  cost_ajax(cart, (cost) => {
    total += cost;

    shipping_ajax(cart, (shipping) => {
      total += shipping;
      callback(total);
    });
  });
}
```

```tsx
// 이후
function calculateCartTotal(cart: Cart, callback: (total: number) => void) {
  let total = 0;
  cost_ajax(cart, (cost) => {
    total += cost;
  });

  shipping_ajax(cart, (shipping) => {
    total += shipping;
    callback(total);
  });
}
```

상품 금액 계산(cost_ajax)과 배송비 계산(shipping_ajax) 로직이 동시에 수행됩니다. 일부 코드를 콜백 밖으로 옮겨서 속도가 빨라지긴 했지만 타이밍이 맞지 않는 문제가 발생한다.

-> cost_ajax에서의 응답이 늦어 total값이 제대로 갱신되지 않은 채 shipping_ajax에서의 callback이 실행되어 정상적이지 않은 total 값이 화면에 업데이트되는 문제가 발생한다.

<br />

## 다이어그램을 그려 해결하기

1. 액션을 확인한다.
2. 각 액션을 그린다.
3. 단순화한다.

<br />

## 실행 가능한 순서 분석하기

> 타임라인이 공유하는 자원은 total 변수 뿐입니다.

두 콜백 타임라인의 실행 가능한 순서를 알아보면,

1. 동시에 실행 되는 경우(왼쪽 콜백과 오른쪽 콜백), 자바스크립트 스레드 모델에서 불가능하므로 고려하지 않는다.
2. 왼쪽 콜백(cost_ajax())를 먼저 실행하는 경우 → 기대한 결과
3. 오른쪽 콜백(shipping_ajax())를 실행하는 경우 → 제품 가격없이 dom이 업데이트되므로 잘못된 동작에 해당합니다.

위 다이어그램은 오른쪽 콜백(shipping_ajax())가 먼저 실행될 수도 있는 다이어그램이므로 버그가 발생할 수 있음을 알 수 있습니다.

이를 해결하기 위해서는 **병렬로 요청된 모든 콜백이 끝날 때까지 기다린** 다음에 DOM을 업데이트하면 해결할 수 있습니다.

![Untitled (6)](https://github.com/dmswl98/careerboost/assets/76807107/00f8e26e-9433-4981-a5b5-ce29efeca0eb)

### Cut

점선을 컷(cut)이라하며 컷은 순서를 보장해주는 역할을 합니다.

컷을 만들면 컷의 앞부분과 뒷부분의 액션이 섞이지 않도록 하고 실행 가능한 순서를 줄이기 때문에 애플리케이션의 복잡성을 줄일 수 있다.

<br />

## 최적화 이전과 이후 비교하기

만약 cost_ajax()의 응답이 3초, shipping_ajax() 응답이 4초가 걸린다면 최적화 이전 다이어그램에서는 총 7초가 걸립니다.

최적화 이후 다이어그램의 경우, 최대 4초가 걸립니다.

그 이유는, 최적화 이전의 다이어그램의 경우 두 응답을 순서대로 기다려야 하지만 최적화 이후의 다이어그램의 경우 두 응답을 병렬로 기다리기 때문이다.

<br />

## 타임라인을 나누기 위한 동시성 기본형

경쟁 조건을 막으면 됩니다.

> 경쟁 조건(race condition)은 어떤 동작이 먼저 끝나는 타임라인에 의존할 때 발생합니다.

동기적을 접근하는 변수를 이용해서 동시성 기본형을 구현할 수 있습니다.

> num은 기다릴 타임라인의 수, callback은 모든 것이 끝났을 때 실행할 콜백, 카운터(num_finished)와 기다릴 타임라인의 수가 같을 때, 즉 마지막 타임라인이 끝났을 때 콜백을 호출한다.

```tsx
function Cut(num, callback) {
  let num_finished = 0;

  return function () {
    num_finished += 1;
    if (num_finished === num) callback();
  };
}
```

<br />

## 코드에 Cut() 적용하기

아래 두 가지를 고려해야 합니다.

1. Cut() 을 보관할 범위 : 응답 콜백 끝에서 done()을 불러야 합니다.
2. Cut() 에 어떤 콜백을 넣을지

```tsx
function calculateCartTotal(cart: Cart, callback: (total: number) => void) {
  let total = 0;
  const done = Cut(2, () => callback(total)); // 2는 두 번의 호출을 기다린다는 의미

  costAjax(cart, (cost) => {
    total += cost;
    done();
  });

  shippingAjax(cart, (shipping) => {
    total += shipping;
    done();
  });
}
```

위 두 개의 콜백이 done()을 호출하고 있으며, done()이 두 번 호출될 때까지 타임 라인은 실행되지 않는다.

> 자바스크립트에서의 동시성 기본형의 예 : Promise.all()

위 타임 라인은 이제 콜백 함수가 병렬로 실행되어 속도가 개선되었고 순서대로 실행됨이 보장되었습니다.

<br />

## 딱 한번만 호출하는 기본형

> Cut()은 마지막으로 실행되는 타임라인에서 콜백 함수가 실행되지만, 첫 번째로 실행되는 타임라인에서 콜백을 실행하는 시간 모델을 만들어보자.

```tsx
function JustOnce(action) {
  var alreadyCalled = false; // 이전에 함수가 실행되었는지 확인

  return function (a, b, c) {
    if (alreadyCalled) return; // 실행한 적이 있다면 바로 종료
    alreadyCalled = true;

    return action(a, b, c); // 인자와 함께 액션 호출
  };
}
```

```tsx
var sendAddToCartTextOnce = JustOnce(sendAddToCartText);

sendAddToCartTextOnce("1234-1234"); // 첫 번째 실행할 때만 메시지를 보낼 수 있다.
sendAddToCartTextOnce("1234-1234");
```

최초로 한 번만 효과가 발생하는 액션을 **멱등원**이라고 하며 JustOnce은 어떤 액션이든 멱등원으로 만들어준다.

<br />

## 좋은 타임라인의 원칙

1. 타임라인은 적을수록 이해하기 쉽습니다.
2. 타임라인은 짧을수록 이해하기 쉽습니다.
3. 공유하는 자원이 적을수록 이해하기 쉽습니다.
4. 자원을 공유한다면 서로 조율해야 합니다.
5. 시간을 일급으로 다룹니다.

## 암묵적 시간 모델 vs 명시적 시간 모델

자바스크립트 시간 모델은 간단합니다.

1. 순차적 구문은 순서대로 실행됩니다.
2. 두 타임라인에 있는 단계는 왼쪽 먼저 실행되거나, 오른쪽 먼저 실행될 수 있습니다.
3. 비동기 이벤트는 새로운 타임라인에서 실행됩니다.
4. 액션은 호출할 때마다 실행됩니다.

암묵적 시간 모델은 실행 방식을 바꿀 수 없기 때문에 필요한 실행 방식에 가깝게 새로운 명시적인 시간 모델을 만들어 해결할 수 있다.

## 요약: 타임라인 사용하기

- 타임라인 수를 줄입니다.
- 타임라인 길이를 줄입니다.
- 공유 자원을 없앱니다.
- 동시성 기본형으로 자원을 공유합니다.
- 동시성 기본형으로 조율합니다.
