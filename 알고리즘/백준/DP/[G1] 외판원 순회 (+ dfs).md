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


모든 경우의 수를 살펴봐야 하는 문제임은 확실하다. 그러나 최악의 상황을 가장해보면 16!이다. 20조 9,227억 8,988만 8천이라는 것이다. ( 11! = 39,916,800 (약 3.99 × 10^7) 안에 들어야 시간복잡도에 통과할 수 있다.) 브루트 포스는 확실한데 시간이 걸린다면 dp 도입을 의심하면 된다. 

dp + dfs 는 **postorder**로 코드 구현한다. 

dp를 재귀함수로 구현하려면 값이 저장된 값(dp 사용)을 리턴해줘야 하므로 리턴 타입은 void가 아니여야 한다. 그렇다면 첫 번쨰 함수 호출, 즉 main 함수에서 호출하고 리턴값을 돌려받아한다. 호출했을 때 넘겨준 파라미터는 dp에서 초기상태이다. **정답으로 쓰는 값은 dp 마지막 상태가 아니라 초기 상태이다.**

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
