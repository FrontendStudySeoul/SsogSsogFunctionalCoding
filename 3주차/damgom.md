## 6장 변경 가능한 데이터 구조를 가진 언어에서 불변성 유지하기
### 동작을 읽기, 쓰기 또는 둘 다로 분류하기
#### 읽기
- 데이터를 바꾸지 않고 정보를 꺼내는 것

#### 쓰기
- 데이터를 바꾼다
   - 객체, 배열을 사용하면 자바스크립트는 `pass by reference` 방식을 사용하기 때문에 외부에서 값이 수정되거나, 함수 내부에서 외부에 영향을 미칠 수 있다.
- 바뀌는 값이 어디서 사용될지 모르기 때문에 바뀌지 않도록 원칙이 필요하다.

<br />

### copy-on-write
- `pass by reference` -> `pass by value`
- 값을 복사해서 수정하면 원본을 바꾸지 않을 수 있다.

 
``` js
/* copy-on-write 3가지 원칙 */

function addElementLast(array, elem) {
  // 1. 복사본 만들기
  const newArray = array.slice();
  // 2. 복사본 변경하기
  newArray.push(elem);
  // 3. 복사본 리턴하기
  return newArray;
}

// spread 표기법을 사용하면 더 간결하게 작성할 수 있다.
const addElementLast = (array, elem) => [...array, elem]
```

- 데이터를 바꾸지 않고 정보를 리턴하기 때문에 읽기 동작
- copy-on-write를 이용해 액션을 계산으로 만들 수 있다.

객체에도 copy-on-write를 적용할 수 있다.
```js
// 원래 코드
function setPriceOrigin(item, newPrice) {
  item.price = newPrice;
}

// copy-on-write
function setPriceCopyOnWrite(item, newPrice) {
  const copiedItem = Object.assign({}, item);
  copiedItem.price = newPrice;
  return copiedItem;
}

const setPriceCopyOnWirte = (item, newPrice) => ({...item, price: newPrice})
```

#### 중첩된 객체에 대한 copy-on-write
``` js
// 원본값을 직접 수정하는 액션
const someMutationAction = (obj) => {
  obj.foo.bar.baz = 200
}

// 계산
const someCalcuation = (obj, value = 200) => {
  return {
    ...obj,
    foo: {
      ...obj.foo,
      bar: {
        ...obj.foo.bar,
        baz: value
      }
    }
  }
}
```

``` js
// immutable-js
// Object, Array대신 불변객체를 제공하는 라이브러리
const obj = Map({foo: {bar: baz: 200}}});
obj.setIn(['foo', "bar", "baz"], 50);

                                      
// immer.js
// proxy를 이용한 라이브러리
const nextState = produce(obj, draft => draft.foo.bar.baz = 200)
```

---

### 불변 데이터 구조에 대한 오해와 사실
> 변경 가능한 데이터 구조보다 메모리를 더 많이 쓰고 느리다.

불변 데이터 구조를 사용하면서 대용량 고성능 시스템을 구현하는 사례는 많다. 즉, 일반 어플리케이션이 쓰기 충분히 빠르다.

#### 1. 언제든 최적화할 수 있다
   - 섣부른 최적화는 하지 않을 것, 불변 데이터 구조를 사용하고 속도가 느린 부분이 있으면 그 때 최적화
#### 2. 가비지 콜렉터는 매우 빠르다
  - 대부분의 언어에서는 가비지 콜렉터 성능이 꾸준히 개선되었다.
#### 3. 생각보다 많이 복사하지 않는다
   - 얕은 복사: 같은 메모리를 가르키는 참조에 대한 복사본을 만든다.
#### 4. 함수형 프로그래밍 언어에는 빠른 구현체가 있다
  - 얕은 복사: 데이터 구조를 복사할 때 최대한 많은 구조적 공유
  - 더 적은 메모리를 사용하고 가바지 콜렉터의 부담을 줄여준다.  
