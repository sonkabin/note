# 第K大的数/第K小的数

[从无序数组中寻找第K大的值](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

## 解法1：利用快排的思想和partition函数, 近似O(n)

```java
public int findKthLargest(int[] nums, int k) {
    int n = nums.length;
    // 找第K大的数，就是找第N-K小的数。那么，如果找第K小的数，传入K-1即可。因为数组下标从0开始，以找第1个最大的数为例，也就是找下标为N-1位置的数；以找第1个最小的数为例，也就是找下标为1-1=0的位置的数
    return search(nums, n - k, 0, n - 1);
}
// 找第K小的数
private int search(int[] nums, int k, int lo, int hi){
    int j = partition(nums, lo, hi);
    if(j == k) return nums[j];
    else if(j < k) return search(nums, k, j+1, hi);
    else    return search(nums, k, lo, j-1);
}
private int partition(int[] nums, int lo, int hi){
    // 选取哨兵更好的方式是随机选取，而不是只把第一个当哨兵，因为对于已经排序的数组来说，会使快排退化成O(n^2)的算法
    /* int randomIndex = lo + random.nextInt(hi - lo);
    swap(nums, lo, randomIndex);
    */
    int pivot = nums[lo], i = lo, j = hi + 1;
    while(true){
        while(nums[--j] > pivot);
        // 若把--j与++i操作放在while循环体里，则需要注意nums[i/j]与pivot的比较其中一个需要取到等号，否则给一个[3,3,3]的数组，因为i与j都只能在循环体里改变，但永远无法到达循环体，因此会陷入死循环。关于这个的详细解释请看 Leetcode总结.md 文档
        while(++i < hi && nums[i] < pivot);
        if(i >= j)  break;
        swap(nums, i, j);
    }
    swap(nums, lo, j);
    return j;
}
private void swap(int[] nums, int i, int j){
    if(i == j)	return;
    nums[i] ^= nums[j];
    nums[j] ^= nums[i];
    nums[i] ^= nums[j];
}
```



## 解法2：优先队列, O(n*logk)

```java
Queue<Integer> que = new PriorityQueue<>(); // 最小堆
for(int a : nums){
    if(que.size() == k){ // 已有K个数时，若当前数大于堆顶，则说明堆顶不是第K大的值，因此弹出，再将当前数放入堆
        if(que.peek() < a){ 
            que.poll();
            que.offer(a);
        }
    }else   que.offer(a);
}
return que.peek();
```

