---
layout: post
title:  "[자료구조/ 알고리즘] DP"
date: 2024-11-04 14:58:13 +0900
categories: Algorithm
tags : Algorithm, Data Structure

---

## DP 동적 프로그래밍


## 동적 프로그래밍  
동적 프로그래밍은 복잡한 문제를 더 작은 하위 문제로 나누고 각 문제를 한 번만 해결하여 해결하는 알고리즘 기법 (프로그래밍 패러다임)


## history
동적 프로그래밍의 개념은 1950년대에 여러 단계 또는 레벨을 가진 시스템을 최적화하기 위한 방법으로 개발한 Richard Bellman에 의해 처음 소개되었다.

## why use dp?

 ### 장점

* 동적 프로그래밍은 하위 문제가 겹치는 문제를 해결하는 데 사용할 수 있어 다른 알고리즘보다 효율적.
* 문제를 더 작은 부분으로 나누어 최적의 해를 찾을 수 있으므로 최적화가 필요한 문제를 해결하는 데에도 유용하다.  
* 동적 프로그래밍은 피보나치 수열과 같은 간단한 문제부터 스케줄링 및 자원 할당과 같은 복잡한 문제까지 다양한 문제에 적용할 수 있다.  

### 단점  

* 동적 프로그래밍의 가장 큰 단점은 모든 하위 문제와 그 해결책을 저장하는 데 많은 메모리가 필요하다는 점입니다. 따라서 대규모 문제에는 비실용적일 수 있습니다.  
* 또한 문제에 최적의 해결책이 있다고 가정하는데, 항상 그렇지 않을 수도 있다 대게 하위 문제가 반복되는 경우 사용한다.


Fibonacci 의 예를 들자면
Fib(n) = fib(n-1)+ fib(n-2) 이를 재귀함수로 풀 경우 무려 O(2의 n제곱)이 나오게된다. 이미 수행된 연산을 반복적으로 하기 때문인데, 이를 쉽게 풀기위한 방식이 DP이다.

DP의 개념은 쉽게말해 이전의 연산결과를 저장해 다음 연산에 활용하는 Memoization, Tabulation 방식을 사용하는 것이다.


### memoization (top down)

동적 프로그래밍에서 사용되는 기술로, 비용이 많이 드는 함수 호출의 결과를 캐시하여 나중에 반복할 필요가 없도록 하는 것. 이렇게 하면 각 하위 문제를 해결하는 데 필요한 횟수를 줄여 프로그램의 성능을 크게 향상시킬 수 있다. (ex. 피보나치 )

- 하향식 접근 방식  
- 함수 호출 결과 캐시    
- 상대적으로 작은 입력 집합이 있는 문제에 적합  
- 하위 문제에 하위 문제가 겹치는 경우 사용



### tabulation (bottom up)

하위 문제의 결과를 테이블에 저장하고 전체 문제를 해결할 때까지 이 결과를 사용하여 더 큰 하위 문제를 해결하는 상향식 접근 방식입니다. 문제를 일련의 하위 문제로 정의할 수 있고 하위 문제가 겹치지 않을 때 사용됩니다. 표 작성은 일반적으로 반복을 사용하여 구현되며 대규모 입력 세트가 있는 문제에 적합합니다.

상향식 접근 방식  
하위 문제의 결과를 테이블에 저장합니다.  
반복 구현

- 입력이 많은 문제에 적합.  
- 하위 문제가 겹치지 않을 때 사용.  


## Coin change DP
특정 동전들로 잔돈을 교환하는 방법을 찾는다.

```cpp
int minNumbercoinsForChange(int m , const vector<int>& denoms) {

	vector<int> dp(m + 1, 0);
	dp[0] = 0;

	//잔돈이 1부터 m 까지 증가할때 잔돈의 계산횟수를 dp에 저장
	for (int i = 1; i <= m; i++){
		dp[i] = INT_MAX;

		for (int c : denoms ){
			//bottom to top 문제
			if (i - c >= 0 and dp[i - c] != INT_MAX) {
				//dp[i] 코인의 해결 값은 = dp[i]와 dp[i-c] (코인을 뺀 결과에서 횟수+1) 중 최소값
				dp[i] = min(dp[i], dp[i - c] + 1);
			}
		}
	}
	return dp[m] == INT_MAX? -1 : dp[m];
}
```


## Longest Common Subsequence

주어진 배열에서 작은값부터 시작해 큰 인덱스를 탐색하면서 진행할 때, 이어붙인 숫자들의 길이가 가장 길게 나타나게 하는 방법을 찾는다.

```cpp
int lis(vector<int> &arr) {
	int n = arr.size();
	vector<int> dp(n, 1);
	int len = 1;

	for (int i = 1; i < n; i++)
	{
		for (int j = 0; j < i; j++)
		{
			if (arr[i] > arr[j]) {
				//만약 arr의 값이 이전 값보다 크다면 해당하는 dp 인덱스 카운트 +1 or max
				dp[i] = max(dp[i], 1 + dp[j]);
				//arr와 dp 간의 상관관계를 유의해서 풀것.
				len = max(len, dp[i]);
			}
		}
	}
	for (int i = 0; i < dp.size(); i++)
	{
		cout << dp[i] << ", ";
	}
	cout << endl;
	return len;
}

```

## Knapsack
일정한 무게의 물품들이 있고, 이를 배낭에 담는 방법을 수한다.

재귀방법 문제 풀이

```cpp
int knapsack(const int N,const int W, vector<int>& weights, vector<int>& prices) {

	if (N == 0 || W == 0) {
		return 0;
	}

	int inc = 0, exc = 0;
	if (weights[N - 1] <= W) {
		inc = prices[N - 1] + knapsack(N-1,W-weights[N-1], weights, prices);
	}
	exc = knapsack(N - 1, W, weights, prices);
	return max(inc, exc);

}

```

<br>
dp를 활용한 풀이

```cpp
int knapsackDP(const int N, const int W, vector<int>& weights, vector<int>& prices) {

	vector<vector<int>> dp(N+1, vector<int>(W+1,0));

	// 2가지 케이스를 나누어 계산 (무게에 포함 안되는 요소 제외)
	for (int n = 1; n <= N; n++)
	{
		for (int w = 1; w <= W; w++)
		{
			int inc = 0, exc = 0;
			if (weights[n-1] <= w) {
				// 포함할경우 차감된 개수, 차감된 무게에 해당하는 인덱스에 inc 값 저장
				inc =  prices[n - 1] + dp[n-1][w-weights [n - 1]];
			}
			exc = dp[n - 1][w];

			dp[n][w] = max(inc, exc);
		}
	}
	return dp[N][W];

}
```

## wines
배열 {2,3,5,1,4 } 가 주어진다고 했을 때, 왼쪽 또는 오른쪽의 와인만 판매할 수 있으며 1번 판매할때마다 1년이 지나게 된다. 년수가 지나면 와인의 가치는 i * y가 될 때 최대 값을 구하기.


재귀를 활용한 풀이
```cpp
int SolutionRecursion(int* dp[10], vector<int>& wines, int L, int R, int y) {
	//base case
	if (L > R) {
		return 0;
	}

	if(dp[L][R] != 0) {
		return dp[L][R];
	}

	//recursion case
	int left = (wines[L] * y) + SolutionRecursion(dp, wines, L + 1, R, y + 1);
	int right = (wines[R] * y) + SolutionRecursion(dp, wines, L, R - 1, y + 1);
	return dp[L][R] = max(left, right);
}
```

<br>

dp를 활용한 풀이

와인의 선택 개수를 2가지의 경우로 나눠야 한다.
> F(l) = wines[l]*y + dp[l+1][r];
> F(r) = wines[r]*y + dp[l][r-1];

와인을 왼쪽, 오른쪽을 선택하는 경우의 값을 가져와 dp 2차원 배열에 높은 값 기준으로 저장한다.
dp의 인덱스 i , j는 각각 선택 가능한 남은 와인의 개수, j는 년수 이다.

```cpp
//R,L의 인덱스는 각각 남은 와인과 시작끝 지점 범위
int Solution( vector<int>& wines, int n) {
	vector<vector<int>> dp(n + 1, vector<int>(n + 1, 0));

	for (int i = n-1; i >= 0; i--)
	{
		for (int j = 0; j < n; j++)
		{
			//마지막으로 선택한 와인의 값 (i==j)
			if (i==j) {
				dp[i][j] = n * wines[i];
			}

			/* dp에 저장된 값은 아래를 따른다. 두 가지 경우를 고려해 max값 입력
			   F(l) = wines[l]*y + dp[l+1][r];
			   F(r) = wines[r]*y + dp[l][r-1];*/
			else if(i<j){
				int y = n - (j - i);
				int left = wines[i] * y + dp[i + 1][j];
				int right = wines[j] * y + dp[i][j -1];

				dp[i][j] = max(left, right);
			}
		}
	}

	return dp[0][n - 1];
}
```


앞서 DP를 공부하며 느낀것이 수학적인 지식이 부족하다고 느꼈다.

배낭 또는 와인 문제에서는 dp를 사용하기 위해 (bottom up , top down)  점화식을 세워 이에 맞는 값을 max 판단하는 경우를 사용하였는데, 점화식에 대한 이해가 부족해 dp를 이해하기 어렵지 않았나 생각한다.

결국 dp는 큰 문제를 어떻게 작은 문제로 나누어 푸느냐이며 그 과정에서 반복되는 연산을 기억한 후 연산 횟수를 줄여나가는것이 핵심이기에 어떤 상황에서 dp를 활용해 풀어야 하는가에 대해 더 공부해야겠다.


[DP 참고 블로그] (https://shoark7.github.io/programming/algorithm/3-ways-to-get-binomial-coefficients)
