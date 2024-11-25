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
            for (int i = 0; i < n; i++) { // 遍历dis 寻找距离最短的点
                if (!done[i] && (x < 0 || dis[i] < dis[x])) {
                    x = i; 
                }
            }
            // 如果 没有找到未处理的点，直接返回
            if (x < 0)
                return ranges::max(dis);
            // 如果找到的距离最小的点的距离还是inf 返回-1，表示不可达
            if (dis[x] == INT_MAX / 2)
                return -1;
            done[x] = true; // 更新该节点为访问过
            for (int y = 0; y < n; y++) {
                dis[y] = min(dis[y], dis[x] + g[x][y]); // 以当前节点为起始节点，更新最小距离
            }
        }
    }
};
```
# 时间复杂度
1. 初始化邻接矩阵为$O(n^2)$
# 空间复杂度
1. 使用了邻接矩阵存储图为$O(n^2)$