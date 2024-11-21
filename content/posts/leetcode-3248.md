+++
date = '2024-11-21T22:19:17+08:00'
draft = false
title = 'Leetcode 3248'
ShowCodeCopyButtons=true
ShowReadingTime=true
ShowBreadCrumbs=true
showToc = true
description="leetcode 3248的bfs题解" 
tags = ['leetcode', '模拟']
summary = "leetcode 每日一题，简单题"
+++
# leetcode 3243

[原题链接](https://leetcode.cn/problems/snake-in-matrix/description/)
本题的求解思路为：模拟即可
```C++
class Solution {
public:
    int finalPositionOfSnake(int n, vector<string>& commands) {
        int startX = 0, startY = 0;
        for (auto& command : commands) {
            if (command == "RIGHT" || command == "LEFT")
                startY = startY + (command == "LEFT" ? -1 : 1);
            else if (command == "UP" || command == "DOWN")
                startX = startX + (command == "UP" ? -1 : 1);
        }
        return ((startX)*n - 1) + startY + 1;
    }
};
```
# 时间复杂度分析
1. 遍历commands数字时间复杂度为**O(N)**
# 空间复杂度分析
1. 常量的存储空间 **O(1)**