---
title: Data Struct
date: 2016-06-15 14:47:43
tags:
---

## 权威题库
[leetCode](https://leetcode.com/)

## 递归

### 猴子吃桃
	孙悟空第一天摘下若干蟠桃，当即吃了一半，还不过瘾，又多吃了一个。第二天又将剩下的蟠桃吃掉一半，还不过瘾，又多吃了一个。之后每天早上都吃前一天剩下桃子的一半零一个，到第10天早上想再吃时，就只剩下一个蟠桃了，问孙悟空第一天共摘了多少个蟠桃？

	前一天剩下的蟠桃 = (前一天剩下的一半 + 1个) + 当天剩下的蟠桃
- 递推算法
```
#include <stdio.h>

int func(){
    int total = 1;
    for(int i=10; i>1; i--){
        total = (total + 1)*2;
    }
    return total;
}

int main(){
    printf("first count: %d\n", func());
}
```

- 递归算法
```
#include <stdio.h>

int func(int n){
    if(n==10){
        return 1;
    }else{
        return func(n+1) * 2 + 2;    
    }
}

int main(){
    printf("first count: %d\n", func(1)); //1534
}
```

### 最大公约数与最小公倍数
- 辗转相除法思想
```
int func2(int m, int n){
    if(n == 1 || m == 1){
        return 1;    
    } 
    while(m!=1 && n != 1){
        int res = m % n;
        if(res != 0){
            m = n;
            n = res;
        }else{
            return n;    
        }
    }
    return 1;
}

int main(){
    printf("first count: %d\n", func2(100, 44));
}
```
### 斐波那契数列递推公式 （爬楼梯问题）
f(n) = f(n-2) + f(n-1)  n>2
f(n) = n  				n=1 || n=2
计算复杂度O(2^n) ~ O(2~(n/2))
解决办法： 备忘录法

### 汉诺塔
将A柱的n个盘子盘子借助B柱移到C柱
思路：
1. 将A柱子n-1个盘子借助C柱移动到B柱。
2. 将A柱最大的盘子移动到C柱子。
3. 然后将B柱的n-1个盘子借助A柱子移动到B柱子。
```
#include <stdio.h>
#include <string.h>
#include <unistd.h>

void hanoi(char A, char B, char C, int n){
    if(n==1){
        printf("%c -> %c\n", A, C);
    }else {
        hanoi(A, C, B, n-1);
        printf("%c -> %c\n", A, C);
        hanoi(B, A, C, n-1);
    }
}

int main(){
   hanoi('A', 'B', 'C', 10); 
}
```

## 链表
### 链表原理
### 逆序打印链表
- 依次将节点压栈
- 递归方式
### 链表的最大元素

## DFS和BFS
- DFS 深度优先搜索算法
- BFS 广度优先搜索算法

- 前序遍历：层级列表的输出、快速排序、图的DFS、排列组合、回溯法
- 中序遍历：从右往左打印二叉树、二叉排序树的迭代、汉诺塔、折纸问题
- 后序遍历：文件的删除、归并排序、区域动态规划
- 层序遍历：二叉链表与顺序表的互转、图的BFS、分支界限法 (符合人类感觉，非递归算法，需要队列)

![](http://www.geesugar.com/BlogImg/%E5%85%88%E5%BA%8F%E9%81%8D%E5%8E%86%E7%AE%97%E6%B3%95%E6%A8%A1%E6%9D%BF.png)


![](http://www.geesugar.com/BlogImg/midbianli.png)

![](http://www.geesugar.com/BlogImg/houxubianlimuban.png)

![](http://www.geesugar.com/BlogImg/cengxubianlimuban.png)

![](http://www.geesugar.com/BlogImg/bianlikoujue.png)

时间复杂度O(n) 空间复杂度树的深度 O(logN)~O(N)

## Stack
