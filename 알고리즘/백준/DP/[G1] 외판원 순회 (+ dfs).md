## **#문제**         

**레벨: G1  
알고리즘: dp + 브루트 포스 + 비트 마스킹**  
**풀이시간: 1시간  
힌트 참조 유무: 유**

[https://www.acmicpc.net/problem/2098](https://www.acmicpc.net/problem/2098)

![image](https://github.com/user-attachments/assets/3ea89df3-4e50-42ce-a917-7b9062bf6bf9)

---

## **#문제 풀이**        

TSP(Traveling Salesman Problem)

시간 복잡도 O(n^2 \* 2^n)

---

## **#풀이 코드**      

```
import java.util.*;
import java.io.*;

public class Main {
    static int n;
    static int[][] w;
    static int[][] dp;
    static final int INF = 1000000000;

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        n = Integer.parseInt(br.readLine());
        w = new int[n][n];
        dp = new int[1 << n][n];

        for (int i = 0; i < n; i++) {
            StringTokenizer st = new StringTokenizer(br.readLine());
            for (int j = 0; j < n; j++) {
                w[i][j] = Integer.parseInt(st.nextToken());
            }
        }

        for (int i = 0; i < (1 << n); i++) {
            Arrays.fill(dp[i], -1);
        }

        System.out.println(tsp(1, 0));
    }

    static int tsp(int mask, int pos) {
        if (mask == (1 << n) - 1) {
            return w[pos][0] != 0 ? w[pos][0] : INF;
        }

        if (dp[mask][pos] != -1) {
            return dp[mask][pos];
        }

        dp[mask][pos] = INF;
        for (int next = 0; next < n; next++) {
            if ((mask & (1 << next)) == 0 && w[pos][next] != 0) {
                dp[mask][pos] = Math.min(dp[mask][pos], 
                                         tsp(mask | (1 << next), next) + w[pos][next]);
            }
        }

        return dp[mask][pos];
    }
}
```
