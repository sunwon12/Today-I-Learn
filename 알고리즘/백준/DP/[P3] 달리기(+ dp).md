## **#문제**         

**레벨: P3  
알고리즘: bfs**   
**풀이시간: 1시간  
힌트 참조 유무: 유**

[https://www.acmicpc.net/problem/16930](https://www.acmicpc.net/problem/16930)

![image](https://github.com/user-attachments/assets/8d094f12-03fb-4817-a1eb-1688a76241bc)

---

## **#문제 풀이**        

위 문제는 dp + bfs(우선순위 큐) 문제이다. 기존 비슷한 문제와 다른 점은 1초에 1칸 움직이는 것이 아니라 1 ~ K 개수 만큼 이동할 수 있다는 거다. 그래서 우선순위큐를 사용하더라도 해당 위치에 도달하기 까지가 최소 이동횟수가 아닐 수도 있다. 최소이동횟수가 아닐 것을 이용해 계속 탐색하면 의미가 없지 않은가?(시간초과) 그래서 우린 이때, dp를 도입하는 거다. dp를 도입하여 각 칸에 도달한 최소 이동횟수를 적어준다. 그보다 많다면 break해줘 탐색시간을 줄이는 거다.

---

## **#풀이 코드**      

```
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.Arrays;
import java.util.Comparator;
import java.util.LinkedList;
import java.util.Objects;
import java.util.PriorityQueue;
import java.util.Queue;
import java.util.StringTokenizer;
import java.util.function.Function;

public class Main {

    public static void main(String[] args) throws Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(br.readLine()," ");
        Function<String,Integer> stoi = Integer::parseInt;
        int n = stoi.apply(st.nextToken());
        int m = stoi.apply(st.nextToken());
        int k = stoi.apply(st.nextToken());
        char[][] map = new char[n][m];
        for(int i = 0 ; i < n ; i++){
            String command = br.readLine();
            for(int j = 0 ; j < m ; j++){
                map[i][j] = command.charAt(j);
            }
        }
        st = new StringTokenizer(br.readLine()," ");
        int startY = stoi.apply(st.nextToken()) - 1;
        int startX = stoi.apply(st.nextToken()) - 1;
        int endY = stoi.apply(st.nextToken()) - 1;
        int endX = stoi.apply(st.nextToken()) - 1;
        System.out.println(bfs(map,n,m,k,startY,startX,endY,endX));
    }
    private static class Node implements Comparable<Node> {
        int y;
        int x;
        int cnt;

        public Node(int y, int x, int cnt) {
            this.y = y;
            this.x = x;
            this.cnt = cnt;
        }

        @Override
        public int compareTo(Node o) {
            return this.cnt - o.cnt;
        }
    }
    private static int bfs(char[][] map, int n ,int m , int k, int startY, int startX, int endY, int endX) {
        final int dy[] = {-1,0,1,0};
        final int dx[] = {0,1,0,-1};
        final int INF = 987654321;
        Queue<Node> q = new PriorityQueue<>();
        int[][] dp = new int[n][m];
        for(int i = 0 ; i < n ; i++){
            Arrays.fill(dp[i],INF);
        }
        q.offer(new Node(startY,startX,0));
        while(!q.isEmpty()){
            Node now = q.poll();
            if(now.y == endY && now.x == endX){
                return now.cnt;
            }
            for(int i = 0 ; i < 4; i++){
                int ny = now.y;
                int nx = now.x;
                for(int j = 1 ; j <= k ; j++){
                    ny += dy[i];
                    nx += dx[i];
                    if(ny >= 0 && ny < n && nx >= 0 && nx < m){
                        if(map[ny][nx] == '#' || dp[ny][nx] < now.cnt +1){
                            break;
                        }
                        if(dp[ny][nx] == INF){
                            dp[ny][nx] = now.cnt + 1;
                            q.offer(new Node(ny,nx,now.cnt + 1));
                        }
                    }
                }
            }
        }
        return -1;
    }
}
```
