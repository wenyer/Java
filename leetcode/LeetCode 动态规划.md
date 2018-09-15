# eetCode 动态规划  

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

------



##  64 Minimum Path Sum 

Given a *m* x *n* grid filled with non-negative numbers, find a path from top left to bottom right which *minimizes* the sum of all numbers along its path.

**Note:** You can only move either down or right at any point in time.

**Example:**

```
Input:
[
  [1,3,1],
  [1,5,1],
  [4,2,1]
]
Output: 7
Explanation: Because the path 1→3→1→1→1 minimizes the sum.
```

### 递归解法（超时，不能AC）

```java
思考：grid[i][j]到最右下角的的值是grid[i][j]的值加上grid[i+1][j]到最右下角的值和grid[i][j+1]到最右下角的值，注意考虑边界条件，当grid[i][j]已经是最下方的元素时就不能再往下只能往右，同理，已经到了最右方就不能往右只能往下。
```

```java
class Solution {
    public int minPathSum(int[][] grid) {
        return minTemp(grid, 0, 0);
    }
    private int minTemp(int[][] grid, int i, int j){
        int m = grid.length;
        int n = grid[m-1].length;
        if (i == m -1 && j == n -1){
            return grid[m-1][n-1];
        }
        if (i == m -1 && j < n - 1){
            return grid[i][j] + minTemp(grid, i, j + 1);
        }
        if (j == n -1 && i < m - 1){
            return grid[i][j] + minTemp(grid, i + 1, j);
        }
        return grid[i][j] + Math.min(minTemp(grid, i + 1, j), minTemp(grid, i, j + 1));         
    }
}
```

### 记忆化搜素（AC）  

```java
由递推法可以很容易地发现，存在大量的重复计算，因为每次计算某个位置到右下角的最短路径时都要把在其下方和左方的元素的最短路径再算一遍，这会产生很多的重复计算。采用记忆化搜索就是考虑将已经计算过的路径结果保留下来，以免多次计算。
```

```java
class Solution {
    public int minPathSum(int[][] grid) {
        int m = grid.length;
        int n = grid[m-1].length;
        int[][] memo = new int[m][n];
        return minTemp(grid, memo, 0, 0);
    }
    private int minTemp(int[][] grid, int[][] memo, int i, int j){
        if (memo[i][j] != 0)    return memo[i][j];
        int m = grid.length;
        int n = grid[m-1].length;
        if (i == m -1 && j == n -1){
            memo[i][j] = grid[m-1][n-1];
            return memo[i][j];
        }
        if (i == m -1 && j < n - 1){
            memo[i][j] = grid[i][j] + minTemp(grid, memo, i, j + 1);
            return memo[i][j];
        }
        if (j == n -1 && i < m - 1){
            memo[i][j] = grid[i][j] + minTemp(grid, memo, i + 1, j);
            return memo[i][j];
        }
        memo[i][j] = grid[i][j] + Math.min(minTemp(grid, memo, i + 1, j), minTemp(grid, memo, i, j + 1));
        return memo[i][j];          
    }
}
```

### 动态规划（AC）

```java
记忆化搜索虽然通过保留计算过的路径的值来消除重复计算，但是不可避免的使用了和原问题数组一样大小的存储空间，这对于需要知道具体路径来说是必要的，但是对于本题而言，只需要知道最短路径的值，而不需要最短路径的具体走法，所以其实是不需要额外的存储空间，只需要将计算的结果值存入原数组中就好。这就需要自底向上地解决问题，也就是动态规划。
```

```java
class Solution {
    public int minPathSum(int[][] grid) {
        int m = grid.length;
        int n = grid[m-1].length;
        for (int j = n-1; j >= 0; j --){
            grid[m-1][j] = j+1 < n ? grid[m-1][j] + grid[m-1][j+1] : grid[m-1][j];
        }
        for (int i = m-2; i >= 0; i --){
            for (int j = n-1; j >= 0; j --){
                grid[i][j] = j + 1 < n ? grid[i][j] + Math.min(grid[i+1][j], grid[i][j+1]) : grid[i][j] + grid[i+1][j];
            }
        }
        return grid[0][0];
    }
}
```

------



## 279 Perfect Squares(经典题)   

Given a positive integer *n*, find the least number of perfect square numbers (for example, `1, 4, 9, 16, ...`) which sum to *n*.

**Example 1:**

```
Input: n = 12
Output: 3 
Explanation: 12 = 4 + 4 + 4.
```

**Example 2:**

```
Input: n = 13
Output: 2
Explanation: 13 = 4 + 9.
```

### 动态规划一  

```java
采用动态规划实现。用 dp[i] 数组存储第 i 个数的完美平方数。递推式为：dp[i] = Math.min(dp[j] + dp[i-j], dp[i]，认为 i 的完全平方数是和为 i 的两个完全平方数 dp[j] 和 dp[i-j]的和，然后从中取最小。
```

```java
public class Solution {
    public int numSquares(int n) {
        int[] dp = new int[n+1];
        Arrays.fill(dp, Integer.MAX_VALUE);
        dp[1] = 1;
        for(int i = 1; i <= n; i++) {
            int sqr = (int)Math.sqrt(i);
            if(sqr * sqr == i) dp[i] = 1;  //如果 i 本身是个平方数，就将 dp[i] 置1
            else {
                for(int j = 1; j <= i/2; j++) {
                    dp[i] = Math.min(dp[j] + dp[i-j], dp[i]);  //从0开始遍历所有和为 i 的 dp，使得 dp[i]取最小
                }
            }
        }
        return dp[n];
    }
}

```

### 动态规划二  

依然采用动态规划。我们仔细思考  

![](F:\GitHub_Notes\Java\leetcode\动态规划_1.png)

如图所示，红色部分表示平方数，所有的完美平方数都可以看做一个普通数加上一个完美平方数，那么递推式就变为了：dp[i + j * j] = Math.min(dp[i] + 1, dp[i + j * j])，

显然这种方案减少了不少计算量：

```java
public class Solution {
    public int numSquares(int n) {
        int[] dp = new int[n+1];
        Arrays.fill(dp, Integer.MAX_VALUE);
        for(int i = 0; i * i <= n; i++) 
            dp[i * i] = 1;
        for(int i = 1; i <= n; i++) {  //选定第一个数为 i
            for(int j = 1; i + j * j <= n; j++) {  //选定另一个数为 j*j
                dp[i + j * j] = Math.min(dp[i] + 1, dp[i + j * j]);  //从小到大查找
            }
        }
        return dp[n];
    }
}
```





































