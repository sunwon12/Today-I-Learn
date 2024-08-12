## **#문제**         

**레벨: P4  
알고리즘: dfs + 카데인 알고리즘**  
**풀이시간: 1시간  
힌트 참조 유무: 유**

[https://www.acmicpc.net/problem/18251](https://www.acmicpc.net/problem/18251)

![image](https://github.com/user-attachments/assets/2a093a43-ee30-44a7-9329-b0dc4df84f00)

---

## **#문제 풀이**        

**문제 풀기에 앞서 카데인 알고리즘을 알고가야 한다.**  카데인 알고리즘이란  최대 부분 합 수열을 구할 때 사용한다. 평소 이 알고리즘 이름은 안적도 없지만, 이러한 방식으로 문제를 푼 경우가 있을 것이다. 이거에 이름을 붙여 더 확실히 기억해보자.

```
예시 배열: [-2, 1, -3, 4, -1, 2, 1, -5, 4]

localMaxValue: 현재 위치까지의 연속된 부분 배열의 최대 합 
globalMaxValue: 지금까지 발견된 전체 최대 부분 배열의 합

초기 상태:
 

   [-2,  1, -3,  4, -1,  2,  1, -5,  4]
    ^
   현재 요소: -2
   localMaxValue: -2
   globalMaxValue: -2
   
두 번째 요소:
 

   [-2,  1, -3,  4, -1,  2,  1, -5,  4]
        ^
   현재 요소: 1
   localMaxValue: 1 (max(-2+1, 1) = 1)
   globalMaxValue: 1
   
세 번째 요소:
 

   [-2,  1, -3,  4, -1,  2,  1, -5,  4]
            ^
   현재 요소: -3
   localMaxValue: -2 (max(1-3, -3) = -2)
   globalMaxValue: 1
   
네 번째 요소:
 

   [-2,  1, -3,  4, -1,  2,  1, -5,  4]
                ^
   현재 요소: 4
   localMaxValue: 4 (max(-2+4, 4) = 4)
   globalMaxValue: 4
   
다섯 번째 요소:
 

   [-2,  1, -3,  4, -1,  2,  1, -5,  4]
                    ^
   현재 요소: -1
   localMaxValue: 3 (max(4-1, -1) = 3)
   globalMaxValue: 4
   
여섯 번째 요소:
 

   [-2,  1, -3,  4, -1,  2,  1, -5,  4]
                        ^
   현재 요소: 2
   localMaxValue: 5 (max(3+2, 2) = 5)
   globalMaxValue: 5
   
일곱 번째 요소:
 

   [-2,  1, -3,  4, -1,  2,  1, -5,  4]
                            ^
   현재 요소: 1
   localMaxValue: 6 (max(5+1, 1) = 6)
   globalMaxValue: 6
   
여덟 번째 요소:
 

   [-2,  1, -3,  4, -1,  2,  1, -5,  4]
                                ^
   현재 요소: -5
   localMaxValue: 1 (max(6-5, -5) = 1)
   globalMaxValue: 6
   
마지막 요소:
 

   [-2,  1, -3,  4, -1,  2,  1, -5,  4]
                                    ^
   현재 요소: 4
   localMaxValue: 5 (max(1+4, 4) = 5)
   globalMaxValue: 6
```

위 카데인 알고리즘을 코드로 구현하면 아래와 같다.

```
class Solution {
    public int maxSubArray(int[] nums) {
        int globalMaxValue = nums[0]; // 최종적으로 리턴할 변수
        int localMaxValue = nums[0]; // 인덱스별로 구한 부분합
        for (int i = 1; i < nums.length; i++) { // 0번 인덱스는 포함되어 있으니 1번부터 마지막까지 돌려줌
            localMaxValue = Math.max(nums[i], nums[i] + localMaxValue); // 나 자신과 이전까지 최대값을 더해서 비교해주면 현재까지의 최대값이 나옴
            globalMaxValue = Math.max(globalMaxValue, localMaxValue); // 리턴할 값과 비교해줌
        }
        
        return globalMaxValue;
    }
}
```

![image](https://github.com/user-attachments/assets/978e756a-878c-4ba0-957d-ee65bc470211)

이렇게 노드가 있다고 하였을 때 깊이 1층에서의 최대 합, 1 ~ 2층에서의 최대 합, 1 ~3층에서의 최대 합, 2층에서의 최대합, 2 ~3층에서의 최대합, 3층에서의 최대 합을 구하면 직사각형에 담긴 최대합을 구할 수 있다. 

 **2 ~3층에서의 최대합을 구하는 과정을 보여주겠다**. 만약 배열이 노드의 번호순대로가 아닌 x를 기준으로 노드의 배치순대로 되어있다면 아래 그림과 같이 순서대로 숫자를 하나씩 추가해가며 sum값을 업데이트 해가면 된다.

![image](https://github.com/user-attachments/assets/ceebe680-1ff7-4c78-9f1b-cce2e90960fe)
![image](https://github.com/user-attachments/assets/db8cf455-ec1f-42dc-b061-38cfc9e14692)
![image](https://github.com/user-attachments/assets/f4b8699d-3384-4909-9ee3-7edb16e147e9)
![image](https://github.com/user-attachments/assets/f5b4b413-e631-4a59-9fe4-77d0fec9bfc5)




---

## **#풀이 코드**      

```
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.*;

public class Main {
    private int getK(int n) {
        int k = 1;
        while ((n/=2)!=1) {
            k++;
        }
        return k;
    }

    private void solution() throws Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in), 1<<17);
        int n = Integer.parseInt(br.readLine()) + 1;
        int k = getK(n);
        int[] arr = new int[n];
        ArrayList<Integer>[] depth = new ArrayList[k];
        for (int i = 0; i < k; i++) depth[i] = new ArrayList<>(1<<i);

        Queue<int[]> q = new ArrayDeque<>();
        q.add(new int[]{0, n/2});
        StringTokenizer st = new StringTokenizer(br.readLine());
        while (!q.isEmpty()) { // 노드 배치순대로 배열에 새겨 넣기 
            int[] cur = q.poll();
            depth[cur[0]].add(cur[1]);
            arr[cur[1]] = Integer.parseInt(st.nextToken());
            if (cur[0] == k-1) continue;

            q.add(new int[]{cur[0]+1, cur[1]-(1<<(k-2-cur[0]))});
            q.add(new int[]{cur[0]+1, cur[1]+(1<<(k-2-cur[0]))});
        }

        long answer = Long.MIN_VALUE;
        for (int i = 0; i < k; i++) {
            for (int j = i; j < k; j++) {
                HashSet<Integer> hs = new HashSet<>();
                for (int s = i; s <= j; s++) {
                    ArrayList<Integer> depthList = depth[s];
                    for (int t = 0; t < depthList.size(); t++) {
                        hs.add(depthList.get(t));
                    }
                }

                long sum = 0;
                for (int x = 1; x < n; x++) {
                    if (!hs.contains(x)) continue;
                    sum+=arr[x];
                    if (sum > answer) answer = sum;
                    if (sum < 0) {
                        sum = 0;
                        continue;
                    }
                }
            }
        }
        System.out.println(answer);
    }

    public static void main(String[] args) throws Exception {
        new Main().solution();
    }
}

// 출처: https://nahwasa.com/entry/%EC%9E%90%EB%B0%94-%EB%B0%B1%EC%A4%80-18251-%EB%82%B4-%EC%83%9D%EA%B0%81%EC%97%90-A%EB%B2%88%EC%9D%B8-%EB%8B%A8%EC%88%9C-dfs-%EB%AC%B8%EC%A0%9C%EA%B0%80-%EC%9D%B4-%EB%8C%80%ED%9A%8C%EC%97%90%EC%84%9C-E%EB%B2%88%EC%9D%B4-%EB%90%98%EC%96%B4%EB%B2%84%EB%A6%B0-%EA%B1%B4%EC%97%90-%EA%B4%80%ED%95%98%EC%97%AC-Easy-java
```
