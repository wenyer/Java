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

### 递归解法

```java
class Solution {
	public int minimumTotal(List<List<Integer>> triangle) {
   	    int h = triangle.size();
        for (int i = 0; i < h; i ++){
            int w = triangle.get(i).size();
            for(int j = 0; j < w; j ++){
                triangle.get(i).get(j) = triangle.get(i).get(j) + Math.min(minimumTotal()
            }
        }
    }
                                                                           }
```

### 记忆化搜索



### 动态规划 





































