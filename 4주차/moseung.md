# 일급 함수 I

### 해당 장에서 살펴볼 내용
- **왜 일급 값이 좋은지**
- **문법을 일급 함수로 만드는 방법**
- **고차 함수로 문법을 감싸는 방법**
- **일급 함수와 고차 함수를 사용한 리팩터링 방식**

<br>
- 코드의 냄새 : 함수 이름에 있는 암묵적 인자
- 리팩터링 : 암묵적 인자를 드러내기
- 리팩터링 : 함수 본문을 콜백으로 바꾸기

### 코드의 냄새

```tsx
function setPriceByName (cart,name,price){
    var item = cart[name]
    var newItem = objectSet(item,"price",price)
    var newCart = objectSet(item,name,newItem)
    return newCart
  }

  function setQuantByName (cart,name,quant){
    var item = cart[name]
    var newItem = objectSet(item,"quantity",quant)
    var newCart = objectSet(item,name,newItem)
    return newCart
  }
```

#### 냄새를 맡는 법
- 함수 구현이 거의 똑같습니다.
- 함수 이름이 구현의 차이를 만듭니다.

### 리팩터링: 암묵적 인자를 드러내기
함수 이름의 일부가 암묵적 인자로 사용되고 있다면 암죽적 인자를 드러내기 리팩터링을 사용할 수 있습니다.
<br>
```tsx
function setFieldByName (cart,name,field,value){
    var item = cart[name]
    var newItem = objectSet(item,field,value)
    var newCart = objectSet(item,name,newItem)
    return newCart
  }
```

이제 암묵적인 이름은 인자로 넘길수 있는 값이 되었으며 추후에 값은 변수나 배열을 넣을 수 있습니다.
<br>
그래서 일급이라고 부르며 일급으로 만드는것이 10장의 주제라고합니다.
<br>
인자가 3개에서 총 4개로 늘어났기에 안전하지 않다고 느끼긴 했습니다.

### 일급인 것과 일급이 아닌 것을 구별하기
자바스크립트에는 일급이 아닌 것과 일급인 것이 섞여 있습니다. 이는 다른 언어에서도 마찬가지입니다.
<br>
예를 들어 string, number, object 등드은 변수에 넣을 수 있고 객체의 항복으로 넣을 수 있고 이런것들을 일급이라고 부릅니다.
<br>
하지만 for, +, -, if와같이 함수의 인자로도 넘길수 없고 변수에 넣을 수 없는것을 일급이 아니라고 하며 이런것들을 일급으로 바꾸는 방법을 아는것이 중요합니다.
<br>
아까의 예시에서 함수명 일부를 값처럼 쓸 수 없기에(일급이 아님) 함수명의 일부를 일급인(string)으로 만들었습니다.

### 필드명을 문자열로 사용하면 버그가 생기지 않을까요?
필드명을 문자로 사용하면 일을 망치지 않을까라고 걱정할 수 있습니다.
<br>
왜냐면 필드명에 대한 명세가 없기에 언제든 오류가 발생할 수 있으며 이를 해결할 두가지 방법이 컴파일 타임에 검사하는것과 런타임에 검사하는 방법이 있습니다.<br>
우선 컴파일에 검사하는 방법은 타입스크립트 같은 것을 사용할 수 있습니다.
```tsx
type Field =  "모승"|"승모"|"moseung"

  function setFieldByName (cart,name,field:Field,value){
    var item = cart[name]
    var newItem = objectSet(item,field,value)
    var newCart = objectSet(item,name,newItem)
    return newCart
  }
```

<img width="438" alt="image" src="https://github.com/FrontendStudySeoul/SsogSsogFunctionalCoding/assets/103626175/29d0277d-b724-423a-8447-863265026a98">

두번째로 런타임에 검사하는 방법입니다.
```tsx
const Field = ["모승","승모","moseung"]

  function setFieldByName (cart,name,field,value){
if(!Field.includes(field)){
throw "not a valid seungmo"
}
    var item = cart[name]
    var newItem = objectSet(item,field,value)
    var newCart = objectSet(item,name,newItem)
    return newCart
  }
```

### 정적 타입 vs 동적 타입
[정적 타입 언어 동적 타입 언어](https://jusung.github.io/Static-and-Dynamic-Typed-Language/)
이를 통해 데이터는 결국 항상 해석이 필요하다는 단점이 보입니다.

### 반복문 예제: 먹고 치우기
```tsx
for(let i =0 ;i<foods.length;i++){
    let food = foods[i];
    cook(food)
    eat(food)
  }

  for(let i =0 ;i<dishes.length;i++){
    let food = dishes[i];
    wash(food)
    dry(food)
    putAway(food)
  }
```
코드가 비슷하지만 두 반복문은 하는 일이 다릅니다. 첫번째는 음식을 먹고 두번쨰는 음식을 먹고난 이후 치우는 과정입니다.
<br>

```tsx
function forEach(array,f){
for(let i =0 ;i<array.length;i++){
    let item = array[i];
    f(item)
  }
}
//

array.forEach((item)=>f(item))
```

### 리패터링: 함수 본문을 콜백으로 바꾸기
```tsx
try {
    saveUserData(user)
  } catch (error) {
    logToSnapErrors(error)
  }


try {
    fetchProduct(productId)
  } catch (error) {
    logToSnapErrors(error)
  }

```
이 두개에서 중복되는 log부분을 해결할 수 있을것 같아 코드를 리팩토링 했습니다.

```tsx
  function withLogging(f){
    try {
      f()
    } catch (error) {
      logToSnapErrors(error)
    }
  }

  withLogging(function(){
    saveUserData(user)
  })
```

### 왜 본문을 함수로 감싸서 넘기나요?

함수의 정의 방법이 3가지가 있습니다.
1. 전역으로 정의하기
2. 지역적으로 정의하기
3. 인라인으로 정의하기

위 방법은 문맥에서 한번만 사용하는 인라인을 정의하여 함수를 넘긴 기법이며 이를 통해 함수의 실행 시점을 미룰 수 있습니다.
<br>
**고차 함수(Higher order function)는 함수를 인자로 전달받거나 함수를 결과로 반환하는 함수를 말한다. 다시 말해, 고차 함수는 인자로 받은 함수를 필요한 시점에 호출하거나 클로저를 생성하여 반환한다.**

#### 만약 감싸지 않았으면 어떻게 될까요?
```tsx
  function withLogging(f){
    try {
      f()
    } catch (error) {
      logToSnapErrors(error``)
    }
  }

  withLogging(
    saveUserData(user)
  )
```

# 일급 함수 II
이 장에서 살펴볼 내용
- 함수 본문을 콜백으로 바꾸기 리팩터링에 대한 심화
- 함수를 리턴하는 함수가 가진 강력한 힘을 이해
- 고차 함수에 익숙해지기 위해 여러 고차 함수를 만들기


### 코드 냄새 하나와 리팩터링 두 개
- 코드의 냄새: 함수 이름에 있는 암묵적 인자
- 리팩터링: 암묵적 인자를 드러내기
- 리팩터링: 함수 본문을 콜백으로 바꾸기

앞서 10장에서 위와 같은 방법을 배웠습니다.

### 배열에 대한 카피온라이트 리팩터링
```tsx
function arraySet(array,idx,value){
    let copy = array.slice() //앞부분
    copy[idx] = value //본문
    return copy // 뒷부분
  }

  function drop_last(array){
    let copy = array.slice() //앞부분
    copy.pop() //본문
    return copy // 뒷부분
  }
```
위 두 함수에서 똑같이 배열을 복사하고 어떤 동작을 한 후 해당 배열을 리턴합니다.
```tsx
function arraySet(array,idx,value){
    withArrayCopy(array,function(copy){
      copy[idx]=value
    })
  }

  function withArrayCopy(array,modify){
    let copy = array.slice();
    modify(copy)
    return copy
  }
```

어떤 경우에는 리팩터링을 하고 중복을 없애면 코드가 짧아집니다. 하지만 이 경우는 그런 장점은 없어졌습니다. 하지만 이제 똑같은 코드를 이곳 저곳에서 작성할 필요가 없어졌기에 총 코드량은 줄일수 있는 이점을 얻었습니다.

### withLogging()함수 리팩토링
앞서 똑같은 tryCatch구문을 해결하기 위한 함수가 있었지만 이를 더 일반적인 함수로 바꾸고 싶었습니다.
```tsx
function withLogging(f){
    try {
      f()
    } catch (error) {
      logToSnapErrors(error)
    }
  }
```

```tsx
function tryCatch(f,errorHandler){
 try {
      return f()
    } catch (error) {
      return errorHandler(error)
    }
}
```

### 함수를 리턴하는 함수
```tsx
function withLogging(f){
    try {
      f()
    } catch (error) {
      logToSnapErrors(error)
    }
  }
withLogging(function(){
saveuserData()
})

```
기존 함수도 잘 리팩터링 됐지만 여전히 두가지 문제가 있습니다.
<br>
- 특정 호출에서 withLogging을 붙이는 걸 까먹을 수 있다.
- 그럼에도 불구하고 모든 withLogging으로 래핑해야하는 번거로움이 있다.

그래서 이를 해결하고자 에러를 잡아 로그를 남길수 있는 기능이 추가된 함수를 일반함수처럼 그냥 호출할 수 있으면 좋겠습니다. 그리고 그런 함수를 자동으로 만들어주는 함수가 있으면 더 좋을 것 같습니다.
<br>
```tsx
function saveUserDataWithLogging(user){//앞부분
    try {
      saveUserDataWithNoLogging(user)//본문
    } catch (error) {
      logToSnapErrors(error)//뒷부분
    }
  }

function fetchProductWithLogging(productId){//앞부분
    try {
      fetchProductWithNoLogging(productId)//본문
    } catch (error) {
      logToSnapErrors(error)//뒷부분
    }
  }
```

하지만 여전히 중복이 존재합니다.

```tsx
function wrapLogging(f){
return function(arg){
try {
      f(arg)
    } catch (error) {
      logToSnapErrors(error)
    }
}
}

const saveUserDataWithLogging = wrapLogging(saveDataUserDataNoLogging)
```
위와 같은 리팩토링을 통해 중복 부분을 없애고 로그를 남기지 않는 버젼을 로그를 남기는 버젼으로 쉽게 바꿀 수 있습니다.<br>
이를 통해 자동으로 정형화된 코드를 함수로 만들 수 있게 됐습니다.

### 다른 숫자에 어떤 숫자를 더하는 함수(예제)
```tsx
function makeAdder(n){
return function(x){
  return n+x
}
}
```
위처럼 함수를 넘기는 게 아니라 어떤 일급이든 가능하다.

## 결론
일급 값, 일급 함수, 이를 이용한 고차 함수가 있으며 이를 통해 코드의 중복을 없앨 수 있고 기존에 특정 기능을 더한 새로운 함수르 쉽게 만들 수 있다.<br>
그리고 일급은 언어에서 변수나 인자 객체 안에 담을 수 있는 모든값이다.
