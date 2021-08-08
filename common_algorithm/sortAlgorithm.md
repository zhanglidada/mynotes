#经典排序算法
##一、算法分类：
![1](/assets/1_ego3e5zrf.jpg)
**常见的排序算法可以分成两类：**
###1.1非线性时间的比较类排序：
通过比较来决定元素之间的相对次序，由于比较的时间复杂度最低也是`O(NlogN)`,所以称之为非线性时间复杂度的排序算法。

###1.2线性时间的非比较类排序：
不通过比较来决定元素间的相对次序，它可以突破基于比较排序的时间下界，达到线性时间运行，因此称为线性时间非比较类排序。
![11](/assets/11_gtu28injw.png)

##二、比较类排序：

###2.1 交换排序：
####2.1.1 冒泡排序：
时间复杂度`O(n^2)`，空间复杂度`O(1)`。通过循环遍历数组，每次将当前未排序数组重的最大值类似于冒泡一样交换到数组的结尾。
```
class solution {
 public:
  void Bubble(vector<int>& nums) {
    // 外层控制循环次数，其实外层最后一次循环已经是有序的，所以-1次
    for (int i = 0; i < nums.size() - 1; i++) {
      for (int j = 0; j < nums.size() - i - 1; j++) {
        if (nums[j] > nums[j + 1])
          swap(nums[j], nums[j + 1]);
      }
    }
    return;
  }
};
```
![bubble](/assets/bubble.gif)

####2.1.2 快速排序：
通过一次排序将待排序的记录分成两个部分，其中一个部分的数值全部比另一部分的数值小；则可分别对这两部分记录继续进行排序，以达到整个序列有序。时间复杂度`O(NlogN)`最好、最差`O(N^2)`，空间复杂度`O(lonN)`,因为递归的栈空间为`O(logN)`深度。
```
  // 此处采用随机打乱的方式，这样每次选取的都是随机打乱后的数组元素作为基准进行排序(随机打乱的数组能保证不会出现最差的性能)
  void QuickSort(vector<int>& nums) {
    QuickSort(nums, 0, nums.size() -1);
  }
  void QuickSort(vector<int>& nums, int st, int en) {
    // 说明此时
    if (st >= en)  return;
    int target = nums[st];
    int i = st, j = en + 1;
    while (i < j) {
      while (nums[++i] < target) {
        if (i > j)  break;
      }
      // 用于保证位置的相对一致
      while (nums[--j] >= target) {
        if (i > j)  break;
      }
      // 在循环过程中只有当i < j时才会进行置换
      if (i < j) {
        swap(nums[i], nums[j]);
      }
    }
    // i >= j时退出最外层循环
    swap(nums[j], nums[st]);
    QuickSort(nums, st, j - 1);
    QuickSort(nums, j + 1, en);
    return;
  }
  void random_shuffle(vector<int>& nums) {
    random_device rd;
    mt19937 rng(rd());
    // 使用一个随机数产生器来打乱数组，推荐使用shuffle代替被删除的random_shuffle
    shuffle(nums.begin(), nums.end(), rng);
    return;
  }
```
![quickSort](/assets/quickSort.gif)

###2.2 插入排序：
####2.2.1 简单插入排序
类似于扑克牌中的理牌，每次针对排序好的序列从后往前查找并插入到对应位置。时间复杂度`O(N^2)`，空间复杂度`O(1)`。
```
void insertSort(vector<int>& nums) {
  int len = nums.size();
  for (int i = 1; i < len; i++) {
    int key = nums[i];
    int j = 0;
    for (j = i - 1; j >= 0; j--) {
      if (nums[j] > key)  nums[j + 1] = nums[j];
      else break;
    }
    nums[j + 1] = key;
  }
  return;
}
```
![insert](/assets/insert_7wrsuwx5v.gif)
####2.2.2 希尔排序
希尔排序是插入排序的一种高效实现方式，也叫做缩小增量排序。先将整个待排序列分为若干个子序列并按照步长并进行直接插入排序，待整个序列基本有序时再对整个序列进行直接插入排序。平均时间复杂度`O(N^1.3)`,空间复杂度`O(1)`。
**问题：**
关于增量序列的取法？
一般情况是取`length / 2`为计算步伐的算法，此处gap采用`gap = gap * 3 + 1`；但是最终步伐一定为1。
```
// 希尔排序
void shellSort(vector<int>& nums) {
  int len = nums.size();
  int gap = 1;
  // 先计算得到最大的步长
  while (gap < len / 3) {
    gap = gap * 3 + 1;
  }
  // 从当前步长gap开始，到下一个步长2gap为止，分成了gap组；并在每一组中从前往后进行插入排序
  while (gap >= 1) {
    for (int i = gap; i < len; i++) {
      // j从i开始保证了可以最少和第一个元素比较一次,每次循环中比较当前元素和它前一个步长位置元素
      // 类似于将较小元素的值冒泡到前面
      for (int j = i; j >= gap && (nums[j] < nums[j - gap]); j -= gap) {
        swap(nums[j], nums[j - gap]);
      }
    }
    gap /= 3;
  }
  return;
}
```
希尔排序按照步长，每次将待排序数组分割为子数组，并且在子数组中进行插入排序；每次递减步长并重新分组并排序，直到最后步长为1时进行直接插入排序。
![shellSort](/assets/shellSort.gif)
###2.3 选择排序：
####2.3.1 简单选择排序
冒泡排序是基于相邻元素的比较和交换；选择排序则是每次从未排序序列中找到最小（大）的元素，并放置到未排序序列的开始位置。
```
void selectionSort(vector<int>& nums) {
  int len = nums.size();
  // 由于在找到未排序元素中最小元素之后需要进行交换，所以需要保存下标
  int min = 0;
  // 每次排序最小元素的插入位置
  for (int i = 0; i < len; i++) {
    min = i;  // 每次默认最小的元素是开始位置的元素
    for (int j = i; j < len; j++) {
      if (nums[j] < nums[min])  min = j;
    }
    swap(nums[min], nums[i]);
  }
  return;
}
```
![selectionSort](/assets/selectionSort_o4vaoexf6.gif)
####2.3.2 堆排序
堆的结构是一个完全二叉树；同时需要满足：子节点的关键值总是大于(小于)根节点的关键值。

**基本思想：**
1.首先将待排序的数组构造成一个大根堆，此时堆顶元素为数组中的最大值。(<font color = red>升序排列为大根堆，降序排列为小根堆</font>)
2.将堆顶元素与堆末尾的元素进行交换，即固定当前堆中的最大值在末尾；此时剩余的 n-1 个堆中的元素为待排序元素。
3.将堆顶元素与左右节点中的较大值进行比较；如果大于较大值则不用操作，如果小于较大值则与其进行交换。如此反复执行2，3步骤即可。
```
// 堆排序，利用一维数组进行存储
void heapSort (vector<int>& arr) {
  // 首先将数组调整为大顶堆
  heapInsert(arr);
  int endIndex = arr.size() - 1;
  while (endIndex > 0) {
    swap(arr[0], arr[endIndex]);
    endIndex --;
    heapify(arr, endIndex);
  }
  return;
}
// 给定一个数组，在依次插入过程中将其调整为大顶堆
void heapInsert(vector<int>& arr) {
  int len = arr.size();
  if (len == 1)  return;
  for (int i = 1; i < len; i++) {
    int currentIndex = i;
    int fatherIndex = (currentIndex - 1) / 2;
    while (arr[currentIndex] > arr[fatherIndex] && currentIndex > 0) {
      swap(arr[currentIndex], arr[fatherIndex]);
      currentIndex = fatherIndex;
      fatherIndex = (currentIndex - 1) / 2;
    }
  }
  return;
}
void heapify (vector<int>& arr, int end) {
  int currentIndex = 0;
  int leftIndex = currentIndex * 2 + 1;
  int rightIndex = currentIndex * 2 + 2;
  int maxValIndex = 0;
  while (leftIndex <= end) {
    if (rightIndex <= end)  maxValIndex = arr[leftIndex] > arr[rightIndex] ? leftIndex : rightIndex;
    else  maxValIndex = leftIndex;
    if (arr[currentIndex] < arr[maxValIndex])  swap(arr[currentIndex], arr[maxValIndex]);
    else  break;
    currentIndex = maxValIndex;
    leftIndex = currentIndex * 2 + 1;
    rightIndex = currentIndex * 2 + 2;
  }
  return;
}
```
![heap](/assets/heap.gif)

###2.4 归并排序：
先使每个子序列有序，然后依次将已经有序的子序列合并，最终得到完全有序的序列。若每次将2个有序序列合并称为2-路归并排序；若每次将k个有序序列合并称为k-路归并排序。归并排序是采用分治法的一个典型应用，且是一个稳定的排序算法，速度仅次于快速排序；但是由于归并排序是`out-place-sort`，所以相对于外部排序，需要很多额外的空间。
#### 2.4.1 二路归并排序
平均时间复杂度`O(NlogN)`，最佳时间复杂度`O(N)`；空间复杂度`O(N)`。
**基本思想：**
分解：将n个元素分解为两个分别含n/2个元素的子序列
解决：将两个子序列分别进行排序
合并：合并两个有序的子序列。

**实现逻辑：**
可以采取迭代或者递归的方式。

1）迭代法
① 申请空间，使其大小为两个已经排序序列之和，该空间用来存放合并后的序列
② 设定两个指针，最初位置分别为两个已经排序序列的起始位置
③ 比较两个指针所指向的元素，选择相对小的元素放入到合并空间，并移动指针到下一位置
④ 重复步骤③直到某一指针到达序列尾
⑤ 将另一序列剩下的所有元素直接复制到合并序列尾
```
// 2路归并排序，采用迭代的方式(自底向上)
void twoPathMergeSort(vector<int>& arr) {
  int len = arr.size();
  vector<int> addition(len, 0);
  // seg表示每次迭代的子数组长度
  for (int seg = 1; seg < len; seg *= 2) {
    for (int start = 0; start < len; start += seg * 2) {
      int low = start, mid = min(start + seg - 1, len - 1), high = min(start + 2 * seg - 1, len - 1);
      merge(arr, addition, low, mid, high);
    }
  }
}
void merge(vector<int>& arr, vector<int>& addition, int low, int mid, int high) {
  // 此时归并到了边界
  if (mid >= high)  return;
  for (int k = low; k <= high; k++)
    addition[k] = arr[k];
  int k = low, i = low, j = mid + 1;
  while (i <= mid && j <= high)
    arr[k++] = addition[i] < addition[j] ? addition[i++] : addition[j++];
  while (i <= mid)  arr[k++] = addition[i++];
  while (j <= high)  arr[k++] = addition[j++];
  return;
}
```
2）递归法
① 将序列每相邻两个数字进行归并操作，形成floor(n/2)个序列，排序后每个序列包含两个元素
② 将上述序列再次归并，形成floor(n/4)个序列，每个序列包含四个元素
③ 重复步骤②，直到所有元素排序完毕
```
// 采用递归的方式进行(自顶向下)
void twoPathMergeSort_recur(vector<int>& arr) {
  vector<int> addition(arr.size(), 0);
  sort(arr, addition, 0, arr.size() - 1);
  return;
}
void sort(vector<int>& arr, vector<int>& addition, int low, int high) {
  if (low >= high)  return;
  int mid = (low + high) / 2;
  sort(arr, addition, low, mid);
  sort(arr, addition, mid + 1, high);
  merge(arr, addition, low, mid, high);
  return;
}
void merge(vector<int>& arr, vector<int>& addition, int low, int mid, int high) {
  // 此时归并到了边界
  if (mid >= high)  return;
  for (int k = low; k <= high; k++)
    addition[k] = arr[k];
  int k = low, i = low, j = mid + 1;
  while (i <= mid && j <= high)
    arr[k++] = addition[i] < addition[j] ? addition[i++] : addition[j++];
  while (i <= mid)  arr[k++] = addition[i++];
  while (j <= high)  arr[k++] = addition[j++];
  return;
}
```
![twoPathMergeSort](/assets/twoPathMergeSort.webp)

#### 2.4.2 多路归并排序


#三、非比较类排序：

###3.1 桶排序

###3.2 基数排序

###3.3 计数排序：
