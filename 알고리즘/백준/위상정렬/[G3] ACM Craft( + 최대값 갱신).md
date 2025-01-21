### **문제**         

**레벨: G3  
알고리즘: 위상정렬 + 최대값 갱신**   
**풀이시간: 2시간  
힌트 참조 유무: 유**

[https://www.acmicpc.net/problem/1005](https://www.acmicpc.net/problem/1005)

![image](https://github.com/user-attachments/assets/cbc10036-ad12-4700-b283-96e4b576fd22)

### **1 번째 시도:  실패**  

-   입력을 받을 때마다 해당번호의 짓는 시간을 업데이트를 해줬다.
-   결과는 실패
-   이유는 입력 순서가 차례대로일거라고 가정했기 때문이다.

```
import java.util.*;
import java.io.*;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        
        int N = Integer.parseInt(br.readLine());
            
        for (int i = 0; i < N; i++) {
            String[] ss = br.readLine().split(" ");
            int bCnt = Integer.parseInt(ss[0]);
            int rCnt = Integer.parseInt(ss[1]);
            
            int[] arr = new int[bCnt];
            int[] answer = new int[bCnt]; 
            
            String[] s2 = br.readLine().split(" ");
            for (int j = 0; j < bCnt; j++) {
                arr[j] = Integer.parseInt(s2[j]);
            }
            
            answer[0] = arr[0]; 
            for (int j = 0; j < rCnt; j++) {
                String[] s3 = br.readLine().split(" ");
                int first = Integer.parseInt(s3[0]);
                int second = Integer.parseInt(s3[1]);
                int d = answer[first] + arr[second];
                answer[second] = Math.max(answer[second], d);        
            }
            int a = Integer.parseInt(br.readLine());
            System.out.println(answer[a]);
        }
    }
}
```

### **2 번째 시도: 성공** 

-   입력 순서와 상관이 없고 배열의 순서를 알려면 **위상정렬**을 써야한다는 것을 알았다

-   주어진 K개의 건설순서 규칙을 토대로 인접리스트를 만들어주었다. 이때, 각 노드의 진입차수가 몇인지도 indegree배열에 저장해주었다.
-   진입차수가 0인 지점이, 탐색 시작지점이 되므로 queue에 넣어주 었고, dp를 해당 건물 건설시간으로 갱신한다.
-   인접한 노드들을 돌면서dp\[next\]=Math.max(dp\[next\], dp\[cur\]+time\[next\])와 같이 최댓값으로 업데이트 해주고, indegree값을 1감소 시켜준다.
-   중요한 것은, 다음 방문 노드를 무조건 queue에 넣는게 아니라 그 노드의 indegree가 0일때만 queue에 넣고, 탐색을 진행해야 한다. ( 이 방법이 위상정렬이다)

```
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.*;
 
 
public class Main {
 
    static List<Integer>[] list;
    static int n, k;
    static int[] time;
    static int[] degree;
 
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int t = Integer.parseInt(br.readLine());
 
        for (int i = 0; i < t; i++) {
            String[] s1 = br.readLine().split(" ");
            n = Integer.parseInt(s1[0]);
            k = Integer.parseInt(s1[1]);
            time = new int[n + 1];
            degree = new int[n + 1];
            list = new ArrayList[n+1];
            String[] s3 = br.readLine().split(" ");
            for (int j = 1; j <= n; j++) {
                time[j] = Integer.parseInt(s3[j - 1]);
                list[j] = new ArrayList<>();
            }
            for (int j = 0; j < k; j++) {
                String[] s2 = br.readLine().split(" ");
                int x = Integer.parseInt(s2[0]);
                int y = Integer.parseInt(s2[1]);
                list[x].add(y);
                degree[y]++; //y건물을 짓기 위해선 x가 지어져야한다. 따라서 y에 대한 degree를 늘려준다.
            }
            int w = Integer.parseInt(br.readLine());
 
            solve(w);
        }
    }
 
    public static void solve(int target){
        Queue<Integer> queue = new ArrayDeque<>();
        int [] result = new int[n+1];
        for(int i=1; i<=n; i++){
            result[i]=time[i]; // 건물들을 짓는 비용
            if (degree[i]==0){ //제약이 없는 건물은 큐에 넣는다.
                queue.add(i);
            }
        }
 
        while (!queue.isEmpty()){
            int poll = queue.poll();
            for(Integer next: list[poll]){
                result[next]= Math.max(result[next],result[poll]+time[next]); // 방금전 지은 건물을 필요로 하는 건물들의 리스트 이므로
                degree[next]--;                                               // 전 건물을 지은 시간중에 가장 큰 값을 찾아 넣어준다.
                if(degree[next]==0){  // 건물의 조건을 만족시켰다면 큐에 넣어준다. 
                    queue.add(next);
                }
            }
        }
        System.out.println(result[target]);
    }
}
```

### **3 번째 시도: 성공** 

**DFS + 최대값 갱신** 

```
import java.util.*;
import java.io.*;

public class Main {
    static List<Integer>[] reverseGraph;  // 역방향 그래프
    static int[] times;                   // 건설 시간
    static int[] dp;                      // 메모이제이션
    static boolean[] visited;             // 방문 체크
    
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int T = Integer.parseInt(br.readLine());
        
        while (T-- > 0) {
            StringTokenizer st = new StringTokenizer(br.readLine());
            int N = Integer.parseInt(st.nextToken());
            int K = Integer.parseInt(st.nextToken());
            
            // 초기화
            reverseGraph = new ArrayList[N + 1];
            times = new int[N + 1];
            dp = new int[N + 1];
            visited = new boolean[N + 1];
            
            for (int i = 1; i <= N; i++) {
                reverseGraph[i] = new ArrayList<>();
            }
            
            // 건설 시간 입력
            st = new StringTokenizer(br.readLine());
            for (int i = 1; i <= N; i++) {
                times[i] = Integer.parseInt(st.nextToken());
            }
            
            // 역방향 그래프 구축
            for (int i = 0; i < K; i++) {
                st = new StringTokenizer(br.readLine());
                int X = Integer.parseInt(st.nextToken());
                int Y = Integer.parseInt(st.nextToken());
                reverseGraph[Y].add(X);  // 역방향으로 저장
            }
            
            int W = Integer.parseInt(br.readLine());
            System.out.println(dfs(W));
        }
    }
    
    static int dfs(int current) {
        if (visited[current]) {
            return dp[current];
        }
        
        visited[current] = true;
        int maxTime = 0;
        
        // 현재 건물을 짓기 위해 필요한 선행 건물들 탐색
        for (int prev : reverseGraph[current]) {
            maxTime = Math.max(maxTime, dfs(prev));
        }
        
        // 현재 건물 건설 시간 + 선행 건물들 중 가장 오래 걸리는 시간
        dp[current] = maxTime + times[current];
        return dp[current];
    }
}
```

### **글을 마친 후**

위상정렬 + 최대값 갱신, DFS + 최대값 갱신 에서도 그렇게 Math.max로 배열의 값을 갱신하는 부분이 있습니다. 저는 이 부분이 이전의 배열의 값을 이용하기 때문에 DP로 착각하였습니다. **DP가 아니라 최대값 갱신임을 지나고 나서야 깨닫습니다.** 개념이 바르게 잡히지 않아 오해했다고 생각합니다. DP를 정의할 때 배열의 값에 이용이 아닌, 큰 문제를 작은 문제로 나눠 푼다는 것을 다시 한 번 리마인드하겠습니다!
