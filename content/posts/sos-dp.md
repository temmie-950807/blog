+++
date = '2025-08-04T02:51:54+08:00'
draft = false
title = 'SOS DP'
+++

<!-- {{< katex >}} -->

這篇文章都是在解決一道問題：給定 $arr$，求 $dp[mask] = \sum\limits_{i \subseteq mask} arr[i]$，其中 $i$ 跟 $mask$ 都是一個集合，其中宇集為 $N$ 個元素的集合。之後的內容我們都用一個 $N$-bit 的二進位表達集合的概念。

這種問題被稱做「Sum over Subsets」，縮寫成 SOS。
## 一、暴力
透過位元枚舉，我們可以依序枚舉 $mask$ 跟 $i$，並找到所有 $i \subseteq mask$ 的情況做轉移，得出以下 $O(n^{4})$ 的程式碼：
```cpp
for (int i=0 ; i<arr.size() ; i++){
	dp[i] += arr[i]
}
for (int mask=0 ; mask<(1<<N) ; mask++){
	for (int i=0 ; i<(1<<N) ; i++){
		if ((mask&i)==i){
			dp[mask] += dp[i];
		}
	}
}
```

## 二、優化暴力
### 枚舉子集
以上的演算法，對於同一個 $mask$，我們會枚舉到很多不必要的 $i$，我們可以改寫成這樣的演算法。
```cpp
for (int i=0 ; i<arr.size() ; i++){
	dp[i] += arr[i]
}
for (int mask=0 ; mask<(1<<N) ; mask++){
	for (int i=mask ; ; i = ((i-1)&mask)){
		dp[mask] += dp[i];
		if (i==0) break;
	}
}
```

當 $i = 11_{(10)} = 01011_{(2)}$ 的時候，它的變化如下圖：
![[Pasted image 20250627010128.png|400]]

簡單來說，就是每一次進入 `i = ((i-1)&mask)`，都會使目前最小編號為 1 的元素消失（也就是上圖紅色箭頭的起點），並使得比他編號更小且在 mask 裡的元素出現（也就是上圖紅色的正方形）。

以上的方法要枚舉 $2^{n}$ 個 $mask$，每個 $mask$ 會再枚舉到 $2^{p}$（其中 $p$ 代表 $mask$ 中 1 的個數） 個 $i$。可以看出，我們總共會枚舉以下的次數：
$$
\sum_{p=0}^{N} \binom{N}{p} 2^{p} \tag{1}
$$
根據二項式定理，我們有：
$$
(x+y)^{N} = \binom{N}{0} x^{N}y^{0} + \binom{N}{1}x^{N-1}y^{1} + \ldots + \binom{N}{N} x^{0}y^{N} \tag{2}
$$
將式 $(2)$ 代入 $x = 1, y = 2$ 後，我們有：
$$
\begin{aligned}
(1+2)^{N}
&= \binom{N}{0} 2^{0} + \binom{N}{1} 2^{1} + \ldots \binom{N}{N}2^{N}\\
&= \sum_{p=0}^{N} \binom{N}{p} 2^{p}\\
\end{aligned}
\tag{3}
$$
式 $(1)$ 與式 $(3)$ 相同，因此時間複雜度為 $O(3^{N})$。

## 三、SOS [[DP]]

### A、DP 的觀點

#### 狀態

考慮以下的 DP 狀態（以下都是 0-based）：
$$
dp[mask][pos] = \text{$mask$ 中 index $< pos$ 的 bit 都已經固定，考慮 index 是 $mask$ 子集的元素總和}
$$
例如：
![[Pasted image 20250627094722.png|300]]

可以看到我用黃底代表被 $pos$ 固定住的 bit，其餘的 bit 則是 $mask$ 的子集。
#### 基底
根據狀態，我們有 $dp[mask][0] = arr[mask]$。

#### 轉移
讓我們用 $pos$ 拉所有 $pos-1$ 的資訊，所以要考慮第 $pos$ 個 bit 要是 1 還是 0，我們可以有以下的遞迴關係式：
$$
dp[mask][pos] = 
\begin{cases}
dp[mask][pos-1]+dp[mask \oplus 2^{pos}][pos-1] & \text{if $pos$-th bit of $mask$ is $1$}\\
dp[mask][pos-1] & \text{otherwise}
\end{cases}
$$
> $mask \oplus 2^{pos}$ 的「意義」是將 $mask$ 的 $pos$-th bit 改變，在這裡可以想成直接將 $pos$-th bit 變成 $0$。
#### 程式碼 1
```cpp
int dp[1<<N][N];
for (int mask=0 ; mask<(1<<N) ; mask++){
	for (int pos=0 ; pos<N ; pos++){
		if ((mask>>pos)&1){
			// pos-th bit of mask is 1
			dp[mask][pos] = dp[mask][pos-1]+dp[mask^(1<<pos)][pos]
		}else{
			dp[mask][pos] = dp[mask][pos-1]
		}
	}
}
```

#### 程式碼 2
我們可以用滾動優化，優化*程式碼 1*：
```cpp
int dp[1<<N];
for (int pos=0 ; pos<N ; pos++){
	for (int mask=0 ; mask<(1<<N) ; mask++){
		if ((mask>>pos)&1){
			// pos-th bit of mask is 1
			dp[mask] += dp[mask^(1<<pos)]
		}else{
			// ignore
		}
	}
}
```

由於上方的轉移方式都有 $mask \oplus 2^{pos} < mask$，因此這樣的轉移順序會是好的。

### B、從高維前綴和的觀點
#### 普通的前綴和
讓我們先從二維前綴和引入，以往我們在做二維前綴和時，可以會以以下的方式預處理：
![[Pasted image 20250705094640.png|400]]
也就是形如 $pre[i][j] = pre[i-1][j]+pre[i][j-1]-pre[i-1][j-1]$ 的方式。

這樣的預處理時間複雜度是多少呢？實際上是 $O(NM 2^{k})$，其中 $2^{k}$ 時排容所貢獻，其中 $k$ 代表是多少維度的前綴和（在這裡，$k=2$）。

對於高維度的前綴和，就沒有更快的（意指，將 $2^{k}$ 降低的方法）演算法了嗎？。
#### 前綴和加速
![[Pasted image 20250705094612.png|600]]
有一種特殊的前綴和方法，同樣以二維前綴和舉例。

第一步會先對所有元素做 $pre[i][j] += pre[i-1][j]$，接著再對所有元素做 $pre[i][j] += pre[i][j-1]$。

可以看出這樣的時間複雜度僅有 $O(NMk)$，我們把指數級別的時間複雜度壓下來了。

#### 與 SOS 問題的關連
$dp[mask] = \sum\limits_{i \subseteq mask} arr[i]$

以 $N=3$ 作為例子，假設 $arr = [0, 1, 2, 3, 4, 5, 6, 7]$，將 $arr$ 放在 $2 \times 2 \times 2$ 的陣列 $mat$，其中 $mat[i][j][k] = arr[i*4 + j*2 + k]$。

下方的圖片中，紅色數字為 $i$，橘色數字為 $j$，綠色數字為 $k$。
![[Pasted image 20250705100516.png|400]]

動手計算 $dp$，不難發現 $dp[i][j][k]$ 的值，正是 $mat[i][j][k]$ 到 $mat[0][0][0]$ 的前綴和。
![[Pasted image 20250705100505.png|400]]
因此，我們可以套用先前處理高維前綴和的方法，在 $O(2^{N} N)$ 的時間內完成。
#### 程式碼
實作上，我們不會真的開一個 $2 \times 2 \ldots \times 2$ 的陣列，而是用 $2^{N}$ 大小的一維陣列模擬
```cpp
int dp[1<<N];
for (int pos=0 ; pos<N ; pos++){
	for (int mask=0 ; mask<(1<<N) ; mask++){
		if ((mask>>pos)&1){
			dp[mask] += dp[mask^(1<<pos)]
		}
	}
}
```

實際上就和上面的程式碼一樣！
## 四、習題
### A、https://oj.ntucpc.org/problems/430

首先因為 $1 \le n \le 17$，所以是可以使用 $O(3^{n})$ 的演算法的。

有個重要的觀察是：我們只要計算「**只能用 2 個任務條完成**的情況下所有任務子集最快可以多快完成」。

令 $\text{twoTask}[mask]$ 為「**只能用 2 個任務條完成** 所有 $mask$ 中的任務的最快速度」。

那麼我們就可以計算 
$$
\text{fourTask} = \min\limits_{mask=0}^{2^{N}-1} (\quad max(\text{twoTask}[mask], \text{twoTask}[\sim mask ])\quad) 
$$
以上為 $O(2^{N})$

對於 $\text{twoTask}$ 的計算，我們可以再套一層 SOS 的方法：
$$
\text{twoTask[i]} = \min_{mask \subseteq i} \text{oneTask}[mask]+\text{oneTask}[mask]
$$
以上為 $O(N \times 3^{N})$

最後，$\text{oneTask[mask]}$ 的預處理是顯然很容易的，為 $O(N \times 2^{N})$。

所以這道題目可以透過 SOS 的技巧，在 $O(N3^{N})$ 內處理掉。
### B、https://cses.fi/problemset/task/1654

題目所求的三個問題，本質上在求：
1. $x$ 的子集數量
2. $x$ 的父集數量
3. $n - (\text{flip}(x) \text{ 的子集數量 } )$

以上三個都在本篇介紹過。

## 五、參考
1. https://littlecube8152.github.io/posts/dp-sum-over-subsets/
2. https://www.cnblogs.com/cyl06/p/SOSDP.html
3. https://codeforces.com/blog/entry/45223
4. https://codeforces.com/blog/entry/105247