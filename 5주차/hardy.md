## 반응형 아키텍처

반응형 아키텍처는 코드에 나타난 순차적 액션의 순서를 뒤집습니다. 앞으로 살펴보겠지만 효과와 그 효과에 대한 원인을 분리해서 코드에 복잡하게 꼬인 부분을 풀 수 있습니다. 핵심 원칙은 이벤트에 대한 반응으로 일어날 일을 지정하는 것입니다. 

### 절충점

반응형 아키텍처를 사용하면 코드를 읽기 쉽고 유지보수하기도 좋습니다. 하지만 꼭 그렇지는 않습니다. 언제 사용하고 어떻게  사용할지는 각자 판단해야 합니다.

**원인과 효과가 결합한 것을 분리합니다.**

어떤 경우에는 원인과 효과를 분리하면 코드가 읽기 어려워집니다. 하지만 코드가 더 유연하고 하려고 하는 것을 정확하게 표현할 수 있습니다.

**여러 단계를 파이프라인으로 처리합니다.**

앞에서 데이터를 변환하는 단계를 파이프라인으로 처리했습니다. 파이프라인은 함수형 도구를 연결해서 만들었습니다. 계산을 조합하는 정말 멋진 방법이었습니다. 반응형 아키텍처로 이와 비슷하게 액션과 계산을 조합할 수 있습니다.

**타임라인이 유연해집니다.**

순서를 표현하는 방법을 뒤집으면 타임라인이 유연해집니다. 물론 이러한 유연성이 기대하지 않은 실행 순서로 이어진다면 좋지 않을 수도 있습니다. 하지만 익숙해지면 더 짧은 타임라인을 만들 수 있습니다.

### 일급 함수를 반응형으로

```jsx
var shopping_cart = {};

function add_item_to_cart(name, price) {
  var item = make_cart_item(name, price);
  shopping_cart = add_item(shopping_cart, item);

  var total = calc_total(shopping_cart);
  set_cart_total_dom(total);
  update_shipping_icons(shopping_cart);
  update_tax_dom(total);
}
```

장바구니는 언제 바뀔지 모릅니다. 전역 변수이고 할당 연산자를 통해 값을 바꾸고 있습니다. 상태를 일급 함수로 만들어 봅시다. 전역변수를 몇 가지 동작과 함께 객체로 만듭니다. 다음은 변경 가능한 값을 일급 함수로 만드는 코드입니다.

```jsx
function ValueCell(initialValue) {
  var currentValue = initialValue;
  return {
    val: function() {
      return currentValue;
    },
    update: function(f) {
      var oldValue = currentValue;
      var newValue = f(oldValue);
      currentValue = newValue;
    }
  }
}

var shopping_cart = ValueCell({});

function add_item_to_cart(name, price) {
  var item = make_cart_item(name, price);
  shopping_cart.update(function(cart) {
    return add_item(cart, item);
  });

  var total = calc_total(shopping_cart.val());
  set_cart_total_dom(total);
  update_shipping_icons(shopping_cart.val());
  update_tax_dom(total);
}
```

ValueCell 코드에 감시자(옵저버) 개념을 추가해 상태가 바뀔 때마다 실행되도록 핸들러 함수를 추가합니다.

```jsx
function ValueCell(initialValue) {
  var currentValue = initialValue;
  var watchers = [];
  return {
    val: function() {
      return currentValue;
    },
    update: function(f) {
      var oldValue = currentValue;
      var newValue = f(oldValue);

      if(old !== newValue) {
        currentValue = newValue;
        watchers.forEach(function(watcher) {
          watcher(newValue, oldValue);
        }); 
      }
    },
    addWatcher: function(watcher) {
      watchers.push(watcher);
    }
  }
}

var shopping_cart = ValueCell({});

function add_item_to_cart(name, price) {
  var item = make_cart_item(name, price);
  shopping_cart.update(function(cart) {
    return add_item(cart, item);
  });

  var total = calc_total(shopping_cart.val());
  set_cart_total_dom(total);
  update_tax_dom(total);
}

shopping_cart.addWatcher(update_shipping_icons);
```

코드를 고치고 나니 두 가지 사실을 알 수 있습니다. 하나는 핸들러 함수가 작아져 원래 하던 것보다 적은 일을합니다. 그리고 아이콘 갱신을 직접 하지 않습니다. 책임은 감시자로 넘어갔습니다. 또 다른 사실은 장바구니를 바꾸는 모든 핸들러에서 update_shipping_icons()를 부르지 않아도 된다는 것입니다. 이제 장바구니가 변경될 때 항상 update_shipping_icons()가 실행됩니다.

### FormulaCell은 파생된 값을 계산합니다

어떤 셀은 다른 셀의 값을 최신으로 반영하기 위해 파생될 수 있습니다. FormulaCell로 이미 있는 셀에서 파생한 셀을 만들 수 있습니다. 다른 셀의 변화가 감지되면 값을 다시 계산합니다.

```jsx
function FormulaCell(upstreamCell, f) {
  var myCell = ValueCell(f(upstreamCell.val()));
  upstreamCell.addWatcher(function() {
    myCell.update(function() {
      return f(upstreamCell.val());
    });
  });
  return {
    val: myCell.val,
    addWatcher: myCell.addWatcher
  }
}

var shopping_cart = ValueCell({});
var cart_total = FormulaCell(shopping_cart, calc_total);

function add_item_to_cart(name, price) {
  var item = make_cart_item(name, price);
  shopping_cart.update(function(cart) {
    return add_item(cart, item);
  });
}

shopping_cart.addWatcher(update_shipping_icons);
cart_total.addWatcher(set_cart_total_dom);
cart_total.addWatcher(update_tax_dom);
```

FormulaCell은 값을 직접 바꿀 수 없습니다. 감시하던 상위(upstream) 셀 값이 바뀌면 FormulaCell 값이 바뀝니다. 상위 셀이 바뀌면 상위 값을 가지고 셀 값을 다시 계산합니다. FormulaCell에는 값을 바꾸는 기능은 없지만 FormulaCell을 감시할 수 있습니다. FormulaCell을 감시할 수 있기 때문에 FormulaCell이 total 값이라면 total 값이 바뀔 때 실행할 액션을 추가할 수 있습니다.

### **반응형 아키텍처가 시스템을 어떻게 바꿨나요**

1. **원인과 효과가 결합한 것을 분리합니다.**

![image](https://github.com/FrontendStudySeoul/SsogSsogFunctionalCoding/assets/59603529/c2282391-bb7b-44e8-8e9e-4de32f9725b4)

일반적인 아키텍처에서는 장바구니를 바꾸는 모든 UI 이벤트 핸들러에 같은 코드를 넣어줘야 합니다. 반응형 아키텍처를 사용하면 원인과 효과가 결합한 것을 분리할 수 있습니다. 어떤 원인에 의해 장바구니가 변경되더라도 배송 아이콘을 갱신합니다.

![image](https://github.com/FrontendStudySeoul/SsogSsogFunctionalCoding/assets/59603529/81f170a0-a776-4c1c-aad6-a5127473ed30)

배송 아이콘 갱신은 한 번만 만들면 됩니다. 정확히 말하면 어떤 이유로든 장바구니가 바뀔 때 배송 아이콘을 갱신합니다.

1. **************************************************************************************************여러 단계를 파이프라인으로 처리합니다.**************************************************************************************************

반응형 아키텍처도 간단한 액션과 계산을 조합해 복잡한 동작을 만들 수 있습니다. 조합된 액션은 파이프라인과 같습니다. 데이터가 파이프라인으로 들어가 각 단계에서 처리됩니다. 파이프라인은 작은 액션과 계산을 조합한 하나의 액션이라고 볼 수 있습니다.

1. **************************************************************타임라인이 유연해집니다.**************************************************************

![image](https://github.com/FrontendStudySeoul/SsogSsogFunctionalCoding/assets/59603529/bc4e7eec-afeb-4c31-a99c-c6db4dc6c64c)

반응형 아키텍처를 사용하면 타임라인이 유연해집니다. 순서를 정의하는 방법을 뒤집기 때문에 자연스럽게 타임라인이 작은 부분으로 분리됩니다.

## 어니언 아키텍처

어니언 아키텍처는 현실 세계와 상호작용하기 위한 서비스 구조를 만드는 방법입니다.

![image](https://github.com/FrontendStudySeoul/SsogSsogFunctionalCoding/assets/59603529/112a8850-7cc3-46f0-85d9-3a9efae82e54)

어니언 아키텍처는 특정 계층이 꼭 필요하다고 강제하지 않습니다. 많은 경우에 위와 같이 3가지 큰 분류로 나눌 수 있습니다. 그림으로 함수형 시스템이 잘 동작할 수 있는 중요한 규칙을 알 수 있습니다.

1. 현실 세계와 상화작용은 인터랙션 계층에서 해야합니다.
2. 계층에서 호출하는 방향은 중심 방향입니다.
3. 계층은 외부에 어떤 계층이 있는지 모릅니다.

### 전통적인 계층형 아키텍처

![image](https://github.com/FrontendStudySeoul/SsogSsogFunctionalCoding/assets/59603529/b91e641a-3a91-41f7-8694-b87e9f28123a)

전통적인 계층형 아키텍처는 데이터베이스를 기반으로 합니다. 도메인 계층은 데이터베이스 동작으로 만듭니다. 그리고 웹 인터페이스는 웹 요청을 도메인 동작으로 변환합니다. 

이런 아키텍처는 함수형 스타일이 아닙니다. 데이터베이스 계층이 가장 아래 있다면 그 위에 있는 모든 것이 액션이 되기 때문에 함수형 스타일이 아닙니다. 모든 것이 계층에 쌓여있고 계산은 따로 관리되지 않고 우연히 사용됩니다. 함수형 아키텍처는 계산과 액션에 대한 명확한 규칙이 있어야합니다.

### 함수형 아키텍처

![image](https://github.com/FrontendStudySeoul/SsogSsogFunctionalCoding/assets/59603529/17ab9752-85a7-4198-87ec-6d4cfa8da323)

함수형 아키텍처는 도메인 계층이 데이터베이스 계층에 의존하지 않는 것이 전통적인 계층형 아키텍처와의 차이점입니다. 데이터베이스 동작은 값을 바꾸거나 데이터베이스에 접근하기 때문에 액션입니다. 그림에서 액션과 계산을 구분하는 선을 그리고 라이브러리나 언어 기능과 계산을 구분하는 선을 그려 함수형 아키텍처를 표현할 수 있습니다.

함수형 개발자는 액션과 계산을 명확하게 구분하려고하고 도메인 로직은 모두 계산으로 만들어야 한다고 생각합니다. 따라서 데이터베이스를 도메인과 분리하는 것이 중요합니다. 가장 위에 있는 액션에서 도메인 규칙과 데이터베이스를 조합합니다.

위의 함수형 아키텍처의 각 점선을 끝으로 이으면 어니언 아키텍처가 됩니다.

### 변경과 재사용이 쉬워야 합니다.

어니언 아키텍처는 인터랙션 계층을 바꾸기 쉽습니다. 인터랙션 계층은 가장 위에 있어서 가장 바꾸기 쉽습니다. 도메인이 데이터베이스나 웹 요청에 의존하지 않기 때문에 인터랙션 계층에 속하는 데이터베이스나 서비스 프로토콜은 쉽게 바꿀 수 있습니다. 그리고 도메인 계층도 데이터베이스나 서비스 같은 것을 사용하지 않으므로 전부 계산으로 만들 수 있습니다. 

### 도메인 규칙은 도메인 용어를 사용합니다.

프로그램의 핵심 로직을 도메인 규칙 또는 비즈니스 규칙이라고 합니다. 모든 로직이 도메인 규칙이 아니므로 어떤 로직이 도메인 규칙인지 판단하기 위해 코드에 나타나는 용어를 참고할 수 있습니다. 다음과 같이 어떤 데이터베이스를 사용할지 결정하는 코드가 있습니다.

```tsx
var image = newImageDB.getImage('123');
if(image === undefined) {
  image = oldImageDB.getImage('123');
}
```

코드가 비즈니스에 중요한 부분이라고 해도 이 코드는 도메인 규칙이 아닙니다. 도메인 용어를 쓰지 않기 때문입니다. 도메인 규칙에는 제품(product), 이미지(image), 가격(price) 등의 용어를 사용합니다. 데이터베이스는 도메인을 나타내는 용어가 아닙니다. 이 코드는 바깥세상과 상호작용하는 인터랙션 계층에 속하는 코드입니다.

```tsx
function getWithRetries(url, retriesLeft, success, error) {
	if(retriesLeft <= 0) {
    error('No more retries');
	} else {
    ajaxGet(url, success,function(e) {
      getWithRetries(url, retriesLeft - 1, success, error);
    })
	}
}

```

위 코드는 웹 요청이 실패할 때 재시도를 하는 로직입니다. 재시도가 비즈니스에 중요한 기능이라고 해도 비즈니스 규칙이 아닙니다. 마찬가지로 도메인 용어를 쓰지 않기 때문입니다. 이 코드는 불안정한 네트워크 문제를 해결하기 위한 코드입니다. 역시 인터랙션 계층에 속하는 코드입니다.

### 가독성을 따져 봐야 합니다.

도메인을 항상 계산으로 만들 수 있지만, 어떤 경우는 문맥에 따라 계산보다 액션이 읽기 좋은 경우가 있습니다.

가독성을 결정하는 요소는 여러 가지 있습니다. 다음은 가독성을 결정하는 몇 가지 요소입니다.

- 사용하는 언어
- 사용하는 라이브러리
- 레거시 코드와 코드 스타일
- 개발자들의 습관

앞에서 이야기한 어니언 아키텍처는 가장 이상적인 모습입니다. 100% 순수한 어니언 아키텍처를 만들면 된다고 쉽게 생각할 수 있습니다. 하지만 세상에 완벽한 것은 없습니다. 설계자의 역할 중 하나는 현실 세계의 문제와 이상적인 다이어그램 사이를 균형 있게 유지하는 것입니다.
