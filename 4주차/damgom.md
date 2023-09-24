- 배열을 효과적으로 다루기 위해 map, filter, reduce 등을 살펴보았다.
- 이번 장에는 객체를 다룰 수 있는 함수형 도구에 대해 알아본다.

### copy-on-write로 객체 값 변경하기
``` js
const incrementSize = (item) => {
  const size = item['size'];
  const newSize = size + 1;
  const newItem = objectSet(item, 'size', newSize);
  return newItem;
}
```

#### 문제점: 함수명에 있는 암묵적 인자
1. 함수명에 필드명: `size` 
2. 함수명에 있는 동작: `increment`

### 함수형 도구:  update
- `size` -> `key`: 명시적인 인자로 변경
- `increment` -> `modify`: 콜백으로 바꾸기

```js
/**
 * 값이 변경된 새로운 객체를 반환
 * @param object 
 * @param key 변경할 값이 어디 있는지 알려주는 키
 * @param modify 값을 변경하는 함수
 */

function update(object, key, modify) {
  const value = object[key];
  const newValue = modify(value);
  const newObject = objectSet(object, key, newValue);
  return newObject;
}
```

- 덧셈 등의 연산을 modify 함수에게 위임

``` js
update(item, 'size', size => size + 1)

// 계산: 사이즈 계산
// 액션: 상태 변경
```

---

## 중첩된 객체에 update 적용하기

![image](https://github.com/FrontendStudySeoul/SsogSsogFunctionalCoding/assets/80238096/913946fd-d48c-4099-b458-0d2c800a6be4)


``` js
function incrementSize(item) {
  return update(item, 'options', function (options) {
    return update(options, 'size', increment);
  }
}

const incrementSize = (item) => update(item, 'options', (options) => update(options, 'size', increment))
```

### 리팩토링 1: 암묵적 인자

- 암묵적 인자: size, increment 
  - `size` -> `option`
  - `increment` -> `modify`

``` js
function updateOption(item, option, modify) {
  return update(item, 'options', function (options) {
    return update(options, 'size', modify);
  }
}

const updateOption = (item, option, modify) => update(item, 'options', (options) => update(options, 'size', modify))
```

### 리팩토링 2: 암묵적 인자
- `option`

``` js
function update2(object, key1, key2, modify) {
  return update(object, key1, function (value) {
    return update(object, key2, modify)
  })
}

const update2 = (object, key1, key2, modify) => update(object, key1, (value) => update(object, key2, modify))
```

``` js
const incrementSize = (item) => update2(item, 'options', 'size', (size) => size + 1)
```

---

## updateN 도출하기
``` js
function update0(value, modify) {
  return modify(value);
}

function update1(object, key1, modify) {
  return update(object, key1, function(value1) {
    return update0(value1, modify);
  });
}

function update2(object, key1, key2, modify) {
  return update(object, key1, function(value1) {
    return update1(value1, key2, modify);
  });
}

function update3(object, key1, key2, key3, modify) {
  return update(object, key1, function(value1) {
    return update2(value1, key2, key3, modify);
  });
}

function update4(object, key1, key2, key3, key4, modify) {
  return update(object, key1, function(value1) {
    return update3(value1, key2, key3, key4, modify);
  });
}
```

- 중첩된 단계만큼 update를 호출해야한다

### 1. depth 인자 추가하기

``` js
function updateX(object, depth, key1, key2, key3, modify) {
  return update(object, key1, function(value1) {
    return updateX(value1, depth-1, key2, key3, modify);
  });
}
```

#### 문제점
- depth와 key 개수를 맞출 수 없다.

### 2. 모든 키를 배열로 넘기기
``` js
function updateX(object, keys, modify) {
  var key1 = keys[0]; // 첫번째로 key로 update 호출 
  var restOfKeys = drop_first(keys); // 나머지 key로 재귀 호출

  return update(object, key1, function(value1) {
    return updateX(value1, restOfKeys, modify);
  });
}

```

#### 문재점
- depth가 0인 객체 (중첩되지 않은 객체)는 updae를 호출하지 않는다

### 3. 0인 경우를 처리

``` js
function nestedUpdate(object, keys, modify) {
  if(keys.length === 0)
    return modify(object);

  var key1 = keys[0];
  var restOfKeys = drop_first(keys);
  return update(object, key1, function(value1) {
    return updateX(value1, restOfKeys, modify);
  });
}
```

---

## 재귀적 사고방식

### 베열
![image](https://github.com/FrontendStudySeoul/SsogSsogFunctionalCoding/assets/80238096/53dfa5c3-901f-47d7-839f-12761c971f12)


### 객체
![image](https://github.com/FrontendStudySeoul/SsogSsogFunctionalCoding/assets/80238096/fb75067b-d184-40d3-8dbe-f18ddf225be3)


---

## 깊이 중첩된 데이터에 추상화 벽 사용하기

### 문제점
> blog에 12번째 글을 가져와 글쓴이 이름을 대문자로 바꾼다
``` js
httpGet("https://my-blog.com/api/category/blog", function (blogCategory) {
  renderCategory(nestedUpdate(blogCategory, ['posts', '12', 'author', 'name'], capitalize);
})
```

- 키 경로가 길면 중간 객체가 어떤 키를 가졌는지 기억하기 어렵다.

### 추상화 벽 사용하기

``` js
function updatePostById(category, id, modifyPost) {
  return nestedUpdate(category, ['posts', id], modifyPost);
}

function updateAuthor(post, modifyUser) {
  return update(post, 'author', modifyUser);
}

function capitalizeName(user) {
  return update(user, 'name', capitalize);
}

// 사용
updatePostById(blogCategory, '12', function(post) {
  return updateAuthor(post, capitalizeUserName);
});
```

- 기억해야할 키 값이 줄어들었다.
- 블로그 글에 글쓴이가 있다는 것만 알면 되고 어떻게 저장되어있는지는 몰라도 된다.
- 동작의 이름으로 각각의 동작을 기억하기 쉽다.

