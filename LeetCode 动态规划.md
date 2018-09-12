# LeetCode 动态规划  

## 120 Triangle

Given a triangle, find the minimum path sum from top to bottom. Each step you may move to adjacent numbers on the row below. 
　　For example, given the following triangle

```
[
     [2],
    [3,4],
   [6,5,7],
  [4,1,8,3]
]
```

　　The minimum path sum from top to bottom is 11 (i.e., `2 + 3 + 5 + 1 = 11`). 
　　Note: 
　　Bonus point if you are able to do this using only O(n) extra space, where n is the total number of rows in the triangle. 

**思路：**

上面的三角形可以看成如下样子：

`2`

`3 4`

`6 5 7` 

`4 1 8 3`

按题中的要求，只能往下访问相邻的数字，和5向下相邻的数字是1和8，和7向下相邻的数字是8和3。

而5和1所处的索引相同，8的索引为5的索引加1。

7和8所处的索引相同，3的索引为7的索引加1。

那么我们可以从下往上得出最小和的三角形：

`11            // 11=min(2+9, 2+10)`

`9 10         // 9=min(3+7, 3+6), 10=min(4+6, 4+10)`

`7  6  10    // 7=min(6+4, 6+1), 6=min(5+1, 5+8), 10=min(7+8, 7+3)`

`4  1  8  3`

那么动态规划方程就是：

`F[i] = V[ i ] + min( F[ i ] + F[ i+1 ] ) , 0 <= i < size of every line`

F[ i ]代表从下往上到索引  i  处的最小和，V[ i ] 代表每行第 i 个数字的值。

那么最后的最小和结果就位于F[0]处。

### 递归解法（不能AC）

```java
class Solution {
    public int minimumTotal(List<List<Integer>> triangle) {
        return minTmp(triangle, 0).get(0);
    }
    private List<Integer> minTmp(List<List<Integer>>triangle, int n){
        if( n == triangle.size()-1 )  return triangle.get(n);
        // 此处如果不新开辟空间来存储当前处理行，就会改变原有triangle中的值，导致递归出错
        List<Integer> list = new ArrayList<>();
        for (int t:triangle.get(n)) {
            list.add(t);
        }
        int w = list.size();
        for(int j = 0; j < w; j ++){
            int now = list.get(j);
            list.set(j,now + Math.min(minTmp(triangle, n + 1).get(j), minTmp(triangle, n + 1).get(j+1)));
        }
        return list;
    }
}
                                                                           }
```

### 记忆化搜索（勉强AC但和别的答案无论在时间还是空间上都相差很远）

从递归解法中可以感受到，存在很多重复计算，每次计算一行，该行以下的每一行都要计算一次，实际上可以将计算过的结果保存下来，再用到某一行数据的时候，若已经计算过，就无需再次计算直接返回。

使用一个二维数组来存储已经计算过的内容，如果已经计算过，就不会为0，直接返回该行对应的数据。

```java
class Solution {
    public int minimumTotal(List<List<Integer>> triangle) {
        int[][] memo = new int[triangle.size()][triangle.get(triangle.size()-1).size()];
        return minTmp(triangle, memo, 0).get(0);
    }
    private List<Integer> minTmp(List<List<Integer>>triangle, int[][] memo, int n){
        if( n == triangle.size()-1 ){
            for (int i = 0; i < triangle.get(n).size(); i ++) {
                memo[n][i] = triangle.get(n).get(i);
            }
            return triangle.get(n);
        }
        if(memo[n][0] != 0){//直接返回该行数据
            List<Integer> list = new ArrayList<>();
            for (int i = 0; i < triangle.get(n).size(); i ++) {
                list.add(memo[n][i]);
            }
            return list;
        }
        List<Integer> list = new ArrayList<>();
        for (int t:triangle.get(n)) {
            list.add(t);
        }
        int w = list.size();
        for(int j = 0; j < w; j ++){
            int now = list.get(j);
            list.set(j,now + Math.min(minTmp(triangle, memo,n + 1).get(j), minTmp(triangle, memo,n + 1).get(j+1)));
        }
        for (int i = 0; i < list.size(); i ++){
            memo[n][i] = list.get(i);
        }
        return list;
    }
}
```

### 动态规划（AC，最好的方法） 

从以上记忆化搜索可以看出，用了大量的额外空间，且在数组和链表的转化过程中也浪费了很多的时间。事实上，对于这道题，它就不适合记忆化搜索，记忆化搜索往往需要开辟一个存储空间且需要对这个存储结构赋初值，通过是否等于初值来判断当前计算是否曾经计算过。而本题是链表，链表本身就不适合去赋初值。实际上，一般记忆化搜索用的都是数组。本题适合自底向上来考虑问题，很适合用动态规划来解决。

自底向上处理问题，最下一层不变，倒数第二层的内容替换为当前位置的值与下一层相邻位置最小值的和，以此类推直到最上层。

```java
class Solution {
    public int minimumTotal(List<List<Integer>> triangle) {
        int h = triangle.size();
        if (h < 2){
            return triangle.get(0).get(0);
        }
        for (int i = h - 2; i >= 0; i --){
            for (int j = 0; j < triangle.get(i).size(); j ++){
                triangle.get(i).set(j,triangle.get(i).get(j)
                                +Math.min(triangle.get(i+1).get(j),triangle.get(i+1).get(j + 1)));
            }
        }
        return triangle.get(0).get(0);
    }
}
```





































