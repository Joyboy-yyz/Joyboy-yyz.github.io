---
title: "LeetCode 18 四数之和"
date: 2026-11-04T14:50:00+08:00
draft: false
tags: ["LeetCode", "双指针", "算法"]
categories: ["刷题笔记"]
---

## 思路

排序 + 双层固定 + 双指针，是三数之和的扩展。

下标约定：a < b < c < d

- a：第一层循环固定
- b：第二层循环固定，从 a+1 开始
- c：左指针，初始 b+1
- d：右指针，初始 n-1

天然保证四个下标互不相同，并且不会重复枚举同一组组合。

## 去重

- a 去重：`if (a > 0 && nums[a] == nums[a - 1]) continue;`
- b 去重：`if (b > a + 1 && nums[b] == nums[b - 1]) continue;`
- 找到答案后 c、d 也要跳过相同值

## 代码

```cpp
class Solution {
public:
    vector<vector<int>> fourSum(vector<int>& nums, int target) {
        vector<vector<int>> res;
        sort(nums.begin(), nums.end());
        int n = nums.size();
        for (int a = 0; a < n; a++) {
            if (a > 0 && nums[a] == nums[a - 1]) continue;
            for (int b = a + 1; b < n; b++) {
                if (b > a + 1 && nums[b] == nums[b - 1]) continue;
                int c = b + 1;
                int d = n - 1;
                while (c < d) {
                    long long sum = (long long)nums[a] + nums[b] + nums[c] + nums[d];
                    if (sum < target) {
                        c++;
                    } else if (sum > target) {
                        d--;
                    } else {
                        res.push_back({nums[a], nums[b], nums[c], nums[d]});
                        while (c < d && nums[c] == nums[c + 1]) c++;
                        while (c < d && nums[d] == nums[d - 1]) d--;
                        c++;
                        d--;
                    }
                }
            }
        }
        return res;
    }
};
```

## 复杂度

- 时间：O(n^3)
- 空间：O(log n)（排序开销）
