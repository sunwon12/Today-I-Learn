## **#문제**         

**레벨: S1  
알고리즘: dp(기본)**  
**풀이시간: 30분  
힌트 참조 유무:**

[https://www.acmicpc.net/problem/1149](https://www.acmicpc.net/problem/1149)

![image](https://github.com/user-attachments/assets/05a889e8-e578-451e-9beb-a5be04d6d2fd)

---

## **#문제 풀이**        

위 문제느 dp 기본 문제이다. 점화식을 도출하고 그를 코드로 구현하는 것이다. 

위 문제의 점화식은 아래와 같다. 

            **// 빨강으로 칠할 경우**  
            **dp\[i\]\[0\] = Math.min(dp\[i-1\]\[1\], dp\[i-1\]\[2\]) + costs\[i\]\[0\]**  
            **// 초록으로 칠할 경우**  
            **dp\[i\]\[1\] = Math.min(dp\[i-1\]\[0\], dp\[i-1\]\[2\]) + costs\[i\]\[1\]**  
            **// 파랑으로 칠할 경우**  
            **dp\[i\]\[2\] = Math.min(dp\[i-1\]\[0\], dp\[i-1\]\[1\]) + costs\[i\]\[2\]**  
        

dp 배열에서 해당 인덱스가 가질 수 있는 가장 작은 값은 이전의 선택할 수  있는 가장 작은 값에 현재 값을 더하는 거기 때문에 위와 같이 도출된다.

---

## **#풀이 코드**      

```
import java.io.*;
import java.util.*;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int N = Integer.parseInt(br.readLine());
        
        int[][] costs = new int[N][3];
        for (int i = 0; i < N; i++) {
            StringTokenizer st = new StringTokenizer(br.readLine());
            for (int j = 0; j < 3; j++) {
                costs[i][j] = Integer.parseInt(st.nextToken());
            }
        }
        
        int[][] dp = new int[N][3];
        
        // 첫 번째 집의 비용 초기화
        dp[0][0] = costs[0][0];
        dp[0][1] = costs[0][1];
        dp[0][2] = costs[0][2];
        
        // 두 번째 집부터 N번째 집까지
        for (int i = 1; i < N; i++) {
            // 빨강으로 칠할 경우
            dp[i][0] = Math.min(dp[i-1][1], dp[i-1][2]) + costs[i][0];
            // 초록으로 칠할 경우
            dp[i][1] = Math.min(dp[i-1][0], dp[i-1][2]) + costs[i][1];
            // 파랑으로 칠할 경우
            dp[i][2] = Math.min(dp[i-1][0], dp[i-1][1]) + costs[i][2];
        }
        
        // 마지막 집까지 칠했을 때의 최소 비용
        int minCost = Math.min(dp[N-1][0], Math.min(dp[N-1][1], dp[N-1][2]));
        System.out.println(minCost);
    }
}
```
