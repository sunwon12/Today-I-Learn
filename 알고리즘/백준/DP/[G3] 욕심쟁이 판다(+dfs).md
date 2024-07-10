## **#문제**         

**레벨: G3  
알고리즘: dp**  
**풀이시간: 1시간  
힌트 참조 유무:**

[https://www.acmicpc.net/problem/1937](https://www.acmicpc.net/problem/1937)

![image](https://github.com/sunwon12/Today-I-Learn/assets/92251131/df2b546d-8681-4e1b-ba05-313219ff4575)

---

## **#문제 풀이**        

각 칸을 모두 순회하며 dfs를 한다. 특이점은 dfs를 하며 방문한 위치값은 값을 새겨놓고 다시는 방문하지 않는다. 다른 dfs루트로 그 위치를 방문했을 때는 전에 얻어놓은 값을 쓴다.

dp\[\]\[\] 배열 채워지는 과정

```
3
3 7 5        //입력값
1 2 4
6 8 2


210
000         //1번째 dp
000

210
000         //2번째 dp
000

212
000         //3번째 dp
000

212
543         //4번째 dp
210

212
543         //5번째 dp
210

212
543         //6번째 dp
210

212
543         //7번째 dp
210

212
543         //8번째 dp
210

212
543         //9번째 dp
214

5
```

---

## **#풀이 코드**      

```
import java.io.*;
import java.util.*;

public class Main {
    static int n;
    static int[][] forest;
    static int[][] dp;
    static int[] dx = {-1, 1, 0, 0};
    static int[] dy = {0, 0, -1, 1};

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        n = Integer.parseInt(br.readLine());
        
        forest = new int[n][n];
        dp = new int[n][n];
        
        for (int i = 0; i < n; i++) {
            StringTokenizer st = new StringTokenizer(br.readLine());
            for (int j = 0; j < n; j++) {
                forest[i][j] = Integer.parseInt(st.nextToken());
            }
        }
        
        int maxPath = 0;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                maxPath = Math.max(maxPath, dfs(i, j));
            }
        }
        
        System.out.println(maxPath);
    }
    
    static int dfs(int x, int y) {
        if (dp[x][y] != 0) return dp[x][y];
        
        dp[x][y] = 1;
        
        for (int i = 0; i < 4; i++) {
            int nx = x + dx[i];
            int ny = y + dy[i];
            
            if (nx >= 0 && nx < n && ny >= 0 && ny < n && forest[nx][ny] > forest[x][y]) {
                dp[x][y] = Math.max(dp[x][y], dfs(nx, ny) + 1);
            }
        }
        
        return dp[x][y];
    }
}
```
