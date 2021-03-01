---
layout: article
title: Python 이진 탐색 구현하기
aside:
    toc: false
---

Leetcode에서 '37. Search Insert Position' 문제를 풀며 이진 탐색을 복습했다. <br/>

#### 문제
Given a sorted array of distinct integers and a target value, return the index if the target is found. If not, return the index where it would be if it were inserted in order. <br/>
Example 1: <br/>
Input: `nums = [1,3,5,6]`, `target = 5` <br/>
Output: 2 <br/>

### 맨 처음 떠올린 해결 방안
시간 복잡도를 전혀 고려하지 않고 가장 먼저 떠오른 방안은 이거다. 아마 가장 일차원적인 답이 아닐까!
```python
class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        if not(target in nums):
            nums.append(target)
            nums.sort
        return nums.index(target)
```

### 이진탐색 적용
위 코드는 `nums` 안에 원소가 많을 수록 문제다. 게다가 `nums`에 마음대로 원소를 추가하고 있다. <br/>
문제에서 `nums`는 이미 오름차순으로 정렬이 되어 있다는 가정이 있기 때문에, 이진탐색이 가장 효율적인 방법으로 보인다.
```python
class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        start, end = 0, len(nums) - 1
        while start <= end:
            mid = (start + end) // 2
            if nums[mid] == target:
                return mid
            elif nums[mid] > target:
                end = mid - 1
            elif nums[mid] < target:
                start = mid + 1
        return start
```

매우 쉬운 문제지만 이진탐색 기초를 다시 다잡기는 좋은 문제였다.

<!--more-->
