---
layout:     post
title:      Leetcode股票问题总结
category: blog
description: 好久没更新博客了，忙完实习和文章再总结一些东西吧～
---

## Leetcode股票问题总结

股票买卖问题是Leetcode上一组经典的算法题，也是各大互联网公司笔试面试可能涉及到的问题，这一组问题涵盖了数组，动态规划，贪心等相关的算法内容，理解这几道题对提高我们的算法能力还是有很大帮助的，下面就开始总结：

#### 1. Best Time to Buy and Sell Stock I

首先，我们先从最简单的开始。

* **Q1: 一组股票的价格，最多只能交易一次，求最大收益。**

只能交易一次，就是得到两个价格之间的最大差值。最简单就是求任意两个价格之间的差值，但是这种算法是O(n^2)的，复杂度太高；

其实我们可以有一种O(n)的算法，维护两个变量表示最小值和最大收益值，从第一个元素开始遍历，如果当前元素比最小值小，则更新最小值；如果当前元素比最小值大，则尝试更新最大收益值，这其实就是简单的动态规划思想。

```java
class Solution {
    public int maxProfit(int[] prices) {
        int min = Integer.MAX_VALUE;
        int maxProfit = 0;
        for(int i = 0;i < prices.length;i++){
            if(prices[i] > min)
                maxProfit = Math.max(maxProfit, prices[i] - min);
            else
                min = prices[i];
        }
        return maxProfit;
    }
}
```

#### 2. Best Time to Buy and Sell Stock II

然后，进入第二个问题

* Q2: 一组股票价格，不限制交易次数，求最大收益

这个问题刚看到时感觉比较复杂，但是经过分析之后，不限制交易次数，那说明只要后数大于前数就把差值加到结果里，因为这样贪心的操作可以得到最大的收益

```java
class Solution {
    public int maxProfit(int[] prices) {
        int profit = 0;
        for(int i = 1;i < prices.length;i++)
            profit += Math.max(0, prices[i] - prices[i-1]);
        return profit;
    }
}
```

#### 3. Best Time to Buy and Sell Stock III

接着，我们看第三个问题，这题开始难度开始增大。

* Q3: 一组股票价格，最多交易两次，求最大收益

对于这个问题，我的最初想法是设计一个变量k作为价格数组的分割，计算0～k和k+1～n两个部分的最大收益和，每个部分里按第一题那样的算法计算，然后k从0遍历到n，取得最大的收益。这种算法的时间复杂度应该在O(n^2)，可以通过测试。

```java
class Solution {
    public int maxProfit(int[] prices) {
        if(prices.length <= 0)
            return 0;
        boolean buy = false;
        boolean sell = false;
        int len = prices.length;
        int trans = 2;
        int max = Integer.MIN_VALUE;
        for(int i = -1;i < len;i++){
            int r = maxStock(0,i+1,prices)+maxStock(i+1,len,prices);
            if(r > max)
                max = r;
        }
        if(max < 0)
            return 0;
        else
            return max;
    }

    public int maxStock(int start, int end, int[] prices){
        if(end - start <= 1)
            return 0;
        int min = prices[start];
        int result = Integer.MIN_VALUE;
        for(int i = 1;i < end-start;i++){
            if(prices[i+start]-min > result)
                result = prices[i+start]-min;
            if(prices[i+start] <= min)
                min = prices[i+start];
            
        }
        return result;
    }
}
```

但是发现时间很慢 ，只击败了大约5%的人，于是我看了大牛的一系列实现，发现有更简单的算法。这种算法基于状态机的思想，通过状态转换来表示两次交易的买入和卖出；

**第一次买入（fstBuy）** 、 **第一次卖出（fstSell）**、**第二次买入（secBuy）** 和 **第二次卖出（secSell）** 表示这四种状态。

* fstBuy = max(fstBuy ， -price[i])
* fstSell = max(fstSell，fstBuy + prices[i] )
* secBuy = max(secBuy ，fstSell -price[i]) (受第一次卖出状态的影响)
* secSell = max(secSell ，secBuy + prices[i] )

然后基于这种状态转移，实现代码

```java
class Solution {
    public int maxProfit(int[] prices) {
        int fstBuy = Integer.MIN_VALUE, fstSell = 0;
        int secBuy = Integer.MIN_VALUE, secSell = 0;
        for(int i = 0; i < prices.length; i++) {
            fstBuy = Math.max(fstBuy, -prices[i]);
            fstSell = Math.max(fstSell, fstBuy + prices[i]);
            secBuy = Math.max(secBuy, fstSell -  prices[i]);
            secSell = Math.max(secSell, secBuy +  prices[i]); 
        }
        return secSell;
        
    }
}
```



