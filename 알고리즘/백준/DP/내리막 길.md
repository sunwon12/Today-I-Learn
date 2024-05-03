### **문제**         

**레벨: G4  
알고리즘: DFS + DP**   
**풀이시간:  40분   
힌트 참조 유무: 유**
https://www.acmicpc.net/problem/1520
![image](https://github.com/sunwon12/Today-I-Learn/assets/92251131/bd740ffe-84d6-4a07-941b-0c14b3aa4b6f)

### **1 번째 시도**   

**1\. 먼저 (0,0) 위치에서 시작하므로 dp\[0\]\[0\]값을 0으로 초기화한다.**

**2\. (0,0) 위치에서 상하좌우로 이동하면서 map\[x\]\[y\] > map\[nx\]\[ny\]인 위치에 대해서만 DFS 메소드를 재귀호출**

**3\. 재귀호출을 통해 (M-1, N-1)위치에 도달하는 경로의 개수를 찾는다.**

**4\. 이렇게 찾은 경로의 개수를 dp\[0\]\[0\] 값에 더해준다.**

**5\. 모든 이동 가능한 위치에 대해 위 과정을 반복**

**6\. dp\[0\]\[0\] 값이 결정되면 해당 값을 리턴한다.**

**위와 같은 과정을 거쳐서 dp\[0\]\[0\] 값을 찾게 된다. 만약 dp\[0\]\[0\] 값이 3이라면 이는 (0,0)에서 (M-1,N-1) 위치까지 이동할 수 잇는 경로의 개수가 3이라는 뜻이다.**

```

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.StringTokenizer;

public class Main {

    static int M,N;
    static int[][] map,dp;
    static int dx[] = {0,1,0,-1};
    static int dy[] = {1,0,-1,0};

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(br.readLine());

        M = Integer.parseInt(st.nextToken());
        N = Integer.parseInt(st.nextToken());

        map = new int[M][N];
        dp = new int[M][N];

        for (int i = 0; i < M ; i++) {
            st = new StringTokenizer(br.readLine());
            for (int j = 0; j < N ; j++) {
                map[i][j] = Integer.parseInt(st.nextToken());
            }
        }

        for (int i = 0; i < M; i++) {
          Arrays.fill(dp[i],Integer.MIN_VALUE);
        }

        System.out.println(DFS(0,0));
    }

    static int DFS(int x, int y){

        if(x == M-1 && y == N-1) return 1;

        if(dp[x][y] != Integer.MIN_VALUE) return dp[x][y]; //방문한 곳이니 탐색 전에 존재하는 값을 던져줌

        dp[x][y] = 0; //아래에서 + 때문에 초기화
        for (int i = 0; i < 4; i++) {
            int nx = x + dx[i];
            int ny = y + dy[i];

            if(nx < 0 || ny < 0 || nx >= M || ny >= N) continue;

            if(map[x][y] > map[nx][ny]) dp[x][y] += DFS(nx,ny); //높이가 낮아야 탐색을 한다.

        }

        return dp[x][y];
    }
}
```
