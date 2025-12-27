[1. 两数之和](https://leetcode.cn/problems/two-sum/)
给定一个整数数组 `nums` 和一个整数目标值 `target`，请你在该数组中找出 **和为目标值** _`target`_  的那 **两个** 整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案，并且你不能使用两次相同的元素。

你可以按任意顺序返回答案。

解题思路：

首先想到的就是使用hash表

```
class Solution {

public:

    vector<int> twoSum(vector<int>& nums, int target) {

        unordered_map<int, int> hashmap;

        vector<int> ans;

        int res = 0;

        for (int i = 0; i < nums.size(); i++) {

            res = target - nums[i];

            if (hashmap.count(res) && hashmap[res] != i) {

                ans = vector<int>({hashmap[res], i});

                break;

            }
  
            hashmap.insert({nums[i], i});

        }

        return ans;

    }

};
```


  

[49. 字母异位词分组](https://leetcode.cn/problems/group-anagrams/)
给你一个字符串数组，请你将 字母异位词 组合在一起。可以按任意顺序返回结果列表。

**示例 1:**

**输入:** strs = ["eat", "tea", "tan", "ate", "nat", "bat"]

**输出:** [["bat"],["nat","tan"],["ate","eat","tea"]]

**解释：**

- 在 strs 中没有字符串可以通过重新排列来形成 `"bat"`。
- 字符串 `"nat"` 和 `"tan"` 是字母异位词，因为它们可以重新排列以形成彼此。
- 字符串 `"ate"` ，`"eat"` 和 `"tea"` 是字母异位词，因为它们可以重新排列以形成彼此。

**示例 2:**

**输入:** strs = [""]

**输出:** [[""]]

**示例 3:**

**输入:** strs = ["a"]

**输出:** [["a"]]

**提示：**

- `1 <= strs.length <= 104`
- `0 <= strs[i].length <= 100`
- `strs[i]` 仅包含小写字母


解题思路：

1）需要根据特征值进行归类的时候，我们会想到散列表

```
class Solution {

public:

  vector<vector<string>> groupAnagrams(vector<string>& strs) {

    unordered_map<string, vector<string>> map;

  

    for(auto str : strs) {

  

      // 不能修改原本字符串的值

      string key = str;

      sort(key.begin(), key.end());

  

      // 将原本串添加到map[key]中

      map[key].emplace_back(str);

    }

  

    vector<vector<string>> res;

  

    for (auto it = map.begin(); it != map.end(); it++) {

      res.emplace_back(it->second);

    }

  

    return res;

  }

};
```

[128. 最长连续序列](https://leetcode.cn/problems/longest-consecutive-sequence/)

给定一个未排序的整数数组 `nums` ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。

请你设计并实现时间复杂度为 `O(n)` 的算法解决此问题。

**示例 1：**

**输入：**nums = [100,4,200,1,3,2]
**输出：**4
**解释：**最长数字连续序列是 [1, 2, 3, 4]。它的长度为 4。

**示例 2：**

**输入：**nums = [0,3,7,2,5,8,4,6,0,1]
**输出：**9

**示例 3：**

**输入：**nums = [1,0,1,2]
**输出：**3

**提示：**

- `0 <= nums.length <= 105`
- `-109 <= nums[i] <= 109`

思路分析：
最开始想到的将数组排序，从前向后遍历，使用一个变量保存全局最大的连续个数。
从第一个下标开始记录，如果第二个和第一个连续就增加计数，否则从第三个开始重新统计连续的个数。

```
class Solution {

  public:

    int longestConsecutive(vector<int>& nums) {

      int curr_count = 1;

      int max_count = 1;

  

      // 空数组直接返回

      if (nums.empty())

        return 0;

  

      sort(nums.begin(), nums.end());

  

      for (auto i = 1; i < nums.size(); i++) {

        if (nums[i] == nums[i - 1] + 1)

          curr_count++;

        else if (nums[i] > nums[i - 1] + 1)

        {

          max_count = max(curr_count, max_count);

          curr_count = 1;

        }

      }

  

      // 循环结束后再比较一次，防止整个序列都顺序递增的情况

      max_count = max(max_count ,curr_count);

  

      return max_count;

    }

};
```


官方题解方法：使用hash表，性能差了很多
```
/**

 * @brief

 *  使用hash表进行查找

    1）将所有不重复的元素插入hash表中

    2）依次遍历每个元素，如果元素值+1的元素在hash表中也能找到，就将计数增加

  

  notes：

    性能还不如第一个方法，很烂

 */

class Solution1 {

  public:

    int longestConsecutive(vector<int>& nums) {

      int curr_count = 1;

      int max_count = 1;

      unordered_set<int> hash_set;

  

      // 空数组直接返回

      if (nums.empty())

        return 0;

  

      // 仅插入不重复元素

      for (auto num : nums) {

        if (hash_set.count(num) == 0) {

          hash_set.insert(num);

        }

      }

  

      // 对hash表中的每个元素，顺序查找其下一个是否存在，如果不存在就接着从hash_set中的下一个元素开始查找

      for (auto num : hash_set) {

  

        // 仅当前元素值的前一个值不存在时，才去找下一个值

        // 即当前元素为一个顺序递增序列的开始元素

        if (hash_set.count(num - 1) == 0) {

          int curr_num = num;

          curr_count = 1;

  

          while (hash_set.count(num + 1) != 0) {

            curr_count ++;

            num = num + 1;

          }

        }

  

        max_count = max(curr_count, max_count);

      }

  

      return max_count;

    }

};
```




[283. 移动零](https://leetcode.cn/problems/move-zeroes/)

给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的相对顺序。

**请注意** ，必须在不复制数组的情况下原地对数组进行操作。

**示例 1:**

**输入:** nums = `[0,1,0,3,12]`
**输出:** `[1,3,12,0,0]`

**示例 2:**

**输入:** nums = `[0]`
**输出:** `[0]`

**提示**:

- `1 <= nums.length <= 104`
- `-231 <= nums[i] <= 231 - 1`


解题思路：
因为题目要求不能进行数组的复制，只能原地对数据进行操作

最容易想到的就是类似于冒泡排序的思路，从最后一个元素开始遍历，如果为0就从当前位置开始将0依次和后面的元素进行交换。

这里可以使用一个尾部的指针记录结尾开始的0位置，后续移动的时候只需要比较到已经移动的0位置就可以了

```
class Solution {

  public:

    void moveZeroes(vector<int>& nums) {

      if (nums.empty())

        return;

  

      int start_pos = 0;

      int end_pos = nums.size() - 1;

      int curr_pos = end_pos;  // 从最后一个元素开始遍历

  

      while (curr_pos >= 0) {

        // 开始将此处的0移动到结尾处

        if (nums[curr_pos] == 0) {

          for (int j = curr_pos + 1; j <= end_pos; j++) {

            // 将当前0和后一个位置的值交换

            swap(&nums[j - 1], &nums[j]);

          }

  

          // 因为当前0已经移动到后一个位置，所以end_pos前移

          end_pos --;

        }

        // 继续查找前一个位置

        curr_pos --;

      }

  

    }

  

    void swap(int* a, int* b) {

      int tmp = *a;

      *a = *b;

      *b = tmp;

  

      return;

    }

};
```


官方给出的解题思路：
使用双指针，左指针指向当前已经处理好的序列的尾部，右指针指向待处理序列的头部。
右指针不断向右移动，每次右指针指向非零数，则将左右指针对应的数交换，同时左指针右移。

注意到以下性质：
左指针左边均为非零数；
右指针左边直到左指针处均为零。

因此每次交换，都是将左指针的零与右指针的非零数交换，且非零数的相对顺序并未改变。
```
class Solution1 {

  public:

    void moveZeroes(vector<int>& nums) {

        int n = nums.size();

        int left = 0;

        int right = 0;

  

        while (right < nums.size()) {

          // 右指针指向非0元素，需要将其和left所在元素交换

          if (nums[right]) {

            swap(nums[left], nums[right]);

  

            // 交换结束后left所在位置指向非0元素，此时需要后移，指向处理好序列的尾部

            left ++;

          }

  

          // 继续访问下一个元素

          right ++;

        }

    }

};
```



[11. 盛最多水的容器](https://leetcode.cn/problems/container-with-most-water/)

给定一个长度为 `n` 的整数数组 `height` 。有 `n` 条垂线，第 `i` 条线的两个端点是 `(i, 0)` 和 `(i, height[i])` 。

找出其中的两条线，使得它们与 `x` 轴共同构成的容器可以容纳最多的水。

返回容器可以储存的最大水量。

**说明：**你不能倾斜容器。

**示例 1：**

![](https://aliyun-lc-upload.oss-cn-hangzhou.aliyuncs.com/aliyun-lc-upload/uploads/2018/07/25/question_11.jpg)

**输入：**[1,8,6,2,5,4,8,3,7]
**输出：**49 
**解释：**图中垂直线代表输入数组 [1,8,6,2,5,4,8,3,7]。在此情况下，容器能够容纳水（表示为蓝色部分）的最大值为 49。

**示例 2：**

**输入：**height = [1,1]
**输出：**1

**提示：**

- `n == height.length`
- `2 <= n <= 105`
- `0 <= height[i] <= 104`

解题思路：
