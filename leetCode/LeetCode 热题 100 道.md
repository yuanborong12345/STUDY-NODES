# LeetCode 热题 100 道

## 一、哈希

### 1.1 [两数之和](https://leetcode.cn/problems/two-sum/description/?envType=study-plan-v2&amp;envId=top-100-liked)

给定一个整数数组 `nums` 和一个整数目标值 `target`，请你在该数组中找出 **和为目标值** *`target`* 的那 **两个** 整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案，并且你不能使用两次相同的元素。

你可以按任意顺序返回答案。

**示例 ：**

```
输入：nums = [2,7,11,15], target = 9
输出：[0,1]
解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。
```

**答案：**

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        //保存（值，下标）
        Map<Integer,Integer> map = new HashMap<Integer,Integer>();
        //遍历数组
        for(int i = 0; i < nums.length; i++){
            //如果包含差值，那么找到该对子
            if(map.containsKey(target - nums[i])){
                return new int[]{map.get(target - nums[i]), i};
            }
            //不包含，插入（值，下标）
            map.put(nums[i],i);
        }
        return new int[0];
    }
}
```

### 1.2 [字母异位词分组](https://leetcode.cn/problems/group-anagrams/description/?envType=study-plan-v2&envId=top-100-liked)

给你一个字符串数组，请你将 字母异位词 组合在一起。可以按任意顺序返回结果列表。

**示例 1:**

```
输入: strs = ["eat", "tea", "tan", "ate", "nat", "bat"]
输出: [["bat"],["nat","tan"],["ate","eat","tea"]]

解释：

- 在 strs 中没有字符串可以通过重新排列来形成 `"bat"`。
- 字符串 `"nat"` 和 `"tan"` 是字母异位词，因为它们可以重新排列以形成彼此。
- 字符串 `"ate"` ，`"eat"` 和 `"tea"` 是字母异位词，因为它们可以重新排列以形成彼此。
```

**答案：**

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        HashMap<String,List<String>> map = new HashMap<>();
        for(String str : strs){
            char[] charArray = str.toCharArray();
            //如果两个字符串是异位词，排序后作为相同的key
            Arrays.sort(charArray);
            String key = new String(charArray);
            List<String> ls = map.getOrDefault(key,new ArrayList<String>());
            ls.add(str);
            map.put(key,ls);
        }
        return new ArrayList<List<String>>(map.values());
    }
}
```

