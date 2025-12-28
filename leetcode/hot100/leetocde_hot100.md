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

**说明：你不能倾斜容器。**

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

**解题思路：**
使用双指针法。
1）相同情况下两边距离越远越好
2）区域受限于最短的那边

**双指针正确的证明：**
1）双指针代表了什么？

双指针代表的是 可以作为容器边界的所有位置的范围。在一开始，双指针指向数组的左右边界，表示 数组中所有的位置都可以作为容器的边界，因为我们还没有进行过任何尝试。在这之后，我们每次将数字较小的那个指针往另一个指针的方向移动一个位置，就表示我们认为 这个指针不可能再作为容器的边界了。

为什么对应的数字较小的那个指针不可能再作为容器的边界了？这里我们定量地进行证明。

考虑第一步，假设当前左指针和右指针指向的数分别为 x 和 y，不失一般性，我们假设 x≤y。同时，两个指针之间的距离为 t。那么，它们组成的容器的容量为：

min(x,y)  * t = x * t

可以断定，如果我们保持左指针的位置不变，那么无论右指针在哪里，这个容器的容量都不会超过 x * t 。注意这里右指针只能向左移动，因为此时我们考虑的是第一步，此时指针还指向数组的左右边界。

我们任意向左移动右指针，指向的数为 y1​，两个指针之间的距离为 t1​，那么显然有 t1​<t，并且min(x,y1​)≤min(x,y)

- 如果 y1​≤y，那么 min(x,y1​)≤min(x,y)；
- 如果 y1​>y，那么 min(x,y1​)=x=min(x,y)。

因此有：min(x,yt​)∗t1​<min(x,y)∗t

==即无论我们怎么移动右指针，得到的容器的容量都小于移动前容器的容量。也就是说这个左指针对应的数不会作为容器的边界了，那么我们就可以丢弃这个位置，将左指针向右移动一个位置，此时新的左指针于原先的右指针之间的左右位置，才可能会作为容器的边界==。

这样以来，我们将问题的规模减小了 1，被我们丢弃的那个位置就相当于消失了。此时的左右指针，就指向了一个新的、规模减少了的问题的数组的左右边界，因此我们可以继续像之前考虑第一步那样考虑这个问题：

求出当前双指针对应的容器的容量；

对应数字较小的那个指针以后不可能作为容器的边界了，将其丢弃，并移动对应的指针。

最后的答案是什么？

答案就是我们每次以双指针为左右边界（也就是「数组」的左右边界）计算出的容量中的最大

```
class Solution {

public:

int maxArea(vector<int>& height) {

int left = 0;

int right = height.size() - 1;

int curr_area = 0;

int max_area = 0;

  

if (height.empty())

return 0;

  

while (left < right) {

curr_area = (right - left) * min(height[left], height[right]);

max_area = max(curr_area, max_area);

  

if (height[left] < height[right])

left ++;

else

right --;

}

  

return max_area;

}

};
```


[15. 三数之和](https://leetcode.cn/problems/3sum/)

给你一个整数数组 `nums` ，判断是否存在三元组 `[nums[i], nums[j], nums[k]]` 满足 `i != j`、`i != k` 且 `j != k` ，同时还满足 `nums[i] + nums[j] + nums[k] == 0` 。请你返回所有和为 `0` 且不重复的三元组。

**注意：答案中不可以包含重复的三元组。**

**示例 1：**

输入：nums = [-1,0,1,2,-1,-4]
输出：[ [-1,-1,2],[-1,0,1] ]
解释：
nums[0] + nums[1] + nums[2] = (-1) + 0 + 1 = 0 。
nums[1] + nums[2] + nums[4] = 0 + 1 + (-1) = 0 。
nums[0] + nums[3] + nums[4] = (-1) + 2 + (-1) = 0 。
不同的三元组是 [-1,0,1] 和 [-1,-1,2] 。
注意，输出的顺序和三元组的顺序并不重要。

**示例 2：**

输入：nums = [0,1,1]
输出：[]
解释：唯一一个三元组，其和不为 0 。

**示例 3：**

输入：nums = [0,0,0]
输出：[ [0,0,0] ]
解释：仅有一个三元组，其和为 0 。

**提示：**

- `3 <= nums.length <= 3000`
- `-105 <= nums[i] <= 105`

==**解题思路：**==

需要**找到所有不重复且和为0的三元组**，「不重复」的本质是什么？我们保持三重循环的大框架不变，只需要保证：

第二重循环枚举到的元素不小于当前第一重循环枚举到的元素。
第三重循环枚举到的元素不小于当前第二重循环枚举到的元素。

也就是说，我们枚举的三元组 (a,b,c) 满足 a≤b≤c，保证了只有 (a,b,c) 这个顺序会被枚举到，而 (b,a,c)、(c,b,a) 等等这些不会，这样就减少了重复。要实现这一点，我们可以将数组中的元素从小到大进行排序，随后使用普通的三重循环就可以满足上面的要求。

同时，在每一层循环中，相邻两次枚举的元素也不能相同，否则也会造成重复。这里的意思就是每一层循环中也需要进行去重处理。

可以发现，如果我们固定了前两重循环枚举到的元素 a 和 b，那么只有唯一的 c 满足 a+b+c=0。当第二重循环往后枚举一个元素 b 时，由于 b′>b，那么满足 a+b′+c′= 0 的 c′ 一定有 c′< c，即 c′ 在数组中一定出现在 c 的左侧，也就是说，我们可以从小到大枚举 b，**同时**从大到小枚举 c。==此时我们就讲最内层的两次循环合并为一个双指针法。==


我们也可以换种思路，因为x + y + z = 0 等价于 x = -（y + z），如果我们将x看作一个明确的值，那么此问题就转化为求 y + z 这两个变量之和为一个特定值的双指针法。

```
class Solution {

public:

vector<vector<int>> threeSum(vector<int>& nums) {

  

// 最终返回的结果集中，每个序列一定是顺序递增的

vector<vector<int>> res;

int n = nums.size();

// 边界情况

if (nums.empty() || nums.size() == 1 || nums.size() == 2)

return res;

// 先将数组排序，确保无重复元素

sort(nums.begin(), nums.end());

  

// 第一重循环

for (int first = 0; first < n; first ++) {

  

// 第一个元素去重，第一遍循环时不满足去重条件

if (first != 0 && nums[first] == nums[first - 1])

continue;

  

int third = n - 1;

int target = -nums[first];

  

// 第二重循环，先固定 second 指针并移动 third 指针

for (int second = first + 1; second < n; second ++) {

  

// 第二重循环中去重,第一遍循环的时候不满足去重条件

if ((second != first + 1) && nums[second] == nums[second - 1])

continue;

  

while (second < third) {

if (nums[second] + nums[third] > target)

third --;

else

{

// 当前 second 只会有一个 third 与之匹配

if (nums[second] + nums[third] == target)

res.push_back(vector<int>({nums[first], nums[second], nums[third]}));

  

// 无论是找到，还是和小于 target，都退出当前循环

break;

}

}

}

}

  

return res;

}

};
```


[3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)

给定一个字符串 `s` ，请你找出其中不含有重复字符的 **最长 子串** 的长度。

**示例 1:**

**输入:** s = "abcabcbb"
**输出:** 3 
**解释:** 因为无重复字符的最长子串是 `"abc"`，所以其长度为 3。注意 "bca" 和 "cab" 也是正确答案。

**示例 2:**

**输入:** s = "bbbbb"
**输出:** 1
**解释:** 因为无重复字符的最长子串是 `"b"`，所以其长度为 1。

**示例 3:**

**输入:** s = "pwwkew"
**输出:** 3
**解释:** 因为无重复字符的最长子串是 `"wke"`，所以其长度为 3。
     请注意，你的答案必须是 **子串** 的长度，`"pwke"` 是一个子序列，_不是子串。

**提示：**

- `0 <= s.length <= 5 * 104`
- `s` 由英文字母、数字、符号和空格组成

**解题思路：**
首先想到，可以从第一个字符开始，查找当前字符开始的子串中，最长的无重复字符子串长度，并使用 hash 表登记当前子串中的字符。

**证明：**
如果我们依次递增地枚举子串的起始位置，那么子串的结束位置也是递增的！假设我们选择字符串中的第 k 个字符作为起始位置，并且得到了不包含重复字符的最长子串的结束位置为$r_{k}$，那么那么当我们选择第 k+1 个字符作为起始位置时，首先从 k+1 到​$r_{k}$位置的字符显然是不重复的，并且由于少了原本的第 k 个字符，我们可以尝试继续增大到​$r_{k}$后的位置，直到右侧出现了重复字符为止。


这样一来，我们就可以使用「滑动窗口」来解决这个问题了：

- 我们使用两个指针表示字符串中的某个子串（或窗口）的左右边界，其中左指针代表着上文中「枚举子串的起始位置」，而右指针即为上文中的​$r_{k}$

- 在每一步的操作中，我们会将左指针向右移动一格，表示 我们开始枚举下一个字符作为起始位置，然后我们可以不断地向右移动右指针，但需要保证这两个指针对应的子串中没有重复的字符。在移动结束后，这个子串就对应着 以左指针开始的，不包含重复字符的最长子串。我们记录下这个子串的长度；

- 在枚举结束后，我们找到的最长的子串的长度即为答案。

```
// 双层循环遍历

class Solution {

  public:

    int lengthOfLongestSubstring(string s) {

      int max_size = 0;

  

      if (s.empty())

        return 0;

  

      if (s.size() == 1)

        return 1;

  

      for (int i = 0; i < s.size(); i++) {

        unordered_set<int> hash_set;

        int curr_count = 0;

  

        for (int j = i; j < s.size(); j++) {

          if (!hash_set.count(s[j])) {

            hash_set.insert(s[j]);

            curr_count ++;

          }

          else

          {

            // 当前子串中存在重复元素，跳出循环

            break;

          }

        }

  

        max_size = max(curr_count, max_size);

      }

      return max_size;

    }

};

  

// 使用滑动窗口进行优化，时间复杂度为 2n

class Solution1 {

  public:

    int lengthOfLongestSubstring(string s) {

      // 左右指针，表示当前的窗口

      int left = 0, right = 0;

      int curr_count = 0;

      unordered_set<int> hash_set;

      int max_size = 1;

  

      if (s.empty())

        return 0;

  

      if (s.size() == 1)

        return 1;

  

      for (left = 0; left < s.size(); left ++) {

  

        while (right < s.size() && hash_set.count(s[right]) == 0) {

          hash_set.insert(s[right]);

  

          // 右指针继续向后移动

          right ++;

        }

  

        max_size = max(max_size, right - left);

  

        if (right == s.size())

          return max_size;

  

        // 将左指针指向的元素从hash_set中删除，左指针的下一个位置为滑动窗口的起始位置

        hash_set.erase(s[left]);

      }

      return max_size;

    }

};

  
  

/*

  滑动窗口进一步优化：

  

  startsubstr为每个可能的子串的开始位置

  hashchar中存放的是遍历过程中字符最近一次出现的位置

  针对滑动窗口的优化，对于重复出现的字符，如果大于startsubstr，则从startsubstr下一个开始进行滑动

*/

class Solution2 {

 public:

  int lengthOfLongestSubstring(string s) {

    unordered_map<char, int> hashchar;

    if (s.size() == 0)  return 0;

    int currentLen = 0, maxLen = 0, startsubstr = 0;

    for (int i = 0; i < s.size(); i++) {

      // 对一个子数组中第一次访问的元素进行插入

      if (hashchar.count(s[i]) == 0)  hashchar[s[i]] = i, currentLen++;

      else {

        // 子串中出现第一个重复的字符

        maxLen = max(maxLen, currentLen);

        startsubstr = max(hashchar[s[i]], startsubstr);

        currentLen = i - startsubstr;

        hashchar[s[i]] = i;

      }

    }

    maxLen = max(maxLen, currentLen);

    return maxLen;

  }

};
```


[438. 找到字符串中所有字母异位词](https://leetcode.cn/problems/find-all-anagrams-in-a-string/)

给定两个字符串 `s` 和 `p`，找到 `s` 中所有 `p` 的 **异位词** 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。

**示例 1:**

**输入:** s = "cbaebabacd", p = "abc"
**输出:** [0,6]
**解释:**
起始索引等于 0 的子串是 "cba", 它是 "abc" 的异位词。
起始索引等于 6 的子串是 "bac", 它是 "abc" 的异位词。

 **示例 2:**

**输入:** s = "abab", p = "ab"
**输出:** [0,1,2]
**解释:**
起始索引等于 0 的子串是 "ab", 它是 "ab" 的异位词。
起始索引等于 1 的子串是 "ba", 它是 "ab" 的异位词。
起始索引等于 2 的子串是 "ab", 它是 "ab" 的异位词。

**提示:**

- `1 <= s.length, p.length <= 3 * 104`
- `s` 和 `p` 仅包含小写字母

解题思路：

