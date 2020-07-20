[TOC]

# 算法
## 对碰指针
### leetcode167-在一个有序数组中找到和为N的两个数
### leetcode11-盛最多的水
给你 n 个非负整数 a1，a2，...，an，每个数代表坐标中的一个点 (i, ai) 。在坐标内画 n 条垂直线，垂直线 i 的两个端点分别为 (i, ai) 和 (i, 0)。找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。


## 滑动窗口
### leetcode209-长度最小的子数组
给定一个含有 n 个正整数的数组和一个正整数 s ，找出该数组中满足其和 ≥ s 的长度最小的连续子数组，并返回其长度。如果不存在符合条件的连续子数组，返回 0。

### leetcode3-无重复的最长子串
给定一个字符串，请你找出其中不含有重复字符的 最长子串 的长度。

示例 1:
```
输入: "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```

代码：
```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        int[] freq = new int[256]; // 优化：用一个char数组来记录各个字符出现的频率，从而判断一个字符是否出现过。
        char[] charArr = s.toCharArray();
        int l=0;
        int r=-1;
        int max = 0;
        while(l<charArr.length){
            if(r+1<charArr.length && freq[charArr[r+1]] == 0){
                r++;
                freq[charArr[r]]++;
                if(r-l+1>max) max=r-l+1;
            }else{
                freq[charArr[l]]--;
                l++;
            }
        }
        return max;
    }
}
```

### leetcode76-最小覆盖子串-hard

给你一个字符串 S、一个字符串 T，请在字符串 S 里面找出：包含 T 所有字符的最小子串。

示例：

> 输入: S = "ADOBECODEBANC", T = "ABC"
> 输出: "BANC"


思路:
```txt
/**时间复杂度O(N)，空间复杂度O(1)；  技术：滑动窗口+计数索引（不知道是不是这样叫，可以理解为简单的Hash表实现）
这道题有一定难度，leetcode438 号题也可使用类似的思路，不过稍微简单一些。

1. 注意到题目的关键："所有字母的最小子串"，也就是说两个串都只能是字母。
2. 于是，可以开辟一个大小为64的数组，来存放数组中字母的频率(Frequency)。准确的说，
   通过字母的ASCII码作为数组的索引，开辟空间的大小为26+6+26=58：26个大写字母，26个小写字母，
   还有中间的6个非字母  A~Z[65~90]  非字母[91~96]  a~z[97~122]
3. 滑动窗口的使用：分三种情况来移动窗口：（这里令当前窗口的左右边界分别为l，r，窗口的大小为winSize=r-l+1）
   1) 当winSize < t.size()  r++;  也就是窗口右边界向右移动
   2) 当winSize == t.size() :
	   2.1) 当窗口中的字符已经符合要求了，直接返回return，已经找到了
	   2.2) 否则r++，窗口右边界向右移动
   3) 当winSize > t.size()
	   3.1) 当窗口中的字符已经符合要求了，l++，窗口左边界向右移动
	   3.2) 否则r++，窗口右边界向右移动

4. 上面是滑动窗口的使用思路，具体实现上有一定的不同，下面是需要考虑到的要点：
   1) 啥叫作窗口中的字符已经符合要求了？
   2) 窗口滑动时的操作是关键
   3) 要考虑到数组越界的问题
```

```java

class Solution {
    public String minWindow(String s, String t) {
        char[] sArr = s.toCharArray();
        char[] tArr = t.toCharArray();
        int[] sCount = new int[58];
        int[] tCount = new int[58];

        for(int i=0; i<tArr.length; i++){
            int index = tArr[i] - 'A';
            tCount[index]++;
        }
        
        int minLen = sArr.length + 1;
        char[] result = null;
        int winSize = tArr.length;

        int l=0;
        int r=-1;
        while(l<sArr.length){
            while(true){
                if(r+1<sArr.length){
                    sCount[sArr[++r]-'A']++;
                    if(r-l+1>=winSize && isContain(sCount,tCount)){
                        if(r-l+1<minLen){
                            minLen = r-l+1;
                            result = new char[minLen];
                            for(int k=0;k<minLen;k++){
                                result[k] = sArr[l+k];
                            }
                        }
                        break;
                    }
                }else{
                    break;
                }
            }
            while(true){
                if(l<sArr.length){
                    sCount[sArr[l]-'A']--;
                    l++;
                    if(isContain(sCount,tCount)){
                        if(r-l+1<minLen){
                            minLen = r-l+1;
                            result = new char[minLen];
                            for(int k=0;k<minLen;k++){
                                result[k] = sArr[l+k];
                            }
                        }
                    }else{
                        break;
                    }
                }else{
                    sCount[sArr[l]-'A']--;
                    l++;
                    break;
                }
            }
            
        }

        String sResult = "";
        if(result!=null){
            sResult = String.valueOf(result);
        }

        return sResult;

    }

    private boolean isContain(int[] s, int[] t){
        for(int i=0; i<t.length;i++){
            if(s[i] < t[i]){
                return false;
            }
        }
        return true;
    }
}

```

## 查找表

### 217 contains-duplicate(easy)

给定一个整数数组，判断是否存在重复元素。

如果任意一值在数组中出现至少两次，函数返回 true 。如果数组中每个元素都不相同，则返回 false 。 

示例 1:

> 输入: [1,2,3,1]
> 输出: true

代码：

```java
// 算法复杂度O(n)
class Solution {
    public boolean containsDuplicate(int[] nums) {
        Set<Integer> set = new HashSet<>();
        for(int i=0;i<nums.length;i++){
            if(set.contains(nums[i])) return true;
            set.add(nums[i]);
        }
        return false;
    }
}
```



### leetcode242-有效的异位字符串(anagram)

给定两个字符串 *s* 和 *t* ，编写一个函数来判断 *t* 是否是 *s* 的字母异位词。

示例 1:

> 输入: s = "anagram", t = "nagaram"
> 输出: true

思路：把字符串s各个字符放到Map中，

代码：

```java
class Solution {
    public boolean isAnagram(String s, String t) {
        HashMap<Character,Integer> map1 = new HashMap<>();

        for(Character c : s.toCharArray()){
            map1.put(c,map1.getOrDefault(c,0)+1);
        }

        for(Character c : t.toCharArray()){
            Integer count = map1.get(c);
            if(count == null) return false;
            if(count == 1){
                map1.remove(c);
            }else{
                map1.put(c,count-1);
            }
        }
        return map1.isEmpty();
    }
}
```



### leetcode1-two sum

给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。你可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍。

示例:

> 给定 nums = [2, 7, 11, 15], target = 9
> 因为 nums[0] + nums[1] = 2 + 7 = 9
> 所以返回 [0, 1]



### leetcode454-4Sum II

给定四个包含整数的数组列表 A , B , C , D ,计算有多少个元组 (i, j, k, l) ，使得 A[i] + B[j] + C[k] + D[l] = 0。

为了使问题简单化，所有的 A, B, C, D 具有相同的长度 N，且 0 ≤ N ≤ 500 。所有整数的范围在 -228 到 228 - 1 之间，最终结果不会超过 231 - 1 。

例如:

> 输入:
> A = [ 1, 2]
> B = [-2,-1]
> C = [-1, 2]
> D = [ 0, 2]
>
> 输出:
> 2

思路：题目中提到长度N的范围是小于500的，由此我们大概可以估计算法的复杂度应该是O(n^2)级别的

代码：

```java
class Solution {
    public int fourSumCount(int[] A, int[] B, int[] C, int[] D) {
        HashMap<Integer,Integer> map = new HashMap<>();
        int len = A.length;
        int sum = 0;
        int result = 0;

        // 建立C+D的查找表
        for(int i=0; i<len; i++){
            for(int j=0; j<len; j++){
                sum = C[i] + D[j];
                if(map.containsKey(sum)){
                    map.put(sum,map.get(sum)+1);
                }else{
                    map.put(sum,1);
                }
            }
        }

        for(int i=0; i<len; i++){
            for(int j=0; j<len; j++){
                sum = 0 - A[i] - B[j];
                if(map.containsKey(sum)){
                    result += map.get(sum);
                }
            }
        }
        return result;
    }
}
```



### 447-number-of-boomerangs-simple

给定平面上 n 对不同的点，“回旋镖” 是由点表示的元组 (i, j, k) ，其中 i 和 j 之间的距离和 i 和 k 之间的距离相等（需要考虑元组的顺序）。

找到所有回旋镖的数量。你可以假设 n 最大为 500，所有点的坐标在闭区间 [-10000, 10000] 中。

示例:

> 输入:
> [[0,0],[1,0],[2,0]]
>
> 输出:
> 2
>
> 解释:
> 两个回旋镖为 [[1,0],[0,0],[2,0]] 和 [[1,0],[2,0],[0,0]]

小技巧：如何计算两点的距离？只需要求两个点之间距离的平方就可以，不用求它们之间的真实距离。

思路：

建立第i个节点到其他节点距离的查找表，值为距离相同的节点数，统计节点数大于等于2的表项，**假如距离相同的节点数为n，满足条件的元组数量为n*(n-1)**：

![image-20200624160935259](../image/image-20200624160935259.png)

代码：

```java
// O(n^2)
class Solution {
    public int numberOfBoomerangs(int[][] points) {
        int len = points.length;
        int result = 0;
        for(int i=0;i<len;i++){ 
            Map<Integer, Integer> map = new HashMap<>();
            int[] point = points[i]; // 以点i为锚点，建立点i到其他点距离的查找表
            for(int j=0;j<len;j++){
                if(j != i){
                    int distance = dis(point,points[j]);
                    map.put(distance,map.getOrDefault(distance,0)+1);
                }
            }
            for(Integer c : map.values()){
                if(c>=2){
                    result += c*(c-1); // 符合条件的元组
                }
            }
        }
        return result;
    }

    // 两点之间距离（的平方）
    private int dis(int[] point1, int[] point2){
        return (point1[0]-point2[0])*(point1[0]-point2[0]) + (point1[1]-point2[1])*(point1[1]-point2[1]);
    }
}
```





### 149-max points on a line-hard-todo



给定一个二维平面，平面上有 n 个点，求最多有多少个点在同一条直线上。

示例 1:

> 输入: [[1,1],[2,2],[3,3]]
> 输出: 3
> 解释:
> ^
> |
> |        o
> |     o
> |  o  
> +------------->
> 0  1  2  3  4







## 查找表&滑动窗口

### 219 contains-duplicate-ii(easy)

给定一个整数数组和一个整数 k，判断数组中是否存在两个不同的索引 i 和 j，使得 nums [i] = nums [j]，并且 i 和 j 的差的 绝对值 至多为 k。

 示例 1:

> 输入: nums = [1,2,3,1], k = 3
> 输出: true

代码：

```java
// 适用滑动窗口
class Solution {
    public boolean containsNearbyDuplicate(int[] nums, int k) {
        int l = 0;
        int r = 1;
        while(r < nums.length){
            if(r-l <= k){
                if(nums[r] == nums[l]) return true;
                r++;
                continue;
            }
            l++;
            while(l<r){                      
                if(nums[r] == nums[l]) return true;       
                l++; 
            }
            r++;      
        }
        return false;
    }
}
```

滑动窗口+查找表解法参考：https://coding.imooc.com/lesson/82.html#mid=2716



## 链表

- 如果空间不做限制的话可以考虑将链表拷到一个数组中进行解决
- 使用虚拟头结点可以简化头结点的处理

### 150 逆波兰表达式

### 71 简化路径





## 栈和队列

队列：可用于树的广度优先遍历(102 103 199都是考这个知识点)

优先队列底层一般是堆结构。

### 102 二叉树的层序遍历

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
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> result = new ArrayList<>();
        if (root == null) return result;
        List<TreeNode> queue = new ArrayList<>();
        List<Integer> a = new ArrayList<>();
        a.add(root.val);
        result.add(a);
        if(root.left !=null) queue.add(root.left);
        if(root.right!=null) queue.add(root.right);
        while(queue.size()>0){
            List<Integer> aa = new ArrayList<>();
            for(int i=0;i<queue.size();i++){
                aa.add(queue.get(i).val);
            }
            result.add(aa);
            int size = queue.size();
            for(int i=0; i<size;i++){
                TreeNode node = queue.remove(0);
                if(node.left!=null) queue.add(node.left);
                if(node.right!=null) queue.add(node.right);
            }
        }
        return result;
    }
}
```



### [103. 二叉树的锯齿形层次遍历](https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal/)-中等

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
    public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
        List<List<Integer>> result = new ArrayList<>();
        if (root == null) return result;
        List<TreeNode> queue = new ArrayList<>();
        List<Integer> a = new ArrayList<>();
        a.add(root.val);
        result.add(a);
        if(root.left !=null) queue.add(root.left);
        if(root.right!=null) queue.add(root.right);
        int count = 1;
        while(queue.size()>0){
            List<Integer> aa = new ArrayList<>();
            if( count % 2 == 0){
                for(int i=0;i<queue.size();i++){
                    aa.add(queue.get(i).val);
                }
            }else{
                for(int i=queue.size()-1;i>=0;i--){
                    aa.add(queue.get(i).val);
                }
            }
            
            result.add(aa);
            int size = queue.size();
            for(int i=0; i<size;i++){
                TreeNode node = queue.remove(0);
                if(node.left!=null) queue.add(node.left);
                if(node.right!=null) queue.add(node.right);
            }
            count++;
        }
        return result;
    }
}
```





### [199. 二叉树的右视图](https://leetcode-cn.com/problems/binary-tree-right-side-view/)-中等

给定一棵二叉树，想象自己站在它的右侧，按照从顶部到底部的顺序，返回从右侧所能看到的节点值。

示例:

> 输入: [1,2,3,null,5,null,4]
> 输出: [1, 3, 4]
> 解释:
>   1            <---
>  /   \
> 2     3         <---
>  \     \
>   5     4       <---

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
    public List<Integer> rightSideView(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        if(root == null) return result;
        Queue<TreeNode> queue = new LinkedBlockingQueue<>();
        queue.offer(root);
        while(queue.size()>0){
            TreeNode right = null;
            int size = queue.size();
            for(int i=0;i<size;i++){
                right = queue.poll();
                if(right.left != null) queue.offer(right.left);
                if(right.right != null) queue.offer(right.right);
            }
            result.add(right.val);
        }
        return result;
    }
}
```

## 二叉树

### [104. 二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

> 给定一个二叉树，找出其最大深度。二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。
>
> 说明: 叶子节点是指没有子节点的节点。
>
> 示例：
> 给定二叉树 [3,9,20,null,null,15,7]，
    3
   / \
  9  20
    /  \
   15   7
返回它的最大深度 3 。

实现：

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
    public int maxDepth(TreeNode root) {
        if(root == null)  return 0;
        int left = maxDepth(root.left);
        int right = maxDepth(root.right);
        int max = left;
        if(right > max) max = right;
        return 1 + max;
    }
}
```



### [111. 二叉树的最小深度](https://leetcode-cn.com/problems/minimum-depth-of-binary-tree/)

给定一个二叉树，找出其最小深度。

最小深度是从根节点到最近叶子节点的最短路径上的节点数量。

说明: 叶子节点是指没有子节点的节点。

> 示例:
>
> 给定二叉树 [3,9,20,null,null,15,7],
    3
   / \
  9  20
    /  \
   15   7
返回它的最小深度  2.

代码实现：

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
    public int minDepth(TreeNode root) {
        if(root==null) return 0;

        int left = minDepth(root.left);
        int right = minDepth(root.right);
        if(left == 0 && right == 0) return 1;
        if(left == 0 && right != 0) return right + 1;  // 求的是到叶子节点的深度，不是层数  
        if(left != 0 && right == 0) return left + 1;

        if(left < right){
            return left + 1;
        }else{
            return right + 1;
        }
    }
}
```

### 226. 翻转二叉树

翻转一棵二叉树。

示例：

> 输入：
>   4
   /   \
  2     7
 / \   / \
1   3 6   9
输出：
    4
   /   \
  7     2
 / \   / \
9   6 3   1

谷歌翻车的面试题 ： https://twitter.com/mxcl/status/608682016205344768

代码：

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
    public TreeNode invertTree(TreeNode root) {
        if(root==null) return null;
        TreeNode left = root.left;
        root.left = root.right;
        root.right = left;
        invertTree(root.left);
        invertTree(root.right);
        return root;
    }
}
```

### [110. 平衡二叉树](https://leetcode-cn.com/problems/balanced-binary-tree/)

### [112. 路径总和](https://leetcode-cn.com/problems/path-sum/)



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
    public boolean hasPathSum(TreeNode root, int sum) {
        return hasPathSum(root,sum,0);
    }

    public boolean hasPathSum(TreeNode root, int sum, int currentSum){
        if(root == null) return false;
        if( root.left == null && root.right == null){
            if(currentSum + root.val == sum) {
                return true;
            }else{
                return false;
            }
        }

        if(hasPathSum(root.left, sum, currentSum+root.val)){
            return true;
        }
        if(hasPathSum(root.right, sum, currentSum+root.val)){
            return true;
        }
        return false;
    }
}
```



### [113. 路径总和 II](https://leetcode-cn.com/problems/path-sum-ii/)

给定一个二叉树和一个目标和，找到所有从根节点到叶子节点路径总和等于给定目标和的路径。

说明: 叶子节点是指没有子节点的节点。

示例:

> 给定如下二叉树，以及目标和 sum = 22，
              5
             / \
            4   8
           /   / \
          11  13  4
         /  \    / \
        7    2  5   1
返回:

     [
        [5,4,11,2],
        [5,8,4,5]
     ]

代码实现：

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
    public List<List<Integer>> pathSum(TreeNode root, int sum) {
        List<List<Integer>> result = new ArrayList<>();
        List<Integer> path = new ArrayList<>();
        findPath(root,sum,path,0,result);
        return result;
    }

    public void findPath(TreeNode root, int sum,List<Integer> path,int total,List<List<Integer>> result){
        if(root == null) return;
        path.add(root.val);
        total += root.val;
        if(root.left == null && root.right == null){
            List<Integer> path2 = new ArrayList<>();
            path2.addAll(path);
            if(total == sum) result.add(path2);    
            path.remove(path.size()-1); 
            return;
        }
        if(root.left != null) findPath(root.left,sum,path,total,result);     
        if(root.right != null) findPath(root.right,sum,path,total,result);
        path.remove(path.size()-1);
    }
}
```

### [129. 求根到叶子节点数字之和](https://leetcode-cn.com/problems/sum-root-to-leaf-numbers/)

给定一个二叉树，它的每个结点都存放一个 0-9 的数字，每条从根到叶子节点的路径都代表一个数字。

例如，从根到叶子节点路径 1->2->3 代表数字 123。

计算从根到叶子节点生成的所有数字之和。

说明: 叶子节点是指没有子节点的节点。

示例 1:

> 输入: [1,2,3]
>     1
>    / \
>   2   3
> 输出: 25
> 解释:
> 从根到叶子节点路径 1->2 代表数字 12.
> 从根到叶子节点路径 1->3 代表数字 13.
> 因此，数字总和 = 12 + 13 = 25.
> 示例 2:
>
> 输入: [4,9,0,5,1]
>     4
>    / \
>   9   0
>  / \
> 5   1
> 输出: 1026
> 解释:
> 从根到叶子节点路径 4->9->5 代表数字 495.
> 从根到叶子节点路径 4->9->1 代表数字 491.
> 从根到叶子节点路径 4->0 代表数字 40.
> 因此，数字总和 = 495 + 491 + 40 = 1026.

代码实现(0ms)：

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
    public int sumNumbers(TreeNode root) {
        return compute(root,0);
    }

    public int compute(TreeNode root,int currentSum) {
        if(root == null) return 0;
        currentSum = currentSum * 10 + root.val;
        if(root.left == null && root.right == null) return currentSum;
        int left = 0;
        int right = 0;
        if(root.left != null) left = compute(root.left,currentSum);
        if(root.right != null) right = compute(root.right,currentSum);
        return left + right;
    }
}
```

### [437. 路径总和 III](https://leetcode-cn.com/problems/path-sum-iii/)

给定一个二叉树，它的每个结点都存放着一个整数值。找出路径和等于给定数值的路径总数。路径不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点）。二叉树不超过1000个节点，且节点数值范围是 [-1000000,1000000] 的整数。

示例：

> root = [10,5,-3,3,2,null,11,3,-2,null,1], sum = 8
      10
     /  \
    5   -3
   / \    \
  3   2   11
 / \   \
3  -2   1
返回 3。和等于 8 的路径有:

1.  5 -> 3
2.  5 -> 2 -> 1
     3.  -3 -> 11

代码实现：

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
    public int pathSum(TreeNode root, int sum) {
        if( root == null)  return 0;
        int found = 0;
        found = getSumStartAtNode(root,sum,0);
        int left = pathSum(root.left,sum);
        int right = pathSum(root.right,sum);
        return found+left+right;
    }

    public int getSumStartAtNode(TreeNode root, int sum, int currentSum){
        if(root == null)  return 0;
        int found = 0;
        currentSum += root.val;
        if(currentSum == sum) found = 1;
        int left = getSumStartAtNode(root.left,sum, currentSum);
        int right = getSumStartAtNode(root.right,sum,currentSum);
        return found + left + right;
    }
}
```



### [235. 二叉搜索树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-search-tree/)

代码实现：

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
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
         if(root == null) return null;
         if(root.val > p.val && root.val > q.val){
             return lowestCommonAncestor(root.left, p,q);
         }else if(root.val < p.val && root.val < q.val){
             return lowestCommonAncestor(root.right,p,q);
         }else {
             return root;
         }
        
    }
}
```



### [98. 验证二叉搜索树](https://leetcode-cn.com/problems/validate-binary-search-tree/)

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
    public boolean isValidBST(TreeNode root) {
        if(root == null) return true;
        boolean left = isValidBSTByNode(root.left,root.val,true);
        boolean right = isValidBSTByNode(root.right,root.val,false);
        if(!left||!right) return false;
        left = isValidBST(root.left); // 判断root.left是不是二叉搜索树
        right = isValidBST(root.right); // 判断root.right是不是二叉搜索树
        return left && right;
    }
    // 以root节点为根的树的所有值是否都小于/大于sum
    public boolean isValidBSTByNode(TreeNode root,int num, boolean left){
        if(root == null) return true;
        if(left){
            if(root.val >= num) return false;
        }else{
            if(root.val <= num) return false;
        }    
        boolean leftResult = isValidBSTByNode(root.left,num,left);
        boolean rightResult = isValidBSTByNode(root.right,num,left);
        return leftResult&&rightResult;
    }
}
```

### [230. 二叉搜索树中第K小的元素](https://leetcode-cn.com/problems/kth-smallest-element-in-a-bst/)

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    int count = 0;
    public int kthSmallest(TreeNode root, int k) {
        if(root == null) return -1;
        int left = kthSmallest(root.left,k);
        if(left != -1) return left;
        count++; // 中序遍历
        if(count == k) return root.val;
        return kthSmallest(root.right,k);
    }
}
```

## 递归

### [17. 电话号码的字母组合](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/)

```java
class Solution {
    Map<Character,Character[]> map = new HashMap<>();
    {
        map.put('2',new Character[]{'a','b','c'});
        map.put('3',new Character[]{'d','e','f'});
        map.put('4',new Character[]{'g','h','i'});
        map.put('5',new Character[]{'j','k','l'});
        map.put('6',new Character[]{'m','n','o'});
        map.put('7',new Character[]{'p','q','r','s'});
        map.put('8',new Character[]{'t','u','v'});
        map.put('9',new Character[]{'w','x','y','z'});

    } 
    public List<String> letterCombinations(String digits) {
        List<String> result = new ArrayList<String>();
        findString(digits,0,"",result);
        return result;
    }
    void findString(String digits,int digIndex,String str, List<String> result ){
        if(digits == null) return;
        if(digits.length() <= digIndex) return;
        char c = digits.charAt(digIndex);
        Character[] cs = this.map.get(c);
        for(int i=0; i<cs.length; i++){
            String curr = str + cs[i];
            if(digits.length() <= digIndex+1){
                result.add(curr);
            }else{
                findString(digits, digIndex+1, curr, result);
            }        
        }
    }
}
```

## 排列组合

### [46. 全排列](https://leetcode-cn.com/problems/permutations/)

```java
class Solution {
    List<List<Integer>> result = new ArrayList<List<Integer>>();
    public List<List<Integer>> permute(int[] nums) {
        List<Integer> input = new ArrayList<>();
        for(int i=0; i<nums.length;i++){
            input.add(nums[i]);
        }
        List<Integer> curr = new ArrayList<>();
        permute(input,curr);
        return this.result;
    }

    public void permute(List<Integer> nums, List<Integer> seq){
        int size = nums.size();
        if(nums == null || size==0) return;
        
        for(int i=0; i<size;i++){       
            // 回溯：把nums,seq恢复  
            List<Integer> list = new ArrayList<>();
            list.addAll(nums);
            List<Integer> cur = new ArrayList<Integer>();
            cur.addAll(seq);
            cur.add(nums.get(i));
            list.remove(i);
            if(list.size()==0){
                result.add(cur);
            }else{
                permute(list, cur);
            }
        }
    }
}
```

### [77. 组合](https://leetcode-cn.com/problems/combinations/)

```java
class Solution {
    List<List<Integer>> result = new ArrayList<List<Integer>>();
    public List<List<Integer>> combine(int n, int k) {
        List<Integer> path = new ArrayList<Integer>();
        combine(1,n,k,path);
        return result;
    }

    public void combine(int start,int n, int k,List<Integer> path){
        if(n == 0 || k == 0) return;
        for(int i = 0; i<n; i++){
            List<Integer> list = new ArrayList<>();
            list.addAll(path);
            list.add(start+i);
            if(list.size() == k){
                this.result.add(list);
            }else if(k-list.size()>n-1-i){ // 剪枝：剩余元素不足，不需要再尝试遍历
                continue;
            }else{
                combine(start+i+1,n-1-i,k,list);
            }
        }
    }
}
```

### [39. 组合总和](https://leetcode-cn.com/problems/combination-sum/)

```java
class Solution {
    List<List<Integer>> result = new ArrayList<List<Integer>>();
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        List<Integer> candidate = new ArrayList<Integer>();
        Arrays.sort(candidates);  // 先排下序
        findResult(candidates,0,candidate,0,target);
        return result;
    }

    public void findResult(int[] candidates,int index,List<Integer> candidate,int cur, int target){
        if(candidates == null) return;

        for(int i=index; i<candidates.length; i++){
            int total = cur + candidates[i];
            if(total > target) continue; // 剪枝
            List<Integer> list = new ArrayList<>();
            list.addAll(candidate);
            list.add(candidates[i]);
            if(total == target){
                result.add(list);
                return;
            }
            findResult(candidates,i,list,total,target);
        }
    }
}
```

### [40. 组合总和 II](https://leetcode-cn.com/problems/combination-sum-ii/)

```java
class Solution {
    List<List<Integer>> result = new ArrayList<List<Integer>>();
    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        List<Integer> candidate = new ArrayList<>();
        Arrays.sort(candidates);
        getResult(candidates,0,candidate,0,target);
        return result;
    }

    void getResult(int[] candidates,int index, List<Integer> candidate, int cur, int target){
        for(int i=index; i<candidates.length; i++){
            if(i>index&&candidates[i] == candidates[i-1]) continue; // 去重:本层递归不能使用相同元素
            int total = cur + candidates[i];
            if(total > target) return;
            List<Integer> list = new ArrayList<>();
            list.addAll(candidate);
            list.add(candidates[i]);
            if(total == target){
                result.add(list);
            }else{
                getResult(candidates,i+1,list,total,target);
            }
        }
    }
}
```



### [216. 组合总和 III](https://leetcode-cn.com/problems/combination-sum-iii/)

```java
class Solution {
    List<List<Integer>> result = new ArrayList<List<Integer>>();
    public List<List<Integer>> combinationSum3(int k, int n) {
        List<Integer> list = new ArrayList<>();
        findResult(1,list,0,k,n);
        return result;
    }
    void findResult(int start,List<Integer> result,int cur,int k ,int n){
        for(int i=start; i<=9;i++){
            int total = cur + i;
            if(total > n) return;
            List list = new ArrayList<>();
            list.addAll(result);
            list.add(i);
            if( total == n ){
                if(list.size() == k) this.result.add(list);  
                return;
            }
            findResult(i+1,list,total,k,n);
        }
    }
}
```



#### [78. 子集](https://leetcode-cn.com/problems/subsets/)

```java
class Solution {
    List<List<Integer>> result = new ArrayList<List<Integer>>();
    public List<List<Integer>> subsets(int[] nums) {
        for(int i=0; i<=nums.length; i++){
            List<Integer> list = new ArrayList<Integer>();
            findN(nums,0,i,list);
        }
        return result;
    }

    // 找到元素个数为n的组合
    public void findN(int[] nums, int start ,int n, List<Integer> result){
        if(n == 0){
            this.result.add(result);
            return;
        }

        for(int i=start; i<nums.length;i++){
            List<Integer> list = new ArrayList<Integer>();
            list.addAll(result);
            list.add(nums[i]);
            if(list.size() == n){
                this.result.add(list);
            }else if(n-list.size()>nums.length-start-1){
                break; //剪枝：剩余元素不足，不需要递归了
            }else{
                findN(nums,i+1,n,list);
            }
        }
    }
}
```



### [90. 子集 II](https://leetcode-cn.com/problems/subsets-ii/)

```java
class Solution {
    List<List<Integer>> result = new ArrayList<List<Integer>>();
    public List<List<Integer>> subsetsWithDup(int[] nums) {
        Arrays.sort(nums);
        for(int i=0; i<=nums.length; i++){
            List<Integer> list = new ArrayList<Integer>();
            findN(nums,0,i,list);
        }
        return result;
    }

    // 找到元素个数为n的组合
    public void findN(int[] nums, int start ,int n, List<Integer> result){
        if(n == 0){
            this.result.add(result);
            return;
        }

        for(int i=start; i<nums.length;i++){
            if(i>start && nums[i] == nums[i-1]) continue; // 去重
            List<Integer> list = new ArrayList<Integer>();
            list.addAll(result);
            list.add(nums[i]);
            if(list.size() == n){
                this.result.add(list);
            }else if(n-list.size()>nums.length-start-1){
                break; //剪枝：剩余元素不足，不需要递归了
            }else{
                findN(nums,i+1,n,list);
            }
        }
    }
}
```



## 计算x的n次方的优化算法

普通算法：x连乘n次，算法复杂度O(n)
可以用递归的方式，pow(x, n/2)，算法复杂度O(logn)

## f(n)=f(n-1)+f(n-1)的时间复杂度
O(n^2)： 2^0 + 2^1 + 2^2 +2^3 + 2^4 +...2^n