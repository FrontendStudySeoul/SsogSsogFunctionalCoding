# Chapter 16 : 타임라인 사이에 자원 공유하기

## 15장에서 다룬 내용

- 코드를 타임라인 다이어그램으로 그리고, 타임라인을 단순화
- 버를 없애기 위해서 공유하는 자원을 줄이는 원칙을 적용

## 16장에서 다룰 내용

- 15장에서 해결하지 못한 공유하는 자원인 DOM을 안전하게 쓸 수 있는 방법에 대해 알아본다.

- 안전하게 자원을 공유할 수 있는 자원 공유 기본형을 만들어본다.

## 좋은 타임라인의 원칙

1. 타임라인은 적을수록 이해하기 쉽다.
2. 타임라인은 짧을 수록 이해하기 쉽다.
3. 공유하는 자원이 적을수록 이해하기 쉽다.

<b>4. 자원을 공유한다면 서로 조율해야 한다.</b>

5. 시간을 일급으로 다룬다.

## 장바구니에 아직 버그가 있다.

> 장바구니 기능은 두 액션이 **DOM 자원의 업데이트를 공유**하기 때문에 버그가 있다.

### 기대한 결과

![](https://velog.velcdn.com/images/nsunny0908/post/cd252750-b40b-4667-b337-63e3933dcf20/image.jpeg)

- 두번째 클릭했을 때 업데이트되는 DOM이 최신 정보이기 때문에 두번째 DOM 업데이트로 첫번째 DOM이 업데이트한 값을 덮어쓴다.

### 기대하지 않은 결과

![](https://velog.velcdn.com/images/nsunny0908/post/76babaec-7450-4dba-84f9-56d793efeeff/image.jpeg)

- 두번째 합계를 첫 번째 합계가 덮어쓰면 안된다.
- 하지만 순서가 보장되지 않기 때문에 막을 수 없다.

## DOM이 업데이트되는 순서를 보장해야한다.

- 타임라인으로 순서를 보장할 방법은 없다.
- 클릭한 순서대로 DOM 업데이트되도록 <u>업데이트 순서를 제한</u>해야한다.
- 큐(queue)를 만들어서 액션 순서를 조율할 수 있다.

## 자바스크립트에서 큐 만들기

> 자바스크립트에는 큐 자료 구조가 없기 때문에 직접 만들어야 한다.

- 큐는 자료 구조지만 타임라인 조율에 사용한다면 **동시성 기본형**이라 부른다.
- **동시성 기본형(concurrency primitive)** : 자원을 안전하게 공유할 수 있는 재사용 가능한 코드

### 클릭핸들러에서 할 일과 큐에서 할 일을 분리

- 순서가 바뀌는 첫번째 액션인 `cost_ajax()`를 큐에 넣고, 그 전에 할 일을 클릭 핸들러로 분리한다.
  <이미지 추가>

### 큐에서 처리할 작업을 큐에 넣기

```js
// 현재 코드
function add_item_to_cart(item) {
  cart = add_item(cart, item);
  calc_cart_total(cart, update_total_dom);
}

function calc_cart_total(cart, callback) {
  var total = 0;
  cost_ajax(cart, function (cost) {
    total += cost;
    shipping_ajax(cart, function (shipping) {
      total += shipping;
      callback(total);
    });
  });
}

// 새로운 코드
function add_item_to_cart(item) {
  cart = add_item(cart, item);
  update_total_queue(cart); // 큐에 항목을 추가
}

function calc_cart_total(cart, callback) {
  var total = 0;
  cost_ajax(cart, function (cost) {
    total += cost;
    shipping_ajax(cart, function (shipping) {
      total += shipping;
      callback(total);
    });
  });
}

// 큐의 기본적인 구현
var queue_items = [];

function update_total_queue(cart) {
  queue_items.push(cart);
}
```

- 여기까지의 큐는 배열 끝에 항목을 추가하는 단순한 코드이다.

### 큐에 있는 첫번째 항목을 실행한다.

```js
var queue_items = [];

// 배열의 첫 번째 항목을 꺼내 cart에 넣는다.
function runNext() {
  var cart = queue_items.shift();
  calc_cart_total(cart, update_total_dom);
}
function update_total_queue(cart) {
  queue_items.push(cart);
  // setTimeout()은 자바스크립트 이벤트 루프에 작업을 추가한다.
  setTimeout(runNext, 0);
}
```

- 아직 동시에 두 항목이 처리되는 것을 막는 코드가 없음

### 두 번째 타임라인이 첫 번째 타임라인과 동시에 실행되는 것을 막기

```js
var queue_items = [];
// 현재 동작하고 있는 다른 작업이 있는지 확인하는 flag 변수
var working = false;

function runNext() {
  // 동시에 두 개가 동작하는 것을 막을 수 있다.
  if (working) {
    return;
  }
  working = true;

  var cart = queue_items.shift();
  calc_cart_total(cart, update_total_dom);
}
function update_total_queue(cart) {
  queue_items.push(cart);
  setTimeout(runNext, 0);
}
```

- 두 타임라인이 동시에 실행되는 것을 막았지만, 장바구니에 추가된 작업이 항상 하나만 실행된다.

### 현재 작업이 끝났을 때 다음 작업을 실행할 수 있도록 콜백함수를 고쳐보자.

```js
var queue_items = [];
var working = false;

function runNext() {
  if (working) {
    return;
  }
  working = true;

  var cart = queue_items.shift();
  calc_cart_total(cart, function (total) {
    update_total_dom(total);
    // 작업 완료를 표시하고 다음 작업을 시작
    working = false;
    runNext();
  });
}
function update_total_queue(cart) {
  queue_items.push(cart);
  setTimeout(runNext, 0);
}
```

- 이제 배열에 있는 항목을 반복할 수 있다. 하지만 배열이 비었을 때 멈추지 않는다.

### 항목이 없을 때 멈추게 하기

- 빈 큐에서 queue_items.shift()를 호출하면 `undefined`를 반환한다.

```js
var queue_items = [];
var working = false;

function runNext() {
  if (working) {
    return;
  }

  // 큐에 항목이 없을 경우 실행을 멈춤
  if (queue_items.length === 0) {
    return;
  }
  working = true;

  var cart = queue_items.shift();
  calc_cart_total(cart, function (total) {
    update_total_dom(total);
    working = false;
    runNext();
  });
}
function update_total_queue(cart) {
  queue_items.push(cart);
  setTimeout(runNext, 0);
}
```

- 하지만 큐 코드에 있는 전역변수를 제거해줘야한다.

### 변수와 함수를 함수 범위로 넣기

```js
function Queue() {
  // 전역변수를 Queue() 의 지역변수로 변경
  var queue_items = [];
  var working = false;

  function runNext() {
    if (working) {
      return;
    }
    if (queue_items.length === 0) {
      return;
    }
    working = true;
    var cart = queue_items.shift();
    calc_cart_total(cart, function (total) {
      update_total_dom(total);
      working = false;
      runNext();
    });
  }

  // 큐에 항목을 넣을 수 있는 함수(기존의 update_total_queue 함수)를 리턴
  return function (cart) {
    queue_items.push(cart);
    setTimeout(runNext, 0);
  };
}

var update_total_queue = Queue();
```

## 큐를 재사용할 수 있도록 만들기

### done()함수 빼내기

- 큐를 재사용할 수 있도록 만들자.

```js
function Queue() {
  var queue_items = [];
  var working = false;

  function runNext() {
    if (working) {
      return;
    }
    if (queue_items.length === 0) {
      return;
    }
    working = true;
    var cart = queue_items.shift();

    // done() 을 콜백 함수로 전달
    function worker(cart, done) {
      calc_cart_total(cart, function (total) {
        update_total_dom(total);
        done(total);
      });
    }

    // 콜백 함수를 선언하며 호출
    worker(cart, function () {
      working = false;
      runNext();
    });
  }

  return function (cart) {
    queue_items.push(cart);
    setTimeout(runNext, 0);
  };
}

var update_total_queue = Queue();
```

- 아직 워커 함수는 장바구니에 제품을 추가하는 일만 할 수 있다.
- 많은 동작에 사용할 수 있도록 일반적인 큐를 만들자.

### 워커 행동을 바꿀 수 있도록 밖으로 빼기

```js
// worker 를 인자로 추가
function Queue(worker) {
  var queue_items = [];
  var working = false;

  function runNext() {
    if (working) {
      return;
    }
    if (queue_items.length === 0) {
      return;
    }
    working = true;
    var cart = queue_items.shift();

    worker(cart, function () {
      working = false;
      runNext();
    });
  }

  return function (cart) {
    queue_items.push(cart);
    setTimeout(runNext, 0);
  };
}

// worker 를 밖에서 정의하여 원하는 동작을 Queue의 인자로 전달
function calc_cart_worker(cart, done) {
  calc_cart_total(cart, function (total) {
    update_total_dom(total);
    done(total);
  });
}

var update_total_queue = Queue(calc_cart_worker);
```

### 작업이 완료되었을 때 실행하는 콜백을 받기

```js
function Queue(worker) {
  var queue_items = [];
  var working = false;

  function runNext() {
    if (working) {
      return;
    }
    if (queue_items.length === 0) {
      return;
    }
    working = true;

    // 이제 장바구니에 물건을 추가하는 일만 하지는 않으므로 일반적인 변수명으로 변경
    var item = queue_items.shift();
    worker(item.data, function () {
      working = false;
      runNext();
    });
  }

  // 배열에 데이터와 콜백을 모두 넣기
  return function (data, callback) {
    queue_items.push({
      data: data,
      callback: callback || function () {},
    });
    setTimeout(runNext, 0);
  };
}

function calc_cart_worker(cart, done) {
  calc_cart_total(cart, function (total) {
    update_total_dom(total);
    done(total);
  });
}

var update_total_queue = Queue(calc_cart_worker);
```

### 작업이 완료되었을 때 콜백 부르기

```js
function Queue(worker) {
  var queue_items = [];
  var working = false;

  function runNext() {
    if (working) {
      return;
    }
    if (queue_items.length === 0) {
      return;
    }
    working = true;

    var item = queue_items.shift();
    worker(item.data, function (val) {
      working = false;
      // callback을 비동기로 부른다.
      // callback에 인자를 전달한다.
      setTimeout(item.callback, 0, val);
      runNext();
    });
  }

  return function (data, callback) {
    queue_items.push({
      data: data,
      callback: callback || function () {},
    });
    setTimeout(runNext, 0);
  };
}

// 큐 함수는 일반적인 함수이기 때문에 일반적인 변수명을 사용하지만, calc_cart_worker는 어떤 값으로 사용할지 알기 때문에 cart, total 같은 구체적인 변수명을 사용한다.
function calc_cart_worker(cart, done) {
  calc_cart_total(cart, function (total) {
    update_total_dom(total);
    done(total);
  });
}

var update_total_queue = Queue(calc_cart_worker);
```

### 큐를 건너뛰도록 만들기

- worker 는 각각의 작업이 끝나야 다음으로 진행되기 때문에 느리다.
- 큐에 있는 마지막 업데이트만 필요하기 때문에, 그 외에 **덮어쓸 항목은 큐에서 제외한다.**

```js
function DroppingQueue(max, worker) {
  var queue_items = [];
  var working = false;

  function runNext() {
    if (working) {
      return;
    }
    if (queue_items.length === 0) {
      return;
    }
    working = true;

    var item = queue_items.shift();
    worker(item.data, function (val) {
      working = false;
      setTimeout(item.callback, 0, val);
      runNext();
    });
  }

  return function (data, callback) {
    queue_items.push({
      data: data,
      callback: callback || function () {},
    });
    // 큐에 추가한 후 항목이 max를 넘으면 버리기
    while (queue_items.length > max) {
      queue_items.shift();
    }
    setTimeout(runNext, 0);
  };
}

function calc_cart_worker(cart, done) {
  calc_cart_total(cart, function (total) {
    update_total_dom(total);
    done(total);
  });
}

// 한 개 이상은 모두 버린다.
var update_total_queue = DroppingQueue(1, calc_cart_worker);
```

## 결론

- 자원 공유 문제에 대해 다루었다.
- 자원 공유 문제를 해결하기 위해 동시성 기본형인 큐를 만들었다.

## 요약

- 액션간의 타이밍 때문에 버그가 생기는 경우는 재현하기 어렵기 때문에 타임라인 다이어그램을 그리고 분석하면 도움이 될 수 있다.
- 자원 공유를 위한 도구를 동시성 기본형이라고 부르고, 이 동시성 기본형은 더 단순하고 깨끗한 코드에 도움을 준다.
