## **#문제**         

**레벨: G1  
알고리즘: dp + 브루트 포스 + 비트 마스킹**  
**풀이시간: 1시간  
힌트 참조 유무: 유**

[https://www.acmicpc.net/problem/2098](https://www.acmicpc.net/problem/2098)

![image](https://github.com/user-attachments/assets/4c569500-5bd3-4165-bd99-b27135c28b2a)

---

## **#문제 풀이**        

TSP(Traveling Salesman Problem)

시간 복잡도 O(n^2 \* 2^n)

이 문제는 전형적인 TSP 문제입니다. 한 노드에서 출발하여 모든 노드를 순회하고 다시 시작점으로 돌아오는데 최소값을 구하는 문제가 TSP입니다.   
  
TSP 문제를 가장 무식하게 푸는 방법은 모든 경우의 수를 다 탐색해보는 것입니다. 모든 경우의 수를 탐색해보는 것은 N!인데, 알고리즘에서 가장 나쁜 경우의 수가 N!입니다.   
  
저희는 DP를 사용해야 합니다. 큰 문제를 작은 문제로 쪼갠다. N = 3 일 때를 가정하면 아래와 같은 경우의 수가 있습니다.

1 번을 방문하고 2,3 노드 방문 최솟값  
2 번을 방문하고 1,3 노드 방문 최솟값  
3 번을 방문하고 1,2 노드 방문 최솟값  
  
위 경우의 수를 살펴보면 점화식을 세워볼 수 있습니다. 현재상태에서 남은 노드들을 순회했을 때의 최적값입니다. 이를 코드로 표현하자면 아래와 같습니다. i는 현재위치를 나타내고 visited는 현재까지 방문했던 노드들을 비트 마스크로 표시한 것입니다.(예를 들면, 011은 첫 번째 노드와 두 번째 노드를 방문한 것입니다.)

```
dp[i][visited] = min(dp[i][visited], 
					dp[j][visited | (1 << j) + graph[i][j])
```

![image](https://github.com/user-attachments/assets/69459a10-173c-4085-b30b-88202c393130)

위 그래프로 dp 값 예시를 들겠습니다. 

![image](https://github.com/user-attachments/assets/8675e571-013f-4be5-96a5-82a11c0e78c1)

1번에서부터 시작한다고 했을 때 2가지 방법을 살펴보겠습니다. 근데 5 -> 4 -> 1 경로가 겹치는 것을 볼 수 있습니다. 굳이 가본 적 있는 경로를 한 번 더 가봐야 할까요? 그래서 가본 경로의 값을 저장하고 활용하는 것입니다.

![image](https://github.com/user-attachments/assets/b2a4048c-1027-4e5d-9aba-27b79ebf4ebc)

1 번째 방법에서 탐색하면서 dp에 이렇게 저장을 했다면, 2번 째 방법 탐색 때는 5 -> 4 -> 1을 탐색하지 않고 dp\[5\]\[1011\] = 80 을 이용하면됩니다.

---

## **#풀이 코드**      

```
import java.io.*;
import java.util.*;

public class Main {
    static int N;
    static int[][] cost;
    static int[][] dp;
    static final int INF = 987654321;
    
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        N = Integer.parseInt(br.readLine());
        
        // 비용 행렬 입력
        cost = new int[N][N];
        for(int i = 0; i < N; i++) {
            StringTokenizer st = new StringTokenizer(br.readLine());
            for(int j = 0; j < N; j++) {
                cost[i][j] = Integer.parseInt(st.nextToken());
            }
        }
        
        // dp 배열 초기화
        dp = new int[N][(1 << N)];
        for(int i = 0; i < N; i++) {
            Arrays.fill(dp[i], -1);
        }
        
        // 0번 도시에서 시작
        System.out.println(tsp(0, 1));
    }
    
    static int tsp(int current, int visited) {
        // 모든 도시를 방문했을 때
        if(visited == (1 << N) - 1) {
            // 현재 도시에서 시작 도시(0)로 돌아갈 수 있는 경우
            if(cost[current][0] > 0) {
                return cost[current][0];
            }
            return INF;
        }
        
        // 이미 계산한 경우
        if(dp[current][visited] != -1) {
            return dp[current][visited];
        }
        
        dp[current][visited] = INF;
        
        // 방문하지 않은 다음 도시 탐색
        for(int next = 0; next < N; next++) {
            // 이미 방문했거나 갈 수 없는 경우
            if((visited & (1 << next)) != 0 || cost[current][next] == 0) {
                continue;
            }
            
            // 현재 도시에서 다음 도시까지의 비용 + 다음 도시에서 나머지 도시들을 순회하는 최소 비용
            dp[current][visited] = Math.min(dp[current][visited], 
                                          cost[current][next] + tsp(next, visited | (1 << next)));
        }
        
        return dp[current][visited];
    }
}
```
