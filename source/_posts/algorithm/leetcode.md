---
title: leetcode
date: 2021-07-15 16:09:58
tags: 算法
---

## 算法心得——LeetCode

所有题目转载自LeetCode：https://leetcode-cn.com/

#### 1、两数之和

> 给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 target  的那 两个 整数，并返回它们的数组下标。
>
> 你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。
>
> 你可以按任意顺序返回答案。

示例 1：

```tex
输入：nums = [2,7,11,15], target = 9
输出：[0,1]
解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。
```

#### 解题：

思路一：使用 `HashMap` 或者 `HashTable`，空间换时间。时间复杂度和空间复杂度都为O(n)。

```java
    public int[] twoSum(int[] nums, int target) {
        HashMap<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            if (map.containsKey(nums[i])) {
                return new int[]{map.get(nums[i]), i};
            }
            map.put(target - nums[i], i);
        }
        return nums;
    }
```



#### 2、两数相加

> 给你两个 非空 的链表，表示两个非负的整数。它们每位数字都是按照 逆序 的方式存储的，并且每个节点只能存储 一位 数字。
>
> 请你将两个数相加，并以相同形式返回一个表示和的链表。
>
> 你可以假设除了数字 0 之外，这两个数都不会以 0 开头。

示例 1：

```tex
输入：l1 = [2,4,3], l2 = [5,6,4]
输出：[7,0,8]
解释：342 + 465 = 807.
```

#### 解题：

```java
	public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        boolean moreThanTen = (l1.val + l2.val) >= 10;
        ListNode root = new ListNode((l1.val + l2.val) % 10);
        ListNode now = root;
        ListNode zero = new ListNode(0, null);
        while (l1.next != null || l2.next != null) {
            l1 = l1.next == null ? zero : l1.next;
            l2 = l2.next == null ? zero : l2.next;
            int val = l1.val + l2.val;
            if (moreThanTen) val++;
            moreThanTen = val >= 10;
            ListNode next = new ListNode(val % 10);
            now.next = next;
            now = next;
        }
        if (moreThanTen) {
            now.next = new ListNode(1, null);
        }
        return root;
    }
```



#### 3、无重复字符的最长子串

> 给定一个字符串，请你找出其中不含有重复字符的 **最长子串** 的长度。

示例 1：

```tex
输入: s = "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```

#### 解题：

思路一：滑动窗口，左右两个index，存在重复的字符，左下标+1，否则右下标+1。

```java
    public int lengthOfLongestSubstring(String s) {
        if (s == null || s.length() == 0) return 0;
        int res = 0, left = 0, right = 1, i;
        while (right != s.length()) {
            for (i = left; i < right; i++) {
                if (s.charAt(right) == s.charAt(i)) {
                    break;
                }
            }
            if (i == right) {
                right++;
            } else {
                res = Math.max(right - left, res);
                left = i + 1;
            }
        }
        return Math.max(right - left, res);
    }
```

思路二：在一的基础上，查找字符是否重复，采用数组存储，减少查找时间。

```java
    public int lengthOfLongestSubstring(String s) {
        int[] arr = new int[128];
        int max = 0, left = 0;
        Arrays.fill(arr, -1);
        for (int i = 0; i < s.length(); i++) {
            int c = s.charAt(i);
            left = Math.max(arr[c] + 1, left);
            max = Math.max(max, i - left + 1);
            arr[c] = i;
        }
        return max;
    }
```



#### [5. 最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring/)

> 给你一个字符串 `s`，找到 `s` 中最长的回文子串。

```tex
输入：s = "babad"
输出："bab"
解释："aba" 同样是符合题意的答案。
```

#### 解题

滑动窗口

```java
    public String longestPalindrome(String s) {
        int left = 0, right = 0;
        for (int i = 0; i < s.length(); i++) {
            int maxSpace = Math.min(i, s.length() - i - 1);
            for (int j = 1; j <= maxSpace; j++) {                // aba
                if (s.charAt(i - j) == s.charAt(i + j)) {
                    if (right - left < 2 * j) {
                        left = i - j;
                        right = i + j;
                    }
                } else {
                    break;
                }
            }
            maxSpace = Math.min(i, s.length() - i - 2);
            for (int j = 0; j <= maxSpace; j++) {
                if (s.charAt(i - j) == s.charAt(i + j + 1)) {
                    if (right - left < 2 * j + 1) {
                        left = i - j;
                        right = i + j + 1;
                    }
                } else {
                    break;
                }
            }
        }
        return s.substring(left, right + 1);
    }
```



#### [8. 字符串转换整数 (atoi)](https://leetcode-cn.com/problems/string-to-integer-atoi/)

> 请你来实现一个 myAtoi(string s) 函数，使其能将字符串转换成一个 32 位有符号整数（类似 C/C++ 中的 atoi 函数）。
>
> 函数 myAtoi(string s) 的算法如下：
>
> 读入字符串并丢弃无用的前导空格
> 检查下一个字符（假设还未到字符末尾）为正还是负号，读取该字符（如果有）。 确定最终结果是负数还是正数。 如果两者都不存在，则假定结果为正。
> 读入下一个字符，直到到达下一个非数字字符或到达输入的结尾。字符串的其余部分将被忽略。
> 将前面步骤读入的这些数字转换为整数（即，"123" -> 123， "0032" -> 32）。如果没有读入数字，则整数为 0 。必要时更改符号（从步骤 2 开始）。
> 如果整数数超过 32 位有符号整数范围 [−231,  231 − 1] ，需要截断这个整数，使其保持在这个范围内。具体来说，小于 −231 的整数应该被固定为 −231 ，大于 231 − 1 的整数应该被固定为 231 − 1 。
> 返回整数作为最终结果。
> 注意：
>
> 本题中的空白字符只包括空格字符 ' ' 。
> 除前导空格或数字后的其余字符串外，请勿忽略 任何其他字符。

#### 解题

找规律：（+-）（\d+）

其他：也可以考虑使用正则匹配

​			考虑使用有限状态机

```java
    public int myAtoi(String s) {
        s = s.trim();
        long res = 0;
        boolean negative = false;
        char c;
        for (int i = 0; i < s.length(); i++) {
            c = s.charAt(i);
            if (i == 0 && c == '-') {
                negative = true;
            } else if (i == 0 && c == '+') {
            } else {
                if (c >= '0' && c <= '9') {
                    res = res * 10 + (c - '0');
                } else {
                    break;
                }
            }
            if (res > Math.pow(2, 31) - 1 && !negative) {
                res = (long) (Math.pow(2, 31) - 1);
                break;
            }
            if (res > Math.pow(2, 31) && negative) {
                res = (long) Math.pow(2, 31);
                break;
            }
        }
        if (negative) return (int) -res;
        return (int) res;
    }
```



#### [11. 盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water/)

> 给你 n 个非负整数 a1，a2，...，an，每个数代表坐标中的一个点 (i, ai) 。在坐标内画 n 条垂直线，垂直线 i 的两个端点分别为 (i, ai) 和 (i, 0) 。找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。
>
> 说明：你不能倾斜容器。

#### 解题

采用双指针的方式：left指向头，right指向末端。

```java
    public int maxArea(int[] height) {        
        int left = 0, right = height.length - 1;        
        int res = 0;        
        while (left < right) {            
            res = Math.max(res, Math.min(height[left], height[right]) 
                           * (right - left));            
            if (height[left] < height[right]) {                
                left++;            
            } else {                
                right--;            
            }        
        }        
        return res;    
    }
```



#### [15. 三数之和](https://leetcode-cn.com/problems/3sum/)

> 给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有和为 0 且不重复的三元组。
>
> 注意：答案中不可以包含重复的三元组。

示例 1：

```tex
输入：nums = [-1,0,1,2,-1,-4]输出：[[-1,-1,2],[-1,0,1]]
```

#### 解题

本题个人感觉最的难点是数据重复问题，使用HashMap的话解决数据重复比较困难。

使用双指针解决，将第二重和第三重循环合并为O(n)的复杂度。

```java
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> res = new ArrayList<>();
        if (nums.length < 3) return res;
        int left, right;
        Arrays.sort(nums);
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] > 0)
                break; // 第一个数大于 0，后面的数都比它大，肯定不成立了
            if (i > 0 && nums[i] == nums[i - 1])
                continue; // 去掉重复情况
            left = i + 1;
            right = nums.length - 1;
            while (left < right) {
                if (nums[i] + nums[left] + nums[right] > 0) {
                    right--;
                } else if (nums[i] + nums[left] + nums[right] < 0) {
                    left++;
                } else {
                    res.add(Arrays.asList(nums[i], nums[left], nums[right]));
                    left++;
                    right--; // 首先无论如何先要进行加减操作                    
                    while (left < right && nums[left] == nums[left - 1]) left++;
                    while (left < right && nums[right] == nums[right + 1]) right--;
                }
            }
        }
        return res;
    }
```



#### [34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

> 给定一个按照升序排列的整数数组 nums，和一个目标值 target。找出给定目标值在数组中的开始位置和结束位置。
>
> 如果数组中不存在目标值 target，返回 [-1, -1]。

**示例 1：**

```tex
输入：nums = [5,7,7,8,8,10], target = 8输出：[3,4]
```

#### 解题

作为已排序的数组查找元素，首先想到的是**二分查找**

```java
    public int[] searchRange(int[] nums, int target) {
        int left = 0, right = nums.length - 1, middle = (left + right) / 2;
        int[] res = new int[2];
        res[1] = nums.length - 1;
        while (left <= right) {
            if (nums[middle] == target) {
                for (int i = middle; i >= 0; i--) {
                    if (nums[i] != target) {
                        res[0] = i + 1;
                        break;
                    }
                }
                for (int i = middle; i < nums.length; i++) {
                    if (nums[i] != target) {
                        res[1] = i - 1;
                        break;
                    }
                }
                return res;
            } else if (nums[middle] > target) {
                right = middle - 1;
            } else {
                left = middle + 1;
            }
            middle = (left + right) / 2;
        }
        res[0] = -1;
        res[1] = -1;
        return res;
    }
```



#### [41. 缺失的第一个正数](https://leetcode-cn.com/problems/first-missing-positive/)

> 给你一个未排序的整数数组 `nums` ，请你找出其中没有出现的最小的正整数。
>
> 请你实现时间复杂度为 `O(n)` 并且只使用常数级别额外空间的解决方案。

**示例 1：**

```tex
输入：nums = [1,2,0]输出：3
```

#### 解题

遍历和HashSet，时间复杂度O(n)

```java
    public int firstMissingPositive(int[] nums) {
        HashSet<Integer> set = new HashSet<>();
        for (int num : nums) {
            if (num > 0) {
                set.add(num);
            }
        }
        int i = 1;
        while (true) {
            if (!set.contains(i)) {
                return i;
            }
            i++;
        }
    }
```



#### [55. 跳跃游戏](https://leetcode-cn.com/problems/jump-game/)

> 给定一个非负整数数组 `nums` ，你最初位于数组的 **第一个下标** 。
>
> 数组中的每个元素代表你在该位置可以跳跃的最大长度。
>
> 判断你是否能够到达最后一个下标。

示例 1：

```tex
输入：nums = [2,3,1,1,4]输出：true解释：可以先跳 1 步，从下标 0 到达下标 1, 然后再从下标 1 跳 3 步到达最后一个下标。
```

#### 解题

采用数组表示，循环向后判断是否可达

```java
    public boolean canJump(int[] nums) {
        if (nums.length < 2) return true;
        boolean[] isArrive = new boolean[nums.length];
        isArrive[0] = true;
        for (int i = 0; i < nums.length; i++) {
            if (isArrive[i]) {
                for (int j = 1; j <= nums[i]; j++) {
                    if (i + j == nums.length - 1) return true;
                    isArrive[i + j] = true;
                }
            }
        }
        return isArrive[nums.length - 1];
    }
```

如果k可达，那么k之前的所有位置都可达，循环判断可以达到的最大距离，如果当前位置不可达，说明后面位置也不可达。

```java
    public boolean canJump(int[] nums) {
        int res = 0;
        for (int i = 0; i < nums.length; i++) {
            if (i > res) return false;
            res = Math.max(i + nums[i], res);
        }
        return true;
    }
```



#### [62. 不同路径](https://leetcode-cn.com/problems/unique-paths/)

> 一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为 “Start” ）。
>
> 机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。
>
> 问总共有多少条不同的路径？

```tex
输入：m = 3, n = 7
输出：28
```

#### 解题

位置为（i，j）的路径 = （i+1，j）的路径 + （i，j+1）的路径

```java
    public int uniquePaths(int m, int n) {
        int[][] arr = new int[m][n];
        for (int i = 0; i < n; i++) {
            arr[m - 1][i] = 1;
        }
        for (int i = 0; i < m; i++) {
            arr[i][n - 1] = 1;
        }
        for (int i = m - 2; i >= 0; i--) {
            for (int j = n - 2; j >= 0; j--) {
                arr[i][j] = arr[i + 1][j] + arr[i][j + 1];
            }
        }
        return arr[0][0];
    }
```



#### [64. 最小路径和](https://leetcode-cn.com/problems/minimum-path-sum/)

> 给定一个包含非负整数的 `*m* x *n*` 网格 `grid` ，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。
>
> **说明：**每次只能向下或者向右移动一步。

```tex
输入：grid = [[1,3,1],[1,5,1],[4,2,1]]
输出：7
解释：因为路径 1→3→1→1→1 的总和最小。
```

#### 解题

动态规划的思想：```min[i][j] = min(min[i-1][j], min[i][j-1])+grid[i][j]```

```java
    public int minPathSum(int[][] grid) {
        int m = grid.length, n = grid[0].length;
        int[][] res = new int[m][n];
        res[0][0] = grid[0][0];
        for (int i = 1; i < n; i++) {
            res[0][i] = res[0][i - 1] + grid[0][i];
        }
        for (int i = 1; i < m; i++) {
            res[i][0] = res[i - 1][0] + grid[i][0];
        }
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                res[i][j] = Math.min(res[i - 1][j], res[i][j - 1]) + grid[i][j];
            }
        }
        return res[m - 1][n - 1];
```

空间优化：```min(min[i-1][j], min[i][j-1])``` 可以通过一个一维数组表示，每次选出一行每个位置的最小距离。

```java
    public int minPathSum(int[][] grid) {
        int m = grid.length, n = grid[0].length;
        int[] res = new int[n];
        Arrays.fill(res, Integer.MAX_VALUE);
        res[0] = grid[0][0];
        for (int i = 0; i < m; i++) {
            if (i > 0) {
                res[0] = res[0] + grid[i][0];
            }
            for (int j = 1; j < n; j++) {
                res[j] = Math.min(res[j - 1], res[j]) + grid[i][j];
            }
        }
        return res[n - 1];
    }
```



#### [71. 简化路径](https://leetcode-cn.com/problems/simplify-path/)

> 给你一个字符串 path ，表示指向某一文件或目录的 Unix 风格 绝对路径 （以 '/' 开头），请你将其转化为更加简洁的规范路径。
>
> 在 Unix 风格的文件系统中，一个点（.）表示当前目录本身；此外，两个点 （..） 表示将目录切换到上一级（指向父目录）；两者都可以是复杂相对路径的组成部分。任意多个连续的斜杠（即，'//'）都被视为单个斜杠 '/' 。 对于此问题，任何其他格式的点（例如，'...'）均被视为文件/目录名称。
>
> 请注意，返回的 规范路径 必须遵循下述格式：
>
> 始终以斜杠 '/' 开头。
> 两个目录名之间必须只有一个斜杠 '/' 。
> 最后一个目录名（如果存在）不能 以 '/' 结尾。
> 此外，路径仅包含从根目录到目标文件或目录的路径上的目录（即，不含 '.' 或 '..'）。
> 返回简化后得到的 规范路径 。

```tex
输入：path = "/home/"
输出："/home"
解释：注意，最后一个目录名后面没有斜杠。 
```

#### 解题

路径存在 `.` 和 `..` 两种特殊形式，将路径放在栈中，遇到返回上一页就执行`pop`操作，正常路径执行`push`操作。

```java
    public String simplifyPath(String path) {
        Stack<String> stack = new Stack<>();
        String[] paths = path.split("/");
        for (String s : paths) {
            if (s.equals("") || s.equals(".")) {
                continue;
            }
            if (s.equals("..")) {
                if (!stack.empty()) stack.pop();
                continue;
            }
            stack.push(s);
        }
        StringBuilder builder = new StringBuilder();
        for (String s : stack) {
            builder.append("/").append(s);
        }
        return builder.length() == 0 ? "/" : builder.toString();
    }
```



#### [72. 编辑距离](https://leetcode-cn.com/problems/edit-distance/)

> 给你两个单词 word1 和 word2，请你计算出将 word1 转换成 word2 所使用的最少操作数 。
>
> 你可以对一个单词进行如下三种操作：
>
> 插入一个字符
> 删除一个字符
> 替换一个字符

```tex
输入：word1 = "horse", word2 = "ros"
输出：3
解释：
horse -> rorse (将 'h' 替换为 'r')
rorse -> rose (删除 'r')
rose -> ros (删除 'e')
```

#### 解题

min(i，j) : word1的前i个元素转化为word2的前i个元素的最少操作。

min(i-1，j)和min(i，j-1)和min(i，j)的区别是word1或者word2添加了一个字符（对应上述三种操作的前两种），那必定在基础上 + 1。

min(i-1，j-1)和min(i，j)的区别是如果word1的第i个元素和word2的第j个元素相等，则操作一样，不相等执行替换操作（word1第i个元素替换成word2第j个元素）。

因此：min(i，j) = Min(min(i-1，j-1)(+1)，min(i-1，j)，min(i，j-1))

```java
    public int minDistance(String word1, String word2) {
        int[][] arr = new int[word1.length() + 1][word2.length() + 1];
        for (int i = 0; i <= word1.length(); i++) {
            arr[i][0] = i;
        }
        for (int i = 0; i <= word2.length(); i++) {
            arr[0][i] = i;
        }
        int t;
        for (int i = 1; i <= word1.length(); i++) {
            for (int j = 1; j <= word2.length(); j++) {
                t = arr[i - 1][j - 1];
                if (word1.charAt(i - 1) != word2.charAt(j - 1)) {
                    t++;
                }
                arr[i][j] = Math.min(t, Math.min(arr[i][j - 1], arr[i - 1][j]) + 1);
            }
        }

        return arr[word1.length()][word2.length()];
    }
```

空间优化：需要两个变量，front存在当前第i个元素的值，方便修改第i个元素后可以找到；pre保存min(i-1, j-1)的值，获取min(i, j)值的时候需要用到。

```java
    public int minDistance2(String word1, String word2) {
        int[] arr = new int[word1.length() + 1];
        for (int i = 0; i <= word1.length(); i++) {
            arr[i] = i;
        }
        int front, pre;
        for (int i = 0; i < word2.length(); i++) {
            pre = arr[0];
            arr[0]++;
            for (int j = 1; j <= word1.length(); j++) {
                front = arr[j];

                if (word2.charAt(i) != word1.charAt(j - 1)) {
                    pre++;
                }
                arr[j] = Math.min(pre, Math.min(arr[j - 1], arr[j]) + 1);
                pre = front;
            }

        }
        return arr[word1.length()];
    }
```



#### [74. 搜索二维矩阵](https://leetcode-cn.com/problems/search-a-2d-matrix/)

> 编写一个高效的算法来判断 m x n 矩阵中，是否存在一个目标值。
>
> 该矩阵具有如下特性：
>
> 每行中的整数从左到右按升序排列。
> 每行的第一个整数大于前一行的最后一个整数。

```tex
输入：matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 3
输出：true
```

#### 解题

```jade
    public boolean searchMatrix(int[][] matrix, int target) {
        if (matrix.length == 0 || matrix[0].length == 0) return false;
        for (int i = 0; i < matrix.length; i++) {
            if (target >= matrix[i][0]) {
                continue;
            }
            if (i == 0) return false;
            for (int j = 0; j < matrix[i - 1].length; j++) {
                if (target < matrix[i - 1][j]) return false;
                if (target == matrix[i - 1][j]) return true;
            }
        }
        int last = matrix.length - 1;
        // 处理最后一行
        for (int i = 0; i < matrix[last].length; i++) {
            if (target < matrix[last][i]) return false;
            if (target == matrix[last][i]) return true;
        }
        return false;
    }
```



#### [202. 快乐数](https://leetcode-cn.com/problems/happy-number/)

> 编写一个算法来判断一个数 n 是不是快乐数。
>
> 「快乐数」定义为：
>
> 对于一个正整数，每一次将该数替换为它每个位置上的数字的平方和。
> 然后重复这个过程直到这个数变为 1，也可能是 无限循环 但始终变不到 1。
> 如果 可以变为  1，那么这个数就是快乐数。
> 如果 n 是快乐数就返回 true ；不是，则返回 false 。

#### 解题

方法一：非快乐数最后一定以某组数进行循环，如何获取重复的数（并不一定第一个数就是循环的数），选取第一个数为可能循环的数，如果超过10次，仍在循环，选取当前计算的数为可能循环的数

```java
    public boolean isHappy(int n) {
        if (n <= 0) return false;
        int next = getNumber(n), first = next, i = 1, pre;
        if (next == 1) return true;
        do {
            pre = next;
            next = getNumber(next);
            if (next == 1) return true;
            i++;
            if (i % 10 == 0) {
                first = pre;
            }
        } while (next != first);
        return false;
    }

	public int getNumber(int n) {
        int sum = 0;
        while (n != 0) {
            sum += Math.pow(n % 10, 2);
            n = n / 10;
        }
        return sum;
    }
```

方法二：快慢指针，一个一倍速前进，一个二倍速前进，在循环中，快指针最好肯定能够追上慢的指针并与他相遇。

```java
    public boolean isHappy2(int n) {
        int slow = n, fast = n;
        do {
            slow = getNumber(slow);
            fast = getNumber(fast);
            fast = getNumber(fast);
            if (slow == 1) return true;
        } while (slow != fast);
        return false;
    }
```



#### [209. 长度最小的子数组](https://leetcode-cn.com/problems/minimum-size-subarray-sum/)

> 给定一个含有 n 个正整数的数组和一个正整数 target 。
>
> 找出该数组中满足其和 ≥ target 的长度最小的 连续子数组 [numsl, numsl+1, ..., numsr-1, numsr] ，并返回其长度。如果不存在符合条件的子数组，返回 0。
>

#### 解题

滑动窗口

```java
    public int minSubArrayLen(int target, int[] nums) {
        int left = 0, right = 0, sum = 0, res = Integer.MAX_VALUE;
        while (right < nums.length) {
            sum += nums[right];
            while (sum >= target) {
                res = Math.min(res, right - left + 1);
                sum -= nums[left++];
            }
            right++;
        }
        if (res == Integer.MAX_VALUE) return 0;
        return res;
    }
```





### 算法思路

#### 一、数组查找

+ 排序数据查找——**二分查找**
+ 多个（一般为两个，如求和）数据查找——可以使用**Hash**来实现，空间复杂度增加。（ `HashMap`、`HashSet` 等）

#### 二、滑动窗口

循环后面的元素需要和前面的元素挂钩，左右两个下标挪动。

#### 三、双指针

和滑动窗口类似，通过两个下标操作，但双指针的右指针指向末尾。

#### 四、动态规划

满足类似 `f(n) = f(n-1) + f(n-2)` 的规律，第i个值可以根据前面的值进行计算，一般通过找到规律列出函数，使用二维数组和循环可以解决。最难的部分是**找规律**。二维数组部分可以简化代码，如果只和最近的值相关，一般可以把二维数组转换为一维数组（参考64、72）

