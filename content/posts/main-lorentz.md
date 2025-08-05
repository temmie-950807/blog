+++
date = '2025-08-05T08:04:52+08:00'
draft = false
title = 'Main Lorentz 演算法'
tags = ["string"]
+++

{{< katex >}}

> [!INFO] 先備知識與說明
> - \(a+b\) 代表字串 \(a\) 和字串 \(b\) 的相接
> - 所有字串表示都是 0-based
> - \(s[l \ldots r]\) 代表 \(s\) 的第 \(l\) 個字元到第 \(r\) 個字元形成的子字串
> - \(\bar{s}\) 代表將 \(s\) 反轉

## 目標
我們定義 \(x\) 是某個任意長度的字串，則 \(x+x\) 就被稱作重串，而 \(x\) 被稱作原串。

透過一種演算法對於長度為 \(n\) 的字串 \(s\)，在 \(O(n \log n)\) 的時間複雜度內，找到最長的重串、重串的數量。

## 演算法

### 主要想法
演算法是透過「分治」完成的。

定義變數：
- \(s\): 原本的字串
- \(u\): \(s\) 的前一半
- \(v\): \(s\) 的後一半

可以假設已經計算完完全在 \(u\) 裡面跟完全在 \(v\) 裡面的重串，則剩下的步驟就是要找到跨越 \(u\) 跟 \(v\) 的重串。

### 定義：左、右中間字元
我們定義「左、右中間字元」分別是 \(u\) 的最後一個字元（\(s[\mid u\mid-1]\)），\(v\) 的第一個字元（\(s[\mid u\mid]\)），因為橫跨 \(u\) 跟 \(v\) 的重串必定會選到這兩個中間字元，因此在演算法中是必要的。

![](main-lorentz/main-lorentz1.png)

先看右中間字元，假設有某個橫跨 \(u\) 跟 \(v\) 的重串，則 \(u\) 裡面必定有個位置會對應到右中間字元（指兩個字元都在原串的同個位置），我們將它的位置設為 \(ptr\)。根據 \(ptr\) 的定義，可知道這種重串的長度 \(l\) 必定為 \(\mid u \mid - ptr\)。

在上圖中可以發現右中間字元是可以找到對應的重串的，但左中間字元不行，這是因為對應的字元可能出現在同一側或是不同側，因此我們才需要兩邊的中間字元去尋找重串。

### 定義：基準字元、左偏、右偏重串
我們定義「基準字元」為某重串 \(a+b, a=b\) 中，\(b\) 的第一個字元。

若基準字元在 \(u\) 裡面，則該重串被定義為「左偏」，否則為「右偏」。

以下的步驟會演示如何找到左偏重串，右偏重串則類似。

### 對於固定的 ptr，尋找所有重串
雖然我們已經固定了 \(ptr\)，也能找到對應的 \(l\)，但這不代表我們找到了重串，因為 \(ptr\) 跟右中間字元仍可能出現在不同的重串內。

![](main-lorentz/main-lorentz2.png)

透過上圖我們可以發現，固定 \(ptr\) 後，可以找到一個區間使得所有長度為 \(2l\) 的子區間都是重串，我們稱這個區間叫做「重串區間」，以 \([L, R]\) 表示。

> 如上圖，\([0, 10]\) 這個區間內選一個長度為 \(10\) 的子字串都是重串。
> 不包含 \(11\) 是因為這樣會包含右偏字串。

我們將重串區間由兩個變數 \(k_1, k_2\) 表示，\([ptr-k_1, \mid u\mid+k_2-1]\)。根據上圖的範例，比較有觀察力的讀者應該可以猜得出來 \(k_1, k_2\) 的條件：

- \(L\): \(s[ptr-k_1 \ldots ptr-1] = s[\mid u \mid-k_1 \ldots \mid u \mid - 1]\) 的最大 \(k_1\)。（下圖粉色箭頭）
- \(R\): \(s[ptr \ldots ptr+k_2-1] = s[\mid u \mid \ldots \mid u \mid + k_2-1]\) 的最大 \(k_2\)。（下圖綠色箭頭）

> [!WARNING] \(k_1, k_2\) 仍有上界
> \(k_1\) 不可大於等於 \(l\)，否則重串區間就不會包含中就會有某個長度為 \(2l\) 的區間不包含 \(\mid u \mid\) 的位置。
> 
> \(k_2\) 不可大於等於 \(l\)，否則 \(s[ptr \ldots \mid u \mid + k_2-1]\) 為重串（以下圖來說，就是指 \(s[2 \ldots 11]\)），但這不符合左偏重串的定義。

![](main-lorentz/main-lorentz3.png)

### 快速求得 k1, k2
透過 z-algorithm，我們可以快速的找到 \(k_1, k_2\)。

> [!INFO] 複習 z-algorithm
> 定義一個函式 \(Z(s)\)，回傳一個長度為 \(\mid s \mid\) 的陣列 \(z\)，其中 \(z[i]\) 為 \(s[0 \ldots |s|-1]\) 跟 \(s[i \ldots |s|-1]\) 的最長共同前綴。
> 
> 這個可以在 \(O(\mid s \mid)\) 的時間複雜度的時間內找到。

- \(k_1\): 對 \(\bar{u}\) 做 z-algorithm
- \(k_2\): 對 \(v + \# + u\)（\(\#\) 是個不在 \(s\) 的字元）做 z-algorithm

### 找到所有跨越 u, v 的重串

#### 枚舉所有字元
上述內容都是針對同一個 \(ptr\)，實際的演算法我們會枚舉 \(u\) 的每個字元，有找到和中間字元相同就設為 \(ptr\) 尋找重串區間 \([L, R]\)。

#### 以不同方向當作基準
別忘了我們剛剛僅有用到右中間字元找左偏重串，還要用左中間字元找到右偏重串才算找完所有跨越 \(u, v\) 的重串。
## 程式碼

```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;

int n, longest = 0;
string s;

vector<int> z_function(string s){
    vector<int> ret(s.size());
    int ll = 0, rr = 0;

    for (int i=1 ; i<s.size() ; i++){
        int j = 0;

        if (i<rr) j = min(ret[i-ll], rr-i);
        while (s[j]==s[i+j]) j++;
        ret[i] = j;

        if (i+j>rr){
            ll = i;
            rr = i+j;
        }
    }

    ret[0] = s.size();
    return ret;
}

int z_value(vector<int> &v, LL p){
    if (0<=p && p<v.size()) return v[p];
    return 0;
}

// 尋找字串 s 中的重串數量
int repetition(string s){

    // 終止條件
    if (s.size()==1){
        return 0;
    }

    // 找到左邊右邊的重串
    int mid = s.size()/2;
    string u = s.substr(0, mid);
    string v = s.substr(mid);
    string ru(u.rbegin(), u.rend());
    string rv(v.rbegin(), v.rend());
    int lc = repetition(u);
    int rc = repetition(v);

    // 尋找中間的重串
    int total = 0;
    vector<int> z1 = z_function(ru);
    vector<int> z2 = z_function(v+'#'+u);
    vector<int> z3 = z_function(ru+'#'+rv);
    vector<int> z4 = z_function(v);

    // 找到左偏重串
    for (int ptr=0 ; ptr<u.size() ; ptr++){
        if (u[ptr]==v[0]){
            int l = u.size()-ptr;
            int L = z_value(z1, u.size()-ptr);
            L = min(L, l-1); // 限制長度，不能和 ptr 重疊
            int R = z_value(z2, v.size()+1+ptr);
            R = min(R, l-1); // 限制長度，確保是左偏重串

            int add = max(0LL, R+L-l+1);
            total += add;
            if (add){
                longest = max(longest, l);
            }
        }
    }

    // 找到右偏重串
    for (int ptr=0 ; ptr<v.size() ; ptr++){
        if (u.back()==v[ptr]){
            int l = ptr+1;
            int L = z_value(z3, u.size()+v.size()-ptr);
            int R = z_value(z4, ptr+1);
            R = min(R, l-1); // 限制長度，不能和 ptr 重疊

            int add = max(0LL, R+L-l+1);
            total += add;
            if (add){
                longest = max(longest, l);
            }
        }
    }

    return lc+rc+total;
}

int main(){

    // input
    cin >> n;
    cin >> s;

    // process
    int res = repetition(s);
    cout << longest*2 << " " << res << "\n";

    return 0;
}
```

## 習題

### 一、尋找重複字串

[題目網址](https://codeforces.com/contestInvitation/4d09eace2246e7893a7ac48d8238483daebc4daa)

自己生的題目，有問題再告訴我 :>

## 參考
- https://oi-wiki.org/string/main-lorentz/
- https://cp-algorithms.com/string/main_lorentz.html
- 跟 dreamoon 老師討論