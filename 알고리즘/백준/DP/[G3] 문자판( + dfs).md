## **#문제**         

**레벨: G3  
알고리즘: dp + dfs**  
**풀이시간:   
힌트 참조 유무:**

[https://www.acmicpc.net/problem/2186](https://www.acmicpc.net/problem/2186)

![image](https://github.com/user-attachments/assets/dc7a3b5d-ee75-4125-b9f5-bc809a555132)

---

## **#문제 풀이**        

처음 문제를 봤을 때 별 생각없이 아래와 같이 dfs 코드를 짰다. 코드는 문제에 맞게 구현됐다. 단지, 시간 초과가 날 뿐이다...

잠시 이 dfs 코드를 흝어봐주고 내려와주라.

**dfs코드**

```
import java.io.*;
import java.util.*;

public class Main {
    static int N, M, K;
    static char[][] board;
    static String word;
    static int[] dx = {-1, 1, 0, 0};
    static int[] dy = {0, 0, -1, 1};
    static long count = 0;

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(br.readLine());
        N = Integer.parseInt(st.nextToken());
        M = Integer.parseInt(st.nextToken());
        K = Integer.parseInt(st.nextToken());

        board = new char[N][M];
        for (int i = 0; i < N; i++) {
            board[i] = br.readLine().toCharArray();
        }

        word = br.readLine();

        for (int i = 0; i < N; i++) {
            for (int j = 0; j < M; j++) {
                if (board[i][j] == word.charAt(0)) {
                    dfs(1, i, j);
                }
            }
        }

        System.out.println(count);
    }

    static void dfs(int index, int x, int y) {
        if (index == word.length()) {
            count++;
            return;
        }

        for (int dir = 0; dir < 4; dir++) {
            for (int k = 1; k <= K; k++) {
                int nx = x + dx[dir] * k;
                int ny = y + dy[dir] * k;
                
                if (nx < 0 || nx >= N || ny < 0 || ny >= M) {
                    break;  // 범위를 벗어나면 더 이상 이 방향으로 진행하지 않음
                }
                
                if (board[nx][ny] == word.charAt(index)) {
                    dfs(index + 1, nx, ny);
                }
            }
        }
    }
}
```

우린 dfs에서 시간초과가 난다면, 규칙이 있는 문제이거나 dp를 고려해야 한다. 이 문제에는 규칙이 없음이 극명해 dfs 코드에 dp를 접목시켰다. 

dp\[x\]\[y\]\[index\]는 (x, y) 위치에서 시작하여 단어의 index번째 문자부터 끝까지 만들 수 있는 경로의 수를 저장한다. 예를 들면

-   dp\[2\]\[3\]\[0\] = 5는 (2, 3) 위치에서 시작하여 단어 전체를 만들 수 있는 경로가 5개 있다는 의미이다.
-   dp\[1\]\[1\]\[2\] = 3은 (1, 1) 위치에서 시작하여 단어의 3번째 문자부터 끝까지 만들 수 있는 경로가 3개 있다는 의미이다.

---

## **#풀이 코드**      

**dp + dfs 코드**

```
import java.io.*;
import java.util.*;

public class Main {
    static int N, M, K;
    static char[][] board;
    static String word;
    static long[][][] dp;
    static int[] dx = {-1, 1, 0, 0};
    static int[] dy = {0, 0, -1, 1};

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(br.readLine());
        N = Integer.parseInt(st.nextToken());
        M = Integer.parseInt(st.nextToken());
        K = Integer.parseInt(st.nextToken());

        board = new char[N][M];
        for (int i = 0; i < N; i++) {
            board[i] = br.readLine().toCharArray();
        }

        word = br.readLine();
        dp = new long[N][M][word.length()];
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < M; j++) {
                Arrays.fill(dp[i][j], -1);
            }
        }

        long result = 0;
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < M; j++) {
                if (board[i][j] == word.charAt(0)) {
                    result += dfs(i, j, 0);
                }
            }
        }

        System.out.println(result);
    }

    static long dfs(int x, int y, int index) {
        if (index == word.length() - 1) return 1;
        if (dp[x][y][index] != -1) return dp[x][y][index];

        dp[x][y][index] = 0;
        for (int i = 0; i < 4; i++) {
            for (int k = 1; k <= K; k++) {
                int nx = x + dx[i] * k;
                int ny = y + dy[i] * k;
                if (nx >= 0 && nx < N && ny >= 0 && ny < M && board[nx][ny] == word.charAt(index + 1)) {
                    dp[x][y][index] += dfs(nx, ny, index + 1);
                }
            }
        }

        return dp[x][y][index];
    }
}
```
