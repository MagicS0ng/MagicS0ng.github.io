+++
date = '2024-11-23T12:11:04+08:00'
draft = false
title = 'Leetcode 3238'
ShowCodeCopyButtons=true
ShowReadingTime=true
ShowBreadCrumbs=true
showToc = true
description="leetcode 3238的题解" 
tags = ['leetcode']
math = true
summary = 'Leetcode 3238 每日一题 简单题'
+++
# leetcode 3238
[原题](https://leetcode.cn/problems/find-the-number-of-winning-players/?envType=daily-question&envId=2024-11-23)  

## 分析
题目定义玩家的某个颜色的球的个数大于玩家id为胜利玩家，
1. 因此需要统计每个玩家的对应颜色的球的数量，也就是要定义一个二维数组 $arr[playerId][ballColor]$ 表示玩家id为playerId，球颜色为ballColor的球数量。
2. 一个玩家只能被记录一次胜利玩家，因此需要一个标记数组$tags[playerCount]$标记玩家是否已经被记录过
3. 一旦某个玩家的$arr[playerId][ballColor]>playerId$，胜利玩家结果数量+1，且要将$tags[playerId]$标记为true，表示该玩家已经被记录为胜利玩家

```C++
// 1.
// const int MXN =20;
// int arr[MXN][11]{0};
// bool tags[MXN]{false};
class Solution {
public:
    int winningPlayerCount(int n, vector<vector<int>>& pick) {
        // 2.
        vector<vector<int>> arr(n,vector<int>(11,0));
        vector<bool> tags(n,false);
        int result = 0;
        for (auto pic : pick) {
            // playerId = pic[0];
            // ballColor = pic[1];
            // tags[playerId]未被记入结果时，且arr[playerId][ballColor]>playerId
            // 结果+1 将tags[playerId] 标记为true
            int playerId = pic[0];
            int ballColor = pic[1];
            ++arr[playerId][ballColor];
            if (tags[playerId] == false &&
                arr[playerId][ballColor] > playerId) {
                result++;
                tags[playerId] = true;
            }
        }
        return result;
    }
};
```
# 时间复杂度
1. 对于每个pic 都需要遍历一次 
2. 对于两个数组，需要对$n*max(y)$的数组初始化
3. 因此时间复杂度为$O(n*max(y)+m)$ $m$为pick的长度，$n$为playerId的数量，$y$为球颜色数量
# 空间复杂度
需要两个数组，因此空间复杂度为$O(n*y)$
# 一些问题
尚未搞明白，为什么注释1的定义方式本地可以通过leetcode无法通过，注释2的可以