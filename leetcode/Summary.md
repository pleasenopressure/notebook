problem:3Sum
https://leetcode.com/problems/3sum/?envType=company&envId=tiktok&favoriteSlug=tiktok-three-months
code:
```
class Solution:

    def threeSum(self, nums: List[int]) -> List[List[int]]:

        ans = set()

        for x in range(len(nums)):

            for y in range(x+1, len(nums)):

                for z in range(y+1,len(nums)):

                    if nums[x]+nums[y]+nums[z] == 0:

                        temp = tuple(sorted((nums[x],nums[y],nums[z])))

                        ans.add(temp)

        for iter in ans :

            iter = list(iter)

        ans = list(ans)

        return ans
```
Summary:
I learned that the data structure **`set`**, which is actually implemented using a **hash table**.  
Therefore, the average time complexity for lookup, insertion, and deletion is **O(1)**.

- Because of this property, `set` is suitable for **duplicate removal**.
    
- When facing the problem of removing duplicates from **multi-element combinations regardless of order**, using `sorted()` (and then converting to a tuple) is a common and reliable approach.
Solution:
The algorithm of first code 's time complexity is O(n^3), whcih is too high. so here I propose a better way:
- sort the original array, which can provide convenience for following duplicate removal way.
- Fix a pivot element ,such as nums[i].
- using Two-pointers way, when the sum of nums[i],nums[left],nums[right] is higher than target value(here is zero), Decrease the right. 反之亦然
- During this period , we can also do some pruning. about the detail, see the following code
```
    def threeSum(self, nums: List[int]) -> List[List[int]]:

        nums = sorted(nums)

        res = []

        for i in range (len(nums)):

            if i>0:

                if nums[i-1] == nums[i]:

                    continue

            if nums[0] > 0:

                return []

            self.TwoSum(nums, i,res)

        return res

  

    def TwoSum(self,nums:List[int],i:int, res:List[List[int]]):

        left = i+1

        right = len(nums)-1

        while left<right:

            target =  nums[i]+ nums[left] + nums[right]

            if target >0:

                while left<right and nums[right-1] == nums[right]:

                    right -= 1

                right -=1

            elif target < 0 :

                while left<right and nums[left+1] == nums[left]  :

                    left += 1

                left+=1

            else:

                res.append([nums[i],nums[left],nums[right]])

                while left<right and nums[right-1] == nums[right]:

                    right -= 1

                right -=1

                while left<right and nums[left+1] == nums[left]  :

                    left += 1

                left+=1

        return []
```