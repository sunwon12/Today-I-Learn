### **문제**         

**레벨: G5  
알고리즘: dp + LIS**   
**풀이시간: 1시간   
힌트 참조 유무: 유**

[https://www.acmicpc.net/problem/2565](https://www.acmicpc.net/problem/2565)

![image](https://github.com/user-attachments/assets/e14b529e-cb33-42c5-b8c5-cf260ab4e202)

### **1 번째 시도**   

### **\[알고리즘 설명\]**

이 문제는 최장 증가 부분 수열(LIS: Longest Increasing Subsequence) 알고리즘을 사용하여 해결할 수 있습니다. 전깃줄이 교차하지 않으려면, B 전봇대의 연결 위치가 증가하는 순서여야 합니다. 따라서 LIS를 찾으면 교차하지 않는 최대 전깃줄의 개수를 알 수 있고, 전체 전깃줄 개수에서 이를 뺀 값이 제거해야 할 최소 전깃줄의 개수가 됩니다.

**시간복잡도 O(n^2) 코드**

```
import java.util.*;
import java.io.*;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(br.readLine());
        
        int[][] wires = new int[n][2];
        for (int i = 0; i < n; i++) {
            StringTokenizer st = new StringTokenizer(br.readLine());
            wires[i][0] = Integer.parseInt(st.nextToken());
            wires[i][1] = Integer.parseInt(st.nextToken());
        }
        
        // A 전봇대 기준으로 정렬
        Arrays.sort(wires, (a, b) -> a[0] - b[0]);
        
        int[] dp = new int[n];
        int maxLength = 0;
        
        for (int i = 0; i < n; i++) {
            dp[i] = 1;
            for (int j = 0; j < i; j++) {
                if (wires[i][1] > wires[j][1]) {
                    dp[i] = Math.max(dp[i], dp[j] + 1);
                }
            }
            maxLength = Math.max(maxLength, dp[i]);
        }
        
        System.out.println(n - maxLength);
    }
}
```

**시간복잡도 O(nlogn) 코드: 이분탐색**

```
import java.util.*;
import java.io.*;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(br.readLine());
        
        int[][] wires = new int[n][2];
        for (int i = 0; i < n; i++) {
            StringTokenizer st = new StringTokenizer(br.readLine());
            wires[i][0] = Integer.parseInt(st.nextToken());
            wires[i][1] = Integer.parseInt(st.nextToken());
        }
        
        // A 전봇대 기준으로 정렬
        Arrays.sort(wires, (a, b) -> a[0] - b[0]);
        
        ArrayList<Integer> lis = new ArrayList<>();
        
        for (int[] wire : wires) {
            int pos = Collections.binarySearch(lis, wire[1]);
            if (pos < 0) {
                pos = -(pos + 1);
            }
            
            if (pos == lis.size()) {
                lis.add(wire[1]);
            } else {
                lis.set(pos, wire[1]);
            }
        }
        
        System.out.println(n - lis.size());
    }
}
```

**\[동작 과정\]**

```
[[1,8], [2,2], [3,9], [4,1], [6,4], [7,6], [9,7], [10,10]]

[1,8] 처리:

lis: [8]


[2,2] 처리:

2는 8보다 작으므로 8을 대체
lis: [2]


[3,9] 처리:

9는 2보다 크므로 뒤에 추가
lis: [2, 9]


[4,1] 처리:

1은 2보다 작으므로 2를 대체
lis: [1, 9]


[6,4] 처리:

4는 1보다 크고 9보다 작으므로 9를 대체
lis: [1, 4]


[7,6] 처리:

6은 4보다 크므로 뒤에 추가
lis: [1, 4, 6]


[9,7] 처리:

7은 6보다 크므로 뒤에 추가
lis: [1, 4, 6, 7]


[10,10] 처리:

10은 7보다 크므로 뒤에 추가
lis: [1, 4, 6, 7, 10]
```

#### **LIS의 배열이 실제 LIS 수열과 달라고 길이가 보존되는 이유는?**

```
원래 수열: 3 2 5 2 3 1 4

실제 LIS: 2 3 4 (길이 3)
dp 배열 최종상태: [1,3,4] (길이 3)

예: 5를 3으로 교체해도 "길이 2인 증가 수열이 존재한다"는 사실은 유지
dp 배열의 각 위치는 "해당 길이의 증가 수열이 존재한다"는 증거로 작동
```

**\[binarySearch 함수\]**

## **#문제**         

**레벨:   
알고리즘:**   
**풀이시간:   
힌트 참조 유무:**

---

## **#문제 풀이**        

---

## **#풀이 코드**      

```
int[] arr = {1, 3, 5, 7, 9};
int index = Arrays.binarySearch(arr, 5);  // 반환값: 2
int index2 = Arrays.binarySearch(arr, 4); // 반환값: -3 (4가 삽입되어야 할 위치는 인덱스 2)
```
