---
layout:     post
title:      刷leetcode算法总结（一）
category: blog
description: 将最近在leetcode上刷的题归纳一下
---

## 384.Shuffle an Array

这道题的主要要求就是设计一个方法将一个给定数组的顺序随机打乱，然后返回这一打乱之后的数组。

为了保证随机性，可以我参考了洗牌算法，即首先从所有元素中随机选取一个与第一个元素进行交换，然后在第二个之后选择一个元素与第二个交换，直到最后一个元素。这样能确保每个元素在每个位置的概率都是1/n。

按照这样的思路，我设计出了如下代码进行实现：

``` java
public int[] shuffle() {
    int length = numbers.length;
    int[] nums1 = new int[100];
    nums1 = numbers.clone();         				//得到原数组的一个拷贝
    for(int i = 0;i < length;i++){
        int random = (int)(Math.random()*length);   //使用java内置的随机数
        int temp = nums1[random];    				//exchange
        nums1[random] = nums1[i];
        nums1[i] = temp;
    }
    return nums1;
}
```

本题的其他两个算法都是简单的赋值，不在这里介绍了，这种洗牌算法的实现还是可以给之后的设计提供一些思路的。

## 383.Ransom Note

刚看这道题时，注意到它的难度标注的是easy，所以最开始把它想简单了，以为只是一道长字符串包含短字符串的问题，结果当然果断是wrong answer了

之后才仔细读了遍题，发现是一个字符组成问题，因为字符串只涉及26个小写字母，于是我就用两个26位的int数组对两个字符串的字符出现次数进行统计，统计之后再进行比较，如果出现某一位上ransom的出现次数多于magazine的话，那么则不成立，返回false；反之，返回true。

上面的算法实现如下：

``` java
public boolean canConstruct(String ransomNote, String magazine) {
    int[] rNum = new int[26];
    int[] mNum = new int[26];
    for(int i = 0;i < ransomNote.length();i++){
        int index = ransomNote.charAt(i) - 'a';
        rNum[index]++;
    }
    for(int i = 0;i < magazine.length();i++){
        int index = magazine.charAt(i) - 'a';
        mNum[index]++;
    }
    for(int i = 0;i < 26;i++){
        if(rNum[i] > mNum[i])
            return false;
    }
    return true;
}
```
其实除了思路上的问题，半个暑假没怎么碰java写代码也是有点生疏了，没有了ide提示的库方法在网页上编程还是有点吃力，什么indexOf()，charAt()之类的写法都忘了，还是任重道远啊，2333....

## 382.Linked List Random Node

这题感觉没什么难度，不知道为什么会划在medium档中，也许是他简单的考察了链表的知识。但是对于我来说，之前学C的时候链表用的还是挺多的，所以这里只需要用一个while循环把链表里的信息都存到一个ArrayList里就可以了。

简单实现如下：

``` java
private ArrayList<Integer> List = new ArrayList<Integer>(); 
public Solution(ListNode head) {
    ListNode p = head;
    while(p != null){
        List.add(p.val);
        p = p.next;
    }
}
    
public int getRandom() {
    int random = (int)(Math.random()*List.size());
    return List.get(random);
}
```
挺久没写Java了，简单用了一下ArrayList小小复习一下QvQ