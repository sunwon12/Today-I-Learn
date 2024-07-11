## **#문제**         

**레벨: G3  
알고리즘: dp + dfs**   
**풀이시간: 1시간  
힌트 참조 유무:**

[https://www.acmicpc.net/problem/2533](https://www.acmicpc.net/problem/2533)

![image](https://github.com/sunwon12/Today-I-Learn/assets/92251131/d2d5979f-d768-4b4e-a2ec-835687aa02d1)

---

## **#문제 풀이**        

dp와 dfs를 섞은 문제이다. 이 문제에서 얻을 수 있는 점은 두가지이다.  
첫 째, 그래프를 배열로 표현하는 것.   
둘 째, 큰 문제를 작은 문제로 쪼개는 방법.  
셋 째, vistied배열 대신 parentNode로 판별하는 방법(중복 탐색 예방)  
넷 째, dp 구현방식

위 문제는 dp\[node\]\[0\], dp\[node\]\[1\]를 가지는 이차원 배열로 풀어내야 한다. 가장 아래에 있는 리프 노트부터 시작하여 루트 노드의 dp 값을 채우는 방식이다. 

예를 들면 dp\[5\]\[0\] 은 노드 5가 얼리어답터가 아닐 때이다. 그러니 반드시 부모(코드에서는 양방향이므로 변수명이 child로 되어 있음)가 얼리어답터야 한다.즉, dp\[4\]\[1\]의 값을 더해준다. dp\[5\]\[1\]같은 경우는 노드 4가 얼리어답터가 아니어도 되고 얼리어답토여도 된다. 그러니 그 중 가장 작은 값을 더해준다.    
  dp\[5\]\[1\] += Math.min(dp\[4\]\[0\], dp\[4\]\[1\]);

그러면 이렇게 반문 할 수 있다. 아니, 아직 노드4를 방문도 안 했는데 dp\[4\]\[0\]과 dp\[4\]\[1\]의 값을 어떡해 아나? 그래서 자식 노드를 순회하는 코드 전에 모든 노드의 dp\[node\]\[0\] = 0, dp\[node\]\[1\] = 1을 저장한다. 다르게 말하면 dfs 코드에서 이 코드를 빼고 dfs 함수 호출 전에 dp\[node\]\[0\] = 0, dp\[node\]\[1\] = 1을 해줘도 된다.

```
      1 (8)
     / | \
    2  3  4
 (4) (5) (7)
   / \    / \
  5   6  7   8
(1) (2) (3) (6)
설명:

노드 5 방문 (리프 노드)
노드 6 방문 (리프 노드)
노드 7 방문 (리프 노드)
노드 2 처리 (5와 6을 방문한 후)
노드 3 처리 (리프 노드)
노드 8 방문 (리프 노드)
노드 4 처리 (7과 8을 방문한 후)
노드 1 처리 (루트 노드, 모든 자식 노드 처리 후)
```

  
  

---

## **#풀이 코드**      

```
import java.util.*;
import java.io.*;

public class Main {
    static ArrayList<Integer>[] graph;
    static int[][] dp;
    static int N;

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        N = Integer.parseInt(br.readLine());
        
        graph = new ArrayList[N + 1];
        dp = new int[N + 1][2];
        
        for (int i = 1; i <= N; i++) {
            graph[i] = new ArrayList<>();
        }
        
        for (int i = 0; i < N - 1; i++) {
            StringTokenizer st = new StringTokenizer(br.readLine());
            int u = Integer.parseInt(st.nextToken());
            int v = Integer.parseInt(st.nextToken());
            graph[u].add(v);
            graph[v].add(u);
        }
        
        dfs(1, 0);
        System.out.println(Math.min(dp[1][0], dp[1][1]));
    }
    
    static void dfs(int node, int parent) {
        dp[node][0] = 0; // 현재 노드가 얼리 어답터가 아닌 경우
        dp[node][1] = 1; // 현재 노드가 얼리 어답터인 경우
        
        for (int child : graph[node]) {
            if (child != parent) {
                dfs(child, node);
                dp[node][0] += dp[child][1];
                dp[node][1] += Math.min(dp[child][0], dp[child][1]);
            }
        }
    }
}
```
