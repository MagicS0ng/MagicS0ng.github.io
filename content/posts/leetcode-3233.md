+++
date = '2024-11-22T22:13:36+08:00'
draft = false
title = 'Leetcode 3233'
ShowCodeCopyButtons=true
ShowReadingTime=true
ShowBreadCrumbs=true
showToc = true
description="leetcode 3233的题解" 
tags = ['leetcode']
math = true
summary = "leetcode 中等题，需要一些分析"
+++
[原题](https://leetcode.cn/problems/find-the-count-of-numbers-which-are-not-special/description/?envType=daily-question&envId=2024-11-22)  
# leetcode 3233
真因数：对于一个数字x x除了x本身以外的所有正因数为其真因数
特殊数字的定义为：仅有两个真音数的数字为特殊数字。
1. 从定义上看，特殊数字$z$一定有一个因数为1，那么其另一个真因数一定是$\sqrt{z}$，
2. 否则就会有加上1的三个真因数，就会导致$z$不是特殊数字，因为$z$有3个真因数
3. $\sqrt{z}$一定为质数，否则$\sqrt{z}$也会有真因数，就会影响$z$本身
区间[l,r]内的特殊数字个数，就是[0,r]特殊数字个数减去[0,l-1]特殊数字个数
```C++
const int MX = 31622;
int pi[MX + 1];
auto init = []() {
    for (int i = 2; i <= MX; i++) {
        if (pi[i] == 0) { // i从2开始  i的倍数已经被标记为-1了
            pi[i] = pi[i - 1] + 1;
            for (int j = i * i; j <= MX; j += i) {
                pi[j] = -1;
            }
        } else {
            pi[i] = pi[i - 1];
        }
    }
    return 0;
}();
class Solution {
public:
    int nonSpecialCount(int l, int r) {
        return r - l + 1 - (pi[int(sqrt(r))] - pi[int(sqrt(l - 1))]);
    }
};
```

# 时间复杂度
1. $O(1)$
# 空间复杂度
1. $O(1)$