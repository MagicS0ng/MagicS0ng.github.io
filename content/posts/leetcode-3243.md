+++
date = '2024-11-20T21:53:37+08:00'
draft = false
title = 'Leetcode 3243'
+++

# leetcode 3243

[原题链接](https://leetcode.cn/problems/shortest-distance-after-road-addition-queries-i/description/)

```C++
class Solution{
private:
    int bfs(int n, vector<vector<int>>&neighbor)
    {
        vector<int> dist(n,-1); // 定义每个点到起点的距离
        queue<int> q;           // 用于bfs遍历的队列
        q.push(0);              // 将起点加入队列
        dist[0]=0;              // 0~0 距离为0
        while(!q.empty())
        {
            int x = q.front(); // 取出当前起点
            q.pop();
            for(auto &y: neighbor[x])  // 遍历下一个节点
            {
                if(dist[y]!=-1)
                    continue;       // 如果已经访问过了，直接跳过
                q.push(y);
                dist[y] = dist[x]+1; // 计算0到x的距离，+1
            }
        }
        return dist[n-1];
    }
public:
    vector<int> shortestDistanceAfterQueries(int n,
                                             vector<vector<int>>& queries) {
        vector<vector<int>> graph(n);   // 定义图
        vector<int> result;             // 定义结果集
        for (int i = 0; i < n - 1; i++) {
            graph[i].emplace_back(i+1); // 建图
        }
        for (auto& query : queries) {
            graph[query[0]].emplace_back(query[1]); // 插入边
            result.emplace_back(bfs(n, graph));         // 计算图更新后的结果
        }
        return result;
    }
};
```
# 时间复杂度分析
1. 每次插入一条边，都要遍历一次
2. 每次bfs，都需要遍历n+q次
时间复杂度为**O(q*(n+q))**
# 空间复杂度分析
1. 存储图和结果集，需要O(n+q)的空间