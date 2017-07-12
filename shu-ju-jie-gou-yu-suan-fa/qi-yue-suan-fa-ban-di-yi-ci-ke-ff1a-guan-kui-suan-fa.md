##1.数数
###问题描述
围棋棋盘由横纵19*19条线组成，这些线共组成多少个正方形？
###思路分析
- 依次计算边长为1、2、...、18的正方形由多少个，然后求和
- 计算以某个点为右下角的正方形共有多少个？然后把所有点的正方形数相加。

##2.逻辑推理:完形填空
###问题描述
皇帝不是穷人，在守财奴中也有穷人，所以有一些(  )并不是(  )。
A.皇帝，皇帝 B.守财奴，守财奴
C.守财奴，皇帝 D.皇帝，守财奴
###思路分析
使用离散数学来分析这个题目
从题目的描述可以得出下面三个命题<br>
p:这个人是皇帝<br>
q:这个人是穷人<br>
r:这个人是守财奴<br>
皇帝不是穷人:p -> ~q<br>
在守财奴之中也有穷人:Эx(x∈r^x∈q)<br>
一个命题和它的逆否命题是等价的，所以：<br>
p -> ~q  等价于 q -> ~p<br>
所以Эx(x∈r^x∈q)等价于Эx(x∈r^x∈~p)<br>
答案为：存在一些人，这些人是守财奴并且不是皇帝
##3.字符串表达式的计算
a+b*(c-d)+e
逆波兰表达式：栈的典型应用
##4.最大连续子数组
###问题描述
给定一个数组A[0,...,n-1]，求A的连续子数组，使得子数组的和最大
例如：
数组：1,-2,3,10,-4,7,2,-5
最大子数组：3,10,-4,7,2
###解法
- 暴力法
- 分治法
- 分析法
- 动态规划法

###暴力法分析
1.直接求解A[i,...,j]的值：<br>
0 <= i < n <br>
i <= j < n <br>
数组A的长度j-i+1最大为n
因此时间复杂度O(n^3)
####暴力法代码
```java
    static class Result{
        int max;
        int start;
        int end;

        private Result(int max, int start, int end) {
            this.max = max;
            this.start = start;
            this.end = end;
        }

        @Override
        public String toString() {
            return "maxSum:"+max+"   start:"+start+"   end:"+end;
        }
    }
    private static Result MaxSubArrayBaoLi(int [] array,int length){
        int maxSum = array[0];
        int currSum,start = 0,end = 0;
        for (int i = 0; i < length; i++) {
            for (int j = i; j < length; j++) {
                currSum = 0;
                for (int k = i; k <= j ; k++) {
                    currSum += array[k];
                }
                if(currSum > maxSum){
                    maxSum = currSum;
                    start = i;
                    end = j;
                }
            }
        }
        return new Result(maxSum,start,end);
    }

```
###分治法分析
- 将数组从中间分开，那么最大子数组要么完全在左半边数组，要么完全在右半边数组，要么跨立在分界点上
- 完全在左数组、右数组递归解决
- 跨立在分界点上：实际上是左数组的最大后缀和右数组的最大前缀的和。因此，从分界点分别向前、后扫即可
- 比较这三种情况，取最大的情况

####分治法实现代码
```java

    static class Result{
        int max;
        int start;
        int end;

        private Result(int max, int start, int end) {
            this.max = max;
            this.start = start;
            this.end = end;
        }

        @Override
        public String toString() {
            return "maxSum:"+max+"   start:"+start+"   end:"+end;
        }
    }
    private static Result MaxAddSubFenZhi(int [] arr,int from,int to){
        if(to == from){
            return new Result(arr[from],0,0);
        }
        int middle = (from + to)/2,start,end;
        Result m1 = MaxAddSubFenZhi(arr,from,middle);
        Result m2 = MaxAddSubFenZhi(arr,middle+1,to);
        int left = arr[middle],now = arr[middle];
        start = middle;
        for (int i = middle - 1; i >= from; i--) {
            now += arr[i];
            if(now > left){
                left = now;
                start = i;
            }
        }
        int right = arr[middle+1];
        now = arr[middle+1];
        end = middle+1;
        for (int i = middle + 2; i <= to; i++) {
            now += arr[i];
            if(now > right){
                right = now;
                end = i;
            }
        }
        int sum = left + right;
        Result m3 = new Result(sum,start,end);
        Result maxSum = max(m1,m2,m3);
        return maxSum;
    }

    private static Result max(Result m1,Result m2,Result m3){
        if(m1.max >= m2.max&&m1.max >= m3.max){
            return m1;
        }

        if(m2.max >= m1.max&&m2.max >= m3.max){
            return m2;
        }

        if(m3.max >= m1.max&&m3.max >= m2.max){
            return m3;
        }

        return new Result(-1,-1,-1);
    }
```
####分治法的时间复杂度
![](http://oqnfoupsj.bkt.clouddn.com/17-7-12/65545069.jpg)
###分析法分析
