## **#문제**         

**레벨: G4  
알고리즘: dp**  
**풀이시간:   
힌트 참조 유무:**

[https://www.acmicpc.net/problem/17404](https://www.acmicpc.net/problem/17404)

![image](https://github.com/user-attachments/assets/28363dc5-6870-4a84-806c-107769d594d2)

---

## **#문제 풀이**        

dp에서 중요한 것은 아래와 같습니다.  
1\. 상태의 정의  
2\. 점화식의 정의  
  
bottom-up방식으로 점화식을 세워보겠습니다.  
현재 상태에서 가장 최솟값임을 보장하려면, 현재 색깔 + 이전색깔 중 최솟값이어야 합니다. 점화식으로 나타내면  
dp\[i\]\[0\] = cost\[i\]\[0\] + Math.min(dp\[i-1\]\[1\],dp\[i-1\]\[2\]);  
dp\[i\]\[1\] = cost\[i\]\[1\] + Math.min(dp\[i-1\]\[0\],dp\[i-1\]\[2\]);   
dp\[i\]\[2\] = cost\[i\]\[2\] + Math.min(dp\[i-1\]\[0\],dp\[i-1\]\[1\]);  
입니다.  
  
dp\[i\]\[0\], dp\[i\]\[1\], dp\[1\]\[2\]와 같이 색깔별로 상태를 저장한 이유는 i는 i-1과 i+1과 같지 않다라는 규칙이 있기 때문입니다. 그렇기 때문에 상태를 나누어 dp값을 저장합니다.

어느 dp와 같이 이 dp의 초기값을 세팅해야 합니다.  
처음집과 마지막집의 색이 같으면 안된다는 규칙 때문에 초기값 세팅의 경우의 수를 3가지로 나누어 탐색합니다.  
  

---

## **#풀이 코드**      

**bottom-up**(반복문)****

```
import java.io.*;
import java.util.*;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(br.readLine());
        int[][] cost = new int[n][3];
        
        // 비용 입력
        for(int i = 0; i < n; i++) {
            StringTokenizer st = new StringTokenizer(br.readLine());
            cost[i][0] = Integer.parseInt(st.nextToken()); // R
            cost[i][1] = Integer.parseInt(st.nextToken()); // G
            cost[i][2] = Integer.parseInt(st.nextToken()); // B
        }
        
        int result = Integer.MAX_VALUE;
        
        // 첫 집의 색을 고정하고 시작
        for(int firstColor = 0; firstColor < 3; firstColor++) {
            int[][] dp = new int[n][3];
            
            // 첫 집의 다른 색은 불가능하도록 설정
            for(int color = 0; color < 3; color++) {
                if(color == firstColor) dp[0][color] = cost[0][color];
                else dp[0][color] = 1000*1000;
            }
            
            // 두 번째 집부터 마지막 집까지
            for(int i = 1; i < n; i++) {
                dp[i][0] = Math.min(dp[i-1][1], dp[i-1][2]) + cost[i][0]; // R
                dp[i][1] = Math.min(dp[i-1][0], dp[i-1][2]) + cost[i][1]; // G
                dp[i][2] = Math.min(dp[i-1][0], dp[i-1][1]) + cost[i][2]; // B
            }
            
            // 첫 집과 마지막 집의 색이 다른 경우만 고려
            for(int lastColor = 0; lastColor < 3; lastColor++) {
                if(lastColor != firstColor) {
                    result = Math.min(result, dp[n-1][lastColor]);
                }
            }
        }
        
        System.out.println(result);
    }
}
```

**top-down**(dfs)****

```
import java.io.*;
import java.util.*;

public class Main {
    static int n;
    static int[][] cost;
    static int[][][] dp;
    static final int INF = 1000*1000;
    
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        n = Integer.parseInt(br.readLine());
        cost = new int[n][3];
        dp = new int[n][3][3]; // [현재집][현재색][첫집색]
        
        // 비용 입력
        for(int i = 0; i < n; i++) {
            StringTokenizer st = new StringTokenizer(br.readLine());
            cost[i][0] = Integer.parseInt(st.nextToken());
            cost[i][1] = Integer.parseInt(st.nextToken());
            cost[i][2] = Integer.parseInt(st.nextToken());
        }
        
        // dp 배열 초기화
        for(int i = 0; i < n; i++)
            for(int j = 0; j < 3; j++)
                Arrays.fill(dp[i][j], -1);
        
        int result = INF;
        // 첫 집의 색을 정하고 시작
        for(int firstColor = 0; firstColor < 3; firstColor++) {
            result = Math.min(result, cost[0][firstColor] + 
                   paint(1, firstColor, firstColor));
        }
        
        System.out.println(result);
    }
    
    static int paint(int house, int prevColor, int firstColor) {
        // 기저 사례: 마지막 집까지 다 칠했을 때
        if(house == n) return 0;
        
        // 이미 계산한 경우
        if(dp[house][prevColor][firstColor] != -1)
            return dp[house][prevColor][firstColor];
        
        dp[house][prevColor][firstColor] = INF;
        
        // 현재 집을 칠할 색 선택
        for(int color = 0; color < 3; color++) {
            // 이전 집과 색이 달라야 함
            if(color != prevColor) {
                // 마지막 집이면 첫 집과도 색이 달라야 함
                if(house == n-1 && color == firstColor) continue;
                
                int next = paint(house + 1, color, firstColor) + cost[house][color];
                dp[house][prevColor][firstColor] = Math.min(
                    dp[house][prevColor][firstColor],
                    next
                );
            }
        }
        
        return dp[house][prevColor][firstColor];
    }
}
```
