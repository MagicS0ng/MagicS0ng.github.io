+++
date = '2024-11-25T23:30:23+08:00'
draft = false
title = 'Leetcode 743'
ShowCodeCopyButtons=true
ShowReadingTime=true
ShowBreadCrumbs=true
showToc = true
description="leetcode 743的题解" 
tags = ['leetcode', 'graph']
math = true
summary = "leetcode 中等题，需要一些分析"
+++

# leetcode 743
[原题](https://leetcode.cn/problems/network-delay-time/description/?envType=daily-question&envId=2024-11-25)  
## 分析
题目要求计算从起始节点到所有节点的最短时间，也就是到最远距离的最小时间。
使用dijkstra算法即可。
```C++
class Solution {
public:
    int networkDelayTime(vector<vector<int>>& times, int n, int k) {
        vector<vector<int>> g(n, vector<int>(n, INT_MAX / 2));// 定义图， g[i][j] 
        for (auto& t : times) {
            g[t[0] - 1][t[1] - 1] = t[2];
        }
        vector<int> dis(n, INT_MAX / 2), done(n); // dis[i] 表示k-> i的最短路
        dis[k - 1] = 0; // 起始节点为k 起始距离为0
        while (true) {
            int x = -1;
            for (int i = 0; i < n; i++) {
                if (!done[i] && (x < 0 || dis[i] < dis[x])) {
                    x = i; // 寻找一个未访问的点，且到它的距离最小
                }
            }
            if (x < 0) 
                return ranges::max(dis);
            if (dis[x] == INT_MAX / 2) // 如果一个点不可达 直接返回
                return -1;
            done[x] = true; // 更新该点访问过
            for (int y = 0; y < n; y++) {
                dis[y] = min(dis[y], dis[x] + g[x][y]); //更新最短距离
            }
        }
    }
};
```
# 时间复杂度
1. 初始化邻接矩阵为$O(n^2)$
# 空间复杂度
1. 使用了邻接矩阵存储图为$O(n^2)$