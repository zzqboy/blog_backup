---
title: ksum问题
date: 2018-06-09 23:28:00
tags: leetcode
categories: 技术
---
# 问题
```
Given array nums = [-1, 0, 1, 2, -1, -4],  
  
A solution set is:  
[  
[-1, 0, 1],  
[-1, -1, 2]  
]
```
# 解决方案
此前还有2sum问题，那么可以把这个问题扩展为k-sum
对于找出3个数，我们可以先固定一个数，在剩下的数里面去找到两个，那么复杂度就是n^2，这太高了，有没有可能不用暴力解决。答案是肯定的，因为最终的和是知道的，那么我们固定了一个数，就可以根据其他两个数来判断是如何移动，达到线性搜索。前提是排好序
![k_sum.jpg](/img/k_sum.jpg)

同理 4sum可以固定两个数，再去找剩余的两个数…

[代码](https://github.com/zzqboy/leetcode/blob/master/LeetCode/015_3Sum.h)