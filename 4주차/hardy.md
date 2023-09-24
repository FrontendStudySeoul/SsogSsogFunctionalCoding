### 체이닝

함수형 도구로 여러 단계를 체인 형태로 조합해 복잡한 계산을 작고 명확한 단계로 표현할 수 있습니다.

**요구 사항**

> 우수 고객이 가장 많은 비용을 쓸 것으로 생각하고 있습니다. 그래서 각각의 우수 고객(3개 이상 구매)의 구매 중 가장 비싼 구매를 알려주세요
> 

요구 사항을 단계로 나누면

1. 우수 고객(3개 이상 구매)을 거릅니다.
2. 우수 고객을 가장 비싼 구매로 바꿉니다.

```jsx
funtion biggestPurchesesBestCustomers(custormers) {
	// 우수 고객을 거릅니다.
	var bestCustormers = filter(custormers, function(custormer) {
		return customer.purchases.length >= 3;
	})
	
	// 우수 고객을 가장 비싼 구매로 바꿉니다.
	var biggestPurchases = map(bestCustomers, function(customer) {
		return reduce(custormer.purchases, { total: 0 }, function(biggestSoFar, purchase) {
			if(biggestSoFar.total > purchase.total)
				return biggestSoFar;
			else
				return purchase;
		})
	})

	return biggestPurchases
}
```

요구 사항을 체인 형식으로 작성했지만 콜백 함수 여러개가 중첩되어 함수가 너무 복잡해졌습니다. 체인을 명확하게 만들기 위해 두 가지 방법이 있습니다. 

**1. 단계에 이름 붙이기**

```jsx
funtion biggestPurchesesBestCustomers(custormers) {
	var bestCustormers = selectBestCustormers(custormers);
	var biggestPurchases = getBiggestPurchases(bestCustormer);
	return biggestPurchases
}
```

각 단계를 함수로 쪼개 이름을 붙이면 훨씬 명확해집니다. 

**2. 콜백에 이름 붙이기**

```jsx
funtion biggestPurchesesBestCustomers(custormers) {
	var bestCustormers = filter(custormers, isGoodCustomer);
	var biggestPurchases = map(bestCustomers, getBiggestPurchase);

	return biggestPurchases
}
```

단계에 이름을 붙이는 대신 콜백을 빼내 이름을 붙입니다. 

두 방법을 비교해보면 고차 함수를 그대로 쓰는 첫 번째 방법보다 콜백 함수에 이름을 붙이는 방법이 재사용하기 더 좋아 보입니다.
