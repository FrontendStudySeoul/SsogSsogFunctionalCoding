
## 12장. 함수형 반복

### 기존 반복문 로직 forEach 로 변경

```js
function emailsForCustomers(customers, goods, bests) {
  var emails = [];
  for(var i = 0; i < customers.length; i++) {
    var customer = customers[i];
    var email = emailForCustomer(customer, goods, bests);
    emails.push(email);
  }
  return emails;
}

function emailsForCustomers(customers, goods, bests) {
  var emails = [];
  forEach(customers, function(customer) {
    var email = emailForCustomer(customer, goods, bests);
    emails.push(email);
  });
  return emails;
}
```

### map 함수 도출하기

```js
function emailsForCustomers(customers, goods, bests) {
  var emails = [];
  for(var i = 0; i < customers.length; i++) {
    var customer = customers[i];
    var email = emailForCustomer(customer, goods, bests);
    emails.push(email);
  }
  return emails;
}

function biggestPurchasePerCustomer(customers) {
  var purchases = [];
  for(var i = 0; i < customers.length; i++) {
    var customer = customers[i];
    var purchase = biggestPurchase(customer);
    purchases.push(purchase);
  }
  return purchases;
}

// map 함수 도출

function emailsForCustomers(customers, goods, bests) {
  return map(customers, function(customer) {
    return emailForCustomer(customer, goods, bests);
  });
}

function map(array, f) {
  var newArray = [];
  forEach(array, function(element) {
    newArray.push(f(element));
  });
  return newArray;
}
```

### 함수형 도구: map()

```js
function map(array, f) {
  var newArray = [];
  forEach(array, function(element) {
    newArray.push(f(element));
  });
  return newArray;
}
```

- 배열과 함수를 인자로 받아서
- 빈배열을 만든 후
- 원래 배열 항목으로 새로운 항목을 만들기 위해 f() 함수를 호출한 후,
- 원래 배열 항목에 해당하는 새로운 항목을 추가
- 새로운 배열을 리턴

### 함수를 전달하는 세가지 방법
- 전역으로 정의하기
- 지역적으로 정의하기
- 인라인으로 정의하기


### filter 함수 도출하기

```js
function selectBestCustomers(customers) {
  var newArray = [];
  forEach(customers, function(customer) {
    if(customer.purchases.length >= 3)
      newArray.push(customer);
  });
  return newArray;
}

function selectCustomersAfter(customers, date) {
  var newArray = [];
  forEach(customers, function(customer) {
    if(customer.signupDate > date)
      newArray.push(customer);
  });
  return newArray;
}

// filter 함수 도출

function selectBestCustomers(customers) {
  return filter(customers, function(customer) {
    return customer.purchases.length >= 3;
  });
}

function filter(array, f) {
  var newArray = [];
  forEach(array, function(element) {
    if(f(element))
      newArray.push(element);
  });
  return newArray;
}
```

### 함수형 도구: filter()

```js
function filter(array, f) {
  var newArray = [];
  forEach(array, function(element) {
    if(f(element))
      newArray.push(element);
  });
  return newArray;
}
```
- 배열과 함수를 인자로 받아서
- 빈배열을 만든 후
- f() 함수를 호출한 후, 결과 배열에 넣을지 확인
- 조건에 맞으면 원래 항목을 결과 배열에 추가
- 결과 배열을 리턴

### reduce 함수 도출하기

```js
function countAllPurchases(customers) {
  var total = 0;
  forEach(customers, function(customer) {
    total = total + customer.purchases.length;
  });
  return total;
}

function concatenateArrays(arrays) {
  var result = [];
  forEach(arrays, function(array) {
    result = result.concat(array);
  });
  return result;
}

// reduce 함수 도출

function countAllPurchases(customers) {
  return reduce( customers, 0, function(total, customer) {
    return total + customer.purchase.length;
  });
}

function reduce(array, init, f) {
  var accum = init;
  forEach(array, function(element) {
    accum = f(accum, element);
  });
  return accum;
}
```

### 함수형 도구: reduce()

```js
function reduce(array, init, f) {
  var accum = init;
  forEach(array, function(element) {
    accum = f(accum, element);
  });
  return accum;
}
```

- 배열, 초기값, 함수를 인자로 받아서
- 초기값을 누적값으로 설정한 후
- 누적 값을 계산하기 위해, 현재 값과 배열의 항목으로 f() 함수를 호출
- 누적된 값을 리턴  

### 예제: 문자열 합치기
```js
reduce(strings, "", function(accum, string) {
  return accum + string;
});
```

### 연습 문제: min, max
```js
function min(numbers) {
  return reduce(numbers, Number.MAX_VALUE, function(m, n) {
    if(m < n) return m;
    else return n;
  });
}

function max(numbers) {
  return reduce(numbers, Number.MIN_VALUE, function(m, n) {
    if(m > n) return m;
    else return n;
  });
}
```

### 연습 문제: 상황에 따른 map, filter, reduce

map 함수에 빈배열을 넘기면?  
-> []

filter 함수에 빈배열을 넘기면?  
-> []

reduce 함수에 빈배열을 넘기면?  
-> 초기값

map() 함수에 인자를 그대로 리턴하는 함수를 넘기면?  
-> 얕은 복사가 된 array

filter() 함수에 항상 true를 리턴하는 함수를 넘기면?  
-> 얕은 복사가 된 array

filter() 함수에 항상 false를 리턴하는 함수를 넘기면?  
-> []

### Iteration protocols (이터레이션 프로토콜)    

**이터러블 프로토콜**  
이터러블: Symbol.iterator 메소드를 구현하거나 프로토타입 체인에 의해 상속한 객체   
Symbol.iterator 메소드는 이터레이터를 반환  

**이터레이터 프로토콜**  
이터레이터: next 메소드를 소유하며 next 메소드를 호출하면 이터러블을 순회하며 value, done 프로퍼티를 갖는 이터레이터 리절트 객체를 반환하는 객체

```js
const array = [1, 2, 3];

const iterator = array[Symbol.iterator]();

console.log('next' in iterator); // true

console.log(iterator.next()); // {value: 1, done: false}
console.log(iterator.next()); // {value: 2, done: false}
console.log(iterator.next()); // {value: 3, done: false}
console.log(iterator.next()); // {value: undefined, done: true}
```

### 이터레이션을 이용한 reduce 구현
```js
const reduce = (f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]()
    acc = iter.next().value
  }
  for (const a of iter) {
    acc = f(acc, a)
  }
  return acc
}
```
