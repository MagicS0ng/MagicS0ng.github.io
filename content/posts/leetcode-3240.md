+++
date = '2024-11-22T21:43:28+08:00'
draft = false
title = 'Leetcode 3240'
ShowCodeCopyButtons=true
ShowReadingTime=true
ShowBreadCrumbs=true
showToc = true
description="leetcode 3240的题解" 
tags = ['leetcode']
math = true
summary = "leetcode 中等题，需要一些分析"
+++
# Leetcode 3240
[原题链接](https://leetcode.cn/problems/minimum-number-of-flips-to-make-binary-grid-palindromic-ii/description/)  

题目要求所有的行和列都是回文，又限制1的个数需要被4整除。
1. 行列均回文即要求 
$$
arr[i][j] = arr[m-1-i][j] = arr[i][n-1-j] = arr[m-1-i][n-1-j]
$$
遍历数组，计算$cnt1=arr[i][j] + arr[n-1-i][j] + arr[i][n-1-j] + arr[n-1-i][n-1-j]$的和，和即为1的个数，将$min(result, cnt1)$计入结果。  
2. 如果为偶数行偶数列，此时已经计算完毕，反之
3. 中心元素必须为0，$result+=arr[m/2][n/2]$, $arr[m/2][n/2]=0$时不会计入修改次数所以直接相加即可
4. 计算中心行或中心列，先计算匹配的1个数，记为$cnt$，计算需要修改的对数$diff$
5. $cnt%4==0$ 1的个数能被4整除，$diff$对全改为0，计入结果
6. $cnt%4==2$ 1的个数不能4整除， $diff==0$，将$cnt%4$余下的两个改为0即可，$diff!=0$ 选择diff中的一对改为1，其余的改为0即可
```C++
class Solution {
public:
    int minFlips(vector<vector<int>>& grid) {
        /*
        grid[i][j]=grid[m-i-1][j] 列相等
        grid[i][j]=grid[i][n-j-1] 行相等
        grid[i][n-j-1] = grid[m-i-1][n-j-1]
        */
        int m = grid.size(), n = grid[0].size();
        int result = 0;
        for (int i = 0; i < m / 2; i++) {
            for (int j = 0; j < n / 2; j++) {
                int cnt1 = grid[i][j] + grid[i][n - 1 - j] +
                           grid[m - 1 - i][j] + grid[m - 1 - i][n - 1 - j]; // 计算的结果就是1的个数，将其变为0即可
                result += min(cnt1, 4 - cnt1); // 全变为1或全变为0
            }
        }
        if (m % 2 && n % 2) {
            result += grid[m / 2][n / 2]; // 确保中间为0，如果为0，+0不会变化，如果为1，+1变换次数+1
        }
        /*  单独讨论奇数行和奇数列
            镜像位置如果都一样，能不改就不改，先统计不改的位置有多少个1，
           记作cnt1, 需要修改的为diff 分类讨论
                cnt1%4 = 0  diff对全部改为0 or
                cnt1%4 = 2  diff>0 选择一对，改为1
                            diff==0  选择原来不改的一对，改为0即可
        */
        // 有奇数行
        int diff = 0, cnt1 = 0;
        if (m % 2) {
            for (int i = 0; i < n / 2; i++) {
                if (grid[m / 2][i] != grid[m / 2][n - 1 - i])
                    diff++;
                else
                    cnt1 += grid[m / 2][i] * 2;
            }
        }
        // 有奇数列
        if (n % 2) {
            for (int i = 0; i < m / 2; i++) {
                if (grid[i][n / 2] != grid[m - 1 - i][n / 2])
                    diff++;
                else
                    cnt1 += grid[i][n / 2] * 2;
            }
        }
        // 如果一致的1的个数能够整除4  返回 result+diff 即将diff中不为0的改为0
        if (cnt1 % 4 == 0)
            return result + diff;
        // 如果一致的1的个数无法整除4  返回 result+(cnt1%4)+diff
        // 将整除4余下的个数全改为1即可
        return result + (diff ? diff : cnt1 % 4);
        // return result + (diff ? diff : cnt1 % 4);
    }
};
```
# 时间复杂度
1. 遍历数组，复杂度为**O(m*n)**
# 空间复杂度
1. 只需要常数量的变量存储结果，所以未**O(1)**