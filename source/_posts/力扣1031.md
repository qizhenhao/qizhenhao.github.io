---
title: 力扣1031
date: 2023-4-26 14:15:00
tags: [算法,动态规划,前缀和]
---

# #两个非重叠子数组的最大和

<br>

<!--more-->

# [1031. 两个非重叠子数组的最大和 - 力扣（LeetCode）](https://leetcode.cn/problems/maximum-sum-of-two-non-overlapping-subarrays/)

## 前缀和+动态规划：

首先用前缀数组可以快速求出数组中两点中的元素的和，首先应该想象到找到first的元素的和，去遍历second的和找到最大和时间复杂度O(n^2),应该不难想到。

用动态规划去解决问题时，遍历一遍找到最大一个first的值，很简单就想到用一个

sum=max(sum,s[i]-s[i-first]);

解决问题，

但是问题是求两端的和

有个方法很难想到，我反正是想不到，首先first与second的大小不同导致不清楚答案是谁在前谁在后

所以分开考虑假如second在前，**则有在每次找first的最大时刻后面更新答案**，

sum = max(sum,s[i-first]-s[i-first-second]);

ans = max(ans,sum+s[i]-s[i-first]);

同理另一种情况也是一样最终得出答案.

```c++
class Solution {
public:
    int maxSumTwoNoOverlap(vector<int>& nums, int firstLen, int secondLen) {
        int s[1001],n = nums.size();
        s[0]=0;

        for(int i=1,sum=0;i<=n;i++)
        {
            sum+=nums[i-1];
            s[i] = sum;
        }   
        int sum1=0,sum2=0,ans1=0,ans2=0;
        for(int i=firstLen+secondLen;i<=n;i++)
        {
            sum1 = max(sum1,s[i-secondLen]-s[i-secondLen-firstLen]);
            ans1 = max(ans1,sum1+s[i]-s[i-secondLen]);
            
            sum2 = max(sum2,s[i-firstLen]-s[i-firstLen-secondLen]);
            ans2 = max(ans2,sum2+s[i]-s[i-firstLen]);
        }
        return max(ans1,ans2);
    }
};
```

