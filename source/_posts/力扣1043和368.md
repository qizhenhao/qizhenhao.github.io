---
title: 力扣1043和368
date: 2023-4-19 10:23:00
tags: [算法,动态规划]
---

#动态规划两道题

<!-- more -->

# [1043. 分隔数组以得到最大和 - 力扣（LeetCode）](https://leetcode.cn/problems/partition-array-for-maximum-sum/)

### 方法一（过不了）：树状数组＋dfs

用dfs暴力搜索时间复杂度为指数型，用树状数组优化，但还是指数型

介绍一下树状数组：

【五分钟丝滑动画讲解 | 树状数组】 https://www.bilibili.com/video/BV1ce411u7qP/?share_source=copy_web&vd_source=c9899d0504fa271ca6db5ef82d1a6bbb

树状数组的作用，

1.以O(logN)找到一个下表的 i 到 j 的和

2.以O(logN)更新一个下表的数字

3.空间复杂度为O(N)

```c++
class Solution {
public:
    vector<int> tr;
    int lowerBit(int i)
    {
        return (i&-i);
    }
    void add(int i,int z)
    {
        for(;i<tr.size();i+=lowerBit(i))
        {
            tr[i]+=z;
        }
    }
    int query(int i)
    {
        int res=0;
        for(;i>0;i-=lowerBit(i))
        {
            res+=tr[i];
        }
        return res;
    }//树状数组的定义
    int maxSumAfterPartitioning(vector<int>& arr, int k) {
        tr = vector<int>(arr.size()+1,0);
        int ma=0;
        function<void(int)> dfs = [&](int dex){
            if(dex==arr.size())//下表超出，找到总和
            {

                ma = max(ma,query(tr.size()-1));
                return;
            }
            int res=0;
            for(int i=dex;i<arr.size()&&i-dex<k;i++)
            {
                
                res = max(res,arr[i]);
                
                for(int j=dex+1;j<=i+1;j++)
                {
                    add(j,res);
                }
                dfs(i+1);//更新tr，递归后删除更新的数值
                for(int j=dex+1;j<=i+1;j++)
                {
                    add(j,-res);
                }
            }
        };
        dfs(0);
        return ma;
    }   
};
```



### 方法二：dp

简单分析一下找到数组的最大值，就是找到n的位置于n的前k项进行组合找到最大值

dp中储存的就是当前下表大小的数组的和最大值

推出动态方程 	dp[i] = max(dp[i],dp[j]+valmax*(i-j));

```c++
class Solution {
public:
   
    int maxSumAfterPartitioning(vector<int>& arr, int k) {
        vector<int> pd(arr.size()+1,0);
        //多一个是为了让以dp[0]=0开头，让下表i存的是前i项之和包括i
        for(int i=1;i<arr.size()+1;i++)
        {
            int maxv=arr[i-1];
            for(int j=i-1;j>=0&&i-j<=k;j--)
            {
                pd[i] = max(pd[i],pd[j]+(i-j)*maxv);
                if(j>0)
                    maxv = max(maxv,arr[j-1]);
            }
        }
        return pd[arr.size()];
    }   
};
```

时间复杂度O(NK) 空间是O(N)

# [368. 最大整除子集 - 力扣（LeetCode）](https://leetcode.cn/problems/largest-divisible-subset/)



## 	dp+数学（简单的数学）：

数学方面就是：在一个集合中使各个元素的最大公因数，最小公倍数是两个数本身

也就是说一个数能被另一个数整除

**一个大于集合的所有数判断是否能被集合中所有元素整除，只需要判断能被集合中最大的数整除**

所以先对nums排序

dp方程为dp[i] = max(dp[j]+1);

保存最大的值，从最大的值开始往前找，找到连续可行的子序列

```c++
class Solution {
public:
    vector<int> largestDivisibleSubset(vector<int>& nums) {
        sort(nums.begin(),nums.end());
        vector<int>ans;
        vector<pair<int,int>> mp(nums.size(),{0,-1});
        int ma=-1,biao=0;
        for(int i=0;i<nums.size();i++)
        {
            for(int j=i-1;j>=0;j--)
            {
                if(nums[i]%nums[j]==0&&mp[i].first<mp[j].first)
                {
                    mp[i].first = mp[j].first;
                    mp[i].second = j;
                }
            }
            mp[i].first++;
            if(ma<mp[i].first)
            {
                ma = mp[i].first;
                biao = i;
            }
        }
        
        while(biao>=0)
        {
            ans.push_back(nums[biao]);
            biao = mp[biao].second;
        }
        return ans;
    }
};
```

时间复杂度O(N^2)

空间复杂度O(2N)

