## **#문제**         

**레벨: G3  
알고리즘; bfs**  
**풀이시간:   
힌트 참조 유무:**

[https://www.acmicpc.net/problem/16988](https://www.acmicpc.net/problem/16988)

![image](https://github.com/user-attachments/assets/2c28100e-b14d-45d7-8ff6-627a41cea0ac)

---

## **#문제 풀이**        

문제를 보는 순간 규칙이 없고 완전탐색(브루트 포스)로 해결해야겠다고 생각이 들었다. 시간복잡도를 구해보자면 2개의 바둑돌을 놓는 경우의 수(400 C 2) X 상대 바둑돌을 잡아먹는 수 구하기(400) = 400 x 399 % 2 X 400 = 대략삼천 이백만 정도 되므로 가능한 경우의 수이다. 

이제 구현할 차례이다. dfs를 돌며 바둑 돌을 놓는 구현은 0인 자리에만 놓고 dfs 재귀 깊이가 2인 함수를 구현하면 된다. 여기서 막혔다. 어떻게 함수를 구현해야 겹치치 않고 400 C 2를 구현할 수 있을까? 정답은 이중 for문을 돌며 dfs를 호출하면 된다. dfs 한 번에 400 C 2를 구현할 생각하지 마라. 어렵다. 아니 없을 수도 있다. 이중 for문에서 첫 번째로 시작된 인덱스는 다음 이중 for문 dfs문에서 포함되면 안된다. 겹치는 경우의 수가 나올 수 있기 때문이다. 예를 들면 첫번째 dfs에서 1를 선택하고 다음 2를 선택했다. 다음 dfs에서 2에서 시작되는데 다음 이전 dfs 시작인덱스인 1를 포함하게 된다면 2,1로 1,2의 경우의 수와 중복되게 된다. 

바둑을 잡아먹는 수 구하기 는 이중 for문을 돌며 2를 발견한 곳에 bfs하면 된다. bfs로 주변을 탐색하며 0을 만나지 않았다면 그 2모임은 잡아먹힌 것으로 간주한다. 참고로 1을 만나면 해당 꼬리물기 주변탐색은 중단한다.    

## **#보완해야 할 생각**      

나는 dfs와 bfs를 분리해서 생각했다. 정답 코드에서는 bfs만 돌면됐다.

처음 생각은 바둑을 놓는 경우의 수를 dfs를 해결하려고 했다. 생각해보니 그럴 필요가 없었다. 이중 for문(코드참조)을 돌며 0인 자리를 **배열에 넣어준다.  그 후 이중 for문을 돌며 bfs를 호출한다.** 

**바둑돌을 놓는 구현이 굉장히 간편해진 거다. 구현은 경험만이 커버칠 수 있기 때문에 구현문제에 대해 더 보완해야겠다.**

---

## **#풀이 코드**      

```
import java.util.*;

public class Main {
    static int N, M;
    static int[][] board;
    static int[] dx = {-1, 1, 0, 0};
    static int[] dy = {0, 0, -1, 1};
    static int maxCaptured = 0;

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        N = sc.nextInt();
        M = sc.nextInt();
        board = new int[N][M];

        for (int i = 0; i < N; i++) {
            for (int j = 0; j < M; j++) {
                board[i][j] = sc.nextInt();
            }
        }

        List<int[]> emptyCells = new ArrayList<>();
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < M; j++) {
                if (board[i][j] == 0) {
                    emptyCells.add(new int[]{i, j});
                }
            }
        }

        for (int i = 0; i < emptyCells.size(); i++) {
            for (int j = i + 1; j < emptyCells.size(); j++) {
                int[] cell1 = emptyCells.get(i);
                int[] cell2 = emptyCells.get(j);
                placeStones(cell1[0], cell1[1], cell2[0], cell2[1]);
            }
        }

        System.out.println(maxCaptured);
    }

    static void placeStones(int x1, int y1, int x2, int y2) {
        board[x1][y1] = 1;
        board[x2][y2] = 1;

        boolean[][] visited = new boolean[N][M];
        int captured = 0;

        for (int i = 0; i < N; i++) {
            for (int j = 0; j < M; j++) {
                if (board[i][j] == 2 && !visited[i][j]) {
                    int groupSize = checkGroup(i, j, visited);
                    if (groupSize > 0) {
                        captured += groupSize;
                    }
                }
            }
        }

        maxCaptured = Math.max(maxCaptured, captured);

        board[x1][y1] = 0;
        board[x2][y2] = 0;
    }

    static int checkGroup(int x, int y, boolean[][] visited) {
        Queue<int[]> queue = new LinkedList<>();
        queue.offer(new int[]{x, y});
        visited[x][y] = true;
        int size = 0;
        boolean isCaptured = true;

        while (!queue.isEmpty()) {
            int[] curr = queue.poll();
            size++;

            for (int i = 0; i < 4; i++) {
                int nx = curr[0] + dx[i];
                int ny = curr[1] + dy[i];

                if (nx < 0 || nx >= N || ny < 0 || ny >= M) continue;

                if (board[nx][ny] == 0) {
                    isCaptured = false;
                } else if (board[nx][ny] == 2 && !visited[nx][ny]) {
                    visited[nx][ny] = true;
                    queue.offer(new int[]{nx, ny});
                }
            }
        }

        return isCaptured ? size : 0;
    }
}
```
