# 문제
- 주어진 배열에서 3개의 수를 더하여 target 과 가장 가까워지는 수를 구하는 문제

## 풀이
- 배열의 크기가 작기 때문에, 단순히 반복문으로 해결
- 당연히 속도가 굉장히 느림
- 사실 다른 방법으로 풀이하려고 정렬하고 시작했는 데, 속도 limit 이 따로 없는 듯함
  - 정렬한 후 이진탐색으로 찾는 형태로 하고 싶었는데 생각할 부분이 많은거 같아 패스
    => 실제로 정답중 속도가 빠른분의 풀이를 보니 상상했던것과 비슷한 내용
    ```java
    class Solution {
        public int threeSumClosest(int[] nums, int target) {
            if(nums==null || nums.length<3){
                return 0;
            }
            int result = nums[0] + nums[1] +nums[2];
            int len = nums.length;
            Arrays.sort(nums);
            for(int i=0; i<len-2; i++){
                if(i>0 && nums[i]==nums[i-1]){
                    continue;
                }
                if(nums[i] +nums[i+1] +nums[i+2] > target){
                    int sum = nums[i] +nums[i+1] +nums[i+2];
                    if((sum>target?sum-target:target-sum) < (result>target?result-target:target-result)){
                        result = sum;
                    }
                    break;
                }
                if(nums[i] + nums[len-1] +nums[len-2] < target){
                    int sum = nums[i] + nums[len-1] +nums[len-2];
                    if((sum>target?sum-target:target-sum) < (result>target?result-target:target-result)){
                        result = sum;
                    }
                    continue;
                }
                int left=i+1, right=len-1;
                while(left<right){
                    int sum = nums[i] + nums[left] + nums[right];
                    if(sum == target){
                        return sum;
                    }
                    if((sum>target?sum-target:target-sum) < (result>target?result-target:target-result)){
                        result = sum;
                    }
                    if(sum > target){
                        right--;
                        while(left<right && nums[right]==nums[right+1])
                            right--;
                    }
                    if(sum < target){
                        left++;
                        while(left<right && nums[left]==nums[left-1]){
                            left++;
                        }
                    }
                }
            }
            return result;
        }
    }
    ```
    
- 나의 풀이 : [https://github.com/pch8388/study-algorithm/blob/caf1d8371362c21116a35d6c54b9623b820d4da7/src/main/java/me/leetcode/ThreeSumClosest.java]
문제 링크 : [https://leetcode.com/problems/3sum-closest/submissions/]
