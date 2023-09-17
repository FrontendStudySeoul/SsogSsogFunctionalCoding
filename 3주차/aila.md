# Chapter 7 : 신뢰할 수 없는 코드를 쓰면서 불변성 지키기

## 레거시 코드와 불변성

```js
function add_item_to_cart(name, price) {
  const item = make_cart_item(name, price);
  shopping_cart = add_item(shopping_cart, item);
  const total = calc_total(shopping_cart);
  set_cart_total_dom(total);
  update_shipping_icons(shopping_cart);
  update_tax_dom(total);

  // 새로 추가해야하는 레거시 코드
  // 이 코드는 장바구니 값을 바꾸기 때문에 불변성을 해친다.
  black_friday_promotion(shopping_cart);
}
```

- 추가된 함수(`black_friday_promotion`)를 호출하면 카피 온 라이트 원칙을 지킬 수 없다.
- 그렇다고 `black_friday_promotion`를 고칠 수도 없다.
- `방어적 복사` : ` 카피-온-라이트` 원칙을 지키면서 안전하게 함수를 사용할 수 있는 다른 원칙.

## 우리가 만든 카피 온 라이트 코드는 신뢰할 수 없는 코드와 상호작용해야 합니다

- 불변성이 지켜진 부분을 `안전지대`라고 한다.
- `안전지대`에 있는 코드는 걱정 없이 쓸 수 있다.
- `black_friday_promotion`은 `안전지대`에 있는 코드가 아니다.
- `black_friday_promotion`의 입출력을 통해 안전지대에 있는 코드와 데이터를 주고 받아야 한다.
- 문제는 안전지대 밖으로 나가는 데이터는 잠재적으로 바뀔 수 있고, 신뢰할 수 없는 코드에서 안전지대로 들어오는 데이터 역시 잠재적으로 바뀔 수 있다는 것이다.

## 방어적 복사

- 기존의 카피-온-라이트는 데이터를 바꾸기 전에 복사한다.
- 하지만, 레거시 코드에서는 무엇을 복사해야 할지 예상할 수 없으며 어떤 일이 일어날지 알 수 없음
- 따라서 데이터가 바뀌는 것을 완벽히 막아주는 원칙이 필요하다 **(방어적 복사)**

방어적 복사는 원본이 바뀌는 것을 막아준다.

- 레거시 코드에서 데이터가 들어오는 상황

1. 신뢰할 수 없는 코드에서 변경 가능한 데이터가 들어온다.
2. 들어온 데이터로 깊은 복사본을 만들고, 변경 가능한 원본 데이터는 버린다.
3. 신뢰할 수 있는 코드만 복사본을 사용하므로 데이터는 바뀌지 않는다. (데이터 보호)

- 레거시 코드로 데이터를 보낼 때의 상황

1. 데이터를 내보내기 전에 깊은 복사를 진행
2. 깊은 복사본의 데이터를 내보내 안전한 코드에서의 데이터가 변경되지 않도록 함

> 방어적 복사를 통해 불변성을 지킬 수 있다.

## 방어적 복사 구현하기

```js
...
shopping_cart = add_item(shopping_cart, item);
...
var cart_copy = deepCopy(shopping_cart);  // 넘기기 전에 복사
black_friday_promotion(cart_copy);
shopping_cart = deepCopy(cart_copy); // 들어오는 데이터를 위한 복사
```

## 방어적 복사 규칙

- 규칙 1 : 데이터가 안전한 코드에서 나갈 때 복사하기
  - 불변성 데이터를 위한 깊은 복사본을 생성
  - 신뢰할 수 없는 코드로 복사본을 전달합니다.
- 규칙 2 : 안전한 코드로 데이터가 들어올 때 복사하기
  - 변경될 수도 있는 데이터가 들어오면 바로 깊은 복사본을 만들어 안전한 코드로 전달
  - 복사본을 안전한 코드에서 사용합니다.

## 신뢰할 수 없는 코드 감싸기

방어적 복사 코드를 분리해 새로운 함수로 만들자.

```js
function add_item_to_cart(name, price) {
...
shopping_cart = black_friday_promotion_safe(shopping_cart);
}

function black_friday_promotion_safe(cart) {
  var cart_copy = deepCopy(cart);
  black_friday_promotion(cart_copy);
  return deepCopy(cart_copy);
}
```

## 카피-온-라이트와 방어적 복사 비교

|        |        카피-온-라이트        |              방어적 복사               |
| :----: | :--------------------------: | :------------------------------------: |
|  언제  | 통제할 수 있는 데이터 변경시 | 신뢰할 수 없는 코드와 데이터 상호작용  |
| 어디서 |   안전한 코드 내 어디서든    | 안전한 코드 경계에서 데이터 오고 갈 때 |
|  방식  |          얕은 방식           |               깊은 복사                |
|  규칙  |      1. 얕은 복사 생성       |    1. 들어온 데이터 깊은 복사 생성     |
|        |        2. 복사본 변경        |     2. 나갈 데이터 깊은 복사 생성      |
|        |        3. 복사본 리턴        |                                        |

## 깊은 복사는 얕은 복사보다 비쌉니다

- 깊은 복사는 원본과 어떤 데이터 구조도 공유하지 않는다.
- 중첩된 모든 객체, 배열을 복사한다.
- 깊은 복사는 얕은 복사보다 비싸기 때문에 카피-온-라이트를 사용할 수 없는 곳에서만 사용한다.
- 모든 항목을 재귀적으로 복사하는 lodash의 cloneDeep 함수를 사용하면 깊은 복사를 할 수 있다.

## structuredClone()

- 깊은 복사를 위한 js의 내장 기능
- 기존에는 Web API의 [structured clone algorithm](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm)을 사용했다.

### structured clone algorithm

```js
function clone(objectToBeCloned) {
  if (!(objectToBeCloned instanceof Object)) {
    return objectToBeCloned;
  }

  var objectClone;

  //객체라면 타입에 따라 객체를 생성하여 리턴
  var Constructor = objectToBeCloned.constructor;
  switch (Constructor) {
    case RegExp:
      objectClone = new Constructor(objectToBeCloned);
      break;
    case Date:
      objectClone = new Constructor(objectToBeCloned.getTime());
      break;
    default:
      objectClone = new Constructor();
  }

  //객체의 속성들을 복사
  for (var prop in objectToBeCloned) {
    // 재귀
    objectClone[prop] = clone(objectToBeCloned[prop]);
  }

  // 복사된 객체 리턴
  return objectClone;
}
```

### structuredClone() vs JSON.parse(JSON.stringify(x))

```js
const Person = {
  name: 'namsun',
  date: new Date('2023-09-16'),
  friends: ['risa', 'jiwoo'],
};

const copyPerson = JSON.parse(JSON.stringify(Person));
```

```js
{
    name: "namsun",
    date: "2023-09-16T00:00:00.000Z",
    friends: Array(2)
}
```

![](https://velog.velcdn.com/images/nsunny0908/post/3f67e9b0-d221-46ab-8130-fe006303b415/image.png)

- JSON은 객체를 문자열로 인코딩하는 형식
- 직렬화를 사용하여 개체를 해당 문자열로 변환하고 역직렬화를 사용(문자열 변환 -> 개체).

```js
const Person = {
  name: 'namsun',
  date: new Date('2023-09-16'),
  friends: ['risa', 'jiwoo'],
};

const copyPerson = structuredClone(Person);
```

```js
{
    name: "namsun",
    date: Object,
    friends: Array(2)
}
```

- 프로토타입, 함수, Error, Dom 노드를 다룰 때 제한사항이 있다.

## 들었던 생각

- 깊은 복사는 강력한 불변성을 보장하지만 비용이 비싸기 때문에, 어느정도의 트레이드 오프가 있다고 생각
- 카피-온-라이트 패턴과 같이 깊은 복사를 적절하게 사용해서 불변성을 지킬 수 있을지에 대한 고민이 필요하다.
