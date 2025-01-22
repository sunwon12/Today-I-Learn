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
나의 경우는 잘못된 접근 방법이다. **무조건 풀이 알고리즘을 생각하고 알고리즘의 시간복잡도를 예측해봐야 한다.** 위 dfs 알고리즘의 빅오는 아래와 같다.

**최악의 경우 계산:**

-   각 위치(N×M)에서 시작 가능
-   각 단계에서 4방향, K칸씩 이동 가능 (4×K 가지 선택)
-   단어 길이(L)만큼 반복

따라서 **시간복잡도는 O(N x M x 4k^L)**와 같고, 숫자를 대입해 보면 100 × 100 × 20^80 이다. 연산 수가 2억이 훨씬은 넘기 때문에 자명하게 통과할 수 없는 알고리즘이라는 것을 확인할 수 있다.

  
우린 dfs에서 시간초과가 난다면, 규칙이 있는 문제이거나 dp를 고려해야 한다. 이 문제에는 규칙이 없음이 극명해 dfs 코드에 dp를 접목시켰다. 

![image](https://github.com/user-attachments/assets/68540446-806a-4e31-a202-7872d9aca450)

만약 위와 같은 예시라면 맨 아래 B에서 시작할 때 위와 같이 탐색 할 것입니다. 

![image](https://github.com/user-attachments/assets/7700f95b-563a-4198-832a-5b8739ef4df1)

만약 dp를 적용하려고 한다면, 두 번 째 B에서 부터 탐색할 때 맨 아래 B가 탐색한 거와 똑같이 탐색할 것입니다. 근데 그럴 필요가 있을 때까요? 그 경로는 이미 R이 탐색을 해봤기 때문에 알고 있습니다. 그래서 dp를 이용하다면 R에서 탐색을 진행하지 않고 이전의 탐색을 이용해서 저장했던 값을 리턴해주는 것입니다.

**dp\[x\]\[y\]\[index\]**는 (x, y) 위치에서 시작하여 단어의 index번째 문자부터 끝까지 만들 수 있는 경로의 수를 저장한다. 예를 들면

-   dp\[2\]\[3\]\[0\] = 5는 (2, 3) 위치에서 시작하여 단어 전체를 만들 수 있는 경로가 5개 있다는 의미이다.
-   dp\[1\]\[1\]\[2\] = 3은 (1, 1) 위치에서 시작하여 단어의 3번째 문자부터 끝까지 만들 수 있는 경로가 3개 있다는 의미이다.

이 문제의 dp의 최악의 시간복잡도는 dp 배열을 한 번씩 값을 넣어보는 것이기 때문에 식은 아래와 같습니다.

**상태 공간 크기×각 상태 계산 비용\=O(N×M×L×4K)**

100 x 100 x 80 x 20 = 1600백만입니다. 2억 미만이기 때문에 충분히 통과 가능한 시간 복잡도입니다.

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
