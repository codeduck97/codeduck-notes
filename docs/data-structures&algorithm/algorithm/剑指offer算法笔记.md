# 剑指offer算法笔记

## 03.数组中重复的数字


在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

**示例 1：**

```
输入：
[2, 3, 1, 0, 2, 5, 3]
输出：2 或 3 
```


限制：

`2 <= n <= 100000`

**解法一：**使用字典集合，对数组中的元素进行遍历，当遇到第一个重复的数字时返回即可

**解法二：**由于长度为`n`的数组内，数值范围在`0 ~ n-1`，则数组内必有重复的数字。可以利用下标对每个数字进行复位（即将他们放到与数值对应的下标上），当遇到下标与下标数值不等且以该下标的值为下标得出的数值相等时，则找到重复的数值。

```markdown
# 下标	  		0		1		2		3		4
# 数值	  		2		4		1		0		0

# 交换			1		4		2		0		0		每次交换，主动交换的下标数值会归位
# 交换			4		1		2		0		0
# 交换			0		1		2		0		4
# 当遍历到下标为3的数值时，发现，下标与下标数值不等且以该下标的值为下标得出的数值相等，则为重复的数字
```

```java
class Solution {
    public int findRepeatNumber(int[] nums) {
        /**
         * 当前下标 i 的数值 m 与以 m 为下标的数值进行交换，则交换一次，m就会归位，此时 nums[m] = m
         * num[i] <=> num[num[i]]
         */
        int temp;
        for (int i = 0; i < nums.length; i++) {
            // 当前下标与下标数值不等，则进行交换
            while (nums[i] != i) {
                // 如果 当前下标数值，与下标的数值为下标的值相等，则找到重复的数字
                // [0,1,2,0,3]， 即 nums[3] = 0 == nums[0] == 0;
                if (nums[i] == nums[nums[i]]) return nums[i];
                temp = nums[i];
                nums[i] = nums[temp];
                nums[temp] = temp;
            }
        }
        return -1;
    }
}
```



## 04. 二维数组中的查找

在一个 n * m 的二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

 **示例:**

现有矩阵 matrix 如下：

```
[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]
```

给定 target = `5`，返回 `true`。

给定 target = `20`，返回 `false`。

**限制：**

```
0 <= n <= 1000
0 <= m <= 1000
```

**解题思路：**从左上角开始遍历二维数组，如果当前数字大于target，则向下遍历，小于向左遍历。

注意：空数组的判断

```java
class Solution {
    public boolean findNumberIn2DArray(int[][] matrix, int target) {
        
        if (matrix != null && matrix.length > 0) {
            int row = 0, col = matrix[0].length - 1;
            while (row < matrix.length && col >= 0) {
                if (matrix[row][col] == target) return true;
                else if (matrix[row][col] > target) col--;
                else row++;
            }
        }
        return false;
    }
}
```



## 05. 替换空格

请实现一个函数，把字符串 `s` 中的每个空格替换成"%20"。

 **示例 1：**

```
输入：s = "We are happy."
输出："We%20are%20happy."
```

 **限制：**

```
0 <= s 的长度 <= 10000
```

`解题思路：`先声明一个原字符串3倍大的字符数组（一个空格要替换为3个字符的"%20"）

然后遍历将整个字符串，判断并赋值给字符数组。

最后将字符数组处理为字符串返回。

PS: 特殊输入"   "三个空格应该输出"%20%20%20"

```java
class Solution {
    public String replaceSpace(String s) {
        int idx = 0;
        char[] array = new char[s.length() * 3];
        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            if (c == ' ') {
                array[idx++] = '%';
                array[idx++] = '2';
                array[idx++] = '0';
            } else {
                array[idx++] = c;
            }
        }
        String res = new String(array, 0, idx);
        return res;
    }
}
```

## 06. 从尾到头打印链表

输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。

 **示例 1：**

```
输入：head = [1,3,2]
输出：[2,3,1]
```

 **限制：**

```
0 <= 链表长度 <= 10000
```

```java
class Solution {
    /**
     * 1.链表长度未知
     * 2.需要返回数组
     * 3.由于是单向链表，因此只能从头到尾记录节点，所以使用栈存储节点，然后出栈返回数据
     * 4.由于链表长度可能为0，因此需要特殊处理
     */
    public int[] reversePrint(ListNode head) {
        Deque<Integer> stack = new LinkedList<>();
        // 入栈
        while (head != null) {
            stack.push(head.val);
            head = head.next;
        }
        // 出栈：栈为空或不为空
        int[] res = new int[stack.size()];
        int idx = 0;
        while (!stack.isEmpty()) res[idx++] = stack.pop();
        return res;
    }
}
```



## 07. 重建二叉树

输入某二叉树的前序遍历和中序遍历的结果，请重建该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。

例如，给出

```
前序遍历 preorder = [3,9,20,15,7]
中序遍历 inorder = [9,3,15,20,7]
```

返回如下的二叉树：

```
    3
   / \
  9  20
    /  \
   15   7
```

 **限制：**

```
0 <= 节点个数 <= 5000
```

**方法一：递归**

![image-20201022152626788](https://jason-01.oss-cn-hangzhou.aliyuncs.com/public/image/markdown/image-20201022152626788.png)

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {

    public TreeNode buildTree(int[] preorder, int[] inorder) {
        if (preorder == null || preorder.length == 0) return null;

        // idxMap记录数组中所有下标以及下标对应的值，用于将数组划分为左右子树
        HashMap<Integer, Integer> idxMap = new HashMap<>();
        int len = preorder.length;
        for (int i = 0; i < len; i++) {
            idxMap.put(inorder[i], i);
        }
        TreeNode root = rebuildTree(preorder, 0, len - 1, inorder, 0, len - 1, idxMap);
        return root;
    }

    private TreeNode rebuildTree(int[] preorder, int preorderStart, int preorderEnd, int[] inorder, int inorderStart, int inorderEnd, HashMap<Integer, Integer> idxMap) {

        /**
         * 判断先序遍历的下标范围的开始和结束，若开始大于结束，则当前的二叉树中没有节点，返回空值 null
         */
        if (preorderStart > preorderEnd) {
            return null;
        }

        // 先序数组的第一个节点总是根节点
        int rootVal = preorder[preorderStart];
        TreeNode root = new TreeNode(rootVal);

        // 如果先序数组或中序数组的起始索引和终点索引相等，
        // 则当前的二叉树中恰好有一个节点，根据节点值创建该节点作为根节点并返回。
        if (preorderStart == preorderEnd) {
            return root;
        } else {
            // 根据根节点的值，在中序数组中获取该根节点所处的索引值
            int rootIdx = idxMap.get(rootVal);
            // 根据根节点的下标，可以计算出以root为根节点的左右子树的下标范围
            int leftNodes = rootIdx - inorderStart, rightNodes = inorderEnd - rootIdx;

            /**
             * 根据下标范围，将数组分为左右子树，分别调用rebuildTree()函数进行处理
             */

            // 左子树调用
            TreeNode leftSubtree = rebuildTree(
                    preorder, preorderStart + 1, preorderStart + leftNodes,
                    inorder, inorderStart, rootIdx - 1, idxMap);
            // 右子树调用
            TreeNode rightSubtree = rebuildTree(
                    preorder, preorderEnd - rightNodes + 1, preorderEnd,
                    inorder, rootIdx + 1, inorderEnd, idxMap);
            root.left = leftSubtree;
            root.right = rightSubtree;
            return root;
        }
    }
}
```

**方法二：迭代**

![image-20201023104722324](https://jason-01.oss-cn-hangzhou.aliyuncs.com/public/image/markdown/image-20201023104722324.png)

![image-20201023104758857](https://jason-01.oss-cn-hangzhou.aliyuncs.com/public/image/markdown/image-20201023104758857.png)

**迭代效果明显好于递归**

执行用时：2 ms, 在所有 Java 提交中击败了98.09%的用户

内存消耗：38.1 MB, 在所有 Java 提交中击败了99.72%的用户

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {

    public TreeNode buildTree(int[] preorder, int[] inorder) {
        if (preorder == null || preorder.length == 0) {
            return null;
        }

        LinkedList<TreeNode> stack = new LinkedList<>();
        TreeNode root = new TreeNode(preorder[0]);
        // 先序数组第一个元素为根节点，将其入栈
        stack.push(root);
        // 记录中序下标
        int inorderIdx = 0;
        for (int i = 1; i < preorder.length; i++) {
            /**
             * 如果栈顶节点值不等于中序索引值，则持续入栈
             */
            int preorderVal = preorder[i];
            TreeNode topNode = stack.peek();
            if (topNode.val != inorder[inorderIdx]) {
                TreeNode node = new TreeNode(preorderVal);
                topNode.left = node;
                stack.push(topNode.left);
            } else {
                /**
                 * 如果栈不为空且栈顶节点值等于中序索引值，则持续出栈
                 */
                while (!stack.isEmpty() && stack.peek().val == inorder[inorderIdx]) {
                    topNode = stack.pop();
                    inorderIdx++;
                }
                topNode.right = new TreeNode(preorderVal);
                stack.push(topNode.right);
            }
        }
        return root;
    }
}
```



##  09. 用两个栈实现队列

用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 `appendTail` 和 `deleteHead` ，分别完成在队列尾部插入整数和在队列头部删除整数的功能。(若队列中没有元素，`deleteHead` 操作返回 -1 )

**示例 1：**

```
输入：
["CQueue","appendTail","deleteHead","deleteHead"]
[[],[3],[],[]]
输出：[null,null,3,-1]
```

**示例 2：**

```
输入：
["CQueue","deleteHead","appendTail","appendTail","deleteHead","deleteHead"]
[[],[],[5],[2],[],[]]
输出：[null,-1,null,null,5,2]
```

**提示：**

- `1 <= values <= 10000`
- `最多会对 appendTail、deleteHead 进行 10000 次调用`

```java
class CQueue {

    Deque<Integer> stack1;
    Deque<Integer> stack2;

    public CQueue() {
        stack1 = new LinkedList<>();
        stack2 = new LinkedList<>();
    }

    // 入队: 将value入栈到stack1
    public void appendTail(int value) {
        stack1.push(value);
    }

    /**
     * 出队:
     * 如果stack2不为空，则弹出栈顶元素
     * 如果stack2为空，stack1不为空，则将stack1中的元素出栈，按顺序入stack2，弹出stack2的栈顶元素
     * 如果stack2为空，stack1为空，则队列为空,返回-1
     */
    public int deleteHead() {
        if (stack2.isEmpty()) {
            while (!stack1.isEmpty()) {
                stack2.push(stack1.pop());
            }
        }

        if (!stack2.isEmpty()) 
            return stack2.pop();
        else 
            return -1;
    }
}

/**
 * Your CQueue object will be instantiated and called as such:
 * CQueue obj = new CQueue();
 * obj.appendTail(value);
 * int param_2 = obj.deleteHead();
 */
```

## 10- I. 斐波那契数列

写一个函数，输入 `n` ，求斐波那契（Fibonacci）数列的第 `n` 项。斐波那契数列的定义如下：

```
F(0) = 0,   F(1) = 1
F(N) = F(N - 1) + F(N - 2), 其中 N > 1.
```

斐波那契数列由 0 和 1 开始，之后的斐波那契数就是由之前的两数相加而得出。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

 

**示例 1：**

```
输入：n = 2
输出：1
```

**示例 2：**

```
输入：n = 5
输出：5
```

 **提示：**

- `0 <= n <= 100`

```java
class Solution {
    public int fib(int n) {
        int[] res = new int[]{0, 1};
        if (n < 2) return res[n];

        int temp;
        for (int i = 1; i < n; i++) {
            temp = res[1];

            if (res[0] + res[1] < (1e9+7)){
                res[1] = res[0] + res[1];
            } else {
                res[1] = (int) ((res[0] + res[1]) % (1e9+7));
            }
            
            res[0] = temp;
        }
        return res[1];
    }
}
```



## 10- II. 青蛙跳台阶问题

一只青蛙一次可以跳上1级台阶，也可以跳上2级台阶。求该青蛙跳上一个 `n` 级的台阶总共有多少种跳法。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

**示例 1：**

```
输入：n = 2
输出：2
```

**示例 2：**

```
输入：n = 7
输出：21
```

**示例 3：**

```
输入：n = 0
输出：1
```

**提示：**

- `0 <= n <= 100`

解题思路：类似于斐波那契数列

题目要求：n=0 和 n=1 时，返回1，则F（0） = F（1）= 1。

有1个台阶时，F（1） = 1，有一种跳法

有2个台阶时，F（2）= 2，有两种跳法

有3个台阶时，F（3） = F（1）+ F（2），有三种跳法

……………

```java
class Solution {
    public int numWays(int n) {
        int[] res = new int[]{1, 1};
        if (n < 2) return res[n];

        int temp;
        for (int i = 1; i < n; i++) {
            temp = res[1];

            if (res[0] + res[1] < (1e9+7)){
                res[1] = res[0] + res[1];
            } else {
                res[1] = (int) ((res[0] + res[1]) % (1e9+7));
            }
            
            res[0] = temp;
        }
        return res[1];
    }
}
```



## 11. 旋转数组的最小数字

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。输入一个递增排序的数组的一个旋转，输出旋转数组的最小元素。例如，数组 `[3,4,5,1,2]` 为 `[1,2,3,4,5]` 的一个旋转，该数组的最小值为1。 

**示例 1：**

```
输入：[3,4,5,1,2]
输出：1
```

**示例 2：**

```
输入：[2,2,2,0,1]
输出：0
```

测试过程中特殊的参数： `[1], [1,1,1], [1,0,1,1,1]`

```java
class Solution {
    public int minArray(int[] numbers) {
        if (numbers == null || numbers.length == 0) return -1;
        int left =0, mid = 0;
        int right = numbers.length - 1;

        // left < right 是防止数组长度为1时，会陷入死循环
        while (numbers[left] >= numbers[right] && left < right) {

            if (right - left == 1) {
                mid = right;
                break;
            }

            mid = (right + left) / 2;

            // 如果碰到三个索引值相同时，二分法就失效了,采用顺序查找
            if (numbers[mid] == numbers[left] && numbers[mid] == numbers[right]) {
                return minInOrder(numbers,left,right);
            }


            if (numbers[mid] >= numbers[left]) {
                left = mid;
            }else if (numbers[mid] <= numbers[right]){
                right = mid;
            }
        }
        return numbers[mid];
    }

    private int minInOrder(int[] numbers, int left, int right) {
        int flag = numbers [left];
        // 遍历索引区间，记录最小的值并返回
        for (int i = left + 1; i < right; i++) {
            if (numbers[i] < flag) 
                flag = numbers[i];
        }
        return flag;
    }
}
```

网友版：TQL

测试过程中特殊的参数： `[1], [1,1,1], [1,0,1,1,1]`

```java
class Solution {
    public int minArray(int[] numbers) {
        int l = 0, r = numbers.length - 1;
        while (l < r) {
            int mid = ((r - l) >> 1) + l;
            //如果最右边的值大于中间值，则右半部分有序
            if (numbers[mid] < numbers[r]) {
                r = mid;
            } else if (numbers[mid] > numbers[r]) {
                l = mid + 1;
             //去重
            } else r--;
        }
        return numbers[l];
    }
}
```





## 12. 矩阵中的路径

请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一格开始，每一步可以在矩阵中向左、右、上、下移动一格。如果一条路径经过了矩阵的某一格，那么该路径不能再次进入该格子。例如，在下面的3×4的矩阵中包含一条字符串“bfce”的路径（路径中的字母用加粗标出）。

[["a","**b**","c","e"],
["s","**f**","**c**","s"],
["a","d","**e**","e"]]

但矩阵中不包含字符串“abfb”的路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入这个格子。

 **示例 1：**

```
输入：board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "ABCCED"
输出：true
```

**示例 2：**

```
输入：board = [["a","b"],["c","d"]], word = "abcd"
输出：false
```

**提示：**

- `1 <= board.length <= 200`
- `1 <= board[i].length <= 200`

```java
class Solution {
    public boolean exist(char[][] board, String word) {
        char[] words = word.toCharArray();

        for (int i = 0; i < board.length; i++) {
            for (int j = 0; j < board[0].length; j++) {
                // 从矩阵[0,0]开始索引，寻找符合word[0]的字符，然后进行深度优先搜索
                if (dfs(board, i, j, words, 0)) return true;
            }
        }
        return false;
    }

    /**
     * desc:
     * <p>
     *
     * @param board:二维矩阵数组
     * @param i:矩阵横向索引
     * @param j:矩阵纵向索引
     * @param words:目标字符数组
     * @param k:words字符数组索引
     * @return
     */
    private boolean dfs(char[][] board, int i, int j, char[] words, int k) {

        // 边界约束条件 以及 递归结束条件
        if (i < 0 || i >= board.length || j < 0 || j >= board[0].length 
                || board[i][j] != words[k]) return false;

        // 搜索成功的条件是当k索引到words最后一个字符时
        // 能到该判断条件说明：words[k] == board[i][j]
        if (k == words.length - 1) return true;

        char temp = board[i][j];
        
        // '/' 把当前索引过的位置标记起来，其他字符无法再次访问
        board[i][j] = '/';
        
        // 以当前索引位置为中心，向四周进行深度搜索，如果任何一个搜索成功时，则找到了符合words的路径
        boolean res = dfs(board, i + 1, j, words, k + 1) || dfs(board, i, j + 1, words, k + 1)
                || dfs(board, i - 1, j, words, k + 1) || dfs(board, i, j - 1, words, k + 1);
        
        // 所有被标记的索引在回溯的过程中要恢复当前索引的初始值
        board[i][j] = temp;
        
        return res;
    }
}
```



## 13. 机器人的运动范围

地上有一个m行n列的方格，从坐标 `[0,0]` 到坐标 `[m-1,n-1]` 。一个机器人从坐标 `[0, 0] `的格子开始移动，它每次可以向左、右、上、下移动一格（不能移动到方格外），也不能进入行坐标和列坐标的数位之和大于k的格子。例如，当k为18时，机器人能够进入方格 [35, 37] ，因为3+5+3+7=18。但它不能进入方格 [35, 38]，因为3+5+3+8=19。请问该机器人能够到达多少个格子？

 **示例 1：**

```
输入：m = 2, n = 3, k = 1
输出：3
```

**示例 2：**

```
输入：m = 3, n = 1, k = 0
输出：1
```

**提示：**

- `1 <= n,m <= 100`
- `0 <= k <= 20`

**解一：深度优先遍历**

```java
class Solution {

    boolean[][] visited;
    int m, n, k;

    public int movingCount(int m, int n, int k) {
        if (m <= 0 || n <= 0 || k < 0) {
            return 0;
        }
        this.m = m;
        this.n = n;
        this.k = k;
        // 初始化时，所有的坐标都是false
        this.visited = new boolean[m][n];
        // 深度优先遍历：从下标[0,0]开始
        return dfs(0, 0);
    }

    private int dfs(int i, int j) {
        /**
         * 深度优先遍历结束条件
         *
         * 1. 横向索引坐标 >= m
         * 2. 纵向索引坐标 >= n
         * 3. 两个坐标的位数之和 > k
         * 4. 此索引坐标已被访问过
         * 则返回 0
         *
         */
        if (i >= m || j >= n || (getDigitSum(i) + getDigitSum(j)) > k || visited[i][j]){
            return 0;
        }
        // 标记访问过的坐标
        visited[i][j] = true;
        // 只需要向下和向右索引
        return 1 + dfs(i + 1, j) + dfs(i, j + 1);
    }

    // 计算一个数的位数总和
    private int getDigitSum(int x) {

        int sum = 0;
        while (x != 0) {
            sum += x % 10;
            x = x / 10;
        }
        return sum;
    }
            
}
```

**解二：深度优先遍历**

- **BFS/DFS ：** 两者目标都是遍历整个矩阵，不同点在于搜索顺序不同。DFS 是朝一个方向走到底，再回退，以此类推；BFS 则是按照“平推”的方式向前搜索。
- **BFS 实现：** 通常利用队列实现广度优先遍历。

算法解析：

- **初始化：** 将机器人初始点 (0, 0)加入队列 queue ；

- **迭代终止条件：** queue 为空。代表已遍历完所有可达解。

- **迭代工作：**
- **单元格出队：** 将队首单元格的 索引、数位和 弹出，作为当前搜索单元格。
  
- **判断是否跳过：** 若 ① 行列索引越界 或 ② 数位和超出目标值 k 或 ③ 当前元素已访问过 时，执行 continue 。
  
- **标记当前单元格 ：**将单元格索引 (i, j) 存入 Set visited 中，代表此单元格 已被访问过 。
  
- **单元格入队：** 将当前元素的 下方、右方 单元格的 索引、数位和 加入 queue 。
  
- **返回值：** Set visited 的长度 len(visited) ，即可达解的数量。

```java
class Solution {

    public int movingCount(int m, int n, int k) {
        
        boolean[][] visited = new boolean[m][n];
        Queue<int[]> queue = new LinkedList<>();
        int res = 0;

        queue.add(new int[]{0, 0});
        while (queue.size() > 0) {
            int[] node = queue.poll();
            int i = node[0], j = node[1];

            /**
             * 广度优先搜索结束条件
             *
             * 1. 横向索引坐标 >= m
             * 2. 纵向索引坐标 >= n
             * 3. 两个坐标的位数之和 > k
             * 4. 此索引坐标已被访问过
             * 则返回 0
             *
             */
            if (i >= m || j >= n || (getDigitSum(i) + getDigitSum(j)) > k || visited[i][j]) {
                continue;
            }
            res++;
            visited[i][j] = true;
            queue.add(new int[]{i + 1, j});
            queue.add(new int[]{i, j + 1});
        }
        return res;
    }

    // 计算一个数的位数总和
    private int getDigitSum(int x) {

        int sum = 0;
        while (x != 0) {
            sum += x % 10;
            x = x / 10;
        }
        return sum;
    }
            
}
```

## 14-I. 剪绳子

给你一根长度为 `n` 的绳子，请把绳子剪成整数长度的 `m` 段（m、n都是整数，n>1并且m>1），每段绳子的长度记为 `k[0],k[1]...k[m-1]` 。请问 `k[0]*k[1]*...*k[m-1]` 可能的最大乘积是多少？例如，当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此时得到的最大乘积是18。

**示例 1：**

```
输入: 2
输出: 1
解释: 2 = 1 + 1, 1 × 1 = 1
```

**示例 2:**

```
输入: 10
输出: 36
解释: 10 = 3 + 3 + 4, 3 × 3 × 4 = 36
```

**提示：**

- `2 <= n <= 58`

```java
class Solution {
    public int cuttingRope(int n) {
        // 绳子长度为3，则可剪为 2,1,所以f(3) = 2 * 1 = 3 -1
        if (n <= 3) return n-1;

        // f(n) = f(i) * f(n-i)
        // 所以只需要保证f(i)和f(n-i)是当前绳段的最大乘积即可
        // 使用dp数组来保存所有绳段的最大乘积
        // 绳段 2、 3 是最优解，以此为条件向上递推
        int[] dp = new int[n + 1];
        dp[2] = 2;
        dp[3] = 3;

        for (int i = 4; i <= n; i++) {
            for (int j = 2; j <= i/2; j++) {
                dp[i] = Math.max(dp[j] * dp[i - j], dp[i]);
            }
        }
        return dp[n];
    }
}
```

## 14- II. 剪绳子 

给你一根长度为 `n` 的绳子，请把绳子剪成整数长度的 `m` 段（m、n都是整数，n>1并且m>1），每段绳子的长度记为 `k[0],k[1]...k[m - 1]` 。请问 `k[0]*k[1]*...*k[m - 1]` 可能的最大乘积是多少？例如，当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此时得到的最大乘积是18。

答案需要取模 **1e9+7（1000000007）**，如计算初始结果为：1000000008，请返回 1。

 **示例 1：**

```
输入: 2
输出: 1
解释: 2 = 1 + 1, 1 × 1 = 1
```

**示例 2:**

```
输入: 10
输出: 36
解释: 10 = 3 + 3 + 4, 3 × 3 × 4 = 36
```

 **提示：**

- `2 <= n <= 1000`

```java
class Solution {
    public int cuttingRope(int n) {
        // 绳子长度为3，则可剪为 2,1,所以f(3) = 2 * 1 = 3 -1
        if (n <= 3) return n-1;
        if (n == 4) return n;

        long res = 1;
        while (n > 4) {
            res = (res * 3)% (1000000007);
            n -= 3;

        }
        return(int) (res * n % 1000000007);
    }
}
```



## 15. 二进制中1的个数

> 输入一个整数，计算出该整数对应二进制数中 1 的个数（整数包含正整数和负整数）

```java
public static void main(String[] args) {
    System.out.println(solution03(-4));	// 30
    System.out.println(Integer.toBinaryString(-4));// 11111111111111111111111111111100
}

/**
 * 该算法 对整数对应的二进制数 进行运算
 * 运算方法为 n = n & (n - 1 )
 * 即 n & (n - 1 ) = 0110 & 0101 = 0100 赋值给 n, COUNT++
 * 0100 & 0011 = 0000 赋值给 n ，COUNT++
 * n = 0 时，结束运算
 */
public static int solution03(int num) {
    int count = 0;
    while (num != 0) {
        num = (num - 1) & num;
        count++;
    }
    return count;
}
```

## 16. 数值的整数次数

> 实现函数double Power(double base, int exponent)，求base的exponent次方。不得使用库函数，同时不需要考虑大数问题。

 示例 1:

```
输入: 2.00000, 10
输出: 1024.00000
```

示例 2:

```
输入: 2.10000, 3
输出: 9.26100
```

示例 3:

```
输入: 2.00000, -2
输出: 0.25000
解释: 2^-2 = 1/(2^2) = 1/4 = 0.25
```


说明:

-100.0 < x < 100.0
**n 是 32 位有符号整数**，其数值范围是 [−2^31, 2^31 − 1] 。

**函数实现并不难，难的是要充分考虑到计算异常**

```java
public static double myPower(double base, int exponent){

    // 底数为0，指数为负数，则会发生计算异常
    if (base == 0 && exponent < 0){
        return 0.0;
    }
	
    // 获取指数的绝对值
    int absExp = exponent;
    if (exponent < 0){
        absExp = -exponent;
    }

    // 进行无符号指数的求指运算
    double result = powerWithUnsignedExponent(base,absExp);
    
    // 如果指数小于0，则对求取结果求倒
    if (exponent < 0){
        result =  1.0 / result;
    }
    return result;
}

/**
 * 无符号整型求指函数
 */
private static double powerWithUnsignedExponent(double base, int exponent) {
    double result = 1.0;
    for (int i = 1; i <= exponent; i++){
        result *= base;
    }
    return result;
}
```

**对无符号整型求指函数进行优化，利用如下公式进行求指运算：**

![image-20210113094814939](images/image-20210113094814939.png)

因此，当n = 16为偶数时，我们只需计算 a ^ 8 * a ^ 8;

​				当n = 17 为奇数时，我们只需计算 a ^ 8 * a ^ 8 * a。因此可以使用递归进行计算。

> 以下算法使用了两个小技巧：
>
> 1. 使用右移进行优化除2操作
> 2. 偶数的二进制数最低位为0，而奇数最低位为1，因此`exponent & 1`可以代替取余运算判断奇偶性

```java
private static double powerWithUnsignedExponent(double base, int exponent) {

    // 指数为0时直接返回1
    if (exponent == 0){
        return 1;
    }
    
    // 正常情况下的递归退出条件
    if (exponent == 1){
        return base;
    }

    // exponent >> 1 = exponent / 2;
    double result = powerWithUnsignedExponent(base,exponent >> 1);
    result *= result;

    // 奇数的二进制数最低位为1,若指数为奇数则还需 * base
    if ((exponent & 1) == 1){
        result *= base;
    }
    return result;
}
```

**:warning:这里会有一个奇怪的数值会导致上述代码发生栈溢出错误！**

例如：`1.00000 -2147483648` 

**分析为什么会发生栈溢出？**

`int`的取值范围为：`-2147483648` 到 `2147483647`，在合法的取值范围内，若输入指数为 `-2147483648` 在转为正整数时会发生取值越界，因此在 `int exponent = -2147483648` 取绝对值时，`int absExp = |exponent|` 依然为 `-2147483648`，若不如此则会发生取值越界（当然负数 -2147483647 的绝对值为 2147483647），那么在指数为负数的情况下，对其进行右移的最终结果是 `-1`（-1 >> 1 = -1），则此时对上述无符号整型求指函数来说，并没有相应的退出条件，因此会陷入无穷的递归调用直至栈溢出！！！

```shell
# 为什么 -1 >> 1 = -1
 1的原码: 0000 0001
-1的原码: 1000 0001
-1的反码: 1111 1110
-1的补码: 1111 1111
# 因此对 1111 1111 >> 1 = 1111 1111 = -1
```

正确的做法是将右移改为除2操作（-1 / 2 = 0）

```java
double result = powerWithUnsignedExponent(base,exponent / 2);
```

## 17. 打印从1到最大的n位数

输入数字 n，按顺序打印出从 1 到最大的 n 位十进制数。比如输入 3，则打印出 1、2、3 一直到最大的 3 位数 999。

示例 1:

```
输入: n = 1
输出: [1,2,3,4,5,6,7,8,9]
```


说明：

用返回一个整数列表来代替打印
n 为正整数

> 不考虑大数的情况下

```java
public static int[] printNumbers(int n) {

    if(n < 0){
        int[] res = {};
        return res;
    }

    int len = 1;
    int i = 0;
    while(i < n){
        i++;
        len *= 10;
    }

    int[] res = new int[len - 1];
    for(int j = 1; j < len; j++){
        res[j-1] = j;
    }
    return res;
}
```

> 若输入一个大数比如 n = 20，则 len = 10^20次方，其长度远超 int的表示范围，如果n无穷大呢？

一般针对大数问题，都采用通过字符串将大数表示出来



## 18. 删除单向链表中的节点

>  * 给定单向链表的头指针和一个要删除的节点的值，定义一个函数删除该节点。
>  * 返回删除后的链表的头节点。
>  * 输入: head = [4,5,1,9], val = 5
>  * 输出: [4,1,9]
>  * 解释: 给定你链表中值为 5 的第二个节点，那么在调用了你的函数之后，该链表应变为 4 -> 1 -> 9.

本题解法有两种：

1. 使用双指针分别记录被删除节点和被删除节点的前一个节点

2. 使用单指针获取被删除节点，然后通过获取下一个节点的值和指向下下一个节点来删除当前节点

   当被删除节点为尾结点时，则遍历获取删除节点的前一个节点

注意事项：删除节点为头结点，尾结点（包括既是头结点也是尾结点）中间节点。

解一：

```java
public ListNode deleteNode(ListNode head, int val) {
    // 判断头节点是否为空
    if(head==null){
        return head;
    }
    // 初始化待删除节点与其前置节点
    ListNode deletedNode = head;
    ListNode previousNode = null;
    // 判断待删除节点是否为头节点
    if(deletedNode.val==val){
        return head.next;
    }
    // 寻找待删除节点
    while(deletedNode.val!=val){
        previousNode = deletedNode;
        deletedNode = deletedNode.next;
    }
    // 通过更新指针来移除待删节点
    // 待删节点的前置节点的指针指向待删除节点的后一个节点
    previousNode.next = deletedNode.next;
    return head;
}
```

解二：

```java
public ListNode deleteNode(ListNode head, int val) {

    // 判断头节点是否为空
    if(head==null){
        return head;
    }
    // 初始化待删除节点与其前置节点
    ListNode delNode = head;

    
    // 判断待删除节点是否为头节点
    if(delNode.val==val){
        return head.next;
    }
    
    // 寻找待删除节点
    while(delNode.val!=val){
        delNode = delNode.next;
    }

    // 删除的节点是尾结点
    if (delNode.next == null){
        // 获取被删除节点的前一个节点
        ListNode preNode = head;
        while (preNode.next.val != val){
            preNode = preNode.next;
        }
        preNode.next = null;
        return head;
    }

    // 删除被删除节点
    delNode.val = delNode.next.val;
    delNode.next = delNode.next.next;

    return head;
}
```



## 19. 正则表达式匹配

请实现一个函数用来匹配包含`'. '`和`'*'`的正则表达式。模式中的字符`'.'`表示任意一个字符，而`'*'`表示它前面的字符可以出现任意次（含0次）。在本题中，匹配是指字符串的所有字符匹配整个模式。例如，字符串`"aaa"`与模式`"a.a"`和`"ab*ac*a"`匹配，但与`"aa.a"`和`"ab*a"`均不匹配。

**示例 1:**

```
输入:
s = "aa"
p = "a"
输出: false
解释: "a" 无法匹配 "aa" 整个字符串。
```

**示例 2:**

```
输入:
s = "aa"
p = "a*"
输出: true
解释: 因为 '*' 代表可以匹配零个或多个前面的那一个元素, 在这里前面的元素就是 'a'。因此，字符串 "aa" 可被视为 'a' 重复了一次。
```

**示例 3:**

```
输入:
s = "ab"
p = ".*"
输出: true
解释: ".*" 表示可匹配零个或多个（'*'）任意字符（'.'）。
```

**示例 4:**

```
输入:
s = "aab"
p = "c*a*b"
输出: true
解释: 因为 '*' 表示零个或多个，这里 'c' 为 0 个, 'a' 被重复一次。因此可以匹配字符串 "aab"。
```

**示例 5:**

```
输入:
s = "mississippi"
p = "mis*is*p*."
输出: false
```

- `s` 可能为空，且只包含从 `a-z` 的小写字母。
- `p` 可能为空，且只包含从 `a-z` 的小写字母以及字符 `.` 和 `*`，无连续的 `'*'`。

```java

class Solution {
    public boolean isMatch(String s, String p) {

        if (s == null && p == null) {
            return true;
        }
        if (s == null || p == null) {
            return false;
        }
        if (s.equals(p)) {
            return true;
        }
        return matchCore(s.toCharArray(),0, p.toCharArray(),0);

    }

    public boolean matchCore(char[] s, int i, char[] p, int j) {

        // 匹配结束
        if (i >= s.length && j >= p.length) {
            return true;
        }

        // 匹配不正确的情况下
        if (i < s.length && j >= p.length) {
            return false;
        }

        // 在模式字符串中 * 前方必有一个字符，代表这个字符可以出现0-n次
        if ((j + 1) < p.length && p[j + 1] == '*') {

            // 判断当前模式字符(包括普通字符和'.')和字符串字符是否匹配
            if (i < s.length && ( p[j] == s[i] || p[j] == '.')) {

                // 有两种方式可以进行处理
                // 1.模式向后移动两个字符,代表*前方的字符出现0次,字符串不动
                // 2.模式不动，字符串移动一个字符
                return matchCore(s, i + 1, p, j) || matchCore(s, i, p, j + 2);


            } else {
                // 不匹配的话，则模式字符索引向后走两位，而字符串索引位置不变
                return matchCore(s, i, p, j + 2);
            }

        }

        // 判断当前模式字符(包括普通字符和'.')和字符串字符是否相等
        if (i < s.length && ( p[j] == s[i] || p[j] == '.')) {

            // 这种情况下 两个字符串的字符索引都向前走一位
            return matchCore(s, i+1, p, j+1);
        } else {
    
            return false;
        }
    }
}
```

## 21. 调整数组顺序使奇数位于偶数前面

输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有奇数位于数组的前半部分，所有偶数位于数组的后半部分。

 **示例：**

```
输入：nums = [1,2,3,4]
输出：[1,3,2,4] 
注：[3,1,2,4] 也是正确的答案之一。
```

 

**提示：**

1. `1 <= nums.length <= 50000`
2. `1 <= nums[i] <= 10000`



```java
class Solution {
    public int[] exchange(int[] nums) {
        
        int left = 0;
        int right = nums.length - 1;
        while (left < right) {
            while (left < right && (nums[left] & 1) != 0) {
                left++;
            }
            while (left < right && (nums[right] & 1) == 0) {
                right--;
            }
            if (left < right) {
                int temp = nums[left];
                nums[left] = nums[right];
                nums[right] = temp;
            }
        }
        return nums;

    }
}
```

## 22. 链表中倒数第K个节点

输入一个链表，输出该链表中倒数第k个节点。为了符合大多数人的习惯，本题从1开始计数，即链表的尾节点是倒数第1个节点。例如，一个链表有6个节点，从头节点开始，它们的值依次是1、2、3、4、5、6。这个链表的倒数第3个节点是值为4的节点。

```
示例：

给定一个链表: 1->2->3->4->5, 和 k = 2.

返回链表 4->5.
```

解题思路：

1. 遍历两次链表。第一次计算出链表总长度，然后第二次遍历到 n - k + 1处，将 head = curNode

2. 使用双指针遍历一次链表，两个指针刚开始指向head，第一个指针遍历到 k处（即向后索引k-1步。

   当k = 3，向后走2步1,2,3），然后第二个指针还是指向head。两个指针同时向后移动，直到第一个指针的next为null。

```java
public ListNode getKthFromEnd(ListNode head, int k) {
    
    if(head == null || k <= 0) 
        return null;
    
    ListNode hNode = head;
    ListNode tNode = head;
    
    for(int i = 0; i < k-1; i++){
        if(tNode != null){
            tNode = tNode.next;
        }else{
            return null;
        }
    }
    
    while(tNode.next != null){
        hNode = hNode.next;
        tNode = tNode.next;
    }
    head = hNode;
    return head;
}
```



## 24. 实现单链表翻转

**算法思想**

>  1、初始化时 使用指针Front、Marker指向头结点、而Tail指针指向 Front.next
>  2、将Tail.val赋值给newNode，并将newNode插入到Front之前
>  3、插入完成后Front前移一位 指针指向newNode
>  4、Tail指针后移一位 Tail = Tail.next;
>  5、当Tail.next == null时，特殊处理

**示例:**

```
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL
```

**算法实现**

```java
public ListNode reverseList(ListNode head) {

    ListNode frontMarker = head;
    ListNode tailMarker = head;
    ListNode marker = head;

    if (frontMarker == tailMarker) {
        tailMarker = frontMarker.next;
        frontMarker.next = tailMarker;
    }

    // 会遗漏掉tailMarker.next == null的节点
    while (tailMarker.next != null) {

        ListNode newNode = new ListNode(tailMarker.val);
        newNode.next = frontMarker;
        frontMarker = newNode;
        tailMarker = tailMarker.next;
    }

    if (tailMarker.next == null) {
        ListNode newNode = new ListNode(tailMarker.val);
        newNode.next = frontMarker;
        frontMarker = newNode;
        marker.next = null;
    }

    return frontMarker;

}
```



解法二：

仅需保存三个节点——前驱节点，当前节点，和后驱节点，

使当前节点.next指向前驱，然后当前指向后驱。

当遍历到为节点时（curNode.next == null），则应该把head指向curNode

```java
public ListNode reverseList(ListNode head) {

    ListNode curNode = head;
    ListNode preNode = null;    // preNode初始为反转链表的尾结点null

    while (curNode != null) {

        ListNode nextNode = curNode.next;   // 保存当前节点的下一个节点

        // 当遍历到尾结点时，则head节点指向curNode，然后进行最后的链接操作
        if (nextNode == null) {
            head = curNode;
        }

        curNode.next = preNode;
        preNode = curNode;
        curNode = nextNode;
        
        // 或者取消上方的 if (nextNode == null),如果当前节点为空则说明是尾结点，然后将head指向前驱节点
        /*
        if (curNode == null){
               head = preNode;
        }
        */

    }

    return head;
}
```



## 25. 合并两个有序链表

输入两个递增排序的链表，合并这两个链表并使新链表中的节点仍然是递增排序的。

**示例1：**

```
输入：1->2->4, 1->3->4
输出：1->1->2->3->4->4
```

**限制：**

```
0 <= 链表长度 <= 1000
```

**递归实现**

解题思路：比较两个链表的头结点，选择最小的头节点返回。

```java
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {

    // 如果链表1为null返回链表2，如果2为null返回1，如果都为空返回null
    if (l1 == null){
        return l2;
    }else if (l2 == null){
        return l1;
    }

    // 递归实现
    if (l1.val < l2.val){
        l1.next = mergeTwoLists(l1.next,l2);
        return l1;
    }else {
        l2.next = mergeTwoLists(l1,l2.next);
        return l2;
    }
}
```

**迭代实现**

```java
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {

    ListNode head = new ListNode(8848);	// 任意设置一个头结点,用来接收一个拼接的链表
    ListNode tailNode = head;			// 指向head链表的尾结点用来接收两个链表的节点

    while (l1 != null && l2 != null) {
        if (l1.val < l2.val) {
            tailNode.next = l1;
            tailNode = tailNode.next;
            l1 = l1.next;
        } else {
            tailNode.next = l2;
            tailNode = tailNode.next;
            l2 = l2.next;
        }
    }

    if (l1 == null) {
        tailNode.next = l2;
    } else {
        tailNode.next = l1;
    }
    return head.next;
}
```



## 26. 树的子结构

输入两棵二叉树A和B，判断B是不是A的子结构。**(约定空树不是任意一个树的子结构)**

B是A的子结构， 即 A中有出现和B相同的结构和节点值。

    例如:
    给定的树 A:
         3
        / \
       4   5
      / \
     1   2
     
    给定的树 B：
    
       4 
      /
     1
    返回 true，因为 B 与 A 的一个子树拥有相同的结构和节点值。


```
示例 1：

输入：A = [1,2,3], B = [3,1]
输出：false
```

```
示例 2：

输入：A = [3,4,5,1,2], B = [4,1]
输出：true
```


限制： 0 <= 节点个数 <= 10000



解题思路：一般情况下，没有特殊要求，树的遍历都是用递归。

> 个人认为这道题可以引起对树的递归遍历的深入思考！！！ 

首先判断树A是否存在B的子结构，就需要逐个遍历A的所有节点。

当遇到与B相等的节点时，进行特殊处理。即：

```java
if(nodeA.val == nodeB.val){
	// 特殊处理就是使用另一个函数对两个树的当前节点的子节点进行判断
	return searchTheSameNode(nodeA,nodeB);
}
```

由于B树是A树的子结构，所以当两个节点有任意一个为null时，都不符合树的子结构条件

```java
if(nodeA == null && nodeB == null){
	return false;
}
```

对树A进行遍历

```java
public boolean isSubStructure(TreeNode nodeA, TreeNode nodeB) {

    boolean res = false;

    // 根据上述分析，当两个节点有一个为空时结束遍历
    if (nodeA!=null && nodeB!=null){

        // 当两个节点的值相等时，使用一个函数对两个树的子节点进行遍历对比
        if (nodeA.val == nodeB.val){
            // 如果发现存在子结构关系，则返回 true，后续的判断都不在进行
            // 否则返回false继续后面的判断
            res = searchTheSameNode(nodeA,nodeB);
        }

        // 如果两个节点不相等，则继续遍历树A的节点
        if (!res){
            res = isSubStructure(nodeA.left,nodeB);
        }
        if (!res){
            res = isSubStructure(nodeA.right,nodeB);
        }
    }
    return res;
}
```

对相同的两个节点的子节点进行遍历并判断是否具有相同的结构

```java
private boolean searchTheSameNode(TreeNode nodeA, TreeNode nodeB) {

    // 此递归遍历的结束条件
    if (nodeB == null){
        return true;
    }

    // 如果nodeB不为null而nodeA为null，这说明不存在子结构
    if (nodeA == null){
        return false;
    }

    if (nodeA.val != nodeB.val){
        return false;
    }

    // 当两个树的左右节点同时相等时返回 true
    return searchTheSameNode(nodeA.left,nodeB.left) 
        && searchTheSameNode(nodeA.right,nodeB.right);
}
```



## 27. 二叉树的镜像

请完成一个函数，输入一个二叉树，该函数输出它的镜像。

```
     4
   /  \
  2   7
 / \  / \
1   3 6  9
```


镜像输出：

```
     4
   /   \
  7     2
 / \   / \
9   6 3   1
```

**示例 1：**

```
输入：root = [4,2,7,1,3,6,9]
输出：[4,7,2,9,6,3,1]
```

**限制：0 <= 节点个数 <= 1000**



解题思路：对树的所有节点进行递归遍历，当遇到一个根节点时，如果左右孩子节点都不为空，则对其进行交换

```java
public TreeNode mirrorTree(TreeNode root) {

    if (root == null) {
        return null;
    }

    // 如果左右子树同时为空，则不存在子节点镜像
    if (root.left == null && root.right == null) {
        return root;
    }

    // 交换两个子节点，即使其中一个为空
    TreeNode tempNode = root.left;
    root.left = root.right;
    root.right = tempNode;

    // 如果左子节点不为空，则继续向下遍历
    if (root.left != null) {
        mirrorTree(root.left);
    }

    // 如果右子节点不为空，则继续向下遍历
    if (root.right != null) {
        mirrorTree(root.right);
    }
    return root;
}
```



## 28. 对称二叉树

> 给定一个二叉树，检查它是否是镜像对称的。
>
> 使用递归 思想判断： 
>
> 以根节点以及其左右子树，
>
> 左子树的左子树和右子树的右子树相同，左子树的右子树和右子树的左子树相同
>
> 以上两个条件都要符合
>
> 所以我们第一个传根节点的左子树和右子树，先判断左右子树根结点的比较。
>
> 然后分别对左子树的左子树和右子树的右子树，并对左子树的右子树和右子树的左子树进行判断。
>
> 只有两个条件都满足则返回的是true，一层一层递归进入，则可以得到结果。

例如，二叉树 `[1,2,2,3,4,4,3]` 是对称的。

```
    1
   / \
  2   2
 / \ / \
3  4 4  3
```

 但是下面这个 `[1,2,2,null,3,null,3]` 则不是镜像对称的:

```
    1
   / \
  2   2
   \   \
   3    3
```



```java
class Solution {
    public boolean isSymmetric(TreeNode root) {
        
        return judSymmetric(root, root);
    }

    private boolean judSymmetric(TreeNode leftNode, TreeNode rightNode) {

        if (leftNode == null && rightNode == null) {      // 若左节点为空，右节点也为空
            return true;
        }

        if (rightNode == null || leftNode ==null) {
            return false;           // 左节点不为空，右节点为空
        }

        if (leftNode.val != rightNode.val) {
            return false;
        }
        return 
            judSymmetric(leftNode.left, rightNode.right) 
            && 
            judSymmetric(leftNode.right, rightNode.left);
    }
}
```



## 29. 顺时针打印数组（打印螺旋数组）


输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字。

**示例 1：**

```
输入：matrix = [[1,2,3],[4,5,6],[7,8,9]]
输出：[1,2,3,6,9,8,7,4,5]
```

**示例 2：**

```
输入：matrix = [[1,2,3,4],[5,6,7,8],[9,10,11,12]]
输出：[1,2,3,4,8,12,11,10,9,5,6,7]
```

**限制：**

- `0 <= matrix.length <= 100`
- `0 <= matrix[i].length <= 100`



```java
public int[] spiralOrder(int[][] matrix) {

    if (matrix == null || matrix.length == 0) {
        return new int[0];
    }

    int rows = matrix.length;
    int cols = matrix[0].length;

    int[] arr = new int[rows * cols];

    // 不管是几乘几的矩阵，每一圈对应的左上角的坐标符合：总列数>每一列*2&&总行数>每一行*2
    int start = 0;
    int count = 0;

    /**
         * 保存每一圈的数值
         */
    while (cols > start * 2 && rows > start * 2) {

        // 根据cols rows start确定每圈的末尾行和列
        // cols - 1 是因为 数组下标是从0开始，假如cols = 5，则下标为用 0-4保存数据，rows同理
        int endCol = cols - 1 - start;
        int endRow = rows - 1 - start;

        // 从左到右保存
        for (int i = start; i <= endCol; i++){
            arr[count++] = matrix[start][i];
        }

        // 从上至下保存:考虑到二维数组只有一行的情况下，从上至下需要判断是否越界
        if (start < endRow){
            for (int i = start + 1; i <= endRow; i++){
                arr[count++] = matrix[i][endCol];
            }
        }

        // 从右至左保存，考虑到只有一列和一行的情况下：因此需要对列和行进行约束
        if (start < endCol && start < endRow){
            for (int i = endCol-1; i >= start; i--){
                arr[count++] = matrix[endRow][i];
            }
        }

        // 从下至上保存,考虑到只有一列和两行的情况，因为从上之下，从左至右已经把所有的数据保存完毕了
        if (start < endCol && start < endRow - 1){
            for (int i = endRow -1; i >= start + 1; i--){
                arr[count++] = matrix[i][start];
            }
        }

        start++; // start++ 进行下一圈数据的遍历
    }
    return arr;
}
```



## 30. 获取栈中最小的元素

设计一个支持 push ，pop ，top 操作，并能在常数时间内检索到最小元素的栈。

push(x) —— 将元素 x 推入栈中。
pop() —— 删除栈顶的元素。
top() —— 获取栈顶元素。
getMin() —— 检索栈中的最小元素。

```
示例:

输入：
["MinStack","push","push","push","getMin","pop","top","getMin"]
[[],[-2],[0],[-3],[],[],[],[]]

输出：
[null,null,null,null,-3,null,0,-2]

解释：
MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
minStack.getMin();   --> 返回 -3.
minStack.pop();
minStack.top();      --> 返回 0.
minStack.getMin();   --> 返回 -2.
```

`提示：`

pop、top 和 getMin 操作总是在 非空栈 上调用。



解题思路：

1. 使用LinkedList模拟两个栈的数据结构，stack用于存储正常的数据，minStack用于存储stack中最小的元素

   PS: 要考虑空栈的情况

   ```java
   stack					minStack
   			push(x)
   3						3
   3,2						3,2
   3,2,4					3,2,2
   3,2,4,1					3,2,2,1
   			pop()
   3,2,4					3,2,2
   			getMin()
   3,2,4					3,2,2 //在min栈中获取最小值2
   			pop()
   3,2						3,2
   			top()
   3,2 // 获取栈顶元素2      3,2
   ```

2. 自定义链表节点，每个节点存储当前入栈值和min值以及下一个节点的指针

   ```java
   private class Node{
   	int val;
   	int minVal;
   	Node next;
   	
   	public Node(int val, int minVal){
   		this.val = val;
           this.minVal = minVal;
           this.next = null;
   	}
   	
   	public Node(int val, int minVal, Node node){
   		this.val = val;
           this.minVal = minVal;
           this.next = node;
   	}
   }
   
   // 数据结构表现为
   
   // push() 操作，使用头插法
   head 
   1,1 -> 4,2 -> 2,2 -> 3,3
       
   // pop()操作
   	   head 
   1,1 -> 4,2 -> 2,2 -> 3,3
   
   // getMin()
   return head.minVal  // 2
   
   // pop()
    	    	  head
   1,1 -> 4,2 -> 2,2 -> 3,3
       
   // top()
   return head.val // 2
   ```

**使用解法一：**

```java
class MinStack {

    
    private Deque<Integer> stack;
    private Deque<Integer> minStack;

    /** initialize your data structure here. */
    public MinStack() {
        stack = new LinkedList<>();
        minStack = new LinkedList<>();
    }

    /**
     * 入栈
     */
    public void push(int x) {
        
        if (stack.isEmpty()){
            stack.push(x);
            minStack.push(x);
        }else {
            stack.push(x);
            minStack.push(Math.min(minStack.peek(),x));
        }
    }

    /**
     * 删除栈顶元素
     */
    public void pop() {
        if (!stack.isEmpty() && !minStack.isEmpty()){
            stack.pop();
            minStack.pop();
        }
    }

    /**
     * 获取栈顶元素
     */
    public Integer top() {
        if (stack.isEmpty()) return null;
        return stack.peek();
    }

    /**
     * 获取栈中最小值
     */
    public Integer getMin() {
        if (minStack.isEmpty()) return null;
        return minStack.peek();
    }
}
```

**使用解法二：**

```java
public class MinStack {

    private Node head;

    /** initialize your data structure here. */
    public MinStack() {
    }

    /**
     * 入栈
     */
    public void push(int x) {
        if (head == null){
            head = new Node(x,x,null);
        }else {
            // 使用头插法进行连接，形成栈的结构
            head = new Node(x,Math.min(x,head.minVal),head);
        }
    }

    /**
     * 删除栈顶元素
     */
    public void pop() {
        head = head.next;
    }

    /**
     * 获取栈顶元素
     */
    public int top() {
        return head.val;
    }

    /**
     * 获取栈中最小值
     */
    public int getMin() {
        return head.minVal;
    }

    private class Node{
        int val;
        int minVal;
        Node next;

        public Node(int val, int minVal){
            this.val = val;
            this.minVal = minVal;
            this.next = null;
        }

        public Node(int val, int minVal, Node node){
            this.val = val;
            this.minVal = minVal;
            this.next = node;
        }
    }
}
```



## 31. 栈的压入、弹出序列

输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如，序列 {1,2,3,4,5} 是某栈的压栈序列，序列 {4,5,3,2,1} 是该压栈序列对应的一个弹出序列，但 {4,3,5,1,2} 就不可能是该压栈序列的弹出序列。

 

**示例 1：**

```
输入：pushed = [1,2,3,4,5], popped = [4,5,3,2,1]
输出：true
解释：我们可以按以下顺序执行：
push(1), push(2), push(3), push(4), pop() -> 4,
push(5), pop() -> 5, pop() -> 3, pop() -> 2, pop() -> 1
```

**示例 2：**

```
输入：pushed = [1,2,3,4,5], popped = [4,3,5,1,2]
输出：false
解释：1 不能在 2 之前弹出。 
```

**提示：**

1. `0 <= pushed.length == popped.length <= 1000`
2. `0 <= pushed[i], popped[i] < 1000`
3. `pushed` 是 `popped` 的排列。

**解题思路：**popped数组是出栈的序列，那么可以使用一个辅助栈来存储popped数组的逆序序列。以示例1为例：

```java
auxStack = [1,2,3,5,4]
```

此时`stack`对`pushed`进行入栈，当两个栈的栈顶元素相同时，两个栈同时出栈

```java
// push
stack = [1,2,3,4]

// pop
stack = [1,2,3]
auxStack = [1,2,3,5]

// push
stack = [1,2,3,5]

// pop
stack = [1,2,3]
auxStack = [1,2,3]

// 依次类推，如果两个栈的入栈和出栈序列匹配时，最终两个栈都会全部出栈
```

```java
class Solution {
    public boolean validateStackSequences(int[] pushed, int[] popped) {

        Deque<Integer> stack = new ArrayDeque<Integer>();
        Deque<Integer> auxStack = new ArrayDeque<Integer>();

        // 辅助栈用来存储出栈序列的逆序列
        for (int i = popped.length - 1; i >= 0; i--){
            auxStack.push(popped[i]);
        }

        // 将pushed压栈进入stack中，如果stack栈顶元素和auxStack栈顶元素相同
        // 则两个栈同时出栈，最终两个栈都为空
        for (int i = 0; i < pushed.length; i++){
            stack.push(pushed[i]);

            while (!stack.isEmpty()&&stack.peek().equals(auxStack.peek())){
                stack.pop();
                auxStack.pop();
            }
        }
        return stack.isEmpty();
    }
}
```



## 32-I. 从上到下打印二叉树 

从上到下打印出二叉树的每个节点，同一层的节点按照从左到右的顺序打印。

例如:
给定二叉树: `[3,9,20,null,null,15,7]`,

```
    3
   / \
  9  20
    /  \
   15   7
```

返回：

```
[3,9,20,15,7]
```

**提示：**

1. `节点总数 <= 1000`

解题思路：使用队列存储每一层的节点，然后逐个出队，将出队节点的左右子节点入队。直到子节点为空，出队队列为空。

```java
public int[] levelOrder(TreeNode root) {
    if (root == null) {
        return new int[0];
    }

    Deque<TreeNode> queue = new LinkedList<>();
    ArrayList<Integer> list = new ArrayList<>();
    queue.add(root);

    while (!queue.isEmpty()) {

        TreeNode node = queue.poll();
        list.add(node.val);

        // 如果当前出队节点存在左孩子节点，则将其入队
        if (node.left != null) {
            queue.add(node.left);
        }

        // 如果当前出队节点存在左孩子节点，则将其入队
        if (node.right != null) {
            queue.add(node.right);
        }
    }
    // ArrayList转为int数组:此种转换方式效率很低，既耗时又耗内存，耗时5 ms
    // return list.stream().mapToInt(Integer::intValue).toArray();

    int[] res = new int[list.size()];
    int count = 0;
    for (int num : list) {
        res[count++] = num;
    }
    // 执行用时:耗时1 ms，击败99.77%，内存消耗：击败60.06%
    return res;
}
```



## 32-II. 从上到下打印二叉树

**示例：**
二叉树：`[3,9,20,null,null,15,7]`,

```
    3
   / \
  9  20
    /  \
   15   7
```

返回其层次遍历结果：

```
[
  [3],
  [9,20],
  [15,7]
]
```

**解：**

```java
public List<List<Integer>> levelOrder(TreeNode root) {

	List<List<Integer>> result = new ArrayList<>();

    if (root == null) {
        return result;
    }
    LinkedList<TreeNode> queue = new LinkedList<>();
    queue.add(root);
    while (!queue.isEmpty()) {

        ArrayList<Integer> list = new ArrayList<>();
        int queueSize = queue.size(); // 获取当前层元素个数（即队列大小） 

        // 将当前层节点逐个出队，并将出队节点的子节点入队
        for (int i = 0; i < queueSize; i++) {
            
            TreeNode node = queue.poll();
            list.add(node.val);
            // 如果当前节点存在左右子节点，则子节点入队
            if (node.left != null) {
                queue.add(node.left); 
            }
            if (node.right != null) {
                queue.add(node.right);
            }
        }
        result.add(list);
    }
    return result;
}
```



## 32-III. 从上到下打印二叉树 

请实现一个函数按照**之字形顺序打印二叉树**，即第一行按照从左到右的顺序打印，第二层按照从右到左的顺序打印，第三行再按照从左到右的顺序打印，其他行以此类推。

例如:
给定二叉树: `[1,2,3,4,5,6,7]`,

```
           1
        /     \
       2        3
      /  \     /  \
     4    5   6    7
```

返回其层次遍历结果：

```
[
  [1],
  [3,2],
  [4,5,6,7]
]
```

**提示：**

1. `节点总数 <= 1000`

解题思路：与1,2题不同，经过模拟发现，奇数层打印的顺序是从左到右（入栈则需从右至左），偶数层打印的顺序是从右到左（入栈则需从左至右）

那么就可以使用两个双向队列，按照一定规则存储奇数层和偶数层节点。

**规则：**

奇数层入栈规则：先右节点入，后左节点入。

偶数层入栈规则：先左节点入，后右节点入。

**PS：用双向队列也可以模拟**

```java
class Solution {
      public List<List<Integer>> levelOrder(TreeNode root) {
        
        List<List<Integer>> result = new ArrayList<>();

        if (root == null) {
            return result;
        }

        // 定义 两个栈，分别保存树奇数层和偶数层节点
        Deque<TreeNode> oddStack = new LinkedList<>();  // 奇数层栈
        Deque<TreeNode> evenStack = new LinkedList<>(); // 偶数层栈
        oddStack.push(root);
        int layer = 1;

        while (!oddStack.isEmpty() || !evenStack.isEmpty()) {

            List<Integer> list = new ArrayList<>();

            int oddStackSize = oddStack.size();
            int evenStackSize = evenStack.size();
            // 判断当前层为奇数层
            if ((layer & 1) == 1) {
                for (int i = 0; i < oddStackSize; i++) {
                    TreeNode node = oddStack.pop();
                    list.add(node.val);
                    //奇数层的下一层为偶数层，偶数层入栈规则：先左节点入栈，后右节点入栈
                    if (node.left != null) {
                        evenStack.push(node.left);
                    }
                    if (node.right != null) {
                        evenStack.push(node.right);
                    }
                }
            }
            // 当前为偶数层
            if ((layer & 1) == 0) {
                for (int i = 0; i < evenStackSize; i++) {
                    TreeNode node = evenStack.pop();
                    list.add(node.val);
                    // 偶数层的下一层是奇数层，奇数层的入栈规则：先右节点入栈，后左节点入栈
                    if (node.right != null) {
                        oddStack.push(node.right);
                    }
                    if (node.left != null) {
                        oddStack.push(node.left);
                    }
                }
            }
            ++layer;
            if (!list.isEmpty()) result.add(list);
        }
        return result;
    }
}
```

## 33. 二叉搜索树的后序遍历序列

输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历结果。如果是则返回 `true`，否则返回 `false`。假设输入的数组的任意两个数字都互不相同。 

参考以下这颗二叉搜索树：

```
     5
    / \
   2   6
  / \
 1   3
```

**示例 1：**

```
输入: [1,6,3,2,5]
输出: false
```

**示例 2：**

```
输入: [1,3,2,6,5]
输出: true 
```

**提示：**

1. `数组长度 <= 1000`

解题思路：排序二叉树中，经过后序遍历后，最后一个必是root根节点。

root左子树的值均小于rootVal，而右子树的值均大于rootVal。

以根节点为划分，依照这个规则递归遍历整个二叉树

```
[ 1,   3,   2,   6,   5]
第一次划分: root = 5, 左子树的root = 2, 右子树的root = 6
```



```java
class Solution {
    // 要点：二叉搜索树中根节点的值大于左子树中的任何一个节点的值，小于右子树中任何一个节点的值，子树也是
    public boolean verifyPostorder(int[] postorder) {
        if (postorder.length < 2) return true;
        return verify(postorder, 0, postorder.length - 1); 
    }

    // 递归实现
    private boolean verify(int[] postorder, int left, int right){
        if (left >= right) return true; // 当前区域不合法的时候直接返回true就好

        int rootValue = postorder[right]; // 当前树的根节点的值

        int k = left;
        while (k < right && postorder[k] < rootValue){ // 从当前区域找到第一个大于根节点的，说明后续区域数值都在右子树中
            k++;
        }

        for (int i = k; i < right; i++){ // 进行判断后续的区域是否所有的值都是大于当前的根节点，如果出现小于的值就直接返回false
            if (postorder[i] < rootValue) return false;
        }

        // 当前树没问题就检查左右子树
        if (!verify(postorder, left, k - 1)) return false; // 检查左子树

        if (!verify(postorder, k, right - 1)) return false; // 检查右子树

        return true; // 最终都没问题就返回true
    }
}
```

## 34-I. 二叉树路径总和 

**示例:** 
给定如下二叉树，以及目标和 `sum = 22`，

```
                              5			
                             / \
                            4   8		(左)sum = sum - 5 = 17
                           /   / \
                          11  13  4		(左)sum	= sum - 4 = 13
                         /  \      \
                        7    2      1	(左)sum = sum - 11 = 2 
```

返回 `true`, 因为存在目标和为 22 的根节点到叶子节点的路径 `5->4->11->2`。

> 使用递归，分别遍历左右子树，当左右子树中有一个路径满足条件，则返回true
>
> 递归结束条件：root.left == null && root.right == null && sum == root.val
>
> sum的值自顶向下 逐层递减 sum = sum - root.val。

```java
class Solution {
    public boolean hasPathSum(TreeNode root, int sum) {
       
        // 当前节点为空，sum 大于等于 0；例如：root = []，sum = 0、sum = 1
        if (root == null && sum >= 0) {
            return false;
        }

        // 当前节点为空，sum 小于 0；例如：root = []，sum = -1
        if (root == null && sum < 0) {
            return false;
        }

        // 当且仅当 左右孩子节点为空 且 sum = root.val 时，则找到目标
        if (root.left == null && root.right == null && sum == root.val) {
            return true;
        }

        // 遍历目标节点的左子树和右子树 ，当左右子树中有一个返回true(即找到目标) 时，则返回true；
        // sum = sum - root.val,当到达叶子节点且sum= root.val则 找到目标
        return hasPathSum(root.left, sum - root.val) || hasPathSum(root.right, sum - root.val);
    }
}
```

## 34-II. 二叉树中和为某一值的路径 

输入一棵二叉树和一个整数，打印出二叉树中节点值的和为输入整数的所有路径。从树的根节点开始往下一直到叶节点所经过的节点形成一条路径。

**示例:**
给定如下二叉树，以及目标和 `sum = 22`，

```
              5
             / \
            4   8
           /   / \
          11  13  4
         /  \    / \
        7    2  5   1
```

返回:

```
[
   [5,4,11,2],
   [5,8,4,5]
]
```

**提示：**

1. `节点总数 <= 10000`

解题思路：从根节点开始递归遍历，每遍历一个节点，则把当前节点入栈。

同时判断 sum == 当前节点.val（每遍历一个节点时，其sum = sum - 父节点.val）。

如果到达叶子节点且sum = 当前节点.val，则说明找到了正确的路径，就将栈内的所有元素保存到链表中。

每次返回父节点前都要将栈顶的节点删除再返回父节点。

```java
class Solution {
    List<List<Integer>> resList = new ArrayList<>();
    LinkedList<TreeNode> stack = new LinkedList<>();

    public List<List<Integer>> pathSum(TreeNode root, int sum) {
        if (root == null) {
            return new ArrayList<>();
        }

        findPath(root, sum);

        return resList;
    }

    public void findPath(TreeNode root, int currentSum) {

        // 将当前根节点入栈
        stack.push(root);

        // 判断节点状态及sum数值,找到合适的路径就将栈内元素保存到链表中
        if ((root.left == null && root.right == null) && currentSum == root.val) {

            ArrayList<Integer> list = new ArrayList<>();

            for (int i = stack.size() - 1; i >= 0; i--) {
                TreeNode node = stack.get(i);
                list.add(node.val);
            }
            resList.add(list);
        }

        // 如果不是叶子节点，则继续遍历其子节点
        if (root.left != null) {
            findPath(root.left, currentSum - root.val);
        }

        if (root.right != null) {
            findPath(root.right, currentSum - root.val);
        }

        // 递归返回其父节点前，将栈内路径上的当前节点删除
        stack.pop();
    }
}
```



## 35. 复制复杂链表

请实现 copyRandomList 函数，复制一个复杂链表。在复杂链表中，每个节点除了有一个 next 指针指向下一个节点，还有一个 random 指针指向链表中的任意节点或者 null。

示例 1：

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/01/09/e1.png)输入：head = [[7,null],[13,0],[11,4],[10,2],[1,0]]
输出：[[7,null],[13,0],[11,4],[10,2],[1,0]]

示例 2：

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/01/09/e2.png)输入：head = [[1,1],[2,1]]
输出：[[1,1],[2,1]]

示例 3：

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/01/09/e3.png)输入：head = [[3,null],[3,0],[3,null]]
输出：[[3,null],[3,0],[3,null]]

示例 4：
输入：head = []
输出：[]

解释：给定的链表为空（空指针），因此返回 null。

`提示：`

- 10000 <= Node.val <= 10000
- Node.random 为空（null）或指向链表中的节点。
- 节点数目不超过 1000 。

解题思路：将复制复杂链表的流程划分为三段。

1. 将原链表复制为：S->s-N->n->Q->q（大写节点的为原链表节点，小写节点为复制节点）
2. 根据原链表节点的random指针设置克隆节点的random指针
3. 将链表 S->s->N->n->Q->q拆分为S->N->Q和 s->n->q，并返回克隆的链表

```java
/*
// Definition for a Node.
class Node {
    int val;
    Node next;
    Node random;

    public Node(int val) {
        this.val = val;
        this.next = null;
        this.random = null;
    }
}
*/
class Solution {
    /**
     * 复制复杂链表
     * <p>
     * 请实现 copyRandomList 函数，复制一个复杂链表。在复杂链表中，每个节点除了有一个 next 指针指向下一个节点，还有一个 random 指针指向链表中的任意节点或者 null。
     */
    public Node copyRandomList(Node head) {
        clonedNodes(head);
        connectExtNodes(head);
        return reconnectNodes(head);
    }

    /**
     * 复制节点
     * <p>
     * 将链表复制为 S->s-N->n->Q->q
     * 其中大写节点的为原链表节点，小写节点为复制节点
     */
    public void clonedNodes(Node head) {
        Node node = head;
        while (node != null) {
            Node clonedNode = new Node(node.val);
            clonedNode.next =node.next;
            node.next = clonedNode;
            node = clonedNode.next;
        }
    }

    /**
     * 连接当前节点 除 next指针外的另一个random指针。
     */
    public void connectExtNodes(Node head){
        Node node = head;
        while (node != null){
            Node clonedNode = node.next;
            if (node.random != null){
                clonedNode.random = node.random.next;
            }
            node = clonedNode.next;
        }
    }

    /**
     * 将原链表拆分为两个链表
     * 将 S->s->N->n->Q->q拆分为S->N->Q和 s->n->q
     * 上述省略了每个节点的额外指针
     */
    public Node reconnectNodes(Node head){

        if (head == null){
            return null;
        }

        Node node = head;
        Node clonedNode = node.next;    // 指向第一个克隆节点：s
        Node clonedHead = clonedNode;   // 设置克隆链表的头结点：clonedHead指向s

        node.next = clonedNode.next;    // 将原链表node.next指向克隆节点的下一个节点：S->N
        node = clonedNode.next;         // node指向克隆节点的下一个节点：node指向N

        while (node != null){
            clonedNode.next = node.next;    // s -> n
            clonedNode = node.next;         // cloneNode 指向 n
            node.next = clonedNode.next;    // N -> Q
            node = clonedNode.next;         // node指向Q
        }
        return clonedHead;
    }
}
```



## 39. 数组中出现次数超过一半的数字

给定一个大小为 *n* 的数组，找到其中的多数元素。多数元素是指在数组中出现次数**大于** `⌊ n/2 ⌋` 的元素。

你可以假设数组是非空的，并且给定的数组总是存在多数元素。


**示例** 1:

```
输入: [3,2,3]
输出: 3
```

**示例 2:**

```
输入: [2,2,1,1,1,2,2]
输出: 2
```

解题思路：可以对数组使用快速排序，如果输入的数组合法，则数组中间索引位置为返回值

还可以用下方思路进行查找，该算法需要细品：

```java
class Solution {
    boolean inputInvalid = false;   // 全局变量用于判断输入是否无效，并返回0
    public int majorityElement(int[] nums) {

        if (nums == null || nums.length == 0){
            inputInvalid = true;
            return 0;
        }

        int result = nums[0];
        int times = 1;
        for (int i = 1; i < nums.length; i++) {
            if (times == 0) {
                result = nums[i];
                times = 1;
            } else if (result == nums[i]) {
                times++;
            } else {
                times--;
            }
        }
        
        // 判断数组中是否有一个数字出现的次数超过数组长度的一半，没有则返回0
        int count = 0;
        for (int i = 0; i < nums.length; i++){
            if (nums[i] == result){
                count++;
            }
        }
        if (count * 2 <= nums.length){
            inputInvalid = true;
            return 0;
        }
        return result;
    }
}
```



## 40. 最小的K个数

入整数数组 arr ，找出其中最小的 k 个数。例如，输入4、5、1、6、2、7、3、8这8个数字，则最小的4个数字是1、2、3、4

示例 1：

```
输入：arr = [3,2,1], k = 2
输出：[1,2] 或者 [2,1]
```

示例 2：

```
输入：arr = [0,1,2,1], k = 1
输出：[0]
```

限制：

```
0 <= k <= arr.length <= 10000
0 <= arr[i] <= 10000
```

解题思路：

解法一：可以构造一个包含K个元素大顶堆，先将数组中前K个元素保存到堆中，然后将数组剩余的元素与堆顶元素进行比较。如果小于堆顶元素则进行出堆与入堆操作。最后返回所有的堆内元素。

解法二：可以对数组进行快排，选择前k个数……

采用堆的结构可以处理海量数据，时间复杂度`O（logN）`

```java
class Solution {
    public int[] getLeastNumbers(int[] arr, int k) {
        if ((arr == null && arr.length == 0)){
            return arr;
        }

        if (k <= 0 || k > arr.length){
            return new int[0];
        }

        // 使用大顶堆保存k个最小的元素
        Queue<Integer> heap = new PriorityQueue<>((o1, o2) -> o2 - o1);
        for (int num: arr) {
            if (heap.size() < k) {
                heap.add(num);
            } else if (num < heap.peek()) {
                heap.poll();
                heap.add(num);
            }
        }
        int[] res = new int[k];
        int count = 0;
        for (int num : heap) {
            res[count++] = num;
        }
        return res;
    }
}
```



## 41. 数据流中的中位数

如何得到一个数据流中的中位数？如果从数据流中读出奇数个数值，那么中位数就是所有数值排序之后位于中间的数值。如果从数据流中读出偶数个数值，那么中位数就是所有数值排序之后中间两个数的平均值。

例如，

```
[2,3,4] 的中位数是 3

[2,3] 的中位数是 (2 + 3) / 2 = 2.5
```

设计一个支持以下两种操作的数据结构：

void addNum(int num) - 从数据流中添加一个整数到数据结构中。
double findMedian() - 返回目前所有元素的中位数



解题思路：

假设数据中的数据个数为奇数，则中位数大于左侧的所有数据，小于右侧的所有数据。

若数据中的数据个数为偶数，则中间的两个数分别大于左侧的所有数据和小于右侧的所有数据。

因此可以把数据源中的数据放入两个堆中，大顶堆存储中位数左侧的数据，小顶堆，存储中位数右侧的数据，这样最靠近中位数的数据将会位于堆顶。

**唯一要注意的点就是两个堆的大小差不能超过1，**因此，当是奇数个数据时，直接取某个堆的堆顶元素即可。若是偶数个数据时，取两个堆顶数据的平均值即可。

```java
public class MedianFinder {

    PriorityQueue<Integer> maxHeap;  // 构造大顶堆，存储中位数左侧的数据
    PriorityQueue<Integer> minHeap;  // 构造小顶堆，存储中位数右侧的数据

    /**
     * initialize your data structure here.
     */
    public MedianFinder() {
        maxHeap = new PriorityQueue<>((o1, o2) -> o2 - o1);
        minHeap = new PriorityQueue<>();
    }

    /**
     * 从数据流中添加一个整数到数据结构中
     */
    public void addNum(int num) {

        // 大顶堆中存储的始终是<中位数的数据
        maxHeap.add(num);

        // 小顶堆中存储的始终是>中位数的数据
        minHeap.add(maxHeap.poll());

        // 平衡两个堆中的元素个数差不超过 1
        if (maxHeap.size() + 1 < minHeap.size()) {
            maxHeap.add(minHeap.poll());
        }
    }

    /**
     * 返回目前所有元素的中位数
     */
    public double findMedian() {

        // 如果数据个数为奇数，则中位数存储在 小顶堆中
        if (maxHeap.size() < minHeap.size()) return minHeap.peek();
        return (double) (maxHeap.peek() + minHeap.peek()) / 2;
    }
}
```



## 42. 连续子数组的最大和

输入一个整型数组，数组中的一个或连续多个整数组成一个子数组。求所有子数组的和的最大值。

要求时间复杂度为O(n)。

**示例1:**

```
输入: nums = [-2,1,-3,4,-1,2,1,-5,4]
输出: 6
解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。
```

**提示：**

- `1 <= arr.length <= 10^5`
- `-100 <= arr[i] <= 100`

```java
class Solution {
    public int maxSubArray(int[] nums) {

        int maxSum = nums[0]; 
        int sum = 0;
        for (int i = 0; i < nums.length; i++){
            
            // 如果sum值小于0，则说明连续子数组中存在大量负数，应当抛弃。从当前元素开始计算求和
            if (sum <= 0){
                sum = nums[i];
            }else {
                sum += nums[i];
            }
            
            // 记录最大和
            if (sum > maxSum){
                maxSum = sum;
            }
        }
        return maxSum;
    }
}
```

## 43. 1～n整数中1出现的次数

输入一个整数 `n` ，求1～n这n个整数的十进制表示中1出现的次数。

例如，输入12，1～12这些整数中包含1 的数字有1、10、11和12，1一共出现了5次。

**示例 1：**

```
输入：n = 12
输出：5
```

**示例 2：**

```
输入：n = 13
输出：6
```

**限制：**

- `1 <= n < 2^31`

[解题思路](https://leetcode-cn.com/problems/1nzheng-shu-zhong-1chu-xian-de-ci-shu-lcof/solution/mian-shi-ti-43-1n-zheng-shu-zhong-1-chu-xian-de-2/)

参考以下进行理解：

```
case 1: cur=0
 n =  2  3   0  4
     千位和百位可以选00 01 02....22  十位可以取到1( 形如[00|01..|22]1[0-9] 都是<2304 ) 个位可以选0-9  共有 23 * 10 中排列
     当千位和百位取23,如果十位取1 那就是形如 231[0-9] > 2304,所以当千位和百位取23，十位只能能取0，个位取0-4即 2300 2301 2302 2303 2304
     但是2301不应该算进来，这个1是 单独  出现在个位的（而11，121,111这种可以被算多次）
     即 23*10
case 2: cur=1
 n =  2  3  1  4
   千位和百位可以选00 01 02....22  十位可以取到1 个位可以选0-9  共有 23 * 10 中排列
   当千位和百位取23,十位取1，个位可以去0-4 即 2310-2314共5个
   即 23 *10 + 4 +1
case 3: cur>1 即2-9
 n =  2  3  2  4
   千位和百位可以选00 01 02....22  十位可以取到1(形如 [00|01...|22]1[0-9] 都是<2324) 个位可以选0-9  共有 23 * 10 中排列
   当千位和百位取23,十位取1，个位可以去0-9 即 2310-2319共10个 （其中2311，被计算了两次，分别是从个位和十位分析得到的1次）
   即 23 *10 + 10
```

大致理解为：对于一个整数 n，计算 1 在每一个位置（个、十、百、千……）出现的次数，并将所有的次数相加。

![image-20200907104406749](http://codeduck.top/md/images/image-20200907104406749.png)

```java
class Solution {
    public static int countDigitOne(int n) {
        
        // 当输入的整数为负数时，返回0
        if(n < 0){
            return 0;
        }

        int digit = 1; // 记录位数
        int count = 0; // 记录1出现的次数
        int high = n / 10; // 记录高位
        int low = 0;   // 记录低位
        int cur = n % 10;      // 记录当前位的数值

        // 终止条件是高位和当前位的数值都为0
        while (high != 0 || cur != 0) {
            if (cur == 0) {
                count += high * digit;
            } else if (cur == 1) {
                count += high * digit + low + 1;
            } else {
                count += high * digit + digit;
            }
            low += cur * digit;
            cur = high % 10;
            high = high / 10;
            digit = digit * 10;
        }
        return count;
    }
}
```

## 45. 把数组排成最小的数

输入一个非负整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。

 **示例 1:**

```
输入: [10,2]
输出: "102"
```

**示例 2:**

```
输入: [3,30,34,5,9]
输出: "3033459"
```

 **提示:**

- `0 < nums.length <= 100`

**说明:**

- 输出结果可能非常大，所以你需要返回一个字符串而不是整数
- 拼接起来的数字可能会有前导 0，最后结果不需要去掉前导 0

[解题思路](https://leetcode-cn.com/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/solution/mian-shi-ti-45-ba-shu-zu-pai-cheng-zui-xiao-de-s-4/)

```java
public class Offer45 {
    // 数组内拼接的数值可能会发生数组越界，因此，返回值为String类型
    public String minNumber(int[] nums) {

        // 使用String数组记录int数组中的元素，容易进行数字之间的拼接比较
        String[] arr = new String[nums.length];
        for (int i = 0; i < nums.length; i++) {
            arr[i] = String.valueOf(nums[i]);
        }

        // 对String数组内的元素使用快排进行排序
        quickSort(arr, 0, arr.length - 1);
        StringBuilder rs = new StringBuilder();
        for (String s : arr) {
            rs.append(s);
        }
        return rs.toString();
    }

    private void quickSort(String[] arr, int left, int right) {
        if (left >= right) return;

        int i = left, j = right;
        String baseElement = arr[i];

        while (i < j) {
            // 1. if x >= y then x.compareTo(y) >= 0
            // 2. if x + y > y + x then x > y
            // 3. if x + y < y + x then x < y
            while ((arr[j] + arr[left]).compareTo(arr[left] + arr[j]) >= 0 && i < j) j--;
            while ((arr[i] + arr[left]).compareTo(arr[left] + arr[i]) <= 0 && i < j) i++;

            if (i < j) {
                String temp = arr[i];
                arr[i] = arr[j];
                arr[j] = temp;
            }
        }

        arr[left] = arr[i];
        arr[i] = baseElement;

        quickSort(arr, left, i-1);
        quickSort(arr, i+1, right);
    }
}
```



## 46. 把数字翻译成字符串

给定一个数字，我们按照如下规则把它翻译为字符串：0 翻译成 “a” ，1 翻译成 “b”，……，11 翻译成 “l”，……，25 翻译成 “z”。一个数字可能有多个翻译。请编程实现一个函数，用来计算一个数字有多少种不同的翻译方法。

**示例 1:**

```
输入: 12258
输出: 5
解释: 12258有5种不同的翻译，分别是"bccfi", "bwfi", "bczi", "mcfi"和"mzi"
```

**提示：**

- `0 <= num < 2^31`

[解题思路](https://leetcode-cn.com/problems/ba-shu-zi-fan-yi-cheng-zi-fu-chuan-lcof/solution/mian-shi-ti-46-ba-shu-zi-fan-yi-cheng-zi-fu-chua-6/)

```java
class Solution {
    public int translateNum(int num) {
        String s = String.valueOf(num);

        int a = 1, b = 1;

        for (int i = 2; i <= s.length(); i++) {
            String tem = s.substring(i - 2, i);
            int c = tem.compareTo("10") >= 0 && tem.compareTo("25") <= 0 ? a + b : a;
            b = a;
            a = c;
        }
        return a;
    }
}
```

## 48. 最长不含重复字符的子字符串

请从字符串中找出一个最长的不包含重复字符的子字符串，计算该最长子字符串的长度。

**示例 1:**

```
输入: "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```

**示例 2:**

```
输入: "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
```

**示例 3:**

```
输入: "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
     请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。
```

 提示：

- `s.length <= 40000`

[解题思路](https://leetcode-cn.com/problems/zui-chang-bu-han-zhong-fu-zi-fu-de-zi-zi-fu-chuan-lcof/solution/mian-shi-ti-48-zui-chang-bu-han-zhong-fu-zi-fu-d-9/)：本题使用双指针索引技巧+HashMap对时间复杂度进行优化

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        Map<Character, Integer> map = new HashMap<>();

        // 初始化变量
        int i = -1; // 指向左索引字符
        int j = 0; // 指向右索引字符
        int maxLen = 0; // 记录最长字符串长度

        for (; j < s.length(); j++) {

            if (map.containsKey(s.charAt(j)))
                // 如果包含相同的字符，则更新左索引的值
                i = Math.max(i, map.get(s.charAt(j)));

            // map中不包含当前索引字符，则记录当前字符信息，并计算自字符串长度
            map.put(s.charAt(j), j);
            maxLen = Math.max(maxLen, j - i);
        }
        return maxLen;
    }
}
```



##  49. 求第n个丑数

我们把`只包含质因子` 2、3 和 5 的数称作丑数（Ugly Number）。求按从小到大的顺序的第 n 个丑数。

**示例:**

```
输入: n = 10
输出: 12
解释: 1, 2, 3, 4, 5, 6, 8, 9, 10, 12 是前 10 个丑数。
```

**说明:** 

1. `1` 是丑数。
2. `n` **不超过**1690。

解法I：使用双重循环遍历，判断每一个数是不是丑数（时间复杂度O(n^2)，运行时会超时）

```java
class Solution {
    public int nthUglyNumber(int n) {

        if (n < 0) return 0;

        int idx = 0;
        int count = 0;
        while (idx < n) {
            count++;
            if (isUglyNumber(count)) {
                idx++;
            }
        }
        return count;
    }

    public boolean isUglyNumber(int n) {
        while (n % 2 == 0) {
            n = n / 2;
        }

        while (n % 3 == 0) {
            n = n / 3;
        }
        while (n % 5 == 0) {
            n = n / 5;
        }
        
        return n == 1 ? true : false; 
    }
}

```

解法II：使用一个O(n)的数组保存所有n个丑数

```java
class Solution {
    public int nthUglyNumber(int n) {

        if (n < 0) return 0;

        // 定义第一个数组用来保存n个丑数
        int[] uglyNum = new int[n];
        uglyNum[0] = 1;

        // 三个索引值，分别指向自己当前记录的最小索引
        int i2 = 0, i3 = 0, i5 = 0;

        int idx = 1;
        while (idx < n) {
            
            // 将2,3,5分别与自己记录的最小的索引值相乘，取最小的一个丑数保存到数组中
            int m2 = uglyNum[i2] * 2, m3 = uglyNum[i3] * 3, m5 = uglyNum[i5] * 5;
            
            uglyNum[idx] = Math.min(m2, Math.min(m3, m5));

            if (uglyNum[idx] == m2) i2++;
            if (uglyNum[idx] == m3) i3++;
            if (uglyNum[idx] == m5) i5++;
            
            idx++;
        }
        return uglyNum[n - 1];
    }
}
```



## 50. 第一个只出现一次的字符

在字符串 s 中找出第一个只出现一次的字符。如果没有，返回一个单空格。 s 只包含小写字母。

**示例:**

```
s = "abaccdeff"
返回 "b"

s = "" 
返回 " "
```

 **限制：**

```
0 <= s 的长度 <= 50000
```

```java
class Solution {
    public char firstUniqChar(String s) {
        char[] chars = s.toCharArray();
        Map<Character, Integer> map = new HashMap<>();
        for (char c : chars) {
            if (map.containsKey(c)) {
                Integer val = map.get(c);
                map.put(c, ++val);
            } else {
                map.put(c, 1);
            }
        }
        for (char c : chars) {
            if (map.get(c) == 1) {
                return c;
            }
        }
        return ' ';
    }
}
```



## 52. 两个链表的第一个公共节点

输入两个链表，找出它们的第一个公共节点。

**示例 1：**

[![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/14/160_example_1.png)](https://assets.leetcode.com/uploads/2018/12/13/160_example_1.png)

```
输入：intersectVal = 8, listA = [4,1,8,4,5], listB = [5,0,1,8,4,5], skipA = 2, skipB = 3
输出：Reference of the node with value = 8
输入解释：相交节点的值为 8 （注意，如果两个列表相交则不能为 0）。从各自的表头开始算起，链表 A 为 [4,1,8,4,5]，链表 B 为 [5,0,1,8,4,5]。在 A 中，相交节点前有 2 个节点；在 B 中，相交节点前有 3 个节点。
```



**示例 2：**

[![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/14/160_example_3.png)](https://assets.leetcode.com/uploads/2018/12/13/160_example_3.png)

```
输入：intersectVal = 0, listA = [2,6,4], listB = [1,5], skipA = 3, skipB = 2
输出：null
输入解释：从各自的表头开始算起，链表 A 为 [2,6,4]，链表 B 为 [1,5]。由于这两个链表不相交，所以 intersectVal 必须为 0，而 skipA 和 skipB 可以是任意值。
解释：这两个链表不相交，因此返回 null。
```

 

**注意：**

- 如果两个链表没有交点，返回 `null`.
- 在返回结果后，两个链表仍须保持原有的结构。
- 可假定整个链表结构中没有循环。
- 程序尽量满足 O(*n*) 时间复杂度，且仅用 O(*1*) 内存。

```java
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {

        ListNode node1 = headA;
        ListNode node2 = headB;

        // 当node1指向null时，就令其重新指向headB
        // 当node2指向null时，就令其重新指向headA
        // 这样的话两个节点走到公共节点的总路程相同，则必相遇
        while (node1 != node2){
            node1 = node1 == null ? headB : node1.next;
            node2 = node2 == null ? headA : node2.next;
        }
        return node1;
    }
}
```



## 53-I. 在排序数组中查找数字 

统计一个数字在排序数组中出现的次数。

**示例 1:**

```
输入: nums = [5,7,7,8,8,10], target = 8
输出: 2
```

**示例 2:**

```
输入: nums = [5,7,7,8,8,10], target = 6
输出: 0
```

**限制：**

```
0 <= 数组长度 <= 50000
```

解题思路：使用`二分法查找`到第一个target出现的下标位置和最后一个target出现的下标位置，然后计算出现的总次数。

```java
class Solution {
    public int search(int[] nums, int target) {
        if (nums.length <= 0) return 0;

        int first = getFirstTarget(nums, target, 0, nums.length -1 );
        int last = getLastTarget(nums, target, 0, nums.length -1 );

        if (first > -1 && last > -1)
            return last - first + 1;
        else
            return 0;
    }
	/**
	 * 求第一个target出现的 索引值
	 */
    public int getFirstTarget(int[] nums, int target, int left, int right) {

        if (left > right) return -1;

        int mid = (right + left) / 2;

        // 判断当前mid索引中的值是否等于target
        if (nums[mid] == target) {
            // 判断mid-1是否等于target
            if ((mid > 0 && nums[mid - 1] != target) || mid == 0) {
                return mid;
            } else {
                // 否则就说明 target 左边的数字也等于target，此时将mid-1设置为尾索引，继续检索
                return getFirstTarget(nums, target, left, mid - 1);
            }
        }

        if (nums[mid] > target)
            return getFirstTarget(nums, target, left, mid - 1);
        else
            return getFirstTarget(nums, target, mid + 1, right);
    }

    /**
	 * 求最后一个target出现的 索引值
	 */
    public int getLastTarget(int[] nums, int target, int left, int right) {
        if (left > right) return -1;

        int mid = (right + left) / 2;

        if (nums[mid] == target) {
            if ((mid < nums.length -1 && nums[mid + 1] != target) || mid == nums.length -1) {
                return mid;
            } else {
                return getLastTarget(nums, target, mid + 1, right);
            }
        }

        if (nums[mid] > target)
            return getLastTarget(nums, target, left, mid - 1);
        else
            return getLastTarget(nums, target, mid + 1, right);
    }
}
```



## 53-II. 0～n-1 中缺失的数字 

一个长度为n-1的递增排序数组中的所有数字都是唯一的，并且每个数字都在范围0～n-1之内。在范围0～n-1内的n个数字中有且只有一个数字不在该数组中，请找出这个数字。

**示例 1:**

```
输入: [0,1,3]
输出: 2
```

**示例 2:**

```
输入: [0,1,2,3,4,5,6,7,9]
输出: 8
```

 **限制：**

```
1 <= 数组长度 <= 10000
```

解题思路：利用数组的值和数组的下标，使用二分法来寻找缺失的值。

若mid == val，则说明缺失的值在数组右侧。

若mid != val，则说明缺失的值在数组左侧。

使用二分法时需要注意mid的边界问题。此题中一个特殊的数组 `{0}`，其 len = n = 1，数值范围在 0~n-1 即 0 ~ 0之间，则返回值应为 1。

```java
class Solution {
    public int missingNumber(int[] nums) {
        
        if (nums.length <= 0) return -1;

        int left = 0, right = nums.length - 1;

        // 凡是二分法寻找数值，需要考虑两头边界问题，防止数组越界
        while (left <= right) {
            int mid = (left + right) >> 1;

            if (mid != nums[mid]){
                if (mid == 0 || (mid - 1) == nums[mid - 1]){
                    return mid;
                }
                // 向左继续遍历
                right = mid - 1;
            }else {
                // 向右继续遍历
                left = mid + 1;
            }
        }
        // 防止{0}这个数组，其返回值为1
        if (left == nums.length) return nums.length;
        
        return  -1;
    }
}
```



## 54. 二叉搜索树的第k大节点

给定一棵二叉搜索树，请找出其中第k大的节点。

 **示例 1:**

```
输入: root = [3,1,4,null,2], k = 1
   3
  / \
 1   4
  \
   2
输出: 4
```

**示例 2:**

```
输入: root = [5,3,6,2,4,null,null,1], k = 3
       5
      / \
     3   6
    / \
   2   4
  /
 1
输出: 4
```

 **限制：**

1 ≤ k ≤ 二叉搜索树元素个数

`解题思路：`对二叉搜索树使用中序遍历可以得出一个有序递增的序列。

如果对该序列进行翻转，则输出第k个数字，即为第k大数字。

所以使用中序遍历，先对右子树遍历，后对左子树遍历，即可翻转中序遍历序列。

```java
class Solution {
    public int kthLargest(TreeNode root, int k) {
        if (k == 0) return 0;
        Deque<TreeNode> stack = new LinkedList<>();
        int result = 0;

        while (root != null || !stack.isEmpty()){
            while (root != null){
                stack.push(root);
                root = root.right;
            }

            if (!stack.isEmpty()){
                root = stack.pop();
                if (k == 1) return root.val;
                root = root.left;
                k--;
            }
        }
        return result;
    }
}
```



## 55-I. 二叉树最大深度 

**示例：**
给定二叉树 `[3,9,20,null,null,15,7]`，返回它的最大深度 3 

```
    3
   / \
  9  20
    /  \
   15   7
```

> “自底向上” 是另一种递归方法。 在每个递归层次上，我们首先对所有子节点递归地调用函数，然后根据返回值和根节点本身的值得到答案

```java
class Solution {
    public int maxDepth(TreeNode root) {
        
        if(root == null){
            return 0;
        }
        
        int leftMaxDepth = maxDepth(root.left);
        int rightMaxDepth = maxDepth(root.right);

        return Math.max( leftMaxDepth, rightMaxDepth ) + 1;
    }
}
```



##  55-II. 平衡二叉树 

输入一棵二叉树的根节点，判断该树是不是平衡二叉树。如果某二叉树中任意节点的左右子树的深度相差不超过1，那么它就是一棵平衡二叉树。

 **示例 1:**

给定二叉树 `[3,9,20,null,null,15,7]`

```
    3
   / \
  9  20
    /  \
   15   7
```

返回 `true` 。

**示例 2:**

给定二叉树 `[1,2,2,3,3,null,null,4,4]`

```
       1
      / \
     2   2
    / \
   3   3
  / \
 4   4
```

返回 `false` 。

 **限制：**

- `1 <= 树的结点个数 <= 10000`

解题思路：基于上一题中的二叉树的最大深度记录每个节点的深度，判断每个节点的左右子树是否平衡。

平衡的条件是 `-1 <= 左子树深度 - 右子树深度 <= 1`，如果不符合平衡条件，则选择一个标志符（下述算法在不平衡的条件下返回 -1）来标记该树不平衡

```java
class Solution {
    public boolean isBalanced(TreeNode root) {
        return recur(root) != -1;
    }

    public int recur(TreeNode node){

        if(node == null) return 0;

        int left = recur(node.left);
        if(left == -1) return -1;

        int right = recur(node.right);
        if(right == -1) return -1;

        return Math.abs(left - right) < 2 ? Math.max(left, right) + 1 : -1;
    }
}
```



## 56-I. 数组中数字出现的次数

一个整型数组 `nums` 里除两个数字之外，其他数字都出现了两次。请写程序找出这两个只出现一次的数字。要求时间复杂度是O(n)，空间复杂度是O(1)。

**示例 1：**

```
输入：nums = [4,1,4,6]
输出：[1,6] 或 [6,1]
```

**示例 2：**

```
输入：nums = [1,2,10,4,1,4,3,3]
输出：[2,10] 或 [10,2]
```

 **限制：**

- `2 <= nums.length <= 10000`

 ```java
class Solution {
    public int[] singleNumbers(int[] nums) {
        
        // 对所有数字进行异或操作，其结果为两个不同的数的异或值
        int res = 0;
        for (int n : nums){
            res ^= n;
        }

        // 根据异或结果res寻找低位第一个为 1 的位，可以由该位将整个数组划分为两个数组，
        // 则两个只出现一次的数字必定分开位于两个不同的数组中
        // 而其他出现了两次的数字必定位于同一个数组中
        int idx = 1;
        while ((idx & res ) == 0){
            idx = idx << 1;
        }

        int a = 0, b = 0;
        // 将数组中所有的数字与idx进行与运算，运算结果会将数组进行划分
        for (int n : nums){
            if ((idx & n) == 0){
                a ^= n;
            }else {
                b ^= n;
            }
        }
        return new int[]{a,b};
    }
}
 ```

## 56-II. 数组中数字出现的次数 

在一个数组 `nums` 中除一个数字只出现一次之外，其他数字都出现了三次。请找出那个只出现一次的数字。

 **示例 1：**

```
输入：nums = [3,4,3,3]
输出：4
```

**示例 2：**

```
输入：nums = [9,1,7,9,7,9,7]
输出：1
```

 **限制：**

- `1 <= nums.length <= 10000`
- `1 <= nums[i] < 2^31`

```java
class Solution {
    /**
     * 将所有数组中所有数字的每一位相加记录到一个数组中，由于只有一个数字出现了一次，其他数字均出现三次。
     * 那么出现三次的数字的每一位相加必能被三整除，所有出现三次的数字同理
     * 如果某一位能被三整除，则出现一次的数字在该位为0，否则为1
     */
    public  int singleNumber(int[] nums) {

        // 将nums中所有数字的每一位进行相加并存入一个数组中
        int[] bitSum = new int[32];

        for (int i = 0; i < nums.length; i++) {
            int bitMask = 1;
            for (int j = 31; j >= 0; j--) {
                int bit = bitMask & nums[i];
                if (bit != 0) bitSum[j] += 1;
                bitMask = bitMask << 1;
            }
        }

        // 解析 bitSum数组中所有的数字，能被三整除则为0，否则为1，最终得到的是只出现一次的数字
        int res = 0;
        for (int i = 0; i < 32; i++) {
            res = res << 1;         // 0 << 1 = 0
            res += bitSum[i] % 3;   // 对 00010001 进行模拟
        }
        return res;
    }
}
```



## 57-I. 和为s的两个数字 

难度简单56

输入一个递增排序的数组和一个数字s，在数组中查找两个数，使得它们的和正好是s。如果有多对数字的和等于s，则输出任意一对即可。

 

**示例 1：**

```
输入：nums = [2,7,11,15], target = 9
输出：[2,7] 或者 [7,2]
```

**示例 2：**

```
输入：nums = [10,26,30,31,47,60], target = 40
输出：[10,30] 或者 [30,10]
```

 

**限制：**

- `1 <= nums.length <= 10^5`
- `1 <= nums[i] <= 10^6`

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int i = 0;
        int j = nums.length - 1;

        while(i < j){
            if(nums[i] + nums[j] == target){
                return new int[]{nums[i],nums[j]};
            }

            if(nums[i] + nums[j] > target){
                j--;
            }else{
                i++;
            }
        }
        return new int[]{};
    }
}
```



##  57-II. 和为s的连续正数序列 

输入一个正整数 `target` ，输出所有和为 `target` 的连续正整数序列（至少含有两个数）。

序列内的数字由小到大排列，不同序列按照首个数字从小到大排列。

 **示例 1：**

```
输入：target = 9
输出：[[2,3,4],[4,5]]
```

**示例 2：**

```
输入：target = 15
输出：[[1,2,3,4,5],[4,5,6],[7,8]]
```

 **限制：**

- `1 <= target <= 10^5`

 解题思路：本题依旧可以使用双指针索引从 `1 - (target+1)/2`的所有数字，然后累加求和两个指针中间所有的数字。并对求和`sum`进行判断是否等于`target`，若等于则对两个指针指向数字和中间所有的数字进行保存。

初始条件：第一个指针指向 `small = 1`；第二个指针指向`big = 2`；`sum = small + big`，当sum<target时，则应该对big索引进行扩增，当sum > target时，则应该去除small所指的数字

```java
class Solution {
    public int[][] findContinuousSequence(int target) {
        
        int mid = (target + 1) / 2;
        int small = 1, big = 2;
        int sum = small + big;
        List<int[]> list = new ArrayList<>();
        
        while (small < mid) {
            // 如果子序列总和等于target则需要对子序列进行保存
            if (sum == target) {
                list.add(saveArr(small, big));
            }

            if (sum < target) {
                big++;
                sum += big;
            } else {
                sum -= small;
                small++;
            }
        }
        return list.toArray(new int[list.size()][]);
    }

    private int[] saveArr(int small, int big) {
        
        int idx = small;
        int[] ints = new int[big - small + 1];
        for (int i = 0; i < ints.length; i++){
            ints[i] = idx;
            idx++;
        }
        return ints;
    }
}
```



## 58-I. 翻转字符串单词顺序

输入一个英文句子，翻转句子中单词的顺序，但单词内字符的顺序不变。为简单起见，标点符号和普通字母一样处理。例如输入字符串"I am a student. "，则输出"student. a am I"。

 **示例 1：**

```
输入: "the sky is blue"
输出: "blue is sky the"
```

**示例 2：**

```
输入: "  hello world!  "
输出: "world! hello"
解释: 输入字符串可以在前面或者后面包含多余的空格，但是反转后的字符不能包括。
```

**示例 3：**

```
输入: "a good   example"
输出: "example good a"
解释: 如果两个单词间有多余的空格，将反转后单词间的空格减少到只含一个。
```

 **说明：**

- 无空格字符构成一个单词。
- 输入字符串可以在前面或者后面包含多余的空格，但是反转后的字符不能包括。
- 如果两个单词间有多余的空格，将反转后单词间的空格减少到只含一个。

解题思路：使用双指针front和tail，从字符串尾部开始遍历，将遇到的第一个空格字符后的第一个单词添加到返回字符串中。

```java
class Solution {
    public String reverseWords(String s) {
        String trim = s.trim();
        char[] chars = trim.toCharArray();
        StringBuilder result = new StringBuilder();
        int idx = 0, tail = chars.length - 1;
        int front = tail;

        while (tail >= 0) {

            // 获取某个单词前的空格字符的索引
            while (front >= 0 && chars[front] != ' ') front--;

            // 此时 front 指针指向 ' '
            for (int i = front + 1; i <= tail; i++) {
                result.append(chars[i]);
            }

            // front重新获取不为' '的字符，同时使tail = front
            while (front >= 0 && chars[front] == ' ') {
                front--;
            }

            // 此时front指向某个单词的末尾字母
            tail = front;
            if (front >= 0) result.append(' ');
        }
        return result.toString();
    }
}
```

**改进版：**直接操作字符串，无需将字符串转换为字符数组

```java
class Solution {
    public String reverseWords(String s) {
        String arr = s.trim();
        StringBuilder result = new StringBuilder();
        int tail = arr.length() - 1;
        int front = tail;

        while (tail >= 0) {

            while (front >= 0 && arr.charAt(front) != ' ') front--;

            // 此时 front指向空格字符
            result.append(arr.substring(front + 1, tail + 1) + ' ');

            // 过滤单词之间的空格字符
            while (front >= 0 && arr.charAt(front) == ' ') front--;
            tail = front;
        }
        return result.toString().trim();
    }
}
```

## 58 - II. 左旋转字符串

字符串的左旋转操作是把字符串前面的若干个字符转移到字符串的尾部。请定义一个函数实现字符串左旋转操作的功能。比如，输入字符串"abcdefg"和数字2，该函数将返回左旋转两位得到的结果"cdefgab"。

 

**示例 1：**

```
输入: s = "abcdefg", k = 2
输出: "cdefgab"
```

**示例 2：**

```
输入: s = "lrloseumgh", k = 6
输出: "umghlrlose"
```

 

**限制：**

- `1 <= k < s.length <= 10000`

```java
class Solution {
    public String reverseLeftWords(String s, int n) {
        StringBuilder res = new StringBuilder();
        res.append(s.substring(n, s.length()));
        res.append(s.substring(0,n));
        return res.toString();
    }
}
```

## 59 - I. 滑动窗口的最大值

给定一个数组 `nums` 和滑动窗口的大小 `k`，请找出所有滑动窗口里的最大值。

**示例:**

```
输入: nums = [1,3,-1,-3,5,3,6,7], 和 k = 3
输出: [3,3,5,5,6,7] 
解释: 

  滑动窗口的位置                最大值
---------------               -----
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7
```

 

**提示：**

提示：你可以假设 k 总是有效的，在输入数组不为空的情况下，1 ≤ k ≤ 输入数组的大小

```java
/**
 * 执行用时：41 ms, 在所有 Java 提交中击败了12.70%的用户
 * 内存消耗：47.1 MB, 在所有 Java 提交中击败了55.22%的用户
 */
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        if(nums == null || nums.length == 0) return new int[]{};
        // 设置首尾指针，固定滑动窗口
        int front = 0, tail = k - 1;
        // 初始化返回数组
        int[] res = new int[nums.length - k + 1];
        int idx = 0;
        // 当tail指针遍历到数组尾部时结束
        while (tail < nums.length) {
            // 在窗口内获取最大值
            int max = searchMaxNum(front, tail, nums);
            res[idx++] = max;
            tail++;
            front++;
        }
        return res;
    }

    private int searchMaxNum(int front, int tail, int[] nums) {
        int max = nums[front];
        while (front <= tail) {
            max = Math.max(max, nums[front++]);
        }
        return max;
    }
}
```

解法二：

使用双端队列保存滑动窗口内的`最大值的下标`（保存下标的原因：判断最大值是否还在滑动窗口的范围内）。

初始化双端队列保存**第一个滑动窗口**的最大值下标，然后依次从k开始遍历整个数组。

依次判断队尾索引的数值和当前索引的数值大小，如果队尾索引数值小于当前索引数值，则依次将队尾索引出队。

判断当前最大值的下标是否还在滑动窗口内：`maxIndex >= 当前索引下标 - 窗口大小`

当前索引的数字小于队尾下标对应的数值时，将其下标加入队列，因为它可能是下一个窗口的最大值下标

```java
/**
 * 执行用时：42 ms, 在所有 Java 提交中击败了16.70%的用户
 * 内存消耗：51.1 MB, 在所有 Java 提交中击败了52.22%的用户
 */
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        Deque<Integer> deque = new LinkedList<>();
        int[] res = new int[nums.length - k + 1];
        int idx = 0;
        if (nums.length >= k && k >= 1){

            // 先遍历前k个数值，将第一个滑动窗口内的最大值的下标保存到队列中
            for (int i = 0; i < k; i++){
                while (!deque.isEmpty() && nums[i] >= nums[deque.peekLast()]){
                    deque.pollLast();
                }
                deque.addLast(i);
            }
            res[idx++] = nums[deque.peekFirst()];
            // 在滑动窗口中寻找最大值和可能成为下一个窗口的最大值的下标
            for (int i = k; i < nums.length; i++){

                // 如果队尾的索引值小于当前索引值，则依次将队尾元素出队
                while (!deque.isEmpty() && nums[i] >= nums[deque.peekLast()])
                    deque.pollLast();

                // 判断队列中最大值的索引是否还在滑动窗口范围内
                if (!deque.isEmpty() && deque.peekFirst() <= i - k)
                    deque.pollFirst();

                // 将当前索引加入队尾
                deque.addLast(i);
                res[idx++] = nums[deque.peekFirst()];
            }
        }
        return res;
    }
}
```

## 59-II. 队列的最大值

请定义一个队列并实现函数 `max_value` 得到队列里的最大值，要求函数`max_value`、`push_back` 和 `pop_front` 的**均摊**时间复杂度都是O(1)。

若队列为空，`pop_front` 和 `max_value` 需要返回 -1

**示例 1：**

```
输入: 
["MaxQueue","push_back","push_back","max_value","pop_front","max_value"]
[[],[1],[2],[],[],[]]
输出: [null,null,null,2,1,2]
```

**示例 2：**

```
输入: 
["MaxQueue","pop_front","max_value"]
[[],[],[]]
输出: [null,-1,-1]
```

**限制：**

- `1 <= push_back,pop_front,max_value的总操作数 <= 10000`
- `1 <= value <= 10^5`

解题思路：对于一个普通队列，`push_back` 和 `pop_front` 的时间复杂度都是 O(1)，因此我们直接使用队列的相关操作就可以实现这两个函数。

对于 `max_value` 函数，可以使用一个`单调队列`来辅助维护一个队列中最大值的元素序列。

![fig3.gif](https://pic.leetcode-cn.com/9d038fc9bca6db656f81853d49caccae358a5630589df304fc24d8999777df98-fig3.gif)

```java
class MaxQueue {
    Deque<Integer> deque, maxDeque;

    public MaxQueue() {
        deque = new LinkedList<Integer>();
        maxDeque = new LinkedList<Integer>();
    }

    public int max_value() {
        if (maxDeque.isEmpty()) return -1;
        return maxDeque.peekFirst();
    }

    public void push_back(int value) {
        deque.addLast(value);
        // 每添加一个元素，则对辅助队列进行更新，同时保持辅助队列的单调性
        while (!maxDeque.isEmpty() && maxDeque.peekLast() < value) {
            maxDeque.removeLast();
        }
        maxDeque.addLast(value);
    }

    public int pop_front() {
        if (deque.isEmpty()) return -1;
        int res = deque.pollFirst();
        // 每出队一个元素，则对辅助队列进行判断更新
        if (maxDeque.peekFirst() == res) {
            maxDeque.removeFirst();
        }
        return res;
    }
}

/**
 * Your MaxQueue object will be instantiated and called as such:
 * MaxQueue obj = new MaxQueue();
 * int param_1 = obj.max_value();
 * obj.push_back(value);
 * int param_3 = obj.pop_front();
 */

```



## 60. n个骰子的点数

把n个骰子扔在地上，所有骰子朝上一面的点数之和为s。输入n，打印出s的所有可能的值出现的概率。

 你需要用一个浮点数数组返回答案，其中第 i 个元素代表这 n 个骰子所能掷出的点数集合中第 i 小的那个的概率。

**示例 1:**

```
输入: 1
输出: [0.16667,0.16667,0.16667,0.16667,0.16667,0.16667]
```

**示例 2:**

```
输入: 2
输出: [0.02778,0.05556,0.08333,0.11111,0.13889,0.16667,0.13889,0.11111,0.08333,0.05556,0.02778]
```

**限制：**

```
1 <= n <= 11
```

```java
/**
 *  n 枚骰子
 *  1 <= n <= 11
 *
 *  所有骰子的所有点数出现的次数和为6^n
 *  即 1个 骰子，点数的次数和为 6
 *  2个骰子，次数和为 36
 *
 *  6^11 = 3 6279 7056 < 21 4748 3647 = int的范围
 *
 */
public double[] twoSum(int n) {
    int[][] dp = new int[n + 1][n * 6 + 1];
    // 只有一枚骰子的情况下
    for (int i = 1; i <= 6; i++) dp[1][i] = 1;

    for (int i = 2; i <= n; i++) {
        // 计算 i 枚骰子的情况下，各个点数出现的次数
        for (int s = i; s <= i * 6; s++) {

// 计算公式: 和为n的骰子出现的次数 = 上一轮中骰子点数和n-1、n-2、n-3、n-4、n-5、n-6的次数总和
// 当有两枚骰子时，点数和12出现的次数，取决于上一轮一枚骰子点数和为 11、10、9、8、7、6出现的次数和，即为 1
// 当有两枚骰子时，点数和5出现的次数，取决于上一轮一枚骰子点数和为 4 3 2 1出现的次数和，即为 4
// 根据点数和5的情况，因此 需要对 s - d 的边界进行约束
            for (int d = 1; d <= 6 && (s - d) >= (i - 1); d++) {
                dp[i][s] += dp[i - 1][s - d];
            }
        }
    }

    double total = Math.pow(6, n);
    // 当有n个骰子时，前n-1个点数是不存在
    // 当有6个骰子时，点数5~1是不存在的，因此数组的大小为 6*n-(n-1) = 5*n + 1
    double[] res = new double[5 * n + 1];
    for (int i = n; i <= 6 * n; i++) {
        res[i - n] = (double) dp[n][i] / total;
    }
    return res;
}
```



## 61. 扑克牌中的顺子

从扑克牌中随机抽5张牌，判断是不是一个顺子，即这5张牌是不是连续的。2～10为数字本身，A为1，J为11，Q为12，K为13，而大、小王为 0 ，可以看成任意数字。A 不能视为 14。

 **示例 1:**

```
输入: [1,2,3,4,5]
输出: True
```

**示例 2:**

```
输入: [0,0,1,2,5]
输出: True
```

 **限制：**

数组长度为 5 

数组的数取值为 [0, 13] 

`解题思路：`首先对数组进行排序，其次统计数组中0的个数和缺失数字的个数，如果`缺失数字的个数<=0的个数`，则为顺子。如果数组中出现两个相同的数字（除0外）则不是顺子。

```java
public boolean isStraight(int[] nums) {
    Arrays.sort(nums);

    int numOfZero = 0;
    int numOfGap = 0;
    
    // 计算数组中 0 的个数
    for (int i = 0; i < nums.length; i++) {
        if (nums[i] == 0) numOfZero++;
    }

    // 计算数组中间隔的数目,初始索引值=零的个数+1，然后从后向前计算numOfGap，避免数组越界
    int idx = numOfZero + 1;
    while (idx < nums.length){
        int gap = nums[idx] - nums[idx - 1];
        if (gap == 0) return false;
        if ( gap > 1) numOfGap += gap - 1;
    }
    return (numOfZero >= numOfGap) ? true : false;
}
```



## 62. 圆圈中最后剩下的数字

0,1,,n-1这n个数字排成一个圆圈，从数字0开始，每次从这个圆圈里删除第m个数字。求出这个圆圈里剩下的最后一个数字。

例如，0、1、2、3、4这5个数字组成一个圆圈，从数字0开始每次删除第3个数字，则删除的前4个数字依次是2、0、4、1，因此最后剩下的数字是3。

 **示例 1：**

```
输入: n = 5, m = 3
输出: 3
```

**示例 2：**

```
输入: n = 10, m = 17
输出: 2
```

 **限制：**

- `1 <= n <= 10^5`
- `1 <= m <= 10^6`

`解题思路：`可以将n个数字放入一个环形链表中，每次删除第m个数字，直到剩下最后一个节点。

下列方法使用数组链表存放数据，用`(idx + m - 1) % n;`和`n--`进行环形数据结构的模拟。

PS：本题不能使用LinkedList存放数据。由于m接近于n，运行时间为O(n^2),会超时。此外本题使用改方法解答效率低下，还有一个`使用数学分析的方法进行解答`

```java
public int lastRemaining(int n, int m) {
    List<Integer> list = new ArrayList<>();
    for (int i = 0; i < n; i++) list.add(i);

    int idx = 0;
    while (list.size() > 1) {
        idx = (idx + m - 1) % n;    // 构造一个环结构
        list.remove(idx);
        n--;
    }
    return list.get(0);
}
```

## 63. 股票的最大利润

假设把某股票的价格按照时间先后顺序存储在数组中，请问买卖该股票一次可能获得的最大利润是多少？

**示例 1:**

```
输入: [7,1,5,3,6,4]
输出: 5
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格。
```

**示例 2:**

```
输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。
```

 **限制：**

```
0 <= 数组长度 <= 10^5
```

解题思路：买入股票的时间必须在卖出股票时间之前，因此，可以寻找并记录最小的价格，判断最小价格和最小价格之后价格的最大差值，并记录返回。

```java
class Solution {
    public int maxProfit(int[] prices) {
        // 数组为空表示不能买入也不能卖出、数组长度为1表示只能买入不能卖出
        if (prices == null || prices.length < 2) return 0;
        // min记录当前索引i以内，即0~i中最小的值
        int min = prices[0];
        int maxProfit = 0;

        for (int i = 1; i < prices.length; i++) {
            if (prices[i] < min) min = prices[i];

            int profit = prices[i] - min;
            if (profit > maxProfit) maxProfit = profit;
        }
        return maxProfit;
    }
}
```



## 64. 求1+2+…+n

求 `1+2+...+n` ，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。

 **示例 1：**

```
输入: n = 3
输出: 6
```

**示例 2：**

```
输入: n = 9
输出: 45
```

**限制：**

- `1 <= n <= 10000`



`解题思路1：`使用逻辑运算表达式模拟判断条件，然后进行递归运算

```java
class Solution {
    public int sumNums(int n) {
        // 当n=0时，开始逐层返回n的值
        boolean  flag = n > 0 && (n += sumNums(n -  1)) > 0;
        return n;
    }
}
```

`解题思路2：` 计算 1+2+...+n 的求和公式为`n * ( n + 1) / 2`，因此可以使用下列的俄罗斯乘法模拟两数相乘，的到的最终值进行右移运算即可。

```java
/**
 * 俄罗斯乘法：计算35 * 72
 * 35 72
 * 17 144      35 >> 1      72  << 1
 * 8 288       17 >> 1      144 << 1
 * 4 576       8  >> 1      288 << 1
 * 2 1152      ...
 * 1 2304
 *
 * 从上到下，对每一行，若左边的数字若为奇数，则将右边的数字取出，累加。
 * 72+144+2304=2520
 * 累加的结果2520即为乘积。
 */
public static int quickMulti(int a, int b) {
    int ans = 0;
    for (; b > 0; b >>= 1) {
        // 如果b为奇数，则累加a
        if ((b & 1) > 0) {
            ans += a;
        }
        a <<= 1;
    }
    return ans;
}


class Solution {
    public int sumNums(int n) {
        int ans = 0, A = n, B = n + 1;
        boolean flag;

        // 如果B为奇数，则对 A 进行累加
        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        return ans >> 1;
    }
}
```



## 65. 不用加减乘除做加法

写一个函数，求两个整数之和，要求在函数体内不得使用 “+”、“-”、“*”、“/” 四则运算符号。

 **示例:**

```
输入: a = 1, b = 1
输出: 2
```

 **提示：**

- `a`, `b` 均可能是负数或 0
- 结果不会溢出 32 位整数

`解题思路：`**计算 5 + 17 其二进制为 00101 + 10001**

```shell
# 1.对两个二进制数的每一位相加，而不进位，其本质就是异或运算
10001 ^ 00101 = 10100

10001
00101
------
10100

# 2. 对两个二进制数进行与运算，然后左移一位
10001 & 00101 << 1

10001
00101
------
00001 << 1 = 00010

# 3. 将1和2两个计算结果进行求和，求和的过程依然是1,2两个步骤。此时计算步骤1时已经不产生进位，所以异或结果就是求和结果，执行2步骤的结果为0，依此条件可以退出循环
10100
00010
------
10110 = 22 = 5 + 17

# 在计算机系统中，数值一律用 补码 来表示和存储。补码的优势： 加法、减法可以统一处理（CPU只有加法器）。因此，以上方法 同时适用于正数和负数的加法 。
```

```java
class Solution {
    public int add(int a, int b) {
        int sum, carry;
        do {
            sum = a ^ b;
            carry = (a & b) << 1;
            a = sum;
            b = carry;
        }while (b != 0);
        return a;
    }
}
```



## 66. 构建乘积数组

给定一个数组 `A[0,1,…,n-1]`，请构建一个数组 `B[0,1,…,n-1]`，其中 `B` 中的元素 `B[i]=A[0]×A[1]×…×A[i-1]×A[i+1]×…×A[n-1]`。不能使用除法。

**示例:**

```
输入: [1,2,3,4,5]
输出: [120,60,40,30,24]
```

**提示：**

- 所有元素乘积之和不会溢出 32 位整数
- `a.length <= 100000`

解题思路：把B数组分解为上三角和下三角分别=计算。

假设`A = {1,2,3,4}`

先将下三角计算出来: 

​		B[0] = 1.

​		B[1] = A[0] * B[0] = 1.  

​		B[2] = A[1] * B[1] = 2. 

​		B[3] = A[2] * B[2] = 6 = A[2] * A[1] * A[0] = 3 * 2 * 1

则最终B[N] 的值就确定了。

而`B[N-1] = A[N] * B[N-1]`

类似的 `B[N-2] = A[N] * A[N-1] * B[N-2]` 

假设`temp = 1,temp *= A[N], temp *= A[N-1]`, 则上式可变为`B[N-1] *= temp, B[N-2] *=temp`

根据上述公式可以得出最终结果：

​		B[3] = 6

​		B[2] = A[3] * B[2] = 4 * 2 = 8

​		B[1] = A[3] * A[2] * B[1] = 4 * 3 * 1 = 12

​		B[0] = A[3] * A[2] * A[1] * B[0] = 24

如果用`temp`来记录每次的`A[N] * A[N-1] * …… * A[1]`则上式可变为	

​		B[3] = 6

​		B[2] = `temp` * B[2] = 4 * 2 = 8

​		B[1] = `temp` * B[1] = 4 * 3 * 1 = 12

​		B[0] = `temp` * B[0] = 24

<img src="https://pic.leetcode-cn.com/6056c7a5009cb7a4674aab28505e598c502a7f7c60c45b9f19a8a64f31304745-Picture1.png" alt="Picture1.png" style="zoom: 80%;" />

```java
class Solution {
    public int[] constructArr(int[] a) {
        if (a == null || a.length < 1) return new int[0];
        int[] b = new int[a.length];
        /**
         * 分别计算上三角和下三角的值
         *
         * 上三角计算公式：C[i] = C[i-1] * A[i-1]
         * 下三角计算公式：D[i] = D[i+1] * A[i+1]
         */
        b[0] = 1;
        int temp = 1;

        // 计算下三角
        for (int i = 1; i < a.length; i++) {
            b[i] = a[i - 1] * b[i - 1];
        }

        // 计算上三角
        for (int i = a.length - 2; i >= 0; i--) {
            temp *= a[i + 1];
            b[i] *= temp;
        }

        return b;
    }
}
```

